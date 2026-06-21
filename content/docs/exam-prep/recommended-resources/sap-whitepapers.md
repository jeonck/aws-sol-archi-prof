---
title: "SAP 시험 대비 필수 화이트페이퍼 5선"
weight: 1
---

SAP-C02는 단순 서비스 스펙이 아니라 **AWS가 공식적으로 권장하는 설계 원칙**을 묻는 시나리오형 문제가 많습니다. 이 원칙들은 대부분 AWS 공식 화이트페이퍼에 그대로 문서화되어 있어, 시험 전 한 번씩 훑어두면 선택지를 판단하는 기준이 명확해집니다. 분량이 방대하므로 아래 5가지 주제 순서로, 각 백서의 핵심 사항만 먼저 파악하는 것을 권장합니다.

## 1. AWS Well-Architected Framework (핵심 중의 핵심)

SAA와 SAP를 통틀어 가장 출제 빈도가 높은 사고 틀입니다. 외부 백서를 새로 읽기 전에, 본 사이트의 **[④ Well-Architected Framework](../../../well-architected/)** 섹션에서 6개 기둥을 먼저 정리하세요.

- **[운영 우수성](../../../well-architected/operational-excellence/)** — IaC, 변경 관리, 장애 대응, 가시성
- **[보안](../../../well-architected/security-pillar/)** — 심층 방어, 최소 권한, 데이터 보호
- **[신뢰성](../../../well-architected/reliability-pillar/)** — 장애 복구 자동화, Multi-AZ/Multi-Region
- **[성능 효율성](../../../well-architected/performance-efficiency-pillar/)** — 적절한 리소스 선택, 서버리스, 글로벌 인프라
- **[비용 최적화](../../../well-architected/cost-optimization-pillar/)** — 소비 모델, 비용 측정, Right-sizing
- **[지속 가능성](../../../well-architected/sustainability-pillar/)** — 리소스 효율화, 다운스트림 영향

{{< callout type="info" >}}
이 6 Pillars는 다른 4개 백서를 읽을 때도 판단 기준으로 계속 재사용됩니다. 아래 화이트페이퍼들을 읽으면서 "이 권고가 어떤 Pillar의 trade-off인가"를 함께 따라가세요.
{{< /callout >}}

## 2. 멀티 계정 전략 및 거버넌스 — Organizing Your AWS Environment

[Organizing Your AWS Environment Using Multiple Accounts](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)

- **다중 계정 전략의 중요성**: 비즈니스 애플리케이션과 데이터를 격리하고 관리하기 위해 여러 AWS 계정을 사용하는 것은 운영 효율성, 보안, 안정성, 비용 최적화를 위해 매우 중요합니다.
- **격리 경계로서의 계정**: 각 AWS 계정은 독립적인 격리 경계 역할을 하며, 기본적으로 계정 간 통신은 차단되어 있습니다.
- **성장 단계에 따른 확장**: 초기 실험 단계에서는 단일 계정으로 시작할 수 있으나, 비즈니스가 성장함에 따라 보안·거버넌스·관리 복잡성을 해결하기 위해 다중 계정 전략으로 전환해야 합니다.
- **Well-Architected 연계**: 이 전략은 Well-Architected Framework의 핵심 기둥(운영 우수성, 보안, 안정성, 비용 최적화)을 직접 지지합니다.
- **관리의 자동화**: 계정 수가 많아질수록 복잡성을 줄이고 보안·거버넌스 요구사항을 충족하기 위해 자동화된 관리(AWS Organizations, SCP, StackSets 등)가 필수적입니다.

## 3. 고가용성 및 재해 복구(DR) — Disaster Recovery of Workloads on AWS

[Disaster Recovery of Workloads on AWS: Recovery in the Cloud](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.html)

