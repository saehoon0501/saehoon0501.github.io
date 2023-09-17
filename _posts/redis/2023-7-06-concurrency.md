---
title: Concurrency Issue
date: 2023-07-06 19:10:00 +/-0
categories: [redis]
tags: [redis-concurrency] # TAG names should always be lowercase
---

uniqueness를 요구하는 경우 Data validation을 하여 이를 유지하도록 한다. 하지만 만약 중복되는 Data 동시에 서버에 요청되면 둘다 동시에 Validation을 통과하여 중복되는 Data가 저장되게 된다. 따라서 이러한 Concurrency를 방지하기 위한 Solution이 요구된다.

해결방안

- Use an atomic update command(Like HINCRBY or HSETNX)
- Use a transaction with the 'WATCH' command
- Use a lock
- Use a custom LUA update script

## Atomic update

GET과 UPDATE를 atomic하게 처리하여 두 operation 사이에 gap을 없앤다.  
Redis는 Single Thread이기에 atomic한 operation을 한번에 하나씩만 수행하게 되어 Data 중복이 사라진다.
하지만 Redis에서 이러한 operation을 지원하지 않는 경우가 많다.

## Transaction with 'WATCH'

순차적으로 실행하고자 하는 operation을 그룹으로 만들어 한번에 처리한다.  
pipelining과 비슷하지만 다른 점은 어떠한 operation도 해당 그룹을 모두 순차적으로 처리하기 전까지 실행되지 않는다.  
다른 DB에서는 Transaction을 되돌리거나 취소할 수 있지만 Redis에서는 불가능하다.  
몇몇 Concurrency 문제를 정확히 해결할 수 있지만, WATCH를 통해 valid한 실행을 단지 다른 곳에서 미리 실행하고 있다는 이유로 극단적으로 취소시켜 또 다른 문제를 야기할 수 있다.  
이러한 Transaction은 필요할 때마다 dedicated connection을 생성해서 사용/종료하고 Transaction 이외 다른 작업들은 별도의 connection에서 작업한다.  
Ex)

```
//c_key에 대한 어떠한 Update가 이뤄지고 있다면 Transaction을 취소한다. 이는 다음 Transaction이 실행될 때까지 유지된다.
WATCH c_key
//Transaction 시작을 알린다.
MULTI
//Transaction 내에서 실행하고자 하는 opeartion들을 작성
SET
SET
SET
//작성한 operation을 시작하고자 Redis에 알림
EXEC
```

실제 코드

```javascript
return client.executeIsolated(async (isolatedClient) => {
  await isolatedClient.watch(itemsKey(attrs.itemId));

  const item = await getItem(attrs.itemId);
  if (!item) {
    throw new Error("Item does not exist");
  }
  if (item.price >= attrs.amount) {
    throw new Error("Bid too low");
  }
  if (item.endingAt.diff(DateTime.now()).toMillis() < 0) {
    throw new Error("Item closed to bidding");
  }

  const serialized = serializeHistory(attrs.amount, attrs.createdAt.toMillis());

  return isolatedClient
    .multi()
    .rPush(bidHistoryKey(attrs.itemId), serialized)
    .hSet(itemsKey(item.id), {
      bids: item.bids + 1,
      price: attrs.amount,
      highestBidUserId: attrs.userId,
    })
    .exec();
});
```

## Use a Lock

RedLock Algorithm 등과 같은 유명한 Lock Algorithm을 라이브러리로 쉽게 사용 가능하다.  
접근하고자 하는 Resource에 대한 Lock을 얻은 작업만 실행되게하고 같은 Resource에 대한 다른 작업들은 Lock을 얻기 위해 계속 기다리고 있는다. Lock을 얻은 작업은 실행을 다 마치고 나면 Lock을 되돌린다.  
여기서 유의할 점

- Lock을 얻은 작업을 실행하는 도중 Server가 다운되어 영원히 Lock이 잠긴 상태로 남아 있을 수 있기에 Redis에서 PX를 이용해 특정 시간이 지나면 Lock이 자동 해체되는 등의 대체 방안을 고려해야 한다.
- 어떠한 이유로 Lock이 시간이 지나 자동으로 풀리면 기존에 Lock을 얻었던 작업은 중단되어야 한다.
  수업에서는 이를 위해 lockedClinet Proxy를 생성하여 해당 시간이 지났는지 check 후 cb함수를 수행하여 작업을 수행했다.
- Lock을 얻었던 작업이 수행된 후 Lock을 삭제하기 직전이나 이전에 Lock이 시간 만료로 자동 삭제가 되었으면 이를 확인해 Lock을 삭제하면 안된다.
  수업에서는 이를 Lua Script로 수행하여 if 조건을 통해 기존 Lock의 값이 해당 작업이 설정한 값과 일치할 경우에만 Lock을 삭제하도록 했다.
