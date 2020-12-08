# Fegin

Feign是一个声明式WebService客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用方法是定义一个接口，然后在上面添加注解，同时也支持JAX-RS标准的注解。Feign也支持可拔插式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters。Feign可以与Eureka和Ribbon组合使用以支持负载均衡。

## Fegin的使用

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class Consumer {
    public static void main(String[] args) {
        SpringApplication.run(Consumer.class,args);
    }
}

@FeignClient("producer")
public interface HelloClient {

    @GetMapping("/hello")
    public Object hello(@RequestParam("name") String name);
}

@RestController
public class HelloClientController {

    @Autowired
    private RestTemplate restTemplate;


    @Autowired
    private HelloClient helloClient;

    @GetMapping("/hello")
    public Object sayHello(String name) {
        //return restTemplate.getForObject("http://producer/hello?name={n}", Object.class,name);
        //等价于
        HashMap<String, String> params = new HashMap<>();
        params.put("name",name);
        return restTemplate.getForObject("http://producer/hello?name={name}", Object.class,params);
    }

    @GetMapping("/hello2")
    public Object sayHello2(String name){
        return helloClient.hello(name);
    }
}
```

## 大致原理

### 先看几个依靠springboot特性注入的几个bean

![image-20201207135148391](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201207135148391-1607320315-5722d5.png)

![image-20201207135219797](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201207135219797-1607320339-e8be37.png)

### 注解@EnableFeignClients

EnableFeignClients可以设置

- value:basePackages的别名
- basePackages：扫描的包范围
- basePackageClasses：Type-safe alternative to {@link #basePackages()}  也是指类的所在包为basePackages
- defaultConfiguration：默认配置文件
- clients：手动指定@FeignClient 不用扫描

![image-20201207135329725](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201207135329725-1607320409-1007d5.png)

### 注解@FeignClient

- value:同name
- contextId:This will be used as the bean name instead of name if present,
- name:服务名称
- qualifier:the <code>@Qualifier</code> value for the feign client.
- url：可以直接指定绝对路径，不用name这些
- decode404：404是否抛出异常，默认false
- configuration：每个client单独的配置，这里可以用来单独配置负载均衡策略等
- fallback：指定一个实现该接口的类，失败后回调的接口
- fallbackFactory：指定一个feign.hystrix.FallbackFactory，返回的必须是实现该接口的实例
- path：path prefix to be used by all method-level mappings
- primary：whether to mark the feign proxy as a primary bean，默认true

### @EnableFeignClients注入了一个FeignClientsRegistrar.class

```java
//ImportBeanDefinitionRegistrar	
@Override
public void registerBeanDefinitions(AnnotationMetadata metadata,
                                    BeanDefinitionRegistry registry) {
    //注入默认配置
    //将EnableFeignClients.defaultConfiguration指定类注入
    registerDefaultConfiguration(metadata, registry);
    //
    registerFeignClients(metadata, registry);
}
```

```java
public void registerFeignClients(AnnotationMetadata metadata,
      BeanDefinitionRegistry registry) {
    //创建一个扫描器ClassPathScanningCandidateComponentProvider
   ClassPathScanningCandidateComponentProvider scanner = getScanner();
   scanner.setResourceLoader(this.resourceLoader);
	
   Set<String> basePackages;
	
   Map<String, Object> attrs = metadata
         .getAnnotationAttributes(EnableFeignClients.class.getName());
    //过滤只扫描@FeignClient
   AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
         FeignClient.class);
    //有没有配置指定FeginClient
   final Class<?>[] clients = attrs == null ? null
         : (Class<?>[]) attrs.get("clients");
   if (clients == null || clients.length == 0) {
      scanner.addIncludeFilter(annotationTypeFilter);
       //value
       //basePackages
       //basePackageClasses
       //EnableFeignClients所在包
      basePackages = getBasePackages(metadata);
   }
   else {
      final Set<String> clientClasses = new HashSet<>();
      basePackages = new HashSet<>();
      for (Class<?> clazz : clients) {
         basePackages.add(ClassUtils.getPackageName(clazz));
         clientClasses.add(clazz.getCanonicalName());
      }
      AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
         @Override
         protected boolean match(ClassMetadata metadata) {
            String cleaned = metadata.getClassName().replaceAll("\\$", ".");
            return clientClasses.contains(cleaned);
         }
      };
      scanner.addIncludeFilter(
            new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
   }
	//扫描每一basePackage
   for (String basePackage : basePackages) {
      Set<BeanDefinition> candidateComponents = scanner
            .findCandidateComponents(basePackage);
      for (BeanDefinition candidateComponent : candidateComponents) {
         if (candidateComponent instanceof AnnotatedBeanDefinition) {
            // verify annotated class is an interface
            AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
            AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
            Assert.isTrue(annotationMetadata.isInterface(),
                  "@FeignClient can only be specified on an interface");

            Map<String, Object> attributes = annotationMetadata
                  .getAnnotationAttributes(
                        FeignClient.class.getCanonicalName());

            String name = getClientName(attributes);
             //注入配置FeignClientSpecification
            registerClientConfiguration(registry, name,
                  attributes.get("configuration"));
			//为每一个Client注入一个FactoryBean
            registerFeignClient(registry, annotationMetadata, attributes);
         }
      }
   }
}

