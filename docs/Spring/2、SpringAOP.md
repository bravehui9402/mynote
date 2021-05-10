## AOP

AOP 其实跟 OOP (面向对象编程)一样，都是一种**编程思想**。它的全称是 `Aspect-oriented Programming`，也就是面向切面编程。

AOP的理念：就是将**分散在各个业务逻辑代码中相同的代码通过横向切割的方式**抽取到一个独立的模块中。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210509204212.png)

Spring AOP的底层原理就是**动态代理**！Spring AOP使用纯Java实现，它不需要专门的编译过程，也不需要特殊的类装载器，它在**运行期通过代理方式向目标类织入增强代码**。在Spring中可以无缝地将Spring AOP、IoC和AspectJ整合在一起。

在Java中动态代理有**两种**方式：

- JDK动态代理
- CGLib动态代理

### 代理模式

代理模式是**为其他对象提供一种代理以控制对这个对象的访问**。在某些情况下，一个对象不适合或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用。

在生活中，我们或多或少会听到一些词，例如“代理商”/“区域代理”等等。其实这些就是现实版的代理模式。假如说身边有一个**篮球鞋代理商**，其实我们都知道他的鞋子不是他自己生产的，他只是代替**实际生产鞋子的厂商**拿出去卖，而且我们也不知道实际上他的货源究竟是来自于哪里的，因为他的鞋子有可能是来自莆田，也可能来自于美国耐克。

从上面的例子，我们可以总结出一些特点：

1. **专人做专门的事情**，代理商对外负责销售，生厂商对内负责生产。
2. **代理商可以作为前台的身份**，统一对接所有需要买鞋子的人，屏蔽了实际生厂商。(毕竟代理不仅仅可以掌握一家厂商的货源，他可以代理多家厂商)

那么将上面的特点转成编程的话，我们可以说：

1. **专人做专门的事情**，对于面向对象来说职责更加分明了，提高代码的重用性，可读性。
2. **代理对象可以作为前台**，这里有利于屏蔽真实的调用代码，这样有利于封装性，同时也调高了代码的扩展性。

根据程序不同的运行时期这个维度上区分的话，代理模式的类型分为两种

- 静态代理

  所谓的**静态代理**，简单来说就是开发人员在开发的过程中直接硬编码，将代理对象与目标对象结合，然后再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

  ```java
  public interface Subject {
      public void sale();
  }
  
  public class RealSubject implements Subject {
      @Override
      public void sale() {
          System.out.println("卖东西");
      }
  }
  
  public class ProxySubject implements Subject {
  
      private Subject realSubject;
  
      public ProxySubject(Subject realSubject) {
          this.realSubject = realSubject;
      }
  
      @Override
      public void sale() {
          realSubject.sale();
      }
  }
  
  public class Client {
      public static void main(String[] args) {
          Subject realSubject = new RealSubject();
          ProxySubject proxySubject = new ProxySubject(realSubject);
          proxySubject.sale();
      }
  }
  
  ```

