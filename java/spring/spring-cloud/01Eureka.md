# Eureka

## 简单使用

```
@SpringBootApplication
@EnableEurekaServer
public class APP {
    public static void main(String[] args) {
        SpringApplication.run(APP.class,args);
    }
}

server:
  port: 8000
eureka:
  server:
    enable-self-preservation: false  #关闭自我保护机制
    eviction-interval-timer-in-ms: 4000 #设置清理间隔（单位：毫秒 默认是60*1000）
  instance:
    hostname: localhost


  client:
    registerWithEureka: false #不把自己作为一个客户端注册到自己身上
    fetchRegistry: false  #不需要从服务端获取注册信息（因为在这里自己就是服务端，而且已经禁用自己注册了）
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
```

```
@SpringBootApplication
@EnableEurekaClient
public class APP {
    public static void main(String[] args) {
        SpringApplication.run(APP.class,args);
    }
}

server:
  port: 8001
spring:
  application:
    name: eureka-client
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8000/eureka #有多个以逗号分割
  instance:
    instance-id: clent-0 #此实例注册到eureka的实例Id
    prefer-ip-address: true # 是否显示IP地址
    lease-renewal-interval-in-seconds: 10 # 多长时间发送心跳
    lease-expiration-duration-in-seconds: 40 # 服务端收到最后一次心跳后多长时间判定过期
```

## Eureka概念

**服务注册**：Eureka Client会通过发送REST请求的方式向Eureka Server注册自己的服务，提供自身的元数据，比如 IP 地址、端口、运行状况指标的URL、主页地址等信息。Eureka Server接收到注册请求后，就会把这些元数据信息存储在一个ConcurrentHashMap中。

**服务续约**：在服务注册后，Eureka Client会维护一个心跳来持续通知Eureka Server，说明服务一直处于可用状态，防止被剔除。Eureka Client在默认的情况下会每隔30秒发送一次心跳来进行服务续约。实质上就是修改Lease对象里面的时间戳。

**服务同步**：Eureka Server之间会互相进行注册，构建Eureka Server集群，不同Eureka Server之间会进行服务同步，用来保证服务信息的一致性。

**获取服务**：服务消费者（Eureka Client）在启动的时候，会发送一个REST请求给Eureka Server，获取上面注册的服务清单，并且缓存在Eureka Client本地，默认缓存30秒。同时，为了性能考虑，Eureka Server也会维护一份只读的服务清单缓存，该缓存每隔30秒更新一次。

**服务调用**：服务消费者在获取到服务清单后，就可以根据清单中的服务列表信息，查找到其他服务的地址，从而进行远程调用。Eureka有Region和Zone的概念，一个Region可以包含多个Zone，在进行服务调用时，优先访问处于同一个Zone中的服务提供者。

**服务下线**：当Eureka Client需要关闭或重启时，就不希望在这个时间段内再有请求进来，所以，就需要提前先发送REST请求给Eureka Server，告诉Eureka Server自己要下线了，Eureka Server在收到请求后，就会把该服务状态置为下线（DOWN），并把该下线事件传播出去。

**服务剔除**：有时候，服务实例可能会因为网络故障等原因导致不能提供服务，而此时该实例也没有发送请求给Eureka Server来进行服务下线，所以，还需要有服务剔除的机制。Eureka Server在启动的时候会创建一个定时任务，每隔一段时间（默认60秒），从当前服务清单中把超时没有续约（默认90秒）的服务剔除。

**自我保护**：既然Eureka Server会定时剔除超时没有续约的服务，那就有可能出现一种场景，网络一段时间内发生了异常，所有的服务都没能够进行续约，Eureka Server就把所有的服务都剔除了，这样显然不太合理。所以，就有了自我保护机制，当短时间内，统计续约失败的比例，如果达到一定阈值（15%），则会触发自我保护的机制，在该机制下，Eureka Server不会剔除任何的微服务，等到正常后，再退出自我保护机制。（根据已经注册的实例计算的预期心跳数和实际的对比）

