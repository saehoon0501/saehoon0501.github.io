---
title: AWS Security & Encryption
date: 2023-08-17 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-security, aws-encryption] # TAG names should always be lowercase
---

# AWS Security & Encryption

Encryption 종류

- Encryption in flight: Client가 Data를 보내기전 Encrypt하고 server에서는 이를 받은 후 decrypt한다. 이를 활용한 protocol이 HTTPS이다.
  이를 통해 man in the middle attack을 방지할 수 있다. TLS/SSL Certificate 필요
- SSE(Server-Side Encryption): encryption에 사용되는 key가 server에서 관리되며 server에서 data를 보관할 때 Encrypt하고 Client에 보낼 때 decrypt해서 보낸다.
- CSE(Client-Side Encryption): client에서만 key를 관리하여 data를 보낼 때 encrypt해서 보내며 server는 단지 보관만 한다.

### KMS(Key Management Service)

AWS에서 'Encryption' 단어가 나오면 보통 KMS일 것이다. aws key를 대신 관리해주는 service.  
KMS에 있는 key에 대한 audit을 CloudTrail을 통해 할 수 있다.  
KMS API call을 통해 secret을 encrypt하면 코드에 작성해도 안전하다.

KMS Keys Types:

- Symmetric(AES-256 keys): Encrypt와 Decrypt에 모두 사용되는 single encryption key이다. 무조건 API를 통해서만 사용 가능
- Asymmetric(RSA & ECC key pairs): Public과 Private 두 종류의 key로 구성된 pair로, Public은 Encryption에 Private은 Decryption에 사용된다. 이를 통해 aws 외부에 있는 user가 public key를 활용해 encryption을 진행할 수 있다.

KMS에서 관리하는 Key Types:

- AWS Owned Keys: S3,SQS,Dynamo DB에서 기본 설정으로 사용되는 key
- AWS Managed Keys: 매년 자동적으로 rotation됨
- Customer managed keys in KMS: 따로 설정을 통해 매년 자동적으로 rotation되게 할 수 있음
- Customer managed keys imported(symmetric만 가능): 직접 rotation을 해야함

KMS Policy

- Default KMS Key policy: 별도로 policy를 정하지 않으면 생성되는 policy로 root user가 key에 접근 가능하다.
- Custom KMS Key policy: 특정 user, role을 명시해 key에 대한 접근을 관리할 수 있다. key에 대한 cross-account access 시 직접 다른 account에 key에 대한 permission을 부여할 수 있기에 용이하다.

Copying Snapshots accross regions  
기본적으로 KMS Key는 Region scoped  
Region A에서는 Key A를 사용해 encrypted된 EBS Volume을 Snapshot으로 생성하면 Snapshot 또한 Encrypted 되어있기에 이를 다른 Region으로 보내기 전 aws에서 decryption을 진행한다.  
Region B에서는 이러한 decrypted된 Snapshot을 Key B를 사용해 다시 Encryption을 진행하고 이를 통해 EBS Volume을 사용할 수 있다.

Copying Snapshots across accounts  
account A가 소유한 customer managed key를 통해 Snapshot을 생성하고 KMS Key Policy를 통해 cross-account 접근을 허가한다.  
Encrypted된 Snapshot을 공유하고 account B에서는 account A에 대한 key에 접근할 수 있기에 이를 decrypt한 후 다시 자신이 가진 key로 encrypt를 진행한다. 이렇게 생성된 Snapshot으로부터 EBS Volume을 생성한다.

KMS Multi-Region Keys  
동일한 Key가 다른 여러 Region에 존재할 수 있다. 이를 통해 한 Region에서 Encrypt 다른 지역에서 Decrypt가 가능하다.  
단, 각 Region에 존재하는 key는 독립적으로 관리되며 Primary와 Replica 관계를 가진다.

