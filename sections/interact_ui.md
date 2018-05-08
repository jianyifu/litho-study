# 与UI交互

## 介绍

数据最终由组件显示在屏幕上之前会在整个 Section 层级结构中流动。 Sections 提供一系列功能，让你可以对数据变化做出响应以及同UI进行交互。

## @OnRefresh

当由 Section 层级结构渲染的 UI 需要对内容进行刷新时，带有 @OnRefresh 注解的方法就会被调用。

### <u>API</u>

调用 SectionTree#refresh() 方法将刷新请求传递给层级结构中的所有 Sections 。在 Section中可以这样处理：

	class SectionSpec {
	
	  @OnRefresh
	  static void onRefresh(SectionContext c, @Prop yourProp, @State yourState) {
	    // Handle your refresh request
	  }
	}


## @OnViewportChanged

 当可见窗口中发生变化时会调用带有 @OnViewportChanged 的方法。
### <u>API</u>

调用 SectionTree#viewPortChanged() 和 SectionTree#viewPortChangedFromScrolling() 方法可以让 Sections 知道当前窗口发生了变化。

	class SectionSpec {
	
	  @OnViewportChanged
	  static void onViewportChanged(
	    SectionContext c,
	    int firstVisiblePosition,
	    int lastVisiblePosition,
	    int totalCount,
	    int firstFullyVisibleIndex,
	    int lastFullyVisibleIndex,
	    @Prop YourProp prop,
	    @State YourState state) {
	  }
	}

1) firstVisiblePosition 窗口中第一个可见组件的位置，组件在窗口中部分被隐藏。

2) lastVisiblePosition 窗口中最后一个可见组件的位置，组件在窗口中部分被隐藏。

3) totalCount Section 层级结构中所有条目的总数，这个 Section 是指包含注解方法的那个根节点 Section 。

4) firstFullyVisibleIndex 窗口中第一个完全可组件的位置。

5) lastFullyVisibleIndex 窗口中最后一个完全可见组件的位置。

### <u>窗口中的变化</u>
 窗口中给的变化可以由下列的这些原因引发的：

1) 添加新的组件到窗口中

2) 从窗口中删除组件

3）滚动列表

4) 窗口中的组件被更新

5) 组件被移入或者移出可见窗口


### <u>位置与数量</u> 

onViewportChanged 方法中返回的位置与数量是相对于该 section 所含有的组件数而言的。例如：

	       Section A
	      /         \
	 Section B   Section C
	    |           |
	  10 items    10 items
	  (visible)   (hidden)
 
当 Section C 的第一个组件因为滑动进入窗口中，Section C 的 firstVisiblePosition 值就是 0 ，同时 Section B 的 lastVisiblePosition 的值就是 10 。

## @OnDataBound
 当此 Section 的层级结构中的数据变化对 SectionTree.Target 可见时，带有这个注解的方法会被调用。

数据变化有这几个原因：
1） 插入 
2)  更新
3） 删除
4） 移动

数据发生变化并不意味着这些数据在窗口中变得可见。为了检测可见性，使用 带有 @OnViewportChanged 注解的方法。

	class SectionSpec {
	
	  @OnDataBound
	  static void OnDataBound(
	    SectionContext c,
	    @Prop YourProp prop
	    @State YourState state) {
	    // Handle data changes
	  }
	}

## requestFocus()
 调用这个方法可以时 Section 中的组件获得焦点。如果一个组件在窗口中被隐藏，section 会被滚动以显示这个组件。

在请求获取焦点之前，用于渲染组件的数据必须可用。所以，仅当带有 @OnDataBound 的方法被调用之后再使用 requestFocus() 这个方法。

### <u>API</u>

requestFocus() 方法有几种不同的形式。


##### SectionLifecycle.requestFocus(SectionContext c, int index)

指定组件在 Section 中的索引来获取焦点，其中 Section 的作用域由  SectionContext 给出。

> 跟 @OnViewportChanged 一样，这里的索引对应于组件所在Section的组件数。

#### SectionLifecycle.requestFocus(SectionContext c, String sectionKey)

指定Section 的 Key ,让此 Section 中第一个组件获取焦点。

#### SectionLifecycle.requestFocusWithOffset(SectionContext c, int index, int offset)

与 SectionLifecycle.requestFocus(SectionContext c, int index) 相同，但是带有偏移量。

#### SectionLifecycle.requestFocusWithOffset(SectionContext c, String sectionKey, int offset)

与 SectionLifecycle.requestFocus(SectionContext c, String sectionKey) 相同，但是带有偏移量。

#### SectionLifecycle.requestFocus(SectionContext c, String sectionKey, FocusType focusType)

这里的FocusType 有两个值： 

1) FocusType.START 让 Section 中的第一个组件获取焦点。

2) FocusType.END  让 Section 中的最后一个组件获取焦点。


## @OnEvent(LoadingEvent.class)

当 Section 的子节点处于加载状态时，Section使用带有这个@OnEvent(LoadingEvent.class)注解的方法 来接受加载事件。

### <u>API</u>

	class YourSectionSpec {
	
	  @OnCreateChildren
	  protected static Children onCreateChildren(SectionContext c) {
	    return Children.create()
	        .child(
	          ChildSection
	            .create(c)
	            .loadingEventHandler(YourSection.onLoadingEvent(c)))
	        .build()
	    }
	  }
	
	  @OnEvent(LoadingEvent.class)
	  static void onLoadingEvent(
	    SectionContext c,
	    @FromEvent LoadingState loadingState,
	    @FromEvent boolean isEmpty,
	    @FromEvent Throwable t) {
	
	    switch (loadingState) {
	        case INITIAL_LOAD:
	        case LOADING:
	          // Handle loading
	          break;
	        case FAILED:
	          // Handle failure
	          break;
	        case SUCCEEDED:
	          // Handle success
	          break;
	    }
	
	    // Dispatch the same loading event up the hierarchy.
	    SectionLifecycle.dispatchLoadingEvent(
	        c,
	        isEmpty,
	        loadingState,
	        t);
	  }
	}

#### LoadingState：


1) INITIAL_LOAD 开始加载。

2) LOADING 加载进行中。

3) SUCCEEDED 加载成功。

4) FAILED 加载失败。

#### isEmpty：

加载完成之后数据集为空时则返回true 。

#### Throwable t

返回加载失败的原因。

### <u>向上传递加载事件</u>

加载事件要向上传递直到某个 Section 决定处理此次加载事件。如果某一 Section 处理加载事件时，这个 Section 的父 Sections 也想要处理此次加载事件，则需要将事件向上传递。

	SectionLifecycle.dispatchLoadingEvent(c, isEmpty, loadingState, t);