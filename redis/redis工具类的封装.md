## 前言
本文为对RedisTemplate方法的封装

## Redis键生成工具类
```java
/**
 * Key 生成规则:自定义前缀:项目名:模块名:自定义名称
 */
public class RedisKeyGeneratorUtil {
    /**
     * 第一部分:自定义前缀
     */
    private static final String PREFIX = "";
    
    /**
     * 第二部分:项目名称 CustomerConfig.projectName
     */
    private static final String PROJECT_NAME = "book";
    
    /**
     * 第三部分:模块名称 CustomerConfig.moduleName
     * 模块名称若有多个单词,则以中划线分隔
     * eg: book-manage
     */
    private static final String MODULE_NAME = "book-manage";
    
    /**
     * 第四部分:自定义名称 自定义名称可以有多个值 生成 Redis 键时自定义名称的值以 : 分隔
     */
     private static List<String> customerNames;
    
    /**
     * Redis 键分隔符
     */
    private static final String KEY_SPLIT_SEPARATOR = ":";
    
    /**
     * Redis 键生成器
     */
    public static String keyGenerator(String... keys) {
        customerNames = Arrays.asList(keys);
        String[] systemProperty = new String[] {PREFIX, PROJECT_NAME, MODULE_NAME};
        return Stream.of(systemProperty, keys)
                .flatMap(Arrays::stream)
                .filter(string -> !StrUtil.isBlankIfStr(string))
                .collect(Collectors.joining(KEY_SPLIT_SEPARATOR)).toString();
    }
    
    /**
     * Redis 键生成器
     * 
     */
    public static String moduleNameKeyGenerator(String moduleName, String... keys) {
        customerNames = Arrays.asList(keys);
        String[] systemProperty = new String[] {PREFIX, PROJECT_NAME, moduleName};
        return Stream.of(systemProperty, keys)
                .flatMap(Arrays::stream)
                .filter(string -> !StrUtil.isBlankIfStr(string))
                .collect(Collectors.joining(KEY_SPLIT_SEPARATOR)).toString();
    }
    public static String getPrefix() {
        return PREFIX;
    }

    public static String getProjectName() {
        return PROJECT_NAME;
    }

    public static String getModuleName() {
        return MODULE_NAME;
    }

    public static List<String> getCustomerNames() {
        return customerNames;
    }

    public static String getKeySplitSeparator() {
        return KEY_SPLIT_SEPARATOR;
    }
    
}
```

