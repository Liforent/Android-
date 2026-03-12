# 编码规范和编码性能

- 尾部逗号  

 Kotlin鼓励使用尾部逗号.

- 注释

  `@param` 与 `@return`注释.  尽量不使用,而是将参数与返回值的描述直接合并到文档注释中,并在提到参数的任何地方加上参数链接。 只有当需要不适合放进主文本流程的冗长描述时才应使用 `@param` 与 `@return`。

- 优先使用不可变集合类型来作为参数和返回值.

- 循环

  优先使用高阶函数,比如filter,map,而不是循环.

- 使用any来取代 map + contains (any会在触发之后立即return)

- 区间上循环

  优先使用 ..<  ,而不是 ..n-1

# Kotlin基础

## 基本类型

### 大小(bit)

Byte,8,-128~127, UByte,8,0~255 3位数,

Short,16 -32768, UShort,16,0~65535 ,5位数

Int,32,-2^31 (-2,147,483,648) 21亿,10位数

Long,64 19位数

Double,64

Flaot,32

### 无符号整形

Kotlin还提供了无符号整型 UInt,大小和范围如上.

除了无符号整型,还有无符号数组与区间等.使用场景比如一些颜色等资源.

```Kotlin
val b: UByte = 1u
```

### 不支持隐式转换,需要显示.

### 不支持8进制.

### 装箱

Java里int是未装箱的,Integer是装箱的,Kotlin里边Int是否装箱要看情况.满足一下条件就不装箱

1. 不可用
2. IntArray,FloatArray之类的.

### JVM对-128到127的整数Interger应用了内存优化.

```Kotlin
val a: Int = 100
val boxedA: Int? = a
val boxedA2: Int? = a
boxedA === boxedA2 // true, 127以内,a的所有可用引用实际上引用的都是统一对象.
如果a = 1000, 那么 boxedA === boxedA2 为false.  (boxedA == cboxedA2 一直为true.)
```

### 位运算

- `shl(bits)` – 有符号左移
- `shr(bits)` – 有符号右移
- `ushr(bits)` – 无符号右移
- `and(bits)` – 位**与**
- `or(bits)` – 位**或**
- `xor(bits)` – 位**异或**
- `inv()` – 位非

### 转义字符

- \n 换行LF
- \r 回车CR 
- \\\\ 反斜杠
- \\`, \`\`` 

### 字符串

可以for循环遍历元素,可以直接用索引访问

### 相等性

- 结构相等  =\=  equals()  
- 引用相等  \=\=\=
- NaN等于其自身,并且比任何数都大
- -0.0 !\= 0.0
- 数组比较 使用contentEquals()

### 数组

- 数组和装箱

- 数组和集合

  优先使用集合.

  1. 集合可以是只读的.
  2. 集合容易增删.(数组大小固定,增删需要创建新数组,效率低.)
  3. 可以用==操作符来检验两个集合在结构上是否相等,但是数组不能.
  4. 数组直接打印是对象实例,而集合打印是集合的内容.

- 比较数组

  不能用==直接比较,而是用.contentEquals()和.contentDeepEquals()来比较.

- 数组转换成集合

  .toList(),toSet(),toMap()

  only an array of Pair<K,V> can be convert to a Map.

  ```kotlin
  val pariArray = arrayOf("apple" to 1, "banana" to 20, "cherry" to 300)
  ```

### 返回与跳转

Kotlin有3种结构化跳转表达式:

return,break,continue,他们都可以作为更大的表达式的一部分,这些表达式的类型都是Nothing类型.

### break与continue标签

可以用标签限定break和continue   格式为 标识符+@  

```kotlin
loop@ for (i in 0..100){
  if (...) break@loop 
}
```

### return标签

- return默认结束的是fun标记的函数,除非带@,结束到指定的位置.
- Lambda表达式里,默认不支持空的return,除非是inline的.

### Nothing类型

这个类型没有值,是用于标记永远不能到达的位置.

throw,return,break,continue等都是Nothing的子类.

```kotlin
/**
 * Nothing has no instances. You can use Nothing to represent "a value that never exists": for example,
 * if a function has the return type of Nothing, it means that it never returns (always throws an exception).
 */
public class Nothing private constructor()
```

### 类

- 构造函数
  - 如果主构造函数没有任何注解或者可见性修饰符,可以省略constructor关键字.
  - 每个次构造函数都需要直接或者间接的委托给主构造函数.(this委托本类或者super委托父类)
  - 初始化块的代码实际上会成为主构造函数的一部分,它的执行早于次构造函数.
    - java中也有初始化代码块,但是前边没有init关键字,并且优先于构造函数执行.
  - 如果一个非抽象类没有声明任何主次构造函数,它会生成一个不带参数的主构造函数.
  - 在JVM上,如果主构造函数的所有参数都有默认值,编译器会生成一个额外的无参构造函数,它将使用默认值.

- 继承

- Any是所有类的父类.

```kotlin
public open class Any {
  public open operator fun equals(other: Any?): Boolean
  public open fun hashCode(): Int
  public open fun toString(): String
}
```

- 重写

  子类可以重写父类的方法或属性,且必须显示的声明override,子类与父类不可以有同名的属性或方法,只能重写.

  var 可以覆盖 val,反之不行.

  Java里的override是注解的形式,而Kotlin的override是关键字.

- 子类初始化顺序

  子类初始化时要先完成父类初始化.(因此在init代码块里不要引用open的属性,可能子类未完成初始化导致程序逻辑异常.)

- 类和接口多继承时冲突处理

  class A:B,C  如果B和C都有抽象方法d(),那么A必须实现d(),并且super<B>.d()这种方式指明实现逻辑.

### 属性

- 任何时候引用变量相当于是调用了getter方法,任何时候对变量赋值,相当于是调用了setter方法,因此在getter和setter里不能操作属性自身,因此有了属性BackingFiled.

- kotlin变量没有默认值,必须初始化或者延迟初始化

- 基本数据类型不能lateinit,String可以

- backing field

  `field` 标识符只能用在属性的访问器内。field不能直接声明,仅作为属性的一部分在内存中保存其值时使用.

- JVM优化了默认getter和setter的调用减少了函数调用开销.

- 编译器常量

  - const修饰
  - objcet或顶层
  - 必须以String或原生类型值初始化
  - 不能有自定义getter

- .isInitialized()可以检测lateinit var是否初始化了.

### 接口

- 接口的方法默认都是open的,可以有方法体或没有
- 接口的属性要么提供访问器的实现,要么是抽象的,需要在实现类里重写.

### 函数式接口(SAM)

fun interface 

函数式接口可以有多个非抽象成员(方法),但只能有一个抽象方法.函数式接口可以通过Lambda表达式实现SAM转换,从而使代码更简洁易读.

- Java接口上的SAM转换

  ```Kotlin
  //此处setOnClickListener,可以直接使用Lambda表达式,是因为OnClickListener是Java代码编写的接口,且只有一个抽象方法,这种场景也可以直接使用Lambda表达式省去object
  textView.setOnClickListener {
    
  }
  ```

### 可见性修饰符

- public

  与Java基本一致,默认不写就是public,Java默认不写是default

- internal

  module内可见.kotlin移除了Java的default,多了internal的概念.

  一个module是编译在一起的一套Kotlin文件,例如

  	- 一个IntelliJ IDEA模块
  	- 一个Maven项目
  	- 一个Gradle源代码集
  	- 一次<kotlinc> Ant任务执行所编译的一套文件.

- protected

  子类可见,不适用于顶层声明.Java是子类和包内可见,包外不可见.

- private

  作为内部类时,对外部类不可见. 而Java对外部可见???

- protected和internal成员被重写之后,可见性不变,除非指定了可见性.

- 局部声明不能有可见性修饰符,包括变量,函数,和类.

### 扩展

扩展成员和扩展函数,伴生对象的扩展原理相似,都是编译时生成,没实际改变类结构,扩展函数与成员函数同名时失效.

一般扩展函数都在顶层定义.

扩展接收者与分发接收者的成员名字冲突时,扩展接收者优先.

```Kotlin
class Connection {
  fun Host.getConnectionString() {
    toString()//扩展接收者优先
    this@Connection.toString() //分发接收者需要用限定的this语法
  }
}
```

### 数据类

- 主构造函数至少有一个参数
- 数据类不能是抽象,开放,密封或者内部的
- 数据类会根据构造方法的参数自动生成equals,toString,copy,componentN方法,如果需要排除某些属性,需要把属性声明到类体里而不是构造方法里.
- copy
- 解构
- 标准数据类 Pair,Triple

### 抽象类

Kotlin里的抽象类可以没有抽象方法. 为什么这么设计?

### 密封类和密封接口

### 嵌套类和内部类

- 类和接口都可嵌套

- 内部类默认持有外部类的引用,inner class.

  外部类的所有成员包括private,都能被内部类访问.

