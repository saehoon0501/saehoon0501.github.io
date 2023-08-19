---
title: Serverless
date: 2023-08-11 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-serverless]
---

## Serverless

Serverless in AWS

- **Lambda**
- **DynamoDb**
- **Cognito**
- API Gateway
- S3
- SNS & SQS
- Kinesis Data Firehose
- Aurora
- Step Functions
- Fargate

### Lambda

Virtual Function으로 짧은 실행 시간을 가진다. 이를 통해 작업이 필요할 때에 즉시 수행하고 종료하여 소모되는 대기 시간없이 원하는 작업을 수행 가능하다. Scaling은 자동적으로 진행된다.  
RAM 또는 CPU 중 하나를 업그레이드하면 나머지 하나도 같이 업그레이드 됨을 참고한다.  
Lambda Runtime API를 구현한 container를 통해 Lambda를 생성할 수도 있다.

Limits:

- Execution
  Memory 할당량 128MB ~ 10GB(1 MB씩 증가)  
  최대 15분동안 작업 수행가능  
  Disk Capacity 512MB ~ 10GB
- Deployment
  압축된 50MB 크기의 lambda function 배포 가능, 압축되지 않으면 최대 250MB

CloudFront Functions & Lambda@Edge  
Edge location에서 작업이 수행될 필요가 있을 때 위 두개 중 하나를 선택해 사용한다.  
Edge Function은 User와 가장 가까운 Edge Location에서 수행되며 이를 위해 작성한 코드를 CloudFront Distribution에 부착한다.

| ---------           | CloudFront Functions | Lambda@Edge                    |
| ------------------- | -------------------- | ------------------------------ |
| Runtime             | JavaScript           | Nodejs, Python                 |
| #of Reqs            | Millions             | Thousands                      |
| can be triggered by | Viewer Req/Res       | Viewer Req/Res, Origin Req/Res |
| Max Execution Time  | < 1ms                | 5~10 sec                       |

Viewer와 Origin에서 오는 모든 Traffic에 대한 작업이 필요하면 Lambda@Edge를 사용한다.  
만약 수행에 필요한 시간이 적은 Traffic이 많이 들어온다면 CloudFront Function을 사용한다.  
수행에 좀더 많은 사양과 시간이 필요하다면 Lambda@Edge를 사용한다.

Lambda in VPC  
Lambda는 기본적으로 VPC 밖에 존재하기에 VPC 내부에 있는 resource에는 접근할 수 없다.
만약 이를 가능케하고 싶다면 VPC ID와 subnet을 Lambda에 할당하고 ENI를 부착한다. 그러면 VPC 내 접근 가능

Lambda RDS Proxy  
만약 Lambda가 직접적으로 DB와 연결된다면 작업량에 따라 DB에 너무나 많은 Lambda가 Req를 보내 부하가 올 수 있다.  
이를 방지하기 위해 RDS Proxy와 Lambda를 연결하여 DB의 효율성을 보장한다.  
RDS Proxy와 Lambda를 연결하기 위해서는 반드시 VPC 내 Lambda를 생성해야 한다. RDS Proxy는 항상 VPC내에만 존재

RDS에서 Lambda Invoke  
PostgreSQL과 Aurora에서 Event를 보내 Lambda를 호출 할 수 있다. 이를 위해 DB Instance에서는 Lambda에 대한 접근 권한을 부여 받아야 한다.  
이를 통해 Lambda에서는 Event를 trigger한 Data에 대한 정보를 받아 볼 수 있다.  
만약 SNS나 SQS 거쳐 Lambda를 호출 할 시 Event에 대한 정보만을 Lambda에서 볼 수 있으며 Data 자체에 대한 정보는 보지 못한다.

### Dynamo DB

single-digit ms의 고성능을 보여 NoSQL DB이다. Multi AZ 지원  
최대 Data size 400kB  
attribute가 쉽게 추가될 수 있어 Rapidly evolving Schema에 대해 사용하기 좋다.  
TTL을 통해 오래된 Data가 자동 삭제되도록 할 수 있다. -> web session 보관하기 용이

Read/Write Capacity

- Provisioned Mode: 미리 RCU(Read Capacity Units)과 WCU를 provision하여 사용한다. 자동 scaling을 지원
- On-Demand Mode: unpredictable, steep sudden spike의 경우에 사용.

DAX(Dynamo Accelerator)  
Read Congestion에 대해 Caching을 통한 성능 개선을 이룬다.  
Cached Data에 대해 Microseconds Latency를 가진다.  
이를 통해 특정 Object만을 Caching하거나 Query를 Caching할 수 있다.  
만약 Aggregation Result를 Caching하기 원하면 ElasticCache를 고려한다.

Stream Processing  
DynamoDB의 경우 24hr retention을 가지는 제한된 수의 consumer를 두는 Stream Processing service를 가진다.  
중간 Processing 과정에서 Lambda를 사용할 수 있다.  
Kinesis의 경우 1y의 retention 그리고 많은 수의 consumer를 가진다.

Disaster Recovery

- Point-In-Time-Recovery
  지속적으로 그때 그때 Backup이 이뤄짐, 이를 통해 S3에 Data를 Export할 수 있다.(S3에서 Import는 이와 상관없이 가능)
- On-Demand Backups
  원할 때 Full Backup을 진행하며 Long Retention period를 가진다.

### API Gateway

AWS Service를 API endpoint로 노출하여 외부에서 사용할 수 있게 관리한다.
Authentication과 Authorization을 진행할 수 있음

Integration High Level

- Lambda Function
- HTTP
- AWS Service

Endpoint Types:

- Edge-Optimized(default): Req가 CloudFront Edge를 통해 들어온다. API Gateway는 한 Region에만 존재한다.(us-east-1)
- Regional
- Private

Security

- User Authentication through: IAM Role, Cognito, Custom Authorizer
- Custom Domain Name HTTPS
  만약 Edge-Optimized를 사용한다면 certificate는 us-east-1에 있어야 한다.  
  만약 Regional이라면 해당 Region에 certificate이 있어야 한다.  
  Route 53에서 CNAME 또는 Alias 설정이 필수다.

### Step Functions

Visual적으로 workflow를 생성할 수 있음
이를 통해 Human Approval feature도 구현 가능

### Cognito

Cognito User Pool는 User를 저장하는 DB로 login을 하면 외부 User에게 API를 사용할 수 있는 Token을 부여한다.  
User는 이 Token을 API Gateway 또는 ALB에 보내면 Resource에 대한 접근이 가능해지는 것이다.
<br>
만약 User가 Service에 직접 Req를 보내고 싶다면 AWS Credential이 필요하다. 이를 위해 Cognito Identity Pool이 존재한다.  
User가 OAuth나 User Pool에 직접 로그인을 통해 Token을 받고 해당 Token을 Cognito Identity Pool로 보내면 임시 Credential를 받을 수 있다.  
이를 통해 aws Service에 직접 req가 가능하다.  
Dynamo DB에서는 Row Level에 대한 Security를 설정할 수 있어 Cognito Identity Pool에서 임시 Credential을 받은 유저는 특정 데이터만 접근하도록 할 수 있다.
