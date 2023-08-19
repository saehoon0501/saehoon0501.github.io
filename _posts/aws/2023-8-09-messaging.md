---
title: Messaging
date: 2023-08-09 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-messaging]
---

## AWS Integration & Messaging: SQS,SNS, Kinesis

여러 Service들을 배포하면 각 Service들이 필수적으로 서로 communicate해야할 상황이 생긴다.  
이럴 경우 2가지 방식으로 communication을 가질 수 있다.

- Synchronous communications: Service간 communication이 직접적으로 바로 연결되어 처리된다.
- Asynchronous/ Event based: Service 사이에 중간 매개체를 통해 communication이 비동기적으로 발생된다.  
  만약 평소와 다르게 갑자기 많은 Traffic이 몰리게 되면 Sync한 경우 한 Service에 과도한 요청을 동시적으로 처리하다 부하가 올 수 있다.  
  따라서 Async와 같이 각 Service들을 decoupling하여 System의 안전성을 증대시킬 수 있고 Service들은 독립적으로 Scaling이 가능하다.

aws에서는 이러한 decoupling을 위한 Service 3가지를 제공한다.

- SQS: queue model
- SNS: pub/sub model
- Kinesis: real-time streaming model

### SQS

decoupling applications  
message를 생성하여 SQS에 넣는 쪽을 producer, polling하여 message를 처리하는 쪽을 consumer라 한다.  
메세지가 중복되어서 consumer한테 보내질 수 있다.(at least once delivery)  
message가 생성된 순서에 맞춰 consumer에게 최대한 보내진다.(best effort ordering)  
생성된 message는 retention 기간(기본 4일, 최대 14일) 내에서는 유지되며 retention 기한 내에는 consumer에서 처리 후 삭제 요청을 해야 삭제된다.  
Buffer 역할을 하여 Request를 처리하는 Consumer에서 timeout없이 성능을 유지할 수 있고 Producer는 보내는 Request가 처리되지 않을 걱정을 하지 않아도 된다.

Multiple EC2 Instances Consumers  
Consumer들은 병렬적으로 SQS으로부터 message를 받아 처리할 수 있다. 처리한 후에 deletion 요청을 SQS에 한다.  
at least once delivery, best effort ordering  
SQS 덕분에 EC2 Instance를 Horizontal Scaling하기 쉽다. -> 여기서 파생되는게 ASG와 SQS의 사용

SQS with ASG  
SQS에 message가 들어오는 양을 CloudWatch에서 보고 있다가 target값을 넘으면 ASG에 알림을 보낸다.  
그러면 ASG에서는 알림을 받은 내용을 바탕으로 Scale in/out을 수행한다.

Security

- Encryption  
  HTTPS를 통한 In-flight encryption 지원  
  At-rest encryption using KMS keys 지원  
  Client-side encryption 지원
- Access Control: IAM policy를 통해 SQS API에 대한 접근 관리 가능
- SQS Access Policies(Resource Policy)를 통해 SQS에 대한 cross-accont access와 다른 Service 상에서 SQS에 메세지 작성을 허가할 수 있음

Message Visibility Timeout  
consumer에서 message를 polling하면 해당 message는 다른 consumer에게 invisible해야한다. 기본적으로 30초 동안 유지됨.  
따라서 만약 30초 내 message가 처리되지 않는다면 다른 consumer에게도 보내져 두번 처리되게 된다.  
만약 Consumer에서 visibility timeout 내 처리하지 못할 것 같으면 ChangeMessageVisibility API를 통해 시간을 좀더 연장할 수 있다.

Long Polling  
Consumer에서 SQS에 계속해서 polling request를 보내는 대신 'wait'을 통해 message가 도착할 때까지 기다리는 기능  
이를 통해 API 호출 횟수를 줄일 수 있어 SQS의 효율성과 latency를 줄일 수 있다.  
Long Polling은 SQS에서 설정하거나 WaitTimeSeconds를 통해 Consumer에서 수행할 수 있다.

FIFO  
Message가 순서에 맞게 Consumer에게 보내지며 Group ID가 없을 시 한 Consumer만 가질 수 있다. Group ID 존재 시 Group ID 마다 Consumer를 가질 수 있음  
300 msg/s(without batching), 3000 msg/s(with batching)  
Message가 정확히 한번만 보내진다.(중복 없음)

### SNS

'Event producer'에서는 오직 하나의 SNS Topic에 message를 보낼 수 있다.  
이 message는 해당 Topic을 subscribe하는 Service나 User들 모두에게 보내지게 된다. 따라서 하나씩 일일히 notification을 보낼 필요 없이 SNS에 쉽게 처리한다.

Security

- Encryption
  HTTPS를 통한 In-flight encryption 지원  
  At-rest encryption using KMS keys 지원  
  Client-side encryption 지원
- Access Control: IAM policy를 통해 SNS API에 대한 접근 관리 가능
- SNS Access Policies(Resource Policy)를 통해 SQS에 대한 cross-accont access와 다른 Service 상에서 SNS에 메세지 작성을 허가할 수 있음

