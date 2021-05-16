# AQS

## 独占模式 
### 独占模式下请求资源

```java
public final void acquire(int arg) {
    	//acquireQueued 主要就是让线程睡在一个死循环中，醒过来的时候可以获取锁
    	//acquireQueued返回值是是否被打断
    	//在lock()方法下被打断是返回true,然后在这里重新将标识设置为true
    	//在lockInterruptibly()方法下 ，判断被打断了就直接抛异常
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```
* 其中tryAcquire方法是子类需要去实现。它的实现内容主要是独占式的抢占对应的资源。因为是独占式抢占，所以在抢占前需要判断当前的资源是否符合抢占的要求。如果抢占失败，则尝试将当前的线程放入一个等待队列。

### addWaiter

```java
private Node addWaiter(Node mode) {
    	//在独占模式想  mode为null
          Node node = new Node(Thread.currentThread(), mode);
        // 首先是按照MPSC入队算法进行第一轮尝试，如果尝试成功就不需要走路到下面的循环流程。
    	//tail 默认为null
        Node pred = tail;
        if (pred != null) {
            //将新创建的node prev指向tail
            node.prev = pred;
            //将新创建的node设置到tail
            if (compareAndSetTail(pred, node)) {
                //将原来的尾节点指向新的Node
                pred.next = node;
                //链表结构已经维护好了 可以直接返回
                return node;
            }
        }
    	//1.链表并没有初始化好
    	//2.出现并发情况，没有成功维护好链表
        enq(node);
        return node;
    }
     private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { 
                    // 首先是初始化head和tail。使用了cas是因为这一步存在并发可能。
                //该node的Thread为null
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
            //重复循环，执行上面的入队算法，直到成功。
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
    private final boolean compareAndSetHead(Node update) {
        return unsafe.compareAndSwapObject(this, headOffset, null, update);
    }
    private final boolean compareAndSetTail(Node expect, Node update) {
        return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
    }
```
### acquireQueued

```java
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                //获取到Node的前一个Node
                final Node p = node.predecessor();
                //如果一个节点的前置节点是head节点，则意味着该节点可以进行尝试抢占资源。
                if (p == head && tryAcquire(arg)) {
                    //抢占资源成功的情况下，可以设置自身节点为head节点。这里不存在并发，所以直接设置。
                    //从代码的全局来看，队列中的head节点代表着是当前持有资源的*某一个线程*
                    //这里的某一个是因为在共享模式中，head可能意味着多个持有资源线程中的任意一个。
                    //而在独占模式中，head节点指代的是持有资源的线程。
                    //这个head节点未必就是当前持有资源线程构建的，也有可能是后面的抢占资源失败的线程在入队之后构建。
                    //无论如何，在独占模式中，head节点均代表当前持有资源的线程。
                    //无论head节点中的thread是否有值。
                    
                    //将自己设置为头节点
                    setHead(node);
                    //取消原来头节点的next指向
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                //shouldParkAfterFailedAcquire主要是为了将将前一个没有cancel的waitStatus设置为-1
                //唤醒的时候知道还有后一个节点
                //parkAndCheckInterrupt 主要是将线程park起来，然后醒过来的时候返回是否被打断这个状态
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
    //由于只有本节点的前置节点为head时才能调用该方法，该方法没有并发，直接设置既可
    private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }
```
* 在入队完毕之后，就进入等待，和不断被唤醒尝试抢占的过程。也就是acquireQueued的代码表达
* 既然是涉及到了线程挂起，那么必然存在一个标识位。AQS选择将该标识位放在前置节点中。这个标志位是一个int参数。值为-1（Node.SINGLE）的时候，意味着后续节点处于线程挂起状态。

### shouldParkAfterFailedAcquire 线程挂起方法

