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

## 浅拷贝和深拷贝

- 引用拷贝

  两个引用指向同一个对象。

- 浅拷贝

  1. 会在堆区创建一个新的对象。

  2. 对于对象的内部属性，会复制基本类型，共享引用类型。

- 深拷贝

  深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

深拷贝与浅拷贝的区别，就是对于对象内部的对象，深拷贝也会复制，但是浅拷贝不会。

## Object类

- Class<?> getClass()

  返回当前运行时对象的Class对象。

- int hashCode()

- boolean equals()

- Object clone()

- String toString()

- void notify(),notifyAll(),wait(..)

- void finalize()

### == 和 equals()区别？

- ==

  基本数据类型比较值，引用数据类型比较对象的内存地址。

- equals

  equals是Object类中的方法，只能用于对象，判断两个对象是否相等。一般我们都重写 `equals()`方法来比较两个对象中的属性是否相等。

### hashCode()有什么用？

`hashCode()` 的作用是获取哈希码（`int` 整数），也称为散列码。这个哈希码的作用是确定该对象在哈希表中的索引位置。

能提升部分容器（HashMap，HashSet等）的查询速度。

hashCode()相等，可能相等，不等则肯定不同。

### 为什么重写 equals() 时必须重写 hashCode() 方法？

因为两个对象相等，他们的hashCode必须相等。如果没有重写，可能会导致在hashMap等数据结构使用异常。

## String

### String、StringBuffer、StringBuilder 的区别？

`String` 中的对象是不可变的，也就可以理解为常量，线程安全

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

### String 为什么是不可变的?

1. 保存字符串的数组被 `final` 修饰且为私有的，并且`String` 类没有提供/暴露修改这个字符串的方法。
2. `String` 类被 `final` 修饰导致其不能被继承，进而避免了子类破坏 `String` 不可变。

~~~Java
public final class String implements java.io.Serializable,Comparable<String>, CharSequence {
    // @Stable 注解表示变量最多被修改一次，称为“稳定的”。
   //Java 9 将 String 的底层实现由 char[] 改成了 byte[] 
    @Stable
    private final byte[] value;
}

abstract class AbstractStringBuilder implements Appendable, CharSequence {
    byte[] value;

}
~~~

### String拼接

字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。`StringBuilder` 对象是在循环内部被创建的，这意味着每循环一次就会创建一个 `StringBuilder` 对象。如果直接使用 `StringBuilder` 对象进行字符串拼接的话，就不会存在这个问题了。

### 字符串常量池的作用了解吗？

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

### String s1 = new String("abc");这句话创建了几个字符串对象？

会创建 1 或 2 个字符串对象。

1. 字符串常量池中不存在 "abc"：会创建 2 个 字符串对象。一个在字符串常量池中，由 `ldc` 指令触发创建。一个在堆中，由 `new String()` 创建，并使用常量池中的 "abc" 进行初始化。
2. 字符串常量池中已存在 "abc"：会创建 1 个 字符串对象。该对象在堆中，由 `new String()` 创建，并使用常量池中的 "abc" 进行初始化。

# 类文件结构

JVM可以理解的代码叫做字节码，也就是.class文件。

ClassFile结构如下：

