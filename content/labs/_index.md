---
title: "실습 (Hands-on Labs)"
cascade:
  type: docs
sidebar:
  open: true
---

## 이 섹션은 무엇이 다른가

[학습 로드맵](../docs/)은 AWS Solutions Architect 자격증 학습에 필요한 개념과 이론을 체계적으로 설명합니다.
반면 이 **실습(Labs)** 섹션은 **그대로 따라 하면 재현되는, 단계별로 실행 가능한 실습 가이드**만 담습니다. 각 실습은 다음과 같은 동일한 구조를 따릅니다.

1. **목표** — 이 실습을 통해 무엇을 구축하고 확인하는가
2. **사전 준비** — 필요한 도구, 권한, AWS 계정 환경
3. **단계별 실행** — 복사해서 실행할 수 있는 AWS CLI 명령어와 콘솔 작업
4. **결과 확인** — 출력을 어떻게 해석하고 무엇을 확인해야 하는가
5. **체크리스트** — 실습을 완료했는지 자가 점검

{{< callout type="info" >}}
모든 실습은 AWS CLI가 설치되고 적절한 권한을 가진 IAM 사용자(또는 역할)로 인증된 환경을 전제로 합니다. 시작하기 전에 다음을 먼저 확인하세요.

```bash
# AWS CLI 버전 확인 (v2 권장)
aws --version

# 자격 증명 설정 (Access Key, Secret Key, 기본 리전 입력)
aws configure

# 현재 인증된 계정/사용자 확인
aws sts get-caller-identity
```
{{< /callout >}}

## 실습 목록

{{< cards >}}
  {{< card link="lab1-static-site" title="Lab 1: S3 정적 웹사이트 + CloudFront" subtitle="S3 호스팅, CDN 배포, 캐시 무효화" icon="globe" >}}
  {{< card link="lab2-ec2-rds-webapp" title="Lab 2: EC2 + RDS 웹앱" subtitle="2-Tier 아키텍처, Security Group 제어" icon="server" >}}
  {{< card link="lab3-three-tier-architecture" title="Lab 3: 3-Tier 고가용성 아키텍처" subtitle="VPC, ALB, Auto Scaling, RDS Multi-AZ, Route53" icon="cloud" >}}
  {{< card link="lab4-serverless-architecture" title="Lab 4: 서버리스 REST API" subtitle="API Gateway, Lambda, DynamoDB, S3" icon="lock-closed" >}}
  {{< card link="lab5-multi-account-organizations" title="Lab 5: 멀티 계정 거버넌스" subtitle="AWS Organizations, SCP 정책" icon="office-building" >}}
  {{< card link="lab6-disaster-recovery-multiregion" title="Lab 6: 멀티 리전 DR" subtitle="S3 CRR, Route53 Failover, RTO/RPO 측정" icon="refresh" >}}
{{< /cards >}}

{{< callout type="warning" >}}
본인이 소유하거나 명시적으로 권한을 부여받은 AWS 계정에서만 실습하세요. 특히 Lab 5(멀티 계정)와 Lab 6(DR)은 추가 비용이 발생할 수 있으니 실습 후 리소스를 반드시 삭제하세요.
{{< /callout >}}
