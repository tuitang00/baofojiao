# Redis



## redis数据结构与使用场景

### string

首先我们从string谈起。string表示的是一个可变的字节数组，我们初始化字符串的内容、可以拿到字符串的长度，可以获取string的子串，可以覆盖string的子串内容，可以追加子串。

Redis的字符串是动态字符串，是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配，如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

**初始化字符串**  需要提供「变量名称」和「变量的内容」

```ruby
> set ireader beijing.zhangyue.keji.gufen.youxian.gongsi
OK
```

**获取字符串的内容** 提供「变量名称」

```ruby
> get ireader
"beijing.zhangyue.keji.gufen.youxian.gongsi"
```

**获取字符串的长度** 提供「变量名称」

```ruby
> strlen ireader
(integer) 42
```

**获取子串** 提供「变量名称」以及开始和结束位置[start, end]

```ruby
> getrange ireader 28 34
"youxian"
```

**覆盖子串** 提供「变量名称」以及开始位置和目标子串

```ruby
> setrange ireader 28 wooxian
(integer) 42  # 返回长度
> get ireader
"beijing.zhangyue.keji.gufen.wooxian.gongsi"
```

**追加子串**

```ruby
> append ireader .hao
(integer) 46 # 返回长度
> get ireader
"beijing.zhangyue.keji.gufen.wooxian.gongsi.hao"
```

遗憾的是字符串没有提供字串插入方法和子串删除方法。

**计数器** 如果字符串的内容是一个整数，那么还可以将字符串当成计数器来使用。

```ruby
> set ireader 42
OK
> get ireader
"42"
> incrby ireader 100
(integer) 142
> get ireader
"142"
> decrby ireader 100
(integer) 42
> get ireader
"42"
> incr ireader  # 等价于incrby ireader 1
(integer) 43
> decr ireader  # 等价于decrby ireader 1
(integer) 42
```

计数器是有范围的，它不能超过Long.Max，不能低于Long.MIN

```ruby
> set ireader 9223372036854775807
OK
> incr ireader
(error) ERR increment or decrement would overflow
> set ireader -9223372036854775808
OK
> decr ireader
(error) ERR increment or decrement would overflow
```

**过期和删除** 字符串可以使用del指令进行主动删除，可以使用expire指令设置过期时间，到点会自动删除，这属于被动删除。可以使用ttl指令获取字符串的寿命。

```ruby
> expire ireader 60
(integer) 1  # 1表示设置成功，0表示变量ireader不存在
> ttl ireader
(integer) 50  # 还有50秒的寿命，返回-2表示变量不存在，-1表示没有设置过期时间
> del ireader
(integer) 1  # 删除成功返回1
> get ireader
(nil)  # 变量ireader没有了
```



### list 

Redis将列表数据结构命名为list而不是array，是因为列表的存储结构用的是链表而不是数组，而且链表还是双向链表。因为它是链表，所以随机定位性能较弱，首尾插入删除性能较优。如果list的列表长度很长，使用时我们一定要关注链表相关操作的时间复杂度。

**负下标** 链表元素的位置使用自然数`0,1,2,....n-1`表示，还可以使用负数`-1,-2,...-n`来表示，`-1`表示「倒数第一」，`-2`表示「倒数第二」，那么`-n`就表示第一个元素，对应的下标为`0`。

**队列／堆栈** 链表可以从表头和表尾追加和移除元素，结合使用rpush/rpop/lpush/lpop四条指令，可以将链表作为队列或堆栈使用，左向右向进行都可以

```ruby
# 右进左出
> rpush ireader go
(integer) 1
> rpush ireader java python
(integer) 3
> lpop ireader
"go"
> lpop ireader
"java"
> lpop ireader
"python"
# 左进右出
> lpush ireader go java python
(integer) 3
> rpop ireader
"go"
...
# 右进右出
> rpush ireader go java python
(integer) 3
> rpop ireader 
"python"
...
# 左进左出
> lpush ireader go java python
(integer) 3
> lpop ireader
"python"
...
```

在日常应用中，列表常用来作为异步队列来使用。

**长度** 使用llen指令获取链表长度

