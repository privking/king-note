# Ribbon

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套**客户端负载均衡**的工具。

服务端的负载均衡是一个url先经过一个**代理服务器**（比如nginx），然后通过这个代理服务器通过算法（轮询，随机，权重等等..）反向代理你的服务，来完成负载均衡

客户端的负载均衡则是一个请求**在客户端的时候已经声明了要调用哪个服务**，然后通过具体的负载均衡算法来完成负载均衡



## 基本使用

![image-20201203095655033](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201203095655033-1606960622-fa0a36.png)

![image-20201203095713590](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201203095713590-1606960633-bffae0.png)

配置隔离

@ExcludeFromComponentScan主要是配置过滤掉扫描，如果本身就扫描不到，就没有加的必要

![image-20201204151344528](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204151344528-1607066024-065b06.png)

![image-20201204151404752](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204151404752-1607066044-f857f7.png)

![image-20201204151416900](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204151416900-1607066056-bc5f00.png)

## 大致原理

### 解析 @RibbonClient @RibbonClients注解

该注解可以不加

但是利用该注解可以作一些配置隔离，比如不同的服务使用不同的负载均衡策略

@RibbonClient @RibbonClients都加了RibbonClientConfigurationRegistrar注解

![image-20201204094803568](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204094803568-1607046492-6a2235.png)

注入的RibbonClientConfigurationRegistrar 实现了ImportBeanDefinitionRegistrar接口，将@RibbonClient的configuration属性以及@RibbonClients的defaultConfiguration属性解析成**RibbonClientSpecification**注入。

### @LoadBalanced

![image-20201203095838860](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201203095838860-1606960718-595788.png)

```java
@Configuration
@Conditional(RibbonAutoConfiguration.RibbonClassesConditions.class)
@RibbonClients
@AutoConfigureAfter(
      name = "org.springframework.cloud.netflix.eureka.EurekaClientAutoConfiguration")
@AutoConfigureBefore({ LoadBalancerAutoConfiguration.class,
      AsyncLoadBalancerAutoConfiguration.class })
@EnableConfigurationProperties({ RibbonEagerLoadProperties.class,
      ServerIntrospectorProperties.class })
public class RibbonAutoConfiguration {
  //这里注入的集合是@RibbonClient解析出来的集合
   //因为可以不加 所以这里不是必须注入的
   @Autowired(required = false)
   private List<RibbonClientSpecification> configurations = new ArrayList<>();

   @Autowired
   private RibbonEagerLoadProperties ribbonEagerLoadProperties;

   @Bean
   public HasFeatures ribbonFeature() {
      return HasFeatures.namedFeature("Ribbon", Ribbon.class);
   }

    //这个Factory是拿来创建子容器，并且获取负载均衡器的
    //所以这里把注解解析出来的配置传进去很关键
   @Bean
   public SpringClientFactory springClientFactory() {
      SpringClientFactory factory = new SpringClientFactory();
      factory.setConfigurations(this.configurations);
      return factory;
   }

    //这个是负载均衡器客户端，执行还没有解析的requestURL,并且能重建url,依赖于LoadBalancerRequest
   @Bean
   @ConditionalOnMissingBean(LoadBalancerClient.class)
   public LoadBalancerClient loadBalancerClient() {
      return new RibbonLoadBalancerClient(springClientFactory());
   }

   @Bean
   @ConditionalOnClass(name = "org.springframework.retry.support.RetryTemplate")
   @ConditionalOnMissingBean
   public LoadBalancedRetryFactory loadBalancedRetryPolicyFactory(
         final SpringClientFactory clientFactory) {
      return new RibbonLoadBalancedRetryFactory(clientFactory);
   }

   @Bean
   @ConditionalOnMissingBean
   public PropertiesFactory propertiesFactory() {
      return new PropertiesFactory();
   }

   @Bean
   @ConditionalOnProperty("ribbon.eager-load.enabled")
   public RibbonApplicationContextInitializer ribbonApplicationContextInitializer() {
      return new RibbonApplicationContextInitializer(springClientFactory(),
            ribbonEagerLoadProperties.getClients());
   }

   @Configuration(proxyBeanMethods = false)
   @ConditionalOnClass(HttpRequest.class)
   @ConditionalOnRibbonRestClient
   protected static class RibbonClientHttpRequestFactoryConfiguration {

      @Autowired
      private SpringClientFactory springClientFactory;

      @Bean
      public RestTemplateCustomizer restTemplateCustomizer(
            final RibbonClientHttpRequestFactory ribbonClientHttpRequestFactory) {
         return restTemplate -> restTemplate
               .setRequestFactory(ribbonClientHttpRequestFactory);
      }

      @Bean
      public RibbonClientHttpRequestFactory ribbonClientHttpRequestFactory() {
         return new RibbonClientHttpRequestFactory(this.springClientFactory);
      }

   }
   
   ..........
```



