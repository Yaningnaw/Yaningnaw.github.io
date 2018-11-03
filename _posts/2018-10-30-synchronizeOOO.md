---
layout: post
title: '一个面试题引发的关于synchronized的体会'
subtitle: 'Java多线程'
date: 2018-10-30
categories: 技术
cover: 'https://wnliam-cn.gitee.io/yaningnaw.github.io/assets/img/tree.jpg'
tags: Java 多线程 设计
---
### 故事的开端是这样的：
我的一个朋友去百度面试，遇到了这样一个问题：  
Q：一个类里定义两个synchronized方法，起两个线程，同一个对象，a线程访问1方法，b线程访问2方法会怎么样？
```
看似没有什么难度的问题，却引发了的很多小伙伴的思考
```
#### 首先 ，让我们回忆一下synchronized的用法
- synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：
1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
- 指定对象，其作用的范围是synchronized后面括号括起来的部分，作用的对象是指定对象。
- 对代码块修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；   
- 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
（参考：https://blog.csdn.net/luoweifu/article/details/46613015)
##### 一、修饰代码块
- 最简单的就使用synchronized(this)关键字进行代码块的修饰就好啦！

举个栗子：
``` Java
public class Demo{
  public void method(){
    synchronized(this){
      //代码块内容
    }
  }
}
```
>此时此刻，一个线程访问一个对象中的synchronized(this)同步代码块时，其他试图访问该对象的线程将被阻塞。

- 指定对象的代码块分为多种
1. 就指定定特定对象  
举个栗子：
``` Java
public class Demo{
  ObjectA  a ;
  public Demo(ObjectA a){
    this.a = a;
  }
  public void method(){
    synchronized(a){
      //代码块内容
    }
  }
}
```
>此时此刻，当一个线程访问account对象时，其他试图访问account对象的线程将会阻塞，直到该线程访问account对象结束。

2. 当没有明确的对象,单纯的想让一段代码块同步
举个栗子：
``` Java
public class Demo{
  //private byte[] o = new byte[0]; //一种性能更好的写法
  /*说明：零长度的byte数组对象创建起来将比任何对象都经济――查看编译后的字节码：生成零长度的byte[]对象只需3条操作码，而Object lock = new Object()则需要7行操作码。*/
  Object  o = new Object() ;
  public void method(){
    synchronized(o){
      //代码块内容
    }
  }
}
```
- 指定类的代码块(.class)
举个栗子：
``` Java
public class Demo{
  public void method(){
    synchronized(Demo.class){
      //代码块内容
    }
  }
}
```
>synchronized作用于一个类T时，是给这个类T加锁，T的所有对象用的是同一把锁。

##### 二、修饰方法
- synchronize修饰非静态方法时，直接在方法上添加关键字就可以啦。  

举个栗子：
``` Java
public class Demo{
  public synchronized void method(){
      //todo()
  }
}
```
>此时此刻，锁的作用范围是该方法，作用对象是调用该方法的对象（后面会细讲），有一个线程正在访问该方法时，其他试图访问该对象该方法的线程会被阻塞。

- synchronize修饰静态方法时
举个栗子：

``` Java
public class Demo{
  public static synchronized void method(){
      //tudo()
  }
}
```
>因为静态方法属于类，所以在静态方法上加的同步锁也属于类和类的任何对象，范围依旧是该方法

#### 回到那个问题，我们来模拟一个实现
- 如果不加锁
``` Java
public class test {
  public static void main(String[] args) {
    People p = new People();
    Thread a = new MyThread(p, 1);
    Thread b = new MyThread(p, 2);
    a.start();
    b.start();
  }
}

  class People{

      public void a() {
          for(int i=0;i<50;i++) {
            System.out.print("A");
          }
        }
      public void b() {
        for(int i=0;i<50;i++) {
          System.out.print("B");
        }
      }
    }
  class MyThread extends Thread{
    People p;
    int c;
    MyThread(People p,int c){
      this.p = p;
      this.c = c;
    }
    @Override
    public void run() {
      if(c == 1)
        p.a();
        else if(c == 2)
        p.b();
      }
  }
}
```
>在多次运行后会出现类似AAABBBBBAAA或BBBAAAANBB的情况，说明了不加同步锁时，两个线程是可以同时运行的

