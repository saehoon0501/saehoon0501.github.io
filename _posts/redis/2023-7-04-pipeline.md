---
title: Pipelining commands
date: 2023-07-04 12:10:00 +/-0
categories: [redis]
tags: [redis-pipeline] # TAG names should always be lowercase
---

Redis에서 Data를 가져올 때는 하나씩 가져올 수 있지만 현실에서 item을 이렇게 반복하기에는 시간이 너무 길게 걸린다.
따라서 원하는 데이터들을 가져오는 query들을 queue에 담아서 Redis에 한번에 Batch처리를 하면 원하는 Data들의 list를 가져오게 된다.  
NodeJs에서는 Promise.all(mapping function)이나 직접 명시하는 등을 사용한다.  
이외에는 pipepline()을 통해 pipe object를 가져온 후 .set이나 .get과 같은 command들을 차례대로 사용한 후 .execute()을 호출하면 된다.
