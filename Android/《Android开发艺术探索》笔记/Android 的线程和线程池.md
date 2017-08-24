# Android 的线程和线程池
从用途上分，线程分为主线程和子线程；主线程主要处理和界面相关的事情，子线程则往往用于耗时操作。

## 主线程和子线程
主线程是指进程所拥有的线程。Android 中主线程交 UI 线程，主要作用是运行四大组件以及处理它们和用户的交互；子线程的作业则是执行耗时任务。

## Android 中的线程形态
1. AsyncTask
    AsyncTask 是一种轻量级的异步任务类，可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新 UI，
    AsyncTask 是一个抽象的泛型类，提供了 Params(参数的类型)、Progress(后台任务执行进度的类型) 和 Result(后台任务的返回结果的类型) 这三个泛型参数，
    AsyncTask 提供了4个核心方法
    - onPreExcute()，在主线程中执行，在异步任务执行之前，此方法会被调用，一般可以用于做一些准备工作。
    - doInBackground(Params...params)，在线程池中执行，此方法用于执行异步任务，params 参数表示异步任务的输入参数。在此方法中可以通过 publishProgress 方法来更新任务的进度，publishProgress 方法会调用 onProgressUpdate 方法，另外此方法需要返回计算结果给 onPostExecute 方法。
    - onProgressUpdate(Progress...values)，在主线程中执行，当后台任务的执行进度发生改变时此方法会被调用。
    - onPostExecute(Resukt result)，在主线程中执行，在异步任务执行之后，此方法会被调用，其中 result 参数是后台任务的返回值，即 doInBackground 的返回值。
    
    onPreExcute 先执行，接着是 doInBackground，最后才是 onPostExecute。
    当异步任务被取消时，onCancelled() 方法会被调用，这个时候 onPostExecute 则不会被调用。
2. AsyncTask 在具体的使用过程中的一些限制条件

    - AsyncTask 的类必须在主线程中加载；
    - AsyncTask 的对象必须在 UI 线程中创建；
    - 不要在程序中直接调用 onPreExecute、onPostExecute、doInBackground 和 onProgressUpdate 方法。
    - 一个 AsyncTask 对象只能执行一次，即只能调用一次 execute 方法，否则会报运行时异常。
    - 在 Android 1.6之前，AsyncTask 是串行执行任务的，Android 1.6的时候 AsyncTask 开始采用线程池处理并行任务，但是从 Android 3.0开始为了避免 AsyncTask 所带来的并发错误，AsyncTask 又采用一个线程来串行执行任务。但是在 Android 3.0 以及后续的版本中，仍然可以通过 AsyncTask 的 executeOnExecutor 方法来并行地执行任务。

3. AsyncTask 的工作原理
    AsyncTask 中有两个线程池(SerialExecutor 和 THREAD_POOL_EXECUTOR) 和一个 Handler(InternalHandler)，线程池 SerialExecutor 用于任务的排队，线程池 THREAD_POOL_EXECUTOR 用于真正地执行任务，InternalHandler 用于将执行环境从线程池切换到主线程。

4. HandlerThread
    HandlerThread 继承了 Thread，是一种可以使用 Handler 的 Thread, 它的实现就是在 run 方法中通过 Looper.prepare() 来创建消息队列，并通过 Looper.loop() 来开启消息循环。

    与普通的 Thread 相比，普通 Thread 主要用于在 run 方法中执行一个耗时任务，而 HandlerThread 在内部创建了消息队列，外界需要通过 Handler 的消息方式来通知 HandlerThread 执行一个具体的任务。

    由于 HandlerThread 的 run 方法是一个无限循环，因此当明确不需要在使用 HandlerThread 时，可以通过它的 quit 或者 quitSafely 方法来终止线程的执行。

5. IntentService
    IntentService 是一种特殊的 Service，继承了 Service 并且是一个抽象类，必须创建它的子类才能使用 IntentService。IntentService可用于执行后台耗时任务，任务执行后会自动停止，并且它的优先级比单纯的线程要高很多，不容易被系统杀死。在实现上，IntentService 封装了 HandlerThread 和 Handler。

