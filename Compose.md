- enableEdgeToEdge 
	- 沉浸式全屏
- Scaffold
	- 一个Material风格的布局骨架，包含了topBar，bottomBar，floatingActionButton，content等。
### 性能优化与最佳实践
### 1. **避免过度绘制（Overdraw）**

- 使用 `Modifier.graphicsLayer(alpha = 0.5f)` 替代透明背景，减少图层叠加。
- 避免嵌套过多 `Box` 或 `Column`，优先使用 `Scaffold` 提供的标准布局。

### 2. **状态管理优化**

- 使用 `derivedStateOf` 计算衍生状态，减少不必要的重组。
- 对于复杂状态，结合 `ViewModel` 和 `LiveData` 实现数据持久化。

### 3. **资源适配**

- 图片资源建议使用 `ImageBitmap.imageResource` 加载本地资源。
- 网络图片推荐使用 [Coil](https://link.juejin.cn?target=https%3A%2F%2Fcoil-kt.github.io%2Fcoil%2F "https://coil-kt.github.io/coil/") 或 [Glide](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fbumptech%2Fglide "https://github.com/bumptech/glide") 库。



## 三个阶段

### 组合 composition

组合阶段会执行可组合函数,并输出表示界面的树结构.

如果内容,大小,布局都没变,可能会跳过组合阶段.

### 布局 (measure+layout)

布局阶段进行界面树的每个节点的测量和放置.布局结束后,每个布局节点都会被分配宽高和应该绘制到的坐标.

### 绘制 draw

### 状态读取

在上述三个阶段中读取快照的state的value时,compose都会自动跟踪当时正在执行的操作,以此为基础实现了compose对状态的观察.

val padding: Dp by remember { mutableStateOf(8.dp) }

上述每个阶段的状态读取都会影响当前阶段和后续阶段.

## side-effects

**附带效应**是指发生在可组合函数作用域之外的应用状态的变化.

### LaunchedEffect

- 在某个可组合项的作用域内运行挂起函数
- 可以传入多个key,每个key发生变化时,LaunchedEffect都会重启.
- 传入Unit时只启动一次.

### rememberCoroutineScope

- 多用于在回调里开启协程比如点击事件等.
- LaunchedEffect是可以composable函数,而rememberCoroutineScope可以在可组合函数之外使用.

### rememberUpdateState

LaunchedEffect的key每次发生变化都会导致LaunchedEffect重启,rememberUpdateState...

### DisposableEffect

需要清理的效应.

### SideEffect

将Compose状态发布为非Compose状态

### produceState

将非Compose状态转换为Compose状态.

### derivedStateOf

将一个或多个状态对象转换为其他状态.

在compose中,每次观察到状态对象或可组合输入变化时都会发生重组,这有时候是不必要的(有时候超过某个阈值才需要对其作出响应.)

**注意：**`derivedStateOf` 的成本较高，只应该在结果没有变化时用来避免不必要的重组。

### snapshotFlow

将compose的state转换为冷流.

## 组件

### TextFiled

输入框组件.

```kotlin
placeholder = hint,
可以设置首尾icon,
colors里设置所有颜色,modifier里设置颜色无效.
modifier里可以设置border,
visualTransformation属性设置可见性,
keyboardOptions设置是否为密码

常规的EditText里需要设置transformationMethod和inputType,然后还要移动光标
 et.text?.length?.let { et.setSelection(it) }
```

- TextFiled为什么输入后不会自动更新?

  compose是声明式工具集,更新他的唯一方法是通过新的参数调用同一可组合函数,这些参数也就是界面状态的表现形式,每当状态更新了,会发生重组,才会进行更新.

### CenterAlignedTopAppBar

顶部标题栏 , 左icon,中标题,右action

### LazyVerticalStaggeredGrid

它类似于 `LazyVerticalGrid`，但允许不同高度的项目交错排列.

### NavigableListDetailPaneScaffold

具有导航功能的列表-详情界面布局的组件(主从布局,能自动适配手机和pad等大屏设备)

### AlertDialog

### FlowRow

Flow布局,自动换行

## 布局

### Box

### Column

### Row

### 通用属性



### 基础组件

Column,Row

Text

Image

​	1. 圆角效果

TextFiled

## 装饰

### Modifier

```kotlin 
.width(IntrinsicSize) fillMaxWidth wrapContentWidth(Alignment.CenterHorizonally) heightIn 最小值
.clip(Shape)  
.backgroud()  //必须先裁剪,再设置背景.
.border()	
.alpha()
```



1. 会话列表优化
2. 限制向非官方客服发起私聊
3. 聊天室消息优化评审



### 能力

#### LocalContext

#### LocalUriHandler

浏览器中打开http地址

localUriHandler.open("")



## Material Design

### Color

​	color = MaterialTheme.colorScheme,已封装的主题颜色

### Typography

​	stype = MaterialTheme.typography...

Shape



### 状态

#### remember

#### Flow

collectAsStateWithLifecycle()可以安全的从ViewModel中使用流,另外LiveData,和RxJava也都有对应的API.

### 



### remembersavebale  // 配置更改也不改变

### 动画

val extraPadding = by animateDpAsState {

​	targetValue = 

​	animationSpec = tween(

​		duratioin = 2000

​	)

}

### 资源使用

​	dimentionResources

​	



ComposeView

AndroidView