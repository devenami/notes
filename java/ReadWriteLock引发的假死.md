## 一个有趣的Java多线程"bug": ReadWriteLock引发的"假死"

### 摘要

在一个Java多线程服务里, 发现有reader和writer线程竞争同一个ReentrantReadWriteLock却双双被卡住. 乍一看, 大概率是因为该服务的多线程实现有bug而导致了死锁. 然而经过排查源码后却没有发现该服务的实现上的问题. 最后发现事故的真相是一场"假死". 而其罪魁祸首是JDK的ReentrantReadWriteLock默认实现里的一个"坑".

### 问题背景及描述

简化一下我们遇到问题: 假设有一个分布式文件系统, 其master节点提供文件系统的元数据(metadata)服务来管理该文件系统中所有的目录和文件. 该文件系统的元数据服务有一个线程池通过RPC来响应不同用户的请求, 并通过为每个目录/文件维护一个ReentrantReadWriteLock(使用默认方式创建)来协同线程保证consistency.

此时我们有三个不同的用户各自通过RPC对master节点发起请求:

1. 客户端A请求对目录"/foo/"下的所有文件以及子目录递归检查和校验. 这个请求会请求"/foo/"对应的读锁, 且当"/foo/"目录下文件过多时耗时会很长;
2. 客户端B请求删除文件"/foo/bar". 这个请求会竞争目录"/foo/"的写锁(因为"bar"是"/foo/" inode里的一个child);
3. 客户端C请求列出目录"/foo/"底下的文件. 该请求会竞争"/foo/"的读锁.

问题表现为客户端A运行的时候, B和C均发生了"卡死"而无法继续. 因为A仅仅是要求拿到"/foo/"的读锁, B由于竞争写锁需要等待A完成还可以理解, 但是C在这里也是要求读锁并不能继续, 令人非常疑惑.

### ReentrantReadWriteLock: 皮一下很开心

最后发现B和C对应的服务端读写线程均发生"卡死"其实是ReentrantReadWriteLock在默认的unfair policy下的一个合法行为. 这里我们先解释一下, 什么是ReentrantReadWriteLock的fair以及unfair policy:

- fair policy: 在这种模式下, ReentrantReadWriteLock会尽量按照锁请求的时间顺序来决定锁竞争结果. 这样的用意是在锁竞争激烈的时候, 保证写锁的竞争线程总有机会拿到锁而不是永远被读锁的竞争线程排斥.
- unfair policy: 在这种模式下, 由一系列的启发函数来决定锁竞争的结果, 而并不是依赖锁请求的顺序. 请注意, 这是ReentrantReadWriteLock默认的模式.

在我们的情况下, ReentrantReadWriteLock已经是使用unfair policy创建了, 但却依然发生请求读锁的客户端C卡死, 这又是为什么呢?

原来在unfair policy的启发函数中有一条规定, 当lock已经被读锁占用, 且锁竞争队列里排第一的是写锁请求的时候, 其他的读锁请求并不能以为是读锁就"插队"抢锁. 这个用意和fair policy一样, 为了防止写锁请求一直不能被轮巡到. 换言之, 所谓的unfair也并非完全效率第一, 而是一定程度上还是在兼顾fairness (参见 [ReadWriteLock: writeLock request blocks future readLock despite policy unfair](https://link.zhihu.com/?target=https%3A//bugs.openjdk.java.net/browse/JDK-6893626))

回到我们的场景: 客户端A成功的使用了读锁霸占住了"/foo/"对应的ReentrantReadWriteLock后并"超长待机"一直霸占, 请求写锁的客户端B排到了队列的头一名, 在unfair policy的作用下, 客户端C无法插队也一直被B的写锁请求抑制. 最后造成的假象就是, 没有线程可以继续工作, 形同"服务器卡死". 但实际上当A在工作了10分钟完成之后, B和C也分别顺利结束.

### 总结

通过对这个问题的排查, 确定了服务的多线程实现并没有问题, 出现的问题是对fairness了解不足. 如果要解决这个问题, 可以考虑以下几种方法:

1. 绕过"lock()"里的启发函数, 转而使用使用"tryLock()"和"while"循环来手动实现一个正真unfair的ReentrantReadWriteLock.
2. 通过业务逻辑层面的修改防止master的文件系统服务端为了响应一个请求而占用锁时间过长, 哪怕这是读锁.
3. 转用非JDK实现或者自己实现一个ReentrantReadWriteLock, 可以自定义各种启发函数来平衡线程间的公平和效率.

### 引用

[一个有趣的Java多线程"bug": ReadWriteLock引发的"假死"](https://zhuanlan.zhihu.com/p/34672421)