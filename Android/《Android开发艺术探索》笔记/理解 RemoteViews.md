# 理解 RemoteViews
RemoteViews 表示的是一个 View 结构，可以在其他进程中显示，RemoteViews 提供了一组基础的操作用于跨进程更新它的界面。RemoteViews 在 Android 中的使用场景有两种：通知栏和桌面小部件。

## RemoteViews 的应用
1. AppWidgetProvider 是 Android 中提供的用于实现桌面小部件的类，其本质是一个广播，即 BroadcastReceiver。
    
    - onEnable：当该窗口小部件第一次添加到桌面时调用该方法，可添加多次但只在带一次调用。
    - onUpdate：小部件被添加时或者每次小部件更新时都会调用一次该方法，小部件的更新时机由 updatePeriodMillis 来指定，每个周期小部件都会自动更新一次。
    - onDeleted：每删除一次桌面小部件就调用一次。
    - onDisabled：当最后一个该类型的桌面小部件被删除时调用该方法，注意是最后一个。
    - onReceive：这是广播的内置方法，用于分发具体的事件给其他方法。会根据不同的 Action 来分别调用 onEnable、onDisabled 和 onUpdate 等方法。

2. PendingIntent
    PendingIntent 表示一种处于 pending 状态的意图，而 pending 状态表示的是一种待定、等待、即将发生的意思。和 Intent 的区别在于：PendingIntent 是在将来的某个待定的时刻发生，而 Intent 是立刻发生。PendingIntent 通过 send 和 cancel 方法来发送和取消特定的待定 Intent。

    PendingIntent 支持三种待定意图：启动 Activity、启动 Service 和发送广播
    **PendingIntent的主要方法**
    ![PendingIntent的主要方法](http://o8fk8z4sl.bkt.clouddn.com/PendingIntent%E7%9A%84%E4%B8%BB%E8%A6%81%E6%96%B9%E6%B3%95.png)
    requestCode 表示 PendingIntent 发送方的请求码，多数情况下设为0即可，也会影响到 flags 的效果。flags 常见类型有：
       
    - FLAG_ONE_SHOT
        当前描述的 PendingIntent 只能使用一次，然后它就会自动 cancel，如果后续还有相同的 PendingIntent，那么它们的 send 方法就会调用失败。对于通知栏消息来说，如果采用此标记位，那么同类的通知只能使用一次，后续的通知单击后将无法打开。
    - FLAG_NO_CREATE
        当前描述的 PendingIntent 不会主动创建，如果当前 PendingIntent 之前不存在，那么 getActivity、getService 和 getBroadcast 方法就会直接返回 null。这个标记位无法单独使用。
    - FLAG_CANCEL_CURRENT
        当前描述的 PendingIntent 如果已经存在，那么它都会被 cancel，然后系统会创建一个新的 PendingIntent。对于消息通知栏来说，那些被 cancel 的消息单击后将无法打开。
    - FLAG_UPDATE_CURRENT
        当前描述的 PendingIntent 如果已经存在，那么它们都会被更新，即它们的 Intent 中的 Extras 会被替换成新的。

    PendingIntent 的匹配规则：如果两个 PendingIntent 它们内部的 Intent 相同并且 requestCode 也相同，那么这两个 PendingIntent 就是相同的。
    Intent 的匹配规则：如果两个 Intent 的 ComponentName 和 intent-filter 都相同，那么这两个 Intent 就是相同的。

## RemoteViews 的内部机制
1. RemoteViews 支持的类型：

    - Layout
        FrameLayout、LinearLayout、RelativeLayout、GridLayout
    - View
        AnalogClock、Button、Chronometer、ImageButton、ProgressBar、TextView、ViewFlipper、ListView、GridView、StackView、AdapterViewFlipper、ViewStub

2. RemoteViews 的部分 set 方法
![RemoteViews的部分set方法](http://o7qv8ih35.bkt.clouddn.com/RemoteViews%E7%9A%84%E9%83%A8%E5%88%86set%E6%96%B9%E6%B3%95.png)

3. RemoteViews 的内部机制
![RemoteViews的内部机制](http://o8fk8z4sl.bkt.clouddn.com/RemoteViews%E7%9A%84%E5%86%85%E9%83%A8%E6%9C%BA%E5%88%B6.png)