~~~Java
ClassFile {
    u4             magic; //Class 文件的标志，魔数为固定值：0xCAFEBABE。不是以0xCAFEBABE开头JVM不会加载。
    u2             minor_version;//Class 的小版本号
    u2             major_version;//Class 的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池，要存放两大常量：字面量（如文本字符串、声明为 final 的常量值）和符号引用（类和接口的全限定名，字段的名称和描述符，方法的名称和描述符）。
    u2             access_flags;//Class 的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口数量
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//字段数量
    field_info     fields[fields_count];//一个类可以有多个字段
    u2             methods_count;//方法数量
    method_info    methods[methods_count];//一个类可以有多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
~~~

通过 IDEA 插件 `jclasslib` 可以更直观看到 Class 文件结构。

# JVM

## Runtime 运行时数据区

### 程序计数器

- 特点：

线程私有，生命周期与线程同步，占用内存小，Runtime里唯一不会OOM的区域。

- 主要作用：

1. **记录当前线程的执行位置**（线程切换的时候能知道上次运行到哪儿了，因此必须线程私有）
2. 字节码解释器通过改变程序计数器来依次读取指令，从而**实现代码的流程控制**。（如：顺序执行、选择、循环、异常处理）

在执行 **Java 方法**（非 native）时，程序计数器记录的是 **当前正在执行的 JVM 字节码指令的地址**。当线程执行的是一个 **native 方法**（本地方法）时，程序计数器的值为 **Undefined（未定义）**。这是因为 native 方法不执行 JVM 字节码，而是通过 JNI 调用本地平台的底层代码，JVM 无需再跟踪字节码地址。

### 虚拟机栈 Java Stack

- 特点

  线程私有，生命周期同线程，由一个个栈帧构成。每个方法调用会有一个栈帧入栈，方法结束会有一个栈帧出栈。

- 栈帧 Stack Frame

  - 局部变量表

    存放编译器可知的各种数据类型（boolean、byte、char、short、int、float、long、double），对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）

    （为了防止线程中的局部变量被其他线程访问，虚拟机栈和本地方法栈都必须是线程私有的）

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

JVM内存中最大的一块，线程共享，存放对象的实例和数组,字符串常量池，静态变量等。是GC的主要区域。也称为GC堆

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

## JVM垃圾回收

这里的JVM都是HotSopt JVM，垃圾回收主要发生在堆区，堆区也称为Garbage Collected Heap。

### 引用类型

1. 强引用 Strong Reference

2. 软引用 Soft Reference

   内存压力大时可能会被回收（OOM之前一定会被清理）。可以实现内存敏感的**高速缓存**。

3. 弱引用 Weak Reference

   发生GC，就会被回收。但是垃圾回收器是一个优先级很低的线程，不一定很快发现那些只具有弱引用的对象。

4. 虚引用 Phantom Reference

   主要用来跟踪对象被垃圾回收的活动，**必须和引用队列ReferenceQueue联合使用**。当垃圾回收期在回收对象之前，如果发现有这个对象有虚引用，那么会把这个虚引用加入与之关联的引用队列中。因此可以通过判断引用队列中是否已经加入虚引用，在对象被回收之前采取措施。

### 对象是否是垃圾判断

- 引用计数

  有循环引用问题，目前主流虚拟机没有选择这个方法来管理内存。（？？？方法有问题都没有虚拟机用为什么还要记这个？？？）

- 可达性分析法

  当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的，需要被回收。

### GC Roots

哪些对象可以作为GC Roots？

- 虚拟机栈(栈帧中的局部变量表)中引用的对象

  ~~~kotlin
  fun test(){
    val obj = Dog() //该方法执行时，obj会被存入局部变量表，obj是一个活动引用，指向堆区中的Dog实例，在方法执行期间，该实例不会被内存回收。
  }
  ~~~

- 本地方法栈(Native 方法)中引用的对象

- 方法区中类静态属性引用的对象

  ~~~Java
  public class StaticReference {
      private static List<String> cache = new ArrayList<>(); // 静态引用
      public static final String CONSTANT_STRING = "Hello"; // 字符串常量
      public static final Object CONSTANT_OBJECT = new Object(); // 常量对象
      //类的静态属性和字符串常量池都已经从方法区移到了堆区（运行时常量池还在方法区），类的静态引用对象和常量对象也会一直作为GC Roots，只要类未被卸载。
      public void addToCache(String value) {
          cache.add(value);
      }
  }
  ~~~

- 方法区中常量引用的对象

- 所有被同步锁持有的对象

- JNI（Java Native Interface）引用的对象

对象可以被回收，就一定会被回收吗？重写一次finalize方法可以有一次逃逸的机会。

`Object` 类中的 `finalize` 方法一直被认为是一个糟糕的设计，成为了 Java 语言的负担，影响了 Java 语言的安全和 GC 的性能。JDK9 版本及后续版本中各个类中的 `finalize` 方法会被逐渐弃用移除。

### 常量是否是垃圾判断

运行时常量池（方法区）主要回收的是废弃的常量。（类的静态变量，字符串常量池都被移到了堆区）。

假如在字符串常量池中存在字符串 "abc"，如果当前没有任何 String 对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池了。

### 类是否是垃圾判断

- 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
- 加载该类的 `ClassLoader` 已经被回收。
- 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

虚拟机可以对满足上述 3 个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样不使用了就会必然被回收。

### 垃圾收集算法

- 标记-清除（Mark-and-Sweep）

  先标记，再清除，是最基础的收集算法，后续算法都是对其改进。存在问题：

  1. 效率问题。标记和清除两个过程效率都不高。
  2. 空间问题。会产生大量不连续的内存碎片。

- 标记-复制算法

  它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

  解决了内存碎片，提升了效率。但是存在下面问题：

  1. 可用内存变小。
  2. 不适合老年代。（如果存活对象数量很大，复制性能会变得很差）

- 标记-整理 Mark-and-Compact

  根据老年代的特点提出的一种标记算法,标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。多了一步整理，因此效率也不高，但是适合老年代这种垃圾回收频率不高的场景。

### 垃圾收集器

1. Serial 收集器 单线程

2. ParNew 收集器 Serial的多线程版本

3. Parallel Scavenge 收集器

   JDK1.8的默认收集器。**新生代采用标记-复制算法，老年代采用标记-整理算法。**

### 自定义GCMonitor监控GC

~~~Java
public class GCMonitor {
    private static final String TAG = "GCMonitor";
    private static final long GC_THRESHOLD_MS = 10; // 超过10ms的GC视为耗时过长
    
    public static void startMonitoring() {
        final long startTime = System.currentTimeMillis();
        
        // 添加GC监听器
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                long duration = System.currentTimeMillis() - startTime;
                if (duration > GC_THRESHOLD_MS) {
                    Log.w(TAG, "GC took " + duration + "ms, which is longer than threshold");
                }
            }
        });
    }
}

// 在Application类中启动监控
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        GCMonitor.startMonitoring();
    }
}
~~~



## JVM 参数设置

https://github.com/Snailclimb/JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md



# 类加载流程

## 类生命周期

1. **加载**

2. **连接**

   1. 验证

      确保字节流中包含的信息符合《Java 虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。可以考虑使用 `-Xverify:none` 参数来关闭大部分的类验证措施。但是这个方法已经被废弃。

      - 文件格式验证
      - 元数据验证
      - 字节码验证
      - 符号引用验证

   2. 准备

      正式为类变量（**静态变量**）分配**内存**并设置类变量**初始值**

   3. 解析

      **解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程**

3. **初始化**

   **初始化阶段是执行初始化方法 `<clinit> ()`方法的过程，是类加载的最后一步，这一步 JVM 才开始真正执行类中定义的 Java 程序代码(字节码)。**

4. 使用

5. 卸载

   卸载类需要满足 3 个要求:

   1. 该类的所有的实例对象都已被 GC，也就是说堆不存在该类的实例对象。
   2. 该类没有在其他任何地方被引用
   3. 该类的类加载器的实例已被 GC

   所以，在 JVM 生命周期内，由 jvm 自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。

## 加载

1. 通过全类名**获取**定义此类的**二进制字节流**。
2. 将字节流所代表的静态存储结构**转换为方法区的运行时数据结构**。
3. 在**内存中生成**一个代表该类的 **`Class` 对象**，作为方法区这些数据的访问入口。

### 类加载器 ClassLoader

主要作用就是动态加载 Java 类的字节码（ `.class` 文件）到 JVM 中（在内存中生成一个代表该类的 `Class` 对象）。其实除了加载类之外，类加载器还可以加载 Java 应用所需的资源如文本、图像、配置文件、视频等等文件资源。本文只讨论其核心功能：加载类。

JVM 启动的时候，并不会一次性加载所有的类，而是根据需要去动态加载。也就是说，大部分类在具体用到的时候才会去加载，这样对内存更加友好。

对于已经加载的类会被放在这个类加载器的成员变量private final Vector<Class<?>> classes中。

#### 内置类加载器

- **`BootstrapClassLoader`(启动类加载器)**：最顶层的加载类，由 C++实现，通常表示为 null，并且没有父级，主要用来加载 JDK 内部的核心类库（ `%JAVA_HOME%/lib`目录下的 `rt.jar`、`resources.jar`、`charsets.jar`等 jar 包和类）以及被 `-Xbootclasspath`参数指定的路径下的所有类。

- **`ExtensionClassLoader`(扩展类加载器)**：主要负责加载 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类以及被 `java.ext.dirs` 系统变量所指定的路径下的所有类。

- **`AppClassLoader`(应用程序类加载器)**：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。

  除了 `BootstrapClassLoader` 是 JVM 自身的一部分之外，其他所有的类加载器都是在 JVM 外部实现的，并且全都继承自 `ClassLoader`抽象类。这样做的好处是用户可以自定义类加载器，以便让应用程序自己决定如何去获取所需的类。

  每个 `ClassLoader` 可以通过`getParent()`获取其父 `ClassLoader`，如果获取到的 `ClassLoader` 为`null`的话，那么该类加载器的父类加载器是 `BootstrapClassLoader` 。（`BootstrapClassLoader` 由 C++ 实现，由于这个 C++ 实现的类加载器在 Java 中是没有与之对应的类的，所以拿到的结果是 null。）

