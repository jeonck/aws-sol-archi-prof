---
title: "SAA-C03 시험 개요"
weight: 1
---

AWS Certified Solutions Architect - Associate(SAA-C03)는 AWS 자격증 중 가장 많은 사람이 응시하는 시험입니다. 실무에서 자주 마주치는 아키텍처 설계 결정을 다루기 때문에, 시험 준비 과정 자체가 실무 역량으로 바로 연결됩니다.

## 시험 구조

| 항목 | 내용 |
|---|---|
| 시험 코드 | SAA-C03 |
| 시험 시간 | 130분 |
| 문항 수 | 65문제 (객관식 + 복수 응답) |
| 합격 기준 | 1000점 만점 중 약 720점 (백분율 환산 아님, 영역별 가중 채점) |
| 응시료 | $150 USD |
| 시험 형식 | Pearson VUE 또는 PSI를 통한 시험 센터 응시 또는 온라인 proctoring |
| 유효 기간 | 합격 후 3년 |

합격 기준 720점은 단순히 "65문제 중 몇 개를 맞혀야 하는가"로 환산할 수 없습니다. AWS는 문제별로 난이도에 따른 가중치를 적용해 채점하기 때문에, 같은 정답 개수라도 어떤 문제를 맞혔는지에 따라 환산 점수가 달라집니다. 따라서 "70% 정도만 맞으면 되겠지"라는 식의 단순 계산보다, 출제 영역 전반에서 고르게 높은 정답률을 유지하는 전략이 안전합니다.

## 등록 방법

1. [AWS Certification 포털](https://aws.amazon.com/certification/)에서 AWS Builder ID로 계정을 만들거나 로그인합니다.
2. 시험을 선택하고 응시 방법을 고릅니다.
   - **Pearson VUE** — 전 세계적으로 시험 센터가 가장 많고, 온라인 proctoring(자택 응시)도 지원합니다.
   - **PSI** — 일부 국가에서는 PSI만 지원하거나 두 업체를 함께 운영합니다. 한국은 Pearson VUE 시험 센터가 일반적입니다.
3. 응시료 결제 후 날짜와 시간을 선택합니다. 시험 센터는 보통 2~4주 전에는 원하는 시간대가 남아 있지만, 인기 있는 슬롯(주말, 저녁)은 빨리 마감됩니다.
4. 온라인 응시를 선택하는 경우, 사전에 시스템 점검(웹캠, 마이크, 방 점검)을 반드시 먼저 진행하세요. 응시 환경 규정(책상에 다른 물건 없음, 다른 사람 출입 금지 등)을 어기면 시험이 중단될 수 있습니다.
5. 합격 시 디지털 배지가 Credly를 통해 발급되며, AWS Certification 계정에서 성적표(영역별 점수 분포)를 확인할 수 있습니다.

{{< callout type="info" >}}
재응시가 필요한 경우 같은 시험은 **14일 대기 기간**이 있어야 다시 응시할 수 있습니다. 응시료 할인이 필요하다면 이전 자격증 합격자에게 제공되는 50% 할인 바우처(Recertification 또는 관련 자격증 보유자 대상)를 확인해보세요.
{{< /callout >}}

## 출제 영역과 비중

SAA-C03은 4개의 도메인으로 구성되며, 각 도메인의 비중은 다음과 같습니다.

| 도메인 | 비중 | 핵심 내용 |
|---|---|---|
| 1. 안전한 아키텍처 설계(Design Secure Architectures) | 30% | IAM, KMS, VPC 보안 그룹/NACL, Shared Responsibility Model |
| 2. 복원력 있는 아키텍처 설계(Design Resilient Architectures) | 26% | Multi-AZ, Auto Scaling, 로드 밸런싱, 디커플링(SQS/SNS) |
| 3. 고성능 아키텍처 설계(Design High-Performing Architectures) | 24% | 캐싱(CloudFront/ElastiCache), 스토리지 계층, 컴퓨팅 선택 |
| 4. 비용 최적화 아키텍처 설계(Design Cost-Optimized Architectures) | 20% | 인스턴스 구매 옵션, S3 스토리지 클래스, 비용 모니터링 |

보안과 복원력 두 영역이 전체의 56%를 차지한다는 점이 중요합니다. 단순히 "이 서비스가 무엇을 하는가"를 외우는 것보다 "왜 이 설계가 더 안전하거나 더 복원력이 있는가"를 설명할 수 있어야 합니다. 이는 결국 Well-Architected Framework의 **Security**, **Reliability** 두 Pillar와 정확히 겹칩니다.

{{< callout type="warning" >}}
4개 도메인 모두 Well-Architected Framework의 사고방식을 요구합니다. 서비스 스펙을 외우는 공부보다, 각 주제를 다룰 때 "이 설계가 6 Pillars 중 무엇을 위한 trade-off인가"를 자문하는 습관이 실제 시험 점수에 더 큰 영향을 줍니다.
{{< /callout >}}

## 실전 감각 채우기

이론 학습이 끝났다면, **[SAA 핵심 Hands-on 프로젝트](../../saa/saa-hands-on-projects/)** 에서 직접 아키텍처를 구성해보는 것을 권장합니다. 콘솔에서 한 번이라도 구성해본 서비스는 시험에서 시나리오를 훨씬 빠르게 이해할 수 있습니다. Practice Exam을 어떻게 활용할지는 **[Practice Exam 전략](../practice-exam-strategy/)** 을 참고하세요.
