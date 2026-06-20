---
title: "비용 통제와 최적화"
weight: 3
---

단일 계정에서는 인스턴스 하나의 비용을 신경 쓰면 되지만, 수십 개 계정과 수천 개 리소스를 운영하는 조직에서는 "누가 무엇에 얼마를 쓰고 있는지"를 파악하는 것 자체가 큰 과제입니다. SAP-C02는 개별 할인 상품 선택뿐 아니라 조직 차원의 비용 거버넌스 설계를 함께 묻습니다.

## 할인 모델: Savings Plans vs Reserved Instances

온디맨드 요금보다 저렴하게 컴퓨팅을 약정하는 두 가지 주요 방법이 있습니다.

- **Savings Plans**: 시간당 사용 금액(예: $10/시간)을 1년 또는 3년간 약정. EC2, Fargate, Lambda에 걸쳐 유연하게 적용되며, 인스턴스 패밀리나 리전이 바뀌어도 할인이 유지됨 (Compute Savings Plans 기준)
- **Reserved Instances (RI)**: 특정 인스턴스 유형과 리전을 1년/3년 약정. Standard RI는 변경 유연성이 낮지만 할인율이 가장 높고, Convertible RI는 다른 인스턴스 유형으로 교환 가능

워크로드가 안정적이고 인스턴스 구성이 거의 바뀌지 않는다면 Standard RI로 최대 할인을 받고, 워크로드 구성이 자주 바뀌거나 Lambda/Fargate를 함께 쓴다면 Savings Plans가 운영 부담을 줄여줍니다.

{{< callout type="info" >}}
**시험 포인트**: "유연성이 필요하다"는 키워드가 나오면 Savings Plans나 Convertible RI, "특정 인스턴스를 장기간 고정적으로 쓴다"는 키워드가 나오면 Standard RI가 정답에 가깝습니다.
{{< /callout >}}

## Cost Explorer와 Compute Optimizer

**Cost Explorer**는 계정/서비스/태그별 비용 추이를 시각화하고, 향후 비용을 예측하며, RI/Savings Plans 구매를 추천해주는 도구입니다. AWS Organizations와 연동하면 모든 멤버 계정의 비용을 통합 계정에서 한 번에 조회할 수 있습니다.

**Compute Optimizer**는 실제 CloudWatch 사용률 데이터를 머신러닝으로 분석해 EC2, EBS, Lambda, Auto Scaling Group의 적정 크기를 추천합니다. "현재 인스턴스가 과도하게 프로비저닝되었는지"를 정량적으로 알려주므로, Right-sizing 작업의 시작점으로 자주 사용됩니다.

## 조직 차원의 비용 거버넌스

개별 도구를 쓰는 것을 넘어, 조직 전체에 비용 통제 문화를 정착시키려면 다음과 같은 구조가 필요합니다.

1. **태깅 전략**: `Project`, `Environment`, `Owner` 같은 필수 태그를 SCP나 Tag Policy로 강제하고, 태그가 없는 리소스 생성을 차단해 비용 추적 누락을 방지
2. **예산 알림(AWS Budgets)**: 계정별/태그별/서비스별로 예산을 설정하고 임계값(80%, 100%, 예측치 초과) 도달 시 SNS로 알림 전송
3. **Cost Anomaly Detection**: 머신러닝 기반으로 평소와 다른 비용 패턴을 자동 감지해 알림
4. **Service Catalog**: 개발팀이 직접 비용 최적화된 사전 승인 템플릿(예: 특정 인스턴스 유형으로 제한된 EC2)만 배포할 수 있도록 제한

{{< callout type="warning" >}}
태깅 전략은 "나중에 추가"하면 늦습니다. 계정과 리소스를 처음 만들 때부터 태그를 강제하지 않으면, 비용이 커진 후에는 어떤 리소스가 어떤 프로젝트 소속인지 역추적하는 데 막대한 시간이 듭니다. 조직 설계 초기 단계([조직 복잡성 관리](../organizational-complexity/))에서부터 태깅 정책을 함께 설계하세요.
{{< /callout >}}

## Right-sizing은 일회성이 아니다

비용 최적화는 한 번 점검하고 끝내는 작업이 아니라 지속적인 프로세스입니다. Compute Optimizer의 추천을 정기적으로 리뷰하고, 트래픽 패턴이 바뀌면 Savings Plans 약정 규모도 재검토해야 합니다. 이런 정기 리뷰 프로세스는 [지속적 개선](../continuous-improvement/)에서 Well-Architected Tool과 함께 다룹니다.
