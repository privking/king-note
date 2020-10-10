# CGLIB

```java
public class CglibTest {
    public void function(){
        System.out.println("hello world");
    }

    @Test
    public  void test1() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(CglibTest.class);
        /**
        enhancer.setCallbackFilter();
        enhancer.setCallbacks();
        enhancer.setInterfaces();
        //生成策略
        enhancer.setStrategy();
        **/

        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                System.out.println("before method run...");
                Object result = proxy.invokeSuper(obj, args);
                System.out.println("after method run...");
                return result;
            }
        });
        CglibTest test = (CglibTest) enhancer.create();
        test.function();
        /**
         * before method run...
         * hello world
         * after method run...
         */
    }

    /**
     * ImmutableBean允许创建一个原来对象的包装类，这个包装类是不可变的，
     * 任何改变底层对象的包装类操作都会抛出IllegalStateException。
     * 但是我们可以通过直接操作底层对象来改变包装类对象。
     */
    @Test
    public void testImmutableBean(){
        Simple bean = new Simple();
        bean.setValue("Hello world");
        Simple immutableBean = (Simple) ImmutableBean.create(bean); //创建不可变类
        System.out.println(immutableBean.getValue());
        bean.setValue("Hello world, again"); //可以通过底层对象来进行修改
        System.out.println(immutableBean.getValue());
        try {
            immutableBean.setValue("Hello cglib"); //直接修改将throw exception
        } catch (Exception e) {
            System.out.println(e);
        }
        /**
         * Hello world
         * Hello world, again
         * java.lang.IllegalStateException: Bean is immutable
         */
    }

    /**
     * Bean generator
     * cglib提供的一个操作bean的工具，使用它能够在运行时动态的创建一个bean。
     */
    @Test
    public void testBeanGenerator() throws Exception{
        BeanGenerator beanGenerator = new BeanGenerator();
        beanGenerator.addProperty("value",String.class);
        Object myBean = beanGenerator.create();
        Method setter = myBean.getClass().getMethod("setValue",String.class);
        setter.invoke(myBean,"Hello cglib");

        Method getter = myBean.getClass().getMethod("getValue");
        System.out.println("getter.invoke(myBean) = " + getter.invoke(myBean));
        /**
         * getter.invoke(myBean) = Hello cglib
         */
    }

    /**
     * Bean Copier
     * cglib提供的能够从一个bean复制到另一个bean中，
     * 而且其还提供了一个转换器，
     * 用来在转换的时候对bean的属性进行操作。
     */
    @Test
    public void testBeanCopier() throws Exception{
        BeanCopier copier = BeanCopier.create(Simple.class, Simple2.class, true);//设置为true，则使用converter
        Simple myBean = new Simple();
        myBean.setValue("Hello cglib");
        Simple2 otherBean = new Simple2();
        copier.copy(myBean, otherBean, new Converter(){
            @Override
            public Object convert(Object value, Class target, Object context) {
                return value+" convert";
            }
        }); //设置为true，则传入converter指明怎么进行转换
        System.out.println("otherBean.getValue() = " + otherBean.getValue());
        //otherBean.getValue() = Hello cglib convert
    }

    /**
     * BeanMap
     * BeanMap类实现了Java Map，
     * 将一个bean对象中的所有属性转换为一个String-to-Obejct的Java Map
     */

    @Test
    public void testBeanMap() throws Exception{
        BeanGenerator generator = new BeanGenerator();
        generator.addProperty("username",String.class);
        generator.addProperty("password",String.class);
        Object bean = generator.create();
        Method setUserName = bean.getClass().getMethod("setUsername", String.class);
        Method setPassword = bean.getClass().getMethod("setPassword", String.class);
        setUserName.invoke(bean, "admin");
        setPassword.invoke(bean,"password");
        BeanMap map = BeanMap.create(bean);
        System.out.println("map.get(\"username\") = " + map.get("username"));
        System.out.println("map.get(\"password\") = " + map.get("password"));
    }
    /**
     * keyFactory
     * keyFactory类用来动态生成接口的实例，
     * 接口需要只包含一个newInstance方法，返回一个Object。
     * keyFactory为构造出来的实例动态生成了Object.equals和Object.hashCode方法，
     * 能够确保相同的参数构造出的实例为单例的。
     *
     */
    @Test
    public void testKeyFactory() throws Exception{
        SampleKeyFactory keyFactory = (SampleKeyFactory) KeyFactory.create(SampleKeyFactory.class);
        Object key = keyFactory.newInstance("foo", 42);
        Object key1 = keyFactory.newInstance("foo", 42);
        System.out.println("key = " + key+"  "+key.hashCode());
        System.out.println("key1 = " + key1+"  "+key1.hashCode());
        /**
         * key = foo, 42  -1181409833
         * key1 = foo, 42  -1181409833
         */
    }
    /**
     * Interface Maker
     * 正如名字所言，Interface Maker用来创建一个新的Interface
     */

    @Test
    public void testInterfaceMarker() throws Exception{
        Signature signature = new Signature("foo", Type.DOUBLE_TYPE, new Type[]{Type.INT_TYPE});
        InterfaceMaker interfaceMaker = new InterfaceMaker();
        interfaceMaker.add(signature, new Type[0]);
        Class iface = interfaceMaker.create();
        System.out.println("iface.getMethods().length = " + iface.getMethods().length);
        System.out.println("iface.getMethods()[0].getName() = " + iface.getMethods()[0].getName());
        System.out.println("iface.getMethods()[0].getReturnType() = " + iface.getMethods()[0].getReturnType());
        /**
         * iface.getMethods().length = 1
         * iface.getMethods()[0].getName() = foo
         * iface.getMethods()[0].getReturnType() = double
         *
         */

    }
}
```
## Enhancer-CallBack
###  Callback-MethodInterceptor
* 方法拦截器
* `Object intercept(Object obj, java.lang.reflect.Method method, Object[] args, MethodProxy proxy) throws Throwable`
* MethodProxy proxy参数一般是用来调用原来的对应方法的。比如可以proxy.invokeSuper(obj, args)。那么为什么不能像InvocationHandler那样用method来调用呢？因为如果用method调用会再次进入拦截器。为了避免这种情况，应该使用接口方法中第四个参数methodProxy调用invokeSuper方法。

