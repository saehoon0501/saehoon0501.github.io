---
title: Advance Identity
date: 2023-08-16 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-advanced_identity]
---

## Advance Identity in AWS

### AWS Organizations

- Global Service로 여러 account를 관리하게 해준다.
- account 전역에 걸쳐 reserved Instance들과 saving plan discount를 공유할 수 있다.
- OU(Organization unit) 단위로 account를 관리하여 security에 대한 설정을 account별로 하기 쉽다.
- SCP(Service Control Policies)를 통해 IAM policies를 OU에 적용할 수 있다. Explicit Allow가 존재해야 Resource를 사용할 수 있다. IAM의 경우와 다름

### IAM Conditions: 모든 Policy에서 사용 가능한 특정 조건에 맞는 값들이다.

- aws:SourceIP: API 호출에 대한 client IP를 restrict할 수 있다.
- aws:RequestedRegion: API 호출에 대한 Region restrict 가능
- ec2:ResourceTag: tags를 기반으로 restrict 가능
- aws:MultiFactorAuthPresent: MFA를 강제 가능
- s3:ListBucket permission은 S3 전체에 대한 permission이며, s3:GetObject, s3:PutObject와 같은 것들은 object level permission이다.
- aws:PrincipalOrgID는 Organization에서 생성한 어떠한 member에 대한 restrict가 가능함

### IAM Roles vs Resource-Based Policy

특정 Resource에 접근하기 위해 account에서 다른 account의 role을 assume하면 기존에 있던 permission이 사라진다.
하지만 Resource-Based Policy의 경우 기존의 permission을 유지한 채 특정 Resource에도 접근 가능하다.

EventBridge의 Service 접근에 대한 Permission 설정 시

- Resource-based Policy를 사용해야 하는 Service들: **Lambda, SNS/SQS, CloudWatch, API Gateway**
- IAM Role를 사용해야 하는 Service들: **Kinesis Stream**

IAM Permission Boundary를 통해 전체 IAM Permission에 대한 경계를 설정하여 이 경계 밖에 있는 Permission은 전부 무효화 가능  
Policy들(SCP, Permission Boundary, IAM) 중 어느 하나라도 Permission이 만족되지 않으면 해당 Resource 사용 불가함

### IAM Identity Center

- SSO(Single Sign-On)을 통해 소유하는 multi account에 대한 로그인을 하나로 가능
- Permission set을 통해 account를 통한 permission control을 할 때 좀더 효율적으로 가능하다.
- FIne-grained permission을 통해 user별로 app에 대한 접근 허용, ABAC(Attribute-Based Access Control)등 세부 허용 설정 가능

### Active Directory

user account, computer, file, SG등 object들을 저장하는 DB이다. 이를 통해 account에 대한 접근이 가능
관련 Service 3가지

- AWS Managed Microsoft AD: AWS에서 AD를 생성하여 on-premise AD와 서로 User를 공유 가능
- AD Connector: auth 요청이 오면 on-premise로 전달하는 proxy
- Simple AD: on-premise와 연결없이 독립적으로 작동하는 AD

### Control Tower

- multi-account에 대한 compliant를 통해 security를 관리할 수 있음
- Guardrails: control tower 환경에 맞게 service에 goverance를 한다.
  - Preventive Guardrail: 사전에 미리 방지하는 역할
  - Detective Guradrail: 환경에 맞지 않은 경우가 발생하면 알림을 보낸다.
