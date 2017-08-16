# View 的工作原理
## ViewRoot 和 DecorView
1. ViewRoot 对应于 ViewRootImpl 类，是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。View 绘制流程是从 ViewRoot 的 performTraversals 方法开始的，经过 measure、layout 和 draw 三个过程才能最终将一个 View 绘制出来，其中 measure 用来测量 View 的宽和高，layout 用来确定 View 在父容器中的放置位置，draw 负责将 View 绘制在屏幕上。

**performTraversals的大致流程**
![performTraversals的大致流程](http://o8fk8z4sl.bkt.clouddn.com/performTraversals%E7%9A%84%E5%A4%A7%E8%87%B4%E6%B5%81%E7%A8%8B.png)
    performTraversals 会依次调用 performMeasure、performLayout 和 preformDraw 三个方法，这三个方法分别完成了顶级 View 的 measure、layout 和 draw 这三大流程，其中 performMeasure 中会调用 measure 方法，在 measure 方法中又回调用 onMeasure 方法，在 onMeasure 方法中则会对所有子元素进行 measure 过程，这个时候 measure 流程就就从父容器传递到子元素中，完成一次 measure 过程，接着子元素会重复父容器的 measure 过程，如此反复就完成了整个 View 树的遍历。