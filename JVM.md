# JVM

## 1. JVM位置和整体结构



### 1.1 Jvm位置

### 是在操作系统之上，不与硬件直接交互

![image-20210606204324420](H:\Notebook\编程笔记\typora-user-images\image-20210606204324420.png)

图中可以看出JDK包括jre、jvm，jre是java运行时需要的环境，即一些api，jvm虚拟机

---



### 1.2 jvm整体结构

![image-20210606204643941](H:\Notebook\编程笔记\typora-user-images\image-20210606204643941.png)

多线程共享 ：方法区、堆

每个线程各自独有：java栈、本地方法栈、程序计数器 

> 很好理解：想一想线程运行需要什么，要维护自己的运行状态，那么就要有自己的程序计数器，方法调用到哪一个，
>
> 而由于所有线程共享进程的数据，所以方法区和堆共享

---



### 1.3 Java代码执行流程

![image-20210606205537951](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210606205537951.png)

---



### 1.4 JVM的架构模型

* 基于栈的指令集架构
  * 指令集小  0地址
* 基于寄存器的指令集架构
  * 完成一个任务需要的指令更少

---

由于跨平台性的设计，**Java的指令都是根据栈**来设计的。

- **优点**：跨平台性，指令集小，指令多；  
- **缺点**：执行性能比基于寄存器的差

---



##  2. JVM的生命周期

**JVM启动**通过引导类加载器创建一个初始类来完成，这个类是由虚拟机的具体实现指定的

父类早于子类加载（因为存在方法继承）







![image-20210608213905431](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210608213905431.png)

java对象的左值是引用，那么数组也是引用，可以进行重新指向



## 3. 虚拟机

### 3.1 Hotspot

占有绝对的市场地位  HotSpot指的是**热点代码探测**技术

* 通过计数器找到最具编译价值代码，触发及时编译或栈上替换
* 通过编译器与解释器协同工作，在最优化的程序响应时间与最佳执行性能中取得平衡

---



### 3.2 JRockit

专注于服务器端应用

* 不太关注程序启动速度，因此JRockit内部不包含解析器实现
* 是世界上最快的JVM

---

### 3.3 J9



---





## 二、类加载子系统

### 2.1 类加载器

加载阶段       链接阶段       初始化阶段

![image-20210615145155605](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210615145155605.png)





![image-20210615150401626](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210615150401626.png)



getClass()方法获取模板，然后调用构造方法实例化一个对象

ClassLoader 是将Car.class文件**以二进制流的方式加载进内存**，放在方法区中（个人理解：因为class文件除了成员属性外，都是方法属性）



---

### 2.2 具体阶段

#### 2.2.1 阶段一 loading 阶段

1. 通过一个类的**全限定名**获取定义此类的二进制字节流
2. 将这个**字节流所代表的静态存储结构转化为*方法区* 的运行时数据结构**
3. **在内存中生成一个代表这个类的java.lang.Class对象**，作为方法区这个类的各种数据的访问入口。

---

**补充**：加载.class文件的方式

* 从本地系统中直接加载    也是我们平时使用的
* 通过网络获取，典型场景：Web Applet
* 从zip压缩包中读取，成为日后jar、war格式的基础     平时开发时使用的一些第三方库
* 运行时计算生成，使用最多的是：动态代理技术
* 由其他文件生成，典型场景：JSP应用
* 从专有数据库中提取.class文件，比较少见
* 从加密文件中获取，典型的防Class文件被反编译的保护措施

---



#### 2.2.2 链接阶段 linking

![image-20210615154542913](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210615154542913.png)

1. 验证
   * 目的是确保Class文件的字节流中包含信息符合当前虚拟机要求
   * 主要包括四种验证

2. 准备
   * 比如**给类变量一个默认值**，是准备阶段做的事
   * 实例变量不会分配初始化，因为实例变量不是放在方法区的，那什么时候定义和初始化？
3. 解析
   * 符号引用：是**描述所引用目标**
   * 直接引用：是**一个指针、相对偏移量 是用于定位目标对象的句柄**

---

