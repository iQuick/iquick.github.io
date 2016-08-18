---
layout:		post
title:		Android 沉浸式状态栏与导航栏
category:	[Android]
tags:		[Android]
published:	true
---
# Android 沉浸式状态栏与导航栏
---

Android 4.4 增加了透明状态栏与导航栏的功能，处女座的福音，这个特性个人觉得还是挺不错的，所以这一功能也只能在4.4以上的系统上才有效果。

![沉浸式状态栏与导航栏]({{ BASE_PATH }}/img/post/immerse-status-and-nav/immerse-status-and-nav-1.jpg)


<!--break-->


## 代码添加
```java
if(VERSION.SDK_INT >= VERSION_CODES.KITKAT) {
	// Translucent status bar
	getWindow().setFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS, WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
	// Translucent navigation bar
	getWindow().setFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION, WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
}
```

## 设置theme属性
```java
android:theme="@android:style/Theme.DeviceDefault.Light.NoActionBar.TranslucentDecor"
android:theme="@android:style/Theme.Holo.Light.NoActionBar.TranslucentDecor"
android:theme="@android:style/Theme.Holo.NoActionBar.TranslucentDecor"
```

自定主题，只需在在 values-19 -> style.xml 文件添加以下属性：

```xml
<style name="AppBaseTheme" parent="android:Theme.Holo.Light.DarkActionBar">
	<!-- API 19 theme customizations can go here. -->
	<item name="android:windowTranslucentStatus">true</item>
	<item name="android:windowTranslucentNavigation">true</item>
</style>
```

直接调用上面方法可以透明，但是你会发现你的 view 跑到 actionbar 上面去了，很明显 google 的意图是使你的 view 可以占据整个屏幕，然后 状态栏和导航栏 透明覆盖在上面很明显这样不可行。

那有没有办法使你的 view 保持原来大小呢？

有，你需要在这个 activity 的 layout xml 文件添加两个属性

```java
android:fitsSystemWindows="true"
android:clipToPadding="true"
```

![]({{ BASE_PATH }}/img/post/immerse-status-and-nav/immerse-status-and-nav-2.jpg)

事实证明这样还是很难看，所以有采用了一个最简单，且有效的方法。

就是调用 actionbar 的 setPadding 方法，使其与顶部保持 20dp 左右的距离，效果还是不错的。


## SystemBarTint

[SystemBarTint](https://github.com/jgilfelt/SystemBarTint)

可以设置 statusbar 背景，原理是在 Window 的 DocView 添加 view，大家可以下载这个项目学习如何使用
