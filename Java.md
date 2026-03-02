参考文档：https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/jvm/memory-area.md

# Java基础

## 变量和常量

- Java中的变量存在JVM哪一块？

  ~~~kotlin
  看变量的申明方式和作用域。
  - 局部变量
  - 成员变量
  - 静态变量
  ~~~

  

## 基本数据类型

**对于浮点数值的等值判断，基本数据类型不能用==，包装数据类型不能用equals.**

为什么浮点数运算有时候会丢失精度？

```
float a = 2.0f - 1.9f;
float b = 1.8f - 1.7f;
System.out.printf("%.9f",a);// 0.100000024
System.out.println(b);// 0.099999905
System.out.println(a == b);// false
```

这个和计算机保存浮点数的机制有很大关系。我们知道计算机是二进制的，而且计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，所以就会导致小数精度发生损失的情况。这也就解释了为什么浮点数没有办法用二进制精确表示。

比如在十进制中，1/3是0.3333..是无法精确表示的，而在二进制中，十进制的0.1这样的小数也无法被精确表示，0.0001100110011...（循环部分为0011）

BigDecimal为什么不会造成浮点数精度丢失？

### BigDecimal

#### BigDecimal原理

绕开了二进制的局限性，使用了十进制整数运算。

- **无标度整数值**
- **标度**

~~~kotlin
BigDecimal("3.4")+BigDecimal("1.22")
BigDecimal会将3.4拆分为无标度的整数值34，和标度1，同理会将1.22拆分为无标度的整数值122和标度2，通过将整数值进行缩放来确保只操作整数来实现operation。而整数的操作肯定是安全的，从而不会有精度问题。
~~~



- BigDecimal的等值比较应实用compareTo,而不是equals。（compareTo会忽略精度。）

  ~~~Java
  BigDecimal a = new BigDecimal("1");
  BigDecimal b = new BigDecimal("1.0");
  System.out.println(a.equals(b));//false
  ~~~

#### 构造函数

推荐使用BigDecimal(String valStr) 来构造，或者BigDecimal.valueOf()系列方法来构造，而不能直接使用BigDecimal(double a),会有精度丢失风险。

#### 常用方法

需要注意除法，a.dive

~~~Java
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
System.out.println(a.add(b));// 1.9
System.out.println(a.subtract(b));// 0.1
System.out.println(a.multiply(b));// 0.90
System.out.println(a.divide(b));// 无法除尽，抛出 ArithmeticException 异常
System.out.println(a.divide(b, 2, RoundingMode.HALF_UP));// 1.11
这里需要注意的是，在我们使用 divide 方法的时候尽量使用 3 个参数版本，并且RoundingMode 不要选择 UNNECESSARY，否则很可能会遇到 ArithmeticException（无法除尽出现无限循环小数的时候），其中 scale 表示要保留几位小数，roundingMode 代表保留规则。
~~~



## 包装类型

### 区别

- 包装类型可以用于泛型

- 包装类型是对象，存放在堆中。

  基本类型的局部成员变量要看它的申明方式和作用域，比如局部变量就存在放在局部变量表中，如果是成员变量，会存放在堆区/方法区/元空间中。

- 占用空间不同

- 默认值不同。包装类默认值是null

- 比较方式不同。

### 自动装箱拆箱

```
Integer i = 10;  //装箱，等价于Integer i = Integer.valueOf(10)
int n = i;   //拆箱，等价于int n = i.intValue();
```

频繁拆装箱也会影响系统的性能，应当尽量避免不必要的拆装箱操作。

### 包装类型缓存机制

Java 基本数据类型的包装类型的大部分都用到了缓存机制来提升性能。

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `TRUE` or `FALSE`。（两种浮点数类型的包装类 `Float`,`Double` 并没有实现缓存机制。）

对于 `Integer`，可以通过 JVM 参数 `-XX:AutoBoxCacheMax=<size>` 修改缓存上限，但不能修改下限 -128。实际使用时，并不建议设置过大的值，避免浪费内存，甚至是 OOM。

