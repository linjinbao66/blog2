---
title: Redis笔记
tags:
  - Redis
  - 缓存
  - 后端
  - development
categories:
  - 缓存
date: 2019-07-29
---

## Redis笔记

### key-value结构

Redis目前有5种数据类型，分别是：

```text
    String（字符串）
    List（列表）
    Hash（字典）
    Set（集合）
    Sorted Set（有序集合）
```

不同的数据类型，有不同的命令方式

String 操作

```text
set google http://www.google
append google .com
get google
set visitors 0
incr visitors
incr visitors
get visitors
incrby visitors 100
get visitors
type google
type visitors
ttl google
rename google google-site
get google
get google-site
```

String 手册

```text
SET key value                   设置key=value
GET key                         获得键key对应的值
GETRANGE key start end          得到字符串的子字符串存放在一个键
GETSET key value                设置键的字符串值，并返回旧值
GETBIT key offset               返回存储在键位值的字符串值的偏移
MGET key1 [key2..]              得到所有的给定键的值
SETBIT key offset value         设置或清除该位在存储在键的字符串值偏移
SETEX key seconds value         键到期时设置值
SETNX key value                 设置键的值，只有当该键不存在
SETRANGE key offset value       覆盖字符串的一部分从指定键的偏移
STRLEN key                      得到存储在键的值的长度
MSET key value [key value...]   设置多个键和多个值
MSETNX key value [key value...] 设置多个键多个值，只有在当没有按键的存在时
PSETEX key milliseconds value   设置键的毫秒值和到期时间
INCR key                        增加键的整数值一次
INCRBY key increment            由给定的数量递增键的整数值
INCRBYFLOAT key increment       由给定的数量递增键的浮点值
DECR key                        递减键一次的整数值
DECRBY key decrement            由给定数目递减键的整数值
APPEND key value                追加值到一个键
DEL key                         如果存在删除键
DUMP key                        返回存储在指定键的值的序列化版本
EXISTS key                      此命令检查该键是否存在
EXPIRE key seconds              指定键的过期时间
EXPIREAT key timestamp          指定的键过期时间。在这里，时间是在Unix时间戳格式
PEXPIRE key milliseconds        设置键以毫秒为单位到期
PEXPIREAT key milliseconds-timestamp        设置键在Unix时间戳指定为毫秒到期
KEYS pattern                    查找与指定模式匹配的所有键
MOVE key db                     移动键到另一个数据库
PERSIST key                     移除过期的键
PTTL key                        以毫秒为单位获取剩余时间的到期键。
TTL key                         获取键到期的剩余时间。
RANDOMKEY                       从Redis返回随机键
RENAME key newkey               更改键的名称
RENAMENX key newkey             重命名键，如果新的键不存在
TYPE key                        返回存储在键的数据类型的值。
```

list列表 list操作

```text
lpush list1 redis
lpush list1 hello
rpush list1 world
llen list1
lrange list1 0 3
lpop list1
rpop list1
lrange list1 0 3
```

list手册

```text
BLPOP key1 [key2 ] timeout 取出并获取列表中的第一个元素，或阻塞，直到有可用
BRPOP key1 [key2 ] timeout 取出并获取列表中的最后一个元素，或阻塞，直到有可用
BRPOPLPUSH source destination timeout 从列表中弹出一个值，它推到另一个列表并返回它;或阻塞，直到有可用
LINDEX key index 从一个列表其索引获取对应的元素
LINSERT key BEFORE|AFTER pivot value 在列表中的其他元素之后或之前插入一个元素
LLEN key 获取列表的长度
LPOP key 获取并取出列表中的第一个元素
LPUSH key value1 [value2] 在前面加上一个或多个值的列表
LPUSHX key value 在前面加上一个值列表，仅当列表中存在
LRANGE key start stop 从一个列表获取各种元素
LREM key count value 从列表中删除元素
LSET key index value 在列表中的索引设置一个元素的值
LTRIM key start stop 修剪列表到指定的范围内
RPOP key 取出并获取列表中的最后一个元素
RPOPLPUSH source destination 删除最后一个元素的列表，将其附加到另一个列表并返回它
RPUSH key value1 [value2] 添加一个或多个值到列表
RPUSHX key value 添加一个值列表，仅当列表中存在
```

