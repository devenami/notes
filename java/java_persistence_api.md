## Entities

**标准的实体类的要求**

* 必须被 `javax.persistence.Entity`注解修饰
* 实体类必须是 `public`或者`protect` 的， 必须得有一个无参的构造器，当然，它可以有其他的构造器。
* 必须不能为`final`修饰，没有方法或实体常量必须被声明为`final`.
* 如果实体需要被远程调用或传输，必须实现`Serializable`接口。
* 实体类可以扩展非实体类和实体类。非实体类可以扩展实体类。
* 实体的属性必须被声明为`private`、`protect`或`package-privare`，并且只能通过实体类的方法访问。客户端必须通过访问者或业务方法访问实体的状态。 

**实体持久化字段和属性的类型**

- Java 基本数据类型
- `java.lang.String`
- 其他可被序列化的类：
  - Java 基本数据类型的包装类
  - `java.math.BigInteger`
  - `java.math.BigDecimal`
  - `java.util.Date`
  - `java.util.Calendar`
  - `java.sql.Date`
  - `java.sql.Time`
  - `java.sql.TimeStamp`
  - 用户定义的可序列化的类型
  - `byte[]`
  - `Byte[]`
  - `char[]`
  - `Character[]`
- 枚举类型
- 其他实体和/或实体的集合
- 内部类

**持久化字段**

如果一个字段没有注解`javax.persitence.Transient`或者没有被标注为`transient`,那么这个字段将会被持久化到数据库。

**实体中的主键**

每个实体都有一个唯一的对象标识。

简单的主键可以使用`javax.persitence.Id`注解来表明这个字段是一个主键。

复合主键必须对应于单个持久属性或字段，或者对应于一组单个持久性属性或字段。必须在主键类中定义组合主键 。复合主键可以使用`javax.persistence.EmbeddedId`和`javax.persistence.IdClass`注解来表明。 

复合主键的类型必须是下面的几种类型

* java 基本数据类型
* java 基本数据类型的包装类
* `java.lang.String`
* `java.util.Date`
* `java.sql.Date`

主键必须是一个整形，而不能是浮点型。

**主键类的要求**

* 访问修饰符必须是`public`
* 如果使用基于属性的访问，则主键类的属性必须是`public`或`protect`的。
* 必须可以被序列化（实现`Serializable`接口）
*  必须表示复合主键并将其映射到实体类的多个字段或属性，或者必须将其表示并映射为可嵌入类。 
* 如果类映射到实体类的多个字段或属性，则主键类中主键字段或属性的名称和类型必须与实体类的名称和类型匹配。 

```java
public final class LineItemKey implements Serializable {
    public Integer orderId;
    public int itemId;

    public LineItemKey() {}

    public LineItemKey(Integer orderId, int itemId) {
        this.orderId = orderId;
        this.itemId = itemId;
    }

    public boolean equals(Object otherOb) {
        if (this == otherOb) {
            return true;
        }
        if (!(otherOb instanceof LineItemKey)) {
            return false;
        }
        LineItemKey other = (LineItemKey) otherOb;
        return (
                    (orderId==null?other.orderId==null:orderId.equals
                    (other.orderId)
                    )
                    &&
                    (itemId == other.itemId)
                );
    }

    public int hashCode() {
        return (
                    (orderId==null?0:orderId.hashCode())
                    ^
                    ((int) itemId)
                );
    }

    public String toString() {
        return "" + orderId + "-" + itemId;
    }
}
```

**实体之间的关系**

* One-to-one：一对一的关系使用`javax.persistence.OneToOne`注解来声明持久化字段或属性
* One-to-many：一对多的关系使用`javax.persistence.OneToMany`来声明持久化字段或属性
* Many-to-one：多对一的关系使用`javax.persistence.ManyToOne`来声明持久化字段或属性
* Many-to-many：多对多的关系使用`javax.persistence.ManyToMany`来声明持久化字段或属性

**实体关系之间的方向**

实体之间的关系可以是双向的，也可以是单向的。

