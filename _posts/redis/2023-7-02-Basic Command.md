---
title: Basic Commands
date: 2023-07-02 18:10:00 +/-0
categories: [redis]
tags: [redis-basic] # TAG names should always be lowercase
---

모든 GET과 SET 관련 명령어를 외울 필요 없다. 이 중 몇몇은 매우 비슷하게 작동하기 때문이다. 모를 땐 Doc을 찾고 대략적인 아이디어를 파악을 하자.  
Doc에서 명령어 설명을 읽을 때 대문자는 keyword이며, 소문자는 값을 집어넣어야 하는 자리, []안에 있는 커맨드는 optional, |는 or를 의미한다. |에 있는 것들 중 하나만 사용하여야 한다.

## SET key value [EX | PX | EXAT | PXAT | KEEPTTL] [XX | NX] [GET] (Type:String/Num)

- [GET] 의 경우 만약 key에 값이 있었을 경우 해당 값을 가져온 후 value를 새로 할당한다.
- [XX \| NX]의 경우 XX는 기존에 key가 존재할 경우 SET을 수행하며, NX는 반대로 기존에 key가 없을 경우 수행한다.
- [EX | PX | EXAT | PXAT | KEEPTTL]의 경우 key value가 언제 만료(삭제)될지 설정하는 값으로 각 keyword마다 시간 단위가 다르다.  
  이를 사용하는 이유는 Data가 Caching 되었을 경우 API에서는 Redis를 먼저 접근해 Data가 있으면 이를 가져와 바로 Respond를 하고 없으면 DB에서 찾아서 이를 사용한다. 이러면 Redis에 Data가 저장되었을 때 빠르긴 하지만 Data가 계속 존재하기에는 Memory가 작기에 결국에는 초과가 날 수 있다.
  따라서 이를 방지하기 위해 몇 초간 해당 데이터가 사용되지 않는다면 이를 삭제하는 것이 효율적이다.
  SETEX은 SET key value EX value와 동일하다.
  SETNX는 SET key value NX와 동일하다.

## MSET key value key value(Type:String/Num)

key value pair를 한번에 여러개 실행하여 저장한다.  
MSETNX는 NX가 있기에 존재하지 않는 key에 대해서만 MSET을 수행한다.

## GET key(Type:String/Num)

key의 value를 가져온다.  
MGET key key  
여러 key에 대한 value들을 array로 가져온다.

## DEL key(Type:All)

key와 value를 삭제하며 모든 Data Type에 적용된다.

## GETRANGE key from to(Type:string)

key에 있는 string에서 from to range에 있는 chars만 가져온다.

## SETRANGE key at value(Type:string)

key에 있는 string에서 at에 있는 부분부터 끝까지 value로 대체한다.

## 이러한 String Command들을 사용한 Example

수많은 item들이 저장된 DB가 존재하며 각 item마다 3가지 properties가 존재한다 가정  
이러한 Item의 properties의 값들을 전부 명시하기보다는 존재할 수 있는 값들 모두 Encode하여 하나의 char로 나타낸다.  
이렇게 한 후 Redis에 item:{id} value 와 같은 key value형식으로 item들을 저장한다.  
만약 사용자가 한 Item에서 특정 property를 가져오고 싶다면 GETRANGE itemnum 0 0와 같이 string 중 특정 부분만을 빠르게 가져온다.
한 Item에서 특정 property를 update -> SETRANGE itemnum 0 value
여러 Item을 가져옴 -> MGET Item:1 Item:2 Item:3
새로운 여러 Item들을 생성 -> MSET Item:4 trq Item:5 nzq
이와 같이 단순해 보이는 Command를 활용해 매우 효율적으로 API 응답 속도를 높힐 수 있다.

## Numbers in Redis

Redis에서는 숫자를 저장할 때 string으로 저장하기에 GET으로 가져온 후 숫자로 parsing한다.  
INCR key : key에 저장된 숫자에 +1  
DECR key : key에 저장된 숫자에 -1  
INCRBY key value : key에 value만큼 +  
DECRBY key value : key에 value만큼 -  
INCRBYFLOAT key value : key에 float value만큼 +, 만약 -를 하고싶다면 value에 그냥 -를 넣자.  
이러한 숫자를 다루는 cmd는 GET SET으로 대체가능하지만 한 keyword로 수행할 수 있게 해주고 만약 여러 Client가 한 Data를 변경시킬 때 Atomic한 수행이 가능하게 하기에 존재한다. 이외 Racing condition을 방지하는 방법에는 WATCH 또는 lock을 사용할 수 있긴하다.
