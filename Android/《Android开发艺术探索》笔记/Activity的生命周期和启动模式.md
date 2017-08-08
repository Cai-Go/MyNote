# Activity的生命周期和启动模式
## Activity的生命周期

####典型情况下的生命周期(指在有用户参与的情况下，Activity所经过的生命周期的改变)。
---
- onCreate:表示Activity正在被创建，是生命周期的第一个方法。这个方法里可以做一些初始化工作。
- onRestart:表示Activity正在重新启动。一般当当前Activity从不可见重新变为可见状态时会调用这个方法。如按下Home键或者打开一个新的Acticity，会执行onPause,onStop方法，接着又回到这个Activity，就会调用这个方法。
- onStar:表示Activity正在被启动，即将开始。此时Activity已经可见，但还没有出现在前台，无法和用户交互。
- onResume:表示Activity已经可见，并且出现在前台并开始活动。
- onPause:表示Activity正在停止，正常情况下会紧接着调用onStop(特殊情况是如果这时候快速回到当前Activity，会调用onResume)。此时可以做一些存储数据，停止动画等工作，但不能太耗时。
- onStop表示Activity即将停止。可以做一些稍微重量级的回收工作，但也不能太耗时。
- onDestroy:表示Activity即将停止，是Activity生命周期中的最后一个回调。可以做一些回收工作和最终的资源释放。
![Activity生命周期的切换过程](http://7xq2jk.com1.z0.glb.clouddn.com/Activity%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%9A%84%E5%88%87%E6%8D%A2%E8%BF%87%E7%A8%8B.jpg)

附加说明：
---
- 针对一个特定的Activity，第一次启动，回调如下:onCreate→onStart→onResume。
- 当用户打开新的Activity或者切换到桌面，回调如下:onPause→onStop。有一种特殊情况是如果新的Activity采用了透明主题，那么当前的Activity不会回调onStop。
当用户再次回到原Activity时，回调如下:onRestart→onStart→onResume。
- 当按Back键回退时，回调如下:onPause→onStop→onDestory。
- 当Activity被系统回收后再次打开，生命周期方法回调过程和正常一样，但只是生命周期方法一样，不代表所有过程都一样。
- 从整个生命周期来说，onCreate和onDestroy是配对的，分别标志着Actovity的创建和销毁，并且只可能有一次调用。从Activity是否可见来说，onStart和onStop是配对的，随着用户的操作或者设备屏幕的点亮和熄灭，这两个方法可能被调用多次；从Activity是否在前台来说，onResume和onPause是配对的，随着用户的操作或者设备屏幕的点亮和熄灭，这两个方法可能被调用多次。

####异常情况下的生命周期(指Activity被系统回收或者由于设备的Configuration发生改变从而导致Activitu被销毁重建)
---