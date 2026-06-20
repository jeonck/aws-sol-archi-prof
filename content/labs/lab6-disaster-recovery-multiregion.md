---
title: "Lab 6: 멀티 리전 DR (S3 CRR + Route 53 Failover)"
weight: 6
---

## 목표

S3 Cross-Region Replication(CRR)으로 콘텐츠를 보조 리전에 자동 복제하고, Route 53 Failover Routing으로 기본 리전 장애 시 보조 리전으로 트래픽을 전환하는 간단한 Multi-Region DR 시나리오를 구성합니다. 장애를 시뮬레이션해 실제 RTO(복구 목표 시간)와 RPO(복구 목표 시점)를 측정하는 것이 이 실습의 핵심입니다.

## 사전 준비

- 2개 이상의 리전에서 S3, Route 53 Health Check, CRR 설정 권한
- Route 53에 등록된 호스팅 영역(테스트 도메인) — 실제 도메인이 없다면 Health Check와 Failover 레코드 설정까지만 진행하고 DNS 전파 확인은 `dig`로 대체
- CRR을 위한 버킷 버저닝 활성화 필요(필수 조건)

```bash
aws sts get-caller-identity
export PRIMARY_REGION="ap-northeast-2"
export SECONDARY_REGION="ap-southeast-1"
export PRIMARY_BUCKET="lab6-primary-$(date +%s)"
export SECONDARY_BUCKET="lab6-secondary-$(date +%s)"
```

## 단계별 실행

### 1. 두 리전에 S3 버킷 생성 및 버저닝 활성화

```bash
aws s3api create-bucket --bucket "$PRIMARY_BUCKET" --region "$PRIMARY_REGION" \
  --create-bucket-configuration LocationConstraint="$PRIMARY_REGION"
aws s3api create-bucket --bucket "$SECONDARY_BUCKET" --region "$SECONDARY_REGION" \
  --create-bucket-configuration LocationConstraint="$SECONDARY_REGION"

aws s3api put-bucket-versioning --bucket "$PRIMARY_BUCKET" --versioning-configuration Status=Enabled
aws s3api put-bucket-versioning --bucket "$SECONDARY_BUCKET" --versioning-configuration Status=Enabled
```

### 2. CRR용 IAM 역할 생성

```bash
cat > /tmp/lab6-crr-trust.json <<'EOF'
{ "Version": "2012-10-17", "Statement": [{ "Effect": "Allow", "Principal": {"Service": "s3.amazonaws.com"}, "Action": "sts:AssumeRole" }] }
EOF
aws iam create-role --role-name lab6-crr-role --assume-role-policy-document file:///tmp/lab6-crr-trust.json
aws iam attach-role-policy --role-name lab6-crr-role --policy-arn arn:aws:iam::aws:policy/service-role/AmazonS3FullAccess
ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
CRR_ROLE_ARN="arn:aws:iam::${ACCOUNT_ID}:role/lab6-crr-role"
sleep 10
```

### 3. Cross-Region Replication 규칙 설정

```bash
cat > /tmp/lab6-replication.json <<EOF
{
  "Role": "${CRR_ROLE_ARN}",
  "Rules": [{
    "ID": "lab6-replication-rule",
    "Status": "Enabled",
    "Priority": 1,
    "Filter": {},
    "DeleteMarkerReplication": { "Status": "Disabled" },
    "Destination": { "Bucket": "arn:aws:s3:::${SECONDARY_BUCKET}" }
  }]
}
EOF

aws s3api put-bucket-replication --bucket "$PRIMARY_BUCKET" --replication-configuration file:///tmp/lab6-replication.json
```

### 4. 복제 동작 확인 — RPO 측정

```bash
echo "<h1>Primary region content - $(date)</h1>" > /tmp/lab6-index.html
START_TIME=$(date +%s)
aws s3 cp /tmp/lab6-index.html "s3://$PRIMARY_BUCKET/index.html"

# 보조 리전에 복제될 때까지 폴링 (RPO에 해당하는 지연 시간 측정)
while ! aws s3api head-object --bucket "$SECONDARY_BUCKET" --key index.html --region "$SECONDARY_REGION" 2>/dev/null; do
  sleep 5
done
END_TIME=$(date +%s)
echo "복제 완료까지 걸린 시간(RPO 근사값): $((END_TIME - START_TIME))초"
```

### 5. 각 리전을 정적 웹사이트로 활성화하고 Route 53 Health Check + Failover 레코드 구성