- 外部类和内部类成员同名时, 需要用this@Outer来消除歧义

### 匿名内部类

### 枚举类

- 每个枚举常量都是一个对象.
- 枚举类可以实现接口,但是不能继承类
- 默认实现了 companion object{},因此访问枚举常量的时候无需实例化.

### 内联类

// for JVM backends

@JvmInline

value class Password(private val s: String)

- 内联类必须含有唯一的一个属性,在构造函数中初始化. 

### 对象表达式和对象声明(object)

- companion object
  - 伴生对象是外部类的实例成员,而非静态成员.使用@JvmStatic注解可以让伴生对象的成员成为真正的静态方法和成员.
  - 伴生对象可以实现接口
  - 一个类只能有一个伴生对象,伴生对象的名字可以省略.



# 高阶函数

## 作用域函数

- also
- let
- apply
- with
- run

区别是参数和返回值。



### 函数类型,接收者,函数字面量

(a:Int,b:Int) -> String    kotlin使用类似(a:Int,b:Int) -> String的函数类型来处理函数的声明

- 带接收者的函数类型

  T.(A) -> String

- 挂起函数

  suspend T.(A) -> Unit

- 可空

  block: ((Int,Int) -> Unti)?

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T {
  
}
apply是个泛型方法,参数是一个带接收者的函数类型.
```

可以通过类型别名给函数类型起一个别称

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

### 函数类型实例化

- 函数字面值代码块

  - 匿名函数
  - Lambda表达式
  - 带接收者的函数字面值

- 使用已有声明的可调用引用

  - ::toString (各种双冒号函数)

- 实现了函数类型接口的自定义类的实例

  ```kotlin
  class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
  }
  val intFunction: (Int) -> Int = IntTransformer()
  ```

  居然还可以这么用.不过想来也是,这个类实现了一个函数类型的接口,那么实例化的时候返回的不就是函数类型的对象?

带与不带接收者的函数类型*非字面*值可以互换，其中接收者可以替代第一个参数.

### Lambda表达式与匿名函数

这两个都是函数字面值,函数字面值也就是==没有声明,而是立即作为表达式传递的函数==.

- Lambda返回值不写return,默认就是根据最后一行自己推断出返回值类型.
- Lambda表达式,不能裸写return(除非这个函数是inline的), kotlin中return默认是结束外层的fun函数.

匿名函数,双冒号函数(FunctionReference),Lambda表达式的本质:

​	本质上都是函数类型的对象,能次能赋值给变量,也能当做参数来传递.

这些也被称为函数字面值(function literal)

- Lambda表达式语法缺少指定函数返回类型的能力(虽然一般来说没必要指定,因为可以自动推断出来),如果需要显示指明返回类型,则可以用匿名函数.

- 匿名函数就是常规的函数缺少了名称.

- 匿名函数和Lambda表达式的另一个区别就是返回

  在kotlin中,一个不带标签的return语句,总是在用fun关键字声明的函数中返回的,也就是Lambda表达式中的return,将从包含它的函数返回,而匿名函数的return会从自身返回.

### 带接收者的函数字面值

- 带接收者的函数类型: A.(B) -> C

- 带接收者的函数字面值:

  - Lambda 

    val sum: Int.(Int) -> Int = { other -> plus(other) }

  - 匿名函数 

    val sum: fun Int.(Int): Int = this + other

在带接收者的函数字面值里,可直接访问接收者的对象成员,或者使用this限定符.

### 内联函数

高阶函数会带来一些运行时效率的损失(每个函数都会生成对象,并且会捕获一个闭包.)

- inline

  inline修饰符会影响函数本身和传递给它的Lambda表达式,所有这些都将会内联到调用处.

  内联可能会导致生成的代码增加,需要避免内联过大函数.

  Lambda表达式默认不能裸return,除非这个Lambda表达式被内联了.

  - inline关键字好处

    1. 减少高阶函数创建函数类型对象的消耗

    2. 函数里直接调用Java的静态方法

       ```Kotlin
       // MathJVM.kt中
       public actual inline fun min(a:Int,b:Int):Int = nativeMath.min(a,b)
       
       //Kotlin中调用
       min(1,3)
       ```

- noinline

  取消某个函数类型的参数内联,函数被内联之后,会无法当做返回值返回.

- crossinline

- 具体化的类型参数

  内联函数还支持具体化的类型参数,需要加上reified关键字 (reify,使具体化)

  ```kotlin
  类型作为参数传递时写法
  treeNode.findParentOfType(MyTreeNode::class.java)
  如果findParentOfType改为内联的reified函数,则可以按照以下方式调用
  treeNode.findParentOfType(MyTreeNode)
  ```

### 操作



### 高阶函数 Higer-Order Function

函数作为参数或者返回值的函数.

- let

  (T)->Unit

  block(this)

- also

  (T)->Unit

  return this

- run

  T.()->Unit

  block()

- with

- apply

  T.()->Unit

  return this

# 反射 Reflection

### 概述

Java反射允许程序在运行时：

1. 获取类的完整结构信息（类名，字段，方法，注解等。）

2. 动态创建类的实例对象。

3. 动态调用方法和访问字段。

4. 修改访问权限（即使是private）

Kotlin的反射是Java反射的加强版

Kotlin反射提供对属性和可控类型的访问权限.

Java和Kotlin使用对比

- Java获取Class对象方式

  ```Kotlin
  - Class<?> clazz = Class.forName("xxx.xxx.MyClass");
  - Class<?> clazz = MyClass.class;
  - MyClass obj = new MyClass();
  	Class<?> clazz = obj.getClass();
  ```

- Kotlin获取Class对象方式

  ```kotin
  val clazz = MyClass::class.java //.java是KClass的扩展属性
  ```

### 核心API

### Android中反射的限制

1. 性能问题。反射调用比正常调用慢很多倍。

   - 动态解析与验证

     反射调用需要查找类，验证方法、字段信息，检查调用者是否有权限，检查参数类型是否匹配等，而直接调用的时候，编译器已经将类的信息保存过了无需上述操作。

   - 编译器优化失效

     内联优化等失效

   - 方法访问器的生成与使用

     为了执行反射方法，JVM/CLR内部需要生成或使用一个“方法访问器”（如 `MethodAccessor`）

   - 参数的装箱拆箱

   - 安全检查

     每次反射操作都可能触发安全管理器的检查，以确保代码有足够的权限。直接调用通常不需要这些运行时检查

2. 兼容性问题。

   - 不同Android版本对反射的支持可能不同。
   - 不同厂家的Android系统API可能不同。

3. ProGuard/R8优化限制

   ```java
   // 反射调用的方法/字段可能被混淆或移除
   // 需要在proguard-rules.pro中配置保留规则
   -keep class com.example.MyClass { *; }
   -keepclassmembers class com.example.MyClass {
       public void myMethod();
   }
   ```

4. API限制.Android P（28+）无法通过反射访问非公开API，会抛NoSuchMethodError或NoSuchFieldError。Android R (30+)做了更严格的限制。

   ```java
   // 无法通过反射访问非公开SDK API（灰名单/黑名单）
   // 调用会抛出NoSuchMethodError或NoSuchFieldError
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
       // 需要特别处理或寻找替代方案
   }
   ```

5. 如何解除和绕过上述限制？

   1. 

### 使用场景和建议

- ✅ 适合：插件化框架、热修复、序列化框架、测试框架

- ❌ 避免：高频调用的业务逻辑、UI更新代码

- ⚠️ 谨慎：系统API调用、跨版本兼容性要求高的代码

### Kotlin的反射与Java的反射有什么不同？

# Coroutines

## 协程介绍

协程是Android上异步编程的推荐方案.主要是用作==线程切换==和==耗时任务==

1. 协程==是一种并发设计模式==.

2. coroutine可以用==同步的方式去写异步代码==(阻塞式的方式写出非阻塞式的代码),解决并发中的回调地狱问题.(在这之前写异步代码主要是靠回调)

3. coroutine提供了一种避免阻塞线程并用更简单的,更可控的操作代替线程阻塞的方法:协程挂起和恢复.==协程挂起时不会阻塞线程,几乎是无代价的==

   ```Kotlin
   1. 协程的挂起和恢复不会阻塞线程,相当于是在线程里按需执行某个挂起函数的代码,这一切都是在用户态处理的,无需进入内核态.
   2. 协程的withContext方法在切换线程的时候,使用的线程池.
   3. Kotlin使用的是无栈协程,挂起时协程的栈帧不会存到内存中(线程栈是有大小限制的),而是将状态存到堆中,相比有栈内存,这进一步降低了开销.
   ```

4. coroutine更像是一种轻量级的线程,==基于线程池API==,一个线程中可以创建N个coroutine(测试了,一下,Coroutine里的线程id会变)

5. coroutine==内存泄露更少==(结构化并发机制在一个作用域内支持多项操作),==支持取消,轻量高效==,有JetPack库支持.

## CoroutineStart

协程启动模式

1. DEFAULT

   饿汉式,创建后就开始调度.(但不是立即执行,可能在执行前被取消)

   ~~~kotlin
   val job = GlobalScope.launch{
     ...
   }
   //job开始调度了,但不是立即执行的,可能已经被取消
   job.cancel()
   ~~~

2. LAZY

   懒汉,启动后不会调度,直到需要执行的时候才会调度.(也就是主动调用job的start,join,wait等方法才会开始调度.)

3. ATOMIC

   同DEFAULT,也是立即调度,但是区别是通过`ATOMIC`模式启动的协程执行到第一个挂起点之前是不响应`cancel `取消操作的，`ATOMIC`一定要涉及到协程挂起后`cancel `取消操作的时候才有意义。

4. UNDISPATCHED

   协程在这种模式下会直接开始在当前线程下执行，直到运行到第一个挂起点。这听起来有点像 `ATOMIC`，不同之处在于`UNDISPATCHED`是不经过任何调度器就开始执行的。当然遇到挂起点之后的执行，将取决于挂起点本身的逻辑和协程上下文中的调度器。

- withContext和launch有什么区别？

  ~~~kotlin
  suspend fun test(){
    launch(Dispatch.IO){
      delay(100)
      Log.d("1 launch")
    }
    withContext(Dispatch.IO){
      delay(50)
      Log.d("2 withContext")
    }
    Log.d("3 test")
  }
  //思考一下打印顺序
  正确顺序是：  2->3->1
  也就是launch跟主流程是异步启动的，而withContext跟主流程是同步启动的。这就是主要区别，使用时需要注意。如果是需要等待代码执行结果的，可以使用withContext来切换线程，如果不依赖执行结果，而是需要并发处理的，可以使用launch来开启一个协程。
  ~~~

  - ViewModel里的viewModelScope.launch()是在哪个线程启动的？可以执行耗时操作吗？会不会阻塞？

  ~~~kotlin
  scope =  CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
  也就是主线程调用的，因此不能执行耗时操作。
  ~~~

  - 协程里可以直接使用ThreadLocal吗？

    ~~~kotlin
    ThreadLocal是线程的私有变量，因此不能直接在协程中使用，会有同步问题。可以使用asContextElement后来使用。
    协程中的局部变量机制是CoroutineContext.
    ~~~

## CoroutineDipatcher

协程调度器,withContext()切换调度器

- Default

  默认,简单的计算任务或者时间短的任务比如Json解析,计算任务

- Main

  主线程,可以UI操作

- IO

  适合IO操作,比如网络,数据库,文件等.

- Unconfined

  非受限调度器

## CoroutinesScope

协程作用域,为协程定义作用范围,是个接口,不建议手动实现该接口,首选委托.

```Kotlin
public interface CoroutineScope {
  // CoroutineContext是一个集合,比如[CoroutineName(fsdf), StandaloneCoroutine{Active}@d5d95d3, Dispatchers.Main.immediate]
  public val coroutineContext: CoroutineContext
}

