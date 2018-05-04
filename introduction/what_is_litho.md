# Litho是什么

Litho是一个用于在Android上构建高效用户界面（UI）的声明性框架。它允许通过基于Java注解的简单功能API来编写高度优化的Android视图，主要用于实现基于RecyclerView的复杂可滚动UI。

在Litho中，使用组件（Component）来构建UI，而不是直接与传统的Android视图进行交互。组件本质上是一个函数，它接受不可变的输入（称为属性 props），并返回描述用户界面的组件层次结构。  

    @LayoutSpec
    class HelloComponentSpec {
    
      @OnCreateLayout
      static Component onCreateLayout(
      		ComponentContext c,
      		@Prop String name) {
    
	    return Text.create(c)
	    .text("Hello, " + name)
	    .textSizeRes(R.dimen.my_text_size)
	    .textColor(Color.BLACK)
	    .paddingDip(ALL, 10)
	    .build();
      }
    }
    

只需声明想要显示的内容，Litho就可以通过在[后台线程中计算布局](https://fblitho.com/docs/asynchronous-layout)，自动[展平视图层次结构](https://fblitho.com/docs/view-flattening)以及[增量渲染](https://fblitho.com/docs/inc-mount-architecture)复杂组件这种最有效的方式来以进行UI的渲染。  


[观看F8讲座](https://developers.facebook.com/videos/f8-2017/litho-a-declarative-framework-for-efficient-uis/)

**继续探索**

查看我们的教程，了解在应用中使用Litho的分步指南。还可以阅读快速入门指南，以了解如何编写和使用自己的Litho组件。
