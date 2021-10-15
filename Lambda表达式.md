## Lambda表达式

### 1、lambda表达式本质是一个匿名方法

 public int add(int x, int y) {
        return x + y;
    }

转成λ表达式后是这个样子：

    (int x, int y) -> x + y;

参数类型也可以省略，Java编译器会根据上下文推断出来：

    (x, y) -> x + y; //返回两数之和

或者

    (x, y) -> { return x + y; } //显式指明返回值





### 2、lambda表达式的类型

lambda表达式**可以被**当作是一个Object。**λ表达式的类型，叫做“目标类型（target type）”**。λ表达式的**目标类型是“函数式接口（functional interface）”，这是Java8新引入的概念**。它的定义是：**一个接口，如果只有一个显式声明的抽象方法，那么它就是一个函数式接口**。**一般用@FunctionalInterface标注出来（也可以不标）。**



* 可以用一个lambda表达式为一个函数式接口赋值：

   Runnable r1 = () -> {System.out.println("Hello Lambda!");};

  然后再赋值给一个Object

  Object obj = r1；

* 但是不能直接将lambda赋值给Object

  Object obj = ()->{System.out.println("hello lambda experssion");};   // ERROR! Object is not a functional interface!

  必须显示的转型成一个函数式接口才可以

  Object o = (Runnable) () -> { System.out.println("hi"); }; // correct

---

* 假设你自己写了一个函数式接口，长的跟Runnable一模一样：

      @FunctionalInterface
      public interface MyRunnable {
          public void run();
      }

  那么

      Runnable r1 =    () -> {System.out.println("Hello Lambda!");};
      MyRunnable2 r2 = () -> {System.out.println("Hello Lambda!");};

  都是正确的写法。这说明一个λ表达式可以有多个目标类型（函数式接口），只要**函数匹配成功即可**。
  但需注意一个λ表达式必须至少有一个目标类型。

  > 因为Runnable接口 只有一个run()方法，没有形参和返回值
  >
  > 而()->{System.out.println("hello Lambda!");};  该lambda表达式没有形参，也没有返回值，是能够和run方法进行匹配的。

  > 这里的函数匹配成功指的就是**创建的lambda和左侧的函数式接口中定义的函数是相同的形参和返回值，即函数定义是一致的**。





### 3、Lambda表达式作用

Java8有一个短期目标和一个长期目标。**短期目标是：配合“集合类批处理操作”的内部迭代和并行处理（下面将要讲到）**；长期目标是**将Java向函数式编程语言这个方向引导（并不是要完全变成一门函数式编程语言，只是让它有更多的函数式编程语言的特性）**，也正是由于这个原因，Oracle并没有简单地使用内部类去实现λ表达式，而是使用了一种更动态、更灵活、易于将来扩展和改变的策略（**invokedynamic**）。



#### 3.1 集合类批处理

**例子1**

Java8之前集合类的迭代（Iteration）都是外部的，即客户代码。而内部迭代意味着改由Java类库来进行迭代，而不是客户代码。例如：

    for(Object o: list) { // 外部迭代
        System.out.println(o);
    }

可以写成：

    list.forEach(o -> {System.out.println(o);}); //forEach函数实现内部迭代

**例子2**

![image-20210815161743368](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210815161743368.png)



**例子3**

```java
// 基类
class Base {
    void print(String... args) {
        System.out.println("Base......test");
    }
}

// 子类，覆写父类方法
class Sub extends Base {
    @Override
    void print(String[] args) {
        // 数组类型转换成为集合类型
        Arrays.asList(args).forEach(s->{
            System.out.println(s);
        });
        System.out.println("Sub......test");
    }
}
```



第一个能编译通过，这是为什么呢？事实上，base对象把子类对象sub做了向上转型，形参列表是由父类决定的，当然能通过。而看看子类直接调用的情况，这时编译器看到子类覆写了父类的print方法，因此肯定使用子类重新定义的print方法，尽管参数列表不匹配也不会跑到父类再去匹配下，因为找到了就不再找了，因此有了类型不匹配的错误。

这是个特例，**覆写的方法参数列表竟然可以与父类不相同，这违背了覆写的定义**，并且会引发莫名其妙的错误。

> 因为JVM对于存储的数据是按字符串数组去读取，但是存进去的时候（也就是编译时判断）是根据父类String变长参数