- 动态代理

  所谓的**动态代理**，与静态代理相反。**动态代理**是在程序编译期或运行期，会根据条件来生成代理类。

  在动态代理中主要分为两种技术：

  1. `Cglib`
  2. `Java Dynamic Proxy`

  #### `Cglib`

  动态生成一个要代理的子类，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截技术拦截所有的父类方法的调用，顺势织入横切逻辑，它比Java反射的jdk动态代理要快

  Cglib是一个强大的、高性能的代码生成包，它被广泛应用在许多AOP框架中，为他们提供方法的拦截

  ```java
     //依赖
  	<!-- cglib 目前最新版本是 3.3.0-->
      <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>3.3.0</version>
      </dependency>
  
      <!-- commons-lang3 -->
      <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.9</version>
      </dependency>
  	
          
      //RealObject 目标对象
      class RealObject {
          public void execute() {
              System.out.println("卖东西");
          }
  	}
      //ProxySubject 代理类
      class ProxySubject implements MethodInterceptor {
  
          @Override
          public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
              System.out.println("before execute");
              Object result = methodProxy.invokeSuper(o, objects);
              System.out.println("afer execute");
              return result;
          }
      }
  	//编写客户端
      public class CglibProxyTest {
          public static void main(String[] args) {
              Enhancer enhancer = new Enhancer();
              enhancer.setSuperclass(RealObject.class);
              enhancer.setCallback(new ProxySubject());
              RealObject executor = (RealObject) enhancer.create();
              executor.execute();
          }
      }
  	//输出
  	before execute
      卖东西
      afer execute
  ```

  #### `Java Dynamic Proxy`

  Java JDK 本身是具备了动态代理的。JDK中的动态代理是通过反射类Proxy以及InvocationHandler回调接口实现的，但是JDK中所有要进行动态代理的类必须要实现一个接口，也就是说只能对该类所实现接口中定义的方法进行代理，这在实际编程中有一定的局限性，而且使用反射的效率也不高

  ```java
  //接口 Subject
  public interface Subject {
  
      public void sell();
  
  }
  //编写目标对象 RealSubject
  public class RealSubject implements Subject {
      @Override
      public void sell() {
          System.out.println("卖东西");
      }
  }
  //编写处理器 ProxyInvocationHandler
  public class ProxyInvocationHandler implements InvocationHandler {
      
      Subject subject;
  
      public ProxyInvocationHandler(Subject subject) {
          this.subject = subject;
      }
  
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          System.out.println("before invoke ");
          subject.sell();
          System.out.println("after invoke ");
          return null;
      }
  }
  //编写客户端 Client
  public class Client {
      public static void main(String[] args) {
          Subject realSubject = new RealSubject();
          ProxyInvocationHandler myInvocationHandler =
                  new ProxyInvocationHandler(realSubject);
          Subject proxyClass = (Subject) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), new Class[]{Subject.class}, myInvocationHandler);
          proxyClass.sell();
      }
  }
  //结果:
  before invoke 
  卖东西
  after invoke 
  ```

  > Cglib和jdk动态代理的区别？

  1、Jdk动态代理：利用拦截器（必须实现InvocationHandler）加上反射机制生成一个代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理

  2、 Cglib动态代理：利用ASM框架，对代理对象类生成的class文件加载进来，通过修改其字节码生成子类来处理

  什么时候用cglib什么时候用jdk动态代理？

  1、目标对象生成了接口 默认用JDK动态代理

  2、如果目标对象使用了接口，可以强制使用cglib

  3、如果目标对象没有实现接口，必须采用cglib库，Spring会自动在JDK动态代理和cglib之间转换

  JDK动态代理和cglib字节码生成的区别？

  1、JDK动态代理只能对实现了接口的类生成代理，而不能针对类

  2、Cglib是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，并覆盖其中方法的增强，但是因为采用的是继承，所以该类或方法最好不要生成final，对于final类或方法，是无法继承的

   Cglib比JDK快？

  1、cglib底层是ASM字节码生成框架，但是字节码技术生成代理类，在JDL1.6之前比使用java反射的效率要高

  2、在jdk6之后逐步对JDK动态代理进行了优化，在调用次数比较少时效率高于cglib代理效率

  3、只有在大量调用的时候cglib的效率高，但是在1.8的时候JDK的效率已高于cglib

  4、Cglib不能对声明final的方法进行代理，因为cglib是动态生成代理对象，final关键字修饰的类不可变只能被引用不能被修改

  Spring如何选择是用JDK还是cglib？

  1、当bean实现接口时，会用JDK代理模式

  2、当bean没有实现接口，用cglib实现

  3、可以强制使用cglib（在spring配置中加入<aop:aspectj-autoproxy proxyt-target-class=”true”/>）

### AOP使用场景

1. 缓存
2. 异常处理，可以用来捕捉异常
3. 调试，性能优化，计算某个方法执行的时间进行精准优化
4. 持久化
5. 事务，这个我们经常使用的 @Transactional 注解
6. 日志
7. 记录追踪，可以记录执行了哪些功能(本质上和日志差不多)

### AOP 术语

**连接点**(Join point)：

- **能够被拦截的地方**：Spring AOP是基于动态代理的，所以是方法拦截的。每个成员方法都可以称之为连接点~

**切点**(Poincut)：

- **具体定位的连接点**：上面也说了，每个方法都可以称之为连接点，我们**具体定位到某一个方法就成为切点**。

**增强/通知**(Advice)：

- 表示添加到切点的一段逻辑代码，并定位连接点的方位信息。
  - 简单来说就定义了是干什么的，具体是在哪干
  - Spring AOP提供了5种Advice类型给我们：前置、后置、返回、异常、环绕给我们使用！

**织入**(Weaving)：

- 将`增强/通知`添加到目标类的具体连接点上的过程。

**引入/引介**(Introduction)：

- `引入/引介`允许我们**向现有的类添加新方法或属性**。是一种**特殊**的增强！

**切面**(Aspect)：

- 切面由切点和`增强/通知`组成，它既包括了横切逻辑的定义、也包括了连接点的定义。

> 通知/增强包含了需要用于多个应用对象的横切行为；连接点是程序执行过程中能够应用通知的所有点；切点定义了通知/增强被应用的具体位置。其中关键的是切点定义了哪些连接点会得到通知/增强。

### Spring对AOP的支持

Spring提供了3种类型的AOP支持：

- 基于代理的经典SpringAOP

  - 需要实现接口，手动创建代理

- 纯POJO切面

  - 使用XML配置，aop命名空间

- ```
  @AspectJ
  ```

  注解驱动的切面

  - 使用注解的方式，这是最简洁和最方便的！