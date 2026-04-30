
Excalidraw白板工具和快捷键

## 1. 工具简要介绍

### 1.1. excalidraw

[Excalidraw](https://excalidraw.com/) 是一个开源的无限画布的手绘效果白板绘图工具，支持多人协作和端到端加密。

Excalidraw 的作者是一位目前在 Facebook 工作的法裔前端开发工程师 Vjeux。除了是 Excalidraw 的作者之外，还是 React Native、Prettier 的联合创始人，也是 CSS-IN-JS、Yoga、React Conf 的创建者。

Excalidraw 也是Github开源[项目](https://github.com/excalidraw/excalidraw)，从2020年开源开始，Star数一路直线增长。截止今天(2024-06-16)已达`75.7K`。

值得一说的是excalidraw里还增加了AI功能，可以根据文字生成控件或者流程图。之前[设计模式示例博客](https://xiaodongq.github.io/2024/05/12/design-pattern-2-3-factory/)里的图就是自动生成的：  
![23种设计模式](/images/2024-05-12-20240512100608.png)

### 1.2. 中文手绘效果

有开源作者基于excalidraw，二次开发增强了“中文手绘效果”和“多画布”能力，分别创建了 [handraw](https://handraw.top/) 和 [revezone](https://revezone.com/index.html)，后者还增加了目录和多标签功能。

字体效果对比（这样看自我感觉还是handraw舒服点，revezone的多目录和多标签看起来也不错）：  
![字体效果对比](/images/2024-06-16-excalidraw-compare.png)

另外用之前画的一张流程图做效果对比：

* excalidraw

![excalidraw示意图](/images/tcp-connect-close.png)

* handraw

上面的`.excalidraw`文件直接拖到网页打开：

![handraw示意图](/images/2024-06-16-handdraw_demo.png)

详情可参考：[Handraw: 支持中文手写效果的 Excalidraw](https://sspai.com/post/80459#comment-364471)

* revezone

![revezone示意图](/images/2024-06-16-revezone_demo.png)

详情可参考：[Revezone: 换个姿势使用Excalidraw手绘效果白板](https://sspai.com/post/82630#!)

本篇主要是记录一下快捷键，便于时常索引。

## 2. 快捷键

最近把各个快捷键（主要是左手边的）都试了一下，左手放键盘上，右手拿鼠标，画图时自我感觉特别拉风，找到了以前体验Vim和SecureCRT快捷键的感觉。

主要是`R`、`A`、`T`、`E`、`V`，还有手动调整时利用`Alt`防止箭头调整歪了。

### 2.1. 工具快捷键

* R → Rectangle，矩形
* D → Diamond，菱形
* O → Oval，椭圆
* A → Arrow，箭头
* T → Text，文字
* P → Pan，画笔
* E → Eraser，擦除
* F → Full screen，全屏
* Shift + H → Flip horizontal，水平翻转
* Shift + V → Flip vertical，垂直翻转

### 2.2. 鼠标快捷键 - `Shift`

长按 Shift 再执行某项操作：

* Shift + 拖动元素 → 水平或纵向移动
* Shift + 画矩形 → 正方形
* Shift + 画菱形 → 正方形
* Shift + 画椭圆 → 正圆
* Shift + 拖动图形控制点 → 等比例缩放
* Shift + 画箭头/划线 → 特定角度箭头/直线
* Shift + 拖动直线控制点变为孤形 → 特定位置

### 2.3. 鼠标快捷键 - `Alt`

长按 Alt （Mac下是 Option，Windows下是 Alt）再执行某项操作：

* Alt + 画矩形/菱形/椭圆 → 从中心缩放
* Alt + 拖动矩形/菱形/椭圆控制点 → 从中心缩放
* Alt + Shift + 拖动矩形/菱形/椭圆控制点 → 从中心等比缩放
* Alt + 拖动图形 → 复制出同样图形并移动

### 2.4. 快速选择颜色

选中图形后通过快捷键 G（Background）或者 S （Stroke）调出颜色选择面板，每种颜色对应的快捷键刚好是左手键盘区域。

### 2.5. 成组与取消成组

成组与取消成组：将多个元素合并成组后，就可以像操作单个图形一样来操作图组了。（Mac下为 Command，Windows下为 Ctrl）

* Ctrl + G → 合并成组
* Ctrl + Shift → 取消成组
* Ctrl + 鼠标单击 → 选中子级图形

### 2.6. Vimium插件快捷键冲突说明

Chrome浏览器开启了Vimium插件，有些键冲突。在设置中的`Custom key mappings`里去掉相关快捷键：

```sh
unmap a
unmap d
unmap e
unmap q
unmap r
unmap t
unmap v
unmap g
unmap s
```

## 3. 参考

1、[风靡科技圈的白板工具 Excalidraw 上手教程](https://www.yangqi.show/posts/excalidraw-tutorial)

2、[Handraw: 支持中文手写效果的 Excalidraw](https://sspai.com/post/80459#comment-364471)

3、[Revezone: 换个姿势使用Excalidraw手绘效果白板](https://sspai.com/post/82630#!)
