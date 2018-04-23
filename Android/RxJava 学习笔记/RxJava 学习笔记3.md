# RxJava 学习笔记3

## Scheduler(调度器)
如果想给 Observable 操作符链添加多线程功能，可以指定操作符(或者特定的 Observable) 在特定的调度器(Scheduler) 上执行。

使用 ObserveOn 和 SubscribeOn 操作符，你可以让 Observable 在一个特定的调度器上执行，ObserveOn 指示一个 Observable 在一个特定的调度器上调用观察者的 onNext，onError 和 onCompleted 方法，SubscribeOn 更进一步，它指示 Observable 将全部的处理过程(包括发射数据和通知)放在特定的调度器上执行。

### 调度器的种类
|调度器类型|效果|
|:-------- |:--|
|Schedulers.computation()|用于计算任务，如事件循环或回调处理，不要用于 IO 操作(IO 操作请使用 Schedulers.io())；默认线程数等于处理器的数量|
|Schedulers.from(executor)|使用指定的 Executor 作为调度器|
|Schedulers.immediate()|在当前线程立即开始执行任务|
|Schedulers.io()|用于 IO 密集型任务，如异步阻塞 IO 操作，这个调度器的线程池会根据需要增长；对于普通的计算任务，请使用 Schedulers.computation()；Schedulers.io() 默认是一个 CachedThreadScheduler，很像一个有线程缓存的新线程调度器|
|Schedulers.newThread()|为每个任务创建一个新线程|
|Schedulers.trampoline()|当其它排队的任务完成后，在当前线程排队开始执行|