private void registerFeignClient(BeanDefinitionRegistry registry,
			AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
		String className = annotationMetadata.getClassName();
    
    	//注入的是FeignClientFactoryBean
		BeanDefinitionBuilder definition = BeanDefinitionBuilder
				.genericBeanDefinition(FeignClientFactoryBean.class);
		validate(attributes);
		definition.addPropertyValue("url", getUrl(attributes));
		definition.addPropertyValue("path", getPath(attributes));
		String name = getName(attributes);
		definition.addPropertyValue("name", name);
		String contextId = getContextId(attributes);
		definition.addPropertyValue("contextId", contextId);
		definition.addPropertyValue("type", className);
		definition.addPropertyValue("decode404", attributes.get("decode404"));
		definition.addPropertyValue("fallback", attributes.get("fallback"));
		definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
		definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

		String alias = contextId + "FeignClient";
		AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

		boolean primary = (Boolean) attributes.get("primary"); // has a default, won't be
																// null

		beanDefinition.setPrimary(primary);

		String qualifier = getQualifier(attributes);
		if (StringUtils.hasText(qualifier)) {
			alias = qualifier;
		}

		BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
				new String[] { alias });
		BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
	}

```

### 注入的FactoryBean--> FeignClientFactoryBean

```java
@Override
public Object getObject() throws Exception {
   return getTarget();
}

<T> T getTarget() {
    FeignContext context = this.applicationContext.getBean(FeignContext.class);
    Feign.Builder builder = feign(context);

    if (!StringUtils.hasText(this.url)) {
        if (!this.name.startsWith("http")) {
            this.url = "http://" + this.name;
        }
        else {
            this.url = this.name;
        }
        this.url += cleanPath();
        return (T) loadBalance(builder, context,
                               new HardCodedTarget<>(this.type, this.name, this.url));
    }
    //url会覆盖，所以url优先级高
    if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
        this.url = "http://" + this.url;
    }
    String url = this.url + cleanPath();
    Client client = getOptional(context, Client.class);
    if (client != null) {
        if (client instanceof LoadBalancerFeignClient) {
            // not load balancing because we have a url,
            // but ribbon is on the classpath, so unwrap
            client = ((LoadBalancerFeignClient) client).getDelegate();
        }
        if (client instanceof FeignBlockingLoadBalancerClient) {
            // not load balancing because we have a url,
            // but Spring Cloud LoadBalancer is on the classpath, so unwrap
            client = ((FeignBlockingLoadBalancerClient) client).getDelegate();
        }
        builder.client(client);
    }
    Targeter targeter = get(context, Targeter.class);
    return (T) targeter.target(this, builder, context,
                               new HardCodedTarget<>(this.type, this.name, url));
}
//当上面url判断没有值的时候
protected <T> T loadBalance(Feign.Builder builder, FeignContext context,
			HardCodedTarget<T> target) {
    	//这里又是去子容器拿的
		Client client = getOptional(context, Client.class);
		if (client != null) {
			builder.client(client);
            //这里又比较关键了
            //返回的Target,就是最终注入的对象了
			Targeter targeter = get(context, Targeter.class);
			return targeter.target(this, builder, context, target);
		}

		throw new IllegalStateException(
				"No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-netflix-ribbon?");
	}

