---
title: "Lab 1: S3 정적 웹사이트 호스팅 + CloudFront CDN"
weight: 1
---

## 목표

S3 버킷에 정적 웹사이트를 호스팅하고, CloudFront로 글로벌 CDN 배포를 구성합니다. 이 과정에서 S3 정적 웹 호스팅과 CloudFront 배포의 차이, Origin Access Control(OAC)을 통한 버킷 비공개 유지, 그리고 콘텐츠 변경 시 캐시 무효화(Invalidation)가 왜 필요한지를 직접 확인합니다.

## 사전 준비

- AWS CLI v2 설치 및 `aws configure`로 자격 증명 설정 완료
- S3 버킷 생성/정책 설정, CloudFront 배포 생성 권한을 가진 IAM 사용자
- 전역적으로 고유한 S3 버킷 이름을 미리 정해두기 (예: `my-saa-lab1-<랜덤문자열>`)

```bash
# 인증 상태와 기본 리전 확인
aws sts get-caller-identity
aws configure get region

# 실습용 변수 설정
export BUCKET_NAME="my-saa-lab1-$(date +%s)"
export REGION="ap-northeast-2"
echo "Bucket: $BUCKET_NAME, Region: $REGION"
```

## 단계별 실행

### 1. S3 버킷 생성 및 정적 웹사이트 호스팅 활성화

```bash
# 버킷 생성 (ap-northeast-2 등 us-east-1 외 리전은 LocationConstraint 필요)
aws s3api create-bucket \
  --bucket "$BUCKET_NAME" \
  --region "$REGION" \
  --create-bucket-configuration LocationConstraint="$REGION"

# 퍼블릭 액세스 차단은 유지한 채로(CloudFront OAC를 쓸 것이므로 버킷은 비공개로 둔다)
aws s3api put-public-access-block \
  --bucket "$BUCKET_NAME" \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

### 2. 샘플 웹페이지 업로드

```bash
mkdir -p /tmp/lab1-site
cat > /tmp/lab1-site/index.html <<'EOF'
<!DOCTYPE html>
<html><head><title>SAA Lab 1</title></head>
<body><h1>S3 + CloudFront 정적 웹사이트 실습</h1></body>
</html>
EOF
cat > /tmp/lab1-site/error.html <<'EOF'
<!DOCTYPE html>
<html><body><h1>404 - Not Found</h1></body></html>
EOF

aws s3 cp /tmp/lab1-site/index.html "s3://$BUCKET_NAME/index.html" --content-type "text/html"
aws s3 cp /tmp/lab1-site/error.html "s3://$BUCKET_NAME/error.html" --content-type "text/html"
```

### 3. CloudFront 배포 생성 (Origin Access Control 사용)

```bash
# OAC 생성
OAC_ID=$(aws cloudfront create-origin-access-control \
  --origin-access-control-config \
  Name="lab1-oac",Description="OAC for lab1 bucket",SigningProtocol=sigv4,SigningBehavior=always,OriginAccessControlOriginType=s3 \
  --query 'OriginAccessControl.Id' --output text)

# 배포 생성 (간단한 구성 파일 사용)
cat > /tmp/lab1-site/distribution-config.json <<EOF
{
  "CallerReference": "lab1-$(date +%s)",
  "Comment": "SAA Lab1 static site",
  "Enabled": true,
  "DefaultRootObject": "index.html",
  "Origins": {
    "Quantity": 1,
    "Items": [{
      "Id": "s3-origin",
      "DomainName": "${BUCKET_NAME}.s3.${REGION}.amazonaws.com",
      "OriginAccessControlId": "${OAC_ID}",
      "S3OriginConfig": { "OriginAccessIdentity": "" }
    }]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "s3-origin",
    "ViewerProtocolPolicy": "redirect-to-https",
    "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6"
  }
}
EOF

DISTRIBUTION_ID=$(aws cloudfront create-distribution \
  --distribution-config file:///tmp/lab1-site/distribution-config.json \
  --query 'Distribution.Id' --output text)
echo "Distribution ID: $DISTRIBUTION_ID"
```

### 4. S3 버킷 정책으로 CloudFront만 접근 허용

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
cat > /tmp/lab1-site/bucket-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowCloudFrontServicePrincipal",
    "Effect": "Allow",
    "Principal": { "Service": "cloudfront.amazonaws.com" },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::${BUCKET_NAME}/*",
    "Condition": {
      "StringEquals": {
        "AWS:SourceArn": "arn:aws:cloudfront::${ACCOUNT_ID}:distribution/${DISTRIBUTION_ID}"
      }
    }
  }]
}
EOF

aws s3api put-bucket-policy --bucket "$BUCKET_NAME" --policy file:///tmp/lab1-site/bucket-policy.json
```

### 5. 배포 완료 확인 후 접속 테스트

```bash
# 배포 상태가 Deployed가 될 때까지 대기 (수 분 소요)
aws cloudfront wait distribution-deployed --id "$DISTRIBUTION_ID"

DOMAIN=$(aws cloudfront get-distribution --id "$DISTRIBUTION_ID" --query 'Distribution.DomainName' --output text)
echo "https://$DOMAIN"
curl -I "https://$DOMAIN"
```

### 6. 콘텐츠 수정 후 캐시 무효화 실습

```bash
# index.html 수정 후 재업로드
sed -i.bak 's/실습/실습 (수정됨)/' /tmp/lab1-site/index.html
aws s3 cp /tmp/lab1-site/index.html "s3://$BUCKET_NAME/index.html" --content-type "text/html"

# 무효화 없이 다시 요청하면 캐시된 이전 버전이 보일 수 있음
curl -s "https://$DOMAIN" | grep -i "실습"

# 캐시 무효화 생성
aws cloudfront create-invalidation --distribution-id "$DISTRIBUTION_ID" --paths "/index.html"

# 무효화 완료 후 다시 요청하면 수정된 버전이 보임
curl -s "https://$DOMAIN" | grep -i "실습"
```

## 결과 확인

- CloudFront 도메인으로 접속했을 때 S3에 업로드한 페이지가 보이는지 확인합니다.
- S3 버킷을 직접 퍼블릭으로 열지 않았는데도(`BlockPublicAcls=true` 유지) CloudFront를 통해서는 콘텐츠가 정상적으로 보이는 이유는, OAC를 통해 CloudFront 서비스 주체에게만 `s3:GetObject` 권한을 부여했기 때문입니다.
- 콘텐츠를 수정한 직후 무효화 없이 요청하면 **이전 캐시된 버전**이 그대로 보이고, `create-invalidation` 실행 후에는 새 버전이 반영되는 것을 비교해 캐시 동작을 체감합니다.

## 체크리스트

- [ ] S3 버킷이 퍼블릭 액세스 차단 설정을 유지한 채로 생성되었다
- [ ] CloudFront 배포가 OAC를 통해 S3 버킷에만 접근 권한을 부여한다
- [ ] CloudFront 도메인을 통해 정적 페이지가 정상적으로 렌더링된다
- [ ] 콘텐츠 변경 후 캐시 무효화 전/후의 응답 차이를 직접 확인했다
- [ ] 실습 종료 후 CloudFront 배포를 비활성화하고 S3 버킷을 삭제했다 (`aws s3 rb s3://$BUCKET_NAME --force`)

다음 실습인 **[Lab 2: EC2 + RDS 웹앱](../lab2-ec2-rds-webapp/)** 에서는 정적 콘텐츠를 넘어 동적 웹 애플리케이션을 직접 배포합니다.
