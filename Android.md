# Android底层

## APK构建流程

### 编译器

- javac，把.java文件编译成JVM能读懂的.class字节码文件

- kotlinc，功能和javac类似，对kotlin的语法做了优化，能把.kt文件也翻译成.class文件。

- aapt2，Android Asset PackgingPackging Tool2，安卓资源打包工具。

  把res目录下的布局、图片、字符串等资源编译成二进制格式，生成resources.arsc文件；还会处理AndroidManifest.xml，给它加上编译后的信息。

- llvm/clang，c/c++代码编译，如果你的项目里用到了 NDK（比如调用 C 语言写的算法库），就需要它把.c/.cpp文件编译成.so动态库。

### 打包工具

- dx工具

  把所有.class文件转换成Android虚拟机能识别的.dex文件。为什么要转？因为 JVM 的.class文件在手机上运行效率太低，.dex文件是专门为移动设备优化的，体积更小、加载更快。

- apkbuilder

  把.dex文件、resources.arsc、AndroidManifest.xml、图片等资源打包成一个未签名的unsigned.apk文件。这一步就像把所有零件装进一个 “半成品盒子” 里。

### 签名工具

- jarsigner

  给未签名的 APK 文件签名，生成signed.apk。签名的作用是什么？一是证明 APP 的身份，防止别人冒充你的 APP；二是保证 APP 的完整性，防止代码被篡改。没有签名的 APK，是无法安装到手机上的（除非手机开启了 “未知来源” 或处于调试模式）。

### 构建系统

- Gradle，整个编译流程的 “总指挥”，负责调用上面所有工具，按顺序执行编译、打包、签名等步骤。你在build.gradle文件里写的配置（比如编译版本、依赖库、签名信息），最终都是由 Gradle 来解析和执行的。

- AGP（Android gradle plugin）

  Gradle 的 “Android 专属插件”，因为 Gradle 本身是一个通用的构建工具（可以用来构建 Java、C++ 项目），AGP 则专门为 Android 项目定制了编译逻辑，比如处理资源、生成多渠道 APK 等。

### 编译构建流程

1. aapt2对res目录下的资源进行预处理

   - 资源编译

     把 XML 布局（比如activity_main.xml）、图片（ic_launcher.png）、字符串（strings.xml）等资源转换成二进制格式。为什么要转成二进制？因为 XML 是文本格式，解析速度慢，二进制格式能让 APP 启动时加载资源更快。

   - 生成R.java文件

     aapt2 会扫描所有资源，给每个资源分配一个唯一的 ID（比如R.layout.activity_main、R.drawable.ic_launcher），然后生成R.java文件。你在代码里引用资源时，其实就是在使用这些 ID，编译器会通过R.java找到对应的资源。

2. javac/kotlinc 编译源代码

   编译器会把你的 Java/Kotlin 代码编译成.class文件（R.java文件也会被一起编译，代码里引用了R文件）

3. 处理依赖库，合并所有.class文件

4. dx工具把所有.class文件转换成.dex文件

   - 去除冗余class信息，减少文件体积
   - 把多个.class合并成一个或者多个.class文件

5. Apkbuilder打包成未签名APK

6. jarsigner签名

7. zipalign工具优化APK。

   它的作用是把 APK 文件中的资源按 4 字节对齐

### 编译加速技巧

。。。

### Android编译底层原理

- ART与Dalvik虚拟的区别
- 基于栈和基于寄存器的区别
- appt2的资源ID为什么是32位整数？
- gradle增量构建原理，为什么修改一行代码可以不用重新编译整个项目？

## Binder机制



1. Linux系统有哪些IPC方式?

2. Android中有哪些IPC方式?(回答出6种)各种IPC方式的对比以及适用场景

3. 传统IPC方式有哪些缺点?(Binder机制有哪些优点?)

4. 说一下Binder通信模型

5. 说一下Binder进行数据传输的具体过程

6. Activity之间传输数据,除了使用Intent,还能如何传输?

## Handler机制

### Message

同步消息和异步消息

同步消息屏障

### MessageQueue

### Looper

### Handler

### Q&A

1. handler机制简介
2. 介绍一下4个核心类和他们的作用
3. 可以直接new Message()吗
4. message的分类
5. Message池的作用
6. 介绍一下MessageQueue存取Message的流程
7. Handler有哪些作用?
8. Handler,Looper,Thread,MessageQueue的对应关系
9. 为什么子线程不能直接toast而主线程可以?
10. 主线程什么时候初始化了Looper?
11. Looper死循环为什么不会导致应用卡死？
12. 主线程的死循环如何处理其他事务?
13. 主线程的死循环是不是特别消耗CPU资源？
14. 使用Handler的postDelay后消息队列会有什么变化？
15. MessageQueue使用的底层数据结构是什么?队列,链表有什么区别?
16. 什么是IdleHandler?什么时候触发?怎么使用?有哪些使用场景?

#### 子线程可以直接创建Handler吗？主线程为什么可以直接使用Handler而不用创建Looper？

~~~kotlin
要先创建子线程的Looper。
public Handler() {
  mLooper = Looper.myLooper();
  if (mLooper == null) {
    throw new RuntimeException("Can't create handler inside thread that has not called Looper.prepare()")
  }
}
主线程在ActivityThread里调用了prepareMainLooper()方法来创建Looper了。
~~~



## 数据传输和存储

### SharedPreference

一种 **轻量级存储方式**，用于保存简单的键值对数据.通过 `synchronized` 保证线程安全（但跨进程不安全）。

#### SP缺点

	1. 不适合存储大数据,XML解析性能差,数据量大的时候IO开销高.
	1. 跨进程不安全
	1. 不支持复杂数据接口,无法直接存储对象或列表(需要序列化)
	1. commit()同步写入可能导致ANR
	1. apply()异步提交可能丢失数据(Crash的时候)
	1. 无事务支持(无法回滚)

#### commit()和apply()区别?

### MMKV

1. 说一下Android常用的数据持久化方法

   - Sqlite

     有哪些操作sqlite的框架

   - SharedPreference

     1. 如何存储的?如何加载的?

     2. 能跨进程吗?

     3. 还有其他什么不足?

     4. commit()和apply()有什么区别?

        apply无返回值,先提交到内存,再异步提交到磁盘.

        commit是同步提交到磁盘,效率低.

     5. 如何保存对象?

   - DataStore

   - MMKV

   - 什么是Room,如何使用

2. 什么是序列化?Android中有哪些序列化的方式?有什么区别?

3. FastJson Gson Jackson对比

# 四大组件

## Activity

### 生命周期

onCreate()，onStart()，onResume()，onPause()，onStop()，onDestory()

1. 生命周期

2. 切换横竖屏生命周期变化

3. android:configChanges其他属性的作用?

4. onPause,onDestroy何时触发,一定会触发吗?

5. onNewIntent什么时候调用?

6. 启动模式和各种flags

7. AMS中如何管理Activity信息的?

   ActivityRecord,TaskRecord,ActivityTask,ActivitySupervisor

### 启动模式

启动模式由launchMode和Intent flag两者共同决定。

主要是单个任务栈包含的Activity，可能是来自于不同的应用。单应用也可以包含多个任务栈，返回栈包含的多个任务栈之间也可以进行顺序切换、甚至任务栈中的 Activity 也可以被迁移到另外一个任务栈、Intent flag 可以多个组合使用。有些启动模式可通过 launchMode 来定义，但不能通过 Intent flag 定义，同样，有些启动模式可通过 Intent flag 定义，却不能在 launchMode 中定义。两者互相补充，但不能完全互相替代，且 Intent flag 的优先级会更高一些。

#### LaunchModel

1. Standard

2. SingleTop

   栈顶复用。如果当前任务栈已存在目标Activity的实例，则会调用其onNewIntent方法将Intent转送给该实例并复用，否则会新建。

3. SingleTask

   如果系统当前不包含目标 Activity 的目标任务栈，那么系统就会先创建出目标任务栈，然后实例化目标 Activity 使之成为任务栈的根 Activity。如果系统当前包含目标任务栈，且该任务栈中已存在该目标 Activity 的实例，则系统会通过调用其 `onNewIntent()` 方法将 Intent 转送给该现有实例，而不会创建新实例，并同时弹出该目标 Activity 之上的所有其它实例，使目标 Activity 成为栈顶。如果系统当前包含目标任务栈，但该任务栈不包含目标 Activity 实例，则会实例化目标 Activity 并将其入栈。因此，系统全局一次只能有一个目标 Activity 实例存在。

4. SingleInstance

   通过 singleInstance 启动的 Activity 会独占一个任务栈，系统不会将其和其它 Activity 放置到同个任务栈中，由该 Activity 启动的任何 Activity 都会在其它的任务栈中打开

#### IntentFlag

在启动 Activity 时，我们可以通过在传送给 `startActivity(Intent)` 方法的 Intent 中设置多个相应的 flag 来修改 Activity 与其任务栈的默认关联，即 Intent flag 的优先级会比 launchMode 高。

- FLAG_ACTIVITY_NEW_TASK
- FLAG_ACTIVITY_SINGLE_TOP
- FLAG_ACTIVITY_CLEAR_TOP
- FLAG_ACTIVITY_CLEAR_TASK
- 。。。

## Service

1. 生命周期

2. Service是什么?它与Thread的区别?

3. IntentService

   JobIntentService,WorkManager

4. HandlerThread

5. 如何保证Service不被杀死?进程保活?

6. Android中有哪些创建线程的方法?AsynkTask有什么缺点?

## ContentProvider

## BroadcastReceiver

1. 广播的分类

   1. 静态动态
   2. 有序,无序
   3. 本地,全局

2. 为什么广播开销高?

3. 有哪些系统广播?

4. 使用广播有什么注意事项?(导致ANR)

5. 广播实现的原理

   AMS,观察者模式

## Fragment

### 生命周期

![image-20260306114537703](/Users/mac/Library/Application Support/typora-user-images/image-20260306114537703.png)![image-20260306114613155](/Users/mac/Library/Application Support/typora-user-images/image-20260306114613155.png)

onCreateView到onDestoryView()之间的生命周期都可能触发多次。

FragmentTransaction操作了回退栈 之后，Fragment生命周期会收到影响触发多次。***\*先后加载不同 Fragment。新加载的 Fragment 就会取代之前的 Fragment 切换到前台，旧的 Fragment 的视图 View 就会被销毁。如果加载新 Fragment 的操作有添加到\**回退栈**中，那么当用户点击返回键时旧的 Fragment 就会重新呈现到前台，此时就会重新走一遍 `onCreateView` 到 `onDestroyView` 方法之间的流程了。

Fragment**getViewLifecycleOwner()**只能在这个期间被调用，否则会直接抛出异常。

### Q&A

#### 下边代码会有什么问题？

~~~kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        supportFragmentManager
           .beginTransaction()
           .replace(R.id.container, MainFragment())
           .commit()
   }
// 配置更改时onCreate()方法可能会被多次调用，会出现多个MainFragment实例。
~~~



1. Fragment有哪些加载到Activity的方式
2. FragmentPagerAdapter和FragmentStatePagerAdapter的区别
3. 开启一个带Fragment的Activity的生命周期变化

## Context

## Intent

### PendingIntent

# View

## View绘制流程

### Window

抽象类,各种view都在它上边显示

### PhoneWindow

Window的唯一实现类,包含了一个DecorView对象,以及各种操作窗口的方法比如setTitle(),setBackgroundDrawable()等.

mWindow是在==Activity的attach方法里创建的==.每个Activity都有一个对应的window.

```kotlin 
//Activity.java
final void attach(...) {
  //伪代码
  mWindow = new PhoneWindow(...);
	mWindow.setCallback();
  mWindow.setUiOptions();
  //窗口管理类
  mWindow.setWindowManager();
  mWindow.setColorMode();
  ...
}
```

### WindowManager

WindowManager是Activity管理Window的管理类.实际上管理的是Window里的DecorView.

两者是在handleResumeActivity()方法里关联在一起的.

WindowManagerImpl将DecorView作为根布局加入到了PhoneWindow中.

WindowManagerImpl并没有直接操作View的相关方法,而是交给了WindowManagerGlobal,这是一个单例类,也就是一个进程中最多仅有一个.

WindowManagerGlobal将decorView添加到了ViewRootImpl中.通过ViewRootImpl对其内部的view进行管理.

### ViewRoot

主要作用是连接WMS和DecorView.

ViewRootImpl是WindowManagerGlobal工作的实际实现者.

1. 与WMS通讯,调整窗口大小和布局
2. 向DecorView派发输入事件

### DecorView

所有Activity的根View,是一个FrameLayout,decoreView中包含ActionBar和ContentParent,也就是setContentView()传入的布局

全局变灰可以考虑监听每个Activity获取到DecorView的时候,把DecorView置灰.

### setContentView()

api29之前是调用PhoneWindow.setContentView()

api29之后是调用AppCompatDelegateImpl.setContentView()

本质上都是先根据窗口风格为该窗口选择不同的布局文件,然后设置contentParent(R.id.content),因此requestFeature()必须在setContentView()之前.

```java
//AppCompatActivity.java
public void setContentView(View view) {
  initViewTreeOwners();
  getDelegate().setContentView(view);
}

//AppCompatDelegateImp.java
public void setContentView(View view) {
  //创建decorview,最后
  ensureSubDecor();
  //这里可以看到setContentView就是把View添加了到了mSubDecor的R.id.content里
  ViewGroup contentParent = mSubDecor.findViewById(android.R.id.content);
  contentParent.removeAllViews();
  contentParent.addView(v);
  mAppCompatWindowCallback.getWrapped().onContentChanged();
}

//AppCompatDelegateImp.java
public ViewGroup creatSubDecor() {
  //获取AppCompatTheme里边设置的各种属性,
  TypedArray a = mContext.obtainStyledAttributes(R.styleable.AppCompatTheme);
  //根据主题风格来inflate subDecor
  subDecor = (ViewGroup) inflater.inflate(R.layout.xxx, null);
  ...
  mTitileView = (TextView) subDecor.findViewById(R.id.title);
  ...
  contentView.setId(android.R.id.content);
  ...
  //这里phoneWindow实际上也就调用mContentParent.addView(view, params)添加了subDecor
  mWindow.setContentView(subDecor);
  return subDecor;
}
```

### WindowInsets

插入物,常见的Insets有

- STATUS_BAR

- NAVIGATION_BAR

- IME

  前两者又被称为System bar

如果开发者绘制的内容出现在了系统UI区域内,就可能出现视觉与手势的冲突.开发者可以借助Insets把View从屏幕边缘向内移动到一个合适的位置.

### Choreographer

4.1之后新增的机制,可以接受Vsync信号,统一管理输入,动画,绘制等任务的执行时机.可以用来监控应用的帧率等.

- 常用类

  - CallbackQueue

  Choreographer添加的任务最后都被封装为CallbackRecord,以链表的形式保存在CallbackQueue中.mCallbackQueue是一个数组,长度为4,保存五个CallbackQueue链表,分别处理输入,动画,遍历绘制等任务.

  - FrameDisplayEventReceiver

  用于接受Vsync事件,并将事件提交给looper.

  - FrameHanlder

  用于处理主线程关于Vsync的事件.执行异步消息.

- 创建

  - ViewRootImpl的构造方法里创建了mChoreographer,类似于Looper,mChoreographer也是ThreadLocal变量,线程私有的.

  - Choreographer的构造方法

    ```java
    private Choreographer(Looper looper,int vsynSource) {
      mLooper = looper;
      mHandler = new FrameHandler(looper);
      // 用于接受Vsync信号
      mDisplayEventReceiver = USE_VSYNC ? new FrameDisplayEventReceiver(looper,vysncSource) : null;
      //上一帧绘制时间点
      mLastFrameTimeNanos = Long.MIN_VALUE;
      //一般都是16.67ms
      mFrameIntervalsNanos = (long)(1000000000 / getRefreshRate());
      //
      mCallbackQueues = new CallbackQueue[CALLBACK_LAST + 1];
      for (int i=0; i<= CALLBACK_LAST; i++) {
        mCallbackQueues[i] = new CallbackQueue();
      }
    }
    ```

- Vsync回调流程

  Vsync信号由SurfaceFlinger中创建HWC触发,唤醒DispSyncThread线程,再到EventThread线程,然后再通过BitTube直接传递到目标进程所对应的目标线程,执行handleEvent方法.

  ```Java
  // FrameDispalyEventReceiver
  private final class FrameDispalyEventReceiver extends DisplayEventReceiver implements Runnable {
    ...
    @Override 
    public void onVsync(long timestampNanos, int buildInDispalyId, int frame) 	{
      ...
      //收到Vsync信号后,由FrameHanlder发送消息,消息的callback为 FrameDispalyEventReceiver自身.
      Message msg = Message.obtain(mHandler, this);  
      msg.setAsynchronous(true);
      mHandler.sendMessageAtTime(...)  
    }
    
    @Override
    public void run() {
      mHavePendingVsync = false;
      doFrame(mTimestampNanos, mFrame);
    }
      
  }
  // doFrame里边最终会请求下次vsync信息处理
  ```

