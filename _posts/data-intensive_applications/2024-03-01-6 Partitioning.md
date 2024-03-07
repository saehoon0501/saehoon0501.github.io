---
title: Partitioning
date: 2024-3-01 22:30:00 +/-0
categories: [data-intensive]
tags: [data-intensive, partitioning, rebalancing] # TAG names should always be lowercase
---

매우 큰 dataset, throughput을 위해서는 replication 뿐만 아니라 partitioning도 필요로 하다.
_partitions_(aka _sharding_)은 HBase에서는 _region_, Cassandra와 Riak에서는 *vnode*라고도 한다.

partitions는 하나의 data(row 또는 document 등)을 한 partition에 저장하는 의미한다.
사실상 partition들은 각각 작은 db를 의미하며, db에서는 여러 partitions를 한번에 사용하는 작업을 제공하기도 한다.

partitions를 사용하는 가장 큰 이유는 *scalability*를 위해서이다.
매우 큰 dataset들을 여러 partitions로 나눠 여러 node에 보관하면 작업을 위한 query을 분산할 수 있기에 과부하가 걸리는 것을 방지할 수 있다.
여러 data들을 한번에 query하는 경우 여러 partition에서 parallel하게 수행이 가능하다.(구현이 좀더 힘들어지긴 한다.)

OLTP와 OLAP에서 둘다 사용된다.

# Partitioning and Replication

partitioning과 replication은 보통 같이 사용된다. 하지만 둘의 scheme은 독립적이기에 어떻게 하든 서로 영향을 미치지 못한다.
큰 dataset을 어떤 기준으로 partitioning을 진행하면 여러 node에 data들이 나눠져 저장된 상태이다.
그럼 여기서 replication을 통해 system에서 partition된 node들을 중복으로 존재하여 fault tolerance를 높힌다.

leader-based replication에서 partitioning을 진행한 모습을 Fig 6-1에서 확인 가능하다.

_Figure 6-1. Combining replication and partitioning: each node acts as leader for some
partitions and follower for other partitions_

![Screenshot 2024-03-02 at 2.24.01 AM.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-1.png)

각 partition의 leader가 하나의 node에 배정되고 나머지 node에 존재하는 partition들은 follower로 존재한다.
따라서 각 node들은 다른 node의 follower인 partition을 가지면서 동시에 leader인 partition을 가질 수 있다.

# Partitioning Key-Value Data

partition을 수행할 때 data들을 어떻게 나눌지에 대해 알아보자.

data를 잘 나눴다고 소문나려면 우선 query가 균등하게 분산될 수 있게 나누는 것이다.
이를 통해 이론상 만약 10개의 node가 존재한다면 10배의 read/write throughput을 보여줄 수 있다.

불균등한 상태를 *skewed*라고 한다.
skew가 발생하면 특정 node에만 query가 몰려 system에 bottleneck 현상이 오게된다.
그리고 이렇게 query가 몰리는 node들을 *hot spot*이라 부른다.

균등하게 나누는 방법 중 가장 쉬운 방법은 random하게 data들을 분산하는 것이다.
하지만 이럴 경우 나중에 query할 때 기준이 없기에 어느 node에 보내야하는지 알지 못해 모든 node에 대해 parallel하게 query를 보내야해서 비효율적이다.

그럼 쓸만한 방법들에 대해 알아보자.

## Partitioning by Key Range

primary key와 같이 특정 key값을 연속되는 범위로 쪼개 partition들에 나누는 방법이다.
마치 백과사전에서 알파벳 순으로 단어들을 나열하듯 여기서도 a~d는 한 partition으로 e~h는 다른 partition으로 나눠 저장하는 방식이다.

_Figure 6-2. A print encyclopedia is partitioned by key range._

![Screenshot 2024-03-02 at 2.40.52 AM.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-2.png)

이렇게 partitioning을 하면 어떤 값이 어떤 node에 가야할지 쉽게 알 수 있기에 request를 보낼 때 쉽게 해당하는 node로 routing이 가능하다.

key를 range로 나누는 것이 data를 균등하게 나눴다는 의미는 아니다.
예를 들어, a와 b로 시작하는 단어가 x,y,z로 시작하는 단어보다 훨씬 많을 수 있다.
따라서 range(partition boundary)는 dataset에 맞춰 이뤄져야 한다.
이는 administrator가 직접 선택하거나 DB에서 자동적으로 선택하게 할 수 있다.

