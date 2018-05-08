# Sections 与 Views

当与 Litho Component 提供的渲染优化相结合时，Sections 最能够发挥其作用。但 Sections API 也支持渲染 Android 传统的 View 。这样过渡到 Sections 会比较容易一些，不管产品的UI是使用Android 传统的 View 还是 Litho 的组件，亦或是两者的混合，仍然可以利用 Litho 的性能优势。

目前只有 DataDiffSection 支持 View 的渲染。我们再看一下 [DataDiffSection](https://fblitho.com/docs/sections-building-blocks#datadiffsection) 的例子，回顾一下针对某一个条目，你是如何声明框架需要渲染什么。

	@GroupSectionSpec
	class MyGroupSectionSpec {
	
	  @OnCreateChildren
	  static Children onCreateChildren(
	      SectionContext c,
	      @Prop ImmutableList<MyModel> dataModel) {
	      return Children.create()
	          .child(DataDiffSection.create(c)
	              .data(dataModel)
	              .renderEventHandler(MyGroupSection.onRenderEvent(c)))
	          .build();
	  }
	
	  @OnEvent(RenderEvent.class)
	  static RenderInfo onRenderEvent(
	      SectionContext c,
	      @FromEvent MyModel model) {
	      return ComponentRenderInfo.create(c)
	          .component(MyModelItemComponent.create(c).item(model).build())
	          .build();
	  }
	}

当一个条目需要被渲染在屏幕上，框架会分发一个 RenderEvent ，调用作为 DataDiffSection prop 的事件处理器 ，针对该条目创建一个 RenderInfo 。 RenderInfo 携带让框架知道某一个条目该如何被渲染的信息。

RenderInfo 有两种实现： [ComponentRenderInfo](https://fblitho.com/javadoc/com/facebook/litho/widget/ComponentRenderInfo.html) 和 [ViewRenderInfo](https://fblitho.com/javadoc/com/facebook/litho/widget/ViewRenderInfo.html) 。

在之前的例子中我们已经了解到怎样用 ComponentRenderInfo 去声明  Litho 的Component 如何渲染条目。如果想要用 Android 中的 Views 来渲染，只需要在 RenderEvent 的事件处理器中返回 ViewRenderInfo 的实例即可。

	@GroupSectionSpec
	class MyGroupSectionSpec {
	
	  private static ViewCreator VIEW_CREATOR =  
	      new ViewCreator<MyView>() {
	          @Override
	          public MyView createView(Context c, ViewGroup parent) {
	              // this call is equivalent to onCreateViewHolder()
	              return new MyView(c);
	          }
	      };
	
	  @OnCreateChildren
	  static Children onCreateChildren(
	      SectionContext c,
	      @Prop ImmutableList<MyModel> dataModel) {
	      return Children.create()
	          .child(DataDiffSection.create(c)
	              .data(dataModel)
	              .renderEventHandler(MyGroupSection.onRenderEvent(c)))
	          .build();
	  }
	
	  @OnEvent(RenderEvent.class)
	  static RenderInfo onRenderEvent(
	      SectionContext c,
	      @FromEvent MyModel model,
	      @FromEvent int index) {
	      return ViewRenderInfo.create(c)
	          .viewCreator(VIEW_CREATOR)
	          .viewBinder(
	              new SimpleViewBinder<MyView>() {
	                  @Override
	                  public void bind(MyView view) {
	                    // this call is equivalent to onBindViewHolder()
	                  }
	              })
	          .build();
	  }
	}

ViewRenderInfo 有两个必须要传的属性： ViewCreator 和  ViewBinder 。


ViewCreator 和  ViewBinder 与 RecyclerView.Adapter 中的 onCreateViewHolder 和 onBindViewHolder 在逻辑上是等价的。

在 RecyclerView 中使用同一个 ViewCreator 创建的 View 会被回收在同一个缓冲池中。可以为不同的 View 类型创建一个 ViewCreator 的静态实例，在 Sections 中把这个静态实例传递给 .viewCreator() 方法以确保有效复用。使用 model 或者 index 来决定在多种View 类型中返回合适的 ViewCreator 实例。

框架提供 ViewBinder 的空实现，叫做 [SimpleViewBinder](https://fblitho.com/javadoc/com/facebook/litho/viewcompat/SimpleViewBinder.html) ，可以在只需要实现某一个 ViewBinder 中的方法时使用，通常是 bind(View) 。

# Components 与 Views 的混合

如果Section 需要渲染的项目既有 Litho Components 也有 Android 的Views ,可以在 RenderEvent 的事件处理器中返回对应的 RenderInfo 实现。下面看一下如何做：

	@GroupSectionSpec
	class MyGroupSectionSpec {
	
	  private static ViewCreator VIEW_CREATOR =  
	      new ViewCreator<MyView>() {
	          @Override
	          public MyView createView(Context c, ViewGroup parent) {
	              // this call is equivalent to onCreateViewHolder()
	              return new MyView(c);
	          }
	      };
	
	  @OnCreateChildren
	  static Children onCreateChildren(
	      SectionContext c,
	      @Prop ImmutableList<MyModel> dataModel) {
	      return Children.create()
	          .child(DataDiffSection.create(c)
	              .data(dataModel)
	              .renderEventHandler(MyGroupSection.onRenderEvent(c)))
	          .build();
	  }
	
	  @OnEvent(RenderEvent.class)
	  static RenderInfo onRenderEvent(
	      SectionContext c,
	      @FromEvent MyModel model) {
	      if (model.canRenderWithComponent()) {
	        return ComponentRenderInfo.create(c)
	            .component(MyModelItemComponent.create(c).item(model).build())
	            .build();
	      }
	
	      return ViewRenderInfo.create(c)
	               .viewCreator(VIEW_CREATOR)
	               .viewBinder(
	                   new SimpleViewBinder<MyView>() {
	                       @Override
	                       public void bind(MyView view) {
	                         // this call is equivalent to onBindViewHolder()
	                       }
	                   })
	               .build();
	  }
	}

