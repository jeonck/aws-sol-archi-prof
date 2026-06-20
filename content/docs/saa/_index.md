---
title: "② Solutions Architect Associate"
weight: 2
---

SAA-C03(AWS Certified Solutions Architect – Associate)은 AWS 자격증 중 가장 실무 활용도가 높은 시험으로 꼽힙니다. 단순히 서비스 이름을 외우는 것이 아니라, **주어진 요구사항에 맞는 아키텍처를 설계**하는 능력을 평가합니다. 이 섹션에서는 복원력·성능/비용·보안이라는 세 가지 설계 축을 익히고, 이를 Well-Architected Framework 관점에서 통합하는 연습을 합니다.

{{< callout type="warning" >}}
SAA-C03 문제는 "정답이 기술적으로 가능한가"가 아니라 "주어진 제약(비용, 운영 부담, 가용성 요구) 안에서 가장 적절한가"를 묻습니다. 서비스 스펙을 암기하는 것만으로는 합격하기 어렵고, 이 섹션의 각 페이지에서 다루는 **trade-off 사고방식**을 반드시 함께 익혀야 합니다.
{{< /callout >}}

## 이 섹션에서 다루는 내용

1. **[복원력 있는 아키텍처](resilient-architectures)** — Multi-AZ, Auto Scaling, 로드 밸런싱으로 장애에 견디는 설계
2. **[성능/비용 아키텍처](performance-cost-architectures)** — 캐싱, 스토리지 클래스, 컴퓨팅 선택, 비용 가시화
3. **[보안 아키텍처](secure-architectures)** — IAM 정책, 네트워크 보안, WAF, KMS 기초
4. **[Well-Architected 실전 적용](well-architected-in-practice)** — 6 Pillars를 SAA 설계 문제에 적용하는 법
5. **[SAA Hands-on 프로젝트](saa-hands-on-projects)** — 3-Tier 웹앱, 서버리스, 마이그레이션, 모니터링 실습

{{< cards >}}
  {{< card link="resilient-architectures" title="복원력 있는 아키텍처" subtitle="Multi-AZ, Auto Scaling, ELB/ALB" icon="refresh" >}}
  {{< card link="performance-cost-architectures" title="성능/비용 아키텍처" subtitle="캐싱, 스토리지, 컴퓨팅, Cost Explorer" icon="chart-bar" >}}
  {{< card link="secure-architectures" title="보안 아키텍처" subtitle="IAM, Security Group, WAF, KMS" icon="lock-closed" >}}
  {{< card link="well-architected-in-practice" title="Well-Architected 실전 적용" subtitle="6 Pillars를 설계 선택에 적용" icon="scale" >}}
  {{< card link="saa-hands-on-projects" title="SAA Hands-on 프로젝트" subtitle="3-Tier, 서버리스, 마이그레이션, 모니터링" icon="puzzle" >}}
{{< /cards >}}

{{< callout type="info" >}}
이 섹션을 마치고 SAA-C03에 합격했다면 다음은 **[③ Solutions Architect Professional](../sap/)** 입니다. SAP-C02는 멀티 계정 거버넌스, 대규모 마이그레이션, 고급 비용 통제처럼 엔터프라이즈 규모의 설계 문제를 다룹니다.
{{< /callout >}}
