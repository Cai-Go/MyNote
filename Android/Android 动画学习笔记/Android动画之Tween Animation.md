# Android 动画之 Tween Animation
Tween动画是操作某个控件让其展现出旋转、渐变、移动、缩放的这么一种转换过程，我们成为补间动画。我们可以以XML形式定义动画，也可以编码实现。
## 以XML形式定义动画
如果以XML形式定义一个动画，我们按照动画的定义语法完成XML，并放置于/res/anim目录下，文件名可以作为资源ID被引用；
````xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android"  
    android:interpolator="@[package:]anim/interpolator_resource"  
    android:shareInterpolator=["true" | "false"] >  
    <alpha  
        android:fromAlpha="float"  
        android:toAlpha="float" />  
    <scale  
        android:fromXScale="float"  
        android:toXScale="float"  
        android:fromYScale="float"  
        android:toYScale="float"  
        android:pivotX="float"  
        android:pivotY="float" />  
    <translate  
        android:fromX="float"  
        android:toX="float"  
        android:fromY="float"  
        android:toY="float" />  
    <rotate  
        android:fromDegrees="float"  
        android:toDegrees="float"  
        android:pivotX="float"  
        android:pivotY="float" />  
    <set>  
        ...  
    </set>  
</set>  
````
xml 文件中必须有一个根元素，可以是<alpha\>、<scale\>、<translate\>、<rotate\>中的任意一个，也可以<set\>来管理一个由前面几个组成的动画的集合。

### <set\>
<set\> 是一个动画容器，管理多个动画的群组，与之相对性的 Java 对象是 AnimationSet。有两个属性：

- android:interpolator ---- 代表一个插值器资源，可以引用系统自带查之前资源，也可以用自定义插值器资源，默认值是匀速插值器。
- android:shareInterpolator ---- 代表多个动画是否要共享插值器，默认值为 true，即共享插值器；如果设置为 false，那么插值器就不再起作用，需要在每个动画中加入插值器。

### <alpha\>
<alpha\> 是渐变动画，可以实现 fadeIn 和 fadeOut 的效果，与之对应的 Java 对象是 AlphaAnimation。

- android:fromAlpha 属性代表起始 alpha 值，浮点值，范围在 0.0-1.0 之间，分别代表透明和完全不透明。
- android:toAlpha 属性代表结尾 alpha 值，浮点值，范围在 0.0-1.0 之间。

### <sacle\>
<scale\> 是缩放动画，可以实现动态调整控件尺寸的效果，与之对应的 Java 对象是 ScaleAnimation。

- android:fromXScale 属性代表起始的 X 方向上相对自身的缩放比例，浮点值。1.0代表自身无变化，0.5代表起始时缩小一倍，2.0代表放大一倍。
- android:toXScale 属性代表结尾的X方向上相对自身的缩放比例，浮点值。
- android:fromYScale 属性代表起始的Y方向上相对自身的缩放比例，浮点值。
- android:toYScale 属性代表结尾的Y方向上相对自身的缩放比例，浮点值。
- android:pivotX 属性代表缩放的中轴点X坐标，浮点值。
- android:pivotY 属性代表缩放的中轴点Y坐标，浮点值，

如果我们想表示中轴点为图像的中心，我们可以把 **android:pivotX** 和 **android:pivotY** 两个属性值定义成0.5或者50%。

### <translate\>
<translate\> 是位移动画，代表一个水平、垂直的位移。

- android:fromXDelta 属性代表起始X方向的位置；
- android:toXDelta 代表结尾X方向上的位置；
- android:fromYScale 属性代表起始Y方向上的位置；
- android:toYDelta 属性代表结尾Y方向上的位置。

以上四个属性都支持三种表示方式：浮点数、num%、num%p；

- 如果以浮点数字表示，代表相对自身原始位置的像素值；
- 如果以 num% 表示，代表相对于自己的百分比，比如 toXDelta 定义为100%就表示在 X 方向上移动自己的1倍距离；
- 如果以 num%p 表示，代表相对于父类组件的百分比。

### <rotate\>
<rotate\> 是旋转动画，与之对应的 Java 对象是 RotateAnimation。

- android:fromDegrees 属性代表起始角度，浮点值，单位：度；
- android:toDegrees 属性代表结尾角度，浮点值，单位：度；
- android:pivotX 属性代表旋转中心的X坐标值，
- android:pivotY 属性代表旋转中心的Y坐标值，