**双向关系**

在一个双向关系中，每一个实体都有一个字段来映射其他的实体。

双向关联的关系必须有如下规则：

* 双向关系的反面必须通过使用`@OneToOne`，`@ OneToMany`或`@ManyToMany`注释的`mappedBy`元素来引用其拥有方。 `mappedBy`元素指定作为关系所有者的实体中的属性或字段。 
* 多对一的关系中，多的一次不能使用`mappedBy`。因为多的一方始终是关系的拥有方。
* 对于一对一的双向关系，拥有方对应于包含相应外键的一侧。
* 对于多对多双向关系，任何一方都可能是拥有方。 

**级联删除关系**

使用关系的实体通常依赖于关系中另一个实体的存在， 如果一方删除，也另一方也应该删除，这种方式叫做"级联删除"。

使用`@OneToOne`和`@OneToMany`关系的`cascade = REMOVE`元素规范指定级联删除关系 

```java
@OneToMany(cascade=REMOVE, mappedBy="customer")
public Set<Order> getOrders() { return orders; }
```

**抽象实体**

```java
@Entity
public abstract class Employee {
    @Id
    protected Integer employeeId;
    ...
}
@Entity
public class FullTimeEmployee extends Employee {
    protected Integer salary;
    ...
}
@Entity
public class PartTimeEmployee extends Employee {
    protected Float hourlyWage;
}
```

**映射超类**

通过使用`javax.persistence.MappedSuperclass`注解对类进行装饰来指定映射的超类。 

```java
@MappedSuperclass
public class Employee {
    @Id
    protected Integer employeeId;
    ...
}
@Entity
public class FullTimeEmployee extends Employee {
    protected Integer salary;
    ...
}
@Entity
public class PartTimeEmployee extends Employee {
    protected Float hourlyWage;
    ...
}
```



## EntityManger

**获取EntityManger**

```java
@PersistenceContext
EntityManager em;
```

**获取EntityMangerFactory**

```java
@PersistenceUnit
EntityManagerFactory emf;
```

我们可以从`EntityMangerFactory`中通过调用`createEntityManger`方法获取`EntityManger`

```java
EntityManager em = emf.createEntityManager();
```

**使用EntityManger从`data store`中通过主键查询实体

```java
@PersistenceContext
EntityManager em;
public void enterOrder(int custID, Order newOrder) {
    Customer cust = em.find(Customer.class, custID);
    cust.getOrders().add(newOrder);
    newOrder.setCustomer(cust);
}
```

**Entity的声明周期**

Entity 的声明周期有 `new`（新实体）, `managed`（被管理）, `detached`（分离）, `removed`（被移除）四种状态

* `new`没有持久化标识(主键)，和持久化上下文(persistence context)没有关联
* `managed`有持久化标识，且和持久化上下文有关联。
* `detached` 有持久化标识，并且当前不与持久性上下文相关联。 
* `removed`有持久化标识，与持久性上下文关联，并计划从数据存储中删除。

**持久化实体(保存)**

通过调用`persist`方法或通过在关系注释中设置了`cascade = PERSIST`或`cascade = ALL`元素的相关实体调用的级联持久操作，新实体实例变为托管和持久化 。如果实体已经是被管理的实例，则会忽略`persist`操作。如果`persist`的方法被 `removed`状态的实体调用，则该实体将会从`removed`状态转变为`managed`状态。如果实体为`detached`状态，则`persist`将会抛出`IllegalArgumentException `, 并且事务的提交会失败。

```java
@PersistenceContext
EntityManager em;
...
public LineItem createLineItem(Order order, Product product,
        int quantity) {
    LineItem li = new LineItem(order, product, quantity);
    order.getLineItems().add(li);
    em.persist(li);
    return li;
}
```

`persist`（持久化）操作将传播到实体中所有的关系注解(OneToMany等)，如果关系注解中的`cascade`  属性被设置为`ALL`或者`PERSIST`。则其注解的对象也会被持久化到数据库。

