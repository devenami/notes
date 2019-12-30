## Future

所谓异步调用其实就是实现一个可无需等待被调用函数的返回值而让操作继续运行的方法。在 Java 语言中，简单的讲就是另启一个线程来完成调用中的部分计算，使调用继续运行或返回，而不需要等待计算结果。但调用者仍需要取线程的计算结果。

JDK5新增了Future接口，用于描述一个异步计算的结果。

### Callable、Runnable

`Callable`和`Runnable`都是线程执行接口，两者的区别是：是否可以获取返回值。

`Runnable`接口的定义为：

```java
public interface Runnable {
    public abstract void run();
}
```

`Callable`接口的定义为：

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

可以看出Callable存在一个泛型V，并且call()方法返回的就是该泛型类型。

### Future

`Future`对当前执行的线程进行了封装，可以检测当前线程的状态，以及打断线程，获取线程结果等。

`Future`接口的定位为：

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

调用`get()`方法时，当前线程会被阻塞，知道该`Future`对象的线程执行完毕或被打断时才被唤醒继续执行。

### FutureTask

`FutureTask`是`Future`接口的实现类，同时`FutureTask`实现了`Future`和`Runnable`接口，也就说明`FutureTask`同时拥有了这两个接口的特性，即可以作为线程方法执行， 也可以获取返回结果。

`FutureTask`定义：

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
public class FutureTask<V> implements RunnableFuture<V>{}
```

`FutureTask`需要接受`Runnable`或`Callable`接口的实现类来操作任务。它提供了一下两种构造器：

```java
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
```

### 使用方式

`Runnable`、`Callable`必须和`ExecuteService`配合使用才能获取到返回结果，而`FutureTask`既可以和`ExecuteService`配合使用，也可以和`Thread`类配合使用，但有一点需要注意，**`FutureTask`对象是无法被重用的**。

### 案例

**MyCallable.java**

```java
public class MyCallable implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println(Thread.currentThread().getName() + "--执行");
        Thread.sleep(3000);
        return Thread.currentThread().getName() + "--get string text";
    }
}
```

**CallableMain.java**

```java
public class CallableMain {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 创建线程池
        ExecutorService executorService = Executors.newCachedThreadPool();

        // 创建callable对象
        MyCallable task = new MyCallable();

        System.out.println("使用callable + future");
        Future<String> future = executorService.submit(task);
        System.out.println(future.get());

        System.out.println();
        System.out.println("使用callable + futureTask + ThreadPool 执行开始");
        FutureTask futureTask = new FutureTask<>(task);
        executorService.submit(futureTask);
        System.out.println(futureTask.get());
        System.out.println("使用callable + futureTask + ThreadPool 执行完成");

        System.out.println();
        System.out.println("使用callable + futureTask + Thread");
        // 构建一个新的 FutureTask
        FutureTask futureTask1 = new FutureTask<>(task);
        Thread thread = new Thread(futureTask1);
        thread.setName("new Thread");
        thread.start();
        System.out.println(futureTask1.get());

        // 销毁线程池
        executorService.shutdown();
    }

}
```

**结果输出**

```
使用callable + future
pool-1-thread-1--执行
pool-1-thread-1--get string text

使用callable + futureTask + ThreadPool 执行开始
pool-1-thread-1--执行
pool-1-thread-1--get string text
使用callable + futureTask + ThreadPool 执行完成

