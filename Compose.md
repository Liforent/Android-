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