## Android 中的线程池
1. 线程池的优点
    - 重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销；
    - 能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象；
    - 能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能。
    
2. ThreadPoolExecutor
    ThreadPoolExecutor 是线程的真正实现。
    ````java
        public ThreadPoolExecutor(int corePoolSize,
                                int maximumPoolSize,
                                long keepAliveTime,
                                TimeUnit unit,
                                BlockingQueue<Runnable> workQueue,
                                ThreadFactory threadFactory)
    ````

    - corePoolSize
        线程池的核心线程数，默认情况下，核心线程会在线程池中一直存活，及时处于闲置状态。
    - maximumPoolSize
        线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务将会被阻塞。
    - keepAliveTime
        非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。
    - unit
        用于指定 keepAliveTime 参数的时间单位，这是一个枚举，常用的有 TimeUnit、MILLSECONDS(毫秒)、TimeUnit.SECONDS(秒) 以及 TimeUnit.MINUTES(分钟)。
    - workQueue
        线程池中的任务队列，通过线程池的 execute 方法提交的 Runnable 对象会存储在这个参数中。
    - threadFactory
        线程工厂，为线程池通过创建新线程的功能。ThreadFactory 是一个接口，只有 一个方法：Thread newThread(Runnable r)。

    ThreadPoolExecutor 执行任务时遵循的规则
    
    - 如果线程池中的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务；
    - 如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行；
    - 如果在步骤2中无法将任务插入到任务队列中，这往往是由于任务队列已满，这个时候如果线程数量为达到线程池规定的最大值，那么会立刻启动一个非核心线程来执行任务。
    - 如果步骤3中线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务， ThreadPoolExecutor 会调用 RejectedExecutionHandler 的 rejectedExecution 方法来通知调用者。
    
3. 线程池的分类
    - FixedThreadPool
        通过 Executors 的 newFixedThreadPool 方法来创建。是一种线程数量固定的线程池，当线程处于空闲状态时并不会被回收，除非线程池被关闭。FixedThreadPool 中只有核心线程并且这些核心线程没有超时机制，另外任务队列也是没有大小限制。
    - CachedThreadPool
        通过 Executors 的 newCachedThreadPool 方法来创建。是一种线程数量不定的线程池，只有非核心线程，最大线程数为 Integer.MAX_VALUE。线程池中的空闲线程都有超时机制，这个超时时长为60秒，超过60秒闲置线程就会被回收。CachedThreadPool 的任务队列相当于一个空集合，这样会导致任何任务都会被立即执行。
    - ScheduledThreadPool
        通过 Executors 的 newScheduledThreadPool 方法来创建。它的核心线程数量是固定的，而非核心线程数是没有限制的，并且当非核心线程闲置时会被立即回收。ScheduledThreadPool 这类线程池主要用于执行定时任务和具有固定周期的重复任务。
    - SingleThreadExecutor
        通过 Executors 的 newSingleThreadExecutor 方法来创建。这类线程池内部只有一个核心线程，它确保所有的任务都在同一个线程中按顺序执行。SingleThreadExecutor 的意义在于统一所有的外界任务到一个线程中，这使得在这些任务之间不需要处理线程同步的问题。


    系统预置4种线程池的典型使用方法：

    ````java
        Runnable command = new Runnable(){
            @Override
            public void run(){
                SystemClock.sleep(2000);
            }

        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(4);
        fixedThreadPool.execute(command);

        ExecutorService cachedThreadPool =Executors.newCachedThreadPool();
        cachedThreadPool.execute(command);

        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(4);
        // 2000ms 后执行 command
        scheduledThreadPool.schedule(command,2000,TimeUnit.MILLISECONDS);
        // 延迟10ms，每个1000ms执行一次 command
        scheduledThreadPool.scheduleAtFixedRate(command,10,1000,TimeUnit.MILLISECONDS);

        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        singleThreadExecutor.execute(command);

        }
    ````                                                                           


                                                                     2017.8.24
                                                                        W.Z.H
