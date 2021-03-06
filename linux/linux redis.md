Redis 官网：https://redis.io/

Redis 在线测试：http://try.redis.io/

# 基本的数据结构

- String: 字符串
- Hash: 散列
- List: 列表
- Set: 集合
- Sorted Set: 有序集合

# 应用场景


类型 | 简介 | 特性 | 场景
---|---|---|---
String(字符串) |二进制安全  | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M  | ---
Hash(字典)  |  键值对集合,即编程语言中的Map类型  |适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) |存储、读取、修改用户属性
List(列表)  |  链表(双向链表)   | 增删快,提供了操作某一段元素的API | 1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列
Set(集合) |哈希表实现,元素不重复 |1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作   |1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐
Sorted Set(有序集合)  |  将Set中的元素增加一个权重参数score,元素按score有序排列  |数据插入集合时,已经进行天然排序  |  1、排行榜 2、带权重的消息队列


注意：

> Redis支持多个数据库，并且每个数据库的数据是隔离的不能共享，并且基于单机才有，如果是集群就没有数据库的概念。
> 
> Redis是一个字典结构的存储服务器，而实际上一个Redis实例提供了多个用来存储数据的字典，客户端可以指定将数据存储在哪个字典中。这与我们熟知的在一个关系数据库实例中可以创建多个数据库类似，所以可以将其中的每个字典都理解成一个独立的数据库。
> 
> 每个数据库对外都是一个从0开始的递增数字命名，Redis默认支持16个数据库（可以通过配置文件支持更多，无上限），可以通过配置databases来修改这一数字。客户端与Redis建立连接后会自动选择0号数据库，不过可以随时使用SELECT命令更换数据库，如要选择1号数据库：
> 
> 
> ```
> redis> SELECT 1
> OK
> redis [1] > GET foo
> (nil)
> ```
> 
> 然而这些以数字命名的数据库又与我们理解的数据库有所区别。首先Redis不支持自定义数据库的名字，每个数据库都以编号命名，开发者必须自己记录哪些数据库存储了哪些数据。另外Redis也不支持为每个数据库设置不同的访问密码，所以一个客户端要么可以访问全部数据库，要么连一个数据库也没有权限访问。最重要的一点是多个数据库之间并不是完全隔离的，比如FLUSHALL命令可以清空一个Redis实例中所有数据库中的数据。综上所述，这些数据库更像是一种命名空间，而不适宜存储不同应用程序的数据。比如可以使用0号数据库存储某个应用生产环境中的数据，使用1号数据库存储测试环境中的数据，但不适宜使用0号数据库存储A应用的数据而使用1号数据库B应用的数据，不同的应用应该使用不同的Redis实例存储数据。由于Redis非常轻量级，一个空Redis实例占用的内在只有1M左右，所以不用担心多个Redis实例会额外占用很多内存。



# 安装

win： https://github.com/microsoftarchive/redis/releases

启动

```
redis-server.exe redis.windows.conf

# 后面的那个 redis.windows.conf 可以省略，如果省略，会启用默认的
```

关闭

```
如果是用apt-get或者yum install安装的redis，可以直接通过下面的命令停止/启动/重启redis

/etc/init.d/redis-server stop
/etc/init.d/redis-server start
/etc/init.d/redis-server restart

如果是通过源码安装的redis，则可以通过redis的客户端程序redis-cli的shutdown命令来重启redis

redis-cli -h 127.0.0.1 -p 6379 shutdown

如果上述方式都没有成功停止redis，则可以使用终极武器 kill -9

```



```
另启一个 cmd 窗口，原来的不要关闭，不然就无法访问服务端了。
切换到 redis 目录下运行:
redis-cli.exe -h 127.0.0.1 -p 6379

设置键值对:
set myKey abc

取出键值对:
get myKey
```

源码安装： https://redis.io/download


