# Atomic

```java
package priv.king.concurrent;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerFieldUpdater;
import java.util.concurrent.atomic.AtomicReference;

/**
 * @author king
 * TIME: 2020/4/3 - 19:34
 **/
public class AtomicDemo {
    volatile int integer0=3;
    static AtomicInteger integer = new AtomicInteger();
    public static void main(String[] args) {
        new Thread(()->{
            incre();
        }).start();
        new Thread(()->{
            incre();
        }).start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(integer.get());

        //默认false
        //private volatile int value;
        //value 的1，0标识true,false
        AtomicBoolean atomicBoolean = new AtomicBoolean();
        //对Integer引用类型操作
        //必须是volatile int
        AtomicIntegerFieldUpdater atomicIntegerFieldUpdater  =AtomicIntegerFieldUpdater.newUpdater(AtomicDemo.class,"integer0");
        AtomicDemo atomicDemo = new AtomicDemo();
        atomicIntegerFieldUpdater.accumulateAndGet(atomicDemo,2,(x,y)->x+y);
        System.out.println(atomicDemo.integer0);
        //实际上还是private volatile int value;
        //对传参无影响 需要get
        Integer integer1 = new Integer(2);
        AtomicReference<Integer> reference  =new AtomicReference<Integer>(integer1);
        reference.compareAndSet(integer1,new Integer(3));
        System.out.println(integer1);
        System.out.println(reference.get());
        //基本上都是调用Unsafe的方法
    }
    private static void incre(){
        integer.getAndIncrement();
    }
}

```