```ruby
> rpush ireader go java python
(integer) 3
> llen ireader
(integer) 3
```

**随机读** 可以使用lindex指令访问指定位置的元素，使用lrange指令来获取链表子元素列表，提供start和end下标参数

```ruby
> rpush ireader go java python
(integer) 3
> lindex ireader 1
"java"
> lrange ireader 0 2
1) "go"
2) "java"
3) "python"
> lrange ireader 0 -1  # -1表示倒数第一
1) "go"
2) "java"
3) "python"
```

使用lrange获取全部元素时，需要提供end_index，如果没有负下标，就需要首先通过llen指令获取长度，才可以得出end_index的值，有了负下标，使用-1代替end_index就可以达到相同的效果。

**修改元素** 使用lset指令在指定位置修改元素。

```ruby
> rpush ireader go java python
(integer) 3
> lset ireader 1 javascript
OK
> lrange ireader 0 -1
1) "go"
2) "javascript"
3) "python"
```

**插入元素** 使用linsert指令在列表的中间位置插入元素，有经验的程序员都知道在插入元素时，我们经常搞不清楚是在指定位置的前面插入还是后面插入，所以antirez在linsert指令里增加了方向参数before/after来显示指示前置和后置插入。不过让人意想不到的是linsert指令并不是通过指定位置来插入，而是通过指定具体的值。这是因为在分布式环境下，列表的元素总是频繁变动的，意味着上一时刻计算的元素下标在下一时刻可能就不是你所期望的下标了。

```ruby
> rpush ireader go java python
(integer) 3
> linsert ireader before java ruby
(integer) 4
> lrange ireader 0 -1
1) "go"
2) "ruby"
3) "java"
4) "python"
```

到目前位置，我还没有在实际应用中发现插入指定的应用场景。

**删除元素** 列表的删除操作也不是通过指定下标来确定元素的，你需要指定删除的最大个数以及元素的值

```ruby
> rpush ireader go java python
(integer) 3
> lrem ireader 1 java
(integer) 1
> lrange ireader 0 -1
1) "go"
2) "python"
```

**定长列表** 在实际应用场景中，我们有时候会遇到「定长列表」的需求。比如要以走马灯的形式实时显示中奖用户名列表，因为中奖用户实在太多，能显示的数量一般不超过100条，那么这里就会使用到定长列表。维持定长列表的指令是ltrim，需要提供两个参数start和end，表示需要保留列表的下标范围，范围之外的所有元素都将被移除。

```ruby
> rpush ireader go java python javascript ruby erlang rust cpp
(integer) 8
> ltrim ireader -3 -1
OK
> lrange ireader 0 -1
1) "erlang"
2) "rust"
3) "cpp"
```

如果指定参数的end对应的真实下标小于start，其效果等价于del指令，因为这样的参数表示需要需要保留列表元素的下标范围为空。





###  hash

哈希等价于Java语言的HashMap或者是Python语言的dict，在实现结构上它使用二维结构，第一维是数组，第二维是链表，hash的内容key和value存放在链表中，数组里存放的是链表的头指针。通过key查找元素时，先计算key的hashcode，然后用hashcode对数组的长度进行取模定位到链表的表头，再对链表进行遍历获取到相应的value值，链表的作用就是用来将产生了「hash碰撞」的元素串起来。Java语言开发者会感到非常熟悉，因为这样的结构和HashMap是没有区别的。哈希的第一维数组的长度也是2^n。



**增加元素** 可以使用hset一次增加一个键值对，也可以使用hmset一次增加多个键值对

```ruby
> hset ireader go fast
(integer) 1
> hmset ireader java fast python slow
OK
```

**获取元素** 可以通过hget定位具体key对应的value，可以通过hmget获取多个key对应的value，可以使用hgetall获取所有的键值对，可以使用hkeys和hvals分别获取所有的key列表和value列表。这些操作和Java语言的Map接口是类似的。

