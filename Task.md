题目来源：
https://blog.csdn.net/weixin_43979260/article/details/114012818

Java基础




    1. String 和StringBuffer和 StringBuilder的区别？
               String 是字符串常量 
tringBuffer 是字符串变量（线程安全）
StringBuilder 是字符串变量（非线程安全）

String对象是一个不可被修改的字符串（内部用于存储的形式为private final char value[];），所以String是字符串常量。
StringBuffer和StringBuilder是“可变的字符串”（内部用于存储的形式为private char value[];），所以是字符串变量
而StringBuffer是线程安全的，StringBuilder是非线程安全的，因为StringBuffer的所有公开方法都是有synchronized修饰的同步方法

对String对象的拼接操作实则是使用StringBuffer的连续append操作，但是！自JDK5及以后的版本中对“锁机制”进行了优化，对字符串的加法操作会转化为StringBuilder的连续append
虚拟机对StringBuffer.append()的操作对象进行逃逸分析，如果该操作对象不会逃逸出当前方法，则会对append操作进行“锁消除”处理，再解释运行时这里仍会加锁，但经过编译之后会忽略所有同步锁直接执行

参考：
https://blog.csdn.net/zhangjg_blog/article/details/18319521
https://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247488178&idx=1&sn=e562f1904cd213b54fd6e1904d493b0f&chksm=eb539784dc241e92cd3a91639b2e7eb3cb91f59f1d1ab75ffd4a521c16d965772dfa858bb19f&scene=21#wechat_redirect
《深入理解Java虚拟机第三版》


    2. sleep() 区间wait()区间有什么区别？
sleep是Thread对象的方法，wait是Object提供的方法
sleep使线程待指定时间，直到时间结束后恢复正常运行，不会释放任何资源（源码注释：The thread does not lose ownership of any monitors），如果该线程在持有锁的同步块中，其他线程无法抢占
wait的使用必须是在同步方法中，调用后将释放当前线程持有的锁资源，等待其他线程调用notify()或者notifyAll()方法将其唤醒

    3. Object 中有哪些方法？其中clone()，怎么实现一个对象的克隆,Java如何实现深度克隆？

1.object、getClass、wait、clone、notify、notifyAll、toString、hashCode
2.实现一个对象的克隆要使该对象实现Cloneable空接口，将Object中protected类型的clone方法重写为public类型。
3.深克隆是相对于“浅拷贝”只能拷贝对象中基本数据类型的参数而说的，其希望完全克隆对象中包含的引用数据类型，
3.1实现方式为在引用数据类型也实现Cloneable空接口并重写clone方法，在外层对象中调用
3.2或者使用序列化/反序列（实现Serializable接口）化手段进行深度克隆，可以借助Gson等序列化插件工具



ThreadLocal 相关
https://www.cnblogs.com/dreamroute/p/5034726.html
ThreadLocal 是 线程局部变量，其是“线程封闭”的一种实现手段，这个类能使线程中的某个值与保存值的对象关联起来。




    4. ThreadLocal作用和实现方式 ?
1.作用：
1.1为每个使用ThreadLocal所保存变量的线程都存一份该变量的副本，防止对单例变量和全局变量进行共享。
1.2可以用来在线程中隐式传递参数。如某个线程中有方法的多级方法调用，而并非所有层级都会使用写带来的参数，此时可将参数存储于ThreadLocal中。

参考：
https://www.zhihu.com/question/341005993
http://www.majiang.life/blog/the-smart-way-of-passing-parameter-by-threadlocal/
《Java 并发编程实战》 Brian Goetz



    ThreadLocal会不会发生内存泄漏?
会存在内存泄漏，因为





    ThreadLocal为什么使用弱引用?

    5. InheritableThreadLocal作用和实现方式 ？
InheritableThreadLocal用于父子线程之间的值传递，创建子线程时会将父线程InheritableThreadLocal的值给子线程。

    6. InheritableThreadLocal所带来的问题？
