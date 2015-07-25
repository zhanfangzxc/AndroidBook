###Volley相关用法

> Volley介绍

Volley所提供的功能：

- Json,图像等的异步下载
- 网络请求的排序
- 网络请求的优先级处理
- 缓存
- 多级别取消请求
- Activity生命周期的联动(Activity结束时取消所有网络请求)

> get请求

```
mQueue = Volley.newRequestQueue(getApplicationContext(());
mQueue.add(new JsonObjectRequest(METHOD.GET,url,null,new Listener(){
	@Override
	public void onResponse(){
		Log.d(TAG,"response:"+response.toString());
	}
},null));

mQueue.start();
```

> 给ImageView设置图片源

```
//第一个参数是Imageview
//第二个参数是默认的图片资源id
//第三个参数是请求失败时的图片资源id
//imageLoader的方法都需要从主线程调用
ImageListener listener = ImageLoader.getImageListener(imageView,android.R.drawable.ic_menu_rotate,android.R.drawable.ic_delete);
mImageLoader.get(url,listener);
```
> NetworkImageView：ImageView的代替

当这个控件从父控件detach的时候，会自动取消网络请求

```
mImageView.setImageUrl(url,imageLoader);

mImageLoader = new ImageLoader(mRequestQueue,new BitmapLruCache());
if(holder.imageRequest != null){
	holder.imageRequest.cancel();
}
holder.imageRequest = new ImageLoader
```
> 使用OKHttp处理Volley的底层HTTP请求

- 导入Volley
- 导入OKHttp,okhttp-urlconnection
- 创建OKHttpStack

```
public class OKHttpStack extends HurlStack{
	private final OKHttpClient client;
	public OKHttpStack(){
		this(new OkHttpClient());
	}
	
	public OKHttpStack(OkHttpClient client){
		if(client == null){
			throw new NullPointerException("client must not be null.");
		}
		this.client = client;
	}
	
	@Overrride
	protected HttpUrlConnection createConnection(URL url) throws IOException{
		return client.open(url);
	}
}
```
- 创建Volley队列

```
Volley.newRequestQueue(context,new OkHttpStack());
```

> 对于OKHttpStack不支持OKHttp2.0的解决方案

```
public class OKHttpStack extends HurlStack{
	private final OKUrlFactory okUrlFactory;
	public OKHttpStack(){
		this(new OkUrlFactory(new OkHttpClient()));
	}
	
	public OkHttpStack(OkUrlFactory okUrlFactory){
		if(okUrlFactory == null){
			throw new NullPointerException("Client must not be null");
		}
		this.okUrlFactory = okUrlFactory;
	}
	
	@Override
	protected HttpURLConnection createConnection(URL url) throws IOException{
		return okUrlFactory.open(url);
	}
}
```