**服务端缓存**：在服务端增量拉取和全量拉取都会走缓存，服务端缓存分为3级缓存，三级缓存的主要目的是为了解决读写冲突。三级缓存是**只读缓存（ConcurrentMap<Key, Value> readOnlyCacheMap）**，有一个线程每**30s**对比二级缓存（二级缓存没有就去以及缓存），更新自己的三级缓存。**所以三级缓存存在30s的误差**。二级缓存是guava提供的 **LoadingCache<Key, Value> readWriteCacheMap**，**没有误差，每次有更新操作的时候都会让缓存中的数据失效**，二级缓存的数据设置了**过期时间默认为180s**。一级缓存是**ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry**，所有的注册都直接保存在这里。当客户端拉取数据的时候，会判断是否可以走三级缓存（默认开启），从三级缓存中获取，如果没有，就从二级缓存中获取，然后同步到三级缓存中，如果二级缓存中依然没有，就会从一级缓存中获取，然后同步到二级缓存，再同步到三级缓存。

**客户端缓存**：客户端同样也是维护了一个Instance缓存，单独开一个线程，默认每30s去服务端拉取数据，然后同步到本地缓存，也会**存在30s的误差**，**所以如果服务端开启三级缓存，就会发生最多1分钟的延迟**，以及后面ribbion也会做缓存30s，还会有更长的延迟。

**Region、Zone**:eureka提供了region和zone两个概念来进行分区，这两个概念均来自于亚马逊的AWS：region：可以简单理解为**地理上的分区**，比如上海地区，或者广州地区，再或者北京等等，没有具体大小的限制。根据项目具体的情况，可以自行合理划分region。zone：可以简单理解为**region内的具体机房**，比如说region划分为北京，然后北京有两个机房，就可以在此region之下划分出zone1,zone2两个zone。**一个机房内的服务优先调用同一个机房内的服务，当同一个机房的服务不可用的时候，再去调用其它机房的服务，以达到减少延时的作用**

**VIP**:在客户端配置vip,可以实现客户端只去拉取配置了vip的注册信息，多个用逗号分割

![image-20201202144221822](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201202144221822-1606891348-40e89f.png)



## Eureka-Server主要原理

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1607781-20191105230638561-211396277-1606716895-228cbe.png)

![image-20201130141642597](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201130141642597-1606717002-2b6b54.png)

![image-20201130141727621](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201130141727621-1606717047-6f313b.png)

![image-20201130141849152](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201130141849152-1606717129-116f30.png)



这里主要是初始化一个Jersey Web服务器，后面注册，同步，下线等等都是使用Http请求完成

```java
EurekaServerAutoConfiguration.java

@Bean
public javax.ws.rs.core.Application jerseyApplication(Environment environment,
      ResourceLoader resourceLoader) {

   ClassPathScanningCandidateComponentProvider provider = new ClassPathScanningCandidateComponentProvider(
         false, environment);

   // Filter to include only classes that have a particular annotation.
   //
   provider.addIncludeFilter(new AnnotationTypeFilter(Path.class));
   provider.addIncludeFilter(new AnnotationTypeFilter(Provider.class));

   // Find classes in Eureka packages (or subpackages)
   //
   Set<Class<?>> classes = new HashSet<>();
   for (String basePackage : EUREKA_PACKAGES) {
      Set<BeanDefinition> beans = provider.findCandidateComponents(basePackage);
      for (BeanDefinition bd : beans) {
         Class<?> cls = ClassUtils.resolveClassName(bd.getBeanClassName(),
               resourceLoader.getClassLoader());
         classes.add(cls);
      }
   }
```

### 服务注册