#### 自定义类加载器

`ClassLoader` 类有两个关键的方法：

- `protected Class loadClass(String name, boolean resolve)`：加载指定二进制名称的类，实现了双亲委派机制 。`name` 为类的二进制名称，`resolve` 如果为 true，在加载时调用 `resolveClass(Class<?> c)` 方法解析该类。
- `protected Class findClass(String name)`：根据类的二进制名称来查找类，默认实现是空方法。

官方 API 文档中写到：

> Subclasses of `ClassLoader` are encouraged to override `findClass(String name)`, rather than this method.
>
> 建议 `ClassLoader`的子类重写 `findClass(String name)`方法而不是`loadClass(String name, boolean resolve)` 方法。

如果我们不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写 `loadClass()` 方法。

#### 双亲委派机制

- `ClassLoader` 实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。
- 除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器

这里的双亲委派机制，意思就是交给父辈去加载。（这翻译多少有点问题）并且不是强制的，只是JDK官方推荐的。

双亲委派机制执行流程:

~~~java 
//
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        //首先，检查该类是否已经加载过
        Class c = findLoadedClass(name);
        if (c == null) {
            //如果 c 为 null，则说明该类没有被加载过
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //当父类的加载器不为空，则通过父类的loadClass来加载该类
                    c = parent.loadClass(name, false);
                } else {
                    //当父类的加载器为空，则调用启动类加载器来加载该类
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                //非空父类的类加载器无法找到相应的类，则抛出异常
            }

            if (c == null) {
                //当父类加载器无法加载时，则调用findClass方法来加载该类
                //用户可通过覆写该方法，来自定义类加载器
                long t1 = System.nanoTime();
                c = findClass(name);

                //用于统计类加载器相关的信息
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            //对类进行link操作
            resolveClass(c);
        }
        return c;
    }
}
~~~

**JVM 判定两个 Java 类是否相同的具体规则**：JVM 不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即使两个类来源于同一个 `Class` 文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相同。

#### 双亲委派模型的好处

双亲委派模型是 Java 类加载机制的重要组成部分，它通过委派父加载器优先加载类的方式，实现了两个关键的安全目标：避免类的重复加载和防止核心 API 被篡改。

然而，即使攻击者绕过了双亲委派模型，Java 仍然具备更底层的安全机制来保护核心类库。`ClassLoader` 的 `preDefineClass` 方法会在定义类之前进行类名校验。任何以 `"java."` 开头的类名都会触发 `SecurityException`，阻止恶意代码定义或加载伪造的核心类。

#### 如何打破双亲委派机制？

双亲委派机制的实现主要是loadClass方法，那么重写loadClass方法，我们自己先加载，不交由父加载器来加载就行了。

我们比较熟悉的 Tomcat 服务器为了能够优先加载 Web 应用目录下的类，然后再加载其他目录下的类，就自定义了类加载器 `WebAppClassLoader` 来打破双亲委托机制。这也是 Tomcat 下 Web 应用之间的类实现隔离的具体原理。

# 集合

https://javaguide.cn/java/collection/java-collection-questions-01.html

## List

### ArrayList

### 特点

动态数组，底层是数组队列。

- 线程不安全

- 可以添加null值，但是不建议。

### 扩容机制



### LinkedList

### Vector

## Set

## Queue

## Map

# 并发

## 线程

- 线程的状态：
  - NEW，初始状态，创建了但是没有start()
  - RUNNABLE，运行状态
  - BLOCKED，阻塞状态，需要等待锁释放
  - WAITING：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）。
  - TIME_WAITING：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
  - TERMINATED：终止状态，表示该线程已经运行完毕。

- 线程的创建：

  - 继承Thread类
  - 实现runnable接口
  - 实现Callable和Futrue接口
  - 线程池等

- 线程的常用方法

  - sleep()

    - sleep和wait有什么区别？
      1. sleep是Thread类里的静态方法，wait和notify是object的方法。sleep不会释放锁，而wait会释放锁。
      2. wait通常被用于线程间的交互和通信，而sleep通常用于暂停执行。
      3. wait之后，必须notify或者notifyAll,线程才会苏醒，而sleep之后线程到时间了会自动苏醒。
    - 为什么wait方法不定义在Thread中？
      - wait方法主要是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。

  - yeiled()

  - run()

    - 可以直接调用Thread类的run()方法吗？

      **调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**

并发和并行的区别？

并发是多个任务在同一时间段内执行，并行是多个任务在同一时刻执行。因此单核CPU可以并发，但是不能并行。

## 线程池

池化思想，降低资源消耗，提高响应速度，提高线程可管理性。

### 线程池创建方式

1. 通过ThreadPoolExecutor构造函数直接创建（**推荐**）
2. 通过Executors工具类创建**(不推荐用于生产环境)**

**线程池在提交任务前，可以提前创建线程吗？**

答案是可以的！`ThreadPoolExecutor` 提供了两个方法帮助我们在提交任务之前，完成核心线程的创建，从而实现线程池预热的效果：

- `prestartCoreThread()`:启动一个线程，等待任务，如果已达到核心线程数，这个方法返回 false，否则返回 true；
- `prestartAllCoreThreads()`:启动所有的核心线程，并返回启动成功的核心线程数。

### 线程池的构造函数

```java
public ThreadPoolExecutor(
  int corePoolSize,//线程池的核心线程数量
  int maximumPoolSize,//线程池的最大线程数
  long keepAliveTime,//当线程数大于核心线程数时，多余的空闲线程存活的最长时间
  TimeUnit unit,//时间单位
  BlockingQueue<Runnable> workQueue,//任务队列，用来储存等待执行任务的队列
  ThreadFactory threadFactory,//线程工厂，用来创建线程，一般默认即可
  RejectedExecutionHandler handler//拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务
) 
```

- corePoolSize 核心线程数量

  核心线程默认不回收，即便是空闲，这是为了减少创建线程的开销。空闲时处于WAITING状态。

  设置allowCoreThreadTimeOut(true)也可以在空闲时回收核心线程。超时被移除后处于TERMINATED状态。

