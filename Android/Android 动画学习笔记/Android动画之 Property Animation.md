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