```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            //如果前置节点的状态已经是signal，意味着已经经历过二次尝试，可以返回true，让线程进入等待状态。
            return true;
        if (ws > 0) {
            //如果前置节点处于取消状态，则不断的向前回溯，直到找到一个不是取消状态的节点。无论如何，至少head节点不会是取消状态，所以最终一定可以找到一个不是取消状态的前置节点。然后将该node的pre指针指向该非取消状态节点。在这个循环中就将AQS中的内部队列的长度缩短了。
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //将前置节点的状态变更为signal。
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        //返回false，提示调用者线程仍然需要继续尝试，不可以进入休眠状态
        return false;
    }
    //让线程进入等待状态，并且返回线程的中断信息
     private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```
### cancelAcquire
* 正常情况下的流程走到上面也就结束了，而如果出现了问题，导致最终失败。则进入节点的取消流程。

```java
private void cancelAcquire(Node node) {
        if (node == null)
            return;
       //首先设置线程为null，避免其他地方导致的唤醒。主要是防止浪费
        node.thread = null;
       //整理内部队列，从该节点开始向前回溯，找到一个不是取消状态的节点
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
        //获得了新的前置节点的next值。
        Node predNext = pred.next;
        //设置节点状态为取消。一旦节点进入该状态则无法变更。而其他的节点变更用的都是cas。所以这里不需要cas，直接写入即可。写入后便不会更改。
        node.waitStatus = Node.CANCELLED;
       //如果自身节点是尾节点，则尝试将之前发现的前置节点设置为尾节点
        if (node == tail && compareAndSetTail(node, pred)) {
            //如果设置成功，则意味着当前节点之后没有后继节点。没有后继节点就不需要唤醒
            //对前置节点的next值进行cas更替。这里要采用cas的原因是为了避免对正常的入队流程造成错误影响。
            //假设取消线程走到这里，有其他线程在入队，入队完毕后必然要设置pred节点的next值。无论是否在该cas之前，其他线程的入队操作最终都应该成功。而这里采用了cas，是为了避免其他线程在入队操作成功后，正确的next值被这里错误的next值覆盖。因为这里的predNext是旧值，所以只应该在next仍然是prexNext的时候才能设置为null
            compareAndSetNext(pred, predNext, null);
        } else {
            //如果该节点不是尾节点，或者设置更新尾节点失败，都意味着该节点还有后继节点。有后继结点就应该尝试执行唤醒操作
            //但是如果所有的取消都唤醒后继结点则意味着不必要的浪费。因此有必要识别出什么情况下是不需要唤醒的。下面的if判断就是在识别，什么情况下，无需唤醒后继结点。
            int ws;
               //如果前置节点是head节点，那么就需要帮忙唤醒后继节点。因为可能前置节点已经释放了资源，并且唤醒后继节点也就是该节点。然而前置节点执行唤醒方法时本节点已经进入取消流程。所以这里要唤醒本节点的后置节点，否则就没有线程去唤醒了。也就意味着唤醒传播被中断。造成死锁。
               //如果前置节点在cas尝试的过程中进入了cancel状态，那就主动进行唤醒。这种情况也是必须执行唤醒的。假设前置节点进入cancel状态，假设前置节点唤醒判定时仍然选择本节点进行唤醒（这是最糟糕的情况，会导致唤醒传播中断，如果此时本节点不唤醒后继结点），而本节点在cancel进程中，自然也不会响应。那如果本节点不唤醒后继结点，则意味着唤醒传播整体中断。
              //总结来说，只有在前置节点不是head节点的情况下并且前置节点的状态成功更新为signal才可以说不要唤醒，单纯设置前置节点的next值。因为能正确设置signal就意味着还会有后续流程，唤醒传播不会断绝
            if (pred != head 
                    &&
                ( (ws = pred.waitStatus) == Node.SIGNAL ||  ( ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL)) )
                    &&
                pred.thread != null
                    )
            {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    //cas操作。因为在代码执行的过程中，竞争线程可能会走的更快，pred的后继结点是哪一个都会出现竞争，而如果出现尾节点的竞争，没有cas就会出现错误。具体原因和上面描述想同。
                    compareAndSetNext(pred, predNext, next);
            }
            else 
            {
                unparkSuccessor(node);
            }
            //一旦一个节点状态进入canceled，则next值失去意义，所以这一步在最后完成。
            node.next = node; // help GC
        }
    }
```
### unparkSuccessor唤醒后续节点

