# DiffSection Specs

一个diff section spec 定义一个section，明确输出在此section层级结构上进行插入、更新、删除操作时产生的变化。

Diff section specs 明确管理section的状态以及props发生变化时在section上执行的插入、删除以及更新操作。 作为每个Section树上叶子节点的Diff Sections ，他们才是实际上对一个列表所做的改变作出说明的section。

一个可能需要编写自定义 diff Section的例子就是希望以增量更新或者变化的形式显示接收到的数据，可能你正在使用类似于DiffUtil的工具处理数据。（如果你正在使用[DiffUtil](https://fblitho.com/docs/diff-sections),考虑使用预定义的[DataDiffSection](https://fblitho.com/javadoc/com/facebook/litho/sections/common/DataDiffSection)代替自己的diff模块。）

通常情况下，不需要自己写diff section specs。在 com.facebook.litho.sections.widget 这个包里，提供了两种diff sections 几乎覆盖了所有可能的用例。

让我们以 [SingleComponentSection](https://fblitho.com/javadoc/com/facebook/litho/sections/common/SingleComponentSection) 为例说明如何编写diff section specs.下面是SingleComponentSection的代码片段：


	@DiffSectionSpec
	class SingleComponentSectionSpec {
	
	  @OnDiff
	  static void onCreateChangeSet(
	      SectionContext c,
	      ChangeSet changeSet,
	      @Prop Diff<Component> component,
	      ...) {
	
	    if (component.getNext() == null) {
	      changeSet.delete(0);
	    } else if (component.getPrevious() == null) {
	      changeSet.insert(
	          0,
	          ComponentRenderInfo.create()
	              .component(component.getNext())
	              ...
	              .build());
	    } else {
	      changeSet.update(
	          0,
	          ComponentRenderInfo.create()
	              .component(component.getNext())
	              ...
	              .build());
	    }
	  }
	}

如你所见，diff section specs 使用@DiffSectionSpec注解。编写 diff section spec 很简单，只需要写一个带有@OnDiff 注解的方法。

这个带有 @OnDiff注解的方法必须用SectionContex作为其第一个参数，ChangeSet作为其第二个参数，跟随这两个参数的其他参数，可以接受任意个数的带有@Prop 和@State 注解的参数。

这些props 和 states 有一个特殊的类型：Diff<T>。 如果某一个prop在另外一个带有注解的方法中被定义为比如 @Prop String prop1，那么在这个带有@OnDiff注解的方法中，则须被定义成 @Prop Diff<String> prop1，添加 Diff<T>这个外壳的原因是我们在计算数据变化时可以对prop的新旧值进行比较。

## 使用 ChangeSet 对列表进行更改

@OnDiff 方法的 ChangeSet参数，被 Diff section spec用来指定section如何应对数据变化。这个 @OnDiff 方法 被调用时总是带有当前的props、state值和旧的props、state值(因为Diff<T>的缘故)。也就是说可以使用当前值和旧的值来决定即将被渲染的条目如何更新。

当你已经决定好要做出哪些改变，须调用 ChangeSet上相应的方法。这些方法对应于RecyclerView.Adapter里的notifyItem* 方法。可以快速理解一下[SingleComponentSectionSpec的onDiff](https://github.com/facebook/litho/blob/d766e3b4965edf84eda0090f58d0020aa302d650/litho-sections-core/src/main/java/com/facebook/litho/sections/common/SingleComponentSectionSpec.java#L25)方法是如何工作的：

* 如果没有新Component（当component.getNext() == null时），需要在列表中删除这一行。
* 或者我们有新Component 并且没有旧Component，需要新插入一行。
* 如果新Component 和旧Component 同时存在，则需要用新的Component对这一行进行更新。

注意：在调用ChangeSet方法时使用的索引，这里是相对于当前Section而言的。SingleComponentSectionSpec 中的索引 0 ，在最终列表中的实际索引可能是100，这依赖于Section的层级结构。在处理ChangeSet时，DiffSection框架来负责将局部索引转换成全局索引。