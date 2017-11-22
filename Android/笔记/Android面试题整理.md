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

6. Intent传递数据时，下列的数据类型哪些可以被传递？
    - Serializable:将Java对象序列化为二进制文件的Java序列化技术是Java系列技术中一个较为重要的技术点，在大部分情况下，开发人员只需要了解被序列化的类需要实现Serializable接口，使用ObjectInputStream和ObjectOutputStream进行对象的读写。
    - charsequence:在JDK1.4中，引入了CharSequence接口，实现了这个接口的类有：CharBuffer、String、StringBuffer、StringBuilder这四个类。CharBuffer为nio里面用的一个类，String实现这个接口理所当然，StringBuffer也是一个CharSequence，StringBuilder是Java抄袭C#的一个类，基本和StringBuffer一样，效率高，但是不保证线程安全，在不需要多线程的环境下可以考虑。
    提供这么一个接口，有些处理String或者StringBuffer的类就不用重载了。但是这个接口提供的方法有限，只有下面几个：charat、length、subSequence、toString这几个方法，如果有必要，重载比较好，避免用instanceof这个操作符。
    - Parcelable:android提供了一只新的类型：Parcel。本类被用作封装数据的容器，封装后的数据可以通过Intent或IPC传递。除了基本类型以外，只有实现了Parcelable接口类才能被放入Parcel中。
    - Bunldle:Bundle是将数据传递到另一个上下文中或报酬或回复你自己状态的数据存储方式。它的数据不是持久化状态。

7. 一个GLSurfaceView类 , 具有以下特点 :
    - 管理一个平面, 这个平面是一个特殊的内存块 , 它可以和 android 视图系统混合 。
    - 管理一个EGL 显示 , 它能够让 OpenGL 渲染到一个平面 。
    - 接受一个用户提供的实际显示的Renderer 对象 。
    - 使用一个专用线程去渲染从而和UI 线程解耦 。
    - 支持on-demand  和连续的渲染。
    - 可选的包, 追踪 和 / 或者错误检查这个渲染器的 OpenGL 调用 。

8. 简述Andriod如何处理UI与耗时操作的通信，有哪些方式及各自的优缺点。
    
    主要有三种方法，一为Handler,二为AsyncTask,三为自己开子线程执行耗时操作，然后调用Activity的runOnUiThread()方法更新ui。
    - handler机制是，在主线程中创建handler对象，当执行耗时操作时，创建一个线程，在这个线程中执行耗时操作，通过调用handler的sendMessage，post等方法，更新ui界面。
    handler机制的优点是结构清晰，功能明确，但是代码过多。
    - AysncTask本质上是一个线程池，所有的异步任务都会在这个线程池中的工作线程中执行，当需要操作ui界面时，会和工作线程通过handler传递消息。
    AysncTask简单，快捷，但是可能会新开大量线程，消耗系统资源，造成Force Close。
    - 自己开子线程执行耗时操作，然后调用Activity的runOnUiThread()方法更新ui。这种方法需要把context对象强制转换成activity后使用。
    第三种方法最好用，代码也非常简单，只是需要传递context对象。

9. Android中Looper的实现原理，为什么调用Looper.prepare()就在当前线程关联了一个Looper对象，它是如何实现的。
    
    - 线程间通信机制
    首先，looper、handler、messagequeue三者共同实现了android系统里线程间通信机制。如在A、B两个子线程之间需要传递消息，首先给每个子线程绑定一套looper、handler、messagequeue，然后这三个对象都与其所属线程对应。然后A线程通过调用B线程的Handler对象，发送消息。这个消息被Handler发送到B线程的messageQueue中，而属于B线程的Looper对象一直在for循环里无限遍历MessageQueue，一旦发现该消息队列里收到新的消息，就会对消息进行处理，处理过程中会回调自身Handler的headleMessage方法。从而实现了不同线程间通信。
    - Looper实现原理
    Looper类里包含了一个消息队列对象和一个线程对象。当创建Looper时，会自动创建一个消息队列，同时将内部线程对象指向创建Looper的线程。当开启Looper后(looper.loop()),会自动进入无限for循环中,不断去遍历消息队列，如果没有消息阻塞，有消息则回调handler的handlemessage方法进行处理。
    - Looper.prepare()
    首先，要使用Looper机制一般会在当前线程创建Hnadler对象，里面会自动创建一个looper对象消息队列，这里面的消息队列属于当前线程空间。但此时的looper还不会去遍历，也没有绑定到当前线程。其中，looper对象内部也包含一个空消息队列对象和空线程。通过Looper.prepare()方法，先让该消息队列指向当前线程的消息队列，让控线程也指向对其线程，从而实现绑定。

10. 分辨率相关
    
     在 Android 中，  1pt 大概等于 2.22sp以上，
 与分辨率无关的度量单位可以解决这一问题。Android支持下列所有单位。 
       px（像素）：屏幕上的点。 
       in（英寸）：长度单位。 
       mm（毫米）：长度单位。 
       pt（磅）：1/72英寸。 
       dp（与密度无关的像素）：一种基于屏幕密度的抽象单位。在每英寸160点的显示器上，1dp = 1px。 
       dip：与dp相同，多用于android/ophone示例中。 
       sp（与刻度无关的像素）：与dp类似，但是可以根据用户的字体大小首选项进行缩放。
    分辨率:整个屏是多少点，比如800x480，它是对于软件来说的显示单位，以px为单位的点。     
    density(密度)值表示每英寸有多少个显示点，与分辨率是两个概念。apk的资源包中，  
       当屏幕density=240时使用hdpi标签的资源  
       当屏幕density=160时，使用mdpi标签的资源  
       当屏幕density=120时，使用ldpi标签的资源。  
       一般android设置长度和宽度多用dip,设置字体大小多用sp. 在屏幕密度为160，1dp=1px=1dip, 1pt = 160/72 sp 1pt = 1/72 英寸.当屏幕密度为240时，1dp=1dip=1.5px. 

11. andorid数据持久化的方法。  
    Android数据持久化有五种方式：
    
    - SharedPreferences
    - 内部存储（例如通过openFileOutput()打开一个文件输入输出流）
    - SQLite Database
    - 网络连接（将数据存储到服务器上）
    - 外部存储（SD卡）