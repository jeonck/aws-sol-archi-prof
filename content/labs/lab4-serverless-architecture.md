---
title: "Lab 4: 서버리스 REST API"
weight: 4
---

## 목표

API Gateway + Lambda + DynamoDB + S3로 완전 관리형 서버리스 REST API를 구축합니다. EC2/RDS 기반 아키텍처(Lab 2, 3)와 비교했을 때 서버 프로비저닝이나 Auto Scaling 설정 없이도 트래픽에 따라 자동으로 확장되는 구조를 직접 체감하는 것이 핵심입니다.

## 사전 준비

- Lambda, API Gateway, DynamoDB, S3, IAM 역할 생성 권한
- Lambda 함수 패키징을 위한 `zip` 명령어
- 실습용 함수 코드를 작성할 간단한 텍스트 에디터(여기서는 heredoc으로 인라인 작성)

```bash
aws sts get-caller-identity
export REGION="ap-northeast-2"
export FUNC_NAME="lab4-items-api"
export TABLE_NAME="lab4-items"
export ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
```

## 단계별 실행

### 1. DynamoDB 테이블 생성

```bash
aws dynamodb create-table \
  --table-name "$TABLE_NAME" \
  --attribute-definitions AttributeName=itemId,AttributeType=S \
  --key-schema AttributeName=itemId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST

aws dynamodb wait table-exists --table-name "$TABLE_NAME"
```

### 2. Lambda 실행 역할(IAM Role) 생성

```bash
cat > /tmp/lab4-trust-policy.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole" }]
}
EOF

aws iam create-role --role-name lab4-lambda-role --assume-role-policy-document file:///tmp/lab4-trust-policy.json
aws iam attach-role-policy --role-name lab4-lambda-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
aws iam attach-role-policy --role-name lab4-lambda-role --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess

ROLE_ARN="arn:aws:iam::${ACCOUNT_ID}:role/lab4-lambda-role"
sleep 10  # IAM 역할 전파 대기
```

### 3. Lambda 함수 작성 및 배포

```bash
mkdir -p /tmp/lab4-lambda
cat > /tmp/lab4-lambda/index.py <<'EOF'
import json, os, uuid, boto3

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def handler(event, context):
    method = event.get('httpMethod', event.get('requestContext', {}).get('http', {}).get('method'))
    if method == 'POST':
        body = json.loads(event['body'])
        item_id = str(uuid.uuid4())
        table.put_item(Item={'itemId': item_id, 'name': body.get('name', 'unnamed')})
        return {'statusCode': 201, 'body': json.dumps({'itemId': item_id})}
    if method == 'GET':
        items = table.scan().get('Items', [])
        return {'statusCode': 200, 'body': json.dumps(items)}
    return {'statusCode': 400, 'body': json.dumps({'error': 'unsupported method'})}
EOF

cd /tmp/lab4-lambda && zip function.zip index.py && cd -

aws lambda create-function \
  --function-name "$FUNC_NAME" \
  --runtime python3.12 \
  --role "$ROLE_ARN" \
  --handler index.handler \
  --zip-file fileb:///tmp/lab4-lambda/function.zip \
  --environment "Variables={TABLE_NAME=$TABLE_NAME}" \
  --timeout 10
```

### 4. API Gateway(HTTP API) 생성 및 Lambda 연결

```bash
API_ID=$(aws apigatewayv2 create-api \
  --name lab4-api --protocol-type HTTP \
  --target "arn:aws:lambda:${REGION}:${ACCOUNT_ID}:function:${FUNC_NAME}" \
  --query 'ApiId' --output text)

# Lambda가 API Gateway 호출을 허용하도록 권한 부여
aws lambda add-permission \
  --function-name "$FUNC_NAME" \
  --statement-id apigw-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:${REGION}:${ACCOUNT_ID}:${API_ID}/*/*"

API_ENDPOINT=$(aws apigatewayv2 get-api --api-id "$API_ID" --query 'ApiEndpoint' --output text)
echo "API Endpoint: $API_ENDPOINT"
```

### 5. S3 정적 프론트엔드 연결 (선택, Lab 1과 동일 패턴)

```bash
# Lab 1에서 만든 정적 호스팅 패턴을 재사용해 프론트엔드에서 $API_ENDPOINT 를 호출하도록 구성할 수 있습니다.
echo "프론트엔드에서 fetch('${API_ENDPOINT}/items')와 같이 호출"
```

### 6. API 테스트

```bash
# 아이템 생성 (POST)
curl -X POST "$API_ENDPOINT/" -d '{"name":"first item"}'

# 아이템 조회 (GET)
curl "$API_ENDPOINT/"
```

## 결과 확인

- POST 요청 후 응답으로 `itemId`가 반환되는지 확인합니다.
- GET 요청 시 DynamoDB에 저장된 항목 목록이 JSON으로 반환되는지 확인합니다.
- `aws dynamodb scan --table-name "$TABLE_NAME"`으로 DynamoDB에 실제로 데이터가 저장되었는지 직접 확인합니다.
- 동시에 여러 요청을 보내도(`for i in {1..20}; do curl ... & done`) 별도의 용량 설정 없이 Lambda가 동시 실행되는 것을 관찰하며, EC2 기반 아키텍처와의 확장 방식 차이를 체감합니다.

## 체크리스트

- [ ] DynamoDB 테이블이 On-Demand 모드로 생성되어 있다
- [ ] Lambda 함수가 IAM 역할을 통해 DynamoDB에 접근할 수 있다 (최소 권한 원칙을 적용하려면 `AmazonDynamoDBFullAccess` 대신 테이블 단위 정책으로 교체)
- [ ] API Gateway를 통해 POST/GET 요청이 Lambda로 정상 라우팅된다
- [ ] DynamoDB에 실제 데이터가 저장되고 조회된다
- [ ] 실습 종료 후 API Gateway, Lambda, IAM 역할, DynamoDB 테이블을 모두 삭제했다

다음 실습인 **[Lab 5: 멀티 계정 거버넌스](../lab5-multi-account-organizations/)** 에서는 단일 워크로드를 넘어 조직 전체의 계정 구조를 다룹니다.
