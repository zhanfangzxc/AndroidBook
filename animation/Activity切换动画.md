###Activity切换动画

- 在SecondActivity的布局中设置顶层的布局为id为root，在SecondActivity中，添加如下代码：

```
@Override
public void onCreate(Bundle savedInstanceState){
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_second);
	
	rootView = findViewById(R.id.root);
	//只需要再首次创建的时候执行动画，因此需要满足条件savedInstanceState == null
	if(savedInstanceState == null){
		rootView.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener(){
			@Override
			public boolean onPreDraw(){
				//防止死循环
				rootView.getViewTreeObserver().removePreDrawListener(null);
				startRootAnimation();
				return true;
			}
		});
	}
}

private void startRootAnimation(){
	rootView.setScaleY(0.1f);
	rootView.setPivotY(rootView.getY+rootView.getHeight()/2);
	rootView.animate()
			 .scaleY(1)
			 .setDuration(1000)
			 .setInterpolator(new AccelerateInterpolator())
			 .start();
}
```
- 设置SecondActivity为透明背景的主题

```
<style name="AppTheme.TransparentActivity">
	<item name="android:windowBackground">@android:color/transparent</item>
	<item name="android:windowIsTranslucent">true</item>
	<item name="android:windowAnimationStyle">@null</item>
</style>
```

> 存在的问题:

这种方式需要为顶级view以及各层子view分别设置动画，是的顶级view和子view同时展开，或者子view延后展开