1.InheritableThreadLocal在传递给子线程时，对于引用数据类型是传递引用，子线程对引用数据的修改会影响到父线程。
2.使用线程池时，线程执行完毕所在线程可能会保存复用，因此不再重新创建线程，因为InheritableThreadLocal值传递是在创建线程阶段，所以可能对导致传值失败


    7. 如何解决线程池异步值传递问题 (transmittable-thread-local)?
阿里开源的transmittable-thread-local可以很好的解决 在线程池情况下,父子线程值传递问题;TransmittableThreadLocal继承了InheritableThreadLocal, 简单的原理就是TTL 中的holder持有的是当前线程内的所有本地变量,被包装的run方法执行异步任务之前，会使用replay进行设置父线程里的本地变量给当前子线程，任务执行完毕，会调用restore恢复该子线程原生的本地变量
即：执行前同步更新一下，执行后恢复一下



HashMap ConcurrentHashMap相关



    9. HashMap为什么线程不安全
JDK1.7在多线程插入进行扩容操作时会有循环引用和数据丢失的风险。
JDK1.8之后多线程做PUT操作会有数据覆盖的风险
因为其操作并非原子操作

    10. HashMap在jdk7和8中的区别
    11. HashMap 为啥将链表改成红黑树？
JDK7中采用数组+单链表的结构
JDK8中采用数组+链表+红黑树的数据结构，当链表长度达到8时，转为红黑树存储，降低查询的时间复杂度O(logN)提高了查询效率


    12. ConcurrentHashMap在jdk7和8中的区别?
ConcurrentHashMap采用锁分段机制，细化了锁粒度，允许多线程并发对集合进行读写

    提到synchronized时候*,顺便说一下javaSE1.6对锁的优化？
*synchronized 两大点：
1.synchronized修饰的同步块对同一个线程是可重入的（同一个线程反复进入同步块不会被锁死）
2.synchronized修饰的同步块 在持有锁的线程完全释放锁之前，其他线程只能阻塞等待。


        轻量级锁
        TODO 是锁优化的一个方案

        重量级锁
           Java线程的阻塞以及唤醒，都是依靠操作系统来完成的,这些操作将涉及系统调用，需要从操作系统 的用户态切换至内核态，其开销非常之大。
如synchronize
其他优化
1.自旋（1.4就有了）与自适应旋转
减少线程切换带来的性能开销，对“简短”的线程有好处，自旋（实际上就是循环调用CAS算法的重试）期间不释放资源，所以不能长期自旋，1.6之前默认自旋10次，1.6之后引入了“自适应自旋”（参考锁拥有者状态和上一个线程自旋时间）
2.锁消除
经过逃逸分析算法确认在堆上的所有数据都不会逃逸出被其他线程访问。那就当做栈上数据（栈上数据是线程独立的）对待，认为是线程私有的，锁可以去除
3.锁粗化
如果一组操作反复对同一个对象加解锁（如StringBuffer的连续append）那么虚拟机会把枷锁操作扩展（粗化）到整个操作序列的外部 （说白了就是一起打包加锁，但前提是对同一个操作对象反复加解锁）
4.轻量级锁
TODO
5.偏向锁
TODO

    ReentrantLock和synchronized的区别？
          相同点对同一条线程都是可重入的

        Synchronized
synchronize是重量级锁，其加解锁操作直接映射为操作系统对线程的操作，会导致从“用户态”到“内核态”之间的切换，从而造成资源浪费

        ReentrantLock
ReentrantLock是JDK5之后基于J.U.C包提供的Lock接口的实现*，相对于synchronize多了三个功能：
1.等待可中断：长期等待时可以选择放弃
2.公平锁：按照申请锁的时间顺序获得锁
3.锁定多个条件：synchronize 的wait、配合notify或notifyAll可以实现一个隐含条件，要想实现多个就必须反复上锁，ReentrantLock则只需要多次调用 new Condition()方法即可
synchronize是系统控制，出现异常也可释放锁，而ReentrantLock需要在finally中释放锁
*自从jdk5以后Java类库中提供了java.util.concurrent包其中的java.util.concurrent.locks.Lock接口，改为在类库层面实现同步，重入锁（ReentrantLock）是Lock接口最常见的实现。






    13. 为什么重写equals时候被要求重写hashCode（）？