```ruby
> hmset ireader go fast java fast python slow
OK
> hget ireader go
"fast"
> hmget ireader go python
1) "fast"
2) "slow"
> hgetall ireader
1) "go"
2) "fast"
3) "java"
4) "fast"
5) "python"
6) "slow"
> hkeys ireader
1) "go"
2) "java"
3) "python"
> hvals ireader
1) "fast"
2) "fast"
3) "slow"
```

**删除元素** 可以使用hdel删除指定key，hdel支持同时删除多个key

```ruby
> hmset ireader go fast java fast python slow
OK
> hdel ireader go
(integer) 1
> hdel ireader java python
(integer) 2
```

**判断元素是否存在** 通常我们使用hget获得key对应的value是否为空就直到对应的元素是否存在了，不过如果value的字符串长度特别大，通过这种方式来判断元素存在与否就略显浪费，这时可以使用hexists指令。

```ruby
> hmset ireader go fast java fast python slow
OK
> hexists ireader go
(integer) 1
```

**计数器** hash结构还可以当成计数器来使用，对于内部的每一个key都可以作为独立的计数器。如果value值不是整数，调用hincrby指令会出错。

```bash
> hincrby ireader go 1
(integer) 1
> hincrby ireader python 4
(integer) 4
> hincrby ireader java 4
(integer) 4
> hgetall ireader
1) "go"
2) "1"
3) "python"
4) "4"
5) "java"
6) "4"
> hset ireader rust good
(integer) 1
> hincrby ireader rust 1
(error) ERR hash value is not an integer
```

**扩容** 当hash内部的元素比较拥挤时(hash碰撞比较频繁)，就需要进行扩容。扩容需要申请新的两倍大小的数组，然后将所有的键值对重新分配到新的数组下标对应的链表中(rehash)。如果hash结构很大，比如有上百万个键值对，那么一次完整rehash的过程就会耗时很长。这对于单线程的Redis里来说有点压力山大。所以Redis采用了渐进式rehash的方案。它会同时保留两个新旧hash结构，在后续的定时任务以及hash结构的读写指令中将旧结构的元素逐渐迁移到新的结构中。这样就可以避免因扩容导致的线程卡顿现象。

**缩容** Redis的hash结构不但有扩容还有缩容，从这一点出发，它要比Java的HashMap要厉害一些，Java的HashMap只有扩容。缩容的原理和扩容是一致的，只不过新的数组大小要比旧数组小一倍。



### set

Java程序员都知道HashSet的内部实现使用的是HashMap，只不过所有的value都指向同一个对象。Redis的set结构也是一样，它的内部也使用hash结构，所有的value都指向同一个内部值。

**增加元素** 可以一次增加多个元素

```ruby
> sadd ireader go java python
(integer) 3
```

**读取元素** 使用smembers列出所有元素，使用scard获取集合长度，使用srandmember获取随机count个元素，如果不提供count参数，默认为1

```ruby
> sadd ireader go java python
(integer) 3
> smembers ireader
1) "java"
2) "python"
3) "go"
> scard ireader
(integer) 3
> srandmember ireader
"java"
```

**删除元素** 使用srem删除一到多个元素，使用spop删除随机一个元素

```ruby
> sadd ireader go java python rust erlang
(integer) 5
> srem ireader go java
(integer) 2
> spop ireader
"erlang"
```

**判断元素是否存在** 使用sismember指令，只能接收单个元素

```ruby
> sadd ireader go java python rust erlang
(integer) 5
> sismember ireader rust
(integer) 1
> sismember ireader javascript
(integer) 0
```



### sortedset

zset底层实现使用了两个数据结构，第一个是hash，第二个是跳跃列表，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。跳跃列表的目的在于给元素value排序，根据score的范围获取元素列表。

**增加元素** 通过zadd指令可以增加一到多个value/score对，score放在前面

```ruby
> zadd ireader 4.0 python
(integer) 1
> zadd ireader 4.0 java 1.0 go
(integer) 2
```

**长度** 通过指令zcard可以得到zset的元素个数

```
> zcard ireader
(integer) 3
复制代码
```

**删除元素** 通过指令zrem可以删除zset中的元素，可以一次删除多个

```
> zrem ireader go python
(integer) 2

```

