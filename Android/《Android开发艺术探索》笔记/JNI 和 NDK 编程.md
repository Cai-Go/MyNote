# JNI 和 NDK 编程
1. Java JNI 本意是 Java Native Interface (Java 本地接口)，是为了方便 Java 调用 C、C++等本地代码所封装的一层接口。
2. NDK 是 Android 所提供的一个工具集合，通过 NDK 可以在 Android 中更加方便地通过 JNI 来访问本地代码。
    使用 NDK 的好处：
    - 提高代码的安全性；
    - 可以很方便地使用目前已有的 C/C++开源库；
    - 便于平台间的移植；
    - 通过程序在某些特定情形下的执行效率，但是并不能明显提升 Android 程序的性能。

## JNI 的开发流程
1. JNI 的开发流程:
   
   - 在 Java 中声明 native 方法；
   - 编译 Java 源文件得到 class 文件，然后通过 Java 命令导出 JNI 的头文件；
   - 实现 JNI 方法；
   - 编译 so 库并在 Java 中调用。
   
## NDK 的开发流程
1. NDK 的开发流程：
    
    - 下载并配置 NDK；
    - 创建一个 Android 项目，并声明所需的 native 方法；
    - 实现 Android 项目中所声明的 native 方法；
    - 切换到 jni 目录的父目录，然后通过 ndk-build 命令编译产生 so 库。

## JNI 的数据类型和类型签名
1. JNI 基本数据类型的对应关系
    ![JNI 基本数据类型的对应关系](http://o8fk8z4sl.bkt.clouddn.com/JNI%20%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)
2. JNI 引用类型的对应关系
    ![JNI 引用类型的对应关系](http://o8fk8z4sl.bkt.clouddn.com/JNI%20%E5%BC%95%E7%94%A8%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png)
3. 基本数据类型的签名
    ![基本数据类型的签名](http://o8fk8z4sl.bkt.clouddn.com/%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%AD%BE%E5%90%8D.png)

## JNI 调用 Java 方法的流程
1. JNI 调用 Java 方法的流程是先通过类名找到类，然后再根据方法名找到方法的 id，最后调用这个方法。
    


                                                                    2017.8.30
                                                                      W.Z.H