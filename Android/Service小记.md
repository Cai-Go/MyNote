---
# 先来扯点
Service作为Android四大组件之一,相信学过Android的都知道这货,它在每一个应用程序中都扮演着重要的角色.
Service是一个可以在后台执行长时间运行操作而不使用用户界面的应用组件.Service可由其他应用组件启动,而且即使用户切换到其他应用,Service仍将在后台继续运行.Service主要用于在后台处理一些耗时的逻辑,或者去执行某些需要长期运行的任务.必要的时候我们甚至可以在程序退出的情况下,让Service在后台继续保持运行状态.
Service和Activity很相似,但是区别在于:Service一直在后台运行,没有用户界面,所以不会到前台,如果Service被启动起来,就和Activity一样,具有自己的声明周期.

>注:在开发中,Activity和Service的选择标准是:如果程序组件在需要在运行时向用户呈现某种界面,或者程序需要与用户进行交互,那么使用Activity,否则考虑使用Service.

另外,需要注意的是,Service和Thread不是一个意思,不要被Service的后台概念所迷惑.实际上Service并不会自动开启线程,所有的代码都是默认运行在主线程中的.因此,我们需要在Service的内部手动创建子线程,并在这里执行具体的任务,否则可能造成ANR的问题.

# Service的形式
Service基本上分为两种形式:

- Started(启动的)
    + 当应用组件（如 Activity）通过调用 startService() 启动Service时,Service即处于“启动”状态.一旦启动,Service即可在后台无限期运行,即使启动Service的组件已被销毁也不受影响.通常,一个开启的Service执行单一操作并且不会给调用者返回结果.例如,它可能通过网络下载或上传文件.操作完成后,Service会自行停止运行.
- Bound(绑定的)
    + 当应用组件通过调用 bindService() 绑定到Service时,Service即处于“绑定”状态.一个绑定的Service提供客户端/服务器接口允许组件和Service交互,甚至跨进程操作使用进行间通信（IPC）.仅当与另一个应用组件绑定时,绑定服务才会运行.多个组件可以同时绑定到该服务,但全部取消绑定后,该服务即会被销毁.

# Service的使用
## 几个方法
想要创建一个service,你必须创建一个Service的子类（或一个它的存在的子类）.在你的实现中,你需要重写一些回调方法来处理Service生命周期的关键方面并且对于组件绑定到Service提供一个机制.应重写的最重要的回调方法包括:

- onStartCommand()
    + 当其它组件(例如activity)通过调用**startService()** 来请求Service启动时系统会调用这个方法.一旦这个方法执行,service就启动并且可以无限地运行在后台.如果你实现这个,它主要负责当任务完成后停止service,通过调用**stopSelf()** 或 **stopService()**.（如果你只想提供绑定,你不需要实现这个方法.）
- onBind()
    + 当其它组件通过调用**bindService()**来绑定到Service时系统会调用这个方法.在实现的这个方法中,你必须提供一个客户端用来和Service交互的接口,通过返回一个IBinder.你必须实现这个方法,但如果你不想允许绑定,你应该返回null.
- onCreate()
    + 首次创建Service时,系统将调用此方法来执行一次性设置程序（在调用 onStartCommand() 或 onBind() 之前）.如果服务已在运行,则不会调用此方法.
- onDestroy()
    + 当Service不再使用且将被销毁时,系统将调用此方法.Service应该实现此方法来清理所有资源,如线程、注册的侦听器、接收器等.这是Service接收的最后一个调用.


>如果希望在Service组件做某些事情,那么只要在onCreate()或onStratCommand()方法中定义相关业务代码即可.而当服务销毁时,我们应该在onDestroy()方法中去回收那些不再使用的资源.

## 使用清单文件声明服务
如同 Activity（以及其他组件）一样,您必须在应用的清单文件中声明所有服务.
````
<manifest ... >
  ...
  <application ... >
      <service android:name=".XXXService" />
      ...
  </application>
</manifest>
````

## 创建启动Service
直接用代码了,比较清楚.

````
  @Override  
    public void onCreate() {  
        super.onCreate();  
        Log.d(TAG, "onCreate() executed");  
    }  
  
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {  
        Log.d(TAG, "onStartCommand() executed");  
        return super.onStartCommand(intent, flags, startId);  
    }  
      
    @Override  
    public void onDestroy() {  
        super.onDestroy();  
        Log.d(TAG, "onDestroy() executed");  
    }  
  
    @Override  
    public IBinder onBind(Intent intent) {  
        return null;  
    }  
