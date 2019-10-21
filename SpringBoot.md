# 一、 SpringBoot 入门
## 1、SpringBoot 简介
>简化Spring应用的一个框架；
>
>整个Spring技术栈的一个大整合；
>
>J2EE开发的一站式解决方案

## 2、 微服务

2014，Martin flower

微服务：架构风格

一个应用应该是由一组小型服务组成；可以通过http的方式进行通信

每一个功能元素最终都是一个可独立替换和独立升级的软件单元

[微服务文档](https://www.martinfowler.com/articles/microservices.html)

## 3、HelloWorld探究

### 1、 POM文件

#### 1、 父项目

~~~xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
它的父项目是
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
  </parent>
它来真正管理Spring Boot应用里面的所有依赖版本；
~~~

Spring Boot的版本仲裁中心；

导入在dependencies中声明版本号的依赖默认是不需要写版本号的

#### 2、导入的依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
~~~

spring-boot-starter-web:

>spring-boot-starter: spring-boot场景启动器
>
>帮我们导入了web模块正常运行所依赖的组件

Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter，相关的场景的所有依赖都会导入进来，要用什么功能就导入什么场景的启动器

## 2、主程序类

~~~java
@SpringBootApplication
public class SpringBootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDemoApplication.class, args);
    }
~~~

@SpringBootApplication：SpringBoot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用

~~~java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
~~~

@SpringBootConfiguration：SpringBoot的配置类

> 被标的类表示这是一个SpringBoot的配置类：
>
> @Configuration：
>
> 配置类——配置文件； 配置类也是容器中的一个组件；@Component

@EnableAutoConfiguration：开启自动配置功能

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

@AutoConfigurationPackage：自动配置包

>@Import(AutoConfigurationPackages.Registrar.class)
>
>Spring的底层注解@Import，给容器中导入一个组件；导入的组件由AutoConfigurationImportSelector.class
>
>将主配置类（@SpringBootApplication注解的类）的所在包及下面所有子包的所有组件扫描到Spring容器



>@Import(AutoConfigurationImportSelector.class)
>
>给容器中导入组件
>
>将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中
>
>会给容器中导入很多自动配置类（xxxAutoConfigration），就是给容器导入这个场景需要的所有组件，并配置好这些组件
>
>Spring Boot在启动的时候从类路径下的META-INF/spring.factorys中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工件

# 二、Spring Boot配置

## 1、配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

- application.properties
- applicatin.yml

配置文件的使用：修改SpringBoot自动配置的默认值；

SpringBoot在底层都给我们自动配置好

~~~yml
server:
  port: 8081
~~~

## 2、YAML语法

### 1、基本语法

K: v——表示一对键值对（空格必须有）；

以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

属性和值也是大小写敏感

### 2、值的写法

字面量：普通的值（数字，字符串，布尔）

>k: v——字面直接来写
>
>字符串默认不用加上单引号或者双绰号
>
>“”——双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
>
>''——单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
>
>

对象、Map：

>k: v——在下一行来写对象的属性和值的关系；注意缩进

~~~yaml
friends:
	lastName: zhangsan
	age: 20
~~~

行内写法

~~~yaml
friends: {lastName: zhangsan,age: 18}
~~~

数据（List,Set）

~~~yaml
pets:
	- cat
	- dog
	- pig
	
	
pets: [cat,dog,pig]
~~~

## 3、配置文件注入

### 1、properties配置文件在idea中默认utf-8可能会乱码

在file encoding中勾选Transparent native-to-ascii conversion即可

### 2、@Value获取值和@ConfigurationProperties获取值比较

|                | @ConfigurationProperties | @Value     |
| -------------- | ------------------------ | ---------- |
| 功能           | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定       | 支持                     | 不支持     |
| SpEL           | 不支持                   | 支持       |
| JSR303数据校验 | 支持                     | 不支持     |
| 复杂类型封闭   | 支持                     | 不支持     |

### 3、数据校验

~~~java
@Component
@ConfigurationProperties(prefix = "person")
@Getter
@Setter
@ToString
@Validated
public class Person {
    @Email
    private String name;
    private Integer age;
    private boolean isBoss;
    private Date birthDay;
    private Map<String, Object> map;
    private List<String> list;
    private Dog dog;
}
~~~

### 4、@PropertySource&@ImportResource

@PropertySource：加载指定配置文件

~~~java
@Component
@ConfigurationProperties(prefix = "person")
@PropertySource(value={"classpath:person.yml"})
@Getter
@Setter
@ToString
public class Person {
    private String name;
    private Integer age;
    private boolean isBoss;
    private Date birthDay;
    private Map<String, Object> map;
    private List<String> list;
    private Dog dog;
}
~~~

@ImportResource：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类上

~~~java
@ImportResource(locations = {"classpath:beans.xml"})
//导入Spring的配置文件
~~~

SpringBoot推荐给容器添加组件的方式；推荐全注释的方式

1. 配置类======Spring配置文件

2. 使用@Bean给容器中添加组件

   ~~~java
   /**
    * @Configuration：指明当前类是一个配置类，就是替代之前的Spring配置文件
    */
   @Configuration
   public class MyConfig {
       /**
        * 将方法的返回值添加到容器中，容器中
        * @return
        */
       @Bean
       public TestService testService(){
           System.out.println("配置类@Bean给容器中添加组件了...");
           return new TestService();
       }
   }
   ~~~

## 4、配置文件占位符

### 1、随机数

~~~java
${random.value}、${random.int}、${random.long}

