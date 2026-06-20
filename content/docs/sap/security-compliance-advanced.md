---
title: "고급 보안과 컴플라이언스"
weight: 5
---

SAA 수준에서는 "IAM으로 최소 권한을 부여한다", "KMS로 데이터를 암호화한다" 정도를 알면 충분했습니다. SAP 수준에서는 그 위에 "수백 개 계정의 보안 상태를 어떻게 한눈에 파악하고, 규정 준수를 어떻게 자동으로 강제하는가"라는 질문이 추가됩니다.

## KMS 심화: 키 정책과 멀티 리전 키

**KMS**(Key Management Service)는 암호화 키를 생성·관리하는 서비스입니다. SAP 수준에서는 다음 두 가지를 깊게 이해해야 합니다.

**키 정책(Key Policy)**: KMS 키에 대한 접근 제어는 IAM 정책만으로 끝나지 않습니다. 모든 KMS 키는 반드시 키 정책을 가지며, 이 정책이 "누가 이 키를 사용할 수 있는가"의 1차 관문입니다. IAM 정책으로 권한을 부여했더라도 키 정책이 허용하지 않으면 접근이 거부됩니다. 다른 계정의 사용자가 키를 사용하게 하려면(Cross-account) 키 정책에서 명시적으로 허용해야 합니다.

**멀티 리전 키(Multi-Region Keys)**: 동일한 키 자료를 여러 리전에 복제해두는 기능입니다. 기본 키(Primary Key)와 복제 키(Replica Key)는 같은 키 ID를 공유하므로, 한 리전에서 암호화한 데이터를 다른 리전에서 복호화할 수 있습니다. Multi-Region Active/Active DR 구조에서 암호화된 데이터를 리전 간 복제할 때 필수적인 기능입니다.

{{< callout type="info" >}}
멀티 리전 키는 "키를 복사"하는 것이 아니라 "같은 키를 여러 리전에서 사용 가능하게 만드는" 것입니다. 일반 KMS 키는 리전 경계를 넘지 않으므로, Cross-Region 복제 아키텍처에서 KMS로 암호화된 데이터(예: RDS 스냅샷, EBS 볼륨)를 다루려면 반드시 멀티 리전 키를 고려해야 합니다.
{{< /callout >}}

## GuardDuty: 위협 탐지

**GuardDuty**는 VPC Flow Logs, CloudTrail, DNS 로그를 머신러닝으로 분석해 비정상적인 행위(예: 알려진 악성 IP와의 통신, 비정상적인 API 호출 패턴, 암호화폐 마이닝 흔적)를 탐지하는 위협 탐지 서비스입니다. Organizations와 연동하면 모든 멤버 계정의 탐지 결과를 보안 계정(Delegated Administrator)에서 한 번에 모아볼 수 있습니다.

## Security Hub: 보안 상태의 단일 창구

**Security Hub**는 GuardDuty, Inspector, Macie, Config 등 여러 보안 서비스의 탐지 결과를 표준화된 형식(AWS Security Finding Format)으로 모아 보여주는 대시보드입니다. CIS AWS Foundations Benchmark, PCI DSS 같은 보안 표준에 대한 준수 점수도 자동으로 계산해줍니다.

## AWS Config로 컴플라이언스 자동화

**Config**는 AWS 리소스의 구성 상태를 지속적으로 기록하고, 정해진 규칙(Config Rule)에 위반되는 리소스를 자동으로 탐지합니다. 더 나아가 **Config Conformance Pack**으로 여러 규칙을 묶어 조직 전체에 일괄 배포할 수 있습니다.

자동 교정(Auto-Remediation)까지 구성하면 컴플라이언스 위반을 사람이 수동으로 고치지 않고 시스템이 자동으로 복구하게 만들 수 있습니다. 예를 들어 다음과 같은 흐름이 가능합니다.

1. Config Rule이 "퍼블릭으로 열린 S3 버킷"을 탐지
2. EventBridge가 이 탐지 이벤트를 받아 Lambda 함수를 트리거
3. Lambda가 자동으로 버킷 정책을 수정해 퍼블릭 접근 차단
4. Security Hub에 교정 완료 상태를 기록

{{< callout type="warning" >}}
GuardDuty, Security Hub, Config는 각각 "탐지", "통합 표시", "구성 준수 검증"이라는 다른 역할을 합니다. 시험에서 이 세 서비스를 혼동하지 않도록, "어떤 행위(activity)가 의심스러운가"는 GuardDuty, "전체 보안 점수와 표준 준수는 어떤가"는 Security Hub, "리소스 설정이 정책을 위반했는가"는 Config로 구분해서 기억하세요.
{{< /callout >}}

## 심층 방어 관점에서 보기

이런 도구들을 개별적으로 켜는 것보다 중요한 것은, 이들이 함께 만들어내는 "심층 방어(Defense in Depth)" 체계를 이해하는 것입니다. 이 관점은 [Well-Architected: 보안 기둥](../../well-architected/security-pillar/)에서 더 구조적으로 다룹니다.