````
然后在activity_main中添加2个按钮.
````
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="vertical" >  
  
    <Button  
        android:id="@+id/start_service"  
        android:layout_width="match_parent"  
        android:layout_height="wrap_content"  
        android:text="Start Service" />  
  
    <Button  
        android:id="@+id/stop_service"  
        android:layout_width="match_parent"  
        android:layout_height="wrap_content"  
        android:text="Stop Service" />  
  
</LinearLayout>  
````

接着在MainActivity中添加以下代码.
````
public class MainActivity extends Activity implements OnClickListener {  
    private Button startService;  
    private Button stopService;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        startService = (Button) findViewById(R.id.start_service);  
        stopService = (Button) findViewById(R.id.stop_service);  
        startService.setOnClickListener(this);  
        stopService.setOnClickListener(this);  
    }  
  
    @Override  
    public void onClick(View v) {  
        switch (v.getId()) {  
        case R.id.start_service:  
            Intent startIntent = new Intent(this, MyService.class);  
            startService(startIntent);  
            break;  
        case R.id.stop_service:  
            Intent stopIntent = new Intent(this, MyService.class);  
            stopService(stopIntent);  
            break;  
        default:  
            break;  
        }  
    }  
}  
````
最后在AndroidManifest.xml中注册.
````
    <service android:name="com.example.servicetest.MyService" >  </service>  
````

这里我们重写了onCreate()、onStartCommand()、onDestroy()这三个方法.

- onCreate():在服务创建的时候调用.
- onStartCommand():在每次服务启动的时候调用.
- onDestroy():在服务销毁的时候调用.
运行之后,我们做了以下事情.
当点击STARTSERVICE按钮后,控制台打印的Log如图:
![](http://7xq2jk.com1.z0.glb.clouddn.com/startService.png)
当在此点击STARTSERVICE按钮后,控制台打印的Log如图:
![](http://7xq2jk.com1.z0.glb.clouddn.com/7241.png)
我们发现此时onCreate()方法没有执行,onStartCommand()继续执行了.多点几次后还是这样.为什么会这样呢？这是由于onCreate()方法只会在Service第一次被创建的时候调用,如果当前Service已经被创建过了,不管怎样调用startService()方法,onCreate()方法都不会再执行.因此你可以再多点击几次Start Service按钮试一次,每次都只会有onStartCommand()方法中的打印日志.
当点击STOPSERVICE按钮后,控制台打印的Log如图:
![](http://7xq2jk.com1.z0.glb.clouddn.com/StopService.png)

>注:在MyService的任何一个位置调用stopSelf()方法就能让Service自己停下来

## Service和Activity通信
接下来,我们让Service和Activity的关系更紧密一些,来实现在Activity中指挥Service去干活.而这需要借助onBind()方法.
还是看代码吧.
修改MyService里面的代码:
````
...
private MyBinder mBinder = new MyBinder();
...
  @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

     class MyBinder extends Binder {
         public void startDownload(){
             Logger.t(TAG).d("执行startDownload()方法");
             //执行具体下载任务
         }
    }

````
这里我们新增了一个MyBinder类继承自Binder类,然后在MyBinder中添加了一个startDownload()方法用于在后台执行下载任务.
然后再在activity_main中添加2个按钮
````
 <Button
        android:id="@+id/bind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Bind Service" />

    <Button
        android:id="@+id/unbind_service"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Unbind Service"
        />
````
修改MainActivity的代码:
````
 public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private Button start, stop;
    private Button bindService;

    private Button unbindService;

    private MyService.MyBinder myBinder;

    private ServiceConnection connection = new ServiceConnection() {

        @Override
        public void onServiceDisconnected(ComponentName name) {
        }

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            myBinder = (MyService.MyBinder) service;
            myBinder.startDownload();
        }
    };


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        start = (Button) findViewById(R.id.Start_Service);
        stop = (Button) findViewById(R.id.Stop_Service);
        bindService = (Button) findViewById(R.id.bind_service);
        unbindService = (Button) findViewById(R.id.unbind_service);
        bindService.setOnClickListener(this);
        unbindService.setOnClickListener(this);


        start.setOnClickListener(this);
        stop.setOnClickListener(this);
    }

    @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.Start_Service:
                Intent startIntent = new Intent(this,MyService.class);
                startService(startIntent);
                break;
            case R.id.Stop_Service:
                Intent stopIntent = new Intent(this,MyService.class);
                stopService(stopIntent);
                break;
            case R.id.bind_service:
                Intent bindIntent = new Intent(this, MyService.class);
                bindService(bindIntent, connection, BIND_AUTO_CREATE);
                break;
            case R.id.unbind_service:
                unbindService(connection);
                break;
        }
    }
}


