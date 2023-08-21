---
title: Well Architected Framework
date: 2023-08-20 18:10:00 +/-0
categories: [aws-saa]
tags: [aws-saa, aws-well_architected] # TAG names should always be lowercase
---

## Well Architected Framework

### Well Architected Framework을 위한 6가지 핵심들:(외우기)

- Operational Excellence
- Security
- Reliability
- Performance Efficiency
- Cost Optimization
- Sustainability

위 6가지들은 trade-off가 아닌 synergy를 만든다.

### Well-Architected Tool

architecture에 대한 6가지 핵심에 대한 리뷰를 제공하는 tool로 best practice를 알려준다.

### Trusted Advisor

account에 대한 5가지 추천을 해준다.(외우기)

- Cost optimization
- Performance
- Security
- Fault Tolerance
- Service limits

### Support Plans(외우기)

|7 Core Checks |Full Checks|
|Basic & Developer Support plan |Business & Enterprise Support plan|
|-----------|--------|
|S3 Bucket Permission| Full Checks available on the 5 categories|
|Security Groups - Specific Ports Unrestricted|Ability to set CloudWatch alarms when reaching limits|
|IAM Use|Programmatic Access using AWS Support API|
|MFA on Root Account|----|
|EBS Public Snapshots|---|
|RDS Public Snapshots|---|
|Service limits|---|
