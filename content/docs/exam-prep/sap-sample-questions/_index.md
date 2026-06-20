---
title: "SAP-C02 샘플 문제 10선"
weight: 7
---

AWS Certified Solutions Architect - Professional (SAP-C02) 시험의 실전 감각을 익히기 위한 샘플 문제 10선입니다. 실제 시험과 동일한 유형의 시나리오형 문제로 구성되어 있으며, 각 문제 하단의 **정답 및 해설 보기**를 클릭하여 정답과 상세한 아키텍처 분석을 확인할 수 있습니다.

{{< callout type="info" >}}
각 문제에 대해 '**지문의 마지막 문장 먼저 읽기 ➔ 선지 스캔 ➔ 힌트 단어 찾기**' 전략을 적용하여 논리적으로 정답을 도출하는 상세 풀이 과정은 **[스키밍 기법을 활용한 10선 상세 풀이 과정](skimming-solutions/)** 에서 확인할 수 있습니다.
{{< /callout >}}

---

### Q1. Cost Control and Autonomy in a Multi-Account Environment

**시나리오:**
A company has many AWS accounts that individual business groups own. One of the accounts was recently compromised. The attacker launched a large number of instances, resulting in a high bill for that account. The company addressed the security breach, but a solutions architect needs to develop a solution to prevent excessive spending in all accounts. Each business group wants to retain full control of its AWS account. Which solution should the solutions architect recommend to meet these requirements?

**선택지:**
*   **A)** Use AWS Organizations. Add each AWS account to the management account. Create an SCP that uses the `ec2:instanceType` condition key to prevent the launch of high-cost instance types in each account.
*   **B)** Attach a new customer-managed IAM policy to an IAM group in each account. Configure the policy to use the `ec2:instanceType` condition key to prevent the launch of high-cost instance types. Place all the existing IAM users in each group.
*   **C)** Turn on billing alerts for each AWS account. Create Amazon CloudWatch alarms that send an Amazon Simple Notification Service (Amazon SNS) notification to the account administrator whenever the account exceeds a designated spending threshold.
*   **D)** Turn on AWS Cost Explorer in each account. Review the Cost Explorer reports for each account on a regular basis to ensure that spending does not exceed the desired amount.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: C**

**해설:**
결제 알람(Billing alarms)을 사용하면 개별 비즈니스 그룹의 제어권을 뺏지 않고도 과도한 지출에 대한 알림을 제공할 수 있습니다. 옵션 A와 B는 비즈니스 그룹이 계정 제어권을 유지하려는 요구사항에 반하며, 인스턴스의 대량 실행 자체를 막지는 못합니다. 옵션 D는 수동 프로세스이므로 즉각적인 알림을 제공하기 어렵습니다.

</details>

---

### Q2. Integrating Third-Party Monitoring in a Multi-Account Environment

**시나리오:**
A company has multiple AWS accounts in an organization in AWS Organizations. The company has integrated its on-premises Active Directory with AWS Single Sign-On (AWS SSO) to grant Active Directory users least privilege permissions to manage infrastructure across all the accounts. A solutions architect must integrate a third-party monitoring solution that requires read-only access across all AWS accounts. The monitoring solution will run in its own AWS account. What should the solutions architect do to provide the monitoring solution with the required permissions?

**선택지:**
*   **A)** Create a user in an AWS SSO directory. Assign a read-only permissions set to the user. Assign all AWS accounts that need monitoring to the user. Provide the third-party monitoring solution with the user name and password.
*   **B)** Create an IAM role in the organization's management account. Allow the AWS account of the third-party monitoring solution to assume the role.
*   **C)** Invite the AWS account of the third-party monitoring solution to join the organization. Enable all features.
*   **D)** Create an AWS CloudFormation template that defines a new IAM role for the third-party monitoring solution. Specify the AWS account of the third-party monitoring solution in the trust policy. Create the IAM role across all linked AWS accounts by using a stack set.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: D**

**해설:**
AWS CloudFormation StackSets를 사용하면 한 번의 작업으로 여러 계정에 IAM 역할을 배포할 수 있습니다. 옵션 A의 SSO 자격 증명은 일시적이어서 세션이 만료되면 다시 로그인해야 하며, 옵션 B는 관리 계정에만 액세스를 허용하게 됩니다. 옵션 C는 조직에 가입한다고 해서 다른 계정에 대한 액세스 권한이 자동으로 부여되는 것이 아니므로 오답입니다.

