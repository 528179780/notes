# SpringBoot启动类

## @SpringBootApplication注解

这个注解其实是一个组合注解，在其源码中就可以看见。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {}
```

### @SpringBootConfiguration注解

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {}
```

该类是由SpringBoot提供的，可以发现它可以注解在Class, interface (including annotation type), or enum declaration上，也就是个类、接口、注解或者枚举类型。

@Configuration注解是由spring提供的，表示这是一个配置类，相当于配置文件。

### @EnableAutoConfiguration

开启自动配置功能，省略大量的配置，约定大于配置，没有配置的东西都遵从行业约定。此注解是为了启用自动配置功能。

```java
...
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

#### @AutoConfigurationPackage 自动配置包

```java
...
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {}
```

@Import是Spring提供的底层注解，给容器中导入一个组件。

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

   @Override
   public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
      register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
   }

   @Override
   public Set<Object> determineImports(AnnotationMetadata metadata) {
      return Collections.singleton(new PackageImports(metadata));
   }

}
```

registerBeanDefinitions这个方法在SpringBoot启动的时候就会执行用，new PackageImports(metadata).getPackageNames()会获取到@SpringBootApplication注解的类的包名，然后将该包中的所有组件注册到Spring容器中，这也就是为什么@Appliacation注解的方法，也就是SpringBoot的主方法必须放在最高级的包下面的原因。

#### @Import(AutoConfigurationImportSelector.class)

给容器中注入一些组件，这里传的就是一个选择器，会选择哪些类自动注入到Spring容器中：

getCandidateConfigurations(metadata,attributes)方法获取所有等待注入的自动配置类，其方法体如下：

```java
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
      getBeanClassLoader());
```

getSpringFactoriesLoaderFactoryClass() 方法获取的就是EnableAutoConfiguration.class，第二个参数就是一个类加载器，用来加载类名。

loadFactoryNames() 作用就是从META-INF/spring.factories中获取EnableAutoConfiguration指定的值，这个文件的地址在导入的类的jar包中，也就是在spring-boot-autoconfigure这个jar包中的META-INF/spring.factories文件所定义的EnableAutoConfiguration指定的值，从文件中看到刚好是127行，也就是过滤之前的所有需要自动加载的类。

![image-20201112221255793](C:\Users\sufu\Desktop\笔记\Java\微服务\Typora_Img\image-20201112221255793.png)

如上图127个配置类，经过一个getConfigurationClassFilter().filter(configurations);过滤之后，剩下23个类需要自动注入，就不用写这些配置类。

# SpringBoot配置

## yaml语法

### 基本语法

键值对表示属性与值：键;(空格)值 这个空格不能省略。

空格缩进表示层级关系，不能使用Tab控制。

### 值的写法

#### 字面量：（数字，字符串，布尔）

k: v 字面量直接写，字符串不用引号

"" (双引号)：不会转义字符串里面的特殊字符；name:"张三\n李四"  输出：张三（换行）李四

''(单引号)：会转义特殊字符，最终输出的只是一个普通的字符串数据；name:"张三\n李四"  输出：张三\n李四

#### 对象、map（键值对）：

k:v 对象还是k:v的方式，直接在下一行写属性和值的关系

```yaml
friends:
	name: zahngsan
	age: 20
```

行内写法：

```yaml
friends: {name: zhangsan,age: 20}
```

#### 数组（List，Set）：

用- 表示数组中的一个元素

```yaml
pets:
  - cat
  - dog
  - pig
```

数组的行内写法

```yaml
pets: [cat,dog,pig]
```

#### 例子

新建一个application-test.yaml文件

```yaml
person:
  name: 张三
  age: 15
  isMan: True
  birth: 1999/08/28
  pets:
    - dog
    - cat
    - pig
```

application.yaml文件设置

```yaml
spring:
  profiles:
    active: test
```

bean

````java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    String name;
    Integer age;
    boolean isMan;
    Date birth;
    List<String> pets;
    getter...
    setter...
}
````

需要注入的bean必须由Spring管理，这里@Component就是将这个JavaBean注册为一个组件，然后通过@Autowired注入。还需要配置注释解析器，添加如下依赖即可：

```xml
<!-- 配置注释解析器，用于从yaml配置文件中读取数据 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

打包的时候排除该解析器：注意下文在\<build>/\<plugins>/\<plugin>节点下面，与groupId，artifactId同级。

```xml
<!--打包的时候排除注释解析器-->
<configuration>
    <excludes>
        <exclude>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </exclude>
    </excludes>