> java 查看字节码 [(21条消息) Java以及IDEA下查看字节码的五种方法_无界编程-CSDN博客_idea查看字节码](https://blog.csdn.net/21aspnet/article/details/88351875)

#### 2.2.3 初始化

将显示初始化和静态代码块中的语句合并

![image-20210615160008346](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210615160008346.png)

![image-20210615161106159](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210615161106159.png)

静态代码块中放的也是静态变量/类变量

输出number结果是10 ，因为是先通过**链接阶段**，而**链接中有个准备阶段**，此时**就进行了变量的定义**

> 如果没有类变量或者是静态代码块就不会存在clinit方法

最后一点：如果不同步加锁，那么会造成变量初始化异常，因为**类变量是多个对象共有**的。

---



**例子**

1. 有类变量 ，那就包含<.clinit>方法

![image-20210624124931773](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210624124931773.png)

2. 如果没有类变量，就不包含<.clinit>方法 ，只有类的构造方法（类构造器）

![image-20210624125238684](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210624125238684.png)

---





### 2.3 类加载器

![image-20210616174300664](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210616174300664.png)



---



**虚拟器自带的加载器**

1. **启动类加载器（引导类加载器  Bootstrap  ClassLoader）**

* 使用c/c++实现，嵌套在JVM内部
* 用来加载Java的核心库（JAVA_HOME/jre/lib/rt.jar、resources.jar

![image-20210616180847919](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210616180847919.png)

---

2. **扩展类加载器**

![image-20210616182707092](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210616182707092.png)

```java
System.out.println("*************扩展类加载器**********");
String extDirs = System.getProperty("java.ext.dirs");
for(String path: extDirs.split(";")){
    System.out.println(path);
}
// 从上面的路径中随意选择一个类，来看看他的类加载器是什么 比如这里是CurveDB类
ClassLoader classLoader1 = CurveDB.class.getClassLoader();
System.out.println(classLoader1);
```

![image-20210616184329315](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210616184329315.png)





---

3. **系统类加载器**

![image-20210616182718274](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210616182718274.png)

---



4. **用户自定义类加载器**

为什么要使用自定义类加载器

* 隔离加载类
* 修改类加载方式
* 扩展加载源
* 防止源码泄露

> 也是要实现把class文件以字符流形式加载进内存中。



### 2.4 获取ClassLoader

![image-20210617142853686](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210617142853686.png)

---



### 2.5 双亲委派机制

1. **定义**

Java虚拟机对class文件采用的是**按需加载**的方式，也就是说***当需要使用该类时才会将它的class文件加载到内存生成class对象***。而且加载某个类的class文件时，Java虚拟机采用的是**双亲委派模式**，即把请求交由父类处理，它是一种任务委派模式。

---

2. **工作原理**

1）如果一个类加载器收到类加载请求，它**并不会自己先去加**载，而是把这个请求委托给父类的加载器去执行。

2）如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归请求最终到达顶层的启动加载器

3）如果**父类加载器可以完成类加载任务，就成功返回**，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。

> 可以防止恶意破坏，比如自己定义了一个java.lang.String类，其中注入了恶意代码



向上委托时，对应的加载器会看这个类是否在自己的加载路径里面

---

3. **双亲委派机制优势**



![image-20210617151702287](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210617151702287.png)

> 第三方的架包是通过引导类加载器加载的，而具体的实现类是通过线程上下文类加载器加载的（默认就是系统类加载器）



* 避免类的重复加载
* 保护程序安全，防止核心API被随意篡改
  * 自定义类：java.lang.String 是不会被加载的 

---



4. **沙箱安全机制**

自定义String类，但是在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件（rt.jar包中 java\lang\String.class)。这样可以保证对java核心源代码的保护，这就是沙箱保护机制。

---



## 三、运行时数据区

![image-20210624131102140](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210624131102140.png)

一个JVM对应一个进程，方法区和堆是一个进程中的多个线程共有的，而每个线程都有其独自的程序计数器（标志该线程运行到哪了）、本地方法栈（方法之间的调用问题）、虚拟机栈。