DynamoDB Global Table + Multi-Region Keys  
client side에서 DynamoDB에 존재하는 특정 attr을 encrypt할 수 있다. 이를 통해 다른 Region에서 민감한 정보를 지닌 attr를 보지 못하게 한다.  
만약 다른 지역에서 이를 확인하고자 한다면 Multi-Region Keys를 통해 다른 Region으로 Encryption에 사용된 Key를 복제한다.  
그리고 다른 지역에 있는 Client가 이를 활용해 attr을 Decrypt한다.  
client-side encryption을 통해 특정 field를 오직 key에 대한 API 접근이 가능한 Client만 확인할 수 있도록 할 수 있는 것이다.

Global Aurora + Multi-Region Keys
DynamoDB와 유사하다. 다만 attr이 아닌 column에 대한 Encryption과 Decryption이 진행된다.

S3 Replication  
Unencrypted 그리고 SSE-S3에 의해 Encrypted된 object는 기본적으로 복제된다. 하지만 SSE-C의 경우에는 client가 key를 관리하기에 replication이 불가능  
KMS Key를 사용하는 SSE-KMS의 경우 IAM Role을 통해 Replication source에는 decrypt 그리고 target에서는 encrypt API 접근 권한을 부여하여 Replication을 진행할 수 있다.  
Multi-Region Keys를 사용할 수도 있지만, S3에서는 이를 독립적인 Key로 취급해 여전히 먼저 Decrypt한 후 다시 Encrypt를 하여 Replication을 진행한다.

AMI Sharing Process Encrypted vis KMS  
account A에 Encrypted된 AMI의 launch permission 설정을 통해 target account에서 AMI를 사용해 EC2를 실행할 수 있다.
이 과정에서 KMS Key를 공유하여야 한다. 그리고 target에서는 자신의 key를 이용해 이를 다시 Encrypt한다.

### SSM Parameter Store

config와 secret을 안전하게 보관하는 service이다.  
KMS를 통해 저장되는 data에 대한 Decryption API를 호출할 수 있다.  
저장되는 data는 Hierarchy를 가지기에 permission 범위의 관리가 용이하다. Ex) /my-department/mp-app/dev/db-password  
Policy를 통해 저장되는 data에 대해 TTL을 설정하여 강제적으로 업데이트 또는 삭제가 가능하다.
EventBridge에서는 특정 만료 시간 전 trigger되어 알림을 보내는 등을 수행할 수 있다.

### Secrets Manager

똑같이 secret을 보관하지만 설정한 기간이 지나면 강제적으로 rotation을 할 수 있는 기능이 있음. 또한 rotation 시 자동적으로 새 secret을 생성도 가능<br>
<b>RDS에 대한 Secret이 나오면 Secret Manager를 생각한다.</b>
<br>
<br>
Multi-Region Store
Secret을 통해 다른 Region으로 복제 가능하다. primary에 대한 변화가 sync되어 replica에 반영된다.

### ACM(AWS Certificate Manager)

TLS Certificate을 관리 및 배포하는 서비스로 HTTPS protocol을 가능케 한다.
Integration이 가능한 서비스들:

- Elastic Load Balancer(ALB, NLB, CLB)
- CloudFront
- API Gateway  
  EC2에는 사용할 수 없음을 유의한다.

Requesting Public Certificates  
먼저 certificate에 추가하고자 하는 Domain name들을 추가한다.  
Validation 방법을 선택한다. DNS 방식 또는 Email 방식이 있다.  
Validation이 끝나면 Public Certificate가 자동 갱신될 수 있게 등록된다.

Importing Public Certificates  
자동 갱신이 불가능  
ACM에서 daily로 expiration event를 보내 EventBridge를 통해 Expire되기 전 알림은 받아 볼 수 있음

ALB에서 ACM을 사용 시 HTTP요청이 오면 이를 HTTPS로 redirect하여 HTTPS를 강제적으로 사용할 수 있게 할 수 있다.

### API Gateway

ACM을 위해 Custom Domain Name을 생성한다.

