---
title: Containers on AWS
date: 2023-08-10 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-docker, aws-ecs, aws-eks]
---

## Containers on AWS

aws에서는 Docker Container 관리 관련 서비스 4가지를 제공한다.

- Amazon ECS(Elastic Container Service)
- Amazon EKS(Elastic K8s Service)
- Amazon Fargate
- Amazon ECR(Elastic Container Registry)

aws에서 실행되는 Docker container는 Task라고 불림

### ECS

Launch Type에는 2가지가 존재한다.

- EC2 Launch Type
  Infra(EC2 Instance)를 직접 provision & maintain한다.  
  각 EC2 Instance에서는 ECS agent를 실행한다.
- Fargate Launch Type
  Serverless  
  Infra를 직접 관리하거나 구축할 필요없이 그냥 Task들이 사용 CPU/RAM에 따라 실행된다.

IAM Roles for ECS

- EC2 Instance Profile
  오직 EC2 Launch Type에서 사용되는 ECS agent를 위한 IAM Role이다.  
  이를 통해 Container 관련 Service(ECR, ECS)들을 사용하는 것을 허가할 수 있다. 알림을 위해서는 CloudWatch도 접근 허가 가능
- ECS Task Role
  Task에 대한 IAM Role이다.  
  이를 통해 Task에서 수행될 때 필요한 Service들(S3, RDB) 접근 가능하다.  
  Task는 Fargate, EC2 Launch Type 모두 존재하기에 양쪽에서 모두 Task Role을 사용한다.

LB integrations

- ALB: 보통 사용되는 LB
- ELB: High Throughput/Performance가 요구될 시 사용되며, AWS Private Link와 연계 가능하다.

Data Volumes(EFS)  
ECS Task들에서 EFS를 사용할 수 있다. 이는 두 Launch Type 모두 적용 가능하다.  
서로 다른 AZ에서 작동되는 Task들이 한 EFS를 공유하여 Data를 공유할 수 있다.  
Fargate + EFS = Serverless combo  
(S3는 File System으로 사용될 수 없음을 참고!)

ECS Service Auto Scaling  
ECS Task에 대한 auto scaling을 담당한다.  
관련 Metrics:

- ECS Service Average CPU Utilization
- ECS Service Average Memory Utilization(RAM 사용량)
- ALB Request Count Per Target

Scaling Policy

- Target Tracking: CloudWatch mertic의 Target Value를 기반으로 Scaling한다.
- Step Scaling: 특정 CloudWatch Alarm에 맞춰 Scaling한다.
- Scheduled Tracking: 예정된 시간에 맞춰 Scaling한다.

Auto Scaling EC2 Instances

- Auto Scaling Group Scaling: ASG를 CPU 사용량을 기반으로 Scaling한다. 이는 EC2 Instance에 대한 Scaling이다.(Task X)
- ECS Cluster Capacity Provider: ECS Task를 위한 Infra를 자동 Scaling한다. Capacity(CPU, Ram etc)가 부족할 시 EC2 Instance를 추가한다.  
  ECS Cluster Capacity Provider 사용을 추천한다.

EventBridge Integration  
EventBridge에서 Event를 통해 ECS Task를 수행할 수 있다. 이를 통해 특정 Event에 대한 필요한 작업을 필요할 때마다 Task를 생성해 수행 가능하다.  
또한 ECS Task에 대한 Event를 EventBridge를 통해 관리자가 알림을 받을 수도 있다.

### ECR

Docker Image를 AWS에서 보관하는 서비스이다. Public과 Private으로 나뉜다.  
이외에는 Docker Hub를 사용해 Dokcer Image를 보관하고 ECS에서 이를 가져다 사용 가능하다.

### EKS

AWS에서 제공하는 K8s Service로, K8s는 cloud-agnositc하다.(즉 어떤 Cloud에서든 실행 가능)  
Pod이라는 단어가 나오면 EKS

Node Types  
여기서 Node는 Pod들을 실행하는 EC2 Instance를 의마한다.

- Managed Node Groups: Node를 알아서 생성하고 관리를 해준다. Node들은 ASG 안에 속한다.
- Self Managed Nodes: 모든 Node에 대한 생성을 직접하며 EKS Cluster에 등록되고 ASG에 의해 관리된다. AMI를 사용해 Instance를 생성 가능하다.
- AWS Fargate: node에 대한 관리를 aws에서 알아서 다 해줌

EKS Data Volumes  
EKS Cluster를 위한 Storage Class들로 Container Storage Interface를 드라이버를 지원한다.  
가능한 옵션들로는 EBS, EFS, FSx for lustre, FSx for NetAPP ONTAP 등이 있다.

### APP Runner

source code 또는 Docker Image만 있으면 모든걸 알아서 구축하여 배포해주는 서비스이다. 신경쓸게 하나도 없다.
