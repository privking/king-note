# spring 基础

## 什么是IOC
* 控制反转（inversion of control），面向对象编程中的一种设计原则，可以用来降低代码的耦合度。其中最常见的方式是依赖注入（Dependency Injection,DI），还有一种方式叫做“依赖查找”（Dependency Lookip）
* DI是实现IOC的一种手段

## 注入的两种方法
* Constructor-based Dependency Injection
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599202196-21fc02.png)

* Setter-based Dependency Injection
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599202249-4fdbde.png)


## 自动装配的方法
1. no默认不进行自动装配，通过显性设置ref属性来进行装配
2. byName通过参数名进行自动装配，Spring容器在配置文件中发现bean的autowire属性被设置为byName之后容器试图匹配
3. byType，通过参数类型自动装配
4. constructor：类似byType，但是要提供给构造器参数，若没有确定的带参数的构造器参数类型，将会抛出异常
5. autodetect：先尝试使用constructor来自动装配，如果无法工作，则使用byType方式

## @Autoware和@Resource区别
* @Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入
* @Resource装配顺序
    * 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
    * 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
    * 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
    * 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；
* @Autoware允许依赖对象为null
    * @Autowire(required=false)
* 如果使用按照名称(by-name)装配，需结合@Qualifier注解使用，即
    * @Autowire @Qualifier("beanName")
* @Resource、@PostConstruct以及@PreDestroy都是JSR-250标准
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599202318-4d8991.png)

## 单例依赖原型
* 引入applicationcontext，applicationcontext.getBean("name");
* @Lookup方法注入
    * 在引用bean的类里面，提供一个被引用bean的get抽象方法（spring会根据这个方法去找 注意方法名要和被引用的bean一样），然后方法上加@Lookup注解
    
    ```java
    @Component
    public abstract class SingletonBean {
        private static final Logger logger = LoggerFactory.getLogger(SingletonBean.class);
        
        public void print() {
            PrototypeBean bean = methodInject();
            logger.info("Bean SingletonBean's HashCode : {}",bean.hashCode());
            bean.say();
        }
        // 也可以写成 @Lookup("prototypeBean") 来指定需要注入的bean
        @Lookup
        protected abstract PrototypeBean methodInject();
    }
    ```

## Using Filters to Customize Scanning
* 注解
```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```
* xml

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203269-18c2bb.png)

## Generating an Index of Candidate Components
* 虽然类路径扫描非常快，但是可以通过在编译时创建静态候选列表来提高大型应用程序的启动性能。在这种模式下，所有作为组件扫描目标的模块都必须使用这种机制
* 现有的@ComponentScan或<context:component-scan指令必须保持不变，以便请求上下文扫描某些包中的候选者。当ApplicationContext检测到这样的索引时，它会自动使用它，而不是扫描类路径
* 要生成索引，需要向每个模块添加一个附加依赖项，其中包含的组件是组件扫描指令的目标。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.2.5.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

## @Profile
* Bean定义配置文件在核心容器中提供了一种机制，允许在不同的环境中注册不同的Bean
  ![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599202371-0b8f53.png)

  ![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599202416-7af301.png)

  ![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599202499-c30863.png)
## beanNameGenerator
```java
public interface BeanNameGenerator {
	String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
}
```
* BeanNameGenerator就一个方法声明，generateBeanName的声明并不复杂，传递BeanDefinition与BeanDefinitionRegistry 返回一个string类型的beanname。

### DefaultBeanNameGenerator

```java
public class DefaultBeanNameGenerator implements BeanNameGenerator {

	@Override
	public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
		return BeanDefinitionReaderUtils.generateBeanName(definition, registry);
	}

}
```

```java
public static String generateBeanName(BeanDefinition beanDefinition, BeanDefinitionRegistry registry) throws BeanDefinitionStoreException {
	return generateBeanName(beanDefinition, registry, false);
}

public static String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry, boolean isInnerBean) throws BeanDefinitionStoreException {
	// ((Class<?>) beanClassObject).getName() 返回的是 class的完全限定名
	// 也可能是类名
	String generatedBeanName = definition.getBeanClassName();
	if (generatedBeanName == null) {
		if (definition.getParentName() != null) {
			//当generatedBeanName为null,parentName不为空。命名方式为parentName+"$child"
			generatedBeanName = definition.getParentName() + "$child";
		}else if (definition.getFactoryBeanName() != null) {
			//当generatedBeanName为null,FactoryBeanName不为空。命名方式为FactoryBeanName+"$child"
			generatedBeanName = definition.getFactoryBeanName() + "$created";
		}
	}
	if (!StringUtils.hasText(generatedBeanName)) {
		throw new BeanDefinitionStoreException("Unnamed bean definition specifies neither " +"'class' nor 'parent' nor 'factory-bean' - can't generate bean name");
	}

	String id = generatedBeanName;
	// generatedBeanName + “#” + value
	// isInnerBean 为true.使用系统identityHashCode作为value，false使用自增的方法作为value
	if (isInnerBean) {
		// Inner bean: generate identity hashcode suffix.
		id = generatedBeanName + GENERATED_BEAN_NAME_SEPARATOR + ObjectUtils.getIdentityHexString(definition);
	}else {
		int counter = -1;
		// 到容器里面看看是否存在同样名字的BeanDefinition
		while (counter == -1 || registry.containsBeanDefinition(id)) {
			counter++;
			id = generatedBeanName + GENERATED_BEAN_NAME_SEPARATOR + counter;
		}
	}
	return id;
}
```

