---
title: "Lab 3: 고가용성 3-Tier 아키텍처"
weight: 3
---

## 목표

VPC 내에 Public/Private 서브넷을 분리하고, ALB(Application Load Balancer) + Auto Scaling Group으로 웹 계층을 구성하며, RDS Multi-AZ로 데이터 계층의 고가용성을 확보하는 전형적인 3-Tier 아키텍처를 구축합니다. Route 53으로 도메인을 ALB에 연결해 SAA-C03 복원력(Resilient Architectures) 도메인의 핵심 패턴을 직접 경험합니다.

## 사전 준비

- VPC, 서브넷, ALB, Auto Scaling, RDS, Route 53 생성 권한
- 보유 중인(또는 테스트용) Route 53 호스팅 영역 — 없다면 ALB 단계까지만 진행하고 Route 53 단계는 건너뛰어도 무방
- Lab 2에서 사용한 AMI/User Data 스크립트 재사용 가능

```bash
aws sts get-caller-identity
export REGION="ap-northeast-2"
export AZ1="${REGION}a"
export AZ2="${REGION}c"
```

## 단계별 실행

### 1. VPC와 4개 서브넷(Public x2, Private x2) 생성

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)
aws ec2 create-tags --resources "$VPC_ID" --tags Key=Name,Value=lab3-vpc

IGW_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id "$VPC_ID" --internet-gateway-id "$IGW_ID"

PUB_SUB1=$(aws ec2 create-subnet --vpc-id "$VPC_ID" --cidr-block 10.0.1.0/24 --availability-zone "$AZ1" --query 'Subnet.SubnetId' --output text)
PUB_SUB2=$(aws ec2 create-subnet --vpc-id "$VPC_ID" --cidr-block 10.0.2.0/24 --availability-zone "$AZ2" --query 'Subnet.SubnetId' --output text)
PRIV_SUB1=$(aws ec2 create-subnet --vpc-id "$VPC_ID" --cidr-block 10.0.11.0/24 --availability-zone "$AZ1" --query 'Subnet.SubnetId' --output text)
PRIV_SUB2=$(aws ec2 create-subnet --vpc-id "$VPC_ID" --cidr-block 10.0.12.0/24 --availability-zone "$AZ2" --query 'Subnet.SubnetId' --output text)

