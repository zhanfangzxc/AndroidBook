###Android Support Annotations

- @NONNULL：表示字段，参数，方法返回值不能为空
- @NULLABLE：表示字段，参数，方法返回值可以为空

- @CHECKRESULT:让使用者知道该函数的返回值是需要使用的
- @STRINGRES/@DRAWABLERES:当你给setText()函数传入一个整型值，TextView将他作为一个String资源id对待，并会进行查找以便设置这个字符串
- @KEEP:被这个注解修饰的函数或者类，在代码进行混淆或者压缩时会被保持住。
- AnimatorRes:表示一个整型参数，字段或方法，返回值是一个动画资源引用
- AnimRes
- AnyRes
- ArrayRes
- AttrRes
- BoolRes
- ColorRes
- DimenRes
- DrawableRes
- FractionRes
- IdRes
- IntDef

```
使用示例

public class IceCreamFlavourManager{
	private int flavour;
	public static final int VANILLA = 0;
	public static final int CHOCOLATE = 1;
	public static final int STRAWBERRY = 2;
	@IntDef({VANILLA,CHOCOLATE,STRAWBERRY})
	public @interface Flavour(){
	}
	@Flavour
	public int getFlavour(){
		return flavour;
	}
	public void setFlavour(@Flavour int flavour){
		this.clavour = flavour;
	}
}

//当调用setFlavour方法的时候，所传的参数必须是@IntDef中所定义的那三种参数

//也可以定义标志位
@IntDef(flag = true,value={VANILLA,CHOCOLATE,STRAWBERRY})
public @interface Flavour(){
}
//调用方法
iceCreamFlavourManager.setFlavour(IceCreamFlavourManager.VANILLA & IceCreamFlavourManager.CHOCKLATE)
```
- IntegerRes
- InterpolatorRes
- LayoutRes
- MenuRes
- PluralsRes
- StringDef
- StringREs
- StyleableRes
- StyleRes
- VisibleForTesting
- XmlRes