## 切面表达式

在使用spring框架配置AOP的时候，不管是通过XML配置文件还是注解的方式都需要定义pointcut"切入点"。

例如定义切入点表达式  **execution (\* com.sample.service.impl..\*.\*(..))**

**execution()**是最常用的切点函数，其语法如下所示：

 整个表达式可以分为五个部分：

1. **execution()**: 表达式主体。
2. 第一个*****号：表示返回类型，*****号表示所有的类型。
3. 包名：表示需要拦截的包名，后面的两个句点(**..**)表示当前包和当前包的所有子包
4. 第二个*****号：表示类名，*****号表示所有的类。
5. ***(..)**:最后这个星号表示方法名，*****号表示所有的方法，后面括号里面表示方法的参数，两个句点表示任何参数。

### excution() 

**表示满足某一匹配模式的所有目标类方法连接点。**

#### 语法

execution(<修饰符模式>? **<返回类型模式> <方法名模式>(<参数模式>)** <异常模式>?)  除了返回类型模式、方法名模式和参数模式外，其它项都是可选的

#### 特殊符号

*****：表示匹配任意的值

**..**：表示匹配任意的数量且不限制类型

**+**：表示匹配指定的类及其子类

#### 例子

1. **execution(public * com.feb13th..*(..))**：匹配 `com.feb13th`包下的所有`public`方法。方法参数不限，返回值不限，方法名不限。
2. **execution(* com.feb13th.Enter+.*(..))**：匹配`Enter`接口及其所有实现类的方法。即使不存在`Enter`中的方法也会被切面。方法参数不限，返回值类型不限，方法修饰符不限。方法名不限。
3. **execution(* com.feb13th..*(String, *))**：匹配`com.feb13th`包下，方法第一个参数为`String`类型的方法。方法返回值不限，修饰符不限，方法名不限。
4. **execution(* com.feb13th..*(Object+))**：匹配`com.feb13th`包下的所有方法第一个入参为`Object`及其子类的方法。方法返回值不限，修饰符不限，方法名不限。



### @annotation()

**表示标注了特定注解的目标方法连接点**

#### 语法

@annotation(注解)

#### 例子

**@annotation(com.feb13th.Enter)**：匹配所有标注了`com.feb13th.Enter`注解的方法。



### args()

**通过判别目标类方法运行时入参对象的类型定义指定连接点**

#### 语法

args(类名..)

#### 例子

**args(com.feb13th.Enter)**：匹配所有入参有且仅有一个类型和`com.feb13th.Enter`匹配的方法。



### @args()

**通过判别目标方法的运行时入参对象的类是否标注特定注解来指定连接点**

#### 语法

@args(注解)

#### 例子

**@args(com.feb13th.Enter)**：匹配有且仅有一个入参，且入参对象的类标注了`com.feb13th.Enter`注解的方法。



### within()

**表示特定域下的所有连接点**

#### 语法

within(类名匹配串)

#### 例子

1. **within(com.feb13th.*)**：匹配`com.feb13th`包下的所有方法
2. **within(com.feb13th.service.*Service)**：匹配`com.feb13th.service`包下所有以`Service`结尾的类中的方法。



### @within()

**假如目标类按类型匹配于某个类A，且类A标注了特定注解，则目标类的所有连接点匹配这个切点。**

#### 语法

@within(类型注解类名)

#### 例子

**@within(com.feb13th.Enter)**：假如一个类标注了`com.feb13th.Enter`注解，那么该类及其子类的所有方法都会有切面。



### target()

**假如目标类按类型匹配于指定类，则目标类的所有连接点匹配这个切点**

#### 语法

target(类名)

#### 例子

**target(com.feb13th.Enter)**：匹配`com.feb13th.Enter`类及其子类中所有的方法。



### @target()

**目标类标注了特定注解，则目标类所有连接点匹配该切点**

#### 语法

@target(类型注解类名)

#### 例子

**@target(com.feb13th.Enter)**：匹配所有标注了`com.feb13th.Enter`类中的所有方法。不包括其子类。



## 组合多个表达式

在多个表达式中可以使用**”&&“** 、 **”||“** 、**”！“**  连接多个表达式。



## Spring 切面通知类型

| 注解            | 通知                       |
| --------------- | -------------------------- |
| @Before         | 在方法调用之前执行         |
| @After          | 在方法返回或抛出异常后执行 |
| @Around         | 环绕通知                   |
| @AfterReturning | 目标方法返回后调用         |
| @AfterThrowing  | 方法抛出异常后执行         |



## Spring 中使用 AOP

### 启用AOP

启用AOP特性可以在`xml`文件中配置或使用注解开启。

**使用XML配置**

```xml
<!-- 支持注解 -->
<context:annotation-config />
<context:component-scan base-package="com.feb13th"/>
<!-- 配置 aop  -->
<aop:aspectj-autoproxy/>
```

**使用注解配置**

```java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan(basePackages = "com.feb13th")
public class Test {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext();
    }
}
```

### 编写切面类

切面类需使用`Aspect`注解进行标注，具体看下面：

```java
@Aspect
@Component
public class MyAspect {

    // 自定义的切入点
    @Pointcut("execution(* android..*(..))")
    public void pointCut() {}

    // 方法执行前执行
    @Before("pointCut()")
    public void before(){}

    // 方法执行后执行
    @After("pointCut()")
    public void after(){}

    // 环绕执行方法, proceed()方法用于调用被切面的方法
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        return joinPoint.proceed();
    }

    // 方法返回后调用
    @AfterReturning("pointCut()")
    public void afterReturning() {}

    // 方法抛出异常后调用
    @AfterThrowing("pointCut()")
    public void afterThrowing() {}
}
```



## 常见问题

### 获取方法信息

```java
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Signature signature = joinPoint.getSignature();
        if (!(signature instanceof MethodSignature)) {
            return joinPoint.proceed();
        }
        // 只有当前切入点为方法时，才能获取到方法信息
        MethodSignature methodSignature = (MethodSignature) signature;
        Method method = methodSignature.getMethod();
        // ...

        return joinPoint.proceed();
    }
```

### 获取入参信息

```java
@Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        // 所有的入参值，可能会存在 null
        Object[] args = joinPoint.getArgs();
        Signature signature = joinPoint.getSignature();
        // 方法名
        String name = signature.getName();
        // ...

        return joinPoint.proceed();
    }
```

### 出现灵异事件，重启后解决

方法正常执行时没问题，方方法抛出错误时，程序不会根据原预定的逻辑执行。

出现这种情况一般是由于操作了共享变量导致的。

正常逻辑中会对共享变量进行修改，但是当程序出错时，对共享变量的修改的代码无法被执行。所有导致了这种灵时间。

解决方案就是将一定要执行的代码写在`finally`块中。