```

### DefaultTargeter

```java
@Override
public <T> T target(FeignClientFactoryBean factory, Feign.Builder feign,
      FeignContext context, Target.HardCodedTarget<T> target) {
   return feign.target(target);
    
}

  public <T> T target(Target<T> target) {
      return build().newInstance(target);
    }

    public Feign build() {
      SynchronousMethodHandler.Factory synchronousMethodHandlerFactory =
          new SynchronousMethodHandler.Factory(client, retryer, requestInterceptors, logger,
              logLevel, decode404, closeAfterDecode, propagationPolicy);
      ParseHandlersByName handlersByName =
          new ParseHandlersByName(contract, options, encoder, decoder, queryMapEncoder,
              errorDecoder, synchronousMethodHandlerFactory);
      return new ReflectiveFeign(handlersByName, invocationHandlerFactory, queryMapEncoder);
    }
  }

//========================== ReflectiveFeign extends Feign
public <T> T newInstance(Target<T> target) {
    //解析方法信息，比如@Param等等,解析成MethodHandler
    Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
    Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
    List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();

    //拿到所有的Method
    for (Method method : target.type().getMethods()) {
      if (method.getDeclaringClass() == Object.class) {
        continue;
      } else if (Util.isDefault(method)) {
          //是不是默认的
          //Default methods are public non-abstract, non-synthetic, and non-static instance methods
        DefaultMethodHandler handler = new DefaultMethodHandler(method);
        defaultMethodHandlers.add(handler);
        methodToHandler.put(method, handler);
      } else {
        methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
      }
    }
    
    //动态代理
    InvocationHandler handler = factory.create(target, methodToHandler);
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),
        new Class<?>[] {target.type()}, handler);

    for (DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
      defaultMethodHandler.bindTo(proxy);
    }
    return proxy;
  }
```

```java
static class FeignInvocationHandler implements InvocationHandler {

  private final Target target;
  private final Map<Method, MethodHandler> dispatch;

  FeignInvocationHandler(Target target, Map<Method, MethodHandler> dispatch) {
    this.target = checkNotNull(target, "target");
    this.dispatch = checkNotNull(dispatch, "dispatch for %s", target);
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if ("equals".equals(method.getName())) {
      try {
        Object otherHandler =
            args.length > 0 && args[0] != null ? Proxy.getInvocationHandler(args[0]) : null;
        return equals(otherHandler);
      } catch (IllegalArgumentException e) {
        return false;
      }
    } else if ("hashCode".equals(method.getName())) {
      return hashCode();
    } else if ("toString".equals(method.getName())) {
      return toString();
    }

    return dispatch.get(method).invoke(args);
  }
```

```java
//dispatch.get(method).invoke(args);
//SynchronousMethodHandler implements MethodHandler
public Object invoke(Object[] argv) throws Throwable {
  RequestTemplate template = buildTemplateFromArgs.create(argv);
  Options options = findOptions(argv);
  Retryer retryer = this.retryer.clone();
  while (true) {
    try {
      return executeAndDecode(template, options);
    } catch (RetryableException e) {
      try {
        retryer.continueOrPropagate(e);
      } catch (RetryableException th) {
        Throwable cause = th.getCause();
        if (propagationPolicy == UNWRAP && cause != null) {
          throw cause;
        } else {
          throw th;
        }
      }
      if (logLevel != Logger.Level.NONE) {
        logger.logRetry(metadata.configKey(), logLevel);
      }
      continue;
    }
  }
}
```

### HystrixTargeter

```java
@Override
public <T> T target(FeignClientFactoryBean factory, Feign.Builder feign,
      FeignContext context, Target.HardCodedTarget<T> target) {
   if (!(feign instanceof feign.hystrix.HystrixFeign.Builder)) {
      return feign.target(target);
   }
   feign.hystrix.HystrixFeign.Builder builder = (feign.hystrix.HystrixFeign.Builder) feign;
   String name = StringUtils.isEmpty(factory.getContextId()) ? factory.getName()
         : factory.getContextId();
   SetterFactory setterFactory = getOptional(name, context, SetterFactory.class);
   if (setterFactory != null) {
      builder.setterFactory(setterFactory);
   }
   Class<?> fallback = factory.getFallback();
   if (fallback != void.class) {
      return targetWithFallback(name, context, target, builder, fallback);
   }
   Class<?> fallbackFactory = factory.getFallbackFactory();
   if (fallbackFactory != void.class) {
      return targetWithFallbackFactory(name, context, target, builder,
            fallbackFactory);
   }

   return feign.target(target);
}
```

