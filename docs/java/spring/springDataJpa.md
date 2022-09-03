## JPA 领域事件发布
source: [6. 基于Spring Data的领域事件发布]( https://blog.csdn.net/qq_33920904/article/details/105261617)
>领域事件发布是一个领域对象为了让其它对象知道自己已经处理完成某个操作时发出的一个通知，事件发布力求从代码层面让自身对象与外部对象解耦，并减少技术代码入侵。

### 一、 手动发布事件

```java
// 实体定义
@Entity
public class Department implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer departmentId;

    @Enumerated(EnumType.STRING)
    private State state;
}

// 事件定义
public class DepartmentEvent {
    private Department department;
    private State state;
    public DepartmentEvent(Department department) {
        this.department = department;
        state = department.getState();
    }
}

// 领域服务
@Service
public class ApplicationService {

    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    @Autowired
    private DepartmentRepository departmentRepository;

    @Transactional(rollbackFor = Exception.class)
    public void departmentAdd(Department department) {
        departmentRepository.save(department);
        // 事件发布
        applicationEventPublisher.publishEvent(new DepartmentEvent(department));
    }
}

```

使用`applicationEventPublisher.publishEvent`在领域服务处理完成后发布领域事件，此方法需要在业务代码中显式发布事件，并在领域服务里引入ApplicationEventPublisher类，但对领域服务本身有一定的入侵性，但灵活性较高。

### 二、 自动发布事件

```java
// 实体定义
@Entity
public class SaleOrder implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer orderId;
   
    @Enumerated(EnumType.STRING)
    private State state;

    // 返回类型定义
    @DomainEvents
    public List<Object> domainEvents(){
        return Stream.of(new SaleOrderEvent(this)).collect(Collectors.toList());
    }

    // 事件发布后callback
    @AfterDomainEventPublication
    void callback() {
        System.err.println("ok");
    }
}

// 事件定义
public class SaleOrderEvent {
    private SaleOrder saleOrder;
    private State state;
    public SaleOrderEvent(SaleOrder saleOrder) {
        this.saleOrder = saleOrder;
        state = saleOrder.getState();
    }
}

// 领域服务
@Service
public class ApplicationService {
    @Autowired
    private OrderRepository orderRepository;
    
    @Transactional(rollbackFor = Exception.class)
    public void saleOrderAdd(SaleOrder saleOrder) {
        orderRepository.save(saleOrder);
    }
}

```

使用`@DomainEvents`定义事件返回的类型，必须是一个集合，使用`@AfterDomainEventPublication`定义事件发布后的回调。  
此方法实事件类型定义在实体中，与领域服务完全解耦，没有入侵。系统会在**orderRepository.save(saleOrder)**后自动调用事件发布，另**delete**方法不会调用事件发布。

### 三、 事件监听

```java
@Component
public class ApplicationEventProcessor {

    @EventListener(condition = "#departmentEvent.getState().toString() == 'SUCCEED'")
    public void departmentCreated(DepartmentEvent departmentEvent) {
        System.err.println("dept-event1:" + departmentEvent);
    }

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT, condition = "#saleOrderEvent.getState().toString() == 'SUCCEED'")
    public void saleOrderCreated(SaleOrderEvent saleOrderEvent) {
        System.err.println("sale-event succeed1:" + saleOrderEvent);
    }

    @TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT, condition = "#saleOrderEvent.getState().toString() == 'SUCCEED'")
    public void saleOrderCreatedBefore(SaleOrderEvent saleOrderEvent) {
        System.err.println("sale-event succeed2:" + saleOrderEvent);
    }

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void saleOrderCreatedFailed(SaleOrderEvent saleOrderEvent) {
        System.out.println("sale-event failed:" + saleOrderEvent);
    }
}

```

#### 1\. 使用`@EventListener`监听事件

`@EventListener`没有事务支持，只要事件发出就可监控到

```java
@Transactional(rollbackFor = Exception.class)
public void departmentAdd(Department department) {
    departmentRepository.save(department);
    applicationEventPublisher.publishEvent(new DepartmentEvent(department));
    throw new RuntimeException("failed");
}
```

上述情况会造成事务失败回滚，但事件监控端已经执行，可能导致数据不一致的情况发生

#### 2\. 使用`@TransactionalEventListener`监听事件

-   `TransactionPhase.BEFORE_COMMIT` 事务提交前
-   `TransactionPhase.AFTER_COMMIT` 事务提交后
-   `TransactionPhase.AFTER_ROLLBACK` 事务回滚后
-   `TransactionPhase.AFTER_COMPLETION` 事务完成后

使用`TransactionPhase.AFTER_COMMIT`可在事务完成后，再执行事件监听方法，从而保证数据的一致性

#### 3\. `TransactionPhase.AFTER_ROLLBACK`回滚事务问题

```java
@Async
@TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK, condition = "#departmentEvent.getState().toString() == 'SUCCEED'")
public void departmentCreatedFailed(DepartmentEvent departmentEvent) {
    System.err.println("dept-event3:" + departmentEvent);
}
```

由于`@DomainEvents`作用在实体上的，只有刚orderRepository.save(saleOrder)执行成功后才会发送事件，故**AFTER\_ROLLBACK**方法只会在同一事务中其它语句执行失败或显式rollback时才会执行，如果save方法执行失败，将不会监听到回滚事件。

#### 4\. `@Async`异步事件监听

-   没有此注解事件监听方法与主方法为一个事务。
-   使用此注解将脱离原有事务，**BEFORE\_COMMIT**也无法拦截事务提交前时刻
-   此注解需要配合`@EnableAsync`一起使用
### 四、使用领域事件时的陷阱
处理领域事件看起来很简单，但有几个陷阱会导致 Spring 不发布事件、不调用观察者或不持久化观察者执行的更改。
-   无保存调用 = 无事件 
如果您在其存储库上调用save或saveAll方法，Spring Data JPA 仅发布实体的域事件。但是，如果您正在使用托管实体（通常是您在当前事务期间从数据库中获取的每个实体对象），您就不需要调用任何存储库方法来持久化您的更改。您只需要在实体对象上调用 setter 方法并更改属性的值。您的持久性提供程序，例如 Hibernate，会自动检测更改并保持不变。

-   无事务 = 无事务观察者  
如果您提交或回滚事务，Spring 只会调用我在第二个示例中向您展示的事务观察者。如果您的业务代码在没有活动事务的情况下发布事件，则 Spring 不会调用这些观察者。
`AFTER_COMMIT` / `AFTER_ROLLBACK` / `AFTER_COMPLETION = 需要新事务
如果您实现一个事务观察者并将其附加到事务阶段`AFTER_COMMIT`、`AFTER_ROLLBACK`或`AFTER_COMPLETION`，Spring 将在没有活动事务的情况下执行观察者。因此，您只能从数据库读取数据，但 Spring Data JPA 不会保留任何更改。
您可以通过使用`@Transactional(propagation = Propagation.REQUIRES_NEW)`注释您的观察者方法来避免该问题。这告诉 Spring Data JPA 在调用观察者之前启动一个新事务并在之后提交它。