### 从Activity启动到View绘制流程

1. 我们从activity.onResume()开始讲述

2. 添加window,会调用ViewRootImpl.setView()

3. setView()会调用requestLayout()请求布局

4. 接着会走到ViewRootImpl的scheduleTraversals(),再接着是performTraversals().

   所有的UI变化都是走到scheduleTraversals().包括requestLayout(),ValueAnimator.start()View.invalidate()等.

   ```Java
   void scheduleTraversals() {
     ...
     //添加同步消息屏障  
     mTraversalBarrier = mhandelr.getLooper().getQueue().postSyncBarrier(); 
     // 发起一个callBack,里边的runnbale里的内容就是doTraversal()
     mChoreographer.postCallback(
     	Chreographer.CALLBACK_TRAVERSAL,mTraversalRunnable,null);
     )
     if (!mUnbufferedInputDispatch) {
                   scheduleConsumeBatchedInput();
               }
               notifyRendererOfFramePending();
               pokeDrawLockIfNeeded();
   }
   
   //也就是scheduleTraversals()不是立刻执行performTraversals(),而是向chreographer发送了一个runnable.
   ```

5. Chreographer处理ViewRootImpl的doTraversal()请求

   1. doTraversal被封装成了一个TraversalRunnable,然后调用Chreographer.postCallback()
   2. callback被存放在chreographer的CallbackQueue里.
   3. 当native层由收到vsync信号后,Chreograper的FrameDisplayEventReceiver会回调onVsync方法
   4. FrameHandler向主线程发送异步消息,msg的callback为FrameDisplayEventReceiver本身
   5. ViewRootImpl的doTraversal方法里设置了同步消息屏障,因此主线程会优先处理FrameDisplayEventReceiver这个异步消息.
   6. FrameDisplayEventReceiver的run方法里调用了Chreographer的doFrame()
   7. doFrame()依次执行了5个doCallbacks()方法,这些callBacks就是在存放Chreographer的CallBackQueue里,其中就有ViewRootImp中的TraversalRunnable.doFrame()里最终还会请求下次vsync信号到了的处理.
   8. TraversalRunnable的run方法会调用doTraversal()
   9. doTraversal()里最终调用performTraversal()方法进入绘制流程

### 绘制入口

ViewRootImpl的performTranversals()开始绘制,依次遍历调用performMeasure(),performLayout(),performDraw()

### 绘制流程

#### measure流程

