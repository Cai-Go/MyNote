
- Nulls
这是一个很大的变化，RxJava 1.x 是允许我们在发射事件的时候传入 null 值的，但现在我们的 2.x 不支持了，这意味着 Observable<Void> 不再发射任何值，而是正常结束或者抛出空指针。
- Flowable
在 RxJava 1.x 中关于介绍 backpressure 部分有一个小小的遗憾，那就是没有用一个单独的类，而是使用 Observable 。而在 2.x 中 Observable 不支持背压了，将用一个全新的 Flowable 来支持背压。
或许对于背压，有些小伙伴们还不是特别理解，这里简单说一下。大概就是指在异步场景中，被观察者发送事件的速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略。感兴趣的小伙伴可以模拟这种情况，在差距太大的时候，我们的内存会猛增，直到OOM。而我们的 Flowable 一定意义上可以解决这样的问题，但其实并不能完全解决，这个后面可能会提到。
- Single/Completable/Maybe
其实这三者都差不多，Single 顾名思义，只能发送一个事件，和 Observable接受可变参数完全不同。而 Completable 侧重于观察结果，而 Maybe 是上面两种的结合体。也就是说，当你只想要某个事件的结果（true or false）的时候，你可以使用这种观察者模式。
- 线程调度相关
这一块基本没什么改动，但 RxJava 2.x 中已经没有了 Schedulers.immediate() 这个线程环境，还有 Schedulers.test()。
- Function相关
熟悉 1.x 的小伙伴一定都知道，我们在1.x 中是有 Func1，Func2.....FuncN的，但 2.x 中将它们移除，而采用 Function 替换了 Func1，采用 BiFunction 替换了 Func 2.....FuncN。并且，它们都增加了 throws Exception，也就是说，妈妈再也不用担心我们做某些操作还需要 try-catch 了。
- 其他操作符相关
如 Func1...N 的变化，现在同样用 Consumer 和 BiConsumer 对 Action1 和 Action2 进行了替换。后面的 Action 都被替换了，只保留了 ActionN。

![](https://upload-images.jianshu.io/upload_images/3994917-14f7e368b8e0596b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-b447bbabccff5506.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-9863b5f713ac86d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-28ce3ee8a0ccdcf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-548c743caff7c3e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-f20109ef808f04dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-93f5aee82d8fc8fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
![](https://upload-images.jianshu.io/upload_images/3994917-4e8d8b566c245606.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

