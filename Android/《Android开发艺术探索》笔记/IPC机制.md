# IPC机制
## Android IPC简介
1. IPC: Inter-Process Communication 的缩写，含义为进程间通信或者跨进程通信，是指两个进程之间进行数据交换的过程。
2. 进程与线程。
    按照操作系统中的描述，线程是 CPU 调度最小单元，是一种有限的系统资源；进程一般指一个执行单元，在 PC 或者移动设备上指一个程序或者与一个应用。
    一个进程可以包含多个线程，因此进程和线程是包含与被包含的关系。
    ANR:Application Not Responding,应用无响应。解决这个问题就需要用到线程，把一些耗时的任务放在线程中即可。

## Android中的多进程模式
1. 正常情况下，在 Android 中多进程是指一个应用中存在多个进程的情况。
2. 在 Andorid 中使用多进程只有一种方法，就是给四大组件在 AndroidManifest 中指定 android:process 属性。
    还有另一种方法就是通过 JNI 在 native 层去 fork 一个新的进程。
3. 默认进程就是包名。
4. ":remote"和"com.ryg.chapter_2.remote" 这两种方式的区别：
    - 首先，":"的含义是指要在当前的进程名前面附加上当前的包名，是一种简写的方法。而另一种是完整的命名方式，不会附加包名信息。
    - 其次，进程名以":"开头的进程属于当前应用的私有进程，进程名不以":"开头的进程属于全局进程，其他应用通过 ShareID 方式可以和它跑在同一个进程中。
5. Android 系统会为每个应用分配一个唯一的 UID ，具有相同 UID 的应用才能共享数据。另外，两个应用需要有相同的 ShareUID 且签名相同才能跑在同一个进程中。可以共享 data 目录、组件信息，还可以共享内存数据。
6. Android 为每一个应用分配了一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，导致在不同的虚拟机中访问同一个类的对象会产生多份副本。
7. 所有运行在不同进程中的四大组件，只要它们之间需要通过内存来共享数据，都会共享失败。
8. 使用多进程会造成的问题：
    - 静态成员和单例模式完全失效。
    - 线程同步机制完全失效。
    - SharePreferences 的可靠性下降。因为 SharePreferences 不支持两个进程同时去执行写操作，否则会导致一定几率的数据丢失，因为 SharePreferences 底层是通过读/写XML文件来实现的，并发写显然是可能出现问题的。
    - Application 会多次创建。运行在同一个进程中的组件是属于同一个虚拟机和同一个 Application 的，同理，运行在不同进程中的组件是属于两个不同的虚拟机和 Application 的。

## IPC基础概念介绍
1. Serializable 是 Java 提供的一个序列化接口，是一个空接口，为对象提供标准的序列化和反序列化操作。
    ````java
        public class User implements Serializable{
            private static final long serialVersionUID = 519061723721295573l;

            public int userId;
            public String userName;
            public boolean isMale;
            ...
        }
    ```` 
只需要采用 ObjectOutputStream 和 ObjectInputStream 即可实现对象的序列化和反序列化。
    ````java
        //序列化过程
        User user = new User(0,"jake",true);
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("cache.txt"));
        out.writeObject(user);
        out.close();

        //反序列化过程
        ObjectInputStream int= new ObjectInputStream(new FileInputStream("cache.txt"));
        User newUser = in.readObject();
        in.close();
    ````
2. serialVersionUID 是用来辅助序列化和反序列化过程的，原则上序列化后的数据中的 serialVersionUID 只有和当前类的 serialVersionUID 相同才能够正常地被反序列化。
serialVersionUID 的工作机制：
> 序列化的时候系统会把当前类的 serialVersionUID 写入序列化的文件中(也可能是其他中介)，当反序列化的时候系统就会去检测文件中的 serialVersionUID，看它是否和当前类的 serialVersionUID 一致，如果一致就说明序列化的类的版本和当前类的版本是相同的，这个时候可以成功反序列化；否则就说明当前类和序列化的类相比发生了某些变化，比如成员变量的数量、类型可能发生了改变，这个时候是无法正常反序列化的。

    一般我们应该手动指定 serialVersionUID 的值，这样序列化和反序列化时两者的 serialVersionUID 是相同的，可以正常进行反序列化；不手动指定 serialVersionUID 的值，反序列化时当前类有所改变，那么系统就会重新计算当前类的 hash 值并把它赋值给 serialVersionUID ，这时候当前类的 serialVersionUID 就会和序列化的数据中的 serialVersionUID 不一致，导致反序列化失败，程序出现crash。
    静态成员变量属于类不属于对象，不会参与序列化过程。
    用 transient 关键字标记的成员变量不参与序列化过程。

