###Dagger详解
> 什么是依赖注入

- 通过构造器传入依赖的对象就是依赖注入，这样就不需要在一个模块中去创建另外一个模块了，对象是在别的地方被创建后然后通过构造器传入依赖他的对象中

> Dagger使用

- 在app/build.gradle文件中配置如下代码

```
compile 'com.squareup.dagger:dagger:1.2.+'
compile 'com.squareup.dagger:dagger-compiler:1.2.+'
```
- Dagger使用@Inject注释构造函数创建类对象，当请求构建新的类对象时，Dagger将自动获取相应的参数，并调用构造函数。

```
class Thermosiphon implements Pump{
	private final Heater heater;
	
	@Inject
	Thermosiphon(Heater heater){
		this.heater = heater;
	}
}
```
- Dagger可以直接注入成员变量，在下面的例子中，他获取Heater对象，并注入到成员变量Heater，同时获取Pump对象并注入到成员变量pump

```
class CoffeeMaker{
	@Inject Heater heater;
	@Inject Pump pump;
}
```
- 当类中有@Inject注释的成员变量，却没有@Inject注释的构造函数时，Dagger将使用类的默认构造函数，若类中缺少@Inject注释，该类是不能被Dagger创建的。
- 若是某些类不需要Inject任何对象，而又希望Dagger创建该类对象，这时，应该添加一个@Inject的默认构造函数，否则会报异常


- Dagger通过构造相应类型的对象来实现依赖关系，当请求一个CoffeeMaker对象时，Dagger将调用new CoffeeMaker()构造函数，病赋值给@Inject标记的成员变量
- 以下类型不能被构造
	- 接口类型不能被构造
	- 第三方的类不能被注释构造
	- 可配置的对象必须被配置好
- 可以使用@Provides注释函数来实现依赖关系

```
@Provides
Heater provideHeater(){
	return new ElectricHeater();
}
```
- @Provides注释的函数也可以有依赖关系

```
@Provides
Pump providePump(Thermosiphon pump){
	return pump;
}
```
- 所有的@Provides函数必须属于一个Module。

```
@Module
class DripCoffeeModule{
	@Provides Heater provideHeater(){
		return new ElectricHeater();
	}
	@Provides Pump providePump(Thermosiphon pump){
		return pump;
	}
}
```
> 构建ObjectGraph对象

- @Inject和@Provides注释的类构建了一个对象图标，这些对象与对象之间通过依赖关系相互关联，通过ObjectGraph.create()获取这个对象图表，这个函数可以接收一个或多个Module作为参数：

```
ObjectGraph objectGraph = ObjectGraph.create(new DripCoffeeModule());
```
- CoffeeApp被用于引导注入，我们请求这个对象图表来构建一个CoffeeApp的对象实例

```
class CoffeeApp implements Runnable{
	@Inject CoffeeMaker coffeeMaker;
	
	@Override public void run(){
		coffeeMaker.brew();
	}
	
	public static void main(String[] args){
		ObjectGraph objectGraph = ObjectGraph.create(new DripCoffeeModule());
		CoffeeApp coffeeApp = objectGraph.get(CoffeeApp.class);
	}
}
```
- 这里唯一缺少的是，这个对象图标并不知道这个CoffeeApp这个可注入类，我们需要在@Module注释中显示的声明

```
@Module(
	injects = CoffeeApp.class
)
class DripCoffeeModule{
	...
}
```
> 单例

- @Singlton,构建的这个对象图标将使用唯一的对象实例

```
@Provides @Singlton Heater provideHeater(){
	return new ElectricHeater();
}
```
- @Singleton注释对Dagger有效，也只在一个ObjectGraph中生效，如果有多个ObjectGraph，则有多个相应的@Singleton对象

> 延迟注入

- 使用Lazy实现延迟初始化

```
class GridingCoffeeMaker{
	@Inject Lazy<Grinder> lazyGrinder;
	
	public void brew(){
		while(needsGrinding){
			lazyGrinder.get().grind();
		}
	}
}
```
> 提供者注入

- 如果需要多个对象实例，可以利用Provider，每次调用Provider的get()函数将返回新的T的对象实例

```
class BigCoffeeMaker(){
	@Inject Provider<Filter> filterProvider;
	
	public void brew(int numberOfPots){
		for(int p = 0;p < numberOfPots;p++){
			maker.addFilter(filterProvider.get());
			maker.addCoffee();
			maker.percolate();
		}
	}
}
```
> 限定符

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named{
	String value() default "";
}

class ExpensiveCoffeeMaker{
	@Inject @Named("water") Heater waterHeater;
	@Inject @Named("hot plate") Heater hotPlateHeater;
}

@Provides @Named("hot plate") Heater provideHotPlateHeater(){
	return new ElectricHeater(70);
}
@Provides @Named("water") Heater provideWaterHeater(){
	return new ElectricHeater(93);
}
```
> 编译时有效性检查

- 若是任何绑定是无效或者不完整的，将引发编译错误

```
@Module
class DripCoffeeModule{
	@Provides Heater provideHeater(Executor executor){
		return new CpuHeater(executor);
	}
}

