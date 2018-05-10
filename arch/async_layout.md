# 异步布局

## 不变性和线程安全性
线程安全的大多数问题都来自于可变对象的并发读写。 下面是 Java 中并发问题的经典示例：
	
public class SomeExampleClass {
	    private int mCounter;
	
	    public String getThisOrThat() {
	      if (mCounter > 10) {
	        return "this":
	      } else {
	        mCounter++;
	        return "that";
	      }
	    }
	  }


如果多个线程在 SomeExampleClass 的共享实例上调用 getThisOrThat ，这将是竞争条件下的最经典示例。当进入方法的第二个线程尝试读取 mCounter 时，第一个线程可能正在执行 mCounter ++ ，我们无法确定第二个线程实际 从mCounter 读取的值。一般的问题是，在这段代码中有一个可变状态（mCounter），多个线程试图对这个可变状态进行读写操作。编写在多线程上分发工作的应用程序时，竞争条件是最常见的问题。

这就是为什么传统上，在多线程上运行 UI 代码一直非常复杂的原因。 Android 视图是有状态和可变的。例如，TextView 必须跟踪它正在显示的文本，同时暴露一个允许开发人员对文本进行改变的 setText（）方法。这意味着如果 Android UI 框架决定把布局计算之类的操作放在在辅助线程中执行，则必须解决用户从另一个线程调用 setText（）并同时在布局计算发生时改变当前文本的问题。

让我们回过头来看看我们的示例代码。我们说主要问题是getThisOrThat（）方法中存在可变状态 mCounter 。有没有办法编写功能等价的代码，而不必依赖这种可变状态？让我们试着想象一下，一个对象的内容在创建之后不能做任何更改。如果什么都不能改变，在读取对象状态时就不存在线程之间的竞争。我们可以重写示例代码，如下所示：

	 public static class Result {
	    public final int mCounter;
	    public final String mValue;
	
	    public Result(int counter, int value) {
	      mCounter = counter;
	      mValue = value;
	    }
	  }
	
	  public class SomeExampleClass {
	    public static Result getThisOrThat(int counterValue) {
	      if (counterValue > 10) {
	        return new Result(counterValue, "this"):
	      } else {
	        return new Result(counterValue + 1, "that");
	      }
	    }
	  }


我们的方法现在完全是线程安全的，因为它从不修改 SomeExampleClass 的任何内部状态。在这个例子中，getThisOrThat（）就是所谓的“纯函数”，因为它的结果只取决于输入，并且没有任何副作用。

在 Litho 中，我们尝试将相同的概念应用于布局计算。组件是一个不可变的对象，它以 @Prop 和 @State 值的形式包含布局函数的所有输入。这也解释了为什么我们需要@Prop和@State是不可变的。如果它们不是，我们将失去布局作为“纯粹功能”的特性。

Java 中的不变性通常需要花费更多的对象分配。即使在我们简单的例子中，现在为每次函数的调用分配一个 Result 对象。 Litho 使用缓冲池和[代码生成](https://fblitho.com/docs/codegen)功能自动将对象分配最小化。

## 同步和异步操作

Litho 为布局计算同时提供同步和异步的 API 。这两个 API 都是线程安全的，可以从任何线程调用。最终布局将始终代表最后调用 setRoot（）或 setRootAsync（）时设置的 Component。

同步的布局计算可确保在 ComponentTree上 调用setRoot（）后立即可以将布局计算的结果挂载到 [LithoView](https://fblitho.com/javadoc/com/facebook/litho/LithoView) 上。

这样做的主要缺点是计算发生在调用线程上，因此从主线程调用setRoot（）是不可取的。另一方面，有些情况下在屏幕上显示某些内容之前，不能等待后台线程计算布局，例如，当要显示的项目已经存在于窗口中时。在这些情况下，同步调用setRoot（）是最佳选择。同步操作也使得将 Litho 与预先存在的线程模型集成起来非常简单。如果您的应用程序已经具有复杂和结构化的线程设计，那么您可能需要将布局计算纳入其中，而无需依赖 Litho 的内置线程。

异步布局计算将使用 Litho 的布局线程来计算组件布局。这意味着计算任务会立即进入另一个单独线程上的队列，并且布局结果在调用线程上不会立即可见。异步布局操作 广泛应用于 [RecyclerBinder](https://fblitho.com/docs/recycler-component) 中的例子。