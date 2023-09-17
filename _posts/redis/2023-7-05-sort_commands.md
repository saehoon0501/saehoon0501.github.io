---
title: Sort Command
date: 2023-07-05 13:10:00 +/-0
categories: [redis]
tags: [redis-sort_command] # TAG names should always be lowercase
---

set, sorted set 그리고 list에서 사용한다. sort 결과는 sorting criteria에 대한 value들은 빠진채 받을 수 있다.  
sort을 실행할 때는 member에 대한 sorting을 수행하며, 여기서는 member를 score라고 부른다. 따라서 여기서는 기존 member를 score라고 생각한다.  
따라서 만약 member가 string으로 이뤄져 있다면 ALPHA option을 넣어 alphabetic 순서로 정렬하도록 한다.

## BY

```
SORT books:likes BY books:*->year
```

작동 순서

1.  books:likes에 있는 member들만을 가져온다.
2.  member들 중 하나를 \*에 집어넣어 books:member의 key를 만들어 Hash에 해당 c_key를 찾는다.
3.  ->year를 수행하여 해당하는 Hash에 year가 있는지 확인한다.
4.  존재하는 경우 해당 year key값을 member와 연관시켜 해당 member를 sorting하는데 사용한다.(=year를 sorting criteria로 사용한다)
5.  2~4번 과정을 books:likes에 있는 나머지 모든 member들에게 적용한다.
6.  sorting 수행 결과에서 member값들만 가져온다.
    만약 BY를 수행하고 싶지 않을 경우 nosort 값 또는 Hash에 존재하지 않는 랜덤한 Key를 넣어주면 된다.

## GET option을 사용한 join

```
SORT books:likes BY books:*->year GET books:*->title
```

BY의 수행결과에서 각 member에 해당하는 Hash c_key에서 title key의 value를 가져와 해당 member 옆에 join한다.  
그리고 member는 버리고 GET에 명시되었던 value만을 return한다. 이러한 GET을 여러번 활용해 원하는 여러 key에 대한 value들을 가져올 수 있다.

## Summary

```
client.sort(itemsByViewsKey(), {
    GET: ['#', `${itemsKey('*')}->name`, `${itemsKey('*')}->views`],
    BY: 'score',
    DIRECTION: order,
    LIMIT: {
        offset,
        count
    }
});
```

1. items:views에 있는 member와 items:member를 가지는 Hash의 name과 views 값을 가져온다.
2. 그 후 items:views의 score를 기준으로 order에 맞게 오름차순/내림차순으로 정렬한다.
3. 거기서 offset만큼 건너뛰고 count만큼의 Data를 가져온다.
4. 가져온 결과는 [member, name, views ...]의 반복되는 결과이다. 사용하려면 Parsing이 필요하다.
   이외 방법으로는 단순히 zRange와 같은 방법으로 모든 ids들을 가져와 하나씩 전부 hGetAll을 하는 Promise들을 Promise all 안에 넣어 수행하는 방법이 존재한다.