```java
@OneToMany(cascade=ALL, mappedBy="order")
public Collection<LineItem> getLineItems() {
    return lineItems;
}
```

**删除 Entity 实例（删除）**

`managed`(被管理得)实例通过调用`remove`方法删除，并且如果被删除的实例中的关系注解的`cascade`属性被设置为`ALL`或`REMOVE`，则对应的对象也会被删除。如果删除的对象是一个新的实体，则删除操作将会被忽略。如果删除的是一个`detached`对象，则会抛出`IllegalArgumentException `，且事务提交也会失败。如果删除的是一个已被删除的实体，则操作将被忽略。当事务提交或者执行`flush`操作时，数据将会从数据库中删除。

```java
public void removeOrder(Integer orderId) {
    try {
        Order order = em.find(Order.class, orderId);
        em.remove(order);
    }...
```

**创建查询语句**

`EntityManager.createQuery `和`EntityManager.createNamedQuery `被用来使用`JPA`查询语言从数据库查询数据。

* `createQuery `被用来创建动态查询

```java
public List findWithName(String name) {
return em.createQuery(
    "SELECT c FROM Customer c WHERE c.name LIKE :custName")
    .setParameter("custName", name)
    .setMaxResults(10)
    .getResultList();
}
```

* `createNamedQuery `被用来创建静态查询，通过使用`javax.persistence.NamedQuery `注解在实体类上声明，该注解中的`name`元素指定了该静态查询语句的名称。`query`  元素指定了查询的语句。

```java
@NamedQuery(
    name="findAllCustomersWithName",
    query="SELECT c FROM Customer c WHERE c.name LIKE :custName"
)
```

```java
@PersistenceContext
public EntityManager em;
...
customers = em.createNamedQuery("findAllCustomersWithName")
    .setParameter("custName", "Smith")
    .getResultList();
```

**查询语句中的参数**

命名参数在查询中使用（:）作为前缀，它会被`javax.persistence.Query.setParameter(String name, Object value) `方法绑定

```java
public List findWithName(String name) {
    Query query = em.createQuery("SELECT c FROM Customer c WHERE c.name LIKE :custName");
    query.setParameter("custName", name);
    return query.getResultList();
}
```

也可以使用参数下标的形式对参数进行赋值

```java
public List findWithName(String name) {
    return em.createQuery(
        “SELECT c FROM Customer c WHERE c.name LIKE ?1”)
        .setParameter(1, name)
        .getResultList();
}
```

**持久化单元(配置文件)**

持久性单元定义由应用程序中的EntityManager实例管理的所有实体类的集合。这组实体类表示单个数据存储中包含的数据。 

持久性单元由persistence.xml配置文件定义，META-INF目录包含persistence.xml的JAR文件或目录称为持久性单元的根，持久性单元的范围由持久性单元的根确定。

```xml
<persistence>
    <persistence-unit name="OrderManagement">
        <description>
            This unit manages orders and customers.
            It does not rely on any vendor-specific features and can
            therefore be deployed to any persistence provider.
        </description>
        <jta-data-source>jdbc/MyOrderDB</jta-data-source>
        <jar-file>MyOrderApp.jar</jar-file>
        <class>com.widgets.Order</class>
        <class>com.widgets.Customer</class>
    </persistence-unit>
</persistence>
```



## JPA 在WEB 中的应用

**定义持久化单元**

持久化单元被定义在`persistence.xml`文件中，其中包含了以下内容：

*  `persistence` 元素，用于标识描述符验证的模式，并包含`persistence-unit`元素。 
* `persistence-unit` 元素定义了持久化单元的名称和事务类型
* `description` 可选
* `jta-data-source` 它指定JTA数据源的全局JNDI名称。 

`jta-data-source` 元素指示实体管理器参与的事务是JTA事务，这意味着事务由容器管理。 也可以使用`resource-local`来管理事务，这是由应用程序本身提供的事务。