```java
private void unparkSuccessor(Node node) {
        //通过cas，更换当前节点状态为0.这样主要的目的也是让后继节点在挂起钱继续更多尝试。所以这个cas是否失败无所谓。但是要注意是需要原生状态不为cancel的情况下，才能执行这个cas。因为cancel状态进入后不能改变。
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        Node s = node.next;
        //寻找合适的后继节点。如果后继结点为null或者状态为取消，则从尾节点开始，不断的向前回溯。直到找到一个非取消状态，排位最靠前的节点。然后进行唤醒。从后向前回溯的原因是在该内部队列内，只有pre指向是正确并且完整的。next指向为null时也许是入队没走完，甚至有些时候有可能是不打扰语义的错误数据。所以必须使用pre指向来进行回溯。
       //唤醒后继节点可能会遭遇并发，可能一个thread会被多次唤醒。不过这并不会影响到程序语义。所以没有关系。
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```
### 独占模式下释放资源

```java
public final boolean release(int arg) {
        //子类需要实现的独占式释放锁
        if (tryRelease(arg)) {
            //独占模式中，持有资源的线程对应的必然是head节点
            Node h = head;
           //如果head节点为null，那就意味着在该线程持有资源到释放资源这段时间都没有竞争，自然head节点为null，也就是没有后继结点
           //如果head节点不为空，并且不是初始化状态。就尝试唤醒head节点的后继节点。
           //如果head节点的状态是0，那就不需要唤醒，因为后继节点会二次重试，不会陷入挂起。 
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```
### tryAcquireNanos
* 独占式获取，还有一种情况就是带超时时间的独占获取


```java
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        //进行尝试获取，失败则尝试尝试超时获取
        return tryAcquire(arg) ||  doAcquireNanos(arg, nanosTimeout);
    }
private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (nanosTimeout <= 0L)
            return false;
        final long deadline = System.nanoTime() + nanosTimeout;
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
                nanosTimeout = deadline - System.nanoTime();
                if (nanosTimeout <= 0L)
                    return false;
                if (shouldParkAfterFailedAcquire(p, node) &&
                    nanosTimeout > spinForTimeoutThreshold)
                    //主要就是在这句话上，这个api调用的线程挂起是存在超时时间的，时间过后线程会自动恢复。
                    LockSupport.parkNanos(this, nanosTimeout);
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

## 共享模式
### acquireShared

```java
public final void acquireShared(int arg) {
        //一个需要子类实现的共享获取尝试。这个方法要求子类实现一种共享模式下的资源请求。说白了，其实就是资源的总数大于1，因而可以同时存在多个线程持有该资源。方法返回负数意味着请求资源失败，开始进入到入队操作。
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
//这个方法大致上看起来和独占模式是很相像的。区别只在于独占模式下，在本方法中获取到资源后，只是将本节点设置为tail节点。而共享模式下，设置完tail节点后，还需要做更多的事情。
private void doAcquireShared(int arg) {
    	//将当前线程包装为一个Node加到尾节点
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    // tryAcquireShared按照jdk文档中的注解应该返回是资源的剩余数。如果大于0，显然应该让后面等待的线程也参与获取。而等于0的时候其实只是需要将本节点设置为head既可。不过那样会让代码变的麻烦。所以这里只要返回结果非负，就执行设置head节点和传播的语句。
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head;
        setHead(node);
        //什么情况下要唤醒后继结点？
        //1.资源剩余数大于0，有剩余资源肯定是要唤醒后继结点的
        //2.头结点不存在。可以看到这个条件会与后文的head！=null相冲突。而且实际上在这个方法执行的时候，head节点是必然存在的，不可能为null。留待下篇文章再做解读。
        //3.头结点状态小于0.这里只有两种可能，一种是SIGNAL（-1），一种PROPAGATE（-3）。这两种数值的出现都意味着后继节点要求node（也就是当前head）唤醒后继结点
        if (
           propagate > 0 
           || h == null 
           || h.waitStatus < 0 
           || (h = head) == null 
 || h.waitStatus < 0)
 {
            Node s = node.next;
            //这里的if判断并非是一个无意义的防御性编程。在AQS的实现类中，存在着所谓读写锁。也就是说在AQS的内部队列，共享节点和独占节点是都存在的。所以共享唤醒传播到独占节点就要停止了。
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```
### doReleaseShared
* 具体的共享唤醒实现

```java
private void doReleaseShared() {
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
                    //由于该方法可能被并发调用，为了避免不必要的唤醒浪费，因为通过cas来抢占唤醒权利。抢占成功者执行真正的后继结点唤醒任务。如果失败则进行重试                 
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            
                    unparkSuccessor(h);
                }
                //如果ws==0，可能是以为head结点已经没有后继结点，也有可能是因为head节点的后继结点唤醒权被其他线程刚抢占成功。
                //如果没有后继结点，显然不需要做任何事情
                //如果是唤醒权被其他线程抢占，则不需要做任何事情。因为唤醒是在队列上进行传播的。所以这里就cas将head节点的状态值修改为 PROPAGATE。用来表达该线程唤醒操作意图已经传达。但是会由别的线程真正的执行后续的唤醒动作。同样，如果失败了，则重试。
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;  
               //其实这里缺少一个else if.用来对h.waitstatus== PROPAGATE做条件说明。如果发现节点状态是 PROPAGATE，说明已经有其他线程尝试抢夺唤醒权失败，也就是已经有线程真正持有了唤醒权，那么本线程就不需要做什么了。
            }
            //如果在上面的流程结束后发现head节点没有变化，意味着本次修改过程队列前端是无变化的。因此方法到这里结束
            //而如果head节点发生了改变，则需要重试上面的过程。这个重试是否是必要的？笔者因为没有找到如果不使用这段代码会导致问题的场景，故而对这段代码的合理性表示存疑。
            //目前来看，这个条件只是用来单纯的简单退出。使用这个条件比较简单粗暴，就忽略细分场景下的错误情况。统一的，如果头结点未变化就退出。会造成一定程度的唤醒浪费。
            if (h == head)            
                break;
        }
    }