```
wget http://download.redis.io/releases/redis-5.0.4.tar.gz
tar xzf redis-5.0.4.tar.gz
cd redis-5.0.4
make

# make完后 redis-2.8.17目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下：

下面启动redis服务
cd src
# 使用的是默认配置

./redis-server
# 通过启动参数告诉redis使用指定配置文件使用下面命令启动
# redis.conf 是一个默认的配置文件。我们可以根据需要使用自己的配置文件
./redis-server ../redis.conf


# 使用测试客户端程序redis-cli和redis服务交互了
cd src
./redis-cli
set foo bar
get foo

```

ubuntu安装：

```
sudo apt-get update
sudo apt-get install redis-server

# 启动 Redis
redis-server
#查看 redis 是否启动？
redis-cli
ping  # 打印PONG则成功安装
```

# redis配置

> Redis 的配置文件位于 Redis 安装目录下，文件名为 redis.conf(Windows 名为 redis.windows.conf)。

查看配置

```
你可以通过 CONFIG 命令查看或设置配置项。
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
```

```
# 示例
redis 127.0.0.1:6379> CONFIG GET loglevel

# 获取所有配置项
redis 127.0.0.1:6379> CONFIG GET loglevel
```

编辑配置

你可以通过修改 redis.conf 文件或使用 CONFIG set 命令来修改配置。

```
CONFIG SET 命令基本语法：
redis 127.0.0.1:6379> CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
```

```
# 实例
redis 127.0.0.1:6379> CONFIG SET loglevel "notice"
```

部分参数说明

redis.conf 配置项说明如下：

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

```
daemonize no
```

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定

```
pidfile /var/run/redis.pid
```

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字

```
port 6379
```

4. 绑定的主机地址

```
bind 127.0.0.1
```

5.当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

```
timeout 300
```

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

```
loglevel verbose
```

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

```
logfile stdout
```

8. 设置数据库的数量，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id

```
databases 16
```

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

```
save <seconds> <changes>

Redis默认配置文件中提供了三个条件：

save 900 1

save 300 10

save 60 10000

分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
```

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

```
rdbcompression yes
```

11. 指定本地数据库文件名，默认值为dump.rdb

```
dbfilename dump.rdb
```

12. 指定本地数据库存放目录

```
dir ./
```

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

```
slaveof <masterip> <masterport>
```

14. 当master服务设置了密码保护时，slav服务连接master的密码

```
masterauth <master-password>
```


15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭

```
requirepass foobared
```

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

```
maxclients 128
```

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

```
maxmemory <bytes>
```

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

```
appendonly no
```

19. 指定更新日志文件名，默认为appendonly.aof

```
appendfilename appendonly.aof
```

20. 指定更新日志条件，共有3个可选值： 
    no：表示等操作系统进行数据缓存同步到磁盘（快） 
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全） 
    everysec：表示每秒同步一次（折中，默认值）

```
appendfsync everysec
```

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）

```
vm-enabled no
```

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

```
vm-swap-file /tmp/redis.swap
```

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

```
vm-max-memory 0
```

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

```
vm-page-size 32
```

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。

```
vm-pages 134217728
```

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

```
vm-max-threads 4
```

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

```
glueoutputbuf yes
```

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

```
hash-max-zipmap-entries 64

hash-max-zipmap-value 512
```

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

```
activerehashing yes
```

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

```
include /path/to/local.conf
```

# redis-cli

```
# 本地连接
$redis-cli
redis 127.0.0.1:6379>
# 检测 redis 服务是否启动
redis 127.0.0.1:6379> PING

PONG



# 远程连接
redis-cli -h host -p port -a password

# 例
redis-cli -h 127.0.0.1 -p 6379 -a "mypass"

```

# Redis key操作

更多命令请参考：https://redis.io/commands

-1. SET key

> 设置key值

0. GET key

> 获取key的值

1. DEL key

> 该命令用于在 key 存在时删除 key。


2. DUMP key 

> 序列化给定 key ，并返回被序列化的值。


3. EXISTS key 

