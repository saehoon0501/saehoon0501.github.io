---
title: Sotred Set in Redis
date: 2023-07-04 16:10:00 +/-0
categories: [redis]
tags: [redis-sorted_set] # TAG names should always be lowercase
---

Hash와 Set의 Mix된 형태이다.  
keys와 values 대신 members와 scores가 존재한다.  
members는 key와 같으며 unique해야 한다. scores는 value와 같으며 unique하지 않다.  
저장되는 Data는 score를 기준으로 오름차순으로 정렬된다.  
구조:  
collection_key

| member   | score |
| -------- | ----- |
| MEMBER#1 | -4    |
| MEMBER#2 | 0     |

## commands:

- ZADD c_key score member : sorted set에 member-score 짝을 추가한다.
- ZSCORE c_key member : c_key의 member에 해당하는 score를 가져온다.
- ZREM c_key member : collection에 존재하는 member를 삭제한다.
- ZCARD c_key : collection에 존재하는 member들의 수(cardinality)를 가져온다.
- ZCOUNT c_key min_score max_score : min~max 사이에 score가 존재하는 member들의 수를 가져온다. min,max에 (를 앞에 붙이면 미만 초과로 바뀐다.
- ZPOPMIN c_key [rank] : 가장 작은 숫자를 삭제한다. rank는 n번째 수를 의미한다.
- ZPOPMAX c_key [rank] : 가장 큰 숫자를 삭제한다. rank는 n번째 수를 의미한다.
- ZINCRBY c_key add member : member에 존재하는 score에 value를 더한다. 뺄셈은 단순히 음수를 더하면 된다.
- ZRANGE c_key min_index max_index [BYSCORE] [WITHSCORES] [REV] [LIMIT] : rank가 min_index~max_index 사이에 존재하는 member들을 가져온다. WITHSCORE를 통해 score까지 가져올 수 있다. BYSCORE를 작성하면 min_index와 max_index가 min_score max_score로 취급된다.  
  REV는 sorted set의 순서를 reversed 후 연산한다. LIMIT은 처음 offset만큼 건너뛰고 limit만큼 가져온다.(Pagination) Ex) limit offset limit

## Use Cases

- 여러 collection of Hashes에서 '가장 큰' 또는 '가장 작은'을 적용하여 결과를 가져와야 할 경우
  Ex) 책에 대한 정보를 가지고 있는 book#id Hash들 중 가장 많은 리뷰를 가진 book은? Sorted Set에서 member로 id score로 리뷰 수를 저장한 후 ZPOPMAX를 사용해 가져온다.
- 여러 Data들간 관계를 생성한 후 어떠한 기준으로 정렬해야하는 경우
  Ex) 작가가 작성한 책들 중 가장 많이 팔린 책은? 작가와 책 모두 Hash로 따로 저장됨 이를 author:book#id를 통해 관계를 나타내고 Sorted set에 저장한 후 가져오면 된다.
  현실에서는 Data가 정렬되어야할 필요가 많기에 Sorted Set을 이용할 일이 많다. 특히 createdAt을 위해 많이 사용

## Bypassing Strings in Sorted Sets

기본적으로 score는 숫자만 가능하다. 하지만 userId와 같이 여러 상황에 따라 String을 저장하고 싶을 때가 존재한다.  
userId의 경우 hexadecimal을 통해 userId를 생성하기에 이를 decimal로 바꾼 다음 저장하면 된다. 그리고 추후 다시 가져올 때는 다시 hexa로 바꾼다.
