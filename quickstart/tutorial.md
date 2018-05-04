# 教程

此教程假定读者已经完成新手入门的学习，确保已经设置Litho的核心库以及Sections库。

在这个教程中，由在屏幕上创建基本的“Hello World!”开始，然后逐渐过渡到创建“Hello World!”列表。在这个过程中，将会了解到有关Litho的构建模块：[Component](https://fblitho.com/javadoc/com/facebook/litho/Component)，以及[LithoView](https://fblitho.com/javadoc/com/facebook/litho/LithoView)的有关知识,也会学习到如何使用Section创建列表以及如果设置组件的属性。

## Hello World

最开始的阶段，在一个视图上展示“Hello World!”。

首先，在应用的Application类中初始化SoLoader:

	public class SampleApplication extends Application {
	
	  @Override
	  public void onCreate() {
	    super.onCreate();
	
	    SoLoader.init(this, false);
	  }
	}

Litho使用[Yoga](https://yogalayout.com/)来进行底层的布局,Yoga有有一些本地依赖库，[SoLoader](https://github.com/facebook/SoLoader)就是用来加载这些本地依赖库的。在此处初始化SoLoader可以保证随后的开发中不会引用到未加载的库。

接下来，在Activity中添加Litho中预定义的[Text](https://fblitho.com/javadoc/com/facebook/litho/widget/Text)组件：

	@Override
	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	
	    final ComponentContext context = new ComponentContext(this);
	
	    final Component component = Text.create(context)
	        .text("Hello World")
	        .textSizeDip(50)
	        .build();
	
	    setContentView(LithoView.create(context, component));
	}

LithoView 是一个Android中的ViewGroup,可以渲染组件；它是Litho中Component与Android中View的桥梁，这个例子把Activity的内容设置为一个LithoView，用来展示一个Text组件。

那么，Component是如何发挥作用的呢？我们来仔细研究一下下面这段代码：

	Text.create(context)
	    .text("Hello World")
	    .textSizeDip(50)
	    .build();

Text是在com.facebook.litho.widget包中定义的一个核心组件。包含一些像上面显示的属性，例如 *text* 和 *textSize*。这些属性叫做*props*，是受[React](https://reactjs.org/)中术语的启发而来的。随后将会学习如何编写自己的组件，但是值得注意的是，这里的Text 类是有一个叫做TextSpec的类生成的。这个生成的组件类提供一个建造者模式的API，通过方法为组件设置属性值。

这个例子中，Text组件作为唯一的一个子组件被添加到LithoView中。可以用唯一的根组件带有多个子组件的形式代替例子中的组件，在接下来的介绍中将会了解如何去做。

一切准备就绪！运行APP，应该看到以下画面：

 <img src="https://fblitho.com/static/images/barebones1.png" style="zoom:30%" alt="Hello World!"  />

虽然不尽完美，但所有需要的确是个不错的开始！

## 第一个自定义组件

在这个教程结束之后，将会得到一个简单的可以滑动的列表。这个列表只是多次显示一个带有标题和副标题的条目。令人激动时刻！在这部分教程中，要编写一个简单的组件，那就是列表中的条目。当然，实际的APP里会定义更复杂的组件，但是从这个例子中会学到所有需要的基础知识。

准备好了吗？是时候深入探索和构建组件了。在Litho中，编写*Spec*类声明组件的布局。这个框架会生成代码中用来创建组件实例的底层组件类。

这个自定义组件叫做ListItem,显示一个标题底下带着一个小一点的副标题。因此，需要用下面的内容创建一个类，名字叫做ListItemSpec：

		@LayoutSpec
		public class ListItemSpec {
		
		  @OnCreateLayout
		  static Component onCreateLayout(ComponentContext c) {
		
		    return Column.create(c)
		        .paddingDip(ALL, 16)
		        .backgroundColor(Color.WHITE)
		        .child(
		            Text.create(c)
		                .text("Hello world")
		                .textSizeSp(40))
		        .child(
		            Text.create(c)
		                .text("Litho tutorial")
		                .textSizeSp(20))
		        .build();
		  }
		}

这就是出上一步见过的Text组件，在这个例子中，这个Text组件作为Colum的子属性传入，把Column想象成HTML里的<div>标签，Column就是一个封装器，主要用于把组件汇集起来，可能再添加一些背景样式。因为Litho使用[Yoga](https://yogalayout.com/),可以使用Flexbox 属性为Column和Row设置子布局。这里，只是简单设置内边距和背景颜色。

那么组件是如何渲染的呢？在Activity里，简单地修改Component定义为如下代码：

	final Component component = ListItem.create(context).build();

**注意**：使用的是ListItem,不是ListItemSpec。

这个ListItem是怎么来的？create方法和build方法是在哪里定义的？这就是Litho *Specs*的魔法所在。

在新手入门指南里，我们了解到如何为项目添加依赖，让Litho的代码生成起作用。这会遍历代码运行注解处理器，注解处理器会去查找FooSpec这个类名，然后在与Spec所在相同的包内生成Foo这个类。这些类会自动地被填充所有Litho需要的方法。此外，根据这个Spec，注解处理器还会生成一些额外的方法（例如Text的textSizeSp 或者 Column 的backgroundColor ）。

就是这么简单! 运行APP，应该看到如下画面：

![](https://fblitho.com/static/images/barebones2.png)

## 创建类表

 使用[RecyclerCollectionComponent ](https://fblitho.com/javadoc/com/facebook/litho/sections/widget/RecyclerCollectionComponent)和Sections库来创建列表。
RecyclerCollectionComponent用于在Litho中创建滚动单元，它会隐藏掉一些与Android RecylerView以及Adapter概念相关的一些复杂性。

使用Sections API，可以把列表中的条目放进Sections中，编写*GroupSectionSpec*类用于声明每一个Section渲染什么以及使用是什么样的数据。

准备好了么？是时候创建这个Section了。这个自定义的Section将会叫做ListSection,它会渲染ListItem子组件。
用下列内容创建一个叫做ListSectionSpec的类:

		@GroupSectionSpec
		public class ListSectionSpec {
		
		  @OnCreateChildren
		  static Children onCreateChildren(final SectionContext c) {
		    Children.Builder builder = Children.create();
		
		    for (int i = 0; i < 32; i++) {
		      builder.child(
		          SingleComponentSection.create(c)
		              .key(String.valueOf(i))
		              .component(ListItem.create(c).build()));
		    }
		    return builder.build();
		  }
		}
SingleComponentSection是在com.facebook.litho.sections.widget包中定义的一个核心Section，用于渲染单一的组件。ListSectionSpec描述了一个包含有32个子Section的section，每个Section负责渲染一个ListItem.可以用这个Section结合RecyclerCollectionComponent来渲染列表。RecyclerCollectionComponent 接受一个Section作为属性，会渲染一个RecylerView，此RecylerView包含此Section输出的任意UI。
它也会管理来自Section的更新和变化例如数据刷新和尾部数据获取。此处我们不使用数据获取功能，所以在这个教程中关闭掉PTR（pull to refresh）功能.在Activity，把Component的定义改成下面的代码：

	final Component component =
	    RecyclerCollectionComponent.create(context)
	        .disablePTR(true)
	        .section(ListSection.create(new SectionContext(context)).build())
	        .build();

**注意**：这里使用的是ListSection,不是ListSectionSpec。

ListSectionSpec跟上一步中用到的ListItemSpec是类似的。Litho遍历代码运行注解处理器，找到ListSectionSpec这个类，并生成ListSection,与早到ListItemSpec这个类并生成ListItem是一样的，生成的类与Spec类在同一个包下。

运行APP，应该会看到包含32个条目的可滚动的列表：
<img src="https://fblitho.com/static/images/barebones3.png" style="zoom:30%">

## 定义组件的属性

如果一个列表只包含一个单一组件的重复拷贝是没有意义的。这个部分，将会看到 *properties* 或者 *props*。可以在组件上设置这些属性来改变组件的行为或者外观。

在组件上添加props 非常简单。 Props 是组件Spec里方法的参数，带有@Prop 这个注解。

把ListItemSpec修改如下：

	@OnCreateLayout
	static Component onCreateLayout(
	    ComponentContext c,
	    @Prop int color,
	    @Prop String title,
	    @Prop String subtitle) {
	
	  return Column.create(c)
	        .paddingDip(ALL, 16)
	        .backgroundColor(color)
	        .child(
	            Text.create(c)
	                .text(title)
	                .textSizeSp(40))
	        .child(
	            Text.create(c)
	                .text(subtitle)
	                .textSizeSp(20))
	        .build();
	}

这里添加了三个属性:title,subtitle 和color,注意Column的背景色以及Text组件显示的文本不再是硬编码，现在是基于onCreateLayout方法传入的参数。

这个魔法就发生在 @Prop 这个注解和注解处理器上。处理器会以一种聪明的方式在组件构造器上生成对应属性的方法。修改ListSectionSpec来指定构造ListItem需要的属性:

	@OnCreateChildren
	static Children onCreateChildren(final SectionContext c) {
	  Children.Builder builder = Children.create();
	
	  for (int i = 0; i < 32; i++) {
	    builder.child(
	        SingleComponentSection.create(c)
	            .key(String.valueOf(i))
	            .component(ListItem.create(c)
	               .color(i % 2 == 0 ? Color.WHITE : Color.LTGRAY)
	               .title(i + ". Hello, world!")
	               .subtitle("Litho tutorial")
	               .build()));
	  }
	  return builder.build();
	}

现在当ListItem被构造时，color,title,subtitle属性会被传入，每一行带有各自的背景色。

运行APP，应该会看到如下画面：

<img src="https://fblitho.com/static/images/barebones4.png" style="zoom:30%">

在使用@Prop注解时还可以指定更多选项，例如，看下面这个属性：

	@Prop(optional = true, resType = ResType.DIMEN_OFFSET) int shadowRadius,

这会告诉注解处理器生成一系列方法，例如shadowRadiusPx, shadowRadiusDip, shadowRadiusSp 还有 shadowRadiusRes。
## 总结

祝贺你完成这个教程！这个基础教程已经提供了开始使用Litho以及创建组件需要的所有构建模块。在[com.facebook.litho.widgets](https://fblitho.com/javadoc/com/facebook/litho/widget/package-frame) 和 [com.facebook.litho.sections.widget](https://fblitho.com/javadoc/com/facebook/litho/sections/widget/package-frame) 这两个包里找到用到的预定义组件。这里是[完整的教程](https://github.com/facebook/litho/tree/master/sample-barebones)。确保下载[示例代码](https://github.com/facebook/litho/tree/master/sample)做深入了解以及阅读Litho的API文档。

**发现更多?**

在这个教程中，我们主要讨论了使用Sections创建列表。Sections框架通过声明式，可组合的方式使复杂列表变得简单。从[这里](https://fblitho.com/docs/sections-tutorial)开始第二部分可选教程的学习。