对于`Byte`,`Short`,`Long` ,`Character` 没有类似 `-XX:AutoBoxCacheMax` 参数可以修改，因此缓存范围是固定的，无法通过 JVM 参数调整。`Boolean` 则直接返回预定义的 `TRUE` 和 `FALSE` 实例，没有缓存范围的概念。

**所有整型包装类对象之间值的比较，全部使用 equals 方法比较**。

~~~kotlin
//Integer缓存源码
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
~~~

### QA

```
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);
//会输出false。   Integer i1 = 40;等同于Integer i1=Integer.valueOf(40)，会从缓存取。
```

# 面向对象

== 和 euals区别？

面向对象三特征：

继承，

封装，

多态

- 一个对象具有多种状态，具体表现为父类的引用，指向子类的实例。

## 构造方法

- 一个类没有设置构造方法，能运行吗？可以的有默认无参构造方法。
- 构造方法可以重载不能重写。

浅拷贝和深拷贝区别？

# JVM

## Runtime 运行时数据区

### 程序计数器

- 特点：

线程私有，生命周期与线程同步，占用内存小，Runtime里唯一不会OOM的区域。

- 主要作用：

1. 记录当前线程的执行位置（线程切换的时候能知道上次运行到哪儿了）
2. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制。（如：顺序执行、选择、循环、异常处理）

在执行 **Java 方法**（非 native）时，程序计数器记录的是 **当前正在执行的 JVM 字节码指令的地址**。当线程执行的是一个 **native 方法**（本地方法）时，程序计数器的值为 **Undefined（未定义）**。这是因为 native 方法不执行 JVM 字节码，而是通过 JNI 调用本地平台的底层代码，JVM 无需再跟踪字节码地址。

### 虚拟机栈 Java Stack

- 特点

  线程私有，生命周期同线程，由一个个栈帧构成。每个方法调用会有一个栈帧入栈，方法结束会有一个栈帧出栈。

- 栈帧 Stack Frame

  - 局部变量表

    存放编译器可知的各种数据类型（boolean、byte、char、short、int、float、long、double），对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）

  - 操作数栈

    主要作为方法调用的中转站使用，用于存放方法执行过程中产生的**中间计算结果**。另外，计算过程中产生的**临时变量**也会放在操作数栈中。

  - 动态链接

    核心作用是**在运行时解析虚方法的调用点，将其链接到正确的方法版本上**。**动态链接**是 Java 虚拟机实现方法调用的关键机制之一。在 Class 文件中，方法调用以**符号引用**的形式存在于常量池。为了执行调用，这些符号引用必须被转换为内存中的**直接引用**。这个转换过程分为两种情况：对于静态方法、私有方法等在编译期就能确定版本的方法，这个转换在**类加载的解析阶段**就完成了，这称为**静态解析**。而对于需要根据对象实际类型才能确定具体实现的**虚方法**（这是实现多态的基础），这个转换过程则被推迟到**程序运行期间**，由**动态链接**来完成。

  - 方法返回地址

  - 。。。

- Java栈里可能的异常

  - OOM

    当栈的内存大小可以动态扩展，虚拟机在动态扩展栈时无法申请到足够的内存空间会抛出。

    实际上HotSpot虚拟机栈容量不可以动态扩展，只要线程申请栈空间成功了就不会OOM，失败了就会OOM。

  - StackOverFlow

    当栈的内存大小不允许动态扩展，当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候会抛出。

### 本地方法栈 Native Stack

与Java栈很类似，Java栈为JVM执行Java方法，也就是字节码服务，而本地方法栈则为JVM使用到的Native方法服务。

### 堆区 Heap

JVM内存中最大的一块，线程共享，存放对象的实例和数组,字符串常量池，静态变量等。是GC的主要区域。

- 新生代 Young Generation

  - Eden
  - Survivor0
  - Survivor1

- 老年代 Old Generation

- 永久代 Permanent Generation

  JDK 8 之后，永久代被元空间 Metaspace取代，元空间使用的是本地内存。为什么要将永久代替换为元空间？

  1. 永久代内存受JVM内存限制，而元空间使用本地内存，减少OOM概率。（可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited。）
  2. 元空间里存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。
  3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