```java
//ApplicationResource.java
//接收服务注册的http请求
@POST
@Consumes({"application/json", "application/xml"})
public Response addInstance(InstanceInfo info,
                            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
    //info:实例信息
    //isReplication:请求头 ，是否是同步的请求
    
    logger.debug("Registering instance {} (replication={})", info.getId(), isReplication);
    // validate that the instanceinfo contains all the necessary required fields
    
	.........
    registry.register(info, "true".equals(isReplication));
    return Response.status(204).build();  // 204 to be backwards compatible
}

//PeerAwareInstanceRegistryImpl.java
  @Override
    public void register(final InstanceInfo info, final boolean isReplication) {
        int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
        if (info.getLeaseInfo() != null && info.getLeaseInfo().getDurationInSecs() > 0) {
            leaseDuration = info.getLeaseInfo().getDurationInSecs();
        }
        //调用父类的注册
        super.register(info, leaseDuration, isReplication);
        //将注册发送到其他的 server,实现同步
        replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
    }


 public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
        try {
            read.lock();
            //private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry
            //最终保存注册信息的是两层Map
            //外层key->applicationName
            //内层key->id
            //内层value->Lease<InstanceInfo>  
            //Lease租债器，用于判断是否过期
            Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
            REGISTER.increment(isReplication);
            if (gMap == null) {
                final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
                gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }
           .......
            
            //放到Map中
            gMap.put(registrant.getId(), lease);
          ............
        } finally {
            read.unlock();
        }
    }
//同步信息
private void replicateToPeers(Action action, String appName, String id,
                                  InstanceInfo info /* optional */,
                                  InstanceStatus newStatus /* optional */, boolean isReplication) {
        Stopwatch tracer = action.getTimer().start();
        try {
            if (isReplication) {
                numberOfReplicationsLastMin.increment();
            }
            // If it is a replication already, do not replicate again as this will create a poison replication
            if (peerEurekaNodes == Collections.EMPTY_LIST || isReplication) {
                return;
            }

            for (final PeerEurekaNode node : peerEurekaNodes.getPeerEurekaNodes()) {
                // If the url represents this host, do not replicate to yourself.
                if (peerEurekaNodes.isThisMyUrl(node.getServiceUrl())) {
                    continue;
                }
                replicateInstanceActionsToPeers(action, appName, id, info, newStatus, node);
            }
        } finally {
            tracer.stop();
        }
    }

private void replicateInstanceActionsToPeers(Action action, String appName,
                                                 String id, InstanceInfo info, InstanceStatus newStatus,
                                                 PeerEurekaNode node) {
        try {
            InstanceInfo infoFromRegistry = null;
            CurrentRequestVersion.set(Version.V2);
            switch (action) {
                case Cancel:
                    node.cancel(appName, id);
                    break;
                case Heartbeat:
                    InstanceStatus overriddenStatus = overriddenInstanceStatusMap.get(id);
                    infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                    node.heartbeat(appName, id, infoFromRegistry, overriddenStatus, false);
                    break;
                case Register:
                    node.register(info);
                    break;
                case StatusUpdate:
                    infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                    node.statusUpdate(appName, id, newStatus, infoFromRegistry);
                    break;
                case DeleteStatusOverride:
                    infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                    node.deleteStatusOverride(appName, id, infoFromRegistry);
                    break;
            }
        } catch (Throwable t) {
            logger.error("Cannot replicate information to {} for action {}", node.getServiceUrl(), action.name(), t);
        }
    }
```

### 生成全量的注册信息