![image-20201203095936336](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201203095936336-1606960776-6bb920.png)



```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RestTemplate.class)
@ConditionalOnBean(LoadBalancerClient.class)
@EnableConfigurationProperties(LoadBalancerRetryProperties.class)
public class LoadBalancerAutoConfiguration {

   //这里会注入所有加了@LoadBalanced注解的RestTemplate
   //因为@LoadBalanced注解中有@Qualifire注解，Spring特性
   @LoadBalanced
   @Autowired(required = false)
   private List<RestTemplate> restTemplates = Collections.emptyList();

   @Autowired(required = false)
   private List<LoadBalancerRequestTransformer> transformers = Collections.emptyList();

    //03注入一个SmartInitializingSingleton
    //主要作用是将加了@LoadBalanced注解的所有RestTempLate调用02注入的RestTemplateCustomizer
    //也就是加入过滤器
    //这里只是单纯的注入SmartInitializingSingleton，SmartInitializingSingleton是spring的类
    //由spring在初始化的时候调用执行
   @Bean
   public SmartInitializingSingleton loadBalancedRestTemplateInitializerDeprecated(
         final ObjectProvider<List<RestTemplateCustomizer>> restTemplateCustomizers) {
      return () -> restTemplateCustomizers.ifAvailable(customizers -> {
         for (RestTemplate restTemplate : LoadBalancerAutoConfiguration.this.restTemplates) {
            for (RestTemplateCustomizer customizer : customizers) {
               customizer.customize(restTemplate);
            }
         }
      });
   }

   @Bean
   @ConditionalOnMissingBean
   public LoadBalancerRequestFactory loadBalancerRequestFactory(
         LoadBalancerClient loadBalancerClient) {
      return new LoadBalancerRequestFactory(loadBalancerClient, this.transformers);
   }

   @Configuration(proxyBeanMethods = false)
   @ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
   static class LoadBalancerInterceptorConfig {

      //01.注入RestTemplate的过滤器
      @Bean
      public LoadBalancerInterceptor ribbonInterceptor(
            LoadBalancerClient loadBalancerClient,
            LoadBalancerRequestFactory requestFactory) {
         return new LoadBalancerInterceptor(loadBalancerClient, requestFactory);
      }
	  
       
      //02.注入一个 RestTemplateCustomizer,
      //主要作用是将传入的RestTemplate加上01注入的过滤器
      @Bean
      @ConditionalOnMissingBean
      public RestTemplateCustomizer restTemplateCustomizer(
            final LoadBalancerInterceptor loadBalancerInterceptor) {
         return restTemplate -> {
            List<ClientHttpRequestInterceptor> list = new ArrayList<>(
                  restTemplate.getInterceptors());
            list.add(loadBalancerInterceptor);
            restTemplate.setInterceptors(list);
         };
      }

   }
    ..........
}
```

### Interceptor