使用callable + futureTask + Thread
new Thread--执行
new Thread--get string text
```



## CompletableFuture

虽然 Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的 CPU 资源，而且也不能及时地得到计算结果。

在Java8中，CompletableFuture提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合 CompletableFuture 的方法。

它可能代表一个明确完成的Future，也有可能代表一个完成阶段（ CompletionStage ），它支持在计算完成以后触发一些函数或执行某些动作。

### Future 接口的局限性

Future接口可以构建异步应用，但依然有其局限性。它很难直接表述多个Future 结果之间的依赖性。实际开发中，我们经常需要达成以下目的：

1. 将多个异步计算的结果合并成一个
2. 等待Future集合中的所有任务都完成
3. Future完成事件（即，任务完成以后触发执行动作）
4. 。。。

### CompletionStage

- CompletionStage代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段
- 一个阶段的计算执行可以是一个Function，Consumer或者Runnable。比如：stage.thenApply(x -> square(x)).thenAccept(x -> System.out.print(x)).thenRun(() -> System.out.println())
- 一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发

### 函数

`Async`后缀存在的区别在于，如果调用`Async`结尾的方法，则会将当前函数放入线程池内执行，否则使用当前任务的线程继续执行。

函数共分为如下几类：

1. `CompletableFuture`对象创建函数
2. 数据处理函数，无返回，仅对传入的数据进行处理(值拷贝，非指针传递，修改该值并不会影响下面的操作)
3. 数据处理函数，有返回，且返回类型不限制
4. 多个`CompletionStage`交叉函数
5. `Future`固定的方法

| 函数名                                        | 解释                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| static CompletableFuture\<Void> runAsync      | 无返回创建CompletableFuture对象，可接收线程池                |
| static \<U> CompletableFuture\<U> supplyAsync | 有返回创建CompletableFuture对象，可接收线程池                |
| whenComplete                                  | 当CompletableFuture的计算结果完成，执行特定的Action。        |
| exceptionally                                 | 当CompletableFuture抛出异常的时，执行特定的Action。          |
| thenApply                                     | 当一个线程依赖另一个线程时，可以使用 thenApply 方法来把这两个线程串行化。 |
| handle                                        | handle 是执行任务完成时对结果的处理。handle 方法和 thenApply 方法处理方式基本一样。不同的是 handle 是在任务完成后再执行，还可以处理异常的任务。thenApply 只可以执行正常的任务，任务出现异常则不执行 thenApply 方法。 |
| thenAccept                                    | 接收任务的处理结果，并消费处理，无返回结果。                 |
| thenRun                                       | 跟 thenAccept 方法不一样的是，不关心任务的处理结果。只要上面的任务执行完成，就开始执行 thenRun。 |
| thenCombine                                   | thenCombine 会把 两个 CompletionStage 的任务都执行完成后，把两个任务的结果一块交给 thenCombine 来处理。 |
| thenAcceptBoth                                | 当两个CompletionStage都执行完成后，把结果一块交给thenAcceptBoth来进行消耗 |
| applyToEither                                 | 两个CompletionStage，使用最先执行完成的CompletionStage的结果进行下一步的转化操作。 |
| acceptEither                                  | 两个CompletionStage，使用最先执行完成的CompletionStage的结果进行下一步的消费操作。 |
| runAfterEither                                | 两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable） |
| runAfterBoth                                  | 两个CompletionStage，都完成了计算才会执行下一步的操作（Runnable） |
| thenCompose                                   | thenCompose 方法允许你对两个 CompletionStage 进行流水线操作，第一个操作完成时，将其结果作为参数传递给第二个操作。 |

### 案例

```java
public class CompletableFutureMain {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future1 = CompletableFuture
                // 创建对象
                .supplyAsync(() -> new Random().nextInt(10))
                // 接受上一步的参数并处理
                .thenApply(integer -> integer + ":2")
                // 接受上一步的参数并处理，同时可以处理异常
                .handle((str, ex) -> Integer.parseInt(str.split(":")[1]));

        CompletableFuture<Integer> future2 = CompletableFuture
                .supplyAsync(() -> new Random().nextInt(20))
                // 上一步处理完成对参数进行处理
                .whenComplete((integer, throwable) -> {
                    // 这里的处理并不会影响下面的处理
                    if (throwable == null) {
                        integer = 0;
                    }
                })
                // 上一步执行完成执行下一步
                .thenApply(integer -> integer += 50)
                // 合并两个future
                .thenCombine(future1, (i1, i2) -> {
                    System.out.println(i1);
                    System.out.println(i2);
                    return i1 + i2;
                });

        System.out.println("程序最终结果：" + future2.get());

    }

}
```