这两个属性也有三种表示方式:

- 数字方式代表相对于自身左边缘的像素值，
- num% 方式代表相对于自身左边缘或顶边缘的百分比，
- num%p 方式代表相对于父容器的左边缘或顶边缘的百分比。

另外，在动画中，如果我们添加了android:fillAfter="true"后，这个动画执行完之后保持最后的状态；android:duration="integer"代表动画持续的时间，单位为毫秒。

如果要把定义在XML中的动画应用在一个ImageView上，代码是这样的：
````xml
	ImageView image = (ImageView) findViewById(R.id.image);  
	Animation testAnim = AnimationUtils.loadAnimation(this, R.anim.test);  
	image.startAnimation(testAnim);  
````

### 插值器
在补间动画中，我们一般只定义关键帧（首帧或尾帧），然后由系统自动生成中间帧，生成中间帧的这个过程可以成为“插值”。插值器定义了动画变化的速率，提供不同的函数定义变化值相对于时间的变化规则，可以定义各种各样的非线性变化函数，比如加速、减速等。下面是几种常见的插值器：

|Interpolator对象|资源ID|功能作用|
|--|:-:|:-:|
|AccelerateDecelerateInterpolator|@android:anim/accelerate_decelerate_interpolator|	先加速再减速|
|AccelerateInterpolator|@android:anim/accelerate_interpolator|加速|
AnticipateInterpolator|@android:anim/anticipate_interpolator|先回退一小步然后加速前进|
AnticipateOvershootInterpolator|@android:anim/anticipate_overshoot_interpolator|在上一个基础上超出终点一小步再回到终点
|BounceInterpolator|@android:anim/bounce_interpolator最后阶段弹球效果|
|CycleInterpolator|@android:anim/cycle_interpolator|周期运动|
|DecelerateInterpolator|@android:anim/decelerate_interpolator|减速|
|LinearInterpolator|@android:anim/linear_interpolator匀速|
|OvershootInterpolator|@android:anim/overshoot_interpolator|快速到达终点并超出一小步最后回到终点|

#### 使用插值器：
````xml
	<set android:interpolator="@android:anim/accelerate_interpolator">  
		...  
	</set> 
````

````xml
	<alpha android:interpolator="@android:anim/accelerate_interpolator"  
    .../> 
````

#### 修改插值器
有时候会不满足现有的插值器，就可以试试个性化插值器。我们可以创建 XML 资源文件，然后将其放于 /res/anim 下，然后在动画元素中引用。
几种常见的插值器可调整的属性：

- <accelerateDecelerateInterpolator\> 无；
- <accelerateInterpolator\> android:factor 浮点值，加速速率，默认为1；
- <anticipateInterploator\> android:tension 浮点值，起始点后退的张力、拉力数，默认为2；
- <anticipateOvershootInterpolator\> android:tension 同上 android:extraTension浮点值，拉力的倍数，默认为1.5（2  * 1.5）；
- <bounceInterpolator\> 无；
- <cycleInterplolator\> android:cycles 整数值，循环的个数，默认为1；
- <decelerateInterpolator\> android:factor 浮点值，减速的速率，默认为1；
- <linearInterpolator\> 无；
- <overshootInterpolator\> 浮点值，超出终点后的张力、拉力，默认为2；

举个例子：
````xml
	<?xml version="1.0" encoding="utf-8"?>  
	<overshootInterpolator xmlns:android="http://schemas.android.com/apk/res/android"  
    	android:tension="7.0"/>  
````
然后引用；
````xml
	<scale xmlns:android="http://schemas.android.com/apk/res/android"  
    	android:interpolator="@anim/my_overshoot_interpolator"  
    .../>  
````

我们也可以实现 Interpolator 接口，因为所有的 Interpolator 都实现了 Interpolator 接口，这个接口定义了一个方法：
> float getInterpolation(float input); 

此方法由系统调用，input 代表动画的时间，在0和1之间，也就是开始和结束之间。
举个例子：

性（匀速）插值器定义如下：
````java
	public float getInterpolation(float input) {  
    	return input;  
	}  
````
加速减速插值器定义如下：
````java
	public float getInterpolation(float input) {  
    	return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;  
	}  

````


## 由编码实现动画
如果由编码实现，我们需要使用到 Animation 对象。与之对应的Java对象是TranslateAnimation。
