---
layout: post
title: 'Java基础面试之：重写equals()为什么重写hashcode()'
subtitle: 'Spring'
date: 2019-02-25
tags: Java基础 面试
---
# 前言
我们往往在编写业务程序时需要对一些类进行equals()方法的重写，其中常用的一种情况就是用来保证可以对这个类属性相同的对象作比较,还有String类中也会对equals进行重写。在重写equals时，我们使用IDE发现往往需要对hashcode方法也进行重写。这是为什么呢？

## 一、Object中的hashcode()和equals()

```Java 
//Object中的hashcode方法
public native int hashcode();

//equals方法
public boolean equals(Object obj) {
        return (this == obj);
    }
```
### hashcode()
参考Object源码中的描述，Object中的hashcode为实例化对象时根据内存地址运算出的值，这个值对于相同的对象/实例是相同的。而equals方法是直接对传入对象和调用equals对象进行对象是否相同的判断。如下：
```
* <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
```

那么为什么要对对象进行hashcode的运算呢？目前我的理解为VM每次new一个Object， 都会将Object丢到一个哈希表中去， 这样的话，下次做Object的比较或者取这个对象的时候， 它会根据对象的hashcode再从Hash表中取这个对象，这样做的目的是提高取对象的效率。

**我理解为在原生Object类中，调用equals时判定对象是否是同一个实例的过程中，会对hashcode方法进行调用（换句话说就是比较hashcode是否相同就可以实现）**。

而关于equals方法，hashcode方法的注释中也提到了
```
* <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
```

其中提到了同一个对象hashcode返回int值必须一样，对象不相等时，不要求调用equals时每个对象调用hashcode产生不同整数结果，但是程序员要明白不同对象产生不同hash接结果可以提高hash表性能。

### equals()

```
/**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     * <p>
     * As much as is reasonably practical, the hashCode method defined by
     * class {@code Object} does return distinct integers for distinct
     * objects. (This is typically implemented by converting the internal
     * address of the object into an integer, but this implementation
     * technique is not required by the
     * Java&trade; programming language.)
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
```

我们可以看到equals主要是对对象进行了比对，==也就是判断是否为同一个对象。
在equals注释中，提到了equals方法实现等价关系、和对非空值与非空引用的对比，也提到了重写equals时对hashcode的重写以便维护，同时写到了equals返回true时一般表示比较的对象相同。

**在最后注释中提到参考hashcode和HashMap的内容。**

## 二、HashMap
在equals中的注释提到查看HashMap的部分，我们找到HashMap中和equals、hashcode相关的内容：
```Java
//HashMap中的equals重写
public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
//HashMap中的hash算法
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

### equals
HashMap中的equals可以看到是调用了Object中的equals进行封装的，换而言之就是也可能需要调用Object中的hashcode方法进行对象对比，从这点上看重写hashcode就可能影响到HashMap的使用，但这里不是绝对的。

### hash
HashMap中还有一个hash方法，逻辑上是对传入的对象的hashCode进行高16位和低16位的异或运算。这里用到了Object的hashcode方法，应该是对不同对象的hashcode进行向hashmap中散列存储。试想如果我们重写equals时不对hashcode进行重写，**equals判断为同一个对象时hashcode却不相同，在使用HashMap时，就会对其hash方法造成影响。**

## 总结
再回到equals重写这个问题，如果一个对象需要重写equals，那么该对象一般是有自己的属性参与到equals对比中，而对象产生时，hashcode是随机的，此时若此时只是重写了equals而没有重写hahscode，那么equals比较的两个对象即使相等，hashcode是不相等的。

*在《Effective Java》第45页，提到了Object.hashCode的通用约定：（即hashcode注释的较官方的翻译）*
```
1. 在一个应用程序执行期间，如果一个对象的equals方法做比较所用到的信息没有被修改的话，那么，对该对象调用hashCode方法多次，它必须始终如一地返回 同一个整数。在同一个应用程序的多次执行过程中，这个整数可以不同，即这个应用程序这次执行返回的整数与下一次执行返回的整数可以不一致。
2. 如果两个对象根据equals(Object)方法是相等的，那么调用这两个对象中任一个对象的hashCode方法必须产生同样的整数结果。
3. 如果两个对象根据equals(Object)方法是不相等的，那么调用这两个对象中任一个对象的hashCode方法，不要求必须产生不同的整数结果。然而，程序员应该意识到这样的事实，对于不相等的对象产生截然不同的整数结果，有可能提高散列表（hash table）的性能。
```

显然，首先如果不重写hashcode下重写equals的话，会违反第二条，因为**每次传入同内容对象，hashcode的值可能不同（即可能不是同一个对象只是相等对象）**

其次，**对于HashSet和HashMap这些基于散列值（hash）实现的类，可能导致HashSet、HashMap不能正常的运作。**


### 注意：
如果equals重写时有变量参与，hashcode也必须有对应变量参与进去，才可以实现equals相等即hashcode相等，因为变量的值有多种可能。