//扩展属性isActive
public val CoroutineScope.isActive: Boolean 
		get() = coroutineContext[Job]?.isActive ?: true

//协程作用域的cancel,调用的就是Job的cancel
public fun CoroutineScope.cancel(cause: CancellationException? = null) {
  ...
  coroutineContext[Job]?.cancel(cause)
}

//泛型方法coroutineScope
public suspend fun <R> coroutineScope(block: suspend CoroutineScope.() -> R): R {
  ...
}

//CoroutineScope是一个接口,这个方法类似于构造函数≈
//子协程异常或取消都会导致整个协程的取消
@Suppress("FunctionName")
public fun CoroutineScope(context: CoroutineContext): CoroutineScope =
		ContextScope(if (context[Job] != null) context else context + Job())

public
```

### 协程作用域分类

1. 顶级作用域

2. 协同作用域

   在协程中启动另一个协程,新协程为子协程,默认是协同作用域,子协程的异常如果未捕获,会传递给父协程处理,如果父协程被取消,所有子协程也会被取消.

3. 主从作用域(监督作用域)

   SupervisorJob,supervisorScope{}

   区别与协同作用域,主从作用域的异常不会导致其他子协程取消,但是父协程取消了所有的子协程还是会被取消.

   父协程子协程

   	1. 子协程会继承父协程上下文中的Element,如自身有相同key的成员则覆盖,覆盖效果仅自身有效.
   	1. 默认子协程的未捕获异常会被父协程捕获.
   	1. 父协程取消会导致所有子协程取消.

### GlobalScope

是个object类,context是EmptyCoroutineContext.

```kotlin 
public object GlobalScope : CoroutineScope {
  override val coroutineContext = EmptyCoroutineContext
}
```

- not bound to any job.

  不与Job绑定

- 全局生命周期

- Do not replace `GlobalScope.launch { ... }` with `CoroutineScope().launch { ... }` constructor function call.

  自定义的协程作用域与GlobalScope.launch有同样的陷进,因此一般使用各种委托实现

### MainScope

```kotlin 
@Suppress("FunctionName")
public fun MainScope(): CoroutineScope = ContextScope(SupervisorJob() + Dispatchers.Main)
```

- 是一个方法,方法名大写是为了更直观的当做构造函数来使用
- 这个协程作用域的上下文是SupervisorJob() + Dispatchers.Main, 也就是主线程运行的

### supervisorScope

the provideed scope inherits its coroutineContext from the outer scope, but overrides Job with SupervisorJob.

### AbstractCoroutine

base class for implementation of coroutines in coroutine builders

- launch等方法创建的协程都是其子类.
- AbstractCoroutine继承JobSupport,Job,Continuation,CoroutineScope

### launch

创建一个StandaloneCoroutine并返回.

```kotlin
public fun CoroutineScope.launch(
  	context: CoroutineContext = EmptyCoroutineContext,
  	start: CoroutineStart = CoroutineStart.DEFAULT,
  	block: Suspend CorountineScope.() -> Unit
): Job {
  //CoroutineScope的扩展函数,顾名思义就是创建新的上下文
  val newContext = newCoroutineContext(context)
  val coroutine = if (start.isLazy) 
  		LazyStandaloneCoroutine(newContext, block) else
  		StandaloneCoroutine(newContext, active = true)
  coroutine.start(start,coroutine,block)
  return coroutine	
}
```

- 返回一个Job对象,实际上返回的是LazyStandaloneCoroutine或者StandaloneCoroutine,两者都是Job的子类

- 类继承关系

  LazyStandaloneCoroutine->

  StandaloneCoroutine(parentContext, active = false)->

  AbstractCoroutine<in T>(...)->

  JobSupport,Job,Continuation<T>, CoroutineScope

## CoroutineContext

是一个类似于Map的集合，元素类型是Element，每个Element有自己的key。

```
kotlin
```

CoroutineContext类解析

1. CoroutineContext里存放了各种Element集合,通过get(key)可以获取到对应的元素

   ```
   val job = coroutineContext[Job]
   ```

   ```
   val name = coroutineContext[CoroutineName]
   ```

2. Element是CoroutineContext的内部接口,同时它又实现了CoroutineContext,这么设计是为了保证Element中一定只能存放Element它自己,而不能存放其他数据结构

   1. ```
      It is an indexed set of [Element] instances,witch is bettween set and map. 是一个包含Element实例的有序的set,介于set和map直接.
      ```

   ```
   
   ```

#### CoroutineContext子类

Job,CoroutineDispatcher,CoroutineExceptionHandler,ContinuationInterceptor,CoroutineName等.

### Element

- Job

  协程的句柄。有cancel，join等方法

- CoroutineDispacher

  指定运行线程。

  - Main
  - IO。适合处理IO密集型任务（读写，网络等。线程数量可以多推荐为2N）
  - Default。适合处理CPU密集型任务（排序，计算等，线程数量推荐为CPU核数+1,多了也没用，因为CPU密集型任务需要大量占用CPU，没有多余的CPU给线程跑）
  - Unconfied。未指定，会在当前线程中执行，挂起恢复后会在恢复的线程中执行 。

  自定义线程：

  ​	可以用Executors的asCoroutineDispatcher方法来把线程池转为CoroutineDispacher。

- CoroutineName

- CoroutineExceptionHandler

  未捕获异常处理器。



### Job

```kotlin
public interface Job : CoroutineContext.Elemet {
  public companion object Key : CoroutineContext.Key<Job>
  public val isActive: Boolean
  public val isCompleted: Boolean
  public val isCancelled: Boolean
  public fun start(): Boolean
  public fun cancel()
  public suspend fun join()
  ...
}
```

- a background job, witch is cancellable and with a life-cycle that culminates in its completion.
- jobs can be arranged into parent-child hierarchies (which can be customized using SupervisorJob)
  - cancellation of a parrent leads to immediate cancellation of all its children recursively.
  - failure of a child with an exception other than CancellationException immediately cancels its parent and children.

#### CompletableJob

```kotlin
public interface CompletableJob: Job {
  public fun complete(): Boolean
}
```

#### SupervisorJob

```kotlin
@Suppress("FunctionName")
public fun SupervisorJob(parent: Job? = null) : CompletableJob = SupervisorJobImpl(parent)

