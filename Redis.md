## Redis学习笔记记录

### 来源：

#### 公众号：

1. **Redis的各项功能解决了哪些问题？**

   > ​    Redis官方解释：redis是一个基于BSD开源的项目，是一个吧结构化数据放在内存中的一个存储系统，你可以把它作为数据库，缓存和消息中间件来使用。
   >
   > ​    同时支持String、slists、hashes、sets、sorted sets、bitmaps、hyperloglogs和geospatial indexes等数据类型。
   >
   > ​    它还内建了复制，lua脚本，LRU，事务等功能，通过redis sentinel实现高可用，通过redis cluster实现了自动分片。以及事务，发布/订阅，自动故障转移等等。
   >
   > - 最初页面刷新频繁，每次API访问都需要重新请求，然后后台执行sql查询返回结果。·
   > - 用于页面缓存，避免频繁刷新，弊端：可能获取到的数据是旧数据
   > - 单台服务器容易宕机出现缓存雪崩，瞬间同时访问服务器造成卡顿或崩溃
   > - 从而产生哨兵模式以及主从模式、集群搭建

2. **一文看透 Redis 分布式锁进化史（解读 + 缺陷分析）**

   > 不论是基于SETNX版本的Redis单实例分布式锁，还是Redlock分布式锁，都是为了保证下特性
   >
   > 1. 安全性：在同一时间不允许多个Client同时持有锁
   > 2. 活性
   >    死锁：锁最终应该能够被释放，即使Client端crash或者出现网络分区（通常基于超时机制）
   >    容错性：只要超过半数Redis节点可用，锁都能被正确获取和释放
   >
   > 所以在开发或者使用分布式锁的过程中要保证安全性和活性，避免出现不可预测的结果。
   >
   > 另外每个版本的分布式锁都存在一些问题，在锁的使用上要针对锁的实用场景选择合适的锁，通常情况下锁的使用场景包括：
   >
   > Efficiency(效率)：只需要一个Client来完成操作，不需要重复执行，这是一个对宽松的分布式锁，只需要保证锁的活性即可；
   >
   > Correctness(正确性)：多个Client保证严格的互斥性，不允许出现同时持有锁或者对同时操作同一资源，这种场景下需要在锁的选择和使用上更加严格，同时在业务代码上尽量做到幂等
   >
   > ***需要了解：redis分布式锁***

3. **Redis 备份、容灾及高可用实战**

   > Redis是一个高性能的key-value非关系型数据库，由于其具有高性能的特性，支持高可用、持久化、多种数据结构、集群等，使其脱颖而出，成为常用的非关系型数据库。
   >
   > **会话缓存（Session Cache）**
   > Redis缓存会话有非常好的优势，因为Redis提供持久化，在需要长时间保持会话的应用场景中，如购物车场景这样的场景中能提供很好的长会话支持，能给用户提供很好的购物体验。
   >
   > **全页缓存**
   > 在WordPress中，Pantheon提供了一个不错的插件wp-redis，这个插件能以最快的速度加载你曾经浏览过的页面。
   >
   > **队列**
   > Reids提供list和set操作，这使得Redis能作为一个很好的消息队列平台来使用。
   >
   > 我们常通过Reids的队列功能做购买限制。比如到节假日或者推广期间，进行一些活动，对用户购买行为进行限制，限制今天只能购买几次商品或者一段时间内只能购买一次。也比较适合适用。
   >
   > **排名**
   >
   > Redis在内存中对数字进行递增或递减的操作实现得非常好。所以我们在很多排名的场景中会应用Redis来进行，比如小说网站对小说进行排名，根据排名，将排名靠前的小说推荐给用户。
   >
   > **发布/订阅**
   > Redis提供发布和订阅功能，发布和订阅的场景很多，比如我们可以基于发布和订阅的脚本触发器，实现用Redis的发布和订阅功能建立起来的聊天系统。
   >
   > ***需要了解：redis主从复制 哨兵模式***

