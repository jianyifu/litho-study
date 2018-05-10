# Layout

Litho 使用 Flexbox 实现的 Yoga 对屏幕上的组件进行测量和布局。如果在 Web 上使用过 Flexbox ，应该非常熟悉。如果更熟悉 Android 通常执行 Layout 的方式，那么 Flexbox 会让你想起 LinearLayout。

在 Litho 中，您可以使用 Row 来实现与水平 LinearLayout 类似的布局。

	Row.create(c)
	    .child(...)
	    .child(...)
	    .build();

或者 Column 实现类似于 垂直 LinearLayout 的布局。

	Column.create(c)
	    .child(...)
	    .child(...)
	    .build();

为了达到类似 LinearLayout 中 weight 的效果，Flexbox 提供了一个名为 flexGrow（<weight>）的概念。

	Row.create(c)
	    .child(
	        SolidColor.create(c)
	            .color(RED)
	            .flexGrow(1))
	    .child(
	        SolidColor.create(c)
	            .color(BLUE)
	            .flexGrow(1))
	    .build();

如果您想将一个视图叠加在另一个视图之上 - 类似于 FrameLayout  -   Flexbox 可以通过 positionType（ABSOLUTE）来实现。

Column.create(c)
    .child(
        Image.create(c)
            .drawableRes(R.drawable.some_big_image)
            .widthDip(100)
            .heightDip(100))
    .child(
        Text.create(c)
            .text("Overlaid text")
            .positionType(ABSOLUTE))
    .build();


有关特定 Flexbox 属性的更多文档，请查阅 [Yoga文档](https://yogalayout.com/docs/) 或查找有关 Flexbox 如何工作的任何网络资源。
