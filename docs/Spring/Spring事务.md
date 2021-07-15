## 【说明】Spring事务管理五大属性

### 事务隔离级别

**`TransactionDefinition.ISOLATION_DEFAULT`** :使用后端数据库默认的隔离级别，MySQL 默认采用的 `REPEATABLE_READ` 隔离级别 Oracle 默认采用的 `READ_COMMITTED` 隔离级别.

**`TransactionDefinition.ISOLATION_READ_UNCOMMITTED`** :最低的隔离级别，使用这个隔离级别很少，因为它允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**

**`TransactionDefinition.ISOLATION_READ_COMMITTED`** : 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**

**`TransactionDefinition.ISOLATION_REPEATABLE_READ`** : 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**

**`TransactionDefinition.ISOLATION_SERIALIZABLE`** : 最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 事务传播行为

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715152938.png)

> 事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进另一个方法的时事务如何传播。

```java
 public void methodA(){
    methodB();
    //doSomething
 }

 @Transaction(Propagation=XXX)
 public void methodB(){
    //doSomething
 }
```

代码中`methodA()`方法嵌套调用了`methodB()`方法，`methodB()`的事务传播行为由`@Transaction(Propagation=XXX)`设置决定。这里需要注意的是`methodA()`并没有开启事务，某一个事务传播行为修饰的方法并不是必须要在开启事务的外围方法中调用。

#### PROPAGATION_REQUIRED

**User1Service 方法：**

```
@Service
public class User1ServiceImpl implements User1Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequired(User1 user){
        user1Mapper.insert(user);
    }
}
```

**User2Service 方法：**

```
@Service
public class User2ServiceImpl implements User2Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequired(User2 user){
        user2Mapper.insert(user);
    }
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequiredException(User2 user){
        user2Mapper.insert(user);
        throw new RuntimeException();
    }
}
```

**场景一**

此场景外围方法没有开启事务。

**验证方法 1：**

```
    @Override
    public void notransaction_exception_required_required(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequired(user2);

        throw new RuntimeException();
    }
```

**验证方法 2：**

```
    @Override
    public void notransaction_required_required_exception(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiredException(user2);
    }
```

分别执行验证方法，结果：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715154755.png)

**结论：通过这两个方法我们证明了在外围方法未开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

**场景二**

外围方法开启事务，这个是使用率比较高的场景。

**验证方法 1：**

```
   @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_exception_required_required(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequired(user2);

        throw new RuntimeException();
    }
```

**验证方法 2：**

```
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_required_required_exception(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiredException(user2);
    }
```

**验证方法 3：**

```
    @Transactional
    @Override
    public void transaction_required_required_exception_try(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        try {
            user2Service.addRequiredException(user2);
        } catch (Exception e) {
            System.out.println("方法回滚");
        }
    }
```

分别执行验证方法，结果：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715154928.png)

**结论：以上试验结果我们证明在外围方法开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会加入到外围方法的事务中，所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。**

#### PROPAGATION_NESTED

为 User1Service 和 User2Service 相应方法加上`Propagation.NESTED`属性。**User1Service 方法：**

```
@Service
public class User1ServiceImpl implements User1Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addNested(User1 user){
        user1Mapper.insert(user);
    }
}
```

**User2Service 方法：**

```
@Service
public class User2ServiceImpl implements User2Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addNested(User2 user){
        user2Mapper.insert(user);
    }

    @Override
    @Transactional(propagation = Propagation.NESTED)
    public void addNestedException(User2 user){
        user2Mapper.insert(user);
        throw new RuntimeException();
    }
}
```

**场景一**

此场景外围方法没有开启事务。

**验证方法 1：**

```
    @Override
    public void notransaction_exception_nested_nested(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addNested(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addNested(user2);
        throw new RuntimeException();
    }
```

**验证方法 2：**

```
    @Override
    public void notransaction_nested_nested_exception(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addNested(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addNestedException(user2);
    }
```

分别执行验证方法，结果：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715155050.png)

**结论：通过这两个方法我们证明了在外围方法未开启事务的情况下`Propagation.NESTED`和`Propagation.REQUIRED`作用相同，修饰的内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰。**