</configuration>
```

## 属性注入

### @value注入属性

可以注入三种：

- 字面量：直接@value("zhangsan")
- 环境变量，配置文件中获取值：@value(${person.name})
- El表达式：@value(#{11*3})

|                      | @ConfigurationProperties   | @value                 |
| -------------------- | -------------------------- | ---------------------- |
| 功能                 | 批量注入配置文件里面的属性 | 注入一个配置文件中的值 |
| 松散绑定（松散语法） | 支持                       | 不支持                 |
| SpringEL表达式       | 不支持                     | 支持                   |
| JSR303数据校验       | 支持                       | 不支持                 |
| 复杂类型封装         | 支持                       | 不支持                 |

松散绑定：配置文件中的值与属性名不需要完全一样，比如lastName和last-name可以对应绑定上。

JSR303数据校验：@Validated校验如下：

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    String name;
    Integer age;
    boolean isMan;
    @Email
    String email;
    Date birth;
    List<String> pets;
    getter...
    setter...
}
```

@Email配合@Validated可以校验注入的值是不是一个合法的邮箱账号

### @PropertySource和@ImportResource

@PropertySource：加载指定的配置文件，@ConfigurationProperties默认从全局配置文件中获取值，配合此注解可以从指定配置文件读取：

```java
@Component
@PropertySource("classpath:person.prorerties")
@ConfigurationProperties(prefix = "person")
public class Person {
    String name;
    Integer age;
    boolean isMan;
    Date birth;
    List<String> pets;
    getter...
    setter...
}
```

```properties
person.name=zhangsan
person.age=15
person.isMan=True
person.birth=1999/08/28
person.pets=dog,cat,pig
```

但是：这种方式不能读取yaml格式的配置文件，如果要读取，参照[这里](https://www.cnblogs.com/mzq123/p/11936075.html)

@ImportResource可以加载Spring以及SpringMVC的配置文件（.xml文件）

## SpringBoot推荐的配置文件方式：配置类

@bean注解：将方法的返回值添加到容器中，默认的对象id就是方法名

## 配置文件占位符

```properties
app.name=test name
message=app name is ${app.name:默认值} !
random=${random.int}
```

1、随机数

2、占位符获取之前配置的值，如果没有， : 指定默认值

## 多配置文件配置

properties配置文件合yaml配置文件都可以使用

```yaml
spring:
  profiles:
    active: dev
```

这种方式来选择要启用的配置文件，dev是application-dev.yaml/application-dev.properties的简写，只有以application开头的文件名可以这么写。

如果是yaml配置文件，还可以写在一个配置文件里，用 --- 隔开，表示一个文档块然后在文档块里面指定

```yaml
spring:
  profiles: dev
```

然后在第一个文档块指定

```yaml
spring:
  profiles:
    active: dev
```

也可以达到一样的效果。完整如下：

```yaml
server:
  port: 8080
spring:
  profiles:
    active: dev
---
spring:
  profiles: dev
server:
  port: 8081
---
spring:
  profiles: prod
server:
  port: 80
```

命令行模式指定配置文件：

--spring.profile.active=dev

jvm参数 -Dspring.profile.active=dev

# SpringBoot自动配置原理

1. SpringBoot的主配置类开启了自动配置功能，即@EnableAutoConfiguration注解。

2. @EnableAutoConfiguration作用：

   - 利用AutoConfigurationImportSelector类来给容器中导入一些组件。

   - selectImports(AnnotationMetadata annotationMetadata)方法获取需要自动导入的组件

   - SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),getBeanClassLoader())获取需要导入的bean的names，方法里面的loadSpringFactories()方法扫描类路径下的META-INF/spring.factories文件，然后将他们包装秤properties对象，然后获取names

     ```properties
     # Auto Configure
     org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
     org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
     org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
     org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
     org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
     org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
     org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
     org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
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
     org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
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
     org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
     org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
     org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
     org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
     org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
     org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
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
     org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
     org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
     org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
     org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
     org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
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
     org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
     org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
     org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
     org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
     org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
     org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
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

     每一个这样的xxxAutoConfiguration类都是一个组件，都会陪加载到容器中，来完成自动配置。例如：

     以HttpEncodingAutoConfiguration为例子：

     ```java
     @Configuration(proxyBeanMethods = false)
     @EnableConfigurationProperties(ServerProperties.class)
     @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
     @ConditionalOnClass(CharacterEncodingFilter.class) // 判断当前项目有无CharacterEncodingFilter类，也就是SpringMVC中乱码解决的拦截器，有的话就使用该类的配置
     @ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)// 判断配置文件中是否存在server.servlet.encoding.enabled，如果不存在则为true，也就是如果配置文件中不配置也是默认生效的。
     ```

     ```java
     public HttpEncodingAutoConfiguration(ServerProperties properties) {
        this.properties = properties.getServlet().getEncoding();
     }// 构造方法，从ServerProperties类中getEncoding()
     ```

      配置文件中能配置的属性都是在xxxrProperties中封装的。自动配置类就是根据不同条件，决定什么配置生效

     后面三个@ConditionalOnxxx都是根据条件配置，如果满足的话就注入注解后面的属性。

