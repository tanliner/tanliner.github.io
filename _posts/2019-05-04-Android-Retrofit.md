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

### Retrofit原理
使用方通过 Retrofit 获得网络数据，必然会经历建立连接并读取网络连接数据这两个重要过程

### 如何构建一个请求
Retrofit在在调用create的时候，动态代理的方式创建一个Interface的实例
#### 创建 Interface 实例
```
public <T> T create(final Class<T> service) {
    // 动态代理
    // Proxy.newProxyInstance(cl, interfaces, handler)
    return
    (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
            new InvocationHandler() {
                private final Platform platform = Platform.get();
                private final Object[] emptyArgs = new Object[0];

                @Override
                public Object invoke(Object proxy, Method method, @Nullable Object[] args)
                    throws Throwable {
                    // If the method is a method from Object then defer to normal invocation.
                    if (method.getDeclaringClass() == Object.class) {
                        return method.invoke(this, args);
                    }
                    if (platform.isDefaultMethod(method)) {
                        return platform.invokeDefaultMethod(method, service, proxy, args);
                    }
                    return loadServiceMethod(method).invoke(args != null ? args : emptyArgs);
                }
            }
}
```

后续调用service.getHttpData(x, y)的时候在实例化一个 HttpServiceMethod。
那到底是如何实例化这个method呢？
前面提到了 `CallAdapterFactory`，这就是如何创建一个请求的工厂，他通过Interface中定义的方法的注解创建
对应的请求。调用步骤如下
```
loadServiceMethod -> ServiceMethod.parseAnnotations(retrofit, method)
    RequestFactory.parseAnnotations
    HttpServiceMethod.parseAnnotations -> createCallAdapter
    HttpServiceMethod.parseAnnotations -> createResponseConverter
invoke()
```
可以看到，创建了发起请求的adapter和转换response的converter，然后

```
abstract class ServiceMethod<T> {
    static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
        // 根据方法的注解 @POST/@GET和方法的参数（post或get请求方式服务端要求的参数）获取一个请求工厂
        // parseParameter解析参数，@Field/@FieldMap等
        RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);
        Type returnType = method.getGenericReturnType();
        ...
        // 实例化一个method
        return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
    }
    abstract T invoke(Object[] args);
}
```
HttpServiceMethod.java extends ServiceMethod
```
static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
    Retrofit retrofit, Method method, RequestFactory requestFactory) {
    callAdapter = createCallAdapter(retrofit, method);
    ...
    responseConverter = createResponseConverter(retrofit, method, responseType);

    okhttp3.Call.Factory callFactory = retrofit.callFactory;
    return new HttpServiceMethod<>(requestFactory, callFactory, callAdapter, responseConverter);
}
```
Retrofit.java
```
// 寻找合适的callAdapter
public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
    CallAdapter<?, ?> adapter = null;
    for (i < factories.size) {
        adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
        if (adapter != null) {
            return adapter;
        }
    }
    throw new IllegalArgumentException("xxx");
}

// 寻找合适的 convertAdapter
public <T> Converter<ResponseBody, T> responseBodyConverter(Type type, Annotation[] annotations) {
    Converter<ResponseBody, ?> converter = null;
    for (i < factories.size) {
        converter = callAdapterFactories.get(i).responseBodyConverter(type, annotations, this);
        if (converter != null) {
            return (Converter<ResponseBody, T>) converter;
        }
    }
    throw new IllegalArgumentException("xxx");
}
```
RxJava2CallAdapterFactory.java
```
public static RxJava2CallAdapterFactory create() {
    return new RxJava2CallAdapterFactory(null, false);
}

// 注意看get 方法的参数，returnType，annotations
@Override
public @Nullable CallAdapter<?, ?> get(
    Type returnType, Annotation[] annotations, Retrofit retrofit) {
    Class<?> rawType = getRawType(returnType);
    // scheduler = null, isAsync = false
    return new RxJava2CallAdapter(responseType, scheduler, isAsync, isResult, isBody, isFlowable,
        isSingle, isMaybe, false);
}
```
RxJava2CallAdapter.java
```
@Override
public Object adapt(Call<R> call) {
    Observable<Response<R>> responseObservable = isAsync
        ? new CallEnqueueObservable<>(call)
        : new CallExecuteObservable<>(call);

    Observable<?> observable;
    if (isResult) {
        observable = new ResultObservable<>(responseObservable);
    } else if (isBody) {
        observable = new BodyObservable<>(responseObservable);
    } else {
        observable = responseObservable;
    }

    if (scheduler != null) {
        observable = observable.subscribeOn(scheduler);
    }

    if (isFlowable) {
        return observable.toFlowable(BackpressureStrategy.LATEST);
    }
    if (isSingle) {
        return observable.singleOrError();
    }
    if (isMaybe) {
        return observable.singleElement();
    }
    if (isCompletable) {
        return observable.ignoreElements();
    }
    return RxJavaPlugins.onAssembly(observable);
}
```

