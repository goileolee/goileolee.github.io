---
layout: post
title:  "基于MVP，集成了RxJava+Retrofit+TabLayout+ButterKnife的天气类App的实现"
category: decompilation
date:   2018-02-07 14:25:00
categories: Android

---


>转载请注明出处：https://juejin.im/post/5ad986656fb9a07a9a106726

GitHub 地址：[https://github.com/goileolee/legleg](https://github.com/goileolee/legleg)

## legleg 是什么？

legleg-腿腿，是一个基于 MVP 结构，集成了 RxJava、Retrofit、TabLayout、ButterKnife 开源库，而实现一个天气预报查询类 APP。

![镇](https://user-gold-cdn.xitu.io/2018/5/17/1636d807233cbb86?w=290&h=188&f=png&s=48093)

## 为什么叫 legleg？

虽然这是个天气预报 APP，但创建工程的目的是为了学习如何将  
MVP + RxJava + Retrofit + TabLayout + ButterKnife 集成在一起使用。

笔者认为以上开发模式和开源库就像是两条腿一样有力，它们的组合为开发项目提供了充足的动力，使得整个开发更加便捷。

## 项目结构

![项目结构](https://user-gold-cdn.xitu.io/2018/4/20/162e1c75533cd1db?w=273&h=461&f=png&s=4317)

包名以下是：  
- base —— 一些全局使用的工具类和 Application等。
    - adapter —— 适配器类
    - util —— 工具类
- http —— 与网络请求相关类
- mvp  
    - model —— 数据模型
    - presenter —— 处理事务
    - view 这里的 view 用来存放 V 与 P 之间的接口文件
- ui
    - ac —— Activity 类
    - customViews —— 自定义视图类
    - fragment —— Fragment 类
    - dialog —— Dialog 类

## 使用

#### MVP

legleg 以天气预报接口为例，实现在 Fragment 中与 Presenter 绑定并发送网络请求。

这里的示例比较简单，在实际的项目中会有更多的事务处理，都可以在 MainView 中加接口方法来实现。

另外在 Activity 和 Dialog 中绑定 Presenter，工程中没有写出来，但实现方式和 Fragment 相同，原理即将 V 和 P 互相绑定。

V - MvpFragment, WeatherFragment  
P - BasePresenter, WeatherPresenter

V 得到 P 的对象，mvpPresenter：
```
public abstract class MvpFragment<P extends BasePresenter> extends BaseFragment {
    protected P mvpPresenter;

    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        mvpPresenter = createPresenter();
    }

    protected abstract P createPresenter();
```

```
public class WeatherFragment extends MvpFragment<WeatherPresenter> implements MainView<RespBean> {

@Override
    protected WeatherPresenter createPresenter() {
        return new WeatherPresenter(this);
    }
```

可以看到是在父类的抽象方法中直接 new 出来的。  
并传了一个 this 对象过去。因为 WeatherFragment 实现了 MainView 接口，所以传过去的 this，是一个 MainView 对象。

P 得到 V 的对象，mvpView：

```
public class WeatherPresenter extends BasePresenter<MainView> {

    public WeatherPresenter(MainView mainView){
        attachView(mainView);
    }
```

```
public class BasePresenter<V> {

    public V mvpView;

    public void attachView(V mvpView) {
        this.mvpView = mvpView;
        apiStores = ApiClient.retrofit().create(ApiStores.class);
    }
```

#### RxJava

在 legleg 工程中，RxJava 仅被用作网络请求，但 RxJava 的威力远不止于此。

工程中用到 RxJava 的地方在 BasePresenter 类中
```
observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(observer);
```

- observable —— 被观察者，即网络请求发起者；
- subscribeOn —— 设置被观察者在什么线程运行；
- Schedulers.io() —— 用于耗时操作，如读写文件，数据库存取，网络通信等。这个调度器根据需要，增加或者减少线程池中的线程数量。需要注意的是 Schedulers.io() 中的线程池数量是无限制大的，大量的 I/O 操作将创建许多线程，我们需要在性能和线程数量中做出取舍。
- observeOn —— 设置观察者在什么线程运行；
- AndroidSchedulers.mainThread() —— 主线程；
- subscribe —— 订阅，即网络请求订阅回调；
- observer —— 观察者，即回调。

RxJava2 博大精深，这里就不对操作符做一一介绍了，用搜索引擎可以搜到 RxJava2 的操作符。  
英语好的同学可直接看官方的操作符说明。[官方操作符说明](http://reactivex.io/documentation/operators.html)

#### Retrofit

Retrofit 的使用在工程中的 ApiStores 类中列出了一些常用的请求方式。  
主要有：  

```
@GET("api/test")
Observable<RespModel<RespBean>> apiTest(@QueryMap Map<String, Object> params);
```

常用 GET 请求方式

```
@POST("api/test")
Observable<RespModel<RespBean>> apiTest(@Body Map<String, Object> params);
```

常用 POST 请求方式

```
@FormUrlEncoded
@POST("api/test")
Observable<RespModel<RespBean>> apiTest(@FieldMap Map<String, Object> params);
```

表单 POST 请求方式

```
@GET
@Streaming
Observable<ResponseBody> apiDownloadFile(@Url String fileUrlStr);
```

文件下载 GET 请求方式

还可以把 url 放到方法中，如：

```
@GET
Observable<RespModel<RespBean>> apiTest(@Url String url, @QueryMap Map<String, Object> params);
```

## 写在后面

演示

![演示](https://user-gold-cdn.xitu.io/2018/4/24/162f67a46f1aa7f1?w=360&h=640&f=gif&s=2000797)

>记录在此，仅为学习！  
感谢您的阅读！欢迎指正！  
[欢迎加入 Android 技术交流群，群号：155495090](https://jq.qq.com/?_wv=1027&k=5hr8OKj)