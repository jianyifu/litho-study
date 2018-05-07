# Sections教程

*注意* 这部分教程建立在[Litho教程](https://fblitho.com/docs/tutorial)基础之上。确保在此之前已经阅读那个教程的所有内容。

回想一下 我们在[Litho教程](https://fblitho.com/docs/tutorial) 留下了一个垂直滚动带有交替背景的列表。在这个教程中，我们会在原有列表的顶部加一个水平的滚动单元，通过修改ListSectionSpec 来充分利用 LItho和 Sections API的可组合性。

## 1. 重构 ListSectionSpec
我们来重新看一下ListSectionSpec。在for循环里的每个SingleComponentSection渲染几乎一样的组件，只有数字发生变化。换句话说，我们的每个ListItem 都依赖于同一个 int 型的模型对象。

Litho 提供了另外一个核心的Section叫做 DataDiffSection，专门用来渲染由列表组成的组件。在这步，我们用DataDiffSection来重构ListSectionSpec.

首先，生成模型对象。在这个例子中，我们只是在ListSectionSpec添加一个静态方法来生成模型对象。实际场景中，需要将生成数据替换成合适的数据获取逻辑。


	private static List<Integer> generateData(int count) {
	  final List<Integer> data = new ArrayList<>(count);
	  for (int i = 0; i < count; i++) {
	    data.add(i);
	  }
	  return data;
	}


接下来，写一个通过给定的模型对象创建ListItem的方法：

	@OnEvent(RenderEvent.class)
	static RenderInfo onRender(final SectionContext c, @FromEvent Integer model) {
	  return ComponentRenderInfo.create()
	      .component(
	          ListItem.create(c)
	              .color(model % 2 == 0 ? Color.WHITE : Color.LTGRAY)
	              .title(model + ". Hello, world!")
	              .subtitle("Litho tutorial")
	              .build())
	      .build();
	}

最后，把两个方法用DataDiffSection结合起来，替换掉我们当前在onCreateChildren中的代码：

	@OnCreateChildren
	static Children onCreateChildren(final SectionContext c) {
	  return Children.create()
	      .child(
	          DataDiffSection.create(c)
	              .data(generateData(32))
	              .renderEventHandler(ListSection.onRender(c)))
	      .build();
	}

哇哦，这个@OnEvent是什么东西？ListSection.onRender()是从哪里来的？

这里对所发生的事情进行快速的解释：

* 当一个列表条目需要被渲染的时候，DataDiffSection会产生一个RenderEvent。
* 创建DataDiffSection的时候，我们传入 自定义的RenderEventHandler ，ListSection.onRender(c)。
* 这个自定义事件处理器在接收到一个RenderEvent的时候，会调用在ListSectionSpec中定义的且包含正确参数的onRender方法。
* 所有的事件处理代码是由@OnEvent注解标注的代码生成的。

运行APP，看到的画面应该跟之前的一样：

<img src="https://fblitho.com/static/images/barebones4.png" style="zoom:30%">

## 2. 添加一个水平滚动列表

还记得我们是如何用RecyclerCollectionComponent来创建列表的么？应为RecyclerCollectionComponent自身也是一个Component，所以我们可以在Section里再创建一个列表，轻松实现嵌套列表。在一个垂直的列表中再嵌套一个垂直的列表是没有意义的，比较常见的是在垂直列表中嵌入一个水平列表。接下来我们就要这么做。

更新 onCreateChildren()中的代码，在DataDiffSection前面加一个SingleComponentSection：

	@OnCreateChildren
	static Children onCreateChildren(final SectionContext c) {
	  return Children.create()
	      .child(
	          SingleComponentSection.create(c)
	              .component(
	                  RecyclerCollectionComponent.create(c)
	                      .disablePTR(true)
	                      .recyclerConfiguration(new ListRecyclerConfiguration(LinearLayoutManager.HORIZONTAL, /*reverse layout*/ false, SNAP_TO_CENTER))
	                      .section(
	                          DataDiffSection.create(c)
	                              .data(generateData(32))
	                              .renderEventHandler(ListSection.onRender(c))
	                              .build())
	                      .canMeasureRecycler(true))
	              .build())
	        .child(
	            DataDiffSection.create(c)
	                .data(generateData(32))
	                .renderEventHandler(ListSection.onRender(c)))
	        .build();
	  }

这里我们看到RecyclerCollectionComponent的几个新props:

* recyclerConfiguration接受一个配置对象用于设置组件的布局以及RecyclerCollectionComponent的对齐方式。
* canMeasureRecycler对于没有固定高度的水平RecyclerCollectionComponent来说须设置成true。RecyclerCollectionComponent会测量第一个孩子的高度并将此高度作为整个水平列表的高度。

运行APP，应该会看到如下画面：

<img src="https://fblitho.com/static/images/barebones5.gif" style="zoom:80%">

## 总结

恭喜完成此教程的第二部分！这部分教程引入了Sections更高级的应用以及一些附加构建模块帮助实现复杂滚动UI。还可以尝试一些更有趣的东西：

* SingleComponentSection有一些有趣的props。看看是否可以把水平列表固定在列表顶部。
* 在创建每行的组件时，RenderEvent还有一些有帮助的属性，看看是否可以每三个条目出现一个水平列表。

可以在此处查看[完整的教程](https://github.com/facebook/litho/tree/master/sample-barebones)。确保更进一步了解 [这个例子](https://github.com/facebook/litho/tree/master/sample)以及Litho API 文档以查找更多信息。