```java
ApplicationsResource.java
@GET
    public Response getContainers(@PathParam("version") String version,
                                  @HeaderParam(HEADER_ACCEPT) String acceptHeader,
                                  @HeaderParam(HEADER_ACCEPT_ENCODING) String acceptEncoding,
                                  @HeaderParam(EurekaAccept.HTTP_X_EUREKA_ACCEPT) String eurekaAccept,
                                  @Context UriInfo uriInfo,
                                  @Nullable @QueryParam("regions") String regionsStr) {

        boolean isRemoteRegionRequested = null != regionsStr && !regionsStr.isEmpty();
        String[] regions = null;
        if (!isRemoteRegionRequested) {
            EurekaMonitors.GET_ALL.increment();
        } else {
            regions = regionsStr.toLowerCase().split(",");
            Arrays.sort(regions); // So we don't have different caches for same regions queried in different order.
            EurekaMonitors.GET_ALL_WITH_REMOTE_REGIONS.increment();
        }

        // Check if the server allows the access to the registry. The server can
        // restrict access if it is not
        // ready to serve traffic depending on various reasons.
        if (!registry.shouldAllowAccess(isRemoteRegionRequested)) {
            return Response.status(Status.FORBIDDEN).build();
        }
        CurrentRequestVersion.set(Version.toEnum(version));
        KeyType keyType = Key.KeyType.JSON;
        String returnMediaType = MediaType.APPLICATION_JSON;
        if (acceptHeader == null || !acceptHeader.contains(HEADER_JSON_VALUE)) {
            keyType = Key.KeyType.XML;
            returnMediaType = MediaType.APPLICATION_XML;
        }
		//创建Key
        Key cacheKey = new Key(Key.EntityType.Application,
                ResponseCacheImpl.ALL_APPS,
                keyType, CurrentRequestVersion.get(), EurekaAccept.fromString(eurekaAccept), regions
        );

        Response response;
    	//如果接收gzip就返回gzip
        if (acceptEncoding != null && acceptEncoding.contains(HEADER_GZIP_VALUE)) {
            response = Response.ok(responseCache.getGZIP(cacheKey))
                    .header(HEADER_CONTENT_ENCODING, HEADER_GZIP_VALUE)
                    .header(HEADER_CONTENT_TYPE, returnMediaType)
                    .build();
        } else {
            //如果不接收gzip就返回正常格式,流程和getGZIP差不多，只是GZIP加压了
            response = Response.ok(responseCache.get(cacheKey))
                    .build();
        }
        return response;
    
    //通过key获取GZIP压缩后的缓存
     public byte[] getGZIP(Key key) {
        //获取数据，可以配置是否从只读缓存获取数据
        Value payload = getValue(key, shouldUseReadOnlyResponseCache);
        if (payload == null) {
            return null;
        }
         //压缩数据
        return payload.getGzipped();
    }
    
    
    Value getValue(final Key key, boolean useReadOnlyCache) {
        Value payload = null;
        try {
            if (useReadOnlyCache) {
                //从只读缓存获取数据
                //有单独的一个线程维护只读缓存，默认30s执行一次，
                //将只读缓存里面所有的数据与读写缓存的数据做对比
                //如果不同就更新只读缓存
                final Value currentPayload = readOnlyCacheMap.get(key);
                if (currentPayload != null) {
                    payload = currentPayload;
                } else {
                    //在只读缓存没有获取到数据，从读写缓存获取数据
                    payload = readWriteCacheMap.get(key);
                    //将读写缓存的数据放到只读缓存
                    readOnlyCacheMap.put(key, payload);
                }
            } else {
                payload = readWriteCacheMap.get(key);
            }
        } catch (Throwable t) {
            logger.error("Cannot get value for key : {}", key, t);
        }
        return payload;
    }
    
    //===========================读写缓存====================================
    this.readWriteCacheMap =
                CacheBuilder.newBuilder().initialCapacity(serverConfig.getInitialCapacityOfResponseCache())
               //上一次写后写后的过期时间,默认是180s
              //该缓存还可以配置距离上一次读/写后的过期时间（expireAfterAccess），只是这里没有用
        .expireAfterWrite(serverConfig.getResponseCacheAutoExpirationInSeconds(), TimeUnit.SECONDS)
        			//过期后的监听器
                        .removalListener(new RemovalListener<Key, Value>() {
                            @Override
                            public void onRemoval(RemovalNotification<Key, Value> notification) {
                                Key removedKey = notification.getKey();
                                if (removedKey.hasRegions()) {
                                    Key cloneWithNoRegions = removedKey.cloneWithoutRegions();
                                    regionSpecificKeys.remove(cloneWithNoRegions, removedKey);
                                }
                            }
                        })
        		//如果没有在缓存中获取到数据，就从执行CacheLoader.load获取数据，并同步到缓存
                        .build(new CacheLoader<Key, Value>() {
                            @Override
                            public Value load(Key key) throws Exception {
                                if (key.hasRegions()) {
                                    Key cloneWithNoRegions = key.cloneWithoutRegions();
                                    regionSpecificKeys.put(cloneWithNoRegions, key);
                                }
                                //从一级缓存获取 registry
                                Value value = generatePayload(key);
                                return value;
                            }
                        });
    
    //当发生修改的时候会手动让缓存失效
    public void invalidate(Key... keys) {
        for (Key key : keys) {
            logger.debug("Invalidating the response cache key : {} {} {} {}, {}",
                    key.getEntityType(), key.getName(), key.getVersion(), key.getType(), key.getEurekaAccept());

            readWriteCacheMap.invalidate(key);
            Collection<Key> keysWithRegions = regionSpecificKeys.get(key);
            if (null != keysWithRegions && !keysWithRegions.isEmpty()) {
                for (Key keysWithRegion : keysWithRegions) {
                    logger.debug("Invalidating the response cache key : {} {} {} {} {}",
                            key.getEntityType(), key.getName(), key.getVersion(), key.getType(), key.getEurekaAccept());
                    readWriteCacheMap.invalidate(keysWithRegion);
                }
            }
        }
    }
    
    //===========================只读缓存更新线程==============================
    //默认是30s更新一次
    private TimerTask getCacheUpdateTask() {
        return new TimerTask() {
            @Override
            public void run() {
                logger.debug("Updating the client cache from response cache");
                for (Key key : readOnlyCacheMap.keySet()) {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Updating the client cache from response cache for key : {} {} {} {}",
                                key.getEntityType(), key.getName(), key.getVersion(), key.getType());
                    }
                    try {
                        CurrentRequestVersion.set(key.getVersion());
                        Value cacheValue = readWriteCacheMap.get(key);
                        Value currentCacheValue = readOnlyCacheMap.get(key);
                        if (cacheValue != currentCacheValue) {
                            readOnlyCacheMap.put(key, cacheValue);
                        }
                    } catch (Throwable th) {
                        logger.error("Error while updating the client cache from response cache for key {}", key.toStringCompact(), th);
                    }
                }
            }
        };
    }
```

