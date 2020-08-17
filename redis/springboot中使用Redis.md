## 前言
本篇博客整理在springboot中如何使用Redis,并记录常用方法

## 1. 在pom文件中添加依赖
```xml
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
```
## 2. 在application.properties中配置参数
```
##端口号
server.port=8088

# Redis数据库索引（默认为0）
spring.redis.database=0 
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379 
# Redis服务器连接密码（默认为空）
spring.redis.password=123456
#连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8 
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1 
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8 
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0 
# 连接超时时间（毫秒）
spring.redis.timeout=300
```
## 3. redis的配置类
```java
package com.dist.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.*;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * redis配置类
 */
@Configuration
@EnableCaching //开启注解
public class RedisConfig extends CachingConfigurerSupport {

    /**
     * retemplate相关配置
     * @param factory
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 配置连接工厂
        template.setConnectionFactory(factory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer jacksonSeial = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper om = new ObjectMapper();
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jacksonSeial.setObjectMapper(om);

        // 值采用json序列化
        template.setValueSerializer(jacksonSeial);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());

        // 设置hash key 和value序列化模式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jacksonSeial);
        template.afterPropertiesSet();

        return template;
    }

    /**
     * 对hash类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public HashOperations<String, String, Object> hashOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForHash();
    }

    /**
     * 对redis字符串类型数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForValue();
    }

    /**
     * 对链表类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForList();
    }

    /**
     * 对无序集合类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForSet();
    }

    /**
     * 对有序集合类型的数据操作
     *
     * @param redisTemplate
     * @return
     */
    @Bean
    public ZSetOperations<String, Object> zSetOperations(RedisTemplate<String, Object> redisTemplate) {
        return redisTemplate.opsForZSet();
    }

}

```
## 4.RedisTeamplate常用方法
### 4.1 String 类型
```java
// 判断是否有key所对应的值，有则返回true，没有则返回false
redisTemplate.hasKey(key)

// 取出key值所对应的值
redisTemplate.opsForValue().get(key)

// 删除单个key值
redisTemplate.delete(key)

// 批量删除key 其中keys:Collection<K> keys
redisTemplate.delete(keys)

// 将当前传入的key值序列化为byte[]类型
redisTemplate.dump(key)

// 设置过期时间
redisTemplate.expire(key, timeout, unit)
redisTemplate.expireAt(key, date)

// 从redis中随机取出一个key
redisTemplate.randomKey()

// 设置当前的key以及value值
redisTemplate.opsForValue().set(key, value)

// 设置当前的key以及value值并且设置过期时间
redisTemplate.opsForValue().set(key, value, timeout, unit)

// 通过increment(K key, long delta)方法以增量方式存储long值（正值则自增，负值则自减）
redisTemplate.opsForValue().increment(key, increment)

// 获取字符串的长度
redisTemplate.opsForValue().size(key)

// 用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始
redisTemplate.opsForValue().set(key, value, offset)

// 重新设置key对应的值，如果存在返回false，否则返回true
redisTemplate.opsForValue().setIfAbsent(key, value)

// 将值 value 关联到 key,并将 key 的过期时间设为 timeout
redisTemplate.opsForValue().set(key, value, timeout, unit)

// 将二进制第offset位值变为value
redisTemplate.opsForValue().setBit(key, offset, value)

// 对key所储存的字符串值，获取指定偏移量上的位(bit)
redisTemplate.opsForValue().getBit(key, offset)
```

