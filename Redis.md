## Redis

1. C/S模式，server是单线程服务器，基于EVENT-LOOP处理client请求。因此不必考虑线程安全问题，减少线程切换损耗。
2. 集群：客户端请求通过负载均衡算法（一致性哈希）分散，扩大了缓存容量，提升了吞吐量。
3. 主从复制Master-Slave模式（1-N的关系），防止某台redis挂掉，或者某个redis成为热点。由此实现了数据高可用（防止挂掉）+高查询效率（master/slave读写分离）；
4. master/slave chains：N个slave导致master备份压力加大；通过分封，使得master只接管两个slave；
5. 其他的NoSQL：MemCache+Cassadra+MongoDB

### 一、基本数据类型及其命令

1.通配符 ？ *  []  \x

2.set/mset get/mget keys del exists increby decr decrby incrbyfloat  strlen setbit getbit bitcount

3.字符串类型：二进制流512MB；

**4.散列类型：**

> 关系型数据库中每一列都是一个属性，每一行都对应一条记录，这就表示每个对象都必须要有这么多的属性，在实际中看来，有些对象可能并没有必要。Redis的散列描述了某个对象的存储结构，但是并不要求每个键都按照这一个pattern来存储。

HSET/HGET命令分别用来给某个字段赋值或读值。hset不区分插入或者更新。

HMSET/HMGET用来多个字段操作。

HGETALL获取所有字段的信息。

HEXISTS判断某个字段是否存在。

HSETNX key field value如果存在不执行，如果不存在新插入。

HINCRBY car price 1000

HDEL car [name price]

HKEYS car只获取字段名

HVALS car只获取字段值

HLEN car  只获取字段数量

**5.列表类型**：存储一个有序的字符串列表，底层使用双向链表实现。列表类型可以实现关系型数据库中难以应付的场景：只获取最新的N条数据。

同时它也适合记录日志，可以保证加入新日志的时候速度不会受到已有日志的影响。

列表与散列类型最多能容纳2^23-1的字段/元素。

命令：

LPUSH/RPUSH/LPOP/RPOP   key value

LLEN key返回列表中元素的个数；

LRANGE key start stop返回闭区间的所有元素，从0开始

支持负索引，-1表示右边的第一个元素。stop=-1；

当stop大于实际的索引范围时，只会返回到最右边的元素。

LREM key count value删除列表中指定值。

**注意**LREM money 0 1删除所有值为1的元素，

​		LREM money 1 1删除前1个值为1的元素；

​		LREM money -1 1删除倒数前1个值为1的元素；

LINDEX money 2获取指定索引为2 的值

LSET key index value 将index处的值设置为value

LTRIM key start end删除索引为start到end闭区间之外的所欲元素；

LINSERT key BEFORE/AFTER value newvalue查找value之后插入在其前方或者后方插入newvalue。

RPOPLPUSH source destination把从source中RPOP出来的数LPUSH到destination中去。

**6.集合类型**

集合类型在redis内部是使用值为空的散列表实现的，常用来给一类数据打标签。

命令：

SADD key member向集合中增加一个不存在的元素，如果已经存在就忽略它。

SREM key member删除几个集合中的元素并返回成功删除的个数。

SMEMBERS key获取集合中的所有元素

SISMENMBER key member判断该member是不是在集合中。

集合间的运算：

SDIFF key...返回key之间的差集，注意被减数和减数；

SINTER key...返回key之间的交集；

SUNION key...返回key之间的并集；

SCARD key获取集合key中的元素个数；

SDIFF/SINTER/SUNION + STORE destinaion key...获取集合间的运算后的结果另存为destination。

SRANDMEMBER：元素所在的桶中元素越小其被选中的可能性越大。

SPOP key 在集合key中随机选择一个元素弹出。

7.有序集合类型

场景：希望访客看到热门的文章

ZADD key [scoreofmember member]...

如果member已经存在了那么新的score会替换原来的score。