```java
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {

   private LoadBalancerClient loadBalancer;

   private LoadBalancerRequestFactory requestFactory;

   public LoadBalancerInterceptor(LoadBalancerClient loadBalancer,
         LoadBalancerRequestFactory requestFactory) {
      this.loadBalancer = loadBalancer;
      this.requestFactory = requestFactory;
   }

   public LoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
      // for backwards compatibility
      this(loadBalancer, new LoadBalancerRequestFactory(loadBalancer));
   }

   @Override
   public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
         final ClientHttpRequestExecution execution) throws IOException {
      final URI originalUri = request.getURI();
      String serviceName = originalUri.getHost();
      Assert.state(serviceName != null,
            "Request URI does not contain a valid hostname: " + originalUri);
      //==============================过滤器关键地方===================================
      return this.loadBalancer.execute(serviceName,
            this.requestFactory.createRequest(request, body, execution));
   }
}


//===============this.requestFactory.createRequest(request, body, execution)
//使用函数表达式的方式返回一个LoadBalancerRequest对象
//该对象T apply(ServiceInstance instance) throws Exception;
//apply最后面的execute就是执行真正发送请求的方法
public LoadBalancerRequest<ClientHttpResponse> createRequest(
			final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) {
		return instance -> {
			HttpRequest serviceRequest = new ServiceRequestWrapper(request, instance,
					this.loadBalancer);
			if (this.transformers != null) {
				for (LoadBalancerRequestTransformer transformer : this.transformers) {
					serviceRequest = transformer.transformRequest(serviceRequest,
							instance);
				}
			}
			return execution.execute(serviceRequest, body);
		};
	}
//=================this.loadBalancer.execute
public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint)
			throws IOException {
    	//从工厂获取ILoadBalancer
    	//默认注入为ZoneAwareLoadBalancer extends DynamicServerListLoadBalancer
		ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
    	//就是调用loadBalancer.chooseServer方法
		Server server = getServer(loadBalancer, hint);
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}
		RibbonServer ribbonServer = new RibbonServer(serviceId, server,
				isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));

		return execute(serviceId, ribbonServer, request);
	}

//============================execute
@Override
	public <T> T execute(String serviceId, ServiceInstance serviceInstance,
			LoadBalancerRequest<T> request) throws IOException {
		Server server = null;
		if (serviceInstance instanceof RibbonServer) {
			server = ((RibbonServer) serviceInstance).getServer();
		}
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}

		RibbonLoadBalancerContext context = this.clientFactory
				.getLoadBalancerContext(serviceId);
		RibbonStatsRecorder statsRecorder = new RibbonStatsRecorder(context, server);

		try {
            //===========================那刚才封装的LoadBalancerRequest去调用apply方法
			T returnVal = request.apply(serviceInstance);
			statsRecorder.recordStats(returnVal);
			return returnVal;
		}
		// catch IOException and rethrow so RestTemplate behaves correctly
		catch (IOException ex) {
			statsRecorder.recordStats(ex);
			throw ex;
		}
		catch (Exception ex) {
			statsRecorder.recordStats(ex);
			ReflectionUtils.rethrowRuntimeException(ex);
		}
		return null;
	}
//====================createRequest
//所以现在关键又来到LoadBalancerRequest 如何封装的真正请求
//最后真正发送请求 传参是一个Wrapper

public class ServiceRequestWrapper extends HttpRequestWrapper {

	private final ServiceInstance instance;

	private final LoadBalancerClient loadBalancer;

	public ServiceRequestWrapper(HttpRequest request, ServiceInstance instance,
			LoadBalancerClient loadBalancer) {
		super(request);
		this.instance = instance;
		this.loadBalancer = loadBalancer;
	}

    
    //这里就是将服务名换成真正的url
	@Override
	public URI getURI() {
		URI uri = this.loadBalancer.reconstructURI(this.instance, getRequest().getURI());
		return uri;
	}

}
public class HttpRequestWrapper implements HttpRequest {
    private final HttpRequest request;

    public HttpRequestWrapper(HttpRequest request) {
        Assert.notNull(request, "HttpRequest must not be null");
        this.request = request;
    }

    public HttpRequest getRequest() {
        return this.request;
    }

    @Nullable
    public HttpMethod getMethod() {
        return this.request.getMethod();
    }

    public String getMethodValue() {
        return this.request.getMethodValue();
    }

    public URI getURI() {
        return this.request.getURI();
    }

    public HttpHeaders getHeaders() {
        return this.request.getHeaders();
    }
}

```

### 获取负载均衡器 getLoadBalancer