### Callback-NoOp
* 这个回调相当简单，就是啥都不干的意思

### Callback-LazyLoader
* LazyLoader是cglib用于实现懒加载的callback。当被增强bean的方法初次被调用时，会触发回调，之后每次再进行方法调用都是对LazyLoader第一次返回的bean调用。

```java
public class EnhancerTest {

    public static void main(String[] args) {
        CarFactory factory = new CarFactory();
        System.out.println("factory built");
        System.out.println(factory.car.getName());
        System.out.println(factory.car.getName());
    }

    static class Car {
        String name;
        Car() {
        }

        String getName() {
            return name;
        }
    }

    static class CarFactory {
        Car car;

        CarFactory() {
            car = carLazyProxy();
        }

        Car carLazyProxy() {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(Car.class);
            enhancer.setCallback(new LazyLoader() {
                @Override
                public Object loadObject() throws Exception {
                    System.out.println("prepare loading");
                    Car car = new Car();
                    car.name = "this is a car";
                    System.out.println("after loading");
                    return car;
                }
            });
            return ((Car) enhancer.create());
        }
    }
}
/**
factory built
prepare loading
after loading
this is a car
this is a car
**/
```
### Callback-Dispatcher
* Dispatcher和LazyLoader作用很相似，区别是用Dispatcher的话每次对增强bean进行方法调用都会触发回调。