## redis 工具类
```java

/**
 * Redis 工具类
 */
public class RedisUtil {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    private Logger loggger = Logger.getLogger(RedisUtil.class);

    // ============================================common======================================================
    /**
     * 指定缓存失效时间
     *
     * @param key   键       
     * @param time  时间(秒)      
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(RedisKeyGeneratorUtil.keyGenerator(key), time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 根据key获取过期时间
     *
     * @param key   键 不能为null       
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(RedisKeyGeneratorUtil.keyGenerator(key), TimeUnit.SECONDS);
    }

    /**
     * 判断RedisKeyGeneratorUtil.keyGenerator(key)是否存在
     *
     * @param key   键      
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(RedisKeyGeneratorUtil.keyGenerator(key));
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key       可以传一个值 或多个
     */
    public void del(String... keys) {
        if (ArrayUtil.isNotEmpty(keys)) {
            if (keys.length == 1) {
                redisTemplate.delete(RedisKeyGeneratorUtil.keyGenerator(keys[0]));
            } else {
                redisTemplate.delete(Stream.of(keys).map(RedisKeyGeneratorUtil::keyGenerator).collect(Collectors.toList()));
            }
        }
    }

    // =======================================String=========================================================
    /**
     * 普通缓存获取
     *
     * @param key   键       
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(RedisKeyGeneratorUtil.keyGenerator(key));
    }

    /**
     * 普通缓存放入
     *
     * @param key   键       
     * @param value 值     
     * @return 
     *            true成功 
     *            false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(RedisKeyGeneratorUtil.keyGenerator(key), value);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }

    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key       键
     * @param value     值
     * @param time      时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(RedisKeyGeneratorUtil.keyGenerator(key), value, time, TimeUnit.SECONDS);
            } else {
                set(RedisKeyGeneratorUtil.keyGenerator(key), value);
            }
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 递增
     * 适用场景:
     * 高并发生成订单号，秒杀类的业务逻辑等。。
     *
     * @param key   键       
     * @param by    要增加几(大于0)      
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(RedisKeyGeneratorUtil.keyGenerator(key), delta);
    }

    /**
     * 递减
     *
     * @param key       键       
     * @param delta     要减少几(小于0)      
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(RedisKeyGeneratorUtil.keyGenerator(key), -delta);
    }

    // ================================Map=================================
    /**
     * HashGet
     *
     * @param key   键 不能为null       
     * @param item  项 不能为null      
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(RedisKeyGeneratorUtil.keyGenerator(key), item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key   键       
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(RedisKeyGeneratorUtil.keyGenerator(key));
    }

    /**
     * HashSet
     *
     * @param key   键       
     * @param map   对应多个键值     
     * @return 
     *          true 成功 
     *          false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(RedisKeyGeneratorUtil.keyGenerator(key), map);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key   键      
     * @param map   对应多个键值     
     * @param time  时间(秒)      
     * @return 
     *          true成功 
     *          false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(RedisKeyGeneratorUtil.keyGenerator(key), map);
            if (time > 0) {
                expire(RedisKeyGeneratorUtil.keyGenerator(key), time);
            }
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key       键
     * @param item      项
     * @param value     值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(RedisKeyGeneratorUtil.keyGenerator(key), item, value);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key       键
     * @param item      项
     * @param value     值
     * @param time      时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(RedisKeyGeneratorUtil.keyGenerator(key), item, value);
            if (time > 0) {
                expire(RedisKeyGeneratorUtil.keyGenerator(key), time);
            }
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key       键 不能为null
     * @param item      项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(RedisKeyGeneratorUtil.keyGenerator(key), item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key       键 不能为null
     * @param item      项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(RedisKeyGeneratorUtil.keyGenerator(key), item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key       键
     * @param item      项
     * @param by        要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(RedisKeyGeneratorUtil.keyGenerator(key), item, by);
    }

    /**
     * hash递减
     *
     * @param key       键
     * @param item      项
     * @param by        要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(RedisKeyGeneratorUtil.keyGenerator(key), item, -by);
    }

    // ============================set=============================
    /**
     * 根据RedisKeyGeneratorUtil.keyGenerator(key)获取Set中的所有值
     *
     * @param key       键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(RedisKeyGeneratorUtil.keyGenerator(key));
        } catch (Exception e) {
            loggger.error(key, e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key       键
     * @param value     值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(RedisKeyGeneratorUtil.keyGenerator(key), value);
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key       键
     * @param values    值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(RedisKeyGeneratorUtil.keyGenerator(key), values);
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key       键
     * @param time      时间(秒)
     * @param values    值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(RedisKeyGeneratorUtil.keyGenerator(key), values);
            if (time > 0) expire(RedisKeyGeneratorUtil.keyGenerator(key), time);
            return count;
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key       键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(RedisKeyGeneratorUtil.keyGenerator(key));
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key       键
     * @param values    值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(RedisKeyGeneratorUtil.keyGenerator(key), values);
            return count;
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    // ============================zset=============================
    /**
     * 根据RedisKeyGeneratorUtil.keyGenerator(key)获取Set中的所有值
     *
     * @param key       键
     * @return
     */
    public Set<Object> zSGet(String key) {
        try {
            return redisTemplate.opsForSet().members(RedisKeyGeneratorUtil.keyGenerator(key));
        } catch (Exception e) {
            loggger.error(key, e);
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key       键
     * @param value     值
     * @return true 存在 false不存在
     */
    public boolean zSHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(RedisKeyGeneratorUtil.keyGenerator(key), value);
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }
    
    /**
         * 根据value放入缓存并设置分值
     *
     * @param key       键
     * @param value     值
     * @param score     得分
     * @return true 存在 false不存在
     */
    public Boolean zSSet(String key, Object value, double score) {
        try {
            return redisTemplate.opsForZSet().add(RedisKeyGeneratorUtil.keyGenerator(key), value, 2);
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key       键
     * @param time      时间(秒)
     * @param values    值 可以是多个
     * @return 成功个数
     */
    public long zSSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(RedisKeyGeneratorUtil.keyGenerator(key), values);
            if (time > 0) expire(RedisKeyGeneratorUtil.keyGenerator(key), time);
            return count;
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key       键
     * @return
     */
    public long zSGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(RedisKeyGeneratorUtil.keyGenerator(key));
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key       键
     * @param values        值 可以是多个
     * @return 移除的个数
     */
    public long zSetRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(RedisKeyGeneratorUtil.keyGenerator(key), values);
            return count;
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }
    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @取出来的元素 总数 end-start+1
     *
     * @param key       键
     * @param start     开始 0 是第一个元素
     * @param end       结束 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(RedisKeyGeneratorUtil.keyGenerator(key), start, end);
        } catch (Exception e) {
            loggger.error(key, e);
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key       键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(RedisKeyGeneratorUtil.keyGenerator(key));
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key       键
     * @param index     索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(RedisKeyGeneratorUtil.keyGenerator(key), index);
        } catch (Exception e) {
            loggger.error(key, e);
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key       键
     * @param value     值
     * @param time      时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(RedisKeyGeneratorUtil.keyGenerator(key), value);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key       键
     * @param value     值
     * @param time      时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(RedisKeyGeneratorUtil.keyGenerator(key), value);
            if (time > 0) expire(RedisKeyGeneratorUtil.keyGenerator(key), time);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key       键
     * @param value     值
     * @param time      时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(RedisKeyGeneratorUtil.keyGenerator(key), value);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key       键
     * @param value     值
     * @param time      时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(RedisKeyGeneratorUtil.keyGenerator(key), value);
            if (time > 0) expire(RedisKeyGeneratorUtil.keyGenerator(key), time);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key       键
     * @param index     索引
     * @param value     值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(RedisKeyGeneratorUtil.keyGenerator(key), index, value);
            return true;
        } catch (Exception e) {
            loggger.error(key, e);
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key       键
     * @param count     移除多少个
     * @param value     值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(RedisKeyGeneratorUtil.keyGenerator(key), count, value);
            return remove;
        } catch (Exception e) {
            loggger.error(key, e);
            return 0;
        }
    }

}

```