### 获取增量数据

```java
//与获取全量数据并没有什么差别，都是走的三级缓存
//只是Key中的参数不同
//所以在读写缓存中获取数据并且没有的时候，区以及缓存获取数据有所区别
//主要体现在readWriteCacheMap-》CacheLoader-》load-》generatePayload(key)

private Value generatePayload(Key key) {
        Stopwatch tracer = null;
        try {
            String payload;
            switch (key.getEntityType()) {
                case Application:
                    boolean isRemoteRegionRequested = key.hasRegions();
					//获取全量数据
                    if (ALL_APPS.equals(key.getName())) {
                        if (isRemoteRegionRequested) {
                            tracer = serializeAllAppsWithRemoteRegionTimer.start();
                            payload = getPayLoad(key, registry.getApplicationsFromMultipleRegions(key.getRegions()));
                        } else {
                            tracer = serializeAllAppsTimer.start();
                            payload = getPayLoad(key, registry.getApplications());
                        }
                      //获取增量数据
                     //在一级缓存中维护了一个队列 recentlyChangedQueue
                     //队列中保存的就是最近有过修改的实例的信息
                     //增量就是获取该队列中的信息，并且返回回去
                    } else if (ALL_APPS_DELTA.equals(key.getName())) {
                        if (isRemoteRegionRequested) {
                            tracer = serializeDeltaAppsWithRemoteRegionTimer.start();
                            versionDeltaWithRegions.incrementAndGet();
                            versionDeltaWithRegionsLegacy.incrementAndGet();
                            payload = getPayLoad(key,
                                    registry.getApplicationDeltasFromMultipleRegions(key.getRegions()));
                        } else {
                            tracer = serializeDeltaAppsTimer.start();
                            versionDelta.incrementAndGet();
                            versionDeltaLegacy.incrementAndGet();
                            //registry.getApplicationDeltas() 就是从队列中获取数据
                            //这里面将HashCode赋值为全部Instance的HashCode
                            payload = getPayLoad(key, registry.getApplicationDeltas());
                        }
                    } else {
                        tracer = serializeOneApptimer.start();
                        payload = getPayLoad(key, registry.getApplication(key.getName()));
                    }
                    break;
                 //VIP就是客户端拉取指定的服务
                case VIP:
                case SVIP:
                    tracer = serializeViptimer.start();
                    payload = getPayLoad(key, getApplicationsForVip(key, registry));
                    break;
                default:
                    logger.error("Unidentified entity type: {} found in the cache key.", key.getEntityType());
                    payload = "";
                    break;
            }
            return new Value(payload);
        } finally {
            if (tracer != null) {
                tracer.stop();
            }
        }
    }

//该队列也是有线程定时去删除
 private TimerTask getDeltaRetentionTask() {
        return new TimerTask() {

            @Override
            public void run() {
                Iterator<RecentlyChangedItem> it = recentlyChangedQueue.iterator();
                while (it.hasNext()) {
                    if (it.next().getLastUpdateTime() <
                            System.currentTimeMillis() - serverConfig.getRetentionTimeInMSInDeltaQueue()) {
                        it.remove();
                    } else {
                        break;
                    }
                }
            }

        };
    }
```





## Eureka-Client

![image-20201201161810469](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201201161810469-1606810697-2a281a.png)

![image-20201201161915459](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201201161915459-1606810755-ce1596.png)