각 partition 내에서는 key들을 sorted order로 정렬하여 저장하는 경우 scanning에 효율성을 높힐 수 있다.
만약 concatenated index를 활용하여 정렬하는 경우 하나의 query에서 조건에 해당하는 data들을 찾을 수 있다. 예를 들어, timestamp(_year-month-day-hour-minute-second_)를 key로 활용하는 경우 특정 원하는 시간대 범위를 조건문으로 하여 data를 쉽게 찾을 수 있는 것이다.

<aside>
☝ concatenated index는 일대다 관계를 아주 효율적으로 나타내준다.
예를 들어, SNS에서 user와 posts 사이의 관계를 나타낼 때 primary key를 (user_id, update_timestamp)로 설정하여 user가 특정 시간대에 post에 update 내용들을 가져올 수 있다.

user들마다 다른 partition에 저장될 수 있지만, 각 user들마다 정해진 partition에서 update 내용들이 timestamp로 sorted ordered되어 저장되어 있는 것이다.

</aside>

key range partitioning의 단점으로는 작업에 따라 특정 node가 hot spot이 될 수 밖에 없다는 점이다.
timestamp를 concatenated index로 활용하는 경우 오늘 작업하는 data들은 모두 해당하는 node에만 write이 되기에 나머지는 idle 상태이고 오늘에 해당하는 node가 hot spot이 되는 것이다.

이를 해결하는 방법은 prefix를 사용하는 것이다.
prefix를 sensor들의 이름으로 활용하는 경우 먼저 sensor name에 따라 partitioning을 하고 그 다음 time에 따라 partitioning이 이뤄지는 방식이다. 이러면 같은 time이지만 sensor name에 따라 할당되는 partition이 다르기에 write이 분산되어 한 node에 몰리는 것을 방지할 수 있다.
read의 경우도 위와 동일하게 분산될 수 있다.

## Partitioning by Hash of Key

skewed data나 hot spot을 피하기 좀더 확실한 방법이다.

32bit hash function을 기준으로 만약 string을 input으로 넣으면 0 ~ $2^{23}$ - 1의 random한 숫자가 나오게 된다. 만약 유사한 string들을 hash값으로 바꿔도 이들의 값은 전혀 다를 것이다. 따라서 skewed data여도 균등하게 나눌 수 있게 된다.

partitioning을 위해 hash function을 사용할 경우 암호학적으로 강력한 것이 필요하진 않다.
값을 암호화 하는 것이 목적이 아니라 유사한 값이라도 균등하게 분산하는 것이 목적이기 때문이다.
programming language에서 자체적으로 제공하는 hash function의 경우에도 만약 같은 key라도 프로세스에 따라 그 값 자체가 완전히 달라질 수 있기에 사용하지 말아야 한다. 사용할 경우 이전에 나눈 값들이 어느 partition에 갔는지 알지 못하게 되기 때문이다.

결과적으로 적절한 hash function을 사용해 data들의 hash 값을 받았다면 이들이 가질 수 있는 값들을 range로 나눠 저장하면 된다. 사실상 partitioning by hash of key range이다.

_Figure 6-3. Partitioning by hash of key._

![스크린샷 2024-03-03 오후 4.21.58.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-3.png)

이를 위해 boundary는 총 범위를 node 수로 나눈 값을 사용할 수 있거나 pseudorandom(_consistent hashing_)하게 설정될 수 있다.

<aside>
☝ Consistent Hashing이란?
CDN과 같이 cache에 들어오는 load를 균등하게 분산하는 방법이다.
이를 위해 랜덤하게 선택된 partition boundary를 사용하여 여러 cache에서 rebalancing에 필요한 consensus 또는 central control의 필요를 없앤다.(이를 위해 hash ring을 사용) 따라서 여기서 consistent는 replica consistency 또는 ACID consistency와는 상관없고 rebalancing을 하는 방법을 설명하는 것이다.

DB에서는 이 방법이 잘 먹히지 않아 실제로는 거의 사용되지 않는다.
consistent hashing이라는 단어 자체가 혼동을 주기에 그냥 *hash partitioning*이라 부르자.

</aside>