ZRANGE key start stop 按照元素分数从小到大的顺序返回索引从start到stop闭区间之间的所有元素

ZCARD key获得有序集合中的元素数量

ZCOUNT key min max返回闭区间元素的个数

ZREM key member [member …]删除一个或者多个元素

### 二、事务transaction

1.事务与命令一样都是redis的最小执行单位，一个事务中的命令要不全执行，要不就全不执行。

2.MULTI-EXEC错误处理：

- 语法错误：直接返回错误，一句也不执行；
- 运行错误：某个错误不会中止后续语句；

redis不支持事务回滚。

3.WATCH事务：保证函数或者流程在结束之前不被别的客户端修改。其逻辑：监控一个或者多个key，如果一旦有一个被修改，那么之后的事务不会被执行。该监控一直持续到EXEC命令或者执行了UNWATCH命令。

### 三、数据的生存时间

关系数据库中一般需要一个额外的字段记录到期时间，然后定期检测过期数据再进行删除。redis可以使用expire命令设置一个key的生存时间，到时间后redis会自动删除它。

set foo 1

expire foo 100

TTL foo 

persist foo将foo恢复成永久的。

使用set/getset为foo重新赋值会清除生存时间。其他只改变键值的命令如HSET，ZSET不会影响key的生存时间。

expire单位是秒，如果使用毫秒应当使用Pexpire命令。

**注意**：如果一个key在watch期间被expire了，watch并不会认为这是一种非法更改。

实现访问频率限制的两个途径：

- 限制每个用户一段时间内的最大访问量N，每访问一次自增INCR键值，并且为每一个连接设定一个expire即可。但仍然存在expire内访问数量激增到N的情况。

- 使用长度为n的列表记录访客最近n次访问的时间，一旦超过n个就看第一个记录是否已经expire，如果没有expire就等着，否则就更新。

- 实现缓存：如果大量使用缓存key并且TTL过长可能导致redis内存占满，而如果TTL过小则导致缓存命中率过低，内存空余很多不用白不用。因此需要按照一定的规则淘汰不需要的缓存键，具体方式为修改配置文件maxmemory参数。LRU：least recently used。

  支持的淘汰键原则：

  > 有TTL的key，使用LRU删除一个键。
  >
  > 对TTL的key，任意删除一个键。
  >
  > 使用LRU任意删除一个键。
  >
  > 随机删除一个键。
  >
  > 删除TTL最近的一个键。
  >
  > 不删除键只返回错误。

  实际上redis并不会准确地找到数据库里最久没有被使用的键，而是随机选取三个键并删除其中最久没有被使用的键。

### 四、排序

1.除了使用有序集合以外，还可以借助redis的sort命令在解决数据排序的问题，sort命令可以对列表类型+集合类型+有序集合类型进行排序。其中对有序集合进行排序时，sort会忽略元素的分数，只针对元素自身的值进行排序。

2.sort除了默认的排序方法（数值从小到大排列）之外，还可以通过alpha命令，这样sort就可以按照字母字典的顺序进行排序。

sort key alpha/desc（降序）/limit（指定返回的结果数）。

/BY可以根据key的字段进行排序。

/GET参数与BY参数一样，支持字符串类型和散列类型的键，可以实现在排序后直接返回排序后的某些字段。

如果需要保存排序结果可以使用STORE参数。

sort命令的时间复杂度是O(n+mlogm)，其中表示列表或者集合中元素的个数，m表示要返回的元素个数。

### 五、消息通知

1.当页面需要进行耗时较长的操作时会阻塞页面的渲染，为了避免用户等待太久，应该使用多个线程或者多个进程来完成这个任务。

2.与任务队列进行交互的实体有两类分别是生产者和消费者，任务队列使用的好处：松耦合+降负载；

3.redis**任务队列**的实现只需要生产者执行LPUSH，而让消费者不断RPOP即可。实际上使用的是BRPOP：

