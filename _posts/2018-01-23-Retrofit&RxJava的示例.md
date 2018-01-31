---
layout:     post
title:      "Retrofit与RxJava的一个简单示例"
subtitle:   "各种框架的使用"
date:       2018-01-23 11:19:00
author:     "LeoHe"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android
    - Java
---



> 写在前面,Retrofit 整个框架运用了很多设计模式思想,而且还使用了一个Java上很重要的动态代理的理念,这个其实是Retrofit的一个设计的核心思想.

> RxJava2 是响应式编程在JVM平台的一个实现,其基本是想是将一切的事件看成是流,然后所有的操作都是针对流的操作.

使用RxJava和Retrofit2 能够写出非常优雅的代码,当然背后的原理也是要了解一下的,否则出现坑了怎么办呢?!



给个例子

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import java.util.HashMap;
import java.util.Map;

import io.reactivex.Observable;
import io.reactivex.Observer;
import io.reactivex.disposables.Disposable;
import io.reactivex.schedulers.Schedulers;
import okhttp3.ResponseBody;
import retrofit2.Call;
import retrofit2.Retrofit;
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;
import retrofit2.converter.gson.GsonConverterFactory;
import retrofit2.http.GET;
import retrofit2.http.QueryMap;

/**
 * Created by hefuduo on 2017/5/4.
 */
public class Main {
    //    http://api.tianapi.com/social/?key=APIKEY&num=10
    public static void main(String[] args) throws InterruptedException {
        Gson gson = new GsonBuilder()
                .setDateFormat("yyyy-MM-dd hh:mm:ss")
                .create();
        Retrofit retrofit = new Retrofit.Builder()
                .addConverterFactory(GsonConverterFactory.create(gson))
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .baseUrl("http://api.tianapi.com/")
                .build();
        Map<String, String> params = new HashMap<String, String>();
        params.put("key", "be3fbfe5c6a3ffa3c680f604d8db30a9");
        params.put("num", "10");
//        Call<ResponseBody> call = retrofit.create(NewsService.class).getNews(params);
//        call.enqueue(new Callback<ResponseBody>() {
//            public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
//                try {
//                    byte[] bytes = response.body().bytes();
//                    String result = new String(bytes);
//                    System.out.println(result);
//                } catch (IOException e) {
//                    e.printStackTrace();
//                }
//            }
//
//            public void onFailure(Call<ResponseBody> call, Throwable t) {
//
//            }
//        });
        //=============================================
//        Call<ResponseBean> call = retrofit.create(NewsService2.class).getNews(params);
//        try {
//            Response<ResponseBean> responseBeanResponse = call.execute();
//            System.out.println(responseBeanResponse.body().toString());
//        } catch (IOException e) {
//            e.printStackTrace();
//        }
        //==============================================
        retrofit.create(NewsService3.class).getNews(params)
//                .subscribeOn(Schedulers.io())
//                .observeOn(Schedulers.newThread())
                .subscribe(new Observer<ResponseBean>() {
                    public void onSubscribe(Disposable disposable) {

                    }

                    public void onNext(ResponseBean responseBean) {
                        System.out.println(responseBean.toString());
                    }

                    public void onError(Throwable throwable) {

                    }

                    public void onComplete() {

                    }
                });
    }
}
//简单的方式
interface NewsService {
    @GET("social")
    Call<ResponseBody> getNews(@QueryMap Map<String, String> params);
}

//Convert Factory
interface NewsService2 {
    @GET("social")
    Call<ResponseBean> getNews(@QueryMap Map<String, String> params);
}
//Convert Factory + CallAdapter
interface NewsService3 {
    @GET("social")
    Observable<ResponseBean> getNews(@QueryMap Map<String, String> params);
}



```



```java
import java.util.Date;
import java.util.List;

/**
 * PACKAGE_NAME
 * Created by hefuduo on 2018/1/23.
 */
public class ResponseBean {
    public int code;
    public String msg;
    List<News> newslist;

    @Override
    public String toString() {
        return "code = " + code + ",msg = " + msg;
    }
}
class News{
    String ctime;
    String title;
    String description;
    String picUrl;
    String url;
}

```



```java
//动态代理的一个简单样子,Retrofit跟这个不太一样的
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * PACKAGE_NAME
 * Created by hefuduo on 2018/1/23.
 */
public interface PersonDao {
    void say();
}

class PersonDaoImpl implements PersonDao {

    public void say() {
        System.out.println("time to go");
    }
}

class PersonHandler implements InvocationHandler{
    private Object mObject;
    
    PersonHandler(Object o){
        mObject = o;
    }
    
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before");
        Object result = method.invoke(mObject,args);
        System.out.println("after");
        return result;
    }
}
//注意动态代理只能针对接口,不能针对类.
```



==//TODO== 待补充,Retrofit2,从源码角度剖析Retrofit2框架用到的设计模式.