- Lock을 얻었던 작업에서 Error를 던질 수 있기에 무조건 실행되는 finally문을 통해 어떠한 일이 발생하더라도 항상 Lock을 해체하고 나가도록 한다.

## Extending Redis with Scripting

Lua를 통해 Script을 작성하여 원하는 Data를 가져오기 위해 모든 Data를 가져온 후 Filtering하지 않아도 된다.
이를 통해 필요한 round trip을 줄일 수 있으며 몇몇 경우 Concurrency 문제를 해결할 수 있다.

문제점

- 항상 접근하고 하는 KEYS에 대한 정보를 Script 작성 전에 미리 알아야한다.(argv로 제공하는 방식을 사용하기에)
- script code를 테스트하기 어렵다.
- Type checking과 같은 언어의 특징을 사용할 수 없다.

### Basic of Lua

Redis에서 사용하기 위해서는 변수는 local을 붙여 지역변수로 만들자. local없이는 전역변수로 생성된다.

```lua
local sum = 1+1// 지역 변수

if sum > 0 then
    //something
end

if sum ~= 0 then// != 과 동일한 ~=
    //something
end
```

Lua에서는 0, ''값은 False로 처리되지 않고 True로 처리된다.

```lua
if nil then// nil은 null로 false 처리된다.
    //something
end
```

Lua에서는 Array의 index가 1부터 시작한다.

```lua
local colors = {'red', 'green', 'blue'} //array
print(#colors)//array size
table.insert(colors, 'orange')//array append

for i, v in ipairs(colors) do//i=index, v=value
    print(i,v)
end

for i=5, 10 do//5~10까지 i가 iterate
    print(i)
end
```

Lua에서는 table이 Object이다.

```lua
local user = {id= 'a1', name = 'james'}//object
print(user[id])//Object에 있는 field 접근 방법

for k,v in pairs(user) do//Object에서는 ipair 대신 pair를 사용해 iterate
    print(k,v)
end
```

### Lua Script in Redis

Lus Script를 작성하여 Redis에 보내면 Redis에서는 나중에 이를 사용하기 위해 저장하고 script ID를 서버에 보낸다.
EVALSHA script_id를 통해 해당 script를 Redis에서 실행한 후 결과를 서버에 return한다.
Ex)
Script를 load하여 Redis에 저장하면 id가 return 된다.

```
SCRIPT LOAD 'return 1+1'//서버에서 값을 받기 위해서는 항상 return을 작성해야 한다.
```

해당 id를 사용하면 Redis에서 실행 결과가 오게 된다.

```
EVALSHA script_id 0
```

Lua Script에 변수 값을 parameter로 대입하여 좀더 generic한 Script를 Redis에 load 할 수 있다.

```
SCRIPT LOAD 'return 1 + tonumber(ARGV[1])'//argument는 항상 string으로 주어지기에 int로 바꾸기 위해서는 이렇게 해야한다.

EVALSHA script_id 0 '1'//argument는 항상 string 형식으로 정해져야 한다.
```

Lua Script을 통해 Redis 작업을 할 때 Redis의 key에 접근해야 한다면 항상 항상 KEYS[n] argv를 통해 접근해야 하기에 key 값을 넣어주는 방식으로 사용한다. 따라서 script를 실행하는 결과에 따라 key를 dynamic하게 생성하여 사용할 수 없다.

```
SCRIPT LOAD 'return redis.call("GET", KEYS[1])'

EVALSHA script_id 1 'color'// scrip_id와 value 사이에 존재하는 number는 KEYS를 위한 argv의 개수를 의미한다.
```

### Javascript에서 Node-Redis Library 실제 사용

필요한 것들

1. JS code내에서 해당 script를 뭐라고 호출할 함수명
2. Script 자체
3. Script에 들어가야할 Key들의 수
4. JS 함수에서 EVALSHA에 argv를 넘기는 코드
5. script 실행 결과를 어떻게 parsing할지에 대한 코드
   Ex)

```javascript
const client = createClient({
  socket: {
    host: process.env.REDIS_HOST,
    port: parseInt(process.env.REDIS_PORT),
  },
  password: process.env.REDIS_PW,
  //Lua script들을 생성하여 작성하는 부분
  scripts: {
    addOneAndStore: defineScript({
      // ------------------1
      NUMBER_OF_KEYS: 1, // ------------------3
      SCRIPT: ` // ------------------2
				return redis.call('SET', KEYS[1], 1+ tonumber(ARGV[1]))
			`,
      transformArguments(key: string, value: number) {
        // ------------------4
        return [key, value.toString()];
      },
      transformReply(reply: any) {
        // ------------------5
        return reply;
      },
    }),
  },
});
```

이를 통해 Redis connection을 나타내는 client에서 lua script을 사용하여 우리가 원하는 결과를 수행하도록 함수를 호출 가능하다.