- 线程池拒绝策略

  当前同时运行的线程数量到达最大线程数量，并且队列也满了，会触发线程池拒绝策略。

  1. ThreadPoolExecutor.AbortPolicy。抛出 `RejectedExecutionException`来拒绝新任务的处理。

  2. ThreadPoolExecutor.DiscardPolicy。不处理新任务，直接丢弃。

  3. ThreadPoolExecutor.DiscardOldestPolicy。丢弃最早未处理的。

  4. ThreadPoolExecutor.CallerRunsPolicy。

     直接在调用`execute`方法的线程中运行(`run`)被拒绝的任务。如果不允许丢弃任务，可以尝试使用这个策略，但是有阻塞主线程的风险。如何避免这种风险？

     1. 将任务持久化
     2. 将任务添加到队列

- 线程池阻塞队列

  新任务来了的时候，会先判断当前运行的线程数量是否达到核心线程数，如果达到了，新任务会被存放到队列中。

  1. LinkedBlockingQueue ，容量没上限，

     `FixedThreadPool` 和 `SingleThreadExecutor` 。

  2. SynchronousQueue 同步队列，CachedThreadPool。

     `SynchronousQueue` 没有容量，不存储元素，目的是保证对于提交的任务，如果**有空闲线程，则使用空闲线程来处理；否则新建一个线程来处理任务**。也就是说，`CachedThreadPool` 的最大线程数是 `Integer.MAX_VALUE` ，可以理解为线程数是可以无限扩展的，可能会创建大量线程，从而导致 OOM。

  3. DelayedWorkQueue 延迟队列。ScheduledThreadPool` 和 `SingleThreadScheduledExecutor

     `DelayedWorkQueue` 的内部元素并不是按照放入的时间排序，而是会按照延迟的时间长短对任务进行排序，内部采用的是“堆”的数据结构，可以保证每次出队的任务都是当前队列中执行时间最靠前的。`DelayedWorkQueue` 是一个无界队列。其底层虽然是数组，但当数组容量不足时，它会自动进行扩容，因此队列永远不会被填满。当任务不断提交时，它们会全部被添加到队列中。这意味着线程池的线程数量永远不会超过其核心线程数，最大线程数参数对于使用该队列的线程池来说是无效的。

  4. `ArrayBlockingQueue`（有界阻塞队列）：底层由数组实现，容量一旦创建，就不能修改。

### 内置线程池

- FixedThreadPool
- SingleThreadExecutor
- CachedThreadPool
- ScheduledThreadPool

为什么不推荐使用内置线程池？

通过 `ThreadPoolExecutor` 构造函数的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

### 线程池处理任务流程

1. 如果当前运行的线程数小于核心线程数，那么就会新建一个线程来执行任务。
2. 如果当前运行的线程数等于或大于核心线程数，但是小于最大线程数，那么就把该任务放入到任务队列里等待执行。
3. 如果向任务队列投放任务失败（任务队列已经满了），但是当前运行的线程数是小于最大线程数的，就新建一个线程来执行任务。
4. 如果当前运行的线程数已经等同于最大线程数了，新建线程将会使当前运行的线程超出最大线程数，那么当前任务会被拒绝，拒绝策略会调用`RejectedExecutionHandler.rejectedExecution()`方法。

### FAQ

#### 线程池中线程异常后，销毁还是复用？

需要分两种情况：

- **使用`execute()`提交任务**：当任务通过`execute()`提交到线程池并在执行过程中抛出异常时，如果这个异常没有在任务内被捕获，那么该异常会导致当前线程终止，并且异常会被打印到控制台或日志文件中。线程池会检测到这种线程终止，并创建一个新线程来替换它，从而保持配置的线程数不变。
- **使用`submit()`提交任务**：对于通过`submit()`提交的任务，如果在任务执行中发生异常，这个异常不会直接打印出来。相反，异常会被封装在由`submit()`返回的`Future`对象中。当调用`Future.get()`方法时，可以捕获到一个`ExecutionException`。在这种情况下，线程不会因为异常而终止，它会继续存在于线程池中，准备执行后续的任务。

简单来说：使用`execute()`时，未捕获异常导致线程终止，线程池创建新线程替代；使用`submit()`时，异常被封装在`Future`中，线程继续复用。

这种设计允许`submit()`提供更灵活的错误处理机制，因为它允许调用者决定如何处理异常，而`execute()`则适用于那些不需要关注执行结果的场景。

#### 如何设置线程池大小？

线程池大小并不是越大越好，大了浪费资源，增大了上下文切换成本。

- **CPU 密集型任务(N+1)：** 这种任务消耗的主要是 CPU 资源，可以将线程数设置为 N（CPU 核心数）+1。比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。

  （**排序类**的等。需要用到CPU计算能力。）

- **I/O 密集型任务(2N)：** 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们可以多配置一些线程，具体的计算方法是 2N。

  （网络**读写**，文件读取等。）

#### 如何动态修改线程池的参数？

对线程池的核心参数实现自定义可配置。这三个核心参数是：

- **`corePoolSize` :** 核心线程数定义了最小可以同时运行的线程数量。
- **`maximumPoolSize` :** 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。
- **`workQueue`:** 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

#### 如何设计一个能够根据任务的优先级来执行的线程池？

使用PriorityBlockingQueue，优先级阻塞队列，并且传入其中的任务必须是具备排序能力的，方式有两种：

1. 提交到线程池的任务实现 `Comparable` 接口，并重写 `compareTo` 方法来指定任务之间的优先级比较规则。
2. 创建 `PriorityBlockingQueue` 时传入一个 `Comparator` 对象来指定任务之间的排序规则(推荐)。

存在风险：

- `PriorityBlockingQueue` 是无界的，可能堆积大量的请求，从而导致 OOM。

  重写`offer` 方法(入队)的逻辑，当插入的元素数量超过指定值就返回 false 。

- 可能会导致饥饿问题，即低优先级的任务长时间得不到执行。

  等待时间过长的任务会被移除并重新添加到队列中，但是优先级会被提升

- 由于需要对队列中的元素进行排序操作以及保证线程安全（并发控制采用的是可重入锁 `ReentrantLock`），因此会降低性能。

  需要排序，没办法避免

## Future 和Callable

### Future

 `java.util.concurrent` 包下的泛型接口，主要功能：

- **取消任务**；
- 判断任务是否被取消;
- 判断任务是否已经执行完成;
- **获取任务执行结果**。

### FutureTast

