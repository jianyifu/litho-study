# Props

Litho 使用具有不可变输入的单向数据流。 按照React建立的名称，Component 采用的输入称为 *props*。

## 定义和使用 props

给定 Component 的 props 是在spec方法中用 @Prop 注解的所有参数的联合。您可以在所有声明为 @Prop 参数的方法中访问 props 的值。

可以在多个生命周期方法中定义和访问相同的 prop 。注解处理器确保所有 spec 方法中使用一致的 prop 类型和一致的注解参数。

以下面的Component Spec 为例：

	@MountSpec
	class MyComponentSpec {
	
	  @OnPrepare
	  static void onPrepare(
	      ComponentContext c,
	      @Prop(optional = true) String prop1) {
	    ...
	  }
	
	  @OnMount
	  static void onMount(
	      ComponentContext c,
	      SomeDrawable convertDrawable,
	      @Prop(optional = true) String prop1,
	      @Prop int prop2) {
	    if (prop1 != null) {
	      ...
	    }
	  }
	}

MyComponentSpec 定义了两个 props：一个名为 prop1 的 String prop 和一个名为 prop2 的 int prop。 prop1 是可选的，它需要在定义它的所有方法中标记，否则注解处理器将抛出异常。

当生命周期方法被调用时，@Prop 参数将保存在创建组件时（或其默认值）从父组件传递的值。

在 LayoutSpecs 和 MountSpecs 中定义和使用 props 的方式相同。

## 设置 props 的值

对于在 Spec 中定义的每个唯一 prop ，注解处理器会在组件生成器上创建一个构建器模式方法，该方法与 prop 具有相同的名称，并且把仅有的参数值设置该 prop 。

通过在生成的组件生成器上调用适当的方法来传递这些 props 的值：

	MyComponent.create(c)
	    .prop1("My prop 1")
	    .prop2(256)
	    .build();

## props 的默认值
可以不必为可选的 prop 设置值，prop 的值会根据其定义类型设置成 Java 默认值。通常需要为组件的 props 定义显式的默认值，而不是简单地依赖于 Java 的默认值。 

通过 @PropDefault 注解把默认的 prop 值定义为 Spe c类中的静态成员。我们来定义上述 prop 的默认值：

	@MountSpec
	public class MyComponentSpec {
	  @PropDefault static final String prop1 = "mydefaultvalue";
	  @PropDefault static final int prop2 = -1;
	
	  ...
	}

通过设置 resType 和 resId ，Prop 默认值还支持来自 Resources 的默认值。我们来为 @PropDefault 注解的变量定义默认资源值：

	@PropDefault(resType = ResType.DIMEN_SIZE, resId = R.dimen.default_spacing) static float prop3;

## 资源类型

在创建布局时，使用 Android 资源系统的值很常见，例如尺寸，颜色，字符串等。Litho 提供了简便方法，通过注解把 Android 资源中的值设为  prop 的值。

我们来看一个简单的例子：
	@LayoutSpec
	public class MyComponentSpec {
	
	  @OnCreateLayout
	  static Component onCreateLayout(
	      LayoutContext context,
	      @Prop CharSequence someString,
	      @Prop int someSize,
	      @Prop int someColor) {
	    ...
	  }
	}


在上面的示例中，MyComponent 希望使用 整型的颜色值（someColor），像素尺寸（someSize）和字符串（someString）作为 props 的值。通常情况下，您需要使用资源值来设置这些 props 的值：

	Resources res = context.getResources();
	
	MyComponent.create(c)
	    .someString(res.getString(R.string.my_string))
	    .someSize(res.getDimensionPixelSize(R.dimen.my_dimen))
	    .someColor(res.getColor(R.color.my_color))

该框架允许使用资源类型标注 props，以便方便快捷地在组件构建器中直接使用资源值。

	@LayoutSpec
	public class MyComponentSpec {
	
	  @OnCreateLayout
	  static Component onCreateLayout(
	      LayoutContext context,
	      @Prop(resType = ResType.STRING) CharSequence someString,
	      @Prop(resType = ResType.DIMEN_SIZE) int someSize,
	      @Prop(resType = ResType.COLOR) int someColor) {
	    ...
	  }
	}


通过以上更改，MyComponent的构建器将根据其资源类型为 prop 生成带有 Res，Attr ，Dip 以及 Px 的方法。所以可以像下面这样：

	MyComponent.create(c)
	    .someStringRes(R.string.my_string)
	    .someSizePx(10)
	    .someSizeDip(10)
	    .someColorAttr(android.R.attr.textColorTertiary)



其他支持的资源类型为 ResType.STRING_ARRAY ，ResType.INT ，ResType.INT_ARRAY ，ResType.BOOL ，ResType.COLOR ，ResType.DIMEN_OFFSET ，ResType.FLOAT 和 ResType.DRAWABLE 。

## 可变参数


有时候，想要参数支持列表。不幸的是，这可能会有点痛苦，因为它需要开发人员创建一个列表，将所有项目添加到列表里，并将这些项目传递给组件创建。 varArg参数旨在使列表型参数更容易一些。

	@LayoutSpec
	public class MyComponentSpec {
	
	   @OnCreateLayout
	   static Component onCreateLayout(
	      LayoutContext context,
	      @Prop(varArg = "name") List<String> names) {
	      ...
	   }
	}

使用方法如下：

	MyComponent.create(c)
	   .name("One")
	   .name("Two")
	   .name("Three")

从版本0.6.2开始，这也适用于带有resType的 prop。例如，给定一个像这样的组件：

	@LayoutSpec
	public class MyComponent2Spec {
	
	   @OnCreateLayout
	   static Component onCreateLayout(
	      LayoutContext context,
	      @Prop(varArg = "sizes", resType = ResType.DIMEN_TEXT) List<Float> sizes) {
	      ...
	   }
	}

您可以通过调用构建器来添加多个 size：

	MyComponent2.create(c)
	   .sizesPx(1f)
	   .sizesRes(resId)
	   .sizesAttr(attrResId)
	   .sizesDip(1f)
	   .sizesSp(1f)

## 不变性

组件的 prop 是只读的。组件的父组件在创建组件时传递 prop 的值，并且在组件的整个生命周期中 prop 的值都不能更改。如果必须更 prop 的值，父组件必须创建一个新的组件并传递 prop 的新值​​。 prop 对象应该是不可变的。由于[异步布局](https://fblitho.com/docs/asynchronous-layout)，可以在多个线程上访问 props。props 的不变性确保组件层次结构中不会出现线程安全问题。
