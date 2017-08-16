# View 的事件体系
## View
1. View 是 Android 中所有控件的基类，是一种界面层的抽象，代表了一个控件。ViewGroup 也继承了 View，意味着 View 本身就可以是单个控件也可以是由多个控件组成的一组控件。
**View 树的结构**
![View 树的结构](http://o8fk8z4sl.bkt.clouddn.com/viewgroup.png)
2. View 的位置主要由它的四个顶点来决定，分别对应于 View 的四个属性：
    - top：左上角纵坐标;
    - left：左上角横坐标;
    - right：右下角横坐标;
    - bottom：右下角纵坐标;
    需要注意的是这些坐标是相对于 View 的父容器来说的，是一种相对坐标。
    **View 的位置坐标和父容器的关系**
    ![View 的位置坐标和父容器的关系](http://o8fk8z4sl.bkt.clouddn.com/View%20%E7%9A%84%E4%BD%8D%E7%BD%AE%E5%9D%90%E6%A0%87%E5%92%8C%E7%88%B6%E5%AE%B9%E5%99%A8%E7%9A%84%E5%85%B3%E7%B3%BB.png)
    Viewd 的宽高和坐标的关系：
    >width = right - left
    height = bottom - top
    
    View 四个参数的获取方式

    - Left = getLeft();
    - Right = getRight();
    - Top = getTop();
    - Bottom = getBottom();
    
    Android 3.0后，View 新增了 x、y、translationX 和 translationY 四个参数。其中 x 和 y 是 View 左上角的坐标，translationX 和 translationY 是 View 左上角相对于父容器的偏移量，并且 translationX 和 translationY 的默认值为0。
    View 在平移过程中，top 和 left 表示的是原始左上角的位置信息，其值不会发生改变，发生改变的是 x、y、translationX 和 translationY。

3. MotionEvent 
    在手指接触屏幕后所产生的一系列事件中，典型的事件类型有：

    - ACTION_DOWN --- 手指刚接触屏幕;
    - ACTION_MOVE --- 手指在屏幕上移动;
    - ACTION_UP --- 手指从屏幕上松开的一瞬间;
    正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件：

    - 点击屏幕后离开松开，事件序列为 DOWN -> UP;
    - 点击屏幕滑动一会再松开，事件序列为 DOWN -> MOVE -> ... -> MOVE -> UP。
    
    getX/getY 返回的是相对于当前 View 左上角的 x 和 y 坐标，getRawX/getRawY 返回的是相对于手机屏幕左上角的 x 和 y 坐标。

4. TouchSlop 是系统所能识别出的被认为是滑动的最小的距离，这是一个常量，和设备有关，可以通过 **ViewConfiguration.get(getContext()).getScaledTouchSlop()** 来获取这个常量。

5. VelocityTracker
    速度追踪，用于追踪手指在滑动过程中的速度，包括水平和竖直方向的速度。
    **使用方法：**
        首先在 View 的 onTouchEvent 方法中追踪当前单击事件的速度：
        ````java
            VelocityTracker velocityTracker = VelocityTracker.obtained();
            velocityTracker.addMovenment(event);
        ````
        接着当先知道当前的滑动速度时，可以采用如下方式获取当前的速度：
        ````java
            velocityTracker.computeCurrentVelocity(1000); //获取速度之前必须先计算速度，所以这个方法必须放在下面2个方法前面。
            int xVelocity = (int) velocityTracker.getXVelocity();
            int yVelocity = (int) velocityTracker.getYVelocity();
        ````
        这里的速度是指一段时间内手指所滑过的像素数，可以为负数，即当手指从右往左滑动时水平方向速度即为负数，公式:
            **速度 = （终点位置 - 起点位置）/ 时间段**
        最后，当不需要使用的时候，调用 clear 方法来重置并回收内存
        ````java
            velocityTracker.clear();
            velocityTracker.recycle();
        ````

6. GustureDetector
    手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为。
    **使用方法：**
        首先创建一个 GustureDetector 对象并实现 OnGestureListener 接口，也可以实现 OnDoubleTapListener 从而能够监听双击行为：
        ````java
            GustureDetector mGustureDetector = new GustureDetector(this);
            //解决长按屏幕后无法拖动的现象
            mGustureDetector.setIsLongpressEnabled(false);
        ````
        接着接管目标 View 的 onTouchEvent 方法，在监听 View 的 onTouchEvent 方法中添加如下实现：
        ````java
            boolean consume = mGustureDetector.onTouchEvent(event);
            return consume;
        ````
        **GustureDetector和onDoubleTaoListener中的方法介绍**
        ![GustureDetector和onDoubleTaoListener中的方法介绍](http://o8fk8z4sl.bkt.clouddn.com/GustureDetector%E5%92%8ConDoubleTaoListener%E4%B8%AD%E7%9A%84%E6%96%B9%E6%B3%95%E4%BB%8B%E7%BB%8D.jpg)
>参考：
        如果只是监听相关滑动，建议自己在 onTouchEvent 中实现，如果要监听双击这种行为的话，就使用 GestureDetector。

7. Scoller
    弹性滑动对象，用于实现 View 弹性滑动，需要与 View 的 computeScroll 方法配合使用。
    **使用方法：**
    ````java
        Scroller scroller = new Scroller(mContext);

        //缓慢滚动到指定位置
        private void smoothScrollTo(int destX,int destY){
            int scrollX = getScrollX();
            int delta = destX - scrollX;
            //1000ms 内滑向 destX，效果就是慢慢滑动
            mScroller.startScroll(scrollX,0,dalta,0,1000);
            invalidate();
        }

        @override
        public void computeScroll(){
            if(mScroller.computeScrollOffset()){
                scrollTp(mScroller.getCurrX(),mScroller.getCurrY());
                postInvalidate();
            }
        }
    ````

## View 的滑动
1. 通过三种方式可以实现 View 的滑动：
    - 通过 View 本身提供的 scrollTo/scrollBy 方法来实现滑动。
        **mScrollX 和 mScrollY 的改变规则：**
            在滑动的过程中，mScrollX 的值总是等于 View 左边缘和 View  内容左边缘在水平方向的距离，mScrollY 的值总是等于 View 上边缘和 View 的内容上边缘在竖直方向的距离。View 边缘是指 View 的位置，由四个顶点组成，View 的内容边缘是指 View 中的内容的边缘，scrollTo 和 scrollBy 只能改变 View 内容的位置而不能改变 View 在布局中的位置，mScrollX 和 mScrollY 的单位为像素，当 View 左边缘在  View 内容边缘的右边时，mScrollX 为正值，反之为负值;当 View 上边缘在 View 内容上边缘的下边时，mScrollY 为正值，反之为负。也就是说如果从左往右滑动，那么 mScrollX 为负值，反之为正值;如果从上往下滑动，那么 mScrollY 为负值，反之为正值。

        使用 scrollTo 和 scrollBy 来实现 View 的滑动，只能将 View 的内容进行移动，并不能将 View 本身进行移动，也就是说，不管怎么滑动，也不可能将当前 View 滑动到附近 View 所在的区域。

    - 通过动画给 View 施加平移效果来实现滑动。 
        通过动画能够让一个 View 进行平移，平移是一种滑动，使用动画来移动 View ,主要操作 View 的translationX 和 translationY 属性，既可以采用传统的 View 动画，也可以采用属性动画。
        使用动画来做 View 的滑动需要注意一点，View 动画是对 View 的影像做操作，并不能真正改变 View 的位置参数，包括宽/高，如果希望动画后的状态得以保留还必须将 fillAfter 属性设置为 true，否则动画完成后其动画结果会消失。使用属性动画不存在这种问题，但是3.0以下无法使用属性动画。
    - 通过改变 View 的 LayoutParams 使得 View 重新布局从而实现滑动。
        ````java
            MarginLayoutParams params = (MarginLayoutParams) mButton1.getLayoutParams();
            params.width += 100;
            params.leftMargin += 100;
            mButton1.requestLayout();
            //或者 mButton1.setLayoutParams(params);
        ````
2. 各种滑动方式的对比

    - scrollTo/scrollBy：操作简单，适合对 View 内容的滑动;
    - 动画：操作简单，主要适用于没有交互的 View 和实现复杂的动画效果;
    - 改变布局参数：操作稍微负责，适用于没有交互的 View。

## 弹性滑动
1. Scoller 如何让 View 弹性滑动的
    invalidate 方法会导致 View 重绘，在 View 的 draw 方法中又会去调用 computeScroll 方法，computeScroll 方法会在 View 中是一个空实现，因为这个 computeScroll 方法， View 才能实现弹性滑动。
2. Scroller 的工作原理
    Scroller 本身并不能实现 View 的滑动，需要配合 View 的 computeScroll 方法才能完成弹性滑动的效果，它不断让 View 重绘，每一次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔 Scroller 就可以得出 View 当前的滑动位置，知道了滑动位置就可以通过 scrollTo 方法来完成 View 的滑动。就这样，View 的每一次重绘都会导致 View 进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动。
3. 延时策略的核心思想是通过发送一系列延时消息从而达到一种渐进式的效果，具体来说可以用 Handler 和 View 的 postDelayed 方法，也可以使用线程的 sleep 方法。

## View 的事件分发机制
1. 所谓点击事件的事件分发，其实就是对 MotionEvent 事件的分发过程，即当一个 MotionEvent 产生了以后，系统需要把这个时间传递给一个具体的 View ,而这个传递的过程就是分发过程。
    点击事件的分发过程由三个方法共同完成。

    - public boolean dispatchTouchEvent(MotionEvent ev)
        用来进行事件的分发。如果事件能够传递给当前 View ,那么此方法一定会被调用，返回结果受当前 View 的 onTouchEvent 和下级 View 的 dispatchTouchEvent 方法的影响，表示是否消耗当前事件。
    - public boolean onInterceptTouchEvent(MotionEvent event)
        在上述方法内部调用，用来判断是否拦截某个事件，如果当前 View 拦截了某个事件，那么在同一个事件序列中，此方法不会被再次调用，返回结果表示是否拦截当前事件。
    - public boolean onTouchEvent(MotionEvent event)
        在 dispatchTouchEvent 方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前 View 无法再次接收到事件。

    **用伪代码表示此三者的关系：**
    ````java
        public boolean dispatchTouchEvent(MotionEvent ev){
            boolean consume = false;
            if(onInterceptTouchEvent(ev)){
                consume = onTouchEvent(ev);
            }else{
                consume = child.dispatchTouchEvent(ev);
            }
            return consume;
        }
    ````
    **传递规则：**
    对于一个根 ViewGroup 来说，点击事件产生后，首先会传递给它，这是它的 dispatchTouchEvent 就会被调用，如果这个 ViewGroup 的 onInterceptTouchEvent 方法返回 true 就表示它要拦截当前事件，接着事件就会交个这个 ViewGroup 处理，即它的 onTouchEvent 方法就会被调用;如果 ViewGroup 的 onInterceptTouchEvent 返回 false 就表示它不拦截当前事件，这时当前事件就会继续传递给它的子元素，接着子元素的 dispatchTouchEvent 方法就会被调用，如此反复我都直到事件被最终处理。

    当一个 View 需要处理事件时，如果设置了 OnTouchListener，那么 OnTouchListener 中的 onTouch 方法会被调用，这时事件如何处理还要看 onTouch 的返回值，如果返回 false，则当前 View 的 onTouchEvent 方法会被调用，如果返回 true，那么 onTouchEvent 方法将不会被调用。所以，给 View 设置的 OnTouchListener 其优先级比 onTouchEvent 要高，onClickListener 的优先级最低。

    当一个事件产生后，它的传递过程遵循如下顺序：Activity -> Window -> View ，即事件总是先传递给 Activity，Activity 再传递给 Window，最后 Window 再传递给顶级 View，顶级 View 接收到事件后，就会按照事件分发机制去分发事件。

2. 当 ViewGroup 决定拦截事件后，那么后续的点击事件将会默认交给它处理并且不再调用它的 onInterceptTouchEvent 方法，FLAG_DISALLOW_INTERCEPT 这个标志的作用是让 ViewGroup 不再拦截事件，但前提是 ViewGroup 不拦截 ACTION_DOWN 事件。

## View 的滑动冲突
1. 常见的滑动冲突场景分为三种
    
    - 场景1 --- 外部滑动方向和背部滑动方向不一致;
    - 场景2 --- 外部滑动方向和内部滑动方向一致;
    - 场景3 --- 上两种情况的嵌套。

2. 处理规则
    
    - 对于场景1，处理规则是：当用户左右滑动时，需要让外包的 View 拦截点击事件，当用户上下滑动时，需要让内部 View 拦截点击事件。具体来说就是根据滑动是水平滑动还是竖直滑动来判断到底由谁来拦截事件。
    - 对于场景2，一般都能在业务上找到突破点。
    - 对于场景3，一般都能在业务上找到突破点。

3. 滑动冲突的解决方式
    
    - 外部拦截法
        是指点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，不需要就不拦截。外部拦截法需要重写父容器的 onInterceptTouchEvent 方法，在内部做相应的拦截即可。
    - 内部拦截法
        是指父容器不拦截任何事件，所有事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交由父容器进行处理，需要配合 requestDisallowInterceptTouchEvent 方法才能正常工作，需要重写子元素的 dispatchTouchEvent 方法。