- 加上方法锁：
``` Java
public class test {
  public static void main(String[] args) {
    People p = new People();
    Thread a = new MyThread(p, 1);
    Thread b = new MyThread(p, 2);
    a.start();
    b.start();
  }
}

  class People{

      public synchronized  void a() {
          for(int i=0;i<50;i++) {
            System.out.print("A");
          }
        }
      public synchronized  void  b() {
        for(int i=0;i<50;i++) {
          System.out.print("B");
        }
      }
    }
  class MyThread extends Thread{
    People p;
    int c;
    MyThread(People p,int c){
      this.p = p;
      this.c = c;
    }
    @Override
    public void run() {
      if(c == 1)
        p.a();
        else if(c == 2)
        p.b();
      }
  }
}
```
>而在加了同步锁之后，发现两个线程不再同时运行，必须等另一个线程下的方法运行结束后才会继续运行

###### 那么问题来了：我们l两个线程调用的明明不是同一个方法，但是为什么会产生同步的效果呢？

>A：还记得吗，代码块定义时，我们对代码块进行了对象的设置，而对方法的锁却没有进行绑定。
- 其实非静态方法的默认修饰对象是(this)。  

不难验证：
``` Java
public synchronized void method(){
    //todu
}
```
和
``` Java
public void method(){
  synchronized(this){
      //tudo
  }
}
```
是等价的，所以，即使在我们使用不同线程调用同一个对象的不同同步方法（或代码块）时，如果一个线程已经调用其中一个，另一个线程也会被阻塞，原因是已经调用的同步方法（或代码块）占用了该对象中的锁！！！！（哇好神奇）

###### 可是又有人问了，synchronized不是支持重入性吗？
- 下面介绍一下synchronized的重入synchronize锁重入：
>1. 关键字synchronize拥有锁重入的功能，也就是在使用synchronize时，当一个线程的得到了一个对象的锁后，再次请求此对象是可以再次得到该对象的锁。  

>2. 当一个线程请求一个由其他线程持有的锁时，发出请求的线程就会被阻塞，然而，由于内置锁是可重入的，因此如果某个线程试图获得一个已经由她自己持有的锁，那么这个请求就会成功，“重入” 意味着获取锁的 操作的粒度是“线程”，而不是调用  

（参考：https://blog.csdn.net/qq_32120645/article/details/72900976 ）


#### 最后写一些synchronized的使用注意事项和与lock的区别
1. 注意事项
>- synchronized关键字不能继承。
虽然可以使用synchronized来定义方法，但synchronized并不属于方法定义的一部分，因此，synchronized关键字不能被继承。如果在父类中的某个方法使用了synchronized关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上synchronized关键字才可以。当然，还可以在子类方法中调用父类中相应的方法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此，子类的方法也就相当于同步了。  

>- 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。   

>- 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。   

>- 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。



2. 与lock的区别：
- synchronized：
```
1. 存在层次：Java的关键字，在jvm层面上
2. 锁的释放：
- 以获取锁的线程执行完同步代码，释放锁   
- 线程执行发生异常，jvm会让线程释放锁
3. 锁的获取:
假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待
4. 锁状态: 无法判断
5. 锁类型: 可重入 不可中断 非公平
6. 性能: 少量同步
```
- Lock：
```
1. 存在层次：是一个类
2. 锁的释放：在finally中必须释放锁，不然容易造成线程死锁
3. 锁的获取:分情况而定，Lock有多个锁获取的方式，具体下面会说道，
大致就是可以尝试获得锁，线程可以不用一直等待
4. 锁状态: 可以判断
5. 锁类型: 可重入 可中断  可公平（两者皆可）
6. 性能: 大量同步
```
（参考：https://blog.csdn.net/u012403290/article/details/64910926 ）  



#### 该文章主要用于个人学习和记录，借鉴了很多相关博客，若 有不足和错误处请指正。