**计数器** 同hash结构一样，zset也可以作为计数器使用。

```
> zadd ireader 4.0 python 4.0 java 1.0 go
(integer) 3
> zincrby ireader 1.0 python
"5"

```

**获取排名和分数** 通过zscore指令获取指定元素的权重，通过zrank指令获取指定元素的正向排名，通过zrevrank指令获取指定元素的反向排名[倒数第一名]。正向是由小到大，负向是由大到小。

```
> zscore ireader python
"5"
> zrank ireader go  # 分数低的排名考前，rank值小
(integer) 0
> zrank ireader java
(integer) 1
> zrank ireader python
(integer) 2
> zrevrank ireader python
(integer) 0

```

**根据排名范围获取元素列表** 通过zrange指令指定排名范围参数获取对应的元素列表，携带withscores参数可以一并获取元素的权重。通过zrevrange指令按负向排名获取元素列表[倒数]。正向是由小到大，负向是由大到小。

```
> zrange ireader 0 -1  # 获取所有元素
1) "go"
2) "java"
3) "python"
> zrange ireader 0 -1 withscores
1) "go"
2) "1"
3) "java"
4) "4"
5) "python"
6) "5"
> zrevrange ireader 0 -1 withscores
1) "python"
2) "5"
3) "java"
4) "4"
5) "go"
6) "1"

```

**根据score范围获取列表** 通过zrangebyscore指令指定score范围获取对应的元素列表。通过zrevrangebyscore指令获取倒排元素列表。正向是由小到大，负向是由大到小。参数`-inf`表示负无穷，`+inf`表示正无穷。

```
> zrangebyscore ireader 0 5
1) "go"
2) "java"
3) "python"
> zrangebyscore ireader -inf +inf withscores
1) "go"
2) "1"
3) "java"
4) "4"
5) "python"
6) "5"
> zrevrangebyscore ireader +inf -inf withscores  # 注意正负反过来了
1) "python"
2) "5"
3) "java"
4) "4"
5) "go"
6) "1"

```

**根据范围移除元素列表** 可以通过排名范围，也可以通过score范围来一次性移除多个元素

```
> zremrangebyrank ireader 0 1
(integer) 2  # 删掉了2个元素
> zadd ireader 4.0 java 1.0 go
(integer) 2
> zremrangebyscore ireader -inf 4
(integer) 2
> zrange ireader 0 -1
1) "python"

```



## 高可用模型与特性



### Sentinel 

当主节点出现故障时，Redis Sentinel能自动完成故障发现和故障转移，并通知应用方，从而实现真正的高可用。

RedisSentine是一个分布式架构，其中包含若干个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和

其余Sentinel节点进行监控，当它发现节点不可达时，会对节点做下线标识。如果被标识的是“主节点”，它还会和

其他的Sentinel节点进行“协商”，当大多数Sentinel节点都认为主节点不可达时，它们会选举一个Sentinel节点来完

成自动故障转移的工作，同时会将这个变化实时通知给Redis应用方。整个过程是自动的，不需要人工干预，解决了Redis的高可用问题。

**Redis Sentinel包含了若干个Sentinel节点，这样做也带来了两个好处：**

1. 对节点的故障判断是由多个Sentinel节点共同完成，这样可以有效的防止误判。
2. Sentinel节点集合是由若干个Sentinel节点组成的，这样即使个别Sentinel节点不可用，整个Sentinel节点集合依然是健壮的。

**Redis Sentinel具有以下几个功能：**

1. 监控：Sentinel会定期检测Redis数据节点、其余Sentinel节点是否可到达

2. 通知：Sentinel会将故障转移的结果通知给应用方。

3. 主节点故障转移：实现从节点晋升为主节点并维护后续正确的主从关系。

4. 配置提供者：在RedisSentinel结构中，客户端在初始化的时候连接的是Sentinel节点集合，从中获取主节点信息。



### Cluster

Redis数据分区：RedisCluster采用虚拟槽分区，所有的键根据哈希函数映射到0-16383整数槽内，

计算公式：slot=CRC16(key) &16383。每一个节点负责维护一部分槽以及槽所映射的键值数据。

 