```java
@Inject
DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args,
                Provider<BackupRegistry> backupRegistryProvider, EndpointRandomizer endpointRandomizer) {
    
    ................

     //判断是否需要从注册中心拉取注册信息
    if (config.shouldFetchRegistry()) {
        this.registryStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRY_PREFIX + "lastUpdateSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
    } else {
        this.registryStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
    }
	//判断时许需要将自己注册到注册中心
    if (config.shouldRegisterWithEureka()) {
        this.heartbeatStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRATION_PREFIX + "lastHeartbeatSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
    } else {
        this.heartbeatStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
    }

   .................

    //初始化两个线程，一个是从注册中心拉取数据，一个是发送心跳
    try {
        // default size of 2 - 1 each for heartbeat and cacheRefresh
        scheduler = Executors.newScheduledThreadPool(2,
                new ThreadFactoryBuilder()
                        .setNameFormat("DiscoveryClient-%d")
                        .setDaemon(true)
                        .build());

        heartbeatExecutor = new ThreadPoolExecutor(
                1, clientConfig.getHeartbeatExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>(),
                new ThreadFactoryBuilder()
                        .setNameFormat("DiscoveryClient-HeartbeatExecutor-%d")
                        .setDaemon(true)
                        .build()
        );  // use direct handoff

        cacheRefreshExecutor = new ThreadPoolExecutor(
                1, clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>(),
                new ThreadFactoryBuilder()
                        .setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d")
                        .setDaemon(true)
                        .build()
        );  // use direct handoff

	......................
	//拉取注册信息
    if (clientConfig.shouldFetchRegistry() && !fetchRegistry(false)) {
        //从备份的server拉取数据
        fetchRegistryFromBackup();
    }

   ......................

    // finally, init the schedule tasks (e.g. cluster resolvers, heartbeat, instanceInfo replicator, fetch
    //启动两个定时器
    initScheduledTasks();
	.......................
}
```

### 拉取注册

```java
private boolean fetchRegistry(boolean forceFullRegistryFetch) {
    Stopwatch tracer = FETCH_REGISTRY_TIMER.start();

    try {
        // If the delta is disabled or if it is the first time, get all
        // applications
        Applications applications = getApplications();

        if (clientConfig.shouldDisableDelta()
                || (!Strings.isNullOrEmpty(clientConfig.getRegistryRefreshSingleVipAddress()))
                || forceFullRegistryFetch
                || (applications == null)
                || (applications.getRegisteredApplications().size() == 0)
                || (applications.getVersion() == -1)) //Client application does not have latest library supporting delta
        {
            .....
             //全量拉取数据
            getAndStoreFullRegistry();
        } else {
            //增量拉取数据
            getAndUpdateDelta(applications);
        }
        applications.setAppsHashCode(applications.getReconcileHashCode());
        logTotalInstances();
    } catch (Throwable e) {
        logger.error(PREFIX + "{} - was unable to refresh its cache! status = {}", appPathIdentifier, e.getMessage(), e);
        return false;
    } finally {
        if (tracer != null) {
            tracer.stop();
        }
    }

    // Notify about cache refresh before updating the instance remote status
    onCacheRefreshed();

    // Update remote status based on refreshed data held in the cache
    updateInstanceRemoteStatus();

    // registry was fetched successfully, so return true
    return true;
}

//全量拉取数据
private void getAndStoreFullRegistry() throws Throwable {
        long currentUpdateGeneration = fetchRegistryGeneration.get();

        logger.info("Getting all instance registry info from the eureka server");

        Applications apps = null;
    	//判断是否设置VIP Address
        EurekaHttpResponse<Applications> httpResponse = clientConfig.getRegistryRefreshSingleVipAddress() == null
                ? eurekaTransport.queryClient.getApplications(remoteRegionsRef.get())
                : eurekaTransport.queryClient.getVip(clientConfig.getRegistryRefreshSingleVipAddress(), remoteRegionsRef.get());
        if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
            apps = httpResponse.getEntity();
        }
        logger.info("The response status is {}", httpResponse.getStatusCode());

        ....
    }

//拉取增量数据
private void getAndUpdateDelta(Applications applications) throws Throwable {
        long currentUpdateGeneration = fetchRegistryGeneration.get();

        Applications delta = null;
        EurekaHttpResponse<Applications> httpResponse = eurekaTransport.queryClient.getDelta(remoteRegionsRef.get());
        if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
            delta = httpResponse.getEntity();
        }

        if (delta == null) {
          	//没有获取到数据，使用全量拉取
            getAndStoreFullRegistry();
        } else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
           
            //将增量拉取下来的Application合并到本地缓存，并且计算HashCode
            String reconcileHashCode = "";
            if (fetchRegistryUpdateLock.tryLock()) {
                try {
                    updateDelta(delta);
                    reconcileHashCode = getReconcileHashCode(applications);
                } finally {
                    fetchRegistryUpdateLock.unlock();
                }
            } else {
                logger.warn("Cannot acquire update lock, aborting getAndUpdateDelta");
            }
            // There is a diff in number of instances for some reason
            //判断本地的HashCode和服务端返回的HashCode是否一致
            if (!reconcileHashCode.equals(delta.getAppsHashCode()) || clientConfig.shouldLogDeltaDiff()) {
                //全量拉取
                reconcileAndLogDifference(delta, reconcileHashCode);  // this makes a remoteCall
            }
        } else {
            logger.warn("Not updating application delta as another thread is updating it already");
            logger.debug("Ignoring delta update with apps hashcode {}, as another thread is updating it already", delta.getAppsHashCode());
        }
    }
```