```java
protected ILoadBalancer getLoadBalancer(String serviceId) {
   return this.clientFactory.getLoadBalancer(serviceId);
}

public ILoadBalancer getLoadBalancer(String name) {
		return getInstance(name, ILoadBalancer.class);
}

public <C> C getInstance(String name, Class<C> type) {
		C instance = super.getInstance(name, type);
		if (instance != null) {
			return instance;
		}
		IClientConfig config = getInstance(name, IClientConfig.class);
		return instantiateWithConfig(getContext(name), type, config);
	}

//super.getInsttance()
//从子容器里面获取！！！！
//第一次没有获取到子容器上下文，就创建一个
//能够实现配置隔离的根本原因
 public <T> T getInstance(String name, Class<T> type) {
        AnnotationConfigApplicationContext context = this.getContext(name);
        return BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context, type).length > 0 ? context.getBean(type) : null;
    }
//创建上下文
protected AnnotationConfigApplicationContext createContext(String name) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    	//这里的configuration 就是刚才注入SpringClientFactory的时候传进来的
    
    	//这就是@RibbonClient指定的配置
		if (this.configurations.containsKey(name)) {
			for (Class<?> configuration : this.configurations.get(name)
					.getConfiguration()) {
				context.register(configuration);
			}
		}
    	//@RibbonClients指定的默认配置
		for (Map.Entry<String, C> entry : this.configurations.entrySet()) {
			if (entry.getKey().startsWith("default.")) {
				for (Class<?> configuration : entry.getValue().getConfiguration()) {
					context.register(configuration);
				}
			}
		}
		context.register(PropertyPlaceholderAutoConfiguration.class,
				this.defaultConfigType);
		context.getEnvironment().getPropertySources().addFirst(new MapPropertySource(
				this.propertySourceName,
				Collections.<String, Object>singletonMap(this.propertyName, name)));
		if (this.parent != null) {
			// Uses Environment from parent as well as beans
			context.setParent(this.parent);
			// jdk11 issue
			// https://github.com/spring-cloud/spring-cloud-netflix/issues/3101
			context.setClassLoader(this.parent.getClassLoader());
		}
		context.setDisplayName(generateDisplayName(name));
		context.refresh();
		return context;
	}
```

### LoadBalancer

![image-20201204111250355](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204111250355-1607051570-0308a0.png)

```java
public DynamicServerListLoadBalancer(IClientConfig clientConfig, IRule rule, IPing ping,
                                     ServerList<T> serverList, ServerListFilter<T> filter,
                                     ServerListUpdater serverListUpdater) {
    //Iping 定期发送心跳，检测ServerList存活
    //IRule 负载均衡规则
    //serverList 实力列表缓存
    super(clientConfig, rule, ping);
    this.serverListImpl = serverList;
    this.filter = filter;
    this.serverListUpdater = serverListUpdater;
    if (filter instanceof AbstractServerListFilter) {
        ((AbstractServerListFilter) filter).setLoadBalancerStats(getLoadBalancerStats());
    }
    restOfInit(clientConfig);
}
```

### 选择server

LoadBalancer的chooseServer实现

```java
public Server chooseServer(Object key) {
    if (counter == null) {
        counter = createCounter();
    }
    counter.increment();
    if (rule == null) {
        return null;
    } else {
        try {
            return rule.choose(key);
        } catch (Exception e) {
            logger.warn("LoadBalancer [{}]:  Error choosing server for key {}", name, key, e);
            return null;
        }
    }
}

public interface IRule{
    /*
     * choose one alive server from lb.allServers or
     * lb.upServers according to key
     * 
     * @return choosen Server object. NULL is returned if none
     *  server is available 
     */

    public Server choose(Object key);
    
    public void setLoadBalancer(ILoadBalancer lb);
    
    public ILoadBalancer getLoadBalancer();    
}
```

Ping

