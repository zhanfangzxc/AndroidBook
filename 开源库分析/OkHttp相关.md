### OkHttp相关

> 相关类库和文章

```
OkHttp使用教程:http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html

OkHttp封装的工具类：https://github.com/hongyangAndroid/okhttp-utils

```

> 主要的类

```
|--OkHttpClient:配置并创建HTTP链接，应用程序中所有的HTTP请求可以使用一个OkHttpClient实例，优点是可以共享缓存，线程池，连接的重复利用。
	|--newCall(Request request):准备在将来某个时候要执行的请求

|--Call:是准备好要执行的请求，可以取消
	|--Reponse excute()
	|--enqueue(Callback responseCallback)
	|--AsyncCall:内部类
		|--excute()
	|--Response getResponse(Request request,boolean forWebSocket):执行Request请求，返回Response
		|--HttpEngine
		
|--Request：代表一个Http请求，该类的实例是不可变的
	|--String urlString
	|--String method
	|--Headers headers
	|--RequestBody body
	|--Object tag
	
|--Response:代表一个Http响应
	|--Protocol protocol
	|--String message
	|--Handshake handshake:握手
	|--ResponseBody body
	|--Response networkResponse:网络响应
	|--Response cacheResponse:缓存响应
	|--Response priorResponse:Http重定向或者授权验证的响应
	
|--HttpEngine:处理一个Http的请求和发送，每个HttpEngine都遵循下面的生命周期：
	Http Request通过sebdRequest()发送message
	Http Response通过readResponse()读取消息
	Request和Response可以通过响应缓存来满足
	
	|--MAX_FOLLOW_UPS=20:重定向和身份验证可以尝试的次数
	|--sendRequest():发送请求
	|--readResponse():读取http的响应结果
	
|--ConnectionPool:连接池，管理HTTP和SPDY连接复用，可降低网络延迟
	|--Connection get(Address address):根据Address获取一个之前被回收的Connection,如果不存在，就返回null
	|--void recycle(Connection connection):回收Connection
	|--evictAll():关闭和移除连接池中的所有Connnection
	|--boolesn performCleanup()

|--Connection:一个Http、Https或Https+SPDY的SocketS和streams的连接。
	|--ConnectionPool pool
	|--Route route
	|--Socket socket
	|--HttpConnection httpConnection
	|--SpdyConnection spdyConnection
	|--Protocol protocol
	|--HandleShake handleshake
	
	||--connect(int connectTimeout,int readTimeout,int writeTimeout,Request tunnelRequest)
	
|--HttpConnection:socket连接，用来发送Http/1.1消息
	|--ConnectionPool pool
	|--Connection connection
	|--Socket socket
	|--BufferedSource source
	|--BufferedSink sink
	
	||--readRequest(Headers headers,String requestLine)
	||--readResponse()

**SPDY:是谷歌开发的基于TCP的应用层协议，用于最小化网络延迟，提升网络速度，优化用户的网络使用体验。不是替代HTTP的协议，而是对HTTP协议的增强，新协议的功能包括数据流的多路复用，请求优先级以及HTTP报头压缩。
```

- Connection类

```
 void connect(int connectTimeout, int readTimeout, int writeTimeout, Request tunnelRequest)
      throws IOException {
    if (connected) throw new IllegalStateException("already connected");

    if (route.proxy.type() == Proxy.Type.DIRECT || route.proxy.type() == Proxy.Type.HTTP) {
      socket = route.address.socketFactory.createSocket();
    } else {
      socket = new Socket(route.proxy);
    }

    socket.setSoTimeout(readTimeout);
    Platform.get().connectSocket(socket, route.inetSocketAddress, connectTimeout);

    if (route.address.sslSocketFactory != null) {
      upgradeToTls(tunnelRequest, readTimeout, writeTimeout);
    } else {
      httpConnection = new HttpConnection(pool, this, socket);
    }
    connected = true;
  }
```

