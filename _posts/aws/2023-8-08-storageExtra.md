---
title: Storage Extras
date: 2023-08-08 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-storage_extra]
---

## AWS Storage Extras

### AWS Snow Family

Highly-secure, portable device to collect and process data at the edge and migrate data into and out of aws  
Use case는 2가지가 존재한다.

- Data migration: Snowcone, Snowball Edge, Snowmobile
- Edge computing: Snowcone, Snowball Edge

#### Data migration

Network으로 모든 data를 옮기기에 성능과 시간에 한계가 있다. 이럴 때는 Snow Family를 사용하여 offline device에 data를 담아 aws에 직접 보내는게 더 빠를 수 있다.  
사용가능한 Device들:

- Snowcone: 작은 휴대용 컴퓨팅 기기로 어느 장소이든 사용 가능하다. Snowball을 사용하기 힘든 곳에서 유용하며 만약 internet이 있다면 aws DataSync을 통해 data를 보낼 수도 있다. 이를 위해 DataSync agent에 미리 설치되어 있다.
  - Snowcone: 8TB의 HDD 제공
  - Snowcone SSD: 14TB의 SSD 제공
- Snowball Edge
  - Snowball Edge Storage Optimized: 최대 80TB HDD를 제공하며 block volume과 s3와 연동 가능
  - Snowball Edge Compute Optimized: 42TB HDD 또는 28TB NVMe를 제공하며 block volume과 s3와 연동 가능
- Snowmobile: EB단위의 데이터들을 device에 담아 직접 트럭으로 옮기는 서비스로 10PB 이상 migration을 진행해야할 때 유용하다.

#### Edge Computing

Internet 접근이 힘든 환경을 Edge location이라 하며, 이러한 곳에서 Data를 처리해야할 필요가 있을 수 있다.  
이럴 땐 Snowcone 또는 Snowball Edge를 사용하여 edge computing을 하여 data를 처리 가능하다.  
사용 가능한 Device들:

- Snowcone & Snowcone SSD(smaller)
- Snowball Edge Storage Optimized: 최대 40 vCPUs, 80GB RAM, 80TB 저장공간 제공
- Snowball Edge Compute Optimized: 최대 104 vCPUs, 416 GB RAM 제공 GPU를 옵션으로 선택 가능하다. 28TB NVMe 또는 42TB HDD 저장 공간 제공

AWS OpsHub: 기존 Snow Family 사용 시 CLI로만 Interaction이 가능했지만 OpsHub이라는 GUI를 만들어 좀더 사용성을 편하게 했다.

Snowball into Glacier  
Snowball에서 Glacier로 바로 Data를 가져갈 수 없다. 따라서 Snowball -> S3 --lifecycle policy-->Glacier를 거친다.

### AWS FSx

고성능 3rd Party File system을 aws에서 사용 가능하게 하는 서비스이다.  
총 4가지가 존재하면 aws에서 전부 관리할 수 있다.

- Fsx for Lustre
  - High Performance Computing(HPC) 요구 시 사용
    S3와 연계 용이(S3를 file system처럼 읽거나 write 결과를 S3에 저장할 수 있음)
    on-premises server에서 사용 가능 (VPN or Direct Connect을 통해)
    SSD와 HDD 중 하나 선택
- Fsx for NetApp ONTAP
  - NFS, SMB, iSCSI protocol 지원
    Point-in-time instantaneous cloning을 지원(새로운 workload에 대한 testing 용이)
- Fsx for Windows File Server
  - NTFS와 SMB protocol 지원
    Linux EC2 Instance 안에서 설치되 사용 가능
    Microsoft의 Distributed File System(DFS) Namespace를 지원해서 여러 Multi FS에서 파일들을 Grouping할 수 있음
    SSD와 HDD 중 하나 선택
    Multi AZ
- Fsx for OpenZFS
  - NFS와 ZFS protocol 지원
    Point-in-time instantaneous cloning을 지원(새로운 workload에 대한 testing 용이)

