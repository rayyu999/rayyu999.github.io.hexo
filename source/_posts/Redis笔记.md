---
title: Redis笔记
date: 2020-11-17 19:19:55
categories: Coding
tags: [Redis, Java, Spring Boot]
---

Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI [C语言](https://baike.baidu.com/item/C语言)编写、支持网络、可基于内存亦可持久化的日志型、Key-Value[数据库](https://baike.baidu.com/item/数据库/103728)，并提供多种语言的API。

<!--more-->



# Redis简介

在实际的开发过程中，多多少少都会涉及到缓存，而 Redis 通常来说是我们分布式缓存的最佳选择。Redis 也是我们熟知的 NoSQL（非关系性数据库）之一，虽然其不能完全的替代关系性数据库，但它可作为其良好的补充。

Redis 是一个开源（BSD 许可）、内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合等数据类型。内置复制、Lua 脚本、LRU 收回、事务以及不同级别磁盘持久化功能，同时通过 Redis Sentinel 提供高可用，通过 Redis Cluster 提供自动分区。



## Redis使用场景

微服务以及分布式被广泛使用后，Redis 的使用场景就越来越多了，下面罗列几种主要的场景：

1. **分布式缓存**：在分布式的系统架构中，将缓存存储在内存中显然不当，因为缓存需要与其他机器共享，这时 Redis 便挺身而出了，缓存也是 Redis 使用最多的场景。
2. **分布式锁**：在高并发的情况下，我们需要一个锁来防止并发带来的脏数据，Java 自带的锁机制显然对进程间的并发并不好使，此时可以利用 Redis 单线程的特性来实现我们的分布式锁，如何实现，可以[参考这篇文章](https://www.ibm.com/developerworks/cn/java/j-spring-boot-aop-web-log-processing-and-distributed-locking/index.html)。
3. **Session 存储/共享**：Redis 可以将 Session 持久化到存储中，这样可以避免由于机器宕机而丢失用户会话信息。
4. **发布/订阅**：Redis 还有一个发布/订阅的功能，您可以设定对某一个 key 值进行消息发布及消息订阅，当一个 key 值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统。
5. **任务队列**：Redis 的 `lpush+brpop` 命令组合即可实现阻塞队列，生产者客户端使用 `lrpush` 从列表左侧插入元素，多个消费者客户端使用 `brpop` 命令阻塞式的”抢”列表尾部的元素，多个客户端保证了消费的负载均衡和高可用性。
6. **限速，接口访问频率限制**：比如发送短信验证码的接口，通常为了防止别人恶意频刷，会限制用户每分钟获取验证码的频率，例如一分钟不能超过 5 次。

当然 Redis 的使用场景并不仅仅只有这么多，还有很多未列出的场景，如计数、排行榜等，可见 Redis 的强大。不过 Redis 说到底还是一个数据库（非关系型），那么我们还是有必要了解一下它支持存储的数据结构。



## Redis数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。



### String（字符串）

string 是 redis 最基本的类型，一个 key 对应一个 value。

string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

#### 实例

```shell
redis 127.0.0.1:6379> SET ray "靓仔"
OK
redis 127.0.0.1:6379> GET ray
"靓仔"
```

在 Redis 中 string 表示的是一个可变的字节数组，我们初始化字符串的内容、可以拿到字符串的长度，可以获取 string 的子串，可以覆盖 string 的子串内容，可以追加子串。

![](http://images.yingwai.top/picgo/image001.png)

如上图所示，在 Redis 中我们初始化一个字符串时，会采用预分配冗余空间的方式来减少内存的频繁分配，如图所示，实际分配的空间 capacity 一般要高于实际字符串长度 len。如果你看过 Java 的 ArrayList 的源码相信会对这种模式很熟悉。



### Hash（哈希）

hash 是一个键值(key=>value)对集合。

hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

#### 实例

`DEL ray` 用于删除前面测试用过的 key，否则会报错：**(error) WRONGTYPE Operation against a key holding the wrong kind of value**

```shell
redis 127.0.0.1:6379> DEL ray
redis 127.0.0.1:6379> HMSET ray field1 "Hello" field2 "World"
"OK"
redis 127.0.0.1:6379> HGET ray field1
"Hello"
redis 127.0.0.1:6379> HGET ray field2
"World"
```

`HMSET` 设置了两个 `field=>value` 对，`HGET` 获取对应 `field` 对应的 `value`。

每个 hash 可以存储 $2^{32} - 1$ 个键值对（4294967295，40多亿）。

hash 与 Java 中的 HashMap 差不多，实现上采用二维结构，第一维是数组，第二维是链表。hash 的 key 与 value 都存储在链表中，而数组中存储的则是各个链表的表头。在检索时，首先计算 key 的 hashcode，然后通过 hashcode 定位到链表的表头，再遍历链表得到 value 值。可能你比较好奇为啥要用链表来存储 key 和 value，直接用 key 和 value 一对一存储不就可以了吗？其实是因为有些时候我们无法保证 hashcode 值的唯一，若两个不同的 key 产生了相同的 hashcode，我们需要一个链表在存储两对键值对，这就是所谓的 hash 碰撞。



### List（列表）

列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

#### 实例

```shell
redis 127.0.0.1:6379> DEL ray
redis 127.0.0.1:6379> lpush ray redis
(integer) 1
redis 127.0.0.1:6379> lpush ray mongodb
(integer) 2
redis 127.0.0.1:6379> lpush ray rabitmq
(integer) 3
redis 127.0.0.1:6379> lrange ray 0 10
1) "rabitmq"
2) "mongodb"
3) "redis"
```

列表最多可以存储 $2^{32} - 1$ 个元素。

在 Redis 中列表 list 采用的存储结构是双向链表，由此可见其随机定位性能较差，比较适合首位插入删除。像 Java 中的数组一样，Redis 中的列表支持通过下标访问，不同的是 Redis 还为列表提供了一种负下标，`-1` 表示倒数一个元素，`-2` 表示倒数第二个数，依此类推。综合列表首尾增删性能优异的特点，通常我们使用 `rpush/rpop/lpush/lpop` 四条指令将列表作为队列来使用。

![](http://images.yingwai.top/picgo/image002.png)

如上图所示，在列表元素较少的情况下会使用一块连续的内存存储，这个结构是 ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成 quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是 int 类型的数据，结构上还需要两个额外的指针 prev 和 next。所以 Redis 将链表和 ziplist 结合起来组成了 quicklist。也就是将多个 ziplist 使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。



### Set（集合）

Set 是 string 类型的无序集合。

集合是通过哈希表实现的，所有的 value 都指向同一个内部值，所以添加，删除，查找的复杂度都是 O(1)。

#### sadd命令

添加一个 string 元素到 key 对应的 set 集合中，成功返回 1，如果元素已经在集合中返回 0。

```shell
sadd key member
```

#### 实例

```shell
redis 127.0.0.1:6379> DEL ray
redis 127.0.0.1:6379> sadd ray redis
(integer) 1
redis 127.0.0.1:6379> sadd ray mongodb
(integer) 1
redis 127.0.0.1:6379> sadd ray rabitmq
(integer) 1
redis 127.0.0.1:6379> sadd ray rabitmq
(integer) 0
redis 127.0.0.1:6379> smembers ray

1) "redis"
2) "rabitmq"
3) "mongodb"
```

以上实例中 rabitmq 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。

集合中最多可以存储 $2^{32} - 1$ 个成员。



### zset（sorted set：有序集合）

zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

#### zadd命令

添加元素到集合，元素在集合中存在则更新对应score

```shell
zadd key score member
```

#### 实例

```shell
redis 127.0.0.1:6379> DEL ray
redis 127.0.0.1:6379> zadd ray 0 redis
(integer) 1
redis 127.0.0.1:6379> zadd ray 0 mongodb
(integer) 1
redis 127.0.0.1:6379> zadd ray 0 rabitmq
(integer) 1
redis 127.0.0.1:6379> zadd ray 0 rabitmq
(integer) 0
redis 127.0.0.1:6379> > ZRANGEBYSCORE ray 0 1000
1) "mongodb"
2) "rabitmq"
3) "redis"
```



### 总结

| 类型       | 简介                                                    | 特性                                                         | 场景                                                         |
| ---------- | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String     | 二进制安全                                              | 可以包含任何数据，比如jpg图片或者序列化的对象，一个键最大能存储512M | -                                                            |
| Hash       | 键值对集合，即编程语言中的Map类型                       | 适合存储对象，并且可以像数据库中update一个属性一样只修改某一项属性值 | 存储、读取、修改用户属性                                     |
| List       | （双向）链表                                            | 增删快，提供了操作某一段元素的API                            | 1. 最新消息排行等功能（比如朋友圈的时间线）<br />2. 消息队列 |
| Set        | 哈希表实现，元素不重复                                  | 1. 添加、删除、查找的复杂度都是 $O(1)$<br />2. 为集合提供了求交集、并集、差集等操作 | 1. 共同好友<br />2. 利用唯一性，统计访问网站的所有独立ip<br />3. 好友推荐时，根据tag求交集，大于某个阈值就可以推荐 |
| Sorted Set | 将Set中的元素增加一个权重参数score，元素按score有序排列 | 数据插入集合时，已经进行天然排序                             | 1. 排行榜<br />2. 带权重的消息队列                           |



# Java使用Redis



## 安装

开始在 Java 中使用 Redis 前， 我们需要确保已经安装了 redis 服务及 Java redis 驱动，且你的机器上能正常使用 Java。

- 首先你需要下载驱动包 [**下载 jedis.jar**](https://mvnrepository.com/artifact/redis.clients/jedis)，确保下载最新驱动包。
- 在你的 classpath 中包含该驱动包。



## 连接到Redis服务

```java
import redis.clients.jedis.Jedis;

public class RedisJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        // 如果 Redis 服务设置来密码，需要下面这行，没有就不需要
        // jedis.auth("123456"); 
        System.out.println("连接成功");
        //查看服务是否运行
        System.out.println("服务正在运行: "+jedis.ping());
    }
}
```

运行结果：

```shell
连接成功
服务正在运行：PONG
```



## Redis Java String（字符串）实例

```java
import redis.clients.jedis.Jedis;
 
public class RedisStringJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //设置 redis 字符串数据
        jedis.set("raykey", "yuyingwai.cn");
        // 获取存储的数据并输出
        System.out.println("redis 存储的字符串为: "+ jedis.get("raykey"));
    }
}
```

运行结果：

```shell
连接成功
redis 存储的字符串为：yuyingwai.cn
```



## Redis Java List（列表）实例

```java
import java.util.List;
import redis.clients.jedis.Jedis;
 
public class RedisListJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
        //存储数据到列表中
        jedis.lpush("site-list", "Ray");
        jedis.lpush("site-list", "Google");
        jedis.lpush("site-list", "Tencent");
        // 获取存储的数据并输出
        List<String> list = jedis.lrange("site-list", 0 ,2);
        for(int i=0; i<list.size(); i++) {
            System.out.println("列表项为: "+list.get(i));
        }
    }
}
```

运行结果：

```shell
连接成功
列表项为：Tencent
列表项为：Google
列表项为：Ray
```



## Redis Java Keys实例

```java
import java.util.Iterator;
import java.util.Set;
import redis.clients.jedis.Jedis;
 
public class RedisKeyJava {
    public static void main(String[] args) {
        //连接本地的 Redis 服务
        Jedis jedis = new Jedis("localhost");
        System.out.println("连接成功");
 
        // 获取数据并输出
        Set<String> keys = jedis.keys("*"); 
        Iterator<String> it=keys.iterator() ;   
        while(it.hasNext()){   
            String key = it.next();   
            System.out.println(key);   
        }
    }
}
```

运行结果：

```shell
连接成功
raykey
site-list
```



# 在 Spring Boot 项目中使用 Redis



## 添加Redis依赖

```xml
<!--SpringBoot 的 Redis 支持-->
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

添加完依赖之后，我们还需要配置 Redis 的地址等信息才能使用，在 `application.yml` 中添加如下配置即可。

```yaml
spring:
  redis:
  	host: localhost
  	port: 6379
  	# Redis 数据库索引（默认为0）
  	database: 0
  	# Redis 服务器连接密码（默认为空）
  	password: 
  	# 连接超时时间，单位毫秒（默认0永不超时）
  	timeout: 2000
  	# 连接池配置
  	jedis:
  	  pool:
  	  	# 最大连接数（使用负值表示没有限制）
  	  	max-active: 8
  	  	# 最大阻塞等待时间（使用负值表示没有限制）
  	  	max-wait: -1
  	  	# 最大空闲连接
  	  	max-idle: 9
  	  	# 最小空闲连接
  	  	min-idle: 0
```

Spring Boot 的 `spring-boot-starter-data-redis` 为 Redis 的相关操作提供了一个高度封装的 RedisTemplate 类，而且对每种类型的数据结构都进行了归类，将同一类型操作封装为 operation 接口。`RedisTemplate` 对五种数据结构分别定义了操作，如下所示：

* 操作字符串：`redisTemplate.opsForValue()`
* 操作 Hash：`redisTemplate.opsForHash()`
* 操作 List：`redisTemplate.opsForList()`
* 操作 Set：`redisTemplate.opsForSet()`
* 操作 ZSet：`redisTemplate.opsForZSet()`

但是对于 string 类型的数据，Spring Boot 还专门提供了 `StringRedisTemplate` 类，而且官方也建议使用该类来操作 String 类型的数据。那么它和 `RedisTemplate` 又有啥区别呢？

1. `RedisTemplate` 是一个泛型类，而 `StringRedisTemplate` 不是，后者只能对键和值都为 String 类型的数据进行操作，而前者则可以操作任何类型。
2. 两者的数据是不共通的，`StringRedisTemplate` 只能管理 `StringRedisTemplate` 里面的数据，`RedisTemplate` 只能管理 `RedisTemplate` 中的数据。



## RedisTemplate 的配置

一个 Spring Boot 项目中，我们只需要维护一个 `RedisTemplate` 对象和一个 `StringRedisTemplate` 对象就可以了。所以我们需要通过一个 `Configuration` 类来初始化这两个对象并且交由的 `BeanFactory` 管理。我们在 `cn.yuyingwai.config` 包下面新建了一个 `RedisConfig` 类，其内容如下所示：

```java
@Configuration
public class RedisConfig {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory redisConnectionFactory) {

        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(jackson2JsonRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    @ConditionalOnMissingBean(StringRedisTemplate.class)
    public StringRedisTemplate stringRedisTemplate(
            RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```



## 操作字符串

`StringRedisTempalte` 在上面已经初始化好了，我们只需要在需要用到的地方通过 `@AutoWired` 注解注入就行。

1. 设置值，对于设置值，可以使用 `opsForValue().void set(K var1, V var2)`;

   ```java
   @Test
   public void testSet() {
           stringRedisTemplate.opsForValue().set("test-string-value", "Hello Redis");
   }
   ```

2. 获取值，与 set 方法相对于 StringRedisTemplate 还提供了 `.opsForValue().get(Object var1)` 方法来获取指定 key 对应的 value 值。

   ```java
   @Test
   public void testGet() {
       String value = stringRedisTemplate.opsForValue().get("test-string-value");
       System.out.println(value);
   }
   ```

3. 设置值的时候设置过期时间。在设置缓存的时候，我们通常都会给他设置一个过期时间，让其能够达到定时刷新的效果。`StringRedisTemplate` 提供了 `void set(K var1, V var2, long var3, TimeUnit var5)` 方法来达到设置过期时间的目的，其中 `var3` 这个参数就是过期时间的数值，而 `TimeUnit` 是个枚举类型，我们用它来设置过期时间的单位，是小时或是秒等等。

   ```java
   @Test
   public void testSetTimeOut() {
           stringRedisTemplate.opsForValue().set("test-string-key-time-out", "Hello Redis", 3, TimeUnit.HOURS);
   }
   ```

   如上面代码所示，我们保存了一个 key 为 test-string-key-time-out 的 String 类型的数据，而这条数据将会在 3 个小时之后被自动删除（失效）。

4. 删除数据，我们同样可以通过 StringRedisTmeplate 来删除数据， `Boolean delete(K key)`方法提供了这个功能。

   ```java
   @Test
   public void testDeleted() {
           stringRedisTemplate.delete("test-string-value");
   }
   ```



## 操作数组

在 Redis 数据类型小节中，我们提到过我们经常使用 Redis 的 lpush/rpush/lpop/rpop 四条指令来实现一个队列。那么这四条指令在 RedisTemplate 中也有相应的实现。

1. `leftPush(K key, V value)`，往 List 左侧插入一个元素，如 从左边往数组中 push 元素：

   ```java
   @Test
    public void testLeftPush() {
           redisTemplate.opsForList().leftPush("TestList", "TestLeftPush");
    }
   ```

2. `rightPush(K key, V value)`，往 List 右侧插入一个元素， 如从右边往数组中 push 元素：

   ```java
   @Test
    public void testRightPush() {
           redisTemplate.opsForList().rightPush("TestList", "TestRightPush");
    }
   ```

   执行完上面两个 Test 之后，我们可以使用 Redis 客户端工具 RedisDesktopManager 来查看 TestList 中的内容，如下图 （Push 之后 TestList 中的内容）所示：

   ![](http://images.yingwai.top/picgo/image003.png)

   此时我们再一次执行 leftPush 方法，TestList 的内容就会变成下图（第二次执行 leftPush 之后的内容）所示：

   ![](http://images.yingwai.top/picgo/20201117153422.png)

   可以看到 `leftPush` 实际上是往数组的头部新增一个元素，那么 `rightPush` 就是往数组尾部插入一个元素。

3. `leftPop(K key)`，从 List 左侧取出第一个元素，并移除， 如从数组头部获取并移除值：

   ```java
   @Test
   public void testLeftPop() {
           Object leftFirstElement = redisTemplate.opsForList().leftPop("TestList");
           System.out.println(leftFirstElement);
   }
   ```

   执行上面的代码之后，您会看到控制台会打印出 `TestLeftPush`，然后再去 `RedisDesktopManager` 中查看 TestList 的内容，如下图 （同数组顶端移除一个元素后）所示。您会发现数组中的第一个元素已经被移除了。

   ![](http://images.yingwai.top/picgo/20201117153525.png)

4. `rightPop(K key)`，从 List 右侧取出第一个元素，并移除， 如从数组尾部获取并移除值：

   ```java
   @Test
    public void testRightPop() {
           Object rightFirstElement = redisTemplate.opsForList().rightPop("TestList");
           System.out.println(rightFirstElement);
    }
   ```



## 操作Hash

Redis 中的 Hash 数据结构实际上与 Java 中的 HashMap 是非常类似的，提供的 API 也很类似。下面我们就一起来看下 `RedisTemplate` 为 Hash 提供了哪些 API。

1. Hash 中新增元素。

   ```java
   @Test
   public void testPut() {
           redisTemplate.opsForHash().put("TestHash", "FirstElement", "Hello,Redis hash.");
           Assert.assertTrue(redisTemplate.opsForHash().hasKey("TestHash", "FirstElement"));
   }
   ```

2. 判断指定 key 对应的 Hash 中是否存在指定的 map 键，使用用法可以见上方代码所示。

3. 获取指定 key 对应的 Hash 中指定键的值。

   ```java
   @Test
   public void testGet() {
           Object element = redisTemplate.opsForHash().get("TestHash", "FirstElement");
           Assert.assertEquals("Hello,Redis hash.", element);
   }
   ```

4. 删除指定 key 对应 Hash 中指定键的键值对。

   ```java
   @Test
   public void testDel() {
           redisTemplate.opsForHash().delete("TestHash", "FirstElement");
           Assert.assertFalse(redisTemplate.opsForHash().hasKey("TestHash", "FirstElement"));
   }
   ```



## 操作集合

集合很类似于 Java 中的 Set，RedisTemplate 也为其提供了丰富的 API。

1. 向集合中添加元素。

   ```java
   @Test
   public void testAdd() {
           redisTemplate.opsForSet().add("TestSet", "e1", "e2", "e3");
           long size = redisTemplate.opsForSet().size("TestSet");
           Assert.assertEquals(3L, size);
   }
   ```

2. 获取集合中的元素。

   ```java
   @Test
   public void testGet() {
           Set<String> testSet = redisTemplate.opsForSet().members("TestSet");
           System.out.println(testSet);
   }
   ```

   执行上面的代码后，控制台输出的是[e1, e3, e2]，当然你可能会看到其他结果，因为 Set 是无序的，并不是按照我们添加的顺序来排序的。

3. 获取集合的长度，在向集合中添加元素的示例代码中展示了如何获取集合长度。

4. 移除集合中的元素

   ```java
   @Test
   public void testRemove() {
           redisTemplate.opsForSet().remove("TestSet", "e1", "e2");
           Set testSet = redisTemplate.opsForSet().members("TestSet");
           Assert.assertEquals("e3", testSet.toArray()[0]);
   }
   ```



## 操作有序集合

与 Set 不一样的地方是，ZSet 对于集合中的每个元素都维护了一个权重值，那么 RedisTemplate 提供了不少与这个权重值相关的 API。

| API                                                          | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `add(K key, V value, double score)`                          | 添加元素到变量中同时指定元素的分值。                         |
| `range(K key, long start, long end)`                         | 获取变量指定区间的元素。                                     |
| `rangeByLex(K key, RedisZSetCommands.Range range)`           | 用于获取满足非 `score` 的排序取值。这个排序只有在有相同分数的情况下才能使用，如果有不同的分数则返回值不确定。 |
| `angeByLex(K key, RedisZSetCommands.Range range, RedisZSetCommands.Limit limit)` | 用于获取满足非 `score` 的设置下标开始的长度排序取值。        |
| `add(K key, Set<ZSetOperations.TypedTuple<V>> tuples)`       | 通过 `TypedTuple` 方式新增数据。                             |
| `rangeByScore(K key, double min, double max)`                | 根据设置的 `score` 获取区间值。                              |
| `rangeByScore(K key, double min, double max,long offset, long count)` | 根据设置的 `score` 获取区间值从给定下标和给定长度获取最终值。 |
| `rangeWithScores(K key, long start, long end)`               | 获取 `RedisZSetCommands.Tuples` 的区间值。                   |

以上只是简单的介绍了一些最常用的 API，`RedisTemplate` 针对字符串、数组、Hash、集合以及有序集合还提供了很多 API，具体有哪些 API，可以参考 [RedisTemplate 提供的 API 列表](https://blog.csdn.net/sinat_27629035/article/details/102652185) 这篇文章。



## 实现分布式锁

上面基本列出了 `RedisTemplate` 和 `StringRedisTemplate` 两个类所提供的对 Redis 操作的相关 API，但是有些时候这些 API 并不能完成我们所有的需求，这个时候我们其实还可以在 Spring Boot 项目中直接与 Redis 交互来完成操作。比如，我们在实现分布式锁的时候其实就是使用了 `RedisTemplate` 的 `execute` 方法来执行 Lua 脚本来获取和释放锁的。

* 获取锁：

  ```java
  Boolean lockStat = stringRedisTemplate.execute((RedisCallback<Boolean>)connection ->
                      connection.set(key.getBytes(Charset.forName("UTF-8")), value.getBytes(Charset.forName("UTF-8")),
                              Expiration.from(timeout, timeUnit), RedisStringCommands.SetOption.SET_IF_ABSENT));
  ```

* 释放锁：

  ```java
  String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
              boolean unLockStat = stringRedisTemplate.execute((RedisCallback<Boolean>)connection ->
                      connection.eval(script.getBytes(), ReturnType.BOOLEAN, 1,
                              key.getBytes(Charset.forName("UTF-8")), value.getBytes(Charset.forName("UTF-8"))));
  ```



# 关于Redis的几个经典问题

## 缓存与数据库一致性问题

对于既有数据库操作又有缓存操作的接口，一般分为两种执行顺序。

1. 先操作数据库，再操作缓存。这种情况下如果数据库操作成功，缓存操作失败就会导致缓存和数据库不一致。
2. 第二种情况就是先操作缓存再操作数据库，这种情况下如果缓存操作成功，数据库操作失败也会导致数据库和缓存不一致。

大部分情况下，我们的缓存理论上都是需要可以从数据库恢复出来的，所以基本上采取第一种顺序都是不会有问题的。针对那些必须保证数据库和缓存一致的情况，通常是不建议使用缓存的。



## 缓存击穿问题

缓存击穿表示恶意用户频繁的模拟请求缓存中不存在的数据，以致这些请求短时间内直接落在了数据库上，导致数据库性能急剧下降，最终影响服务整体的性能。这个在实际项目很容易遇到，如抢购活动、秒杀活动的接口 API 被大量的恶意用户刷，导致短时间内数据库宕机。对于缓存击穿的问题，有以下几种解决方案，这里只做简要说明。

1. 使用互斥锁排队。当从缓存中获取数据失败时，给当前接口加上锁，从数据库中加载完数据并写入后再释放锁。若其它线程获取锁失败，则等待一段时间后重试。
2. 使用布隆过滤器。将所有可能存在的数据缓存放到布隆过滤器中，当黑客访问不存在的缓存时迅速返回避免缓存及 DB 挂掉。



## 缓存雪崩问题

在短时间内有大量缓存失效，如果这期间有大量的请求发生同样也有可能导致数据库发生宕机。在 Redis 机群的数据分布算法上如果使用的是传统的 hash 取模算法，在增加或者移除 Redis 节点的时候就会出现大量的缓存临时失效的情形。

1. 像解决缓存穿透一样加锁排队。
2. 建立备份缓存，缓存 A 和缓存 B，A 设置超时时间，B 不设值超时时间，先从 A 读缓存，A 没有读 B，并且更新 A 缓存和 B 缓存。
3. 计算数据缓存节点的时候采用一致性 hash 算法，这样在节点数量发生改变时不会存在大量的缓存数据需要迁移的情况发生。



## 缓存并发问题

这里的并发指的是多个 Redis 的客户端同时 set 值引起的并发问题。比较有效的解决方案就是把 set 操作放在队列中使其串行化，必须得一个一个执行。