4. **9个提升逼格的redis命令**

   > # **keys**
   >
   > 我把这个命令放在第一位，是因为笔者曾经做过的项目，以及一些朋友的项目，都因为使用`keys`这个命令，导致出现性能毛刺。这个命令的时间复杂度是O(N)，而且redis又是单线程执行，在执行keys时即使是时间复杂度只有O(1)例如SET或者GET这种简单命令也会堵塞，从而导致这个时间点性能抖动，甚至可能出现timeout。 
   >
   > > **强烈建议生产环境屏蔽keys命令**（后面会介绍如何屏蔽）。
   >
   > # scan
   >
   > 既然keys命令不允许使用，那么有什么代替方案呢？有！那就是`scan`命令。如果把keys命令比作类似`select * from users where username like '%afei%'`这种SQL，那么scan应该是`select * from users where id>? limit 10`这种命令。
   >
   > 官方文档用法如下：
   >
   > ```
   > SCAN cursor [MATCH pattern] [COUNT count]
   > ```
   >
   > 初始执行scan命令例如`scan 0`。SCAN命令是一个基于游标的迭代器。这意味着命令每次被调用都需要使用上一次这个调用返回的游标作为该次调用的游标参数，以此来延续之前的迭代过程。当SCAN命令的游标参数被设置为0时，服务器将开始一次新的迭代，而**当redis服务器向用户返回值为0的游标时，表示迭代已结束**，这是唯一迭代结束的判定方式，而不能通过返回结果集是否为空判断迭代结束。
   >
   > 使用方式：
   >
   > ```
   > 127.0.0.1:6380> scan 0
   > 1) "22"
   > 2)  1) "23"
   >     2) "20"
   >     3) "14"
   >     4) "2"
   >     5) "19"
   >     6) "9"
   >     7) "3"
   >     8) "21"
   >     9) "12"
   >    10) "25"
   >    11) "7"
   > ```
   >
   > 返回结果分为两个部分：第一部分即1)就是下一次迭代游标，第二部分即2)就是本次迭代结果集。
   >
   > # slowlog
   >
   > 上面提到不能使用keys命令，如果就有开发这么做了呢，我们如何得知？与其他任意存储系统例如mysql，mongodb可以查看慢日志一样，redis也可以，即通过命令`slowlog`。用法如下：
   >
   > ```
   > SLOWLOG subcommand [argument]
   > ```
   >
   > subcommand主要有：
   >
   > - **get**，用法：slowlog get [argument]，获取argument参数指定数量的慢日志。
   > - **len**，用法：slowlog len，总慢日志数量。
   > - **reset**，用法：slowlog reset，清空慢日志。
   >
   > 执行结果如下：
   >
   > ```
   > 127.0.0.1:6380> slowlog get 5
   > 1) 1) (integer) 2
   >    2) (integer) 1532656201
   >    3) (integer) 2033
   >    4) 1) "flushddbb"
   > 2) 1) (integer) 1  ----  慢日志编码，一般不用care
   >    2) (integer) 1532646897  ----  导致慢日志的命令执行的时间点，如果api有timeout，可以通过对比这个时间，判断可能是慢日志命令执行导致的
   >    3) (integer) 26424  ----  导致慢日志执行的redis命令，通过4)可知，执行config rewrite导致慢日志，总耗时26ms+
   >    4) 1) "config"
   >       2) "rewrite"
   > ```
   >
   > > 命令耗时超过多少才会保存到slowlog中，可以通过命令`config set slowlog-log-slower-than 2000`配置并且不需要重启redis。注意：单位是微妙，2000微妙即2毫秒。
   >
   > # rename-command
   >
   > 为了防止把问题带到生产环境，我们可以通过配置文件重命名一些危险命令，例如`keys`等一些高危命令。操作非常简单，只需要在conf配置文件增加如下所示配置即可：
   >
   > ```
   > rename-command flushdb flushddbb
   > rename-command flushall flushallall
   > rename-command keys keysys
   > ```
   >
   > # bigkeys
   >
   > 随着项目越做越大，缓存使用越来越不规范。我们如何检查生产环境上一些有问题的数据。`bigkeys`就派上用场了，用法如下：
   >
   > ```
   > redis-cli -p 6380 --bigkeys
   > ```
   >
   > 执行结果如下：
   >
   > ```
   > ... ...
   > -------- summary -------
   > 
   > Sampled 526 keys in the keyspace!
   > Total key length in bytes is 1524 (avg len 2.90)
   > 
   > Biggest string found 'test' has 10005 bytes
   > Biggest   list found 'commentlist' has 13 items
   > 
   > 524 strings with 15181 bytes (99.62% of keys, avg size 28.97)
   > 2 lists with 19 items (00.38% of keys, avg size 9.50)
   > 0 sets with 0 members (00.00% of keys, avg size 0.00)
   > 0 hashs with 0 fields (00.00% of keys, avg size 0.00)
   > 0 zsets with 0 members (00.00% of keys, avg size 0.00)
   > ```
   >
   > 最后5行可知，没有set,hash,zset几种数据结构的数据。string类型有524个，list类型有两个；通过`Biggest ... ...`可知，最大string结构的key是`test`，最大list结构的key是`commentlist`。
   >
   > 需要注意的是，这个**bigkeys得到的最大，不一定是最大**。说明原因前，首先说明`bigkeys`的原理，非常简单，通过scan命令遍历，各种不同数据结构的key，分别通过不同的命令得到最大的key：
   >
   > - 如果是string结构，通过`strlen`判断；
   > - 如果是list结构，通过`llen`判断；
   > - 如果是hash结构，通过`hlen`判断；
   > - 如果是set结构，通过`scard`判断；
   > - 如果是sorted set结构，通过`zcard`判断。
   >
   > > 正因为这样的判断方式，虽然string结构肯定可以正确的筛选出最占用缓存，也可以说最大的key。但是list不一定，例如，现在有两个list类型的key，分别是：numberlist--[0,1,2]，stringlist--["123456789123456789"]，由于通过llen判断，所以numberlist要大于stringlist。而事实上stringlist更占用内存。其他三种数据结构hash，set，sorted set都会存在这个问题。使用bigkeys一定要注意这一点。
   >
   > # monitor
   >
   > 假设生产环境没有屏蔽keys等一些高危命令，并且slowlog中还不断有新的keys导致慢日志。那我们如何揪出这些命令是由谁执行的呢？这就是`monitor`的用处，用法如下：
   >
   > ```
   > redis-cli -p 6380 monitor
   > ```
   >
   > 如果当前redis环境OPS比较高，那么建议结合linux管道命令优化，只输出keys命令的执行情况：
   >
   > ```
   > [afei@redis ~]# redis-cli -p 6380 monitor | grep keys 
   > 1532645266.656525 [0 10.0.0.1:43544] "keyss" "*"
   > 1532645287.257657 [0 10.0.0.1:43544] "keyss" "44*"
   > ```
   >
   > 执行结果中很清楚的看到keys命名执行来源。通过输出的IP和端口信息，就能在目标服务器上找到执行这条命令的进程，揪出元凶，勒令整改。
   >
   > # info
   >
   > 如果说哪个命令能最全面反映当前redis运行情况，那么非info莫属。用法如下：
   >
   > ```
   > INFO [section]
   > ```
   >
   > section可选值有：
   >
   > - **Server**：运行的redis实例一些信息，包括：redis版本，操作系统信息，端口，GCC版本，配置文件路径等；
   > - **Clients**：redis客户端信息，包括：已连接客户端数量，阻塞客户端数量等；
   > - **Memory**：使用内存，峰值内存，内存碎片率，内存分配方式。这几个参数都非常重要；
   > - **Persistence**：AOF和RDB持久化信息；
   > - **Stats**：一些统计信息，最重要三个参数：OPS(`instantaneous_ops_per_sec`)，`keyspace_hits`和`keyspace_misses`两个参数反应缓存命中率；
   > - **Replication**：redis集群信息；
   > - **CPU**：CPU相关信息；
   > - **Keyspace**：redis中各个DB里key的信息；
   >
   > # config
   >
   > config是一个非常有价值的命令，主要体现在对redis的运维。因为生产环境一般是不允许随意重启的，不能因为需要调优一些参数就修改conf配置文件并重启。redis作者早就想到了这一点，通过config命令能热修改一些配置，不需要重启redis实例，可以通过如下命令查看哪些参数可以热修改：
   >
   > ```
   > config get *
   > ```
   >
   > 热修改就比较容易了，执行如下命令即可：
   >
   > ```
   > config set 
   > ```
   >
   > 例如：`config set slowlog-max-len 100`，`config set maxclients 1024`
   >
   > 这样修改的话，如果以后由于某些原因redis实例故障需要重启，那通过config热修改的参数就会被配置文件中的参数覆盖，所以我们需要通过一个命令将config热修改的参数刷到redis配置文件中持久化，通过执行如下命令即可：
   >
   > ```
   > config rewrite
   > ```
   >
   > 执行该命令后，我们能在config文件中看到类似这种信息：
   >
   > ```
   > # 如果conf中本来就有这个参数，通过执行config set，那么redis直接原地修改配置文件
   > maxclients 1024
   > # 如果conf中没有这个参数，通过执行config set，那么redis会追加在Generated by CONFIG REWRITE字样后面
   > # Generated by CONFIG REWRITE
   > save 600 60
   > slowlog-max-len 100
   > ```
   >
   > # set
   >
   > set命令也能提升逼格？是的，我本不打算写这个命令，但是我见过太多人没有完全掌握这个命令，官方文档介绍的用法如下：
   >
   > ```
   > SET key value [EX seconds] [PX milliseconds] [NX|XX]
   > ```
   >
   > 你可能用的比较多的就是`set key value`，或者`SETEX key seconds value`，所以很多同学用redis实现分布式锁分为两步：首先执行`SETNX key value`，然后执行`EXPIRE key seconds`。很明显，这种实现有很严重的问题，因为两步执行不具备原子性，如果执行第一个命令后出现某些未知异常导致无法执行`EXPIRE key seconds`，那么分布式锁就会一直无法得到释放。
   >
   > 通过`SET`命令实现分布式锁的正式姿势应该是`SET key value EX seconds NX`（EX和PX任选，取决于对过期时间精度要求）。另外，value也有要求，最好是一个类似UUID这种具备唯一性的字符串。当然如果问你redis是否还有其他实现分布式锁的方案。你能说出redlock，那对方一定眼前一亮，心里对你竖起大拇指，但嘴上不会说。
   >
   > 需要了解：[redis官方文档](http://redis.cn/documentation.html)

5. **单线程的Redis为什么这么快？**

   > 二八定律、热数据和冷数据、缓存雪崩、缓存穿透、缓存预热、缓存更新、缓存降级
   >
   > 持久化
   >
   > [微信](https://mp.weixin.qq.com/s?__biz=MzAxNjM2MTk0Ng==&mid=2247484943&idx=1&sn=982607670e18d2dc5f25ab1357909e92&chksm=9bf4b6baac833fac8814fba5731b65d38592ba2a09824aba5ea9601dd68cc3100fae16f784ea&mpshare=1&scene=24&srcid=0823jyHFp5Xgie6itIbUAKvI#rd)