### 4.2 Hash类型
```java

// 获取变量中的指定map键是否有值,如果存在该map键则获取值，没有则返回null。
redisTemplate.opsForHash().get(key, field)

// 获取变量中的键值对
public Map<Object, Object> hGetAll(String key) {
    return redisTemplate.opsForHash().entries(key);
}

// 新增hashMap值
redisTemplate.opsForHash().put(key, hashKey, value)

// 以map集合的形式添加键值对
public void hPutAll(String key, Map<String, String> maps) {
    redisTemplate.opsForHash().putAll(key, maps);
}

// 仅当hashKey不存在时才设置
public Boolean hashPutIfAbsent(String key, String hashKey, String value) {
    return redisTemplate.opsForHash().putIfAbsent(key, hashKey, value);
}

// 删除一个或者多个hash表字段
public Long hashDelete(String key, Object... fields) {
    return redisTemplate.opsForHash().delete(key, fields);
}

// 查看hash表中指定字段是否存在
public boolean hashExists(String key, String field) {
    return redisTemplate.opsForHash().hasKey(key, field);
}

// 给哈希表key中的指定字段的整数值加上增量increment
public Long hashIncrBy(String key, Object field, long increment) {
    return redisTemplate.opsForHash().increment(key, field, increment);
}
public Double hIncrByDouble(String key, Object field, double delta) {
    return redisTemplate.opsForHash().increment(key, field, delta);
}

// 获取所有hash表中字段
redisTemplate.opsForHash().keys(key)

// 获取hash表中字段的数量
redisTemplate.opsForHash().size(key)

// 获取hash表中存在的所有的值
public List<Object> hValues(String key) {
    return redisTemplate.opsForHash().values(key);
}

// 匹配获取键值对，ScanOptions.NONE为获取全部键对
public Cursor<Entry<Object, Object>> hashScan(String key, ScanOptions options) {
    return redisTemplate.opsForHash().scan(key, options);
}
```
### 4.3 List类型
```java
// 通过索引获取列表中的元素
redisTemplate.opsForList().index(key, index)

// 获取列表指定范围内的元素(start开始位置, 0是开始位置，end 结束位置, -1返回所有)
redisTemplate.opsForList().range(key, start, end)

// 存储在list的头部，即添加一个就把它放在最前面的索引处
redisTemplate.opsForList().leftPush(key, value)

// 把多个值存入List中(value可以是多个值，也可以是一个Collection value)
redisTemplate.opsForList().leftPushAll(key, value)

// List存在的时候再加入
redisTemplate.opsForList().leftPushIfPresent(key, value)

// 如果pivot处值存在则在pivot前面添加
redisTemplate.opsForList().leftPush(key, pivot, value)

// 按照先进先出的顺序来添加(value可以是多个值，或者是Collection var2)
redisTemplate.opsForList().rightPush(key, value)
redisTemplate.opsForList().rightPushAll(key, value)

// 在pivot元素的右边添加值
redisTemplate.opsForList().rightPush(key, pivot, value)

// 设置指定索引处元素的值
redisTemplate.opsForList().set(key, index, value)

// 移除并获取列表中第一个元素(如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止)
redisTemplate.opsForList().leftPop(key)
redisTemplate.opsForList().leftPop(key, timeout, unit)

// 移除并获取列表最后一个元素
redisTemplate.opsForList().rightPop(key)
redisTemplate.opsForList().rightPop(key, timeout, unit)

// 从一个队列的右边弹出一个元素并将这个元素放入另一个指定队列的最左边
redisTemplate.opsForList().rightPopAndLeftPush(sourceKey, destinationKey)
redisTemplate.opsForList().rightPopAndLeftPush(sourceKey, destinationKey, timeout, unit)

// 删除集合中值等于value的元素(index=0, 删除所有值等于value的元素; index>0, 从头部开始删除第一个值等于value的元素; index<0, 从尾部开始删除第一个值等于value的元素)
redisTemplate.opsForList().remove(key, index, value)

// 将List列表进行剪裁
redisTemplate.opsForList().trim(key, start, end)

// 获取当前key的List列表长度
redisTemplate.opsForList().size(key)
```
### 4.4 Set 类型
```java
// 添加元素
redisTemplate.opsForSet().add(key, values)

// 移除元素(单个值、多个值)
redisTemplate.opsForSet().remove(key, values)

// 删除并且返回一个随机的元素
redisTemplate.opsForSet().pop(key)

// 获取集合的大小
redisTemplate.opsForSet().size(key)

// 判断集合是否包含value
redisTemplate.opsForSet().isMember(key, value)

// 获取两个集合的交集(key对应的无序集合与otherKey对应的无序集合求交集)
redisTemplate.opsForSet().intersect(key, otherKey)

// 获取多个集合的交集(Collection var2)
redisTemplate.opsForSet().intersect(key, otherKeys)

// key集合与otherKey集合的交集存储到destKey集合中(其中otherKey可以为单个值或者集合)
redisTemplate.opsForSet().intersectAndStore(key, otherKey, destKey)

// key集合与多个集合的交集存储到destKey无序集合中
redisTemplate.opsForSet().intersectAndStore(key, otherKeys, destKey)

// 获取两个或者多个集合的并集(otherKeys可以为单个值或者是集合)
redisTemplate.opsForSet().union(key, otherKeys)

// key集合与otherKey集合的并集存储到destKey中(otherKeys可以为单个值或者是集合)
redisTemplate.opsForSet().unionAndStore(key, otherKey, destKey)

// 获取两个或者多个集合的差集(otherKeys可以为单个值或者是集合)
redisTemplate.opsForSet().difference(key, otherKeys)

// 差集存储到destKey中(otherKeys可以为单个值或者集合)
redisTemplate.opsForSet().differenceAndStore(key, otherKey, destKey)

// 随机获取集合中的一个元素
redisTemplate.opsForSet().randomMember(key)

// 获取集合中的所有元素
redisTemplate.opsForSet().members(key)

// 随机获取集合中count个元素
redisTemplate.opsForSet().randomMembers(key, count)

// 获取多个key无序集合中的元素（去重），count表示个数
redisTemplate.opsForSet().distinctRandomMembers(key, count)

// 遍历set类似于Interator(ScanOptions.NONE为显示所有的)
redisTemplate.opsForSet().scan(key, options)
```

