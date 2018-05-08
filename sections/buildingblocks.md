# Sections构建模块
Sections API 提供几种内置的Section可以作为几乎任何类型页面的构建模块。

大多数滚动页面通常可以描述成同种类型交替出现条目的分组。举个例子，假设一个按照字母序排列的联系人列表，由显示联系人姓名首字母的头部分成各个分组。基于这样的假设，Sections API 包含两种DiffSectionSpec 的实现，通过结合使用两者可以表示几乎任何页面结构：SingleComponentSection 和 DataDiffSection。

## SingleComponentSection

SingleComponentSection 是 Sections 层级结构中最简单的Section，可被用来描述复杂列表中的一行。顾名思义，这个 Section 用来渲染单一组件，把这个单一组件作为仅有的 prop 传入这个 Section。

SingleComponentSection 的典型用例是在列表尾部加一个加载转轮：

	final Section loadingSection = SingleComponentSection.create(c)
	    .component(Progress.create(c).build())
	    .build();

## DataDiffSection

DataDiffSection 用来表示同类数据的列表。需要传给DataDiffSection 最少的信息就是需要渲染的列表项以及需要在列表中渲染每一个条目的回调。

	@GroupSectionSpec
	class MyGroupSection {
	
	  @OnCreateChildren
	  static Children onCreateChildren(
	      SectionContext c,
	      @Prop ImmutableList<MyModel> dataModel) {
	      return Children.create()
	          .child(DataDiffSection.create(c)
	              .data(dataModel)
	              .renderEventHandler(MyGroupSection.onRenderEdge(c)))
	          .build();
	    }
	
	  @OnEvent(RenderEvent.class)
	  static RenderInfo onRenderEdge(
	      SectionContext c,
	      @FromEvent MyModel model) {
	      return ComponentRenderInfo.create(c)
	          .component(MyModelItemComponent.create(c).item(model).build())
	          .build();
	  }
	}

DataDiffSection 是用来高效渲染页面中处理大量数据流的部分。需要被显示在屏幕上的某一位置的条目，section 框架会去检查接收到的新数据里，这个条目上距一次渲染之后是否发生变化。如果此位置的条目数据发生改变，section框架会为这个条目发送一个 RenderEvent ，DataDiffSection 会用 RenderEvent 处理器创建组件并显示这个条目。

DataDiffSection 的实现使用Android 的 DiffUtil，以决定给定新旧列表数据时需要执行更新 UI 操作的最小集合。 

默认情况下， DataDiffSection 通过检查实例的等同性然后调用数据列表中对象的 equals() 方法来会检测数据变化。但是，大多数情况下，数据模型会比较复杂，需要通过自定义的方式声明如何对数据项进行比较，基于这种比较，在决定何时更新数据。通过给 DataDiffSection 传入两个事件处理器就可以轻松实现这种比较： OnCheckIsSameItemEvent 和 OnCheckIsSameContentEvent。

当 DataDiffSection 需要计算新旧列表数据的变化时，它会去检查两个数据项是否代表同一个数据，只有当两个数据代表同一条数据的情况下，才会去检查此数据项的内容是否发生变化。

下面展示了如果想要向 DataDiffSection 提供自己的比较方法，如何修改上面的例子：

	@OnCreateChildren
	Children onCreateChildren(
	    SectionContext c,
	    @Prop ImmutableList<MyModel> dataModel) {
	  return Children.create()
	    .child(DataDiffSection.create(c)
	        .data(dataModel)
	        .onCheckIsSameItemEventHandler(MyGroupSection.onCheckIsSameItem(c))
	        .onCheckIsSameContentEventHandler(MyGroupSection.onCheckIsSameContent(c))
	        .renderEventHandler(MyGroupSection.onRenderEdge(c)))
	    .build();
	}
	
	
	@OnEvent(OnCheckIsSameItemEvent.class)
	boolean onCheckIsSameItem(
	    SectionContext c,
	    @FromEvent MyModel previousItem,
	    @FromEvent MyModel nextItem) {
	  return previousItem.getId() == nextItem.getId();
	}
	
	@OnEvent(OnCheckIsSameContentEvent.class)
	boolean onCheckIsSameContent(
	    SectionContext c,
	    @FromEvent MyModel previousItem,
	    @FromEvent MyModel nextItem) {
	  return MyModel.compareContent(previousItem, nextItem);
	}

## Litho 集成：RecyclerCollectionComponent

想要简易集成 Litho ，sections 框架提供可以渲染 Sections 层级结构的内置组件，叫做 RecyclerCollectionComponent 。

RecyclerCollectionComponent是 Litho 的一个常规组件，接受 Section 作为prop，包含有创建底层结构的逻辑，此逻辑用于渲染嵌入到Section 层级结构中的组件， 同时此处的 Section 层级结构又存在于由 [RecyclerBinder](https://fblitho.com/javadoc/com/facebook/litho/widget/RecyclerBinder) 管理的 [Recyler](https://fblitho.com/docs/recycler-component) 中。

Section 层级结构成为了 RecyclerCollectionComponent 的“数据源”，同时列表中数据处理的复杂性例如插入、删除等，被底层结构隐藏和处理。