${random.int(10)}、${random.int[1024,65535]}
~~~

### 2、占位符获取之前配置的值，如果没有可以用:指定默认值

~~~yml
person:
  name: wxh${random.uuid}${random.value}
  age: ${random.int[1,100]}
  birth-day: 2001/01/23
  map: {k1: v1,k2: v2}
  list: [l1,l2,l3]
  dog:
    name: ${person.hello:hello}_dog
    age: ${random.int(10)}
~~~

## 5、Profile

### 1、多Profile文件

我们在主配置文件编写的时候，文件名可以是application-{profile}.properties/yml

默认使用application.properties的配置；

### 2、yml支持多文档块方式

~~~yaml
server:
  port: 8080
spring:
  profiles:
    active: dev

---
server:
  port: 8083
spring:
  profiles: dev

---
server:
  port: 8084
spring:
  profiles: prod
~~~

### 3、激活指定profile

1. 在配置文件指定spring.profiles.active=prod

2. 命令行：--spring.profiles.active=dev

   ```
   java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev
   ```

3. 虚拟机参数：

   -Dspring.profiles.active=dev

## 6、配置文件加载位置

-file:./config/

-file:./

-classpath:/config/

-classpath:/

优先级由高到低，高优先级的配置会覆盖低优先级的配置；

SpringBoot会从这四个位置全部加载主配置文件；互补配置；

我们还可以通过spring.config.location来改变默认的配置文件位置

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置，指定配置文件和默认配置加载的这些配置文件共同起作用

## 7、外部配置加载顺序

SpringBoot也可以从以下位置加载配置，优先级从高到低；高优先级和低优先级配置共同起作用

1. 命令行参数
2. 来自java:/comp/env的JNDI属性
3. java系统属性（System.getProperties()）
4. 操作系统环境变量
5. RandomValuePropertySource配置的random.*属性值
6. jar包外部的application-{profile}.properties或application.yml（带spring.profile）配置文件
7. jar包内部的application-{profile}.properties或application.yml（带spring.profile）配置文件
8. jar包外部的application.properties或application.yml（不带spring.profile）配置文件
9. jar包内部的application.properties或application.yml（不带spring.profile）配置文件
10. @Configuration注解类上的@PropertySource
11. 通过SpringApplication.setDefaultProperties指定的默认属性

## 8、自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties)

自动配置原理：

1. SpringBoot启动的时候加载主配置类，开启了自动配置功能@EnableAutoConfiguration

2. @EnableAutoConfiguration作用：

   利用EnableAutoConfigurationImportSelector给容器中导入一些组件？

   可以查看selectImports()方法的内容；

   ```
   List<String> configurations = getCandidateConfigurations(annotationMetadata,
         attributes);
   ```

   获取候选的配置

   ```java
   SpringFactoriesLoader.loadFactoryNames()
   扫描所有jar包类路径下META-INF/spring.factories
   把扫描到的这些文件的内容包装成properties对象
   从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加到容器中
   ```

   将类路径下META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到了容器中；

   ```properties
   # Auto Configure
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
   org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
   org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
   org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
   org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
   org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
   org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
   org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
   org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
   org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
   org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
   org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
   org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
   org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
   org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
   org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
   org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
   org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
   org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
   org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
   org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
   org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
   org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
   org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
   org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
   org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
   org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
   org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
   org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
   org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
   org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
   org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
   org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
   org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
   org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
   org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
   org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
   org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
   org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
   org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
   org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
   org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
   org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
   
   ```

   每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置

3. 第一个自动配置类进行自动配置功能

4. 以HttpEncodingAutoConfiguration为例

   ~~~java
   @Configuration //这是一个配置类，也可以给容器中添加组件
   @EnableConfigurationProperties(HttpProperties.class)//启用类的ConfigurationProperties功能，将配置文件 中对应的值和HttpEncodingProperties绑定起来
   @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET) //Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效
   @ConditionalOnClass(CharacterEncodingFilter.class) //判断当前项目有没有这个类CharacterEncodingFilter，SpringMVC中进行乱码解决的过滤器
   @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled",
   		matchIfMissing = true)//判断配置文件中是否存在某个配置，spring.http.encoding.enabled如果不存在，判断也是成立的
   public class HttpEncodingAutoConfiguration {
     //只有一个有参构造器，参数的值会从容器中获取
     public HttpEncodingAutoConfiguration(HttpProperties properties) {
   		this.properties = properties.getEncoding();
   	}
     
     @Bean //给容器中添加一个组件，这个组件的某些值需要从properties中获取
   	@ConditionalOnMissingBean
   	public CharacterEncodingFilter characterEncodingFilter() {
   		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
   		filter.setEncoding(this.properties.getCharset().name());
   		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
   		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
   		return filter;
   	}
   ~~~

   根据当前不同的条件判断，决定这个配置类是否生效；

   一旦这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的properties类里获取的

   精髓：

   1）SpringBoot启动会加载大量的自动配置类

   2）我们看我们需要的功能有没有SpringBoot默认写好的自动配置类

   3）我们再来看这个自动配置类中到底配置了哪些组件，只要我们要用的组件有，我们就不需要再配置了

   4）给容器中自动配置类添加组件的时候，会从properties类中获取属性的值

5. 所有在配置文件 中能配置的属性都是在xxxProperties类中封装着

   ~~~java
   @ConfigurationProperties(prefix = "spring.http")//从配置文件中获取指定的值和bean的属性进行绑定
   public class HttpProperties {
   ~~~

   