> 检查给定 key 是否存在。


4. EXPIRE key seconds

> 为给定 key 设置过期时间，以秒计。


5. EXPIREAT key timestamp 

> EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。


6. PEXPIRE key milliseconds 

> 设置 key 的过期时间以毫秒计。


7. PEXPIREAT key milliseconds-timestamp 

> 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计


8. KEYS pattern 

> 查找所有符合给定模式( pattern)的 key 。


9. MOVE key db 

> 将当前数据库的 key 移动到给定的数据库 db 当中。


10. PERSIST key 

> 移除 key 的过期时间，key 将持久保持。


11. PTTL key 

> 以毫秒为单位返回 key 的剩余的过期时间。


12. TTL key 

> 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。


13. RANDOMKEY 

> 从当前数据库中随机返回一个 key 。


14. RENAME key newkey 

> 修改 key 的名称


15. RENAMENX key newkey 

> 仅当 newkey 不存在时，将 key 改名为 newkey 。


16. TYPE key 

> 返回 key 所储存的值的类型。

17. MSET key value key value

> 设置多个key

18. KEYS *

> 查看数据库内所有key

19. FLUSHDB

> 删除当前数据库所有 key



# Redis String操作

1. SET key value 

> 设置指定 key 的值

2. GET key 

> 获取指定 key 的值。

3. GETRANGE key start end 

> 返回 key 中字符串值的子字符

4. GETSET key value

> 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

5. GETBIT key offset

> 对 key 所储存的字符串值，获取指定偏移量上的位(bit)。

6. MGET key1 [key2..]

> 获取所有(一个或多个)给定 key 的值。

7. SETBIT key offset value

> 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

8. SETEX key seconds value

> 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。

9. SETNX key value

> 只有在 key 不存在时设置 key 的值。

10. SETRANGE key offset value

> 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。

11. STRLEN key

> 返回 key 所储存的字符串值的长度。

12. MSET key value [key value ...]

> 同时设置一个或多个 key-value 对。

13. MSETNX key value [key value ...] 

> 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

14. PSETEX key milliseconds value

> 这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。

15. INCR key

> 将 key 中储存的数字值增一。

16. INCRBY key increment

> 将 key 所储存的值加上给定的增量值（increment） 。

17. INCRBYFLOAT key increment

> 将 key 所储存的值加上给定的浮点增量值（increment） 。

18. DECR key

> 将 key 中储存的数字值减一。

19. DECRBY key decrement

> key 所储存的值减去给定的减量值（decrement） 。

20. APPEND key value

> 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。


# Redis Hash操作

> Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

> Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

1. HMSET key field1 value1 [field2 value2 ] 

> 同时将多个 field-value (域-值)对设置到哈希表 key 中。

2. HDEL key field1 [field2] 

> 删除一个或多个哈希表字段

3. HEXISTS key field 

> 查看哈希表 key 中，指定的字段是否存在。

4. HGET key field 

> 获取存储在哈希表中指定字段的值。

5. HGETALL key 

> 获取在哈希表中指定 key 的所有字段和值

6. HINCRBY key field increment 

> 为哈希表 key 中的指定字段的整数值加上增量 increment 。

7. HINCRBYFLOAT key field increment 

> 为哈希表 key 中的指定字段的浮点数值加上增量 increment 。

8. HKEYS key 

> 获取所有哈希表中的字段

9. HLEN key 

> 获取哈希表中字段的数量

10. HMGET key field1 [field2] 

> 获取所有给定字段的值

11. HSET key field value 

> 将哈希表 key 中的字段 field 的值设为 value 。

12. HSETNX key field value 

> 只有在字段 field 不存在时，设置哈希表字段的值。

13. HVALS key 

> 获取哈希表中所有值

14. HSCAN key cursor [MATCH pattern] [COUNT count] 

> 迭代哈希表中的键值对。


# Redis List操作

> Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边

> 一个列表最多可以包含 2的32-1次方 个元素 (4294967295, 每个列表超过40亿个元素)。

