# Kotlin基础

# Kotlin进阶

## 反射 Reflection

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

## CoroutineStart

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


## 协程的取消机制

- 协程取消的原理
		1. 协程作用域内抛出CancellationException
		2. 协程检测到这个异常之后主动结束执行
	
	- CancellationException
		不是普通的异常，如果协程因CancellationException而结束，应该当做协程正常终止。为了让CancellationException能正常终止协程，不应该在代码里捕获这个异常，而是继续抛出这个异常，能让上层去进行后续操作。如果反复捕获并忽略这个异常，会产生僵尸协程。

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

- buffer()
	创建缓冲区，允许消费者来不及消费时放入缓冲区。
- conflate()
	处理旧数据之前，如果有新数据到达，会丢弃旧数据。
	buffer可以缓存数据，conflate会丢弃数据，如果同时设置了buffer()和conflate()，是否会丢弃缓冲区的数据？
	- 这两个操作符语义上是冲突的，以操作符顺序为准，后一个会覆盖前一个的行为。
- collectLatest()
	处理数据时，如果有新数据到达，会取消当前正在进行的处理，进行处理下一个。
	conflate()作用是丢弃中间值，只处理最新一个，

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

