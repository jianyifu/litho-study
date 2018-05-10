# RecyclerCollectionComponent

[RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView) 是包含滚动项目列表的任何 Android 应用程序的基本构建块之一。 Litho 建议使用 RecyclerCollectionComponent 和 Sections 轻松构建滚动列表。借助这些 API，您可以构建从简单、同类数据的列表到复杂、异构的列表，支持多个数据源，同时可以利用例如异步布局和增量挂载等功能。

## 创建一个 RecyclerCollectionComponent

您可以像框架中的任何其他组件使用RecyclerCollectionComponent，并将其作为布局中的子项添加。

	@OnCreateLayout
	static Component onCreateLayout(
	    final ComponentContext c) {
	  return RecyclerCollectionComponent.create(c)
	      .section(createSection())
	      .build();
	}

这段代码最终将渲染成一个 RecyclerView ，每一行的内容由 Section 提供。您可以通过了解库中的某些[构建模块](https://fblitho.com/docs/sections-building-blocks)获得有关如何创建 Section 的更多信息。

## 一系列功能
RecyclerCollectionComponent 包含许多用于处理列表的实用功能。可以在 javadoc 中它可接收的属性列表，但这里有一些显着的特点：

### **<u>水平列表</u>**
RecyclerCollectionComponent 使用 RecyclerConfiguration prop来确定要使用的布局管理器。默认情况下，如果未指定此 prop，它将使用名为 ListRecyclerConfiguration 的 RecyclerConfiguration 实现，该实现将创建一个垂直方向的 LinearLayoutManager，供RecyclerCollectionComponent 使用。

对于水平布局，您可以传入一个水平方向ListRecyclerConfiguration：

	final RecyclerCollectionComponentSpec.RecyclerConfiguration
	      recyclerConfiguration =
	          new ListRecyclerConfiguration(
	              LinearLayoutManager.HORIZONTAL, false /* reverse layout */);
	
	final Component component =
	    RecyclerCollectionComponent.create(context)
	        .section(FooSection.create(new SectionContext(context)).build())
	        .recyclerConfiguration(recyclerConfiguration)
	        .build();


也可以使用 [GridRecyclerConfiguration](https://fblitho.com/javadoc/com/facebook/litho/sections/widget/GridRecyclerConfiguration) 创建一个网格列表。


### **<u>对齐方式</u>**
在水平滚动列表中，RecyclerCollectionComponent 的对齐模式也可以通过 ListRecyclerConfiguration 进行配置：

	final RecyclerCollectionComponentSpec.RecyclerConfiguration
	      recyclerConfiguration =
	          new ListRecyclerConfiguration(
	              LinearLayoutManager.HORIZONTAL, false /* reverse layout */, SNAP_TO_START);
	
	final Component component =
	    RecyclerCollectionComponent.create(context)
	        .section(FooSection.create(new SectionContext(context)).build())
	        .recyclerConfiguration(recyclerConfiguration)
	        .build();

其他对齐选项是SNAP_NONE，SNAP_TO_END和SNAP_TO_CENTER。

### **<u>设置水平RecyclerCollectionComponent的高度</u>**

水平的 RecyclerCollectionComponent 通过接收高度 prop 来设置高度。如果未指定高度，则高度为 0 。如果您不知道RecyclerCollectionComponent的高度，则可以通过启用 canMeasure prop 来配置让它自行决定其高度。这会测量 Sections 层次结构中的第一个子组件的高度，并把它作为整个 RecyclerCollectionComponent 的高度。

	final Component component =
	    RecyclerCollectionComponent.create(context)
	        .section(FooSection.create(new SectionContext(context)).build())
	        .recyclerConfiguration(recyclerConfiguration)
	        .disablePTR(true)
	        .canMeasureRecycler(true)
	        .build();

只要有可能，应该在 RecyclerCollectionComponent 上指定一个确切的高度，以避免额外测量确定其高度。

### **<u>下拉刷新</u>**
RecyclerCollectionComponen t默认启用下拉刷新，并向底层的 Recycler 发送一个事件处理器，以触发 SectionTree 的刷新。要禁用此功能，您需要将 disablePTR prop 设置为true：

	final Component component =
	    RecyclerCollectionComponent.create(context)
	        .section(FooSection.create(new SectionContext(context)).build())
	        .recyclerConfiguration(recyclerConfiguration)
	        .disablePTR(true)
	        .build();


### **<u>Loading, Empty, and error 的处理</u>**
通过 Section API，还可以通过 [Loading Events](https://fblitho.com/docs/communicating-with-the-ui#null__loadingstate-loadingstate) 和 [Services](https://fblitho.com/docs/services) 来进行数据获取。 RecyclerCollectionComponent 可以监听这些加载事件，并会做出相应的响应。通过 prop loadingComponent，emptyComponent和errorComponent，可以指定在获取数据时发生某些事情时要显示的内容：

* loadingComponent：正在加载数据，列表中没有任何内容
* emptyComponent：数据已完成加载并没有任何可显示的内容。
* errorComponent：数据加载失败，列表中没有任何内容。


		final Component component =
		    RecyclerCollectionComponent.create(context)
		        .section(FooSection.create(new SectionContext(context)).build())
		        .recyclerConfiguration(recyclerConfiguration)
		        .loadingComponent(
		            Progress.create(c)
		                .build())
		        .errorComponent(
		            Text.create(c)
		                .text("Data Fetch has failed").build())
		        .emptyComponent(
		            Text.create(c)
		                .text("No data to show").build())
		        .build();


你可以在[此处](https://fblitho.com/javadoc/com/facebook/litho/sections/widget/RecyclerCollectionComponent.html)查看 RecyclerCollectionComponent 支持的其他 props 。