Fsx for Lustre - File system deployment options

- Scratch File System: 저장되는 Data는 복제되지 않아 FS가 사라지면 Data들도 다 사라짐. 임시적으로 저장에 용이하며 High burst 지원
- Persistent File System: 저장되는 Data가 같은 AZ 내 복제되어 FS가 사라져도 Data는 유지됨. 장기적인 저장에 용이

### AWS Storage Gateway

Storage를 위한 Hybrid Cloud 구축 시 S3와 같이 aws에 저장된 Data는 aws만의 기술로 NFS/EFS 등과 다르기에 on-premises에서 해당 data를 가지려면 별도의 translate 과정이 필요로 하다. 이때 on-premises와 aws 사이에 Storage Gateway를 둬 이를 가능케 한다.  
4가지 Types

- S3 File Gateway
  NFS와 SMB protocol을 사용하는 FS에서 S3에 접근할 수 있게함
  가장 최근에 사용된 data는 file gateway에서 cached됨
  S3의 glacier tier를 제외한 나머지에서 사용 가능하며 lifecycle policy를 통해 glacier로 데이터를 보관할 수 있음
  Active Directory(AD)를 통해 user authentication을 강제할 수 있음
- FSx File Gateway
  window file server에 대한 FSx File gateway로 window이기에 on-premises에서 바로 접근하여 사용할 수 있음
  그럼에도 FSx File Gateway를 사용하는 이유는 자주 접근되는 data caching을 활용하기 위해서이다.
- Volume File Gateway
  iSCSI protocol을 사용하는 Block Storage로 S3에 접근 시 사용
  cached volume을 통해 자주 접근되는 data에 low latency로 접근 가능
  stored volume을 통해 on-premise에 저장된 모든 data를 S3에 저장 가능
- Tape File Gateway
  physical tape을 사용한 on-premise에서 data를 aws에 저장할 때 사용한다.
  Virtual Tape Library(VTL)을 s3와 glacier에서 지원함.

Storage gateway를 사용 하는 방법은 2가지가 존재한다.

- on-premise에서 Virtualization을 사용
- Storage Gateway Hardware Appliance를 직접 사용한다.

### AWS Transfer Family

S3 또는 EFS에서 in/out file transfer를 FTP protocol을 사용해서 하고 싶을 때 사용한다.
지원하는 protocol들:

- AWS Transfer for FTP
- AWS Transfer for FTPS(Encryption in flight)
- AWS Transfer for SFTP(Encryption in flight)

### AWS DataSync

on-premise과 aws에서 DataSync를 사용해 많은 Data를 받거나 보내려면 virtualization을 수행할 agent가 필요하다.  
aws간에는 필요없음  
Replication 작업은 스케쥴되어 시간/일/주 단위로 진행되며 연속적이지 않다.  
Data가 transfer되어도 **File permissions and metadata are preserved**  
만약 Internet이 없는 환경이라면 Agent대신 Snowcone을 통해 DataSync를 사용 가능하다.

### Storage Comparison

- S3: Object Storage
- S3 Glacier: Object Archive
- EBS Volumes: Network storage for one EC2 instance at a time
- Instance Storage: Physical storage for EC2 Instance(High IOPS)
- EFS: Network file system for Linux instance, POSIX filesystem
- Fsx for Windows: Network file system for Windows servers
- Fsx for Lustre: High Performance Computing Linux file system
- Fsx for NetApp ONTAP: High OS compatibility
- Fsx for OpenZFS: managed ZFS file system
- Storage Gateway: S3/Fsx/Volume gateway(cached&stored) ,Tape gateway
- Transfer family: FTP,FTPS,SFTP interface on top of S3 or EFS
- DataSync: Schedule data sync from on-premises to AWS, AWS to AWS
- Snowcone/Snowball/Snowmobile: 물리적으로 많은 양의 data를 옮김
- Database: for specific workloads, 보통 indexing and querying과 함께 사용