`FutureTask` 提供了 `Future` 接口的基本实现，常用来封装 `Callable` 和 `Runnable`。ExecutorService的submit()` 方法返回的其实就是 `Future` 的实现类 `FutureTask` 。

`FutureTask` 不光实现了 `Future`接口，还实现了`Runnable` 接口，因此可以作为任务直接被线程执行。

### CompletableFuture

`Future` 在实际使用过程中存在一些局限性，比如不支持异步任务的编排组合、获取计算结果的 `get()` 方法为阻塞调用。

- 一个任务需要依赖另外两个任务执行完之后再执行，怎么设计？

  这种任务编排场景非常适合通过`CompletableFuture`实现。通过 `CompletableFuture` 的 `allOf()` 这个静态方法来并行运行 T1 和 T2，当 T1 和 T2 都完成后，再执行 T3。

- 在使用 CompletableFuture 的时候为什么要自定义线程池？

  `CompletableFuture` 默认使用全局共享的 `ForkJoinPool.commonPool()` 作为执行器，所有未指定执行器的异步任务都会使用该线程池。这意味着应用程序、多个库或框架（如 Spring、第三方库）若都依赖 `CompletableFuture`，默认情况下它们都会共享同一个线程池。

  虽然 `ForkJoinPool` 效率很高，但当同时提交大量任务时，可能会导致资源竞争和线程饥饿，进而影响系统性能。

  为避免这些问题，建议为 `CompletableFuture` 提供自定义线程池，带来以下优势：

  - 隔离性：为不同任务分配独立的线程池，避免全局线程池资源争夺。
  - 资源控制：根据任务特性调整线程池大小和队列类型，优化性能表现。
  - 异常处理：通过自定义 `ThreadFactory` 更好地处理线程中的异常情况。

### Callable

## ThreadLocal

每个线程独有的本地变量。

Thread类里持有ThreadLocal.ThreadLocalMap类型的集合threadLocals，用于存放每个线程独有的变量。

ThreadLocalMap可以存储ThreadLocal为key，Object对象为value的键值对。

ThreadLocal内存泄漏怎么导致的？

`ThreadLocalMap` 的 `key` 和 `value` 引用机制：

- **key 是弱引用**：`ThreadLocalMap` 中的 key 是 `ThreadLocal` 的弱引用 (`WeakReference<ThreadLocal<?>>`)。 这意味着，如果 `ThreadLocal` 实例不再被任何强引用指向，垃圾回收器会在下次 GC 时回收该实例，导致 `ThreadLocalMap` 中对应的 key 变为 `null`。
- **value 是强引用**：即使 `key` 被 GC 回收，`value` 仍然被 `ThreadLocalMap.Entry` 强引用存在，无法被 GC 回收。

当 `ThreadLocal` 实例失去强引用后，其对应的 value 仍然存在于 `ThreadLocalMap` 中，因为 `Entry` 对象强引用了它。如果线程持续存活（例如线程池中的线程），`ThreadLocalMap` 也会一直存在，导致 key 为 `null` 的 entry 无法被垃圾回收，即会造成内存泄漏。

也就是说，内存泄漏的发生需要同时满足两个条件：

1. `ThreadLocal` 实例不再被强引用；
2. 线程持续存活，导致 `ThreadLocalMap` 长期存在。

虽然 `ThreadLocalMap` 在 `get()`, `set()` 和 `remove()` 操作时会尝试清理 key 为 null 的 entry，但这种清理机制是被动的，并不完全可靠。

### ThreadLocal如何保证了线程安全？

1. ThreadLocal是线程私有的变量，
2. get和set方法，都是先调用了Thread.currentThread()保证了当前线程。

总结就是ThreadLocal的存取数据，都是从当前线程的map来取的。

**如何避免内存泄漏的发生？**

1. 在使用完 `ThreadLocal` 后，务必调用 `remove()` 方法。 这是最安全和最推荐的做法。 `remove()` 方法会从 `ThreadLocalMap` 中显式地移除对应的 entry，彻底解决内存泄漏的风险。 即使将 `ThreadLocal` 定义为 `static final`，也强烈建议在每次使用后调用 `remove()`。
2. 在线程池等线程复用的场景下，使用 `try-finally` 块可以确保即使发生异常，`remove()` 方法也一定会被执行。

**如何跨线程传递 ThreadLocal 的值？**

为了解决这个问题，业界有两套主流的解决方案，一套是 JDK 原生的，另一套是阿里巴巴开源的。

1. `InheritableThreadLocal` ：JDK1.2 提供的一个类，继承自 `ThreadLocal` 。使用 `InheritableThreadLocal` 时，会在创建子线程时，令子线程继承父线程中的 `ThreadLocal` 值，但是无法支持线程池场景下的 `ThreadLocal` 值传递。
2. `TransmittableThreadLocal` ： `TransmittableThreadLocal` （简称 TTL） 是阿里巴巴开源的工具类，继承并加强了`InheritableThreadLocal`类，可以在线程池的场景下支持 `ThreadLocal` 值传递。项目地址：https://github.com/alibaba/transmittable-thread-local。

InheritableThreadLocal 原理

`InheritableThreadLocal` 实现了创建异步线程时，继承父线程 `ThreadLocal` 值的功能。该类是 JDK 团队提供的，通过改造 JDK 源码包中的 `Thread` 类来实现创建线程时，`ThreadLocal` 值的传递。

## AQS 

AbstractQueuedSynchronizer，抽象队列同步器。AQS 解决了开发者在实现同步器时的复杂性问题。它提供了一个通用框架，用于实现各种同步器，例如 **可重入锁**（`ReentrantLock`）、**信号量**（`Semaphore`）和 **倒计时器**（`CountDownLatch`）。通过封装底层的线程同步机制，AQS 将复杂的线程管理逻辑隐藏起来，使开发者只需专注于具体的同步逻辑。

简单来说，AQS 是一个抽象类，为同步器提供了通用的 **执行框架**。它定义了 **资源获取和释放的通用流程**，而具体的资源获取逻辑则由具体同步器通过重写模板方法来实现。 因此，可以将 AQS 看作是同步器的 **基础“底座”**，而同步器则是基于 AQS 实现的 **具体“应用”**。

AQS 的原理是什么？

...

## Semaphore

`Semaphore`(信号量)可以用来控制同时访问特定资源的线程数量。(`synchronized` 和 `ReentrantLock` 都是一次只允许一个线程访问某个资源)

`Semaphore` 通常用于那些资源有明确访问数量限制的场景比如限流（仅限于单机模式，实际项目中推荐使用 Redis +Lua 来做限流）。

`Semaphore` 有两种模式：。

- **公平模式：** 调用 `acquire()` 方法的顺序就是获取许可证的顺序，遵循 FIFO；

  ~~~java
  public Semaphore(int permits) {
      sync = new NonfairSync(permits);
  }
  ~~~

- **非公平模式：** 抢占式的。

  ~~~java
  public Semaphore(int permits, boolean fair) {
      sync = fair ? new FairSync(permits) : new NonfairSync(permits);
  }
  ~~~

Semaphore 的原理是什么？

`Semaphore` 是共享锁的一种实现，它默认构造 AQS 的 `state` 值为 `permits`，你可以将 `permits` 的值理解为许可证的数量，只有拿到许可证的线程才能执行。

调用`semaphore.acquire()` ，线程尝试获取许可证，如果 `state >= 0` 的话，则表示可以获取成功。如果获取成功的话，使用 CAS 操作去修改 `state` 的值 `state=state-1`。如果 `state<0` 的话，则表示许可证数量不足。此时会创建一个 Node 节点加入阻塞队列，挂起当前线程。

调用`semaphore.release();` ，线程尝试释放许可证，并使用 CAS 操作去修改 `state` 的值 `state=state+1`。释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修改 `state` 的值 `state=state-1` ，如果 `state>=0` 则获取令牌成功，否则重新进入阻塞队列，挂起线程。

## 锁

### 死锁

1. 互斥
2. 占有并等待
3. 循环等待
4. 不可抢占

如何检测死锁？

- 使用jmap,jstack等命令查看JVM 线程栈和堆内存的情况。如果有死锁，`jstack` 的输出中通常会有 `Found one Java-level deadlock:`的字样，后面会跟着死锁相关的线程信息。

### 锁分类

#### 乐观锁悲观锁

##### 悲观锁

总是假设最坏情况，每次获取临界资源会加锁，其他线程阻塞，用完再让给其他资源。

Synchronized,ReentrantLock等就是悲观锁。

##### 乐观锁

只是在提交的时候验证数据是否被修改。具体实现方式是**CAS算法**和**版本号机制**。

java.util.concurrent.atomic包下的原子类AtomicInteger，LongAdder等就是使用CAS来实现的。

###### 版本号机制

1. 数据增加version字段
2. 更新时，对比version，version不同重试，version相同更新成功版本号+1

案例：银行取钱。

###### CAS算法

Compare And Swap（比较与交换）

CAS涉及到三个操作数

1. V （Var,要更新的变量值） 

2. E （Expected, 预期值）

3. N （New, 拟写入的新值）

   当且仅当 V 的值等于 E 时，CAS 通过原子方式用新值 N 来更新 V 的值。如果不等，说明已经有其它线程更新了 V，则当前线程放弃更新。

   **举一个简单的例子**：线程 A 要修改变量 i 的值为 6，i 原值为 1（V = 1，E=1，N=6，假设不存在 ABA 问题）。

   1. i 与 1 进行比较，如果相等， 则说明没被其他线程修改，可以被设置为 6 。
   2. i 与 1 进行比较，如果不相等，则说明被其他线程修改，当前线程放弃更新，CAS 操作失败。

   当多个线程同时使用 CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。

   Java 语言并没有直接实现 CAS，CAS 相关的实现是通过 C++ 内联汇编的形式实现的（JNI 调用）。因此， CAS 的具体实现和操作系统以及 CPU 都有关系。

   `sun.misc`包下的`Unsafe`类提供了`compareAndSwapObject`、`compareAndSwapInt`、`compareAndSwapLong`方法来实现的对`Object`、`int`、`long`类型的 CAS 操作

- Java如何实现CAS？

在 Java 中，实现 CAS（Compare-And-Swap, 比较并交换）操作的一个关键类是`Unsafe`。

`Unsafe`类位于`sun.misc`包下，是一个提供低级别、不安全操作的类。由于其强大的功能和潜在的危险性，它通常用于 JVM 内部或一些需要极高性能和底层访问的库中，而不推荐普通开发者在应用程序中使用。关于 `Unsafe`类的详细介绍，可以阅读这篇文章：📌[Java 魔法类 Unsafe 详解](https://javaguide.cn/java/basis/unsafe.html)。

`Unsafe`类中的 CAS 方法是`native`方法。`native`关键字表明这些方法是用本地代码（通常是 C 或 C++）实现的，而不是用 Java 实现的。这些方法直接调用底层的硬件指令来实现原子操作。也就是说，Java 语言并没有直接用 Java 实现 CAS，而是通过 C++ 内联汇编的形式实现的（通过 JNI 调用）。因此，CAS 的具体实现与操作系统以及 CPU 密切相关。

`AtomicInteger`是 Java 的原子类之一，主要用于对 `int` 类型的变量进行原子操作，它利用`Unsafe`类提供的低级别原子操作方法实现无锁的线程安全性。

- CAS 算法存在哪些问题？

  1. ABA问题

     被多次修改最后的值同原值，

     解决思路是在变量前面追加上**版本号或者时间戳**。

  2. 循环时间长开销大

     CAS 经常会用到自旋操作来进行重试，也就是不成功就一直循环执行直到成功。如果长时间不成功，会给 CPU 带来非常大的执行开销。如果 JVM 能够支持处理器提供的`pause`指令，那么自旋操作的效率将有所提升。

  3. 只能保证一个共享变量的原子操作

     CAS只支持单个共享变量原子操作，不过从JDK1.5开始，可以通过将多个变量封装在一个对象中，我们可以使用`AtomicReference`来执行 CAS 操作。

##### 对比

1. 悲观锁适合多写场景，避免频繁失败和重试，悲观锁的开销是固定的。不过，如果乐观锁解决了频繁失败和重试这个问题的话（比如`LongAdder`），也是可以考虑使用乐观锁的，要视实际情况而定。
2. 乐观锁通常多用于写比较少的情况（多读场景，竞争较少），这样可以避免频繁加锁影响性能。不过，乐观锁主要针对的对象是单个共享变量（参考`java.util.concurrent.atomic`包下面的原子变量类）。

#### 公平锁非公平锁

- 公平锁

  先申请的线程先得到锁，保证了顺序，性能查差一些

- 非公平锁

  后申请的线程可能先得到锁，随机或者按照其他优先级排序，性能更好但是可能会导致某些线程永远无法获得锁。

#### 可重入锁（递归锁）和非可重入锁

可重锁线程可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果是不可重入锁的话，就会造成死锁。

synchronized和ReentrantLock都是可重入锁。JDK 提供的所有现成的 `Lock` 实现类，包括 `synchronized` 关键字锁都是可重入的。

~~~Java
public class SynchronizedDemo {
    public synchronized void method1() {
        System.out.println("方法1");
        method2();
    }

    public synchronized void method2() {
        System.out.println("方法2");
    }
}
由于 synchronized锁是可重入的，同一个线程在调用method1() 时可以直接获得当前对象的锁，执行 method2() 的时候可以再次获取这个对象的锁，不会产生死锁问题。假如synchronized是不可重入锁的话，由于该对象的锁已被当前线程所持有且无法释放，这就导致线程在执行 method2()时获取锁失败，会出现死锁问题。
~~~

#### 共享锁独占锁

一把锁是否可以被多个线程同时获得。

读锁是共享锁，写锁是独占锁。

线程持有读锁还能获取写锁吗？

- 线程持有读锁时不能获取写锁
- 线程持有写锁时可以继续获取读锁。

读锁为什么不能升级成写锁？

- 读锁不能升级成写锁

  写锁是独占锁，升级成写锁会引起线程间的争夺，而且可能会导致死锁。

- 写锁可以降级成读锁

#### 可中断锁和不可中断锁

它们的区别在于：**线程在获取锁的过程中被阻塞时，是否能够因为中断而提前放弃等待。**

- `synchronized`，`ReentrantLock#lock()`  属于典型的不可中断锁。
- `ReentrantLock#lockInterruptibly()` ，`ReentrantLock#tryLock(long time, TimeUnit unit)` （带超时的尝试获取）也是可中断的。