</details>

---

### Q3. Prerequisites for Connecting Static Websites and API Gateway

**시나리오:**
A team is building an HTML form that is hosted in a public Amazon S3 bucket. The form uses JavaScript to post data to an Amazon API Gateway API endpoint. The API endpoint is integrated with AWS Lambda functions. The team has tested each method in the API Gateway console and has received valid responses. Which combination of steps must the team complete so that the form can successfully post to the API endpoint and receive a valid response? (Select TWO.)

**선택지:**
*   **A)** Configure the S3 bucket to allow cross-origin resource sharing (CORS).
*   **B)** Host the form on Amazon EC2 rather than on Amazon S3.
*   **C)** Request a quota increase for API Gateway.
*   **D)** Enable cross-origin resource sharing (CORS) in API Gateway.
*   **E)** Configure the S3 bucket for web hosting.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: D, E**

**해설:**
CORS(교차 출처 리소스 공유)는 브라우저 보안 기능으로, 다른 도메인에 호스팅된 API에 액세스할 때 필요합니다. 따라서 API Gateway에서 CORS를 활성화(D)해야 합니다. 또한 HTML 폼을 웹사이트 엔드포인트를 통해 제공하려면 S3 버킷을 웹 호스팅용으로 구성(E)해야 합니다.

</details>

---

### Q4. Resolving API Gateway 502 (Bad Gateway) Errors

**시나리오:**
A company runs a serverless mobile app that uses Amazon API Gateway, AWS Lambda functions, Amazon Cognito, and Amazon DynamoDB. During large surges in traffic, users report intermittent system failures. The API Gateway API endpoint is returning HTTP status code 502 (Bad Gateway) errors to valid requests. Which solution will resolve this issue?

**선택지:**
*   **A)** Increase the concurrency quota for the Lambda functions. Configure Amazon CloudWatch to send notification alerts when the ConcurrentExecutions metric approaches the quota.
*   **B)** Configure notification alerts for the quota of transactions per second on the API Gateway API endpoint. Create a Lambda function that will increase the quota when the quota is reached.
*   **C)** Shard users to Amazon Cognito user pools in multiple AWS Regions to reduce user authentication latency.
*   **D)** Use DynamoDB strongly consistent reads to ensure that the client application always receives the most recent data.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: A**

**해설:**
AWS Lambda 함수가 동시성 할당량(concurrency quota)을 초과하면 API Gateway는 간헐적으로 HTTP 502 (Bad Gateway) 오류를 반환합니다. 옵션 B의 경우 API Gateway는 429 오류를 반환하게 되며, 옵션 C(인증 문제)나 D(오래된 데이터)는 502 오류의 직접적인 원인이 아닙니다.

</details>

---

### Q5. Remote Instance Management under Strict Inbound Security Group Rules

**시나리오:**
A company is launching a new web service on an Amazon Elastic Container Service (Amazon ECS) cluster. The cluster consists of 100 Amazon EC2 instances. Company policy requires the security group on the cluster instances to block all inbound traffic except HTTPS (port 443). Which solution will meet these requirements?

**선택지:**
*   **A)** Change the SSH port to 2222 on the cluster instances by using a user data script. Log in to each instance by using SSH over port 2222.
*   **B)** Change the SSH port to 2222 on the cluster instances by using a user data script. Use AWS Trusted Advisor to remotely manage the cluster instances over port 2222.
*   **C)** Launch the cluster instances with no SSH key pairs. Use AWS Systems Manager Run Command to remotely manage the cluster instances.
*   **D)** Launch the cluster instances with no SSH key pairs. Use AWS Trusted Advisor to remotely manage the cluster instances.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: C**

**해설:**
AWS Systems Manager Run Command는 인바운드 포트를 열 필요가 없습니다. 이 서비스는 기본적으로 허용되는 아웃바운드 HTTPS를 통해 작동하므로, 인바운드 443 포트만 허용해야 한다는 보안 정책을 준수하면서 인스턴스를 관리할 수 있습니다.

</details>

---

### Q6. Least-Privilege Cross-Account Access and Single Credentials Design

**시나리오:**
A company has two AWS accounts: one account for production workloads and one account for development workloads. A development team and an operations team create and manage these workloads. The company needs a security strategy that meets the following requirements:
*   Developers need to create and delete development application infrastructure.
*   Operators need to create and delete development and production application infrastructure.
*   Developers must have no access to production infrastructure.
*   All users must have a single set of AWS credentials.
Which strategy will meet these requirements?

