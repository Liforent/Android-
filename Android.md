# Android底层

## Handler机制

### Q&A

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

# 四大组件

## Fragment

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

# JetPack

## Lifecycle

## LiveData

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

# 功能实现和三方库

## AI

- 人脸检测，文字识别，条码识别，人脸网格检测，图像标签，姿势检测 等 https://github.com/jenly1314/MLKit

## Banner

io.github.youth5201314:banner

## 布局圆角设置 ShapeView

com.github.getActivity:ShapeView

## 大图加载 SubsamplingScaleImageView

com.davemorrissey.labs:subsampling-scale-image-view-androidx

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

## 图片优化

1. 图片压缩
2. Glide加载图片
3. 大图分区加载SubsamplingScaleImageView
4. RecyclerView里的图片优化
   1. 加载过程中，使用Glide的pauseRequests()和resumeRequests()来控制加载的暂停和继续。
   2. 压缩图片，加载大图时才展示高清
   3. 限制图片宽高，不设置自适应，或者使用oss获取宽高
   4. Glide开启缓存

## WebView优化



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

1. 需要USB调试权限和root权限才能访问部分日志文件
2. 日志保留时间取决于设备型号和系统版本
3. 建议在发现崩溃后尽快收集日志，避免被覆盖