# Android动画之Frame Animation
Frame动画是一系列图片按照一定的顺序展示的过程，和放电影的机制很相似，我们称为逐帧动画。Frame动画可以被定义在XML文件中，也可以完全编码实现。

## 被定义在XML文件中
如果是被定义在 XML 中，可以放置在 /res 下的 anim 或 drawable 目录中 (/res/[anim|drawable]/filename.xml)，文件名可以作为资源 ID 在代码中引用，
语法如下：
````xml
	<?xml version="1.0" encoding="utf-8"?>  
	<animation-list xmlns:android="http://schemas.android.com/apk/res/android"  
    	android:oneshot=["true" | "false"] >  
    	<item  
        	android:drawable="@[package:]drawable/drawable_resource_name"  
        	android:duration="integer" />  
	</animation-list> 
````
**注意：**

<animation-list\> 元素是必须的，并且必须作为根元素，可以包含一个或多个 <item\> 元素；android:onshot 如果定义为 true 的话，此动画只会执行一次，如果为 false 则一直循环。

<item\> 代表一帧动画，android:drawable 指定此帧动画所对应的图片资源，android:duration 代表此帧持续的时间，整数，单位为毫秒。

**问题：**

为什么在 onCreate() 方法中执行动画会失败？
> 这种现象是因为当我们在 onCreate 中调用 AnimationDrawable 的 start 方法时，窗口 Window 对象还没有完全初始化，AnimationDrawable 不能完全追加到窗口 Window 对象中，那么该怎么办呢？我们需要把这段代码放在 onWindowFocusChanged 方法中，当 Activity 展示给用户时，onWindowFocusChanged 方法就会被调用，我们正是在这个时候实现我们的动画效果。当然，onWindowFocusChanged 是在 onCreate 之后被调用的。






##由编码实现
如果完全由编码实现，我们需要用到 AnimationDrawable 对象。