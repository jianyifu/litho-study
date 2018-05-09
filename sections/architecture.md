# Sections 架构

在核心部分，Sections框架负责从不可变的 props 以及 Sections 层次结构生成 [ChangeSet](https://fblitho.com/javadoc/com/facebook/litho/sections/ChangeSet.html) 。只要 SectionTree 设置了带有新 props 的 Section，或者每当层次结构中的 Section 更新内部状态并将新旧层次结构进行比较时，该框架就会创建新的 Section 层级结构来生成这些 ChangeSet 。

## SectionTree 是什么

使用 Sections 框架从创建 [SectionTree](https://fblitho.com/javadoc/com/facebook/litho/sections/SectionTree) 开始。 SectionTree 实例负责：

* 每当状态或者属性值发生改变时，计算以及重新计算更改。
* 与 [Target](https://fblitho.com/javadoc/com/facebook/litho/sections/SectionTree.Target) 的实现类通信，更新UI（包括将新的更改告知 Target）。

SectionTrees 创建必须带有 Target 的实现类。[Target](https://fblitho.com/javadoc/com/facebook/litho/sections/SectionTree.Target) 接口是负责 SectionsTree 与 UI 之间通信的API。在计算完一个 Section 层次结构的 ChangeSet 之后， SectionTree 实例会把这些更改传递给 Target .你可以为你想要的任何形式的自定义 UI 创建一个 Target ，但是 Sections 框架已经有了一些 Target 的实现类。 [SectionBinderTarget](https://fblitho.com/javadoc/com/facebook/litho/sections/widget/SectionBinderTarget) 就是 Target 的一个实现类，它将更改传递给RecyclerBinder进行渲染。

## 更新 SectionTree

只要任何 props 或状态值发生变化，框架就可以对 Sections 的结构执行增量和条件更新。基础架构还计算需要在现有层次结构上执行的最小操作，以更新列表反映新数据。

要更新 SectionTree 以反映新的 props ，请使用新的 props 值创建 Section 并调用 SectionTree＃setRoot（）。这也是在一棵SectionTree 上设置一个初始根节点的方式，因为它本质上是用一个空层次结构与新层次结构做差分。

要在状态值更改时更新 Section 树，只需执行 Litho [State](https://fblitho.com/docs/state) 文档中所述的常规状态更新。


您可能会注意到 setRoot（）和 updateState（）方法也有“异步”实现（ setRootAsync（）和 updateStateAsync（））。 * async（）方法将确保生成的ChangeSet计算在后台线程上执行。否则，所产生的ChangeSet计算将在任何 setRoot（）或 updateState（）被调用的线程同步完成。这就像Litho的[异步布局](https://fblitho.com/docs/asynchronous-layout#sync-and-async-operations)。

## 计算 ChangeSets

SectionTree实例分两步计算更改：根据 属性值或状态值 生成树，然后通过比较两棵树来创建变更集。

一棵 Section 树由单一的根 Section 生成，通过递归调用 GroupSecctionSpec 中的 @OnCreateChildren，直到抵达叶子部分， DiffSectionSpecs 。在遇到一个新的 Section 时，SectionTree 将：

* 创建一个新的SectionContext作用于这个新的 Section
* 检查当前层次结构中是否存在相应的 Section（通过 [key](https://fblitho.com/docs/state#keys-and-identifying-components) ），并将任何状态和service 值转移到新的 Section 。
* 检查新的 Section 是否有挂起的状态更新（通过 [key](https://fblitho.com/docs/state#keys-and-identifying-components) ），如果存在则执行更新。
* 通过调用 SectionLifecycle＃createChildren 创建新的子 Section ，并递归访问这些子 Section 。

生成新的树结构后，SectionTree 将递归遍历新树并将其与当前树进行比较以生成 ChangeSet 。这就是我们在 Diff Sections 上调用SectionLifecycle＃generateChangeSet的地方。遍历新树时，框架将局部索引转换为全局索引，因为它将所有 ChangeSet 合并为整个层次结构的单个 ChangeSet 。


注： [SectionContext](https://fblitho.com/javadoc/com/facebook/litho/sections/SectionContext) 是一个对象，用于将层次结构中的每个 Section 实例与其 SectionTree 关联。 SectionContext 实例在每次 SectionTree 重新计算其变化集（只要属性或状态发生变化）时被释放和重新创建。这意味着您不应该依赖传入 spec 委托方法的SectionContext去关联一个有效的Section实例。通常，SectionContext对象仅在@OnBindService和@OnUnbindService方法之间有效。您不应该在此窗口之外保留一个 SectionContext 的实例。

## SectionTree 和 RecyclerCollectionComponent

RecyclerCollectionComponent 是一个Litho组件，它可以在幕后创建一个 SectionTree 并将其绑定到一个 Recycler ，以便非常容易地使用Sections 框架。 RecyclerCollectionComponent 创建并保留一个 SectionTree 实例作为状态并暴露一个新属性来接受新 Sections 。使用 RecyclerCollectionComponent 时更新 SectionTree 非常简单，只需更传入的 section 属性。

