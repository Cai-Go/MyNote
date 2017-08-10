# Android面试题整理

1. 什么是ANR和Force Close？如何避免？
    ANR: Application Not Responding
    产生原因:
    - 主线程（UI线程）响应用户操作事件事件超过5秒。
    - BroadcastReceive超过10秒钟未执行完毕。
    避免方法：
    Android应用程序完全运行在一个独立的线程中。任何在主线程中运行的，需要消耗大量时间的操作都会引发ANR。因此，需要消耗大量事件的操作如访问网络和数据库，都要放到子线程中或者使用异步方式来完成。

    Force Close
    产生原因：
    程序出现异常，一般像空指针、数组越界、类型转换异常等。
    避免方法：
    编写程序时要思维缜密，异常出现时可以通过logcat查看抛出异常代码出现的位置，然后去程序中进行修改。
    
2. 请描述下Activity的生命周期。
    ![Activity的生命周期](https://uploadfiles.nowcoder.com/images/20170123/976341_1485165828993_C4FBAB681EDE2673E8C75536C5B00931)

    - onCreate
     >在Activity第一次被创建的时候调用，可以在这个方法中完成Activity的初始化操作，比如加载布局，绑定事件。
    - onStart
     >在Activity由不可见变为可见时候调用。
    - onResume
     >在Activity准备好和用户交互时调用，此时Activity一定位于返回栈的栈顶，并处于运行状态。
    - onPause
     >在系统准备去启动或者恢复另一个Activity时候调用。可以在这个方法中将一些消耗CPU的资源释放掉，以及保持一些关键数据（但不能做耗时操作，否则影响新的栈顶Activity的使用）。
    - onStop
     >在Activity完全看不见的时候调用。与onPause的区别：如果启动一个对话框。那么onPause会执行，onStop不会。
    - onDestroy
     >在Activity被销毁前调用，之后活动变为销毁状态。
    - onRestart 
     >这个活动由停住状态变化运行状态之前调用，也就是被重新启动了。

    我们可以将Activity分为三种生存周期

    - 完整生存期
     >由 onCreate->onDestroy
    - 可见生存期
     >由 onStart->onStop
    - 前台生存期
    >由 onResume->onPause


3. 多线程有几种实现方法，都是什么？同步有几种实现方法，都是什么？
    多线程的实现方法：

    - 继承Thread类。
    1. 自定义类MyThread继承Thread类。
    2. MyThread类里面重写run()。
    3. 创建对象。
    4. 启动线程。
    - 实现Runnable结口。
    1. 自定义类MyRunnable实现Runnable接口。
    2. 重新run()方法。
    3. 创建MyRunnable类对象。
    4. 创建Thread类的对象，并把上一步的对象作为构造参数传递。
    - 实现Callable接口，重写call()方法。但是这个方法不常用。

    同步的实现方法：

    - synchronized关键字：包括synchronized方法和synchronized块。
    - wait()方法和notify()方法。
    - Lock接口及其实现类RenntrantLock。

4. 如何退出Activity？如何安全退出已调起多个Activity的Application？
    - finish() 。
    - 新建一个类ActivityCollector用于管理全部的Activity对象，每生成一个Activity对象就将其添加到ActivityCollector一个List中，在ActivityCollector中实现一个finishAll()方法，用于结束list中所有的Activity对象。

5. 用至少两种方式实现一个Singleton（单例模式）。
    - 静态内部类实现：这种方式是 Singleton 类被装载了， instance 不一定被初始化。因为 SingletonHolder 类没有被主动使用，只有显示通过调用 getInstance 方法时，才会显示装载 SingletonHolder 类，从而实例化 instance 。
      ````
        public class Signleton{
            private static class SingletonHolder{
                private static final Singleton INSTANCE = new Singleton();
            }
            private Singleton(){}
            public static final Singleton getInstance(){
                return SingletonHolder.INSTANCE;
            }
        }
      ````
    - 双重检查锁：在JDK1.5之后，双重检查锁定才能够正常达到单例效果。（因为volitile关键字）。
        ````
        public class Signleton{
            private volatile static Singleton singleton;
            private Singleton(){}
            public static Singleton getSingleton(){
                if(singleton == null){
                    synchronized(Singleton.class){
                        if(singleton == null){
                            singleton = new Singleton()；
                        }
                    }
                }
                return singleton;
            }
        }
        ````