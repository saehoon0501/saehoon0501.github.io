---
title: CloudFront
date: 2023-08-07 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-cloudfront, aws-global_accelerator]
---

## CloudFront & Global Accelerator

### CloudFront

전세계에 있는 edge location에서 origin으로부터 오는 content를 caching하여 User에게 빠르게 전달한다.  
**Content Delivery Network**  
AWS Shield와 연계를 통해 DDos로부터 origin을 보호한다.  
가능한 Origins:

- S3 Bucket: Origin Access Control(OAC)를 통해 S3에 들어오는 traffic을 cloudfront에서만 가능하게 설정 가능. 따라서 edge location에서는 S3와 private IP를 통해 상호작용 가능하다. edge location을 통해 upload 또한 빠르게 가능
- Custom Origin(HTTP): ALB, EC2 Instance, S3 website(Bucket을 static S3 website로 설정), HTTP Backend들을 사용 가능
  edge location과 S3를 제외한 나머지 origin은 Public IP를 사용해야 하기에 Origin에 대한 SG에서 edge location에 대한 public IP를 허락해야 한다.

### CloudFront vs S3 Replication

CloudFront의 경우 Global한 Edge location을 가지며 File들을 Caching하기에 TTL이 존재한다. Static content를 전세계에 제공할 때 좋다.  
S3 Replication의 경우 사용하고자하는 각 Region마다 설정해야 하며, Caching하지 않기에 File의 업데이트를 바로 반영할 수 있다. 따라서 Dynamic content를 few Region에 대해 low latency로 제공하기 좋음

### Geo Restriction

Allowlist 또는 Blocklist에 원하는 country를 넣으면 3rd party Geo-IP DB기반으로 IP에 대한 country가 나오면 list에 따라 allow 또는 block이 가능하다.

### Price classes

Price에 따른 3가지 class가 존재한다.

- Price Class All: 전세계 Edge Location을 사용, 제일 빠르지만 제일 비싸다.
- Price Class 200: 대부분의 Region을 커버하며 제일 비싼 몇몇 Region만을 제왼한다.
- Price Class 100: 가장 싼 Region들에서만 CloudFront를 사용한다.

### Cache Invalidation

CloudFront에서는 File들을 edge location에서 caching하여 User Req가 오면 이를 바로 돌려주기에 실제 Data의 업데이트가 바로 반영되지 않는다.
만약 업데이트된 File을 바로 Cache에 반영하고 싶을 때 사용하는 기능으로 강제적으로 전체 또는 부분적으로 cache를 새로고침한다.
원하는 File에 대한 path 또는 \*를 통해 모든 File을 Invalidate할 수 있음

### Global Accelerator

만약 한 Region에 대해서 Application을 배포하여 전 세계 유저들이 사용하고 있다면 거리에 따라 특정 Region의 User들은 latency와 불안정한 connection을 가질 수 있다. 이를 해소하고자 Global Acceleartor에서는 2개의 Global한 Anycast IP를 제공하여 모든 서버가 같은 IP주소를 가질 때 User가 가장 가까운 서버로 연결되도록 한다.  
(Unicast IP의 경우 한 서버 당 하나의 별도 IP를 가짐)  
Elastic IP, EC2 Instances, ALB, NLB, public or private에서 작동한다.  
이를 통해 일관성있는 성능, app에 대한 Health Check 그리고 오직 2가지 IP만을 가지기에 수월한 Security 설정의 이점들을 얻을 수 있다.

### CloudFront vs Global Accelerator

둘다

- global network와 edge location을 사용한다.
- aws Shield를 통한 DDos 보호를 한다.

  CloudFront는

- CDN으로 Edge에서 caching을 통해 어떤 Content를 User에게 제공할 때 좋다.

  Global Accelerator는

- Caching 없이 application에 대한 connection을 edge location에서 더 빠르게 진행시켜 주는 것이다.
- TCP/UDP를 사용하는 app에 대한 성능 향상에 좋고 static IP 주소 필요 시 사용하기 좋다.
