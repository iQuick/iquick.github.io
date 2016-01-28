---
layout:		post
title:		浅谈 RxAndroid + Retrofit + Databing
category:	[Android]
tags:		[Android]
published:	true
---
# 浅谈 RxAndroid + Retrofit + Databinding
---

最近 RxAndroid 、MVP、MVVM 一直是 Android 程序猿茶余饭后的谈资，于是我也抱着凑热闹的态度试试了试水。这里就谈谈试水后的感受

## 什么是 RxAndroid ?
要说什么是 RxAndroid ，得从 RxJava 说起。RxJava 在 GitHub 主页上的自我介绍是 "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。这就是 RxJava ，概括得非常精准。

RxJava 的本质可以压缩为异步这一个词。说到根上，它就是一个实现异步操作的库，而别的定语都是基于这之上的。

而RxAndroid是RxJava的一个针对Android平台的扩展，主要用于 Android 开发。

<!--break-->

## 什么是 Retrofit ?
Retrofit 是一套 RESTful 架构的 Android（Java） 客户端实现，基于注解，提供 JSON to POJO（Plain Ordinary Java Object ，简单 Java 对象），POJO to JSON，网络请求（POST，GET， PUT，DELETE 等）封装。

既然只是一个网络请求封装库，现在已经有了那么多的大家已经耳熟能详的网络请求封装库了，为什么还要介绍它呢，原因在于 Retrofit 是一套注解形的网络请求封装库，让我们的代码结构更给为清晰。它可以直接解析JSON数据变成JAVA对象，甚至支持回调操作，处理不同的结果。主要是 Retrofit 能很好的与 RxAndroid 配合使用。