3. Parcelable 接口是 Android 中提供的新的序列化方式，实现这个接口，一个类的对象就可以实现序列化并可以通过 Intent 和 Binder 传递。
    ````java
        public class User implements Parcelable{
            public int userId;
            public String userName;
            public boolean isMale;

            public Book book;

            public User(int userId,String userName,boolean isMale){
                this.userId = userId;
                this.userName = userName;
                this.isMale = isMale;
            }

            //内容描述功能
            public int describeContents(){
                return 0; //几乎在所有情况下这个方法都返回0，仅当当前对象中存在文件描述时，返回1。
            }

            //序列化
            public void writeToParcel(Parcel out, int flags){
                out.writeInt(userId);
                out.writeString(userName);
                out.writeInt(isMale ? 1 : 0);
                out.writeParcelable(book,0);
            }

            public static final Parcelable.Creator<User> CREATOR = new Parcelable.Creator<User>(){
                //创建序列化对象
                public User createFromParcel(Parcel in){
                    return new User(in);
                }
                //创建序列化数组
                public User[] newArray(int size){
                    return new User[size];
                }
            };

            //完成反序列化
            private User(Parcel in){
                userId = in.readInt();
                userName = in.readString();
                isMale = in.readInt() == 1;
                book = in.readParcelable(Thread.currentThread().getContextClassLoader());
            }
        }
    ````
    Parcel 内部包装了可序列化的数据，可以在 Binder 中自由传输。

    Parcelable的方法说明。            
        1. createFromParcel(Parcel in):从序列化后的对象中创建原始对象。
        2. newArray(int size) :创建指定长度的原始对象数组。
        3. User(Parcel in):从序列化后的对象中创建原始对象。
        4. writeToParcel(Parcel out,int flags):将当前对象写入序列化结构中，其中 flags 标识两种值：0或者1(标记位：PARCELABLE_WRITE_RETURN_VALUE)。为1时标识当前对象需要作为返回值返回，不能立即释放资源，几乎所有情况都为0。
        5. describeContents：返回当前对象的内容描述，如果含有文件描述符，返回1(标记位：CONTENTS_FILE_DESCRIPTOR)，否则返回0，几乎所有情况都为0。
4. Serializable 和 Parcelable 的区别。
    Serializable 是 Java 中的序列化接口，其使用起来简单但是开销很大，序列化和反序列化过程需要大量I/O操作，Parcelable 是 Android 中的序列化方式，主要用在内存序列化上，适合在 Android 平台上使用，缺点就是使用起来稍微麻烦，但是效率很高。如果要将对象序列化到存储设备中或者将对象序列化后通过网络传输建议使用 Serializable。

## Binder

1. Binder 是 Android 中的一个类，实现了 IBinder 接口。
    从 IPC 角度来说，Binder 是 Android 中的一种跨进程通信方式。
    从 Android Framework 角度来说，Binder 是 ServiceManager 链接各种 Manager(ActivityManager、WindowManager)和相应 ManagerService 的桥梁。
    从 Android 应用层来说，Binder 是客户端和服务端进行通信的媒介。
    Android 开发中，Binder 主要用 Service 中，包括 AIDL 和 Messenger。
2.解释
    - DESCRIPTOR
        Binder 的唯一标识，一般用当前 Binder 的类名表示。
    - asInterface(android.os.IBinder obj)
    用于将服务端的 Binder 对象转换成客户端所需的 AIDL 接口类型的对象，这种转换过程是区分进程的，如果客户端和服务端位于同一进程，此方法返回的就是服务端的 Stub 对象本身，否则返回的是系统封装后的 Stub.proxy 对象。
    - asBinder
        此方法用于返回当前 Binder 对象。
    - onTransact
        这个方法运行在服务端中的 Binder 线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。如果此方法返回 false,那么客户端的请求会失败。
    - Proxy#getBookList
    - Proxy#addBook
    
    **Binder的工作机制**
    ![Binder 的工作机制](http://7xq2jk.com1.z0.glb.clouddn.com/Binder%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png)

3. 手动实现一个Binder
    1. 声明一个AIDL性质的接口，只需要继承 IInterface 接口即可，IInterface 接口中只有一个 asBinder 方法。
    2. 实现 Stub 类和 Stub 类中的 Proxy 代理类。

4. Binder 中提供了两个配对的方法 linkToDeath 和 unlinkToDeath,通过 linkToDeath 可以给 Binder 设置一个死亡代理，当 Binder 死亡时，就会收到通知，从而重新发起连接请求来恢复连接。
    首先声明一个 DeathRecipient 对象，DeathRecipient 是一个接口，其内部只有一个方法 binderDied,当 Binder 死亡的时候，系统就会回调 binderDied 方法，然后移除之前绑定的 binder 代理并重新绑定远程服务：
    ````java
        private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient(){
            @override
            public void binderDied(){
                if(mBookManager == null)
                    return;
                mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
                mBookManager = null;
                //TODO: 这里重新绑定远程 Service
            }
        }
    ````
    其次，在客户端绑定远程服务成功后，给 binder 设置死亡代理：
    ````java
        mSerivce = IMessageBoxManager.Stub.asInterface(binder);
        binder,linkToDeath(mDeathRecipient,0);
    ````

## Android中的IPC方式
1. 