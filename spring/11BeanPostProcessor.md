# BeanPostProcessor
## 几个比较重要的BeanPostProcessor
### AutowiredAnnotationBeanPostProcessor 
* 能够实现对@Autowire @Value @Lookup的解析
* "context:annotation-config" and "context:component-scan" 也会注入注入该后置处理器
* 注解注入会先于xml,后解析的覆盖先前解析的
* 调用栈：doCreateBean->populateBean->AutowiredAnnotationBeanPostProcessor.postProcessProperties
->injectMetaData.inject->AutowiredAnnotationBeanPostProcessor$AutowiredFildElement.inject
->DefaultListableBeanFactory.resolveDependency->DefaultListableBeanFactory.doResoveDependency()<支持Stream，Map,Array,Collection等注入>
->resovleCandicate()<单个注入>->getBean()

### CommonAnnotationBeanPostProcessor
* 支持 @Resource，@PostConstruct， @PreDestory ，jndi等等
* @Resource注入流程与AutowiredAnnotationBeanPostProcessor   差不多，最终调用到AbstractAutowireCapableBeanFactory.resolveBeanByName(String name, DependencyDescriptor descriptor) 
也会调用到getBean
* @PostConstruct， @PreDestory由父类InitDestroyAnnotationBeanPostProcessor支持

### ApplicationContextAwareProcessor
* 能注入:
    1. EnvironmentAware
    1. EmbeddedValueResolverAware
    1. ResourceLoaderAware
    1. ApplicationEventPublisherAware
    1. MessageSourceAware
    1. ApplicationContextAware
* 在initializeBean中调用了BeanNameAware BeanClassLoaderAware BeanFactoryAware

### ImportAwarBeanPostProcessor
1. 对EnhancedConfiguration设置BeanFactory
2. 对ImportAwar支持 `void setImportMetadata(AnnotationMetadata importMetadata);`

### AsyncAnnotationBeanPostProcessor
1. @EnableAsync
2. @Async

### ApplicationListenerDetector
* postProcessAfterInitialization添加Listener
* postProcessBeforeDestruction移除Listener

### AnnotationAwareAspectJAutoProxyCreator
* Spring aop

## BeanPostProcessor的子接口以及方法
### BeanPostProcessor
- Object postProcessBeforeInitialization(Object bean, String beanName) 
- Object postProcessAfterInitialization(Object bean, String beanName) 

### InstantiationAwareBeanPostProcessor extends BeanPostProcessor
- Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) 
- boolean postProcessAfterInstantiation(Object bean, String beanName) 
- PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
- PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) 

### SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor
- Class<?> predictBeanType(Class<?> beanClass, String beanName) 
- Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName) 
- Object getEarlyBeanReference(Object bean, String beanName) 

### MergedBeanDefinitionPostProcessor extends BeanPostProcessor
- void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName); 
- void resetBeanDefinition(String beanName);

## 循环依赖解决方案
- "A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。
- A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，
- 此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，
- 发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，
- 尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，
- 尝试二级缓存earlySingletonObjects（也没有），
- 尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，
- 所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，
- B拿到A对象后顺利完成了初始化阶段1、2、3，
- 完全初始化之后将自己放入到一级缓存singletonObjects中。
- 此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，
- 长大成人，进去了一级缓存singletonObjects中，而且更加幸运的是，
- 由于B拿到了A的对象引用，所以B现在hold住的A对象也蜕变完美了。

### 为什么需要三级缓存，按理来说二级缓存就能实现
- ObjectFactory.getObject()实际上就是遍历SmartInstantiationAwareBeanPostProcessor.getEarlyBeanReference()
- 能够实现动态代理等，返回的对象可能不是原对象。
- 为什么不像动态代理，再填充属性？
- 先动态代理过后，就没有办法属性注入了(解析不到父类注解)

## getSingleton主要流程
- 标记beanName正在创建中
- 调用Object InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation(Class<?> beanClass, String beanName),
- 如果Object!=null  调用InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation(Object bean, String beanName) 
- 如果返回的Object依旧不为空,直接返回放到单例池
- 实例化Object(先判断是否用FactoryMethod),
- 没有的话,调用SmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors拿到构造器,判断拿到的构造器是不是null,或者说是AUTOWIRE_CONSTRUCTOR,按构造器自动注入,调用有参的构造器,如果多个构造器,还要比较使用哪个,不然则使用无参默认构造器,最终拿到实例对象
- 实例化完成后调用void MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName); 解析注解
- 然后把该实例放到SingletonFactory中,也就是三级缓存
- 然后填充属性(populateBean)
- 调用InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation(Object bean, String beanName) 返回false就不填充bean了,直接return
- 解析AUTOWIRE_BY_NAME,AUTOWIRE_BY_TYPE
- 调用InstantiationAwareBeanPostProcessor.postProcessProperties(PropertyValues pvs, Object bean, String beanName)
- 调用InstantiationAwareBeanPostProcessor.postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) <废弃方法>
- 然后initializeBean
- 调用三个Aware接口
- 调用BeanPostProcessor.postProcessBeforeInitialization()
- 调用InitializingBean接口,和initMethod
- 调用BeanPostProcessor.postProcessAfterInitialization
- DisposableBean放到集合
- 移除正在创建中的标记
- 放进单例池
- 返回FactoryBean或bean


- 初始化的最后会执行SmartInitializingSingleton.afterSingletonsInstantiated()