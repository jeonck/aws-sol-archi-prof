---
title: "표준 및 참고 문서"
weight: 3
---

강의와 모의고사로 패턴을 익혔다면, 결국 가장 정확한 답은 항상 AWS 공식 문서에 있습니다. 특히 Well-Architected Framework는 원문을 한 번이라도 정독해두면 SAA, SAP 전 구간의 트레이드오프 판단 문제를 푸는 기준이 됩니다.

## AWS Well-Architected Framework Whitepaper

- **링크**: https://aws.amazon.com/architecture/well-architected/
- **설명**: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability **6 Pillars**를 정의하는 AWS 공식 백서입니다. 각 Pillar는 설계 원칙(Design Principles)과 핵심 질문(예: "OPS 1. How do you determine what your priorities are?") 형식으로 구성되어 있어, 단순 요약본보다 원문을 직접 읽었을 때 시험과 실무 모두에서 훨씬 정확한 판단 기준을 얻을 수 있습니다.
- **시작 방법**: 위 링크에서 PDF를 내려받거나 웹 페이지로 바로 읽을 수 있습니다. 로드맵에서 각 Pillar를 깊이 다루는 페이지는 [Well-Architected Framework 섹션](../../docs/well-architected/)에서 확인하세요.

## AWS re:Invent 세션 영상

- **링크**: https://www.youtube.com/@AWSEventsChannel
- **설명**: 매년 re:Invent에서 발표되는 세션 중 "Well-Architected", "Architecture deep dive", "Resilience" 등의 키워드가 붙은 발표는 실제 AWS 솔루션스 아키텍트들이 현장에서 겪은 트레이드오프와 설계 근거를 구체적인 사례로 설명합니다. 텍스트 문서로는 전달되지 않는 디자인 의사결정 과정을 따라가기에 좋습니다.
- **시작 방법**: YouTube에서 "AWS re:Invent Well-Architected" 또는 관심 있는 서비스명 + "deep dive"로 검색합니다. 세션 코드(예: ARC, SEC로 시작하는 코드)로 분야를 가늠할 수 있습니다.

## AWS 공식 문서 (docs.aws.amazon.com)

- **링크**: https://docs.aws.amazon.com
- **설명**: 서비스별 사용자 가이드, API 레퍼런스, FAQ가 모여 있는 가장 권위 있는 1차 자료입니다. 강의나 블로그 글의 설명이 실제 동작과 다르거나 오래된 경우가 있으므로, 시험에서 애매하게 느껴지는 서비스 옵션이나 한도(limit)는 항상 공식 문서로 최종 확인하는 습관을 들이는 것이 좋습니다.
- **시작 방법**: 서비스명으로 검색하면 해당 서비스의 User Guide로 바로 연결됩니다. 각 서비스 페이지의 "FAQs" 탭은 시험에 자주 출제되는 한계치·과금 기준을 빠르게 확인하기에 유용합니다.
