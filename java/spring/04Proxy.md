# Proxy
## Example


```java
public interface Base {
    public void hello(String name);
}

public class LoginImpl implements Base{
    @Override
    public void hello(String name) {
        System.out.println("welcome "+name+", success !!1");
    }
}

class DynamicProxy implements InvocationHandler {

        Object originalObj;

        Object bind(Object originalObj) {
            this.originalObj = originalObj;
            return Proxy.newProxyInstance(originalObj.getClass().getClassLoader(), originalObj.getClass().getInterfaces(), this);
        }

    /**
     * 切入点 对所有对象的方法都进行调用
     * method.invoke方法对应代理对象调用login方法
     * @param proxy 代理对象
     * @param method 代理对象的方法
     * @param args  代理对象调用接口方法的参数值
     * @return 代理对象调用方法的返回值
     * @throws Throwable
     */
  @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    before();
    Object invoke = method.invoke(originalObj, args);
    if (invoke != null){
        result(invoke);
    }
    after();

  return invoke;
  }

  private void before() {
  System.out.println("方法执行之前");
  }
  private void after() {
  System.out.println("方法执行之后");
  }
  private void result(Object o) {
  o.toString();
  }
}

public class LoginClient {
    public static void main(String[] args) {
  //用于生成代理文件      //System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
        Base hello = (Base) new DynamicProxy().bind(new LoginImpl());
        hello.hello("zhangsan");
    }
}
```
## 反编译结果

```java
import ProxyDemo.Base;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;
//$Proxy0是生成代理的格式决定的
final class $Proxy0 extends Proxy implements Base {
  //将基础的tostring,equils,hashcode,还有base接口的方法生成method的对象
    private static Method m1;
    private static Method m2;
    private static Method m4;
    private static Method m3;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final void hello(String var1) throws  {
        try {
            super.h.invoke(this, m4, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void out() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    //具体的实现
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m4 = Class.forName("ProxyDemo$Base").getMethod("hello", Class.forName("java.lang.String"));
            m3 = Class.forName("ProxyDemo$Base").getMethod("out");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```
## 源码

```
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException {
        Objects.requireNonNull(h);
        //获取需要代理类的所有实现的接口
        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
        //检查是否有生成代理类的权限
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }
        //查找或者生成代理类
        Class<?> cl = getProxyClass0(loader, intfs);

        //生成构造函数
        try {
            if (sm != null) {
         //检查是否有权限
         checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            //public $Proxy0(InvocationHandler var1)
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            //访问修饰符设置
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            //返回代理类的对象
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

```
 private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        //从缓存中获取,如果不存在就创建
        return proxyClassCache.get(loader, interfaces);
    }
```

```
//获取或生成代理类 此处因为不是线程安全的做了多次判断
    public V get(K key, P parameter) {
        Objects.requireNonNull(parameter);
        //删除过期条目
        expungeStaleEntries();
        //创建cacheKey
        Object cacheKey = CacheKey.valueOf(key, refQueue);

        //查看key是否已经存在valuemaps中
        ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
        if (valuesMap == null) {
            //不存在的话通过,再次尝试尝试获取,如果没有就插入
            ConcurrentMap<Object, Supplier<V>> oldValuesMap
                    = map.putIfAbsent(cacheKey,
                    valuesMap = new ConcurrentHashMap<>());
            if (oldValuesMap != null) {
                valuesMap = oldValuesMap;
            }
        }
        //生成代理对象的key 为弱引用类型
        Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
        //尝试从valuemap中获取
        Supplier<V> supplier = valuesMap.get(subKey);
        Factory factory = null;

        while (true) {
            //如果确实已经有线程创建了
            if (supplier != null) {
                //直接获取 supplier might be a Factory or a CacheValue<V> instance
                V value = supplier.get();
                if (value != null) {
                    //最终返回value
                    return value;
                }
            }
            // 不存在创建一个supplier factory实现了supplier
            if (factory == null) {
                factory = new Factory(key, parameter, subKey, valuesMap);
            }


            if (supplier == null) {
                //如果不存在则保存到valuemap中
                supplier = valuesMap.putIfAbsent(subKey, factory);
                if (supplier == null) {
                    // 添加成功
                    supplier = factory;
                }
                // 创建的时候发现已经有了,尝试替换
            } else {
                if (valuesMap.replace(subKey, supplier, factory)) {
                    //替换成功
                    supplier = factory;
                } else {
                    // retry with current supplier
                    supplier = valuesMap.get(subKey);
                }
            }
        }
    }