```
### 释放资源

```java
 public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            doReleaseShared();
            return true;
        }
        return false;
    }
```

## Condition
### await
* 这个方法用于在获取了等待某一个条件满足，在等待时会释放掉当前持有的锁。下面来看代码实现

```java
public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            //创建一个创建一个条件节点，并且放入条件队列
            Node node = addConditionWaiter();
            //将当前持有的锁完全释放
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            //如果当前节点不在AQS的内部队列中，则保持等待状态。除非线程中断，或者是节点取消。
            //节点实际上是会被signal方法从条件队列移动到等待队列。
            while (!isOnSyncQueue(node)) {
                LockSupport.park(this);
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            //进入锁资源争夺
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            //争夺到锁资源后，帮忙清除条件队列中已经取消的节点
            if (node.nextWaiter != null)
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
private Node addConditionWaiter() {
            Node t = lastWaiter;
            //如果发现尾节点已经处于取消状态，则清理该条件队列上的节点。
            if (t != null && t.waitStatus != Node.CONDITION) {
                //清除条件队列上所有状态为cancel的节点。具体实现不复杂，这里不展开说明
                unlinkCancelledWaiters();
                t = lastWaiter;
            }
            //为本线程创建一个节点，并且放到lastWaiter的next位置上，然后设置lastWaiter的值。因为这个方法在持有锁的情况下执行，所以不需要担心并发。
            Node node = new Node(Thread.currentThread(), Node.CONDITION);
            if (t == null)
                firstWaiter = node;
            else
                t.nextWaiter = node;
            lastWaiter = node;
            return node;
        }
final int fullyRelease(Node node) {
        try {
            int savedState = getState();
            if (release(savedState))
                return savedState;
        } catch (RuntimeException ex) {
            node.waitStatus = Node.CANCELLED;
            throw ex;
        }
        //如果程序走入到这里，说明上面的释放返回了false。则意味着调用出现了异常。多半是因为没有获取持有锁的情况下，就执行await操作导致。
        node.waitStatus = Node.CANCELLED;
        throw new IllegalMonitorStateException();
    }
```
### signal
* 该方法用于唤醒在某个条件上等待的某一个线程


```java
 public final void signal() {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }
private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
final boolean transferForSignal(Node node) {
       //只有一种情况会失败，该节点进入了cancel状态。 
       if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        Node p = enq(node);
        int ws = p.waitStatus;
        //前置节点已经取消，则唤醒该节点线程执行一些同步动作。比如节点连接到合适的前置节点
        //更换前置状态为signal失败，有可能是因为进入了取消状态。此时要唤醒该节点的线程，让其自行执行一些同步动作。
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
```
## 为什么需要SIGNAL状态
* 在共享唤醒中，多线程并发争夺唤醒权，必然存在一个cas的过程。也就是需要一个从有状态值cas到0的过程。所以要存在这样的一个状态值，最后就是SIGNAL了。从另外一个角度来看，节点一旦进入取消状态就不可恢复，因此需要存在一个不同的状态用来表示该节点需要唤醒，这也就是signal。

## 为什么需要PROPAGATE状态
* 在共享唤醒中，所有的节点都不断的抢夺唤醒权是没有意义而且浪费的。同时需要一个与初始状态不同的状态用来表达多线程竞争唤醒权的结果。因为从SIGNAL到0是表示唤醒权被某一个线程抢夺完成，因此需要有一个额外的状态可以用来通知其他竞争线程可以停止竞争了。所以就有了 PROPAGATE状态


## stampedLock
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154603-d53289.png)

## CyclicBarrier
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154628-c20855.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154646-45ddbe.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154662-7f2c69.png)


```java
public class CyclicBarrierDemo {

    static class TaskThread extends Thread {
        
        CyclicBarrier barrier;
        
        public TaskThread(CyclicBarrier barrier) {
            this.barrier = barrier;
        }
        
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
                System.out.println(getName() + " 到达栅栏 A");
                barrier.await();
                System.out.println(getName() + " 冲破栅栏 A");
                
                Thread.sleep(2000);
                System.out.println(getName() + " 到达栅栏 B");
                barrier.await();
                System.out.println(getName() + " 冲破栅栏 B");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) {
        int threadNum = 5;
        CyclicBarrier barrier = new CyclicBarrier(threadNum, new Runnable() {
            
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " 完成最后任务");
            }
        });
        
        for(int i = 0; i < threadNum; i++) {
            new TaskThread(barrier).start();
        }
    }
    
}
```

```
Thread-1 到达栅栏 A
Thread-3 到达栅栏 A
Thread-0 到达栅栏 A
Thread-4 到达栅栏 A
Thread-2 到达栅栏 A
Thread-2 完成最后任务
Thread-2 冲破栅栏 A
Thread-1 冲破栅栏 A
Thread-3 冲破栅栏 A
Thread-4 冲破栅栏 A
Thread-0 冲破栅栏 A
Thread-4 到达栅栏 B
Thread-0 到达栅栏 B
Thread-3 到达栅栏 B
Thread-2 到达栅栏 B
Thread-1 到达栅栏 B
Thread-1 完成最后任务
Thread-1 冲破栅栏 B
Thread-0 冲破栅栏 B
Thread-4 冲破栅栏 B
Thread-2 冲破栅栏 B
Thread-3 冲破栅栏 B
```

## Semaphore
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599154684-563c62.png)



## ReentrantLock

```
ReentrantLock{
Sync{
    extends AbstractQueuedSynchronizer{
		state
		Node tail{
			waitStatus
			prev
			next
			thread
			nextWaiter(SHARED,EXCLUSIVE)
		}
		Node head

    }
}
NonfairSync
FairSync

}
```



内部维护了公平锁和非公平锁

公平锁：acquire 先判断有没有锁，判断需不需要排队，是不是重入锁

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

非公平锁

```java
final void lock() {
    //先就试一试
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}