//SupervisorJobImpl与普通的JobImpl区别就是重写了childCancelled方法,
private class SupervisorJobImpl(parent: Job?) : JobImpl(parent) {
  override fun childCancelled(cause: Throwable): Boolean = false
}

//JobSupport.kt
//parent decides whether it cancels itself
public open fun childCancelled(cause: Throwable): Boolean {
  if (cause is CancellationException) return true
  return cancelImpl(cause) && handlesException
}

//supervisorScope作用同SupervisorJob
public suspend fun <R> supervisorScope(block: suspend CoroutineScope.() -> R): R {
  contract {
    callsInPlace(block, InvocationKind.EXACTLY_ONCE)
  }
  return suspendCoroutineUninterceptedOrReturn {
    //启动了一个SupervisorCoroutine
    val coroutine = SupervisorCoroutine(it.context, it)
    coroutine.startUndispatchedOrRetrun(coroutine, block)
  }
}
```

- 接口同名构造方法,返回的是一个CompletableJob的实现类SupervisorJobImpl

- SupervisorJob的childCancelled方法返回的是false

- supervisorScope启动了一个SupervisorCoroutine,

  1. SupervisorCoroutine->ScopeCoroutine,重写了childCancelled方法,返回false

  2. ScopeCoroutine->AbstractCoroutine,CoroutineStackFrame
  3. AbstractCoroutine继承了JobSupport,Continuation,Job,CoroutineScope

JobImpl

#### JobSupport

- a concrete implementation of Job.

  Job的具体实现,包括job.cancel()等.

##### Job的cancel过程

- Job.cancel()->

- JobSupport.cancel()->

- JobSupport.cancelInternal()->

- JobSupport.cancelImpl()->

  根据job状态

  - cancelMakeCompleting(cause)

  - makeCancelling(cause)

    - if (state is Finishing)

      notifyCancelling()

    - if (state is Incomplete)

      - tryMakeCancelling
      - tryMakeCompleting

#### Job系列继承关系

JobSupport -> Job, ChildJob, ParentJob

JobImpl -> JobSupport,CompletableJob

#### 协程的取消机制

- 协程只有处于挂起状态时才能被取消
- 父协程被取消时，会取消所有子协程。

- 协程取消的原理
		1. 协程作用域内抛出CancellationException
		2. 协程检测到这个异常之后主动结束执行
	
	- CancellationException
		不是普通的异常，如果协程因CancellationException而结束，应该当做协程正常终止。为了让CancellationException能正常终止协程，不应该在代码里捕获这个异常，而是继续抛出这个异常，能让上层去进行后续操作。如果反复捕获并忽略这个异常，会产生僵尸协程。
	
- 不可取消的协程
	
	withContext(NonCancellable){}里的协程不可取消，执行完了才会返回。

## 超时处理

withTimeout(limit){...}

#### 异常处理

Launch和Async的异常处理方式不同.

##### Launch

- try catch

​	可以捕捉launch方式启动的协程.使用try catch需要注意的是,它不能捕捉到别的线程或者回调的异常.

```kotlin
try {
  CoroutineScope().launch { //会立即返回Job对象.
    ...
  }
} catch(e: Exception) {
  
}
// 注意这样就是不行的. lanch方法会立即返回Job对象,而launch里的代码块,可能是在别的线程完成的,当异常发生时,try-catch块早就执行完毕了.回调函数同理.
```

- CoroutineExceptionHandler

除了使用try catch之外,更推荐使用CoroutineExceptionHandler对异常进行统一处理.

异常会层层传递到根协程,所以CoroutineExceptionHandler只能在根协程中才能生效.

```kotlin
val exceptionHandler = CoroutineExceptionHandler { coroutineContext,exception ->
  
}
coroutineScope.launch(exceptionHandler) {
  throw RuntimeException("") //异常会被exceptionHandler捕获.
}
```

- CoroutineExceptionHandler只能在根协程或者supervisorJob的直接子协程里生效

  ```kotlin
  //普通Job子协程不生效
  coroutineScope.launch{
    	launch(CoroutineExceptionHandler{_,e-> }){ //使用CoroutineExceptionHandler了,但是不会生效,因为不是根协程,还是会崩溃.
        3/0
      }
  }
  // 子协程异常会代理给父协程,会一直向上传递到根协程.如果根协程没有CoroutineExceptionHandler,就会走UncaughtExceptionHandle
  
  //supervisorJob直接子协程生效
  supervisorScope {
    launch(CoroutineExceptionHandler{_,e-> }){ 3/ 0}  // 这里可以正常被捕捉,异常往上传递到supervisorScope时会交由子协程自己处理.
  }
  ```

- 构造函数里的CoroutineExceptionHandler,和launch方法里的CoroutineExceptionHandler,哪个生效?

  ```kotlin
  CoroutineScope(CoroutineExceptionHandler { _, throwable ->
     LL.e("xdd11 $throwable")
  }).launch(CoroutineExceptionHandler { _, throwable ->
     LL.e("xdd22 $throwable")
  }){
    launch(CoroutineExceptionHandler { _, throwable ->
     LL.e("xdd33 $throwable")
     3/0
  	}
  }
  // 会输出 xdd22.
  // luanch方法里的CoroutineExceptionHandler生效,如果launch里没有设置,那么构造方法里的会生效.
  ```

##### Async

 - try catch

   



以下代码会不会闪退?为什么?

```kotlin
lifecycleScope.launch{
  try {
      val deffer = async{
        3/0
        return @async null
  		}
    val res = deffer.await()
  } catch (e:Exception){
    e.printStacktrace()
  }
}

```

lifecycleScope本身是一个SupervisorJob,不会因为子协程的失败而取消, 

deffer.await()处会抛出异常,这个异常会直接传播到launch的CoroutineExceptionHandler里,而不会被try-catch捕获,这是因为,Coroutines的设计中,launch或async启动的协程内部的未捕获异常,不会像普通JAVA代码一样一层层上抛给try-catch,相反他们会走协程的异常处理机制.

对于 `launch` 协程，异常会传播到其父 `Job`，然后最终传播到 `CoroutineExceptionHandler`。

对于 `async` 协程，异常会被 `Deferred` 持有，并在 `await()` 时重新抛出。这个重新抛出的异常，就相当于在 `await()` 所在的协程内部抛出的一个新异常。

因此需要设置一个CoroutineExceptionHandler来处理异常就能避免上述代码崩溃.

## 协程的本质

状态机+跳转。

### CPS（Continuation Passing Style）

续体传递风格。函数并不直接返回结果，而是接收一个代码块作为最后一个参数。其实这个代码块就是回调函数，称之为续体(Continuation)，它会决定程序接下来的行为。整个程序就通过一个一个的Continuation拼接在一起。

### 挂起点

协程的代码**并不是随时挂起，只有遇到了挂起点才会挂起**。什么是挂起点呢？就是执行到了被*suspend*修饰的函数时，就会被挂起。

### Continuation

~~~kotlin
public interface Continuation<in T> {
  public val context: CoroutineContext
  public fun resumeWith(result: Result<T>)
}
~~~



## QA

下边代码会有什么问题？

~~~kotlin
launch {
  while (true) {
    try {
      doWork()
      delay(1000)
    } catch (e: Exception) {
      logError(e)
    }
  }
}
1. while(true)导致了无限循环，即使收到了CancellationException也退出不了。
2. 捕获了CancellationException异常，没有抛出，也就是吞掉了取消信号。这段代码会导致 死循环继续跑，变成僵尸协程（占着资源不干活，也不退出）。
如何解决上述问题？
- 第一步，while循环增加isActive判断协程是否被取消，
- 第二步，不捕获CancellationException异常，重新抛出该异常。
		- 或者delay()移除try-catch，让delay()方法能直接抛出该异常 。
~~~



# Flow

An asynchronous data stream that sequentially emits values and completes normally or with an exception. 

Flow is Reactive Streams   compliant, you can safely interop it with reactive streams using Flow. asPublisher and Publisher. asFlow from kotlinx-coroutines-reactive module.

## Flow,LiveData,RxJava,Channel

- LiveData缺点

  - 数据倒灌(粘性)

    - 新注册的订阅者会收到LiveData存储的数据

      SingleLiveData或者UnPeekLiveData解决

  - 不防抖

    - 重复setValue会收到多次

      distinctUntilChanged()解决

  - 不支持背压

- RxJava缺点

- Flow优点

## 冷流和热流

```kotlin
#SharedFlow.kt

A hot Flow that shares emitted values among all its collectors in a broadcast fashion, so that all collectors get all emitted values. A shared flow is called hot because its active instance exists independently of the presence of collectors. This is opposed to a regular Flow, such as defined by the flow { ... } function, which is cold and is started separately for each collector.

热流发射的数据通过广播机制在收集者之间共享.热流之所以被称为热流,是因为它的活动实例独立于收集者而存在,冷流则相反,每个收集者都会对应一个独立的流.
```

- 冷流(Flow)

  订阅者collect数据时才按需执行发射数据的代码.冷流和订阅者是一对一关系,不共享,没有缓存机制.

- 热流(SharedFlow,StateFlow等)

  共享,有缓存机制,无论是否有订阅者collect,都可以生产数据并且缓存起来.热流和订阅者一对多,多个订阅者共享一个数据流.当一个订阅者停止监听时,数据流不会自动关闭(除非使用WhileSubscribed策略)

## 普通Flow

数据源会延迟到消费者开始collect时,才生产数据,并且每次订阅都会创建一个全新的数据流.

```kotlin
public interface Flow<out T> {
  public suspend fun collect(collector: FlowCollector<T>)
}
```

### Flow Builders

- flowOf()
- asFlow()
- flow{}
- channelFlow{}

### Flow Constraints

1. Context Preservation

   flow具有上下文保留属性,并且永远不会将其传播或者泄露到下游,除非使用flowOn()来改变上下文.

   flow里应当禁止使用GloableScope,withContext等改变上下文的方法

2. Exception Transparency

flow接口的所有实现都必须遵循上下文保留和异常透明性这两个关键属性.

### Reactive streams 

响应式流,可以与遵循响应式流规范的其他库或框架（如 Reactor、RxJava 等）无缝集成.

## FlowCollector

is used as an intermediate or a terminal collector of the flow and represents an entity that accepts values emitted by the Flow.

通常不直接实现此接口,用作流的中间或者终端收集器来接受Flow发出的值,非线程安全.

```kotlin
public fun interface FlowCollector<in T> {
  public suspend fun emit(value: T)
}
```

注意到FlowCollector是一个fun interface,这是一种特殊的接口,包含且只包含一个抽象成员函数,这种接口允许使用更简洁的语法来创建其实例,特别是在Lambda表达式等场景下,因此可以使用如下写法:

```kotlin
val flow = getEvents()
//collect()方法的参数就是一个FlowCollector,由于是fun interface,可以简写.
flow.collect{ value->
  
}
```

如果不是fun interface,而是Interface呢?

```kotlin
//常规的Interface写法如下
flow.collect(object:FlowCollector{
  override fun emit(value:Int){
    
  }
})
```



## SharedFlow

热流,高配版LiveData,StateFlow是其子类.

- emitted values among all its collectors in a broadcast fashion.

  发射的数据在订阅者之间共享.

- its active instance exists independently of the presence of collectors. 

  热流的活跃实例独立于订阅者而存在

- Shared flow never completes. A call to Flow. collect on a shared flow never completes normally, and neither does a coroutine started by the Flow. launchIn function. An active collector of a shared flow is called a subscriber.

  热流不会主动完成,订阅者的collect方法不会导致热流停止. 热流的活跃的收集者也被称为订阅者.

- A subscriber to a shared flow is always cancellable

  热流的订阅者是可以被取消的(比如协程作用域被取消了),因此需要check cancellation.

  热流的大部分的终端操作符,比如Flow.toList(),可能也不会完成,但可以是用截断操作符(flow-truncating)比如Flow.take,Flow.takeWhile来把共享流转换成可以完成的流.

- 冷流可以通过sharedIn操作符来转换成热流

- There is a specialized implementation of shared flow for the case where the most recent state value needs to be shared.

  `SharedFlow` 提供了在多个订阅者之间共享事件或状态值的机制，而 `StateFlow` 是 `SharedFlow` 的一个特化版本，专门用于共享最新的状态值。

- Replay cache and buffer

  - A shared flow keeps a specific number of the most recent values in its replay cache. Every new subscriber first gets the values from the replay cache and then gets new emitted values. 

    热流的replay cache缓存了最近发射的值,订阅者首先从replay cache获取值,再获取新发射的值.

  - A replay cache also provides buffer for emissions to the shared flow, allowing slow subscribers to get values from the buffer without suspending emitters.

    repaly cache提供了缓冲区,使得慢速的订阅者可以从缓冲区获取数据,而不会发射者被挂起.

  - onBufferOverflow

- Concurrency

  All methods of shared flow are thread-safe and can be safely invoked from concurrent coroutines without external synchronization.

- Operator fusion

  操作符融合

  Application of flowOn, buffer with RENDEZVOUS capacity, or cancellable operators to a shared flow has no effect.

```kotlin 
public interface SharedFlow<out T> : Flow<T> {
  //缓存的数据列表
  public val replayCache: List<T>
  override suspend fun collect(collector: FlowCollector<T>): Nothing
}
```

- replay
		-  新订阅者进来就能看到回放的数据
	- extraBufferCapacity
		- 缓冲区容量，这个缓存区对所有消费者来说是共享的。
		- 如果没有活跃的消费者，那么extraBufferCapacity里不会存放数据。
		- 如果有多个活跃的消费者，每个消费者都可以从extraBufferCapacity里取出数据，触发缓冲区更新，那么消费速度慢的，可能会丢失缓冲区里的数据的收集。
	- extraBufferCapacity和buffer操作符的区别？
		- extraBufferCapacity是SharedFlow的构造函数里的，所有下游消费者共享。
		- buffer()是操作符，适用于下游的单个收集者。
	- extraBufferCapacity为0，如果SharedFlow有两个订阅者，一个活跃一个非活跃，此时上游emit数据，下游的活跃的收集者能收到订阅吗？

- emit与tryEmit有什么区别？
	 -  emit是挂起函数，必须协程中使用，无返回值，需要等待下游收集者处理
	 - tryEmit非挂起，可以在任意地方使用，返回发送成功或者失败，不等待下游的收集，需要自己处理发送失败的情况。  （当出现背压，下游来不及处理，缓存区满了，协程取消了等场景会失败。）!
- 什么是flow的挂起点？Suspension points
	- delay(),ensureActive()等
	- flow会在挂起点检查取消状态
- flow的异常处理
	- CancellableException
	- runCatching和try...catch什么区别？

## MutableSharedFlow

- a SharedFlow that also provides the abilities to emit a value, to tryEmit without suspension if possible, to track the subscriptionCount, and to resetReplayCache.
- thread-safe

```kotlin
public interface MutableSharedFlow<T> : SharedFlow<T>, FlowCollector<T> {
 	//可以发射数据
  override suspend fun emit(value: T)
  //不阻塞的方式发射,如果订阅者繁忙或者没接收到数据返回false.
  public fun tryEmit(value: T): Boolean
  //The number of subscribers (active collectors) to this shared flow.
  //实时的活跃的订阅者数量,这是一个StateFlow
  public val subscriptionCount: StateFlow<Int>
  public fun resetRepalyCache()
}
```

构造方法

```kotlin
public fun <T> MutableSharedFlow(
	replay: Int = 0, 
  extraBufferCapacity: Int = 0,
  onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND
): MutableSharedFlow<T> {
  ...
  return SharedFlowImpl(replay, bufferCapacity, onBufferOverflow)
}

```

- replay 

  重放缓存大小.

- bufferCapacity

  缓冲区容量.

- BufferOverflow

  缓冲区溢出策略.

  - SUSPEND
  - DROP_LATEST
  - DROP_OLDEST
  - DROP

Flow的发射和收集流程

```kotlin


```

遇到一个场景,我的appViewModel里监听到了recallMsg事件后emit了,我的ConversationList和ChatActivity都会对该SharedFlow进行observer,然后我的Activity里只收集到了第一个recallMsg的值,并没有收集到后续.

配置了replay和buffer之后问题解决.这就涉及到了SharedFlow的特性.

SharedFlow的核心是共享,但在默认配置下,它对所有消费者是一视同仁的严格模式,当你调用 `emit` 发送一个值时，`SharedFlow` 会等待**所有当前活跃的消费者**都接收到这个值后，才会继续处理下一个值。但如果存在以下情况，就会出问题：



1. **某个消费者处理速度慢**
   比如第一个消费者在处理第一个值时耗时较长（比如在主线程做了复杂计算），而第二个值已经发射过来。此时 `SharedFlow` 会因为「第一个消费者还没处理完」，可能将第二个值缓存（如果缓冲没满）或直接丢弃（如果缓冲满了）。

2. **消费者在动态增减**
   比如第一个值发射时，A、B 两个消费者都在监听，成功接收；但第二个值发射时，B 消费者刚好退出（比如页面关闭），而 `SharedFlow` 默认不会为「已退出的消费者」保留值，也可能导致后续值传递异常。

3. **缓冲池被占满**
   默认 `SharedFlow` 的 `extraBufferCapacity = 0`（缓冲池大小为 0），如果多个消费者处理速度跟不上发射速度，缓冲池很快会满，后续的值会根据 `onBufferOverflow` 策略处理（默认是挂起发射方，直到缓冲有空间，但如果是并发发射，就可能丢值）

   ### 多接收者场景的正确配置

   如果需要多个地方接收，且希望尽量不丢值，建议显式配置 `SharedFlow` 的参数：

## StateFlow

- skips fast updates, but always collects the most recently emitted value

- StateFlow is useful as a data-model class to represent any kind of state

- Stateflow is a special-purpose, high-performance, and efficient implementation of SharedFlow for the narrow, but widely used case of sharing a state.

  是一个专用,高性能,高效的SharedFlow,针对共享双状态这一狭隘但是广泛使用的场景.

- Stateflow always has an initial value, replays one most recent value to new subscribers, does not buffer any more values, but keeps the last emitted one, and does not support resetReplayCache.

- Operator fusion

  flowOn,conflate,buffer with CONFLATED or RENDEZVOUS capacity, distinctUntilChanged,cancellable这些操作符都是无效的.

```kotlin
//StateFlow是SharedFlow的子类,实现如下
val shared = MutableSharedFlow(     
  replay = 1,     
  onBufferOverflow = BufferOverflow. DROP_OLDEST) 
	shared. tryEmit(initialValue) // emit the initial value val state = shared. 
	distinctUntilChanged() // get StateFlow-like
```



```kotlin 
public interface StateFlow<out T> : SharedFlow<T> {
	public val value: T
}
```

## MutableStateFlow

```kotlin
public interface MutableStateFlow<T> : StateFlow<T>, MutableSharedFlow<T> {
  public override var value: T
  public fun compareAndSet(expect: T, update: T): Boolean
}
```

MutableStateFlow更新值原理

## flow的各种操作符

- 副作用函数
  - onStart,onEach,onCompletion,onSubscription(有消费者collect的时候)

- buffer()
	创建缓冲区，允许消费者来不及消费时放入缓冲区。
	
- conflate()
	处理旧数据之前，如果有新数据到达，会丢弃旧数据。
	buffer可以缓存数据，conflate会丢弃数据，如果同时设置了buffer()和conflate()，是否会丢弃缓冲区的数据？
	- 这两个操作符语义上是冲突的，以操作符顺序为准，后一个会覆盖前一个的行为。
	
- collectLatest()
	处理数据时，如果有新数据到达，会取消当前正在进行的处理，进行处理下一个。
	conflate()作用是丢弃中间值，只处理最新一个。
	
	buffer，conflate，collectLast都是flow的背压处理API。
	
- flowOn()

  切换线程，flow中不能使用withContext来切换，只能使用flowOn来切换线程，并且**flowOn只影响上游，不影响下游**（终端是最下游肯定不会影响到）

  终端永远都是在调用的线程。

  ~~~Kotlin
  withContext(Dispatchers.Main) {
      val singleValue = intFlow // will be executed on IO if context wasn't specified before
          .map { ... } // Will be executed in IO
          .flowOn(Dispatchers.IO)
          .filter { ... } // Will be executed in Default
          .flowOn(Dispatchers.Default)
          .single() // Will be executed in the Main
  }
  ~~~

- catch

  与flowOn一样，只影响到上游发生的异常，关不了下游

- retry系列

冷流转为热流

stateIn,sharedIn

# 高级用法

### 中缀函数

### 数组,String,和高阶函数

- joinToString

  ```kotlin
  val arr = listOf(1,2,3), // [1,2,3]
  arr.joinToString(separator = "|", prefix = "{", postFix = "}"), // "{1|2|3}"
  listOf(1,2,3).toString() // 
  arrayOf(1,2,3).toString(),//
  public fun <T> Iterable<T>.joinToString(separator: CharSequence = ", ", prefix: CharSequence = "", postfix: CharSequence = "", limit...){
  
  }
  ```

  

- ifEmpty，ifNotEmpty,  orEmpty

### SAM转换

 Single Abstract Method Conversions. fun Interface的使用。

### runCatching
- 
- 
- runCatching和try...catch有什么区别？
   runCatching其实就是Kotlin里对try catch的一个封装，对请求成功或者失败返回了请求结果Result<T>。
   那runCatching的优势在哪里？
   优雅的链式调用，适用于，对结果进行map转换等场景。

QA

## 解构

## contract

协议,约定,能解决一些场景下编译器不够智能的问题.

1. 仅用于顶层方法
2. 必须置于方法开头,且至少包含一个Effect.

```kotlin
//这个方法居然是一个空实现就离谱,contract的方法的参数是ContractBuilder的扩展函数
public inline fun contract(builder: ContractBuilder.() -> Unit) {}
```

```kotlin
@ContractsDsl
@ExperimentalCntracts
@SinceKotlin("1.3")
public interface ContractBuilder {
  // Returns和CallsInPlace都是是SimpleEffect子类
  @ContractsDsl public fun returns(): Returns
  @ContractsDsl public fun returns(value: Any?): Returns
  @ContractsDsl public fun returnsNotNull(): ReturnsNotNull
  @ContractsDsl public fun <R> callsInPlace(lambda: Function<R>, kind: InvocationKind = InvocationKind.UNKNOWN): CallsInPlace

}

```

ContractBuilder提供了callsInPlace,returns的方法, return

```kotlin
public interface SimpleEffect : Effect {
  public infix fun implies(booleanExpression: Boolean): ConditionalEffect
}
```

Returns -> SimpleEffect -> Effect

```kotlin
// 返回value时,condition成立
contract {
  retruns($value) implies ($condition)
}
```



apksigner sign \  --ks /Users/zhanghui/Library/Android/sdk/build-tools/30.0.3/pxappkey.jks \  --ks-key-alias pxb7app \  --ks-pass pass:pxb7667788 \  --in "/Users/zhanghui/Library/Android/sdk/build-tools/30.0.3/px.apk" \  --out /Users/zhanghui/Library/Android/sdk/build-tools/33.0.2/px2.apk

```kotlin
@OptIn(ExperimentalContracts::class)
fun String?.isAvailable(): Boolean {
  contract {
    // 返回为true 暗示 值不为null
    returns(true) implies (this@isAvailable != null)
  }
  return this != null && this.lenth > 0
}

如果不加上述contract,那么下边代码编译会报错
var b:String
var a:String? = getString()
if(a.isAvailable){
  //contract作用就是让编译器知道返回为true 暗示 值不为null,加了contract就能编译通过
  b = a
}


//调用一下block,并且返回自身
public inline fun <T> T.apply(block: T.() -> Unit): T {
  contract {
    callsInPlace(block, InvocationKind.EXACTLY_ONCE)
  }
  block()
  return this
}
1. 执行一下Lambda表达式
2. 返回调用者
3. Lambda表达式具有接收者类型T,因此在Lambda表达式中可以用this来访问自身

```



# 泛型

泛型generic,也就是参数化类型ParameterizedType.

## 泛型优点

1. 更健壮,编译时进行更强的类型检查
2. 更通用,代码适用多种类型无需重载
3. 更简洁,消除强转,编译后会自动增加强转

## 泛型使用

泛型类

泛型方法

	- fun <T> ...
	- fun <T :BaseRepository>..  //添加了泛型约束的泛型方法

## 泛型约束

上界约束(upper bound), T: BaseRepository, 同Java的 T extend BaseRepository, 默认上界是 Any?

对于多个上界的约束,可以用where

```kotlin
fun <T> setData(t: T)
	where T: Cloneable,
				T: Parcelable
{
  ...
}
```

## 型变 variance

什么是型变?

Java中的泛型是型变的吗?

Kotlin中没有通配符类型?.它有声明处型变和类型投影

声明处型变(declarationi-site variance)

#### 协变 covariant

带上界限定的,? extends Number, T: Number, 只读, 不能写入,PE

不能调用add(T) 方法写入数据,可以调用clear()

#### 逆变 contravariance

下界限定, ? super ,CS,

助记符  PECS  product-extends,consumer-super,

协变,out,

## 声明处型变

当一个类C的类型参数T被声明为out时,它就只能出现C的成员的输出位置(返回值,不能是参数),但是同时,C<Base>可以安全地作为C<Drived>的超类.(类C是T的生产者,而不是T的消费者.) 

```kotlin 
interface Source<out T> {
  fun nextT(): T // T只能出现在Source的成员的输出位置
}
fun demo(strs: Source<String>) {
  val objs: Source<Any> = strs //编译通过.
}
```

型变注解in,使得一个类型参数逆变,也就是只可以消费,不可以生产.

```kotlin
interface Comparable<in T> {
  operator fun compareTo(other: T):Int 
}
```

## 类型投影

interface Function<in T, out U>

\* 表示in Nothing, out Any?

## 类型擦除(Type Erasure)

泛型声明和类型安全检查仅在编译期间,在运行时不会保存泛型的任何信息,也就是类型信息在运行时都会被擦除.Foo<Bar>会被擦除成Foo<*>.

泛型本质是JVM的语法糖,泛型是在JDK1.5引入的新特性,为了向下兼容,JVM和Class文件并没有提供泛型的支持,而是让编译器擦除Code属性中的所有泛型信息.

### 类型擦除后如何获取泛型相关信息?

1. 使用反射和ParameterizedType

Java反射提供了一种机制，可以在运行时查询类的信息。对于泛型，尽管泛型类型参数在运行时被擦除，但反射API中的`Type`接口及其子接口（如`ParameterizedType`）提供了一种机制来获取泛型声明时的类型参数信息。

- **步骤**：
  1. 获取类的`Class`对象。
  2. 使用反射获取泛型参数化类型的`Type`对象。
  3. 检查该`Type`对象是否是`ParameterizedType`的实例。
  4. 如果是，则通过`ParameterizedType`的`getActualTypeArguments`方法获取实际的类型参数。
- **示例**：

```java
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
 
public class GenericTypeFetcher<T> {
    private final Type type;
 
    public GenericTypeFetcher() {
        this.type = ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0];
    }
 
    public Type getType() {
        return this.type;
    }
 
    public static void main(String[] args) {
        GenericTypeFetcher<List<String>> fetcher = new GenericTypeFetcher<List<String>>() {};
        Type type = fetcher.getType();
        System.out.println("The type is: " + type.getTypeName());
    }
}
```

2. 使用TypeToken模式

TypeToken模式是一种设计模式，用于在运行时保存和提供泛型类型的类型信息。这种方法通常与一些库（如Gson、Jackson等）一起使用，以处理复杂的泛型结构。

- **步骤**：
  1. 创建一个匿名内部类来捕获泛型类型的类型信息。
  2. 通过调用该匿名内部类的`getType`方法获取泛型类型的类型信息。
- **示例**（使用Gson的TypeToken）：

```java
import com.google.gson.reflect.TypeToken;
import java.lang.reflect.Type;
import java.util.List;
 