```

```
//KeyFactory.apply
//根据接口个数的不同选择生成不同的key对象
  public Object apply(ClassLoader classLoader, Class<?>[] interfaces) {
      switch (interfaces.length) {
          case 1: return new Key1(interfaces[0]); // the most frequent
          case 2: return new Key2(interfaces[0], interfaces[1]);
          case 0: return key0;
          default: return new KeyX(interfaces);
      }
  }
```

```
//判断是否存在同时其他线程生成,然后就是尝试着保存添加信息,如果已经有了就尝试替换.最终通过supplier.get()方法获取
//supplier.get()
public synchronized V get() { // serialize access
     // 再次检查是否匹配
     Supplier<V> supplier = valuesMap.get(subKey);
     if (supplier != this) {
         //因为此方法调用之前有可能发生valuesMap.replace(subKey, supplier, factory)
         return null;
     }
     // 创建
     V value = null;
     try {
         //真正的逻辑,重点方法
         value = Objects.requireNonNull(valueFactory.apply(key, parameter));
     } finally {
         if (value == null) { 
             // 如果最终没能生成代理对象,从valuemap移除
             valuesMap.remove(subKey, this);
         }
     }
     // the only path to reach here is with non-null value
     assert value != null;

     //包装value为acacheValue
     CacheValue<V> cacheValue = new CacheValue<>(value);

     // 保存到reverseMap
     reverseMap.put(cacheValue, Boolean.TRUE);

     // 尝试这替换valuemap中的cacheValue
     if (!valuesMap.replace(subKey, this, cacheValue)) {
         throw new AssertionError("Should not reach here");
     }
     return value;
 }

```

```
//value的ProxyClassFactory.apply
//apply方法详解
  public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

      Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
      for (Class<?> intf : interfaces) {
          Class<?> interfaceClass = null;
          try {
              //使用给定的类加载器加载接口
              interfaceClass = Class.forName(intf.getName(), false, loader);
          } catch (ClassNotFoundException e) {
          }
          if (interfaceClass != intf) {
              throw new IllegalArgumentException(
                      intf + " is not visible from class loader");
          }
          //验证是否为接口
          if (!interfaceClass.isInterface()) {
              throw new IllegalArgumentException(
                      interfaceClass.getName() + " is not an interface");
          }
           //验证接口不是重复的
          if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
              throw new IllegalArgumentException(
                      "repeated interface: " + interfaceClass.getName());
          }
      }

      String proxyPkg = null;
      //修饰符
      int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

          /*
           * 验证接口的可见性
           * 如果不是public类型的接口又不在同一个包下抛出异常
           */
      for (Class<?> intf : interfaces) {
          int flags = intf.getModifiers();
          if (!Modifier.isPublic(flags)) {
              accessFlags = Modifier.FINAL;
              String name = intf.getName();
              int n = name.lastIndexOf('.');
              String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
              if (proxyPkg == null) {
                  proxyPkg = pkg;
              }
              //如果不是public类型的接口又不在同一个包下抛出异常
              else if (!pkg.equals(proxyPkg)) {
                  throw new IllegalArgumentException(
                          "non-public interfaces from different packages");
              }
          }
      }

      if (proxyPkg == null) {
          // 没有包使用默认的包 com.sun.proxy
          proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
      }

          /*
           * 代理类的名称 按顺序递增 => $proxy0
           */
      long num = nextUniqueNumber.getAndIncrement();
      String proxyName = proxyPkg + proxyClassNamePrefix + num;

          /*
           * 生成代理类的字节数组
           */
      byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
              proxyName, interfaces, accessFlags);
      try {
          //调用native方法生成Class
          return defineClass0(loader, proxyName,
                  proxyClassFile, 0, proxyClassFile.length);
      } catch (ClassFormatError e) {
          throw new IllegalArgumentException(e.toString());
      }
  }
}
```