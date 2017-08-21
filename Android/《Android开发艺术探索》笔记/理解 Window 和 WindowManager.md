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
1. 