hash를 이용할 경우 partitioning 내 값들을 sorted order로 가질 수 없어지기에 scanning이 비효율적이게 된다는 단점이 존재한다. key range의 경우 서로 인접해야할 data들이 hash에 따라 서로 골고루 분산되면서 range query를 할 때 parallel하게 모든 node에 보내야된다.
MongoDB은 위와 같이 작동하며, Riak, Couchbase, Voldemort는 primary key에 대해 query 자체를 지원하지 않는다.

Cassandra의 경우 key range와 hash of key를 혼합하여 사용한다.
table에서 *compound primary key*를 선언하면 \*\*맨 앞의 값은 hash of key를 통해 partition을 할당하는 데에만 사용하며, 나머지 값들은 concatenated index로, key range로 사용하여 sorted order로 값들을 저장할 수 있게 된다.
따라서 compound key에서 맨 앞부분은 range query에 사용될 수 없지만 나머지 부분에 대해서는 수행이 가능하다.

## Skewed Workloads and Relieving Hot Spots

key를 hashing하는 것이 hot spot을 줄이고 skewed data를 균등하게 분산할 수 있다고 했지만 여전히 극단적인 상황에서는 하나의 node에 write/read query가 몰릴 수 있다.

만약 유명한 셀럽이 post를 작성하였고 많은 팬들이 댓글 작성과 post를 보려고 몰릴 경우 아무리 hash를 통해 data를 분산했다고 하여도 특정 post 자체(hot key)에 load가 집중되기에 hot spot이 발생하게 된다.

이럴 경우 특정 post의 key에 2자리 랜덤한 숫자를 붙여 같은 data에 대한 write을 여러 개의 partition을 통해 write load를 분산할 수 있지만 이에 대한 내용을 추적하는 bookkeeping이 필요하게 된다.

이때 post를 read하는 경우 이러한 bookkeeping에 기록된 내용들을 거쳐 write된 내용들을 가져와야하는 추가적인 작업이 발생하며, 단순히 전체 작업을 이렇게 만들기에는 소수의 hot key들로 인해 나머지 key들 또한 불필요한 overhead가 발생하게 된다. 따러서 hot key와 일반 key를 구분하여 처리하기 위해 분산한 소수의 hot key들에 대한 내용도 따로 추적해야 한다.

현재까지 이러한 hot key들을 알아서 탐지하고 보완하는 storage system이 없기에 application layer에서 이를 직접 처리해야 한다.

# Partitioning Secondary Indexes by Document

지금까지 key-value data에서 primary key를 통해 고유한 data들을 access하는 pattern에 대해 다뤘다면 secondary index의 경우 이와 다르기에 다른 partitioning이 필요하다.

secondary index는 이에 해당하는 고유한 data를 찾는 것이 아닌 이를 포함하는 모든 data를 찾아야 한다.(searching for occurrences of a particular value)

- user123이 했던 모든 action들을 찾는다.
- hogwash라는 단어를 포함하는 모든 기사를 찾는다.
- red 색상을 가진 모든 자동차 data를 찾는다.

secondary index는 RDB에서 매우 중요하기에 많은 key-value DB에서 이를 지원하며, 특히 Elasticsearch와 Solr의 경우에 핵심으로 사용된다.

## Partitioning Secondary Indexes by Document

중고 차를 파는 사이트에서 document format으로 중고 차를 저장하고 고유한 document id(or primary key in RDB)들을 통해 partitioning을 진행하였다고 가정해보자.

만약 유저가 filter를 통해 특정 color와 maker를 통해 자동차들을 검색한다면 이를 secondary index로 활용하는 것이다. 따라서 color:red에 대한 entry에 field color:red에 해당하는 모든 document id를 list로 가지고 있다가 이를 결과로 return한다.

_Figure 6-4. Partitioning secondary indexes by document._

![스크린샷 2024-03-03 오후 5.49.34.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-4.png)

이 indexing 방법의 경우 각 partition들은 다른 partition들과 완전히 분리되어 있다.
partition 각자 자신들이 저장한 data에 대해서만 secondary indexing을 가지고 관리하기에 add 또는 update가 발생하여도 발생한 partition 안의 document id에 대해서만 secondary index의 변화를 처리하면 된다.
따라서 document-partitioned index를 *local index*라 부르기도 한다.