hash字典，哈希表 hash操作

```text
hset person name jack
hset person age 20
hset person sex female
hgetall person
hkeys person
hvals person
```

hash手册

```text
HDEL key field[field...] 删除对象的一个或几个属性域，不存在的属性将被忽略
HEXISTS key field 查看对象是否存在该属性域
HGET key field 获取对象中该field属性域的值
HGETALL key 获取对象的所有属性域和值
HINCRBY key field value 将该对象中指定域的值增加给定的value，原子自增操作，只能是integer的属性值可以使用
HINCRBYFLOAT key field increment 将该对象中指定域的值增加给定的浮点数
HKEYS key 获取对象的所有属性字段
HVALS key 获取对象的所有属性值
HLEN key 获取对象的所有属性字段的总数
HMGET key field[field...] 获取对象的一个或多个指定字段的值
HSET key field value 设置对象指定字段的值
HMSET key field value [field value ...] 同时设置对象中一个或多个字段的值
HSETNX key field value 只在对象不存在指定的字段时才设置字段的值
HSTRLEN key field 返回对象指定field的value的字符串长度，如果该对象或者field不存在，返回0.
HSCAN key cursor [MATCH pattern] [COUNT count] 类似SCAN命令
```

Set集合 Set集合操作

```text
SADD myset "Hello"
SADD myset "World"
SMEMBERS myset
SADD myset "one"
SISMEMBER myset "one"
SISMEMBER myset "two"
```

Set手册

```text
SADD key member [member ...] 添加一个或者多个元素到集合(set)里
SCARD key 获取集合里面的元素数量
SDIFF key [key ...] 获得队列不存在的元素
SDIFFSTORE destination key [key ...] 获得队列不存在的元素，并存储在一个关键的结果集
SINTER key [key ...] 获得两个集合的交集
SINTERSTORE destination key [key ...] 获得两个集合的交集，并存储在一个集合中
SISMEMBER key member 确定一个给定的值是一个集合的成员
SMEMBERS key 获取集合里面的所有key
SMOVE source destination member 移动集合里面的一个key到另一个集合
SPOP key [count] 获取并删除一个集合里面的元素
SRANDMEMBER key [count] 从集合里面随机获取一个元素
SREM key member [member ...] 从集合里删除一个或多个元素，不存在的元素会被忽略
SUNION key [key ...] 添加多个set元素
SUNIONSTORE destination key [key ...] 合并set元素，并将结果存入新的set里面
SSCAN key cursor [MATCH pattern] [COUNT count] 迭代set里面的元素
```

Sorted Set 有序集合 Sorted Set 操作

```text
zadd dbs 100 redis
zadd dbs 98 memcached
zadd dbs 99 mongodb
zadd dbs 99 leveldb
zcard dbs
zcount dbs 10 99
zrank dbs leveldb
zrank dbs other
zrangebyscore dbs 98 100
```

Sorted Set手册

```text
ZADD key score1 member1 [score2 member2] 添加一个或多个成员到有序集合，或者如果它已经存在更新其分数
ZCARD key 得到的有序集合成员的数量
ZCOUNT key min max 计算一个有序集合成员与给定值范围内的分数
ZINCRBY key increment member 在有序集合增加成员的分数
ZINTERSTORE destination numkeys key [key ...] 多重交叉排序集合，并存储生成一个新的键有序集合。
ZLEXCOUNT key min max 计算一个给定的字典范围之间的有序集合成员的数量
ZRANGE key start stop [WITHSCORES] 由索引返回一个成员范围的有序集合（从低到高）
ZRANGEBYLEX key min max [LIMIT offset count]返回一个成员范围的有序集合（由字典范围）
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 返回有序集key中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员，有序集成员按 score 值递增(从小到大)次序排列
ZRANK key member 确定成员的索引中有序集合
ZREM key member [member ...] 从有序集合中删除一个或多个成员，不存在的成员将被忽略
ZREMRANGEBYLEX key min max 删除所有成员在给定的字典范围之间的有序集合
ZREMRANGEBYRANK key start stop 在给定的索引之内删除所有成员的有序集合
ZREMRANGEBYSCORE key min max 在给定的分数之内删除所有成员的有序集合
ZREVRANGE key start stop [WITHSCORES] 返回一个成员范围的有序集合，通过索引，以分数排序，从高分到低分
ZREVRANGEBYSCORE key max min [WITHSCORES] 返回一个成员范围的有序集合，以socre排序从高到低
ZREVRANK key member 确定一个有序集合成员的索引，以分数排序，从高分到低分
ZSCORE key member 获取给定成员相关联的分数在一个有序集合
ZUNIONSTORE destination numkeys key [key ...] 添加多个集排序，所得排序集合存储在一个新的键
ZSCAN key cursor [MATCH pattern] [COUNT count] 增量迭代排序元素集和相关的分数
```

