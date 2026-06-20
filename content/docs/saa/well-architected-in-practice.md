---
title: "Well-Architected 실전 적용"
weight: 4
---

지금까지 복원력, 성능/비용, 보안을 각각 개별 주제로 다뤘습니다. 하지만 실제 설계 결정은 이 세 축이 동시에 얽혀 있는 trade-off 문제입니다. **AWS Well-Architected Framework**의 6 Pillars는 이런 trade-off를 판단할 때 쓰는 공통 언어이며, SAA-C03 시험은 사실상 이 6 Pillars를 다른 말로 풀어서 묻는 시험이라고 볼 수 있습니다.

## 6 Pillars를 SAA 문제에 매핑하기

SAA 문제를 풀 때 "이 선택지가 어떤 Pillar를 강화하고 어떤 Pillar를 희생하는가"를 자문하면 정답을 좁히기 훨씬 쉬워집니다.

| Pillar | SAA에서 자주 등장하는 형태 |
|---|---|
| Operational Excellence | CloudFormation/IaC로 배포 자동화, CloudWatch로 운영 가시성 확보 |
| Security | IAM 최소 권한, Security Group/NACL, KMS 암호화 |
| Reliability | Multi-AZ, Auto Scaling, ELB, 백업/복구 전략 |
| Performance Efficiency | 적절한 인스턴스/스토리지 타입 선택, 캐싱, 서버리스 활용 |
| Cost Optimization | 적절한 과금 모델 선택, 스토리지 클래스, 미사용 리소스 정리 |
| Sustainability | 리소스 사용률 최적화, 관리형 서비스 활용으로 유휴 자원 최소화 |

이전 페이지들에서 다룬 [복원력](../resilient-architectures/), [성능/비용](../performance-cost-architectures/), [보안](../secure-architectures/) 내용은 각각 Reliability, Performance Efficiency/Cost Optimization, Security Pillar에 직접 대응합니다.

{{< callout type="info" >}}
6 Pillars 각각의 상세 설계 원칙과 체크리스트는 **[④ Well-Architected Framework](../../well-architected/)** 섹션에 별도로 정리되어 있습니다. 이 페이지는 "SAA 단계에서 어떻게 적용하는가"에 집중하고, 각 Pillar의 심화 내용은 해당 섹션을 참고하세요.
{{< /callout >}}

## Trade-off 사고 연습 — 예시 시나리오

다음과 같은 SAA 스타일 시나리오로 6 Pillars 사고를 연습해 봅니다.

> "트래픽이 예측 불가능하게 변동하는 신규 서비스의 백엔드를 설계하라. 운영 인력은 1명뿐이다."

- **Reliability** 관점: 트래픽 변동에 견디려면 Auto Scaling이 필요하지만, 서버리스(Lambda + API Gateway)를 쓰면 확장 자체를 AWS가 대신 처리해줍니다.
- **Operational Excellence** 관점: 운영 인력이 1명이라면 EC2 fleet을 직접 관리하는 부담을 줄여야 합니다. 서버리스는 패치, 스케일링 설정을 직접 관리할 필요가 없습니다.
- **Cost Optimization** 관점: 트래픽이 간헐적이라면 상시 가동되는 EC2보다 호출당 과금되는 Lambda가 유휴 비용을 없애줍니다.
- **Performance Efficiency** 관점: Lambda는 콜드 스타트 지연이 있을 수 있어, 응답 시간 요구사항이 매우 엄격하다면 Provisioned Concurrency 같은 보완책을 함께 고려해야 합니다.

이 시나리오에서는 네 개 Pillar가 모두 "서버리스"라는 같은 방향을 가리키지만, 실제 시험에서는 Pillar 간에 충돌이 발생하는 경우(예: 가용성을 높이려면 비용이 늘어나는 경우)가 더 흔하게 출제됩니다.

## 시험장에서 쓰는 판단 기준

문제에 명시된 제약 조건(비용 민감, 운영 인력 부족, 컴플라이언스 요구 등)이 곧 "이 문제에서 어떤 Pillar에 가중치를 둘 것인가"를 알려주는 힌트입니다.

{{< callout type="warning" >}}
"무조건 가장 안전한 선택" 또는 "무조건 가장 저렴한 선택"이 항상 정답은 아닙니다. 문제 지문에 있는 제약 조건을 무시하고 한 Pillar만 극대화하는 선택지는 SAA의 대표적인 오답 함정입니다.
{{< /callout >}}