````
可以看到,这里我们首先创建了一个ServiceConnection的匿名类,在里面重写了onServiceConnected()方法和onServiceDisconnected()方法,这两个方法分别会在Activity与Service建立关联和解除关联的时候调用.在onServiceConnected()方法中,我们又通过向下转型得到了MyBinder的实例,有了这个实例,Activity和Service之间的关系就变得非常紧密了.
当然,现在Activity和Service其实还没关联起来了呢,这个功能是在Bind Service按钮的点击事件里完成的.可以看到,这里我们仍然是构建出了一个Intent对象,然后调用bindService()方法将Activity和Service进行绑定.*bindService()方法接收三个参数,第一个参数就是刚刚构建出的Intent对象,第二个参数是前面创建出的ServiceConnection的实例,第三个参数是一个标志位.*这里传入BIND_AUTO_CREATE表示在Activity和Service建立关联后自动创建Service,这会使得MyService中的onCreate()方法得到执行,但onStartCommand()方法不会执行.
然后如何我们想解除Activity和Service之间的关联怎么办呢？调用一下unbindService()方法就可以了,这也是Unbind Service按钮的点击事件里实现的逻辑.
好的,我们点击试试.
点击BIND SERVICE按钮,输出如图:
![](http://7xq2jk.com1.z0.glb.clouddn.com/7242.png)
当再多次点击则不会再有Log输出.
点击UBIND SERVICE按钮,输出如图:
![](http://7xq2jk.com1.z0.glb.clouddn.com/7243.png)

>注:任何一个Service在整个应用程序范围内都是通用的,即MyService不仅可以和MainActivity建立关联,还可以和任何一个Activity建立关联,而且在建立关联时它们都可以获取到相同的MyBinder实例.

## 销毁Service的方法
根据之前的测试,我们知道,点击STARTSERVICE按钮启动Service,再点击STOPSERVICE按钮停止Service,这样MyService就被销毁了.
如果我们是点击的Bind Service按钮呢？由于在绑定Service的时候指定的标志位是BIND_AUTO_CREATE,说明点击BindService按钮的时候Service也会被创建,这时应该呢？其实也很简单,点击一下UnbindService按钮,将Activity和Service的关联解除,从而销毁Service.
如果我们既点击了StartService按钮,又点击了BindService按钮,这个时候你会发现,不管你是单独点击Stop Service按钮还是Unbind Service按钮,Service都不会被销毁,必要将两个按钮都点击一下,Service才会被销毁.也就是说,点击StopService按钮只会让Service停止,点击Unbind Service按钮只会让Service和Activity解除关联,一个Service必须要在既没有和任何Activity关联又处理停止状态的时候才会被销毁.
我们测试下.
````
public void onClick(View v) {  
    switch (v.getId()) {  
    case R.id.start_service:  
        Intent startIntent = new Intent(this, MyService.class);  
        startService(startIntent);  
        break;  
    case R.id.stop_service:  
        Log.d("MyService", "click Stop Service button");  
        Intent stopIntent = new Intent(this, MyService.class);  
        stopService(stopIntent);  
        break;  
    case R.id.bind_service:  
        Intent bindIntent = new Intent(this, MyService.class);  
        bindService(bindIntent, connection, BIND_AUTO_CREATE);  
        break;  
    case R.id.unbind_service:  
        Log.d("MyService", "click Unbind Service button");  
        unbindService(connection);  
        break;  
    default:  
        break;  
    }  
}  

````
重新运行程序,先点击一下STARTSERVICE按钮,再点击一下BindService按钮,这样就将Service启动起来,并和Activity建立了关联.然后点击StopService按钮后Service并不会销毁,再点击一下UnbindService按钮,Service就会销毁了,打印日志如下所示:
![](http://7xq2jk.com1.z0.glb.clouddn.com/7224.4.png)

>注:根据Android系统的机制,一个服务只要被启动或者被绑定之后,就会一直处于运行状态

## Service的生命周期
服务的生命周期比 Activity 的生命周期要简单得多.但是,密切关注如何创建和销毁服务反而更加重要,因为服务可以在用户没有意识到的情况下运行于后台.
服务生命周期（从创建到销毁）可以遵循两条不同的路径:

- 启动服务
    + 该服务在其他组件调用 startService() 时创建,然后无限期运行,且必须通过调用 stopSelf() 来自行停止运行.此外,其他组件也可以通过调用 stopService() 来停止服务.服务停止后,系统会将其销毁.
- 绑定服务
    + 该服务在另一个组件（客户端）调用 bindService() 时创建.然后,客户端通过 IBinder 接口与服务进行通信.客户端可以通过调用 unbindService() 关闭连接.多个客户端可以绑定到相同服务,而且当所有绑定全部取消后,系统即会销毁该服务. （服务不必自行停止运行.）
这两条路径并非完全独立.也就是说,您可以绑定到已经使用 startService() 启动的服务.例如,可以通过使用 Intent（标识要播放的音乐）调用 startService() 来启动后台音乐服务.随后,可能在用户需要稍加控制播放器或获取有关当前播放歌曲的信息时,Activity 可以通过调用 bindService() 绑定到服务.在这种情况下,除非所有客户端均取消绑定,否则 stopService() 或 stopSelf() 不会真正停止服务.
### 实现生命周期回调
与 Activity 类似,服务也拥有生命周期回调方法,您可以实现这些方法来监控服务状态的变化并适时执行工作.
看代码:
````
public class ExampleService extends Service {
    int mStartMode;       // indicates how to behave if the service is killed
    IBinder mBinder;      // interface for clients that bind
    boolean mAllowRebind; // indicates whether onRebind should be used

    @Override
    public void onCreate() {
        // The service is being created
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }
    @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService()
        return mBinder;
    }
    @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }
    @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }
    @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
````

>注:与 Activity 生命周期回调方法不同,您不需要调用这些回调方法的超类实现.

![Service生命周期](http://7xq2jk.com1.z0.glb.clouddn.com/service_lifecycle.png)
服务生命周期左图显示了使用 startService() 所创建的服务的生命周期,右图显示了使用 bindService() 所创建的服务的生命周期.对于启动服务,有效生命周期与整个生命周期同时结束（即便是在 onStartCommand() 返回之后,服务仍然处于活动状态）.对于绑定服务,有效生命周期在 onUnbind() 返回时结束.

通过实现这些方法,您可以监控服务生命周期的两个嵌套循环:

- 服务的整个生命周期从调用 onCreate() 开始起,到 onDestroy() 返回时结束.与 Activity 类似,服务也在 onCreate() 中完成初始设置,并在 onDestroy() 中释放所有剩余资源.例如,音乐播放服务可以在 onCreate() 中创建用于播放音乐的线程,然后在 onDestroy() 中停止该线程.
无论服务是通过 startService() 还是 bindService() 创建,都会为所有服务调用 onCreate() 和 onDestroy() 方法.
- 服务的有效生命周期从调用 onStartCommand() 或 onBind() 方法开始.每种方法均有 Intent 对象,该对象分别传递到 startService() 或 bindService().

>注:尽管启动服务是通过调用 stopSelf() 或 stopService() 来停止,但是该服务并无相应的回调（没有 onStop() 回调）.因此,除非服务绑定到客户端,否则在服务停止时,系统会将其销毁—onDestroy() 是接收到的唯一回调.


---
参考:
[Service](https://developer.android.com/guide/components/services.html)
[Android Service完全解析,关于服务你所需知道的一切(上)](http://blog.csdn.net/guolin_blog/article/details/11952435#t0)
[ Android Service完全解析,关于服务你所需知道的一切(下)](http://blog.csdn.net/guolin_blog/article/details/9797169)
[Android开发指南——Service](http://yuweiguocn.github.io/2016/04/02/android-guide-service/)
《第一行代码》第9章
《疯狂Android讲义第2版》第10章