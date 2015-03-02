---
layout:		post
title:		下拉刷新PullListView
category:	[Android]
tags:		[Android]
published:	true
---
# 下拉刷新PullListView
---

最近由于需要一个下拉刷新、上拉加载更多的组件，网上许多不同时支持自动加载和上拉加载更多，于是就自己动手写了个PullListView

![PullListView]({{ BASE_PATH }}/images/pulllistview.gif)

<!--break-->

## 使用
用法与平常的ListView一样
<pre class="prettyprint linenums">
mListView = (PullListView) findViewById(R.id.listview);
mAdapter = new DataAdapter(this);
mListView.setAdapter(mAdapter);
mListView.setPrestrain(true); // 设置预加载
mListView.setOnPullListViewListener(this); // 设置刷新/加载监听
mListView.setTheEnd(); // 设置已经到底
</pre>

## 特色

* 增加了预加载功能
* 设置到底（即不再加载更多）

## Github:
[https://github.com/iQuick/PullListView](https://github.com/iQuick/PullListView)