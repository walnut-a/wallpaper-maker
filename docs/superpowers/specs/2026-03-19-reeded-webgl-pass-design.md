# 长虹玻璃 WebGL Pass 设计文档

## 1. 背景

当前 `reeded-column` 已经从“居中的闭合柱体”改成了更偏全画面介质场的做法，但视觉上还是停在 `Canvas 2D` 的近似层：

- 棱线存在了，但介质感还不够强
- 折射和亮度压缩还是偏涂抹
- 预览里有玻璃提示，但不够像真正穿过一层材料

公开实现里，`liquid-glass`、`liquidGL` 这类项目提供了很好的参考方向：不是单纯加条纹，而是把背景图像送进一个独立 pass，再做位移、亮暗调制和局部聚焦。

这次迭代不再继续堆 2D 近似，而是只给 `长虹玻璃` 增加一个全屏 WebGL pass。

## 2. 目标与非目标

### 2.1 目标

- 只在 `shapeMode === "reeded-column"` 时启用一个独立的全屏 WebGL pass
- 让 `长虹玻璃` 更像“整张画面都穿过同一种介质”
- 让预览结果和 `PNG` 导出继续保持一致
- 保持单文件、纯前端、静态部署的项目结构不变
- 没有 WebGL 时自动回退到现有 2D 长虹玻璃实现

### 2.2 非目标

- 这次不把整个渲染系统改成 WebGL
- 不改其他主体图形的渲染路径
- 不引入 npm 依赖或构建工具
- 不追求照片级物理写实
- 不把现成 GitHub 玻璃库直接塞进项目

## 3. 方案比较

### 方案 A：继续只用 Canvas 2D 近似

优点：

- 结构最简单
- 不增加新的渲染上下文

缺点：

- 视觉上限已经很明显
- 很难做出更可信的折射和介质场

### 方案 B：直接接现成玻璃库

参考：

- [shuding/liquid-glass](https://github.com/shuding/liquid-glass)
- [naughtyduk/liquidGL](https://github.com/naughtyduk/liquidGL)

优点：

- 快速看到玻璃效果

缺点：

- 这些实现更偏 DOM 或组件层效果
- 很难和当前 `Canvas` 导出链路严丝合缝对齐
- 后续参数体系会被外部库牵着走

### 方案 C：保留 Canvas 主链，只给长虹玻璃加 WebGL pass

优点：

- 只改一条渲染分支，范围可控
- 预览和导出可以共用同一条结果链
- 可以直接把当前参数映射进 shader

缺点：

- 需要自己维护一个很小的 WebGL 渲染器
- 需要处理浏览器不支持时的回退

### 推荐

采用 `方案 C`。

## 4. 总体设计

### 4.1 渲染顺序

当主体不是 `reeded-column` 时，继续走现有流程：

1. `drawBackgroundMode`
2. `drawShapeMode`
3. `drawAccentMode`
4. `drawTextureMode`

当主体是 `reeded-column` 时，改成：

1. 先把背景画进一个中间 2D 画布
2. 用 WebGL pass 读取这张背景图
3. 在 fragment shader 里做长虹玻璃介质场
4. 把 shader 输出绘回目标 2D canvas
5. 再执行现有的 `accent` 和 `texture`

### 4.2 介质场的核心信号

这个 pass 只抓最值钱的 4 件事：

1. 纵向棱线位移  
   用周期性的横向采样偏移，让背景看起来被竖向玻璃肋压缩和拉伸。

2. 呼吸感  
   棱线间距和强度要有轻微起伏，避免机械平铺。

3. 局部增强区  
   一到两个纵向偏移更强、亮部更聚焦的区域，用来形成视觉重心。

4. 雾场融合  
   玻璃层不是硬贴在背景上，而是和背景亮暗一起漂在一个更柔的场里。

## 5. 参数映射

现有参数继续沿用，不改 UI 结构：

- `ridgeDensity` -> 棱线频率
- `ridgeBreath` -> 频率调制与强弱起伏
- `refractionField` -> 主位移幅度
- `focusField` -> 局部增强区强度
- `fieldBlend` -> 雾场融合和整体柔化

这些参数在 shader 里只映射到：

- 波形频率
- 位移幅度
- 局部 mask 强度
- 亮度抬升和对比压缩

不再继续引入新的长虹玻璃专属参数。

## 6. 技术结构

### 6.1 新增的最小单元

在 `index.html` 内增加一个很小的 WebGL 子系统：

- `getReededShaderRenderer()`
- `createReededShaderRenderer()`
- `renderReededGlassPass()`

职责划分：

- `get...` 负责懒加载和缓存 renderer
- `create...` 负责创建隐藏 canvas、program、buffer、uniform 绑定
- `render...` 负责接收背景画布和参数，输出处理后的结果

### 6.2 输入与输出

输入：

- 已经画好的背景 2D canvas
- 输出尺寸
- 当前调色板
- 当前 `advanced` 参数
- 当前随机种子派生出的局部增强区数据

输出：

- 一张已经过长虹玻璃 pass 的位图

### 6.3 回退策略

如果下面任一条件不满足，就回退到当前 2D 实现：

- 浏览器拿不到 `webgl`
- shader 编译失败
- texture 上传失败

回退时不报错打断，只在内部静默切换。

## 7. 导出一致性

这一点是这次方案的核心约束。

预览和导出必须走同一条长虹玻璃 pass，而不是：

- 预览走 WebGL
- 导出回到旧的 2D 模拟

具体做法：

- `renderArtwork()` 接收目标 `ctx` 后，内部创建临时中间画布
- 无论是预览尺寸还是导出尺寸，长虹玻璃都调用同一个 `renderReededGlassPass()`
- `exportPng()` 不需要改下载策略，只要继续导出最终 2D canvas

## 8. 验收标准

### 8.1 视觉

- 第一眼是“整张壁纸都经过同一种纵向介质”
- 局部增强区存在，但不会读成独立闭合物体
- 放到手机屏幕上时，玻璃棱线和明暗压缩仍然能被看见

### 8.2 功能

- `reeded-column` 预览正常
- 参数变化能即时影响预览
- `PNG` 导出成功，结果与预览方向一致
- Safari / `file://` 的导出兼容保持不变
- 无 WebGL 时能自动回退到旧路径

### 8.3 工程

- 只新增最小量的 WebGL 代码
- 不影响其他主体图形
- 控制台没有新的报错
