---
title: Redis Modules
date: 2023-07-07 12:10:00 +/-0
categories: [redis]
tags: [redis-module] # TAG names should always be lowercase
---

Redis에는 기본적으로 존재하는(Core) 자료구조로 Strings, Sets, Sorted Sets, Hashes, Lists, HyperLogLogs가 존재한다.
하지만 좀더 원활한 작업을 위해 Module을 통해 추가적인 자료구조를 사용할 수 있으며, 이를 Module로 가져온다.
이러한 자료구조에는 RedisJSON과 RediSearch 등이 존재한다. 이는 Redis Stack 다운로드를 통해 사용 가능하다.
하지만 배포하는 경우에 따라 이러한 Module을 사용하지 못할 수 있다. Ex) AWS에서 자체적으로 운영하는 Redis를 사용하는 경우
이럴 경우 Redis에서 지원하는 Redis Lab Manager를 사용하거나 직접 Redis를 setup하여 사용한다.

RediSearch
기본적으로 서로 다른 Hash들 사이에서 Filtering을 통해 Data를 가져오기 위해서는 Sorted Sets을 사용해야 한다.
이는 Lua 또는 sort command를 통해 이뤄질 수 있지만 매우 번거롭다.
RediSearch에서는 이러한 여러 Hash/ReidsJSON들을 다른 Data Structure를 사용하지 않고 쉽게 Search할 수 있도록 해준다.

사용 순서

1. Create an Index : Index는 우리가 찾고자 하는 Data들을 가리키는 Data이다. 이러한 Data들을 찾기 위한 Index는 한번만 생성하면 된다.
   Ex) 'items#'으로 시작하는 모든 c_key를 찾고 이들의 key와 value를 저장한다.
2. Run a query : Index를 사용해 query의 기준에 맞는 Data에 들어있는 특정 key에 대한 value를 가져올 수 있다.
   이러한 querying은 기준에 정확히 부합하는 결과를 가져오는 것을 말한다. 이와 다르게 기준에 정확히 부합하지 않지만 가장 가깝거나 또는 최선의 결과를 가져오는 것은 Searching이라고 하며, RediSearch에는 이러한 두 가지 나눠서 이해하면 편하다.

Creating index
FT.CREATE idx:name ON [HASH|JSON] PREFIX num prefix*name1 SCHEMA attr1 data_type attr2 data type*

- idx는 index의 이름을 말한다.
- ON은 찾고자하는 Data Structure를 설정하며, hash 또는 RedisJSON을 설정할 수 있다.
- PREFIX는 찾고자하는 Data Structure에 저장된 c_key들의 고정되는 이름으로 num의 크기만큼 prefix name을 주어 이를 포함하는 모든 c_key에 대한 정보를 저장한다.
- SCHEMA의 경우 해당하는 Data에서 어떠한 key를 저장할지 나타낸다.
  가능한 Data Type에는 NUMERIC, GEO, VECTOR, TAG, TEXT가 존재한다.
  이러한 index를 생성하기 전 FT.\_LIST를 통해 이미 기존에 index가 생성되었는지 확인 후 App이 시작될 때 생성한다.
  기존에 생성되어 있는 index에 결과에 원하는 field를 추가할 수 있는지만 기존에 field에 변경하는 것은 반영이 안된다. 따라서 별도로 삭제 후 App을 재시작하여야 한다.

Querying/Searching
TAG Data Type의 경우 Querying에 해당하며, TEXT의 경우 Searching에 해당한다.
따라서 TAG의 경우 정해진 값들 중 하나를 선택하여 찾는 거라면, TEXT의 경우 어떠한 값을 넣어도 되며 이에 최대한 가까운 것을 찾는다.
()은 TEXT, {}은 TAG, []은 숫자
Ex)

```
FT.SEARCH idx:cars '-@year: [(1995 (1980] @color:{blue|red}'//index cars에서 year가 1996~1979가 아닌, Tag값이 blue or red인 것
```

''안에 띄워쓰기는 \로 대체한다.

TEXT나 TAG를 query/search할 때 stop word가 존재하여 filtering이 된다. Ex) a this to or
TEXT에서는 stemming을 통해 유저가 검색한 단어를 변형하여 searching한다. Ex) fastly -> fast
단순히 value만 넣으면 해당하는 TEXT value를 모두 가져온다.
@name:(fast car) = fast and car
@name:(fast | car) = fast or car
-@name:(fast car) = not including fast

Fuzzy Search
%를 이용해 단어가 다르더라도 비슷한 value를 찾는다. %수 만큼 글자가 다른걸 찾으며 최대 3개까지 가능.
주로 추천 검색어에 사용한다.

```
FT.SEARCH idx:cars '@name:(%dar%)'
```

Prefix Search \*를 이용해 TEXT 중 일부분을 포함하는 결과를 찾는다. 주로 auto-complete에서 사용

```
FT.SEARCH idx:cars '@name:(fa*)'
```

Ex)
유저가 검색어로 fast ca를 입력하면 이를 split하고 각 단어에 _를 붙여 'fast_' and 'ca*' 또는 'fast*'or 'ca\*'의 searching 결과를 가져올 수 있다.

sort
index를 생성할 때 sort할 key에 대한 sortable: true 정보를 추가시키면 추후 query에서 결과를 찾은 후 이에 따라 sort할 수 있다.
단 sort는 한번에 한 key에 대해서만 가능하다.