### 4.5 zSet类型
```java
// 添加元素(有序集合是按照元素的score值由小到大进行排列)
redisTemplate.opsForZSet().add(key, value, score)

// 删除对应的value,value可以为多个值
redisTemplate.opsForZSet().remove(key, values)

// 增加元素的score值，并返回增加后的值
redisTemplate.opsForZSet().incrementScore(key, value, delta)

// 返回元素在集合的排名,有序集合是按照元素的score值由小到大排列
redisTemplate.opsForZSet().rank(key, value)

// 返回元素在集合的排名,按元素的score值由大到小排列
redisTemplate.opsForZSet().reverseRank(key, value)

// 获取集合中给定区间的元素(start 开始位置，end 结束位置, -1查询所有)
redisTemplate.opsForZSet().reverseRangeWithScores(key, start,end)

// 按照Score值查询集合中的元素，结果从小到大排序
redisTemplate.opsForZSet().reverseRangeByScore(key, min, max)
redisTemplate.opsForZSet().reverseRangeByScoreWithScores(key, min, max)

// 从高到低的排序集中获取分数在最小和最大值之间的元素
redisTemplate.opsForZSet().reverseRangeByScore(key, min, max, start, end)

// 根据score值获取集合元素数量
redisTemplate.opsForZSet().count(key, min, max)

// 获取集合的大小
redisTemplate.opsForZSet().size(key)
redisTemplate.opsForZSet().zCard(key)

// 获取集合中key、value元素对应的score值
redisTemplate.opsForZSet().score(key, value)

// 移除指定索引位置处的成员
redisTemplate.opsForZSet().removeRange(key, start, end)

// 移除指定score范围的集合成员
redisTemplate.opsForZSet().removeRangeByScore(key, min, max)

// 获取key和otherKey的并集并存储在destKey中（其中otherKeys可以为单个字符串或者字符串集合）
redisTemplate.opsForZSet().unionAndStore(key, otherKey, destKey)

// 获取key和otherKey的交集并存储在destKey中（其中otherKeys可以为单个字符串或者字符串集合）
redisTemplate.opsForZSet().intersectAndStore(key, otherKey, destKey)
```
## 5. StringRedisTemplate和RedisTemplate区别
StringRedisTemplate 和 RedisTemplate 用法一致，两者之间的区别主要在于他们使用的序列化类不同：
```
RedisTemplate使用的是 JdkSerializationRedisSerializer
StringRedisTemplate使用的是 StringRedisSerializer

```
RedisTemplate使用的序列类在在操作数据的时候，比如说存入数据会将数据先序列化成字节数组然后在存入Redis数据库，这个时候打开Redis查看的时候，你会看到你的数据不是以可读的形式展现的，而是以字节数组显示。
StringRedisTemplate顾名思义，存入的数据是字符串，取出时也是字符串的形式。
**总结**： 
- 当你的redis数据库里面本来存的是字符串数据或者你要存取的数据就是字符串类型数据的时候，那么你就使用StringRedisTemplate即可。
- 如果你的数据是复杂的对象类型，而取出的时候又不想做任何的数据转换，直接从Redis里面取出一个对象，那么使用RedisTemplate是更好的选择。