# 理解 Window 和 WindowManager
Window 是一个抽象类，它的具体实现是 PhoneWindow，创建一个 Window 只需要通过 WindowManager 即可完成，WindowManager 是外界访问 Window 的人口，Window 的具体实现位于 WindowManagerService 中，WindowManager 和 WindowManagerService 的交互是一个 IPC 过程，Android 中所有的视图都是通过 Window 来呈现的，Window 实际是 View 的直接管理者。

## Window 和 WindowManager
1. WindowManager.LayoutParams 中的 flags 和 type 参数。
    Flags 参数表示 Window 的属性，
    - FLAG_NOT_FOCUSABLE：表示 Window 不需要获取焦点，也不需要接受各种输入事件，同时会开启 FLAG_NOT_TOUVH_MODAL，最终事件会直接传递给下层的具有焦点的 Window。
    - FLAG_NOT_TOUVH_MODAL：在此模式下，系统会将当前 Window 区域以外的单击事件传递给底层的 Window，当前 Window 区域以内的单击事件则自己处理。
    - FLAG_SHOW_WHEN_LOCKED：开启此模式可以让 Window 显示在锁屏的界面上。
    
    Type 参数表示 Window 的类型。Window有三种类型：

    - 应用 Window：对应着一个 Activity；
    - 子 Window：不能单独存在，需要附属在特定的父 Window 之中；
    - 系统 Window：是需要声明权限才能创建的 Window。
 
    Window 是分层的，每个 Window 都有对应的 z-ordered，层级大的会覆盖在层级小的 Window 上面，应用 Window 的层级范围是1-999，子 Window 的层级范围是 1000-1999，系统 Window 的层级范围是 2000-2999。这些层级范围对应着 WindowManager.LayoutParams 的 type 参数。

    WindowManager 提供的三个常用方法：添加 View，更新 View，删除 View。

## Window 的内部机制 
1. Window 是一个抽象的概念，每一个 Window 都对应着一个 View 和一个 ViewRootImpl，Window 和 View 通过 ViewRootImpl 来建立联系，因此 Window 并不是实际存在的，是以 View 的形式存在，View 才是 Window 存在的实体。

2. Window 的添加过程
    Window 的添加过程需要通过 WindowManager 的 addView 来实现。
    - 检查参数是否合法，如果是子 Window 那么还需要调整一些布局参数；
    - 创建 ViewRootImpl 并将 View 添加到列表中；
    - 通过 ViewRootImpl 来更新界面并完成 Window 的添加过程。

3. Window 的删除过程
    先通过 WindowManagerImpl，再通过 WindowManagerGlobal 来实现。
    removeView 的逻辑是首先通过 findViewLocked 来查找待删除的 View 的索引，这个查找过程就是建立的数组遍历，然后再调用 removeViewLocked 来做进一步的删除。
    在 WindowManager 中提供了两种删除接口 removeView 和 removeViewImmediate，分别表示异步删除和同步删除。
    异步删除操作：由 ViewRootImpl 的 die 方法完成，在异步删除的情况下，die 方法只是发生了一个请求删除的消息后就立刻返回，这个时候 View 并没有完成删除操作，所以最后会将其添加到 mDyingViews 中，mDyingViews 表示待删除的 View 列表。
    dispatchDetachedFromWindow 方法主要做四件事：
    - 垃圾回收相关的工作，比如清除数据和消息、移除回调；
    - 通过 Session 的 remove 方法删除 Window：mWindowSession.remove(mWindow)，这同样是一个 IPC 过程，最终会调用 WindowManagerService 的 removeWindow 方法。
    - 调用 View 的 dispatchDetachedFromWindow 方法，在内部会调用 View 的 onDetachedFromWindow() 以及 onDetachedFromWindowInternal()。
    - 调用 WindowManagerGlobal 的 doRemoveView 方法刷新数据，包括 mRoots、mParams 以及 mDyingViews，需要将当前 Window 所关联的这三类对象从列表中删除。

