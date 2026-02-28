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

# 协程

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

    

# Flow

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

QA