- Edge-Optimized(기본): Req는 CloudFront Edge location으로 가지만 API Gateway는 us-east-1 하나에만 존재하기에 Certificate 또한 해당 지역에 존재해야 한다.
- Regional: API Gateway가 존재하는 Region에 Certificate가 존재해야 한다.

### AWS WAF(Web Application Firewall)

Layer 7(HTTP)에 대한 보안을 설정한다.<br>
적용 가능한 Service들:

- ALB
- API Gateway
- CloundFront
- AppSync GraphQL API
- Cognito User pool  
  Rule group을 통해 재사용이 가능한 rule들을 설정하고 Access Control List에 추가할 수 있다.  
   Rule에는 IP, SQL injection, Rate-baed rules(초당 req를 보내는 횟수) 등이 들어있다.

ALB에 대해 WAF를 적용하기 위해서는 고정된 IP가 필요하기에 Global Accelerator를 활용한다.

### AWS Shield

DDos 공격에 특화된 보호를 제공해주는 Service이다.
이를 통해 DDos 전문 팀을 제공받고 DDos를 방지하기 위한 WAF rule들을 자동 생성할 수 있다.<br>
적용 가능한 Service들:

- EC2
- ELB(ALB, NLB, CLB)
- CloudFront
- Global Accelerator
- Route 53

### AWS Firewall Manager

Organization에 존재하는 모든 계정에 있는 rule을 관리할 수 있다.
추후 새 Resource를 생성한다면 바로 기존에 있는 Rule들을 적용할 수 있다.

### WAF vs Shield vs Firewall Manager

모두 함께 사용되어 범용적인 보호를 제공 받을 수 있다.

WAF를 통해서는

- ACL을 설정
- Resource에 대해 세밀한 보호를 하고 싶다면 WAF 하나만으로 충분

Firewall Manager를 통해서는

- WAF를 모든 계정에 적용 할 수 있고 새 Resource에 대한 자동적으로 기존 WAF 설정을 적용 가능

Shield를 통해서는

- DDos 전문 대응 팀과 진보한 레포트를 받아 볼 수 있다.

### Best Practices for DDos Resiliency

- Edge Location을 사용하는 CloudFront, Global accelerator와 DNS인 Route 53와 같은 곳에서는 DDos로부터 좀더 나은 보호를 제공할 수 있다.
- Edge Location,DNS, API Gateway 그리고 ELB를 통해 EC2나 Lambda와 같은 Resource를 외부에 노출하지 않고 Req가 도달하기전에 이를 방지하는 보호 layer를 제공할 수 있다.
- ASG와 ELB를 통해 만약 Req가 앞 Layer를 지나서 온다고 하더라도 Scaling과 Distribution을 통해 Traffic을 버틸 수 있다.
- Edge Location에 WAF를 적용을 통해 의심되는 request에 대한 detection 및 filtering이 가능하다.
- Edge Location, ELB에 Shield를 적용하여 DDos 보호에 특화된 WAF를 설정 가능하다.
- Security Group과 Network ACL을 통해 의심되는 IP에 대한 차단과 traffic에 대한 filtering이 가능하다. Elastic IP를 통해 Shield 적용도 가능

### GuradDuty

account에 존재할 수 있는 잠재 위혐을 ML을 통해 찾아내고 Event를 보낼 수 있다. 사용되는 Data로는 CloudTrail log들고 VPC Flow log, DNS Log 등이 있다.<br>
Cryptocurrency attack에 대한 보호가 가능하다.

### Inspector

EC2에 대해서는 System Manager agent를 활용해 보안 검사를 실시한다.<br>
ECS에는 Container Image에 넣어 진행한다.<br>
Lambda에 대해서도 검사 진행 가능<br>
오직 EC2, Container Image, Lambda에만 적용 가능을 기억하자.<br>

### Macie

ML 그리고 패턴 매칭을 통해 AWS에 존재하는 민감한 정보를 찾고 이를 알려줘 보호할 수 있게 한다.
