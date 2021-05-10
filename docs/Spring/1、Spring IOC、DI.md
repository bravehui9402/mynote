## IOC

### 1、IOC是什么

IOC—Inversion of Control，即“控制反转”，是一种设计思想。Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

**谁控制谁，控制什么：**

传统Java SE程序设计，直接在对象内部通过new进行对象的创建，由程序去主动创建依赖对象；而IOC是有专门一个容器来创建这些对象，即由IOc容器来控制对象的创建而不再显式地使用new；谁控制谁？是IOC容器控制了对象的创建；控制什么？主要控制了外部资源获取和生命周期（不只是对象也包括文件等）。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210509181525.png)

**为何是反转，哪些方面反转了：** 

有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210509181600.png)

### 2、IoC能做什么

IOC是一种思想，一个重要的面向对象编程的法则，它能指导我们如何设计出松耦合、更优良的程序。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；

有了IOC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。

IOC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IOC/DI思想中，应用程序就变成被动的了，被动的等待IOC容器来创建并注入它所需要的资源了。IOC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”；即由IOC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。

### 3、DI

DI—Dependency Injection，即“依赖注入”：是组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

> 理解DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么”

谁依赖于谁：是应用程序依赖于IOC容器；

- 为什么需要依赖：应用程序需要IOC容器来提供对象需要的外部资源；
- 谁注入谁：很明显是IOC容器注入应用程序某个对象，应用程序依赖的对象；
- 注入了什么：就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。

IOC和DI有什么关系呢？其实它们是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IOC而言，“依赖注入”明确描述了“被注入对象依赖IOC容器配置依赖对象”。

### 4、IOC和DI的意义

在平时的Java应用开发中，我们要实现某一个功能或者说是完成某个业务逻辑时至少需要两个或以上的对象来协作完成，在没有使用Spring的时候，每个对象在需要使用他的合作对象或者依赖对象时，自己均要使用像new object() 这样的语法来将合作对象创建出来，这个合作对象是由自己主动创建出来的，创建合作对象的主动权在自己手上，自己需要哪个合作对象，就主动去创建，创建合作对象的主动权和创建时机是由自己把控的，而这样就会使得对象间的耦合度高了，A对象需要使用合作对象B来共同完成一件事，A要使用B，那么A就对B产生了依赖，也就是A和B之间存在一种耦合关系，并且是紧密耦合在一起。

而使用了Spring之后就不一样了，创建合作对象B的工作是由Spring来做的，Spring创建好B对象，然后存储到一个容器里面，当A对象需要使用B对象时，Spring就从存放对象的那个容器里面取出A要使用的那个B对象，然后交给A对象使用.，至于Spring是如何创建那个对象，以及什么时候创建好对象的，A对象不需要关心这些细节问题(你是什么时候生的，怎么生出来的我可不关心，能帮我干活就行)，A得到Spring给我们的对象B之后，两个人一起协作完成要完成的工作即可。

所以控制反转IOC(Inversion of Control)是说创建对象的控制权进行转移，以前创建对象的主动权和创建时机是由自己把控的，而现在这种权力转移到第三方，比如转移交给了IOC容器，它就是一个专门用来创建对象的工厂，你要什么对象，它就给你什么对象，有了 IOC容器，依赖关系就变了，原先的依赖关系就没了，它们都依赖IOC容器了，通过IOC容器来建立它们之间的关系。

## IOC过程

Spring 启动时读取应用程序提供的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表，然后根据这张注册表实例化Bean，装配好Bean之间的依赖关系，为上层应用提供准备就绪的运行环境。

Resource 定位：我们一般使用外部资源来描述 Bean 对象，所以 IOC 容器第一步就是需要定位 Resource 外部资源 。Resource 的定位其实就是 BeanDefinition 的资源定位，它是由 ResourceLoader 通过统一的 Resource 接口来完成的，这个 Resource 对各种形式的 BeanDefinition 的使用都提供了统一接口 。
载入：第二个过程就是 BeanDefinition 的载入 ,BeanDefinitionReader 读取 , 解析 Resource 定位的资源，也就是将用户定义好的 Bean 表示成 IOC 容器的内部数据结构也就是 BeanDefinition, 在 IOC 容器内部维护着一个 BeanDefinition Map 的数据结构，通过这样的数据结构， IOC 容器能够对 Bean 进行更好的管理 。 在配置文件中每一个都对应着一个 BeanDefinition 对象 。
注册：第三个过程则是注册，即向 IOC 容器注册这些 BeanDefinition ，这个过程是通过 BeanDefinitionRegistery 接口来实现的 。


![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/4bec75c5c268262272488001c7151788_watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzI0NDY5OA==,size_16,color_FFFFFF,t_70)



## Spring Bean 的生命周期

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/328d97e59f325652170e081b2de6091c_java0-1558500658.jpg)

1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
2. Bean实例化后对将Bean的引入和值注入到Bean的属性中
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。