1.Object原生equals是采用==比较地址进行判断，重写equals是为了根据对象属性判断量对象属性是否相等
2..重写hashCode是因为集合存储中是先根据hash code判断对象是否存在，所以如果重写后的equals认为两对象属性相等，则应使用equals中作为判定条件的属性计算hash code

    14. 什么时候回发生内存泄露？让你写一段内存泄露的代码你会怎么写？

1.内存泄漏指的是：一个不再会被使用到的对象没有被回收导致的内存占用
2.比如一个短生命周期的方法，使用了类对象中的属性，方法执行完毕，则对象属性理应被回收，而根绝GC垃圾回收算法，只有当该类无对象引用时才会清除，就造成了内存泄漏。
3.解决，手动将该对象置空 Object=null

Vector v = new Vector(10);
for (int i = 0; i < 100; i++) {
Object o = new Object();
v.add(o);
o = null;
}
这里内部 Object o 虽然置为null 但外部Vector v仍然引用o 所以o不可被回收

Java多线程内存模型

线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory）

    Java 内存模型中的 happen-before 是什么？
Java内存模型采用共享主内存的形式，在并发过程中围绕“原子性、可见性、有序性”三原则，而happen-before 就是对其有序性的顺序约束，如：将变量修改后同步到主内存，在变量读取前从主内存刷新变量，依靠此种happen-before的顺序操作来实现共享变量的可见性
参考：
《深入理解Java虚拟机》

    简单聊聊volatile 的特性？以及内存语义
volatile有两项特性：
1.保证此变量对所有线程的可见性。即对volatile的写操作都能立刻反馈到其他线程中
2..保证此变量多线程下的读写原子性。读写原子性利用内存屏障禁止指令重排序优化
（但不能保证并发下的安全性，因为Java运算符不是原子操作，运算符最终会形成多条机器代码执行）


Java内存空间模型






GC垃圾回收



    垃圾回收主要是针对 内存区的哪些区域？
堆内存：主要针对堆内存的新生代，大部分对象都是朝生夕灭的，针对新生代的回收性价比最高
方法区（次要）：方法区保存静态变量、常量、类型信息，在大量使用“反射、动态代理、CGLib等字节码框架”，这种频繁自定义类加载器的场景中有必要进行回收，《Java虚拟机规范》提到可以不针对该区域进行垃圾回收。


    垃圾检查有哪些算法？

1.引用计数算法
2.可达性分析算法，以GCRoot根节点开始，以图论中的指向性分析可达性。“不可达”的将被清理

    垃圾回收方法有哪些？
基于分代收集理论的算法有三种
1.标记->清除
2.标记->整理
3.标记->复制


    什么时候会触发Full GC
当垃圾回收获得的可用空间不足以分配新建对象时

    **GC机制**简要说明一下，不同区使用的算法。
新生代收集器：
Serial（客户端默认新生代）：复制算法，单线程回收,回收时暂停所有用户线程
ParNew（Serial的多线程）：复制算法，多线程并行回收，但回收时也会暂停所有用户线程
Parallel Scavenge（吞吐量）：复制算法，多线程并行回收，关注吞吐量（吞吐量=运行用户代码时间/处理器总消耗时间）,回收时暂停所有用户线程可控制停顿时间

老年代收集器：
Serial Old：标记整理，单线程回收，回收时暂停所有用户线程
Parallel Old：标记整理，多线程并行回收，回收时暂停所有用户线程吞吐量优先
CMS（短停顿）：标记清除（初始标记GCRoot直接引用暂停所有用户线程，并发标记引用链，重新标记做修正暂停所有用户线程，并发清除）