**선택지:**
*   **A)** In the production account: Create an operations IAM group. Create an IAM user for each operator. Add the operators to the operations group. In the development account: Create a development IAM group. Create an IAM user for each developer. Add the developers to the development group. Create an IAM user for each operator and developer.
*   **B)** In the production account: Create an operations IAM group. Create an IAM user for each operator. Add the operators to the operations group. In the development account: Create a development IAM group. Create an IAM user for each developer. Add the developers to the development group. Create an IAM user for each operator. Assign these users to the development group and to the operations group in the production account.
*   **C)** In the development account: Create a shared IAM role that can create and delete development application infrastructure. Create a development IAM group. Create an operations IAM group that can assume the shared role. Create an IAM user for each developer. Assign these users to the development group. Create an IAM user for each operator. Assign these users to the development group and to the operations group.
*   **D)** In the production account: Create a shared IAM role that can create and delete application infrastructure. Add the development account to the trust policy for the shared role. In the development account: Create a development IAM group that can create and delete application infrastructure. Create an operations IAM group that can assume the shared role in the production account. Create an IAM user for each developer. Assign these users to the development group. Create an IAM user for each operator. Assign these users to the development group and to the operations group.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: D**

**해설:**
이 답변은 사용자가 제어하는 두 계정 간에 교차 계정 액세스(cross-account access)를 부여하는 표준 지침을 따릅니다. 옵션 A는 운영자에게 두 세트의 자격 증명이 필요하게 만들며, 옵션 B는 다른 계정의 IAM 그룹에 사용자를 추가할 수 없다는 점에서 불가능합니다. 옵션 C는 역할이 리소스가 있는 계정과 동일한 계정에 있어야 한다는 원칙에 어긋납니다.

</details>

---

### Q7. Reducing Costs in Big Data Pipelines and Exception Processing

**시나리오:**
A solutions architect needs to reduce costs for a big data application. The application environment consists of hundreds of devices that send events to Amazon Kinesis Data Streams. The device ID is used as the partition key, so each device gets a separate shard. Each device sends between 50 KB and 450 KB of data each second. An AWS Lambda function polls the shards, processes the data, and stores the result in Amazon S3. Every hour, another Lambda function runs an Amazon Athena query against the result data to identify outliers. This Lambda function places the outliers in an Amazon Simple Queue Service (Amazon SQS) queue. An Amazon EC2 Auto Scaling group of two EC2 instances monitors the queue and runs a 30-second process to address the outliers. The devices submit an average of 10 outlying values every hour. Which combination of changes to the application will MOST reduce costs? (Select TWO.)

**선택지:**
*   **A)** Change the Auto Scaling group launch configuration to use smaller instance types in the same instance family.
*   **B)** Replace the Auto Scaling group with a Lambda function that is invoked when messages arrive in the queue.
*   **C)** Reconfigure the devices and data stream to set a ratio of 10 devices to 1 data stream shard.
*   **D)** Reconfigure the devices and data stream to set a ratio of 2 devices to 1 data stream shard.
*   **E)** Change the desired capacity of the Auto Scaling group to a single EC2 instance.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: B, D**

**해설:**
이상치(outliers)를 처리하는 데 필요한 시간은 시간당 평균 300초에 불과하므로, Lambda 함수를 사용하여 처리(B)하면 사용한 만큼만 비용을 지불하여 EC2 인스턴스를 유지하는 것보다 비용을 크게 절감할 수 있습니다. 또한 장치와 데이터 스트림의 비율을 2:1로 재구성(D)하면 Kinesis 샤드 시간 비용을 줄일 수 있습니다. (옵션 C는 데이터 전송량이 샤드 할당량을 초과하게 됩니다)

</details>

---

### Q8. Asynchronous Decoupling and Rate Limiting for Third-Party APIs

**시나리오:**
A company operates an ecommerce application on Amazon EC2 instances behind an Application Load Balancer. The instances run in an Amazon EC2 Auto Scaling group across multiple Availability Zones. After an order is successfully processed, the application immediately posts order data to a third-party affiliate’s external tracking system that pays sales commissions for order referrals. During a successful marketing promotion, the number of EC2 instances increased from 2 to 20. The application continued to work correctly during this time. However, the increased request rate overwhelmed the third-party affiliate and resulted in failed requests. Which combination of architectural changes should a solutions architect make to ensure that the entire process functions correctly under load? (Select TWO.)