# Public 서브넷 라우팅 테이블: IGW로 향하는 기본 라우트
RT_ID=$(aws ec2 create-route-table --vpc-id "$VPC_ID" --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id "$RT_ID" --destination-cidr-block 0.0.0.0/0 --gateway-id "$IGW_ID"
aws ec2 associate-route-table --route-table-id "$RT_ID" --subnet-id "$PUB_SUB1"
aws ec2 associate-route-table --route-table-id "$RT_ID" --subnet-id "$PUB_SUB2"
aws ec2 modify-subnet-attribute --subnet-id "$PUB_SUB1" --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id "$PUB_SUB2" --map-public-ip-on-launch
```

### 2. Security Group 구성 (ALB ← Internet, EC2 ← ALB, RDS ← EC2)

```bash
ALB_SG=$(aws ec2 create-security-group --group-name lab3-alb-sg --description "ALB SG" --vpc-id "$VPC_ID" --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id "$ALB_SG" --protocol tcp --port 80 --cidr 0.0.0.0/0

WEB_SG=$(aws ec2 create-security-group --group-name lab3-web-sg --description "Web SG" --vpc-id "$VPC_ID" --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id "$WEB_SG" --protocol tcp --port 80 --source-group "$ALB_SG"

DB_SG=$(aws ec2 create-security-group --group-name lab3-db-sg --description "DB SG" --vpc-id "$VPC_ID" --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id "$DB_SG" --protocol tcp --port 3306 --source-group "$WEB_SG"
```

### 3. Launch Template + Auto Scaling Group 생성

```bash
AMI_ID=$(aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query 'Parameter.Value' --output text)

cat > /tmp/lab3-userdata.sh <<'EOF'
#!/bin/bash
dnf update -y && dnf install -y httpd
systemctl enable --now httpd
echo "<h1>Lab3 - $(hostname -I)</h1>" > /var/www/html/index.html
EOF

aws ec2 create-launch-template \
  --launch-template-name lab3-lt \
  --launch-template-data "{
    \"ImageId\":\"$AMI_ID\",
    \"InstanceType\":\"t3.micro\",
    \"SecurityGroupIds\":[\"$WEB_SG\"],
    \"UserData\":\"$(base64 -i /tmp/lab3-userdata.sh)\"
  }"

aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name lab3-asg \
  --launch-template "LaunchTemplateName=lab3-lt,Version='$Latest'" \
  --min-size 2 --max-size 4 --desired-capacity 2 \
  --vpc-zone-identifier "$PRIV_SUB1,$PRIV_SUB2" \
  --target-group-arns "$(echo $TG_ARN)"
```

### 4. ALB + Target Group 생성 (3단계보다 먼저 생성해야 ASG 연결 가능 — 순서 주의)

```bash
TG_ARN=$(aws elbv2 create-target-group \
  --name lab3-tg --protocol HTTP --port 80 --vpc-id "$VPC_ID" \
  --health-check-path "/" --target-type instance \
  --query 'TargetGroups[0].TargetGroupArn' --output text)

ALB_ARN=$(aws elbv2 create-load-balancer \
  --name lab3-alb --subnets "$PUB_SUB1" "$PUB_SUB2" \
  --security-groups "$ALB_SG" --scheme internet-facing --type application \
  --query 'LoadBalancers[0].LoadBalancerArn' --output text)

aws elbv2 create-listener \
  --load-balancer-arn "$ALB_ARN" --protocol HTTP --port 80 \
  --default-actions "Type=forward,TargetGroupArn=$TG_ARN"

# ASG 생성 시 --target-group-arns "$TG_ARN" 을 포함해 3단계 명령을 재실행
aws autoscaling update-auto-scaling-group --auto-scaling-group-name lab3-asg --target-group-arns "$TG_ARN" 2>/dev/null || true
```

### 5. RDS Multi-AZ 데이터베이스 생성

```bash
aws rds create-db-subnet-group \
  --db-subnet-group-name lab3-db-subnet \
  --db-subnet-group-description "Lab3 DB subnet group" \
  --subnet-ids "$PRIV_SUB1" "$PRIV_SUB2"

aws rds create-db-instance \
  --db-instance-identifier lab3-mysql \
  --db-instance-class db.t3.micro \
  --engine mysql --engine-version 8.0 \
  --master-username admin --master-user-password 'ChangeMe12345!' \
  --allocated-storage 20 \
  --vpc-security-group-ids "$DB_SG" \
  --db-subnet-group-name lab3-db-subnet \
  --multi-az \
  --no-publicly-accessible

aws rds wait db-instance-available --db-instance-identifier lab3-mysql
```

### 6. Route 53 레코드로 ALB 연결 (호스팅 영역 보유 시)

```bash
HOSTED_ZONE_ID="<보유한 호스팅 영역 ID>"
ALB_DNS=$(aws elbv2 describe-load-balancers --load-balancer-arns "$ALB_ARN" --query 'LoadBalancers[0].DNSName' --output text)
ALB_ZONE_ID=$(aws elbv2 describe-load-balancers --load-balancer-arns "$ALB_ARN" --query 'LoadBalancers[0].CanonicalHostedZoneId' --output text)

cat > /tmp/lab3-route53.json <<EOF
{
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "lab3.example.com",
      "Type": "A",
      "AliasTarget": { "HostedZoneId": "$ALB_ZONE_ID", "DNSName": "$ALB_DNS", "EvaluateTargetHealth": true }
    }
  }]
}
EOF
aws route53 change-resource-record-sets --hosted-zone-id "$HOSTED_ZONE_ID" --change-batch file:///tmp/lab3-route53.json
```

## 결과 확인

- `curl http://$ALB_DNS`를 여러 번 호출했을 때 응답하는 인스턴스(hostname/IP)가 바뀌는지 확인해 ALB의 로드 밸런싱을 검증합니다.
- Auto Scaling Group의 인스턴스 하나를 강제로 종료(`aws ec2 terminate-instances`)한 뒤, ASG가 자동으로 새 인스턴스를 시작하는지 콘솔 또는 `aws autoscaling describe-auto-scaling-groups`로 확인합니다.
- RDS Multi-AZ는 콘솔에서 `Multi-AZ: Yes`로 표시되며, `aws rds describe-db-instances`의 `MultiAZ` 필드가 `true`인지 확인합니다.

## 체크리스트

- [ ] Public 서브넷에는 ALB만, Private 서브넷에는 EC2와 RDS만 위치한다
- [ ] ALB로 들어온 요청이 여러 AZ의 EC2 인스턴스로 분산된다
- [ ] Auto Scaling Group이 인스턴스 장애 시 자동으로 교체한다
- [ ] RDS가 Multi-AZ로 구성되어 있다
- [ ] (선택) Route 53 레코드를 통해 도메인으로 ALB에 접속할 수 있다
- [ ] 실습 종료 후 ASG, ALB, Target Group, RDS, NAT/IGW, VPC를 역순으로 모두 삭제했다

다음 실습인 **[Lab 4: 서버리스 REST API](../lab4-serverless-architecture/)** 에서는 같은 문제를 서버 관리 없이 해결하는 방법을 다룹니다.