### 注册

```java
boolean register() throws Throwable {
    logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
    EurekaHttpResponse<Void> httpResponse;
    try {
        //发送Http请求
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    if (logger.isInfoEnabled()) {
        logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
    }
    return httpResponse.getStatusCode() == Status.NO_CONTENT.getStatusCode();
}
```

### 发送心跳

```java
//renewalIntervalInSecs 默认30 
scheduler.schedule(
                    new TimedSupervisorTask(
                            "heartbeat",
                            scheduler,
                            heartbeatExecutor,
                            renewalIntervalInSecs,
                            TimeUnit.SECONDS,
                            expBackOffBound,
                            new HeartbeatThread()
                    ),
                    renewalIntervalInSecs, TimeUnit.SECONDS);

private class HeartbeatThread implements Runnable {

        public void run() {
            if (renew()) {
                lastSuccessfulHeartbeatTimestamp = System.currentTimeMillis();
            }
        }
    }

boolean renew() {
    EurekaHttpResponse<InstanceInfo> httpResponse;
    try {
        //构建http请求
        httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
        logger.debug(PREFIX + "{} - Heartbeat status: {}", appPathIdentifier, httpResponse.getStatusCode());
        if (httpResponse.getStatusCode() == Status.NOT_FOUND.getStatusCode()) {
            REREGISTER_COUNTER.increment();
            logger.info(PREFIX + "{} - Re-registering apps/{}", appPathIdentifier, instanceInfo.getAppName());
            long timestamp = instanceInfo.setIsDirtyWithTime();
            boolean success = register();
            if (success) {
                instanceInfo.unsetIsDirty(timestamp);
            }
            return success;
        }
        return httpResponse.getStatusCode() == Status.OK.getStatusCode();
    } catch (Throwable e) {
        logger.error(PREFIX + "{} - was unable to send heartbeat!", appPathIdentifier, e);
        return false;
    }
}
```

### 刷新缓存

```java
//registryFetchIntervalSeconds默认30
scheduler.schedule(
        new TimedSupervisorTask(
                "cacheRefresh",
                scheduler,
                cacheRefreshExecutor,
                registryFetchIntervalSeconds,
                TimeUnit.SECONDS,
                expBackOffBound,
                new CacheRefreshThread()
        ),
        registryFetchIntervalSeconds, TimeUnit.SECONDS);

 class CacheRefreshThread implements Runnable {
        public void run() {
            refreshRegistry();
        }
    }

void refreshRegistry() {
        try {
            boolean isFetchingRemoteRegionRegistries = isFetchingRemoteRegionRegistries();

            boolean remoteRegionsModified = false;
            // This makes sure that a dynamic change to remote regions to fetch is honored.
            String latestRemoteRegions = clientConfig.fetchRegistryForRemoteRegions();
            if (null != latestRemoteRegions) {
                String currentRemoteRegions = remoteRegionsToFetch.get();
                if (!latestRemoteRegions.equals(currentRemoteRegions)) {
                    // Both remoteRegionsToFetch and AzToRegionMapper.regionsToFetch need to be in sync
                    synchronized (instanceRegionChecker.getAzToRegionMapper()) {
                        if (remoteRegionsToFetch.compareAndSet(currentRemoteRegions, latestRemoteRegions)) {
                            String[] remoteRegions = latestRemoteRegions.split(",");
                            remoteRegionsRef.set(remoteRegions);
                            instanceRegionChecker.getAzToRegionMapper().setRegionsToFetch(remoteRegions);
                            remoteRegionsModified = true;
                        } else {
                            logger.info("Remote regions to fetch modified concurrently," +
                                    " ignoring change from {} to {}", currentRemoteRegions, latestRemoteRegions);
                        }
                    }
                } else {
                    // Just refresh mapping to reflect any DNS/Property change
                    instanceRegionChecker.getAzToRegionMapper().refreshMapping();
                }
            }
			//拉取注册信息
            boolean success = fetchRegistry(remoteRegionsModified);
            if (success) {
                registrySize = localRegionApps.get().size();
                lastSuccessfulRegistryFetchTimestamp = System.currentTimeMillis();
            }

        } catch (Throwable e) {
            logger.error("Cannot fetch registry from server", e);
        }
    }
```

