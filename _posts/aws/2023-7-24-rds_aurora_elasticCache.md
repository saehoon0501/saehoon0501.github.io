---
title: RDS, Aurora & ElasticCache
date: 2023-07-24 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-rds, aws-aurora, aws-elastic_cache]
---

## RDS, Aurora & ElastiCache

### RDS

SQL query language를 사용하는 DB로 Relation DataBase라 불린다.  
AWS에서 지원하는 DB들:

- MySQL
- Postgres
- MariaDB
- Oracle
- Microsoft SQL Server
- Aurora

RDS를 사용하면 AWS에서 자동적으로 provisioning, Multi-AZ setup, read replica 등을 설정해주기에 직접 EC2에 RDS를 설치하여 사용하는 것보다 더 편리하게 사용 가능하다.(단, SSH로 RDS 접근 불가)

Storage Auto Scaling  
RDB DB instance에 더 많은 storage가 요구되면 자동적으로 이를 dynamic하게 늘려준다.  
이를 위해서는 Maximum Storage Threshold를 설정해야한다.  
예측불가능한 workload에 대한 작업 시 유용하다.

Read Replicas와 Multi AZ를 구분  
Read Replicas  
최대 15개의 Read Replica를 생성하여 read 작업(Select구문)을 더 빠르게 수행 가능  
AZ 내, Multi AZ, Cross Region을 지원한다.  
Replication은 async하게 진행된다. 따라서 Read Replication 수행 시 data consistency는 바로 보장되지 않는다.  
Replica들 중 하나가 DB로 승격될 수 있으며, 그러면 해당 Replica에서는 write 작업도 수행 가능해진다.  
이를 통해 app에서 사용하는 DB는 그대로 작업하면서 Replica를 또 다른 app에서 성능에 영향없이 Read 작업을 수행 가능  
같은 Region 내에서 Replication 작업은 무료, 하지만 Cross-Region에서는 유료 기능이다.

Multi AZ(Data recovery)  
Standby instance를 Multi AZ에서 생성하여 Sync replication 작업으로 data를 복사한다. 추후 한 DNS 이름 내에서 Master DB가 다운될 시 Standby instance을 승격 시켜 바로 사용 가능하다.
Availability를 증가시킨다. 하지만 Scaling에서 사용되지는 않음

RDS DB를 Single-AZ에서 Multi-AZ로 바꾸는 방법  
zero downtime 작업으로 DB는 이 과정에서 계속 작동될 수 있다.  
DB에 "modify"와 Multi AZ 설정을 클릭하면 StandBy DB에 Sync replication이 자동 수행된다.

RDS Custom  
오직 Oracle과 Microsoft SQL Server DB에서만 가능한 기능으로 직접 SSH 또는 SSH Session Manager를 통해 RDB를 실행하고 있는 EC2 Instance에 대한 접근이 가능하다. 이를통해 좀더 세세한 설정이 가능해진다.  
Custom을 하기 전 DB Snapshot을 통해 먼저 Backup이 권장되며, Automation Mode off를 통해 Customization 도중 DB가 자동적으로 설정되는 것을 막는다.  
따라서 AWS에서 DB를 관리하는 대신 OS와 DB에 직접 접근하여 setting 가능

RDS Backups

- Automated Backups: 자동으로 매일 전부 Backup되며, 1d~35d까지의 retention을 가질 수 있다. 설정에서 해제 가능
- Manual DB Snapshots: 직접 유저가 작동 시켜야하며 Backup된 내용에 대해 원하는 기간만큼의 Retention을 가질 수 있다.

오랫동안 DB를 사용하지 않을 꺼 같으면 Backup을 한 후 DB를 삭제하고 Backup을 보관하면 Data를 보관하는데 비용을 아낄 수 있다.

RDS Proxy  
VPC 내 Service들에서 동시에 DB에 많은 connection이 열려 DB에 부하가 오는 것을 방지해줘 효율성을 올린다.  
RDS 그리고 Aurora의 failover time을 66% 감소시킨다.  
DB에 IAM Authentication을 강제하고 credential들을 안전하게 보관 가능하다.  
RDS Proxy에 public access는 불가능 항상 VPC에서 들어와야함  
대표적으로 Lambda function이 특정 작업을 수행하고자 DB에 연결할 때 Proxy를 통해 pool에서 관리되어 connect시 timeout 발생을 방지할 수 있다.

### Aurora

Postgres와 MySQL을 지원하는 AWS만의 DB이며, 기존 DB에서 성능을 대폭 향상시켰다.

Aurora High Availability and Scalbility
3 AZ에 걸쳐 6개의 Data 복사본을 생성한다. 또한 Read 작업을 Auto Scaling하여 최대 15개의 Read Replica를 생성 가능하다.  
이러한 Read Replica들은 모두 Reader Endpoint에서 자동적으로 연결하기에 직접 추적할 필요가 없다. Auto Scaling에서 Scale out된 replica 수에 맞춰 Reader Endpoint에서는 이들과 모두 연결된다. Read Endpoint에서는 Replica에 대해 Load Balancing을 수행한다.  
Self Healing을 통해 Data가 corrupt되면 다른 복사본에서 Data를 가져와 이를 복구 시키는 기능  
write 작업은 Master에서만 가능하며 만약 반응 없을 시 Read Replica를 승격시켜 Master로 사용한다.  
Cross Region Replication을 지원한다.