**선택지:**
*   **A)** Move the code that calls the affiliate to a new AWS Lambda function. Modify the application to invoke the Lambda function asynchronously.
*   **B)** Move the code that calls the affiliate to a new AWS Lambda function. Modify the application to place the order data in an Amazon Simple Queue Service (Amazon SQS) queue. Invoke the Lambda function from the queue.
*   **C)** Increase the timeout of the new AWS Lambda function.
*   **D)** Decrease the reserved concurrency of the new AWS Lambda function.
*   **E)** Increase the memory of the new AWS Lambda function.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: B, D**

**해설:**
Amazon SQS(B)를 사용하여 주문 데이터 게시 작업을 분리(decouple)하면 외부 시스템의 부하가 본래 애플리케이션에 영향을 주는 것을 방지할 수 있습니다. 또한 Lambda의 예약된 동시성(reserved concurrency)(D)을 낮게 설정하면 외부 시스템이 감당할 수 있는 수준으로 요청 속도를 제어하여 과부하를 막을 수 있습니다.

</details>

---

### Q9. Active-Active Multi-Region Deployment with Minimal Changes

**시나리오:**
A company has built an online ticketing web application on AWS. The application is hosted on AWS App Runner and uses images that are stored in an Amazon Elastic Container Registry (Amazon ECR) repository. The application stores data in an Amazon Aurora MySQL DB cluster. The company has set up a domain name in Amazon Route 53. The company needs to deploy the application across two AWS Regions in an active-active configuration. Which combination of steps will meet these requirements with the LEAST change to the architecture? (Select THREE.)

**선택지:**
*   **A)** Set up Cross-Region Replication to the second Region for the ECR images.
*   **B)** Create a VPC endpoint from the ECR repository in the second Region.
*   **C)** Edit the App Runner configuration by adding a second deployment target to the second Region.
*   **D)** Deploy App Runner to the second Region. Set up Route 53 latency-based routing.
*   **E)** Change the database by using Amazon DynamoDB global tables in the two desired Regions.
*   **F)** Use an Aurora global database with write forwarding enabled in the second Region.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: A, D, F**

**해설:**
아키텍처 변경을 최소화하면서 액티브-액티브 구성을 하려면, ECR 교차 리전 복제(A)로 이미지를 복사하고, 두 번째 리전에 App Runner를 배포한 뒤 Route 53 지연 시간 기반 라우팅을 설정(D)하며, Aurora 글로벌 데이터베이스(F)를 사용하여 리전 간 데이터를 관리하는 것이 가장 효율적입니다.

</details>

---

### Q10. Modernizing Legacy Stacks and Minimizing Operational Overhead

**시나리오:**
A company has deployed a multi-tier web application in the AWS Cloud. The application consists of the following tiers:
*   A Windows-based web tier that is hosted on Amazon EC2 instances with Elastic IP addresses
*   A Linux-based application tier that is hosted on EC2 instances that run behind an Application Load Balancer (ALB) that uses path-based routing
*   A MySQL database that runs on a Linux EC2 instance
All the EC2 instances are using Intel-based x86 CPUs. A solutions architect needs to modernize the infrastructure to achieve better performance. The solution must minimize the operational overhead of the application. Which combination of actions should the solutions architect take to meet these requirements? (Select TWO.)

**선택지:**
*   **A)** Run the MySQL database on multiple EC2 instances.
*   **B)** Place the web tier instances behind an ALB.
*   **C)** Migrate the MySQL database to Amazon Aurora Serverless.
*   **D)** Migrate all EC2 instance types to Graviton2.
*   **E)** Replace the ALB for the application tier instances with a company-managed load balancer.

<details>
<summary>🔍 정답 및 해설 보기</summary>

**정답: B, C**

**해설:**
웹 티어를 ALB 뒤에 배치(B)하면 가용성과 확장성을 높일 수 있습니다. 또한 MySQL 데이터베이스를 Amazon Aurora Serverless(C)로 마이그레이션하면 성능을 높이면서도 운영 오버헤드를 최소화할 수 있습니다. 옵션 D는 Windows 인스턴스가 Graviton2를 지원하지 않으므로 오답입니다.

</details>
