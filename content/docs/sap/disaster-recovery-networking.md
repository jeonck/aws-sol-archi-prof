---
title: "재해 복구와 네트워킹"
weight: 4
---

"리전 전체가 장애를 일으키면 우리 서비스는 얼마나 빨리 복구되는가?"는 SAP-C02에서 가장 자주 등장하는 질문 유형입니다. 이 질문에 답하려면 재해 복구(DR) 전략과, 여러 계정·리전·온프레미스를 안전하게 연결하는 네트워킹 설계를 함께 이해해야 합니다.

## RTO와 RPO: DR 전략의 출발점

DR 전략을 선택하기 전에 반드시 정의해야 하는 두 지표가 있습니다.

- **RTO (Recovery Time Objective)**: 장애 발생 후 서비스가 복구되기까지 허용 가능한 최대 시간
- **RPO (Recovery Point Objective)**: 장애 발생 시 허용 가능한 최대 데이터 손실 범위(시간 기준)

RTO/RPO가 짧을수록(예: 몇 분 이내) 더 많은 비용과 운영 복잡성이 필요합니다. 비즈니스가 실제로 감당 가능한 다운타임과 데이터 손실 범위를 먼저 합의한 뒤 전략을 골라야 과도한 투자를 피할 수 있습니다.

## DR 전략 4단계

AWS는 비용과 복구 속도의 trade-off에 따라 4가지 DR 전략을 제시합니다.

1. **Backup and Restore**: 데이터를 정기적으로 백업(S3, AWS Backup)해두고, 장애 시 백업으로부터 인프라를 새로 구성. 비용은 가장 낮지만 RTO/RPO가 가장 길음 (시간~하루 단위)
2. **Pilot Light**: 핵심 데이터(DB 복제본 등)만 보조 리전에서 항상 최신 상태로 유지하고, 나머지 인프라는 장애 시점에 빠르게 기동. RTO는 수십 분 단위
3. **Warm Standby**: 보조 리전에 축소된 규모로 전체 스택을 항상 가동. 장애 시 Auto Scaling으로 용량만 확장. RTO는 수 분 단위
4. **Multi-Site Active/Active**: 두 리전 모두에서 동시에 운영 트래픽을 처리. Route 53으로 트래픽을 분산하고 한쪽이 죽으면 즉시 페일오버. RTO/RPO가 거의 0에 가깝지만 비용과 데이터 동기화 복잡성이 가장 높음

{{< callout type="info" >}}
**시험에서 자주 쓰이는 패턴**: 문제에 등장하는 예산 제약과 RTO/RPO 요구사항을 먼저 읽고 4단계 중 어디에 해당하는지 매칭하세요. "최소 비용"이라는 키워드가 있으면 Backup and Restore, "수 분 내 복구"이면 Warm Standby 이상을 가리킵니다.
{{< /callout >}}

## Transit Gateway: 네트워크의 허브

계정과 VPC가 늘어나면 VPC Peering만으로는 N개 VPC 사이에 N(N-1)/2개의 연결을 일일이 관리해야 하는 문제가 생깁니다. **Transit Gateway**는 모든 VPC와 온프레미스 연결이 하나의 중앙 허브를 거치도록 만들어, 연결 관리를 단순화합니다.

- 여러 리전의 Transit Gateway를 피어링하여 글로벌 네트워크 구성 가능
- Route Table을 통해 VPC 간 통신을 세밀하게 제어 (예: Production VPC는 Security VPC만 경유 가능)
- Direct Connect Gateway, VPN 연결과도 통합되어 온프레미스-클라우드 하이브리드 구조의 중심이 됨

## Direct Connect와 Site-to-Site VPN

온프레미스 데이터센터를 AWS와 연결하는 두 가지 주요 방법입니다.

- **Direct Connect**: 전용 회선으로 AWS와 물리적으로 직접 연결. 안정적인 대역폭과 낮은 지연시간을 제공하지만 구축에 몇 주가 소요됨. 이중화를 위해 두 개의 다른 위치에서 두 회선을 구성하는 것이 권장 패턴
- **Site-to-Site VPN**: 인터넷 위에 IPsec 터널을 구성. 구축이 빠르고(분 단위) 비용도 낮지만 인터넷 대역폭에 의존하므로 성능이 가변적

실무에서는 Direct Connect를 주 연결로, Site-to-Site VPN을 페일오버 백업 경로로 함께 구성하는 것이 일반적입니다.

{{< callout type="warning" >}}
Multi-Region DR 구조는 네트워킹 설계 없이는 완성되지 않습니다. 보조 리전으로 페일오버했을 때 온프레미스나 다른 VPC와의 연결까지 함께 전환되도록, Transit Gateway와 Direct Connect Gateway 구성을 DR 전략과 같이 설계해야 합니다. 이 신뢰성(Reliability) 관점은 [Well-Architected: 신뢰성 기둥](../../well-architected/reliability-pillar/)에서 더 자세히 다룹니다.
{{< /callout >}}
