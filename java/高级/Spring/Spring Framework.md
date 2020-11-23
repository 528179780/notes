# 1 Spring Framework简介

## 1.1 设计理念

- 在每个级别提供选择。Spring 允许您尽可能晚地推迟设计决策。例如，可以通过配置切换持久性提供程序，而无需更改代码。许多其他基础结构问题以及与第三方 API 的集成也是如此。
- 适应不同的观点。春天拥抱灵活性，对应该如何做事没有意见。它以不同视角支持广泛的应用需求。
- 保持强大的向后兼容性。春天的演变经过精心管理，迫使版本之间几乎没有重大更改。Spring 支持精心挑选的 JDK 版本和第三方库系列，以便于维护依赖于 Spring 的应用程序和库。
- 关心 API 设计。Spring 团队投入大量思考和时间，制作具有直观性且跨多个版本和多年的 API。
- 为代码质量设置高标准。Spring框架强调有意义、最新和准确的 javadoc。它是少数可以声明干净代码结构且包之间没有循环依赖关系的项目之一。

## 1.2 优点

- 开源，免费的框架（容器）！
- 轻量级的非入侵式的框架！
- 控制反转（IOC），面向切面编程（AOP）。
- 支持事务的处理，对框架整合的支持。

## 1.3 组成 

七大模块，三大思想。

