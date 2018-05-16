# OkHttp 学习笔记

![OkHttp流程图](..\images\android\okhttp.jpg)

## OkHttpClient

![okhttpclient](..\images\android\okhttpclient.png)

`OkHttpClient`采用**Builder模式**包装了很多功能模块，采用**外观模式** 简洁的提供对外的API。

### Builder 内部参数

```java
Dispatcher dispatcher;  //分发器
Proxy proxy;  //代理
List<Protocol> protocols; //协议
List<ConnectionSpec> connectionSpecs; //传输层版本和连接协议
List<Interceptor> interceptors; //拦截器
List<Interceptor> networkInterceptors; //网络拦截器
ProxySelector proxySelector; //代理选择
CookieJar cookieJar; //cookie
Cache cache; //缓存
InternalCache internalCache;  //内部缓存
SocketFactory socketFactory;  //socket 工厂
SSLSocketFactory sslSocketFactory; //安全套接层socket 工厂，用于HTTPS
CertificateChainCleaner certificateChainCleaner; //验证确认响应证书 适用 HTTPS 请求连接的主机名。
HostnameVerifier hostnameVerifier;    //主机名字确认
CertificatePinner certificatePinner;  //证书链
Authenticator proxyAuthenticator;     //代理身份验证
Authenticator authenticator;      // 本地身份验证
ConnectionPool connectionPool;    //连接池,复用连接
Dns dns;  //域名
boolean followSslRedirects;  //安全套接层重定向
boolean followRedirects;  //本地重定向
boolean retryOnConnectionFailure; //重试连接失败
int connectTimeout;    //连接超时
int readTimeout; //read 超时
int writeTimeout; //write 超时
```

## Request

`Request`是对网络请求参数的统一封装，例如：URL、请求方式、请求头等。



## Call

`Call`对象是**一次**网络请求任务的封装，可用的基本操作都定义在这个接口里面；`Call`只能被执行一次，不论`execute`还是`enqueue`。

### RealCall

`RealCall`为具体的`Call`实现


#### execute 
同步请求

#### enqueue 
异步请求

