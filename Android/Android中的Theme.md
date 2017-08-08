#Androdi中的Theme
在AndroidManifest.xml文件中有
````
android:theme="@style/AppTheme"
````
这段代码，用来引用res/values/styles.xml 中的主题样式。
````
 <!-- Base application theme. -->
    <style name="AppTheme" parent="android:Theme.Material.Light.NoActionBar" >
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
````
AppTheme的主题样式继承自parent中定义的主题，我们修改parent后的值可以实现不同的主题。

使用android系统中自带的主题要加上“android:”，如：android:Theme.Black
使用v7兼容包中的主题不需要前缀，直接：Theme.AppCompat

下面列举下一些主题
系统自带主题：
````
API 1:
android:Theme 根主题
android:Theme.Black 背景黑色
android:Theme.Light 背景白色
android:Theme.Wallpaper 以桌面墙纸为背景
android:Theme.Translucent 透明背景
android:Theme.Panel 平板风格
android:Theme.Dialog 对话框风格

API 11:
android:Theme.Holo Holo根主题
android:Theme.Holo.Black Holo黑主题
android:Theme.Holo.Light Holo白主题

API 14:
Theme.DeviceDefault 设备默认根主题
Theme.DeviceDefault.Black 设备默认黑主题
Theme.DeviceDefault.Light 设备默认白主题

API 21: (网上常说的 Android Material Design 就是要用这种主题)
Theme.Material Material根主题
Theme.Material.Light Material白主题


兼容包v7中带的主题：
Theme.AppCompat 兼容主题的根主题
Theme.AppCompat.Black 兼容主题的黑色主题
Theme.AppCompat.Light 兼容主题的白色主题
````

Theme.AppCompat主题是兼容主题,是指如果运行程序的手机API是21则就相当于是Material主题，如果运行程序的手机API是11则就相当于是Holo主题，以此类推

---
参考：
[总结一下Android中主题(Theme)的正确玩法](http://www.cnblogs.com/zhouyou96/p/5323138.html)