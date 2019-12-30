## 类结构

* >  JmsAccessor 实现 InitializingBean 
  >
  > -- 实现 afterPropertiesSet 方法，对当前ConnectionFactory 进行了非空判断
  >
  > 
  >
  > 拥有ConnectionFactory 对象及创建 session 的两个属性
  >
  > -- sessionTransacted
  >
  > -- sessionAcknowledgeMode
  >
  >  
  >
  > 创建Connection
  >
  > 创建 Session
  >
  > 

* > ```java
  > public abstract class JmsDestinationAccessor extends JmsAccessor
  > ```
  >
  > JmsDestinationAccessor 集成 JmsAccessor
  >
  > 持有两个消息接收超时时间设置参数
  >
  > 持有 DestinationResolver 对象
  >
  > 持有 是否是订阅模式的参数 pubSubDomain = false
  >
  >  
  >
  > 方法 
  >
  > ```java
  > // 通过调用 DestinationResolver  对象的方法创建 destination 对象
  > Destination resolveDestinationName(Session session, String destinationName) 
  > ```
  >
  >  ```java
  > // 接收消息
  > Message receiveFromConsumer(MessageConsumer consumer, long timeout) 
  >     
  > // 通过对超时时间的判断，调用不同的接收方法：
  > if (timeout > 0) {
  >     return consumer.receive(timeout);
  > }
  > else if (timeout < 0) {
  >     return consumer.receiveNoWait();
  > }
  > else {
  >     return consumer.receive();
  > }
  >  ```

* > ```java
  > public abstract class AbstractJmsListeningContainer extends JmsDestinationAccessor implements BeanNameAware, DisposableBean, SmartLifecycle
  > ```
  >
  >  所有需要实现 JMS 链接的容器的公共基类。
  >
  > 提供了最基本的生命周期管理，主要管理 shared JMS Connection 。
  >
  > 子类通过实现 `sharedConnectionEnabled()`、`doInitialize()`、`doShutdown()`这三个模板方法来接入声明周期。
  >
  > 
  >
  > 属性：
  >
  > ```java
  > private String clientId;
  > private boolean autoStartup = true;
  > private int phase = Integer.MAX_VALUE;
  > private String beanName;
  > private Connection sharedConnection;
  > private boolean sharedConnectionStarted = false;
  > protected final Object sharedConnectionMonitor = new Object();
  > private boolean active = false;
  > private boolean running = false;
  > private final List<Object> pausedTasks = new LinkedList<Object>();
  > ```
  >
  >  钩子方法
  >
  > ```java
  > protected void validateConfiguration() {
  > }
  > 
  > void initialize() {
  >     synchronized (this.lifecycleMonitor) {
  >         this.active = true;
  >         // 唤醒所有该声明周期管理的线程
  >         this.lifecycleMonitor.notifyAll();
  >     }
  > }
  > 
  > void doInitialize()
  > boolean sharedConnectionEnabled()
  > void doShutdown()
  > ```

* >  ```java
  >  public abstract class AbstractMessageListenerContainer extends AbstractJmsListeningContainer implements MessageListenerContainer
  >  ```
  >
  >  属性：
  >
  >  ```java
  >  private volatile Object destination;
  >  private volatile String messageSelector;
  >  private volatile Object messageListener;
  >  
  >  private boolean subscriptionDurable = false;
  >  private boolean subscriptionShared = false;
  >  
  >  private String subscriptionName;
  >  
  >  private Boolean replyPubSubDomain;
  >  private boolean pubSubNoLocal = false;
  >  
  >  private MessageConverter messageConverter;
  >  private ExceptionListener exceptionListener;
  >  private ErrorHandler errorHandler;
  >  
  >  private boolean exposeListenerSession = true;
  >  private boolean acceptMessagesWhileStopping = false;
  >  ```
  >

* >  ```java
  >  public abstract class AbstractPollingMessageListenerContainer extends AbstractMessageListenerContainer
  >  ```
  >
  >  轮询的监听器实现类。为 consumer 提供支持，提供可选事务管理。
  >
  >  属性：
  >
  >  ```java
  >  public static final long DEFAULT_RECEIVE_TIMEOUT = 1000;
  >  
  >  private final MessageListenerContainerResourceFactory transactionalResourceFactory = new MessageListenerContainerResourceFactory();
  >  
  >  private boolean sessionTransactedCalled = false;
  >  
  >  private PlatformTransactionManager transactionManager;
  >  
  >  private DefaultTransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
  >  
  >  private long receiveTimeout = DEFAULT_RECEIVE_TIMEOUT;
  >  ```

* > public class DefaultMessageListenerContainer extends AbstractPollingMessageListenerContainer
  >
  > 定义了5个缓存等级
  >
  > 属性
  >
  > ```java
  > private Executor taskExecutor;
  > private BackOff backOff = new FixedBackOff(DEFAULT_RECOVERY_INTERVAL, Long.MAX_VALUE);
  > 
  > private int cacheLevel = CACHE_AUTO;
  > private int concurrentConsumers = 1;
  > private int maxConcurrentConsumers = 1;
  > private int maxMessagesPerTask = Integer.MIN_VALUE;
  > private int idleConsumerLimit = 1;
  > private int idleTaskExecutionLimit = 1;
  > 
  > private final Set<AsyncMessageListenerInvoker> scheduledInvokers = new HashSet<AsyncMessageListenerInvoker>();
  > 
  > private int activeInvokerCount = 0;
  > private int registeredWithDestination = 0;
  > 
  > private volatile boolean recovering = false;
  > private volatile boolean interrupted = false;
  > 
  > private Runnable stopCallback;
  > private Object currentRecoveryMarker = new Object();
  > private final Object recoveryMonitor = new Object();	
  > ```
  >
  >  
  >
  > 





