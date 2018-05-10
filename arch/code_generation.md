# 代码生成

正如[编写组件](https://fblitho.com/docs/writing-components)中所述，Litho 依靠代码生成来创建组件 Spec 中的组件。此过程利用叫做 [SpecModels](https://fblitho.com/javadoc/com/facebook/litho/specmodels/model/SpecModel) 的中间 ComponentSpec 模型表示，它们是不可变的Java对象。

代码生成包括三个主要步骤：

* 从 Component Spec 创建 Spec Model 。
* Spec Model 验证。
* 从给定的 Spec Model 生成 Component 。

### Spec Model 创建

Spec Model 是注解处理器在编译时创建的，注解处理器 是 javac 中用于扫描和处理注释的工具。 Litho 注解处理器将处理组件 Spec 上的注解，方法和字段，并为每个组件创建一个 Spec Model。

未来，我们将增加以其他方式创建 Spec Model 的功能。例如，我们希望能够直接在Android Studio / Intellij 中创建 Spec Model，这将允许我们无需构建源代码即可生成组件。

### Spec Model 验证
Spec Models 有一个名为 validate（）的方法，它返回一个[SpecModelValidationErrors](https://fblitho.com/javadoc/com/facebook/litho/specmodels/model/SpecModelValidationError) 列表。如果这个列表是空的，那么Spec是格式良好的，可以用来生成一个有效的组件。如果不是，那么它将包含在生成有效的Component之前需要修复的错误列表。

### 组件生成
如果 Spec Model 的验证步骤成功，则可以调用生成方法。这将创建一个[Javapoet](https://github.com/square/javapoet) TypeSpec，然后可以很容易地使用 Javapoet 来创建一个 Component 类文件。

### 为您的项目设置代码生成
如果您使用[入门指南](https://fblitho.com/docs/getting-started)部分中的说明设置项目，则代码生成将自动在项目上执行。