이때 read 작업의 경우 filter에 해당하는 모든 data를 찾기 위해서는 모든 partition을 뒤져봐야 된다.
만약 이를 parallel하게 처리한다고 하여도 tail latency amplification 현상으로 인해 delay가 발생한다.
이러한 query방식을 *scatter/gather*라 부르며, read query를 처리하는 비용이 매우 비싸다.

MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud, and VoltDB all use document-partitioned secondary indexes. 따라서 이 indexing 방식을 따를 경우 최대한 secondary index가 하나의 partition 내에서 처리 될 수 있게 partitioning을 하는 것이 최선이다.
하지만 만약 query에 여러 secondary index들이 들어가게 되면 결국에는 partition들을 다 뒤져봐야 한다.

## Partitioning Secondary Indexes by Term

partition마다 각자 secondary index들만 관리하는 대신 global indexing을 통해 모든 partition에 있는 data들을 포함하는 방법이다. 이러한 global index들도 partitioned되어 존재하며, primary key index와는 별개로 partitioned 될 수 있다.

_Figure 6-5. Partitioning secondary indexes by term._

![스크린샷 2024-03-03 오후 6.03.35.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-5.png)

위 그림에서는 global index인 color가 a~r로 시작하는 경우 partition 0로 나머지는 partition 1에 저장되어 관리되며, maker의 경우에도 같은 방식이다.
이렇게 찾는 term이 partition을 결정하는 경우 *term-partitioned*라 부른다.

<aside>
☝ term이라는 단어는 full-text index에서 나왔으며, document에 등장하는 모든 단어들을 통칭한다.

</aside>

primary key index와 같이 term range 또는 term의 hash 값을 활용하여 partitioning이 가능하며 각 방법마다 같는 장단점 또한 동일하다.

document-partitioned 방식에 비해 term-partitioned 방식 얻는 이점은 read query를 처리하는데 더 효율적이다는 점이다. 하나의 partition에서 가지는 global index가 모든 partition에 대한 내용을 가지고 있기에 딱딱 필요한 partition들만 접근하여 data를 가지고 올 수 있다.

반면에 모든 partition에서 data가 update 또는 add될 때마다 global index에서 이러한 변경을 반영해줘야 하기에 write 작업이 더 느리고 복잡해진다.(심지어 하나의 data에 대한 변경이 여러 global index에 변경을 가져올 수도 있다.)

이상적으로는 write에 의한 변경사항이 global index에 즉각적으로 반영되어야 하지만, 이는 distributed transaction across all partitions affected by write을 요구하기에 대부분 async하게 처리된다.
따라서 현실에서는 변경사항에 대한 반영에 delay가 존재한다.

# Rebalancing Partitions

시간이 지남에 따라 DB에 발생하는 변화는 3가지 이다:

- query throughput이 증가함에 따라 이를 처리하기 위한 CPU를 더 증가시켜야 한다.
- dataset의 크기가 증가함에 맞춰 disk의 크기와 RAM의 크기가 증가해야 한다.
- 한 machine이 fail한다면 다른 machine이 이를 대체하여 기존 작업을 수행할 수 있어야 한다.

위의 모든 변화들은 모두 한 node에서 다른 node로 data 및 request load가 이동해야하게 하며, 이를 위한 작업을 *rebalancing*이라 한다.

어떤 scheme이든 rebalancing이 달성해야하는 최소한의 목표는 아래와 같다:

- load(data, read/write request)가 cluster 내 node들에게 분산되도록 한다.
- rebalancing 와중에 계속해서 들어오는 read/write request를 처리할 수 있어야 한다.
- 필요한 만큼의 data를 이동시켜 최대한 빠른 rebalancing 작업을 통해 disk 및 network IO에 부담을 주지 않는다.

## Strategies for Rebalancing

node에 partition들을 할당하는 방법들은 아래와 같다.

### How not to do it: hash mod N

partitioning에 있어서 hash of key를 사용할 때 왜 mod(the % operator)를 사용하지 않는지 의아해 할 수 있다. 만약 hash 값을 % N(node 수)으로 한다면 hash값 자체를 범위로 설정할 필요없이 쉽게 0~N-1 범위에서 range를 잡아 data를 나눌 수 있기 때문이다.

하지만 이렇게 구현했을 때 만약 N이 변화한다면 어떻게 될까?
이전에 partition에 할당되었던 대부분의 key들의 mod N 값이 바뀌게 되어 rebalancing할 때 이동해야 할 것이다. 이는 필요 이상의 data를 이동하게 만들어 불필요한 load를 가져온다.

