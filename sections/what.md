# Sections是什么


Litho主要是一个渲染API，但高效的UI渲染仅仅是编写高性能页面所面临挑战的一部分。如果仔细查看设备上的应用程序，会注意到它们许多都以可滚动的页面为中心，该页面可以获取并显示数据列表。当使用RecyclerView构建列表时，必须考虑如何维护适配器与数据同步，并将数据变化通知给适配器。这通常需要大量的手动索引处理，并且导致状态维护以及硬性代码的存在，随着产品不断迭代更新，这样的代码很难维护和追溯。RecyclerView适配器也很难对数据源进行组合，同时在同一个页面内整合多种数据源又是必要的。

Sections构建在Litho之上，为编写高度优化的列表页提供声明性和可组合的API。虽然Litho Components用于显示UI部分，但Sections是构建数据并将其转换为Litho Components的一种方式。如果将页面形象化比作组件树，则树的根节点和子树的根节点就是Sections，而树叶是Litho组件，代表在屏幕上显示的各个项目。

![Tree model](https://fblitho.com/static/images/sections-intro.png)

Sections 使用与Litho同样的声明式数据模型，在后台透明处理例如计算数据更新的最小变更集合以及细粒度UI刷新。作为Litho的一部分，Sections API 同样采用例如基于注解的代码生成、事件处理、props以及 状态更新等主要概念。

为了方便集成Litho，Sections 框架提供预定义Component用于渲染Sections层级结构，叫做[RecyclerCollectionComponent](https://fblitho.com/javadoc/com/facebook/litho/sections/widget/RecyclerCollectionComponent.html)。Sections 层级结构成为 RecyclerCollectionComponent的数据源，用于渲染数据的Components在后台成为RecyclerView Adapter中的条目。所有在Adapter中的操作，例如插入和删除等的复杂性，都被底层处理及隐藏。

[Litho教程](https://fblitho.com/docs/tutorial)简单介绍了如何把Section集成到LithoView内。更多详情，参考[Sections 教程](https://fblitho.com/docs/sections-tutorial) 了解如何创建和组合 Sections。