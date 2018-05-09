# 使用Component


生成的组件类为您在组件 Spec 中定义的 Prop 提供了一个简单的构建器。为了在UI中使用生成的组件，您需要一个LithoView，它是一个能够渲染组件的 Android ViewGroup 。

您可以按如下方式指定要由LithoView渲染的组件：


	final Component component = MyComponent.create(componentContext)
	    .title("My title")
	    .imageUri(Uri.parse("http://example.com/myimage")
	    .build();
	LithoView view = LithoView.create(context, component);


在这个例子中，MyComponent将通过宿主LithoView进行布局，您像用 Android View 一样使用 LithoView 。有关如何在 Activity 中使用 LithoView 的例子，请参阅[教程](https://fblitho.com/docs/tutorial)。

> 重要提示：本示例中的LithoView如果直接用于 View 层次结构中，将在主线程上同步执行布局计算。有关从主线程执行布局的更多信息，请参阅[异步布局](https://fblitho.com/docs/asynchronous-layout)。