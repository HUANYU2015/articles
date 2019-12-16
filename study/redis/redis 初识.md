# redis

* remote dictionary server
* 使用 ANSI C语言比那些的开源数据库
* 数据库、缓存、消息中间件
* 高性能的 key-value 数据库
* 内存数据库，支持数据持久化
* https:/redis.cn
* 2.8.0

## Redis 的安装

``` sh
# 解压
tar -zxvf redis-2.8.0.tar.gz

# 进入 Redis 主目录
# 编译
make

# 编译测试
make test

# src文件夹下 启动 Redis    
# 占用命令行窗口
./redis-server
# 不占用命令行窗口
./redis-server &

# src 文件夹下，运行客户端
./redis-cli
```

## redis 单实例配置

* redis.conf 配置文件
* port 端口
* requirepass 密码
* masterauth 主从同步中在 salve 配置 master 的密码(只在配置主从同步，并且需要密码的时候才会用到)
* databases redis.conf 中的一个配置项，可以用来设定 keyspace的范围

>> requirepass 和 masterauth都在 redis.conf 中配置，前一个在 Redis 的 master 节点中配置，后一个在 Redis 的 salve节点中配置，且值相等。配置方式：`key value`，不需要等号。

**服务启动方式**

```sh
# 单实例服务端启动方式
./redis-server

./redis-server --port ${port}
./redis-server ${redis.conf}
./redis-server ../redis.conf
```

>> 注意指定端口（--port）启动 Redis 服务的优先级大于指定配置文件及默认端口启动

**客户端连接方式**

``` sh
# Redis 单实例客户端启动
./redis-cli

./redis-cli -p ${port}

./redis-cli -h  ${ip}

./redis-cli -a ${password}

# 密码 设置在配置文件中 的requirepass字段
./redis-cli -p ${port} -h ${ip} -a ${password}
```

**关闭 Redis 服务**

``` sh
# 关闭 Redis 服务
# 关闭服务端
kill -p ${pid}
redis-cli shutdown
redis-cli -p ${port} shutdown
redis-cli -h ${ip} shutdown
redis-cli -p ${port} -h ${ip} shutdown
```

>> 注意如果启动服务时，使用了对应的参数，那么启动客户端的时候，也需要添加对应的参数，同样关闭服务时，也许如此。

**单实例验证**

``` sh
# 单实例环境验证
> ping
< pong

> 执行 Redis set 命令相关
keys *
set a aa

> 查看 Redis get 命令获取相关值
get a
```

## redis 基础命令

* info
  * 查看系统信息
* ping
  * 验证客户端连接是否正常，通常会回复 pong
* quit
  * 退出 client 连接
* save
  * 使用 **./redis-cli shutdown** 关闭 Redis 服务，Redis 会自动触发数据持久化；如果使用 Control + C 会终止 Redis 服务，则相关数据会丢失。为此，若在使用 Control+C终止 Redis 服务之前使用 **save** 命令便可进行数据存储，而不会丢失。
* dbsize
  * 查看当前 keyspace 中键值对对数，或者 keys的值，可以通过 info 命令查看
* select
  * 用来选择 keyspace （0~databases -1）
* flushdb
  * 清除当前 keysapce 中的键值对
* flushall
  * 清除所有 keysapce 中的键值对

## redis 键命令

* set
  * 设置键值对
* del
  * 删除键值对
* exists
  * 判断键是否存在
* expire
  * 设置键值对的剩余生存时间，单位是秒
* ttl
  * 查看键值对是否设置有剩余生存时间，如该键值对为设置剩余生存时间，则返回 `-1` ，如果返回 `-2` 则该键值对对不存在
* type
  * 返回键值对的类型
* randomkey
  * 随机返回已有的键值对
* rename
  * 重命名键值对的键，rename a d，如 d 已经存在，则会覆盖已存在
* renamenx
  * 重命名键值对的键，renamenx a d，如 d 已经存在，则不会操作成功

## 五种数据结构的命令