#### 无锁，偏向锁，轻量级锁，重量级锁

- 

## synchronized

- 早期属于重量级锁，效率低。

  这是因为监视器锁（monitor）是依赖于底层的操作系统的 `Mutex Lock` 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

- Java6之后，`synchronized` 引入了大量的优化如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多。因此， `synchronized` 还是可以在实际项目中使用的，像 JDK 源码、很多开源框架都大量使用了 `synchronized` 。

  关于偏向锁多补充一点：由于偏向锁增加了 JVM 的复杂性，同时也并没有为所有应用都带来性能提升。因此，在 JDK15 中，偏向锁被默认关闭（仍然可以使用 `-XX:+UseBiasedLocking` 启用偏向锁），在 JDK18 中，偏向锁已经被彻底废弃（无法通过命令行打开）。

#### 使用方法

##### 1.修饰实例方法

锁了当前对象实例，进入代码块前需要获取当前对象实例的锁

~~~Java
synchronized void method(){...}
~~~

##### 2.修饰静态方法

锁了改类所有对象的实例，进入同步代码前要获得 **当前 class 的锁**。

~~~Java
synchroized static void method(){...}
~~~

##### 3.修饰代码块

- synchroized(object) 锁某个对象
- synchronized(类.class) 锁整个类

主要区别就是锁一个类的某个实例还是所有实例。