## 如何在Client获取注册信息

```java

@Component
public class ApplicationUtil {
 
	@Autowired
	private DiscoveryClient discoveryClient;
 
	public Map<String, List<ServiceInstance>> serviceUrl() {
		Map<String, List<ServiceInstance>> msl = new HashMap<String, List<ServiceInstance>>();
		List<String> services = discoveryClient.getServices();
		for (String service : services) {
			List<ServiceInstance> sis = discoveryClient.getInstances(service);
			msl.put(service, sis);
		}
		return msl;
 
	}
	
	public boolean isExitservice(String applicationName) {
		List<String> services = discoveryClient.getServices();
		for (String service : services) {
			if(service.toUpperCase().equals(applicationName.toUpperCase())) {
				return true;
			}
		}
		return false;
	}
}
```

## Eureka常用配置

### 通用配置

```properties
# 应用名称，将会显示在Eureka界面的应用名称列
spring.application.name=config-service

# 应用端口，Eureka服务端默认为：8761
server.port=3333
```

###  eureka.server

```properties
# org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean

# 是否允许开启自我保护模式，缺省：true
# 当Eureka服务器在短时间内丢失过多客户端时，自我保护模式可使服务端不再删除失去连接的客户端
eureka.server.enable-self-preservation = false

# Peer节点更新间隔，单位：毫秒
eureka.server.peer-eureka-nodes-update-interval-ms = 

# Eureka服务器清理无效节点的时间间隔，单位：毫秒，缺省：60000，即60秒
eureka.server.eviction-interval-timer-in-ms = 60000
```

### eureka.instance

```properties
# org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean

# 服务名，默认取 spring.application.name 配置值，如果没有则为 unknown
eureka.instance.appname = eureka-client

# 实例ID
eureka.instance.instance-id = eureka-client-instance1

# 应用实例主机名
eureka.instance.hostname = localhost

# 客户端在注册时使用自己的IP而不是主机名，缺省：false
eureka.instance.prefer-ip-address = false

# 应用实例IP
eureka.instance.ip-address = 127.0.0.1

# 服务失效时间，失效的服务将被剔除。单位：秒，默认：90
eureka.instance.lease-expiration-duration-in-seconds = 90

# 服务续约（心跳）频率，单位：秒，缺省30
eureka.instance.lease-renewal-interval-in-seconds = 30

# 状态页面的URL，相对路径，默认使用 HTTP 访问，如需使用 HTTPS则要使用绝对路径配置，缺省：/info
eureka.instance.status-page-url-path = /info

# 健康检查页面的URL，相对路径，默认使用 HTTP 访问，如需使用 HTTPS则要使用绝对路径配置，缺省：/health
eureka.instance.health-check-url-path = /health
```

### eureka.client

```properties
# org.springframework.cloud.netflix.eureka.EurekaClientConfigBean

# Eureka服务器的地址，类型为HashMap，缺省的Key为 defaultZone；缺省的Value为 http://localhost:8761/eureka
# 如果服务注册中心为高可用集群时，多个注册中心地址以逗号分隔。 
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka

# 是否向注册中心注册自己，缺省：true
# 一般情况下，Eureka服务端是不需要再注册自己的
eureka.client.register-with-eureka = true

# 是否从Eureka获取注册信息，缺省：true
# 一般情况下，Eureka服务端是不需要的
eureka.client.fetch-registry = true

# 客户端拉取服务注册信息间隔，单位：秒，缺省：30
eureka.client.registry-fetch-interval-seconds = 30

# 是否启用客户端健康检查
eureka.client.health-check.enabled = true

# 
eureka.client.eureka-service-url-poll-interval-seconds = 60

# 连接Eureka服务器的超时时间，单位：秒，缺省：5
eureka.client.eureka-server-connect-timeout-seconds = 5

# 从Eureka服务器读取信息的超时时间，单位：秒，缺省：8
eureka.client.eureka-server-read-timeout-seconds = 8

# 获取实例时是否只保留状态为 UP 的实例，缺省：true
eureka.client.filter-only-up-instances = true

# Eureka服务端连接空闲时的关闭时间，单位：秒，缺省：30
eureka.client.eureka-connection-idle-timeout-seconds = 30

# 从Eureka客户端到所有Eureka服务端的连接总数，缺省：200
eureka.client.eureka-server-total-connections = 200

# 从Eureka客户端到每个Eureka服务主机的连接总数，缺省：50
eureka.client.eureka-server-total-connections-per-host = 50
```

