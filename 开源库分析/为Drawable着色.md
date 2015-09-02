###为Drawable着色

继承BitmapDrawable，重写draw方法

```
public final class TintedBitmapDrawable extends BitmapDrawable{
	private int tint;
	private int alpha;
	
	public TintedBitmapDrawable(final Resource res,final Bitmap bitmap,final int tint){
		super(res,bitmap);
		this.tint = tint;
		this.alpha = Color.aplha(tint);
	}
	
	public TintedDrawableBitmap(final Resources res,final int resId,final int tint){
		super(res,BitmapFactory.decodeResource(res,resId));
		this.tint = tint;
		this.alpha = Color.alpha(tint);
	}
	
	public void setTint(final int tint){
		this.tint = tint;
		this.alpha = Color.alpha(tint);
	}
	
	@Override
	public void draw(final Canvas canvas){
		final Paint paint = getPaint();
		if(paint.getColorFilter() == null){
			paint.setColorFilter(new LightingColorFilter(tint,0));
			paint.setAplha(alpha);
		}
		super.draw(canvas);
	}
}
```

> 谷歌图标集：https://www.google.com/design/icons/