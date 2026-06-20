---
title: "Lab 5: 멀티 계정 거버넌스 (AWS Organizations + SCP)"
weight: 5
---

## 목표

AWS Organizations로 멀티 계정 구조(관리 계정 + 멤버 계정 + OU)를 만들고, Service Control Policy(SCP)로 특정 리전 사용을 제한하거나 특정 서비스 사용을 금지하는 거버넌스를 구성합니다. 이 실습은 SAP-C02의 "조직 복잡성 관리" 도메인에서 다루는 핵심 패턴을 직접 다룹니다.

{{< callout type="warning" >}}
이 실습은 **관리 계정(Management Account)** 권한이 필요하며, 신규 멤버 계정을 생성하면 해당 계정도 별도의 AWS 계정으로 간주되어 결제 책임이 발생합니다(사용량이 없으면 대부분 무료지만 일부 리소스는 과금될 수 있음). 회사 계정이 아니라 개인 테스트용 계정에서 진행하고, 실습 후 정리하세요.
{{< /callout >}}

## 사전 준비

- AWS Organizations를 활성화할 수 있는 관리 계정 권한
- 멤버 계정 생성에 사용할 유효한 이메일 주소 (계정마다 고유해야 함)
- Organizations, SCP 관리 권한

```bash
aws sts get-caller-identity
# 관리 계정이 맞는지 확인 (Organizations가 비활성 상태라면 이 명령은 에러를 반환함)
aws organizations describe-organization 2>&1 || echo "Organizations 미활성 상태 - 다음 단계에서 생성"
```

## 단계별 실행

### 1. AWS Organizations 활성화

```bash
aws organizations create-organization --feature-set ALL

ORG_ROOT_ID=$(aws organizations list-roots --query 'Roots[0].Id' --output text)
echo "Root OU ID: $ORG_ROOT_ID"
```

### 2. OU(Organizational Unit) 구조 생성

```bash
# 워크로드 환경별로 OU 분리 (Sandbox / Production 패턴)
SANDBOX_OU_ID=$(aws organizations create-organizational-unit \
  --parent-id "$ORG_ROOT_ID" --name "Sandbox" \
  --query 'OrganizationalUnit.Id' --output text)

PROD_OU_ID=$(aws organizations create-organizational-unit \
  --parent-id "$ORG_ROOT_ID" --name "Production" \
  --query 'OrganizationalUnit.Id' --output text)

echo "Sandbox OU: $SANDBOX_OU_ID, Production OU: $PROD_OU_ID"
```

### 3. 멤버 계정 생성 및 Sandbox OU로 이동

```bash
CREATE_REQUEST_ID=$(aws organizations create-account \
  --email "lab5-sandbox-account@example.com" \
  --account-name "Lab5-Sandbox" \
  --query 'CreateAccountStatus.Id' --output text)

# 계정 생성 완료 대기 (수 분 소요)
while true; do
  STATUS=$(aws organizations describe-create-account-status \
    --create-account-request-id "$CREATE_REQUEST_ID" \
    --query 'CreateAccountStatus.State' --output text)
  echo "Status: $STATUS"
  [ "$STATUS" != "IN_PROGRESS" ] && break
  sleep 15
done

MEMBER_ACCOUNT_ID=$(aws organizations describe-create-account-status \
  --create-account-request-id "$CREATE_REQUEST_ID" \
  --query 'CreateAccountStatus.AccountId' --output text)

aws organizations move-account \
  --account-id "$MEMBER_ACCOUNT_ID" \
  --source-parent-id "$ORG_ROOT_ID" \
  --destination-parent-id "$SANDBOX_OU_ID"
```

### 4. SCP 활성화 및 리전 제한 정책 생성

```bash
aws organizations enable-policy-type --root-id "$ORG_ROOT_ID" --policy-type SERVICE_CONTROL_POLICY

cat > /tmp/lab5-region-restrict-scp.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyOutsideApprovedRegions",
    "Effect": "Deny",
    "NotAction": [
      "iam:*", "organizations:*", "route53:*", "support:*", "cloudfront:*", "budgets:*"
    ],
    "Resource": "*",
    "Condition": {
      "StringNotEquals": { "aws:RequestedRegion": ["ap-northeast-2", "us-east-1"] }
    }
  }]
}
EOF

REGION_SCP_ID=$(aws organizations create-policy \
  --name "RestrictToApprovedRegions" \
  --description "Deny all actions outside ap-northeast-2 and us-east-1" \
  --type SERVICE_CONTROL_POLICY \
  --content file:///tmp/lab5-region-restrict-scp.json \
  --query 'Policy.PolicySummary.Id' --output text)

aws organizations attach-policy --policy-id "$REGION_SCP_ID" --target-id "$SANDBOX_OU_ID"
```

