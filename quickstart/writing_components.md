# 编写Component

## Component Specs
组件 spec 会生成在UI中使用的实际组件类。有两种类型的组件 spec：

* Layout Spec：将其他组件组合到特定的布局中。这相当于 Android 上的 ViewGroup 。
* Mount Spec：可以渲染 View 或 Drawable 组件。


现在，让我们来看看 Layout Spec 的整体结构：


	@LayoutSpec
	class MyComponentSpec {
	
	  @OnCreateLayout
	  static Component onCreateLayout(
	      ComponentContext c,
	      @Prop String title,
	      @Prop Uri imageUri) {
	    ...
	  }
	}


有几点要注意：

* 组件 Spec 只是一个普通的java类，带有一些特殊的注解。
* 组件 Spec 是完全无状态的，没有任何类成员。
* 使用 @Prop 标注的参数将自动成为组件构建器的一部分。
* 对于从组件 Spec 创建的组件，需要将 Litho 注解处理器添加到 BUCK 或 Gradle 文件中。请参阅[入门指南](https://fblitho.com/docs/getting-started)了解如何操作。您可以通过将生成的类设成包级私有，在类的注解上加上  isPublic = false 。


## Spec, Lifecycle, and Component 类


组件 Spec 类将被处理以生成与 Spec 名相同但没有 Spec 后缀的ComponentLifecycle 子类。例如，MyComponentSpec 类会生成一个 MyComponent 类。

真正在产品中使用的是这个生成的 ComponentLifecycle 子类。 spec 类将在运行时用作这个生成代码的委托。

生成的类种暴露的唯一 API 是 create（...）方法，它为您在 spec 类中声明的 @Props 返回相应的Component.Builder。

在运行时，特定类型的所有组件实例共享相同的 ComponentLifecycle 引用。这意味着每个组件类型只有一个spec实例，而不是每个组件实例。