- **재해 복구의 정의**: 비즈니스 목표를 달성할 수 없게 만드는 시스템·워크로드 장애 상황을 대비하고 복구하는 과정입니다.
- **클라우드 재해 복구의 차별점**: 온프레미스와는 다른 클라우드만의 전략과 기술을 활용해 더 효율적인 복구를 지원합니다.
- **핵심 지표**: 워크로드별로 허용 가능한 데이터 손실 범위인 **RPO(복구 시점 목표)**와 복구 소요 시간인 **RTO(복구 시간 목표)**를 달성하는 것이 중요합니다.
- **전략 수립**: 리스크를 완화하기 위해 상황에 맞는 다양한 재해 복구 옵션(Backup & Restore, Pilot Light, Warm Standby, Multi-Site Active-Active)을 제공합니다.
- **지속적인 관리**: 재해 복구 계획은 수립으로 끝나지 않고, 정기적인 테스트를 통해 실효성을 검증해야 합니다.
- **고가용성(HA)과의 구분**: 고가용성은 가동 시간을 극대화하는 것이며, 재해 복구와는 별개의 개념입니다.

## 4. 마이크로서비스 및 서버리스 아키텍처 — Serverless Application Lens

[Microservices on AWS: Serverless Technologies](https://docs.aws.amazon.com/ko_kr/whitepapers/latest/microservices-on-aws/microservices-on-serverless-technologies.html)

- **서버리스 기술의 이점**: 마이크로서비스에 서버리스 기술을 도입하면 운영 복잡성을 크게 줄일 수 있습니다.
- **주요 구성 요소**: AWS Lambda, API Gateway, AWS Fargate 등을 조합하여 완전한 서버리스 애플리케이션 구현이 가능합니다.
- **Lambda 응답 스트리밍**: 함수가 응답을 생성하는 즉시 부분적으로 클라이언트에 전송하여 '첫 번째 바이트까지의 시간(TTFB)'을 크게 개선하고 성능을 높이는 기능입니다.
- **확장성 및 고가용성**: 서버리스 아키텍처는 인프라 관리 부담을 줄여주며, Amazon Aurora Serverless 같은 서비스를 통해 요구 사항에 따른 자동 크기 조정이 가능합니다.

## 5. 데이터 이전 및 현대화 전략 — Migration Strategies (7 Rs)

[AWS Prescriptive Guidance: Migration Strategies](https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html)

AWS의 7가지 마이그레이션 전략(7 Rs)입니다.

- **Retire (은퇴)**: 더 이상 필요 없거나 비즈니스 가치가 없는 애플리케이션을 폐기합니다.
- **Retain (유지)**: 마이그레이션할 준비가 되지 않았거나 현재 환경에 두는 것이 더 나은 애플리케이션을 그대로 유지합니다.
- **Rehost (리호스트)**: 애플리케이션 변경 없이 그대로 AWS로 이전하는 '리프트 앤 시프트(lift and shift)' 방식입니다.
- **Relocate (재배치)**: 인프라 변경 없이 애플리케이션을 클라우드 버전의 플랫폼으로 이전합니다.
- **Repurchase (재구매)**: 기존 애플리케이션을 클라우드 기반 SaaS 등으로 교체하는 '드롭 앤 숍(drop and shop)' 방식입니다.
- **Replatform (리플랫폼)**: 약간의 최적화를 거쳐 클라우드로 이전하는 '리프트, 팅커 앤 시프트(lift, tinker, and shift)' 방식입니다.
- **Refactor/Re-architect (리팩터/재설계)**: 클라우드 네이티브 기능을 활용하여 애플리케이션 아키텍처를 근본적으로 재설계합니다.

{{< callout type="warning" >}}
대규모 마이그레이션 시에는 일반적으로 **Rehost, Replatform, Relocate, Retire** 전략이 주로 권장됩니다. 시험에서 "수천 개 서버를 단기간에 이전"하는 시나리오가 나오면 Refactor보다 이 네 가지를 먼저 의심하세요.
{{< /callout >}}

## 더 둘러보기

이 페이지에서 다루지 않은 다른 학습 자료(강의, Practice Exam 등)는 **[추천 학습 자료](../)** 로 돌아가 확인하세요.
