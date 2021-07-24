# Spring-quickStart

## xml

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>xxxxx</groupId>
    <artifactId>xxxxxxxx</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>xxxxxx</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Greenwich.SR3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- logback -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.11</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.11</version>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.2</version>
        </dependency>

        <!-- swagger -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
            <exclusions>
                <exclusion>
                    <groupId>io.swagger</groupId>
                    <artifactId>swagger-annotations</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>io.swagger</groupId>
                    <artifactId>swagger-models</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-annotations</artifactId>
            <version>1.5.21</version>
        </dependency>
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-models</artifactId>
            <version>1.5.21</version>
        </dependency>
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>swagger-bootstrap-ui</artifactId>
            <version>1.9.6</version>
        </dependency>

        <!-- joda-time -->
        <dependency>
            <groupId>joda-time</groupId>
            <artifactId>joda-time</artifactId>
            <version>2.10.5</version>
        </dependency>

        <!-- http -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.5</version>
        </dependency>

        <!-- fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.60</version>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.13</version>
        </dependency>

        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.9</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>

     


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

## yaml

```yaml
server:
  port: 8080
  servlet:
    context-path: /xxxx



mybatis:
  configuration:
    map-underscore-to-camel-case: true
    auto-mapping-unknown-column-behavior: none
    auto-mapping-behavior: full
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: xxxxx.domain


spring:
  profiles:
    active: dev
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```

```yaml
spring:
  datasource:
    url: jdbc:mysql://xxx/xx?autoReconnect=true&useUnicode=true&createDatabaseIfNotExist=true&characterEncoding=utf8&serverTimezone=GMT%2B8
    username: root
    password: 'xxx'
    driver-class-name: com.mysql.cj.jdbc.Driver
    druid:
      username: root
      password: 'xxxx'
      initial-size: 5
      max-active: 20
      time-between-eviction-runs-millis: 100000
      min-evictable-idle-time-millis: 200000
      filter:
        stat:
          enabled: true
          log-slow-sql: true
          db-type: mysql
          slow-sql-millis: 10000
        config:
          enabled: true
        encoding:
          enabled: true
        wall:
          enabled: true
  servlet:
    multipart:
      max-request-size: 100MB
      max-file-size: 100MB

redis:
  database: 4
  host: xxxx.xx.xx.xx
  port: 6379
  timeout: 10000
  password: 'xxxxx'

jedis:
  pool:
    max-active: 10
    max-idle: 5
    min-idle: 2
    max-wait: 10000

eureka:
  client:
    serviceUrl:
      defaultZone: http://xxxxxxx:9000/eureka/

ribbon:
  ReadTimeout: 500000
  ConnectTimeout: 500000
```

## App

```java
@SpringBootApplication(scanBasePackages = "com.xxx")
@EnableEurekaClient
@EnableFeignClients
@EnableScheduling
@MapperScan(basePackages = "com.xxx.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

## Filter

```java
@Configuration
public class InterceptorConfigure implements WebMvcConfigurer {

    @Autowired
    private AppInterceptor appInterceptor; // HandlerInterceptorAdapter

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(appInterceptor).addPathPatterns("/**")
                .excludePathPatterns("/error")
                .excludePathPatterns("/file/upload")
            	//文件映射
                .excludePathPatterns("/swagger-resources/**", "/webjars/**", "/v2/**", "/swagger-ui.html/**","/doc.html", AppFilePath.FILE_BASE_FOLDER + "**");
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/resources/")
                .addResourceLocations("classpath:/static/");

        registry.addResourceHandler("doc.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        registry.addResourceHandler( AppFilePath.FILE_BASE_FOLDER + "**")
                .addResourceLocations("file:" + AppFilePath.FILE_BASE_PATH);
    }
}
```

## FeignClient

```java
@FeignClient(name = "server-name")
public interface GisServerClient {

    @GetMapping("/xxx")
    DoMain queryLonLat(@RequestParam("address") String address);

    @PostMapping("/xxxx")
    DoMain queryLonLatInfo(@RequestBody DoMain doomain);
}
```

