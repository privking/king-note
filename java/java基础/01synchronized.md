# synchronized



## synchronized用法

```java
public class ThreadDemo {
    //类锁
    public synchronized static void access(){
        System.out.println("1111");
    }
    //对象锁
    public synchronized  void access1(){
        System.out.println("2222");
    }
    //对象锁
    public void access2(){
        synchronized (this){
            System.out.println("3333");
        }
    }
    //类锁
    public static void access3(){
        synchronized (ThreadDemo.class) {
            System.out.println("4444");
        }
    }
    public static void main(String[] args) {
        ThreadDemo demo = new ThreadDemo();
        new Thread(ThreadDemo::access).start();
        new Thread(ThreadDemo::access3).start();
        new Thread(demo::access1).start();
        new Thread(demo::access2).start();
    }
}
```

## 原理
- 在 Java 中，每个对象都会有一个 monitor 对象，监视器。
- 某一线程占有这个对象的时候，先monitor 的计数器是不是0，如果是0还没有线程占有，这个时候线程占有这个对象，并且对这个对象的monitor+1；如果不为0，表示这个线程已经被其他线程占有，这个线程等待。当线程释放占有权的时候，monitor-1；
- 同一线程可以对同一对象进行多次加锁，+1，+1，重入性

## 原理分析
### jstack pid
![1585836098(1)](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1585836098-1--1599154113-2821f3.png)
### javap -v ThreadDemo.class
- 对象锁
![1585836427(1)](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1585836427-1--1599154160-6b895c.png)
- 类锁
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154183-00991e.png)

## 使用synchronized注意的问题
- 与moniter关联的对象不能为空
- synchronized作用域太大
- 不同的monitor企图锁相同的方法
- 多个锁的交叉导致死锁

## Java虚拟机对synchronized的优化
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154222-d8bd0a.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154254-53fde7.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154273-87aba8.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154291-4ff6e2.png)



![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154333-229ab0.png)

- 初期锁对象刚创建时，还没有任何线程来竞争，对象的Mark Word是下图的第一种情形，这**偏向锁标识位是0，锁状态01**，说明该对象处于无锁状态（无线程竞争它）。
- 当有一个线程来竞争锁时，先用偏向锁，表示锁对象偏爱这个线程，这个线程要执行这个锁关联的任何代码，不需要再做任何检查和切换，这种竞争不激烈的情况下，效率非常高。这时Mark Word会记录自己偏爱的线程的ID，把该线程当做自己的熟人。如下图第二种情形。
- 当有两个线程开始竞争这个锁对象，情况发生变化了，不再是偏向（独占）锁了，锁会升级为轻量级锁，两个**线程公平竞争**，**哪个线程先占有锁对象并执行代码，锁对象的Mark Word就执行哪个线程的栈帧中的锁记录**。如下图第三种情形。
- 如果竞争的这个锁对象的线程更多，导致了更多的切换和等待，JVM会把该锁对象的锁升级为重量级锁，这个就叫做同步锁，这个锁对象Mark Word再次发生变化，会指向一个**监视器对象**，这个监视器对象用集合的形式，来登记和管理排队的线程

![img](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/20190111091608949-1605752086-4f6ae1.jpeg)