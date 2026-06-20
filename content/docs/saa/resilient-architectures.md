---
title: "복원력 있는 아키텍처"
weight: 1
---

복원력(Resilience)은 장애가 발생해도 서비스가 멈추지 않거나, 멈추더라도 빠르게 회복되는 능력입니다. SAA-C03에서 가장 자주 등장하는 설계 축이며, 핵심 도구는 Multi-AZ 배치, Auto Scaling, 로드 밸런싱 세 가지로 요약됩니다.

## High Availability와 Multi-AZ

AWS 리전은 여러 개의 가용 영역(Availability Zone, AZ)으로 구성되며, 각 AZ는 독립된 전력·네트워크·물리적 위치를 가집니다. 하나의 AZ가 장애를 겪어도 다른 AZ는 영향을 받지 않도록 설계하는 것이 **Multi-AZ**의 핵심입니다.

- **EC2**: 동일 애플리케이션의 인스턴스를 최소 2개 이상의 AZ에 분산 배치합니다.
- **RDS Multi-AZ**: 기본(Primary) DB 인스턴스와 동기 복제되는 대기(Standby) 인스턴스를 다른 AZ에 자동으로 유지하며, 장애 발생 시 자동으로 Failover합니다.
- **S3**: 버킷 자체가 리전 내 여러 AZ에 자동으로 복제되므로 별도 설정 없이 높은 내구성을 가집니다.

{{< callout type="info" >}}
**가용 영역(AZ) ≠ 리전(Region)**. 리전 간 복제는 재해 복구(DR) 영역이며, AZ 간 복제는 같은 리전 안에서의 고가용성 확보입니다. 이 둘을 구분하지 못하면 SAA 문제에서 흔히 함정에 빠집니다.
{{< /callout >}}

## Auto Scaling

**Auto Scaling Group**(ASG)은 정의된 조건(CPU 사용률, 요청 수 등)에 따라 EC2 인스턴스 수를 자동으로 늘리거나 줄입니다. 복원력 관점에서 ASG는 두 가지 역할을 합니다.

1. **장애 복구** — 헬스 체크에 실패한 인스턴스를 자동으로 종료하고 새 인스턴스로 교체합니다.
2. **트래픽 대응** — 부하가 늘어나면 인스턴스를 추가하고, 줄어들면 다시 회수하여 항상 적정 용량을 유지합니다.

ASG는 단일 AZ가 아니라 **여러 AZ에 걸친 서브넷**을 대상으로 구성해야 진짜 복원력을 확보할 수 있습니다. 하나의 AZ에만 ASG를 묶어두면 AZ 장애 시 전체 서비스가 중단됩니다.

## ELB / ALB로 트래픽 분산하기

**ELB**(Elastic Load Balancer)는 들어오는 트래픽을 여러 인스턴스로 분산시켜 단일 인스턴스 장애가 전체 서비스 장애로 이어지지 않게 합니다. SAA에서 자주 등장하는 유형은 다음과 같습니다.

- **ALB**(Application Load Balancer) — HTTP/HTTPS 계층(L7)에서 동작하며, URL 경로나 호스트 헤더 기준으로 라우팅할 수 있어 마이크로서비스나 다중 애플리케이션 구조에 적합합니다.
- **NLB**(Network Load Balancer) — TCP/UDP 계층(L4)에서 동작하며, 매우 낮은 지연 시간과 고정 IP가 필요한 경우에 사용합니다.

ALB는 자체적으로 여러 AZ에 걸쳐 배치되며, Target Group을 통해 ASG와 자연스럽게 연결되어 "트래픽 분산 + 자동 확장 + 장애 인스턴스 교체"가 하나의 구조로 동작합니다.

## 복원력 설계 패턴 요약

| 구성 요소 | 단일 장애점 제거 방법 |
|---|---|
| 컴퓨팅 | 여러 AZ에 걸친 ASG + ALB |
| 데이터베이스 | RDS Multi-AZ, 읽기 부하는 Read Replica로 분산 |
| 스토리지 | S3(자동 다중 AZ 복제) |
| DNS | Route 53 Health Check + Failover 라우팅 |

{{< callout type="warning" >}}
"고가용성"을 "재해 복구"와 혼동하지 마세요. Multi-AZ는 같은 리전 내 가용성 확보이고, 리전 전체 장애에 대비하려면 별도의 Multi-Region DR 전략이 필요합니다. DR은 [SAP 도메인 1](../../sap/domain1-organizational-complexity/)에서 더 깊이 다룹니다.
{{< /callout >}}