java连接测试

```java
package redis;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import org.junit.Before;
import org.junit.Test;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool; 

public class TestRedisManyCommands { 
    JedisPool pool; 
    Jedis jedis; 
    @Before 
    public void setUp() { 
        jedis = new Jedis("localhost"); 
    } 
    /**
     * Redis存储初级的字符串
     * CRUD
     */ 
    @Test 
    public void testBasicString(){ 
        //-----添加数据---------- 
        jedis.set("name","meepo");//向key-->name中放入了value-->meepo 
        System.out.println(jedis.get("name"));//执行结果：meepo 

        //-----修改数据----------- 
        //1、在原来基础上修改 
        jedis.append("name","dota");   //很直观，类似map 将dota append到已经有的value之后 
        System.out.println(jedis.get("name"));//执行结果:meepodota 

        //2、直接覆盖原来的数据 
        jedis.set("name","poofu"); 
        System.out.println(jedis.get("name"));//执行结果：poofu 

        //删除key对应的记录 
        jedis.del("name"); 
        System.out.println(jedis.get("name"));//执行结果：null 

        /**
         * mset相当于
         * jedis.set("name","meepo");
         * jedis.set("dota","poofu");
         */ 
        jedis.mset("name","meepo","dota","poofu"); 
        System.out.println(jedis.mget("name","dota")); 

    } 

    /**
     * jedis操作Map
     */ 
    @Test 
    public void testMap(){ 
        Map<String,String> user=new HashMap<String,String>(); 
        user.put("name","meepo"); 
        user.put("pwd","password"); 
        jedis.hmset("user",user); 
        //取出user中的name，执行结果:[meepo]-->注意结果是一个泛型的List 
        //第一个参数是存入redis中map对象的key，后面跟的是放入map中的对象的key，后面的key可以跟多个，是可变参数 
        List<String> rsmap = jedis.hmget("user", "name"); 
        System.out.println(rsmap); 

        //删除map中的某个键值 
//        jedis.hdel("user","pwd"); 
        System.out.println(jedis.hmget("user", "pwd")); //因为删除了，所以返回的是null 
        System.out.println(jedis.hlen("user")); //返回key为user的键中存放的值的个数1 
        System.out.println(jedis.exists("user"));//是否存在key为user的记录 返回true 
        System.out.println(jedis.hkeys("user"));//返回map对象中的所有key  [pwd, name] 
        System.out.println(jedis.hvals("user"));//返回map对象中的所有value  [meepo, password] 

        Iterator<String> iter=jedis.hkeys("user").iterator(); 
        while (iter.hasNext()){ 
            String key = iter.next(); 
            System.out.println(key+":"+jedis.hmget("user",key)); 
        } 

    } 

    /**
     * jedis操作List
     */ 
    @Test 
    public void testList(){ 
        //开始前，先移除所有的内容 
        jedis.del("java framework"); 
        // 第一个是key，第二个是起始位置，第三个是结束位置，jedis.llen获取长度 -1表示取得所有
        System.out.println(jedis.lrange("java framework",0,-1)); 
       //先向key java framework中存放三条数据 
       jedis.lpush("java framework","spring"); 
       jedis.lpush("java framework","struts"); 
       jedis.lpush("java framework","hibernate"); 
       //再取出所有数据jedis.lrange是按范围取出， 
       // 第一个是key，第二个是起始位置，第三个是结束位置，jedis.llen获取长度 -1表示取得所有 
       System.out.println(jedis.lrange("java framework",0,-1)); 
    } 

    /**
     * jedis操作Set
     */ 
    @Test 
    public void testSet(){ 
        //添加 
        jedis.sadd("sname","meepo"); 
        jedis.sadd("sname","dota"); 
        jedis.sadd("sname","poofu"); 
        jedis.sadd("sname","noname");
        //移除noname 
        jedis.srem("sname","noname"); 
        System.out.println(jedis.smembers("sname"));//获取所有加入的value 
        System.out.println(jedis.sismember("sname", "meepo"));//判断 meepo 是否是sname集合的元素 
        System.out.println(jedis.srandmember("sname")); 
        System.out.println(jedis.scard("sname"));//返回集合的元素个数 
    } 

    @Test 
    public void test() throws InterruptedException { 
        //keys中传入的可以用通配符 
        System.out.println(jedis.keys("*")); //返回当前库中所有的key  [sose, sanme, name, dota, foo, sname, java framework, user, braand] 
        System.out.println(jedis.keys("*name"));//返回的sname   [sname, name] 
        System.out.println(jedis.del("sanmdde"));//删除key为sanmdde的对象  删除成功返回1 删除失败（或者不存在）返回 0 
        System.out.println(jedis.ttl("sname"));//返回给定key的有效时间，如果是-1则表示永远有效 
        jedis.setex("timekey", 10, "min");//通过此方法，可以指定key的存活（有效时间） 时间为秒 
        Thread.sleep(5000);//睡眠5秒后，剩余时间将为<=5 
        System.out.println(jedis.ttl("timekey"));   //输出结果为5 
        jedis.setex("timekey", 1, "min");        //设为1后，下面再看剩余时间就是1了 
        System.out.println(jedis.ttl("timekey"));  //输出结果为1 
        System.out.println(jedis.exists("key"));//检查key是否存在 
        System.out.println(jedis.rename("timekey","time")); 
        System.out.println(jedis.get("timekey"));//因为移除，返回为null 
        System.out.println(jedis.get("time")); //因为将timekey 重命名为time 所以可以取得值 min 

        //jedis 排序 
        //注意，此处的rpush和lpush是List的操作。是一个双向链表（但从表现来看的） 
        jedis.del("a");//先清除数据，再加入数据进行测试 
        jedis.rpush("a", "1"); 
        jedis.lpush("a","6"); 
        jedis.lpush("a","3"); 
        jedis.lpush("a","9"); 
        System.out.println(jedis.lrange("a",0,-1));// [9, 3, 6, 1] 
        System.out.println(jedis.sort("a")); //[1, 3, 6, 9]  //输入排序后结果 
        System.out.println(jedis.lrange("a",0,-1)); 

    } 
}
```