final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                //我也不看排不排队
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

reentrantLock.lockInterruptibly()

```java
private void doAcquireInterruptibly(int arg)
    throws InterruptedException {
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                //这里直接抛出了打断异常
                //在lock中是将interrupt设置为true返回回去，然后再将设置为interrupt状态，并没有抛异常
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

interrupt

- 在sleep的时候会被唤醒并抛出异常
- 在park的时候，会被唤醒但是没有抛出异常
- 在你运行的时候需要手动去判断，要不然就相当于只改了一个状态值

 

## ReentrantReadWriteLock

state高16位为读锁，低16位为写锁，支持锁降级（写锁是排他的）

不支持锁升级原因（读锁是共享的）：读锁升级为写锁，首先写锁的是需要没有读锁的，就是加上与当前线程绑定，**每个读锁来升级的时候也需要等待其他读锁释放，就会造成死锁**

rwLock.writeLock().lock();

```java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
    //获取state
    int c = getState();
    //获取写锁数量
    int w = exclusiveCount(c);
    //表明有锁
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        
        //写锁是0（有共享锁没有释放）
        //或者是说自己不是占用写锁的那个线程
        //表示没有获取到锁
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //超出了最大的重入次数
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        //修改state,获取锁成功
        setState(c + acquires);
        return true;
    }
    //表示没有锁
    //判断需不需要排队（非公平锁直接就不需要排队，公平锁去双向链表里面看有没有）
    //不需要排队的话就尝试加锁
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```



rwLock.writeLock().unlock();

```java
public final boolean release(int arg) {
    	//释放成功就去唤醒后面的线程
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    //releases == 1
    int nextc = getState() - releases;
    //写锁的数量为0表示
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

rwLock.readLock().lock();

```java
protected final int tryAcquireShared(int unused) {
    /*
     * Walkthrough:
     * 1. If write lock held by another thread, fail.
     * 2. Otherwise, this thread is eligible for
     *    lock wrt state, so ask if it should block
     *    because of queue policy. If not, try
     *    to grant by CASing state and updating count.
     *    Note that step does not check for reentrant
     *    acquires, which is postponed to full version
     *    to avoid having to check hold count in
     *    the more typical non-reentrant case.
     * 3. If step 2 fails either because thread
     *    apparently not eligible or CAS fails or count
     *    saturated, chain to version with full retry loop.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    //有写锁，或者是当前线不是不是占有写锁的线程，返回-1，<0表示加锁失败
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    //获取读锁的数量
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}

final int fullTryAcquireShared(Thread current) {
            /*
             * This code is in part redundant with that in
             * tryAcquireShared but is simpler overall by not
             * complicating tryAcquireShared with interactions between
             * retries and lazily reading hold counts.
             */
            HoldCounter rh = null;
            for (;;) {
                int c = getState();
                if (exclusiveCount(c) != 0) {
                    if (getExclusiveOwnerThread() != current)
                        return -1;
                    // else we hold the exclusive lock; blocking here
                    // would cause deadlock.
                } else if (readerShouldBlock()) {
                    // Make sure we're not acquiring read lock reentrantly
                    if (firstReader == current) {
                        // assert firstReaderHoldCount > 0;
                    } else {
                        if (rh == null) {
                            rh = cachedHoldCounter;
                            if (rh == null || rh.tid != getThreadId(current)) {
                                rh = readHolds.get();
                                if (rh.count == 0)
                                    readHolds.remove();
                            }
                        }
                        if (rh.count == 0)
                            return -1;
                    }
                }
                if (sharedCount(c) == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                if (compareAndSetState(c, c + SHARED_UNIT)) {
                    if (sharedCount(c) == 0) {
                        firstReader = current;
                        firstReaderHoldCount = 1;
                    } else if (firstReader == current) {
                        firstReaderHoldCount++;
                    } else {
                        if (rh == null)
                            rh = cachedHoldCounter;
                        if (rh == null || rh.tid != getThreadId(current))
                            rh = readHolds.get();
                        else if (rh.count == 0)
                            readHolds.set(rh);
                        rh.count++;
                        cachedHoldCounter = rh; // cache for release
                    }
                    return 1;
                }
            }
        }

```

rwLock.readLock().unlock();

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```