# Mount Specs

Mount Specs 定义了一个可以渲染 View 或 drawable 的组件。

只有需要用Litho 集成 View和 Drawable 时，才需要创建 Mount Specs 。这里的 Mount 是指由布局树中的所有组件提取渲染状态（ View 或 Drawable ）进行显示所执行的操作。

Mount spec 类应使用 @MountSpec 进行标注并至少实现 @OnCreateMountContent 方法。下面列出的其他方法是可选的。

Mount Specs 组件的生命周期如下所示：

* 在布局计算之前运行 @OnPrepare 一次。
* 在布局计算期间选择性运行 @OnMeasure 。
* 在布局计算之后运行 @OnBoundsDefined 一次。
* 在组件挂接到宿主 View 之前运行 @OnCreateMountContent 。
* 在组件挂接到宿主 View 之前运行 @OnMount 。
* 在组件挂接到宿主 View 后运行 @OnBind 。
* 在将组件从宿主 View 分离之前运行 @OnUnbind 。
* 在组件从宿主 View 分离后，选择性运行 @OnUnmount 。

## Mounting

我们从一个简单的 ColorComponent 开始，该 ColorComponent  接收 color name 作为属性，将 对应的 ColorDrawable 挂载上去。

	@MountSpec
	public class ColorComponentSpec {
	
	  @OnCreateMountContent
	  static ColorDrawable onCreateMountContent(ComponentContext c) {
	    return new ColorDrawable();
	  }
	
	  @OnMount
	  static void onMount(
	      ComponentContext context,
	      ColorDrawable colorDrawable,
	      @Prop String colorName) {
	    colorDrawable.setColor(Color.parseColor(colorName));
	  }
	}

* 挂载操作有一个非常类似于Android的RecyclerView Adapter的API。它有一个 onCreateMountContent 方法，用于在回收池为空时创建和初始化 View 和 Drawable 内容 onMount 方法用于使用当前信息对复用的内容进行更新。
* onCreateMountContent 的返回类型应该始终与 onMount 的第二个参数的类型匹配。它们必须是 View 或 Drawable 子类。参数将由注解处理器在构建时进行校验。
* 挂载总是在主线程中进行，因为它可能必须处理 Android View（Android View 是绑定到主线程的）。
* onCreateMountContent 不能接收 @Prop 或任何带有其他注解的参数。
* 假定 @OnMount 方法总是在UI线程中运行，那么此处不应该执行昂贵的操作。

## 级间输入与输出

您可以通过在@OnPrepare方法中执行这些操作来避免在UI线程中的执行繁重操作，该方法在执行布局计算之前只运行一次，并且可以在后台线程中执行。

假设我们要在上面的 ColorComponent 中，把颜色名称解析工作放到 UI 线程外。为了做到这一点，我们需要一种方法将 @OnPrepare 方法中生成的值传递给 @OnMount 的实现。 Litho提供了级间的输入和输出，让你可以做到这一点。

我们来看看 ColorComponent 的@OnPrepare方法。

	@MountSpec
	public class ColorComponentSpec {
	
	  @OnPrepare
	  static void onPrepare(
	      Context context,
	      @Prop String colorName,
	      Output<Integer> color) {
	    color.set(Color.parseColor(colorName));
	  }
	
	  @OnCreateMountContent
	  static ColorDrawable onCreateMountContent(ComponentContext c) {
	    return new ColorDrawable();
	  }
	
	  @OnMount
	  static void onMount(
	      ComponentContext context,
	      ColorDrawable colorDrawable,
	      @FromPrepare int color) {
	    colorDrawable.setColor(color);
	  }
	}


在任何 @MountSpec 方法中使用Output <？>会自动为之后的阶段创建一个输入。在这种情况下，@OnPrepare 输出为 @OnMount 的输入。

注解处理器将确保在构建时考虑到这些级间不变量的顺序传递，例如，您无法将 @OnMeasure 的输出作为 @OnPrepare 的输入，因为 @OnPrepare 总是先于 @OnMeasure 运行。

## 测量 

