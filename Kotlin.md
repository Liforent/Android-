# Kotlin基础

# Kotlin进阶

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

