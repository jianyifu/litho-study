# Layout Specs

Layout Spec 与 Android 上的复合View 逻辑等价。它只需现有组件组合到一个不可变的布局树中。

实现 Layout Spec 非常简单：只需编写一个带有 @OnCreateLayout 注解的方法，该方法返回包含 ComponentLayout 对象的不可变树。

我们从一个简单的例子开始：

	@LayoutSpec
	public class MyComponentSpec {
	  @OnCreateLayout
	  static Component onCreateLayout(
	      ComponentContext c,
	      @Prop int color,
	      @Prop String title) {
	
	      return Row.create(c)
	          .alignItems(CENTER)
	          .child(
	              SolidColor.create(c)
	                  .colorRes(color)
	                  .widthDip(40)
	                  .heightDip(40))
	          .child(
	              Text.create(c)
	                  .text(title)
	                  .textSizeRes(R.dimen.my_text_size)
	                  .flexGrow(1f))
	          .build();
	  }
	}


如您所见，Layout Spec 类使用 @LayoutSpec 注释。

使用 @OnCreateLayout 注解的方法必须具有 [ComponentContext](https://fblitho.com/javadoc/com/facebook/litho/ComponentContext) 作为其第一个参数，后跟使用 @Prop 标注的参数列表。注解处理器将在构建时对参数列表以及API中其他约束条件进行验证。

在上面的示例中，布局树具有一个根容器，带有两个孩子水平叠放（ Row.create ）并垂直居中（ Align.CENTER ）。

第一个孩子是一个 [SolidColor](https://fblitho.com/javadoc/com/facebook/litho/widget/SolidColor) 组件，接收 colorRes 作为Prop ，并具有40dp的宽度和高度。

	SolidColor.create(c)
	    .colorRes(color)
	    .width(40)
	    .height(40)

第二个孩子是一个 [Text](https://fblitho.com/javadoc/com/facebook/litho/widget/Text) 组件，接收名为 text 的属性，使用 grow(1f) 填充 MyComponent 中剩余可用的水平空间（等同于 Android 中LinearLayout 中的 layoutWeight）。文本大小在 my_text_size dimension 资源中定义。

	Text.create(c)
	    .text(title)
	    .textSizeRes(R.dimen.my_text_size)
	    .grow(1f)

您可以查看完整的 [Yoga](https://facebook.github.io/yoga/docs/learn-more/) 文档以查看该框架公布的所有布局功能。