SNS+SQS = Fan out  
SNS에 message를 한번 보내면 이를 구독하는 모든 SQS에서 message를 받아 각 SQS consumer에서 처리될 수 있다. 이를 통해 Fully decoupled, no data loss  
SQS의 Access Policy에서 SNS에 write 권한을 부여해야 한다.  
Cross-Region Delivery가 가능해 다른 region에 있는 SQS에도 message가 간다.  
이를 응용하여 S3에서는 하나의 Event만을 보낼 수 있는데 이를 SNS에 보내면 이를 구독하는 많은 Service들에게 이제 Event를 보낼 수 있게 된다.

FIFO  
SQS FIFO와 동일하게 message의 순서를 지켜 보내며 보내는 메서지는 중복되지 않는다.(Ordering, Deduplication)

Message Filter  
모든 Subscriber가 모든 message를 받는 것이 아닌 filter policy를 통해 특정 message만을 받을 수 있게 한다.

### Kinesis

streaming data 실시간으로 collect, process and analyze 할 수 있게 한다.  
종류

- Kinesis Data Streams  
  Data를 shard들로 나눠서 처리한다. 이 shard 수에 따라 ingestion/congestion 성능이 결정된다.  
  모든 consumer가 2mb/sec per shard를 공유하거나 각 consumer당 2mb/sec per shard를 제공할 수 있다.  
  한번 Data가 들어가면 삭제되지 않는다.(Immutable)  
  같은 Partition ID를 가지는 Data는 같은 Shard에만 들어가게 된다.(ordering)  
  Mode 2가지
  - Provisioned Mode: 예상되는 data에 맞게 사용 전 미리 Shard를 provision하여 정한다. provisioned Shard를 시간당으로 계산
  - On-Demand Mode: 예상되는 data를 모를 때 그때 요구사항에 맞게 Shard를 사용한다. stream을 사용한 시간당 그리고 Data 사용량 만큼 계산
- Kinesis Data Firehose
  serverless하며 자동 scaling을 한다.(provisioningX)
  Near Real Time으로 Data를 목적지에 write한다. 가능한 AWS 목적지는 S3, Redshift, OpenSearch가 있다. 이외에는 HTTP endpoint 또는 제3 파트너가 존재한다.
- Kinesis Data Analytics

Kinesis Data Streams vs Firehose

| Kinesis Data Streams                           | Firehose                                                       |
| ---------------------------------------------- | -------------------------------------------------------------- |
| Stream Data를 받는다.                          | Stream Data를 S3/Redshit/OpenSearch 그리고 이외 곳에 저장한다. |
| producer/cosumer에서 직접 코드를 작성해서 사용 | 모든게 자동으로 aws에서 관리한다.                              |
| Real time(~200ms)                              | Near Real time(60sec)                                          |
| sharding을 통해 scaling을 직접 설정 가능       | 자동 Scaling                                                   |
| 1~1y동안 데이터 보관 가능                      | 데이터 보관 X                                                  |
| Data를 Replay할 수 있다.                       | replay 기능 없음                                               |

Ordering data into Kinesis  
같은 Partition key를 가지는 Data들은 항상 같은 Shard에 들어가서 처리되기에 이를 통해 Data를 순서에 맞게 처리 가능하다. Shard 당 하나의 Consumer를 가진다.

Ordering data into SQS  
FIFO를 사용하여 Data를 순서에 맞게 처리 가능하다.  
만약 Group ID가 없다면 오직 one consumer에게만 Message를 보낼 수 있다. 하지만 Group ID가 존재한다면 각 ID 당 Consumer가 존재한다.

### SQS vs SNS vs Kinesis

| SQS                                      | SNS                                                                        | Kinesis                                |
| ---------------------------------------- | -------------------------------------------------------------------------- | -------------------------------------- |
| Consumer는 data를 pull한다.              | 많은 Subscriber들에게 data를 push한다.                                     | 2Mb per shard로 pull data              |
| data는 consumer에서 처리 후 삭제된다.    | Data가 지속되지 않고 전달 후 바로 사라진다. 따라서 전달 되지 않으면 소실됨 | Data를 Replay할 수 있다.               |
| 원하는 만큼의 Consumer 둘 수 있음        | 최대 12500000 구독자와 100000 topic를 가진다                               | Shard 당 하나의 consumer               |
| provisioning 불필요                      | provisioning 불필요                                                        | provisioned mode와 on-demand mode 존재 |
| FIFO에서 ordering과 Deduplication을 보장 | FIFO에서 ordering과 Deduplication을 보장                                   | Shard를 통해 ordering을 보장한다.      |

### Amazon MQ

3rd Party SQS나 SNS를 이용하는 경우 이를 지원하기 위한 broker service이다.  
SQS/SNS처럼 scale하지 않지만 Multi-AZ를 통해 failover 가능하다. 이러한 failover는 EFS를 통해 Backup을 저장하여 다른 AZ로 failover 시 해당 EFS를 사용함으로 이뤄진다.