## 比较

Redis和Memcache区别，优缺点对比

1. Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例如图片、视频等等。
2. Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储。
3. 虚拟内存–Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘
4. 过期策略–memcache在set时就指定，例如set key1 0 0 8,即永不过期。Redis可以通过例如expire 设定，例如expire name 10
5. 分布式–设定memcache集群，利用magent做一主多从;redis可以做一主多从。都可以一主一从
6. 存储数据安全–memcache挂掉后，数据没了；redis可以定期保存到磁盘（持久化）
7. 灾难恢复–memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复
8. Redis支持数据的备份，即master-slave模式的数据备份。

关于redis和memcache的不同，下面罗列了一些相关说法，供记录：

redis和memecache的不同在于：

1. 存储方式：

   memecache 把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小

   redis有部份存在硬盘上，这样能保证数据的持久性，支持数据的持久化（笔者注：有快照和AOF日志两种持久化方式，在实际应用的时候，要特别注意配置文件快照参数，要不就很有可能服务器频繁满载做dump）。

2. 数据支持类型：

   redis在数据支持上要比memecache多的多。

3. 使用底层模型不同：

   新版本的redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

4. 运行环境不同：

   redis目前官方只支持LINUX 上去行，从而省去了对于其它系统的支持，这样的话可以更好的把精力用于本系统 环境上的优化，虽然后来微软有一个小组为其写了补丁。但是没有放到主干上

个人总结一下，有持久化需求或者对数据结构和处理有高级要求的应用，选择redis，其他简单的key/value存储，选择memcache。
