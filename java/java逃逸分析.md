## 问题

**java中的对象都在堆中分配内存吗？**

## 什么是逃逸分析？

定义：逃逸分析（Escape Analysis）简单来讲就是，Java Hotspot 虚拟机可以分析新创建对象的使用范围，并决定是否在 Java 堆上分配内存的一项技术。

**逃逸分析的 JVM 参数如下：**

* 开启逃逸分析： -XX:+DoEscapeAnalysis
* 关闭逃逸分析： -XX:-DoEscapeAnalysis
* 显示分析结果： -XX:+PrintEscapeAnalysis

逃逸分析技术在 Java SE 6u23+ 开始支持，并默认设置为启用状态，可以不用额外加这个参数。

## 逃逸分析算法

Java Hotspot 编译器实现下面论文中描述的逃逸算法：

> [Choi99] Jong-Deok Choi. Manish Gupta. Mauricio Seffano.
>
> ​				Vugranam C. Sreedhar . Sam Midkiff .
>
> ​				"Escape Analysis for Java".  Procedings of ACM SIGPLAN 
>
> ​				OOPSLA Conference. November 1, 1999

根据 Jong-Deok Choi. Manish Gupta. Mauricio Seffano. Vugranam C. Sreedhar . Sam Midkiff .等大牛在论文《Escape Analysis for Java》中描述的算法进行逃逸分析的。

该算法引入了连通图，用连通图来构建对象和对象引用之间的可达性关系，并在此基础上，提出一种组合数据流分析流。

由于算法是上下文相关和流敏感的，并且模拟了对象任意层次的嵌套关系，所以分析精度较高，只是运行时间和内存消耗相对较大。

## 对象逃逸状态

### 1、全局逃逸（GlobalEscape）

即一个对象的作用范围逃出了当前方法或者当前线程，有一下几种场景：

* 对象是一个静态变量
* 对象是一个已经发生逃逸的对象
* 对象作为当前方法的返回值

### 2、参数逃逸（ArgEscape）

即一个对象被作为方法参数传递或者被参数引用，但在调用过程中不会发生全局逃逸，这个状态时通过被调方法的字节码确定的。

### 3、没有逃逸

即方法中的对象没有发生逃逸。

## 逃逸分析优化

针对以上三点，当一个对象没有逃逸时，可以得到以下几个虚拟机的优化。

### 1、锁消除

我们知道线程同步锁是非常牺牲性能的，当编译器确定当前对象只有在当前线程使用，那么就会移除该对象的同步锁。

例如，StringBuffer 和 Vactor 都是用 synchronized 修饰线程安全的，但大部分清苦下，他们都只能在当前线程中用到，这样编译器就会优化移除掉这些锁操作。

**锁消除的 JVM 参数如下：**

* 开启锁消除：-XX:+EliminateLocks
* 关闭锁消除：-XX:-EliminateLocks

锁消除在 JDK8 中默认开启，并且锁消除都要建立在逃逸分析的基础上。

### 2、标量替换

首先要明白标量和聚合量，基础类型和对象类型的引用可以理解为标量，他们不能被进一步分解。而能被进一步分解的量就是聚合量，比如：对象。

对象是聚合量，它又可以被进一步分解为标量，将其成员分解为分散的变量，这就叫做标量替换。

这样，如果一个对象没有发生逃逸，那根本就不会创建它，只会在栈或者寄存器上创建它用到的成员标量，节省了内存空间，也提升了应用程序性能。

**标量替换的 JVM 参数如下：**

* 开启标量替换：-XX:+EliminateAllocations
* 关闭标量替换：-XX:-EliminateAllocations
* 显示标量替换详情：-XX:+PrintEliminateAllocations

标量替换在 JKD8 中都是默认开启的，并且都要建立在逃逸分析的基础上。

### 3、栈上分配

当对象没有发生逃逸时，该对象就可以通过标量替换分解为成员标量分配在栈内存中，和方法的声明周期一致，随着栈针出栈时销毁，减少了 GC 压力，提高了应用程序性能。

## 总结

逃逸分析是为了优化 JVM 内存和提升程序性能的。

在写代码时，尽量缩小变量的作用范围，如：

```java
return stringBuilder;
```

可以优化为：

```java
return stringBuilder.toString();
```

将 StringBuilder 对象的范围尽量控制在方法内部，防止对象逃逸。