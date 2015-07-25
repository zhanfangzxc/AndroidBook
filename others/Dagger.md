###Dagger详解

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
@Provides
```
