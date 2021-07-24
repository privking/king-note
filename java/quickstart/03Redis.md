# Redis

```java
@Configuration
public class AppRedisPoolConfig {

    @Value("${jedis.pool.max-active}")
    private int maxActive;

    @Value("${jedis.pool.max-idle}")
    private int maxIdle;

    @Value("${jedis.pool.min-idle}")
    private int minIdle;

    @Value("${jedis.pool.max-wait}")
    private int maxWait;

    @Value("${redis.host}")
    private String host;

    @Value("${redis.port}")
    private int port;

    @Value("${redis.password}")
    private String password;

    @Value("${redis.timeout}")
    private int timeOut;

    @Value("${redis.database}")
    private int database;

    @Bean
    public JedisPoolConfig jedisPoolConfig() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxWaitMillis(maxWait);
        poolConfig.setMinIdle(maxIdle);
        poolConfig.setMinIdle(minIdle);
        poolConfig.setMaxTotal(maxActive);
        return poolConfig;
    }

    @Bean
    public JedisPool jedisPool() {
        return new JedisPool(jedisPoolConfig(),host, port,timeOut, password, database);
    }
}
```

```java
@Component
public class ApRedisClient {
    /**
     * 日志
     */
    private final static Logger logger = LoggerFactory.getLogger(AppRedisClient.class);

    @Autowired
    private JedisPool jedisPool;

    public void set(String key, String value) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.set(key, value);
        } catch (Exception e) {
            logger.error("redis set 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public void sadd(String key, String[] value) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.sadd(key, value);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public Boolean sismember(String key, String value) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.sismember(key, value);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return false;
    }

    public Set<String> sinter (String[] keys) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.sinter(keys);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public Set<String> smembers (String key) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.smembers(key);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public Set<String> sunion (String[] keys) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.sunion(keys);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public void sunionstore (String key, String[] keys) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.sunionstore(key, keys);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public void hset(String key1, String key2, String value) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.hset(key1, key2, value);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public String get(String key) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.get(key);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public Long incr(String key) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.incr(key);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public String hget(String key1, String key2) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.hget(key1, key2);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public String hdel(String key1, String key2) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.hdel(key1, key2);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public Map<String, String> hgetAll(String key1) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.hgetAll(key1);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public Long hincrBy(String key1, String key2, Long value) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.hincrBy(key1, key2, value);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public void del(String key1) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.del(key1);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public Boolean exists(String key1) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.exists(key1);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return false;
    }

    public Set<String> keys(String key1) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.keys(key1);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public Set<String> hkeys(String key1) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.hkeys(key1);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public void expireKey(String key, int timeOut) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.expire(key, timeOut);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public void zadd(String key, Map<String, Double> valueMap) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.zadd(key,valueMap);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public Set<String> zrevrange(String key, int start, int end) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.zrevrange(key,start, end);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return null;
    }

    public <T> void hsetWithPipeline(String key, Map<String, T> valueMap) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            Pipeline pipeline = jedis.pipelined();
            for (Map.Entry<String, T> entry : valueMap.entrySet()) {
                pipeline.hset(key, entry.getKey(), JSONObject.toJSONString(entry.getValue()));
            }
            pipeline.sync();
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }

    public long hlen(String key) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.hlen(key);
        } catch (Exception e) {
            logger.error("redis 操作失败:" + e);
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return 0;
    }
}
```