```bash
aws s3 website "s3://$PRIMARY_BUCKET" --index-document index.html
aws s3 website "s3://$SECONDARY_BUCKET" --index-document index.html

# Primary 엔드포인트에 대한 Health Check 생성
HEALTH_CHECK_ID=$(aws route53 create-health-check \
  --caller-reference "lab6-$(date +%s)" \
  --health-check-config "Type=HTTP,ResourcePath=/index.html,FullyQualifiedDomainName=${PRIMARY_BUCKET}.s3-website.${PRIMARY_REGION}.amazonaws.com,Port=80,RequestInterval=30,FailureThreshold=3" \
  --query 'HealthCheck.Id' --output text)

HOSTED_ZONE_ID="<보유한 호스팅 영역 ID>"
cat > /tmp/lab6-primary-record.json <<EOF
{
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "dr-lab.example.com",
      "Type": "CNAME",
      "SetIdentifier": "primary",
      "Failover": "PRIMARY",
      "TTL": 60,
      "ResourceRecords": [{ "Value": "${PRIMARY_BUCKET}.s3-website.${PRIMARY_REGION}.amazonaws.com" }],
      "HealthCheckId": "${HEALTH_CHECK_ID}"
    }
  }]
}
EOF
cat > /tmp/lab6-secondary-record.json <<EOF
{
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "dr-lab.example.com",
      "Type": "CNAME",
      "SetIdentifier": "secondary",
      "Failover": "SECONDARY",
      "TTL": 60,
      "ResourceRecords": [{ "Value": "${SECONDARY_BUCKET}.s3-website.${SECONDARY_REGION}.amazonaws.com" }]
    }
  }]
}
EOF
aws route53 change-resource-record-sets --hosted-zone-id "$HOSTED_ZONE_ID" --change-batch file:///tmp/lab6-primary-record.json
aws route53 change-resource-record-sets --hosted-zone-id "$HOSTED_ZONE_ID" --change-batch file:///tmp/lab6-secondary-record.json
```

### 6. 장애 시뮬레이션 및 RTO 측정

```bash
# Primary 버킷의 웹사이트 설정을 제거해 장애 상태를 시뮬레이션
FAILOVER_START=$(date +%s)
aws s3api delete-bucket-website --bucket "$PRIMARY_BUCKET"

# Health Check가 Unhealthy로 전환되고 DNS가 Secondary로 failover 될 때까지 대기
while [ "$(aws route53 get-health-check-status --health-check-id "$HEALTH_CHECK_ID" \
  --query 'HealthCheckObservations[0].StatusReport.Status' --output text | grep -o 'Success\|Failure')" != "Failure" ]; do
  sleep 10
  echo "대기 중... ($(($(date +%s) - FAILOVER_START))초 경과)"
done
FAILOVER_END=$(date +%s)
echo "Health Check 장애 감지까지 걸린 시간(RTO 구성 요소): $((FAILOVER_END - FAILOVER_START))초"

dig dr-lab.example.com CNAME +short
```

## 결과 확인

- RPO 측정값(4단계)은 CRR이 비동기 복제이기 때문에 0이 아니며, 일반적으로 수 초~수 분 수준입니다. 이 값이 비즈니스 요구사항(RPO 목표)을 만족하는지가 DR 전략 선택의 기준이 됩니다.
- RTO 측정값(6단계)은 Health Check의 `FailureThreshold`(이 실습에서는 3회 연속 실패, 간격 30초이므로 최소 90초 이상)와 DNS TTL(60초)의 합으로 결정됩니다. TTL과 FailureThreshold를 줄이면 RTO는 짧아지지만, 일시적 네트워크 흔들림에도 불필요하게 failover가 발생할 위험이 커지는 trade-off가 있습니다.
- `dig` 결과가 Secondary 버킷의 웹사이트 엔드포인트로 바뀌었는지 확인합니다.

## 체크리스트

- [ ] 두 리전의 S3 버킷에 버저닝이 활성화되어 있고 CRR이 정상 동작한다
- [ ] Primary에 업로드한 객체가 Secondary에 자동 복제되는 데 걸리는 시간(RPO)을 직접 측정했다
- [ ] Route 53 Health Check와 Failover Routing 레코드가 구성되어 있다
- [ ] Primary 장애 시뮬레이션 후 Secondary로 전환되는 데 걸리는 시간(RTO)을 직접 측정했다
- [ ] 측정한 RTO/RPO와 TTL/FailureThreshold 설정의 trade-off 관계를 설명할 수 있다
- [ ] 실습 종료 후 Route 53 레코드, Health Check, 두 리전의 S3 버킷, IAM 역할을 모두 삭제했다 (CRR이 적용된 버킷은 모든 버전을 먼저 삭제해야 버킷 삭제가 가능함)

모든 실습을 완료했다면 **[실습 목록](../)** 으로 돌아가 전체 아키텍처 흐름(정적 사이트 → 2-Tier → 3-Tier → 서버리스 → 멀티 계정 → DR)을 복습하거나, **[Practice Exam 전략](../../docs/exam-prep/practice-exam-strategy/)** 으로 이동해 시험 준비를 이어가세요.
