---
layout:		post
title:		最近流行的下拉刷新SwipeRefreshLayout
category:	[Android]
tags:		[Android]
published:	true
---
# 最近流行的下拉刷新SwipeRefreshLayout
---

最近看到好多应用都在使用这种下拉刷新，于是自己搜索了一下，原来这玩意儿是Google提供的SwipeRefreshLayout控件，以前也使用过家伙，但是效果不是这个样子的，是下图的样子，现在Google又改变了其效果

<!--break-->

![新的]({{ BASE_PATH }}/images/swiperefreshlayout-1.gif)
![旧的]({{ BASE_PATH }}/images/swiperefreshlayout-2.gif)

## 使用
<pre class="prettyprint linenums">
mHandler = new Handler();

mListView = (ListView) findViewById(R.id.list);
mList = new ArrayList<String>();
for (int i = 0; i < 20; i++) {
	mList.add("item " + i);
}
mAdapter = new ArrayAdapter<String>(this, R.layout.list_item, mList);
mListView.setAdapter(mAdapter);

mSwipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.swl);

// 设置颜色
mSwipeRefreshLayout.setColorSchemeResources(android.R.color.holo_blue_bright,
											android.R.color.holo_green_light,
											android.R.color.holo_orange_light,
											android.R.color.holo_red_light);

mSwipeRefreshLayout.setOnRefreshListener(new OnRefreshListener() {
	
	@Override
	public void onRefresh() {
		mHandler.postDelayed(new Runnable() {
			
			@Override
			public void run() {
				mList.add(0, "new item");
				mAdapter.notifyDataSetChanged();
				mSwipeRefreshLayout.setRefreshing(false);
			}
		}, 2000);
	}
});
</pre>

## Github:
[https://github.com/iQuick/SwipeRefreshLayout](https://github.com/iQuick/SwipeRefreshLayout)