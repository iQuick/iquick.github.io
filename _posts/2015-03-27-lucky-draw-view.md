---
layout:		post
title:		九宫格抽奖控件 LuckyDrawView
tags:		[Android]
published:	true
---
# 九宫格抽奖控件 LuckyDrawView
---

今天正好需要写一个九宫格抽奖的部分，在网上搜索了一番，感觉使用起来不太方便，于是便自己动手写了一个。

![LuckyDrawView]({{ BASE_PATH }}/img/post/lucky-draw-view/lucky-draw-view.jpg)

<!--break-->

## 使用方法

直接 xml 中使用

```xml
<cn.uue.yixiaoba.widget.luckydraw.LuckyDrawView
    android:id="@+id/ldv_campaign_luck_draw"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:prizeBackgroud="@drawable/lucky_draw_item_selector"
    app:startBackgroud="@drawable/lucky_draw_btn_selector"
    app:startSrc="@drawable/ic_launcher" />
```

代码中使用

```java
LuckyDrawView luckyDrawView = new LuckyDrawView(getActivity());
luckyDrawView.setStartResource(R.drawable.ic_launcher);
luckyDrawView.setStartBackgroudResource(R.drawable.lucky_draw_btn_selector);
luckyDrawView.setPrizeBackgroudResource(R.drawable.lucky_draw_item_selector);
luckyDrawView.setOnLuckyDrawListener(new LuckyDrawView.OnLuckyDrawListener() {
	
	@Override
	public void stop() {
		// TODO Auto-generated method stub
		
	}
	
	@Override
	public void start() {
		// TODO Auto-generated method stub
		
	}
});
```

停止方法

```
# 需要停止时，只需要调用 stop(int) 方法，传入停止位置即可
luckyDrawView.stop(4);
```


## 注
这里有个小Bug，在 xml 中 app:prizeBackgroud="@drawable/lucky_draw_item_selector" 这属性有些问题，点击开始后，跳下一个奖品里，上一个奖品的选中状态没有取消，原因暂时不清楚。但是不影响使用，大家只要不用那个属性就可以了。

## Github
[LuckyDraw](https://github.com/iQuick/LuckyDraw)