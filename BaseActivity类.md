##BaseActivit如何写
###一：BaseFrameActivity
####Fragement如何支持3.0以下的系统

继承`FragmentActivity`，就可以使用，Fragement的标准写法参见[这个网址](Fg)。虽然代码写的有些问题，但该表达的表达的还算清晰。比如为什么v4包要导入，在导不同的包的时候如何如何获取`FragementManage`，两者使用的方法是不一样的。

[Fg]:http://www.aplesson.com/?p=377

####FinishActivityUtil的使用
添加一个集中关闭的类，方便集中控制
<pre><code>这是一个代码块
	protected void onCreate(Bundle savedInstanceState)
	{
		mApplication = (GreeApplaction) getApplication();
		FinishActivityUtil.getInstance().mActivityList.add(this);
		super.onCreate(savedInstanceState);
	}

         protected void onDestroy()
	{
		super.onDestroy(); 
		if (mHelper != null)
		{
			OpenHelperManager.releaseHelper();
			mHelper = null;
		}

		FinishActivityUtil.getInstance().mActivityList.remove(this);
	}
</code></pre>
注意在不同的生命周期内的调用，这个类主要负责优雅的回退，结束activity，主要依靠的就是这个工具类。

####广播的接收集中处理
将广播的处理放在这里处理

---
###二：BaseIntentFrameActivity
####添加了intent的跳转
<pre><code>
    protected void onCreate(Bundle savedInstanceState)
	{
		if (GreeApplaction.allDeviceList == null)
		{
			Intent intent = new Intent(BaseIntentFrameActivity.this, LoadingActivity.class);
			startActivity(intent);
			return;
		}
		super.onCreate(savedInstanceState);
	}
</pre></code>

在列表为空的时候会主动加载loading页面

---
###三：TitleActivity
是一个FrameLayout，在界面内需要全部自己去摆位置
####FrameLayout
body
####RelativeLayout
内部放了10个控件，通过设置不可见的方式，设置出不同的title格式

* `Button`是返回箭头
* `LinearLayout`放了左右两个tab（是`button`）布局，仅仅用于放在preset定时界面
* `TextView`居中放置，用于显示当前页面的title文字
* `Button`,`ImageButton`右侧的加号，点击出现添加