资源本地实体管理器不能参与全局事务。此外，Web容器不会回滚由编写不良的应用程序留下的待处理事务。 

**创建持久化实体**

```java
@Entity
@Table(name="WEB_BOOKSTORE_BOOKS")
public class Book implements Serializable {
	
    @Id
    private String bookId;
    private String title;
	... //getter and setter
}
```

**通过EntityManager访问数据**

```java
public final class ContextListener implements SerlvetContextListener {
...
@PersistenceUnit
private EntityManagerFactory emf;

public void contextInitialized(ServletContexEvent event) {
    context = event.getServletContext();
    ...
    try {
        BookDBAO bookDB = new BookDBAO(emf);
        context.setAttribute("bookDB", bookDB);
    } catch (Exception ex) {
        System.out.println(
            "Couldn’t create bookstore database bean: "
                 + ex.getMessage());
    }
}
}
```

`BookDBAO`源码如下

```java
private EntityManager em;

public BookDBAO (EntityManagerFactory emf) throws Exception {
    em = emf.getEntityManager();
    ...
}
```

也可以使用下面的方式在 `DAO` 中直接获取 `EntityManager`

```java
public class BookDBAO {

    @PersistenceContext
    private EntityManager em;
...
```

我们可以通过`EntityManager`内提供的方法执行`CRUD`操作

**事务**

```java
@Resource
UserTransaction utx;
...
try {
    utx.begin();
    bookDBAO.buyBooks(cart);
    utx.commit();
} catch (Exception ex) {
    try {
        utx.rollback();
    } catch (Exception exe) {
        System.out.println("Rollback failed: "+exe.getMessage());
}
...
```



## JAVA 持久化查询语言

**基本查询语句**

```sql
SELECT p FROM Player p
```

`FROM` 元素后面跟的不再试 SQL 语言中的表名，而是JAVA 中的实体名称。

**去重**

```sql
SELECT DISTINCT
 p
FROM Player p
WHERE p.position = ?1
```

**命名参数**

```sql
SELECT DISTINCT p
FROM Player p
WHERE p.position = :position AND p.name = :name
```

**关联查询**

***一对多 或 多对多***

```sql
SELECT DISTINCT p
FROM Player p, IN(p.teams) t
```

也可以使用 `JOIN` 操作符

```sql
SELECT DISTINCT p
FROM Player p JOIN p.teams t

-- 也可以重写为：
SELECT DISTINCT p
FROM Player p
WHERE p.team IS NOT EMPTY
```

***一对一***

```sql
SELECT t
 FROM Team t JOIN t.league l
 WHERE l.sport = ’soccer’ OR l.sport =’football’
```

***整合使用***

```sql
SELECT DISTINCT p
FROM Player p, IN (p.teams) AS t
WHERE t.city = :city
```

***遍历多条关系***

```sql
SELECT DISTINCT p
FROM Player p, IN (p.teams) t
WHERE t.league = :league
```

***通过关联对象的属性查询***

```sql
SELECT DISTINCT p
FROM Player p, IN (p.teams) t
WHERE t.league.sport = :sport
```

***LIKE 查询***

```sql
SELECT p
 FROM Player p
 WHERE p.name LIKE ’Mich%’
```

***IS NULL***

```sql
SELECT t
 FROM Team t
 WHERE t.league IS NULL
```

***IS EMPTY***

```SQL
SELECT p
FROM Player p
WHERE p.teams IS EMPTY
```

***BETWEEN***

```SQL
SELECT DISTINCT p
FROM Player p
WHERE p.salary BETWEEN :lowerSalary AND :higherSalary
```

***比较***

```sql
SELECT DISTINCT p1
FROM Player p1, Player p2
WHERE p1.salary > p2.salary AND p2.name = :name
```

***更新***

```sql
UPDATE Player p
SET p.status = ’inactive’
WHERE p.lastPlayed < :inactiveThresholdDate
```

***删除***

```sql
DELETE
FROM Player p
WHERE p.status = ’inactive’
AND p.teams IS EMPTY
```





















