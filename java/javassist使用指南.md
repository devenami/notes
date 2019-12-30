## 1. 读写字节码

我们知道 Java 字节码以二进制的形式存储在 class 文件中，每一个 class 文件包含一个 Java 类或接口。Javaassist 就是一个用来处理 Java 字节码的类库。

在 Javassist 中，类 `Javaassit.CtClass` 表示 class 文件。一个 GtClass (编译时类）对象可以处理一个 class 文件，下面是一个简单的例子：

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("test.Rectangle");
cc.setSuperclass(pool.get("test.Point"));
cc.writeFile();
```

这段代码首先获取一个 ClassPool 对象。ClassPool 是 CtClass 对象的容器。它按需读取类文件来构造 CtClass 对象，并且保存 CtClass 对象以便以后使用。

为了修改类的定义，首先需要使用 ClassPool.get() 方法来从 ClassPool 中获得一个 CtClass 对象。上面的代码中，我们从 ClassPool 中获得了代表 test.Rectangle 类的 CtClass 对象的引用，并将其赋值给变量 cc。使用 getDefault() 方法获取的 ClassPool 对象使用的是默认系统的类搜索路径。

从实现的角度来看，ClassPool 是一个存储 CtClass 的 Hash 表，类的名称作为 Hash 表的 key。ClassPool 的 get() 函数用于从 Hash 表中查找 key 对应的 CtClass 对象。如果没有找到，get() 函数会创建并返回一个新的 CtClass 对象，这个新对象会保存在 Hash 表中。

从 ClassPool 中获取的 CtClass 是可以被修改的（稍后会讨论细节）。

在上面的例子中，test.Rectangle 的父类被设置为 test.Point。调用 writeFile() 后，这项修改会被写入原始类文件。writeFile() 会将 CtClass 对象转换成类文件并写到本地磁盘。也可以使用 toBytecode() 函数来获取修改过的字节码：

```java
byte[] b = cc.toBytecode();
```

你也可以通过 toClass() 函数直接将 CtClass 转换成 Class 对象:

```java
Class clazz = cc.toClass();
```

toClass() 请求当前线程的 ClassLoader 加载 CtClass 所代表的类文件。它返回此类文件的 java.lang.Class 对象，更多细节，请参考[下面的章节](https://github.com/jboss-javassist/javassist/wiki/Tutorial-1#toclass)。

定义新类

使用 ClassPool 的 makeClass() 方法可以定义一个新类。

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.makeClass("Point");
```

这段代码定义了一个空的 Point 类。Point 类的成员方法可以通过 CtNewMethod 类的工厂方法来创建，然后使用 CtClass 的 addMethod() 方法将其添加到 Point 中。

使用 ClassPool 中的 makeInterface() 方法可以创建新接口。接口中的方法可以使用 CtNewMethod 的 abstractMethod() 方法创建。

将类冻结

如果一个 CtClass 对象通过 writeFile(), toClass(), toBytecode() 被转换成一个类文件，此 CtClass 对象会被冻结起来，不允许再修改。因为一个类只能被 JVM 加载一次。

但是，一个冷冻的 CtClass 也可以被解冻，例如：

```java
CtClasss cc = ...;
    :
cc.writeFile();
cc.defrost();
cc.setSuperclass(...);    // 因为类已经被解冻，所以这里可以调用成功
```

调用 defrost() 之后，此 CtClass 对象又可以被修改了。

如果 ClassPool.doPruning 被设置为 true，Javassist 在冻结 CtClass 时，会修剪 CtClass 的数据结构。为了减少内存的消耗，修剪操作会丢弃 CtClass 对象中不必要的属性。例如，Code_attribute 结构会被丢弃。一个 CtClass 对象被修改之后，方法的字节码是不可访问的，但是方法名称、方法签名、注解信息可以被访问。修剪过的 CtClass 对象不能再次被解冻。ClassPool.doPruning 的默认值为 false。

stopPruning() 可以用来驳回修剪操作。

```java
CtClasss cc = ...;
cc.stopPruning(true);
    :
cc.writeFile(); // 转换成一个 class 文件
// cc is not pruned.
```

这个 CtClass 没有被修剪，所以在 writeFile() 之后，可以被解冻。

**注意**：调试的时候，你可能临时需要停止修剪和冻结，然后保存一个修改过的类文件到磁盘，debugWriteFile() 方法正是为此准备的。它停止修剪，然后写类文件，然后解冻并再次打开修剪（如果开始时修养是打开的）。

类搜索路径

通过 ClassPool.getDefault() 获取的 ClassPool 使用 JVM 的类搜索路径。如果程序运行在 JBoss 或者 Tomcat 等 Web 服务器上，ClassPool 可能无法找到用户的类，因为 Web 服务器使用多个类加载器作为系统类加载器。在这种情况下，ClassPool 必须添加额外的类搜索路径。

下面的例子中，pool 代表一个 ClassPool 对象：

```java
pool.insertClassPath(new ClassClassPath(this.getClass()));
```

上面的语句将 this 指向的类添加到 pool 的类加载路径中。你可以使用任意 Class 对象来代替 this.getClass()，从而将 Class 对象添加到类加载路径中。

也可以注册一个目录作为类搜索路径。下面的例子将 /usr/local/javalib 添加到类搜索路径中：

```java
ClassPool pool = ClassPool.getDefault();
pool.insertClassPath("/usr/local/javalib");
```

类搜索路径不但可以是目录，还可以是 URL ：

```java
ClassPool pool = ClassPool.getDefault();
ClassPath cp = new URLClassPath("www.javassist.org", 80, "/java/", "org.javassist.");
pool.insertClassPath(cp);
```

上述代码将 [http://www.javassist.org:80/java/](http://www.javassist.org/java/) 添加到类搜索路径。并且这个URL只能搜索 `org.javassist` 包里面的类。例如，为了加载 `org.javassist.test.Main`，它的类文件会从获取 [http://www.javassist.org:80/java/org/javassist/test/Main.class](http://www.javassist.org/java/org/javassist/test/Main.class) 获取。

此外，也可以直接传递一个 byte 数组给 ClassPool 来构造一个 CtClass 对象，完成这项操作，需要使用 ByteArrayPath 类。示例：

```java
ClassPool cp = ClassPool.getDefault();
byte[] b = a byte array;
String name = class name;
cp.insertClassPath(new ByteArrayClassPath(name, b));
CtClass cc = cp.get(name);
```

示例中的 CtClass 对象表示 b 代表的 class 文件。将对应的类名传递给 ClassPool 的 get() 方法，就可以从 ByteArrayClassPath 中读取到对应的类文件。

如果你不知道类的全名，可以使用 makeClass() 方法：

```java
ClassPool cp = ClassPool.getDefault();
InputStream ins = an input stream for reading a class file;
CtClass cc = cp.makeClass(ins);
```

makeClass() 返回从给定输入流构造的 CtClass 对象。 你可以使用 makeClass() 将类文件提供给 ClassPool 对象。如果搜索路径包含大的 jar 文件，这可能会提高性能。由于 ClassPool 对象按需读取类文件，它可能会重复搜索整个 jar 文件中的每个类文件。 makeClass() 可以用于优化此搜索。由 makeClass() 构造的 CtClass 保存在 ClassPool 对象中，从而使得类文件不会再被读取。

用户可以通过实现 ClassPath 接口来扩展类加载路径，然后调用 ClassPool 的 insertClassPath() 方法将路径添加进来。这种技术主要用于将非标准资源添加到类搜索路径中。

## 2. ClassPool

ClassPool 是 CtClass 对象的容器。因为编译器在编译引用 CtClass 代表的 Java 类的源代码时，可能会引用 CtClass 对象，所以一旦一个 CtClass 被创建，它就被保存在 ClassPool 中.

例如，一个 CtClass 类代表 Point 类，并给 CtClass 添加 getter() 方法。然后，程序尝试编译一段代码，代码中包含了 Point 的 getter() 调用，然后将这段代码添加了另一个类 Line 中，如果代表 Point 的 CtClass 丢失，编译器就无法编译 Line 中的 Point.getter() 方法。注：原来的 Point 类中无 getter() 方法。因此，为了能够正确编译这个方法调用，ClassPool 必须在程序执行期间包含所有的 CtClass 实例。

避免内存溢出

如果 CtClass 对象的数量变得非常大（这种情况很少发生，因为 Javassist 试图以各种方式减少内存消耗），ClassPool 可能会导致巨大的内存消耗。 为了避免此问题，可以从 ClassPool 中显式删除不必要的 CtClass 对象。 如果对 CtClass 对象调用 detach()，那么该 CtClass 对象将被从 ClassPool 中删除。 例如：

```java
CtClass cc = ... ;
cc.writeFile();
cc.detach();
```

在调用 detach() 之后，就不能调用这个 CtClass 对象的任何方法了。但是如果你调用 ClassPool 的 get() 方法，ClassPool 会再次读取这个类文件，创建一个新的 CtClass 对象。

另一个办法是用新的 ClassPool 替换旧的 ClassPool，并将旧的 ClassPool 丢弃。 如果旧的 ClassPool 被垃圾回收掉，那么包含在 ClassPool 中的 CtClass 对象也会被回收。要创建一个新的 ClassPool，参见以下代码：

```java
ClassPool cp = new ClassPool(true);
// if needed, append an extra search path by appendClassPath()
```

这段代码创建了一个 ClassPool 对象，它的行为与 ClassPool.getDefault() 类似。 请注意，ClassPool.getDefault() 是为了方便而提供的单例工厂方法，它保留了一个`ClassPool`的单例并重用它。getDefault() 返回的 ClassPool 对象并没有特殊之处。

**注意**：new ClassPool(true) 构造一个 ClassPool 对象，并附加了系统搜索路径。
调用此构造函数等效于以下代码：

```java
ClassPool cp = new ClassPool();
cp.appendSystemPath();  // or append another path by appendClassPath()
```

级联的 ClassPools

如果程序正在 Web 应用程序服务器上运行，则可能需要创建多个 ClassPool 实例; 应为每一个 ClassLoader 创建一个 ClassPool 的实例。 程序应该通过 ClassPool 的构造函数，而不是调用 getDefault() 来创建一个 ClassPool 对象。
多个 ClassPool 对象可以像 java.lang.ClassLoader 一样级联。 例如，

```java
ClassPool parent = ClassPool.getDefault();
ClassPool child = new ClassPool(parent);
child.insertClassPath("./classes");
```

如果调用 child.get()，子 ClassPool 首先委托给父 ClassPool。如果父 ClassPool 找不到类文件，那么子 ClassPool 会尝试在 ./classes 目录下查找类文件。

如果 child.childFirstLookup 返回 true，那么子类 ClassPool 会在委托给父 ClassPool 之前尝试查找类文件。 例如：

```java
ClassPool parent = ClassPool.getDefault();
ClassPool child = new ClassPool(parent);
child.appendSystemPath();         // the same class path as the default one.
child.childFirstLookup = true;    // changes the behavior of the child.
```

拷贝一个已经存在的类来定义一个新的类

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
cc.setName("Pair");
```

这个程序首先获得类 Point 的 CtClass 对象。然后它调用 setName() 将这个 CtClass 对象的名称设置为 Pair。在这个调用之后，这个 CtClass 对象所代表的类的名称 Point 被修改为 Pair。类定义的其他部分不会改变。

**注意**：CtClass 中的 setName() 改变了 ClassPool 中的记录。从实现的角度来看，一个 ClassPool 对象是一个 CtClass 对象的哈希表。setName() 更改了与哈希表中的 CtClass 对象相关联的 Key。Key 从原始类名更改为新类名。

因此，如果后续在 ClassPool 对象上再次调用 get("Point")，则它不会返回变量 cc 所指的 CtClass 对象。 而是再次读取类文件 Point.class，并为类 Point 构造一个新的 CtClass 对象。 因为与 Point 相关联的 CtClass 对象不再存在。示例：

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
CtClass cc1 = pool.get("Point");   // cc1 is identical to cc.
cc.setName("Pair");
CtClass cc2 = pool.get("Pair");    // cc2 is identical to cc.
CtClass cc3 = pool.get("Point");   // cc3 is not identical to cc.
```

cc1 和 cc2 指向 CtClass 的同一个实例，而 cc3 不是。 注意，在执行 cc.setName("Pair") 之后，cc 和 cc1 引用的 CtClass 对象都表示 Pair 类。

ClassPool 对象用于维护类和 CtClass 对象之间的一对一映射关系。 为了保证程序的一致性，Javassist 不允许用两个不同的 CtClass 对象来表示同一个类，除非创建了两个独立的 ClassPool。

如果你有两个 ClassPool 对象，那么你可以从每个 ClassPool 中，获取一个表示相同类文件的不同的 CtClass 对象。 你可以修改这些 CtClass 对象来生成不同版本的类。

通过重命名冻结的类来生成新的类

一旦一个 CtClass 对象被 writeFile() 或 toBytecode() 转换为一个类文件，Javassist 会拒绝对该 CtClass 对象的进一步修改。因此，在表示 Point 类的 CtClass 对象被转换为类文件之后，你不能将 Pair 类定义为 Point 的副本，因为在 Point 上执行 setName() 会被拒绝。 以下代码段是错误的：

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
cc.writeFile();
cc.setName("Pair");    // wrong since writeFile() has been called.
```

为了避免这种限制，你应该在 ClassPool 中调用 getAndRename() 方法。 例如：

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
cc.writeFile();
CtClass cc2 = pool.getAndRename("Point", "Pair");
```

如果调用 getAndRename()，ClassPool 首先读取 Point.class 来创建一个新的表示 Point 类的 CtClass 对象。 而且，它会在这个 CtClass 被记录到哈希表之前，将 CtClass 对象重命名为 Pair。因此，getAndRename() 可以在表示 Point 类的 CtClass 对象上调用 writeFile() 或 toBytecode() 后执行。

## 3. 类加载器 (Class Loader)

如果事先知道要修改哪些类，修改类的最简单方法如下：

1. 调用 ClassPool.get() 获取 CtClass 对象，
2. 修改 CtClass
3. 调用 CtClass 对象的 writeFile() 或者 toBytecode() 获得修改过的类文件。

如果在加载时，可以确定是否要修改某个类，用户必须使 Javassist 与类加载器协作，以便在加载时修改字节码。用户可以定义自己的类加载器，也可以使用 Javassist 提供的类加载器。

### 3.1 CtClass.toClass()

CtClass 的 toClass() 方法请求当前线程的上下文类加载器，加载 CtClass 对象所表示的类。要调用此方法，调用者必须具有相关的权限; 否则，可能会抛出 SecurityException。示例：

```java
public class Hello {
    public void say() {
        System.out.println("Hello");
    }
}

public class Test {
    public static void main(String[] args) throws Exception {
        ClassPool cp = ClassPool.getDefault();
        CtClass cc = cp.get("Hello");
        CtMethod m = cc.getDeclaredMethod("say");
        m.insertBefore("{ System.out.println(\"Hello.say():\"); }");
        Class c = cc.toClass();
        Hello h = (Hello)c.newInstance();
        h.say();
    }
}
```

Test.main() 在 Hello 中的 say() 方法体中插入一个 println()。然后它构造一个修改过的 Hello 类的实例，并在该实例上调用 say() 。
**注意**：上面的程序要正常运行，Hello 类在调用 toClass() 之前不能被加载。 如果 JVM 在 toClass() 调用之前加载了原始的 Hello 类，后续加载修改的 Hello 类将会失败（LinkageError 抛出）。
例如，如果 Test 中的 main() 是这样的：

```java
public static void main(String[] args) throws Exception {
    Hello orig = new Hello();
    ClassPool cp = ClassPool.getDefault();
    CtClass cc = cp.get("Hello");
        :
}
```

那么，原始的 Hello 类在 main 的第一行被加载，toClass() 调用会抛出一个异常，因为类加载器不能同时加载两个不同版本的 Hello 类。

如果程序在某些应用程序服务器（如JBoss和Tomcat）上运行，toClass() 使用的上下文类加载器可能是不合适的。在这种情况下，你会看到一个意想不到的 ClassCastException。为了避免这个异常，必须给 toClass() 指定一个合适的类加载器。 例如，如果 'bean' 是你的会话 bean 对象，那么下面的代码：

```java
CtClass cc = ...;
Class c = cc.toClass(bean.getClass().getClassLoader());
```

可以工作。你应该给 toClass() 传递加载了你的程序的类加载器（上例中，`bean`对象的类）。

toClass() 是为了简便而提供的方法。如果你需要更复杂的功能，你应该编写自己的类加载器。

### 3.2 Java的类加载机制

在Java中，多个类加载器可以共存，每个类加载器创建自己的名称空间。不同的类加载器可以加载具有相同类名的不同类文件。加载的两个类被视为不同的类。此功能使我们能够在单个 JVM 上运行多个应用程序，即使这些程序包含具有相同名称的不同的类。

**注意**：JVM 不允许动态重新加载类。一旦类加载器加载了一个类，它不能在运行时重新加载该类的修改版本。因此，在JVM 加载类之后，你不能更改类的定义。但是，JPDA（Java平台调试器架构）提供有限的重新加载类的能力。参见[3.6节](https://blog.csdn.net/21aspnet/article/details/81671777#hotswap)。

如果相同的类文件由两个不同的类加载器加载，则 JVM 会创建两个具有相同名称和定义的不同的类。由于两个类不相同，一个类的实例不能被分配给另一个类的变量。两个类之间的转换操作将失败并抛出一个 ClassCastException。
例如，下面的代码会抛出异常：

```java
MyClassLoader myLoader = new MyClassLoader();
Class clazz = myLoader.loadClass("Box");
Object obj = clazz.newInstance();
Box b = (Box)obj;    // this always throws ClassCastException.
```

Box 类由两个类加载器加载。假设类加载器 CL 加载包含此代码片段的类。因为这段代码引用了 MyClassLoader，Class，Object 和 Box，CL 也加载这些类（除非它委托给另一个类加载器）。 因此，变量 b 的类型是 CL 加载的 Box 类。 另一方面， myLoader 也加载了 Box class。 对象 obj 是由 myLoader 加载的 Box 类的一个实例。 因此，最后一个语句总是抛出 ClassCastException ，因为 obj 的类是一个不同的 Box 类的类型，而不是用作变量 b 的类型。

多个类加载器形成一个树型结构。 除引导类加载器之外的每个类加载器，都有一个父类加载器，它通常加载该子类加载器的类。 因为加载类的请求可以沿类加载器的这个层次委派，所以即使你没有请求加载一个类，它也可能被加载。因此，已经请求加载类 C 的类加载器可以不同于实际加载类 C 的加载器。为了区分，我们将前加载器称为 C 的发起者，将后加载器称为 C 的实际加载器 。

此外，如果请求加载类 C（C的发起者）的类加载器 CL 委托给父类加载器 PL，则类加载器 CL 不会加载类 C 引用的任何类。因为 CL 不是那些类的发起者。 相反，父类加载器 PL 成为它们的启动器，并且加载它们。

请参考下面的例子来理解：

```java
public class Point {    // loaded by PL
    private int x, y;
    public int getX() { return x; }
        :
}

public class Box {      // the initiator is L but the real loader is PL
    private Point upperLeft, size;
    public int getBaseX() { return upperLeft.x; }
        :
}

public class Window {    // loaded by a class loader L
    private Box box;
    public int getBaseX() { return box.getBaseX(); }
}
```

假设一个类 Window 由类加载器 L 加载。Window 的启动器和实际加载器都是 L。由于 Window 的定义引用了 Box，JVM 将请求 L 加载 Box。 这里，假设 L 将该任务委托给父类加载器 PL。Box 的启动器是 L，但真正的加载器是 PL。 在这种情况下，Point 的启动器不是 L 而是 PL，因为它与 Box 的实际加载器相同。 因此，Point 不会被 L 加载。

接下来，看一个稍微修改过的例子：

```java
public class Point {
    private int x, y;
    public int getX() { return x; }
        :
}

public class Box {      // the initiator is L but the real loader is PL
    private Point upperLeft, size;
    public Point getSize() { return size; }
        :
}

public class Window {    // loaded by a class loader L
    private Box box;
    public boolean widthIs(int w) {
        Point p = box.getSize();
        return w == p.getX();
    }
}
```

现在，Window 的定义也引用了 Point。 在这种情况下，如果请求加载 Point，类加载器 L 也必须委托给 PL。 **你必须避免有两个类加载器两次加载同一个类**。两个加载器之一必须委托给另一个。

当 Point 加载时，如果 L 不委托给 PL，widthIs() 就会抛出一个 ClassCastException 异常。因为 Box 的实际加载器是 PL，在 Box 中引用的 Point 也由 PL 加载。 getSize() 的结果值是由 PL 加载的 Point，widthIs() 中的变量 p 是由 L 加载的 Point。JVM 认为它们是不同的类型，因此它会抛出类型不匹配的异常。

这种设计有点不方便，但也是必须的。

```java
Point p = box.getSize();
```

如果上面的语句没有抛出异常，那么 Window 的程序员可以破坏 Point 对象的封装。 例如，字段 x 在 PL 中加载的 Point 中是私有的。 然而，如果 L 加载具有以下定义的 Point，则 Window 类可以直接访问 x 的值：

```java
public class Point {
    public int x, y;    // not private
    public int getX() { return x; }
        :
}
```

有关 Java 类加载器的更多详细信息，可以参看以下文章：

> Sheng Liang 和 Gilad Bracha，“Dynamic Class Loading in the Java Virtual Machine”，* ACM OOPSLA'98 *，pp.36-44,1998。

### 3.3 使用 javassist.Loader

Javassit 提供一个类加载器 javassist.Loader。它使用 javassist.ClassPool 对象来读取类文件。
例如，javassist.Loader 可以用于加载用 Javassist 修改过的类。

```java
import javassist.*;
import test.Rectangle;

public class Main {
  public static void main(String[] args) throws Throwable {
     ClassPool pool = ClassPool.getDefault();
     Loader cl = new Loader(pool);
     CtClass ct = pool.get("test.Rectangle");
     ct.setSuperclass(pool.get("test.Point"));
     Class c = cl.loadClass("test.Rectangle");
     Object rect = c.newInstance();
         :
  }
}
```

这个程序将 test.Rectangle 的超类设置为 test.Point。然后再加载修改的类，并创建新的 test.Rectangle 类的实例。

如果用户希望在加载时按需修改类，则可以向 javassist.Loader 添加事件监听器。当类加载器加载类时会通知监听器。事件监听器类必须实现以下接口：

```java
public interface Translator {
    public void start(ClassPool pool)
        throws NotFoundException, CannotCompileException;
    public void onLoad(ClassPool pool, String classname)
        throws NotFoundException, CannotCompileException;
}
```

当事件监听器通过 addTranslator() 添加到 javassist.Loader 对象时，start() 方法会被调用。在 javassist.Loader 加载类之前，会调用 onLoad() 方法。可以在 onLoad() 方法中修改被加载的类的定义。

例如，下面的事件监听器在类加载之前，将所有类更改为 public 类。

```java
public class MyTranslator implements Translator {
    void start(ClassPool pool) throws NotFoundException, CannotCompileException {}
    void onLoad(ClassPool pool, String classname) throws NotFoundException, CannotCompileException {
        CtClass cc = pool.get(classname);
        cc.setModifiers(Modifier.PUBLIC);
    }
}
```

注意，onLoad() 不必调用 toBytecode() 或 writeFile()，因为 javassist.Loader 会调用这些方法来获取类文件。

要使用 MyTranslator 对象运行一个应用程序类 MyApp，主类代码如下：

```java
import javassist.*;

public class Main2 {
  public static void main(String[] args) throws Throwable {
     Translator t = new MyTranslator();
     ClassPool pool = ClassPool.getDefault();
     Loader cl = new Loader();
     cl.addTranslator(pool, t);
     cl.run("MyApp", args);
  }
}
```

执行下面的命令来运行程序：

```shell
% java Main2 arg1 arg2...
```

类 MyApp 和其他应用程序类会被 MyTranslator 监听。

注意，MyApp 不能访问 loader 类，如 Main2，MyTranslator 和 ClassPool，因为它们是由不同的加载器加载的。 应用程序类由 javassist.Loader 加载，而加载器类（例如 Main2）由默认的 Java 类加载器加载。

javassist.Loader 以不同的顺序从 java.lang.ClassLoader 中搜索类。ClassLoader 首先将加载操作委托给父类加载器，只有当父类加载器无法找到它们时才尝试自己加载类。另一方面，javassist.Loader 尝试在委托给父类加载器之前加载类。它仅在以下情况下进行委派：

1. 在 ClassPool 对象上调用 get() 找不到这个类；
2. 这些类已经通过 delegateLoadingOf() 来指定由父类加载器加载。

此搜索顺序允许 Javassist 加载修改过的类。但是，如果找不到修改的类，它将委托父类加载器来加载。一旦一个类被父类加载器加载，那个类中引用的其他类也将被父类加载器加载，因此它们是没有被修改的。 回想一下，C 类引用的所有类都由 C 的实际加载器加载的。如果你的程序无法加载修改的类，你应该确保所有使用该类的类都是由 javassist 加载的。

### 3.4 自定义类加载器

下面看一个简单的带 Javassist 的类加载器：

```java
import javassist.*;

public class SampleLoader extends ClassLoader {
    /* Call MyApp.main(). */
    public static void main(String[] args) throws Throwable {
        SampleLoader s = new SampleLoader();
        Class c = s.loadClass("MyApp");
        c.getDeclaredMethod("main", new Class[] { String[].class })
         .invoke(null, new Object[] { args });
    }

    private ClassPool pool;

    public SampleLoader() throws NotFoundException {
        pool = new ClassPool();
        pool.insertClassPath("./class"); // MyApp.class must be there.
    }
    /* 
     * Finds a specified class.
     * The bytecode for that class can be modified.
     */
    protected Class findClass(String name) throws ClassNotFoundException {
        try {
            CtClass cc = pool.get(name);
            // *modify the CtClass object here*
            byte[] b = cc.toBytecode();
            return defineClass(name, b, 0, b.length);
        } catch (NotFoundException e) {
            throw new ClassNotFoundException();
        } catch (IOException e) {
            throw new ClassNotFoundException();
        } catch (CannotCompileException e) {
            throw new ClassNotFoundException();
        }
    }
}
```

MyApp 类是一个应用程序。 要执行此程序，首先将类文件放在 ./class 目录下，它不能包含在类搜索路径中。 否则，MyApp.class 将由默认系统类加载器加载，它是 SampleLoader 的父加载器。目录名 ./class 由构造函数中的 insertClassPath() 指定。然后运行：

```shell
% java SampleLoader
```

类加载器会加载类 MyApp (./class/MyApp.class)，并使用命令行参数调用 MyApp.main()。

这是使用 Javassist 的最简单的方法。 但是，如果你编写一个更复杂的类加载器，你可能需要更详细地了解 Java 的类加载机制。 例如，上面的程序将 MyApp 类放在与 SampleLoader 类不同的命名空间中，因为这两个类由不同的类装载器加载。 因此，MyApp 类不能直接访问类 SampleLoader。

### 3.5 修改系统的类

像 java.lang.String 这样的系统类只能被系统类加载器加载。因此，上面的 SampleLoader 或 javassist.Loader 在加载时不能修改系统类。系统类必须被静态地修改。下面的程序向 java.lang.String 添加一个新字段 hiddenValue：

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("java.lang.String");
CtField f = new CtField(CtClass.intType, "hiddenValue", cc);
f.setModifiers(Modifier.PUBLIC);
cc.addField(f);
cc.writeFile(".");
```

这段程序生成一个新文件 ./java/lang/String.class

可以使用 MyApp 这样测试修改过的 String 类：

```shell
% java -Xbootclasspath/p:. MyApp arg1 arg2...
```

MyApp 的定义如下：

```java
public class MyApp {
    public static void main(String[] args) throws Exception {
        System.out.println(String.class.getField("hiddenValue").getName());
    }
}
```

如果修改过的 String 类被加载，MyApp 会打印出 hiddenValue。

**注意**：如果应用使用此技术来覆盖 rt.jar 中的系统类，那么部署这个应用会违反 Java 2 运行时二进制代码许可协议。

### 3.6 在运行时重新加载类

如果 JVM 在启用 JPDA（Java平台调试器体系结构）的情况下启动，那么类可以被动态地重新加载。在 JVM 加载类之后，旧版本的类可以被卸载，新版本可以再次重新加载。也就是说，该类的定义可以在运行时动态被修改。然而，新的类定义必须与旧的类定义有些兼容。JVM 不允许两个版本之间的模式更改。它们必须具有相同的方法和字段。

Javassist 提供了一个方便的类，用于在运行时重新加载类。更多相关信息，请参阅javassist.tools.HotSwapper 的 API 文档。

## 4. 自省和自定制 (Introspection and customization)

CtClass 提供了自省的方法。Javassist 的自省能力与 Java 反射 API 兼容。 CtClass 提供了 getName()，getSuperclass()，getMethods() 等方法来获取类的信息，也提供了修改类定义的方法（添加字段，添加构造函数、添加方法），同时也可以对方法体的语句进行检测。

方法由 CtMethod 对象表示。CtMethod 提供了几个函数来修改方法的定义。
**注意**，如果一个方法继承自一个超类，那么表示继承方法的 CtMethod 对象，同样也表示该超类中声明的方法。

例如，如果类 Point 声明方法 move() ， Point 的子类 ColorPoint 不覆盖 move() ，那么在 Point 中声明的 move 和 ColorPoint 的 move 具有相同的 CtMethod。 如果由这个 CtMethod 对象表示的方法定义被修改，那么修改将反映在这两种方法上。 如果你只想修改 ColorPoint 中的 move() 方法，你首先必须在 Point 中加入 CtMethod 对象的副本 move() 。CtMethod 对象的副本可以通过 CtNewMethod.copy() 获得。

Javassist 不允许删除方法或字段，但它允许更改名称。所以，如果一个方法是没有必要的，可以通过调用 CtMethod 的 setName() 和 setModifiers() 中将其改为一个私有方法。

Javassist 不允许向现有方法添加额外的参数。你可以通过新建一个方法达到同样的效果。 例如，如果你想为一个方法添加一个额外的 int 参数 newZ 到 Point 类，

```java
void move(int newX, int newY) { x = newX; y = newY; }
```

你可以添加一个这样的方法到 Point 类：

```java
void move(int newX, int newY, int newZ) {
    // do what you want with newZ.
    move(newX, newY);
}
```

Javassist 还提供了用于直接编辑原始类文件的低级API。
例如，CtClass 中的 getClassFile() 返回一个表示类文件的 ClassFile 对象。CtMethod 的 getMethodInfo() 方法返回一个 MethodInfo 对象，表示类文件中的 method_info 结构。 低级API使用 Java 虚拟机规范中的词汇表。 用户必须具有类文件和字节码的知识。有关更多详细信息，请参考 [javassist.bytecode 包](https://link.jianshu.com/?t=tutorial3.html#intro)。

由 Javassist 修改的类文件只有在使用以 $ 开头的特殊标识符时才需要 javassist.runtime 包来提供运行时支持。接下来的内容会讨论这些特殊标识符。在没有这些特殊标识符的情况下，在运行时修改类文件不需要 javassist.runtime 包或任何其他 Javassist 包。有关更多详细信息，请参阅 javassist.runtime 包的API文档。

### 4.1 在方法体的开始/结尾处添加代码

CtMethod 和 CtConstructor 提供了 insertBefore()，insertAfter() 和 addCatch() 方法。 它们可以将用 Java 编写的代码片段插入到现有方法中。Javassist 包括一个用于处理源代码的简单编译器，它接收用 Java 编写的源代码，并将其编译成 Java 字节码，并内联方法体中。

也可以按行号来插入代码段（如果行号表包含在类文件中）。向 CtMethod 和 CtConstructor 中的 insertAt() 方法提供源代码和原始类定义中的源文件的行号，就可以将编译后的代码插入到指定行号位置。

方法 insertBefore() ，insertAfter()，addCatch() 和 insertAt() 接收一个表示语句或语句块的 String 对象。一个语句是一个单一的控制结构，比如 if 和 while 或者以分号结尾的表达式。语句块是一组用大括号 {} 包围的语句。因此，以下每行都是有效语句或块的示例：

```java
System.out.println("Hello");
{ System.out.println("Hello"); }
if (i < 0) { i = -i; }
```

语句和语句块可以引用字段和方法。如果使用 -g 选项（在类文件中包含局部变量属性）编译该方法，则它们还可以引用方法的参数。 否则，它们必须通过特殊变量 $0，$1，$2，... 来访问方法参数，下面会讨论。不允许访问在方法中声明的局部变量，尽管在块中声明一个新的局部变量是允许的。但是，insertAt() 允许语句和块访问局部变量，前提是这些变量在指定的行号处可用，并且目标方法是使用 -g 选项编译的。

传递给方法 insertBefore() ，insertAfter() ，addCatch() 和 insertAt() 的 String 对象是由Javassist 的编译器编译的。 由于编译器支持语言扩展，以 $ 开头的几个标识符有特殊的含义：

| 符号                  | 含义                                            |
| :-------------------- | :---------------------------------------------- |
| `$0`, `$1`, `$2`, ... | `this` and 方法的参数                           |
| `$args`               | 方法参数数组.它的类型为 `Object[]`              |
| `$$`                  | 所有实参。例如, `m($$)` 等价于 `m($1,$2,`...`)` |
| `$cflow(`...`)`       | `cflow` 变量                                    |
| `$r`                  | 返回结果的类型，用于强制类型转换                |
| `$w`                  | 包装器类型，用于强制类型转换                    |
| `$_`                  | 返回值                                          |
| `$sig`                | 类型为 java.lang.Class 的参数类型数组           |
| `$type`               | 一个 java.lang.Class 对象，表示返回值类型       |
| `$class`              | 一个 java.lang.Class 对象，表示当前正在修改的类 |

$0, $1, $2, ...

传递给目标方法的参数使用 $1，$2，... 访问，而不是原始的参数名称。 $1 表示第一个参数，$2 表示第二个参数，以此类推。 这些变量的类型与参数类型相同。 $0 等价于 `this` 指针。 如果方法是静态的，则 $0 不可用。

下面有一些使用这些特殊变量的例子。假设一个类 Point:

```java
class Point {
    int x, y;
    void move(int dx, int dy) { x += dx; y += dy; }
}
```

要在调用方法 move() 时打印 dx 和 dy 的值，请执行以下程序：

```java
ClassPool pool = ClassPool.getDefault();
CtClass cc = pool.get("Point");
CtMethod m = cc.getDeclaredMethod("move");
m.insertBefore("{ System.out.println($1); System.out.println($2); }");
cc.writeFile();
```

请注意，传递给 insertBefore() 的源文本是用大括号 {} 括起来的。insertBefore() 只接受单个语句或用大括号括起来的语句块。

修改后的类 Point 的定义是这样的：

```java
class Point {
    int x, y;
    void move(int dx, int dy) {
        { System.out.println(dx); System.out.println(dy); }
        x += dx; y += dy;
    }
}
```

`$1` and `$2` are replaced with `dx` and `dy`, respectively.
`$1`, `$2`, `$3` ... are updatable. If a new value is assigend to one of those variables, then the value of the parameter represented by that variable is also updated.

$1 和 $2 分别替换为 dx 和 dy。
$1，$2，$3 ...是可更新的。如果这些变量被赋予新值，则由该变量表示的参数的值也将被更新。

$args

变量 $args 表示所有参数的数组。该变量的类型是 Object 类的数组。如果参数类型是原始类型（如 int），则该参数值将转换为包装器对象（如java.lang.Integer）以存储在 $args 中。 因此，如果第一个参数的类型是不是原始类型，那么 $args[0] 等于 $1。注意 $args[0] 不等于 $0，因为 $0 表示 `this`。

如果 Object 的数组被分配给 $args，那么该数组的每个元素都被分配给每个参数。如果参数类型是基本类型，则相应元素的类型必须是包装类型。 在将值分配给参数之前，必须将该值从包装器类型转换为基本类型。

$$

变量 $$ 是所有参数列表的缩写，用逗号分隔。 例如，如果方法 move() 的有 3 个参数，则

```
move($1, $2, $3)
```

如果 move() 不带任何参数，则 move（

）等同于move()。）等同于move()。

可以与其他方法一起使用。 如果你写一个表达式：

```
exMove($$, context)
```

这个表达式等价于

```
exMove($1, $2, $3, context)
```

注意，$$ 开启了通用符号方法调用，它通常与稍后要介绍的 $proceed 一起使用。

$cflow

$ cflow表示 控制流。此只读变量返回特定方法的递归调用的深度。

假设下面所示的方法由CtMethod对象cm表示：

```java
int fact(int n) {
    if (n <= 1)
        return n;
    else
        return n * fact(n - 1);
}
```

要使用 $cflow，首先声明使用 $cflow 监视方法 fact() 的调用：

```java
CtMethod cm = ...;
cm.useCflow("fact");
```

useCflow() 的参数是 $cflow 变量的标识符。任何有效的 Java 名称都可以用作标识符。标识符还可以包括 `.` ，例如，“my.Test.fact”是有效的标识符。

然后，$cflow(fact) 表示由 cm 指定的方法的递归调用深度。$cflow(fact) 的值在方法第一次调用时为 0，而当方法在方法中递归调用时为 1。 例如，

```java
cm.insertBefore("if ($cflow(fact) == 0)"
              + "    System.out.println(\"fact \" + $1);");
```

翻译方法fact()，以便它显示参数。因为检查了 $cflow(fact) 的值，所以如果在 fact() 中递归调用，则方法 fact() 不会显示参数。

$cflow 的值是当前线程的最顶层堆栈帧下与 cm 相关联的堆栈帧数。 $cflow 也可以不在 cm 方法中访问。

$r

$r 表示方法的结果类型（返回类型）。它用在 cast 表达式中作 cast 转换类型。 下面是一个典型的用法：

```java
Object result = ... ;
$_ = ($r)result;
```

如果结果类型是原始类型，则 ($r) 遵循特殊语义。 首先，如果 cast 表达式的操作数是原始类型，($r) 作为普通转换运算符。 另一方面，如果操作数是包装类型，($r) 将从包装类型转换为结果类型。 例如，如果结果类型是 int，那么 ($r) 将从 java.lang.Integer 转换为 int。

如果结果类型为void，那么 ($r) 不转换类型; 它什么也不做。 但是，如果操作数是对 void 方法的调用，则 ($r) 将导致 null。 例如，如果结果类型是 void，而 foo() 是一个 void 方法，那么

```
$_ = ($r)foo();
```

是一个正确的表达式。

cast 运算符 ($r) 在 return 语句中也很有用。 即使结果类型是 void，下面的 return 语句也是有效的：

```
return ($r)result;
```

这里，result是局部变量。 因为指定了 ($r)，所以结果值被丢弃。此返回语句被等价于:

```
return;
```

$w

$w 表示包装类型。它用在 cast 表达式中作 cast 转换类型。($w) 把基本类型转换为包装类型。 以下代码是一个示例：

```
Integer i = ($w)5;
```

包装后的类型取决于 ($w) 后面表达式的类型。如果表达式的类型为 double，则包装器类型为 java.lang.Double。

If the type of the expression following `($w)` is not a primitive type, then `($w)` does nothing.
如果下面的表达式 ($w) 的类型不是原始类型，那么($w) 什么也不做。

$_

CtMethod 中的 insertAfter() 和 CtConstructor 在方法的末尾插入编译的代码。传递给insertAfter() 的语句中，不但可以使用特殊符号如 $0，$1。也可以使用 $_ 来表示方法的结果值。
该变量的类型是方法的结果类型（返回类型）。如果结果类型为 void，那么 $_ 的类型为Object，$_ 的值为 null。
虽然由 insertAfter() 插入的编译代码通常在方法返回之前执行，但是当方法抛出异常时，它也可以执行。要在抛出异常时执行它，insertAfter() 的第二个参数 asFinally 必须为true。
如果抛出异常，由 insertAfter() 插入的编译代码将作为 finally 子句执行。$_ 的值 0 或 null。在编译代码的执行终止后，最初抛出的异常被重新抛出给调用者。注意，$_ 的值不会被抛给调用者，它将被丢弃。

$sig

$sig 的值是一个 java.lang.Class 对象的数组，表示声明的形式参数类型。

$type

$type 的值是一个 java.lang.Class 对象，表示结果值的类型。 如果这是一个构造函数，此变量返回 Void.class。

$class

$class 的值是一个 java.lang.Class 对象，表示编辑的方法所在的类。 即表示 $0 的类型。

addCatch()

addCatch() 插入方法体抛出异常时执行的代码，控制权会返回给调用者。 在插入的源代码中，异常用 $e 表示。

例如：

```java
CtMethod m = ...;
CtClass etype = ClassPool.getDefault().get("java.io.IOException");
m.addCatch("{ System.out.println($e); throw $e; }", etype);
```

转换成对应的 java 代码如下：

```java
try {
    // the original method body
} catch (java.io.IOException e) {
    System.out.println(e);
    throw e;
}
```

请注意，插入的代码片段必须以 throw 或 return 语句结束。

### 4.2 修改方法体

CtMethod 和 CtConstructor 提供 setBody() 来替换整个方法体。他将新的源代码编译成 Java 字节码，并用它替换原方法体。 如果给定的源文本为 null，则替换后的方法体仅包含返回语句，返回零或空值，除非结果类型为 void。

在传递给 setBody() 的源代码中，以 $ 开头的标识符具有特殊含义：

| 符号                  | 含义                                            |
| :-------------------- | :---------------------------------------------- |
| `$0`, `$1`, `$2`, ... | `this` and 方法的参数                           |
| `$args`               | 方法参数数组.它的类型为 `Object[]`              |
| `$$`                  | 所有实参。例如, `m($$)` 等价于 `m($1,$2,`...`)` |
| `$cflow(`...`)`       | `cflow` 变量                                    |
| `$r`                  | 返回结果的类型，用于强制类型转换                |
| `$w`                  | 包装器类型，用于强制类型转换                    |
| `$sig`                | 类型为 java.lang.Class 的参数类型数组           |
| `$type`               | 一个 java.lang.Class 对象，表示返回值类型       |
| `$class`              | 一个 java.lang.Class 对象，表示当前正在修改的类 |

注意 $_ 不可用。

替换表达式

Javassist 只允许修改方法体中包含的表达式。javassist.expr.ExprEditor 是一个用于替换方法体中的表达式的类。用户可以定义 ExprEditor 的子类来指定修改表达式的方式。

要运行 ExprEditor 对象，用户必须在 CtMethod 或 CtClass 中调用 instrument()。
例如，

```java
CtMethod cm = ... ;
cm.instrument(
    new ExprEditor() {
        public void edit(MethodCall m) throws CannotCompileException {
            if (m.getClassName().equals("Point")
                          && m.getMethodName().equals("move"))
                m.replace("{ $1 = 0; $_ = $proceed($$); }");
        }
    });
```

上述代码，搜索由 cm 表示的方法体，并用使用下面的代码替换 Point 中的 move()调用：

```
{ $1 = 0; $_ = $proceed($$); }
```

因此 move() 的第一个参数总是0。注意，替换的代码不是一个表达式，而是一个语句或块。 它不能是或包含 try-catch 语句。

方法 instrument() 搜索方法体。 如果它找到一个表达式，如方法调用、字段访问和对象创建，那么它调用给定的 ExprEditor 对象上的 edit() 方法。 edit() 的参数表示找到的表达式。 edit() 可以检查和替换该表达式。

调用 edit() 参数的 replace() 方法可以将表达式替换为我们给定的语句。如果给定的语句是空块，即执行replace("{}")，则将表达式删除。如果要在表达式之前或之后插入语句（或块），则应该将类似以下的代码传递给 replace()：

```
{ *before-statements;*
  $_ = $proceed($$);
  *after-statements;* }
```

无论表达式是方法调用、字段访问还是对象创建或其他。

如果表达式是读操作，第二个语句应该是：

```
$_ = $proceed();
```

如果表达式是写操作，则第二个语句应该是：

```
$proceed($$);
```

如果由 instrument() 搜索的方法是使用 -g 选项（类文件包含一个局部变量属性）编译的，目标表达式中可用的局部变量，也可以传递给 replace() 的源代码中使用。

javassist.expr.MethodCall

MethodCall 表示方法调用。MethodCall 的 replace() 方法用于替换方法调用，它接收表示替换语句或块的源代码。和 insertBefore() 方法一样，传递给 replace 的源代码中，以 $ 开头的标识符具有特殊的含义。

| 符号          | 含义                                                         |
| :------------ | :----------------------------------------------------------- |
| `$0`          | 方法调用的目标对象。它不等于 this，它代表了调用者。 如果方法是静态的，则 $0 为 null |
| `$1`, `$2` .. | 方法的参数                                                   |
| `$_`          | 方法调用的结果                                               |
| `$r`          | 返回结果的类型，用于强制类型转换                             |
| `$class`      | 一个 java.lang.Class 对象，表示当前正在修改的类              |
| `$sig`        | 类型为 java.lang.Class 的参数类型数组                        |
| `$type`       | 一个 java.lang.Class 对象，表示返回值类型                    |
| `$class`      | 一个 java.lang.Class 对象，表示当前正在修改的类              |
| `$proceed`    | 调用表达式中方法的名称                                       |

这里的方法调用意味着由 MethodCall 对象表示的方法。

其他标识符如 $w，$args 和 $$ 也可用。

除非方法调用的返回类型为 void，否则返回值必须在源代码中赋给 $_，$_ 的类型是表达式的结果类型。如果结果类型为 void，那么 $_ 的类型为Object，并且分配给 $_ 的值将被忽略。

$proceed 不是字符串值，而是特殊的语法。 它后面必须跟一个由括号括起来的参数列表。

javassist.expr.ConstructorCall

ConstructorCall 表示构造函数调用，例如包含在构造函数中的 this() 和 super()。ConstructorCall 中的方法 replace() 可以使用语句或代码块来代替构造函数。它接收表示替换语句或块的源代码。和 insertBefore() 方法一样，传递给 replace 的源代码中，以 $ 开头的标识符具有特殊的含义。

| 符号            | 含义                                            |
| :-------------- | :---------------------------------------------- |
| `$0`            | 构造调用的目标对象。它等于 this                 |
| `$1`, `$2`, ... | 构造函数的参数                                  |
| `$class`        | 一个 java.lang.Class 对象，表示当前正在修改的类 |
| `$sig`          | 类型为 java.lang.Class 的参数类型数组           |
| `$proceed`      | 调用表达式中构造函数的名称                      |

这里的构造函数调用是由 ConstructorCall 对象表示的。

其他标识符如 $w，$args 和 $$ 也可用。

由于任何构造函数必须调用超类的构造函数或同一类的另一个构造函数，所以替换语句必须包含构造函数调用，通常是对 $proceed() 的调用。

$proceed 不是字符串值，而是特殊的语法。 它后面必须跟一个由括号括起来的参数列表。

javassist.expr.FieldAccess

FieldAccess 对象表示字段访问。 如果找到对应的字段访问操作，ExprEditor 中的 edit() 方法将接收到一个 FieldAccess 对象。FieldAccess 中的 replace() 方法接收替源代码来替换字段访问。

在源代码中，以 $ 开头的标识符具有特殊含义：

| 符号       | 含义                                                         |
| :--------- | :----------------------------------------------------------- |
| `$0`       | 表达式访问的字段。它不等于 this。this 表示调用表达式所在方法的对象。如果字段是静态的，则 $0 为 null |
| `$1`       | 如果表达式是写操作，则写的值将保存在 $1 中。否则 $1 不可用   |
| `$_`       | 如果表达式是读操作，则结果值保存在 $1 中，否则将舍弃存储在 $_ 中的值 |
| `$r`       | 如果表达式是读操作，则 $r 读取结果的类型。 否则 $r 为 void   |
| `$class`   | 一个 java.lang.Class 对象，表示字段所在的类                  |
| `$type`    | 一个 java.lang.Class 对象，表示字段的类型                    |
| `$proceed` | 执行原始字段访问的虚拟方法的名称                             |

其他标识符如 $w，$args 和 $$ 也可用。
如果表达式是读操作，则必须在源文本中将值分配给 $。 $的类型是字段的类型。

javassist.expr.NewExpr

NewExpr 表示使用 new 运算符（不包括数组创建）创建对象的表达式。 如果发现创建对象的操作，NewEditor 中的 edit() 方法将接收到一个 NewExpr 对象。NewExpr 中的 replace() 方法接收替源代码来替换字段访问。

在源文本中，以 $ 开头的标识符具有特殊含义：

| 符号       | 含义                                            |
| :--------- | :---------------------------------------------- |
| `$0`       | null                                            |
| `$1`       | 构造函数的参数                                  |
| `$_`       | 创建对象的返回值。一个新的对象存储在 $_ 中      |
| `$r`       | 所创建的对象的类型                              |
| `$sig`     | 类型为 java.lang.Class 的参数类型数组           |
| `$type`    | 一个 java.lang.Class 对象，表示创建的对象的类型 |
| `$proceed` | 执行对象创建虚拟方法的名称                      |

其他标识符如 $w，$args 和 $$ 也可用。

javassist.expr.NewArray

NewArray 表示使用 new 运算符创建数组。如果发现数组创建的操作，ExprEditor 中的 edit() 方法一个 NewArray 对象。NewArray 中的 replace() 方法可以使用源代码来替换数组创建操作。

在源文本中，以$开头的标识符具有特殊含义：

| 符号       | 含义                                            |
| :--------- | :---------------------------------------------- |
| `$0`       | null                                            |
| `$1`, `$1` | 每一维的大小                                    |
| `$_`       | 创建数组的返回值。一个新的数组对象存储在 $_ 中  |
| `$r`       | 所创建的数组的类型                              |
| `$type`    | 一个 java.lang.Class 对象，表示创建的数组的类型 |
| `$proceed` | 执行数组创建虚拟方法的名称                      |

其他标识符如 $w，$args 和 $$ 也可用。
例如，如果按下面的方式创建数组：

```
String[][] s = new String[3][4];
```

那么 $1 和 $2 的值分别是 3 和 4。 $3 不可用。

例如，如果按下面的方式创建数组：

```
String[][] s = new String[3][];
```

那么 $1 的值为 3，但 $2 不可用。

javassist.expr.Instanceof

一个 InstanceOf 对象表示一个 instanceof 表达式。 如果找到 instanceof 表达式，则ExprEditor 中的 edit() 方法接收此对象。Instanceof 中的 replace() 方法可以使用源代码来替换 instanceof 表达式。

在源文本中，以$开头的标识符具有特殊含义：

| 符号       | 含义                                                         |
| :--------- | :----------------------------------------------------------- |
| `$0`       | null                                                         |
| `$1`       | instanceof 运算符左侧的值                                    |
| `$_`       | 表达式的返回值。类型为 boolean                               |
| `$r`       | instanceof 运算符右侧的值                                    |
| `$type`    | 一个 java.lang.Class 对象，表示 instanceof 运算符右侧的类型  |
| `$proceed` | 执行 instanceof 表达式的虚拟方法的名称。它需要一个参数（类型是 java.lang.Object）。如果参数类型和 instanceof 表达式右侧的类型一致，则返回 true。否则返回 false。 |

其他标识符如 $w，$args 和 $$ 也可用。

javassist.expr.Cast

Cast 表示 cast 表达式。如果找到 cast 表达式，ExprEditor 中的 edit() 方法会接收到一个 Cast 对象。 Cast 的 replace() 方法可以接收源代码来替换替换 �cast 表达式。

在源文本中，以$开头的标识符具有特殊含义：

| 符号       | 含义                                                         |
| :--------- | :----------------------------------------------------------- |
| `$0`       | null                                                         |
| `$1`       | 显示类型转换的目标类型（？）                                 |
| `$_`       | 表达式的结果值。$_ 的类型和被括号括起来的类型相同（？）      |
| `$r`       | 转换之后的类型，即被括号括起来的类型（？）                   |
| `$type`    | 一个 java.lang.Class 对象，和 $r 的类型相同                  |
| `$proceed` | 执行类型转换的虚拟方法的名称。它需要一个参数（类型是 java.lang.Object）。并在类型转换完成后返回它 |

其他标识符如 $w，$args 和 $$ 也可用。

javassist.expr.Handler

Handler 对象表示 try-catch 语句的 catch 子句。 如果找到 catch，ExprEditor 中的 edit() 方法会接收此对象。 Handler 中的 insertBefore() 方法会将收到的源代码插入到 catch 子句的开头。

在源文本中，以$开头的标识符具有意义：

| 符号    | 含义                                                   |
| :------ | :----------------------------------------------------- |
| `$1`    | catch 分支获得的异常对象                               |
| `$r`    | catch 分支获得的异常对象的类型，用于强制类型转换       |
| `$w`    | 包装类型，用于强制类型转换                             |
| `$type` | 一个 java.lang.Class 对象，表示 catch 捕获的异常的类型 |

如果一个新的异常分配给 $1，它将作为捕获的异常传递给原始的 catch 子句。

### 4.3 添加新方法和字段

添加新方法

Javassist 可以创建新的方法和构造函数。CtNewMethod 和 CtNewConstructor 提供了几个工厂方法来创建 CtMethod 或 CtConstructor 对象。make() 方法可以通过源代码来CtMethod 或 CtConstructor 对象。

例如：

```java
CtClass point = ClassPool.getDefault().get("Point");
CtMethod m = CtNewMethod.make(
                 "public int xmove(int dx) { x += dx; }",
                 point);
point.addMethod(m);
```

上面的代码向类 Point 添加了一个公共方法 xmove()。在这个例子中，x 是类 Point 的一个int 字段。

传递给 make() 和 setBody() 的源文本可以包括以 $ 开头的标识符 ($_ 除外)。 如果目标对象和目标方法名也被传递给 make() 方法，源文本中也可以包括 $proceed。

例如：

```java
CtClass point = ClassPool.getDefault().get("Point");
CtMethod m = CtNewMethod.make(
                 "public int ymove(int dy) { $proceed(0, dy); }",
                 point, "this", "move");
```

这个程序创建一个 ymove() 方法，定义如下：

```
public int ymove(int dy) { this.move(0, dy); }
```

注意，$proceed 已经被替换为 this.move。

Javassist 还提供了另一种添加新方法的方式。 你可以先创建一个抽象方法，然后给它一个方法体：

```java
CtClass cc = ... ;
CtMethod m = new CtMethod(CtClass.intType, "move",
                          new CtClass[] { CtClass.intType }, cc);
cc.addMethod(m);
m.setBody("{ x += $1; }");
cc.setModifiers(cc.getModifiers() & ~Modifier.ABSTRACT);
```

因为 Javassist 在类中添加了的方法是抽象的，所以在调用 setBody() 之后，必须将类显式地改回非抽象类。

相互递归的方法 (Mutual recursive methods)

Javassist 不能这种方法：如果它调用另一个方法，而另一个方法没有被添加到一个类（Javassist可以编译一个以递归方式调用的方法）。如果要向类添加相互递归方法，需要使用如下的技巧。假设你想要将方法 m() 和 n() 添加到由 cc 表示的类中：

```java
CtClass cc = ... ;
CtMethod m = CtNewMethod.make("public abstract int m(int i);", cc);
CtMethod n = CtNewMethod.make("public abstract int n(int i);", cc);
cc.addMethod(m);
cc.addMethod(n);
m.setBody("{ return ($1 <= 0) ? 1 : (n($1 - 1) * $1); }");
n.setBody("{ return m($1); }");
cc.setModifiers(cc.getModifiers() & ~Modifier.ABSTRACT);
```

你必须先创建两个抽象方法，并将它们添加到类中。然后设置它们的方法体，即使方法体包括互相递归的调用。 最后，必须将类更改为非抽象类。

添加一个字段

Javassist 还允许用户创建一个新字段。

```java
CtClass point = ClassPool.getDefault().get("Point");
CtField f = new CtField(CtClass.intType, "z", point);
point.addField(f);
```

该程序向类 Point 添加一个名为 z 的字段。
如果必须指定添加字段的初始值，那么上面的程序必须修改为：

```java
CtClass point = ClassPool.getDefault().get("Point");
CtField f = new CtField(CtClass.intType, "z", point);
point.addField(f, "0");  // initial value is 0
```

现在，方法 addField() 接收两个参数，第二个参数表示计算初始值的表达式。这个表达式可以是任意 Java 表达式，只要其结果与字段的类型匹配。 请注意，表达式不以分号结尾。

此外，上述代码可以重写为更简单代码：

```java
CtClass point = ClassPool.getDefault().get("Point");
CtField f = CtField.make("public int z = 0;", point);
point.addField(f);
```

删除成员

要删除字段或方法，请在 CtClass 的 removeField() 或 removeMethod() 方法。 一个CtConstructor 可以通过 CtClass 的 removeConstructor() 删除。

### 4.4 注解 (Annotations)

CtClass，CtMethod，CtField 和 CtConstructor 提供 getAnnotations() 方法，用于读取注解。 它返回一个注解类型的对象。

例如，假设有以下注解：

```java
public @interface Author {
    String name();
    int year();
}
```

下面是使用注解的代码：

```java
@Author(name="Chiba", year=2005)
public class Point {
    int x, y;
}
```

然后，可以使用 getAnnotations() 获取注解的值。 它返回一个包含注解类型对象的数组。

```java
CtClass cc = ClassPool.getDefault().get("Point");
Object[] all = cc.getAnnotations();
Author a = (Author)all[0];
String name = a.name();
int year = a.year();
System.out.println("name: " + name + ", year: " + year);
```

这段代码输出：

```
name: Chiba, year: 2005
```

由于 Point 的注解只有 @Author，所以数组的长度是 1，all[0] 是一个 Author 对象。 注解成员值可以通过调用Author对象的 name() 和 year() 来获取。

要使用 getAnnotations()，注释类型（如 Author）必须包含在当前类路径中。它们也必须也可以从 ClassPool 对象访问。如果未找到注释类型的类文件，Javassist 将无法获取该注释类型的成员的默认值。

### 4.5 运行时支持类

在大多数情况下，使用 Javassist 修改类不需要运行 Javassist。 但是，Javassist 编译器生成的某些字节码需要运行时支持类，这些类位于 javassist.runtime 包中（有关详细信息，请阅读该包的API文档）。请注意，javassist.runtime 是修改的类时唯一可能需要使用的包。 修改类的运行时不会再使用其他的 Javassist 类。

### 4.6 导入（Import）

源代码中的所有类名都必须是完整的（必须包含包名，java.lang 除外）。例如，Javassist 编译器可以解析 Object 以及 java.lang.Object。

要告诉编译器在解析类名时搜索其他包，请在 ClassPool中 调用 importPackage()。 例如，

```java
ClassPool pool = ClassPool.getDefault();
pool.importPackage("java.awt");
CtClass cc = pool.makeClass("Test");
CtField f = CtField.make("public Point p;", cc);
cc.addField(f);
```

第二行导入了 java.awt 包。 因此，第三行不会抛出异常。 编译器可以将 Point 识别为java.awt.Point。

注意 importPackage() 不会影响 ClassPool 中的 get() 方法。只有编译器才考虑导入包。 get() 的参数必须是完整类名。

### 4.7 限制 (Limitations)

在目前实现中，Javassist 中包含的 Java 编译器有一些限制：

- J2SE 5.0 引入的新语法（包括枚举和泛型）不受支持。注释由 Javassist 的低级 API 支持。 参见 javassist.bytecode.annotation 包（以及 CtClass 和 CtBehavior 中的� getAnnotations()）。对泛型只提供部分支持。更多信息，请参阅后面的部分；
- 初始化数组时，只有一维数组可以用大括号加逗号分隔元素的形式初始化，多维数组还不支持；
- 编译器不能编译包含内部类和匿名类的源代码。 但是，Javassist 可以读取和修改内部/匿名类的类文件；
- 不支持带标记的 continue 和 break 语句；
- 编译器没有正确实现 Java 方法调度算法。编译器可能会混淆在类中定义的重载方法（方法名称相同，查参数列表不同）。例如：

```java
class A {} 

class B extends A {} 

class C extends B {} 

class X { 
    void foo(A a) { .. } 
    void foo(B b) { .. } 
}
```

如果编译的表达式是 `x.foo(new C())`，其中 `x` 是 `X` 的实例，编译器将产生对 `foo(A)` 的调用，尽管编译器可以正确地编译 `foo((B) new C())` 。

- 建议使用 # 作为类名和静态方法或字段名之间的分隔符。 例如，在常规 Java 中，

```
javassist.CtClass.intType.getName()
```

在 javassist.CtClass 中的静态字段 intType 指示的对象上调用一个方法 getName()。 在Javassist 中，用户也可以写上面的表达式，但是建议写成这样：

```
javassist.CtClass#intType.getName()
```

使编译器可以快速解析表达式。

## 5. 字节码操作

Javassist 还提供了用于直接编辑类文件的低级级 API。 使用此 API之前，你需要详细了解Java 字节码和类文件格式，因为它允许你对类文件进行任意修改。

如果你只想生成一个简单的类文件，使用`javassist.bytecode.ClassFileWriter`就足够了。 它比`javassist.bytecode.ClassFile`更快而且更小。

### 获取 ClassFile 对象

javassist.bytecode.ClassFile 对象表示类文件。要获得这个对象，应该调用 CtClass 中的 getClassFile() 方法。
你也可以直接从类文件构造 javassist.bytecode.ClassFile 对象。 例如：

```java
BufferedInputStream fin
    = new BufferedInputStream(new FileInputStream("Point.class"));
ClassFile cf = new ClassFile(new DataInputStream(fin));
```

这代码段从 Point.class 创建一个 ClassFile 对象。
ClassFile 对象可以写回类文件。ClassFile 的 write() 将类文件的内容写入给定的 DataOutputStream。

### 5.2 添加和删除成员

ClassFile 提供了 addField()，addMethod() 和 addAttribute()，来向类添加字段、方法和类文件属性。

注意，FieldInfo，MethodInfo 和 AttributeInfo 对象包括到 ConstPool（常量池表）对象的链接。 ConstPool 对象必须对 ClassFile 对象和添加到该 ClassFile 对象的 FieldInfo（或MethodInfo 等）对象是通用的。 换句话说，FieldInfo（或MethodInfo等）对象不能在不同的ClassFile 对象之间共享。

要从 ClassFile 对象中删除字段或方法，必须首先获取包含该类的所有字段的 java.util.List 对象。 getFields() 和 getMethods() 返回列表。可以通过在List对象上调用 remove() 来删除字段或方法。可以以类似的方式去除属性。在 FieldInfo 或 MethodInfo 中调用 getAttributes() 以获取属性列表，并从列表中删除一个。

### 5.3 遍历方法体

使用 CodeIterator 可以检查方法体中的每个字节码指令，要获得 CodeIterator 对象，参考以下代码：

```java
ClassFile cf = ... ;
MethodInfo minfo = cf.getMethod("move");    // we assume move is not overloaded.
CodeAttribute ca = minfo.getCodeAttribute();
CodeIterator ci = ca.iterator();
```

CodeIterator 对象允许你逐个访问每个字节码指令。下面展示了一部分 CodeIterator 中声明的方法：

- void begin（）
  移动到第一条指令。
- void move（int index）
  移动到指定位置的指令。
- boolean hasNext（）
  是否有下一条指定
- int next（）
  返回下一条指令的索引。注意，它不返回下一条指令的操作码。
- int byteAt（int index）
  返回索引处的无符号8位整数。
- int u16bitAt（int index）
  返回索引处的无符号16位整数。
- int write（byte [] code，int index）
  在索引处写入字节数组。
- void insert（int index，byte [] code）
  在索引处插入字节数组。自动调整分支偏移量。

以下代码段打印了方法体中所有的指令：

```java
CodeIterator ci = ... ;
while (ci.hasNext()) {
    int index = ci.next();
    int op = ci.byteAt(index);
    System.out.println(Mnemonic.OPCODE[op]);
}
```

### 5.4 生成字节码序列

`Bytecode` 对象表示字节码指令序列。它是一个可扩展的字节码数组。
以下是示例代码段：

```java
ConstPool cp = ...;    // constant pool table
Bytecode b = new Bytecode(cp, 1, 0);
b.addIconst(3);
b.addReturn(CtClass.intType);
CodeAttribute ca = b.toCodeAttribute();
```

这段代码产生以下序列的代码属性：

```
iconst_3
ireturn
```

您还可以通过调用 Bytecode 中的 get() 方法来获取包含此序列的字节数组。获得的数组可以插入另一个代码属性。
Bytecode 提供了许多方法来添加特定的指令，例如使用 addOpcode() 添加一个 8 位操作码，使用 addIndex() 用于添加一个索引。每个操作码的值定义在 Opcode 接口中。
addOpcode() 和添加特定指令的方法，将自动维持最大堆栈深度，除非控制流没有分支。可以通过调用 Bytecode 的 getMaxStack() 方法来获得这个深度。它也反映在从 Bytecode对象构造的 CodeAttribute 对象上。要重新计算方法体的最大堆栈深度，可以调用 CodeAttribute 的 computeMaxStack() 方法。

### 5.5 注释（元标签）

注释作为运行时不可见（或可见）的注记属性，存储在类文件中。调用 getAttribute（AnnotationsAttribute.invisibleTag）方法，可以从 ClassFile，MethodInfo 或 FieldInfo 中获取注记属性。更多信息，请参阅 `javassist.bytecode.AnnotationsAttribute`和`javassist.bytecode.annotation` 包的 javadoc 手册。

Javassist还允许您通过更高级别的API访问注释。 如果要通过CtClass访问注释，请在CtClass或CtBehavior中调用getAnnotations（）。

## 6. 泛型

Javassist 的低级别 API 完全支持 Java 5 引入的泛型。但是，高级别的API（如CtClass）不直接支持泛型。

Java 的泛型是通过擦除技术实现。 编译后，所有类型参数都将被删除。 例如，假设您的源代码声明一个参数化类型 Vector<String>：

```java
Vector<String> v = new Vector<String>();
  :
String s = v.get(0);
```

编译后的字节码等价于以下代码：

```java
Vector v = new Vector();
  :
String s = (String)v.get(0);
```

因此，在编写字节码变换器时，您可以删除所有类型参数，因为 Javassist 的编译器不支持泛型。如果源代码使用 Javassist 编译，例如通过 CtMethod.make()，源代码必须显式类型转换。如果源代码由常规 Java 编译器（如javac）编译，则不需要做类型转换。

例如，如果你有一个类：

```java
public class Wrapper<T> {
  T value;
  public Wrapper(T t) { value = t; }
}
```

并想添加一个接口 Getter<T> 到类 Wrapper<T>：

```java
public interface Getter<T> {
  T get();
}
```

那么你真正要添加的接口其实是Getter（将类型参数<T>掉落），最后你添加到 Wrapper 类的方法是这样的：

```java
public Object get() { return value; }
```

注意，不需要类型参数。 由于 get 返回一个 Object，如果源代码是由 Javassist 编译的，那么在调用方需要进行显式类型转换。 例如，如果类型参数 T 是 String，则必须插入（String），如下所示：

```java
Wrapper w = ...
String s = (String)w.get();
```

## 7.可变参数

目前，Javassist 不直接支持可变参数。 因此，要使用 varargs 创建方法，必须显式设置方法修饰符。假设要定义下面这个方法：

```java
public int length(int... args) { return args.length; }
```

使用 Javassist 应该是这样的：

```java
CtClass cc = /* target class */;
CtMethod m = CtMethod.make("public int length(int[] args) { return args.length; }", cc);
m.setModifiers(m.getModifiers() | Modifier.VARARGS);
cc.addMethod(m);
```

参数类型`int ...`被更改为`int []`，`Modifier.VARARGS`被添加到方法修饰符中。

要在由 Javassist 的编译器编译的源代码中调用此方法，需要这样写：

```
length(new int[] { 1, 2, 3 });
```

而不是这样：

```
length(1, 2, 3);
```

## 8. J2ME

如果要修改 J2ME 执行环境的类文件，则必须先执行预验证。预验证基本上是生成堆栈映射，这类似于在 JDK 1.6 中引入 J2SE 的堆栈映射表。当`javassist.bytecode.MethodInfo.doPreverify` 为 true 时，Javassist 才会维护 J2ME 的堆栈映射。

对于指定的 CtMethod 对象，你可以调用以下方法，手动生成堆栈映射：

```java
m.getMethodInfo().rebuildStackMapForME(cpool);
```

这里，cpool 是一个 ClassPool 对象，通过在 CtClass 对象上调用 getClassPool() 可以获得。 ClassPool 对象负责从给定类路径中查找类文件。要获得所有的 CtMethod 对象，需要在 CtClass 对象上调用 getDeclaredMethods() 方法。

## 9.装箱/拆箱

Java 中的装箱和拆箱是语法糖。没有用于装箱或拆箱的字节码。所以 Javassist 的编译器不支持它们。 例如，以下语句在 Java 中有效：

```java
Integer i = 3;
```

因为隐式地执行了装箱。 但是，对于 Javassist，必须将值类型从 int 显式地转换为 Integer：

```java
Integer i = new Integer(3);
```

## 10. 调试

将 CtClass.debugDump 设为本地目录。 然后 Javassist 修改和生成的所有类文件都保存在该目录中。要停止此操作，将 CtClass.debugDump 设置为 null 即可。其默认值为 null。

例如，

```
CtClass.debugDump =“./dump”;
```

所有修改的类文件都保存在 ./dump 中。

## 11.引用

**原文出处**：

<https://github.com/jboss-javassist/javassist/wiki/Tutorial-1>

[https://github.com/jboss-javassist/javassist/wiki/Tutorial-2](https://link.jianshu.com/?t=https://github.com/jboss-javassist/javassist/wiki/Tutorial-2)

<https://github.com/jboss-javassist/javassist/wiki/Tutorial-3>

**翻译：**

作者：二胡

连接：https://www.jianshu.com/p/43424242846b

来源：简书

**第一来源**

作者：[21aspnet](https://me.csdn.net/21aspnet)

连接：https://blog.csdn.net/21aspnet/article/details/81671777

来源：CSDN