# Android 动画之 Property Animation

属性动画是控制属性来实现动画。最为强大的动画，弥补了补间动画的缺点，实现位置+视觉的变化。并且可以自定义插值器，实现各种效果

## Property Animation相关类
|类名|用途|
|----|:-|
|ValueAnimator|属性动画主要的计时器，也计算动画后的属性的值，动画的执行类|
|ObjectAnimator|ValueAnimator的一个子类，允许你设置一个目标对象和对象的属性进行动画，动画的执行类|
|AnimatorSet|提供组织动画的结构，使它们能相关联得运行，用于控制一组动画的执行|
|AnimatorInflater|用户加载属性动画的xml文件|
|Evaluators|属性动画计算器，告诉了属性动画系统如何计算给出属性的值|
|Interpolators|动画插入器，定义动画的变化率|

关系如下：

![](http://www.lightskystreet.com/img/propertyviewanalysis/12.png)

## 被定义在XML文件中
如果是被定义在 XML 中，需要注意是的属性动画文件存放目录为 res/animator，文件名可以作为资源 ID 在代码中引用，

xml 代码：

````xml
	
<set 
	android:ordering=["together" | "sequentially"]>
    <objectAnimator
        android:propertyName="string"
        android:duration="int"
        android:valueFrom="float | int | color"
        android:valueTo="float | int | color"
        android:startOffset="int"
        android:repeatCount="int"
        android:repeatMode=["repeat" | "reverse"]
        android:valueType=["intType" | "floatType"]/>
    <animator
        android:duration="int"
        android:valueFrom="float | int | color"
        android:valueTo="float | int | color"
        android:startOffset="int"
        android:repeatCount="int"
        android:repeatMode=["repeat" | "reverse"]
        android:valueType=["intType" | "floatType"]/>

    <set>
        ...
    </set>
</set>
````
**注意：** XML 文件的根元素必须为 <set\> 或者 <objectAnimator\>, or <valueAnimator\>。你也可以在一个 set 中放置不同的动画，来嵌套其他元素。

### 元素介绍
动画集合节点，有个 ordering 属性，表示它的子动画启动方式是先后有序还是同时。

|属性|说明|
|-|-|
|sequentially|动画按照先后顺序|
|together(default)|动画同时启动|

### <objectAnimator\>
属性：

- android:propertyName：String类型，必须要设定的值，代表要执行动画的属性，通过名字引用，比如你可以指定了一个View的”alpha” 或者 backgroundColor”，这个objectAnimator元素没有暴露target属性，因此比不能够在XML中执行一个动画，必须通过调用loadAnimator() 填充你的XML动画资源，并且调用setTarget() 应用到拥有这个属性的目标对象上。
- android:valueTo：Float、int或者color，也是必须值，表明了动画结束的点，颜色由6位十六进制的数字表示。
- android:valueFrom：相对应valueTo，动画的起始点，如果没有指定，系统会通过属性身上的get方法获取，颜色也是6位十六进制的数字表示。
- android:duration：动画的时长，int类型，以毫秒为单位，默认为300毫秒。
- android:startOffset：动画延迟的时间，从调用start方法后开始计算，int型，毫秒为单位，
- android:repeatCount：一个动画的重复次数，int型，”-1“表示无限循环，”1“表示动画在第一次执行完成后重复执行一次，也就是两次，默认为0，不重复执行。
- android:repeatMode：重复模式：int型，当一个动画执行完的时候应该如何处理。该值必须是正数或者是-1，
“reverse”会使得按照动画向相反的方向执行，可实现类似钟摆效果。“repeat”会使得动画每次都从头开始循环。
- android:valueType：关键参数，如果该value是一个颜色，那么就不需要指定，因为动画框架会自动的处理颜色值。
有intType和floatType两种：分别说明动画值为int和float型。

### <animator\>
在一个特定的时间里执行一个动画。相对应的是 ValueAnimator 所有的属性和一样

- android:valueTo
- android:valueFrom
- android:duration
- android:startOffset
- android:repeatCount
- android:repeatMode
- android:valueType

valueType 的值有两种：

- intType
- floatType(default)
 
举个例子
````xml
	<set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
	</set>
````
为了执行该动画，必须在代码中将该动画资源文件填充为一个AnimationSet对象，然后在执行动画前，为目标对象设置所有的动画集合。
简便的方法就是通过setTarget方法为目标对象设置动画集合，代码如下：
````java
	AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,R.anim.property_animator);
	set.setTarget(myObject);
	set.start();
````
## 由编码实现
如果完全由编码实现，我们需要用到 ObjectAnimator 对象。