1. RPUSHX key value 

> 为已存在的列表添加值 尾部

2. LPUSH key value1 [value2] 

> 将一个或多个值插入到列表头部

3. BLPOP key1 [key2 ] timeout 

> 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

4. BRPOP key1 [key2 ] timeout 

> 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

5. BRPOPLPUSH source destination timeout 

> 从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

6. LINDEX key index 

> 通过索引获取列表中的元素

7. LINSERT key BEFORE|AFTER pivot value 

> 在列表的元素前或者后插入元素

8. LLEN key 

> 获取列表长度

9. LPOP key 

> 移出并获取列表的第一个元素

10. LPUSHX key value 

> 将一个值插入到已存在的列表头部

11. LRANGE key start stop 

> 获取列表指定范围内的元素

12. LREM key count value 

> 移除列表元素

13. LSET key index value 

> 通过索引设置列表元素的值

14. LTRIM key start stop 

> 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

15. RPOP key 

> 移除列表的最后一个元素，返回值为移除的元素。

16. RPOPLPUSH source destination 

> 移除列表的最后一个元素，并将该元素添加到另一个列表并返回

17. RPUSH key value1 [value2] 

> 在列表中添加一个或多个值


# Redis Set操作

> Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。  
Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。

> 集合中最大的成员数为 2的32-1次方 (4294967295, 每个集合可存储40多亿个成员)。

1. SADD key member1 [member2] 

> 向集合添加一个或多个成员

2. SCARD key 

> 获取集合的成员数

3. SDIFF key1 [key2] 

> 返回给定所有集合的差集

4. SDIFFSTORE destination key1 [key2] 

> 返回给定所有集合的差集并存储在 destination 中

5. SINTER key1 [key2] 

> 返回给定所有集合的交集

6. SINTERSTORE destination key1 [key2] 

> 返回给定所有集合的交集并存储在 destination 中

7. SISMEMBER key member 

> 判断 member 元素是否是集合 key 的成员

8. SMEMBERS key 

> 返回集合中的所有成员

9. SMOVE source destination member 

> 将 member 元素从 source 集合移动到 destination 集合

10. SPOP key 

> 移除并返回集合中的一个随机元素

11. SRANDMEMBER key [count] 

> 返回集合中一个或多个随机数

12. SREM key member1 [member2] 

> 移除集合中一个或多个成员

13. SUNION key1 [key2] 

> 返回所有给定集合的并集

14. SUNIONSTORE destination key1 [key2] 

> 所有给定集合的并集存储在 destination 集合中

15. SSCAN key cursor [MATCH pattern] [COUNT count] 

> 迭代集合中的元素


# Redis sorted set操作（有序集合）

> Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。  
不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。  
有序集合的成员是唯一的,但分数(score)却可以重复。

> 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 2的32-1次方 (4294967295, 每个集合可存储40多亿个成员)。

> 原文中说，集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)其实不太准确。

> 其实在redis sorted sets里面当items内容大于64的时候同时使用了hash和skiplist两种设计实现。这也会为了排序和查找性能做的优化。所以如上可知：   
添加和删除都需要修改skiplist，所以复杂度为O(log(n))。   
但是如果仅仅是查找元素的话可以直接使用hash，其复杂度为O(1)   
其他的range操作复杂度一般为O(log(n))  
当然如果是小于64的时候，因为是采用了ziplist的设计，其时间复杂度为O(n)  


1. ZADD key score1 member1 [score2 member2] 

> 向有序集合添加一个或多个成员，或者更新已存在成员的分数

2. ZCARD key 

> 获取有序集合的成员数

3. ZCOUNT key min max 

> 计算在有序集合中指定区间分数的成员数

4. ZINCRBY key increment member 

> 有序集合中对指定成员的分数加上增量 increment

5. ZINTERSTORE destination numkeys key [key ...] 

> 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中

6. ZLEXCOUNT key min max 

