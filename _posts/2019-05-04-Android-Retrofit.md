---
layout:     post
title:      Android Retrofit 概览
subtitle:   Retrofit
date:       2019-05-04
header-img: img/post-bg-desk.jpg
catalog:    true
tags:
    - Android
    - Retrofit
    - Note
---
### Retrofit介绍
Retrofit 是基于OkHttp网络请求框架的二次封装，提供了更灵活易用的API，底层的网络请求仍然是OkHttp
> AndroidAsyncHttp: 基于HttpClient，已废弃，Android5.0 不再使用 HttpClient<br>
> Volley: 基于 HttpUrlConnection, google官方推出，比较适合轻量级的网络交互，不适合大文件上传下载场景<br>
> Retrofit: API 简洁易用，注解配置高度解耦，支持RxJava

### 集成 Retrofit
1. 依赖库
    ```
    versions.okhttp = "3.9.0"
    versions.retrofit = "2.5.0"
    
    // retrofit
    def retrofit = [:]
    retrofit.retrofit = "com.squareup.retrofit2:retrofit:$versions.retrofit"
    retrofit.gson = "com.squareup.retrofit2:converter-gson:$versions.retrofit"
    retrofit.rxjava2 = "com.squareup.retrofit2:adapter-rxjava2:$versions.retrofit"
    deps.retrofit = retrofit
    // okhttp
    def okhttp = [:]
    okhttp.okhttp = "com.squareup.okhttp3:okhttp:$versions.okhttp"
    okhttp.interceptor = "com.squareup.okhttp3:logging-interceptor:$versions.okhttp"
    deps.okhttp = okhttp
    ```
2. 网络权限
    ```
    <use-permission android:name="android.permission.INTERNET"/>
    ```

### 创建接口设置 get/post
```
@GET、@POST: 请求方式，还有PUT/HEAD/OPTIONS等
@Path: 请求URL的字符代替, GET("/image/{id}") ResponseBody.example(@Path("id") int id);
@Query: 传递的参数
@QueryMap: 需要添加过多的参数时，用Map包裹起来
@Field: 传递的参数
@FieldMap: 需要添加过多的参数时，用Map包裹起来
@Body: 请求实体对象
@FormUrlEncoded: URL 编码，一般会配合@post @feild一起使用
```

### 常用的数据解析器

[Moshi: squre-JSON-for-kotlin-and-Java](https://github.com/square/moshi)<br>
[Protobuf: google-data-interchange-format](https://github.com/protocolbuffers/protobuf)<br>

```
versions.okhttp = "3.9.0"
versions.retrofit = "2.5.0"

Gson: "com.squareup.retrofit2:converter-gson:$versions.retrofit"
Jackson: "com.squareup.retrofit2:converter-jackson:$versions.retrofit"
SimpleXML: "com.squareup.retrofit2:converter-simplexml:$versions.retrofit"
Protobuf: "com.squareup.retrofit2:converter-protobuf:$versions.retrofit"
Moshi: "com.squareup.retrofit2:converter-moshi:$versions.retrofit"
Wire: "com.squareup.retrofit2:converter-wire:$versions.retrofit"
Scalars: "com.squareup.retrofit2:converter-scalars:$versions.retrofit"
```

### 使用
```
Retrofit retrofit = new Retrofit.Builder()
    .client(okHttpClient)
    .addConverterFactory(GsonConverterFactory.create())
    .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
    .baseUrl(baseUri)
    .build();

public interface UserService {
    // Resule<T> { int code; String message; T data }
    // server return most like { code: 200, message: 'success', data: {} }
    @GET
    Call<Result<UserInfo>> login(@Query("name") String name, @Query("pass") String pass)
    // rxjava
    // Flowable<Result<UserInfo>> login(@Query("name") String name, @Query("pass") String pass)
}

UserService uService = retrofit.create(UserService.class)
Call<Result<UserInfo>> call = uService.login("name", "password")
// sync:call.excute()/ async:call.enqueue()
Response<Result<UserInfo>> rsp = call.excute()
android.Util.Log.d("TAG", "" + rep.code() + " and " + rep.body().code())

// 异步方式
call.enqueue(new CallBack<Result<UserInfo>>() {
    onResponse(Call<Result<UserInfo>> call, Response<Result<UserInfo>> rsp);
    onFailer(Call<Result<UserInfo>> call, Throwable t)
})

// rxjava
// Flowable<Result<UserInfo>> flowable = uService.login("name", "password")
// ...

// --------- //
// Kotlin
// val uService: UserService = retrofit.create(UserService::class.java)
// val call: Call<UserInfo> = uService.login("name", "password")
```

**Note**<br>
使用post请求时，如果请求参数使用了field，需要指定FormUrlEncoded

### 总结
Retrofit 是OkHttp的二次封装，使用注解方式，使得网络请求参数配置更为灵活，扩展性更好
多数的开源库也使用Retrofit进行网络请求