예를 들어 , N = 10이였다고 하면 hash(key) = 123456일 때 partition 6에 할당될 것이다.
하지만 N이 11이 된다면 partition 3으로 이동해야할 것이고 12가 된다면 0으로 또 이동해야 된다.

### Fixed number of partitions

system에 존재하는 partition의 수를 고정으로 하는 방법이다.
이를 통해 처음 1000개의 partition을 100개의 node에서 가지고 있다가 새 node가 추가될 때 마다 partition을 기존 node에서 *steal*해 오는 것이다. 만약 node가 제거된다면 반대로 진행하면 된다.
따라서 partition 단위로 data 이동이 진행된다.

partition 단위로 data가 이동하는데 시간이 꽤 걸리기에 기존 node의 partition은 rebalancing이 완료될 때까지 정상적으로 기능을 수행하고 완료되는 시점에서 삭제된다.

_Figure 6-6. Adding a new node to a database cluster with multiple partitions per node._

![Screenshot 2024-03-03 at 9.22.13 PM.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-6.png)

이러한 방식은 system에 node의 hardware 성능의 mismatch를 고려하여 partition을 할당할 수 있다.
만약 특정 node의 성능이 더 좋은 경우 partition을 더 많이 할당하고 더 적은 경우 더 적게 할당하여 load를 균등하게 배분할 수 있다.

This approach to rebalancing is used in Riak, Elasticsearch, Couchbase, and Voldemort.

원칙적으로 기존의 partition을 합치거나 쪼갤 수 있지만 사용에 있어 심플함을 위해 많은 DB에서 이를 제공하지 않는다. 따라서 프로젝트 초기 partition의 수를 설정할 때 추후 scaling에 대한 고려를 한 partition 수를 설정해야 한다. 하지만 그렇다고 너무 많은 partition을 설정하면 각 partition마다 management overhead가 발생하기에 성능을 저하시킬 수 있다.

문제는 시간이 지남에 따라 dataset의 크기가 어떻게 변화할지 정확히 예측하는 힘들다는 점이다.
이렇게 fixed number of partition 방식을 사용하면 dataset 크기가 딱 알맞게 적당한 수의 partition을 가져가야 성능이 최적화 되는데, 너무 많게 설정할 경우 management overhead가 발생하고 너무 적은 경우 partition의 크기가 dataset의 크기에 따라 증가함에 따라 rebalancing과 recovery에 발생되는 시간과 비용이 증가하게 된다.

또한 key range 방식으로 partitioning을 진행할 경우 range에 맞춰 고정된 수의 partition을 설정할 때 비효율적이다. 만약 range를 잘못 설정할 경우 data가 특정 partition에 몰리게 되고 이를 수정하기 위해 매번 직접 boundary를 재설정해주는 건 너무 귀찮은 작업이기 때문이다.

### Dynamic partitioning

key range 방식에 알맞은 partitioning 방법이다.
시간이 지남에 따라 partition이 설정된 크기(HBase에서는 default로 10GB)보다 커진다면 partition을 대략 두개로 나누며, 반대로 크기가 작아질 경우 이를 인접한 다른 partition과 합친다.(마치 B-Tree에서 처럼)
partition이 2개로 나뉘는 경우 하나의 partition은 다른 node로 이동할 수도 있다.

이렇게 dynamic partitioning에서는 dataset의 크기가 맞춰 적당한 수의 partition을 자동적으로 조절해준다.

하지만 이 방식의 경우 처음 비어있는 DB가 시작할 때 data에 대한 어떠한 사전정보도 알지 못하기에 partition 크기가 만족할 때까지 하나의 partition을 가진다. 그리고 data가 증가할 때마다 split을 통해 boundary을 정하고 partition이 증가하여 load가 분산된다.
따라서 이러한 문제를 해결하기 위해 MongoDB와 HBase에서는 처음 DB를 시작할 때 초기 partition의 수를 설정할 수 있게 한다.(*pre-splitting*이라 한다.) 이 경우 boundary 설정을 위해 사전에 필요한 data 정보를 알고 있어야 한다.

hash of key 방식에도 적합하며 MongoDB에서는 두 가지 rebalancing 방식 모두 지원해준다.

### Partitioning proportionally to nodes