> 在有序集合中计算指定字典区间内成员数量

7. ZRANGE key start stop [WITHSCORES] 

> 通过索引区间返回有序集合成指定区间内的成员

8. ZRANGEBYLEX key min max [LIMIT offset count] 

> 通过字典区间返回有序集合的成员

9. ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 

> 通过分数返回有序集合指定区间内的成员

10. ZRANK key member 

> 返回有序集合中指定成员的索引

11. ZREM key member [member ...] 

> 移除有序集合中的一个或多个成员

12. ZREMRANGEBYLEX key min max 

> 移除有序集合中给定的字典区间的所有成员

13. ZREMRANGEBYRANK key start stop 

> 移除有序集合中给定的排名区间的所有成员

14. ZREMRANGEBYSCORE key min max 

> 移除有序集合中给定的分数区间的所有成员

15. ZREVRANGE key start stop [WITHSCORES] 

> 返回有序集中指定区间内的成员，通过索引，分数从高到底

16. ZREVRANGEBYSCORE key max min [WITHSCORES] 

> 返回有序集中指定分数区间内的成员，分数从高到低排序

17. ZREVRANK key member 

> 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序

18. ZSCORE key member 

> 返回有序集中，成员的分数值

19. ZUNIONSTORE destination numkeys key [key ...] 

> 计算给定的一个或多个有序集的并集，并存储在新的 key 中

20. ZSCAN key cursor [MATCH pattern] [COUNT count] 

> 迭代有序集合中的元素（包括元素成员和元素分值）


# Redis HyperLogLog操作

> Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

> 在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

> 但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素


什么是基数?
> 比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

1. PFADD key element [element ...] 

> 添加指定元素到 HyperLogLog 中。

2. PFCOUNT key [key ...] 

> 返回给定 HyperLogLog 的基数估算值。

3. PFMERGE destkey sourcekey [sourcekey ...] 

> 将多个 HyperLogLog 合并为一个 HyperLogLog


# Redis 发布订阅

> Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

> Redis 客户端可以订阅任意数量的频道。


流程： 一个客户端订阅，另一个客户端想同一个频道push消息

1. PSUBSCRIBE pattern [pattern ...] 

> 订阅一个或多个符合给定模式的频道。

2. PUBSUB subcommand [argument [argument ...]] 

> 查看订阅与发布系统状态。

3. PUBLISH channel message 

> 将信息发送到指定的频道。

4. PUNSUBSCRIBE [pattern [pattern ...]] 

> 退订所有给定模式的频道。

5. SUBSCRIBE channel [channel ...] 

> 订阅给定的一个或多个频道的信息。

6. UNSUBSCRIBE [channel [channel ...]] 

> 指退订给定的频道。


# Redis 事务操作

> Redis 事务可以一次执行多个命令， 并且带有以下两个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

> 一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。


```
# 例子

redis 127.0.0.1:6379> MULTI
OK

redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED

redis 127.0.0.1:6379> GET book-name
QUEUED

redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED

redis 127.0.0.1:6379> SMEMBERS tag
QUEUED

redis 127.0.0.1:6379> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"

```
注意

> 单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

> 事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

1. DISCARD 

> 取消事务，放弃执行事务块内的所有命令。

2. EXEC 

> 执行所有事务块内的命令。

3. MULTI 

> 标记一个事务块的开始。

4. UNWATCH 

> 取消 WATCH 命令对所有 key 的监视。

5. WATCH key [key ...] 

> 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。


# Redis 脚本（Lua语法）

1. EVAL script numkeys key [key ...] arg [arg ...] 

> 执行 Lua 脚本。

2. EVALSHA sha1 numkeys key [key ...] arg [arg ...] 

> 执行 Lua 脚本。

3. SCRIPT EXISTS script [script ...] 

> 查看指定的脚本是否已经被保存在缓存当中。

4. SCRIPT FLUSH 

> 从脚本缓存中移除所有脚本。

