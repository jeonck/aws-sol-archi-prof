---
title: "Lab 2: EC2 + RDS 2-Tier 웹앱 배포"
weight: 2
---

## 목표

EC2 인스턴스 위에서 동작하는 웹 애플리케이션과 RDS(MySQL) 데이터베이스로 구성된 전통적인 2-Tier 아키텍처를 배포합니다. 핵심은 Security Group을 계층별로 분리해 **EC2에서만 RDS로 접근 가능하도록 제한**하는 것이며, 이는 SAA-C03 보안 도메인에서 가장 자주 등장하는 패턴입니다.

## 사전 준비

- VPC, EC2, RDS, Security Group 생성 권한을 가진 IAM 사용자
- 기본 VPC(Default VPC) 사용 가능 또는 직접 생성한 VPC와 최소 2개의 서브넷(RDS는 서브넷 그룹에 2개 AZ 이상 필요)
- EC2 접속용 키 페어

```bash
aws sts get-caller-identity
export REGION="ap-northeast-2"
export VPC_ID=$(aws ec2 describe-vpcs --filters Name=is-default,Values=true --query 'Vpcs[0].VpcId' --output text)
echo "Default VPC: $VPC_ID"

# 키 페어 생성 (이미 있다면 생략)
aws ec2 create-key-pair --key-name lab2-key --query 'KeyMaterial' --output text > ~/.ssh/lab2-key.pem
chmod 400 ~/.ssh/lab2-key.pem
```

## 단계별 실행

### 1. Security Group 2개 생성 — Web용, DB용

```bash
# 웹 서버용 SG: 80/443/22 인바운드 허용
WEB_SG_ID=$(aws ec2 create-security-group \
  --group-name lab2-web-sg --description "Web tier SG" --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress --group-id "$WEB_SG_ID" --protocol tcp --port 22 --cidr "$(curl -s ifconfig.me)/32"
aws ec2 authorize-security-group-ingress --group-id "$WEB_SG_ID" --protocol tcp --port 80 --cidr "0.0.0.0/0"

# DB용 SG: 3306 포트는 오직 Web SG로부터만 허용 (CIDR 아님 — SG를 소스로 지정)
DB_SG_ID=$(aws ec2 create-security-group \
  --group-name lab2-db-sg --description "DB tier SG" --vpc-id "$VPC_ID" \
  --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress \
  --group-id "$DB_SG_ID" --protocol tcp --port 3306 \
  --source-group "$WEB_SG_ID"

echo "WEB_SG: $WEB_SG_ID, DB_SG: $DB_SG_ID"
```

### 2. RDS MySQL 인스턴스 생성

```bash
# 기본 VPC의 서브넷 2개 이상으로 서브넷 그룹 생성
SUBNET_IDS=$(aws ec2 describe-subnets --filters Name=vpc-id,Values="$VPC_ID" --query 'Subnets[*].SubnetId' --output text)
aws rds create-db-subnet-group \
  --db-subnet-group-name lab2-subnet-group \
  --db-subnet-group-description "Lab2 subnet group" \
  --subnet-ids $SUBNET_IDS

aws rds create-db-instance \
  --db-instance-identifier lab2-mysql \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password 'ChangeMe12345!' \
  --allocated-storage 20 \
  --vpc-security-group-ids "$DB_SG_ID" \
  --db-subnet-group-name lab2-subnet-group \
  --no-publicly-accessible \
  --backup-retention-period 1

# 생성 완료까지 대기 (수 분 소요)
aws rds wait db-instance-available --db-instance-identifier lab2-mysql
RDS_ENDPOINT=$(aws rds describe-db-instances --db-instance-identifier lab2-mysql \
  --query 'DBInstances[0].Endpoint.Address' --output text)
echo "RDS Endpoint: $RDS_ENDPOINT"
```

### 3. EC2 인스턴스 생성 및 웹 서버/DB 클라이언트 설치 (User Data)

```bash
cat > /tmp/lab2-userdata.sh <<'EOF'
#!/bin/bash
dnf update -y
dnf install -y httpd mariadb105
systemctl enable --now httpd
echo "<h1>Lab2 EC2 Web Server - $(hostname)</h1>" > /var/www/html/index.html
EOF

AMI_ID=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
  --query 'Parameter.Value' --output text)

INSTANCE_ID=$(aws ec2 run-instances \
  --image-id "$AMI_ID" \
  --instance-type t3.micro \
  --key-name lab2-key \
  --security-group-ids "$WEB_SG_ID" \
  --user-data file:///tmp/lab2-userdata.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=lab2-web}]' \
  --query 'Instances[0].InstanceId' --output text)

aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"
PUBLIC_IP=$(aws ec2 describe-instances --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' --output text)
echo "EC2 Public IP: $PUBLIC_IP"
```

### 4. EC2에서 RDS 연결 테스트

```bash
ssh -i ~/.ssh/lab2-key.pem ec2-user@"$PUBLIC_IP" \
  "mysql -h $RDS_ENDPOINT -u admin -p'ChangeMe12345!' -e 'SELECT VERSION();'"
```

### 5. Security Group 격리 검증 — 로컬에서 RDS 직접 접근 시도 (차단 확인)

```bash
# 로컬 머신에서 RDS 포트로 직접 접근 시도 — Security Group이 막아 타임아웃 발생해야 정상
timeout 5 bash -c "echo > /dev/tcp/$RDS_ENDPOINT/3306" && echo "연결 성공 (예상치 못한 결과)" || echo "연결 차단됨 (정상 - Security Group이 EC2 외 접근을 막고 있음)"
```

## 결과 확인

- 웹 브라우저나 `curl http://$PUBLIC_IP`로 EC2의 Apache 페이지가 정상적으로 응답하는지 확인합니다.
- EC2 인스턴스 내부에서는 `mysql` 클라이언트로 RDS에 정상 접속되지만, 로컬 머신(또는 EC2가 아닌 다른 곳)에서는 RDS 포트(3306)로 연결이 차단되는지 비교합니다. 이 차이가 바로 **Security Group을 소스로 지정**(`--source-group`)했을 때의 효과입니다. DB SG에 CIDR(예: `0.0.0.0/0`)이 아니라 SG ID를 소스로 지정하면, 그 SG가 붙은 인스턴스에서 나온 트래픽만 허용됩니다.

## 체크리스트

- [ ] Web용 SG와 DB용 SG가 분리되어 있고, DB SG의 인바운드 소스가 CIDR이 아니라 Web SG ID로 지정되어 있다
- [ ] RDS 인스턴스가 `--no-publicly-accessible`로 생성되어 퍼블릭 접근이 차단되어 있다
- [ ] EC2에서는 RDS에 정상 접속되지만 외부에서는 직접 접근이 차단됨을 확인했다
- [ ] 웹 서버가 정상적으로 응답한다
- [ ] 실습 종료 후 EC2 인스턴스, RDS 인스턴스, Security Group을 모두 삭제했다

다음 실습인 **[Lab 3: 3-Tier 고가용성 아키텍처](../lab3-three-tier-architecture/)** 에서는 이 2-Tier 구조를 Auto Scaling, Multi-AZ, ALB로 확장합니다.
