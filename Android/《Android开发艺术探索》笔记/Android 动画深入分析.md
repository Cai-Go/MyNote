#  Android 动画深入分析
## View 动画
1. View 动画的作用对象是 View,支持4种动画效果，分别是平移、缩放动画、旋转动画和透明度动画。帧动画也属于 View 动画。
2. View 动画的种类
    View 动画的四种变换效果对应着 Animation 的四个子类：TranslateAnimation、ScaleAnimation、RotateAnimation 和 AlphaAnimation。

    View动画的四种变换
    ![View动画的四种变换](http://o8fk8z4sl.bkt.clouddn.com/View%E5%8A%A8%E7%94%BB%E7%9A%84%E5%9B%9B%E7%A7%8D%E5%8F%98%E6%8D%A2.png)

    <set\>标签表示动画集合，对呀 AnimationSet 类，可以包含若干个动画。
    两个属性：

    - android:interpolator
        表示动画集合所采用的插值器，插值器影响动画的速度。
    - android:shareInterpolator
        表示集合中的动画是否和集合共享同一个插值器，如果集合不指定插值器，那么子动画就需要单独指定所需要的插值器或使用默认值。

    <translate\>标签表示平移动画，对应 TranslateAnimation 类，可以使一个 View 在水平和竖直方向完成平移的动画效果。
    属性含义：

    - android:fromXDelta --- 表示 x 的起始值；
    - android:toXDelta --- 表示 x 的结束值；
    - android:fromYDelta --- 表示 y 的起始值；
    - android:toYDelta --- 表示 y 的结束值。
    
    <scale\>标签表示缩放动画，对应 ScaleAnimation，可以使 View 具有放大或者缩小的动画效果。
    属性含义：

    - android:fromXScale --- 水平方向缩放的起始值；
    - android:toXScale --- 水平方向缩放的结束值；
    - android:fromYScale --- 竖直方向缩放的起始值；
    - android:toYScale --- 竖直方向缩放的结束值；
    - android:pivotX --- 缩放的轴点的 x 坐标，它会影响缩放的效果；
    - android:pivotY --- 缩放的轴点的 y 坐标，它会影响缩放的效果。
    
    <rote\>标签表示旋转动画，对应 RotateAnimation,可以使 View 具有旋转的动画效果。
    属性含义：

    - android:fromDegrees --- 旋转开始的角度；
    - android:toDegrees --- 旋转结束的角度；
    - android:pivotX --- 旋转的轴点的 x 坐标；
    - android:pivotY --- 旋转的轴点的 y 坐标。
    
    <alpha\>标签表示透明度的动画，对应 AlphaAnimation，可以改变 View 的透明度。
    属性含义：

    - android:fromAlpha --- 表示透明度的起始值；
    - android:toAlpha --- 表示透明度的结束值。
    
    View 动画的一些常用属性：

    - android:duration --- 动画的持续时间；
    - android:fillAfter --- 动画结束以后 View 是否停留在结束位置，true 表示 View 停留在结束位置，false 则不停留。

3. 自定义 View 动画，需要继承 Animation 这个抽象了，重写它的 initialize 和 applyTransformation 方法，在 initialize 方法中做一些初始化，在 applyTransformation 中进行相应的矩阵变换，需要采用 Camera 来简化矩阵变换的过程。
4. 帧动画是顺序播放一组预先定义好的图片，类似于电影播放。另外帧动画比如容易引起 OOM，所以在使用帧动画时应尽量避免使用过多尺寸较大的图片。

## View 动画的特殊使用场景
1. LayoutAnimation
    LayoutAnimation 作用于 ViewGroup。
    步骤：
    - 定义 LayoutAnimation
        相关属性
        - android:delay --- 表示子元素开始动画的时间延迟。
        - android:animationOrder --- 表示子元素动画的顺序，有三种选项：
            + normal：表示顺序显示；
            + reverse ：表示逆向显示；
            + random：表示随机播放入场动画。
        - anroid:animation --- 为子元素指定具体的入场动画。
    - 为子元素指定具体的如此动画
    - 为 ViewGroup 指定 android:layoutAnimation 属性：android:layoutAnimation = "@anim/anim_layout"。
2. Activity 的切换效果
    主要用到 overridePendingTransition(int enterAnim,int exitAnim)这个方法，这个方法必须在 startActivity(Intent) 或者 finish() 之后调用才能生效。
    参数含义：

    - enterAnim --- Activity 被打开时，所需要的动画资源 id;
    - exitAnim --- Activity 被暂停时，所需要的动画资源 id。

## 属性动画
1. 属性动画可以对任意对象的属性进行动画而不仅仅是 View，动画默认时间间隔300ms，默认帧率是10ms/帧。
    属性动画需要定义在 **res/animator/**目录下。

2. <objectAnimator\>标签属性的含义
    - android:propertyName --- 表示属性动画的作用对象的属性的名称；
    - android:duration --- 表示动画的时长；
    - android:valueFrom --- 表示属性的起始值；
    - android:valueTo --- 表示属性的结束值；
    - android:startOffset --- 表示动画的延迟时间，当动画开始画，需要延迟多少毫秒才会真正播放此动画；
    - android:repeatCount --- 表示动画的重复次数，默认值是0，-1表示无限循环；
    - android:repeatMode --- 表示动画的重复模式，有“repeat”和“reverse”选项，分别表示连续重复和逆向重复；
    - android:valueType --- 表示 android:propertyName 所指定的属性的类型，有“intType”和“floatType” 两个可选项，分别表示属性的类型为整型和浮点型。另外，如果 android:propertyName 所指定的属性表示的是颜色，那么不需要指定 android:valueType，系统会自动对颜色类型的属性做处理。

3. TimeInterpolator，时间插值器，作业是根据时间流逝的百分比来计算出当前属性值改变的百分比，系统预置的有
    - LinearInterpolator：线性插值器：匀速动画；
    - AccelerateDecelerateInterpolator：加速减速插值器：动画两头慢中间快；
    - DecelerateInterpolator：减速插值器：动画越来越慢。
    
4. TypeEvaluator,估值器，作用是根据当前属性改变的百分比来计算改变后的属性值，系统预置的有
    - IntEvaluator：针对整型属性；
    - FloatEvaluator：针对浮点型属性；
    - ArgbEvaluator：针对 Color 属性。

5. 自定义插值器需要实现 Interpolator 或者 TimeInterpolator，自定义估值算法需要实现 TypeEvaluator。如果要实现对其他类型(非 int、float、Color)做动画，那么必须要自定义类型估值算法。

6. 属性动画提供了监听器用于监听动画的播放过程，主要有两个接口
    - AnimatorUpdateListener：可以监听动画开始、结束、取消以及重复播放；
    - AnimatorListener：会监听整个动画过程。

7. 属性动画的原理：属性动画要求动画作用的对象提供该属性的 get 和 set 方法，属性动根据外界传递的该属性的初始值和最终值，以动画的效果多次去调用 set 方法，每次传递给 set 方法的值都不一样，确切来说是随时间的推移，所传递的值越来约接近最终值。

8. 使用动画的注意事项
    - OOM 问题
        主要出现在帧动画中，实际开发中尽量避免使用帧动画。
    - 内存泄漏
         主要出现在属性动画中的无限循环动画，需要在 Activity 退出时及时停止。
    - 兼容性问题
    - View 动画的问题
        当出现动画完成后 View 无法隐藏的现象，即 setVisiblity(View.GONE)失效时，需要调用 view.clearAnimation() 清楚 View 动画来解决此问题。
    - 不要使用 px
    - 动画元素的交互
    - 硬件加速
                

                                                                      2017.8.21
                                                                        W.Z.H
