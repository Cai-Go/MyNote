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

4. TouchSlop 是系统所能识别出的被认为是滑动的最小的距离，这是一个常量，和设备有关，可以通过**ViewConfiguration.get(getContext()).getScaledTouchSlop()**来获取这个常量。

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