测量View的宽高,调用View.measure()方法递归测量,深度优先,最后返回各个子view的测量值.某些情况下(比如子view设置了match_parent)需要多次measure才能确定View的宽高.所以measure过程后得到的宽高不一定准确.[(如何获取View的宽高?)](#getViewHeight).

##### MeasureSpec

- MeasureSpec概括了从父布局传递给子布局的要求,也就是测量说明书,从Window开始,到DecorView,到ViewGroup,每一层结合自己的LayoutParams,生成自己的测量说明书,交给下一层.从而实现了父view对子view的尺寸的限制.

- 32位int型.由mode+size组成(2+30位)

- mode

  - EXACTLY

    精确值模式,也就是父布局设定了layout_height,layout_width属性,子view的最大值会被限定在这个值之内.

  - AT_MOST

    最大值模式,也就是父布局设置了wrap_content

  - UNSPECIFIED

    未指定的,父布局没有对子view做限制很少用

- MeasureSpec的使用场景

  - onMeasure()中要用父view给自己生成的MeasureSpec,测量自己的尺寸

  - onMeasure()中测量子View的时候,生成childMeasureSpec提供给子view,让子view去测量尺寸.

    1. onMeasure()-->

    2. measureChildWidthWithMargins(child,widthMeasureSpec..)-->

    3. child.measure(childWidthMeasureSpec)

       这里的childWidthMeasureSpec要通过getChildMeasureSpec()来生成

- <a name="getChildMeasureSpec">计算childMeasureSpec</a>

  ```Kotlin
  //根据父view剩余大小,MeasureSpec的值,和子view的layoutParams计算
  int getChildMeasureSpec(int spec,int padding,int childDimension){
    switch(childDimesion){
      case 具体数值:
        childSpec.mode = EXACTLY;
        childSpec.size = min(数值,parentLeftSize);
        break;
      case WRAP_CONTENT:
        childSpec.mode = AT_MOST;
        childSpec.size = less than parentLeftSize;
        break;
      case MATCH_PARENT:
        childSpec.mode = parentSpec.mode;
        childSpec.size = if(parentSpec.mode == EXACTLY) parentLeftSize
          else if(parentSpec.mode == AT_MOST) less than parentLeftSize
      	break; 
    }
  }
  ```

  ![img](https://img-blog.csdnimg.cn/img_convert/4e78b5c10216902e04c7c2c98bc2f0a3.png)

- 总结

  - 子view为具体数值,那么子view的mode是exactly和具体数字


  - 子view为match_parent,再看父view的mode,如果是exactly,那么子view的mode是exactly,size是父容器剩余的size

  - 其他情况子view的mode都是AT_MOST,size都是不超过父容器的剩余空间.

- Measure流程分析

  1. ViewRootImpl.performTranversals()

  2. ViewRootImpl.performMeasure()

     ViewRootImpl.performMeasure获取到了DecorView和它的MeasureSpec之后,开始performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec)

  3. View.measure()

     先从decorView开始measure.

     ```Java
     //1.这是一个final方法
     //2.参数是父view的measueSpec,调用这个方法来测量出view的大小.
     //3.里边调用了onMeasure()来进行实际的测量工作
     public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
       ...
       //是否需要强制layout,比如调用View.requestLayout()会在mPrivateFlags加入PFLAG_FORCE_LAYOUT标记来强制layout 
       final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
       ...
       //需要layout的条件  
      	final boolean needsLayout = specChanged && (sAlwaysRemeasureExactly || !isSpecExactly || !matchesSpecSize); 
       
       if (forceLayout || needsLayout) {
         //满足条件,并且缓存里没有layout数据或者忽略缓存的时候onMeasure()
          onMeasure(widthMeasureSpec, heightMeasureSpec);
         ...
       }  
       // 如果自定义view重写了onMeasure()之后,但是没有调用setMeasureDimension(),会抛出异常.
       if (mPrivateFlags & PFLAG_MEASURED_DIMENSION_SET != PFLAG_MEASURED_DIMENSION_SET) {
         throw new IllegalStateException(...)
       }
       mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
       mOldWidthMeasureSpec = widthMeasureSpec;
       mOldHeightMeasureSpec = heightMeasureSpec;
        // 保存到缓存中
       mMeasureCache.put(key, ((long) mMeasuredWidth) << 32 |
           (long) mMeasuredHeight & 0xffffffffL); // suppress sign extension
     }
     
     //measure流程总结
     
     1. 判断是否强制layout或者needLayout,是则onMeasure
     2. 检测是否setMeasureDimension
     3. 保存到缓存
     ```

  4. onMeasure()

     - ViewGroup中没有重写onMeasure(),而是交给了其子类去重写.(不同ViewGroup有不同的测量规则.)

     - DecorView继承自FrameLayout.View绘制里最先触发的是FrameLayout里的onMeasure()

     - ViewGroup在测量的时候,如果子view设置了match_parent,那么需要测量两次.第一次遍历子view,遇到match_parent就给他大小赋0或者其他固定值,最终测量出父view大小. 知道父view大小了之后,再次测量,才能得出match_parent的子view的大小.

     - RelativeLayout的子view有相互依赖关系,也需要多次测量,第一次测量进行布局确定子view顺序,第二次测出大小.

       LinearLayout如果使用了weight的情况下,也需要多次测量.

     - 这种多次测量是指数级的,布局层级过深后会影响性能.而Compose虽然一直在套娃,但是没有布局嵌套问题,Compose只允许测量一次.

       ```java 
       //View的onMeasure()
       protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
         setMeasureDimesion(getDefaultSize(getSuggestionMinimumWidth(),widthMeasureSpec), getDefaultSize(getSuggestionMinimumHegiht(),heightMeasureSpec));
       }
       
       
       
       //以FrameLayout.onMeasure为例简单分析一下ViewGroup的onMeasure()流程.
       protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
         //1.遍历测量子View,按照需要更新maxWidth和maxHeight.这里就是传measureSpec给子view,再调用子view.measure().
         for (...) {
           ...
           measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);
         }
         //2.确定父View的宽高
         setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                       resolveSizeAndState(maxHeight, heightMeasureSpec,
                               childState << MEASURED_HEIGHT_STATE_SHIFT));
         //3.如果子view的宽高是MATCH_PARENT,需要等确定父View的宽高确定后,再重新计算这些子view的MeasureSpec,再重新计算这些子View的宽高.
       }
       ```

#### layout流程

### LayoutInflater

布局加载有两种方式

1. 填充View树--addView(view)

2. 加载解析xml文件--LayoutInflater.inflate()

   ```java 
   // LayoutInflater.java
   public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
     ...
     // 1.解析XML,最终会调用loadXmlResourceParser()
     XmlResourceParser parser = res.getLayout(resource);
     try {
     // 2.填充View树.这个方法是根据XmlPullParser,用createViewFromTag()生成view
         return inflate(parser, root, attachToRoot);
     } finally {
         parser.close();
     }
   }
   
   //ResourcesImpl.java 
   XmlResourceParser loadXmlResourceParser(@NonNull String file,@AnyRes int id,int assetCookie, @NonNull String type) throws NotFoundException {
     //一堆代码,这个方法的作用就是将xml文件读取到内存中,并进行一些数据解析和封装.这个方法本质是一个IO操作,比较耗费性能.
     ...
   }
   
   //这个方法主要是利用反射生成view
   View parent,String name,Context context,AttributeSet attrs,ignoreThemeAttr) {
     ...
     //1.对view,bink等特殊标签做处理.
     //2.进行对Factory,Factory2的设置判断.(可以同过这两个直接生成view)
     //	这里可以通过Factory2()来实现全局换肤换字体等功能.
     //3.如果没有设置2,那么就用LayoutInflater默认方式生成View(通过反射)
   }
   ```

## 事件分发机制

### 本质

将点击事件(Touch事件)传递到某个具体的View上然后处理,用户触碰屏幕时,会产生touch事件,相关细节会被封装成MotionEvent对象

### MotionEvent对象

DOWN,MOVE,UP,CANCEL

### 事件分发顺序

Activity->ViewGroup->View

### 核心方法

1. dispatchTouchEvent,分发,默认传递到下一层.
2. onInterceptTouchEvent,拦截,ViewGroup中才有,默认不拦截.
3. onTouchEvent

### 流程详解

#### 伪代码

```Kotlin
fun dispatchTouchEvent(ev: MotionEvent): Boolean{
 return if (isViewGroup) {
          if (onInterceptTouchEvent(ev)) {
            onTouchEvent(ev)
          } else {
            child.dispatchTouchEvent(ev)
          } 
        } else {
          onTouchEvent(ev)
        }
}
```

#### dispatchTouchEvent()

- 默认会传递到下一层,如果当前是ViewGroup,还可以拦截了直接交给本层的onTouchEvent处理,默认是不拦截.
- 如果下一层消费了,那么直接返回false
- 如果下一层没有消费,那么返回本层的onTouchEvent处理结果
- dispatch结果
  - 返回true表示被当前视图消费了.
  - 返回super.dispatchTouchEvent()表示继续分发该事件.
  - 返回false表示交给父View的onTouchEvent()处理.

#### onInterceptTouchEvent()

- 只在ViewGroup中,如果拦截了交给本层ViewGroup的onToucheEvent处理,默认是不拦截,传递到下一层.
- 要拦截时,可以重写onInterceptTouchEvent方法,也可以在子view中调用requestDisallowInterceptTouchEvent(boolean)方法,让父view拦截或不拦截.

### onTouchEvent() 

- 如果返回true,那么最终消费了事件
- 如果返回false,那么没有消费事件,事件传递给上一层的onTouchEvent

### onTouch,onTouchEvent,onClick

```Kotlin
//伪代码,优先级是onTouch,onTouchEvent,onClick
fun consumeEvent(ev:MotionEvent){
  if(setOnTouchEventListener){
    if(!onTouch()){
      onTouchEvent()
    }
  }else{
    onTouchEvent()
  }
  if(setOnclickListener){
    onClick()
  }
}
事件进入到view.dispatchTouchEvent的时候,会先判断是否设置了onTouchListener,如果设置了onTouchListener,那么会调用onTouch方法,如果onTouch()返回false,也就是没有消费,onTouchEvent才会执行.onClick()方法只有设置了onClickListener才会被调用.
```

### 滑动冲突

#### 外部拦截

```Kotlin
//外部拦截只要重写拦截方法,根据需要拦截事件往下分发就可以了,
public boolean onInterceptTouchEvent(MotionEvent ev){
  boolean intercepted = false;
  boolean parentCanIntercept;
  switch (ev.getActionMasked()){
    case MotionEvent.ACTION_DOWN:
      intercepted = fasle;
      break;
    case MotionEvent.ACTION_MOVE:
      if(parentCanIntercept) {
        intercepted = true;
      }else{
        intercepted = false;
      }
      break;
    case MotionEvent.ACTION_UP:
      intercepted = false;
      break;
  }
  return intercepted;
}
```

#### 内部拦截

```Kotlin
//父view中,ACTION_DOWN不能拦截,否则子view接收不到后续事件了,其他事件要允许拦截,因为要配合子view处理
public boolean onInterceptTouchEvent(MotionEvent ev){
  if(ev.getActionMask() == MotionEvent.ACTION_DOWN){
    return false;
  }else{
    return true;
  }
}

//子view中,ACTION_MOVE的时候,根据需要让父view去拦截
public boolean dispatchTouchEvent(MotionEvent ev){
  boolean parentCanIntercept;
  switch (ev.getActionMask()){
    case MotionEvent.ACTION_DOWN:
      getParent().requestDisallowInterceptTouchEvent(true);
      break;
    case MotionEvent.ACTION_MOVE:
      if(parentCanIntercept){
         getParent().requestDisallowInterceptTouchEvent(false);
      }
      break;
    case MotionEvent.ACTION_UP:
      break;
  }
  return super.dispatchTouchEvent(ev);
}
```



RecyclerView

- scrollToPosition

  ```kotlin
  如果item已经在可见范围内了,还会滑动吗? 答案是不会.需要指明对齐方式
  layoutManager?.let {
          val smoothScroller = LinearSmoothScroller(context, snapPreference)
          smoothScroller.targetPosition = position
          it.startSmoothScroll(smoothScroller)
      }
  ```



## View滑动控制

### 如何实现View的滑动

View#scrollBy, View#scrollTo, View#onScrollChanged

### Scroll手势

GestureDetecor可以识别手势，或者自己处理onTouchEvent

### Fling手势

Fling是快速滑动，对Fling来说最重要的是速度。GestureDetecor识别Fling的逻辑是判断速度是否超过了阈值，超过了便会回调。

### VelocityTracker

这个是GestureDetecor里边帮忙处理一些速度逻辑的对象

### Scroller

对滑动的封装，是一个滚动计算器，作用在于实现平稳滑动，不让View的滚动出现跳跃。

### 滑动冲突处理

## 常用控件

## RecyclerView

1. 相关类的作用,以及他们的常用方法

   - ViewFlinger

   - ItemDecoration

   - ItemAnimator

   - Recycler

   - SnapHelper

   - SmoothScroller

   - ViewHolder

     - onCreateViewHolder()
     - onBindViewHolder()

   - Adpater\<VH:ViewHolder>

     - getItemViewType()

     - onCreateViewHolder()

     - onViewAttachedToWindow()

     - onBindViewHolder()

     - onViewDetachedFromWindow()

     - onViewRecycled()

     - 刷新数据的方法以及区别,原理

       - notifyDataSetChanged()

       - notifyItemInserted(position) //removed,changed

       - 观察者模式.

         recyclerView.setAdapter()的时候注册了AdpaterDataObserver

2. 四级缓存机制

   哪四级,默认个数多少,缓存流程,缓存的是ViewHolder

   scrap

   cache

   ViewCacheExtension

   RecycledViewPool

3. 一些常用方法的作用和注意事项

   1. 获取item

      - recyclerView.getChildAt(position)

        这个方法有啥注意事项?

   2. 获取positioin

      - recyclerView.getLayoutPosition()

      - getAdapterPosition()

        两个方法有啥区别?获取position有什么注意事项?最好用哪个方法?

   3. 滑动到指定位置

      显示在何处,当前item已经处于屏幕内了还能滑吗?

      1. recyclerView或者layoutManager里的scrollToPosition(),smoothScrollToPosition()
      2. LinearLayoutManager里的scrollToPositionWithOffset()
      3. recyclerView的scrollBy()

4. 一些常用功能如何实现

   1. 左滑删除item
   2. item出现的动画

## ShapeView

轮子哥的,方便设置圆角的view

- 外阴影实现

  外层嵌套一个View,从外层布局设置阴影,再设置paidding

## TextView

- URL识别
- 长按复制
- 富文本展示

### SpannableString

可以更改TextView中的部分文本颜色，添加点击事件等。

### Baseline

TextView文本的底部，如何实现两个TextView内容和大小不一样，但是底部对齐？（比如“￥40”这样的价格文本）

这时候就可以通过ConstraintLayout的layout_constraintBaseline_toBaselineOf来实现文本底部对齐。

## Bitmap

### BitmapConfig

- ARGB_4444, 

  2个字节，有透明度

- ARGB_8888

  4个字节。BitmapFactory的默认编码格式。

- RGB_565

  2个字节，没透明度



### Bitmap内存计算

内存 =  宽 * 高 * 位数 （比如 1920px * 1080px * 4(ARGB_8888编码格式)）= 8294400 byte = 7.91MB

### 与drawable文件夹的关系

将上述图片从drawable-xxhdpi移动到drawable-xhdpi，会发生什么变化？

假设设备的dpi是480，如果xxhdpi目录下没有目标图片，而是从xhdpi目录下拿到的，那么系统会将图片放大1.5倍（480/320），相应的内存会增加2.25倍。

### 与ImageView宽高的关系

默认不会影响，设置到ImageView之前，肯定已经加载到内存了。但是如果是Glide等框架，会根据ImageView的宽高对图片进行缩放，因此内存会有变化，如果图片是个大长图，导致了图片被放大了很多倍，就有可能导致OOM。

### Bitmap加载优化

1. 压缩图片

   图片大小1920 * 1080，假设加载到540*540的ImagView里，如果按照原图加载肯定会浪费很大内存，因此可以对图片进行压缩。我们在正式加载 Bitmap 前要先获取到 Bitmap 的实际宽高大小，这可以通过 inJustDecodeBounds 属性来实现。

2. 调整图片编码格式

   使用 RGB_565替换ARGB_8888。

## ConstraintLayout

- goneMargin()

  如果布局被设置了Gone时生效。

- layout_constraintBaseline_toBaselineOf

  对齐两个TextView的文字底部

- 环状排列

  骚气但是不常用

  layout_constraintCircle

  layout_constraintCircleRadius

  layout_constraintCircleAngle

- bias 偏向比例

  layout_constraintHorizontal_bias = "0.2" 偏向左边20%

- ratio 自身比例

  自身宽高比

  app:layout_constraintDimensionRatio="16:9"   <!-- 宽高比16:9 -->

- 最大最小限制

  - max与min限制具体数值

  - layout_constraintWidth_percent

    layout_constraintHeight_percent

    设置默认比例

- chains

  - Style

    spread

    inside 

    packed

  - Weighted 

    给子View加上layout_constraintHorizontal_weight后，就会按比例分配，这个与LinearLayout的layoutWeight用法是一样的。

- groups

  

  

  ​	

# JetPack

## Lifecycle



## LiveData

生命周期安全的可观察数据类。

- 一个Observer对象只能和一个Lifecycle对象绑定。

- 一个Observer对象不能同时使用observe和observeForever方法。

  两个方法的区别就是是否绑定了生命周期。observeForever需要手动移除observer避免内存泄漏和NPE

值的更新

- setValue()

  ```java
      assertMainThread("setValue"); //只能在主线程纸执行
  ```

- postValue()

  不限定调用者所在线程，存在多线程竞争的可能性的。`postValue` 方法将值回调的逻辑放到 Runnable 中再 Post 给 Handler，最终交由主线程来执行。在 `mPostValueRunnable` 被执行前，所有通过 `postValue` 方法传递的 value 都会被保存到变量 `mPendingData` 上，且只会保留最后一个，直到 `mPostValueRunnable` 被执行后 `mPendingData` 才会被重置，所以使用 `postValue` 方法在多线程同时调用或者单线程连续调用的情况下是存在丢值（外部的 Observer 只能接收到最新值）的可能性的。

### MutableLiveData

只是将 `setValue()` 和 `postValue()` 方法的访问权限提升为了 `public`，从而让外部可以直接调用这两个方法。

### MediatorLiveData

## ViewModel

### QA

下边代码会有什么问题？

~~~kotlin
1. 
class MainActivity: AppCompatActivity() {
    private val viewModel = MainViewModel()
}
ViewModel持有liveData等数据类，并且能在配置更改时仍然保存。也就是说ViewModel的生命周期比Activity长，它是存放于ViewModelStore里的，因此如果在Activity里直接实例化，会导致ViewModel的特性失效了
~~~



## CameraX

### CameraController

提供大多数 CameraX 核心功能,并且可自动处理相机初始化、用例管理、目标旋转、点按对焦、双指张合缩放等操作.

### CameraSelector

相机选择器. 

CameraSelector.DEFAULT_BACK_CAMERA  默认后摄像头

### ImageCapture

```kotlin
val imageCapture = ImageCapture.Builder()
	.setBufferFormat(ImageFormat)
	.setCaptureMode(ImageCapture.CAPTURE_MODE_ZERO_SHUTTER_LAG) 
```

#### BufferFormat/OutputFormat

输出图片的格式

- ImageFormat.JPEG

- ImageFormat.YUV_420_888  //这个格式Android所有设备都兼容.

  经测试,使用ImageFormat.JPEG,拍照时长900ms,ImageFormat.YUV_420_888能减少到500ms,最快能到300ms,拍摄速度快.



#### CaptureMode

- ImageCapture.CAPTURE_MODE_ZERO_SHUTTER_LAG // 零延迟快门,设置了也没什么用.
- ImageCapture.CAPTURE_MODE_MINIMIZE_LATENCY // 低延迟模式,默认,设置了没什么效果
- ImageCapture.CAPTURE_MODE_MAXIMIZE_QUALITY //高质量模式,

#### FlashMode

闪光灯模式.

- ImageCapture.FLASH_MODE_ON
- ImageCapture.FLASH_MODE_OFF  //默认
- ImageCapture.FLASH_MODE_AUTO  
- ImageCapture.FLASH_MODE_SCREEN   //这个不能直接用, a non-null `ScreenFlash` instance must also be set with `setScreenFlash`.

设置了FlashMode之后,会使拍照时长增加到900ms左右.不设置会减少200~300ms



## Hilt



### 使用

1. 导入hilt依赖
2. 申明插件(必须先导入依赖再申明插件)
3. 在app目录的Application里添加@HiltAndroidApp注解(注意不能是在base module里的application)

### 各地方注解

- 普通类

  1. @Inject constructor()在构造函数处注解,

  2. 使用 @Inject laterinite var xx:XX 来获取

- ViewModel里

  @HiltViewModel

  注解了VM之后,应当直接使用by viewmodel,不用@Inject



# 三方库和源码

## LeakCanary

内存泄露检测框架.

### 原理简述

核心原理是利用Refercence及ReferenceQueue来实现对对象是否被回收的监听.

### Reference和ReferenceQueue

Reference是四大引用父类,ReferenceQueue是一个单向队列,存储的是Reference对象.Reference对象可以关联一个对象和ReferenceQueue,JVM在回收该对象时,会将reference对象添加到ReferenceQueue里,根据这个原理,可以通过检测ReferenceQueue里是否存在reference对象,来判定是否发生了内存泄露.

### LeakCanary工作流程

1. 使用ContentProvider初始化.
2. 对相关对象的销毁进行监听.(Activity,Fragment,Fragment的View,ViewModel)
3. 收到销毁回调后,创建KeyedWeakReference和ReferenceQueue并关联.
4. 延迟5秒检查引用队列里是否有对象,如果没有,那么说明发生了内存泄露.
5. 通过dump heap获取hprof文件.
6. 通过Shark库解析hprof文件,获取泄露对象,并计算泄露对象到GC roots的最短路径.
7. 合并多个泄露路径并输出分析结果展示.

### 源码解析

## MMKV

Memory Mapped Key-Value，轻量级跨平台键值存储框架。

### 对比SharedPreference

1. 性能高，比SP快10到100倍。
2. 数据安全，基于CRC校验确保数据完整，支持加密。
3. 支持跨平台，多进程。
4. 更友好的API，简单易用支持多种数据类型。
5. 更好的内存管理，采用内存映射技术，减少IO操作。

### 核心原理：mmap（内存映射）机制。

传统文件需要通过系统调用读写数据，涉及用户态和内核态切换，效率低。而 mmap 将文件直接映射到进程的虚拟内存空间，使得文件操作像内存操作一样高效。

具体来说，MMKV 在初始化时会将存储文件映射到内存：

1. 通过mmap系统调用创建文件映射
2. 操作系统负责维护内存与磁盘文件的同步
3. 写入操作直接修改内存，由 OS 异步刷盘
4. 即使进程意外崩溃，OS 也能保证数据写入磁盘

这种机制带来了两个显著优势：一是避免了频繁的 IO 操作，大幅提升性能；二是提高了数据安全性，即使应用崩溃也不会丢失数据。

MMKV 对内存映射区域的大小进行了智能管理。初始映射大小为 4KB，当需要更多空间时，会自动扩容为当前大小的 1.5 倍（若超过则直接翻倍），并重新创建映射。这种动态扩容策略既保证了性能，又避免了内存浪费。

### 数据存储结构：高效的增量更新机制

MMKV 的文件结构设计充分考虑了性能和可靠性需求，主要包含两个文件：

- 主文件：存储实际的键值对数据

- CRC 文件：存储校验信息，确保数据完整性

主文件的结构如下：

- 前 4 个字节：记录存储数据的总大小

- 后续部分：一系列键值对，每个键值对由 "键长度 + 键 + 值长度 + 值" 组成

这种结构设计使得 MMKV 能够实现**增量更新**机制。与标准 protobuf 需要全量写入不同，MMKV 采用 append-only（只追加）的方式更新数据：当更新一个键值对时，新数据被直接追加到文件末尾，而不是覆盖旧数据。在读取时，只需用后面出现的值覆盖前面的值即可获得最新数据。

这种机制极大地提升了写入性能，特别是对于频繁更新的场景。不过，这也会导致文件中存在冗余数据，因此 MMKV 会在适当的时候（如文件大小达到阈值）进行数据重整，剔除冗余信息。

### 序列化方式：protobuf 的优化使用

MMKV 选择了 protobuf 作为数据序列化格式，而非 JSON 或 XML，主要看中了其以下优势：

- 二进制格式，体积小
- 解析速度快
- 跨平台支持好

但 MMKV 并没有直接使用标准的 protobuf，而是进行了针对性优化：

1. 自定义了更轻量的编码逻辑
2. 针对基本数据类型做了特殊处理
3. 结合内存映射实现高效读写

例如，对于 int 类型的存储，MMKV 会先计算 protobuf 编码所需的字节数，然后直接写入内存映射区域，避免了不必要的内存拷贝

### 内存与存储优化

1. **避免存储过大数据**：MMKV 适合存储小数据（如配置项、用户偏好），不适合存储大量二进制数据
2. **合理设置加密**：加密会带来约 10% 的性能损耗，仅对敏感数据使用
3. **及时清理无用数据**：定期调用trim()方法进行数据重整，剔除冗余
4. **多进程谨慎使用**：多进程模式会引入锁开销，非必要不使用

## OkHttp

## Retrofit

# 功能效果实现

## AI

- 人脸检测，文字识别，条码识别，人脸网格检测，图像标签，姿势检测 等 https://github.com/jenly1314/MLKit

## Banner

io.github.youth5201314:banner

## 布局圆角设置 ShapeView

com.github.getActivity:ShapeView

## 大图加载 SubsamplingScaleImageView

com.davemorrissey.labs:subsampling-scale-image-view-androidx

- 展示异常时没有兜底
  	出现一个线上图片 https://pxb7-oss-test.pxb7.com/pxb7-test/im/image/20251210/1765360040862.jpeg 在使用大图展示控件展示时，format不支持的问题，会在onImageLoadError处回调异常，直接使用ImageView.setImageUri(File.toUri())能直接展示了，但是缩放不了。

## 导航栏 DslTabLayout

com.github.angcyo.DslTablayout:TabLayout

## 动画库 Pag

com.tencent.tav:libpag

## 富文本加载 markwon

io.noties.markwon:core

## 工具类

- utilcodex 

  com.blankj:utilcodex

## 骨架屏 skeleton

com.ethanhua:skeleton

## 滑块 Seekbar

- com.github.Jay-Goo:RangeSeekBar 支持双向滑动

## Loading库 spinKit

com.github.ybq:Android-SpinKit

## 路由跳转 

- TheRouter

## 屏幕适配 AutoSize



##### AutoSize适配折叠屏原理

在每次屏幕尺寸发生改变时 AutoSize.autoConvertDensityOfGlobal(activity) 设置 Density 

并手动刷新所有的 UI 内容

[折叠屏、平行视窗、分屏及悬浮窗参考方案](https://blog.csdn.net/weixin_38702864/article/details/127071799?app_version=5.7.4&code=app_1562916241&csdn_share_tail={"type"%3A"blog"%2C"rType"%3A"article"%2C"rId"%3A"127071799"%2C"source"%3A"weixin_38702864"}&uLinkId=usr1mkqgl919blen&utm_source=app)

##### 列表中使用Glide加载图片特定行出现放大问题 

![img](https://ecnj5070ng8v.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTMxNzBmYTE5NjEyMzNhMmRjOTM4ZDMyNjVhOTdlNmNfYUptZGpWck9aVkJrS0hiY1J5aGRqTHJlSnlGV3RlV0tfVG9rZW46WkpyUGJuWHdsb1FmcTJ4bzJUSmM1d3hQbmNkXzE3NzIyNDkyMDM6MTc3MjI1MjgwM19WNA)

在item的getView()或onBindViewHolder中进行设置

由于GridView、RecyclerView等组件存在item复用机制，屏幕开合或旋转时，item宽高可能不正确。在getView()中加入AutoSize.autoConvertDensityOfGlobal(activity)手动设置density可解决此问题。

AutoSizeConfig.getInstance().setExcludeFontScale(true) 即可防止系统字体大小影响 app 的字体大小，即使你使用的是 sp 也可以奏效

##### AutoSize适配失效

今日头条方案唯一的不足就是修改后的 density 会在某些情况，或者在某些机型上被恢复成默认的 density，导致适配失效或者适配异常

原因无非就是 **修改后的 density 会在某些情况，或者在某些机型上被恢复成默认的 density**，你保证 View 绘制前，density 是正确值即可

重新进入 App 也需要调用 AutoSize.autoConvertDensity，比如在 onResume 中调用

你只要在当前页面的 view 被显示到屏幕上之前，把 density 修改成你期望的值就可以保证一定能完成适配，应用到框架中就是在合适的时机调用 AutoSize#autoConvertDensity() 手动修改 density，[#13 (comment)](https://github.com/JessYanCoding/AndroidAutoSize/issues/13#11) 中已经描述的非常清楚了，这个就是万能的解决方案，所有适配失效都可以通过这个方式解决，如果这个方式都不能解决，那基本很难解决了

onCreateView 中调用 AutoSize.autoConvertDensity() 即可，不需要实现 CustomAdapt，AutoSize.autoConvertDensity() 中填写你 manifest 中写的宽度

##### glide中加载图片density不一致导致bitmap大小不一致

在transform方法中调用了

Bitmap result = pool.get(toTransform.getWidth(), toTransform.getHeight(), Bitmap.Config.ARGB_8888); 方法后改变了bitmap的density,但是宽高并没有变化，所以我这里直接调用了bitmap的setdensity方法，设回正确的density。

result.setDensity(getResources().getDisplayMetrics().densityDpi);

修改 sDefaultDensity 的值就好了，可以通过反射：

```Plain
private static void setBitmapDefaultDensity(int dpi) {
    try {
        // 修改 Bitmap 中的静态成员 private static volatile int sDefaultDensity = -1;
        Method method = Bitmap.class.getDeclaredMethod("setDefaultDensity", int.class);
        method.setAccessible(true);
        method.invoke(null, dpi);
    } catch (Exception e) {
        Log.w(TAG, "setBitmapDefaultDensity()", e);
    }
}
```

##### 横竖屏状态获取

需要通过判断AppWindow的宽高比判断是横屏还是竖屏。

（1）宽:高 >= 1为横屏

（2）宽:高 < 1为竖屏

传递给应用的Configuration中的orientation会标识当前是横屏还是竖屏 。

（1）Configuration.ORIENTATION_LANDSCAPE为横屏

（2）Configuration.ORIENTATION_PORTRAIT为竖屏

```Plain
  fun isVivoFoldableDevice(): Boolean {
    try {
        val c = Class.forName("android.util.FtDeviceInfo")
        //phone、tablet 和 foldable
        val m: Method = c.getMethod("getDeviceType")
        val dType: Any = m.invoke(c)
        Log.d("fold", "getDeviceType=$dType")
        return "foldable" == dType
    } catch (e: Exception) {
        e.printStackTrace()
    }
    return false
}
  package com.pxb7.base.util

import androidx.core.util.Consumer
import androidx.window.java.layout.WindowInfoTrackerCallbackAdapter
import androidx.window.layout.FoldingFeature
import androidx.window.layout.WindowInfoTracker
import androidx.window.layout.WindowLayoutInfo
import com.pxb7.base.constants.FoldState
import timber.log.Timber

object FoldStateManager {
    /**
     * 窗口信息适配器
     */
    private val mWindowInfoAdapter by lazy {
        WindowInfoTrackerCallbackAdapter(WindowInfoTracker.getOrCreate(AppGlobal.sCurrentActivity))
    }

    /**
     * 折叠状态回调
     */
    private val mFoldBack by lazy {
        Consumer<WindowLayoutInfo> {
            val state = covertFoldState(it, isFoldSpecial())
            Timber.d("BaseActivity state $state, foldState $foldState")
            isFoldStateChanged = state != foldState
            if (isFoldStateChanged) {
                foldStateChangedBlock?.invoke()
            }
            foldState = state
        }
    }

    private var foldStateChangedBlock: (() -> Unit)? = null

    fun onStart(foldStateChangedBlock: (() -> Unit)? = null) {
        this.foldStateChangedBlock = foldStateChangedBlock
        mWindowInfoAdapter.addWindowLayoutInfoListener(
            AppGlobal.sCurrentActivity, Runnable::run, mFoldBack
        )
    }

    fun onStop() {
        mWindowInfoAdapter.removeWindowLayoutInfoListener(
            mFoldBack
        )
        foldStateChangedBlock = null
    }

    fun isFoldStateChanged(): Boolean {
        return isFoldStateChanged
    }

    //屏幕是否是折叠状态
    fun isFoldState(): Boolean {
        return foldState == FoldState.FOLD_NORMAL
    }

    /**
     * 折叠屏特殊模式
     */
    private fun isFoldSpecial(): Boolean {
        return false
    }

    //屏幕折叠状态
    @FoldState
    private var foldState = FoldState.FOLD_NORMAL

    //是否改变屏幕折叠状态
    private var isFoldStateChanged = false

    /**
     * @param windowLayoutInfo 当系统传递的折叠信息为空的时候，默认是折叠状态，即小屏状态
     * @param isSpecial 特殊模式，则会返回书本和桌面模式，默认不返回，只返回展开状态
     */
    private fun covertFoldState(
        windowLayoutInfo: WindowLayoutInfo?,
        isSpecial: Boolean = false
    ): Int {
        val displayFeatures = windowLayoutInfo?.displayFeatures
        var foldState = FoldState.FOLD_NORMAL
        //数据不为空的时候
        if (!displayFeatures.isNullOrEmpty()) {
            for (display in displayFeatures) {
                if (display is FoldingFeature) {
                    foldState =
                        (display.state == FoldingFeature.State.FLAT || display.state == FoldingFeature.State.HALF_OPENED).flatHandle(
                            isSpecial, display
                        )
                    break
                }
            }
        }
        return foldState
    }

    private fun Boolean.flatHandle(isSpecial: Boolean, display: FoldingFeature): Int {
        return if (this) {
            isSpecial.specialHandle(display)
        } else {
            FoldState.FOLD_NORMAL
        }
    }

    private fun Boolean.specialHandle(display: FoldingFeature): Int {
        return if (!this) {
            FoldState.FLAT_NORMAL
        } else {
            specialFlat(display)
        }
    }

    private fun specialFlat(display: FoldingFeature): Int {
        return when {
            isBookState(display) -> {
                FoldState.FLAT_BOOK
            }

            isTableTopState(display) -> {
                FoldState.FLAT_TABLE_TOP
            }

            isSeparating(display) -> {
                if (display.orientation == FoldingFeature.Orientation.HORIZONTAL) {
                    FoldState.FLAT_TABLE_TOP
                } else {
                    FoldState.FLAT_BOOK
                }
            }

            else -> {
                FoldState.FLAT_NORMAL
            }
        }
    }

    /**
     * 折叠动作
     */
    private fun isSeparating(foldingFeature: FoldingFeature?): Boolean {
        return foldingFeature != null && foldingFeature.state == FoldingFeature.State.FLAT && foldingFeature.isSeparating
    }

    /**
     * book状态
     */
    private fun isBookState(feature: FoldingFeature?): Boolean {
        return feature != null && feature.state == FoldingFeature.State.HALF_OPENED && feature.orientation == FoldingFeature.Orientation.VERTICAL
    }

    /**
     * tableTop状态
     */
    private fun isTableTopState(feature: FoldingFeature?): Boolean {
        return feature != null && feature.state == FoldingFeature.State.HALF_OPENED && feature.orientation == FoldingFeature.Orientation.HORIZONTAL
    }
}
 package com.haohua.demo

import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.lifecycleScope
import androidx.lifecycle.repeatOnLifecycle
import androidx.window.layout.FoldingFeature
import androidx.window.layout.WindowInfoTracker
import androidx.window.layout.WindowLayoutInfo
import com.yunzhijia.logsdk.YZJLog
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.launch

class WindowManagerConfig private constructor(){

    var uiStatus: MutableLiveData<FoldState> = MutableLiveData<FoldState>()

    private object SingletonHolder {
        val instance = WindowManagerConfig()
    }

    companion object {
        val instance = SingletonHolder.instance
    }

    // 是否折叠状态，正常手机状态
    fun isNormalPhoneStatus(): Boolean {
        return uiStatus.value == FoldState.NORMAL_PHONE
    }

    fun isUnFoldStatus(): Boolean {
        return !isNormalPhoneStatus()
//        return uiStatus.value == FoldState.FLAT || uiStatus.value == FoldState.HALF_OPENED
    }

    fun register(activity: AppCompatActivity) {
        activity.lifecycleScope.launch(Dispatchers.Main) {
            activity.lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED)
            {
                WindowInfoTracker.getOrCreate(activity)
                    .windowLayoutInfo(activity)
                    .collect { newLayoutInfo ->
                        uiStatus.value = getCurrentFoldState(newLayoutInfo)
                        YZJLog.d("im-fold", "折叠屏状态变化 = ${uiStatus.value}, act = ${activity.javaClass.simpleName}")
                    }
            }
        }
    }

    private fun getCurrentFoldState(layoutInfo: WindowLayoutInfo): FoldState {
        for (displayFeature in layoutInfo.displayFeatures) {

            val foldFeature = displayFeature as? FoldingFeature

            foldFeature?.let {
                return when (it.state) {
                    FoldingFeature.State.FLAT -> {
                        FoldState.FLAT
                    }

                    FoldingFeature.State.HALF_OPENED -> {
                        FoldState.HALF_OPENED
                    }

                    else -> {
                        FoldState.NORMAL_PHONE
                    }
                }
            }
        }
        return FoldState.NORMAL_PHONE

    }

    class FoldState private constructor(private val description: String) {

        override fun toString(): String {
            return description
        }

        public companion object {

            @JvmField
            val FLAT: FoldState = FoldState("FLAT")

            @JvmField
            val HALF_OPENED: FoldState = FoldState("HALF_OPENED")

            @JvmField
            val NORMAL_PHONE: FoldState = FoldState("NORMAL_PHONE")
        }
    }

}
 class CommonDialog(private val builder: Builder) : Dialog(
    FoldStateManager.autoSizeContextWrapper(builder.context),
    if (builder.style == 0) R.style.CommonDialogStyle else builder.style
), LifecycleEventObserver {
     private fun addLifecycleObserver() {
        (builder.context as? AppCompatActivity)?.let { activity ->
            val lifecycle = activity.lifecycle
            lifecycle.addObserver(this)
        }
    }
    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        Timber.d("FoldState onStateChanged event= ${event.name}")
        when (event) {
            Lifecycle.Event.ON_RESUME -> {
                setDialogLayout()
            }
    
            Lifecycle.Event.ON_DESTROY -> {
                if (isShowing) {
                    dismiss()
                }
            }
    
            else -> {
    
            }
        }
    }
}

override fun foldStateChangedBlock() {
    binding.ivTopBg.setImageResource(
        if (FoldStateManager.isFoldState()) {
            R.mipmap.game_icon_intermediary_guarantee_content_bg
        } else {
            R.mipmap.game_icon_intermediary_guarantee_content_bg_flat
        }
    )
}
ConstraintLayout知识记录 
 
 https://blog.csdn.net/w296365959/article/details/111700810
 https://blog.51cto.com/u_16175505/10082445

 ivImage.apply {
    val newLayoutParams = layoutParams
    if (newLayoutParams is ConstraintLayout.LayoutParams) {
        if (getItemCount()>1){
            newLayoutParams.dimensionRatio = "h,1:1"
        }else{
            newLayoutParams.dimensionRatio = "h,224:110"
        }
    }
    layoutParams = newLayoutParams
}
```

##### 其他

1. 折叠屏适配 375 x 812

2480 x 2200

vivo 折叠屏三方应用开发适配指南

[Android今日头条UI适配完善版 - 优博客](https://ubock.com/archives/androidjin-ri-tou-tiao-uigua-pei-wan-shan-ban)

[cloud.tencent.com](https://cloud.tencent.com/developer/article/1845432)

[Android 折叠屏适配攻略](https://juejin.cn/post/7395127766185836584#heading-3)

[解决华为折叠屏获取状态异常问题](https://blog.csdn.net/Tqiusheng/article/details/136022240?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Ctr-3-136022240-blog-141566115.235^v43^pc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Ctr-3-136022240-blog-141566115.235^v43^pc_blog_bottom_relevance_base7&utm_relevant_index=6)

[Android 折叠屏型号判断（包含pad）](https://blog.csdn.net/marchlqq/article/details/125673121)

[【Android折叠屏适配】基于AutoSize框架适配折叠屏并兼容多窗口模式](https://blog.csdn.net/weixin_38702864/article/details/127071799)

[代码中动态获取尺寸与 AndroidAutoSize 设置的尺寸不一样](https://blog.csdn.net/haha223545/article/details/128336432)

[android折叠屏仿微信适配](https://blog.csdn.net/weixin_42097256/article/details/137859766?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-4-137859766-blog-127071799.235^v43^pc_blog_bottom_relevance_base7&spm=1001.2101.3001.4242.3&utm_relevant_index=7)

[Lifecycle：生命周期感知型组件的基础 —— Jetpack 系列（1）](https://segmentfault.com/a/1190000042174440)

[Android屏幕横竖屏切换小记](https://lastwarmth.win/2016/07/01/about-screen/)

[android 折叠屏折叠展开触发的生命周期](https://blog.51cto.com/u_16099264/7373761)

[京东小程序折叠屏适配探索_android:resizeableactivity="true-CSDN博客](https://blog.csdn.net/JDDTechTalk/article/details/141869280?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2~default~YuanLiJiHua~Position-4-141869280-blog-136022240.235^v43^pc_blog_bottom_relevance_base7&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~YuanLiJiHua~Position-4-141869280-blog-136022240.235^v43^pc_blog_bottom_relevance_base7&utm_relevant_index=9)

[Android 主流屏幕适配方案](https://github.com/leavesCZY/AndroidGuide/blob/master/一文读懂 Android 主流屏幕适配方案.md)

[Android屏幕适配的几种方案前言 由于 Android设备存在有不同的屏幕尺寸，屏幕分辨率，像素密度，Android - 掘金](https://juejin.cn/post/7220769136208756791)

[一种极低成本的Android屏幕适配方式](https://mp.weixin.qq.com/s/d9QCoBP6kV9VSWvVldVVwA)

[理解Android WebView的加载流程与事件回调](https://mp.weixin.qq.com/s/9IuTYGwHJAHvU5TkApI8mA)

[Kotlin 中的 joinToString 函数_kotlin jointostring-CSDN博客](https://blog.csdn.net/chuyouyinghe/article/details/135484289)

https://github.com/ladingwu/dimens_sw

- density根据屏幕像素动态改变（**[今日头条适配方案](https://juejin.cn/user/976022014539326/posts)**）
- dp根据屏幕像素动态改变（**[最小宽度适配方案](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FX-aL2vb4uEhqnLzU5wjc4Q)**）

[JessYan: 今日头条屏幕适配方案常见问题汇总，提问前必看！ · Issue #13 · JessYanCoding/AndroidAutoSize](https://github.com/JessYanCoding/AndroidAutoSize/issues/13)

[通过Gradle动态修改Manifest文件](https://www.jianshu.com/p/f5cd48019109)

[Android插桩修改Manifest文件编译阶段修改Manifest文件属性 起因:在开发业务过程中，遇到了需要根据不 - 掘金](https://juejin.cn/post/7385199889053040678?from=search-suggest)

[用Gradle脚本管理Manifest文件很多Android项目都会区分debug和release的manifest文件 - 掘金](https://juejin.cn/post/6844904178133762062?from=search-suggest)

[Android 自定义Gradle插件(二):修改AndroidManifest文件本篇文章介绍了如何在自定义Gradl - 掘金](https://juejin.cn/post/7053348546560393224)

[developer.huawei.com](https://developer.huawei.com/consumer/cn/doc/other/basic_experience-0000001295870457)

![img](https://ecnj5070ng8v.feishu.cn/space/api/box/stream/download/asynccode/?code=NDcwN2E1MTNhZjkxMDA0NGZjNDI3OWIzZTNjM2U0NjRfbGljdzk5ZFVxTmI2NzZsS0hRRms0VjZodmlRQnhnRjBfVG9rZW46TUJ5cGJyNXhDbzNTc1l4WFdBTGNpb1VpbnhjXzE3NzIyNDkyMDM6MTc3MjI1MjgwM19WNA)

## 事件总线 LiveEventBus

com.github.neo-turak:LiveEventBus

## 时间日期滚动控件

com.github.weidongjian:androidWheelView

## 数字滚动 RollingText

com.github.YvesCheung.RollingText:RollingText

## 弹窗库 xpop

- com.github.li-xiaojun:XPopup
  - 各种弹窗
  - 时间日期城市选择器  XPopupExt： https://github.com/li-xiaojun/XPopupExt

## 图片选择器 pictureselector

可选带压缩和裁剪

## 吸顶布局

### 分组和吸顶recyclerView

com.github.donkingliang:GroupedRecyclerViewAdapter

com.github.Gavin-ZYX:StickyDecoration

### ConsecutiveScroller实现嵌套滑动和吸顶

- com.github.donkingliang:ConsecutiveScroller， 支持各种嵌套滑动以及多层吸顶效果

### CoordinatorLayout和AppBarLayout一起实现联动的效果

典型布局：

1. 一直显示的顶部布局
2. 上滑动可以收缩的布局
3. 上滑后吸顶的tab
4. 和Tab联动的ViewPager2

~~~kotlin

<SmartRefreshLayout
    <CoordinatorLayout
        <AppBarLayout
            <ConstraintLayout
                app:layout_scrollFlags="scroll|exitUntilCollapsed"/>
            <ConstraintLayout
                <DslTabLayout>
        </AppBarLayout>
        <ViewPager2>
    </CoordinatorLayout>
</SmartRefreshLayout>

AppBarLayout控制收起和吸顶的布局,CollapsingToolbarLayout在收起后，会保留ToolBar,contentScrim是吸顶后toolBar的背景颜色
<CollapsingToolbarLayout
    app:contentScrim="@color/game_color_BDE1FC"
    app:layout_scrollFlags="scroll|exitUntilCollapsed"/>
    <Toolbar>
~~~

## 网络抓包 chunker

## 验证码输入框

- 自定义SplitEditTextColor
- https://github.com/jenly1314/SplitEditText

## 状态栏 immersionbar

com.geyifeng.immersionbar:immersionbar

## 字母导航栏

实现垂直的字母导航效果。见自定义GameLetterView

## 黑暗模式适配

https://juejin.cn/post/7587264707285106698

# 适配

## 屏幕适配

### dpi 

Dots per Inch，每英寸的像素点数量。通过 DisplayMetrics 来获取。

```kotlin
TAG: densityDpi: 480，//像素密度480
TAG: density: 3.0，// 1dp = 3px
TAG: widthPixels: 1080px  //360dp
TAG: heightPixels: 2259px //753dp

px = dp * (dpi / 160) 
```

Android 系统定义的屏幕像素密度基准值是 160dpi，该基准值下 1dp 就等于 1px，依此类推 320dpi 下 1dp 就等于 2px, 480dpi 下 1dp等于3px。

480dpi 对应的资源文件是 xxhdpi

### 为什么要适配

不管我们在布局文件中使用的是什么单位，最终系统在使用时都需要将其转换为 px，由于不同手机的屏幕像素尺寸会相差很大，我们自然不能在布局文件中直接使用 px 进行硬编码。因此 Google 官方也推荐开发者尽量使用 dp 作为单位值，因为系统会根据屏幕的实际情况自动完成 dp 与 px 之间的对应换算。但是，使用 dp 只能适配大部分宽高比例比较常规的机型，对于特殊机型就无能为力了……

## 不同版本Android API适配

### Android 15

- 16KPageSize

  **自 2025 年 11 月 1 日起，所有提交到 Google Play 且面向 Android 15+ 设备的新应用和现有应用的更新都必须支持 16 KB 的页面大小**。

# 编译器

- D8编译器

  这是开发者在日常开发中使用的**调试编译器**，它速度快、支持增量编译，目的是让开发者能够以最快的速度在设备或模拟器上编辑、运行和调试代码，但它不会对代码进行深度的性能优化

- R8编译器

  这是用于发布 (Release build) 的**全局优化编译器**，它会将整个应用作为一个整体进行静态分析，花费可能高达数十分钟的时间来应用各种复杂的优化，从而生成体积最小、运行最快的最终代码

## R8的核心优化机制

- **Tree Shaking** ：正常应用里通常会引入大量第三方库，但往往只使用其中的一小部分功能，而 R8 能够通过全局静态分析，**找出哪些代码在运行时永远不会被调用，并将其完全移除** （*当然，这也是很多时候大家讨厌 R8 的原因，因为 R8 在高版本默认开启后，应用升级就崩溃*）
- **方法与参数内联** ：R8 会分析代码的调用情况，将总是接收相同参数的方法，或调用频繁但非常简短的方法直接内联到调用处，从而减少虚拟方法调用的开销
- **合并接口**：如果 R8 发现一个接口在实际生产代码中只有一个实现类（例如仅仅为了单元测试而提取的接口），它可以将该接口直接合并到实现类中，减少一层指针查找的性能损耗
- **与 ART 运行时的协同优化**：R8 的工作会为设备上的 ART (Android Runtime) 编译器“铺平道路”，结合 Baseline Profiles（基准配置文件），R8 知道哪些代码是启动时的“热代码”，并会在 DEX 文件中留下提示，帮助 ART 在用户设备上进一步生成高效的机器码
- **移除 Kotlin 空安全检查**：Kotlin 会在底层插入大量的非空检查 (如`checkNotNullParameter` 等)，在发布版本中，R8 可以基于全局分析证明某些值永远不为空从而移除检查，或者将其替换为不带冗长字符串的精简指令，既减小了体积又提升了性能

## R8 面临的最大挑战：反射与 Keep Rules

**反射与 JNI 带来的问题** ：由于 R8 是静态分析工具，它很难预测在运行时通过字符串动态调用的类、方法或字段（如反射），如果 R8 看不到某个类的静态调用，就会将其当作“死代码”移除，导致应用在运行时抛出 `ClassNotFoundException` 崩溃

**保留规则 (Keep Rules)** ：为了反射和 JNI 问题，开发者必须编写 Keep Rules 来明确告诉 R8 哪些类和方法不能被优化或移除，但是很多第三方库为了省事，会编写过于宽泛的规则（例如要求保留整个库的所有代码），这会严重阻碍 R8 发挥作用并拖累整体性能

## 开发者体验与工具链的改进

**反混淆与崩溃日志**：R8 会将类名和方法名重命名为简短的字母（如 "a", "b", "c"）以节省空间，为了让崩溃日志可读，R8 会输出 Mapping 文件，现在的 Android Studio 提供了不错的集成（*Android Studio 的 App Quality Insights*），开发者可以在 IDE 中直接点击按钮，自动下载和映射线上的崩溃堆栈，体验上和调试 D8 构建一样顺畅（*它会利用线上的 mapping 文件对崩溃堆栈进行反混淆*）

**官方文档大翻新**：在此之前，R8 的文档一言难尽，而现在 R8 团队对官方文档进行了大规模翻新，详细解释了 R8 的工作原理、Keep Rules 的语法规范、最佳实践以及如何排查不合理的保留规则

**R8 会大量更改源码的情况，所以栈轨迹无法指向原始代码**，例如行号以及类和方法的名称可能会发生变化，所以在使用 `retrace` 时，需要 mapping 文件，例如 ： `$ANDROID_HOME/cmdline-tools/latest/bin/retrace app/build/outputs/mapping/$releaseVariant/mapping.txt trace.txt` 。

## R8 的新特性与未来方向

**局部/包级别的 R8 优化 (Gradual R8)** ： 为了帮助历史包袱沉重的大型应用逐步迁移到 R8，R8 团队推出了按包范围开启 R8 的功能，开发者可以先对 AndroidX 库、Compose 等确定安全的包开启优化，然后逐步将自己的业务代码加入优化范围，从而降低崩溃风险

**@UsesReflection 注解**：相比于在单独的配置文件中写晦涩的 Keep Rules，R8 团队正在推广一种新的 `@UsesReflection` 注解，开发者可以直接在**发起反射调用的地方**添加注解，指明它反射了什么目标，这种方式让配置跟随代码移动，支持条件保留，且能被 IDE 直接识别和检查

**优化的资源缩减 (Optimized Resource Shrinking)** ：过去，代码缩减和资源缩减是分开进行的，现在 R8 将这两者结合起来，可以跨越代码和 XML 资源进行全局追踪，如果一个 Activity 的代码被判定为未使用并被移除了，R8 会顺藤摸瓜将它引用的无用 XML 布局、图片资源一并剔除

# 踩坑记录

## Try Catch未捕获到

发生异常的线程不对，try catch是无法捕捉到别的线程的崩溃的。

1. Try Catch范围尽可能少。
2. Try Catch尽可能抛出异常类型。

## Application

Application里存放数据导致的闪退，Application可能会被回收，里边存放的数据也会丢失。

## Activity

## Service

进行IPC通信时Service崩溃了，如何避免客户端崩溃?

1. try...catch

2. IBinder.linkToDeath()

   当 Service 因为任何原因崩溃时，所有通过 `linkToDeath` 方法注册的 `DeathRecipient` 都会通过其 `binderDied()` 方法得到通知。这样，客户端可以在这个回调中做兜底处理，比如尝试重新绑定 Service 或通知用户.

   但是这个方法不能捕获所有异常，比如：

   - 调用远程方法时由于 Service 端崩溃导致的 `RemoteException`。即使 Service 端最终会崩溃，但在它崩溃之前，客户端可能已经尝试调用了某个方法。
   - 网络问题或其他通信问题导致的异常。
   - Service 端在处理请求时，因为业务逻辑抛出的异常。

## Fragment

### Fragment系列的构造方法里直接传了参数导致的异常

配置更改重建后，会导致构造方法里的参数丢失

### Fragment XXX not attached to a context.

~~~kotlin
  @NonNull
    public final Context requireContext() {
        Context context = getContext();
        if (context == null) {
            throw new IllegalStateException("Fragment " + this + " not attached to a context.");
        }
        return context;
    }
~~~

这种异常大多是发生在接口回调里，在Fragment中与视图相关的操作，务必使用viewLifecycleOwner.lifecycleScope方法来观察数据,不能直接使用lifecycleScope。

- lifecycleScope与viewLifecycleOwner.lifecycleScope的区别？

  生命周期不同.

  lifecycleScope关联Fragment本身，生命周期从Fragment.onCreate()开始到Fragment.onDestory()结束，因此，如果Fragment没有处于attached时，调用requireFragment会抛出异常。

  viewLifecycleOwner.lifecycleScope，关联的是Fragment的视图，生命周期从`Fragment.onCreateView()` 开始，到 `Fragment.onDestroyView()` 结束。所有的UI操作都应该与视图生命周期绑定。

  App里直接用lifecycleScope的地方其实很多，大部分捕捉了异常，少部分直接launch了。上述说法待写个demo验证。

- Fragment内部如果判断了isAdd()，是否就能避免上述异常了？换个说法，isAdd()，是否代表着fragmentAttach到Activity了？

  并不是。这里涉及到Fragment的生命周期了，后续梳理。

- 什么情况下，Fragment的视图被销毁了，但是Fragment的实例未被销毁？

  Fragment的transction对Fragment进行替换等操作时。栈下边的Fragment的视图会被销毁，但是实例仍然存在。当一个Fragment被另一个Fragment替换时，会调用Fragment的onDestroyView()方法,但是不会调用onDestroy()方法。

## Dialog

### Dialog not attached to an activity

- **问题现象/出错点**
  - 调用 dialog.show()/dismiss() 时抛出 IllegalStateException: Fragment/Dialog not attached to an activity，或 WindowManager$BadTokenException。
- **原因**
  - Activity/Fragment 已经销毁或正在 finish，异步回调还在尝试显示/关闭 Dialog。
  - 使用普通 Dialog 而非 DialogFragment，生命周期未跟随 Fragment/Activity 重建。
- **优化建议**
  - 优先使用 DialogFragment，交由 FragmentManager 管理显示与恢复，避免直接持有 Activity 引用。
  - 异步回调中操作 Dialog 前，判断 activity.isFinishing / isDestroyed、fragment.isAdded，再安全调用 show/dismiss。
  - 避免在子线程直接操作 Dialog，统一在主线程 post 到 UI 线程处理。
  - ```Java
    private val currentActivity: FragmentActivity?
        get() = if (isAdded && activity != null) activity else null
    
    // 创建一个安全执行的扩展函数
    private inline fun <T> safeExecute(action: (FragmentActivity) -> T): T? {
        return currentActivity?.let { action(it) }
    }
    //示例
    safeExecute { act ->
        //账号禁用
        XPopup.Builder(act)
            .dismissOnTouchOutside(false)
            .asCustom(LoginAccountDisableDialog(act)).show()
    }
    ```

## DialogFragment

### IllegalStateException

DialogFragment的dismiss导致，需要判断DialogFragment的状态，因此DialogFragment组件慎用，弹窗类考虑用XPop.

## Intent

### android.os.TransactionTooLargeException

- **问题现象/出错点**
  - Activity 跳转或保存状态时抛 TransactionTooLargeException / FAILED BINDER TRANSACTION，多发生在传递大 List、大图片或复杂对象时。
- **原因**
  - 通过 Intent/Bundle 传递过大数据（如整页列表、bitmap、长文本等），超过 Binder 事务大小限制。
- **优化建议**
  - 避免通过 Bundle 传递大对象，只传 key/id，实际数据用本地缓存、数据库或单例仓库获取。
  - onSaveInstanceState 只保存必要的轻量状态，大数据交给 ViewModel/本地存储管理。

## RecyclerView

### getChildAt(position)

只能取到视野能见到的item,屏幕外的item会导致空指针异常。

### RecyclerView is computing a layout 多并发导致 crash



- **问题现象/出错点**
  - RecyclerView 报 IllegalStateException: is computing a layout or scrolling，或多并发刷新导致同一列表快速增删元素崩溃。
- **原因**
  - 多线程/并发修改同一个数据源（列表）并立即 notifyDataSetChanged；在 RecyclerView 正在 layout/scroll 时调用 notifyItemInserted/Removed。
- **优化建议**
  - 对列表数据的修改统一切到主线程，并且尽量在动画/滚动结束后批量更新（使用 submitList + DiffUtil）。
  - 在对过滤条件/数据源的操作中，优先生成新列表再替换，而不是在原 list 上频繁插入/删除。
  - ```Java
    private fun notifyChanged() {
        Handler(Looper.getMainLooper()).post {
            notifyItemRangeChanged(0, itemCount)
        }
    }
    ```

## ViewTreeObserver NullPointerException

### 

- **问题现象/出错点**
  - 对 View.getViewTreeObserver().addOnGlobalLayoutListener 时 View 已经为 null/未 attach，或在 onDestroyView 后仍持有监听导致空指针。
- **原因**
  - 未在 onDestroyView/onStop 中移除监听；对已经被 removeView 的对象调用 getViewTreeObserver。
- **优化建议**
  - 统一封装 add/removeViewTreeObserverListener，在 Fragment.onDestroyView 中必定移除。
  - 添加监听前判断 view.isAttachedToWindow（API 19+），避免对未 attach 的 View 做操作。
  - 如果应用程序长期引用 ViewTreeObserver，则在调用任何其他方法之前，应始终检查 isAlive() 的结果
  - ```Java
    // When adding the listener
    ViewTreeObserver viewTreeObserver = myView.getViewTreeObserver();
    if (viewTreeObserver.isAlive()) {
        viewTreeObserver.addOnGlobalLayoutListener(myGlobalLayoutListener);
    }
    
    // When removing the listener (e.g., in onDestroyView for Fragments)
    if (myGlobalLayoutListener != null) {
        ViewTreeObserver viewTreeObserver = myView.getViewTreeObserver();
        if (viewTreeObserver.isAlive()) {
            viewTreeObserver.removeOnGlobalLayoutListener(myGlobalLayoutListener);
        }
    }
    ```

  - **确保视图已初始化**
  -  确保在添加监听器之前，视图已经被正确初始化并且已经附加到窗口上。

  - ```Plain
    View myView = findViewById(R.id.my_view);
    if (myView != null && myView.getWindowToken() != null) {
        ViewTreeObserver observer = myView.getViewTreeObserver();
        observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                // 执行相关操作
            }
        });
    }
    ```

  - **避免内存泄漏**
  -  确保在不需要监听器时及时移除，避免长时间持有对视图的引用。

  - ```Plain
    ViewTreeObserver observer = myView.getViewTreeObserver();
    final ViewTreeObserver.OnGlobalLayoutListener listener = new ViewTreeObserver.OnGlobalLayoutListener() {
        @Override
        public void onGlobalLayout() {
            // 执行相关操作
            // 完成后移除监听器
            myView.getViewTreeObserver().removeOnGlobalLayoutListener(this);
        }
    };
    observer.addOnGlobalLayoutListener(listener);
    ```

  - **检查Fragment生命周期**
  -  避免持有已销毁视图：在`Fragment`的`onDestroyView()`中移除监听器并释放引用：

  - ```Java
    @Overridepublic void onDestroyView() {
        super.onDestroyView();
        if (layoutListener != null) {
            view.getViewTreeObserver().removeOnGlobalLayoutListener(layoutListener);
            layoutListener = null; // 释放引用
        }
    }
    ```

## Glide 

### You cannot start a load for a destroyed activity

- **问题现象/出错点**
  - Glide 报错 java.lang.IllegalArgumentException: You cannot start a load for a destroyed activity，多发生在异步回调/列表滚动时加载图片。
- **原因**
  - Glide 会跟随 Activity/Fragment 生命周期管理请求，在 Activity 已经 finish()/destroy 或 Fragment 已经分离时仍然调用 Glide.with(activity/fragment) 去 load 导致崩溃。
- **优化建议**
  - 永远优先使用与生命周期绑定的 with：Fragment 里用 Glide.with(this)，Adapter 里传入 Fragment/Activity 的生命周期 owner，而不是随手用 ApplicationContext。
  - 在异步回调中加载图片前先判断宿主是否还“存活”，例如 isAdded && !isDetached && !requireActivity().isDestroyed。
  - 对于列表/Adapter，确保 Glide 调用只发生在 onBindViewHolder 等 UI 生命周期正常阶段，避免在 item 被回收后仍持有旧 View/Context。
  - ```Java
    if (!ActivityCompatHelper.assertValidRequest(context)) return
     // 执行glide操作
    ```

### 回调里再次加载图片导致crash

- **问题现象/出错点**
  - 在 Glide 的回调（如 onResourceReady）中再次触发新的 Glide.with(context).load(...)，在宿主销毁或 View 回收时崩溃。
- **原因**
  - 回调执行时上下文可能已无效，或者复用的 ImageView 已被 RecyclerView 回收。
- **优化建议**
  - 尽量避免在 Glide 回调里再发起新的 UI 级请求，如需链式处理使用 Transformation/thumbnail 等内建能力。
  - 必须二次加载时，同样检查 Fragment/Activity 是否 still alive，再进行加载

## JSON转换异常

- 一般的代码里边，已经对字段类型不匹配等场景做了捕获，但是如果数据类未设置可空，而后端返回了空，这种场景，解析数据的时候不会出错，使用的时候会NPE。

## 数组和集合

- IndexOutOfBoundsException
  - Spannable.setSpan()等set操作未检查索引
  - arr.get，arr[i]等get系列方法未检查索引

## 数据类

- String.toInt()/toLong()/toDouble

## 颜色类

- String.toColorInt()

## Uri

Uri的很多操作都抛了异常，注意捕捉。

## 空指针

### Kotlin访问Java导致的NPE

三方库是Java代码，务必进行空判断，编译器不会检查容易导致NPE。

### 后端返回数据解析到值的NPE

data class 里未声明可空，而后端返回了空，这种场景使用的时候未判空就会NPE。

## NoSuchMethodError

一般都是版本兼容导致的。@RequiresApi 或 Build.VERSION.SDK_INT 判定的地方需要注意。

## OOM

### ImageView.setImageBitmap()

内存不够直接会crash。哪怕限制了图片的大小，一张压缩后10M的图片，加载到内存，可能会有两三百兆，极其容易OOM。

## ConcurrentModificationException

**原因**

- **非线程安全集合的并发修改** Kotlin 的 `MutableList` 和 `MutableMap` 默认**不是线程安全的**。当多个协程同时执行以下操作时，会引发 `ConcurrentModificationException` 或空指针：
  - 一个协程在遍历集合（如 `forEach`）
  - 另一个协程同时在添加/删除元素（如 `add`/`remove`）
  - 未同步的读写操作导致数据状态不一致 。
- **协程调度器切换线程** 协程可能在不同线程（如 `Dispatchers.IO` 和 `Dispatchers.Main`）间切换。若未同步集合访问，线程 A 修改集合后，线程 B 可能读取到失效状态 。
- **延迟初始化问题** 在协程中懒初始化集合（如 `val map by lazy { mutableMapOf() }`），多个协程同时触发初始化可能导致部分协程获取未完整初始化的集合 。

## WindowManager$BadTokenException

- **问题现象/出错点**
  - 显示 Dialog、PopupWindow、Toast 时抛 BadTokenException，常见于 Activity 已销毁或使用了错误的 Context。
- **原因**
  - 使用 Activity context 创建窗口，但在 Activity 已经 finish 时 show；或用 Application Context 显示需要 token 的窗口。
- **优化建议**
  - 严格区分：
    - Toast/全局提示可用 ApplicationContext。
    - Dialog/Popup 必须使用当前活跃 Activity 的 context，并在 show 前判断其未 finish。
  - 对需要跨页面持久显示的 UI，考虑使用独立 Activity/Fragment 承载，而不是在任意页面上直接弹窗。
  - ```Java
    if (anchorView.windowToken == null || !ActivityCompatHelper.assertValidRequest(it.context)) {
        return@doOnAttach
    }
    try {
        showAtLocation(
            anchorView,
            Gravity.NO_GRAVITY,
            0,
            0
        )
    } catch (e: WindowManager.BadTokenException) {
        // 忽略异常或记录日志
        Timber.e("Failed to show popup window $e")
    }
    //或者
    private fun showSafely(anchorView: View?) {
        if (anchorView == null) {
            return
        }
        try {
            if (anchorView.windowToken != null) {
                // 检查 context 是否仍然是有效的 Activity
                if (context is android.app.Activity) {
                    val activity = context as android.app.Activity
                    if (!activity.isFinishing && !activity.isDestroyed) {
                        pop?.showAtLocation(anchorView, Gravity.CENTER, 0, 0)
                    }
                } else {
                    pop?.showAtLocation(anchorView, Gravity.CENTER, 0, 0)
                }
            }
        } catch (e: WindowManager.BadTokenException) {
            Timber.e("Failed to show popup window safely $e")
        }
    }
    ```

## App异常重启导致的问题

### **一、问题根本原因**

1. **系统级权限变更机制** 当用户在系统设置中动态关闭应用的关键权限（如存储、位置等）时：
   1. 部分厂商（如华为、vivo）会强制终止应用进程 [2](https://blog.csdn.net/dirksmaller/article/details/76737545)；
   2. 系统尝试恢复应用时**跳过闪屏页（SplashActivity）**，直接重建最后显示的 Activity 栈 [3](https://blog.csdn.net/jifashihan/article/details/82978204)[5](https://blog.csdn.net/zpswz/article/details/92836356)。
2. **初始化****流程被绕过**
   1. 应用的核心初始化（如全局配置、数据库加载）通常放在 `SplashActivity` 中；
   2. 权限变更后的重启流程未执行闪屏页逻辑 → 后续页面因依赖未初始化而崩溃（如空指针、资源缺失）[4](https://blog.csdn.net/wzj_what_why_how/article/details/115455905)[5](https://blog.csdn.net/zpswz/article/details/92836356)。
3. **状态恢复异常**
   1. 系统通过 `savedInstanceState` 恢复 Activity 状态，但 `FragmentManager` 可能保留旧的 Fragment 引用，导致数据不一致 [2](https://blog.csdn.net/dirksmaller/article/details/76737545)[3](https://blog.csdn.net/jifashihan/article/details/82978204)。

### **二、解决方案**

#### **方案 1：全局检测异常重启（推荐）**

在 **基类 Activity** 的 `onCreate()` 中拦截重启逻辑：

java

复制

```SCSS
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState); 
    if (savedInstanceState != null) {
        // 检测到异常重启（权限变更等）
        Intent intent = new Intent(this, SplashActivity.class); 
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK  | Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
        finish();
        return;
    }
    // 正常初始化流程 
}
```

**优势**：

- 覆盖所有 Activity，确保任何页面被直接重启时都重定向到闪屏页 [4](https://blog.csdn.net/wzj_what_why_how/article/details/115455905)[5](https://blog.csdn.net/zpswz/article/details/92836356)。

#### **方案 2：分层次****初始化**

**① 基础****初始化****迁移到** **`Application`** **类** 将必要组件（如全局配置、轻量级库）提前初始化：

```SCSS
public class MyApp extends Application {
    @Override 
    public void onCreate() {
        super.onCreate(); 
        initNetwork(); // 网络库等基础组件 
    }
}
```

**② 延迟****初始化****依赖权限的功能** 在需要权限的页面（如主页）动态检查：

```SCSS
void onHomePageCreate() {
    if (!checkStoragePermission()) {
        requestPermission(); // 动态申请权限 
    } else {
        initFileSystem(); // 初始化文件模块 
    }
}
```

#### **方案 3：****进程****死亡恢复策略**

1. **持久化关键状态**
   1. 使用 `SharedPreferences` 或 `ViewModel` 保存初始化标记（如 `isAppInitialized`）；
   2. 在主页面的 `onCreate()` 中校验该标记，未初始化则跳转闪屏页 [3](https://blog.csdn.net/jifashihan/article/details/82978204)[4](https://blog.csdn.net/wzj_what_why_how/article/details/115455905)。
2. **处理 Fragment 状态残留** 在 Activity 重建时清除无效 Fragment：

```SCSS
@Override
protected void onCreate(Bundle savedInstanceState) {
    if (savedInstanceState != null) {
        savedInstanceState.remove("android:support:fragments");  // 清除 Fragment 状态 
    }
    super.onCreate(savedInstanceState); 
}
```

### **三、厂商适配建议**

1. **华为设备****兼容性**
   1. 实测发现华为设备在关闭权限后必杀进程 [2](https://blog.csdn.net/dirksmaller/article/details/76737545)，需强制验证方案 1 的有效性。
2. **添加权限变更监听** 动态注册广播接收权限变更事件：

```Java
IntentFilter filter = new IntentFilter();
filter.addAction("android.intent.action.PACKAGE_RESTARTED");  // 部分厂商广播 
registerReceiver(permissionChangeReceiver, filter);
```

### **总结最佳实践**

| 措施                     | 适用场景                         | 优先级 |
| ------------------------ | -------------------------------- | ------ |
| 基类 Activity 拦截重启   | 所有页面需重定向初始化           | ⭐⭐⭐⭐   |
| Application 级基础初始化 | 不依赖权限的组件（如日志、分析） | ⭐⭐⭐    |
| 动态权限依赖延迟初始化   | 需要权限的功能模块（如文件读写） | ⭐⭐     |

> 关键提示：优先采用 **方案 1（基类重定向）** + **Application 基础****初始化**，覆盖 90% 异常场景。测试需覆盖华为、vivo 等厂商机型 [2](https://blog.csdn.net/dirksmaller/article/details/76737545)[5](https://blog.csdn.net/zpswz/article/details/92836356)。

### Android权限变更触发应用重启的机制

Android系统在6.0及更高版本中引入动态权限管理后，当用户在系统设置中变更应用权限（如禁用相机、存储权限）时，部分场景下会触发应用进程被终止并重启的机制。这一机制源于系统对权限变更安全性的强制保障，具体表现为：权限变更后系统会结束应用所有进程，重新启动时直接恢复栈顶Activity，而非完整冷启动流程[2](https://blog.csdn.net/Androidbye/article/details/119153662)[3](https://blog.csdn.net/weixin_39947864/article/details/81909390)[4](https://blog.csdn.net/dirksmaller/article/details/76737545)。

#### 权限变更导致应用重启的核心原因

- **系统级****进程****管理机制**：对于涉及用户隐私或系统安全的敏感权限（如相机、位置、存储），权限变更会被系统判定为“进程环境变更”，进而触发进程终止。例如，禁用相机权限后，华为、vivo等厂商设备会直接杀死应用进程[2](https://blog.csdn.net/Androidbye/article/details/119153662)[4](https://blog.csdn.net/dirksmaller/article/details/76737545)。
- **Activity栈恢复逻辑**：进程重启后，系统通过`ActivityManagerService`尝试恢复之前的Activity栈，导致应用从最后显示的页面启动，而非重新执行闪屏页（SplashActivity）[2](https://blog.csdn.net/Androidbye/article/details/119153662)[3](https://blog.csdn.net/weixin_39947864/article/details/81909390)。
- **厂商定制化策略差异**：不同厂商对权限变更的处理逻辑存在差异，部分设备仅终止单个进程，部分则终止所有关联进程（如主进程、推送进程）[2](https://blog.csdn.net/Androidbye/article/details/119153662)[4](https://blog.csdn.net/dirksmaller/article/details/76737545)。

#### 应用重启后初始化异常的典型场景

- **闪屏页****初始化****被跳过**：应用核心初始化逻辑（如全局配置、数据库加载、第三方SDK初始化）通常在闪屏页完成，直接重启到业务页面会导致依赖项缺失，引发空指针异常（NPE）或资源加载失败[3](https://blog.csdn.net/weixin_39947864/article/details/81909390)[4](https://blog.csdn.net/dirksmaller/article/details/76737545)。
- **内存****数据丢失**：存储在`Application`或静态变量中的临时数据（如用户登录状态、缓存配置）会因进程重启被清空，若后续页面直接引用这些数据，会导致逻辑异常[3](https://blog.csdn.net/weixin_39947864/article/details/81909390)。
- **Fragment状态残留**：`FragmentManager`在进程重启后可能保留旧的Fragment引用，导致页面重建时出现状态不一致（如重复添加Fragment）[4](https://blog.csdn.net/dirksmaller/article/details/76737545)。

#### 解决方案：保障权限变更后初始化完整性

1. 初始化逻辑全局化改造

- **核心步骤**：将闪屏页的初始化逻辑迁移至`Application`类或独立的初始化管理器（如`AppInitializer`），确保进程重启时自动执行。
- **实现示例**：

```SCSS
public class MyApplication extends Application { 
    @Override 
    public void onCreate() { 
        super.onCreate();  
        // 执行全局初始化（仅在进程创建时触发） 
        if (getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) { 
            initDebugConfig(); 
        } 
        initDatabase(); 
        initThirdPartySDK(); 
    } 
} 
```

- **关键注意**：避免在`Application`中存储临时数据，改用持久化存储（如`SharedPreferences`、数据库）保存必要状态[3](https://blog.csdn.net/weixin_39947864/article/details/81909390)。

1. 进程重启状态检测与处理

- **通过****`Bundle`****判断重启场景**：在Activity的`onCreate`中检查`savedInstanceState`，若为非空且检测到权限变更，主动跳转至闪屏页重新初始化：

```SCSS
@Override 
protected void onCreate(Bundle savedInstanceState) { 
    super.onCreate(savedInstanceState);  
    if (savedInstanceState != null && isPermissionChanged()) { 
        Intent intent = new Intent(this, SplashActivity.class);  
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK  | Intent.FLAG_ACTIVITY_NEW_TASK); 
        startActivity(intent); 
        finish(); 
        return; 
    } 
} 
```

- **权限变更检测**：通过`ActivityLifecycleCallbacks`监听应用前后台切换，结合`PackageManager`检查权限状态变化[2](https://blog.csdn.net/Androidbye/article/details/119153662)。

1. 数据持久化与状态恢复优化

- **临时数据持久化**：将`Application`或静态变量中的关键数据存储到`SharedPreferences`或数据库，使用时先检查有效性，无效则重新加载：

```SCSS
public UserInfo getUserInfo() { 
    if (mUserInfo == null) { 
        // 从数据库恢复数据 
        mUserInfo = UserDatabase.getInstance().queryUser();  
    } 
    return mUserInfo; 
} 
```

- **Activity状态安全恢复**：重写`onSaveInstanceState`和`onRestoreInstanceState`，仅保存必要的页面状态，避免存储大对象或临时数据[2](https://blog.csdn.net/Androidbye/article/details/119153662)[3](https://blog.csdn.net/weixin_39947864/article/details/81909390)。

1. 厂商适配与异常兜底

- **针对华为等厂商的特殊处理**：通过反射或系统属性检测设备厂商，对强制重启逻辑的机型单独适配（如主动触发闪屏页跳转）[4](https://blog.csdn.net/dirksmaller/article/details/76737545)。
- **白屏问题缓解**：在`AndroidManifest.xml` 中为Activity设置透明主题或启动背景，减少重启时的视觉闪烁：

```SCSS
<activity 
    android:name=".MainActivity" 
    android:theme="@style/Theme.TransparentSplash"> 
</activity> 
```

#### 验证与测试建议

1. **权限变更场景覆盖**：在主流机型（华为、小米、OPPO、vivo）上测试关键权限（相机、存储、位置）的开启/关闭流程，检查是否触发重启及初始化完整性。
2. **异常监控**：集成Crash上报工具（如Bugly、Firebase Crashlytics），重点关注权限变更后重启导致的崩溃类型（如NPE、ClassCastException）。
3. **冷启动****性能优化**：迁移初始化逻辑后需评估启动耗时，通过异步初始化（如`WorkManager`）或延迟加载非核心组件，避免影响用户体验。

通过上述方案，可有效解决权限变更导致的初始化不全问题，确保应用在系统强制重启后仍能稳定运行。核心原则是**解耦****页面与初始化逻辑**，并通过持久化和状态检测保障数据一致性。

# 性能和优化

## 包体积优化

1. lint检查。
2. 开启混淆和shrinkResource，去除无用代码和资源
3. 图片压缩，使用webp，使用svg
4. 精简so库，移除非必要cpu架构

### so压缩

#### so文件本质

SharedObject，共享对象。本质上是ELF(Executable and Linkable Format)格式的二进制文件，就像一个装满了原生代码、资源和符号表的"大包裹"。

#### so文件体积大的核心原因

1. 冗余代码泛滥。
2. 冗余调试信息。
3. 依赖库超标。

更要命的是，Android系统对SO的存储有特殊要求：如果在Manifest中声明`android:extractNativeLibs="true"`，SO会以压缩形式存放在APK中，但安装时会解压到本地，导致占用双倍空间；如果声明为`false`，SO必须以未压缩形式存放，直接让APK体积原地起飞。这就陷入了"压缩占双倍空间，不压缩APK太大"的两难境地，也催生了各种SO压缩方案。

#### 主流so压缩方案盘点

- UPX工具类压缩

  UPX是一款通用的可执行文件压缩工具，支持ELF格式的SO文件，堪称SO压缩的"入门神器"。它的核心原理很简单：对SO文件的代码段和数据段进行压缩，在SO加载时由内置的解压代码自动解压到内存，全程对应用透明。

- 自定义框架压缩——Nano

  优点：压缩率极高（Zstd可达50%-60%，XZ可达60%-70%）；支持分组分块和并发解压，兼顾体积和启动速度；可自定义压缩算法和解压时机，灵活性强；完美适配厂商预装的Store模式要求。

  缺点：集成成本高于UPX；需要修改应用代码，对原生开发不熟悉的开发者有一定门槛；分块逻辑需要根据SO大小合理配置，否则可能影响解压效率。

  

- 原生系统兼容压缩——Deflate的"官方玩法"

  这是Android系统自带的SO压缩方案，核心依赖APK的ZIP压缩特性。当Manifest中声明`android:extractNativeLibs="true"`时，SO会以Deflate算法压缩存放在APK中，安装时系统自动解压到`/data/app/(包名)/lib`目录，运行时直接加载。

  优点：零开发成本，只需一行配置；系统原生支持，兼容性极佳；解压过程由系统管理，稳定可靠。

  缺点：压缩率低（仅30%-40%）；安装时解压会增加安装时间和磁盘占用（APK+解压后的SO）；不支持厂商预装的Store模式要求（厂商通常禁止此配置）。

#### 从源头解决

1. 编译优化，减少冗余
2. 精简依赖
3. 动态下载，按需加载so库







## 图片优化

1. 图片压缩
2. Glide加载图片
3. 大图分区加载SubsamplingScaleImageView
4. RecyclerView里的图片优化
   1. 加载过程中，使用Glide的pauseRequests()和resumeRequests()来控制加载的暂停和继续。
   2. 压缩图片，加载大图时才展示高清
   3. 限制图片宽高，不设置自适应，或者使用oss获取宽高
   4. Glide开启缓存

## 内存优化

### 内存泄漏 MemoryLeak

长生命周期对象，持有短生命周期对象的引用，导致短生命周期对象无法被回收的场景。

#### 场景

1. 资源未关闭

2. 注册反注册，订阅反订阅

3. 匿名内部类

   - 为什么点击事件不会导致内存泄漏？

4. Handler未被声明为static

   非静态的内部类，和匿名内部类，都会持有外部类的引用。

   延迟消息导致的MemoryLeak:

   message持有Handler引用，Handler持有Activity的隐式引用

5. 静态类持有了Activity的引用

6. ThreadLocal

   key是被weakRere

7. 长生命周期对象持有短生命周期对象的引用

   - ViewModel中如果持有了Activity或者Fragment的引用

### Java内存和Native内存

- Java堆

  由JVM管理，有GC自动回收不用手动操心内存释放，有大小限制，超出会报Java层的OOM。

- Native堆

  直接向系统申请内存，不受虚拟机管理，没有自动回收机制。分配多少，释放多少全由开发者手动控制。看似空间更大，但是一旦失控可能会拖慢整个系统。

### Native内存分配与释放API

- C标准库API

  ~~~C
  #include <stdlib.h>
  
  // 分配内存：未初始化，内容是随机值
  void* malloc(size_t size); 
  // 分配内存：初始化所有字节为0
  void* calloc(size_t num, size_t size); 
  // 重新分配内存：扩大/缩小已有内存块，可能会移动内存地址
  void* realloc(void* ptr, size_t size); 
  // 释放内存：必须和分配API成对使用，否则内存泄漏
  void free(void* ptr); 
  ~~~

- C++标准库API

  ~~~c++
  #include <new>
  
  // 分配内存并调用构造函数
  int* p1 = new int; 
  int* p2 = new int[10]; // 分配数组
  // 释放内存并调用析构函数
  delete p1; 
  delete[] p2; // 数组必须用delete[]，否则只释放首元素，内存泄漏
  ~~~

- Android专用API

  ~~~c
  #include <cutils/memory.h>
  #include <android/log.h>
  
  // 分配可缓存的内存，适合频繁分配/释放的小内存块
  void* ashmem_create_region(const char* name, size_t size);
  // 分配共享内存，用于跨进程通信
  void* mmap(void* addr, size_t length, int prot, int flags, int fd, off_t offset);
  // 释放共享内存/映射内存
  int munmap(void* addr, size_t length);
  ~~~

  这类API适合特殊场景（比如跨进程共享数据），但使用门槛更高，比如mmap分配的内存，必须用munmap释放，且要确保addr和length与分配时一致，否则会导致内存泄漏或系统异常。

### Native内存泄漏

本质是分配了的内存没有被释放，且指向该内存的指针被销毁，导致系统无法回收这部分的内存。举个典型的泄漏场景：在Native层写一个工具类，提供一个初始化方法，分配内存后存储在全局指针中，但没有提供对应的释放方法。每次调用初始化方法，都会分配新的内存，旧的内存指针被覆盖，再也无法释放，内存会像滚雪球一样越滚越大。

### 如何定位Native内存泄漏

1. AndroidStudioProfiler

   适用场景：快速查看Native内存整体趋势，判断是否存在内存泄漏、内存暴涨。

   操作步骤：

   1. 打开Android Studio，连接真机/模拟器，运行APP；
   2. 点击底部「Profiler」，选择「Memory」标签，在顶部选择要监控的APP进程；
   3. 在「Memory Usage」图表中，选择「Native」选项，即可查看Native内存的实时变化。

   关键解读：

   - 如果Native内存曲线**持续上涨，且操作完成后（比如关闭页面、停止操作）不下降**，大概率存在内存泄漏；
   - 如果内存曲线**频繁波动、暴涨暴跌**，可能是频繁分配/释放小内存块，导致内存碎片严重；
   - 缺点：只能看到整体趋势，无法定位到具体的泄漏代码行，**适合“初步排查”**。

2. Native Memory Tracking（NMT）：Google官方神器

   适用场景：精准统计Native内存分配详情，定位泄漏的模块/API，支持ART虚拟机的Android 7.0+设备。NMT是ART虚拟机提供的Native内存跟踪工具，能统计不同类型的Native内存分配（比如malloc分配、mmap分配、线程栈内存等），还能生成详细的内存报告，帮你找到“内存刺客”。

   #### 实战操作：

   ##### 步骤1：开启NMT跟踪

   在APP启动时，通过adb命令开启NMT（需root权限，或APP拥有DEBUG权限）：

   ```bash
   bash
   
    体验AI代码助手
    代码解读
   复制代码# 开启NMT，模式为full（完整跟踪，会有一定性能开销）
   adb shell am start -n 包名/主Activity名 --es android.nmt.app_level full
   ```

   也可以在Native代码中手动开启：

   ```c
   c
   
    体验AI代码助手
    代码解读
   复制代码#include <art/runtime/native_memory_tracking.h>
   
   // 开启full模式跟踪
   art::NativeMemoryTracking::StartTracking(art::NativeMemoryTracking::kFullTracking);
   ```

   ##### 步骤2：生成内存报告

   执行相关操作（比如触发初始化、跳转页面）后，通过adb命令生成内存报告：

   ```bash
   bash
   
    体验AI代码助手
    代码解读
   复制代码# 生成NMT报告，保存到本地文件
   adb shell dumpsys meminfo 包名 --native-heap > nmt_report.txt
   ```

   ##### 步骤3：分析报告

   打开nmt_report.txt，重点关注「Native Heap Allocations」部分，会按分配类型统计内存使用情况，示例如下：

   ```text
   text
   
    体验AI代码助手
    代码解读
   复制代码Native Heap Allocations:
   Total: 120MB
     malloc: 80MB (66.7%)
       libnative-lib.so: 75MB  # 重点！该so库分配了大量内存
         0x7f1234567890: 1MB (malloc, 函数名: init_buf)
         0x7f12345678a0: 1MB (malloc, 函数名: init_buf)
         ...
     mmap: 30MB (25.0%)
     thread stack: 10MB (8.3%)
   ```

   从报告中可以看出，「libnative-lib.so」库的「init_buf」函数分配了大量内存，结合代码排查，就能快速定位到泄漏点——比如该函数频繁分配内存却未释放。

   小技巧：如果APP使用了多个Native库，可以通过报告中的「libxxx.so」分组，快速锁定内存占用最高的模块，缩小排查范围。

3. Valgrind：Linux老牌工具，精准定位泄漏与野指针

   适用场景：本地调试Native代码，精准定位内存泄漏、野指针、重复释放等问题，适合C/C++原生开发。

4. AddressSanitizer（ASAN）：快速捕捉内存异常（Android 8.0+）

适用场景：真机调试，快速捕捉内存泄漏、野指针、缓冲区溢出等问题，性能开销比Valgrind小，适合开发后期验证优化效果。

ASAN是Google推出的内存错误检测工具，Android 8.0+支持将其集成到APP中，能在运行时实时检测内存异常，并输出详细的错误日志，包括错误类型、代码行号。

### 实战优化

1. 避免内存泄漏

   场景场景：

- 全局指针未释放
- c++对象未调用析构函数
- 跨模块调用导致的内存泄漏

2. 内存复用

   - 对象池
   - 内存池

3. 内存碎片优化

4. 特殊场景优化（图片、文件、线程栈内存）

5. 避坑指南

   - 重复释放内存
   - 野指针访问
   - c++数组用delete释放（未加[]）
   - 智能指针循环引用
   - 忽略了内存分配失败的情况

   

## 渲染优化

造成UI不流畅的原因：

- 主线程耗时导致掉帧甚至ANR

  列表排序，计算等CPU密集型也可能导致主线程阻塞。

- 布局太复杂或者嵌套过深

- 布局的重绘被触发多次

  - 这通常出现在需要动画的场景，比如以改变View的布局（大小）的方式来实现动画，或者频繁的改变View的层次，比如频繁的addView和removeView。这都会不断的触发measure/onMeasure，layout/onLayout和View的重绘。

- 频繁的GC

  GC会对所有的线程产生影响，对UI线程也是有影响。

### 布局优化

1. 删除无用View
2. 减少嵌套层次
3. Merge和Incude，布局复用
4. ViewStub延迟加载
5. 减少过度绘制
6. 延迟加载和按需加载

可以用LayoutInspector来看布局层级

### View绘制优化

1. 当局部更新时不要触发整体重绘

   这里需要，首先，不要故意的去触发整体刷新（除非非常的有必要，比如多个View都需要刷新数据时）；另外，就是要小心防止触发整体刷新的坑，因为某些原因，即使小心的更新局部也会造成整体的刷新。

2. 避免频繁的触发整体的重绘

   千万不要直接改变View的大小的方式来做动画，或者在做动画的同时改变View的布局，更不要添加或者移除View，这都会直接触发整体的重绘

3. 避免onDraw中做额外事情

4. 对象提前初始化，避免重复创建

### 动画优化，让动画“丝滑不卡顿”

动画是提升用户体验的“利器”，但写不好就会变成“卡器”。动画卡顿的根源是：**频繁触发`measure`和`layout`（比如修改`width`、`height`、`margin`），导致每一帧都要重新计算布局**。

### 1. 用属性动画，别用补间动画

补间动画（`AlphaAnimation`、`TranslateAnimation`）只是“视觉欺骗”，不会真正改变View的属性（比如移动一个按钮后，点击原位置仍会触发点击事件），而且容易导致过度绘制。

属性动画（`ObjectAnimator`）直接修改View的属性（如`translationX`、`scaleY`），性能更好，且交互正常：

```kotlin
kotlin

 体验AI代码助手
 代码解读
复制代码// 推荐：属性动画移动View（不触发measure/layout）
ObjectAnimator.ofFloat(button, "translationX", 0f, 300f)
    .duration = 500
    .start()

// 不推荐：补间动画（视觉移动，实际位置没变）
val translateAnim = TranslateAnimation(0f, 300f, 0f, 0f)
translateAnim.duration = 500
button.startAnimation(translateAnim)
```

### 2. 优先用“不触发布局”的属性

动画时修改以下属性，不会触发`measure`和`layout`，只触发`draw`，性能更好：

- 位移：`translationX`、`translationY`
- 缩放：`scaleX`、`scaleY`
- 旋转：`rotation`、`rotationX`、`rotationY`
- 透明度：`alpha`

避免修改以下属性（会触发`measure`或`layout`）：

- `width`、`height`、`layoutParams`
- `margin`、`padding`
- `top`、`left`、`right`、`bottom`

### 3. 开启硬件加速：让GPU帮你干活

硬件加速能让GPU分担一部分绘制工作（比如处理纹理、透明度），提升动画流畅度。Android 3.0（API 11）以上默认开启全局硬件加速，但可以针对单个View优化：

```xml
xml
<!-- 在布局中开启硬件加速 -->
<View
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layerType="hardware"/> <!-- 硬件加速 -->
```

注意：硬件加速不是万能的，某些自定义View（如用`Canvas.drawTextOnPath`）在硬件加速下可能显示异常，这时可关闭：`android:layerType="software"`。

### StriceMode

## ANR

ApplicationNotResponding





现象：

1. 触摸事件5秒钟未响应

2. Service20秒钟未响应

3. BroadcastReceiver

   优先级是前台Intent，10~20S

   优先级是后台Intent，60~120S

原因：

![image-20260309182322442](/Users/mac/Library/Application Support/typora-user-images/image-20260309182322442.png)



## WebView

Android加载H5页面流程：

1. 初始化Webview内核
2. 请求Html document解析网址
3. 加载解析JS页面等
4. 请求数据
5. 渲染

Webview提速优化的思路可以按照上述流程来。

### WebView相关类

#### WebViewClient

### WebView加载优化

1. 使用腾讯X5替换原生WebView

   X5优势：

   - 不同设备上体验统一，解决碎片化严重问题。
   - 宣传速度提升35%，节省流量40%，崩溃率低于0.15%（云端加速和智能路由技术）
   - 安全性强，可进行网址和脚本检测，能一定程度防范网络攻击。
   - 有视频，文件播放能力，并且体验一致。

   X5缺陷：

   - X5内核会动态下载代码，海外可能无法上架。

2. WebView预加载

   - 利用idleHandler在空闲时创建Webview池。

   - 每次使用时直接从池里获取Webview。

     （PX项目里的WebView预热是直接创建了PXWebView实例调用了loadUrl后延迟3秒关闭了，能否起到预热作用？）

   - context如何选择？

     每个 WebView 应该是对应于特定的 Activity Context 实例的，可以通过MutableContextWrapper来解决，MutableContextWrapper 允许外部替换它的 baseContext，因此我们可以在一开始的时候使用 Application 作为 baseContext，等到 WebView 和 Activity 进行实际绑定的时候再来替换。

   - Webview池占用额外内存的问题

     空间换时间，会有额外内存开销导致OOM等，可以根据版本来实现。

3. 预置离线包

   js,html,ico等文件

   1. 离线包管理，静默下载，更新
   2. 内置离线包解压，SHA值对比防篡改等。

4. 网络优化

   - 并行请求（预加载接口）

     H5在加载模版文件的时候，原生同时进行正文数据请求，Native 端再通过 JS 将正文数据传给 H5，以此来实现并行请求从而缩短总耗时。

     实现预加载接口可通过接口配置下发。

   - 浏览器的Http缓存策略

   - 拦截请求与共享缓存（重写WebViewClient的shouldInterceptRequest方法）

     1. 先判断是否命中本地缓存，如果命中了，返回本地资源构建的WebResourceResponse。

     2. 通过OKHttp的Cache功能来实现资源缓存，比如图片，HTML，JS等类型。

        拦截上述类型的请求，全部通过原生的OKHttp去请求，这样的好处是能图片WebView自带的缓存容量的上限。（WebView默认也有缓存机制，但是缓存空间相对比较小。）参考[CacheWebView](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fyale8848%2FCacheWebView)实现。

   - CDN加速

     构建CDN网络，将JS，CSS，图片视频等静态类型文件托管到CDN提升下载速度。

5. 延迟加载

   双端非首屏必须的网络请求，JS调用，埋点上报等，都后置到首屏显示之后再执行。

6. 页面静态直出 ？？？

   并行请求正文数据虽然能够缩短总耗时，但还是需要完成解析 JSON、构造 DOM、应用 CSS 样式等一系列耗时操作，最终才能交由内核进行渲染上屏，此时 **组装 HTML** 这个操作就显得比较耗时了。为了进一步缩短总耗时，可以改为由后端对正文数据和前端代码进行整合，直出首屏内容，直出后的 HTML 文件已经包含了首屏展现所需的内容和样式，无需进行二次加工，内核可以直接渲染。其它动态内容可以在渲染完首屏后再进行异步加载

   由于客户端可能向用户提供了控制 WebView 字体大小，夜间模式的选项，为了保证首屏渲染结果的准确性，服务端直出的 HTML 就需要预留一些占位符用于后续动态回填，客户端在 loadUrl 之前先利用正则匹配的方式查找这些占位字符，按照协议映射成端信息。经过客户端回填处理后的 HTML 内容就已经具备了展现首屏的所有条件

7. 复用Webview ？？？

   更进一步的做法就是可以尝试复用 WebView。由于 WebView 使用的模板文件已经是固定的了，因此我们可以在 WebView 预加载缓存池的基础上增加复用 WebView 的逻辑，当 WebView 使用完毕后可以将其正文数据全部清空并再次存入缓存池中，等下次需要时就可以直接注入新的正文数据进行复用了，从而减少了频繁创建 WebView 和预热模板文件带来的开销

8. 白屏检测和降级

   通过截屏实现白屏检测。（无法感知白屏？？？）







### 原生WebView和腾讯X5

### EC离线包技术

#### 概述

本文档介绍了一个基于Android的H5离线包技术解决方案，实现资源的动态更新、缓存与校验。核心目标包括：

- 启动时自动检测更新；
- 支持包版本比对与增量更新；
- 在离线状态下使用本地缓存资源；
- 提供覆盖式更新与缓存清理机制。

#### 架构设计

##### 模块结构

| 模块                          | 功能说明                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| PkgInitializer                | 启动初始化器，应用启动时延迟触发资源更新任务。               |
| PkgInfoManager                | 离线包核心管理器，包含版本校验、资源下载、缓存验证、资产包导入等逻辑。 |
| PkgDatabase / PkgInfoDao      | 本地数据库，记录包信息（资源 Key、版本号、MD5、路径等）。    |
| PkgApiManager / PkgApiService | 网络接口层，通过 API 拉取资源信息。                          |
| DownloadWorker                | 公用下载任务执行器（业务层未展示实现）。                     |

#### 数据持久化

PkgInfoDao（数据访问对象）使用Room数据库进行离线包信息的本地存储：

- getAll(): 获取全部离线包信息
- loadPkgByResKey(): 根据资源键获取包信息
- updateAll(): 更新包信息
- insertAll(): 插入包信息
- delete(): 删除包信息

#### 离线包管理机制

1. ### 更新检查

> 每次 App冷启动从服务器获取离线包列表；
>
> 在 IO 线程中延迟 2 秒执行；
>
> 从服务器拉取最新包信息；
>
> 校验与解压本地 Asset 预置包；
>
> 更新所有本地缓存包。

```Kotlin
CoroutineScope(SupervisorJob() + Dispatchers.IO).launch(handler) {
    delay(2000)
    PkgInfoManager.checkUpdate()
    PkgInfoManager.checkAssetsPkg()
    PkgInfoManager.updateLocalPkg()
}
```

1. #### 更新检测 `checkUpdate()`

从服务端拉取最新的资源包列表，并与本地数据库版本对比：

- 如果本地存在记录：更新版本号与远程信息；
- 如果本地不存在：插入新包信息；
- 所有包信息均存入 `Room` 数据库。

接口： `POST common/hippy/resource/client/latest-pkg`

参数：

- `deviceId`: 设备唯一标识；
- `resourceKey`: 可选，用于单包更新。

1. #### 本地包更新 `updateLocalPkg()`

负责根据数据库记录下载最新 zip 包或使用本地缓存：

逻辑顺序：

1. 校验本地缓存包是否存在且 MD5 匹配；
2. 若校验失败或版本落后，进行下载；
3. 下载完成后进行 MD5 对比、解压、更新数据库记录；
4. 删除旧版本文件与缓存。

下载实现： 通过 `DownloadWorker.startDownload()` 执行，成功后：

- 校验下载包 MD5；
- 解压至目标文件夹；
- 更新 `cacheFileRootPath`, `cacheFileVersion`, `cacheFileMd5`。

1. #### Assets 预置包检测 `checkAssetsPkg()`

首次安装应用时，通过 `assets/h5` 目录中的 zip 包作为基础资源包，流程如下：

1. 列出 `assets/h5` 下所有资源目录；离线包目录参考下图，目录命名规则是：**h5/$resKey/$version/$fileName**
2. 对比数据库中已存在的版本；
3. 如果 assets 版本更高，则解压覆盖；
4. 若数据库中无记录，自动插入新包数据。

此机制确保初次安装或无网情况下仍有默认资源可用。

![img](https://ecnj5070ng8v.feishu.cn/space/api/box/stream/download/asynccode/?code=MDEzOTg5ZGFlMWM3MTk0NWQ3ZmMzNWI0NjVhZTBhMWVfbmFRdVpDaWFaNzVCNWpGR25zaFl2RFlUdXZzMFZhMm1fVG9rZW46RWM3bmJKQldQb0Q3TG94QW5PeWMybHdybm5jXzE3NzIyNDc3NzU6MTc3MjI1MTM3NV9WNA)

![img](https://ecnj5070ng8v.feishu.cn/space/api/box/stream/download/asynccode/?code=NDM0MzIzNjVlYTQ4ZGJmN2NjNGQ5MTY2Yzk1ZGFlNWZfeDI5VWU1eVF5SVNCS0dHb3pmaHZOWjlsdlZobW5qZjNfVG9rZW46UUxyN2IzZmtsb1RNMlp4QnpNd2NKNWRpbm5mXzE3NzIyNDc3NzU6MTc3MjI1MTM3NV9WNA)

1. ### 包校验

获取单个资源包的使用信息，流程包括：

- 校验数据库记录与文件一致性；
- 若本地文件缺失或损坏：
  - 自动重新拉取更新；
  - 更新后返回包路径。

> 版本对比：验证缓存版本是否过期
>
> MD5校验：确保文件完整性
>
> 自动清理：删除过期缓存

返回结构为 `PkgInfoBean`，包含：

- `resourceKey`
- `version`
- `cacheFileRootPath`
- `cacheFileVersion`
- `cacheFileMd5`
- `cdnUrl`

1. ### 包加载与解压

- 创建版本隔离的缓存目录
- 下载完成后进行MD5验证
- 解压并更新缓存信息

1. ### 缓存清理与删除

- `clearCache()`: 清空数据库与所有包记录；
- `deletePkgInfo(resKey)`: 删除指定资源包及本地文件。

#### WebView客户端实现

### PXWebViewClient（资源拦截处理）

1. ##### 本地资源加载

```Java
private fun requestLocalResource(uri: Uri): WebResourceResponse? {
    // 1. 检查合法路径
    if (PathUtils.isLegalPath(uri.toString())) {
        val realPath = PathUtils.pathToReal(AppGlobal.sCurrentActivity, uri.toString())
        val inputStream = FileInputStream(realPath)
        return WebResourceResponse(getMime(realPath), Charset.defaultCharset().name(), inputStream)
    }
    
    // 2. 检查离线包资源
    if (!pkgInfo?.cacheFileRootPath.isNullOrEmpty()) {
        val localFilePath = "${pkgInfo?.cacheFileRootPath}/$host$path"
        val localFile = File(localFilePath)
        if (localFile.exists() && localFile.isFile) {
            val inputStream = FileInputStream(localFile)
            val response = WebResourceResponse(mimeType, Charset.defaultCharset().name(), inputStream)
            val headers = mutableMapOf<String, String>()
            headers.put("Access-Control-Allow-Origin", "*") // 解决跨域问题
            response.responseHeaders = headers
            return response
        }
    }
    return null
}
```

1. ##### MIME类型识别

```Java
private fun getMime(url: String): String {
    var mime = "text/html"
    val path = currentUri.path
    path?.apply {
        if (endsWith(".css")) mime = "text/css"
        else if (endsWith(".js")) mime = "text/javascript"
        else if (endsWith(".jpg") || endsWith(".gif") || 
                endsWith(".png") || endsWith(".jpeg")) mime = "image/*"
        // ... 更多类型
    }
    return mime
}
```

1. ##### 外部协议处理

```TypeScript
override fun shouldOverrideUrlLoading(webView: WebView?, url: String?): Boolean {
    if (url?.startsWith(ALIPAYS_SCHEME) == true || url?.startsWith(ALIPAY_SCHEME) == true) {
        try {
            AppGlobal.sCurrentActivity.startActivity(
                Intent("android.intent.action.VIEW", url.toUri())
            )
            AppGlobal.sCurrentActivity.finish()
        } catch (_: Exception) {
            showError("未检测到支付宝客户端，请安装后重试。")
        }
        return true
    }
    // 处理其他外部协议
    return super.shouldOverrideUrlLoading(webView, url)
}
```

##### WebChromeClient扩展功能

PXWebChromeClient（增强功能）

1. ##### 文件选择器

```JavaScript
override fun onShowFileChooser(
    webView: WebView?,
    callback: ValueCallback<Array<Uri>>?,
    params: FileChooserParams?
): Boolean {
    getBase()?.let {
        ImageAbility.chooseImage(
            it, 9,
            success = { paths ->
                callback?.onReceiveValue(
                    paths.map { media -> media.availablePath.toUri() }
                        .toTypedArray()
                )
            },
            failed = { code, msg ->
                callback?.onReceiveValue(null)
            })
    }
    return true
}
```

1. ##### 全屏视图处理

```TypeScript
override fun onShowCustomView(var1: View?, var2: IX5WebChromeClient.CustomViewCallback?) {
    getBase()?.apply {
        requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE
        val decorView = window.decorView as FrameLayout
        decorView.addView(var1)
    }
    customView = var1
    mcustomViewCallback = var2
}

override fun onHideCustomView() {
    // 恢复竖屏并移除全屏视图
    getBase()?.apply {
        requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_PORTRAIT
        val decorView = window.decorView as FrameLayout
        decorView.removeView(customView)
    }
    // ...
}
```

1. ##### 标题同步

```TypeScript
override fun onReceivedTitle(webView: WebView?, title: String?) {
    super.onReceivedTitle(webView, title)
    engineContext.getBridgeManager().doCallNativeModule(
        PXCallNativeParams(
            PXModule.MODULE_NAME, "setNavigationBarTitle",
            JSONObject().apply {
                put("title", title)
            })
    )
}
```

##### 核心组件

1. ##### PXEngineContext（引擎上下文）

> 提供引擎运行所需的基础上下文信息
>
> 管理WebView实例和业务代码
>
> 负责页面加载完成后的回调处理

```Java
interface PXEngineContext {
    fun getContext(): Context
    fun getWebView(): WebView
    fun getBizCode(): String // 需要返回业务唯一的 Code 码
    fun getBizUrl(): String
    fun getBridgeManager(): PXBridgeManager
    fun onLoadFinish()
}
```

1. ##### PkgInfoBean（离线包信息实体）

> resourceKey: 资源唯一标识符
>
> version: 远程资源版本号
>
> cdnUrl: 资源下载地址
>
> md5Hash: 资源完整性校验
>
> preLoad: 预下载开关
>
> cacheFileRootPath: 本地缓存路径
>
> cacheFileMd5: 缓存文件MD5
>
> cacheFileVersion: 缓存文件版本

```TypeScript
@Entity(tableName = "PkgInfoTable")
data class PkgInfoBean(
    @PrimaryKey
    var resourceKey: String = "",
    var version: String = "",
    var cdnUrl: String = "",
    var md5Hash: String = "",
    var preLoad: Boolean = false,
    var cacheFileRootPath: String? = null,
    var cacheFileMd5: String? = null,
    var cacheFileVersion: String? = null,
)
```

##### 关键设计点说明

1. ### 协程异常处理

通过 `CoroutineExceptionHandler` 捕获异步异常，防止崩溃：

```Kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e("PkgInitializer", exception.message ?: "")
}
```

1. ### 并发与健壮性

- 所有网络与文件操作在 `Dispatchers.IO` 执行；
- 使用 `SupervisorJob` 避免单任务失败影响整体流程；
- 使用 `suspendCancellableCoroutine` 管控下载任务。

1. ### 校验机制

- 双重 MD5 校验（下载包与解压目录）；
- 本地版本与服务端版本对比；
- 非 WiFi 环境下自动跳过非强制下载（节省流量）。

1. ### 数据一致性

- 数据库存储所有包元信息；
- 清除旧版本目录与 MMKV 缓存；
- 通过统一入口方法 `getPkgInfo()` 确保调用方拿到最新、有效资源。

#### 文件组织结构

```Kotlin
com.pxb7.bizpackage
├── initializer/
│   └── PkgInitializer.kt          # 自动初始化入口
├── manager/
│   └── PkgInfoManager.kt          # 离线包核心逻辑
├── database/
│   ├── PkgDatabase.kt             # Room Database 实体定义
│   └── PkgInfoDao.kt              # DAO 接口
├── api/
│   ├── PkgApiManager.kt           # Retrofit Service 管理
│   └── PkgApiService.kt           # 资源更新 API
└── bean/
    └── PkgInfoBean.kt             # 数据模型
```

#### 运行时行为总结

| 阶段     | 操作                           | 结果                                |
| -------- | ------------------------------ | ----------------------------------- |
| 应用启动 | PkgInitializer.create()        | 启动后台协程任务                    |
| 网络检查 | checkUpdate()                  | 同步服务端包版本信息                |
| 资源检测 | checkAssetsPkg()               | 校验并导入预置资源                  |
| 包更新   | updateLocalPkg()               | 下载或校验本地包                    |
| 资源使用 | getPkgInfo(resKey)             | 读取包路径供 WebView 或业务模块加载 |
| 清理阶段 | deletePkgInfo() / clearCache() | 清除缓存数据                        |

聊天室内嵌H5方案

其他方案

ServiceWorker

多进程

小程序

## 代码性能优化

### 自动装箱

1. 慎用**arrayOf**

   val intArr = arrayOf(1,2,3) //得到的是包装类型Integer的数组，有自动装箱的开销。

   用intArrayOf(1,2,3)来替换。

### 高阶函数的使用

- toLowerCase()， equals(igNoreCase: Boolean)

  toLowerCase()使用时会创建一个新的字符串，如果是用来比较，可以使用equals(igNoreCase: Boolean)来替换。

### RecyclerView

1. adapter的Item显隐控制。

   如果在adapter的onBind()里涉及到部分控件的显隐要注意了，Item布局会复用，加载前务必重置显隐状态，或者同时处理好显隐的逻辑，只设置了满足XXX条件后显示或者隐藏，那么下一个item会因为复用而被影响到。

2. 使用Diff来优化数据加载

3. ScrollView的优化

   如果一个页面的布局很长，需要上下滑动，那么考虑一下使用RecyclerView加上多布局来优化，RecyclerView的回收机制能提升页面的性能。

### 按钮防抖

1. 按钮能加都加个防抖吧，尤其涉及到调用接口的，一般按钮加了防抖之后都是没啥问题的，但是如果不加防抖，一旦有问题都是你的问题。
2. 部分三方库的使用时也得注意，比如Xpop的confirm弹窗，如果按钮加不了防抖那在接口加个防抖也行。

# Debug技巧

## 定位到当前Activity

Logcat 清空所有过滤条件，输入 start u0。

原理：system_server输出了ActivityRecord的相关日志。（所以不能是package: mine）

## AndroidProfiler

https://juejin.cn/post/7591416714694344714

## Crash日志获取

### 获取 crash 日志

1. #### 获取bugreport

```
adb bugreport <output_directory>
```

这会在指定目录生成一个包含设备详细信息的zip文件。

1. #### 使用logcat循环缓冲区

Android系统会维护一个循环缓冲区，可能保留部分历史日志：

```
adb logcat -b all -d > all_logs.txt
```

然后搜索您的应用包名和崩溃关键词：

```
grep -A 30 -B 5 ``"your.package.name\|CRASH\|Exception"`` all_logs.txt
```

1. #### 检查系统持久化日志

##### DropBox管理器

Android会将重要崩溃信息持久化存储在DropBox中：

```
adb shell dumpsys dropbox --``print
```

过滤您的应用包名：

```
adb shell dumpsys dropbox --``print`` | grep ``"your.package.name"
```

##### 提取特定崩溃文件

```
adb shell ``"ls /data/system/dropbox/"` `adb pull /data/system/dropbox/<crash_file>
```

1. #### 检查tombstones目录（Native崩溃）

```
adb shell ``ls`` /data/tombstones/ ``adb pull /data/tombstones/tombstone_XX
```

1. #### 检查ANR日志

```
adb shell ``ls`` /data/anr/ ``adb pull /data/anr/traces.txt
```

1. #### 使用第三方日志收集工具

如果上述方法都无法获取前一天日志，建议未来集成以下工具预防日志丢失：

1. Firebase Crashlytics：自动收集崩溃日志
2. ACRA：开源崩溃报告框架
3. 腾讯Bugly：国内常用的崩溃收集服务
4. 系统设置调整（预防未来日志丢失）

### 分析崩溃日志

1. #### 查找崩溃信息

   1. 解压bugreport文件后，查找以下内容：
      1. `bugreport-<device>-<timestamp>/FS/data/anr/traces.txt`- ANR(应用无响应)日志
      2. `bugreport-<device>-<timestamp>/FS/data/tombstones/`- native崩溃日志
      3. `bugreport-<device>-<timestamp>/main_entry.txt`- 主要系统日志
   2. 使用grep搜索特定APP的崩溃信息：
   3. `grep -A 20 -B 5 ``"your.package.name"`` bugreport.txt`

1. #### 关键信息查看

- 查找"CRASH"、"ANR"、"Exception"等关键词
- 查看崩溃堆栈信息(StackTrace)
- 注意崩溃时的系统状态(内存、CPU使用情况)

1. #### 使用工具分析

##### Android Studio分析工具：

- 打开Android Studio
- 选择"Analyze" > "Analyze Stack Trace"
- 粘贴崩溃日志进行分析

##### 特定APP崩溃分析技巧

1. 过滤特定APP日志：

 `adb logcat | grep -E ``"your.package.name|AndroidRuntime|Crash"`

1. 获取更详细的崩溃信息：

 `adb shell dumpsys dropbox --``print`` <package_name>`

1. 检查ANR原因：

 `adb pull /data/anr/traces.txt`

1. #### 常见崩溃原因分析

NullPointerException：对象未初始化就使用

OutOfMemoryError：内存泄漏或大对象分配

ANR：主线程阻塞超过5秒

Native Crash：JNI代码或第三方库问题

### 其他

1. ##### 系统设置调整（预防未来日志丢失）

*`增加logcat缓冲区大小`* `adb shell stop ``adb shell setprop logd.logpersistd.size 10M ``adb shell start ` *`# 查看当前缓冲区大小`* `adb shell getprop logd.logpersistd.size`

1. ##### 厂商特定日志工具

某些手机厂商提供了自己的日志收集工具：

- 华为：`adb shell bbox`
- 小米：`adb shell getmiuilog`
- OPPO/Vivo：通常需要进入工程模式获取完整日志

1. ##### 注意事项

2. 需要USB调试权限和root权限才能访问部分日志文件
3. 日志保留时间取决于设备型号和系统版本
4. 建议在发现崩溃后尽快收集日志，避免被覆盖



# Android逆向

Hook

热修复

插件化



# 待梳理

https://juejin.cn/user/2788017216685784/posts