## RxJava 学习笔记2
Observable : 被观察者。
Observer : 观察者；有时也叫做 Subscriber、Watcher、Reactor。

## 异步模型中的流程
1. 定义一个方法，它完成某些任务，然后从异步调用中返回一个值，这个方法是观察者的一部份
2. 将这个异步调用本身定义为一个 Observable
3. 观察者通过订阅（Subscribe）操作关联到那个 Observable
4. 继续你的业务逻辑，等方法返回时，Observable 会发射结果，观察者的方法会开始处理结果或结果集

````
def myOnNext = {it -> do something useful with it};

def myObservable = someObservable(itsParameters);

myObservable.subscribe(myOnNext);
````

## 回调方法（onNext，onCompleted,onError）
Subscribe 方法用于将观察者连接到 Observable,你的观察者需要实现以下方法的一个子集：

- onNext(T item)
  Observable 调用这个方法发射数据，方法的参数就是 Observable 发射的数据，这个方法可能会被调用多次。
- onError(Exception ex)
  当 Observable 遇到错误或者无法返回期望的数据时会调用这个方法，这个调用会终止 Observable，后续不会再调用 onNext 和 onCompleted，onError 方法的参数是抛出的异常。
- onComplete
  正常终止，如果没有遇到错误，Observable 在最后哦一次调用 onNext 之后调用此方法。
根据 Observable 协议的定义，onNext 可能会被调用零次或者很多次，最后会有一次 onCompleted 或者 onError 调用(不会同时),传递数据给 onNext 通常被称作发射，onCompleted 和 onError 被称作通知。
````
def myOnNext = {item -> /* do something useful with item */};

def myError = {throwable -> /*react sensibly to a failed call*/};

def myComplete = {/* clean up after the final response */};

def myObserable = someMethod(itsParameters);

myObservable.subscribe(myOnNext,myError,myComplete);
````

## 取消订阅（Unsubscribing）
取消订阅的结果会传递这个 Observable 的操作符链，而且会导致这个链条上的每个环节都停止发射数据项。这些并不保证会立即发生，然而，对一个 Observable 来说，及时没有观察者了，它也可以在一个 while 循环中继续生产并尝试发射数据项。

## 操作符
1. 创建操作：Create , Defer , Empty/Never/Throw , From , Interval , Just , Range , Repeat , Start , Timer
2. 变换操作: Buffer , FlatMap , GroupBy , Map , Scan , Window
3. 过滤操作: Debounce , Distinct , ElementAt , Filter , IgnoreElements , Last , Sample , Skip , SkipLast , Take , TakeLast
4. 组合操作: And/Then/When , CombineLatest , Join , Merge , StartWith , Switch , Zip
5. 错误处理: Catch , Retry
6. 辅助操作: Delay , Do , Materialize/Dematerialize , ObserbeOn , Serialize , Subscribe , SubscribeOn , TimeInterval , Timeout , Timestamp , Using
7. 条件和布尔操作: All , Amb , Contains , DefaultlfEmpty , SequeneceEqual , SkipUntil , SkipWhile , TakeUntil , TakeWhile
8. 算术和集合操作: Average , Concat , Count , Max , Min , Reduce , Sum
9. 转换操作: To
10. 连接操作: Connect , Publish , RefCount , Replay
11. 反压操作: 用于增加特殊的流程控制策略的操作符。

## RxJava
在 RxJava 中，一个实现了 Observable 接口的对象可以订阅(subscribe) 一个 Observable 类的实例，订阅者(subscriber)对 Observable 发射(emit)的任何数据或者序列作出响应，这种模式简化了并发操作，因为它不需要阻塞等待 Observable 发射数据，而是创建一个处于待命状态的观察者哨兵，哨兵在未来某个时刻响应 Observable 的通知。