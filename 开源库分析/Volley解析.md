###Volley

> 一些主要类的介绍

```
 Volley:通过newRequestQueue()创建请求队列
 
 Request:代表网络请求的抽象类,包含请求的一些信息
 	|--StringRequest
 	|--JsonRequest
 	|--ImageRequest
 	
 	||--Response<T> parseNetworkResponse(NetworkResponse reponse):将网络返回的原生字节内容转换为合适的类型
 	||--deliverResponse(T response):将解析成合适类型的内容传递给它们的监听回调
 	||--getBody()
 	||--getParams()
 	
 RequestQueue:请求队列，里面包含了一个CacheDispatcher,一个NetworkDispatcher，一个ResponseDelivery,通过start函数会启动CacheDispatcher和NetworkDispatchers
 	|--CacheDispatcher：用于处理走缓存请求的调度线程
 	|--NetworkDispatcher:用于处理走网络请求的调度线程
 	|--ResponseDelivery:返回结果分发接口
 	
 	||--mCacheQueue:缓存请求队列
 	||--mNetworkQueue:网络请求队列
 	||--mCurrentRequests:正在进行中，尚未完成的请求集合
 	||--mWaitingRequests:一个等待请求的集合，如果一个请求正在被处理并且可以被缓存，后续的相同的url的请求，将进入此等待队列。
 	||--start():创建一个CacheDispatcher实例和一些NetworkDispatcher实例并分别start
 	||--add(Request<T> request):添加一个请求到调度队列中
 	||--finish(Request<?> request):请求完成
 	||--cancelAll(RequestFilter filter):取消请求，filter表示可以按照自定义的过滤器过滤需要取消的请求
 	||--cabcelAll(final Object tag):取消请求，按照Request.setTag设置好的tag取消请求
 	
 	请求结束之后所要做的操作：
 	|--首先从正在进行中请求集合mCurrentRequests中移除该请求。
 	|--然后查找请求队列集合mWaitingRequests是否存在等待的请求，如果存在，则将等待队列移除，并将等待队列所有的请求添加到缓存请求队列中，让缓存请求处理线程CacheDispatcher自动处理
 	
CacheDispatcher:一个线程，用于调度处理走网络的请求，启动后会不断从缓存请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给ResponseDelivery去执行后续处理，当结果未缓存过或者缓存失效或缓存需要刷新的情况下，请求都需要重新进入NetworkDispatcher去调度处理
	|--mCacheQueue:缓存请求队列
	|--mNetworkQueue:网络请求队列
	|--Cache mCache:缓存类
	|--ResponseDelivery mDelivery:请求结果传递类

NetworkDispatcher:一个线程，用于调度处理走网络的请求，启动后会不断从网络请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给ResponseDelivery去执行后续处理，并判断结果是否要进缓存。
	|--mQueue:网络请求队列
	|--mNetwork:网络类，代表一个可以执行请求的网络
	|--mCache:缓存类，代表了一个可以获取请求结果，存储请求结果的缓存
	|--mDelivery:请求结果传递类，可以传递请求结果或者错误到调用者。
ResponseDelivery:返回结果分发接口，目前只有基于
	|--postResponse(request,reponse)
	|--postResponse(request,reponse,runnable)
	|--postError(request,error)
	
ExecutorDelivery的在入参Handler对应想成内进行分发

HttpStack:处理Http请求，返回请求结果，基于HttpUrlConnection
	|--HttpResponse performRequest(Request<?> request,Map<String,String> additionalHeaders):执行Request代表的请求
	
HttpClientStack:实现HttpStack，利用HttpClient进行各种请求方式的请求

HurlStack:利用HttpURLConnection进行各种请求方式的请求。

Response:封装了经过解析后的数据，用于传输，并且有两个内部接口Listener和ErrorListener分别表示请求失败和成功后的回调。

Network:调用HttpStack处理请求，并将结果转换为可被ResponseDelivery处理的NetworkReponse
	|--NetworkResponse performRequest(Request<?> request):用于执行特定请求

NetworkResponse:是Network中方法performRequest的返回值，Request的parseNetworkResponse()方法需要的参数
	|--int statusCode:Http状态响应码
	|--byte[] data:Body数据
	|--Map<String,String> headers:响应headers
	|--boolean notModified:表示是否为304响应
	|--long networkTimeMs:请求耗时
	
Cache:缓存请求结果，Volley默认使用的是基于sdcard的DiskBasedCache。NetworkDispatcher得到请求结果后判断是否需要存储在cache，CacheDispatcher会从Cache中取缓存结果
	|--get(String key):根据key获取请求的缓存实体
	|--put(String key,Entry entry):存入一个请求的缓存实体
	|--remove(String key):移除指定的缓存实体
	|--clear():清空缓存
	
	|--Entry:代表缓存实体的内部类
		|--data:请求返回的数据(Body实体)
		|--String etag:Http响应首部中用于缓存新鲜度验证的ETag
		|--long serverDate:Http响应首部中的响应产生时间
		|--long ttl:缓存的过期时间
		|--long softTtl:缓存的新鲜时间
		|--Map<String,String> responseHeaders:响应的Headers
		|--boolean isExpired():判断缓存是否过期
		|--boolean regreshNeeded():判断缓存是否新鲜，不新鲜的缓存需要发到服务器做新鲜度的检测
```

