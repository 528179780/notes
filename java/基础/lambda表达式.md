# 概述

​		在之前的多线程Demo中，创建了很多的匿名内部类，因为创建新线程要实现`Runnable`接口或者new一个`Thread`类，重写其run方法。在以上的两种方式中，重要的只是 **run方法** 里面的代码块，也就是方法体，其他的都是因为面向对象编程所冗余的代码。因此可以使用lambda表达式简化代码。

Demo:

```java
package com.sufu.basic.demo.Lambda;

/**
 * @author sufu
 * @date 2020/7/22
 */
public class LambdaDemo1 {
    public static void main(String[] args) {
        //匿名内部类实现多线程
        new Thread(new Runnable(){
            @Override
            public void run() {
                System.out.println("匿名内部类创建并且启动了"+Thread.currentThread().getName()+"线程");
            }
        }).start();
        System.out.println("主线程"+Thread.currentThread().getName());
        //使用lambda表达式简化写法，本质同上一样，括号里是传进去的参数
        new Thread(() -> {
            System.out.println("匿名内部类创建并且启动了"+Thread.currentThread().getName()+"线程");
        }).start();
    }
}
```

# 匿名内部类的好处和弊端 

- 可以省去实现类的定义。
- 语法比较复杂。

# Lambda表达式的标准格式

标准格式有三个部分组成：

- 一些参数。
- 一个箭头。
- 一段代码。

**标准格式**为：

```java
(参数类型 参数名称) -> { 代码块}
```

格式说明：

- 小括号内的语法与传统方法参数一致：无参留空，多个参数用逗号隔开。
- `->`是引入的语法格式，代表指向，把参数传递给代码块。
- 大括号内的代码块与传统方法要求基本一致，重写接口的抽象方法的方法体。

# 示例

>无参数，无返回值的抽象方法。
>
>```java
>package com.sufu.basic.demo.Lambda;
>
>/**
> * 只有一个方法的父类
> * @author sufu
> * @date 2020/7/22
> */
>public interface Cook {
>    void makeFood();
>}
>//-----------------------------------------------------
>package com.sufu.basic.demo.Lambda;
>
>/**
> * @author sufu
> * @date 2020/7/22
> */
>public class LambdaDemo1 {
>    public static void main(String[] args) {
>        //调用Cook接口的无参，无返回值的方法，使用Lambda表达式
>        invokeCook(()-> System.out.println("make food"));
>    }
>    private static void invokeCook(Cook cook){
>        cook.makeFood();
>    }
>}
>```

>需求：
>
>​		使用数组存储多个Person对象，然后使用Arrays的sort方法通过年龄进行升序排序，即是带参数和返回值的lambda表达式。
>
>```java
>		Person[] people = {
>                new Person("张三", 18),
>                new Person("李四", 10),
>                new Person("王五", 70),
>                new Person("赵四", 43)
>        };
>        Arrays.sort(people, (Person o1, Person o2) -> {
>            return o1.getAge()-o2.getAge();
>        });
>```

# Lambda省略格式

Lambda强调的是"做什么"而不是"怎么做"，也因此凡是可以根据上下文推导得知的信息，都可以省略。

**省略规则**

- 小括号内参数的类型可以省略；
- 如果小括号内**有且仅有一个**参数，则小括号可以省略；
- 如果大括号内**有且仅有一条**语句，则无论是否有返回值，都可以省略大括号、return关键字以及句尾分号( ' ; ' )

比如上例可以改为：

```java
Arrays.sort(people, (o1, o2) -> o1.getAge()-o2.getAge());
```

# Lambda使用前提

- 使用Lambda表达式必须具有接口，且要求接口中**有且仅有一个抽象方法**。
- 使用Lambda表达式必须有**上下文推断**，也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用欧冠Lambda作为该接口的实例

>注意：有且仅有一个抽象方法的接口称为**函数式接口**