终于创建成功了一个 `httpservicemethod`，已经完成了post/get的参数解析。看起来
似乎只要调用这个 httpservicemethod 的时候即可创建Http请求并获取返回值
```
// HttpServiceMethod.java super ServiceMethod
// final class HttpServiceMethod<ResponseT, ReturnT> extends ServiceMethod<ReturnT> {
@Override
ReturnT invoke(Object[] args) {
    return callAdapter.adapt(
        new OkHttpCall<>(requestFactory, args, callFactory, responseConverter));
}
```
### 原理-获取请求的返回值
接着看adapt
```
@Override
public Object adapt(Call<R> call) {
    // 注意这里的 isAsync，默认create创建的是同步请求，isAsync = false
    Observable<Response<R>> responseObservable = isAsync
        ? new CallEnqueueObservable<>(call)
        : new CallExecuteObservable<>(call);

    Observable<?> observable;
    if (isResult) {
        observable = new ResultObservable<>(responseObservable);
    } else if (isBody) {
        observable = new BodyObservable<>(responseObservable);
    } else {
        observable = responseObservable;
    }

    if (scheduler != null) {
        observable = observable.subscribeOn(scheduler);
    }

    if (isFlowable) {
        return observable.toFlowable(BackpressureStrategy.LATEST);
    }
    ...
    return RxJavaPlugins.onAssembly(observable);
}
```
咦，怎么就返回了一个Observable/Flowable。原来，调用servicemethod并没有发生http请求。那如何创建request呢？
#### 创建OkHttp
```
// CallExecuteObservable.java
protected void subscribeActual(Observer<? super Response<T>> observer) {
    Call<T> call = originalCall.clone();
    boolean terminated = false;
    try {
        Response<T> response = call.execute();
        if (!disposable.isDisposed()) {
            observer.onNext(response);
        }
        if (!disposable.isDisposed()) {
            terminated = true;
            observer.onComplete();
        }
    } catch (Throwable t) {
        ...
    }
}
// OkHttpCall
@Override
public Response<T> execute() throws IOException {
    okhttp3.Call call;

    synchronized (this) {
        // 一个 call 只能执行一次
        if (executed) throw new IllegalStateException("Already executed.");
        executed = true;

        ...

        call = rawCall;
        if (call == null) {
            try {
                // 创建一个 okhttp3.Call
                call = rawCall = createRawCall();
            } catch (IOException | RuntimeException | Error e) {
                throwIfFatal(e); //  Do not assign a fatal error to creationFailure.
                creationFailure = e;
                throw e;
            }
        }
    }

    if (canceled) {
        call.cancel();
    }
    // 执行 okhttp3.call后，解析并转换请求回来的body
    return parseResponse(call.execute());
}

private okhttp3.Call createRawCall() throws IOException {
    // 就是通过 OkHttpClient 创建了一个 okhttp3.Call
    // 从哪里获取的这个 callFactory
    okhttp3.Call call = callFactory.newCall(requestFactory.create(args));
    if (call == null) {
        throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
}
```
#### callFactory
原来，Retrofit 设置 Client 的时候，就是在设置 callFactory，即设置OkHttpClient为callFactory
```
// Retrofit.java
public Builder client(OkHttpClient client) {
    return callFactory(checkNotNull(client, "client == null"));
}

public Builder callFactory(okhttp3.Call.Factory factory) {
    this.callFactory = checkNotNull(factory, "factory == null");
    return this;
}
```
#### 执行 http 同步请求
```
@Override
public Response execute() throws IOException {
    Response result = getResponseWithInterceptorChain();
}
```

#### 执行 http 异步请求
```
@Override
public void enqueue(Callback responseCallback) {
    synchronized (this) {
        if (executed) throw new IllegalStateException("Already Executed");
        executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}

for (int i = 0, size = executableCalls.size(); i < size; i++) {
    AsyncCall asyncCall = executableCalls.get(i);
    asyncCall.executeOn(executorService());
    // executorService.execute(this);
}


// 是一个 Runable，run 方法再次调用 execute 方法
final class AsyncCall extends NamedRunnable {
    @Override
    protected void execute() {
        Response response = getResponseWithInterceptorChain();
        responseCallback.onResponse(RealCall.this, response);
    }
}
```
#### 核心请求逻辑
```
// 同步或者异步最终都会调用这个方法
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    // 添加自定义的拦截器
    interceptors.addAll(client.interceptors());
    // 这里开始创建 StreamAllocation
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    // 建立 socket 连接
    interceptors.add(new ConnectInterceptor(client));
    // network intercepters，这些拦截器是在socket连接建立成功后才会执行的
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    // 向服务端发送请求，header+body等
    // 这个拦截器最后不会再调用 chain.process，已经处于链的末端了
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    return chain.proceed(originalRequest);
}
```

### 总结
Retrofit 是OkHttp的二次封装，使用注解方式，使得网络请求参数配置更为灵活，扩展性更好
多数的开源库也使用Retrofit进行网络请求