想更详细的了解 Retrofit，可以查看[官方文档](http://square.github.io/retrofit/)

## 什么是 MVVP ?
MVC（Model-View-Controller）和 MVP（Model-View-Presenter）是最常见的软件架构之一，业界有着广泛应用，大家一定不陌生。但知道什么事 MVVP 的就不多了，它本身很容易理解，但是要讲清楚，这几个架构的区别就不容易了。

MVVM（Model-View-ViewModel），它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。Angular 和 Ember 都采用这种模式。

而 Google 新推出的 Databinding 正是采用了这种模式。

## RxAndroid + Retrofit + Databinding
上面已经分别介绍了 RxAndroid、Retrofit、Databinding ，想必大家也有了个初步的认识，那我们就看看 RxAndroid + Retrofit + Databinding 产生的“化学反应”。

<pre>
private void initActionBar() {
    setSupportActionBar(getBinding().toolbar);

    DrawerLayout drawer = getBinding().drawLayout;
    ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(this, drawer, getBinding().toolbar, R.string.navigation_drawer_open, R.string.navigation_drawer_close);
    drawer.setDrawerListener(toggle);
    toggle.syncState();

    getBinding().navigationView.setNavigationItemSelectedListener(this);
}
</pre>

代码中不再充斥着 findViewById 这样的代码了，将 etContentView() 换成下面的方法。

<pre>
this.mBinding = DataBindingUtil.setContentView(context, layout_id);
</pre>

系统会将我们的 layout 和 data 进行绑定并返回 bind 对象，bind.*** 或者 bind.set 方法来取得控件或修改值。当然还有其它的方法，但是你此时再使用 findViewById() 方法不再有效了。


<pre>
public interface NewsApi {

    /**
     * 根据 ID 请求新闻列表
     * @param id
     * @return
     */
    @Headers("apikey: 2c61a1cd1f64216e92f7da1603697bf7")
    @GET(ApiConst.NEWS)
    Observable<News.NewsData> queryNewsByID(@Query("channelId") String id, @Query("page") int page);

    /**
     * 根据 ChannelName (标题)请求新闻列表
     * @param title
     * @return
     */
    @Headers("apikey: 2c61a1cd1f64216e92f7da1603697bf7")
    @GET(ApiConst.NEWS)
    Observable<News.NewsData> queryNewsByCName(@Query("channelName") String title, @Query("page") int page);

    /**
     * 根据 title (标题)请求新闻列表
     * @param title
     * @return
     */
    @Headers("apikey: 2c61a1cd1f64216e92f7da1603697bf7")
    @GET(ApiConst.NEWS)
    Observable<News.NewsData> queryNewsByTitle(@Query("title") String title, @Query("page") int page);

}
</pre>

<pre>
private void initObservables() {
    Observable.Transformer<List<News>, List<News>> networkingIndicator = RxNetworking.bindRefreshing(getBinding().refresher);

    observableRefresherNewsData = Observable.defer(() -> mNewApi.queryNewsByCName(getArguments().getString(BUNDLE_NAME), 1))
            .doOnUnsubscribe(() -> this.unsubcribe("observableNewsData"))
            .flatMap(data -> Observable.just(data.contentlist))
            .flatMap(list -> getApp().getDB().putList(Const.DB_NEWS_NAME, list, News.class))
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .compose(networkingIndicator);

    observableLoadMoreNewsData = Observable.defer(() -> mNewApi.queryNewsByCName(getArguments().getString(BUNDLE_NAME), mCurrPage + 1))
            .doOnUnsubscribe(() -> this.unsubcribe("observableNewsData"))
            .map(data -> {
                mCurrPage = data.currentPage;
                return data.contentlist;
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .compose(networkingIndicator);

    // 刷新/加载更多
    RxSwipeRefreshLayout.refreshes(getBinding().refresher)
            .doOnUnsubscribe(() -> this.unsubcribe("SwipeRefreshLayout"))
            .flatMap(avoid -> observableRefresherNewsData)
            .compose(bindToLifecycle())
            .subscribe(RxList.prependTo(mNews, getBinding().content), this::showError);

    RxEndlessRecyclerView.reachesEnd(getBinding().content)
            .doOnUnsubscribe(() -> this.unsubcribe("Recycler"))
            .flatMap(avoid -> observableLoadMoreNewsData)
            .compose(bindToLifecycle())
            .subscribe(RxList.appendTo(mNews), this::showError);

    // 首次进入手动加载
    observableRefresherNewsData
            .map(list -> {
                mNews.clear();
                return list;
            })
            .compose(bindToLifecycle())
            .subscribe(RxList.prependTo(mNews, getBinding().content), this::showError);

}
</pre>

上面代码是使用 Retrofit 以 Get 形式从服务器中获取对应的新闻数据，大家可以看到代码的逻辑非常清晰，代码也很简洁（这里使用了 lambda 表达式，不使用的话，代码会长些，但是逻辑依然清晰），如果是按以前的写法的话，我们的代码会比这复杂的多，还涉及到复杂的线程之间的通信。而通过 RxJava ，我们只需要简单的使用 subscribeOn(Schedulers.io()) 和 observeOn(AndroidSchedulers.mainThread()) 就可以完成 IO 线程和 UI 线程的切换。

帅的简直不敢相信，原来还可以这样玩。

## 总结

#### 优点：
1. 代码逻辑更多加清晰。
2. 线程之间的切换更加方便、自如。
3. 代码可扩展性高，便于维护。
4. 不再为 findViewById() 方法而烦，为 Activity 减负，整体结构更加清晰。

#### 缺点：
1. 代码出错时，由于 RxJava 的原因，将不太容易找到具体出错位置。
2. 由于 RxJava 结构问题，部分需要捕捉的错误可能被 RxJava 消化掉。
3. Databinding 在部分情况使用不太如意，如 include 进来的 layout 里对应的 id 不会被关联起来。
4. 需要一定的学习成本（当然这不是问题）。


## 广告
这里打个小广告，介绍下我最近开发的几个小应用

* 小白球：[http://www.wandoujia.com/apps/me.imli.whiteball](http://www.wandoujia.com/apps/me.imli.whiteball)
![小白球](http://img.wdjimg.com/mms/screenshot/d/41/cc78a38a603fdce059b0799b0d50341d_320_570.jpeg)

* 私人订制：[http://www.wandoujia.com/apps/me.imli.newme](http://www.wandoujia.com/apps/me.imli.newme)
![私人订制](http://www.wandoujia.com/apps/me.imli.newme)

* 轻截：[http://www.wandoujia.com/apps/me.imli.lightcrop](http://www.wandoujia.com/apps/me.imli.lightcrop)
![轻截](http://img.wdjimg.com/mms/screenshot/3/52/d4dc8ec4ee96aeac453712650669a523_320_569.jpeg)

大家多支持下，如果下载达到 1000 的话，我会将其中一两个项目开源出来的哦。


## 扩展阅读
1. RxJava / RxAndroid
> * 官方 WIKI：[https://github.com/ReactiveX/RxJava/wiki](https://github.com/ReactiveX/RxJava/wiki)
> * 极力推荐的代码家干货：[http://gank.io/post/560e15be2dca930e00da1083](http://gank.io/post/560e15be2dca930e00da1083)

2. Retrofit:
> * Retrofit 官方文档：[http://square.github.io/retrofit/](http://square.github.io/retrofit/)
> * Retrofit 使用介绍：[http://www.cnblogs.com/angeldevil/p/3757335.html](http://www.cnblogs.com/angeldevil/p/3757335.html)
> * Retrofit 离线缓存策略：[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0115/3873.html](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0115/3873.html)

3. Databinding 
> * MVC，MVP 和 MVVM 的图示：[http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)
> * DataBinding 用户指南：[http://segmentfault.com/a/1190000002876984](http://segmentfault.com/a/1190000002876984)
> * Github 上比较全面的：[https://github.com/LyndonChin/MasteringAndroidDataBinding](https://github.com/LyndonChin/MasteringAndroidDataBinding)