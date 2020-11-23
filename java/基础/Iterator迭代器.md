# Iterator迭代器

# 1 Iterator接口

jdk提供了一个遍历集合中元素的接口`java.util.Iterator`。主要用于迭代访问集合中的元素，因此`Iterator`对象也被称为迭代器。获取方式：

- `public Iterator iterator()`：获取集合对应的迭代器，用来遍历集合。

迭代：

- 即Collocation几何元素的通用获取方式，取出元素之前先判断集合中有没有该元素，如果有，就取出这个元素，然后继续判断，还有就再取出，一直把集合中的所有元素都取出，这种取出的方式叫做迭代。

Iterator接口的常用方法如下：

- `public E next()`：返回迭代的下一个元素。
- `public boolean hasNext()`：如果仍有元素可以迭代，返回true

使用：

```java
package com.sufu.basic.demo;

import java.util.Iterator;
import java.util.LinkedList;

public class IteratorDemo {
    public static void main(String[] args) {
        LinkedList<String> list = new LinkedList<>();
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");
        Iterator<String> iterator = list.iterator();
        //三种迭代器的使用方法
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }
        for(Iterator<String> iterator1 = list.iterator();iterator1.hasNext();){
            System.out.println(iterator1.next());
        }
        for (String s : list){
            System.out.println(s);
        }
    }
}
```

 