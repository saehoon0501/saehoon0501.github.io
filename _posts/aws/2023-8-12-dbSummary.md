---
title: DB Summary
date: 2023-08-12 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-db_summary]
---

## Database summary

### RDS

- PostgreSQL/MySQL/Oracle/SQL Server/MariaDB 지원
- RDS Instance size, EBS volume provisioning해야함
- Storage에 대한 Auto-Scaling
- Read replica and Multi AZ
- Security: IAM, KMS, SSL in transit, SG
- Point in time recovery로 자동 백업(최대 35일) or Manual Backup으로 장기간 보관 가능
- IAM Authentication과 Secrets Manager 지원

### Aurora

- PostgreSQL/MySQL 지원
- 데이터를 6 replica in 3AZ에 저장함(self-healing, high availability, auto-scaling)
- Cluster of DB Instance in Multi AZ, auto scaling of Read replicas
- Security는 RDS와 동일
- 관련 backup & restore 암기
- Serverless
- Multi Master: 여러 개의 Instance에서 write 작업 가능
- Global: 최대 16개의 Read Instance 각 Region마다 생성
- Machine Learning
- Database Cloning: test 환경 구축 시 유용

### ElastiCache

- Redis 그리고 Memcached를 관리해주는 서비스
- In-memory data로 sub-mili-s단위 latency
- Redis에 대한 Clustering 그리고 Multi AZ Recovery 제공
- Security: IAM, SG, KMS, Redis Auth
- Backup/Snapshot/Point in time recovery 지원
- 사용 시 application 코드에 변화가 불가피함

### DynamoDB

- Key-Value로 데이터를 저장하는 AWS만의 NoSQL DB
- 미리 provisioned된 Capacity Mode와 On-Demand mode 두 가지 존재
- TTL 기능이 있어 ElastiCache 대용으로 사용 가능
- Serverless
- Multi AZ by default, Read and Write decoupled
- DAX cluster로 read작업 cache, macro-sec latency
- PITR 그리고 on demand Backup 지원
- RCU를 통해 S3로 export(사전에 PITR 설정 on 필요) WCU를 통해 S3에서 import 가능
- Schema가 빠르게 변화하는 Data가 있을 때 사용하기 좋음

### S3

- Key-Value로 Object를 저장
- Object는 크기가 큰 걸 저장하기 좋음, 작고 많은 Object 사용 시 overhead 증가로 성능 저하
- Serverless
- Tiers: Standard, Standard IA, intelligent, Glacier(Lifecycle Policy)
- Features: Versioning, MFA-Delete, Encryption, Replication
- Security: IAM Role, Bucket Policy, SG, ACL, CORS, Object/Vault policy, access points
- Encryption: SSE-C, SSE-S3, SSE-KMS, Client-Side, TTL in transit
- Batch operations
- Performance: Multi-part upload, S3 Transfer Accelerator, S3 Select
- Automation: S3 Event Notifications(SNS,SQS,EventBridge)

### DocumentDB

- MongoDB를 aws에서 지원하는 것

### Neptune

- Graph DB

### Keyspaces

- Apache Cassandra

### OLDB

- Ledger DB, reading financial transactions
- Immutable

### Timestream

- Time series database
