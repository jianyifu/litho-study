# Services

## Service


**<u>数据流</u>**

数据在被渲染在屏幕上之前会流经 Sections 。 Sections 在数据源与 UI 之间高效计算组件渲染的变更集。

为了利用更优的性能，我们只须做需要做的工作。因此，理想情况下，只有 Section 需要数据的时候才获取数据。这就是Service 的工作。

**<u>介绍</u>**

Sections 使用 Services 命令式的控制数据源。通过 Service API , 可以在 Section 层级结构之外使用 [SectionLifecycle](https://fblitho.com/javadoc/com/facebook/litho/sections/SectionLifecycle.html) 。 这可以让你知道何时开始获取数据。

因为 Service 是被绑定到特定的 Section 上，也就是说 Service 可以使用所有 Prop 以及 State ，并和它们交互，也意味着 Service 可以对例如 @OnViewportChanged 和 @OnRefresh 等事件作出响应。当新数据抵达时，请求状态更新，让数据沿着 Section 层级结构流动 。 



**@<u>OnCreateService</u>**

Services 存活于状态更新以及 Sections 停留在层级结构的期间。 第一个也是仅有的 Service 实例应该在生命周期方法 @OnCreateService 中创建。

	@GroupSectionSpec
	public ServiceSectionSpec {
	  ...
	  @OnCreateService
	  static DataLoader onCreateServices(
	    SectionContext c,
	    @Prop Configuration config,
	    ...) {
	      /**
	       * onCreateServices() is called only once when the Section is first created.
	       * In this function you should create and return your service.
	       **/
	      return new DataLoaderFactory.createLoader(config);
	  }
	}


**@<u>OnBindService</u> 与 @<u>OnUnbindService</u>**


@OnBindService 是个回调方法，里面定义 Service 如何与 Section 进行交互。当数据加载完成，新的数据被传到 Section 中时调用这个方法。

@OnUnbindService 提供回调用于清理以及撤销在 @OnBindService 执行过的操作。


	@GroupSectionSpec
	public ServiceSectionSpec {
	  ...
	  @OnBindService
	  static void onBindService(
	    final SectionContext c,
	    final DataLoader service,
	    ...) {
	      /**
	       * onBindService() is called (along with onUnbindService()) every time
	       * the section tree is updated (usually because of a state update).
	       * This function is passed the service created by onCreateService as the second function parameter.
	       * In this function you should bind any EventHandler that will make state changes to your service.
	       **/
	      service.registerEventLoader(ServiceSection.dataLoaded(c));
	  }
	
	  @OnUnbindService
	  static void onUnbindService(
	    final SectionContext c,
	    final DataLoader service,
	    ...) {
	      /**
	       * onUnBindService() is called (along with onBindService()) every time
	       * the section tree is updated (usually because of a state update).
	       * This function is passed the service created by onCreateService as the second function parameter.
	       * This should be the inverse of onBindService(). Anything set or bound in onBindService() should be undone here.
	       **/
	      service.unregisterEventLoader();
	  }
	
	  @OnEvent(YourData.class)
	  static void dataLoaded(
	    final SectionContext c,
	    @FromEvent final Data data) {
	      // Update your state with the new data
	      ServiceSection.updateData(c, data);
	  }
	
	  @UpdateState
	  static void updateData(
	    final StateValue<Data> connectionData,
	    @Param Data data) {
	      connectionData.set(data);
	  }
	}


**<u>数据获取</u>**

如上所述， services 可以对例如 @OnViewportChanged 和 @OnRefresh 等事件作出响应：

	 @OnRefresh
	 static void onRefresh(
	   SectionContext c,
	   DataLoader service,
	   ...) {
	     service.refreshData();
	 }
	
	 @OnViewportChanged
	 static void onViewportChanged(
	   SectionContext c,
	   int firstVisibleIndex,
	   int lastVisibleIndex,
	   int totalCount,
	   int firstFullyVisibleIndex,
	   int lastFullyVisibleIndex,
	   DataLoader service,
	   ...) {
	     service.makeTailFetch();
	 }

