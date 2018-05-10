# 增量挂载

尽管组件提供了更加平坦的视图层次结构并在[主线程之外执行布局计算](https://fblitho.com/docs/asynchronous-layout)，但对于非常复杂的组件（尤其是包含许多视图的组件），挂载操作（创建，回收和挂接View 和 Drawable ）仍然会在UI线程中产生成本。

Litho可以透明地分摊在UI框架上挂载组件的成本，以避免混乱。

启用增量挂载（默认情况下），LithoView 将仅挂载足够的内容以填充其可见区域并解挂（和回收）不再可见的组件。

![增量挂载图](https://fblitho.com/static/images/incremental-mount.png)

如果使用Litho的 [异步RecyclerView](https://fblitho.com/docs/recycler-component) 支持，该框架将无缝执行增量挂载。

