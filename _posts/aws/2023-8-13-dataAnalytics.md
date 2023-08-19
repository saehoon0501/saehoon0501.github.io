---
title: Data Analytics
date: 2023-08-13 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-analytics]
---

## Data Analytics

### Athena

- Serverless query service only for **S3**
- SQL을 통한 query
- Performance: Columnar data, Compress data, Partition datasets, Larger files in S3
- Ferderated Query using Data source connector(Lambda)
- ad-hoc query

### Redshift

- OLAP(OnLine Analytic Processing)
- Athena보다 더 빠른 join, query 실행을 가지기에 Data warehouse처럼 많은 Data에 대한 query 수행에 이점이 있음<- index가 존재해서 가능함
- Cluser: Leader(query planing Compute Node 관리),Compute Node(query 수행) 두 종류.
- Snapshots & DR: 특정 Redshift cluster에서는 Multi AZ를 지원함. Redshift Snapshot을 자동적으로 다른 Region에 저장하게 하여 Backup 가능
- S3에서는 Enhanced VPC Routing을 통해 더 Redshift에 더 빠른 Batch 작업 가능
- Kinesis Data Firehose와 EC2 Instance에서 Redshift 이용 가능
- Spectrum: S3에 있는 데이터를 직업 가져오지 않고 Redshift에서 query를 수행 가능(단, 이를 위해서 사전 Redshift cluster 구축 필요)

### Opensearch

- DynamoDB에 대한 field query가 가능하게 하는 Search Service

### EMR

- Hadoop Cluster(Big Data)를 생성하는걸 도와줌
- Machine Learning, Big Data 등에서 사용
- Node 종류: Master(cluster 관리), Core(Task와 Data 수행), Task(Task 수행 시 필요한 EC2 Instance)

### QuickSight

- Serverless
- Machine Learning을 통한 Data Visualization을 해주는 Dashboard Service
- SPICE 기능을 통해 import된 data에 대한 빠른 수행 가능
- Users 또는 Group들 간 Dashboard 공유 가능

### Glue

- 데이터에 대한 ETL작업(Extract, Transform, Load)을 수행해준다.
- Serverless
- Bookmarks: 이전 data re-processing되는걸 방지
- Elastic View: 여러 data를 sql query를 통해 combine해 저장 가능
- DataBrew: data normalization 그리고 cleaning을 수행
- Studio: GUI 제공
- Streaming ETL: Streaming Data에 대한 ETL 작업 수행

### Lake Formation

- Data Lake = central place to store all your data for analytics purposes
- Out of the box source blueprints
- Fine-Grained Control for ur apps
- Centralized Permissions: Analyze에 사용되는 Data 전체에 대한 Permission 관리가 쉽다.

### Kinesis Data Analytics

- Kinesis Data Streams & Firehose의 Data에 대한 query 수행 가능

### MSK

- Kinesis Stream의 대안으로 사용 가능
- kafka를 통해 Data를 원하는 만큼 보관 가능
- Serverless

### Kinesis Data Streams vs MSK

- Data Streams with Shard vs Kafka Topic with Partitions
- Shard Splitting and Merging vs Topic에만 partition 추가 가능
- 둘다 KMS at rest encryption 그리고 TLS in transit 가능.
- 최대 1MB message size vs 기본 1MB 최대 10MB size까지 가능
