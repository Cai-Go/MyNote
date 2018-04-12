# Android 自定义 View 的知识点
## Android 中 getWidth() 和 getMeasuredWidth() 之间的区别
getMeasuredWidth() 获取的是 View 原始的大小，也就是这个 View 在 XML 文件中配置或者是代码中设置的大小。getWidth() 获取的是这个 View 最终显示的大小，这个大小有可能等于原始的大小也有可能不等于原始大小。

## View 的坐标系
屏幕的左上角为坐标的原点，屏幕上边缘往右为X轴正方向，屏幕左边缘往下为Y轴正方向。

- View 自身坐标：
    - getLeft()：子 View 的左边缘相对于父 View 左边缘的距离，单位是像素。
    - getTop()：子 View 的右边缘相对于父 View 左边缘的距离，单位是像素。
    - getRight()：子 View 的上边缘相对于父 View 上边缘的距离，单位是像素。
    - getBottom()：子 View 的下边缘相对于父 View 上边缘的距离，单位是像素。
- View 自身宽高： getWidth()，getMeasuredWidth()，getHeight()，getMeasuredHeight()
- MotionEvent 获取坐标：
    - getX()：点击事件的点在控件中的X坐标，即点相对于控件左边缘的距离。
    - getY()：点击事件的点在控件中的Y坐标，即点相对于控件上边缘的距离：
    - getRawX()：点击事件的点在整个屏幕中的X坐标，即点相对于屏幕左边缘的距离。
    - getRawY()：点击事件的点在整个屏幕中的Y坐标，即点相对于屏幕上边缘的距离。
- View 的 Padding：
    - getPaddingLeft()：View 里的 content 距离 View 左边缘的距离。
    - getPaddingTop()：View 里的 content 距离 View 上边缘的距离。
    - getPaddingRight()：View 里的 content 距离 View 右边缘的距离。
    - getPaddingBottom()：View 里的 content 距离 View 下边缘的距离。

![  
自定义 View 的示例](http://7xq2jk.com1.z0.glb.clouddn.com/%E8%87%AA%E5%AE%9A%E4%B9%89%20View%20%E7%9A%84%E7%A4%BA%E4%BE%8B.png)
**图片摘自：http://blog.csdn.net/jason0539/article/details/42743531**