```java
public class EnhancerTest {

    public static void main(String[] args) {
        CarFactory factory = new CarFactory();
        System.out.println("factory built");
        System.out.println(factory.car.getName());
        System.out.println(factory.car.getName());
    }

    static class Car {
        String name;
        Car() {
        }

        String getName() {
            return name;
        }
    }

    static class CarFactory {
        Car car;

        CarFactory() {
            car = carLazyProxy();
        }

        Car carLazyProxy() {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(Car.class);
            enhancer.setCallback(new Dispatcher() {
                @Override
                public Object loadObject() throws Exception {
                    System.out.println("prepare loading");
                    Car car = new Car();
                    car.name = "this is a car";
                    System.out.println("after loading");
                    return car;
                }
            });
            return ((Car) enhancer.create());
        }
    }
}
/**
factory built
prepare loading
after loading
this is a car
prepare loading
after loading
this is a car
**/
```

### Callback-InvocationHandler
* 如果对参数中的method再次调用，会重复进入InvocationHandler。

```java
public class EnhancerTest {

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Car.class);
        enhancer.setCallback(new InvocationHandler() {
            @Override
            public Object invoke(Object obj, Method method, Object[] args) throws Throwable {
                if (method.getReturnType() == void.class) {
                    System.out.println("hack");
                }
                return null;
            }
        });
        Car car = (Car) enhancer.create();
        car.print();
    }

    static class Car {
        void print() {
            System.out.println("I am a car");
        }
    }
}
```
### Callback-FixedValue
* FixedValue一般用于替换方法的返回值为回调方法的返回值，但必须保证返回类型是兼容的，否则会出转换异常。

```
public class EnhancerTest {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Car.class);
        enhancer.setCallback(new FixedValue() {
            @Override
            public Object loadObject() throws Exception {
                return "hack!";
            }
        });

        Car car = (Car) enhancer.create();
        System.out.println(car.print1());
        System.out.println(car.print2());
    }

    static class Car {
        String print1() {
            return "car1";
        }
        String print2() {
            return "car2";
        }
    }
}
```
## CallbackFilter
* 上面都是为增强bean配置了一种代理callback，但是当需要作一些定制化的时候，CallbackFilter就派上用处了。
* 当通过设置CallbackFilter增强bean之后，bean中原方法都会根据设置的filter与一个特定的callback映射。我们通常会使用cglib中CallbackFilter的默认实现CallbackHelper，它的getCallbacks方法可以返回生成的callback数组。


```java 
///被代理的类
public class ConcreteClassNoInterface {
	public String getConcreteMethodA(String str){
		System.out.println("ConcreteMethod A ... "+str);
		return str;
	}
	public int getConcreteMethodB(int n){
		System.out.println("ConcreteMethod B ... "+n);
		return n+10;
	}
	public int getConcreteMethodFixedValue(int n){
		System.out.println("getConcreteMethodFixedValue..."+n);
		return n+10;
	}
}

//CallbackFilter
public class ConcreteClassCallbackFilter implements CallbackFilter{
	public int accept(Method method) {
		if("getConcreteMethodB".equals(method.getName())){
			return 0;//Callback callbacks[0]--->interceptor
		}else if("getConcreteMethodA".equals(method.getName())){
			return 1;//Callback callbacks[1]--->noOp
		}else if("getConcreteMethodFixedValue".equals(method.getName())){
			return 2;//Callback callbacks[2]--->fixedValue
		}
		return 1;//Callback callbacks[1]--->noOp
	}
}
//
Enhancer enhancer=new Enhancer();
enhancer.setSuperclass(ConcreteClassNoInterface.class);
CallbackFilter filter=new ConcreteClassCallbackFilter();
enhancer.setCallbackFilter(filter);
 
Callback interceptor=new ConcreteClassInterceptor();
Callback noOp=NoOp.INSTANCE;
Callback fixedValue=new ConcreteClassFixedValue();
Callback[] callbacks=new Callback[]{interceptor,noOp,fixedValue};
enhancer.setCallbacks(callbacks);
ConcreteClassNoInterface proxyObject=(ConcreteClassNoInterface)enhancer.create();

```