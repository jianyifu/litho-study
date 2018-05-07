# GroupSection Specs

一个Group Section Spec 用于把数据组织成Sections层级结构，每一个section 负责一组数据的渲染。

Group Section Specs 用 @GroupSectionSpec进行标注。编写GroupSectionSpec很简单：你只需要写一个方法，用@OnCreateChildren 进行标注，这个方法返回Sections树，以当前Section为根节点。子节点可以是由其他GroupSectionSpec类或者DiffSectionSpec classes创建的Section实例。

我们来看一下如何声明一个简单的包含有头部的字符串列表。
用[SingleComponentSection](https://fblitho.com/docs/sections-building-blocks#singlecomponentsection)作为头部，[DataDiffSection](https://fblitho.com/docs/sections-building-blocks#datadiffsection)作为字符串列表，再将两者结合到一个层次结构中：

	@GroupSectionSpec
	class FooSectionSpec {
	
	  @OnCreateChildren
	  static Children onCreateChildren(
	      SectionContext c,
	      @Prop String headerTitle,
	      @Prop List<String> data) {
	    return Children.create()
	        .child(
	            SingleComponentSection.create(c)
	                .component(
	                    Text.create(c)
	                        .text(headerTitle)
	                        .build()))
	        .child(
	            DataDiffSection.<String>create(c)
	                .data(data)
	                .renderEventHandler(FooSection.onRender(c)))
	        .build();
	  }
	
	  @OnEvent(RenderEvent.class)
	  static ComponentRenderInfo onRender(
	      SectionContext c,
	      @FromEvent String item) {
	    return ComponentRenderInfo.create(c)
	        .component(
	            Text.create(c)
	                .text(item)
	                .build())
	        .build();
	    }
	}


假设一个页面由多个带有头部的字符串列表这样的子Section组成。比如根据首字母分组的联系人列表，这个效果可以轻松实现，只需要创建一个
GroupSectionSpec，其中每个首字母用一个FooSection作为子Section。

	@GroupSectionSpec
	class BarSectionSpec {
	
	  @OnCreateChildren
	  static Children onCreateChildren(
	      SectionContext c,
	      @Prop List<String> data) {
	
	      final Children.Builder builder = Children.create();
	
	      for(char firstLetter = 'A'; firstLetter <= 'Z'; firstLetter++) {
	          builder.child(FooSection.create(c)
	              .key(String.valueOf(firstLetter))
	              .headerTitle(String.valueOf(firstLetter))
	              .data(getItemsForLetter(firstLetter, data)));
	      }
	
	      return build.build();
	  }
	}

下面这张图代表了一棵Sections树，树的根节点是BarSection。BarSection的每个子节点是一个Section，叶子节点是一些可以渲染的屏幕上的Components。树中的每一个Section都充当了一个数据源，section的责任就是描述需要什么样的数据以及要如何渲染数据。
![section_tree](https://fblitho.com/static/images/group-section-spec.png)