public class Main {
    public static void main(String[] args) {
        Type listType = new TypeToken<List<String>>(){}.getType();
        System.out.println("Captured type: " + listType);
    }
}
```

3. 在类文件中查看泛型签名

虽然泛型信息在运行时被擦除，但在编译后的类文件中，泛型签名仍然保留在常量池中。这可以通过反编译工具或查看类文件的字节码来查看。然而，这种方法通常用于调试或分析目的，而不是在运行时获取泛型信息。

总结

尽管Java的泛型信息在编译后会被擦除，但通过反射API中的`Type`接口及其子接口（如`ParameterizedType`）以及TypeToken模式等技巧，我们仍然可以在运行时获取到泛型声明时的类型参数信息。这些方法对于需要在运行时处理泛型类型的高级应用场景非常有用

### 类型擦除步骤

1. 擦除所有类型参数信息，如果类型参数是有界的，则将每个参数替换为其第一个边界；如果类型参数是无界的，则将其替换为 Object
2. （必要时）插入类型转换，以保持类型安全
3. （必要时）生成桥接方法以在子类中保留多态性

与[Java泛型](Java.md#泛型Generics)区别。

- 上界 out

  extend,out,协变，生产，只能作为返回值，不能set

- 下界 in

  super ,in ,逆变，消费，只能作为参数，不能get

- 星号投影 * 

  表示任意具体类型的泛型都可以

  - Function<*, String> 意思是Function<in Nothing, String>
  - Function<Int, *> 意思是Function<Int, out Any?>
  - Function<*, *> 意思是Function<in Nothing, out Any?>

  换句话来理解，就是当不指定具体的类型参数，用星星就代表着不知道具体的类型参数，那么视具体的上下文不同星号会被解释不同的意思。不过这玩意儿可读性较差，除非必不得已，否则还是能不用就用它。

- reified 

  /ˈriːɪfaɪd/ ，reify的过去式，使具体化的

  泛型擦除会导致泛型对象无法做类型检查和转换（is T, as T）。

  reified作用就是保留泛型类型，需要跟inline一起使用

- 绝不为空类型

  可以**用& Any来声明**，如<T & Any>来声明T是『绝不为空类型』。在纯Kotlin代码中是用不到这个特性的。只有当涉及Java的@ NonNull时才需要『绝不为空类型』。



# 委托

委托与代理的区别

委托：是一种实现机制，不自己实现，委派给别的对象去实现。代理模式就是委托的一种实现机制。

代理：是一种正式的设计模式，强调权限和隔离，client只能访问到proxy，而并不知道realOjbect。

委托模式是实现继承的一个很好的替代方式.

什么是委托?

​	委托就是一个对象将某些实现交给另一个对象来处理.

kotlin的委托解决了什么问题?

​	kotlin同过by关键字可以更优雅的实现委托.

### 类委托

```kotlin
interface PictureLoader{
  fun loadPicture()
}

