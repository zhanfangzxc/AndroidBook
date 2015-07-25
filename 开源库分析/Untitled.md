###问题收集

> 监听软键盘的弹出和隐藏

- 前提：为Activity配置如下属性

```
android:windowSoftInputMode="adjustResize"
```

- 创建自定义的RootView

```
public class ResizeView extends LinearView{
	private OnResizeListener listener;
	
	public ResizeView(Context context,Attributes attrs){
		super(context,attrs);
	}
	
	public interface OnResizeView{
		void OnResize(int w, int h, int oldw, int oldh);
	}
	
	@Override
	protected void onSizeChanged(int w, int h, int oldw, int oldh){
		if(listener != null){
			listener.OnResize(w,h,oldw,oldh);
		}
	}
	
	public void setOnResizeListener(OnResizeListener l){
		listener = l;
	}
}

ResizeView rootView = (ResizeView)findViewById(R.id.root_view);

rootView.setOnResizeListener（new OnResizeListener{
	@Override
	public void OnResize(int w, int h, int oldw, int oldh){
		if(h < oldh){
			//TODO 软键盘弹出的状态
		}
	}
});
```

> 使Listview显示最后一条的方法

```
listView.clearFocus();
listView.post(new Runnable(){
	@Override
	public void run(){
		listView.setSelection(int position);
	}
});
```