全区收集器：
G1：面向服务端,基于Region的内存划分，收集时间可控制，按受益大小回收。每个Region来看是“标记-复制”，宏观上是“标记-整理”


    两个对象循环引用会不会被被GC？
要看GCRoot是否对其中的对象“可达”

    哪些可以算作根节点？
栈中引用的对象（虚拟机栈、本地方法栈）
静态属性、常量引用的对象
被同步锁持有的对象
虚拟机内部引用的对象


    垃圾收集器 G1有什么样的特性了解吗？ CMS呢？
G1是可预测停顿时间的全堆垃圾回收器
CMS是以获取最短停顿时间为目标的老年代收集器

    CMS收集器和G1收集器的区别
区别一： 使用范围不一样
CMS收集器是老年代的收集器，可以配合新生代的Serial和ParNew收集器一起使用
G1收集器收集范围是老年代和新生代。不需要结合其他收集器使用
区别二： STW的时间
CMS收集器以最小的停顿时间为目标的收集器。
G1收集器可预测垃圾回收的停顿时间（建立可预测的停顿时间模型）
区别三： 垃圾碎片
CMS收集器是使用“标记-清除”算法进行的垃圾回收，容易产生内存碎片
G1收集器使用的是“标记-整理”算法，每个Region来看是“标记-复制”，进行了空间整合，降低了内存空间碎片。



Jvm相关


    Jvm内存结构简要说一些，栈里面一般存储了什么？

    Java内存模型简要描述一下?




    **类加载机制**简要描述一下?
虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的java类型。类加载和连接的过程都是在运行期间完成的。

        类的加载方式
类加载实际是对class文件的加载，class文件的存在形式包括但不限于，网络、数据库、内存、磁盘、动态生成、jar/zip包

    类加载的过程
类加载的生命周期：
加载（Loading）–>验证（Verification）–>准备（Preparation）–>解析（Resolution）–>初始化（Initialization）–>使用（Using）–>卸载（Unloading）
加载（读取class）–>验证（校验是否可用）–>准备（分配内存）–>解析（引用指向实体）–>初始化（构造方法）–>使用（调用）–>卸载（卸载）


    JVM三种预定义类型类加载器


        双亲委派加载

        由不同的类加载器加载的指定类型还是相同的类型吗
不是，JAVA中的类使用完全限定名标识，JVM中使用完全限定名+类加载器（ClassLoader）标识
equals() / isInstance()/ instanceof / isAssignableFrom的比较只有在同一个类加载器下才有意义

        在代码中直接调用Class.forName（String name）方法，到底会触发那个类加载器进行类加载行为？
Class.forName(String name)默认会使用调用类的类加载器来进行类加载

        在编写自定义类加载器时，如果没有设定父加载器，那么父加载器是?
AppClassLoader

        编写自定义类加载器时，一般有哪些注意点？
一般尽量不要覆写已有的loadClass（…）方法中的委派逻辑; 这样做极有可能引起系统默认的类加载器不能正常工作

        如何在运行时判断系统类加载器能加载哪些路径下的类？
JDK9之前：获取到类加载器，调用ClassLoader.getURLs()
JDK9之后：引入java模块化系统，双亲委派关系发生变化。先判断要加载的类能否归属到一个模块，如果可以则委派给相应模块的类加载器加载。

    在Java的反射中，Class.forName和ClassLoader的区别
Class.forName()也是使用ClassLoader实现类加载，里面调用forName0（）方法，initialize参数默认为true，即类加载时对其进行初始化操作（执行静态代码块、静态参数的赋值），而ClassLoader只是加载类到内存中
https://www.cnblogs.com/jimoer/p/9185662.html


    Java 类加载机制及常见异常

        ClassNotFoundException 
发生在加载阶段，原因是没有找到要加载的类，导致原因可能为Class.forName()参数写错，没有相应的jar包导入

     NoClassDefFoundError 通常在链接阶段
