---
title: "③ Solutions Architect Professional"
weight: 3
---

SAA-C03을 통과했다면 이제 "정답이 하나가 아닌" 문제를 다룰 차례입니다. SAP-C02는 단일 워크로드 설계가 아니라, 수십 개의 AWS 계정과 수백 명의 엔지니어가 함께 움직이는 조직 전체의 아키텍처를 다룹니다. 마이그레이션 전략, 비용 거버넌스, 재해 복구, 컴플라이언스 자동화처럼 "기술적으로는 여러 방법이 가능하지만, 비즈니스 제약 안에서 최선의 trade-off를 골라야 하는" 문제가 시험과 실무 모두에서 핵심이 됩니다.

{{< callout type="warning" >}}
SAP-C02 시험은 응시료 $300, 시험 시간 180분, 문항 수 75개로 SAA-C03보다 훨씬 깁니다. 단순 암기형 문제는 거의 없고, 시나리오 기반으로 "가장 비용 효율적이면서 운영 부담이 적은 선택"을 묻는 문제가 대부분입니다. 이 섹션의 각 주제를 실제 AWS 콘솔에서 한 번이라도 구성해보지 않으면, trade-off 판단 감각이 생기지 않습니다.
{{< /callout >}}

이 섹션은 AWS 공식 SAP-C02 Exam Guide의 **4개 콘텐츠 도메인** 구조를 그대로 따릅니다. 각 도메인 가중치와 도메인 안의 작업(Task) 전체 목록은 **[SAP-C02 시험 청사진](../exam-prep/sap-exam-domains/)** 에서 표와 다이어그램으로 따로 정리했습니다.

## 이 섹션에서 다루는 내용

1. [도메인 1: 조직 복잡성을 위한 솔루션 설계](domain1-organizational-complexity/) — 네트워크 연결, 보안 제어, DR/복원력, 다중 계정(Organizations·Control Tower·SCP), 비용 가시성 (가중치 26%)
2. [도메인 2: 새로운 솔루션을 위한 설계](domain2-new-solution-design/) — IaC·CI/CD 배포 전략, 비즈니스 연속성, 보안·신뢰성·성능·비용을 신규 워크로드에 적용 (가중치 29%)
3. [도메인 3: 기존 솔루션의 지속적인 개선](domain3-continuous-improvement/) — Well-Architected Tool 정기 리뷰, 보안·성과·신뢰성 개선, 비용 최적화 기회 파악 (가중치 25%)
4. [도메인 4: 워크로드 마이그레이션 및 현대화 가속화](domain4-migration-modernization/) — 6R 전략, Migration Hub, DMS, 모놀리식에서 마이크로서비스로의 전환 (가중치 20%)

{{< cards >}}
  {{< card link="domain1-organizational-complexity" title="도메인 1: 조직 복잡성" subtitle="네트워크·보안·DR·다중 계정·비용 가시성 (26%)" icon="office-building" >}}
  {{< card link="domain2-new-solution-design" title="도메인 2: 새로운 솔루션 설계" subtitle="배포 전략·연속성·보안·신뢰성·성능·비용 (29%)" icon="cube" >}}
  {{< card link="domain3-continuous-improvement" title="도메인 3: 지속적인 개선" subtitle="Well-Architected Tool 기반 정기 리뷰 (25%)" icon="chart-bar" >}}
  {{< card link="domain4-migration-modernization" title="도메인 4: 마이그레이션·현대화" subtitle="6R 전략, Migration Hub, DMS, 마이크로서비스 전환 (20%)" icon="refresh" >}}
{{< /cards >}}

{{< callout type="info" >}}
이 섹션을 끝까지 학습했다면 SAP-C02 응시 준비가 거의 끝난 셈입니다. 실제 시험 일정과 문제 유형, Practice Exam 전략은 [시험 전략 & 리소스](../exam-prep/) 섹션에서 이어서 확인하세요.
{{< /callout >}}
