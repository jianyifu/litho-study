# State

Litho 组件可以包含两种类型的数据：

* props：从父组件传下来，在组件的生命周期中不能改变。
* state：封装由组件管理的实现细节，并且对父组件透明。

需要状态的常见示例是渲染一个复选框。该组件在选中和未选中状态时显示不同的 Drawable ，但这是父组件不需要知道的复选框组件的内部细节。

## 声明组件状态
您可以通过在 Spec 生命周期方法中使用 @State 注解来定义组件的状态，这与定义Prop类似。

在Layout Specs 和 Mount Specs 的生命周期方法上启用状态定义。

	@LayoutSpec
	public class CheckboxSpec {
	
	  @OnCreateLayout
	  static Component onCreateLayout(
	      ComponentContext c,
	      @State boolean isChecked) {
	
	    return Column.create(c)
	        .child(Image.create(c)
	            .drawableRes(isChecked
	                ? R.drawable.is_checked
	                : R.drawable.is_unchecked))
	        .child(Text.create(c)
	            .text("Submit")
	            .clickHandler(Checkbox.onClickedText(c))
	        .build;
	  }
	
	  @OnEvent(ClickEvent.class)
	  static void onClickedText(
	      ComponentContext c,
	      @State boolean isChecked) {
	    ...
	  }
	}

## 初始化状态值

要为状态设置初始值，必须在 Spec 中编写一个用 OnCreateInitialState 注解的方法。

编写@OnCreateInitialState方法时需要以下内容：

* 第一个参数必须是 ComponentContext 类型。
* 可以用 @Prop 参数。
* 所有其他参数必须在其他生命周期方法中有一个其对应的参数，这个参数带有 @State 注解（译者注：想要在此方法中对状态进行初始化，此方法中的参数一定是在其他生命周期方法中定义过的，而且是对应的）。并且它们的类型必须是 [StateValue](https://fblitho.com/javadoc/com/facebook/litho/StateValue) ，它对应于 @State 元素参数化过的类型。
* @OnCreateInitialState方法不是强制性的。如果没有定义状态，或者只是初始化某些状态，那么未初始化的状态将采用 Java 默认值。
* 首次添加到 ComponentTree 时，组件只调用一次 @OnCreateInitialState 。如果组件的 Key 没有更改，则针对同一个 ComponentTree 进行的后续布局重新计算将不会再次调用此方法。
* 永远不需要自己调用 @OnCreateInitialState 。
以下是如何通过父组件传递的值来初始化复选框状态：

		@LayoutSpec
		public class CheckboxSpec {
		
		  @OnCreateInitialState
		  static void createInitialState(
		      ComponentContext c,
		      StateValue<Boolean> isChecked,
		      @Prop boolean initChecked) {
		
		    isChecked.set(initChecked);
		  }
		}


## 定义状态更新

在带有 @OnUpdateState 注解的方法中定义如何更新组件的状态。

根据想要更新的状态或状态所依赖的参数，可以拥有尽可能多的 @OnUpdateState 方法。

每次调用 @OnUpdateState 方法都会触发 ComponentTree 重新计算布局。为了获得更好的性能，如果某些情况需要更新多个状态，则只定义一个 @OnUpdateState 方法来更新所有这些状态的值。将它们捆绑在同一次调用中可减少重新计算布局的次数并提高性能。

以下是编写@OnUpdateState方法时需要知道的内容：

* 表示状态的参数必须与用@State注解的参数的名称相匹配，并且它们的类型必须是参数化的 StateValue 类型。
* 可以使用 @Param 参数。如果状态值取决于 prop 的值，可以像这样声明它们，并在触发更新调用时传递 prop 的值。
* 所有其他参数必须在其他生命周期方法中有一个其对应的参数，这个参数带有 @State 注解。

下面讲解如何为复选框定义状态更新方法：

	@LayoutSpec
	public class CheckboxSpec {
	
	  @OnUpdateState
	  static void updateCheckboxState(StateValue<Boolean> isChecked) {
	    isChecked.set(!isChecked.get());
	  }
	}


如果您想要在单个方法中捆绑多个状态更新，则只需将所有这些状态作为参数添加到同一个 @OnUpdateState 方法即可：

	@OnUpdateState
	static void updateMultipleStates(
	    StateValue<Boolean> stateOne,
	    StateValue<String> stateTwo,
	    @Param int someParam) {
	
	  final boolean thresholdReached = someParam > 100;
	  stateOne.set(thresholdReached);
	  stateTwo.set(thresholdReached ? "reached" : "not reached");
	}

## 调用状态更新
对于 Spec 中的每个 @OnUpdateState 方法，生成的组件都会有两个方法委托给 @OnUpdateState 方法：

* 一个具有相同名称的静态方法，它将同步应用状态更新。
* 一个具有相同名称的静态方法和一个  Async 后缀，它将异步触发状态更新。两种方法都将 ComponentContext 作为第一个参数，然后是 @OnUpdateState 方法中用 @Param 声明的所有参数。

下面介绍如何在用户点击时调用状态更新方法来更新复选框：

	@LayoutSpec
	public class CheckboxSpec {
	
	  @OnCreateLayout
	  static Component onCreateLayout(
	      ComponentContext c,
	      @State boolean isChecked) {
	
	    return Column.create(c)
	        .child(Image.create(c)
	        .drawableRes(isChecked
	            ? R.drawable.is_checked
	            : R.drawable.is_unchecked))
	        .clickHandler(Checkbox.onCheckboxClicked(c)))
	    .build;
	  }
	
	  @OnUpdateState
	  static void updateCheckbox(StateValue<Boolean> isChecked) {
	    isChecked.set(!isChecked.get());
	  }
	
	  @OnEvent(ClickEvent.class)
	  static void onCheckboxClicked(ComponentContext c) {
	    Checkbox.updateCheckboxAsync(c);
	    // Checkbox.updateCheckbox(c); for a sync update
	  }
	}


这是调用状态更新方法时需要牢记的内容：

* 在调用状态更新方法时，作为第一个参数传递的 ComponentContext 实例，必须始终在触发状态更新的生命周期方法中作为参数传递。此上下文包含有关当前已知状态值的重要信息，在重新计算布局期间将这些值从旧组件转移到新组件是也很重要的。
* 在 LayoutSpecs 中，应该避免在 onCreateLayout 中调用状态更新方法，除非可以完全确定 onCreateLayout 只会发生确定的少量次数。每次调用状态更新方法都会在 ComponentTree 上触发一个新的布局计算，然后它将在其所有组件上调用 onCreateLayout，因此非常容易进入无限循环。你应该考虑是否延迟状态更新（如下所述）适合你的用例。
* 在 MountSpecs 中，绝不应该在 bind 和 mount 方法之间调用更新状态方法。如果需要在这些方法中更新状态值，则应该使用下面描述的延迟状态更新。
* 状态是组件的局部概念。不能从组件外调用状态更新方法。[Props](https://fblitho.com/docs/props) 是根据外部变化更新组件的机制。想要了解更多，请参阅[这里](https://fblitho.com/docs/best-practices#props-vs-state)。


## Keys 和组件的标识

该框架根据组件的类型和父组件的 Key 给每个组件设置一个 Key 。这个 Key 用于确定在调用状态更新时我们想要更新哪个组件，并在遍历树时查找此组件。

具有同一父组件的相同类型的组件将被分配相同的密钥，所以我们需要一种唯一标识它们的方式。

此外，当组件的状态或 Props 被更新并且 ComponentTree 被重新创建时，有些情况组件会被删除，添加或被重新安排在树中。因为组件是动态的，所以我们需要一种跟踪组件的方式，所以即使在 ComponentTree 发生变化之后，我们也知道哪些组件可以应用状态更新。

每当在 ComponentTree 中检测到 Key 冲突时（在父组件创建多个相同类型的子组件时发生），我们会为这些同级分配一个唯一键，这取决于它们添加到父组件的顺序。然而，就目前的实现而言，当层次结构中的组件顺序发生更改时，我们没有简单的方法来检测组件是否相同。这意味着自动生成的键通过组件移动不稳定。如果您希望组件四处移动，则必须手动分配 Keys 。

Component.Builder类暴露了一个 .key（）方法，可以在创建组件时调用该方法，以向其分配唯一的 Key，该 Key 将用于识别此组件。

只要在一个组件内有多个相同类型子组件或者希望布局的内容是动态的，就应该设置此项。使用 key prop 在组件上设置的手动键将始终优先于自动生成的键。

当必须手动定义组件的 Key 时，最常见的情况是在循环中创建并添加它们作为子组件：

	@OnCreateLayout
	static Component onCreateLayout(
	    ComponentContext c,
	    @State boolean isChecked) {
	
	  final Component.Builder parent = Column.create(c);
	  for (int i = 0; i < 10; i++) {
	    parent.child(Text.create(c).key("key" +i));
	  }
	  return parent.build();
	}


## 延迟状态更新
对于要更新状态值但不需要立即触发重新计算布局的情况，可以使用**延迟状态更新**。在调用延迟状态更新之后，组件将为该状态保持相同的值，直到下一个布局计算由其他事件触发（接收新的 prop 或常规状态更新）时值将被更新。这对于不需要即时布局计算时，两次 ComponentTree 布局计算之间，更新和保留组件内部信息是有用的。

要使用延迟状态更新，需要把 @State 注解上的 canUpdateLazily 参数设置为true。

对于状态参数 foo ，把它标记为 canUpdateLazily ，框架将生成一个名为lazyUpdateFoo的静态更新方法，该方法将新值作为参数，将其设置为foo的新值。

标记为 canUpdateLazily 的状态仍然可以用于常规状态更新。

我们来看一个例子：

	@OnCreateLayout
	static Component onCreateLayout(
	    final ComponentContext c,
	    @State(canUpdateLazily = true) String foo) {
	
	  FooComponent.lazyUpdateFoo(context, "updated foo");
	  return Column.create(c)
	      .child(
	          Text.create(c)
	              .text(foo))
	      .build();
	}
	
	@OnCreateInitialState
	static void onCreateInitialState(
	    ComponentContext c,
	    StateValue<String> foo) {
	  foo.set("first foo");
	}


FooComponent第一次被渲染时，它的子 Text 组件将显示第一个 foo ，即使 foo 被其他值延迟更新也是如此。当常规状态更新或接收新的 Prop 触发新的布局计算时，将应用延迟更新，并且 Text 将渲染更新后的 foo 。

## 不变性

由于[异步布局](https://fblitho.com/docs/asynchronous-layout)，状态可以在任何时候被多个线程访问。为了确保线程安全，State 对象应该是不可变的（如果出于某种不可能出现的罕见原因，那么至少线程安全）。最简单的解决方案是用 primitives 表达你的状态，因为 primitives 在定义上是不可变的。