---
title: Redis Design Methodology
date: 2023-07-03 18:10:00 +/-0
categories: [redis]
tags: [redis-design_methodology] # TAG names should always be lowercase
---

일반적인 DB의 경우 어떤 Table 형식에 Data를 보관할지 먼저 고민한 후 어떤 query를 이용해 Data를 가져올지 고민한다.
Redis의 경우 먼저 어떤 query를 이용해 Data를 가져올지 고민 후 Data에 맞는 Data structure를 고민한다.  
Design Considerations

- What type of data are we storing?
- Should we be concerned about the size of data?
  Data당 평균적인 크기 _ 유저 수 _ 유저당 필요한 Data를 계산하면 대략적인 감을 찾을 수 있다.
- Do we need to expire this data?
- What will the key naming policy be for this data?  
  key는 항상 Unique해야 한다. 그리고 저장하는 Data를 직관적으로 나타내어야 한다.  
  :을 사용하여 관계를 나타내어 체계적인 naming 가능하다. 또한 identifier에만 #를 사용하여 searching이 더 쉽게 할 수 있다. Ex) users:posts:15, posts#3  
  보통 이러한 key는 매번 직접 작성하는 것이 아니라 함수를 작성하여 typo가 발생하지 않게 사용한다.

```javascript
export const pageCacheKey = (id: string) => `pagecache#${id}`;
```

- Any business-logic concerns?