BRPOP（BLPOP）与RPOP命令类似，唯一的区别是当列表中没有元素时BRPOP会一直处于阻塞状态，直到有新元素加入。

BRPOP接受两个命令分别是key和超时时间，如果设置了超市时间那么再超时后仍然没有元素的话会返回nil；

4.任务队列可以有多个key，他们之间根据前后存在优先级。

5.发布/订阅模式中包含两种角色分别是发布者和订阅者，订阅者可以订阅若干个频道，而发布者可以向指定的频道发送信息，所有订阅这个频道的订阅者都可以收到这个信息。

对应的命令分别是subscribe和publish

subscribe channel<——>publish channel information

### 六、管道

1.客户端可以通过管道一次性发送多条命令并在执行完后一次性将结果返回，通过减少客户端与redis通信的次数来减少往返累计时延来降低时间消耗。

### 七、节省空间

1.精简键名和键值，非常直观但是要把握好尺度。

2.内部编码优化：redis为每种数据类型都提供了两种内部编码方式，比如当散列的元素很少的时候，即便使用O(n)的数据结构也并不会有很高的延迟。

3.其他：安装与脚本

### 八、管理

1.持久化：redis支持两种方式的持久化，分别是RDB方式和AOF方式。可以单独使用或者结合使用。

2.RDB，redis的默认持久化方式，一定时间内改动一定个数的键时触发快照，并存储在磁盘上。快照的过程如下：

redis使用fork创建子进程+父进程继续处理客户端的命令子进程则将内存中的数据存入磁盘的临时文件+子进程写完之后使用临时文件替换就的RDB文件；快照时使用copy-on-write策略。

除了自动快照以外还可以手动发送save或者bgsave命令执行快照，前者是阻塞其他命令进行快照，而后者是子进程进行快照。

通过组合设置自动快照的方式来将数据的损失控制在可接受范围内，但是RDB总是要损失快照后的数据更新，如果数据重要到无法承受任何损失，可以使用AOF方式进行持久化。

3.AOF方式（append only file）开启AOF持久化后每执行一条会更改redis中数据的命令，都会将其写入硬盘中的AOF文件；由于AOF是纯文本文件，有时候前面的命令会被后边的命令覆盖，redis会自动保留有用的记录。

在更改数据库中的数据时，并没有真正写入硬盘，这就需要redis在写入AOF后主动要求系统将缓存内容同步到硬盘中，可以通过appendfsync参数设置同步的时机。默认是每秒钟执行一次同步。

4.复制：持久化无法避免单点故障带来的问题，所以数据库复制副本以部署到不同的服务器上，因此，redis提供了复制功能自动实现同步的过程。同步后的数据库分为两类，分别是master和slave。

5.复制功能非常简单，只需要在从数据库启动时让其监听master的IP和端口号即可。比如master的ip:port=0.0.0.0:0000;则slave可以通过命令：

> redis-server --port 1111 --slaveof 0.0.0.0 0000

6.复制的原理：slave启动以后，向master发送sync命令，master收到后开始在后台保存快照（进行RDB），当快照完成以后，master将快照文件以及所有的缓存命令发送给slave，slave收到后会载入快照并执行收到的缓存中的命令，当master和slave断开重连以后，会重新执行上述过程，不支持断点续传。需要注意的是slave在同步的过程中并不会阻塞，而是会用slave之前同步的数据对命令进行响应。或者salve可以设置在同步完成前拒绝其他所有命令。

7.读写分离：当单机redis无法应付大量的读请求时，可以通过复制简历slave，master只进行write，slave负责read。

为了在master-slave模式提高性能，可以在从数据库中启用持久化，在主数据库中禁用持久化。当master崩溃，可以将slave升级成mater继续服务，并将原来的mater变成新master的slave。

8.安全

redis的安全设计是建立在“运行在可信环境”这一前提下的。此外可以通过配置文件为redis设置一个密码；还可以在配置文件中将命令重命名，保证只有自己的应用可以使用这个命令。



