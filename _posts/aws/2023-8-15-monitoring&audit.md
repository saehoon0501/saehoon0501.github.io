---
title: Monitoring & Audit
date: 2023-08-15 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-monitoring, aws-audit]
---

## AWS Monitoring & Audit

### CloudWatch

- Metric을 통해 Service의 성능을 모니터링 할 수 있다.
- Custom Metric을 통해 직접 사용하고자하는 Metric 생성 가능(EC2 Memory Usage)
- Metric Streams: Firehose를 통해 near-real-time latency로 Metric을 저장 가능.(Filter Metric을 통해 일부 Metric만 Streaming 가능)
- Logs: 기본적으로 Encryption되는 Log로 Agent로부터 Log를 생산하여 다른 Service로 보낼 수 있음  
  Log Agent는 2가지 존재
  - CloudWatch Logs Agent: 오래된 버전, 오직 Log만 생성할 수 있음
  - CloudWatch Unified Agent: 신 버전, Log 및 추가적인 system-level metric을 수집할 수 있음. 이를 통해 여러 Metric에 대한 granularity
- Cross-Account Subscription을 통해 Log event들을 다른 AWS 계정의 Resource로 보낼 수 있음
- Alarms: Alarm Event를 보내는 기능으로 Target으로는 SNS, ASG. EC2 Instance 가능  
  Composite Alarm을 통해 AND/OR으로 여러 조건에서 발생되는 Alarm의 조합으로 원하는 Alarm을 만들 수 있음
- Alarm state에 대한 Testing을 하고 싶다면 set the alarm state 명령을 사용
- Insights: Data에 대한 Search 그리고 analyze를 수행  
  Logs Insight: Log에 대한 query와 analyze를 수행 가능  
  Container Insight: Container에 대한 metric과 log를 수집 및 분석, EKS에서는 Container insight agent가 하나의 container로 수행된다.  
  Lambda Insight: Lambda에 대한 metric과 log를 수집 및 분석  
  Contributor Insights: log Data를 통해 특정 Metric에 기여하는 top-N contributers를 찾을 수 있게 분석. Ex) 가장 많은 Req를 보내는 User  
  Application Insights: app에 대해 troubleshoot을 하기위한 automated dashboard를 제공

### EventBridge

- 여러 Service에서 발생한 Event를 JSON형태의 Rule로 Destination들에 보내준다. 이를 통해 CRON Job 또는 Event Pattern을 만들 수 있음
- 이는 Event Bus를 통해 가능하며 Event Bus는 Resource-based Policy를 가진다. 또한 Archive Events 그리고 Replay arhived events 기능 사용 가능
- JSON을 사용하기에 이에 대한 Schema Registry를 활용 가능.

### CloudTrail

- 계정에 안에 있는 모든 Service들에 수행된 API와 같은 활동기록들에 대한 Goverance,Compilance 그리고 audit를 제공한다.
- All Region 또는 1 Region에 적용될 수 있다.
- Event 종류 3가지
  Management Events: Resource에 대한 모든 작업, 기본적으로 logging된다.  
  Data Events: 저장되는 Data에 대한 모든 작업, 따로 설정해야 logging됨  
  CloudTrail Insights Events: 활동기록에서 대한 Baseline을 설정하고 continously analyzes write events to detect unusual pattern
- Events Retention: 90일 동안 Event들이 보관된다. S3에 이를 보관하고 Athena를 통해 analyze도 가능하다.

### CloudConfig

- Configuration에 대한 Compilance를 적용한다.
- Rules를 통해 config rule을 정하고 이를 따르지 않는 경우 Event를 발생하거나 Remediation을 수행할 뿐 강제하지 못한다.
- Remediation retries를 설정해 resource가 compliant할 때까지 최대 시도 횟수를 설정할 수 있다.
- Notifications: non-comliant가 발생할 경우 EventBridge를 trigger하여 Event를 보낼 수 있다. 또는 SNS를 trigger하여 admin에게 알림 보낼 수 있음

### CloudWatch vs CloudTrail vs Config

CloudWatch

- Performance Monitoring
- Events & Alerting
- Log Aggregation & analysis

CloudTrail

- Record API calls made within account
- global service

Config

- Record configuration changes
- Evaluate resources against compilance rules
