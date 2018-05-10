# 铺平 View 结构


让我们来看看下面例子中的布局。它包含一个图像，一个标题和副标题。使用传统的 Android 视图系统，您可以查看所有这些元素，并将其包含在几个View Group 中对这些元素进行排列。

![铺平 View 结构](https://fblitho.com/static/images/viewflatteningnobounds.jpeg)

Litho 会自动减少最终用户界面层次结构中的视图数量。布局计算步骤产生的布局树只是 UI 的图纸，不会与 Android 视图直接耦合。这允许框架在挂接组件之前处理布局树以获得最佳的渲染性能。
我们以两种方式来做到这一点。

首先，Litho 可以在布局计算后完全跳过容器，因为它们不会在挂载步骤中用到。在我们的例子中，挂载时将不会有一个包含标题和副标题的 View Group。

其次，Litho 可以挂载 View 或 Drawable 。事实上，框架中的大多数核心小部件（例如Text和Image）都挂载成Drawable对象，而不是 View  。

这些优化的结果，示例中UI的组件实际上将呈现为单一的完全平坦的视图。您可以在以下屏幕截图中看到这一点，并[启用显示布局边界开发者选项](https://fblitho.com/docs/developer-options#debughighlightmountbounds)。

![铺平 View 结构](https://fblitho.com/static/images/viewflattening.png)

虽然平坦的视图结构对内存使用和绘图时间有重要的好处，但它们并非万能的灵丹妙药。Litho 有一个非常通用的系统来自动“ unflatten（堆叠起来） ” 已挂载组件的层次结构，当使用 Android View 里一些不可或缺的功能，如触摸事件的处理，无障碍功能，或限制失效等。例如，如果要在示例中启用单击图像或文本，则框架会自动将它们包装在视图中，前提是它们具有[点击处理程序](https://fblitho.com/docs/events-overview#callbacks)。