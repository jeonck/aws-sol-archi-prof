---
title: "보안 아키텍처"
weight: 3
---

AWS 보안은 책임 공유 모델(Shared Responsibility Model)을 기반으로 합니다. AWS는 인프라(클라우드 자체의 보안)를 책임지고, 사용자는 클라우드 안에서의 보안(IAM 설정, 네트워크 구성, 데이터 암호화)을 책임집니다. SAA-C03은 후자, 즉 사용자가 직접 설계해야 하는 영역을 집중적으로 다룹니다.

## IAM 정책 설계

IAM 정책은 JSON 문서로 "누가(Principal) 무엇에(Resource) 어떤 작업을(Action) 허용/거부(Effect)하는가"를 정의합니다.

- **최소 권한 원칙(Least Privilege)** — 필요한 작업만 허용하고, 와일드카드(`*`)로 모든 작업을 허용하는 정책은 지양합니다.
- **역할(Role) vs 사용자(User)** — EC2, Lambda 같은 서비스가 다른 AWS 리소스에 접근해야 할 때는 액세스 키를 하드코딩하는 대신 **IAM 역할**을 부여합니다. 역할은 임시 자격 증명을 자동으로 발급하고 갱신하므로 키 유출 위험이 없습니다.
- **정책 유형** — Identity-based 정책(사용자/역할에 직접 부착)과 Resource-based 정책(S3 버킷 정책처럼 리소스에 직접 부착)을 구분해서 이해해야 합니다. 둘 다 평가 대상이 되며, 명시적 거부(Explicit Deny)가 있으면 다른 허용 정책보다 항상 우선합니다.

{{< callout type="info" >}}
SAA 문제에서 "EC2가 S3에 접근해야 한다"는 시나리오가 나오면, 액세스 키 발급이 아니라 **IAM 역할을 EC2 인스턴스 프로필에 연결**하는 것이 거의 항상 정답입니다.
{{< /callout >}}

## 네트워크 보안 — Security Group과 NACL

VPC 내부 트래픽을 통제하는 두 가지 메커니즘은 자주 비교 대상으로 등장합니다.

| 구분 | Security Group | Network ACL |
|---|---|---|
| 적용 대상 | 인스턴스(ENI) 단위 | 서브넷 단위 |
| 규칙 평가 | 허용 규칙만 존재 (Stateful) | 허용/거부 모두 존재 (Stateless) |
| 상태 추적 | 응답 트래픽 자동 허용 | 인바운드/아웃바운드 모두 명시적으로 허용 필요 |
| 용도 | 인스턴스별 세밀한 접근 제어 | 서브넷 단위의 광범위한 차단(블랙리스트 등) |

실무에서는 Security Group을 기본 방어선으로 사용하고, NACL은 특정 IP를 명시적으로 차단해야 하는 경우처럼 보조적인 용도로 사용하는 경우가 많습니다.

## WAF로 애플리케이션 계층 보호하기

**AWS WAF**(Web Application Firewall)는 SQL Injection, XSS 같은 일반적인 웹 공격 패턴을 차단하는 L7 방화벽입니다. CloudFront, ALB, API Gateway에 연결하여 사용하며, Security Group/NACL이 막지 못하는 **애플리케이션 계층의 악의적인 요청 패턴**을 룰 기반으로 필터링합니다.

- Managed Rule Groups을 사용하면 AWS나 마켓플레이스 제공 업체가 관리하는 룰셋을 바로 적용할 수 있습니다.
- Rate-based Rule로 특정 IP의 과도한 요청(예: 무차별 대입 공격)을 자동으로 차단할 수 있습니다.

## KMS 기초 — 암호화 키 관리

**KMS**(Key Management Service)는 데이터 암호화에 사용하는 키를 생성·관리하는 서비스입니다.

- S3, RDS, EBS 등 대부분의 AWS 스토리지 서비스는 KMS 키를 사용한 저장 데이터 암호화(Encryption at Rest)를 기본 옵션으로 지원합니다.
- **AWS 관리형 키**는 설정이 간단하지만 키 정책을 세밀하게 제어할 수 없고, **고객 관리형 키**(CMK)는 키 정책과 교체(rotation) 주기를 직접 통제할 수 있어 규제 요구사항이 있는 경우에 선호됩니다.
- 전송 중 암호화(Encryption in Transit, 예: TLS)와 저장 데이터 암호화(Encryption at Rest)는 서로 다른 문제이며, 둘 다 챙겨야 한다는 점이 SAA 문제에서 자주 강조됩니다.

{{< callout type="warning" >}}
보안 설계는 한 가지 도구로 완성되지 않습니다. IAM(누가 접근하는가) + Security Group/NACL(네트워크 경계) + WAF(애플리케이션 계층 공격 방어) + KMS(데이터 암호화)가 계층적으로 함께 동작해야 합니다. 이 계층적 사고는 Well-Architected의 [Security Pillar](../../well-architected/security-pillar/)와 그대로 연결됩니다.
{{< /callout >}}