**Redis虚拟槽分区的特点：**

1. 解耦数据和节点之间的关系，简化了节点扩容和收缩的难度

2. 节点自身维护槽的映射关系，不需要客户端或者代理服务维护槽分区元数据

3. 支持节点、槽、键之间的映射查询，用于数据路由、在线伸缩等场景



**Redis Cluster 限制：**

1. Key批量操作支持有限。目前只支持同slot内的key执行批量操作（如mget,mset）。

2. Key事务操作支持有限。只支持多key在同一个节点上的事务操作，多个key分布在不同节点上时无法使用事务功能。

3. Key作为数据分区的最小粒度，因此不能将一个大的键值对象如hash，list等映射到不同节点。

4. 不支持多数据库空间，集群模式下只能使用db0空间。

5. 复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构。





## Redis 使用场景

**1、显示最新的项目列表**

 下面这个语句常用来显示最新项目，随着数据多了，查询毫无疑问会越来越慢。

```sql
SELECT * FROM foo WHERE … ORDER BY time DESC LIMIT 10
```

在Web应用中，“列出最新的回复”之类的查询非常普遍，这通常会带来可扩展性问题。这令人沮丧，因为项目本来就是按这个顺序被创建的，但要输出这个顺序却不得不进行排序操作。

类似的问题就可以用Redis来解决。比如说，我们的一个Web应用想要列出用户贴出的最新20条评论。在最新的评论边上我们有一个“显示全部”的链接，点击后就可以获得更多的评论。

我们假设数据库中的每条评论都有一个唯一的递增的ID字段。

