# Android 的消息机制
1. Android 的消息机制主要是指 Hadnler，Hnadler 的运行需要底层的 MessageQueue 和 Lopper 支撑。MessageQueue 是消息队列，它的内部存储了一组信息，以队列的形式对外提供插入和删除的工作；Looper 是指消息循环，会以无限循环的形式去查找是否有新消息，如果有的话就处理，否则一直等待。ThreadLocal 是 Looper 中的一个特殊概念，作用是可以在每个线程中存储数据。ThreadLocal 可以在不同的线程中互不干扰地存储并提供数据，通过 ThreadLocal 可以获取每个线程的 Looper，另外线程是默认没有 Looper 的，需要使用 Handler 就必须为线程创建 Looper。

## Android 的消息机制概述
1. 系统不允许在子线程中访问 UI 的原因：因为 Android 的 UI 控件不是线程安全的，如果在多线程中并发访问可能会导致 UI 控件处于不可预期的状态。如果对 UI 控件的访问加上锁机制会有两个缺点：首先加上锁机制会让 UI 访问的逻辑变得复杂；其次锁机制会降低 UI 访问的效率，因为锁机制会阻塞某些线程的执行。
2. Handler 的工作过程
    ![Handler 的工作过程](http://o8fk8z4sl.bkt.clouddn.com/Handler%20%E7%9A%84%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B.png)

## Android 的消息机制分析
1. ThreadLocal 的工作原理
    ThreadLocal 是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储后，只有在指定线程中可以获取到存储的数据，其他线程则无法获取到数据。
    当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用 ThreadLocal。

    存储规则：ThreadLocal 的值在 table 数组中的存储位置总是为 ThreadLocal 的 reference 字段所标识的对象的下一个位置。

    ThreadLocal 可以在多个线程中互不干扰地存储和修改数据，是因为 ThreadLocal 的 set 和 get 方法所操作的对象都是当前线程的 localValues 对象的 table 数组，可以在不同线程中访问同一个 ThreadLocal 的 set 和 get 方法，它们对 ThreadLocal 所做的读/写操作仅限于各自线程发内部。

2. 消息队列的工作原理
    消息队列在 Android 中指的是　MessageQueue，MessageQueue主要包含插入和读取两个操作，对应 enqueueMessage(作用是往消息队列中插入一条消息) 和 next(作用是从消息队列中取出一条消息并将其从消息队列中移除) 两个方法。

3. Looper 的工作原理
    Looper.prepare() 为当前线程创建一个 Looper，接着通过 Looper.loop() 来开启消息循环。
    prepareMainLooper 方法主要是给主线程 ActivityThread 创建 Looper 使用的。
    Looper 提供的 getMainLooper 方法可以在任何地方获取到主线程的 looper。
    Looper 提供了 quit 和 quitSafely 来退出 Looper，区别是：quit 会直接退出 Looper；quitSafely 只是设定了一个退出标记，然后把消息队列中的已有消息处理完毕后才安全地退出。
    在子线程中，如果收到为其创建了Looper，那么所有的事情完成以后应该调用 quit 方法来终止消息循环，否则这个子线程就会一直处于等待状态。
    Looper 最重要的一个方法是 loop 方法，只有调用了 loop 方法，消息循环系统才会真正地起作用。loop 是一个死循环，唯一跳出循环的方式是 MessageQueue 的 next 方法返回 null。

4. Handler 的工作原理
    Handler 的工作主要包含消息的发生和接收过程，消息的发送可以通过 post 的一系列方法以及 send 的一系列方法来实现，post 的一系列方法最终是通过 send 的一系列方法来实现的。
    Handler 发生消息的过程仅仅是向消息队列中插入一条消息，MessageQueue 的 next 方法就会返回这条消息给 Looper，Looper 收到消息后就开始处理，最终消息由 Looper 交由 Hnadler 处理，即 Handler 的 dispatchMessage 方法会被调用，这时 Hnadler 就进入了处理消息的阶段。

    Hnadler 出来消息的过程：

    - 首先，检查 Message 的 callback 是否为 null，不为 null 就通过 handleCallback 来处理消息。callback 是一个 Runnable 对象，实际上就是 Handler 的 post 方法所传递的 Runnable 参数。
    - 其次，检查 mCallback 是否为 null，不为 null 就调用 mCallback 的 handleMessage 方法来处理消息。Callback 是个接口，其意义是可以用来创建一个 Handler 的实例但不需要派生 Handler 的子类。
    - 最后，调用 Handler 的 handleMessage 方法来处理消息。
         Handler 处理消息流程图
        ![Handler 处理消息流程图](http://o8fk8z4sl.bkt.clouddn.com/Handler%20%E5%A4%84%E7%90%86%E6%B6%88%E6%81%AF%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

## 主线程的消息循环
1. Android 的主线程就是 ActivityThread，主线程的入口方法为 main，在 main 方法中系统会通过 Looper.prepareMainLooper() 来创建主线程的 Looper 以及 MessageQueue，并通过 Looper.loop() 来开启主线程的消息循环。
    主线程的消息循环开始后，ActivityThread 还需要一个 Handler 来和消息队列进行交互，这个 Handler 即 ActivityThread.H，它内部定义了一组消息类型，包含了四大组件的启动和停止等过程。

    ActivityThread 通过 ApplicationThread 和 AMS 进行进程间通信， AMS 以进程间通信的方式完成 ActivityThread 的请求后会回调 ActivityThread 中的 Binder 方法，然后 ApplicationThread 会向 H 发送消息，H 收到消息后会将 ApplicationThread 中的逻辑切换到 ActivityThread 中去执行，即切换到主线程中去执行，这个过程就是主线程的消息循环模型。




                                                                    2017.8.24
                                                                      W.Z.H