### 5. 특정 서비스 사용 금지 SCP 생성 (예: EC2의 특정 인스턴스 타입 제한)

```bash
cat > /tmp/lab5-deny-large-instances-scp.json <<'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyExpensiveInstanceTypes",
    "Effect": "Deny",
    "Action": "ec2:RunInstances",
    "Resource": "arn:aws:ec2:*:*:instance/*",
    "Condition": {
      "ForAnyValue:StringNotLike": { "ec2:InstanceType": ["t2.*", "t3.*", "t3a.*"] }
    }
  }]
}
EOF

COST_SCP_ID=$(aws organizations create-policy \
  --name "RestrictInstanceTypesToBurstable" \
  --description "Deny launching non-burstable (expensive) EC2 instance types" \
  --type SERVICE_CONTROL_POLICY \
  --content file:///tmp/lab5-deny-large-instances-scp.json \
  --query 'Policy.PolicySummary.Id' --output text)

aws organizations attach-policy --policy-id "$COST_SCP_ID" --target-id "$SANDBOX_OU_ID"
```

### 6. SCP 효과 검증 (멤버 계정 역할로 전환 후 시도)

```bash
# 멤버 계정의 OrganizationAccountAccessRole로 전환
CREDS=$(aws sts assume-role \
  --role-arn "arn:aws:iam::${MEMBER_ACCOUNT_ID}:role/OrganizationAccountAccessRole" \
  --role-session-name lab5-verify --query 'Credentials' --output json)

export AWS_ACCESS_KEY_ID=$(echo "$CREDS" | jq -r '.AccessKeyId')
export AWS_SECRET_ACCESS_KEY=$(echo "$CREDS" | jq -r '.SecretAccessKey')
export AWS_SESSION_TOKEN=$(echo "$CREDS" | jq -r '.SessionToken')

# 허용되지 않은 리전(예: eu-west-1)에서 리소스 생성 시도 -> AccessDenied 예상
aws ec2 describe-vpcs --region eu-west-1 2>&1 | grep -i "AccessDenied" && echo "SCP가 정상적으로 차단함"

# 자격 증명 원복
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

## 결과 확인

- 멤버 계정 역할로 전환한 상태에서 허용 리전 외 API 호출 시 `AccessDenied`가 반환되는지 확인합니다. SCP는 **명시적 허용을 추가하지 않고, 오직 권한의 상한선**(허용 범위를 깎는 역할)으로만 동작한다는 점이 핵심입니다 — IAM 정책에서 권한이 있어도 SCP가 막으면 거부됩니다.
- `aws organizations list-policies-for-target --target-id "$SANDBOX_OU_ID" --filter SERVICE_CONTROL_POLICY`로 OU에 정책이 정상적으로 부착되었는지 확인합니다.
- 같은 계정에서 허용된 리전(`ap-northeast-2`)에서는 정상적으로 API가 호출되는지 비교합니다.

## 체크리스트

- [ ] Organizations가 활성화되고 최소 1개의 멤버 계정이 OU에 속해 있다
- [ ] 리전 제한 SCP가 Sandbox OU에 부착되어 있다
- [ ] 인스턴스 타입 제한 SCP가 Sandbox OU에 부착되어 있다
- [ ] 멤버 계정에서 허용되지 않은 리전 호출 시 차단됨을 직접 확인했다
- [ ] 실습 종료 후 SCP를 분리(detach)하고 정책을 삭제했으며, 더 이상 필요 없는 멤버 계정은 Organizations에서 제거 절차(계정 폐쇄)를 진행했다

다음 실습인 **[Lab 6: 멀티 리전 DR](../lab6-disaster-recovery-multiregion/)** 에서는 조직이 아니라 워크로드 자체의 재해 복구 전략을 다룹니다.
