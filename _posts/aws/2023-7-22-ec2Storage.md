---
title: EC2 Storage
date: 2023-07-22 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-ec2, aws-ec2_stroage] # TAG names should always be lowercase
---

## EC2 Instance Storage

### EBS(Elastic Block Store) Volume

Network drive로 instance에 부착되어 사용된다. 이를 통해 instance에서 data를 지속적으로 가질 수 있게하며, instance 제거 이후에도 보존할 수 있다.  
Network drive이기에 한 instance가 아닌 다른 instance 옮겨 부착되어 사용할 수 있다.(Failover에 사용 가능) 하지만 또한 Network이기에 latency가 발생함  
AZ에 bounded된다.  
Volume이기에 사용 시 capacity(size, IO Per Sec)에 대한 사전 확보가 진행되어야 한다.  
Delete on Termination attribute: Instance 삭제 시 EBS도 함께 같이 삭제되는 option이다. root volume의 경우 기본값으로 on 설정 되있지만 추가적인 volume은 off로 되어있다.

EBS Volume 6Types:
여기서 오직 gp2/gp3 그리고 io1/io2만 OS가 저장되는 boot volume으로 사용 가능하다.

- gp2/gp3(SSD): General purpose SSD  
  gp3의 경우 size와 iops를 따로 선택 가능  
  gp2의 경우 둘 중 하나라도 올리면 둘다 올라감
- io1/io2(SSD): Highest performance SSD  
  32000IOPS 이상을 원한다면 선택한다. 이거보다 크면 Instance Store로 간다.  
  io2가 io1보다 성능이 좋음
  Multi-Attach를 지원하여 한번에 최대 16개 instance에 부착 가능
- st1(HDD): Low cost HDD  
  싸지만 sc1보다 더 좋은 성능을 원할 때 사용
- sc1(HDD):Lowest cost HDD

### EBS Encryption

encryption EBS는 저장된 데이터와 IO하는 데이터 그리고 생성되는 snapshot들 그리고 이 snapshot에서 생성되는 EBS 모두 Encrypt 된다.
Unencrypted EBS를 encrypt하는 방법
Snapshot을 생성한다. -> Snapshot을 Encrypt한다.(copy를 사용해서) -> 해당 Snapshot을 사용해 새 EBS를 만든다.

### EBS Snapshots

EBS Volume에 대한 Backup을 의미한다. 이를 통해 다른 AZ에 있는 EBS에서 같은 데이터를 사용 가능하다.(EBS 자체는 AZ Bounded이기에)  
Backup 과정에서 EBS IO를 사용하기에 app에서 사용 중일 때는 사용 유의  
features:

- EBS Snapshot Archive: Snapshot을 'archive tier'로 옮겨 더 싸게 사용, 데이터를 다시 가져오는데 1~3일의 시간이 걸린다.
- Recycle Bin for EBS Snapshots: Snapshot을 삭제하였더라도 이를 다시 복구할 수 있다. 1d~1y까지 retention을 설정 가능
- Fast Snapshot Restore(FSR): 매우 빠르게 Snapshot을 생성할 수 있는 기능, 매우 비싸다.

### AMI(Amazon Machine Image)

EC2 instance의 customization으로 OS,config, software 등을 마음대로 추가 가능하다. 그리고 이를 활용 더 빨리 instance를 시작할 수 있다. Docker Image의 개념이라고 볼 수 있음  
특정 Region에서 생성되어 직접 다른 Region에서 사용 불가하지만 Copy를 통해 우회 가능하다.  
종류:

- public AMI: AWS에서 제공
- your own AMI: 직접 만든 AMI
- AWS marketplace AMI: 다른 사람들이 만들어 market에 올린 AMI
  AMI Process
  EC2 Instance를 실행 후 customize한다.
  instance를 멈추고 AMI를 build한다. -> EBS Snapshot도 만들어짐
  다른 Instance에서 build한 AMI를 실행한다.

### EC2 Instance Store

EBS는 Network drive이기에 성능에 한계가 있다. 만약 EC2에서 더 좋은 성능의 drive를 원하면 이를 사용한다.
하지만 EC2가 중단될 시 안에 저장된 내용 또한 사라지기에 ephemeral 즉, 장기간 보관해야하는 데이터에는 부적합하다.

### Amazon EFS(Elastic File System)

Network File System으로 많은 EC2에 부착될 수 있다.  
서로 다른 AZ에 있는 Instance들에 대해서도 부착되어 사용 가능 이를 통해 data sharing 또는 web serving이 가능하다.  
오직 Linux 기반 AMI에서 사용 가능하다.(Window 불가)  
기본적으로 POSIX File system을 가진다.  
Performance Mode 2가지:

- General Purpose
- Max I/O

Throughput Mode 3가지:

- Bursting: 원할 때 추가적인 Throughput 제공
- Provisioned: size에 상관없이 throughput을 설정 가능
- Elastic: workload에 따라 Throughput을 알아서 조절

EFS Storage Classes  
Storage Tiers  
lifecycle manangement feature로 standard인 경우 자주 접근되는 파일에 해당하며, 아주 가끔 접근되는 파일들의 경우 lower-tier인 Infrequent Access를 통해 더 싼 값에 보관 가능하다.  
Availability and durability

- Standard: Multi-AZ를 지원 실제 배포에서 활용
- one zone: 한 AZ만 지원 이를 Infrequent Access와 함께 사용해 개발 환경에서 싸게 잘 활용할 수 있음

### EBS vs EFS vs Instance Store(3가지 전부 root volume으로 사용 가능 but 오직 EBS만 EC2의 system boot volume으로 사용된다.)

- EBS의 경우  
  io1/io2를 제외 한 instance에만 부착 가능  
  한 AZ에 Bounded, 다른 AZ로 데이터를 옮길려면 Snapshot을 만든 후 Snapshot을 이용해 다른 AZ에 EBS를 만들어야함  
  Root EBS는 기본적으로 instance 삭제 시 같이 삭제된다. 하지만 설정에서 해제 가능  
  Backup의 경우 IO를 사용하기에 사용 중일 때 유의해서 Backup해야함
- EFS의 경우  
  Multi-AZ를 지원하여 수백개의 instance에 부착되어 공유 가능(오직 Linux instance에만 부착 가능)  
  EFS에서는 website file 공유 가능  
  EBS보다 가격이 비싸다. 하지만 Infrequent Access를 통해 가격 절감 가능
- Instance Store  
  Network가 아닌 실제 physical disk를 사용하기에 latency가 없어 매우 성능이 좋다.  
  하지만 Instance가 제거될 때 저장된 내용들이 모두 사라지기에 caching 등의 용도로만 사용해야한다.