5. SCRIPT KILL 

> 杀死当前正在运行的 Lua 脚本。

6. SCRIPT LOAD script 

> 将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。



# Redis 连接

1. AUTH password 

> 验证密码是否正确

2. ECHO message 

> 打印字符串

3. PING 

> 查看服务是否运行

4. QUIT 

> 关闭当前连接

5. SELECT index 

> 切换到指定的数据库

# Redis 服务器

> Redis 服务器命令主要是用于管理 redis 服务。

1. BGREWRITEAOF 

> 异步执行一个 AOF（AppendOnly File） 文件重写操作

2. BGSAVE 

> 在后台异步保存当前数据库的数据到磁盘

3. CLIENT KILL [ip:port] [ID client-id] 

> 关闭客户端连接

4. CLIENT LIST 

> 获取连接到服务器的客户端连接列表

5. CLIENT GETNAME 

> 获取连接的名称

6. CLIENT PAUSE timeout 

> 在指定时间内终止运行来自客户端的命令

7. CLIENT SETNAME connection-name 

> 设置当前连接的名称

8. CLUSTER SLOTS 

> 获取集群节点的映射数组

9. COMMAND 

> 获取 Redis 命令详情数组

10. COMMAND COUNT 

> 获取 Redis 命令总数

11. COMMAND GETKEYS 

> 获取给定命令的所有键

12. TIME 

> 返回当前服务器时间

13. COMMAND INFO command-name [command-name ...] 

> 获取指定 Redis 命令描述的数组

14. CONFIG GET parameter 

> 获取指定配置参数的值

15. CONFIG REWRITE 

> 对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写

16. CONFIG SET parameter value 

> 修改 redis 配置参数，无需重启

17. CONFIG RESETSTAT 

> 重置 INFO 命令中的某些统计数据

18. DBSIZE 

> 返回当前数据库的 key 的数量

19. DEBUG OBJECT key 

> 获取 key 的调试信息

20. DEBUG SEGFAULT 

> 让 Redis 服务崩溃

21. FLUSHALL 

> 删除所有数据库的所有key

22. FLUSHDB 

> 删除当前数据库的所有key

23. INFO [section] 

> 获取 Redis 服务器的各种信息和统计数值

24. LASTSAVE 

> 返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示

25. MONITOR 

> 实时打印出 Redis 服务器接收到的命令，调试用

26. ROLE 

> 返回主从实例所属的角色

27. SAVE 

> 同步保存数据到硬盘

28. SHUTDOWN [NOSAVE] [SAVE] 

> 异步保存数据到硬盘，并关闭服务器

29. SLAVEOF host port 

> 将当前服务器转变为指定服务器的从属服务器(slave server)

30. SLOWLOG subcommand [argument] 

> 管理 redis 的慢日志

31. SYNC 

> 用于复制功能(replication)的内部命令


# Redis 数据备份与恢复

```
创建备份，将在 redis 安装目录中创建dump.rdb文件
redis 127.0.0.1:6379> SAVE 
OK

恢复数据
需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"

以上命令 CONFIG GET dir 输出的 redis 安装目录为 /usr/local/redis/bin


Bgsave
创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。
127.0.0.1:6379> BGSAVE
Background saving started
```

# Redis 安全

```
查看是否设置了密码验证
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) ""

默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。

你可以通过以下命令来修改该参数：
127.0.0.1:6379> CONFIG set requirepass "runoob"
OK
127.0.0.1:6379> CONFIG get requirepass
1) "requirepass"
2) "runoob"
设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令
```



# Redis 性能测试

```
指令，linux下执行
redis-benchmark [option] [option value]

以下实例同时执行 10000 个请求来检测性能：
$ redis-benchmark -n 10000  -q

以下实例我们使用了多个参数来测试 redis 性能：

$ redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q

以上实例中主机为 127.0.0.1，端口号为 6379，执行的命令为 set,lpush，请求数为 10000，通过 -q 参数让结果只显示每秒执行的请求数
```
可选参数如下

