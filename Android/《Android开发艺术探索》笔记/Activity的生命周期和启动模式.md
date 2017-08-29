# Activity的生命周期和启动模式
## Activity的生命周期

#### 典型情况下的生命周期(指在有用户参与的情况下，Activity所经过的生命周期的改变)。
---
- onCreate:表示Activity正在被创建，是生命周期的第一个方法。这个方法里可以做一些初始化工作。
- onRestart:表示Activity正在重新启动。一般当当前Activity从不可见重新变为可见状态时会调用这个方法。如按下Home键或者打开一个新的Acticity，会执行onPause,onStop方法，接着又回到这个Activity，就会调用这个方法。
- onStar:表示Activity正在被启动，即将开始。此时Activity已经可见，但还没有出现在前台，无法和用户交互。
- onResume:表示Activity已经可见，并且出现在前台并开始活动。
- onPause:表示Activity正在停止，正常情况下会紧接着调用onStop(特殊情况是如果这时候快速回到当前Activity，会调用onResume)。此时可以做一些存储数据，停止动画等工作，但不能太耗时。
- onStop表示Activity即将停止。可以做一些稍微重量级的回收工作，但也不能太耗时。
- onDestroy:表示Activity即将停止，是Activity生命周期中的最后一个回调。可以做一些回收工作和最终的资源释放。
![Activity生命周期的切换过程](http://7xq2jk.com1.z0.glb.clouddn.com/Activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%9A%84%E5%88%87%E6%8D%A2%E8%BF%87%E7%A8%8B.jpg)

附加说明：
---
- 针对一个特定的Activity，第一次启动，回调如下:onCreate→onStart→onResume。
- 当用户打开新的Activity或者切换到桌面，回调如下:onPause→onStop。有一种特殊情况是如果新的Activity采用了透明主题，那么当前的Activity不会回调onStop。
当用户再次回到原Activity时，回调如下:onRestart→onStart→onResume。
- 当按Back键回退时，回调如下:onPause→onStop→onDestory。
- 当Activity被系统回收后再次打开，生命周期方法回调过程和正常一样，但只是生命周期方法一样，不代表所有过程都一样。
- 从整个生命周期来说，onCreate和onDestroy是配对的，分别标志着Actovity的创建和销毁，并且只可能有一次调用。从Activity是否可见来说，onStart和onStop是配对的，随着用户的操作或者设备屏幕的点亮和熄灭，这两个方法可能被调用多次；从Activity是否在前台来说，onResume和onPause是配对的，随着用户的操作或者设备屏幕的点亮和熄灭，这两个方法可能被调用多次。

#### 异常情况下的生命周期(指Activity被系统回收或者由于设备的Configuration发生改变从而导致Activitu被销毁重建)
---

- 资源相关的系统配置发生改变导致Activity被杀死并重新创建。
    >默认情况下，如果Activity不做特殊处理，当系统配置发生改变后，Activity就好被销毁并重新创建，其生命周期如下。
    
    ![异常情况下Activity的重建过程](http://7xq2jk.com1.z0.glb.clouddn.com/%E5%BC%82%E5%B8%B8%E6%83%85%E5%86%B5%E4%B8%8BActivity%E7%9A%84%E9%87%8D%E5%BB%BA%E8%BF%87%E7%A8%8B.png)

    onSaveInstanceState方法只会在Activity被异常终止的情况下调用，调用时机是在onStop之前，但和onPause没有既定的时序关系，可能在onPause之前调用，也可能在onPause之后调用。当Activity被重新创建后，系统会调用onRestoreInstanceState,并且把Activity销毁时onSaveInstanceState方法所保存的Bundle对象作为参数同时传递给onRestoreInstanceState和onCreate方法。onRestoreInstanceState方法在onStart方法之后调用。
 
    - 关于保存和恢复View层次结构，系统工作的流程是：
        + 首先Activity被意外终止时，Activity会调用onSaveInstanceState去保存数据
        + 然后Activity会委托Window去保存数据
        + 接着Window在委托它上面的顶级容器去保存数据，顶级容器是一个ViewGroup，一般来说很可能是一个DecorView
        + 最后顶层容器再去一一通知它的子元素来保存数据
    - 接收的位置选择onRestoreInstanceState或onCreate的区别：
        >onRestoreInstanceState一旦被调用，其参数Bundle savedInstanceState一定是有价值的，不需要额外地判断是否为空；但是onCreate不行吗，onCreate如果正常启动的话，其参数Bundle savedInstanceState为null,所以必须额外判断。通时官方建议建议使用onRestoreInstanceState去恢复数据。
    - 针对onSaveInstanceState方法，需要说明的是系统只会在Activity即将被销毁并且有机会重新显示的情况下才会回去调用。
    
- 资源内存不足导致低优先级的Activity被杀死。

1. Activity按照优先级从高到低，分为如下三种：
    - 前台Activity---正在和用户交互的Activity,优先级最高；
    - 可见但非前台Activity---比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户直接交互；
    - 后台Activity---已经被暂停的Activity，比如执行了onStop，优先级最低。
     >当系统内存严重不足时，系统就会按照上述优先级去杀死目标Activity所在的进程，并在后续通过onSaveInstanceState和onRestoreInstanceState来存储和恢复数据。另外一些后台工作不适合脱离四大组件而独自运行在后台中，这样进程容易被杀死，比较好的方法是将后台工作放入Service中从而保证进程有一定的优先级，从而不会轻易地被系统杀死。

2. 解决当系统配置发生改变后，Activity会被重建的问题
如果当某项内容发生改变后，不想系统重建Activity，可以给Activity指定configChanges属性。
比如不想让Activity在屏幕旋转的时候重新创建，就可以给configChanges属性添加orientation值。
````
android:configChanges = "orientation"
````
系统会去调用Activity的onConfigurationChangesd方法。
3.系统配置的项目和含义
![configChanges的项目和含义](http://7xq2jk.com1.z0.glb.clouddn.com/configchanges%E7%9A%84%E9%A1%B9%E7%9B%AE%E5%92%8C%E5%90%AB%E4%B9%89.jpg)

>注意：
screenSize和smallestScreenSize两个比较特殊，它们的行为和编译选项有关，和运行环境无关。

## Activity的启动模式
1. Android四种启动模式：
    - standard: 标准模式，是系统的默认模式，每次启动一个Activity都会重新创建一个新的实例，即使实例已经存在。创建的实例的生命周期符合典型情况下的Activity生命周期。一个任务栈中可以有多个实例，每个实例可以属于不同的任务栈。谁启动了Activity,那这个Activity,就在启动它的那个Activity所在的栈中运行。当用ApplicationContext去启动standard模式的Activity时就会出错，因为standard模式的Activity默认会进入启动它的Activity所属的任务栈中，但Context属于非Activity类型，并没有所谓的任务栈。解决方法是为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，为它在启动的时候创建一个新的任务栈，但这时候实际是以singleTask模式启动。
     >当我们使用ApplicationContext去启动standard模式的Activity的时候会报错，因为非Activity类型的Context并没有所谓的任务栈，解决的方法就是为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就会创建一个新的任务栈，其实这时候待启动的Activity实际上是以singleTask模式启动的。
    - singleTop: 栈顶复用模式。在这模式下，若新的Activity已经位于任务栈的栈顶，那么此Activity就不会被重新创建，同时它的onNewIntent方法会被回调，我们可以通过此方法取出当前请求的信息。另外，该Activity的onCreat和onStart方法不会被系统调用。如果新Activity的实例已存在，但不位于栈顶，仍然会被重建。
    - singleTask: 栈内复用模式。是一种单实例模式，只要Activity在一个栈中存在，即使多次启动Activity都不会重建实例，系统也会回调其onNewIntent方法。另外，singleTask模式的Activity切换到栈顶会导致它之上的栈内的Activity出栈。
    ![singleTask示例](http://7xq2jk.com1.z0.glb.clouddn.com/singleTask.png)
    - singleInstance: 单实例模式。是一种加强的singleTask模式，具有此种模式的Activity只能单独位于一个任务栈中，并且由于栈内复用的特性，后续的请求均不会创建新的Activity，除非任务栈被系统销毁。
2. Activity所需任务栈。
    首先从**TaskAffinity**这个参数说起，这个参数标识了一个Activity所需要的任务栈的名字，默认情况下，所有的Activity都单独指定TaskAffinity属性，这个属性值必须不能和包名相同，否则就相当于没有指定。TaskAffinity属性主要和singletTask启动模式或者allowTaskReparenting属性配对使用。另外任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity位于暂停状态。
    当TaskAffinity和singletTask启动模式配对使用的时候，是具有该模式的Activity的目前任务栈的名字，待启动的Activity会运行在名字和TaskAffinity相同的任务栈中。
    当TaskAffinity和allowTaskReparenting结合的时候，会产生特殊的效果。当一个应用A启动了应用B的某个Activity后，如果这个Activity的allowTaskReparenting属性为true的话，那个当应用B被启动后，此Activity会直接从应用A的任务栈转移到应用B的任务栈中。
3. Activity指定启动模式的方法。
    
    - 第一种是通过AndroidManifest为Activity指定启动模式。
    ````java
        <activity 
            android:name="com.ryg.chapter_1.SecondActivity"
            android:configChanges="screenLayout"
            android:launchMode="singleTask"
            android:label="@string/app_name"/>
    ````
    - 第二种是通过在Intent中设置标志位来为Activity指定启动模式。
    ````java
        Intent intent = new Intent();
        intent.setClass(ManiActivity.this,SecondActivity.class);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
    ````
    区别：
        - 首先，优先级上第二种方式要高于第一种。当两种同时存在时，以第二种方式为准。
        - 其次，两种方式在限定范围上有所不同，第一种方式无法直接为Activity设定FLAG_ACTIVITY_CLEAR_TOP标识，第二种方式无法为Activity指定singleInstance模式。
    >Tips:
        singleTask模式的Activity切换到栈顶会导致在它之上的栈内的Activity出栈。

4. Activity的Flags

    - FLAG_ACTIVITY_NEW_TASK
      >这个标记位的作用是为Activity我都指定“singleTask”启动模式，其效果和在XML中指定该启动模式相同。
    - FLAG_ACTIVITY_SINGLE_TOP
      >这个标记的作用是为Activity指定“singleTop”启动模式，其效果和在XML中指定该启动模式相同。
    - FLAG_ACTIVITY_CLEAR_TOP
      >具有此标记位的Activity,当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。这个模式一般需要和FLAG_ACTIVITY_NEW_TASK配合使用，在这种情况下，被启动的Activity的实例如果存在，那么系统就会调用它的onNewIntent；如果被启动的Activity采用standard模式启动，那么它连同它之上的Activity都要出栈，系统会创建新的Activity实例并放入栈顶，
    - FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
      >具有这个标记的Activity不会出现在历史的Activity的列表中，当某些情况下，我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用，等同于XML中指定Activity的属性android:excludeFromRecents="true"。

## IntentFilter的匹配规则
1. Activity的启动分为两种
    
    - 显示调用
    - 隐式调用
    显示调用需要明确地指定被启动对象的组件信息，包括包名和类名；隐式调用则不需要明确地指定组件信息。原则上一个Intent不应该即时显示调用又是隐式调用，如果二者共存的话以显示调用为主。
    IntentFilter中的过滤信息有action,category,data。
    下面是一个过滤规则：
    ````java
    <activity
    android:name="com.ryg.chapter_1.ThirdActivity"
    android:configChanges="screenLayout"
    android:label="@string/app_name"
    android:launchMode="singleTask"
    android:taskAffinity="com.ryg.task1" >
        <intent-filter>
            <action android:name="com.ryg.charpter_1.c" />
            <action android:name="com.ryg.charpter_1.d" />
            <category android:name="com.ryg.category.c" />
            <category android:name="com.ryg.category.d" />
            <category android:name="android.intent.category.DEFAULT" />
            <data android:mimeType="text/plain" />
            </intent-filter>
    </activity>
    ````

2. 只有一个Intent同时匹配action类别，category类别，data类别才算是完全匹配，只有完全匹配才能成功启动目标Activity。另外，一个Activity中可以有多个intent-filter，一个Intent只要能匹配任何一组intent-filter即可成功启动对应的Activity。
3. 匹配规则。
    
    - action匹配规则：
        只要Intent中的action能够和过滤规则中的任何一个action相同即可匹配成功，action匹配区分大小写。
        一个过滤规则中可以有多个action,那么只要Intent中的action能够和过滤规则中的任何一个action相同即可匹配成功。
        action的匹配要求Intent中的action存在且必须和过滤规则中的其中一个action相同。
    - category匹配规则：
        Intent中如果有category那么所有的category都必须和过滤规则中的其中一个category相同，如果没有category的话那么就是默认的category，即android.intent.category.DEFAULT，所以为了Activity能够接收隐式调用，配置多个category的时候必须加上默认的category。
        也就是说，Intent中出现了category，不管有几个category，对于每个category来说，它必须是过滤规则中已经定义了的category。当然，Intent也可以没有actegory，因为系统在用startActivity或者startActivityForResult的时候会默认为Intent加上"android.intent.category.DEFAULT"这个category。
    - data匹配规则：
        Intent中必须含有data数据，并且data数据能够完全匹配过滤规则中的某一个data。
        data的结构很复杂，语法大致如下：
        ````java
        <data   android:scheme="string"
                android:host="string"
                android:port="string"
                android:path="string"
                android:pathPattern="string"
                android:pathPrefix="string"
                android:mimeType="string" />

        ````
        主要由mimeType和URI组成，其中mimeType代表媒体类型，如image/jpeg、audio/mpeg4-generic和video、*等，可以表示图片、文本、视频等不同的媒体格式；而URI的结构也复杂，大致如下：
        ````
            <scheme>://<host>:<port>/[<path>]|[<pathPrefix>]|[pathPattern]
        ````
        例如：
        ````
            content://com.example.project:200/folder/subfolder/etc
            http://www.baidu.com:80/search/info
        ````
        scheme:URI的模式，比如http、file、content等。
        host:URI的主机名，比如www.baidu.com.
        port:URI的端口号，比如80.
        其中如果scheme或者host未指定那么URI就无效。
        path、pathPattern、pathPrefix都是表示路径信息，其中path表示完整的路径信息，pathPrefix表示路径的前缀信息；pathPattern表示完整的路径，但是它里面包含了通配符"*",表示0个或者多个任意字符。
        **data匹配规则的几种情况：**
        1.
            ````java
            //URI有默认值
            <intent-filter>
                <data android:mimeType="image/*"/>
                ...
            </intent-filter>
            ````
        如果过滤规则中的mimeType指定为"image/*"或者"text/*"等这种类型的话，那么即使过滤规则中没有指定URI，URI有默认的scheme是content和file！如果过滤规则中指定了scheme的话那就不是默认的scheme了。
        例：
        ````
        intent.setDataAndType(Uri.parse("file://abc"),"image/png");
        ````
        如果要为Intent指定完整的data，必须要调用setDataAndType方法！
        不能先调用setData然后调用setType，因为这两个方法会彼此清除对方的值。
        ````java
            public Intent setData(Uri data){
                mData= data;
                mType = null;
                return this;
            }    
        ````
        可以发现，setData会把mimeType置为null,setType也会把URI置为null。
        2.
        ````java
            <intent-filter>
                <data android:mimeType="video/mpeg" android:scheme="http/..."/>
                <data android:mimeType="audio/mpeg" android:scheme="http/..."/>
            </intent-filter>
        ````
        这种规则指定了两组data规则，且每个data都指定了完整的属性值，既有URI又有mimeType。
        例：
        ````java
            intnet.setDataAndType(Uri.parse("http://abc"),"video/mpeg");
        ````
        或
        ````java
            intnet.setDataAndType(Uri.parse("http://abc"),"audio/mpeg");
        ````
        另外，data的下面两种写法作用是一样的：
        ````java
        <intent-filter>
            <data android:scheme="file" android:host="www.github.com"/>
        </intent-filter>
        <intent-filter>
            <data android:scheme="file"/>
            <data android:host="www.github.com"/>
        </intent-filter>
        ````
        Intent-filter的匹配规则对于Service和BroadcastReceiver也同样道理，不过系统对于Service的建议是尽量使用显示调用方式来启动服务。
        **如何判断是否有Activity能够匹配我们的隐式Intent？**
          - PackageManager的resolveActivity方法或者Intent的resolveActivity方法：如果找不到就会返回null
          - PackageManager的queryIntentActivities方法：它返回所有成功匹配的Activity信息。
          针对Service和BroadcastReceiver等组件，PackageManager同样提供了类似的方法去获取成功匹配的组件信息，例如queryIntentServices、queryBroadcastReceivers等方法。有一类action和category比较重要，它们在一起用来标明这是一个入口Activity，并且会出现在系统的应用列表中。
          在action和category中，有一类action和category比较重要。
          ````
            <action android:name="android.intent.action.MAIN"/>
            <action android:name="android.intent.category.LAUNCHER"/>
          ````
          这二者共同作用是用来标明这是一个入口Activity并且会出现在系统的应用列表中，缺一不可。

                                                                    2017/8/13
                                                                    W.Z.H