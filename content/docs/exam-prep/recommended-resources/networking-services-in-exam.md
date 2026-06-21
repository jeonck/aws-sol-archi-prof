---
title: "시험에 자주 나오는 네트워킹 서비스와 활용 사례"
weight: 3
---

SAP-C02 샘플 문제를 분석하면, 네트워킹 서비스는 단독으로 출제되기보다 **글로벌 트래픽 제어, 로드 밸런싱, 서버리스 엔드포인트 보안** 같은 구체적인 활용 사례에 묶여서 등장합니다. 서비스 이름만 아는 것으로는 부족하고, "이 서비스가 왜 이 시나리오의 정답인가"까지 연결해서 학습해야 합니다.

## 1. Amazon Route 53 — 글로벌 트래픽 관리

가장 빈번하게 등장하는 네트워킹 서비스 중 하나로, 단순히 도메인을 연결하는 것을 넘어 글로벌 아키텍처에서 트래픽을 제어하는 핵심 역할을 합니다.

- **지연 시간 기반 라우팅(Latency-based routing)**: 전 세계 여러 리전에 애플리케이션을 배포했을 때, 사용자에게 가장 낮은 지연 시간을 제공하는 리전으로 트래픽을 유도하기 위해 사용됩니다.
- **사용자 지정 도메인 및 리전 간 라우팅**: 여러 AWS 리전에 있는 리소스에 대해 하나의 커스텀 도메인을 호스팅하고 트래픽을 분산하는 데 활용됩니다.

{{< callout type="info" >}}
App Runner를 두 번째 리전에 배포하고 Route 53 지연 시간 기반 라우팅으로 Active-Active를 구성하는 패턴은 **[최소한의 변경 원칙](../../architecture-principles/least-change-active-active/)** 에서 다룹니다.
{{< /callout >}}

## 2. Application Load Balancer(ALB) — 계층 7 로드 밸런싱

고가용성과 확장성을 설계할 때 필수적으로 등장하는 서비스입니다.

- **경로 기반 라우팅(Path-based routing)**: URL 경로에 따라 트래픽을 서로 다른 대상 그룹(Target Group)으로 전달하는 데 사용됩니다.
- **확장성 및 가용성 개선**: 웹 티어 인스턴스들을 ALB 뒤에 배치함으로써 단일 접점을 제공하고, 트래픽을 효율적으로 분산하여 가용성을 높입니다.
- **보안 그룹과의 결합**: 특정 포트(예: HTTPS 443)만 허용하는 보안 그룹과 함께 구성되어 보안 가드레일을 형성합니다.

{{< callout type="info" >}}
EC2 + Elastic IP 직접 운영 구조를 ALB 뒤로 옮기는 현대화 사례는 **[운영 우수성 관점의 아키텍처 핵심 원리](../../architecture-principles/operational-excellence-modernization/)** 에서 자세히 다룹니다.
{{< /callout >}}

## 3. Amazon API Gateway — 서버리스 엔드포인트

서버리스 아키텍처의 관문 역할을 하며 네트워킹 설정 관련 문제가 자주 나옵니다.

- **CORS(교차 출처 리소스 공유) 설정**: 다른 도메인(예: S3 웹 호스팅 도메인)에서 API를 호출할 때 브라우저 보안 정책을 준수하기 위해 API Gateway 수준에서 CORS를 활성화해야 합니다.
- **할당량 및 스로틀링 관리**: 초당 요청 수 제한(10,000 rps 등) 및 동시성 제한으로 인한 502/429 오류 해결 시나리오에서 중요하게 다뤄집니다.

{{< callout type="info" >}}
429 vs 502 오류의 원인 차이와 Usage Plan 조정·재시도·캐싱 전략은 **[API Gateway 429 오류와 Throttling 전략](../../architecture-principles/api-gateway-throttling/)** 에서 다룹니다.
{{< /callout >}}

## 4. Amazon VPC Endpoint — 프라이빗 연결

비록 샘플 문제에서는 특정 제약 사항(리전 간 이미지 접근 불가)을 설명하는 예시로 등장했지만, 보안 설계를 위해 꼭 알아야 할 서비스입니다.

- **VPC 내부 연결**: 인터넷을 거치지 않고 VPC 내에서 ECR과 같은 AWS 서비스에 안전하게 접근하기 위해 사용됩니다.
- **리전 범위의 한계**: VPC 엔드포인트는 리전별 서비스이므로, 다른 리전의 리소스에 직접 접근하는 데는 한계가 있다는 점이 시험에서 함정으로 자주 등장합니다.

## 5. 기타 보안 및 주소 지정

- **Elastic IP addresses**: 기존 아키텍처에서 개별 인스턴스에 고정 IP를 부여하는 방식으로 등장하지만, 현대화 과정에서는 ALB를 통한 추상화가 권장됩니다.
- **Security Groups**: 인바운드·아웃바운드 트래픽을 포트 단위로 제어하는 기본 네트워킹 보안 도구입니다. 특히 Systems Manager Run Command가 인바운드 포트 개방 없이 아웃바운드 HTTPS(443)만으로 작동한다는 점이 강조됩니다.

{{< callout type="info" >}}
인바운드 없이 아웃바운드 HTTPS만으로 인스턴스를 관리하는 SSM Run Command의 원리는 **[Systems Manager Run Command: 접속이 아니라 폴링이다](../../architecture-principles/ssm-run-command/)** 에서 다룹니다.
{{< /callout >}}

{{< callout type="warning" >}}
**소스 외 정보**: 제공된 10개의 샘플 문제에는 직접적으로 나오지 않았지만, 실제 SAP 시험에서는 복잡한 온프레미스 연결을 위해 **AWS Direct Connect, AWS Site-to-Site VPN, AWS Transit Gateway, VPC Peering** 같은 서비스도 매우 비중 있게 다뤄집니다. 이 서비스들 간의 성능·비용·아키텍처적 차이점을 추가로 학습하는 것을 권장합니다.
{{< /callout >}}

## 더 둘러보기

서비스 조합 관점의 핵심 서비스 정리는 **[시험에 반복 출제되는 핵심 서비스 5가지 영역](../core-services-deep-dive/)**, 화이트페이퍼 기반 학습은 **[SAP 시험 대비 필수 화이트페이퍼 5선](../sap-whitepapers/)** 을 참고하세요.
