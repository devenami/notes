## CountDownLatch

### 解释

`CountDownLatch` 是 `java.util.concurrent`包中的一个工具类，用于线程计数。

它提供了一种同步的功能，允许一个或多个线程等待其他一系列在其他线程的操作完成后，继续执行当前线程。

并且，该对象是不可重用的，对象内存储的状态值一旦被减少为“0”，就无法再次重新设置。



### 入门demo

```java
    public static void main(String[] args) throws InterruptedException {
        int num = 3;	// 创建的线程数量
        CountDownLatch countDownLatch = new CountDownLatch(num);
        for (int i = 0; i < num; i++) {
            new Thread(new Worker(countDownLatch)).start();
        }
        countDownLatch.await(); // 等待所有线程执行完毕，再向下执行
        System.out.println("主线程执行完毕");
    }

    static class Worker implements Runnable {
        private CountDownLatch countDownLatch;
        public Worker(CountDownLatch countDownLatch) {
            this.countDownLatch = countDownLatch;
        }
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "正在执行...");
            countDownLatch.countDown(); // 通知count执行完毕
        }
    }
```

上面的程序定义了一个`num`变量用于指定`CountDownLatch`的`await()`方法等待的线程数量。

初始化一个`CountDownLatch`对象，并将该对象传入子线程中。子线程中通过调用`CountDownLatch`的`countDown()`方法来通知`CountDownLatch`当前线程执行完毕，由`CountDownLatch`对象进行计数。

主线程通过调用`CountDownLatch`的`await()`方法等待线程计数等于指定的线程数量。当两者相等时，唤醒主线程继续执行。

程序输出如下：

```
Thread-1正在执行...
Thread-2正在执行...
Thread-0正在执行...
主线程执行完毕
```



### 源码解析

#### 原理

`CountDownLatch`通过使用线程锁的方式，对线程进行计数。

#### 内部锁

```java
private static final class Sync extends AbstractQueuedSynchronizer {}
```

上述对象是`CountDownLatch`内部的锁实现类。

该类中通过设置父类的`state`状态值变量，并重写`tryAcquireShared`和`tryReleaseShared`对此状态进行修改。`tryAcquireShared`方法通过检查状态值是否为`0`尝试获取共享锁，`tryReleaseShared`释放锁的同时修改状态值，释放成功后对状态值进行`-1`操作。

#### 构造器

```java
public CountDownLatch(int count) {}
```

`CountDownLatch`仅提供了上面一个构造器，用于指定监听计数的线程数量。

#### await()

```java
public void await() throws InterruptedException {}
public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {}
```

该方法用于获取共享锁，内部通过调用`Sync`对象的`tryAcquireShared`获取锁，但是该方法会检测锁的状态值，只有状态值为`0`，也就是所有的监听线程都执行完毕后，锁才能正常获取到，否则，调用该方法的线程进行阻塞。

#### countDown()

```java
public void countDown() {}
```

`countDown()`方法通过释放锁的操作来修改状态值，每调用一次，状态值就减少1，知道状态值减少为`0`时，通知调用`await()`方法的线程再次进行锁获取操作。