4. Window 的更新过程
    updateViewLayout 方法：
    首先需要更新 View 的 LayoutParams 并替换掉老的 LayoutParams，接着再更新 ViewRootImpl 中的 LayoutParams，这是通过 ViewRootImpl 的 setLayoutParams 方法来实现的。在 ViewRootImpl 中会通过 scheduleTraversals 方法来对 View 重新布局，包括测量、布局、重绘这三个过程。ViewRootImpl 还会通过 WindowSession 来更新 Window 的视图，这个过程最终室友 WindowManagerService 的 relayoutWindow() 来具体实现，同样也是一个 IPC。

## Window 的创建过程
1. Activity 的 Window 的创建过程
    PhoneWindow 的 setContentView 方法大致遵循如下几个步骤
    - 如果没有 DecorView，那么就创建它；
        DecorView 的创建过程由 installDecor 方法来完成，在方法内部会通过 generateDecor 方法来直接创建 DecorView。为了初始化 DecorView 的结构，PhoneWindow 还需要通过 generateLayout 方法来加载具体的布局文件到 DecorView 中。
    - 将 View 添加到 DecorView 的 mContentParent 中；
        mLayoutInflater.inflate(layoutResID,mContentParent)；
    - 回调 Activity 的 onContentChanged 方法通知 Activity 视图已经发生改变。
    
    在 ActivityThread 的 handleResumeActivity 方法中，首先会调用 Activity 的 onResume 方法，接着会调用 Activity 的 makeVisible()，这时候 DecorView 才真正地完成了添加和显示这两个过程，Activity 才能被用户看到。

2. Dialog 的 Window 的创建过程
    Dialog 的 Window 的创建过程和 Activity 类似，
    - 创建 Window
    - 初始化 DecorView 并将 Dialog 的视图添加到 DecorView 中
    - 将 DecorView 添加到 Window 中并显示
    
    当 Dialog 被关闭时，它会通过 WindowManager 来移除 DecorView：mWindowManager.removeViewImmediate(mDecor)。

3. Toast 的 Window 的创建过程
    在 Toast 内部有两个 IPC 过程：第一类是 Toast 访问 NotificationManagerService，第二类是 NotificationManagerService 回调 Toast 里的 TN 接口。
    Toast 的显示过程：它调用了 NMS 中的 enqueueToast 方法，enqueueToast 方法第一个参数表示当前应用的包名，第二个参数 tn 表示远程回调，第三个参数表示 Toast 的时长，enqueueToast 首先将 Toast 请求封装为 ToastRecord 对象并将其添加到一个名为 mToastQueue 的队列中，mToastQueue 其实是一个 ArrayList，mToastQueue 中最多能同时存在50个 ToastRecord，这样做是为了放置 DOS；当 ToastRecord 被添加到 mToastQueue 中后，NMS 就好通过 showNextToastLocked 方法来显示当前的 Toast，Toast 的显示是由 ToastRecord 的 callback 来完成的，这个 callback 实际上就是 Toast 中的 TN 对象的远程 Binder；Toast 显示以后， NMS 还会通过 scheduleTimeoutLocked 方法来发送一个延时消息。

    Toast 的隐藏也是通过 ToastRecord 的 callback 来完成的，也是一次 IPC 过程。

    Toast 的显示和隐藏过程实际上是通过 Toast 中的 TN 这个类来实现的，有 show 和 hide 两个方法，分别对应 Toast 的显示和隐藏，因为这两个方法是被 NMS 以跨进程的方式调用的，运行在 Binder 线程池中，内部使用了 Handler 来将执行环境切换到 Toast 请求所在的线程，两个方法中的 mShow 和 mHide 是两个 Runnable，内部分别调用了 handleShow 和 handleHide 方法，handleShow 和 handleHide 才是真正完成显示和隐藏 Toast 的地方，TN的 handleShow 中会将 Toast 的视图添加到 Window 中，handleHide 中会将 Toast 的视图从 Window 中移除。



                                                                     2017.8.22
                                                                       W.Z.H