### AnnotationBeanNameGenerator

```java
public class AnnotationBeanNameGenerator implements BeanNameGenerator {

	private static final String COMPONENT_ANNOTATION_CLASSNAME = "org.springframework.stereotype.Component";

	@Override
	public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
		// 判断是AnnotatedBeanDefinition的实现，就从annotation获得。
		if (definition instanceof AnnotatedBeanDefinition) {
			String beanName = determineBeanNameFromAnnotation((AnnotatedBeanDefinition) definition);
			// 是文本就返回这个beanName,但是也有可能annotation的value是null，就后从buildDefaultBeanName获得
			if (StringUtils.hasText(beanName)) {
				return beanName;
			}
		}
		return buildDefaultBeanName(definition, registry);
	}

	protected String determineBeanNameFromAnnotation(AnnotatedBeanDefinition annotatedDef) {
		// 获得类或者方法上所有的Annotation
		AnnotationMetadata amd = annotatedDef.getMetadata();
		// 得到所有annotation的类名
		Set<String> types = amd.getAnnotationTypes();
		String beanName = null;
		for (String type : types) {
			// 把annotation里面的字段与value，解读出来成map，字段名是key，value为value
			AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(amd, type);
			// 判断annotation是否有效，是否存在作为beanName的字段有value
			if (isStereotypeWithNameValue(type, amd.getMetaAnnotationTypes(type), attributes)) {
				// 从注解中获得value字段的值，
				Object value = attributes.get("value");
				if (value instanceof String) {
					String strVal = (String) value;
					if (StringUtils.hasLength(strVal)) {
						if (beanName != null && !strVal.equals(beanName)) {
							throw new IllegalStateException("Stereotype annotations suggest inconsistent " +"component names: '" + beanName + "' versus '" + strVal + "'");
						}
						beanName = strVal;
					}
				}
			}
		}
		return beanName;
	}

	
	protected boolean isStereotypeWithNameValue(String annotationType,Set<String> metaAnnotationTypes, Map<String, Object> attributes) {
		// 判断annotation的类型是否是这三种.
		// org.springframework.stereotype.Component
		// javax.annotation.ManagedBean
		// javax.inject.Named
		
		boolean isStereotype = annotationType.equals(COMPONENT_ANNOTATION_CLASSNAME) ||
				(metaAnnotationTypes != null && metaAnnotationTypes.contains(COMPONENT_ANNOTATION_CLASSNAME)) ||
				annotationType.equals("javax.annotation.ManagedBean") ||
				annotationType.equals("javax.inject.Named");
		// 并且value存在值。才会返回true
		return (isStereotype && attributes != null && attributes.containsKey("value"));
	}

	
	protected String buildDefaultBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {
		return buildDefaultBeanName(definition);
	}

	
	protected String buildDefaultBeanName(BeanDefinition definition) {
		// 获得类名
		String shortClassName = ClassUtils.getShortName(definition.getBeanClassName());
		// 把类名第一个字母大写转小写
		return Introspector.decapitalize(shortClassName);
	}

}
```
### 往ApplicationContext注册BeanNameGeneratord对象

```java
//AnnotationConfigApplicationContext
public void setBeanNameGenerator(BeanNameGenerator beanNameGenerator) {
		this.reader.setBeanNameGenerator(beanNameGenerator);
		this.scanner.setBeanNameGenerator(beanNameGenerator);
		getBeanFactory().registerSingleton(
				AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
	}
```

## 引入applicationContext的方法
* 直接注入

```java
@Resource
private ApplicationContext ctx;
```

* 实现ApplicationContextAware接口
    * 创建一个实体类并实现ApplicationContextAware接口，重写接口内的setApplicationContext方法来完成获取ApplicationContext实例的方法
* 在自定义AutoConfiguration中获取
```java
@Configuration
@EnableFeignClients("com.yidian.data.interfaces.client")
public class FeignAutoConfiguration {

    FeignAutoConfiguration(ApplicationContext context) {
        // 在初始化AutoConfiguration时会自动传入ApplicationContext
         doSomething(context);
    }
}
```
* 启动时获取ApplicationContext

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class WebApplication {

    private static ApplicationContext applicationContext;

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(WebApplication.class, args);
        SpringBeanUtil.setApplicationContext(applicationContext);
    }
}
```
* 通过WebApplicationContextUtils获取
```java
WebApplicationContextUtils.getRequiredWebApplicationContext(ServletContext sc);
WebApplicationContextUtils.getWebApplicationContext(ServletContext sc);
```