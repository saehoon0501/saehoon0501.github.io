---
title: AWS Disaster Recovery & Migrations
date: 2023-08-19 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-disaster_recovery, aws-migrations] # TAG names should always be lowercase
---

## Disaster Recovery & Migrations

### RPO & RTO

RPO(Recovery Point Objective): How often you run backups  
RTO(Recoevery Time Objective): Recover point from disaster  
RPO와 RTO를 어떻게 Optimize할 것인가에 따라 Solution Architecture decision이 따른다.  
RPO가 길어질 수록 소실되는 Data의 양이 많아지며 RTO가 길어질 수록 Server가 down되어 있는 상태가 길어진다.

### Disaster Recovery Strategies

- Backup and Restore: High RPO/RTO로 Snowball이나 Storage Gateway를 사용할 시 RPO는 1주일이며 주기적인 snapshot backup 시 1hr~1d 정도의 RPO를 가진다. 이렇게 Backup된 Data를 가지고 Disaster 발생 시 직접 Backup System(EC2 Instance)을 실행한다.
- Pilot Light: cirtical core에 대한 replication을 진행하여 aws RDS에 복제한다. 그리고 재난 발생 시 RDS에 복제된 Data를 가지고 대기 중인 EC2를 실행한다. Backup and Restore보다 낮은 RPO/RTO를 가진다.
- Warm Standby: RDS에 replication이 Pilot Light과 같이 진행되며 ASG의 EC2가 최소 상태로 실행 중이다 재난 발생 시 RDS Slave에서 데이터를 사용하며 Scaling한다.
- Hot Site/ Multi Site Approach: 매우 낮은 RTO/RPO를 가지며 Full Production Scale의 EC2가 실행되고 있다 재난 발생 시 failover를 통해 사용된다.

### DMS(Database Migration Service)

source에서 target으로 아니면 반대로 DB를 이전해주는 서비스이다.  
source에 있는 DB이 migration이 진행 중이여도 계속되어 작동할 수 있다.  
Oracle to Oracle와 같이 Homogeneous migration과 Microsoft SQL server to Aurora와 같이 Hetetrogeneouse migration도 지원한다.  
무조건 EC2 Instance를 활용해 replication을 진행해야 하며, continuous data replication이 가능하다.  
source에는 on-premise DB들과 aws RDS 그리고 S3 등이 가능하며 target으로는 source와 같은 것들 + dynamoDB, Data analyze service들이 있다.

SCT(Schema Conversion Tool)  
기존의 DB schema에서 다른 DB로 convert해주는 서비스이다.Ex) PostgreSQL -> MySQL
만약 같은 DB engine이라면 SCT를 사용할 필요가 없다.

Multi AZ Deployment  
만약 DMS 사용 시 Multi-AZ 설정을 사용하면 다른 AZ에 대해서도 sync하게 replica를 유지한다.
이를 통해 Data Redundancy 증가와 latency를 줄일 수 있다.

RDS & Aurora MySQL Migration

- RDS MySQL to Aurora MySQL
  - option 1: RDS MySQL에서 Snapshot을 생성하고 이로부터 Aurora MySQL을 생성한다.
  - option 2: Aurora Read Replica를 RDS MySQL로부터 생성하고 replication이 끝나면 이를 own DB cluster로 승격한다.
- External MySQL to Aurura MySQL
  - option 1: Percona XtraBackup을 통해 RDS MySQL로부터 backup을 s3에 만들고 이로부터 Aurora MySQL을 생성한다.
  - option 2: mysqldump를 사용하여 생성한 Aurura MySQL로 바로 migrate한다.

만약 두 DB 모두 작동 중이라면 DMS를 사용한다.

RDS & Aurora PostgreSQL Migration

- RDS PostgreSQL to Aurora PostgreSQL
  MySQL의 경우 똑같이 option 2개를 선택 가능하다.
- External PostgreSQL to Aurora PostgreSQL
  - S3에 backup을 저장하고 Aurora에서 이를 importgksek.

만약 두 DB 모두 작동 중이라면 DMS를 사용한다.

### On-premise strategy with aws

- amazon linux 2 AMI를 VM으로 다운 가능
- VM Import/export: 기존에 있는 app을 EC2로 이전
- AWS Application Discovery Service: on-premise에 대한 정보를 모아 aws로 mapping을 통해 migrate
- AWS Database Migration Service: on-premise -> aws, aws -> aws, aws -> on-premise로 migrate
- AWS Server Migration Service: on-premise에 있는 server에 대한 replication을 aws로 증가시킨다.

### AWS Backup

말 그대로 backup을 aws에서 모두 알아서 해주는 service이다.  
cross-region과 cross-account에 대한 backup도 가능하다.  
PITR(Point in time recovery)를 지원하며 Backup Plan을 만들 수 있다. 이를 통해 plan에 맞춰 S3에 backup들을 저장할 수 있다.

Valut Lock  
WORM(Wirte Once Read Many) 상태를 강제하여 Backup된 Data를 삭제하지 못하게 한다.
이를 통해 Backup에 대한 추가적인 보안을 제공한다.

### AWS Application Discovery Service

on-premise에 대한 정보를 모아 aws로 mapping을 통해 migration을 진행한다.
두 가지 방식이 존재:

- Agentless Discovery(AWS Agentless Discovery Connector): on-premise에 설치되는 agent없이 on-premise 관련 정보를 모은다.
- Agent-based Discovery(AWS Application Discovery Agent): on-premise에 agent를 설치하여 on-premise에 대한 더 자세한 정보를 모은다.

나온 결과는 AWS Migration Hub를 통해 확인 할 수 있다.

### AWS Application Migration Service

on-premise에 있는 서버들을 aws에서 native하게 작동되도록 해준다.
이는 continuouse replication을 aws로 진행하다가 작업이 완료되면 옮겨진 데이터들을 원하는 환경에 맞게 cutover하여 실행하면서 진행된다.

### Transferring larget amount of data into AWS

- Site to Site VPN: 인터넷을 사용하여 진행하기에 구축하는데 매우 짧은 시간이 걸리지만 매우 큰 데이터들을 옮기는데 많은 시간이 소요된다.
- Direct connect: 구축하는데 시간이 좀 걸리지만 1Gbps 등을 제공하여 큰 데이터를 옮길 수 있다.
- Snowball: 직접 on-premise에서 data를 physical device에 옮긴 후 이를 aws로 배송보내기에 큰 데이터를 옮길 수 있다.
- For on-going replication/transfers: Site-to-Stie VPN, Direct Connect with DMS 또는 DataSync

### VMware Cloud

on-premise에서 VMware을 사용하고 있는 경우 이를 aws로 migrate할 수 있게 한다.
아니면 추가적인 Capacity를 aws에서 증가시키고 기존 VMware을 계속 On-premise에서 사용하는 방법도 가능하다.