![Spring 七大模块](http://oos.sfzzz.xyz/2020_07_28_09_51_56_1.jpg)

# 2 IOC

## 2.1 理论推导

```java
package com.sufu.service.impl;

import com.sufu.dao.UserDao;
import com.sufu.dao.impl.UserDaoImpl;
import com.sufu.service.UserService;

/**
 * @author sufu
 * @date 2020/7/28
 */
public class UserServiceImpl implements UserService {
    // 这种方式是程序主动new对象，控制权在程序上
    // private UserDao userDao = new UserDaoImpl();

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void getUser() {
        userDao.getUser();
    }
}
```

- 原来的方式是程序主动创建对象，控制权在程序员上。
- 用一个set注入之后，程序不再具有主动性，而是被动接收对象，使用者传入什么对象就是用什么对象。

这种思想，从本质上解决了问题，框架开发的程序员不用管理对象的创建了，系统耦合性大大降低，可以更加专注在业务的实现上了。对于一个框架来说，我们就是使用者，我们set哪个类就使用哪个类。这是IOC的原型。

**控制反转是通过描述（XML或者注解）并通过第三方去生产或获取特定对象的方式，在Spring中，实现控制反转的是IOC容器，其实现方法是依赖注入（Dependency Injection，DI）。**

## 2.2 Hello Spring

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDaoMySqlImpl" class="com.sufu.IOC.dao.impl.UserDaoMySqlImpl"/>
    <bean id="userDaoImpl" class="com.sufu.IOC.dao.impl.UserDaoImpl"/>
    <bean id="userServiceImpl" class="com.sufu.IOC.service.impl.UserServiceImpl">
        <!-- ref 引用已经定义好的bean -->
        <property name="userDao" ref="userDaoImpl"/>
    </bean>
</beans>
```

```java
import com.sufu.IOC.dao.impl.UserDaoMySqlImpl;
import com.sufu.IOC.service.impl.UserServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author sufu
 * @date 2020/7/28
 */
public class IOCTest {
    public static void main(String[] args) {
        // new一个 UserService的实现类，这里需要自己 new，也就是硬编码在文件里，实际调用的是业务层
//        UserServiceImpl u = new UserServiceImpl();
//        u.setUserDao(new UserDaoMySqlImpl());

        // 不用自己new对象了，这一步相当于相当于拿到容器。这样修改程序只需要修改xml就可以了。
        ApplicationContext a = new ClassPathXmlApplicationContext("beans.xml");
        UserServiceImpl u = (UserServiceImpl)a.getBean("userServiceImpl");
        u.getUser();
    }
}
```

写好配置文件之后就可以这样从IOC容器中获取对象了。如果要对`UserService`接口的实现类进行修改，则只需要改配置文件就可以了，对象由Spring来创建，管理，装配。

## 2.3 IOC创建对象的方式

1. Spring默认调用无参构造方法创建对象

   ```xml
   <bean id="user" class="com.sufu.ioc.entity.User">
           <property name="name" value="苏伏"/>
           <property name="age" value="18"/>
       </bean>
   ```

   

2. 全参构造方法创建对象

   1. 下标注入

      ```xml
      <bean id="user" class="com.sufu.ioc.entity.User">
              <constructor-arg index="0" value="苏伏"/>
              <constructor-arg index="1" value="18"/>
          </bean>
      ```

   2. 参数类型注入

      ```xml
      <bean id="user" class="com.sufu.ioc.entity.User">
          <constructor-arg type="java.lang.String" value="苏伏"/>
          <constructor-arg type="int" value="18"/>
      </bean>
      ```

   3. 参数名称注入（常用）

      ```xml
      <bean id="user" class="com.sufu.ioc.entity.User">
              <constructor-arg name="name" value="苏伏"/>
              <constructor-arg name="age" value="18"/>
      </bean>
      ```

### 2.3.1 IOC创建对象的时间

```xml
<bean id="userT" class="com.sufu.ioc.entity.UserT">
        <constructor-arg name="name" value="苏伏"/>
        <constructor-arg name="age" value="18"/>
    </bean>
```

再配置一个`userT`的bean，实际上在

```java
ApplicationContext a = new ClassPathXmlApplicationContext("beans.xml");
```

这一句载入配置文件的时候就创建了所有bean.xml中定义好的对象，存放到了IOC容器中。因为调用了相应的构造方法，只是等用户获取。如果获取两次对象，则获取到的其实是同一个对象，内存中只有一个实例，可以直接用 == 运算符检查。

# 3 Spring配置

## 3.1 别名（alias）

```xml
<alias name="user" alias="asasdasd"/>
```

即给定义好的Bean起一个别名，两个名字都可以获取。原名字长度太长的话，就可以起一个别名简化。

## 3.2 Bean配置

- `id`：Bean的唯一标识符
- `class`：Bean对象所对应的全限定名：包名+类名
- `name`：也是别名，可以写多个，可以通过空格，逗号，分号等等来分割。
- `scop`：作用域：默认单例模式

## 3.3 import

一般是团队开发使用，可以将多个配置文件导入合并为一个。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <import resource="beans.xml"/>
</beans>
```

开发的时候，项目只需要读取这一个applicationContext.xml文件就可以了，这个xml文件整合项目中所有的其他的xml。

# 4 DI（依赖注入）

## 4.1 构造器注入（上文已有）

## 4.2 Set方式注入【重点】

- 依赖注入：set方法注入，所以必须要有setter。
  - 依赖：Bean对象的创建依赖于容器。
  - 注入：Bean对象的所有属性，由容器来注入。

## 4.3 其他方式注入

## 4.4 Bean的作用域

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton) | (默认) 单例模式，将单个 bean 定义范围到每个 Spring IoC 容器的单个对象实例。 |
| [prototype](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-prototype) | 原型模式，每次从容器中get的都是一个新的对象。                |
| [request](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring. |
| [session](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring. |
| [application](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring. |
| [websocket](https://docs.spring.io/spring/docs/5.2.8.RELEASE/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring. |

# 5 Bean的自动装配

## 5.1 byName/byType装配

- 自动装配是Spring满足Bean以来的一种方式
- Spring会在上下文中自动寻找，并且自动给Bean装配属性

在Spring有三种装配方式

1. 在xml中显式配置
2. 在java中显式配置
3. 隐式的自动装配【重要】

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="cat" class="com.sufu.spring.auto.entity.Cat"/>
    <bean name="dog" class="com.sufu.spring.auto.entity.Dog"/>

<!--    <bean name="person" class="com.sufu.spring.auto.entity.Person">-->
<!--        <property name="name" value="苏伏"/>-->
<!--        <property name="dog" ref="dog"/>-->
<!--        <property name="cat" ref="cat"/>-->
<!--    </bean>-->
    <!--byName 自动在容器上下文中查找，该对象set后面对象同名的bean 自动装配-->
    <bean name="person" class="com.sufu.spring.auto.entity.Person" autowire="byName">
        <property name="name" value="苏伏"/>
    </bean>
    <!--byName 自动在容器上下文中查找，该对象相同的属性的bean自动装配-->
    <bean name="person" class="com.sufu.spring.auto.entity.Person" autowire="byType">
        <property name="name" value="苏伏"/>
    </bean>
</beans>
```

## 5.2 基于注解的自动装配

1. 导入约束 context约束

2. ######  配置注解支持`<context:annotation-config/> `

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           https://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           https://www.springframework.org/schema/context/spring-context.xsd">
   
       <!--开启注解支持-->
       <context:annotation-config/>
       <!--配置包扫描-->
       <context:component-scan base-package="com.sufu.spring"/>
   
   </beans>
   ```

`@autowired(required=false)`

默认是以上文中的byType方式实现的自动装配，可以不用set方法，因为它是用反射实现的。required=false表示这个对象可以为null，默认为true，对象不能为null；如果自动注入的不是想要的对象，或者有多个对象等待注入的话，可以和`@Qualifier("classname")`注解配合使用，指定要注入的类。

# 5 使用Java代码配置Spring

JavaConfig 是一个Spring的子项目，在Spring4之后，他成为了一个核心功能。

定义：

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

相当于使用 xml 如下配置

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

使用：获得上下文对象来获取bean

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

自动注入：

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

相当于使用 xml 如下配置

```xml
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```

然后配合`@Component`注解。再开启组件扫描注解`@ComponentScan(basePackages = "com.acme") `，配置好扫描的包之后，就可以注入了。

# 6 代理模式

## 6.1 为什么要学习代理模式？

因为这就是SpringAOP的底层。

## 6.2 代理模式的分类

- 静态代理
- 动态代理

## 6.3 静态代理

角色分析：

- 抽象角色：使用接口或者抽象类解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色之后会做一些附属操作
- 客户：访问代理对象的人

案例：

![代理模式图解](http://oos.sfzzz.xyz/2020_07_29_16_14_18_1.png)

示例：

抽象角色：

```java
package com.sufu.spring.proxy.demo1;

/**
 * 租房的接口
 * @author sufu
 * @date 2020/7/29
 */
public interface Rent {
    /**
     * 租房的方法
     * @author sufu
     * @date 2020/7/29 16:16
     **/
    void rent();
}
```

代理角色：

```java
package com.sufu.spring.proxy.demo1;

/**
 * 代理角色，代理房东
 * @author sufu
 * @date 2020/7/29
 */
public class Proxy implements Rent{
    private Host host;
    public Proxy(Host host) {
        this.host = host;
    }
    public Proxy() {
    }
    public Host getHost() {
        return host;
    }
    public void setHost(Host host) {
        this.host = host;
    }
    public void rent() {
        showHouse();
        host.rent();
        signContract();
    }
    public void showHouse(){
        System.out.println("中介带看房子");
    }
    public void signContract(){
        System.out.println("签租赁合同");
    }
}

```

为什么代理角色也要实现相应的接口？

个人理解：因为作为代理者，要能够尽量实现真实角色的所有方法，相当于自己是一个虚拟的角色，客户把虚拟角色直接当做真实角色来使用，而不管这个虚拟角色的真实角色到底是谁！真实角色实际上是实现了该接口的一个类，因此代理角色也要实现接口。

真实角色：

```java
package com.sufu.spring.proxy.demo1;

/**
 * 房东
 * @author sufu
 * @date 2020/7/29
 */
public class Host implements Rent{
    public void rent() {
        System.out.println("房东出租房子");
    }
}
```

客户：

```java
package com.sufu.spring.proxy.demo1;

/**
 * 客户要租房子
 * @author sufu
 * @date 2020/7/29
 */
public class Client {
    public static void main(String[] args) {
        // 直接找房东租房子,这种方式就只能租房子，不能干其他的
        Host h = new Host();
//        h.rent();

        // 代理的方式获取Host,这样执行业务逻辑，可以加一些自己想要的操作
        Proxy proxy = new Proxy(h);
        proxy.rent();
    }
}
```



代理模式好处：

- 可以使真实角色操作更加纯粹，不用关注一些公共的业务。
- 公共业务交给了代理角色，拆分了业务
- 公共业务扩展的时候不会改变真实角色，方便管理。

缺点：一个真实角色就会产生一个代理角色，代码量翻倍。

## 6.4 动态代理

- 动态代理和静态代理的角色一样
- 动态代理的代理类是动态生成的，不是直接写的。
- 动态代理也分为两大类：基于接口的动态代理、基于类的动态代理。
  - 基于接口 --JDK的代理
  - 基于类：cglib

需要了解两个类：Proxy，InvocationHandler

```java
package com.sufu.spring.proxy.demo2;

import com.sufu.spring.proxy.demo1.Rent;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * 用这个类自动生成一个 代理类
 * @author sufu
 * @date 2020/7/29
 */
public class ProxyInvocationHandler implements InvocationHandler {

    private Rent rent;

    public void setRent(Rent rent) {
        this.rent = rent;
    }

    /**
     * 得到代理对象
     * @author sufu
     * @date 2020/7/29 17:20
     * @return java.lang.Object
     **/
    public Object getProxy(){
        // 这里的 rent.getClass().getInterfaces() 的rent不是接口对象，而是一个接口的实现类的对象，这个方法是得到这个类实现的接口
        // 第二个参数穿进去的是要代理的类所实现的接口，也就是代理中的抽象角色，实际上也可以传一个 Class[] c = {Rent.class} 的 c 对象
        return Proxy.newProxyInstance(this.getClass().getClassLoader(), rent.getClass().getInterfaces(), this);
    }

    /**
     * 处理代理类实例，并且返回结果，在这里增加操作。
     * @author sufu
     * @date 2020/7/29 17:15
     * @param proxy 代理类
     * @param method 代理的方法
     * @param args 代理的方法的参数
     * @return java.lang.Object
     **/
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 其他操作
        System.out.println("执行了 "+method.getName()+" 方法");
        // 这里才是执行代理类中要执行的方法，也就是在Client中调用的代理类的方法，参数是指定的参数
        return method.invoke(rent, args);
    }
}
```

使用：

```java
package com.sufu.spring.proxy.demo2;

import com.sufu.spring.proxy.demo1.Host;
import com.sufu.spring.proxy.demo1.Rent;

/**
 * @author sufu
 * @date 2020/7/29
 */
public class Client {
    public static void main(String[] args) {
        // 真实角色，无论如何都是需要，只是横向扩展不需要修改而已
        Host host= new Host();
        // 创建代理角色生成器
        ProxyInvocationHandler proxyInvocationHandler = new ProxyInvocationHandler();
        // 指定要生成的代理类代理的接口
        proxyInvocationHandler.setRent(host);
        // 得到代理角色，这个代理类是动态生成的，这里的rent实际上是一个代理类
        Rent rentProxy = (Rent) proxyInvocationHandler.getProxy();
        // 通过执行类代理执行方法
        rentProxy.rent();

    }
}
```

# 7 AOP

## 7.1 什么是AOP

AOP(Aspect Oriented Programming)，意思为面向切面编程，通过预编译的方式和运行期间动态代理实现程序功能的统一维护技术。AOP是OOP的延续，是Spring框架中的重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑个部分之间的耦合性降低，提高程序可重用性，同时提高了开发效率。

## 7.2 AOP在Spring中的作用

提供声明式事务：允许用户自定义切面

- 横切关注点：跨越应用程序多个模块的方法或功能。即是，与我们业务逻辑无关的，但是我们需要关注的部分，就是横切关注点。比如日志、安全、缓存、事务等等。
- 切面（Aspect）：横切关注点模块化的特殊对象，是一个类。
- 通知（Advice）：切面必须要完成的工作，是切面类中的一个方法。
- 目标（Target）：被通知的对象，相当于前述代理模式中的真实对象。
- 代理（Proxy）：向目标对象应用通知后创建的对象。
- 切入点（Pointcut）：切面通知执行的"地点"的定义，在配置文件里可以是一个特殊的表达式，定义了要切入的地方
- 连接点（JoinPoint）：与切入点匹配的执行点。

![aop图示](http://oos.sfzzz.xyz/2020_07_30_10_21_35_1.png)

## 7.3 使用

### 方式一：使用api接口

这种方法需要实现接口

```java
package com.sufu.spring.aop.log;

import org.springframework.aop.MethodBeforeAdvice;

import java.lang.reflect.Method;

/**
 * 测试日志类
 * @author sufu
 * @date 2020/7/30
 */
public class Log implements MethodBeforeAdvice {
    /**
     * 前置增强，因为这个是执行前织入，所以不能获得返回值，返回值也还不能用。
     * @author sufu
     * @date 2020/7/30 9:20
     * @param method 要执行的方法
     * @param objects 要执行的方法参数
     * @param o 目标对象
     **/
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println(o.getClass().getName()+" 类执行了 "+method.getName()+"方法");
    }
}
```

写一个类实现MethodBeforeAdvice等等几个接口，实现要织入的方法，然后再写一个配置类：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <bean id="userService" class="com.sufu.spring.aop.service.impl.UserServiceImpl"/>
    <bean id="log" class="com.sufu.spring.aop.log.Log"/>

    <!--方式一：使用原生的Spring API接口-->
    <!--配置aop 需要导入aop约束-->
    <aop:config>
        <!--切入点  execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..)) 执行操作的位置-->
        <!--表达式：任意位置下的UserServiceImpl类的所有任意参数的方法-->
        <aop:pointcut id="pointcut" expression="execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..))"/>
        <!--执行一个环绕增加 也就是要执行的操作-->
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
    </aop:config>
</beans>
```

测试类：

```java
import com.sufu.spring.aop.service.UserService;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AopTest {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("beans.xml");
        // 动态代理代理的是接口，不能是一个实现类,相当于这里返回的是一个代理类，不是真正的UserService的实现类，经过了一层代理，因为被代理的类也实现了相应的接口
        UserService userService = classPathXmlApplicationContext.getBean("userService", UserService.class);
        userService.add();// 会先输出com.sufu.spring.aop.service.impl.UserServiceImpl 类执行了 add方法，再输出add方法里面写的：增加一个用户
        userService.delete();
    }
}

```

### 方式二：自定义类实现aop

这种方法可以自定义一个切面类，不用实现接口，但是不能获取执行方法的相关信息。

```java
package com.sufu.spring.aop.pointcut;

/**
 * 自定义切面类
 * @author sufu
 * @date 2020/7/30
 */
public class MyPointcut {
    public void before(){
        System.out.println("================方法执行前==================");
    }
    public void after(){
        System.out.println("================方法执行后==================");
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <bean id="userService" class="com.sufu.spring.aop.service.impl.UserServiceImpl"/>
    <bean id="log" class="com.sufu.spring.aop.log.Log"/>
    <!--方式二：自定义类 这种方法不能获取类相关信息-->
    <bean id="myPointcut" class="com.sufu.spring.aop.pointcut.MyPointcut"/>
    <aop:config>
        <!--自定义切面-->
        <aop:aspect ref="myPointcut">
            <!--切入点 就是要被代理的方法-->
            <aop:pointcut id="point" expression="execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..))"/>
            <!--配置通知 也就是在切入点方法执行之前还是之后 或者都执行的方法-->
            <aop:before method="before" pointcut-ref="point"/>
            <aop:after method="after" pointcut-ref="point"/>
        </aop:aspect>
    </aop:config>

</beans>
```

输出结果与方式一类似。

### 方式三：使用注解实现

```java
package com.sufu.spring.aop.pointcut;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

/**
 * 使用注解的方式实现AOP @Aspect注解标注这个类是一个切面
 * @author sufu
 * @date 2020/7/30
 */
@Aspect
public class MyPointcut2 {
    @Before("execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..))")
    public void before(){
        System.out.println("=======方法执行前==========");
    }
    @After("execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..))")
    public void after(){
        System.out.println("=======方法执行后==========");
    }
    @Around("execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..))")
    public void around(ProceedingJoinPoint p) throws Throwable {

        System.out.println("===环绕====方法执行前==========");
        Object proceed = p.proceed();
        System.out.println("===环绕====方法执行后==========");
        System.out.println(p.getSignature());
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">
    <bean id="userService" class="com.sufu.spring.aop.service.impl.UserServiceImpl"/>
    <bean id="log" class="com.sufu.spring.aop.log.Log"/>

    <!--方式三：使用注解实现-->
    <!--注册bean，或者@component-->
    <bean id="myAspect" class="com.sufu.spring.aop.pointcut.MyPointcut2"/>
    <!--开启注解支持-->
    <aop:aspectj-autoproxy/>

</beans>
```

注意：`execution(* com.sufu.spring.aop.service.impl.UserServiceImpl.*(..))`表达式子中，实际上定义的是方法，第一个 * 为返回值，*代表所有返回值的方法。com.sufu.spring.aop.service.impl.UserServiceImpl.\*表示UserServiceImpl类下面的所有方法，(..)代表所有方法的所有参数列表，也就是无论什么方法，无论什么返回值，无论什么参数列表，都是切入点。

# 8 整合MyBatis

步骤：

- 导入jar包
  - junit
  - MyBatis
  - mysql数据库
  - spring相关
  - aop织入
  - mybatis-spring
- 写配置文件
- 测试

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- DataSource-->
    <bean id="datasource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/db"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="datasource"/>
        <property name="mapperLocations" value="mapper/AccountMapper.xml"/>
    </bean>
    <!--因为没有set方法，所以只能通过构造方法注入-->
    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>

</beans>
```

然后写一个mapper类

```java
package com.sufu.spring.mybatis.dao;

import com.sufu.spring.mybatis.entity.Account;

import java.util.List;

/**
 * @author sufu
 * @date 2020/7/30
 */
public interface AccountMapper {
    /**
     * 查询所有账户
     * @author sufu
     * @date 2020/7/30 11:35
     * @return java.util.List<com.sufu.spring.mybatis.entity.Account>
     **/
    List<Account> selectAll();
}
```

# 9 Spring中的事务管理

- 编程式事务：需要在代码中修改
- 声明式事务：AOP

再议。。。