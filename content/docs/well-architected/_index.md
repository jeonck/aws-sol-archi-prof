---
title: "④ Well-Architected Framework"
weight: 4
---

지금까지 AWS 기초, SAA, SAP을 별도의 단계처럼 소개했지만, 이 6개의 기둥(Pillar)은 그 단계들과 나란히 놓인 또 하나의 단계가 아닙니다. **Fundamentals에서 첫 EC2 인스턴스를 띄울 때부터, SAP에서 멀티 리전 DR을 설계할 때까지 전 구간에서 반복적으로 적용해야 하는 사고 방식**입니다. "이 설계가 운영 우수성, 보안, 신뢰성, 성능 효율성, 비용 최적화, 지속 가능성 중 어디에 영향을 주는가"를 매번 자문하는 습관이 곧 Well-Architected Framework를 실천하는 것입니다.

{{< callout type="warning" >}}
이 섹션을 "이론으로 한 번 읽고 넘어가는 챕터"로 취급하면 효과가 없습니다. SAA나 SAP에서 새로운 서비스나 패턴을 배울 때마다 이 6 Pillars로 돌아와 "이게 어떤 기둥에 해당하는 트레이드오프인가"를 확인하는 습관을 들이세요. 실제로 SAP-C02 시험 문제의 상당수가 6 Pillars 사이의 trade-off를 묻는 형태로 출제됩니다.
{{< /callout >}}

## 이 섹션에서 다루는 내용

1. [운영 우수성](operational-excellence/) — IaC, 변경 관리, 장애 대응, CloudWatch/X-Ray 가시성
2. [보안](security-pillar/) — 심층 방어, 최소 권한, 데이터 보호, 사고 대응 준비
3. [신뢰성](reliability-pillar/) — 장애 복구 자동화, 수평적 확장, Multi-AZ/Multi-Region
4. [성능 효율성](performance-efficiency-pillar/) — 적절한 리소스 선택, 서버리스, 글로벌 인프라 활용
5. [비용 최적화](cost-optimization-pillar/) — 소비 모델, 비용 측정, Right-sizing
6. [지속 가능성](sustainability-pillar/) — 리소스 효율화, 재생에너지 리전, 다운스트림 영향

{{< cards >}}
  {{< card link="operational-excellence" title="운영 우수성" subtitle="IaC, 변경 관리, 장애 대응, 가시성" icon="cog" >}}
  {{< card link="security-pillar" title="보안" subtitle="심층 방어, 최소 권한, 데이터 보호" icon="lock-closed" >}}
  {{< card link="reliability-pillar" title="신뢰성" subtitle="장애 복구, 수평 확장, Multi-AZ/Region" icon="refresh" >}}
  {{< card link="performance-efficiency-pillar" title="성능 효율성" subtitle="리소스 선택, 서버리스, 글로벌 인프라" icon="chart-bar" >}}
  {{< card link="cost-optimization-pillar" title="비용 최적화" subtitle="소비 모델, 비용 측정, Right-sizing" icon="currency-dollar" >}}
  {{< card link="sustainability-pillar" title="지속 가능성" subtitle="리소스 효율화, 재생에너지, 다운스트림 영향" icon="globe" >}}
{{< /cards >}}

{{< callout type="info" >}}
이 6 Pillars를 실제 아키텍처에 적용하는 구체적인 연습은 [SAA: 실전 Well-Architected 적용](../saa/well-architected-in-practice/)에서 먼저 다룹니다. 이 섹션과 그 페이지를 함께 오가며 학습하는 것을 권장합니다.
{{< /callout >}}