- 构造方法可以用synchronized方法修饰吗？

  不能，但是内部可以使用synchronized代码块，另外，构造方法本身是线程安全的，但如果在构造方法中涉及到共享资源的操作，就需要采取适当的同步措施来保证整个构造过程的线程安全。

#### 底层原理

##### Monitor 对象监视器

Monitor基于C++实现，每个对象都内置了一个ObjectMonitor对象，另外，`wait/notify`等方法也依赖于`monitor`对象，这就是为什么只有在同步的块或者方法中才能调用`wait/notify`等方法，否则会抛出`java.lang.IllegalMonitorStateException`的异常的原因。

##### synchronized修饰代码块实现

`synchronized` 同步语句块的实现使用的是 **monitorenter**和 **monitorexit** 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。

当执行monitorenter指令时，线程会尝试获取对象的锁，也就是monitor的持有权，如果锁的计数为0则可以获取，对象锁的拥有者线程，才可以执行monitorexit指令来释放锁。

一个monitorenter指令可能对应两个monitorexit指令，确保异常状态也能释放锁。

如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

##### synchronized修饰方法实现

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取而代之的是 **ACC_SYNCHRONIZED** 标识，该标识指明了该方法是一个同步方法。JVM 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

如果是实例方法，JVM 会尝试获取实例对象的锁。如果是静态方法，JVM 会尝试获取当前 class 的锁。

**不过，两者的本质都是对对象监视器 monitor 的获取。**

#### JDK1.6后底层做了哪些优化？

引入自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

- 偏向锁为什么被废弃了？

  JDK15默认关闭，JDK18废弃。

  - **性能收益不明显：**

  偏向锁是 HotSpot 虚拟机的一项优化技术，可以提升单线程对同步代码块的访问性能。

  受益于偏向锁的应用程序通常使用了早期的 Java 集合 API，例如 HashTable、Vector，在这些集合类中通过 synchronized 来控制同步，这样在单线程频繁访问时，通过偏向锁会减少同步开销。

  随着 JDK 的发展，出现了 ConcurrentHashMap 高性能的集合类，在集合类内部进行了许多性能优化，此时偏向锁带来的性能收益就不明显了。

  偏向锁仅仅在单线程访问同步代码块的场景中可以获得性能收益。

  如果存在多线程竞争，就需要 **撤销偏向锁** ，这个操作的性能开销是比较昂贵的。偏向锁的撤销需要等待进入到全局安全点（safe point），该状态下所有线程都是暂停的，此时去检查线程状态并进行偏向锁的撤销。

  - **JVM 内部代码维护成本太高：**

  偏向锁将许多复杂代码引入到同步子系统，并且对其他的 HotSpot 组件也具有侵入性。这种复杂性为理解代码、系统重构带来了困难，因此， OpenJDK 官方希望禁用、废弃并删除偏向锁。

## volatile

### volatile作用

1. 保证变量可见性

   CPU都是有行缓存的，volatile能让行缓存无效，因此每个线程都能读到内存中最新的值。

2. 保证赋值操作原子性

   boolean，int的赋值本身就是原子性的。但是long和double则不一定，如果是32位的处理器，对于64位的变量的操作可能会被分解成为二个步骤：高32位和低32位，由此可能会发生线程切换，从而导致线程不安全。如果变量声明为volatile，那么虚拟机会保证赋值是原子的，是不可被打断的。

3. 禁止指令重排

   正常情况下，虚拟机会对指令进行重排，当然是在不影响程序结果的正确性的前提下。volatile能够在一定程度上禁止虚拟机进行指令重排。还有就是对于volatile变量的写操作，保证是在读操作之前完成。

   ~~~Java
   private volatile boolean flag;
   	
   public void setFlag(boolean flag) {
   	this.flag = flag;
   }
   	
   public void getFlag() {
   	return flag;
   }
   
   假设线程A来调用setFlag(true)，线程B同时来调用getFlag，对于一般的变量，是无法保证B能读到A设置的值的，因为它们执行的顺序是未知的。但是像上面，加上volatile修饰以后，虚拟机会保证，线程A的写操作在线程B的读操作之前完成，换句话，B能读到最新的值。当然了，用锁机制也能达到同样的效果，比如在方法前面都加上synchronized关键字，但是性能会远不如使用volatile。
   ~~~

