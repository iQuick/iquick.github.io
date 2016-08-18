---
layout:		post
title:		Material Designer 的低版本兼容实现
category:	[Android]
tags:		[Android, Material Designer]
published:	false
---
# Material Designer 的低版本兼容实现
---

## Material Designer

宗旨：让不同大小不同用途的设备上拥有同一种设计风格

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-1.png)

<!--break-->

1. 纸张

这种设计模式大量参考了纸墨的模式，将空间变得像纸张一样，而用户的手指就是毛笔。用户按到控件上就会产生墨晕效果。这样的好处是明确的告诉用户是否点击了控件，而且还能让用户一下子明白控件的布局思路。毕竟一张一张的纸叠加起来的控件是很容易让人接受的。这里还有一个词“引喻”，虽然控件像纸张，但是它具有变大变小，改变颜色等能力，所以完全可以不用拘泥于现实纸张。

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-2.gif)

2. 深度

新的设计中希望所有的控件都是现实世界中的隐喻，比如你按下按钮，按钮就应该有被按下的状态，这里就要用到了涟漪（Ripple）效果了。其实涟漪效果是来表示你手指按上去后墨晕扩散的效果的，下面的图能很明白的说明这点。

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-3.gif)

3. 动画

动画贯穿于Material Designer之中，官方文档中用了很大的篇幅来讲解动画效果，希望让设计的动画效果很美观。但我个人认为为了动画而动画是完全不可取的，比如下面的例子

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-4.gif)

4. 排版

新的设计里面很在意排版，里面列出了很多详细的数据来支持我们的设计。对于留白也有了详细的说明。优秀的排班会让你的应用看起来干净，优雅，这点十分重要。在之后的文章中我也会多少说到这方面的知识。

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-5.png)

## 设计文档

* [http://design.1sters.com](http://design.1sters.com/)
* [http://www.ui.cn/Material](http://www.ui.cn/Material/)

## 兼容

### [Material Designer的低版本兼容实现（二）—— Theme](http://www.cnblogs.com/tianzhijiexian/p/4081562.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-6.png)

### [Material Designer的低版本兼容实现（三）—— Color](http://www.cnblogs.com/tianzhijiexian/p/4081888.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-7.png)

### [Material Designer的低版本兼容实现（四）—— ToolBar](http://www.cnblogs.com/tianzhijiexian/p/4082892.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-8.jpg)

### [Material Designer的低版本兼容实现（五）—— ActivityOptionsCompat](http://www.cnblogs.com/tianzhijiexian/p/4087917.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-9.gif)

### [Material Designer的低版本兼容实现（六）—— Ripple Layout](http://www.cnblogs.com/tianzhijiexian/p/4133672.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-10.gif)

### [Material Designer的低版本兼容实现（七）—— Rectange Button](http://www.cnblogs.com/tianzhijiexian/p/4135993.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-11.png)

### [Material Designer的低版本兼容实现（八）—— Flat Button](http://www.cnblogs.com/tianzhijiexian/p/4143709.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-12.png)

### [Material Designer的低版本兼容实现（九）—— Float Button & Small Float Button](http://www.cnblogs.com/tianzhijiexian/p/4146924.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-13.png)

### [Material Designer的低版本兼容实现（十）—— CheckBox & RadioButton](http://www.cnblogs.com/tianzhijiexian/p/4147982.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-14.png)

### [Material Designer的低版本兼容实现（十一）—— Switch](http://www.cnblogs.com/tianzhijiexian/p/4148131.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-15.png)

### [Material Designer的低版本兼容实现（十二）—— Slider or SeekBar](http://www.cnblogs.com/tianzhijiexian/p/4148638.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-16.png)

### [Material Designer的低版本兼容实现（十三）—— ProgressBar](http://www.cnblogs.com/tianzhijiexian/p/4149326.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-17.gif)

### [Material Designer的低版本兼容实现（十四）—— CardView](http://www.cnblogs.com/tianzhijiexian/p/4150557.html)

![material-esigner]({{ BASE_PATH }}/img/post/material-designer-compatible/material-esigner-18.png)

### Material Designer的低版本兼容实现（XX）—— EditText & Typography

* [https://github.com/rengwuxian/MaterialEditText](https://github.com/rengwuxian/MaterialEditText)

### Material Designer的低版本兼容实现（XX）—— Dialog

* MaterialDialog： [https://github.com/drakeet/MaterialDialog](https://github.com/drakeet/MaterialDialog)
* L-Drawer：[https://github.com/ikimuhendis/LDrawer](https://github.com/ikimuhendis/LDrawer)

### Material Designer的低版本兼容实现（XX）—— Drawer

* MaterialNavigationDrawer：[https://github.com/neokree/MaterialNavigationDrawer](https://github.com/neokree/MaterialNavigationDrawer)
* L-Drawer：[https://github.com/ikimuhendis/LDrawer](https://github.com/ikimuhendis/LDrawer)

### Material Designer的低版本兼容实现（XX）—— Animation

* Transitions-Everywhere：[https://github.com/andkulikov/transitions-everywhere](https://github.com/andkulikov/transitions-everywhere)
* Android-UI：[https://github.com/markushi/android-ui](https://github.com/markushi/android-ui)
* CircularReveal：[https://github.com/ozodrukh/CircularReveal](https://github.com/ozodrukh/CircularReveal)

## 参考

* [https://github.com/lightSky/MaterialDesignCenter](https://github.com/lightSky/MaterialDesignCenter)

* [http://blog.csdn.net/xushuaic/article/details/40627389](http://blog.csdn.net/xushuaic/article/details/40627389)

* [http://blog.csdn.net/xushuaic/article/details/40627389](http://blog.csdn.net/xushuaic/article/details/40627389)

