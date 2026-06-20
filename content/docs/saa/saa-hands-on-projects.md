---
title: "SAA Hands-on 프로젝트"
weight: 5
---

SAA-C03 학습의 마지막 단계는 배운 패턴을 실제로 조립해보는 것입니다. 이 페이지는 추천하는 네 가지 Hands-on 프로젝트의 개요를 다루며, 클릭 단위의 상세 가이드는 [Labs](../../../labs/) 섹션에서 진행합니다.

## 프로젝트 1 — 3-Tier 웹 애플리케이션

가장 전통적이면서도 SAA 전반의 개념을 통합적으로 연습할 수 있는 구조입니다.

- **구성**: VPC(퍼블릭/프라이빗 서브넷) + EC2/Auto Scaling Group + RDS + ELB(ALB) + Route 53
- **배우는 것**: 프레젠테이션(ALB) – 애플리케이션(EC2/ASG) – 데이터(RDS) 계층을 네트워크 수준에서 분리하고, 각 계층에 알맞은 Security Group을 설계하는 경험. [복원력 있는 아키텍처](../resilient-architectures/)에서 배운 Multi-AZ와 ASG를 실제로 조립합니다.

{{< callout type="info" >}}
상세 가이드: [Lab 3: 3-Tier 아키텍처](../../../labs/lab3-three-tier-architecture/)
{{< /callout >}}

## 프로젝트 2 — 서버리스 아키텍처

EC2 없이 완전히 관리형 서비스로만 구성하는 아키텍처입니다.

- **구성**: Lambda + API Gateway + DynamoDB + S3
- **배우는 것**: API Gateway가 HTTP 요청을 Lambda로 라우팅하고, Lambda가 DynamoDB에서 데이터를 읽고 쓰는 이벤트 기반 흐름. 서버 프로비저닝이 전혀 없는 구조에서 IAM 역할을 어떻게 세분화해서 부여하는지 연습합니다.

{{< callout type="info" >}}
상세 가이드: [Lab 4: 서버리스 아키텍처](../../../labs/lab4-serverless-architecture/)
{{< /callout >}}

## 프로젝트 3 — Migration & Hybrid

온프레미스와 AWS를 연결하거나 데이터를 이전하는 시나리오를 다룹니다.

- **구성**: Storage Gateway(온프레미스-클라우드 스토리지 연동) + DMS(Database Migration Service)
- **배우는 것**: Storage Gateway로 온프레미스 애플리케이션이 마치 로컬 스토리지처럼 S3를 사용하게 만드는 방식, DMS로 운영 중인 데이터베이스를 최소 중단으로 RDS로 이전하는 흐름. SAP-C02에서 본격적으로 다루는 [마이그레이션](../../sap/domain4-migration-modernization/) 주제의 기초가 됩니다.

## 프로젝트 4 — Monitoring & Logging

운영 가시성을 확보하는 구조입니다.

- **구성**: CloudWatch(메트릭/로그/알람) + X-Ray(분산 추적)
- **배우는 것**: EC2/Lambda의 메트릭과 로그를 CloudWatch로 수집하고 알람을 구성하는 방법, X-Ray로 여러 서비스에 걸친 요청의 지연 구간을 추적하는 방법. 앞의 세 프로젝트에 모니터링을 추가하면 실제 운영 환경과 훨씬 가까운 구조가 됩니다.

## SAA-C03 시험 목표

일반적으로 위 네 가지 프로젝트와 이론 학습을 병행하면 **합격까지 1~2개월** 정도가 소요됩니다. 시험 응시료는 **$150**입니다. 시험 도메인, 문항 구성, Practice Exam 전략 등 자세한 시험 정보는 [SAA 시험 개요](../../exam-prep/saa-exam-overview/)에서 확인하세요.

{{< callout type="warning" >}}
네 가지 프로젝트를 모두 끝내야 시험을 볼 수 있는 것은 아닙니다. 다만 최소한 3-Tier 웹 애플리케이션과 서버리스 아키텍처, 이 두 가지는 직접 구축해본 뒤 시험에 응시하는 것을 권장합니다. 이 두 패턴이 SAA 시험 문제의 절반 이상에서 변형되어 등장합니다.
{{< /callout >}}
