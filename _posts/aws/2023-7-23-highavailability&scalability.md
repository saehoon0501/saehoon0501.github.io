---
title: High Availability & Scalability
date: 2023-07-23 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-availability, aws-scalability] # TAG names should always be lowercase
---

## High Availability & Scalability: ELB, ALB, NLB, ASG

Scalability: app/system에서 많은 작업량을 얼마나 잘 적응해 처리하는지를 의미한다.  
이는 Horizontal과 Vertical 두 가지 방식이 존재한다.

- Vertical Scalability  
  Instance의 capacity를 늘려 많은 작업량을 처리하는 방식  
  DB와 같이 non distributed system에서 흔히 볼 수 있다.
- Horizontal Scalability  
  Instance의 수를 늘려 많은 작업량을 처리하는 방식  
  이는 곧 Distributed System을 의미해 각 Machine에서 한 기능들을 수행하고 다른 Machine간 RPC를 통해 application을 구성한다.  
  Cloud에서 매우 쉽게 가능  
  관련 Service: ASG(Auto Scaling Group), LB(Load Balancer)

High Availability: 고가용성은 서버, 네트워크, 프로그램 등의 정보 시스템이 상당히 오랜 기간 동안 지속적으로 정상 운영이 가능한 성질을 말한다.  
이는 여러 AZ에서 운영될 수 있는 Horizontal Scaling하고 잘 맞아 떨어진다.  
만약 서로 다른 Region이나 AZ에 Data Center가 존재한다면 Availbility가 높은 것이다.  
관련 Service: ASG(Auto Scaling Group) in Multi AZ, LB(Load Balancer) in Multi AZ

### ELB(Elastic Load Balancer)

Multi AZ 가능  
Load Balance란 들어오는 Traffic을 다른 여러 Server로 forwarding해주는 서버를 의미한다.  
사용 이점들:

- Multiple downstream instance들에게 Traffic을 Balance있게 분산시킬 수 있다.
- app을 Single point of access(DNS)만을 통해 외부에 노출 가능하다.
- High Availibility
- Instance들에 Health check 수행
  ELB는 이러한 LB를 AWS에서 전부 관리해줘 Client는 사용만 하면된다. 그리고 수많은 AWS Service과 함께 사용 가능하다.

Health Check  
LB에서 Instance에 req를 보내서 대한 작동 여부를 확인하여 작동하지 않는 Instance에는 Traffic을 보내지 않는다.  
만약 res가 200이 아니라면 unhealthy로 판별된다.

Types:

- Application LB: HTTP, HTTPS, WebSocket
- Network LB: TCP,TLS,UDP(가장 성능 좋음)
- Gateway LB: OSI Layer 3에서 작동(IP Protocol)
  Internal(private) 또는 external(public)으로 setup된다.

SG in LB  
LB에서는 외부 모든 ip를 받아들인다. 하지만 Instance에서는 LB의 SG만을 reference하여서 Instance에 들어오는 traffic은 모두 LB를 지나야만 가능하도록 설정할 수 있다.

### Application LB

Layer 7에서 작동  
Routing Table을 가지기에 path, hostname, query string 그리고 header에 따라 다른 Target group으로 routing이 가능하다.  
Target Group에는 EC2 Instance, ECS task, Lambda function 그리고 private IP들이 들어갈 수 있다.  
따라서 여러 Target Group들에 대한 Load Balancing 수행 그리고 여러 Container들에 대한 LB도 수행 가능  
기본적으로 고정된 hostname 주소를 제공해줘 브라우저를 통해 쉽게 접근 가능하며 Client에서는 항상 LB를 거쳐서 가기에 Instance에서 Client IP를 보기 위해서는 X-Forwarded-For라는 field를 봐야한다.  
Elastic IP 사용 불가

### Network LB

Layer 4에서 작동  
TCP그리고 UDP Traffic을 instance로 forwarding한다.  
Target Group으로는 EC2 instance, private IP 주소들 그리고 Application LB가 존재한다.  
Health check으로 HTTP/HTTPS, TCP를 지원한다.  
AZ 당 하나의 IP를 가지며 각 AZ당 Elastic IP를 지원한다.

### Gateway LB

Layer 3(Network Layer)에서 작동한다. IP Packet 관련 따라서 Gateway LB를 통해 Network Traffic을 분석할 수 있다.  
User의 Traffic을 Routing Table을 통해 Gateway LB로 보내면 Firewall 등 3rd party security virtual appliance들을 먼저 거쳐 Traffic이 valid하면 다시 Gateway LB를 거쳐 최종 App에 forwarding되는 방식이다.  
port 6081에서 GENEVE protocol을 사용한다.  
Target Group에는 EC2 Instance, private IP들이 있다.

### Sticky Sessions(Session Affinity)

같은 Client가 Request를 보내면 항상 같은 Instance로 가는 것을 말한다. 이는 ALB 그리고 NLB에서 지원한다.
만약 User가 Req를 보낼 때마다 다른 Instance로 가면 Instance간 Session 공유를 하지 않는 이상 cookie가 가지고 있는 Session을 잃어버려 매번 Login을 새로 해야하는 경우를 방지해준다.(NLB는 cookie를 사용하지는 않는다.)

