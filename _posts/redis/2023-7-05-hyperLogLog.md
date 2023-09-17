---
title: HyperLogLog
date: 2023-07-05 15:10:00 +/-0
categories: [redis]
tags: [redis-hyperLogLog] # TAG names should always be lowercase
---

set과 비슷하게 저장하는 Data의 uniqueness를 필요하지만, 이 경우 대략적으로만 필요로하여 사용하는 Data Structure로 Logging과는 아무 상관이 없다.  
차이점은 set의 경우 매 Data 하나하나 정확히 value 그대로 저장하여 결과를 매우 정확하게 가져오지만, HyperLogLog의 경우 Data가 기존에 추가되었는지 정확히 알려주지만 유일하게 가져올 수 있는 정보인 value들의 Count는 오차 범위 내 Approximate하여 알려준다. 이는 HyperLogLog의 특성 상 value를 직접 저장하는 것이 아니라 Algorithm을 거쳐 저장하여 value들의 수에 상관없이 항상 일정한 크기를 유지하기 때문이다.

## commands:

PFADD c_key values : c_key에 기존 value가 추가되었다면 0 처음 추가된다면 1을 return  
PFCOUNT c_key : 저장되었던 value들의 수를 0.81% Error를 가지고 return한다.

## Use Case

- uniqueness가 필요로 하지만 정확히 모든 Data들 추적할 필요가 없다면 Memory saving이 크기 때문에 사용한다.
  Ex) Item이 각 유저마다 한번씩만 view가 되도록 count해야할 때, Set의 경우 Item을 본 모든 User를 정확히 추적하지만 나중에 Size가 너무 커질 수 있다. 따라서 HyperLogLog를 사용하여 view한 모든 유저를 단 한번씩만 count하면서 약간의 오차를 감수하여 Size를 유지하면서 비교적 정확한 view를 count 할 수 있다.