无论何时如果要在布局计算过程中自定义组件的测量，就要实现 @OnMeasure 方法。

现在，让我们假设想要 ColorComponent 具有默认宽度，并在其高度未定义时强制执行特定的高宽比。

	@OnMeasure
	static void onMeasure(
	    ComponentContext context,
	    ComponentLayout layout,
	    int widthSpec,
	    int heightSpec,
	    Size size) {
	
	  // If width is undefined, set default size.
	  if (SizeSpec.getMode(widthSpec) == SizeSpec.UNSPECIFIED) {
	    size.width = 40;
	  } else {
	    size.width = SizeSpec.getSize(widthSpec);
	  }
	
	  // If height is undefined, use 1.5 aspect ratio.
	  if (SizeSpec.getMode(heightSpec) == SizeSpec.UNSPECIFIED) {
	    size.height = width * 1.5;
	  } else {
	    size.height = SizeSpec.getSize(heightSpec);
	  }
	}

您可以在@OnMeasure中使用 @Prop 注解访问组件的属性。 SizeSpec 的API 类似于 Android的 MeasureSpec。

就像@OnPrepare一样，@OnMeasure方法也可以生成级间输出（可通过@FromMeasure参数注释访问），并且可以在后台线程中执行。
## ShouldUpdate

MountSpec 可以定义一个用 @ShouldUpdate 标注的方法，以避免在更新时重新计算和重新挂载。
 @ShouldUpdate 的调用取决于 Component 是否是只有渲染功能。如果渲染结果仅取决于其属性和状态，则 Component 只有渲染功能。这意味着 Component 在@OnMount 期间不应该访问任何可变全局变量。
通过使用 @MountSpec 注解上的 pureRender 参数，可以将 @MountSpec 定义只具备渲染功能。只具备渲染功能的组件才能够在属性不发生变化的时候不重新挂载。@ShouldUpdate 函数可以定义如下：

	@ShouldUpdate(onMount = true)
	public boolean shouldUpdate(Diff<String> someStringProp) {
	  return !someStringProp.getPrevious().equals(someStringProp.getNext());
	}

从 shouldUpdate 接收到的参数是属性或状态的[差分 diff](https://fblitho.com/javadoc/com/facebook/litho/Diff)。 Diff 是包含属性或者状态在新旧组件层次结构中值的对象。在这个例子中，这个组件将 someStringProp 定义为一个 String @Prop。 shouldUpdate 将收到一个 Diff <String>，以便能够比较此 @Prop 的旧值和新值。
shouldUpdate 必须考虑 @OnMount 时使用的任何属性和状态。它可以安全地忽略仅在 @ OnMount 或者 @OnUnbind 时使用的属性和状态，因为这两个方法无论如何都会被执行。

@ShouldUpdate 注解中的 onMount 属性控制着这个 shouldUpdate 是否可以在挂载时发生。默认情况下，Litho 会在布局时尝试执行此调整，但是如果关闭布局差分，则可以将 onMount 设置为true，以便在 mount时 执行此检查。默认情况下，onMount 属性设置为 false ，因为相等性检查操作本身可能很沉重，并使挂载性能变差。

带有 @ShouldUpdate 注解的方法目前仅在 @MountSpec 中支持。我们计划在将来扩展对复杂布局的支持，但目前@LayoutSpec中的  @ShouldUpdate 注释方法将不起作用。

## 预分配

当挂载 MountSpec 组件时，其 View 或 Drawable 内容需要从回收池中初始化或重用。如果池为空，那么将创建一个新实例，这可能会使UI线程过于繁忙并丢弃一个或多个帧。为了缓解这种情况，Litho 可以预先分配一些实例并放入回收池中。

	@MountSpec(poolSize = 3, canPreallocate = true)
	public class ColorComponentSpec {
	  ...
	}

canPreallocate 可为此 MountSpec 启用预分配，使用 poolSize 定义需要预分配的实例数量。对于此 ColorComponent 示例，将创建三个ColorDrawable 实例并放入回收池。对于夸大复杂视图的 MountSpec 组件，建议使用此选项。