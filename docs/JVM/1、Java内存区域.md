## Java程序内存分布

![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210818154646.png)

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域有各自用途以及创建和销毁的时间，有的区域随虚拟机进程启动而一直存在，有的则依赖用户现成的启动和结束而建立和销毁。

- 程序计数器

  程序计数器是一块较小的内存空间，可以看作是当前线程执行字节码的`行号指示器`。

  在Java虚拟机的概念模型里，字节码解释器工作时通过改变程序计数器的值来选取下一条需要执行的字节码指令。

  由于Java虚拟机的多线程通过线程轮流切换、分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（一个内核）都只会执行一条线程中的指令。

  因此，为了线程切换后能正确恢复到正确的执行位置，每条线程都有一个独立的程序计数器，各线程间计数器互不影响，独立存储。

  如果线程执行的是一个Java方法，程序计数器记录的是正在执行的虚拟机字节码指令地址；如果正在执行的是本地（navicat）方法，这个计数器值应为空（Undefined）。.

  - 不会存在内存溢出

- Java虚拟机栈

  - Java虚拟机栈线程私有，生命周期与线程相同。

  - 虚拟机栈描述的是Java方法执行的线程内存模型：每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧用于存储`局部变量表、操作数栈、动态连接、方法出口`等信息。每个方法被调用至执行完毕的过程，就对应这一个栈帧在虚拟机栈中从入栈到出栈的过程。

    > 局部变量表存放了编译器可知的各种Java基本数据类型（boolean、byte、char。short、int、float、long、double）、对象引用和returnAddress类型（指向字节码指令的地址）。

  - 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；如果Java虚拟机栈容量可以动态扩展，当栈扩展时无法申请到足够的内存会抛出OutOfMemoryError。

  > #### 问题辨析
  >
  > - 垃圾回收是否涉及栈内存？
  >   - **不需要**。因为虚拟机栈中是由一个个栈帧组成的，在方法执行完毕后，对应的栈帧就会被弹出栈。所以无需通过垃圾回收机制去回收内存。
  > - 栈内存的分配越大越好吗？
  >   - 不是。因为**物理内存是一定的**，栈内存越大，可以支持更多的递归调用，但是可执行的线程数就会越少。
  > - 方法内的局部变量是否是线程安全的？
  >   - 如果方法内**局部变量没有逃离方法的作用范围**，则是**线程安全**的
  >   - 如果如果**局部变量引用了对象**，并**逃离了方法的作用范围**，则需要考虑线程安全问题

- 本地方法栈

  本地方法栈和虚拟机栈发挥作用类似，区别是虚拟机栈为Java方法服务，本地方法栈为虚拟机用到的本地方法服务。

- Java堆

  - 线程共享，虚拟机启动时创建、该内存唯一目的是存放对象实例，几乎所有对象实例在此分配内存。

  - 堆可与处于物理上不连续的内存空间，但在逻辑上应被视为连续的。

  - 堆可实现成固定大小，也可以扩展（通过-Xmx和-Xms设定）。如果堆中没有内存完成实例分配，且堆无法继续扩展，Java虚拟机将抛出OutOfMemoryError。

  > 堆内存诊断工具
  >
  > **jps**
  >
  > **jmap**
  >
  > **jconsole**
  >
  > **jvirsalvm**

- 方法区

  ![](https://gitee.com/bravehui/PicGoPictureBed/raw/master/img/markmap/20210818171025.png)

  

  - 线程共享，虚拟机启动时创建，用于存储被虚拟机加载`的类型信息、常量、静态变量、即时编译器编译后的代码缓存`等数据。
  - 可与处于物理上不连续的内存空间，但在逻辑上应被视为连续的。

  - 如果方法区无法满足新的内存分配需求时，将抛出OutOfMemoryError异常。
    - 1.8以前会导致**永久代**内存溢出
    - 1.8以后会导致**元空间**内存溢出

- 运行时常量池

  运行时常量池是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表，用于存放编译器生成的各种字面量和符号引用，该部分内容将在类加载后存放到方法区的运行时常量池中。

## 对象的创建

当Java虚拟机遇到一条字节码new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并检查该符号引用代表的类是否已被加载、解析和初始化过。如果没有，先执行相应的类加载过程。

当类加载检查通过后，将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定，为对象分配空间的任务实际上等同于把一块确定大小的内存块从堆中划分出来。

> 假设堆内存绝对规整，使用和未被使用的分届存放，中间放一个指针作为分界点指示器，那所分配内存就仅仅是把指针向空闲方向挪动与所分配对象大小相等的距离，这种分配方式成为“指针碰撞”。
>
> 如果堆中内存不规整，已被使用和未被使用的交错在一起，就无法只用指针碰撞，虚拟机必须维护一个列表，记录那些内存块可用，在分配的时候从表中找一块足够大的空间划分给对象，并更新表上记录，这种分配方式成为“空闲列表”。
>
> 选择哪一种分配方式由Java堆是否规整决定，Java堆是否规整又由所采用的垃圾收集器是否带有空间压缩整理的能力决定。
>
> 对象创建在虚拟机中是非常频繁的行为，即使仅仅修改一个指针所指向的位置，在并发情况下不是线程安全的。可能正在给对象A分配内存，指针还未修改，对象B又同时使用原来位置的指针分配内存。这种问题有两种可选方案：
>
> ​	1、对分配内存空间的动作进行同步处理，CAS+失败重试保证更新操作的原子性。
> ​	2、把内存分配的动作按照线程划分在不同空间中进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（TLAB），线程的分配只在该线程本地缓冲区中分配，只有本地缓冲区用完了，分配新的缓存区时才需要同步锁定。

内存分配完成后，虚拟机将分配到的内存空间都初始化为零值。这步保证了对象的实例字段在Java代码中可以不赋初值就直接使用，使程序能访问到这些字段的数据类型所对应的零值。

上面工作完成后，从虚拟机视角来看，一个新的对象产生了。但是从Java程序的视角来看，对象创建才刚刚开始，构造函数即Class文件中<init>()方法还没有执行，所有的字段都为默认零值，对象需要的其他资源和状态信息也还没有按照预定的意图构造好。一般来说（由字节 码流中new指令后面是否跟随invokespecial指令所决定，Java编译器会在遇到new关键字的地方同时生成 这两条字节码指令，但如果直接通过其他方式产生的则不一定如此），new指令之后会接着执行<init> ()方法，按照程序员的意愿对对象进行初始化，这样一个真正可用的对象才算完全被构造出来。