发生在 链接 阶段；类生命周期有7个阶段：
加载-验证-准备-解析-初始化-使用-卸载
其中 “验证-准备-解析”统称为链接阶段

    Exception和Error的区别
Exception是可捕获的异常，Error是不可预料的错误无法通过异常处理来进行逻辑处理

    平时有没有遇到一些栈溢出或者内存溢出，内存泄露的问题吗？如何去分析这个问题？
栈：
如果线程请求的栈深度大于虚拟机所允许的最大深度时：StackOverFlowError
如果虚拟机内存的栈内存允许动态扩展，当栈容量无法申请到足够内存时：OutOfMemoryError
堆：
当堆内存无法满足对象的内存分配需要时：OutOfMemoryError
先确定错误类型，再确定报错区域（堆/栈/方法区/运行时常量池）
栈：查看是否有死循环调用
堆：使用内存快照分析工具，确定问题原因



    如果内存猛增，怎么去排查？

使用分析工具jstack分析每个线程的内存占用

多线程



    为什么《阿里巴巴Java开发手册》强制不允许使用Executor创建线程池
Executors创建出来的线程池使用的全都是无界队列，而使用无界队列会带来很多弊端，最重要的就是，它可以无限保存任务，因此很有可能造成OOM异常。


ThreadPoolExecutor机制

    下面是ThreadPoolExecutor最核心的构造方法参数：
    1）corePoolSize核心线程池的大小
    2）maximumPoolSize 最大线程池大小，当队列满了 就会创建新线程直至最大
    3）keepAliveTime 线程池中超过corePoolSize数目的空闲线程最大存活时间；可以allowCoreThreadTimeOut(true)使得核心线程超出有效时间也关闭
    4）TimeUnit keepAliveTime的时间单位
    5）workQueue阻塞任务队列
    6）threadFactory新建线程工厂,可以自定义工厂
    7）RejectedExecutionHandler当提交任务数超过maximumPoolSize+workQueue之和时，任务会交给RejectedExecutionHandler来处理

重点讲解

    corePoolSize，maximumPoolSize，workQueue三者之间的关系
    1）当线程池小于corePoolSize时，新提交的任务会创建一个新线程执行任务，即使线程池中仍有空闲线程。
    2）当线程池达到corePoolSize时，新提交的任务将被放在workQueue中，等待线程池中的任务执行完毕
    3）当workQueue满了，并且maximumPoolSize > corePoolSize时，新提交任务会创建新的线程执行任务
    4）当提交任务数超过maximumPoolSize，新任务就交给RejectedExecutionHandler来处理
    5）当线程池中超过 corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程
    6）当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭

RejectedExecutionHandler拒绝策略

    1、AbortPolicy策略：该策略会直接抛出异常，阻止系统正常工作；
    2、CallerRunsPolicy策略：如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行；
    3、DiscardOledestPolicy策略：该策略会丢弃任务队列中最老的一个任务，也就是当前任务队列中最先被添加进去的，马上要被执行的那个任务，并尝试再次提交；
    4、DiscardPolicy策略：该策略会默默丢弃无法处理的任务，不予任何处理。当然使用此策略，业务场景中需允许任务的丢失；


参考
https://blog.csdn.net/qq_34436819/article/details/103207685?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.no_search_link&spm=1001.2101.3001.4242.0


    线程设置越多越好吗？设置到什么值比较合理？
看业务具体定论



锁



    CAS实现机制？
CAS实际是一个CPU指令：比较并交换，对一个值得修改操作要先取得该值，获取到的值与预估值如果一直，则进行赋值操作，如果不一致，则重新获取再次尝试


    CAS的ABA问题
取到的值和预估值一样，但不排除从获取到比较之间该值有“变化再变回”的可能，但ABA问题并不影响操作结果，要避免可以使用java提供的原子操作，或使用互斥锁



算法


    有哪些常用的排序算法？
    插入排序、希尔排序、堆排序、归并排序、快速排序