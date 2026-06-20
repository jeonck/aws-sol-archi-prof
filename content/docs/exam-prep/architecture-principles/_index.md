---
title: "AWS 아키텍처 핵심 원리와 제약"
weight: 8
---

AWS 아키텍처를 설계하고 자격증 시험(SAP-C02 등)을 준비할 때, 서비스의 개별 기능만큼 중요한 것이 바로 **AWS의 아키텍처 설계 철학과 시스템적 제약**(Constraints)을 이해하는 것입니다.

이 섹션에서는 단순한 스펙 암기를 넘어, AWS가 특정 아키텍처를 권장하거나 금지하는 원리적 배경과 운영 철학을 개념도와 함께 다룹니다.

## 다루는 주제

1. **[서비스 할당량(Quota) 자동 증설의 한계와 제어 철학](service-quotas-limitations)** — Lambda 등을 통한 할당량 즉시 자동 증설이 불가능한 이유와 아키텍처적 대안
2. **[마크다운 볼드체 작성 규칙 가이드](markdown-bold-rules)** — 문장 부호(따옴표)와 볼드체 기호 인접 시 발생하는 마크다운 렌더링 오류 해소 가이드
3. **[자동화 금지 영역 4가지와 판단 체크리스트](automation-control-boundaries)** — IAM·보안 그룹, Stateful 마이그레이션, 비용 자동 삭제 등 자동화가 함정이 되는 패턴과 4가지 판단 기준
4. **[API Gateway 429 오류와 Throttling 전략](api-gateway-throttling)** — 429 vs 502 오류의 원인 차이, Usage Plan 조정·재시도·캐싱을 통한 해결 전략
5. **[Systems Manager Run Command: 접속이 아니라 폴링이다](ssm-run-command)** — 인바운드 없이 서버를 관리하는 폴링 구조와 IAM Role 기반 신뢰 모델