- CacheDispatcher

```
 @Override
    public void run() {
        if (DEBUG) VolleyLog.v("start new dispatcher");
        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

        // Make a blocking call to initialize the cache.
        mCache.initialize();

        while (true) {
            try {
                // Get a request from the cache triage queue, blocking until
                // at least one is available.
                final Request<?> request = mCacheQueue.take();
                request.addMarker("cache-queue-take");

                // If the request has been canceled, don't bother dispatching it.
                if (request.isCanceled()) {
                    request.finish("cache-discard-canceled");
                    continue;
                }

                // Attempt to retrieve this item from cache.
                Cache.Entry entry = mCache.get(request.getCacheKey());
                //判断是否存在缓存标志，如果不存在就添加到网络请求队列
                if (entry == null) {
                    request.addMarker("cache-miss");
                    // Cache miss; send off to the network dispatcher.
                    mNetworkQueue.put(request);
                    continue;
                }

                // If it is completely expired, just send it to the network.
                //判断缓存是否过期
                if (entry.isExpired()) {
                    request.addMarker("cache-hit-expired");
                    request.setCacheEntry(entry);
                    mNetworkQueue.put(request);
                    continue;
                }

                // We have a cache hit; parse its data for delivery back to the request.
                request.addMarker("cache-hit");
                Response<?> response = request.parseNetworkResponse(
                        new NetworkResponse(entry.data, entry.responseHeaders));
                request.addMarker("cache-hit-parsed");
					//缓存是否需要刷新
                if (!entry.refreshNeeded()) {
                    // Completely unexpired cache hit. Just deliver the response.
                    mDelivery.postResponse(request, response);
                } else {
                    // Soft-expired cache hit. We can deliver the cached response,
                    // but we need to also send the request to the network for
                    // refreshing.
                    request.addMarker("cache-hit-refresh-needed");
                    request.setCacheEntry(entry);

                    // Mark the response as intermediate.
                    response.intermediate = true;

                    // Post the intermediate response back to the user and have
                    // the delivery then forward the request along to the network.
                    mDelivery.postResponse(request, response, new Runnable() {
                        @Override
                        public void run() {
                            try {
                                mNetworkQueue.put(request);
                            } catch (InterruptedException e) {
                                // Not much we can do about this.
                            }
                        }
                    });
                }

            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }
        }
   }
```
- RequestQueue

```
public <T> Request<T> add(Request<T> request) {
        // Tag the request as belonging to this queue and add it to the set of current requests.
        request.setRequestQueue(this);
        synchronized (mCurrentRequests) {
            mCurrentRequests.add(request);
        }

        // Process requests in the order they are added.
        request.setSequence(getSequenceNumber());
        request.addMarker("add-to-queue");

        // If the request is uncacheable, skip the cache queue and go straight to the network.
        //如果该Request不是可缓存的，那么直接将该Request添加到网络调度队列中
        if (!request.shouldCache()) {
            mNetworkQueue.add(request);
            return request;
        }

        // Insert request into stage if there's already a request with the same cache key in flight.
        synchronized (mWaitingRequests) {
            String cacheKey = request.getCacheKey();
            if (mWaitingRequests.containsKey(cacheKey)) {
                // There is already a request in flight. Queue up.
                Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);
                if (stagedRequests == null) {
                    stagedRequests = new LinkedList<Request<?>>();
                }
                stagedRequests.add(request);
                mWaitingRequests.put(cacheKey, stagedRequests);
                if (VolleyLog.DEBUG) {
                    VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
                }
            } else {
                // Insert 'null' queue for this cacheKey, indicating there is now a request in
                // flight.
                mWaitingRequests.put(cacheKey, null);
                mCacheQueue.add(request);
            }
            return request;
        }
    }
 }
```
