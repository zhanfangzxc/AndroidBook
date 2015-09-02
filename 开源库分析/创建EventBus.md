###创建EventBus

> 第一步：接口的定义

```
public interface IBus {
	//用于注册事件接收器的目标对象
    void register(Object target);
	//用于取消注册目标对象
    void unregister(Object target);
	//用于发送事件给目标对象
    void post(Object event);
}
```

事件接收器通过@BusReceiver这个注解指定，因此我们需要在register(target)的时候查找目标对象中所有使用了这个注解的放大，并排除非public，和static方法，没有参数或者参数超过一个的也需要排除

> 创建@BusReceiver的方法

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface BusReceiver {
}
```
> 创建findAnnotatedMethods(class,annotation)方法，用于查找给定class里面使用了Annotation的方法，先忽略从父类继承过来的方法，只查找指定类的方法。

```
public class Helper{
	public static List<Method> findAnnotatedMethods(final Class<?> type,final Class<? extends Annotation> annotation){
		final List<Method> methods = new ArrayList<Method>();
		Method[] ms = type.getDeclaredMethods();
		for(Method method:ms){
			if(Modifier.isStatic(method.getModifiers())){
				continue;
			}
			if(!Modifier.isPublic(method.getModifiers())){
				continue;
			}
			if(method.getParameterTypes().length != 1){
				continue;
			}
			if(!method.isAnnotationPresent(annotation)){
				continue;
			}
			methods.add(method);
		}
		return methods;
	}
}
```

> 实现register(target)方法，该方法需要找出所有符合条件的事件接收器方法

```
public class Bus implements IBus{
	//保存的是该类中对应的所有的带@BusReceiver注解的方法
	private Map<Object,List<Method>> mMethodMap = new HashMap<Object,List<Method>>();
	
	 @Override
    public void register(Object target) {
        List<Method> methods = Helper.findAnnotatedMethods(target.getClass(), BusReceiver.class);
        if (methods == null || methods.isEmpty()) {
            return;
        }
        mMethodMap.put(target, methods);
    }

    @Override
    public void unregister(Object target) {
        mMethodMap.remove(target);
    }

    /**
     * 这个方法会遍历所有注册过target里包含的，接受这个事件事件接收器方法，因此需要从事件对象找到事件接收器
     *
     * @param event
     */
    @Override
    public void post(Object event) {
        final Class<?> eventClass = event.getClass();
        for (Map.Entry<Object, List<Method>> entry : mMethodMap.entrySet()) {
            final Object target = entry.getKey();
            final List<Method> methods = entry.getValue();
            if (methods == null || methods.isEmpty()) {
                continue;
            }
            for (Method method : methods) {
            //如果时间类型相符，就调用对应的方法发送事件
            //这里的类型是要求精确匹配的，没有考虑继承
                if (eventClass.equals(method.getParameterTypes()[0])) {
                    try {
                        method.invoke(target, event);
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