Aurora Replicas Auto Scaling  
Custom Endpoints를 통해 Reader Endpoint와 다른 Endpoint를 만들어 각기 사용 목적이 다른 Aurora Instance들에 맞게 Endpoint를 나눌 수 있다.  
만약 빅 데이터 분석을 위한 Aurora Instance를 가지고 있다면 기존 다른 spec Instance들과 함께 Reader Endpoint에 연결되어 사용되는 대신 또 다른 Endpoint를 만들어 Task에 맞는 Aurora Instance 활용이 가능하다.

Aurora Serverless  
불규칙하고 예측 불가능한 workload에 대해서 미리 provisioning을 하기 어렵기에 Serverless를 통해 AWS 자동적으로 사용량에 따라 DB 인스턴스화 그리고 Auto Scaling을 진행하게 한다. Client에서는 Proxy Fleet에 접근하면 이와 연결된 Aurora Instance들이 사용되는 방식이다.

Aurora Multi-Master: 모든 Aurora Instance들이 Master가 되어 write을 수행할 수 있다. continuous write availability 필요 시 사용

Global Aurora  
Cross Region Read Replicas: Data recovery에 유용  
Aurora Global DB: 1개의 주요 Region에서는 write/read를 하고 나머지 5개 부가 Region에서는 read만을 하는 방식이다. 부가 Region 당 최대 16개의 Read Replica를 생성할 수 있다. Typical cross-region replication takes less then 1 sec <-이거 나오면 Global Aurora

Aurora Machine Learning: SageMaker와 Comprehend와 같이 사용되어 data로부터 예측을 가능

Aurora Backups

- Automated Backups: 1d~35d의 retention을 가질 수 있으며, 해제될 수 없다.
- Manual DB Snapshots: 원하는 기간만큼의 Retention을 가질 수 있다.

RDS & Aurora Restore options

- RDS/Aurora 모두 backup 또는 Snapshot을 통해 새로운 DB를 생성할 수 있다.
- RDS를 S3에서 Backup file을 그대로 가져와 이를 수행할 수 있으며, Aurora의 경우 Backup 시 XtraBackup을 사용해야 한다.

Aurora DB Cloning  
기존 Aurora에서 새로운 Aurora를 만든다. 이는 copy-on write protocol을 사용한다.  
즉, 기존에 존재하는 Data는 기존 DB에서 가져오고 새로 write되는 Data는 cloning하여 추가적인 DB에 같이 복사하여 저장하는 방식  
이를 통해 DB를 실제 환경에서 어떻게 동작할지 'staging'할 수 있다.

RDS & Aurora Security

- At-rest encryption: 만약 master가 encrypted되지 않으면 read replica들도 encrypt불가. 따라서 처음 DB 생성 시 encryption 설정을 하거나 DB Snapshot에서 encryption 후 다시 재생성하는 방식 2가지가 있다.
- In-flight encryption: TLS certifiacte을 통해 들어오는 data가 encrypted될 수 있다.
- IAM Authentication: Service에서 DB 연결 시 id/password말고 IAM Role을 통해 이를 대신할 수 있다.
- Security Groups: DB에서 inbound/outbound Network access를 조절할 수 있음
- No SSH available except RDS Custom
- Audit logs can be enabled and sent to CloudWatch

### ElastiCache

Redis와 같이 in-memory DB로 매우 높은 성능과 낮은 latency를 가지지만 크기가 작다.  
AWS에서는 setup관련 작업을 모두 대신 해주며, ElastiCache 사용 시 app에서의 직접 Cache에 query를 하는 코드가 필요하다.  
DB caching, User Session store 등에 활용될 수 있다.  
ElastiCache내 Memcached vs Redis

| Redis                                                | Memcached                              |
| ---------------------------------------------------- | -------------------------------------- |
| Multi AZ with Auto Failover                          | Data sharding                          |
| Read Replicas를 통한 Scalability와 high availability | No Replication == No High availability |
| Backup 그리고 Restore 기능                           | No Backup and Restore                  |

Cache Security  
IAM Authentication For Redis를 지원한다.(나머진 id/password를 사용해야함)  
Redis AUTH: SSL in flight encryption을 지원한다. Redis cluster 생성 시 'password/token'을 설정할 수 있다.  
Memcached의 경우 SASL-based Authentication을 지원한다.

Patterns for ElastiCache

- Lazy Loading: 모든 데이터를 Cache에서만 가져옴 없으면 DB에서 가져옴(Data가 오래될 수 있음)
- Write Through: DB에 write 시 Cache에도 write(Data가 항상 업데이트된 상태)
- Session Store: TTL을 설정하여 Cache에 Session Data가 특정 시간 만큼만 Data저장

Redis Use cases

- Gaming Leaderboard: Redis Sorted sets을 통해 score에 따라 rank를 가져올 수 있다.
