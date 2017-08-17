# View 的工作原理
## ViewRoot 和 DecorView
1. ViewRoot 对应于 ViewRootImpl 类，是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。View 绘制流程是从 ViewRoot 的 performTraversals 方法开始的，经过 measure、layout 和 draw 三个过程才能最终将一个 View 绘制出来，其中 measure 用来测量 View 的宽和高，layout 用来确定 View 在父容器中的放置位置，draw 负责将 View 绘制在屏幕上。

**performTraversals的大致流程**
![performTraversals的大致流程](http://o8fk8z4sl.bkt.clouddn.com/performTraversals%E7%9A%84%E5%A4%A7%E8%87%B4%E6%B5%81%E7%A8%8B.png)
    performTraversals 会依次调用 performMeasure、performLayout 和 preformDraw 三个方法，这三个方法分别完成了顶级 View 的 measure、layout 和 draw 这三大流程，其中 performMeasure 中会调用 measure 方法，在 measure 方法中又回调用 onMeasure 方法，在 onMeasure 方法中则会对所有子元素进行 measure 过程，这个时候 measure 流程就就从父容器传递到子元素中，完成一次 measure 过程，接着子元素会重复父容器的 measure 过程，如此反复就完成了整个 View 树的遍历。

## MeasureSpec
1. MeasureSpec 代表一个32为 int 值，高2位代表 SpecMode，低30位代表 SpecSize，SpecMode 是指测量模式，SpecSize 是指在某种测量模式下的规格大小。
2. SpecMode有三类
    
    - UNSPECIFIED
        父容器不对 View 有任何限制，这种情况一般用于系统内部，表示一种测量状态。
    - EXACTLY
        父容器已经检测出 View 所需要的精确大小，这个时候 View 的最终代大小就是 SpecSize 所指定的值，对应于 LayoutParams 中的 match_parent 和具体的数值这两种模式。
    - AT_MOST
        父容器指定了一个可用大小即 SpecSize，View 的大小不能大于这个值，具体要看不同 View 的具体实现，对应于 LayoutParams 中的 wrap_content。

3. MeasureSpec 不是唯一由 LayoutParams 决定的，LayoutParams 需要和父容器一起才能决定 View 的 MeasureSpec，从而进一步决定 View 的宽/高。
    对于 DecorView，其 MeasureSpec 由窗口的尺寸和其自身的 LayoutParams 来共同确定; 对于普通 View，其 MeasureSpec 由父容器的 MeasureSpec 和自身的 LayoutParams 来共同决定。
4. DecorView 的 MeasureSpec 的产生过程遵守如下规则：
    
    - LayoutParams.MATCH_PARENT：精确模式，大小就是窗口的大小。
    - LayoutParams.WRAP_CONTENT：最大模式，大小不定，但是不能超过窗口的大小。
    - 固定大小：精确模式，大小为 LayoutParams 中指定的大小。

5. 当 View 采用固定宽/高的时候，不过父容器的 MeasureSpec 是什么，View 的 MeasureSpec 都是精确模式并且其大小遵循 LayoutParams 中的大小。
    当 View 的宽/高是 match_parent 时，如果父容器的模式是精准模式，那么 View 也是精准模式并且其大小是父容器的剩余空间; 如果父容器是最大模式，那么 View 也是最大模式并且其大小不会超过父容器的剩余空间。
    当 View 的宽/高是 wrap_content 时，不管父容器的模式是精准还是最大化，View 的模式总是最大化并且大小不能超过父容器的剩余空间。

6. 普通View的MeasureSpec的创建规则
![普通View的MeasureSpec的创建规则](http://o7qv8ih35.bkt.clouddn.com/%E6%99%AE%E9%80%9AView%E7%9A%84MeasureSpec%E7%9A%84%E5%88%9B%E5%BB%BA%E8%A7%84%E5%88%99.png)

## View 的工作流程
1.  View 的工作流程主要是指 measure、layout、draw 这三大流程，即测量、布局、绘制。其中 measure 确定 View 的测量宽/高，layout 确定 View 的最终宽/高和四个顶点的位置，drea 则将 View 绘制到屏幕上。
2.  measure过程
    
    - View 的 measure 过程
        View 的 measure 过程由其 measure 方法来完成，measure 方法是一个 final 类型方法，在 View 的 measure 方法中回去调用 View 的 onMeasure 方法。
       
        getSuggestedMinimumWidth 的逻辑：如果 View 没有设置背景，那么返回 android:miniWidth 这个属性所指定的值，这个值可以为0；如果 View 设置了背景，则返回 android:minWidth 和背景的最小宽度这两者中的最大值，getSuggestedMinimumWidth 和 getSuggestedMinimumHeight 的返回值就是 View 在 UNSPECIFIED 情况下的测量宽/高。

        直接继承 View 的自定义控件需要重写 onMeasure 方法并设置 wrap_content 时的自身大小，否则在布局中使用 wrap_content 就相当于使用 match_parent。

    - ViewGroup 的 measure 过程
        对于 ViewGroup 来说，除了完成自己的 measure 过程以外，还会遍历去调用所有子元素的 measure 方法，各个子元素再递归去执行这个过程。和 View 不同的是，ViewGroup 是一个抽象类，没有重写 View 的 onMeasure 方法，但是提供了一个 measureChildren 的方法。
        
        measureChildren 的思想就是取出子元素的 LayoutParams，然后再通过 getChildMeasureSpec 来创建子元素的 MeasureSpec，接着将 MeasureSpec直接传递给 View 的 measure 方法来进行测量。

        实际上在 onCreate、onStart、onResume 中均无法正确得到某个 View 的宽/高信息，因为 View 的 measure 过程和 Activity 的生命周期方法不是同步执行的，无法保证 Activity 执行了 onCreate、onStart、onResume 时某个 View 已经测量完毕了。
        解决方法：
        
        - Activity/View#onWindowFocusChanged
            这个方法的含义是：View 已经初始化完毕，宽/高已经准备好了，这个时候去获取宽/高是没有问题，但是 onWindowFocusChanged 会被调用多次，当 Activity 的窗口得到焦点和失去焦点时均会被调用一次。
        - view.post(runnable)
            通过 post 可以将一个 runnable 投递到消息队列的尾部，然后等待 Looper 调用此 runnable 的时候，View 也已经初始化好了。
        - ViewTreeObserver
        - view.measure(int widthMeasureSpec,int heightMeasureSpec)
            通过手动对 View 进行 measure 来得到 View 的宽/高。这个需要根据 View 的 LayoutParams 来分情况处理:
                1. match_parent:
                    直接放弃，无法 measure 出具体的宽/高，因为我们无法知道 parentSize 的大小，理论上不可能测量出 View 的大小。
                2. 具体的数值(dp/px)
                3. wrap_content
            
            **两种错误的用法**
            一是违背了系统内部实现规范；其次不能保证一定能 measure 出正确的结果。
            第一种错误用法
            ````java
                int widthMeasureSpec = MeasureSpec.makeMeasureSpec(-1,MeasureSpec.UNSPECIFIED);
                int heightMeasureSpec = MeasureSpec.makeMeasureSpec(-1,MeasureSpec.UNSPECIFIED);
                view.measure(widthMeasureSpec,heightMeasureSpec);
            ````
            第二种错误用法
            ````java
                view.measure(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);
            ````

    - layout 过程        
        Layout 的作用是 ViewGroup 用来确定子元素的位置，当 ViewGroup 的位置被确定后，它在 onLayout 中会遍历所有子元素并调用其 layout 方法,在 layout 方法中 onLayout 方法又会被调用。layout 方法确定 View 本身的位置，onLayout 方法则会确定所有子元素的位置。

        大致流程：
        首先会通过 setFrame 方法来设定 View 的四个顶点的位置，即初始化 mLeft、mRight、mTop 和 mBottom 这四个值，从而确定 View 在父容器中的位置。
        接着会调用 onLayout 方法，用途是父容器确定子元素的位置。

        和 onMeasure 方法类似，onLayout 的具体实现同样和具体的布局有关，所以 View 和 ViewGroup 均没有真正实现 onLayout 方法。

    - draw 过程
        作用是将 View 绘制到屏幕上。
        View 的绘制过程遵循如下几步：

        - 绘制背景 background.draw(canvas);
        - 绘制自己 (onDraw);
        - 绘制 children（dispatchDraw）;
        - 绘制装饰（onDrawScrollBars）。

## 自定义 View
1. 自定义 View 的分类
    
    - 继承 View 重写 onDraw 方法
        主要用于实现一些不规则的效果往往需要静态或者动态地实现一些不规则的图形。采用这种方式需要自己支持 wrap_content，并且 padding 也需要自己处理。
    - 继承 ViewGroup 派生特殊的 Laout
        主要用于实现自定义的布局，需要合适地处理 ViewGroup 的测量、布局这两个过程，并同时处理子元素的测量和布局过程。
    - 继承特定的 View(比如 TextView)
        用于扩展某种已有的 View 的功能。
    - 继承特定的 ViewGroup (比如 LinearLayout)
        与方法2的区别是方法2更接近 View 的底层。

2. 自定义 View 须知
    
    - 让 View 支持 wrap_content;
    - 如果有必要，让 View 支持 padding;
    - 尽量不要在 View 中使用 Handler，没必要;
    - View 中如果有线程或者动画，需要及时停止，参考 View#onDtachedFromWindow;
    - View 带有滑动嵌套情形时，需要处理好滑动冲突;

3. 添加自定义属性的方法

    - 在 values 目录下面创建自定义属性的 XML，在 XML 中声明一个自定义属性的集合；
    - 在 View 的构造方法中解析自定义属性的值并做相应处理；
    - 在布局文件中使用自定义属性。
        需要注意的是为了使用自定义属性，必须在布局文件中添加 schemas 声明
        ````
            xmlns:app = http://schemas.android.com/apk/res-auto
        ````
        或
        ````
            xmlns:app = http://schemas.android.com/apk/res-auto/com.ryg_chapter_4
        ````



                                                                    2017.8.18
                                                                      W.Z.H