Cookie Names

- Application-based Cookie: target에서 생성하는 custom 쿠키로 app에서 원하는 attr을 설정할 수 있다. 단 AWSALB, AWSALBAPP, AWSALBTG attr로 사용할 수 없다. application cookie일 경우 LB에서 생성하며 이때 attr은 AWSALBAPP이다.
- Duration-based Cookie: LB에서 생성하는 cookie로 attr은 AWSALB for ALB, AWSELB for CLB으로 만료 시점을 가진다.
  종류가 2가지 있는 정도만 기억하자

### Cross-Zone Load Balancing

각 LB에서 등록된 모든 Instance에 균일하게 Load를 분산한다.  
사용될 경우 각 LB에서 연결된 모든 AZ 내 Instance에 Load를 분산시킨다.  
이를 사용하지 않는 경우 LB에서 Load를 균등하게 나눠 갖고 각 LB에 연결된 Instance들 내에서만 LB에서 오는 Load만을 균일하게 가진다.  
ALB에서는 기본적으로 on 설정되어 있다.  
NLB,GLB에서는 기본적으로 off되어 있고 유료 기능이다.

### SSL(Secure Sockets Layer)/TLS(Transport Layer Security) Certificates

SSL Certificate은 Client와 LB 사이에 Traffic을 encrypt하고 Client와 LB만 이를 decrypt할 수 있다.  
현재 대부분 TLS certificates를 사용하지만 ssl이라도 더 많이 부른다.

Client가 HTTPS를 보내면 LB에서는 SSL/TLS Certificate termination을 수행할 수 있다. 그러면 LB와 Instance간에는 private IP를 통해 HTTP로 traffic을 주고받을 수 있다. Client에서는 SNI(Server Name Indication)을 통해 Traffic이 도달하고자하는 Hostname을 남기면 LB에서는 해당하는 Hostname에 맞는 Certificate를 알고 사용할 수 있다.

SNI(Server Name Indication)  
한 LB에서 Mulitple SSL Certificate를 사용할 수 있게 해준다. 이는 새로운 protocol로 Client에서 Hostname을 나타내 Req를 보내면 LB에서는 해당 Hostname에 맞는 Certificate를 사용하는 방식이다. 따라서 여러 HTTPS App을 한 LB에 Expose 할 수 있다. 오직 ALB 그리고 NLB에서만 사용 가능

Connection Draining(or Deregistration Delay)  
특정 시간동안 이미 진행 중인 Req를 de-registering Instance에 보낸다. 그리고 새로운 Req는 다른 Instance들로만 보낸다.  
만약 설정 시간이 다 지날 때까지 Instance가 되돌아오지 않으면 완전 shut down한다.

### ASG(Auto Scaling Group)

현실에서는 Traffic 양은 항상 바뀐다. 그래서 많을 때는 Instance를 늘리는 Scale out 적을 때는 Instance를 줄이는 Scale in을 알아서 수행한다.  
또한 작동하고 있던 Instance가 제거되면 알아서 설정에 맞춰서 새 Instance를 재생성한다.  
ASG를 위한 설정들:

- Launch Template: 생성하고자 하는 Instance에 대한 정보와 관련 SG, IAM, LB, EBS에 대한 정보들을 가진다.
- Minimum Capacity:최대 Scale in 수
- Desired Capacity: 가장 optimal한 수
- Maximum Capacity: 최대 Scale out instance 수
- Scaling Policy: CloudWatch에서 특정 metric을 ASG 내 모든 Instance에 대해 모니터링하다가 기준을 넘어서면 Alarm을 보낸다. 그러면 해당 Alaram을 받은 ASG에서는 Scaling Policy에 따라 Scale in/out을 수행하게 된다.

ELB+ASG  
ELB에서 Health Check을 하고 Instance가 반응이 없으면 ASG에서 이를 제거하고 새로 실행하는 것이 가능하다. 또한 Traffic이 늘어나면 LB에 연결되는 Instance들을 ASG에서 자동적으로 Scale out해준다.

Dynamic Scaling Policies

- Target Tracking Scaling: 특정 Target 값을 유지하는 것을 목표로 Scaling
- Simple/Step Scaling: CloudWatch를 통해 customize하여 특정 metric이 기준 값 이상 또는 이하일 때 어떤 Scaling Policy를 수행할지 정한다.
- Scheduled Scaling: 특정 시간대에 수행하는 Scaling Policy를 정한다.
- Predictive Scaling: 지속적으로 Traffic Load를 ML로 예측하여 알아서 Scaling을 미리 Schedule한다.

Good mertics to scale on:

- CPU Utilization
- Request Count Per Target
- Average Network In/Out
- Any custom metric(using CloudWatch) Ex) Memory usage of EC2

Scaling Cooldown: Scaling 동작 후 cooldown 기간(기본 300초)이 존재하며 이는 ASG가 그 동안 추가적인 Instance를 생성/제거하지 않을 것을 의미한다.
