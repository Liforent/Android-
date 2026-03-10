# Kotlin基础



# 高阶函数

## 作用域函数

- also
- let
- apply
- with
- run

区别是参数和返回值。



# 反射 Reflection

### 概述

Java反射允许程序在运行时：

1. 获取类的完整结构信息（类名，字段，方法，注解等。）

2. 动态创建类的实例对象。

3. 动态调用方法和访问字段。

4. 修改访问权限（即使是private）

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

## 协程启动方式CoroutineStart

1. DEFAULT
2. LAZY
3. ATOMIC
4. UNDISPATCHED

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


## 协程作用域CoroutinesScope

### 有哪些协程作用域？

#### GlobalScope

结构化并发Structured Concurrency

### 创建协程（作用域）

1. 

## 协程上下文CoroutineContext

是一个类似于Map的集合，元素类型是Element，每个Element有自己的key。

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

## 协程的取消机制

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

## SharedFlow

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

# 泛型

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

# 注解

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

