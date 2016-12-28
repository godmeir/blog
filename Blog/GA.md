##如何使用谷歌分析 API v4 为 Android 
最近一次的项目需求更改时，外国客户要求加上Google Analytics,对GA进行了一下研究，现将测试代码记录一下。

由于原项目一直保留在ecplise上没有进行迁移，所以就对[GA官网][GA]上的方式进行了一下研究，官网的方式是针对android studio来做的。在ecplise上做的时候发现找不到v4版本的GA的API，经过一番费劲才明白，GA的功能已经集成到了service里面了，所以直接添加一下service就可以了。

基于android studio来做的话，直接配置gradle来做就好，但是记住`play-services-analytics`是基于`google-services`的，所以添加modle时注意添加`apply plugin: 'com.google.gms.google-services'`这句话。在获取配置文件的步骤，在这个步骤可以获取到我们track ID,这按官网的步骤就好，到最后我们这个包名什么的都是随意的，以项目可以没有关系。而且最后我们得到一个`google-services.json`文件，官网上对这个文件这样说:
>将您刚刚下载的 google-services.json 文件复制到您 Android Studio 项目的 app/ 或 mobile/ 目录中。

其实这个是没有必要的，下面详细介绍一下ecplise和android studio下的添加方式。

###1. 使用ecplise开发android应用如何接入Google Analytics

1. 创建andlytic账号
	
	在[这个网址][newid]上创建你的账号，可以一步步的获取到你的trackid，在你已经有ga的账户的情况下，也可以由上文的**获取配置文件**的方式获取id。最后获取到的trackid为UA-xxxx-x,这个就是你需要的追踪码。
2. 下载`google-services`SDK

	在v3版本中需要下载`Google Analytics Services SDK`，实际得到的是`libGoogleAnalyticsServices.jar`文件，但是在v4版本的API中，这个方式已经不再适用了，因为官方就没有专门的GA SDK，所以是无法得到文件的。不过我们可以直接使用`google-services`,这个文件已经存在在android sdk中，我们直接复制出来就好。路径是`<android-sdk>/extras/google/google_play_services/libproject/google-play-services_lib/`复制到你的libs文件夹，添加一下路径就好。

3. 配置analystics.xml，放在res/xml文件夹下即可文件内容如下：


<resources>

    <!-- 您要向其发送数据的Google Analytics（分析）跟踪ID。ID中的短划线必须未经编码。您可以不提供此值，以此停用跟踪功能。 -->
    <string name="ga_trackingId">UA-xxxxxx-5</string>

    <!-- 每次用户启动Activity时自动跟踪屏幕浏览量。默认值为false。 -->
    <bool name="ga_autoActivityTracking">false</bool>

    <!-- 每次您的应用中出现未捕获的异常时，自动对其进行跟踪。默认值为false。 -->
    <bool name="ga_reportUncaughtExceptions">true</bool>

    <!-- SDK日志记录器的详细程度。从最简略到最详细的有效值分别为：error、warning、info、verbose。日志级别默认设置为warning。 -->
    <string name="ga_logLevel">warning</string>

    <!-- 数据发送间隔，以秒为单位。默认值为30分钟。 -->
    <integer name="ga_dispatchPeriod">20</integer>

    <!-- 要使用的抽样率。默认值为100.0。可以是0.0和100.0之间的任何值。 -->
    <string name="ga_sampleFrequency">100.0</string>

    <!-- 您的应用在会话结束前可在后台停留的时间（以秒为单位）。默认值为30秒。将此值设为负值即可停用EasyTracker会话管理。 -->
    <integer name="ga_sessionTimeout">30</integer>
    
	 <!-- 设置自动追踪的屏幕的名称 -->
    <screenName name="com.example.myweixin.MainActivity">MainScreen</screenName>

</resources>
4. 在`AndroidManifest.xml`文件中添加权限：

	    <uses-permission android:name="android.permission.INTERNET"/>
        <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

	添加：

	    <meta-data android:name="com.google.android.gms.version"
                   android:value="@integer/google_play_services_version" />
	将其作为一个你的application的一个item。

5.添加追踪代码

