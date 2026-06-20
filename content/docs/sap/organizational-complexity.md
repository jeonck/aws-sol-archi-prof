---
title: "조직 복잡성 관리"
weight: 1
---

스타트업 시절에는 AWS 계정 하나로 충분합니다. 하지만 조직이 커지면 "개발팀과 운영팀이 같은 계정을 쓰다가 사고를 낸다", "보안팀이 모든 계정의 규정 준수 여부를 일일이 확인할 수 없다" 같은 문제가 반드시 생깁니다. SAP-C02는 이런 멀티 계정 환경을 안전하고 일관되게 운영하는 방법을 묻습니다.

## AWS Organizations: 계정을 묶는 기본 단위

**Organizations**는 여러 AWS 계정을 하나의 조직 구조로 묶어 관리하는 서비스입니다. 계정들을 OU(Organizational Unit)라는 트리 구조로 그룹화하고, 결제를 통합(Consolidated Billing)하며, 정책을 OU 단위로 일괄 적용할 수 있습니다.

일반적인 OU 구조는 다음과 같이 설계합니다.

- **Security OU**: 로그 집계 계정, 보안 도구 계정 — 다른 팀의 접근을 엄격히 제한
- **Infrastructure OU**: 네트워킹 공유 계정(Transit Gateway, Direct Connect 허브)
- **Workloads OU**: Production, Staging, Development 하위 OU로 세분화
- **Sandbox OU**: 개인 학습/실험용 계정 — 강한 SCP로 비용 폭증 방지

## Control Tower와 Landing Zone

**Control Tower**는 Organizations, IAM Identity Center, Config, CloudTrail을 조합해 멀티 계정 환경의 모범 사례를 자동으로 구성해주는 관리형 서비스입니다. Control Tower가 만들어내는 표준화된 멀티 계정 구조를 **Landing Zone**이라고 부릅니다.

Landing Zone은 새 계정을 생성할 때마다 다음을 자동으로 적용합니다.

1. 계정 생성과 동시에 표준 OU에 배치
2. 필수 가드레일(Guardrail) 적용 — 예: "특정 리전 사용 금지", "퍼블릭 S3 버킷 금지"
3. 중앙 로깅 계정으로 CloudTrail/Config 로그 자동 전송
4. IAM Identity Center를 통한 SSO 접근 권한 부여

직접 Landing Zone을 설계하는 것(Custom Landing Zone)도 가능하지만, 대부분의 조직은 Control Tower로 시작해 필요한 부분만 커스터마이징하는 것이 운영 부담을 줄이는 길입니다.

{{< callout type="info" >}}
**핵심 구분**: Organizations는 "계정을 묶는 뼈대"이고, Control Tower는 "그 뼈대 위에 모범 사례를 자동으로 얹어주는 도구"입니다. SAP 시험에서 "수동으로 멀티 계정 거버넌스를 구축하는 작업을 줄이려면?"이라는 문제가 나오면 Control Tower가 정답일 확률이 높습니다.
{{< /callout >}}

## SCP로 권한의 상한선 정하기

**SCP**(Service Control Policy)는 OU나 계정 단위로 적용되는 정책으로, IAM 정책과 달리 권한을 "부여"하지 않고 권한의 **상한선**을 정의합니다. 즉 SCP가 허용하지 않은 API 호출은 해당 계정의 루트 사용자를 포함해 누구도 실행할 수 없습니다.

대표적인 SCP 활용 패턴은 다음과 같습니다.

- 특정 리전(예: 서울, 버지니아 북부)을 제외한 모든 리전에서의 리소스 생성 차단
- 루트 사용자의 액세스 키 생성 차단
- 특정 OU(Sandbox)에서 고비용 인스턴스 유형(예: GPU 인스턴스) 생성 차단
- 보안 계정에서 CloudTrail 로깅을 비활성화하려는 시도 차단

SCP는 명시적 Deny가 항상 명시적 Allow보다 우선한다는 점에서 IAM 정책 평가 로직과 동일하지만, **SCP 자체에는 Allow가 없으면 아무것도 허용되지 않는** 기본 동작(Deny List 모드의 FullAWSAccess 정책이 기본 첨부됨)을 정확히 이해해야 합니다.

{{< callout type="warning" >}}
SCP는 마스터(관리) 계정에는 적용되지 않습니다. 마스터 계정은 워크로드를 직접 운영하지 않고 결제와 거버넌스 관리 전용으로만 사용하는 것이 권장 패턴입니다.
{{< /callout >}}

## 마이그레이션 전략과의 연결

조직 구조를 설계한 다음 단계는 보통 기존 워크로드를 이 멀티 계정 구조 안으로 옮기는 작업입니다. 이어지는 [마이그레이션과 현대화](../migration-modernization/)에서 6R 전략과 실제 이전 도구들을 다룹니다.
