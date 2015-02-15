---
layout:		post
title:		一款好用的文本编辑器
category:	[工具]
tags:		[Sublime Text, 工具]
published:	true
---
# 一款好用的文本编辑器
---

之前编辑文本都是用的Notepad++，个也觉得这个工具也是比较好用的，体积小、功能也较实用，但是最近由于接触jekyll和Ruby才知道的Sublime Text这个同样的轻量级的编辑工具。

Sublime Text 是一个轻量、简洁、高效、跨平台的编辑器，方便的配色以及兼容vim快捷键等各种优点博得了很多前端开发人员的喜爱，最新版本已经更新到了Sublime Text 3了。之前也不并知道它有这么多插件的扩展与支持，直到vincent问到有没有在用cTags插件，才知道原来Sublime通过插件也可以实现一些大型IDE的功能，遂google一下，本篇Blog就来介绍下Sublime下经常使用的插件。

<!--break-->

## 安装
Sublime Text的安装很简单，只要按照安装步骤一步一步来就可以了，这里说下注册问题。

Sublime Text是可以无限试用的，所以不用担心没有注册不能用的问题，没激活的情况下只会在你保存文件时会弹出激活窗口提示你注册。

如果你觉得这样很烦，你也可以使用下面任意一个正的注册码，Sublime Text Build 3065 License key

<pre class="prettyprint linenums">
----- BEGIN LICENSE -----
K-20
Single User License
EA7E-940129
3A099EC1 C0B5C7C5 33EBF0CF BE82FE3B
EAC2164A 4F8EC954 4E87F1E5 7E4E85D6
C5605DE6 DAB003B4 D60CA4D0 77CB1533
3C47F579 FB3E8476 EB3AA9A7 68C43CD9
8C60B563 80FE367D 8CAD14B3 54FB7A9F
4123FFC4 D63312BA 141AF702 F6BBA254
B094B9C0 FAA4B04C 06CC9AFC FD412671
82E3AEE0 0F0FAAA7 8FA773C9 383A9E18
------ END LICENSE ------
----- BEGIN LICENSE -----
J2TeaM
2 User License
EA7E-940282
45CB0D8F 09100037 7D1056EB A1DDC1A2
39C102C5 DF8D0BF0 FC3B1A94 4F2892B4
0AEE61BA 65758D3B 2EED551F A3E3478C
C1C0E04E CA4E4541 1FC1A2C1 3F5FB6DB
CFDA1551 51B05B5D 2D3C8CFE FA8B4285
051750E3 22D1422A 7AE3A8A1 3B4188AC
346372DA 37AA8ABA 6EB30E41 781BC81F
B5CA66E3 A09DBD3A 3FE85BBD 69893DBD
------ END LICENSE ------
</pre>

## 主题
Sublime Text 的主题安装也很简单，只需要将下载的主题解压到packages路径下（也可以在Preferences -> Browes Packages直接打开目录）,并以"Theme - ***"形式命名，最后在Preferences -> Setting-User打开设置在里面添加你的主题，重启Sublime Text即可。

###下面推荐几款主题：
<pre class="prettyprint linenums">
http://www.topthink.com/topic/4407.html
https://github.com/buymeasoda/soda-theme
</pre>

## 插件
###安装包控制（Package Control）
打开Sublime Text，Ctrl + ` 调出控制台Console；

将以下代码粘贴进命令行中并回车：

<pre class="prettyprint linenums">
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
</pre>

重启 Sublime Text，如果在 Preferences -> Package Settings中见到Package Control这一项，就说明安装成功了。

###安装Alignment插件
对于某些喜欢整齐的程序员来说，看到下面这种情况可能是让其无法忍受的：
<pre class="prettyprint linenums">
var joe = 'joe';
var johnny = 'johnny';
var quaid = 'quaid';
</pre>

一定要改成这样才会安心：
<pre class="prettyprint linenums">
var joe    = 'joe';
var johnny = 'johnny';
var quaid  = 'quaid';
</pre>
Sublime Text 之中，一个 Sublime Alignment 插件也可以轻松实现，传说最新版Sublime 已经集成。

1.按下 Ctrl + Shift + P 调出命令面板。

2.输入 install 调出 Package Control: Install Package 选项，按下回车。

3.在列表中找到 Alignment，按下回车进行安装。

4.重启 Sublime Text 使之生效。现在通过选中文本并按 Ctrl + Shift + A 就可以进行对齐操作了。

###Sublime CodeIntel

代码自动提示

###Git
基本实现了Git的所有功能