我们可以使用[分页](https://link.jianshu.com?t=http://www.111cn.net/tags.php/%B7%D6%D2%B3/)来制作主页和评论页，使用Redis的模板，每次新评论发表时，我们会将它的ID添加到一个

Redis列表：

 `LPUSH latest.comments`

 我们将列表裁剪为指定长度，因此Redis只需要保存最新的5000条评论：

 `LTRIM latest.comments 0 5000`

 每次我们需要获取最新评论的项目范围时，我们调用一个函数来完成(使用伪代码)：

```sql
FUNCTION get_latest_comments(start, num_items):
id_list = redis.lrange(“latest.comments”,start,start+num_items – 1)
IF id_list.length < num_items
id_list = SQL_DB(“SELECT … ORDER BY time LIMIT …”)
END
RETURN id_list
END
```

这里我们做的很简单。在Redis中我们的最新ID使用了常驻缓存，这是一直更新的。但是我们做了限制不能超过5000个ID，因此我们的获取ID函数会一直询问Redis。只有在start/count参数超出了这个范围的时候，才需要去访问数据库。

 我们的系统不会像传统方式那样“刷新”缓存，Redis实例中的信息永远是一致的。SQL数据库(或是硬盘上的其他类型数据库)只是在用户需要获取“很远”的数据时才会被触发，而主页或第一个评论页是不会麻烦到硬盘上的数据库了。



 **2、删除与过滤**
 我们可以使用LREM来删除评论。如果删除操作非常少，另一个选择是直接跳过评论条目的入口，报告说该评论已经不存在。

 有些时候你想要给不同的列表附加上不同的过滤器。如果过滤器的数量受到限制，你可以简单的为每个不同的过滤器使用不同的Redis列表。毕竟每个列表只有5000条项目，但Redis却能够使用非常少的内存来处理几百万条项目。



 **3、排行榜相关**
 另一个很普遍的需求是各种数据库的数据并非存储在内存中，因此在按得分排序以及实时更新这些几乎每秒钟都需要更新的功能上数据库的性能不够理想。
 典型的比如那些在线游戏的排行榜，比如一个Facebook的游戏，根据得分你通常想要：

- 列出前100名高分选手
-  列出某用户当前的全球排名

 这些操作对于Redis来说小菜一碟，即使你有几百万个用户，每分钟都会有几百万个新的得分。
 模式是这样的，每次获得新得分时，我们用这样的代码：

 `ZADD leaderboard`

 你可能用userID来取代username，这取决于你是怎么设计的。

 得到前100名高分用户很简单：

 `ZREVRANGE leaderboard 0 99`。

 用户的全球排名也相似，只需要：

 `ZRANK leaderboard`。

 **4、按照用户投票和时间排序**

 排行榜的一种常见变体模式就像Reddit或Hacker News用的那样，新闻按照类似下面的公式根据得分来排序：
 `score = points / time^alpha`

 因此用户的投票会相应的把新闻挖出来，但时间会按照一定的指数将新闻埋下去。下面是我们的模式，当然算法由你决定。

 模式是这样的，开始时先观察那些可能是最新的项目，例如首页上的1000条新闻都是候选者，因此我们先忽视掉其他的，这实现起来很简单。

 每次新的新闻贴上来后，我们将ID添加到列表中，使用

 `LPUSH + LTRIM`，确保只取出最新的1000条项目。

 有一项后台任务获取这个列表，并且持续的计算这1000条新闻中每条新闻的最终得分。计算结果由ZADD命令按照新的顺序填充生成列表，老新闻则被清除。这里的关键思路是排序工作是由后台任务来完成的。

 **5、处理过期项目**

 另一种常用的项目排序是按照时间排序。我们使用unix时间作为得分即可。

 模式如下：

-  每次有新项目添加到我们的非Redis数据库时，我们把它加入到排序集合中。这时我们用的是时间属性，current_time和time_to_live。
-  另一项后台任务使用ZRANGE…SCORES查询排序集合，取出最新的10个项目。如果发现unix时间已经过期，则在数据库中删除条目。

 **6、计数**

 Redis是一个很好的计数器，这要感谢INCRBY和其他相似命令。

 我相信你曾许多次想要给数据库加上新的计数器，用来获取统计或显示新信息，但是最后却由于写入敏感而不得不放弃它们。

 好了，现在使用Redis就不需要再担心了。有了原子递增(atomic increment)，你可以放心的加上各种计数，用GETSET重置，或者是让它们过期。

 例如这样操作：

```
INCR user: EXPIRE
user: 60
```



你可以计算出最近用户在页面间停顿不超过60秒的页面浏览量，当计数达到比如20时，就可以显示出某些条幅提示，或是其它你想显示的东西。

 **7、特定时间内的特定项目**

 另一项对于其他数据库很难，但Redis做起来却轻而易举的事就是统计在某段特点时间里有多少特定用户访问了某个特定资源。比如我想要知道某些特定的注册用户或IP地址，他们到底有多少访问了某篇文章。
 每次我获得一次新的页面浏览时我只需要这样做：

 `SADD page:day1:`

 当然你可能想用unix时间替换day1，比如time()-(time()%3600*24)等等。

 想知道特定用户的数量吗?只需要使`用SCARD page:day1:`。

 需要测试某个特定用户是否访问了这个页面?SISMEMBER page:day1: 。

 **8、实时分析正在发生的情况，用于数据统计与防止垃圾邮件等**

 我们只做了几个例子，但如果你研究Redis的命令集，并且组合一下，就能获得大量的实时分析方法，有效而且非常省力。使用Redis原语命令，更容易实施垃圾邮件过滤系统或其他实时跟踪系统。

 **9、Pub/Sub**
 Redis的Pub/Sub非常非常简单，运行稳定并且快速。支持模式匹配，能够实时订阅与取消频道。

 **10、队列**
你应该已经注意到像list push和list pop这样的Redis命令能够很方便的执行队列操作了，但能做的可不止这些：比如Redis还有list pop的变体命令，能够在列表为空时阻塞队列。

现代的互联网应用大量地使用了消息队列(Messaging)。消息队列不仅被用于系统内部组件之间的通信，同时也被用于系统跟其它服务之间的交互。消息队列的使用可以增加系统的可扩展性、灵活性和用户体验。非基于消息队列的系统，其运行速度取决于系统中最慢的组件的速度(注：短板效应)。而基于消息队列可以将系统中各组件解除耦合，这样系统就不再受最慢组件的束缚，各组件可以异步运行从而得以更快的速度完成各自的工作。

此外，当服务器处在高并发操作的时候，比如频繁地写入日志文件。可以利用消息队列实现异步处理。从而实现高性能的并发操作。

 **11、缓存**

 Redis的缓存部分值得写一篇新文章，我这里只是简单的说一下。Redis能够替代memcached，让你的缓存从只能存储数据变得能够更新数据，因此你不再需要每次都重新生成数据了



## Redis 持久化策略

redis提供了两种持久化的方式，分别是RDB（Redis DataBase）和AOF（Apend Only File）。

### RDB方式

RDB方式是一种快照式的持久化方法，将某一时刻的数据持久化到磁盘中。

 - redis在进行数据持久化的过程中，会先将数据写入到一个临时文件中，待持久化过程都结束了，才会用这个临时文件替换上次持久化好的文件。正是这种特性，让我们可以随时来进行备份，因为快照文件总是完整可用的。
 - 对于RDB方式，redis会单独创建（fork）一个子进程来进行持久化，而主进程是不会进行任何IO操作的，这样就确保了redis极高的性能。
 - 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

### AOF方式

AOF方式是将执行过的写指令记录下来，在数据恢复时按照丛前到后的顺序再将指令执行一遍。

 - AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大.默认的AOF持久化策略是每秒钟fsync一次（fsync是指把缓存中的写指令记录到磁盘中），因为在这种情况下，redis仍然可以保持很好的处理性能，即使redis故障，也只会丢失最近1秒钟的数据。
 - 如果在追加日志时，恰好遇到磁盘空间满、inode满或断电等情况导致日志写入不完整，也没有关系，redis提供了redis-check-aof工具，可以用来进行日志修复。
 - 因为采用了追加方式，如果不做任何处理的话，AOF文件会变得越来越大，为此，redis提供了AOF文件重写（rewrite）机制，即当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，假如我们调用了100次INCR指令，在AOF文件中就要存储100条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET指令，这就是重写机制的原理。
 - 在进行AOF重写时，仍然是采用先写临时文件，全部完成后再替换的流程，所以断电、磁盘满等问题都不会影响AOF文件的可用性。



# 优缺点

## RDB方式

- 优点：

 1. RDB是一个单一的紧凑文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.
 2. RDB是一个紧凑的单一文件,方便传送，适用于灾难恢复.
 3. RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.
 4. 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.

- 缺点：

 1. Redis意外宕机,可能会丢失几分钟的数据（取决于配置的save时间点）。RDB方式需要保存珍整个数据集，是一个比较繁重的工作，通常需要设置5分钟或者更久做一次完整的保存。
 2. RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续更久。

## AOF方式

- **优点**

 1. 使用AOF 会让Redis数据更加耐久: 你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.
 2. AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.
 3. Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
 4. AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析也很轻松。 导出AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

- **缺点**

 1. 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
 2. 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间。



# 缓存穿透、雪崩、击穿



## 缓存穿透

缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。在流量大时，可能DB就挂掉了，要是有人利用不存在的key频繁攻击我们的应用，这就是漏洞。



### 解决方案

有很多种方法可以有效地解决缓存穿透问题，最常见的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。另外也有一个更为简单粗暴的方法（我们采用的就是这种），如果一个查询返回的数据为空（不管是数 据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。



## 缓存雪崩

缓存雪崩是指在我们设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到DB，DB瞬时压力过重雪崩。



### 解决方案

缓存失效时的雪崩效应对底层系统的冲击非常可怕。大多数系统设计者考虑用加锁或者队列的方式保证缓存的单线 程（进程）写，从而避免失效时大量的并发请求落到底层存储系统上。这里分享一个简单方案就时讲缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。



## 缓存击穿

对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题，这个和缓存雪崩的区别在于这里针对某一key缓存，前者则是很多key。

缓存在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。



### 解决方案

- 使用互斥锁(mutex key)

  业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。

- "提前"使用互斥锁(mutex key)

  在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。

- 这里的“永远不过期”包含两层意思：

  (1) 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期。

  (2) 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期

- 采用netflix的hystrix，可以做资源的隔离保护主线程池，如果把这个应用到缓存的构建也未尝不可。