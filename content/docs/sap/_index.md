---
title: "③ Solutions Architect Professional"
weight: 3
---

SAA-C03을 통과했다면 이제 "정답이 하나가 아닌" 문제를 다룰 차례입니다. SAP-C02는 단일 워크로드 설계가 아니라, 수십 개의 AWS 계정과 수백 명의 엔지니어가 함께 움직이는 조직 전체의 아키텍처를 다룹니다. 마이그레이션 전략, 비용 거버넌스, 재해 복구, 컴플라이언스 자동화처럼 "기술적으로는 여러 방법이 가능하지만, 비즈니스 제약 안에서 최선의 trade-off를 골라야 하는" 문제가 시험과 실무 모두에서 핵심이 됩니다.

{{< callout type="warning" >}}
SAP-C02 시험은 응시료 $300, 시험 시간 180분, 문항 수 75개로 SAA-C03보다 훨씬 깁니다. 단순 암기형 문제는 거의 없고, 시나리오 기반으로 "가장 비용 효율적이면서 운영 부담이 적은 선택"을 묻는 문제가 대부분입니다. 이 섹션의 각 주제를 실제 AWS 콘솔에서 한 번이라도 구성해보지 않으면, trade-off 판단 감각이 생기지 않습니다.
{{< /callout >}}

## 이 섹션에서 다루는 내용

1. [조직 복잡성 관리](organizational-complexity/) — AWS Organizations, Control Tower, Landing Zone, SCP로 멀티 계정 거버넌스 구축
2. [마이그레이션과 현대화](migration-modernization/) — 6R 전략, Migration Hub, DMS, 모놀리식에서 마이크로서비스로의 전환
3. [비용 통제와 최적화](cost-control-optimization/) — Savings Plans, Reserved Instances, Cost Explorer, 조직 차원의 비용 거버넌스
4. [재해 복구와 네트워킹](disaster-recovery-networking/) — DR 전략 4단계, RTO/RPO, Transit Gateway, Direct Connect
5. [고급 보안과 컴플라이언스](security-compliance-advanced/) — KMS 심화, GuardDuty, Security Hub, Config 기반 컴플라이언스 자동화
6. [지속적 개선](continuous-improvement/) — Well-Architected Tool을 이용한 정기 워크로드 리뷰

{{< cards >}}
  {{< card link="organizational-complexity" title="조직 복잡성 관리" subtitle="Organizations, Control Tower, Landing Zone, SCP" icon="office-building" >}}
  {{< card link="migration-modernization" title="마이그레이션과 현대화" subtitle="6R 전략, Migration Hub, DMS, 마이크로서비스 전환" icon="refresh" >}}
  {{< card link="cost-control-optimization" title="비용 통제와 최적화" subtitle="Savings Plans, Cost Explorer, 비용 거버넌스" icon="currency-dollar" >}}
  {{< card link="disaster-recovery-networking" title="재해 복구와 네트워킹" subtitle="DR 전략, RTO/RPO, Transit Gateway, Direct Connect" icon="map" >}}
  {{< card link="security-compliance-advanced" title="고급 보안과 컴플라이언스" subtitle="KMS 심화, GuardDuty, Security Hub, Config" icon="shield-check" >}}
  {{< card link="continuous-improvement" title="지속적 개선" subtitle="Well-Architected Tool 기반 정기 리뷰" icon="chart-bar" >}}
{{< /cards >}}

{{< callout type="info" >}}
이 섹션을 끝까지 학습했다면 SAP-C02 응시 준비가 거의 끝난 셈입니다. 실제 시험 일정과 문제 유형, Practice Exam 전략은 [시험 전략 & 리소스](../exam-prep/) 섹션에서 이어서 확인하세요.
{{< /callout >}}