### 1. 虚拟机栈

**栈管运行，堆管存储。**

1. 作用：主管Java程序的运行，它**保存方法的局部变量**（八种基本数据类型，对象的引用地址）、部分结果，并**参与方法的调用与返回**。
   * 局部变量 vs 成员变量（或属性）
   * 基本数据类型变量  vs  引用类型变量（类、数组、接口）

---

2. 栈中可能出现的异常

   * 线程请求分配的**栈容量超过Java虚拟机栈**允许的最大容量，Java虚拟机将会抛出一个StackOverflowError  异常。

     * 调用栈深度超过限制 ， 递归运算时会遇到

   * 如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存区创建对应的虚拟机栈（虚拟机栈是线程独有），那Java虚拟机将会抛出一个**OutOfMemoryError**异常。

     > 两者区别 [OOM与StackOverFlow区别及产生的原因 (mstacks.com)](http://www.mstacks.com/37/1214.html)
     >
     > 一个是已经分配了栈，然后由于方法的调用层数过多 --- StackOverflowError
     >
     > 一个是给线程分配虚拟机栈等这些线程各自拥有的结构而内存不足   ---- OOM



---

3. Java方法返回的两种方式
   * 一种是使用return
   * 一种是方法内部发生异常，会导致栈帧出栈，方法返回到上一个调用

---







#### 1.1 局部变量表

> 定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量

* 局部变量表所需的**容量大小是在编译期确定下来**的，并保存在方法的Code属性的maximum，在**方法运行过程中不会改变**。

  > java的数组其实是引用，所以数组元素实际存储的位置是在堆中？

* 方法嵌套调用的次数由栈的大小决定。对于一个方法而言，它的**参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大**，以满足方法调用所需传递的信息增大的需求。进而方法调用就会占用更多的栈空间，导致其嵌套调用次数就会减少。



1. **main方法解析**

   * 右侧的Start PC和Line Number是字节码指令行号和java代码行号之间的对应关系

   ![image-20210811141443790](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210811141443790.png)

   

   * 可以看到main方法中有3个变量，第一个是args，**L表示是一个引用类型**

     StartPC依旧是字节码指令的行号，而**length和Start PC相加就是该变量对应的作用域范围**。

     ![image-20210811141838653](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210811141838653.png)

---





2. **关于slot理解**

   * 局部变量表，最基本的存储单元是Slot（变量槽）
   * 在局部变量表中，32位以内的类型只占用一个slot（包括returnAddress类型，引用类型（reference）），**64位类型（long和double）占用两个slot**。
   * 如果需要访问局部变量表中一个64bit的局部变量值时，只需要使用前一个索引即可。（那如何知道对应变量的类型？即取数据时怎么判断它是32还是64位的）
   * slot存在重复利用，如果一个局部变量出了作用域，那么之后的变量可以使用这个slot

   * **this 只存在构造方法和实例方法的局部变量表中**

     因为构造方法中的this代表当前正在创建的对象，该对象引用this将会存放在index为0的slot处。

     ![image-20210710125525392](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210710125525392.png)

     >  如上图所示，第一个变量是this，索引值为0，而double对应的weight对应两个索引3，4x

   ---

   3. **静态变量和局部变量的区别**

      * 变量的分类

        * 按类型分： 基本数据类型 和 引用数据类型

        * 按照在类中声明的位置分：

          1. **成员变量**：在使用前**都经历过默认初始化赋值**

          ​		**类变量**：准备（Prepare）阶段会给类变量一个默认赋值，在        初始化（Initial）阶段会显式赋值，即静态代码块赋值

          ​		**实例变量**：随着对象创建，会在堆空间中分配实例变量空间，并进行默认赋值

          2. **局部变量**：在使用前，必须要进行显示赋值，否则，编译不通过。 

             > 为什么要显示赋值，因为局部变量表是放在栈中的，如果需要jvm进行初始化，会影响性能
             >
             > 局部变量表中的变量也是重要的垃圾回收根节点

---

补充数据结构知识：https://blog.csdn.net/yldmkx/article/details/109996386



#### 1.2 操作数栈

> 在方法执行过程中，根据字节码指令，往栈中写入数据或提取数据，即入栈（push）/出栈（pop）
> java的解释引擎是基于栈的执行引擎，其中的栈指的就是操作数栈

个人理解： 因为有些操作并不是直接一步得到结果的，所以需要一个地方存放中间变量



1. **字节码指令是先将数值放到操作数栈，然后再放到局部变量表**。因为就像操作系统需要寄存器一样，局部变量表是用于存放变量，而对变量的具体操作（可能改变了变量的值）是在操作数栈中，就像后缀表达式一样，是需要一个结构去存放每一步计算结果，而使用局部变量表存放有点浪费。

   ![image-20210812184900723](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210812184900723.png)

   > 蓝框的都是涉及到操作数栈的
   >
   > iload_1 是加载局部变量表中第2个变量，然后压入操作数栈中。
   >
   > iadd 是操作数栈中的前两个变量相加，并将结果压入操作数栈顶
   >
   > istore_1  是将操作数栈中的栈顶元素出战，然后放入局部变量表对应的位置（第2个位置）
   >
   > iinc 1 by 1 将局部变量表中位置为“1”的 i 加 1  没有入操作数栈操作

   ![image-20210812200252515](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210812200252515.png)

   > * 所以i3++是局部变量表中增加1，没有把i3压入操作数栈，操作数栈中依旧是放的加1之前的i3（也就是10）
   >
   > * i5是先局部变量表中加1，然后将新的i5（11）压入操作数栈，再把操作数栈中的值（11）放入局部变量表第7个位置（也就是i6）

   

2. 如果被调用的方法带有返回值的话，其**返回值将会被压入当前栈帧的操作数栈中**，并更新PC寄存器中下一条需要执行的字节码指令。



#### 1.3 栈顶缓存 (Top of Stack Cashing) 技术

HotSpot JVM的设计者将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读写次数





#### 1.4 动态链接

> 帧数据区：动态链接、方法返回地址、一些附加信息

* 每一个栈帧内部都包含一个指向**运行时常量池**中**该栈帧所属方法的引用**。包含这个引用的目的就是为了支持当前方法的代码能够实现**动态链接**（Dynamic Linking）

  方法是对应一个栈帧的。

  > 动态链接的作用是为了将这些**符号引用转化为调用方法的直接引用**
  >
  > * 符号引用：是**描述所引用目标**
  > * 直接引用：是**一个指针、相对偏移量 是用于定位目标对象的句柄**

​	在字节码文件中有一个区域就叫**常量池**（是字节码文件中的）   不同于运行时常量池 （是方法区里面的）

​	![image-20210813141616998](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210813141616998.png)

* 在Java源文件倍编译到字节码文件中时，**所有的变量和方法引用**都作为**符号引用**保存在class文件的**常量池里**。

![image-20210813153754092](C:\Users\ZhuFenglong\AppData\Roaming\Typora\typora-user-images\image-20210813153754092.png)

> 不同的栈帧可能会指向同一个方法（如果把每一个方法都直接放在栈帧中，不好维护且开销大），通过引用指向方法，更方便
>
> 为什么需要常量池？
>
> 常量池的作用，就是为了提供一些符号和常量，便于指令的识别



#### 1.5 方法的调用

JVM中，将符号引用转换为调用方法的直接引用与方法的绑定机制相关。

> 字节码中方法调用其他方法，是需要将符号引用转化为直接引用，而转化过程需要借助字节码中常量池

* 静态链接

  被调用的目标方法在编译期可知，且运行期保持不变时。这种情况下将调用方法的符号引用转化为直接引用过程称之为静态链接

* 动态链接

  被调用方法在编译期无法被确定下来，只能够在程序运行期将调用方法的符号引用转换为直接引用，这种引用转换过程具备动态性，因此也就称之为动态链接

> 这和c++的好像有点不一样，静态链接的过程就已经把**要链接的内容已经链接到了生成的可执行文件中**（就是打包在一起），就算你再去把静态库删除也不会影响可执行程序的执行；而动态链接这个过程却没有把内容链接进去，而是**在执行的过程中，再去找要链接的内容**，生成的可执行文件中并没有要链接的内容，所以当你删除动态库时，可执行程序就不能运行。
>
> [静态链接和动态链接区别 - 怀想天空_2013 - 博客园 (cnblogs.com)](https://www.cnblogs.com/cyyljw/p/10949660.html)
>
> [md文档]: 静态链接和动态链接区别.md

* 对应的方法绑定机制：早期绑定(Early Binding)和晚期绑定

  > 绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生次

  * 早期绑定：**目标方法如果在编译器可知**，且运行期保持不变时，即可将这个方法与所属的类型进行绑定。
  
  * 晚期绑定：如果被调用的方法在编译期无法被确定下来，只能够**在程序运行期根据实际的类型绑定相关的方法**。
  
    ```python
    package early_later_bind;
    
    import javax.sound.midi.Soundbank;
    import java.sql.SQLOutput;
    
    class AnimalTest {
        public void eat(){
            System.out.println("动物进食");
        }
    
        public void showHunt(Huntable h){
            h.hunt(); // 表现为：晚期绑定   
        }
    }
    interface Huntable{
        void hunt();
    }
    
    class Dog extends AnimalTest implements Huntable{
        @Override
        public void eat() {
            System.out.println("狗吃骨头");
        }
    
        @Override
        public void hunt() {
            System.out.println("狗拿耗子，多管闲事");
        }
    
    }
    
    class Cat extends AnimalTest implements Huntable{
    
        public Cat(){
            super();
        }
    
        public Cat(String name){
            this();   // 表现为： 早期绑定
        }
    
        @Override
        public void eat() {
            super.eat();
            System.out.println("猫吃鱼");
        }
    
        @Override
        public void hunt() {
            System.out.println("猫抓老鼠，天经地义");
        }
    }
    ```
  
    > 早期绑定就是编译时就知道具体方法，晚期绑定就是不知道方法的具体对象，也就不知道具体方法实现是哪个
    
    * java中**任何一个普通的方法其实都具备虚函数**的特征，他们相当于C++中的虚函数（C++中则需要使用关键字virtual来进行定义）。其实也就是这个方法可以被重写，具有多态的特性（允许父类的指针指向子类的实例）。

---

* 虚方法与非虚方法

  如果**方法在编译期就确定了具体的调用版本**，这个版本在运行时是不可变的，这样的方法称为非虚方法。

  * 静态方法、私有方法、final方法、实例构造器、父类方法（**因为子类调用父类方法是super. ，这样并不会有多态，而是invokespecial**  [父类方法属于非虚方法吗？ | HeapDump性能社区](https://www.heapdump.cn/question/246602?type=parent&last=247702)）都是非虚方法 

  > 静态方法是不能被重写的，因为静态方法其实也就是类方法，是确定的
  >
  > 带有final修饰符的方法也不能被重写，所以也是非虚方法
  >
  > ---
  >
  > 如果是类自己的实例方法，因为不知道是否还有其他子类是否重写该方法，所以实例方法是虚方法（编译时不能确定具体方法）。  
  >
  > 接口的方法也不确定具体实现是哪一个（因为接口可以指向实现它的类实例）
  

---

2021-8-26



符号引用和直接引用

> 符号引用就是在加载时, class文件里不能确定的, 给他个符号 比如#1 ,表示某个方法
>
> 直接引用就是指向内存中对象的指针、相对偏移量、一个能间接定位的句柄  所以引用的目标必定已经被加载入内存中了。

[ref]:符号引用.md



##### 虚方法表

> 作用：为了提高性能，JVM采用在类的方法区建立一个虚方法表（virtual method table）来实现。使用索引表来替代查找。  因为每次使用虚方法，会频繁使用到动态分配
>
> 虚方法表会在类加载的链接阶段被创建并开始初始化，类的变量初始化值准本完成之后，JVM会把该类的方法表也初始化完毕。
>
> 

<img src="C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20210826145236436.png" alt="image-20210826145236436" style="zoom:50%;" />

这里的Cat实现了Friendly接口，并且由于Java的每一个类都会继承Object类，所以Cat的虚方法表会包括两部分

* 一部分是继承来自Object的方法
* 一部分是Cat自己的方法   

> 为什么这两类是虚方法，因为Object的方法是Cat类继承来的，之后Cat可能会存在继承他的子类，所以这些方法并不是之前提的父类方法（super.调用的）
>
> 而Cat实现的接口方法也是同样的道理，之后可能会被子类继承并重写，要在运行时才知道具体的调用方法。
>
> static、final、private关键字修饰的方法不会被继承，也就不可能重写，编译器就能明确调用的具体方法

![image-20210826154439413](..\..\typora-user-images\image-20210826154439413.png)



> 可以看到常量池中就是包括这些虚方法的符号引用





#### 1.6 方法返回地址

* 存放调用该方法的pc寄存器的值。

* 一个方法的结束，有两种方式：
  * 正常执行完成
  * 出现未处理的异常，非正常退出
* 无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。**方法正常退出时，调用者的pc计数器值作为返回地址，即调用该方法的指令的下一条指令的地址**。而异常退出，**返回地址是要通过异常表来确定**，栈帧中一般不会保存这部分信息。

---

> 1. 调用者的pc计数器值作为**返回地址**
> 2. 具体的方法返回数据是**返回值**

##### 1. 正常执行完

执行引擎遇到任意一个方法返回的字节码指令，会有返回值传递给上层的方法调用者。

* 一个方法正常调用完成后，根据方法返回类型使用具体的返回指令

![image-20210915204039370](..\..\typora-user-images\image-20210915204039370.png)

##### 2. 发生异常

方法在执行过程中遇到了异常，并且这个异常没有在方法内进行处理，也就是在本方法的异常表中没有搜索到匹配的异常处理器，会导致方法退出。简称异常完成出口。

#### 1.7 一些附加信息



#### 1.8 栈的相关面试题

1. 举例栈溢出的情况

   * StackOverflowError   栈帧放不下的时候报错
     * 通过-Xss设置栈的大小
     * OutOfMemory OOM  是申请栈的扩充，或者是新的栈，但是内存空间不足导致的错误

2. 调整栈大小，就能保证不出现溢出吗？

   >  不能

3. 分配的栈内存越大越好吗？

   >  不是，理论上栈分配的越大越能避免出现栈溢出的情况,但是会挤占其他内存结构的空间

4. 垃圾回收是否会涉及到虚拟机栈？

   > 不会，因为虚拟机栈是先进后出

5. 方法中定义的局部变量是否线程安全？

   > 具体问题具体分析
   >
   > 何为线程安全？ 
   >
   > 	* 如果只有一个线程才可以操作此数据，则必是线程安全 
   > 	* 如果有多个线程操作此数据，则此数据是共享数据。如果不考虑同步机制，会存在线程安全问题。





### 3.2 本地方法

一个Native Method就是一个Java调用非Java代码的接口

在定义一个native method时，并不提供实现体（有些像定义一个java interface），其实现体是由非java语言在外部实现。

1. 为什么要使用Native Method

   * 有些任务用java实现不方便
   * 与java环境外交互。本地方法：为我们提供了一个非常简介的接口，我们无需了解Java应用之外的繁琐的细节
   * 与操作系统交互。JVM依赖底层系统的支持，而底层系统有一部分就是c写的。
   * sun的解释器是用c实现的，这使得它能像一些普通的c一样与外部交互

   



### 3.3 本地方法栈

> Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于**管理本地方法的调用**

* 本地方法栈，也是线程私有的
* 允许被实现成固定或者是可动态扩展的内存大小

![本地方法栈](../../typora-user-images/本地方法栈.png)



当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界。它和虚拟机拥有同样的权限。

* 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区。
* 甚至可以直接使用本地处理器中的寄存器。
* 直接从本地内存的堆中分配任意数量的内存。

---



### 4 堆

[ ]: 堆.md