앞서 언급한 partitioning 모두 node의 수와는 독립적이다.
fixed number의 경우 dataset의 크기에 따라 partition의 크기가 비례하며, dynamic의 경우 partition의 수가 비례하게 증가한다.

Cassandra와 Ketama에서 사용하는 이번 방식은 partition의 수를 node 수에 비례하여 증가시키는 방법이다.
즉, node마다 고정된 수의 partition 수를 가지게 되는 것이다.

만약 dataset이 증가하게 된다면 partition의 크기가 증가할 것이지만 system에 node 수를 추가할 경우 partition의 크기가 다시 균등하도록 만들 수 있다.
node가 추가될 때 발생하는 과정은 다음과 같다.

1. data를 가져올 기존 partition들을 설정된 fixed number만큼 랜덤하게 선정한다.
2. 선정된 partition들에서 절반의 data만 이동하고 나머지는 그대로 둔다.

랜덤하게 partition을 선택하는 이 방식은 load의 불균등을 초래할 것 같지만 실제로 Cassandra처럼 node 당 256개의 partition을 가지고 있는 경우 랜덤하게 data를 가져와도 평균적으로 적당한 load를 분산하게 된다.

이렇게 랜덤하게 partition을 쪼개는 것은 partition boundary를 랜덤하게 설정하는 것과 같다.
따라서 hash-based partitioning을 사용해야 하며 그 중 consistent hashing을 통한 partitioning과 가장 잘 사용된다.

## Operations: Automatic or Manual Rebalancing

Rebalancing을 수행을 조작하는 여러 방식은 완전히 자동적으로 이뤄지는 방식과 완전히 수동적으로 이뤄지는 방식 사이에 존재한다.

Couchbase, Riak, and Voldemort의 경우 자동적으로 partition 할당에 대한 추천을 제공하지만 administrator의 허가를 받아야 한다.

만약 이를 전부 자동화 한다면 직접 관리를 할 필요가 없다는 점에서 유지보수의 이점이 있지만 rerouting, 많은 data의 이동들을 포함하는 작업이기에 쉽게 잘못될 수 있다.
특히 automatic failure detection 기능과 엮여 위험을 초래할 수 있는데, 만약 한 node가 overloaded되어 응답이 느려질 경우 다른 node들에서는 이를 failure로 보고 rebalancing을 수행할 수 있다. 그러면 overloaded된 node에 data를 가져오는 IO가 발생하여 load가 더 증가하게 된다. 그리고 이렇게 계속 node들에게 이어져 system 전체가 down될 수 있다.

따라서 자동화 과정에서 관리자가 직접 개입하여 rebalancing을 수행하는 경우가 가장 이상적이다.

# Request Routing

많은 node들이 partitioned된 data를 가지고 작업을 수행하고 있는 system에서 client는 어느 node로 request를 찾아서 보내야할까? 그리고 rebalancing이 이뤄져 data가 이동하면 rerouting은 어떻게 진행될까?에 대한 내용이다.

이러한 문제는 service discovery라 부르며, DB에 국한되지 않고 network에 접근 가능한 모든 app에서 발생되는 문제이다.

high level에서 보면 이 문제에 접근하는 방법들은 아래와 같다:

1. client에서는 system에 있는 어떤 node에도 request를 보낼 수 있으며, 해당 node가 data를 가지고 있으면 처리해주고 아닌 경우 이를 가진 node에 request를 보내 처리한 후 client에 response를 전달해 준다.
2. client와 db 사이에 routing tier를 둔다. 이는 partition aware load balancer로 들어오는 request가 찾는 data에 맞춰 이에 해당하는 node들을 알고 이를 forwarding 후 결과를 리턴한다.
3. client에서 어떤 node에 원하는 data가 있는지 알고 있다. 따라서 중간 단계를 거칠 필요없이 바로 원하는 node에 접근하여 data를 request한다.

즉 routing decision을 어느 단계에서 하냐에 따라서 방법이 달라지게 된다.

_Figure 6-7. Three different ways of routing a request to the right node._

![Screenshot 2024-03-03 at 10.49.01 PM.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-7.png)

routing decision을 하는 요소에서는 rebalancing이 발생할 때 partition에서 발생하는 data 이동에 맞춰 request routing을 알맞게 하는 것은 어려우며, system에 있는 node들의 consensus가 필요로 하다.

