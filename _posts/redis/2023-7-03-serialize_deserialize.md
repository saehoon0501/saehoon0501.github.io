---
title: Serialization and Derialization
date: 2023-07-03 21:10:00 +/-0
categories: [redis]
tags: [redis-serialize] # TAG names should always be lowercase
---

같은 Data라도 Redis에 저장할 때는 Sorting이 가능한 형식이나 저장에 불필요하다 생각되는 attr가 존재할 수 있고 Redis에서 가져올 때에요 App에 사용하는 형식에 맞게 Data를 바꿔줘야할 수 있다. 따라서 이를 위해 Serialization와 Deserialization 함수를 작성한다.  
Ex)

- DateTime
  createdAt과 같이 Date format을 다룰 때 Datetime을 바로 toString()하는 Redis 라이브러리이기에 저장되는 형식이 Sorting이나 Searching이 불가능한 형식이다. 이를 위해 serialization 함수에서는 createdAt에 들어가는 값을 toMillis()로 바꾼 후 return한 후 Redis에 저장한다.
  반대로 이제 가져올 경우 Deserialization 함수에서는 Datetime.fromMillis(parseInt(item.createdAt))을 통해 Datetime 형식으로 바꾼 후 return하여 App에서 사용할 수 있는 형식으로 바꾼다.
- session
  session을 hash로 저장하여 collection_key에는 Data에 대한 id가 이미 들어가있기에 굳이 attr에 또 id를 집어넣어줄 필요는 없으며, Redis 특성 상 Memory는 비싸기에 최대한 작게하기 위해 serialize하여 없앤다.
  App에서는 collection_key에 존재하는 id가 다시 필요하기에 deserialize하여 Data에 id 필드를 추가시켜 return한다.

generate ID and Key  
Redis에서는 저장하는 데이터 key의 id를 생성해주지 않는다. 따라서 crypto와 같은 라이브러리를 통해 매번 랜덤 + 고유한 id를 만들어주는 함수를 작성한다.  
또한 매번 key를 직접 입력하는 것은 typo 가능성을 높히기에 각 Data마다 key 이름을 생성하는 함수들을 하나의 파일에 몰아서 작성한 후 이를 가져와 사용한다.