**场景二**

外围方法开启事务。

**验证方法 1：**

```
    @Transactional
    @Override
    public void transaction_exception_nested_nested(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addNested(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addNested(user2);
        throw new RuntimeException();
    }
```

**验证方法 2：**

```
    @Transactional
    @Override
    public void transaction_nested_nested_exception(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addNested(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addNestedException(user2);
    }
```

**验证方法 3：**

```
    @Transactional
    @Override
    public void transaction_nested_nested_exception_try(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addNested(user1);

        User2 user2=new User2();
        user2.setName("李四");
        try {
            user2Service.addNestedException(user2);
        } catch (Exception e) {
            System.out.println("方法回滚");
        }
    }
```

分别执行验证方法，结果：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715155127.png)

**结论：以上试验结果我们证明在外围方法开启事务的情况下`Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外围主事务回滚，子事务一定回滚，而内部子事务可以单独回滚而不影响外围主事务和其他子事务**

#### REQUIRED,REQUIRES_NEW,NESTED 异同

> **NESTED 和 REQUIRED 修饰的内部方法都属于外围方法事务，如果外围方法抛出异常，这两种方法的事务都会被回滚。但是 REQUIRED 是加入外围方法事务，所以和外围事务同属于一个事务，一旦 REQUIRED 事务抛出异常被回滚，外围方法事务也将被回滚。而 NESTED 是外围方法的子事务，有单独的保存点，所以 NESTED 方法抛出异常被回滚，不会影响到外围方法的事务。**

> **NESTED 和 REQUIRES_NEW 都可以做到内部方法事务回滚而不影响外围方法事务。但是因为 NESTED 是嵌套事务，所以外围方法回滚之后，作为外围方法事务的子事务也会被回滚。而 REQUIRES_NEW 是通过开启新的事务实现的，内部事务和外围事务是两个事务，外围事务回滚不会影响内部事务。**

#### PROPAGATION_REQUIRES_NEW

为 User1Service 和 User2Service 相应方法加上`Propagation.REQUIRES_NEW`属性。**User1Service 方法：**

```
@Service
public class User1ServiceImpl implements User1Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiresNew(User1 user){
        user1Mapper.insert(user);
    }
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void addRequired(User1 user){
        user1Mapper.insert(user);
    }
}
```

**User2Service 方法：**

```
@Service
public class User2ServiceImpl implements User2Service {
    //省略其他...
    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiresNew(User2 user){
        user2Mapper.insert(user);
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void addRequiresNewException(User2 user){
        user2Mapper.insert(user);
        throw new RuntimeException();
    }
}
```

**场景一**

外围方法没有开启事务。

**验证方法 1：**

```
    @Override
    public void notransaction_exception_requiresNew_requiresNew(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequiresNew(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiresNew(user2);
        throw new RuntimeException();

    }
```

**验证方法 2：**

```
    @Override
    public void notransaction_requiresNew_requiresNew_exception(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequiresNew(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiresNewException(user2);
    }
```

分别执行验证方法，结果：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715155715.png)

**结论：通过这两个方法我们证明了在外围方法未开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**

**场景二**

外围方法开启事务。

**验证方法 1：**

```
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_exception_required_requiresNew_requiresNew(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiresNew(user2);

        User2 user3=new User2();
        user3.setName("王五");
        user2Service.addRequiresNew(user3);
        throw new RuntimeException();
    }
```

**验证方法 2：**

```
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_required_requiresNew_requiresNew_exception(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiresNew(user2);

        User2 user3=new User2();
        user3.setName("王五");
        user2Service.addRequiresNewException(user3);
    }
```

**验证方法 3：**

```
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public void transaction_required_requiresNew_requiresNew_exception_try(){
        User1 user1=new User1();
        user1.setName("张三");
        user1Service.addRequired(user1);

        User2 user2=new User2();
        user2.setName("李四");
        user2Service.addRequiresNew(user2);
        User2 user3=new User2();
        user3.setName("王五");
        try {
            user2Service.addRequiresNewException(user3);
        } catch (Exception e) {
            System.out.println("回滚");
        }
    }
```

分别执行验证方法，结果：

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715155812.png)

**结论：在外围方法开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法依然会单独开启独立事务，且与外部方法事务也独立，内部方法之间、内部方法和外部方法事务均相互独立，互不干扰。**

#### 模拟用例

假设我们有一个注册的方法，方法中调用添加积分的方法，如果我们希望添加积分不会影响注册流程（即添加积分执行失败回滚不能使注册方法也回滚），我们会这样写：

```
   @Service
   public class UserServiceImpl implements UserService {

        @Transactional
        public void register(User user){
            try {
                membershipPointService.addPoint(Point point);
            } catch (Exception e) {
               //省略...
            }
            //省略...
        }
        //省略...
   }
```

我们还规定注册失败要影响`addPoint()`方法（注册方法回滚添加积分方法也需要回滚），那么`addPoint()`方法就需要这样实现：

```
   @Service
   public class MembershipPointServiceImpl implements MembershipPointService{

        @Transactional(propagation = Propagation.NESTED)
        public void addPoint(Point point){

            try {
                recordService.addRecord(Record record);
            } catch (Exception e) {
               //省略...
            }
            //省略...
        }
        //省略...
   }
```

我们注意到了在`addPoint()`中还调用了`addRecord()`方法，这个方法用来记录日志。他的实现如下：

```java
  @Service
   public class RecordServiceImpl implements RecordService{

        @Transactional(propagation = Propagation.NOT_SUPPORTED)
        public void addRecord(Record record){


            //省略...
        }
        //省略...
   }
```

我们注意到`addRecord()`方法中`propagation = Propagation.NOT_SUPPORTED`，因为对于日志无所谓精确，可以多一条也可以少一条，所以`addRecord()`方法本身和外围`addPoint()`方法抛出异常都不会使`addRecord()`方法回滚，并且`addRecord()`方法抛出异常也不会影响外围`addPoint()`方法的执行。



### 事务超时属性

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 `TransactionDefinition` 中以 int 的值来表示超时时间，其单位是秒，默认值为-1。

### 事务只读属性

> 从这一点设置的时间点开始（时间点a）到这个事务结束的过程中，其他事务所提交的数据，该事务将看不见！（查询中不会出现别人在时间点a之后提交的数据）

应用场合：

如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持SQL执行期间的读一致性； 
如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询SQL必须保证整体的读一致性，否则，在前条SQL查询之后，后条SQL查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。
【注意是一次执行多次查询来统计某些信息，这时为了保证数据整体的一致性，要用只读事务】

**在将事务设置成只读后，相当于将数据库设置成只读数据库，此时若要进行写的操作，会出现错误**

如果不加`Transactional`，每条`sql`会开启一个单独的事务，中间被其它事务改了数据，都会实时读取到最新值。

### 事务回滚规则

这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常（RuntimeException 的子类）时才会回滚，Error 也会导致事务回滚，但是，在遇到检查型（Checked）异常时不会回滚。

如果你想要回滚你定义的特定的异常类型的话，可以这样：

```
@Transactional(rollbackFor= MyException.class)
```

## Spring事务配置

**Spring 事务管理**分为编程式和声明式的两种方式。编程式事务指的是通过编码方式实现事务；声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。声明式事务管理使业务代码逻辑不受污染, 因此在实际使用中声明式事务用的比较多。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210715160927.png)

**声明式事务有两种方式**，一种是在配置文件（xml）中做相关的事务规则声明，另一种是基于 `@Transactional` 注解的方式。

spring的事务管理是通过Aop的方式来实现；

声明式事务是spring对事务管理的最常用的方式，因为这种方式对代码的影响最小，因此也就符合非侵入式的轻量级的容器的概念；

**需要明确几点：**

　　1、默认配置下 Spring 只会回滚运行时、未检查异常（继承自 RuntimeException 的异常）或者 Error。
　　2、@Transactional 注解只能应用到 public 方法才有效。

　　3、@Transactional 注解可以被应用于接口定义和接口方法、类定义和类的 public 方法上。然而仅仅 @Transactional 注解的出现不足以开启事务行为，它仅仅是一种元数据，能够被可以识别 @Transactional 注解和上述的配置适当的具有事务行为的beans所使用。其实是 <tx:annotation-driven/>元素的出现开启了事务行为。

　　4、注解不可以继承，建议在具体的类（或类的方法）上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上。当然可以在接口上使用 @Transactional 注解，但是这将只有当你设置了基于接口的代理时它才生效。

### xml配置

```xml
<bean id="serviceAspect" class="com.myspring.app.aop.MyAdvice"/> //切面代码

<!--  配置事务传播特性 -->
<tx:advice id="TestAdvice" transaction-manager="transactionManager">
  <tx:attributes>
    <tx:methodname="save*" propagation="REQUIRED"/>
    <tx:methodname="del*"  propagation="REQUIRED"/>
    <tx:methodname="update*" propagation="REQUIRED"/>
    <tx:methodname="add*"  propagation="REQUIRED"/>
    <tx:methodname="find*" propagation="REQUIRED"/>
    <tx:methodname="get*"  propagation="REQUIRED"/>
    <tx:methodname="apply*" propagation="REQUIRED"/>
  </tx:attributes>
</tx:advice>

<!--  配置参与事务的类 -->
<aop:config>
  <!--定义向导，引起切点和通知-->
  <aop:advisor pointcut-ref="allTestServiceMethod" advice-ref="TestAdvice"/>
</aop:config>
```

【说明】

（1）<tx> 命名空间用于配置声明式事务。得益于 <aop> 命名空间的切点表达式支持，声明式事务也变得更加强大。

<tx:advice/>标签，该标签会创建一个事务处理通知。结合<aop> 命名空间，就会把begin()、commit()等方法插入到事务的前后，配置变得更加简单和灵活。

> 不配置tx，它是不会开启事务的。我们调用增删查改，代码里面没有transition.begin()，就是aop帮你调用了。aop就是在你配置的那些方法，帮他们在被调用前，和被调用后，执行相应的逻辑。我们那些执行事务的开启执行的方法，比如commit都是通过aop的方式切入的。它可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。

（2） tx:attribute标签所配置的是作为事务的方法的命名类型。如<tx:method name="save*" propagation="REQUIRED"/>，其中*为通配符，即代表以save为开头的所有方法，即表示符合此命名规则的方法作为一个事务。

### 注解式事务

@Transactional采用注解式事务，所有标记为这个注解的并且能被spring扫描到的方法都会根据@Transactional的配置来使用事务。
  @Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

```java
//必须的（默认的）
@Transactional(propagation = Propagation.REQUIRED) 
//强制的（必须有transaction）
@Transactional(propagation = Propagation.MANDATORY)
//内嵌的**transaction**
@Transactional(propagation = Propagation.NESTED)
//绝不能有transaction
@Transactional(propagation = Propagation.NEVER)
//方法要执行，遇到的transaction挂起。
@Transactional(propagation = Propagation.NOT_SUPPORTED)
//方法要执行，遇到的transaction挂起,创建新的transaction。
@Transactional(propagation = Propagation.REQUIRES_NEW)
//有没有无所谓
@Transactional(propagation = Propagation.SUPPORTS)
```

如果要使用@Transactional，首先要在配置文件中配置如下事务管理器：

```xml
<!-- 定义一个数据源 -->
<bean id="dataSource" class="org.apache.tomcat.jdbc.pool.DataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/spring_test" />
        <property name="username" value="root" />
        <property name="password" value="root" />
</bean>

<!-- 配置事务管理器 -->
<bean id="txManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
        p:dataSource-ref="dataSource">
</bean>

<!-- enables scanning for @Transactional annotations -->
<tx:annotation-driven transaction-manager="txManager" />
```

然后我们就可以代码中使用@Transactional：

```csharp
public class userService{
    @Transactional(readonly=false)
    public void addUser(){..}

    @Transactional(readonly=true)
    public void getUserById(int id){..}
}
```

**@Transactional标注的位置**
  @Transactional通常应该写在service层，因为service层是业务处理类。在service加@Transactional，声明这个service所有方法需要事务管理，每个业务方法开始时都会打开一个事务。