//缺少对象绑定
//修正这个错误，可以添加Executor的@Provides函数，或者标记该Module为不完整的，不完整的Module允许缺少对象引用

@Module(complete = false)
class DripCoffeeModule{
	@Provides Heater provideHeater(Executor executor){
		return new CpuHeater(executor);
	}
}
```
- 在Module中若provide的类，没有在injects列表中使用，将在编译时出发错误

```
@Module(injects = Example.class)
class DripCoffeeModule{
	@Provides Heater provideHeater(){
		return new ElectricHeater();
	}
	@Provides Chiller provideChiller(){
		return new ElectricChiller();
	}
}

//如果Example.class的注入声明中仅使用了Heater,而没有使用Chiller
//若是这个Module提供的对象绑定，可能被injects列表中以外的类使用，可以将Module标记为Library

@Module(
	injects = Example.class,
	library = true
)
class DripCoffeeModule{
	@Provides Heater provideHeater(){
		return new ElectricHeater();
	}
	@Provides Chiller provideChiller(){
		rerurn new ElectricChiller();
	}
}
```
- 所有需要依赖注入的类，需要被显示声明在相应的Module中
- 一个Module中所有的@Provides函数的参数都必须在这个Module中提供相应的被@Provides修饰的函数，或者在@Module注解后添加”complete = false“注明这是一个不完整的Module，表示他依赖不属于这个Module的其他Dependency。
- 一个Module中所有的@Provides函数都要被它声明的注入对象所使用，或者在@Module注解后添加"library=true"注明他含有对外的Dependency，可能被其他Module依赖

> Module重载

- 若对一个依赖关系有多个@Provides函数，Dagger将会报错，解决办法，在@Module中可以使用override＝true,重载其绑定关系

> Inject和Provide的区别
- @Inject用于注入可实例化的类，@Provides可用于注入所有类
- @Inject可用于修饰属性和构造函数，@Provides修饰的函数必须以provide开头
- @Inject修饰的函数只能是构造函数，@Provides修饰的函数必须以provide开头

- @Inject注解函数的单例模式

```
@Singleton
public class Boss{
	@Inject
	public Boss(){
	
	}
}
```
- @Provide注解函数的单例模式

```
@Provides
@Singleton
Coder provideCoder(Boss boss){
	return new Coder(boss);
}
```

> Dagger相关概念

- Module:指被@Module注解修饰的类，为Dagger提供需要依赖注入的Host信息以及一些Denpendency的生成方式

- ModuleAdapter：指由APT根据@Module注解自动生成的类，父类是Dagger的ModuleAdapter.java

- Binding:指由APT根据@Inject注解和@Prodes注解自动生成，最终继承自Binding.java的子类

- InjectAdapter:每个属性或构造函数被@Inject修饰的类都会生成一个继承自Binding.java的子类，生成类以修饰类的ClassName加上$$InjectAdapter命名。

- ProvidesAdapter:每个被@Provides修饰的生成函数都会生成一个继承自ProvidesBinding.java的子类，ProvidesBinding.java继承自Binding.java,是Provide函数所在的Module对应生成的ModuleAdapter中的静态内部类。

- Binding安装:指将Binding添加到Binding库中

- Binding连接:把当前的Binding和它的内部依赖的Binding进行连接，即初始化这个Binding内部的所有Binding，使他们可用。对DAG的角度来说，就是把某个节点与其所依赖的其他各个节点连接起来。

> 工作流程

- 编译时：InjectAdapter,ProvidesAdapter,ModuleAdapter，这些文件用于运行时辅助DAG的创建和完善，然后将这些生成的java文件和项目原有的java文件一并编译成class文件

- 运行时：在Application或某个具体模块初始化处，使用ObjectGraph类来加载部分依赖(实质上是利用编译时生成ModulesAdapters加载了所有的ProvidesBinding)，形成一个不完整的依赖关系图。

- 这个不完整的依赖关系生成之后，就可以调用ObjectGraph的相应函数来获取实例和注入依赖了，实现依赖注入的函数有两个：

```
ObjectGraph.get(Class<T> type):用于直接获取对象
ObjectGraph.inject(T instance):用于对指定对象进行属性的注入

//在这些获取实例和注入依赖的过程中，如果用到了还未加载的依赖，程序会自动对他们进行加载
```

> Dagger源码

- ObjectGraph
	- T get(Class<T> type)
	- T inject(T instance)
	- ObjectGraph plus(Object... modules)
	- void validate()
	- void injectStatics
	- ObjectGraph create(Object... modules)
	- ObjectGraph createWith(Loader loader,Object... modules)
	- DaggerObjectGraph extends ObjectGraph
	
	```
	DaggerObjectGraph base
	Linker linker
	Loader plugin
	Map<Class<?>,StaticInjection> staticInjections
	Map<String,Class<?>> injectableTypes
	List<SetBinding<?>> setBindings
	```
		