序号 | 选项 | 描述 | 默认值
---|---|---|---
1  | -h | 指定服务器主机名  |  127.0.0.1
2  | -p | 指定服务器端口 | 6379
3  | -s | 指定服务器 socket    
4  | -c | 指定并发连接数 | 50
5  | -n | 指定请求数  | 10000
6  | -d | 以字节的形式指定 SET/GET 值的数据大小 | 2
7  | -k | 1=keep alive 0=reconnect  |  1
8  | -r | SET/GET/INCR 使用随机 key, SADD 使用随机值   
9  | -P | 通过管道传输 <numreq> 请求 | 1
10 | -q | 强制退出 redis。仅显示 query/sec 值  
11 | --csv  | 以 CSV 格式输出  
12 | -l | 生成循环，永久执行测试 
13 | -t | 仅运行以逗号分隔的测试命令列表。    
14 | -I | Idle 模式。仅打开 N 个 idle 连接并等待。 


# Redis 客户端连接

> Redis 通过监听一个 TCP 端口或者 Unix socket 的方式来接收来自客户端的连接，当一个连接建立后，Redis 内部会进行以下一些操作：

- 首先，客户端 socket 会被设置为非阻塞模式，因为 Redis 在网络事件处理上采用的是非阻塞多路复用模型。
- 然后为这个 socket 设置 TCP_NODELAY 属性，禁用 Nagle 算法
- 然后创建一个可读的文件事件用于监听这个客户端 socket 的数据发送

```
最大连接数，默认10000
config get maxclients
1) "maxclients"
2) "10000"

实例
以下实例我们在服务启动时设置最大连接数为 100000：

redis-server --maxclients 100000
```

客户端命令

S.N. |   命令 | 描述
---|---|---
1  | CLIENT LIST |返回连接到 redis 服务的客户端列表
2  | CLIENT SETNAME | 设置当前连接的名称
3  | CLIENT GETNAME | 获取通过 CLIENT SETNAME 命令设置的服务名称
4  | CLIENT PAUSE   | 挂起客户端连接，指定挂起的时间以毫秒计
5  | CLIENT KILL | 关闭客户端连接


# Redis管道技术（找时间深入下）

> Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。这意味着通常情况下一个请求会遵循以下步骤：

- 客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
- 服务端处理命令，并将结果返回给客户端。

Redis 管道技术
> Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应

以上实例中我们通过使用 PING 命令查看redis服务是否可用， 之后我们设置了 runoobkey 的值为 redis，然后我们获取 runoobkey 的值并使得 visitor 自增 3 次。

在返回的结果中我们可以看到这些命令一次性向 redis 服务提交，并最终一次性读取所有服务端的响应


# Redis 分区

> 分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。

优势
- 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
- 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

不足
- 涉及多个key的操作通常是不被支持的。举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
- 涉及多个key的redis事务不能使用。
- 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
- 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。然而，一种叫做presharding的技术对此是有帮助的。


分区类型

> Redis 有两种类型分区。 假设有4个Redis实例 R0，R1，R2，R3，和类似user:1，user:2这样的表示用户的多个key，对既定的key有多种不同方式来选择这个key存放在哪个实例中。也就是说，有不同的系统来映射某个key到某个Redis服务。


范围分区

> 最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。

> 比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。

> 这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。

哈希分区

> 另外一种分区方法是hash分区。这对任何key都适用，也无需是object_name:这种形式，像下面描述的一样简单：

- 用一个hash函数将key转换为一个数字，比如使用crc32 hash函数。对key foobar执行crc32(foobar)会输出类似93024922的整数。
- 对这个整数取模，将其转化为0-3之间的数字，就可以将这个整数映射到4个Redis实例中的一个了。93024922 % 4 = 2，就是说key foobar应该被存到R2实例中。注意：取模操作是取除的余数，通常在多种编程语言中用%操作符实现。

