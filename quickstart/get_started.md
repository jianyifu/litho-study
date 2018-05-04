# 新手入门

##  将Litho引入到项目中

我们把Litho发布到了Bintray的JCenter仓库中。要将Litho引入到Android项目中，确保在build.gradle文件中添加对JCenter的引用。
	
	repositories {
	  jcenter()
	}

之后添加如下依赖：

	dependencies {
	  // ...
	  // Litho
	  implementation 'com.facebook.litho:litho-core:0.14.0'
	  implementation 'com.facebook.litho:litho-widget:0.14.0'
	  compileOnly 'com.facebook.litho:litho-annotations:0.14.0'
	
	  annotationProcessor 'com.facebook.litho:litho-processor:0.14.0'
	
	  // SoLoader
	  implementation 'com.facebook.soloader:soloader:0.2.0'
	
	  // For integration with Fresco
	  implementation 'com.facebook.litho:litho-fresco:0.14.0'
	
	  // For testing
	  testImplementation 'com.facebook.litho:litho-testing:0.14.0'
	}

##  在项目中添加Sections的依赖

Litho自带Sections可选库用于声明式构建列表，引入Sections需要额外把以下依赖添加到build.gradle文件中。

	dependencies {
	
	  // Sections
	  implementation 'com.facebook.litho:litho-sections-core:0.14.0'
	  implementation 'com.facebook.litho:litho-sections-widget:0.14.0'
	  compileOnly 'com.facebook.litho:litho-sections-annotations:0.14.0'
	
	  annotationProcessor 'com.facebook.litho:litho-sections-processor:0.14.0'
	}

##  使用快照版本

>IMPORTANT: 快照版本可能会摧毁或者把
>家的房子烧掉（ps:对项目造成较大影响），快照版本未经签名并且由我们的CI系统自动发布，确保仅将快照版本用于测试目的。

首先，将Sonatype快照库添加到Gradle配置中：

	repositories {
	  maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
	}

之后就可以访问到我们发布的所有快照版本：

	dependencies {
	  // ...
	  // Litho
	  implementation 'com.facebook.litho:litho-core:0.14.1-SNAPSHOT'
	  implementation 'com.facebook.litho:litho-widget:0.14.1-SNAPSHOT'
	  compileOnly 'com.facebook.litho:litho-annotations:0.14.1-SNAPSHOT'
	
	  annotationProcessor 'com.facebook.litho:litho-processor:0.14.1-SNAPSHOT'
	
	  // SoLoader
	  implementation 'com.facebook.soloader:soloader:0.2.0'
	
	  // For integration with Fresco
	  implementation 'com.facebook.litho:litho-fresco:0.14.1-SNAPSHOT'
	
	  // For testing
	  testImplementation 'com.facebook.litho:litho-testing:0.14.1-SNAPSHOT'
	}

##  安装测试

在Activity中添加一个用Litho创建的View来测试安装是否成功。

首先，初始化**SoLoader**.Litho依赖[SoLoader](https://github.com/facebook/SoLoader)帮助加载由底层的布局引擎[Yoga](https://facebook.github.io/yoga/)提供的本地库。应用的Application里适合进行初始化：

	[MyApplication.java]

	public class MyApplication extends Application {
	
	  @Override
	  public void onCreate() {
	    super.onCreate();
	
	    SoLoader.init(this, false);
	  }
	}

之后添加Litho预定义的**Text**组件到Activity中显示 “Hello World!”。

	[MyActivity.java]

	import com.facebook.litho.ComponentContext;
	import com.facebook.litho.LithoView;
	
	public class MyActivity extends Activity {
	
	  @Override
	  public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	
	    final ComponentContext c = new ComponentContext(this);
	
	    final LithoView lithoView = LithoView.create(
	    	this /* context */,
	    	Text.create(c)
	            .text("Hello, World!")
	            .textSizeDip(50)
	            .build());
	
	    setContentView(lithoView);
	  }
	}

现在运行APP，可以在屏幕上看到 “Hello World!”。