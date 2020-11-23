# Collection 集合

## 1 集合概述

- 集合：集合是Java中的一种容器，可以用来存储多种数据。

集合与数组的区别：

- 数组长度固定，集合长度可变
- 数组中存的是同一类型的元素，可以存储基本数据类型的值。集合中存储的都是对象，对象类型可以不一致。

## 2 集合框架

集合按照其存储结构可以分为两大类：分别是单列集合`java.util.Collection`和双列集合`java.util.Map`

`Collection`：单列集合类的根接口，用于存出一系列符合某种规则的元素，它有两个重要的子接口，分别是`java.util.List`和`java.util.Set`。

- `List`的特点是元素有序、元素可重复、有索引，可以使用普通的for循环遍历。其主要实现类有`java.util.ArrayList`和`java.util.LinkedList`。
- `Set`的特点是元素无序，元素不可重复，没有索引，只能使用迭代器遍历。其实现类主要有`java.util.HashSet`和`java.util.TreeSet`。

## 3 Collocation常用方法

Collocation中定义了一些公共的方法：

- `public boolean add(E e)`：讲给定对象存到集合中
- `public void clear()`：清空集合
- `public boolean remove(E e)`：删除集合中的指定对象
- `public boolean contains(E e)`：判断当前集合是否包含给定的对象
- `public boolean isEmpty()`：判断当前集合是否为空
- `public int size()`：返回集合中元素个数
- `public object[] toArray`：把集合中的元素存到数组中

![java.util.Collection下的关系图](http://oos.sfzzz.xyz/2020_07_21_19_22_20_1.png)

![java.util.Map下的关系图](http://oos.sfzzz.xyz/2020_07_21_19_23_03_1.png)