### volatitle和synchronized区别

- volatile轻量级，性能更好
- volatile只能用于变量，只保证可见性，不保证原子性，synchronized两者都保证。
- volatile主要用于解决变量在多个线程之间的可见性，synchronized解决多个线程访问资源的同步性。

### volatile的典型使用场景

- 读操作，多于写操作
- 写操作，不依赖于变量的当前值，也就是说要是纯赋值操作
- 只需要读取的值，不需要等待某一特定的值

1. 多线程情况下的标志位。

   比如，有一个检查新版本的按扭，点击时会发起去检查新版本，因为检查新版本涉及网络请求，可能会比较耗时，所以需要放在单独的线程中去做。为了避免多次同时触发检查请求，做一个限制：上一个请求没有完成时，再次点击无效。这时就可以用volatile来做个标志位。

2. CAS无锁同步的变量声明

### volatile的实现机制

volatile是靠内存屏障和[MESI](https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FMESI_protocol)（缓存一致性协议）来达成的它的作用的。[内存屏障](https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FMemory_barrier)(Memory Barriers)是处理器提供的一组内存操作指令，它的作用是限制内存操作的顺序，也就是说内存屏障像一个栅栏一样，它前面的指令要在它后面的指令之前完成；还能强制把缓存写入到主存；再有的就是触发缓存一致性，就是当有写变量时，会把其他CPU核心的缓存变为无效。

## ReentrantLock

实现了Lock接口，可重入，独占，与synchronized类似，但是更强大和灵活。增加了轮询，超时，中断，公平锁和非公平锁等高级功能。

ReentrantLock和synchronized区别

- 两者都是可重入锁

- synchronized依赖JVM，ReentrantLock是JDK层面实现的（也就是 API 层面，需要 `lock()` 和 `unlock()` 方法配合 `try/finally` 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

- ReentrantLock功能更强大和灵活

  增加了轮询，超时，中断，公平锁和非公平锁等高级功能。

## Atomic原子类

## Java内存屏障

内存屏障（Memory Barrier）是确保数据可见性和指令执行顺序的核心机制。它通过强制编译器和CPU遵循特定的规则，解决了以下**可见性和有序性**问题

https://juejin.cn/post/7536944463635087370

# 泛型Generics

参考[kotlin泛型](Kotlin.md#泛型)

## 本质

**参数化类型**

## 好处

1. 确保类型安全
2. 代码复用

## 用法

- 泛型类
- 泛型接口
- 泛型方法

## 界限通配符

Bounded Wildcards。

- 上界协变 Covariance  

  <? exntends T>

- 下界逆变 Contravariance

  <? super T>

- 无界

## 泛型擦除 Type Erasure

Java1.5引入了泛型，为了保持向后兼容，编译器会对泛型的类型信息进行擦除，使其变成常规的对象。

泛型擦除带来的限制。

1. 泛型不能使用类型判断符instanceo，
2. 也不能强制类型转换。
3. 反射不能用泛型（反射是在运行时，这时候已经泛型擦除了。

kotlin中可以使用reified关键字来保留泛型的类型。

## Java泛型的限制

- 不支持基础类

  泛型的类型参数必须是对象。

- 不支持数组

  不能用泛型声明数组，比如 List<String>[ ] ，

  （数组可以当做泛型的类型参数 List<int[]> ，因为数组也是个类型 ）



# 注解 Annotation

# 序列化反序列化

- **序列化**：将数据结构或对象转换成可以存储或传输的形式，通常是二进制字节流，也可以是 JSON, XML 等文本格式
- **反序列化**：将在序列化过程中所生成的数据转换为原始数据结构或者对象的过程

# I/O

# 反射

https://javaguide.cn/java/basis/reflection.html

**在程序运行时，动态地获取类的信息并操作类或对象（方法、属性）的能力**

### 反射有什么优缺点？



**优点：**

1. **灵活性和动态性**：反射允许程序在运行时动态地加载类、创建对象、调用方法和访问字段。这样可以根据实际需求（如配置文件、用户输入、注解等）动态地适应和扩展程序的行为，显著提高了系统的灵活性和适应性。
2. **框架开发的基础**：许多现代 Java 框架（如 Spring、Hibernate、MyBatis）都大量使用反射来实现依赖注入（DI）、面向切面编程（AOP）、对象关系映射（ORM）、注解处理等核心功能。反射是实现这些“魔法”功能不可或缺的基础工具。
3. **解耦合和通用性**：通过反射，可以编写更通用、可重用和高度解耦的代码，降低模块之间的依赖。例如，可以通过反射实现通用的对象拷贝、序列化、Bean 工具等。

**缺点：**

1. **性能开销**：反射操作通常比直接代码调用要慢。因为涉及到动态类型解析、方法查找以及 JIT 编译器的优化受限等因素。不过，对于大多数框架场景，这种性能损耗通常是可以接受的，或者框架本身会做一些缓存优化。
2. **安全性问题**：反射可以绕过 Java 语言的访问控制机制（如访问 `private` 字段和方法），破坏了封装性，可能导致数据泄露或程序被恶意篡改。此外，还可以绕过泛型检查，带来类型安全隐患。
3. **代码可读性和维护性**：过度使用反射会使代码变得复杂、难以理解和调试。错误通常在运行时才会暴露，不像编译期错误那样容易发现。

# 代理

## 动态代理

## 静态代理

# Java 异常机制

## Exception 和 Error 有什么区别？

- **`Exception`** :程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 Checked Exception (受检查异常，必须处理) 和 Unchecked Exception (不受检查异常，可以不处理)。
- **`Error`** ：`Error` 属于程序无法处理的错误 ，~~我们没办法通过 `catch` 来进行捕获~~不建议通过`catch`捕获 。例如 Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

## Checked Exception 和 Unchecked Exception 有什么区别？

- **Checked Exception** 

  即 受检查异常 ，Java 代码在编译过程中，如果受检查异常没有被 `catch`或者`throws` 关键字处理的话，就没办法通过编译。

- **Unchecked Exception** 

   **不受检查异常** ，编译能通过。

  `RuntimeException` 及其子类都统称为非受检查异常。比如NullPointerException，IllegalArgumentException等。

## finally代码块中的代码一定会被执行吗？

1. 程序所在的线程死亡。
2. 关闭 CPU。
3. 虚拟机被终止运行。

