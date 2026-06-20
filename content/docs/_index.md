---
title: 학습 로드맵
linkTitle: 로드맵
cascade:
  type: docs
sidebar:
  open: true
---

## AWS Solutions Architect, 어떻게 공부해야 할까

AWS는 이론만으로는 실력이 붙지 않습니다. **4개의 지식 축**과 **하나의 실전 축(Well-Architected Framework)**으로 나누어, 매 단계마다 직접 콘솔과 코드로 손을 움직이며 학습하는 것을 전제로 이 로드맵을 구성했습니다.

1. **[AWS 기초](fundamentals)** — 핵심 서비스 개요와 Free Tier 환경에서의 첫 Hands-on
2. **[Solutions Architect Associate](saa)** — 고가용성·보안·성능/비용 아키텍처 설계, SAA-C03 합격
3. **[Solutions Architect Professional](sap)** — 멀티 계정, 마이그레이션, 비용 통제, DR/네트워킹 등 엔터프라이즈 설계
4. **[시험 전략 & 리소스](exam-prep)** — SAA/SAP 시험 정보와 Practice Exam, 추천 자료

그리고 이 모든 단계를 관통하는 사고 축이 있습니다.

- **[Well-Architected Framework](well-architected)** — Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability 6 Pillars를 모든 설계 결정에 적용하는 mindset

{{< callout type="info" >}}
**추천 학습 순서: AWS 기초 → SAA-C03 → SAP-C02 → 시험 전략**

Well-Architected Framework는 별도 단계가 아니라 SAA, SAP 전 구간에서 반복적으로 등장해야 합니다. SAA를 공부하며 "이 설계가 6 Pillars 중 어디에 해당하는가"를 매번 자문하면, SAP 단계의 trade-off 판단 문제를 훨씬 수월하게 풀 수 있습니다.
{{< /callout >}}

## 배경별 추천 진입점

{{< cards >}}
  {{< card link="fundamentals" title="IT 경험이 거의 없는 입문자" subtitle="AWS 기초부터 차근차근 — Cloud Practitioner 수준 먼저" icon="user" >}}
  {{< card link="saa" title="개발자 / 인프라 엔지니어" subtitle="기초는 건너뛰고 SAA-C03부터 바로 시작" icon="code" >}}
  {{< card link="sap" title="SAA 합격자 / 실무 경험자" subtitle="SAP-C02 심화와 Well-Architected 실전 적용으로 직행" icon="badge-check" >}}
{{< /cards >}}

## 24주 학습 플랜 (3~6개월 단기 집중)

| 주차 | 내용 |
|---|---|
| 1~4주차 | **Phase 1: AWS 기초** — Cloud Practitioner 수준 개념, Free Tier 계정 생성, 첫 Hands-on 프로젝트 |
| 5~12주차 | **Phase 2: SAA-C03** — 핵심 서비스와 아키텍처 패턴 학습, Hands-on 프로젝트 4종, SAA 합격 |
| 13~20주차 | **Phase 3: SAP-C02** — 엔터프라이즈 설계, 멀티 계정 구조, 비용/마이그레이션 전략 |
| 21~24주차 | **Phase 4: 심화 + 실전** — 심화 프로젝트, Practice Exam 반복, SAP 합격 |

매주 루틴은 이론 1~2시간 + Hands-on 1~2시간 + 주 1회 Practice Exam과 오답 분석을 기본으로 하고, 각 Phase 종료 시점에 작은 아키텍처를 직접 구축하는 것을 권장합니다. 자세한 실습 가이드는 [실습 (Labs)](/labs) 섹션을 참고하세요.

## 전체 섹션

{{< cards >}}
  {{< card link="fundamentals" title="① AWS 기초" subtitle="핵심 서비스 개요, Free Tier, 첫 Hands-on" icon="cloud" >}}
  {{< card link="saa" title="② Solutions Architect Associate" subtitle="고가용성·보안·성능/비용 아키텍처, SAA-C03" icon="academic-cap" >}}
  {{< card link="sap" title="③ Solutions Architect Professional" subtitle="멀티 계정, 마이그레이션, 비용 통제, DR/네트워킹" icon="badge-check" >}}
  {{< card link="well-architected" title="④ Well-Architected Framework" subtitle="6 Pillars 실전 적용" icon="scale" >}}
  {{< card link="exam-prep" title="⑤ 시험 전략 & 리소스" subtitle="SAA/SAP 시험 정보, Practice Exam, 추천 자료" icon="clipboard-check" >}}
{{< /cards >}}