- 字符串常量池 StringTable

  是一个固定大小的HashTable，容量为 `StringTableSize`（可以通过 `-XX:StringTableSize` 参数来设置），保存的是字符串（key）和 字符串对象的引用（value）的映射关系，字符串对象的引用指向堆中的字符串对象。

  JDK1.7之前存放在永久代，JDK1.7开始，字符串常量池和静态变量都存放在堆中。

  - 为什么从永久代移动到了堆区？

    主要是因为永久代（方法区实现）的 GC 回收效率太低，只有在整堆收集 (Full GC)的时候才会被执行 GC。Java 程序中通常会有大量的被创建的字符串等待回收，将字符串常量池放到堆中，能够更高效及时地回收字符串内存。

### 方法区 Method Area

方法区是《Java 虚拟机规范》中规定的定义和作用，不同虚拟机实现不同。永久代和元空间就是方法区的具体实现方式。

当JVM加载一个类时，它会从Class文件中解析出相应的信息，并将这些元数据存入方法区。方法区主要存储以下核心数据：

1. 类的元数据

   包括类的完整结构，如类名、父类、实现的接口、访问修饰符，以及字段和方法的详细信息（名称、类型、修饰符等）。

2. 方法的字节码

   每个方法的原始指令序列。

3. 运行时常量池 Constant Pool Table

   每个类独有的，由 Class 文件中的常量池转换而来，用于存放编译期生成的各种字面量和对类型、字段、方法的符号引用。

需要特别注意的是，以下几类数据虽然在逻辑上与类相关，但在 HotSpot 虚拟机中，它们并不存储在方法区内：

- **静态变量（Static Variables）**：自 JDK 7 起，静态变量已从方法区（永久代）**移至 Java 堆（Heap）中**，与该类的 `java.lang.Class` 对象一起存放。
- **字符串常量池（String Pool）**：同样自 JDK 7 起，字符串常量池也**移至 Java 堆中**。
- **即时编译器编译后的代码缓存（JIT Code Cache）**：JIT 编译器将热点方法的字节码编译成的本地机器码，存放在一个**独立的、名为“Code Cache”的内存区域**，而不是方法区本身。这样做是为了实现更高效的执行和内存管理。

JDK1.7起元空间常用参数：

~~~xml
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
~~~

JDK1.7之前永久代常用参数：

~~~xml
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
~~~

## 直接内存

直接内存是一种特殊的内存缓冲区，并不在 Java 堆或方法区中分配的，而是通过 JNI 的方式在本地内存上分配的。这部分内存也会被频繁使用，也会导致OOM。

JDK1.4 中新加入的 **NIO（Non-Blocking I/O，也被称为 New I/O）**，引入了一种基于**通道（Channel）与缓存区（Buffer）的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在 Java 堆和 Native 堆之间来回复制数据**。

类似的概念还有堆外内存，在一些文章中将直接内存等价于堆外内存，个人觉得不是特别准确。堆外内存就是把内存对象分配在堆外的内存，这些内存直接受操作系统管理（而不是虚拟机），这样做的结果就是能够在一定程度上减少垃圾回收对应用程序造成的影响。

## 对象

### 对象的内存布局

1. 对象头 Header

   1. 标记字段 Mark Word

      存放对象自身的运行时数据，比如HashCode, GC分代年龄，锁状态标志，线程持有的锁，偏向线程ID，偏向时间戳等。

   2. 类型指针 Klass Pointer

      对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

2. 实例数据 InstanceData

   **实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

3. 对齐填充 Padding

   **对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用**。

   Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

### 对象的创建流程

1. 类加载检查
2. 分配内存
3. 初始化
4. 设置对象头
5. 执行init方法

### 对象的访问方式

1. 句柄
2. 指针

# 集合

## ArrayList

### 特点

动态数组，底层是数组队列。

- 线程不安全

- 可以添加null值，但是不建议。

### 扩容机制

## LinkedList