* string(字符串)
  * set a aa
  * setex a 100 aa 设置键值对的同时设置该键值对的剩余生存时间
  * psetex b 10000 b 剩余生命时间单位为毫秒
  * setnx [key] [value] 设置键值对，如该键已经存在，则设置不成功
  * getrange [key] [indexStart] [indexEnd] 截取键所对应值从 indexStart 到 indexEnd 范围内的字符串
  * getset a aa 设置键a 的新值为 aa,并将键a 的旧值返回
  * mset a1 a1 b1 b1 c1 c1 同时设置多个键值对
  * mget a1 b1 c1 同时获取 多个键的值
  * strlen a 获得字符串的长度
  * msetnx 兼顾批量和判断的设置，且具有原子性，及多个设置时，要么都成功，要么都失败
  * incr key 将key 对应的数值（integer）增加 1
    * incrby key [step] 指定步长增加
  * decr key 与 incr 作用相反
    * decrby key [step] 指定步长减少
  * apend key value 将 value 附加到 key 对应值的末尾

* hash(hash 表)
  * hset hash key value
  * hexists hash key 判断 名为hash的 hash是否包含名为 key 的键
  * hget hash key 获得名为 hash 的 hash 中键为 key 对应的值
  * hgetall hash 获取名为 hash 的 hash 中的键值对，键值，键值顺序展示
  * hkeys hash 获取所有键
  * hvals hash 获取所有值
  * hlen map key的个数
  * hmget hash [key1] [key2]
  * hmset hash [key1] [value1] [key2] [value2]
  * hdel hash [key1] [key2]
  * hsetnx hash [key] [value] 若 key已经存在，则设置失败
* list(链表)
  * lpush list 1 2 3 4 5 向名为 list 的 list 类型数据添加元素
  * llen list 返回 list 的长度
  * lrange list 0 2 注意从后向前取出，后放先出
  * lset list [index] [value] 设置固定索引的新值
  * lindex list [index] 获取固定索引的值
  * lpop list 移除 list 的第一个元素，即就是最后一个放进去的数
  * rpop list 与 lpop命令作用相反
* set(无序集合)
  * sadd set [v1] [v2] [v3] 向 set 中添加元素，不能重复添加，前面添加 nx 后缀的命令才有此功能
  * scard set 获取集合元素的数量
  * smembers set1 查看 set1中的成员，无序
  * sdiff set1 set2 获取 set1相较 set2 独有的元素
  * sinter set1 set2 获取两者共有的元素
  * sunion set1 set2 或缺两者元素的集合
  * srandmember set1 2 随机获取 set1 中的 2 个元素
  * sismember set1 a 判断一个元素是否存在于 set1 中
  * srem set1 a b 移除 set1 中的元素
  * spop set1 移除 set1 中随机的一个元素，并返回该元素
* sorted set(有序集合) 类似linkedhashSet
  * zadd sortedset1 100 a 200 b 300 c
  * rename sortedset1 sortedset
  * zcard sortedset
  * zscore sortedset a 获取a 的分数
  * zcount sortedset 0 300 返回固定分数区间内的元素个数
  * zrank sortedset a 返回对应元素的索引，依据分数从小到大排列
  * zincrby sortedset 1000 a 将 a 的分数增加 1000
  * zrange sortedset 0 100 获取固定索引范围内的元素
  * zrange sortedset 0 100 withscores 获取固定索引范围内的元素及对应分数

>> string 类型数据 对应 raw、int ,,,多种编码方式

## 什么是Redis

答：用C语言开发的高性能的键值对的数据库，支持多种键值对类型，非关系型数据库。

应用场景：
1)缓存
2)任务队列
3)网站访问统计
4)数据过期的处理
5)应用的排行版
6)分布式集群架构中的session分离

## 什么是NoSQL

答：NoSQL=not only sql，不仅仅适用SQL，非关系型数据库，web2.0的兴起的带动下催生出来的。

## 为什么要用NoSQL

High performance -高并发读写
Huge storge -海量数据的高效存储和访问
High scalability && high availability高扩展性和可用性

特点：
易扩展 去掉数据库之间的关系，
灵活的数据类型
大量数据、高性能
高可用

## NoSQL的主要产品

答：couchDB、Redis、MongoDB、membase、riak、Cassandra等等。

## NoSQL数据库的四大存储类型

1）键值（key-value）存储，代表产品Redis，优势快速查询，劣势存储数据缺少结构化；
2）列存储，代表产品couchDB，优势速度快，扩展性强，劣势功能局限；
3）文档数据库，代表产品MongoDB，数据结构要求不是太严格，缺少统一的查询语法；
4）图形数据库，代表产品infograd。

![图片1](/assets/图片1.png)


