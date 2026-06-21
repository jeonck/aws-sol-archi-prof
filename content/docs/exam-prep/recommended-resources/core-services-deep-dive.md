---
title: "시험에 반복 출제되는 핵심 서비스 5가지 영역"
weight: 2
---

SAP-C02는 단순히 기본적인 서비스 사용법을 넘어, **복잡한 기업 요구사항을 해결하기 위해 여러 서비스를 조합**하는 능력을 평가합니다. 샘플 문제와 화이트페이퍼를 가로질러 반복적으로 등장하는 핵심 서비스를 5가지 영역으로 묶었습니다. 단순 기능 암기보다 아래 영역별 "조합 패턴"을 익히는 것이 훨씬 효율적입니다.

## 1. 멀티 계정 관리 및 배포 서비스

전문가(Professional) 레벨에서는 수십, 수백 개의 계정을 효율적으로 관리하는 능력을 평가합니다.

- **AWS Organizations & SCP(Service Control Policies)**: 여러 계정의 중앙 집중식 관리와 계정별 권한 제한(예: 특정 인스턴스 유형 실행 금지)을 설정할 때 필수적입니다.
- **AWS CloudFormation StackSets**: 단일 작업으로 여러 AWS 계정과 리전에 걸쳐 IAM 역할이나 인프라를 일관되게 배포할 때 핵심적인 역할을 합니다.
- **AWS IAM & AWS IAM Identity Center(구 SSO)**: 온프레미스 Active Directory와의 통합이나, 계정 간 교차 액세스(Cross-Account Access)를 위한 역할(Role) 및 신뢰 정책(Trust Policy) 구성 능력이 자주 요구됩니다.

{{< callout type="info" >}}
단일 자격 증명으로 여러 계정의 권한을 위임하는 원리는 **[AssumeRole의 원리](../../architecture-principles/assume-role-single-credentials/)** 에서, 자동화가 오히려 함정이 되는 IAM·SCP 패턴은 **[자동화 금지 영역 체크리스트](../../architecture-principles/automation-control-boundaries/)** 에서 더 자세히 다룹니다.
{{< /callout >}}

## 2. 고성능 및 글로벌 데이터베이스 서비스

단순한 DB 운영이 아니라 리전 간 복제와 확장을 다루는 서비스들이 중요합니다.

- **Amazon Aurora(Global Database & Serverless)**: 리전 간 쓰기 전달(Write Forwarding) 기능이 포함된 글로벌 데이터베이스 구성이나, 운영 오버헤드를 줄이기 위한 서버리스 마이그레이션 시나리오로 자주 등장합니다.
- **Amazon DynamoDB**: 서버리스 모바일 앱의 백엔드나 글로벌 테이블을 이용한 다중 리전 활성-활성(Active-Active) 구성 요소로 활용됩니다.

{{< callout type="info" >}}
Aurora Global Database의 Write Forwarding을 활용한 LEAST-change Active-Active 설계는 **[최소한의 변경 원칙](../../architecture-principles/least-change-active-active/)** 에서, Aurora Serverless로의 현대화 전환은 **[운영 우수성 관점의 핵심 원리](../../architecture-principles/operational-excellence-modernization/)** 에서 시나리오로 다룹니다.
{{< /callout >}}

## 3. 서버리스 및 컨테이너화된 애플리케이션 서비스

현대적인 애플리케이션 아키텍처 설계를 위한 서비스 조합입니다.

- **AWS App Runner & Amazon ECR**: 컨테이너화된 웹 애플리케이션을 빠르게 배포하고, 리전 간 복제(Cross-Region Replication)를 통해 이미지를 관리하는 시나리오에 사용됩니다.
- **Amazon API Gateway & AWS Lambda**: 동시성 할당량(Concurrency Quotas) 관리 및 502 오류 해결, CORS 설정 등 상세한 운영 설정 능력이 시험에 나옵니다.
- **Amazon Cognito**: 사용자 인증 및 여러 리전의 사용자 풀(User Pools) 관리와 관련된 시나리오에 등장합니다.

{{< callout type="info" >}}
App Runner + ECR + Aurora Global Database를 묶은 Active-Active 시나리오는 **[최소한의 변경 원칙](../../architecture-principles/least-change-active-active/)**, API Gateway의 429/502 오류와 Throttling 전략은 **[API Gateway Throttling 전략](../../architecture-principles/api-gateway-throttling/)** 에서 다룹니다.
{{< /callout >}}

## 4. 분석 및 데이터 처리 아키텍처

대규모 데이터를 수집하고 처리하는 비용 효율적인 설계를 묻는 문제가 많습니다.

- **Amazon Kinesis Data Streams**: 샤드(Shard)와 파티션 키(Partition Key)를 이용한 데이터 수집 및 처리량 최적화가 중요하게 다뤄집니다.
- **Amazon Athena**: S3에 저장된 데이터에 대해 서버 구축 없이 표준 SQL로 즉각적인 쿼리를 실행하여 비용을 절감하는 시나리오에 등장합니다.

{{< callout type="info" >}}
Kinesis 샤드 비율 조정과 Lambda 기반 이상치 처리로 비용을 절감하는 패턴은 **[SAP-C02 샘플 문제 Q7](../../sap-sample-questions/)** 에서 실제 문제로 확인할 수 있습니다.
{{< /callout >}}

## 5. 운영 및 보안 관리 도구

안전하고 효율적인 인프라 관리를 위한 서비스들입니다.

- **AWS Systems Manager(SSM) Run Command**: 인바운드 SSH 포트를 열지 않고도 보안 정책을 준수하며 대규모 인스턴스 군을 관리할 수 있는 최적의 솔루션으로 제시됩니다.
- **Amazon CloudWatch & Amazon SNS**: 결제 알람(Billing Alarms)을 설정하거나, Lambda의 동시성 지표를 모니터링하여 경보를 보내는 등 시스템 가시성 확보에 사용됩니다.

{{< callout type="info" >}}
인바운드 없이 폴링 방식으로 인스턴스를 관리하는 SSM Run Command의 원리는 **[Systems Manager Run Command: 접속이 아니라 폴링이다](../../architecture-principles/ssm-run-command/)** 에서 자세히 다룹니다.
{{< /callout >}}

{{< callout type="warning" >}}
이 서비스들은 샘플 문제에서 비용 절감, 보안 강화, 고가용성 확보라는 구체적인 목적을 달성하기 위해 활용됩니다. 서비스 이름만 외우지 말고 **Reserved Concurrency, Write Forwarding, SCP Condition Key** 같은 상세 설정값까지 "왜 그 설정이 정답인지" 함께 파악하세요.
{{< /callout >}}

## 더 둘러보기

화이트페이퍼 기반 학습은 **[SAP 시험 대비 필수 화이트페이퍼 5선](../sap-whitepapers/)**, 다른 학습 자료는 **[추천 학습 자료](../)** 로 돌아가 확인하세요.