class GlideEngine(): PictureLoader {
  override fun loadPicture(){
    Glide.load()
  }
}

class DefaultLoader(loader: PictureLoader): PictureLoader by loader

fun main(){
  val myLoader = DefaultLoader(GlideEngine())
  myLoader.loadPicture()
}

fun loaders(): PictureLoader{
  val loader = GlideEngine()
  return loader
}

val loader:PictureLoader by loaders()
```

### 委托和重写

```kotlin
interface A {
  fun Test()
  val d:Int
}
class B : A {
  override val d = 3
  override fun Test() {
    //
  }
}
class C(b:B): A by b {
  override val d = 5 // b中访问不到这个d
  override fun Test() {} // 生效的是C中的实现,而不是被委托类B中的实现
}
```

### 属性委托

属性委托不需要提供接口,但是需要一个getter,(var 还要有setter函数.) 属性委托实际上就是类委托的简化,被代理的逻辑就是这个属性的get/set方法.

#### kotlin内置属性委托模版

- lazy properties

- observable properties

  ```kotlin 
  // observable观察值变化, vetoable()可以在赋值之前截获并处理.
  by Delegates.observable(""){prop,old,new->
     
  }
  ```

- map

  将属性储存在映射中.

#### Kotlin提供了两个接口用来方便实现属性委托

```Kotlin
interface ReadOnlyProperty<in R, out T> {
  operator fun getValue(thisRef: R, property: KProperty<*>): T
}
interface ReadWriteProperty<in R, T> {
  operator fun getValue(thisRef: R, property: KProperty<*>): T
  operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```



#### 委托给另一属性

属性委托可以委托给另一个属性,这是很有用的.例如，当想要以一种向后兼容的方式重命名一个属性的时候：引入一个新的属性、 使用 `@Deprecated` 注解来注解旧的属性、并委托其实现。

#### 局部委托属性

可以把局部变量声明为委托属性.

#### by lazy

```kotlin 

```

# 集合

## Sequence

惰性集合操作,Kotlin中的新容器,和iterator一样用来遍历.Sequence不会立即计算结果，而是尽可能延迟计算结果。当对Sequence进行中间操作时，这些操作不会立即执行，而是返回一个新的Sequence对象，该对象封装了原始Sequence和要执行的操作。只有在进行终端操作时，Sequence才会实际执行所有中间操作，并按需计算每个元素的值。

Sequence高效的原因:

- 一旦满足条件,退出遍历
- 代码不会立即执行,被调用的时候才执行.
- 避免每一步都创建集合对象
- 适用于大量数据有多个中间操作的场景,能显著提升性能.

Sequence的创建

1. **`sequenceOf`**
2. **`asSequence`**
3. **`generateSequence`**

## 常用操作

- filter {}

  过滤掉不满足的,并且过滤后数组会变成list

- drop/take

- distinct



# 注解

注解是将元数据附加到代码的方法. annotation class Test

- @Target 元素的可能类型
- @Retention 是否存储在编译后的Class文件中,以及运行时能否通过反射可见.
- @Repeatable 
- @MustBeDocumented

## 常用注解

- @JvmOverloads

  让编译器自动生成重载的方法(比如有默认参数的函数)

## KSP

Kotlin Symbol Processing。专门为Kotlin语言设计的新一代注解处理工具。跳过了KAPT生成Java存根的繁琐步骤直接与Kotlin编译器交互。

**工作原理**：KSP 直接分析 Kotlin 代码的抽象语法树（AST），即代码的符号结构。它提供了一个与 Kotlin 编译器紧密集成的、基于符号处理的 API。处理器可以直接访问和理解 Kotlin 特有的语法结构，如扩展函数、可空类型、伴生对象等。

如何实现 KSP processor?



## KAPT

Kotlin Annotation Processing Tool。kapt的出现主要是为了让Java的注解处理器能在Kotlin项目中正常工作。

工作原理：

1. 先编译Kotlin源码
2. 再调用Java注解处理工具APT来分析和处理
3. 最后将生成的代码和原始代码一起编译。

# Kotlin各个版本特性

## 1.9.20

### New Kotlin K2 compiler updates

K2编译器升级,提升编译速度

### Kotlin/Wasm支持

Kotlin/Wasm使得kotlin可以被编译成Wasm字节码(WebAssembly),从而在Web上运行.

### Custom memory allocator enabled by default

默认开启了内存分配机制优化,垃圾收集和分配效率提升,减少了GC时的等待时间

### Companion object initialization on class constructor calls

提升了Companion object在跨平台时候的表现

### Cinterop 生成声明的强制选择加入 (Opt-in) 要求

cinterop工具是用来从C和Object-C库生成Kotlin声明用的.

### Kotlin Multiplatform is Stable

kotlin跨平台现在稳定了,可以支持Android,web,ios,macOS,那么问题来了,要怎么用?怎么配置?

### Improved performance of HashMap operations in Kotlin/JS

## 1.8.0



# Kotlin跨平台

### KMM,Kotlin Multiplatform Mobile

AndroidStudio安装KMM插件后,可以运行KMM项目. 配合XCode上运行IOS,可以实现跨平台开发. 

Shared Model作为公用Model实现通用功能,但是UI需要在各自端开发,JetPackCompose编写的UI只能在Android端运行.

# 面试专题

## Kotlin基础

1. == , === , equals() 区别?Java中呢?

   ```Kotlin
   Kotlin中
   	==同equals(),比较结构是否相同,===比较内存地址
   Java中
   equals()比较结构,==比较内存地址.
   ```

2. Array和IntArray区别?

   ```Kotlin
   Array<T>相当于引用类型数组Integer[],IntArray相当于数值类型数组int[]
   ```

3. 顶级成员/属性,函数的原理?

   ```Kotlin
   本质是Java静态成员,编译后会生成文件名kt的类.
   ```

4. Kotlin有哪些循环的方式?

   ```Kotlin
   for(i in 0..100)
   map.forEach{}
   for(i in map.indices)
   while()
   iterator()
   repeat{}
   ```

5. 默认参数的原理?Java可以直接调用带默认参数的函数吗?

   ```kotlin
   本质是将参数固化到调用位置,因此在Java中无法直接调用带默认参数的函数,需要在kotlin方法中添加@JvmOverloads注解来让编译器生成重载的方法.
   ```

6. 解构声明的原理?

   ```kotlin
   本质是局部变量
   ```

7. 委托机制的原理?

8. Sequences序列的原理?为什么能提升性能?

   ```kotlin
   1. 惰性
   2. 中间步骤不生成队列,而是到末端操作才统一处理.
   ```

9. 扩展函数,中缀函数原理?

   ```kotlin
   静态函数,函数第一个参数为类本身
   ```

10. 高阶函数原理?

11. let,apply,with,also,run区别?

    ```kotlin
    这几个都是高阶函数,它们的参数都是函数类型的,主要区别就是这个函数类型的参数不同,以及他们返回值不同.
    let,传入的Lambda表达式参数是本身,返回执行结果block()
    also,传入的Lambda表达式参数是本身,返回本身
    apply,传入的是带接收者的Lambda表达式,返回本身
    run,传入的是带接收者的Lambda表达式,返回block()
    ```

12. object和companion object的区别?

    ```kotlin
    object
    	1. 静态类
    	2. 静态匿名内部类
    companion object
    	伴生对象,一个类中只能有一个,代表了类的静态成员
    ```

13. 说说Kotlin的特点

14. 什么是backing field

15. laterinit能用于声明基本数据类型吗?为什么?by lazy呢?

16. Kotlin可见性?与Java的区别?

17. 说说Kotlin的构造函数

18. object关键字作用?是饿汉还是懒汉?手写单例

19. companion object作用?一个类中可以有多个companion object吗?为什么?

20. 说说Kotlin的内部类,嵌套类,密封类,枚举类,数据类.

21. Kotlin反射和Java反射的区别?

22. Kotlin有哪些遍历的方法?

23. lambda表达式的本质?与Java有什么区别?有返回值吗?

24. 什么是扩展函数?中缀函数?他们的原理是什么?可以用扩展函数的方式调用吗?

25. 什么是高阶函数?let,run,also,apply,with的区别?

26. 什么是inline,noinline,crossinline

## 泛型

1. 什么是泛型?泛型有什么优点?

   ```kotlin
   - 参数化类型.
   - 简洁,健壮,通用
   ```

2. 什么是泛型擦除?泛型擦除会带来什么影响(泛型擦除带来的限制)?泛型擦除后,怎么获取泛型的相关信息?类型擦除的步骤?为什么类型擦除之后,反编译还是能看到类型参数T?

   ```kotlin
   - 泛型是JVM的语法糖,Jdk1.5中引入,为了向下兼容,泛型只在编译时有效,运行时会被擦除.
   - 1. 不能将基类用于泛型参数
   	2. 无法直接创建类型参数的实例,可以通过反射创建.(1是由于擦除,2是编译器无法验证T是否有默认构造函数)
   	3. 无法声明类型参数的静态字段或方法.(类型实参是在创建对象时指定的,而静态成员的声明周期在这之外)
   	4. 无法创建参数类型数组
   	5. 无法将casts或instanceof与类型参数使用
   	6. 无法重载原始类型相同的方法
   - 1. 利用反射和ParameterizedType
   	2. 使用Gson等的TypeToken模式
   	3. 在类文件中查看泛型签名
   - 1. 擦除所有类型参数的信息,并将参数替换为它的第一个边界或者Object(如果参数是无界的)
   	2. 按需插入类型转换
   	3. 按需生成桥接方法
   - 泛型中所谓的类型擦除,其实只擦除Code属性中的泛型信息,在类常量池属性(Signature属性,LocalVariableTypeTable属性)中其实还保留着泛型信息,这也是在运行时可以反射获取泛型信息的根本依据.
   ```

3. 什么是协变?什么是逆变?什么是无界通配符?

   ```kotlin
   - out T, ? extends T , 生产者, 返回值,或者参数的类型
   - in ,super ,消费者, 参数
   - 
   ```

   

4. Kotlin泛型与Java有什么区别?

5. ```java 
   以下代码中,编译出错的是?
   public class MyClass<T> {
     private T t0;
     private static T t1;
     private T func0(T t){
       return t;
     }
      private static T func1(T t){
       return t;
     }
     private static <T> T func2(T t) { return t; } // 4
   } 
   ```

6. 

## Coroutine

1. 什么是协程?协程跟线程的区别?协程有哪些优点?

   ```Kotlin
   Kotlin官方推荐的异步编程方式. 可以以同步的方式写异步代码, 消除常规编程中回调地狱的问题.  协程它更像是一种轻量级的线程管理框架, 一个线程里可以开启多个协程,  一个协程可以挂起并且不阻塞当前线程,从而也不影响其他协程的执行.
   ```

2. 协程为什么比线程更高效?

   ```Kotlin
   1. 
   ```

   

3. 协程有哪些启动方式?有哪些作用域?

   ```Kotlin
   
   ```

4. 协程如何实现挂起操作的?

   

5. 协程在执行CPU密集型任务的时候如何取消?

6. 协程怎么处理异常的?

7. 

## Flow

1. 什么是flow?

2. flow跟LiveData比有什么优点?

3. flow怎么创建?

4. 什么是冷流热流?

5. flow有哪些常用的操作符?




数据结构

iCommon{

}



KOOM



```kotlin 
class BaseWidget{
  val color:String,
  val margin: IntArray,
 	val positon: Int,
  val isHidden: Boolean,
  
}

class TextWidget: BaseWidget{
  val value: String,
  val textStyle: Bold,Italic,Normal
}
class Image: BaseWidget{
  
}
ca
class Impu
```











点击返回->conversationList刷新了页面,更新了新信息->聊天





刷新





```
1. 我通过getConversationList方法获取到了某个会话的最新一条信息,但是点击进入该会话列表后第一时间没有拉取到这条最新的信息,而是过了半秒钟收到了这条最新消息,会存在这种情况吗?
2. 




1. APP在聊天页面里等

第一次, 外边先刷新->进入页面没拉到->收到了消息和提示音
第二次, 外边先刷新->进入页面拉到了信息,并且没收到消息提示


```













文章

# 