대부분의 system에서는 coordination service(Ex. ZooKeeper)를 통해 cluster 내 partition에 대한 metadata를 관리한다.
_Figure 6-8. Using ZooKeeper to keep track of assignment of partitions to nodes._

![Screenshot 2024-03-03 at 11.03.50 PM.png](../../assets/data_intensive_apps/6%20Partitioning/fig6-8.png)

각 node는 직접 ZooKeeper에 자신을 등록하면 ZooKeeper에서는 partition과 node간 mapping을 유지관리한다. 그러면 Client나 Routing tier에서는 ZooKeeper를 subsribing하며 원하는 data가 해당하는 partition과 mapped된 node를 찾아 request를 보내는 방식이다.
ZooKeeper에서는 node가 추가 또는 제거되거나 partition이 이동하게 된다면 subscriber에게 이를 알린다.

LinkedIn의 Espresso, HBase, Kafka 등이 ZooKeeper를 활용하며, MongoDB에서는 이와 유사한 architecture를 자체적으로 구현하여 사용한다.

Cassandra와 Riak의 경우 위 방법1 방법을 사용한다: 방법1에서 추가적으로 node들에서 *gossip protocol*을 통해 cluster state에 변화를 알 수 있다. 이는 3rd party tool인 ZooKeeper와 같은 것들에 의존성 없이 system이 돌아가게 하지만 훨씬 구현이 복잡해진다.

마지막으로 node 또는 load balancer에 연결하기 위한 ip의 경우 이들의 변화는 자주 발생하지 않기에 DNS를 통해 처리하여도 충분하다.

## **Parallel Query Execution**

OLAP와 같은 경우 analytic 목적으로 대량의 data 및 복잡한 query를 제공해야한다.
이를 위한 기능으로 MPP(_Massive Parallel Processing_)이 존재한다.

MPP query optimizer는 복잡한 query를 여러 execution 및 partition으로 구성된 단계 별로 나눈 후 node들에 대해 parallel하게 수행한다. 따라서 많은 partition들을 scanning해야하는 query의 경우에 큰 이점을 가진다.

# Summary

In this chapter we explored different ways of partitioning a large dataset into smaller subsets. Partitioning is necessary when you have so much data that storing and pro‐ cessing it on a single machine is no longer feasible.

The goal of partitioning is to spread the data and query load evenly across multiple machines, avoiding hot spots (nodes with disproportionately high load). This requires choosing a partitioning scheme that is appropriate to your data, and rebalancing the partitions when nodes are added to or removed from the cluster.

We discussed two main approaches to partitioning:

- _Key range partitioning_, where keys are sorted, and a partition owns all the keys from some minimum up to some maximum. Sorting has the advantage that efficient range queries are possible, but there is a risk of hot spots if the application often accesses keys that are close together in the sorted order.

In this approach, partitions are typically rebalanced dynamically by splitting the range into two subranges when a partition gets too big.

- _Hash partitioning_, where a hash function is applied to each key, and a partition owns a range of hashes. This method destroys the ordering of keys, making range queries inefficient, but may distribute load more evenly.

When partitioning by hash, it is common to create a fixed number of partitions in advance, to assign several partitions to each node, and to move entire partitions from one node to another when nodes are added or removed.
Dynamic partitioning can also be used.

Hybrid approaches are also possible, for example with a compound key: using one part of the key to identify the partition and another part for the sort order.

We also discussed the interaction between partitioning and secondary indexes. A secondary index also needs to be partitioned, and there are two methods:

- _Document-partitioned indexes_ (local indexes), where the secondary indexes are stored in the same partition as the primary key and value. This means that only a single partition needs to be updated on write, but a read of the secondary index requires a scatter/gather across all partitions.
- _Term-partitioned indexes_ (global indexes), where the secondary indexes are partitioned separately, using the indexed values. An entry in the secondary index may include records from all partitions of the primary key. When a document is written, several partitions of the secondary index need to be updated; however, a read can be served from a single partition.

Finally, we discussed techniques for routing queries to the appropriate partition, which range from simple partition-aware load balancing to sophisticated parallel query execution engines.

By design, every partition operates mostly independently—that’s what allows a partitioned database to scale to multiple machines. However, operations that need to write to several partitions can be difficult to reason about: for example, what happens if the write to one partition succeeds, but another fails? We will address that question in the following chapters.
