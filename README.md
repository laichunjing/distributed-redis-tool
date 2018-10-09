# distributed-redis-tool

Set bean:

```java
@Configuration
public class RedisLockConfig {
    private Logger logger = LoggerFactory.getLogger(RedisLockConfig.class);
    
    
    @Autowired
    private JedisConnectionFactory jedisConnectionFactory;
    
    @Bean
    public RedisLock build() {
        RedisLock redisLock = new RedisLock.Builder(jedisConnectionFactory,RedisToolsConstant.SINGLE)
                .lockPrefix("lock_")
                .sleepTime(100)
                .build();

        return redisLock;
    }
}

```

#### Non-blocking lock:

```java
    @Autowired
    private RedisLock redisLock ;

    public void use() {
        String key = "key";
        String request = UUID.randomUUID().toString();
        try {
            boolean locktest = redisLock.tryLock(key, request);
            if (!locktest) {
                System.out.println("locked error");
                return;
            }


            //do something

        } finally {
            redisLock.unlock(key,request) ;
        }

    }

```

#### Blocking lock

```java
redisLock.lock(String key, String request);
```

#### Blocking lock, Custom block time

```java
redisLock.lock(String key, String request,int blockTime);
```


----

## Distributed limiting
### Features

- [x] High performance.
- [x] native API.
- [x] Annation API.
- [x] Support Redis cluster, single.
- [x] Suppport Spring4.x+


### Quick start

maven dependency:

```xml
<dependency>
    <groupId>top.crossoverjie.opensource</groupId>
    <artifactId>distributed-redis-tool</artifactId>
    <version>1.0.4</version>
</dependency>
```

1. Set bean:

```java
@Configuration
public class RedisLimitConfig {
    private Logger logger = LoggerFactory.getLogger(RedisLimitConfig.class);
    
    @Value("${redis.limit}")
    private int limit;
    
    @Autowired
    private JedisConnectionFactory jedisConnectionFactory;
    
    @Bean
    public RedisLimit build() {
        RedisLimit redisLimit = new RedisLimit.Builder(jedisConnectionFactory, RedisToolsConstant.SINGLE)
                .limit(limit)
                .build();
        return redisLimit;
    }
}
```

2. Scan `com.crossoverjie.distributed.intercept` package.

```java
@ComponentScan(value = "com.crossoverjie.distributed.intercept")
```

#### Native API:

```java
  	
    boolean limit = redisLimit.limit();
    if (!limit){
        res.setCode(StatusEnum.REQUEST_LIMIT.getCode());
        res.setMessage(StatusEnum.REQUEST_LIMIT.getMessage());
        return res ;
    }
```

Other apis:

#### @ControllerLimit

```java
    @ControllerLimit
    public BaseResponse<OrderNoResVO> getOrderNoLimit(@RequestBody OrderNoReqVO orderNoReq) {
        BaseResponse<OrderNoResVO> res = new BaseResponse();
        res.setReqNo(orderNoReq.getReqNo());
        if (null == orderNoReq.getAppId()){
            throw new SBCException(StatusEnum.FAIL);
        }
        OrderNoResVO orderNoRes = new OrderNoResVO() ;
        orderNoRes.setOrderId(DateUtil.getLongTime());
        res.setCode(StatusEnum.SUCCESS.getCode());
        res.setMessage(StatusEnum.SUCCESS.getMessage());
        res.setDataBody(orderNoRes);
        return res ;
    }
```

Used for `@RequestMapping`.


#### @SpringControllerLimit

If you are using native Spring:

```java
    @SpringControllerLimit(errorCode = 200,errorMsg = "request has limited")
    @RequestMapping("/createOptimisticLimitOrderByRedis/{sid}")
    @ResponseBody
    public String createOptimisticLimitOrderByRedis(@PathVariable int sid) {
        logger.info("sid=[{}]", sid);
        int id = 0;
        try {
            id = orderService.createOptimisticOrderUseRedis(sid);
        } catch (Exception e) {
            logger.error("Exception",e);
        }
        return String.valueOf(id);
    }
```

Spring xml:

```xml
   <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/> 
            <bean class="com.crossoverjie.distributed.intercept.SpringMVCIntercept"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

#### @CommonLimit

```java
@CommonLimit
public void anyMethod(){}
```

It can be used for any Methods.