追踪代码写在自定义的application中，这样可以不用再每次使用的时候都进行创建
<pre><code>这是写在application中的代码
    /**
	 * Gets the default {@link Tracker} for this {@link Application}.
	 * @return tracker
	 */
	synchronized public Tracker getDefaultTracker() {
        if (mTracker == null) {
            GoogleAnalytics analytics = GoogleAnalytics.getInstance(this);
            // To enable debug logging use: adb shell setprop log.tag.GAv4 DEBUG
            mTracker = analytics.newTracker(R.xml.analytics);
        }
        return mTracker;
    }
</code></pre>
<pre><code>这是写在需要追踪的位置的代码，一个事件可以分为四个部分类型、动作、标签
值（category， action，lable, value）
       
        application = (AnalyticsApplication) this.getApplication();
        mTracker = application.getDefaultTracker();

        bn.setOnClickListener(new OnClickListener() {
		@Override
		public void onClick(View v) {
			// [START custom_event]
		       mTracker.send(new HitBuilders.EventBuilder()
		           .setCategory("Action")
		           .setAction("ButtonPress")
				   .setLabel("Edit")
		           .build());
		    // [END custom_event]
		       
			}
		});
</code></pre>
<pre><code>需要追踪的页面上也可以自定义，因为我们设置的页面自动追踪获取的是包名.类名
当然我们可以通过analystics.xml配置自动追踪的屏幕的名称（看该文件的最后一行）
	 mTracker.setScreenName("Image~");
	 mTracker.send(new HitBuilders.ScreenViewBuilder().build());

</code></pre>



###2. 使用android studio开发android应用如何接入Google Analytics
as下的用法完全转自[GA官方API][GAPI]

本指南介绍了如何将 Google Analytics（分析）添加到您的 Android 应用以衡量用户在已命名屏幕上的活动。如果您目前没有应用，而是仅仅想了解一下 Google Analytics（分析）的工作原理，请参阅我们的[示例应用][Exam]。

1. 设置项目
	更新您项目的 `AndroidManifest.xml` 文件，以使其包含 `INTERNET` 和 `ACCESS_NETWORK_STATE` 权限.做法相同。
	
	适用于 Gradle 的 Google 服务插件会解析 `google-services.json` 文件中的配置信息。通过更新您的顶级 `build.gradle` 和应用一级`build.gradle` 文件来将该插件添加到您的项目中，
	
	具体操作如下所示：
	将下面的依赖关系添加到您项目的顶级 build.gradle 中：
	`classpath 'com.google.gms:google-services:1.3.0-beta1'`
	
	将下面的插件添加到您的应用一级 build.gradle 中：
	`apply plugin: 'com.google.gms.google-services'`
	
	现在，您需要为 Google Play 服务添加一个依赖关系。为此，请在您应用的 build.gradle 中添加以下内容：
	  `compile 'com.google.android.gms:play-services-analytics:7.3.0' `
2. 将您刚刚下载的 `google-services.json` 文件复制到您 Android Studio 项目的 app/ 或 mobile/ 目录中.（也可以不添加完全手动添加）
3. 添加事件的方式和在ecplise相同，只要以上的配置成功，就不会有问题。
如若采用自动添加时，application类中的获取方式可以直接使用：
<pre><code>不用自己去设定track的属性了，而且包括trackid在内，都包含
`google-services.json`中了，但是不是自己配置的毕竟不放心，这一步也可以通过创建
`res/xml/global_tracker.xml`来生成。

 synchronized public Tracker getDefaultTracker() {
    if (mTracker == null) {
      GoogleAnalytics analytics = GoogleAnalytics.getInstance(this);
      // To enable debug logging use: adb shell setprop log.tag.GAv4 DEBUG
      mTracker = analytics.newTracker(R.xml.global_tracker);
    }
    return mTracker;
  }
</code></pre>
###总结
建议使用Android Studio，毕竟Google的亲儿子，支持更好。后续的版本的跟进也更方便。

GA的使用还没有达到很熟练的地步，而且完全是按照AS的方式写的ecplise的，如有错误，欢迎斧正。后续会就深入研究再写一篇。
	






	



 
	

	



[GA]:https://developers.google.com/analytics/devguides/collection/android/v4/app?configured=true#application
[newid]:https://www.google.com/analytics/
[GAPI]:https://developers.google.com/analytics/devguides/collection/android/v4/app
[Exam]: https://developers.google.com/analytics/devguides/collection/android/v4/start