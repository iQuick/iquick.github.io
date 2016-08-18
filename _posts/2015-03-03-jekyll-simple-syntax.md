---
layout:		post
title:		Jekyll简单语法
category:	[Jekyll]
tags:		[Jekyll]
published:	true
---
# Jekyll简单语法
---

模板语法实际上分两部分, 一部分是头部定义,另一部分是语法.

## 头部定义
<pre>
---
layout:     post
title:      title
category: blog
description: description
published: true # default true
---
</pre>

<!--break-->

## 语法

### 使用变量

所有的变量是都一个树节点, 比如模板中定义的头部变量,需要使用下面的语法获得
<pre>
page.title
</pre>

具体变量请参照：[Jekyll简介]({{ BASE_PATH }}/2015/03/03/jekyll-introduce)

### 字符转义
有时候想输出 { 了,怎么办,使用 \ 转义即可.
<pre>
\{ => {
</pre>

**注** 去掉“{ {”或“{ %”中的空格，下列如同

### 输出变量
输出变量直接使用两个大括号括起来即可.
<pre>
{ { page.title } }
</pre>

### 循环
和平常的解释性语言很想.
<pre>
{ % for post in site.posts % }
	<a href="http://blog/2014/11/10/jekyll-study/{ { post.url } }">{ { post.title } }</a>
{ % endfor % }
</pre>

### 自动生成摘要
<pre>
{ % for post in site.posts % }
	{ { post.url } } { { post.title } }
	{ { post.excerpt | remove: 'test' } }
{ % endfor % }
</pre>

### 删除指定文本
remove 可以删除变量中的指定内容
<pre>
{ { post.url | remove: 'http' } }
</pre>

### 删除 html 标签
这个在摘要中很有用.
<pre>
{ { post.excerpt | strip_html } }
</pre>

### 代码高亮
<pre>
{ % highlight ruby linenos % }
\# some ruby code
{ % endhighlight % }
</pre>


### 数组的大小
<pre>
{ { array | size } }
</pre>

### 赋值
<pre>
{ % assign index = 1 % }
</pre>

### 格式化时间
<pre>
{ { site.time | date_to_xmlschema } } 2014-01-07T13:07:54-08:00
{ { site.time | date_to_rfc822 } } Mon, 01 Nov 2014 13:07:54 -0800
{ { site.time | date_to_string } } 01 Nov 2014
{ { site.time | date_to_long_string } } 01 November 2014
</pre>

### 搜索指定key
<pre>
# Select all the objects in an array where the key has the given value.
{ { site.members | where:"graduation_year","2014" } }
</pre>

### 排序
<pre>
{ { site.pages | sort: 'title', 'last' } }
</pre>

### to json
<pre>
{ { site.data.projects | jsonify } }
</pre>

### 序列化
把一个对象变成一个字符串
<pre>
{ { page.tags | array_to_sentence_string } }
</pre>

### 单词的个数
<pre>
{ { page.content | number_of_words } }
</pre>

### 指定个数
得到数组指定范围的结果集
<pre>
{ % for post in site.posts limit:20 % }
</pre>

### 内容名字规范
对于博客,名字必须是 YEAR-MONTH-DAY-title.MARKUP 的格式.如：
<pre>
2015-02-12-talk-nonsense.md
2015-02-13-sublime-text.md
2015-02-14-volley-manager.md
</pre>