```java
void setupPingTask() {
    if (canSkipPing()) {
        return;
    }
    if (lbTimer != null) {
        lbTimer.cancel();
    }
    lbTimer = new ShutdownEnabledTimer("NFLoadBalancer-PingTimer-" + name,
            true);
    //默认30s
    lbTimer.schedule(new PingTask(), 0, pingIntervalSeconds * 1000);
    forceQuickPing();
}

class PingTask extends TimerTask {
        public void run() {
            try {
            	new Pinger(pingStrategy).runPinger();
            } catch (Exception e) {
                logger.error("LoadBalancer [{}]: Error pinging", name, e);
            }
        }
    }

class Pinger {

        private final IPingStrategy pingerStrategy;

        public Pinger(IPingStrategy pingerStrategy) {
            this.pingerStrategy = pingerStrategy;
        }

        public void runPinger() throws Exception {
            if (!pingInProgress.compareAndSet(false, true)) { 
                return; // Ping in progress - nothing to do
            }
            
            // we are "in" - we get to Ping

            Server[] allServers = null;
            boolean[] results = null;

            Lock allLock = null;
            Lock upLock = null;

            try {
                /*
                 * The readLock should be free unless an addServer operation is
                 * going on...
                 */
                allLock = allServerLock.readLock();
                allLock.lock();
                allServers = allServerList.toArray(new Server[allServerList.size()]);
                allLock.unlock();

                int numCandidates = allServers.length;
                results = pingerStrategy.pingServers(ping, allServers);

                final List<Server> newUpList = new ArrayList<Server>();
                final List<Server> changedServers = new ArrayList<Server>();

                for (int i = 0; i < numCandidates; i++) {
                    boolean isAlive = results[i];
                    Server svr = allServers[i];
                    boolean oldIsAlive = svr.isAlive();

                    svr.setAlive(isAlive);

                    if (oldIsAlive != isAlive) {
                        changedServers.add(svr);
                        logger.debug("LoadBalancer [{}]:  Server [{}] status changed to {}", 
                    		name, svr.getId(), (isAlive ? "ALIVE" : "DEAD"));
                    }

                    if (isAlive) {
                        newUpList.add(svr);
                    }
                }
                upLock = upServerLock.writeLock();
                upLock.lock();
                upServerList = newUpList;
                upLock.unlock();

                notifyServerStatusChangeListener(changedServers);
            } finally {
                pingInProgress.set(false);
            }
        }
    }
```

Ribbon对实例的缓存

DynamicServerListLoadBalancer.ServerList

eureka整合下是DiscoveryEnabledNIWSServerList

![image-20201204143100213](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204143100213-1607063460-3dd31c.png)

初始化的时候会从EurekaClient拉取数据

Ribbon对实例缓存的更新

DynamicServerListLoadBalancer  -> ServerListUpdater.UpdateAction updateAction  更新的具体流程

DynamicServerListLoadBalancer -> ServerListUpdater serverListUpdater 更新器，用来执行updateAction

**updateAction**  在 Eureka整合情况下，默认为DiscoveryEnabledNIWSServerList，就是从EurekaClient获取列表

**serverListUpdater**  默认实现是 PollingServerListUpdater 新开线程定时去执行updateAction,**这也就成了常说的Ribbon和EurekaClient之间还有30s的延迟！！！**

![image-20201204144058587](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204144058587-1607064058-5c8908.png)

**serverListUpdater**还有一个实现 **EurekaNotificationServerListUpdater**，这个的原理就是将updateAction封装成一个EurekaEventListener，**当EurekaClient发生变化的时候通知修改，没有延迟**。

![image-20201204144830256](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204144830256-1607064510-f56b8c.png)

一些默认的配置类

RibbonClientConfiguration

EurekaRibbonClientConfiguration

........

### IRule实现

- RoundRobinRule：轮询；
- RandomRule：随机；
- AvailabilityFilteringRule：会先过滤掉由于多次访问故障而处于断路器状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问；
- WeightedResponseTimeRule：根据平均响应时间计算所有服务的权重，响应时间越快的服务权重越大被选中的概率越大。刚启动时如果统计信息不足，则使用RoundRobinRule（轮询）策略，等统计信息足够，会切换到WeightedResponseTimeRule；
- RetryRule：先按照RoundRobinRule（轮询）策略获取服务，如果获取服务失败则在指定时间内进行重试，获取可用的服务；
- BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务；
- ZoneAvoidanceRule：复合判断Server所在区域的性能和Server的可用性选择服务器；

### Iping实现类

![image-20201204150910517](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201204150910517-1607065750-c48d75.png)