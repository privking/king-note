# Druid配置多数据源

## 单数据源

```yaml
spring:
  datasource:
    url: jdbc:mysql://1xxxxxxxxxxxxxxxxx
    username: root
    password: 'xxxxx'
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
          config:
            multi-statement-allow: true
```

## 多数据源

```yaml
spring:
  datasource:
    db1:
      url: jdbc:mysql://xxxxxxxxxxxxxxx
      username: root
      password: 'xxxx'
      driver-class-name: com.mysql.cj.jdbc.Driver
      initial-size: 5
      max-active: 20
      time-between-eviction-runs-millis: 100000
      min-evictable-idle-time-millis: 200000
      filter:
        stat:
          enabled: true
          log-slow-sql: true
          db-type: sqlserver
          slow-sql-millis: 10000
        config:
          enabled: true
        encoding:
          enabled: true
        wall:
          enabled: true
          config:
            multi-statement-allow: true
    db2:
      url: jdbc:sqlserver://xxxxxxxxxxxxxxxxx
      username: root
      password: 'xxxxx'
      driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
      initial-size: 5
      max-active: 20
      time-between-eviction-runs-millis: 100000
      min-evictable-idle-time-millis: 200000
      filter:
        stat:
          enabled: true
          log-slow-sql: true
          db-type: sqlserver
          slow-sql-millis: 10000
        config:
          enabled: true
        encoding:
          enabled: true
        wall:
          enabled: true
          config:
            multi-statement-allow: true
```

###  DataSourceConfig.java

```yaml
@Configuration
public class DataSourceConfig {
	
	//ConfigurationProperties注解 指定配置文件中的前缀
	//将过滤后的字段注入到返回结果的对象中
    @Bean(name = "db1")
    @ConfigurationProperties(prefix = "spring.datasource.db1")
    public DataSource db1() {
       // return DataSourceBuilder.create().build();
        return new DruidDataSource();
    }

    @Bean(name = "db2")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    public DataSource db2() {
       // return DataSourceBuilder.create().build();
        return new DruidDataSource();
    }

}
```

### Db1Config

```java
@Configuration
@MapperScan(basePackages = {"com.bigdatacd.main.overview.mapper.db1"}, sqlSessionFactoryRef = "sqlSessionFactoryDb1")
public class Db1Config {

    @Autowired
    @Qualifier("db1")
    private DataSource dataSourceDb1;


	//ConfigurationProperties注解 不行 
	//因为MapperLocations 需要的是所有xml数组，但是配置文件中为匹配文件，不是具体文件数组
    @Bean
    public SqlSessionFactory sqlSessionFactoryDb1() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceDb1);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/db1/*.xml"));
        factoryBean.setTypeAliasesPackage("com.bigdatacd.main.overview.domain");
        org.apache.ibatis.session.Configuration con  = new org.apache.ibatis.session.Configuration();
        con.setLogImpl(StdOutImpl.class);
        con.setMapUnderscoreToCamelCase(true);
        con.setAutoMappingBehavior(AutoMappingBehavior.FULL);
        con.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.NONE);
        factoryBean.setConfiguration(con);
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplateDb1() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactoryDb1());
    }
}
```

### Db2Config

```java
@Configuration
@MapperScan(basePackages = {"com.bigdatacd.main.overview.mapper.db2"}, sqlSessionFactoryRef = "sqlSessionFactoryDb2")
public class Db2Config {

    @Autowired
    @Qualifier("db2")
    private DataSource dataSourceDb2;


    @Bean
    public SqlSessionFactory sqlSessionFactoryDb2() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceDb2);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/db2/*.xml"));
        factoryBean.setTypeAliasesPackage("com.bigdatacd.main.overview.domain");
        org.apache.ibatis.session.Configuration con  = new org.apache.ibatis.session.Configuration();
        con.setLogImpl(StdOutImpl.class);
        con.setMapUnderscoreToCamelCase(true);
        con.setAutoMappingBehavior(AutoMappingBehavior.FULL);
        con.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.NONE);
        factoryBean.setConfiguration(con);
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplateDb2() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactoryDb2());
    }

}
```

### App

```java
@SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class
})
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 常见配置

| 配置                                      | 缺省值             | 说明                                                         |
| ----------------------------------------- | ------------------ | ------------------------------------------------------------ |
| name                                      |                    | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是："DataSource-" + System.identityHashCode(this). 另外配置此属性至少在1.0.5版本中是不起作用的，强行设置name会出错。 |
| url                                       |                    | 连接数据库的url，不同数据库不一样。例如： mysql : jdbc:mysql://10.20.153.104:3306/druid2 oracle : jdbc:oracle:thin:@10.20.149.85:1521:ocnauto |
| username                                  |                    | 连接数据库的用户名                                           |
| password                                  |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。 |
| driverClassName                           | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName |
| initialSize                               | 0                  | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                                 | 8                  | 最大连接池数量                                               |
| maxIdle                                   | 8                  | 已经不再使用，配置了也没效果                                 |
| minIdle                                   |                    | 最小连接池数量                                               |
| maxWait                                   |                    | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements                    | false              | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxPoolPreparedStatementPerConnectionSize | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| validationQuery                           |                    | 用来检测连接是否有效的sql，要求是一个查询语句，常用select 'x'。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。 |
| validationQueryTimeout                    |                    | 单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法 |
| testOnBorrow                              | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                              | false              | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testWhileIdle                             | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| keepAlive                                 | false （1.0.28）   | 连接池中的minIdle数量以内的连接，空闲时间超过minEvictableIdleTimeMillis，则会执行keepAlive操作。 |
| timeBetweenEvictionRunsMillis             | 1分钟（1.0.14）    | 有两个含义： 1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。 2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明 |
| numTestsPerEvictionRun                    | 30分钟（1.0.14）   | 不再使用，一个DruidDataSource只支持一个EvictionRun           |
| minEvictableIdleTimeMillis                |                    | 连接保持空闲而不被驱逐的最小时间                             |
| connectionInitSqls                        |                    | 物理连接初始化的时候执行的sql                                |
| exceptionSorter                           | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                                   |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat 日志用的filter:log4j 防御sql注入的filter:wall |
| proxyFilters                              |                    | 类型是List<com.alibaba.druid.filter.Filter>，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系 |