#什么是内存泄漏
内存泄露，简单的说，就是该被释放的内存没有被释放，一直被某个或某些实例所引用但不能被使用，导致GC不能回收，造成内存泄漏。总结的说，可以理解为长生命周期的对象一直持有短生命周期对象的引用，导致短生命周期对象一直被引用而无法被GC回收，内存泄漏是造成OOM的主要原因之一，当一个应用中产生的内存泄漏比较多时，就难免会导致应用所需要的内存超过这个系统分配的内存限额，这就造成了内存溢出而导致应用Crash。
#安卓中常见的内存泄漏场景
##1.单例造成内存泄漏
因为单例模式有其静态的特点，其生命周期和应用一样长，如果单例对象中包含了一个其他对象的引用，那么即使这个对象不再使用，依然存在一个单例对象引用它，造成无法回收。比如：

<pre><code>
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.context = context;
    }
    public static AppManager getInstance(Context context) {
        if (instance != null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
</code></pre>
这个单例对象包含了一个Context的引用，所以此时要考虑，

* 如果传入一个Application Context,它的生命周期和应用一样长，没问题。
* 如果传入一个Activity Context,Activity退出销毁但任然被单例所引用了，会导致内存泄漏。


所以正确的单例应该修改为下面这种方式：

<pre><code>
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.context = context.getApplicationContext();
    }
    public static AppManager getInstance(Context context) {
        if (instance != null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
</code></pre>

##2.非静态内部类创建其静态实例造成内存泄漏
有的时候我们可能会在启动频繁的Activity中，为了避免重复创建相同的数据资源，会出现这种写法：


<pre><code>
public class MainActivity extends AppCompatActivity {
    private static TestResource mResource = null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mManager == null){
            mManager = new TestResource();
        }
        //...
    }
    class TestResource {
        //...
    }
}
</code></pre>
这样就在Activity内部创建了一个非静态内部类的单例，每次启动Activity时都会使用该单例的数据，这样虽然避免了资源的重复创建，不过这种写法却会造成内存泄漏，因为非静态内部类默认会持有外部类的引用，而又使用了该非静态内部类创建了一个静态的实例，该实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，导致Activity的内存资源不能正常回收。正确的做法为：

将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，请使用Application Context 。

##3.匿名内部类/异步线程造成内存泄漏
如下这两个示例可能每个人都这样写过：

<pre><code>
//——————test1
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                SystemClock.sleep(10000);
                return null;
            }
        }.execute();
//——————test2
        new Thread(new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(10000);
            }
        }).start();
</code></pre>
上面的异步任务和Runnable都是一个匿名内部类，因此它们对当前Activity都有一个隐式引用。如果Activity在销毁之前，任务还未完成， 那么将导致Activity的内存资源无法回收，造成内存泄漏。正确的做法还是使用静态内部类的方式，如下：
<pre><code>
static class MyAsyncTask extends AsyncTask<Void, Void, Void> {
        private WeakReference<Context> weakReference;
 
        public MyAsyncTask(Context context) {
            weakReference = new WeakReference<>(context);
        }
 
        @Override
        protected Void doInBackground(Void... params) {
            SystemClock.sleep(10000);
            return null;
        }
 
        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            MainActivity activity = (MainActivity) weakReference.get();
            if (activity != null) {
                //...
            }
        }
    }
    static class MyRunnable implements Runnable{
        @Override
        public void run() {
            SystemClock.sleep(10000);
        }
    }
//——————
    new Thread(new MyRunnable()).start();
    new MyAsyncTask(this).execute();
</code></pre>

这样就避免了Activity的内存资源泄漏，当然在Activity销毁时候也应该取消相应的任务AsyncTask::cancel()，避免任务在后台执行浪费资源。

##4.Handler机制造成内存泄漏
这个在上一篇也有说到，再次总结下。一般我们写Handler:

<pre><code>
public class MainActivity extends AppCompatActivity {
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            //...
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        loadData();
    }
    private void loadData(){
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
</code></pre>
其中mHandler作为一个非静态匿名内部类，持有一个外部类—MainActivity的引用，我们知道对于消息机制是Looper不断的轮询从消息队列取出未处理的消息交给handler处理，而对于这个例子，每一个消息又持有一个mHandler的引用，每一个mHandler又持有MainActivity的引用，所以如果在Activity退出后，消息队列中还存在未处理完的消息，导致该Activity一直被引用，其内存资源无法被回收，导致了内存泄漏。一般我们使用静态内部类和弱引用的写法写Handler。
<pre><code>
public class MainActivity extends AppCompatActivity {
    private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }
 
    private void loadData() {
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
</code></pre>

将Activity的引用声明为弱引用，可以被GC回收。（引用不一定为Context,也可以是Context的子类）

Java对引用的分类有 Strong reference, SoftReference, WeakReference, PhatomReference 四种。

![image][table]

在Android应用的开发中，为了防止内存溢出，在处理一些占用内存大而且声明周期较长的对象时候，可以尽量应用软引用和弱引用技术。

##5.资源未回收导致内存泄漏
对于使用了BraodcastReceiver，ContentObserver，File，Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏(这个可以根据IDE的警告改进一部分)。

#AS中查看内存泄漏
参考http://www.2cto.com/kf/201512/455421.html
![image][as]

#总结
对 Activity 等组件的引用应该控制在 Activity 的生命周期之内； 如果不能就考虑使用 getApplicationContext 或者 getApplication，以避免 Activity 被外部长生命周期的对象引用而泄露。

尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context )，即使要使用，也要考虑适时把外部成员变量置空；也可以在内部类中使用弱引用来引用外部类的变量。

对于生命周期比Activity长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄漏：

<pre><code>
    将内部类改为静态内部类
    静态内部类中使用弱引用来引用外部类的成员变量
</code></pre>

Handler 的持有的引用对象最好使用弱引用，资源释放时也可以清空 Handler 里面的消息。比如在 Activity onStop 或者 onDestroy 的时候，取消掉该 Handler 对象的 Message和 Runnable.

在 Java 的实现过程中，也要考虑其对象释放，最好的方法是在不使用某对象时，显式地将此对象赋值为 null，比如使用完Bitmap 后先调用 recycle()，再赋为null,清空对图片等资源有直接引用或者间接引用的数组（使用 array.clear() ; array = null）等，最好遵循谁创建谁释放的原则。

正确关闭资源，对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。

保持对对象生命周期的敏感，特别注意单例、静态对象、全局性集合等的生命周期。

###参考  
https://github.com/GeniusVJR/LearningNotes/blob/master/Part1/Android/Android%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E6%80%BB%E7%BB%93.md

原文：
http://www.iwfu.me/2016/08/02/%E5%AE%89%E5%8D%93%E9%9D%A2%E8%AF%95%E9%A2%98-5-%E5%85%B3%E4%BA%8E%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F/

<pre><code></code></pre>

[table]:https://camo.githubusercontent.com/068950c506eddc68677d6a84b5d7ffe1c03c6cf9/68747470733a2f2f67772e616c6963646e2e636f6d2f7470732f5442315536544e4c565858585863685846585858585858585858582d3634342d3534362e6a7067

[as]:http://www.2cto.com/uploadfile/Collfiles/20151227/20151227094556184.png