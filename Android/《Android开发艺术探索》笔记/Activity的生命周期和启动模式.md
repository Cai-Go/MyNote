# Activity的生命周期和启动模式
## Activity的生命周期

#### 典型情况下的生命周期(指在有用户参与的情况下，Activity所经过的生命周期的改变)。
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

#### 异常情况下的生命周期(指Activity被系统回收或者由于设备的Configuration发生改变从而导致Activitu被销毁重建)
---

- 资源相关的系统配置发生改变导致Activity被杀死并重新创建。
 >默认情况下，如果Activity不做特殊处理，当系统配置发生改变后，Activity就好被销毁并重新创建，其生命周期如下。
![异常情况下Activity的重建过程](http://7xq2jk.com1.z0.glb.clouddn.com/%E5%BC%82%E5%B8%B8%E6%83%85%E5%86%B5%E4%B8%8BActivity%E7%9A%84%E9%87%8D%E5%BB%BA%E8%BF%87%E7%A8%8B.png)
 onSaveInstanceState方法只会在Activity被异常终止的情况下调用，调用时机是在onStop之前，但和onPause没有既定的时序关系，可能在onPause之前调用，也可能在onPause之后调用。当Activity被重新创建后，系统会调用onRestoreInstanceState,并且把Activity销毁时onSaveInstanceState方法所保存的Bundle对象作为参数同时传递给onRestoreInstanceState和onCreate方法。onRestoreInstanceState方法在onStart方法之后调用。
     - 关于保存和恢复View层次结构，系统工作的流程是：
        + 首先Activity被意外终止时，Activity会调用onSaveInstanceState去保存数据
        + 然后Activity会委托Window去保存数据
        + 接着Window在委托它上面的顶级容器去保存数据，顶级容器是一个ViewGroup，一般来说很可能是一个DecorView
        + 最后顶层容器再去一一通知它的子元素来保存数据
    - 接收的位置选择onRestoreInstanceState或onCreate的区别：
     >onRestoreInstanceState一旦被调用，其参数Bundle savedInstanceState一定是有价值的，不需要额外地判断是否为空；但是onCreate不行吗，onCreate如果正常启动的话，其参数Bundle savedInstanceState为null,所以必须额外判断。通时官方建议建议使用onRestoreInstanceState去恢复数据。
    - 针对onSaveInstanceState方法，需要说明的是系统只会在Activity即将被销毁并且有机会重新显示的情况下才会回去调用。
    
- 资源内存不足导致低优先级的Activity被杀死。

1. Activity按照优先级从高到低，分为如下三种：
    - 前台Activity---正在和用户交互的Activity,优先级最高；
    - 可见但非前台Activity---比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户直接交互；
    - 后台Activity---已经被暂停的Activity，比如执行了onStop，优先级最低。
     >当系统内存严重不足时，系统就会按照上述优先级去杀死目标Activity所在的进程，并在后续通过onSaveInstanceState和onRestoreInstanceState来存储和恢复数据。另外一些后台工作不适合脱离四大组件而独自运行在后台中，这样进程容易被杀死，比较好的方法是将后台工作放入Service中从而保证进程有一定的优先级，从而不会轻易地被系统杀死。

2. 解决当系统配置发生改变后，Activity会被重建的问题
如果当某项内容发生改变后，不想系统重建Activity，可以给Activity指定configChanges属性。
比如不想让Activity在屏幕旋转的时候重新创建，就可以给configChanges属性添加orientation值。
````
android:configChanges = "orientation"
````
系统会去调用Activity的onConfigurationChangesd方法。
3.系统配置的项目和含义
![configChanges的项目和含义](http://7xq2jk.com1.z0.glb.clouddn.com/configchanges%E7%9A%84%E9%A1%B9%E7%9B%AE%E5%92%8C%E5%90%AB%E4%B9%89.jpg)

>注意：
screenSize和smallestScreenSize两个比较特殊，它们的行为和编译选项有关，和运行环境无关。

## Activity的启动模式
1. Android四种启动模式：
    - standard: 标准模式，是系统的默认模式，每次启动一个Activity都会重新创建一个新的实例，即使实例已经存在。创建的实例的生命周期符合典型情况下的Activity生命周期。一个任务栈中可以有多个实例，每个实例可以属于不同的任务栈。谁启动了Activity,那这个Activity,就在启动它的那个Activity所在的栈中运行。当用ApplicationContext去启动standard模式的Activity时就会出错，因为standard模式的Activity默认会进入启动它的Activity所属的任务栈中，但Context属于非Activity类型，并没有所谓的任务栈。解决方法是为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，为它在启动的时候创建一个新的任务栈，但这时候实际是以singleTask模式启动。
     >当我们使用ApplicationContext去启动standard模式的Activity的时候会报错，因为非Activity类型的Context并没有所谓的任务栈，解决的方法就是为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就会创建一个新的任务栈，其实这时候待启动的Activity实际上是以singleTask模式启动的。
    - singleTop: 栈顶复用模式。在这模式下，若新的Activity已经位于任务栈的栈顶，那么此Activity就不会被重新创建，同时它的onNewIntent方法会被回调，我们可以通过此方法取出当前请求的信息。另外，该Activity的onCreat和onStart方法不会被系统调用。如果新Activity的实例已存在，但不位于栈顶，仍然会被重建。
    - singleTask: 栈内复用模式。是一种单实例模式，只要Activity在一个栈中存在，即使多次启动Activity都不会重建实例，系统也会回调其onNewIntent方法。另外，singleTask模式的Activity切换到栈顶会导致它之上的栈内的Activity出栈。
    ![singleTask示例](http://7xq2jk.com1.z0.glb.clouddn.com/singleTask.png)
    - singleInstance: 单实例模式。是一种加强的singleTask模式，具有此种模式的Activity只能单独位于一个任务栈中，并且由于栈内复用的特性，后续的请求均不会创建新的Activity，除非任务栈被系统销毁。