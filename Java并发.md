#### 小Tips

* CPU处理指令速度很快，可是内存读写速度很慢，每次CPU都在等待内存的读写，代价太高，所以需要缓存
* JMM是Java内存模型，是一个规范，对共享数据的可见性原子性有序性的规则和保障

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220701202234550.png" alt="image-20220701202234550" style="zoom:67%;" />

* 关于主内存与工作内存的交互，即变量如何从主内存拷贝到工作内存、从工作内存同步回主内存，JMM
  定义了 8 种原子操作 ：

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220701203707527.png" alt="image-20220701203707527" style="zoom:67%;" />

#### as-if-serial  

不管怎么重排序，单线程程序的执行结果不能改变，编译器和处理器必须遵循 as-if-serial 语义。为了遵循 as-if-serial，编译器和处理器不会对存在数据依赖关系的操作重排序，因为这种重排序会改变执行结果。但是如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。as-if-serial 把单线程程序保护起来，给程序员一种幻觉：单线程程序是按程序的顺序执行的  

#### happens-before  

* **程序次序规则：**一个线程内写在前面的操作先行发生于后面的。
* **管程锁定规则：** unlock 操作先行发生于后面对同一个锁的 lock 操作。
* **volatile 规则**：对 volatile 变量的写操作先行发生于后面的读操作。
* **线程启动规则：**线程的 start 方法先行发生于线程的每个动作。
* **线程终止规则：**线程中所有操作先行发生于对线程的终止检测。
* **对象终结规则：**对象的初始化先行发生于 finalize 方法。
* **传递性：**如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作C 。  

#### 原子性、可见性、有序性

* JMM 还提供了 lock 和 unlock 操作满足需求，尽管 JVM 没有把这两种操作直接开放给用户使用，但是提供了更高层次的字节码指令 monitorenter（莫倪特安特） 和monitorexit（莫尼特e克c特），这两个字节码指令反映到 Java 代码中就是 synchronized。  

![image-20220701201904193](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220701201904193.png)

* 可见性指当一个线程修改了共享变量时，其他线程能够立即得知修改。JMM 通过在变量修改后将值同步回主内存，在变量读取前从主内存刷新的方式实现可见性，无论普通变量还是 volatile 变量都是如此，区别是 volatile 保证新值能立即同步到主内存以及每次使用前立即从主内存刷新主要因为调用了lock的底层方法，通过lock前缀指令以及通过缓存一致性来保证可见性 缓存一致性协议是硬件底层的机制（可采用缓存行填充的方式减少更新频率）。除了volatile外，synchronized和final也可以保证可见性，同步块可见性由"对一个变量执行 unlock前必须先把此变量同步回主内存，即先执行 store 和 write"这条规则获得。 final 的可见性指：被 final修饰的字段在构造方法中一旦初始化完成，并且构造方法没有把 this 引用传递出去，那么其他线程就能看到 final 字段的值 。

* 有序性可以总结为：在本线程内观察所有操作是有序的，在一个线程内观察另一个线程，所有操作都是无序的。前半句指 as-if-serial 语义，后半句指指令重排序和工作内存与主内存延迟现象。Java 提供 volatile 和 synchronized 保证有序性，volatile 本身就包含禁止指令重排序的语义具体流程，是底层通过屏障机制（通过锁定缓存行或者锁定CPU内存的总线来实现设置屏障），来阻止了指令重排。

  **创建一个对象的过程**

  分为3步:

  1、new–>申请一段内存空间，此时对象的属性值都是默认值；

  2、调用构造方法，初始化对象，给对象中的各个属性赋值，赋值完后，该对象才是完整的对象；

  3、建立关联，将引用指向对象地址。

   如果二三步骤发生指令重排，先建立关联，引用指向的对象此时是默认值，别的线程此时读到的是对象的默认值而出现错误。（不上锁的代码可以读到上锁代码的中间态）

   什么情况下允许指令重排？

  单线程下保证最终一致性的话就可以允许指令重排。

   如何防止指令重排？

  Java层面上设置屏障指令——volatile

  JVM层面上是使用了四种屏障——针对读读LoadLoad，读写LoadStore，写读StoreLoad，写写StoreStore的屏障。

  源码中是用C语言写的lock指令，通过锁定缓存行或者是CPU访问内存的总线来设置屏障。

  而synchronized 保证一个变量在同一时刻只允许一条线程对其进行 lock 操作，确保持有同一个锁的两个同步块只能串行进入。  

#### 进程和线程

* 进程是系统正在运行的应用程序，系统进行资源分配和调度的基本单位
* 线程就是操作系统进行运算调度的最小单位，包含在进程之中

#### 线程的状态

- **创建状态(new)** ：进程正在被创建，尚未到就绪状态。
- **运行状态(runnable)** ：进程正在CPU上运行(单核 CPU 下任意时刻只有一个进程处于运行状态)。
- **阻塞状态（blocked）：**被阻塞等待监视器锁定的线程处于此状态
- **等待状态(waiting)** ：进程正在等待某一事件而暂停运行如等待某资源为可用或等待 IO 操作完成。即使处理器空闲，该进程也不能运行。
- **限期等待状态(time_waiting)：**限期等待状态，可以在指定时间内自行返回。 可能由于调用了带参的 wait 和join 方法。  
- **结束状态(terminated)** ：进程正在从系统中消失。可能是进程正常结束或其他原因中断退出运行。

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220702163330903.png" alt="image-20220702163330903" style="zoom:67%;" />

#### sleep和wait区别

* sleep是Thread里的静态方法，wait是Object里的方法，任何实例对象都可以调用
* sleep不会释放锁，它也不需要占用锁。wait会释放锁，但调用它的前提是当前线程占用锁（代码要在synchronized中）
* sleep会设置时间自动唤醒，wait设置时间会唤醒，不设置的话需要notify函数进行唤醒，**但是notify不释放锁，唤醒了继续等**
* 他们都可以被interrupted（interr ra不踢的）方法打断

#### 缓存一致性协议（解决线程可见性问题）

我们考虑空间时间的方面，在CPU和内存间引入缓存的概念，多采用三级缓存的架构，其中我们一次性读取的数据块称之缓存行，如果缓存行大：命中率就高，但读取效率低。如果缓存行小：命中率就低，但读取效率高，目前大多采用64bytes为一行，由于缓存行的存在，我们必须有一种机制，来保证缓存数据的一致性，这种机制被称为缓存一致性协议。

#### 不要在构造函数中启动线程 可能读到中间值

#### 锁的前言

创建100个线程，每个对一个变量a = 0进行自增，由于我从内存拿到本地，在本地寄存器进行++，++完后放回内存，在放回这个操作还没完成呢，0又被拿了，导致自增不起来。

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220702153728254.png" alt="image-20220702153728254" style="zoom:67%;" />

#### synchronized

* 不可中断的

* 可重入的

* synchonized实现过程
  * java代码: synchronized

  * monitorenter moniterexit
  * 执行过程中自动升级
  * lock comxchg

#### 对象在内存中的存储布局

![image-20220630154755332](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220630154755332.png)

![image-20220630154509533](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220630154509533.png)

如上图的例子： markdown里包含8个字节 class pointer里包含4个字节 int 4个字节 long8个字节  boolean 1个字节 s引用类型4个字节 要被8整除所以补全 最后得出32个字节

* synchronized是在对象的markdown中做个标记

#### CAS（自旋锁）属于轻量级锁、乐观锁

* 全称 compare and swap
* 不用上锁也可以保证结果正确
* 自增操作 0读到本地，寄存器自增变成1，1读到内存的过程中，判断和我之前拿的时候的值是否是**同一个**，相等则改成1，不相等就重新拿入这个值到本地重新自增，譬如1回来发现变成3了，那么我就拿3进本地重新加1，变成4。 

* 和上一点的加粗问题，可能值相同，但是不是同一个，ABA操作，可能拿到的0，是别的线程+8 -8后的，数字可能没差，但是引用出现ABA问题就出大问题，加版本号可以解决。boolean类型的版本号解决，被修改过一次就改为false。（StampedReference）版本号（MarkableReferences）布尔类型的版本号
* CAS还有个问题就是判断为0的过程，判断完后准备把它改成1，这时候另一个线程又进来提前改成别的数，你再改成1就不对了
* 解决在unsafe类，native源码里也有一个unsafe类，调用了c++里的cmpxchg，compare exchange代码，最终指令实现是Lock cmpxchg，lock如何实现原子性：锁缓存、关中断、锁总线（就是只允许我的总线进行操作，操作完了才可以别的进来）

#### 虚拟机实现

JVM和操作系统出现线程调度问题，JVM偷懒直接交给操作系统，golang中的协程就是自己解决这个问题。

#### 重量级锁

与轻量级的区别，JVM解决不了交给操作系统老大处理的都是重量级锁。

* 自旋消耗CPU资源，如果锁的时间长，或者自选线程时间长，或者自旋线程多， CPU会被大量消耗，会升级重量级锁

* 排队：在操作系统里每一把锁都对应一个队列，重量级进入等待序列，轻量级锁自旋就行。
* 如果并发量高、临界区比较长线程等待时间过长的话等待序列的好处节省CPU资源，自旋的话消耗CPU资源 
* 把过来拿不到锁的送到waitSet（等待队列）里不会占用CPU资源

#### 轻量级如何升级为重量级锁

* wait() 直接进入队列，直接升级
* 可以参数调优 比如自旋次数多了就升级，比如并发数多了就升级，自旋线程数超过CPU数的一半

#### 偏向锁 不太重要

JDK13后 默认开启改成默认关闭了

第一任线程有优先权 ，markdown中标记，因为我们现实中代码基本80%都是单线程，基本都是自己的线程，所以解决繁琐的锁过程，引用偏向锁，看到是自己的标记直接进入线程即可。偏向锁默认4s后启动，明知道多线程情况，没必要启动偏向锁，JVM启动过程多线程启动，偏向设为多线程启动之后再启动-XX：BiasedLockingStartupDelay = 0,直接启动

#### 偏向锁是否一定比自旋锁效率高

* 不一定，在明确知道会有多线程竞争的情况下，偏向锁肯定会涉及锁撤销，这时候直接使用自旋锁
* JVM启动过程，会有很多线程竞争（明确），所以默认情况启动时不打开偏向锁，等启动完成后打开偏向锁

#### 可重入锁

锁了一下再进去锁一下，很少遇见不可重入，譬如子类调用父类的syn的方法，还要super一下所以syn都是可重入的

#### 死锁（五个哲学家吃饭）

* 死锁一般具有两把以上的锁，在锁定一把的时候等待一把锁
* 两把锁合并一把锁（5把锁合成一把锁，锁的粗化，锁定整个对象）
* 混进一个左撇子 甚至奇数 偶数 分开 分进一半左撇子

 #### 启动线程的三种方式

* 继承Thread类
* 实现Runnable的接口
* lambda表达式 不满意就回答线程池

#### 线程的三个方法

* sleep 方法会导致当前线程进入休眠状态，与 wait 不同的是该方法不会释放锁资源，进入的是TIMED-WAITING 状态。
* yiled 方法使当前线程让出 CPU 时间片给优先级相同或更高的线程，回到 RUNNABLE 状态，与其他线程一起重新竞争CPU时间片。
* join 方法用于等待其他线程运行终止，如果当前线程调用了另一个线程的 join 方法，则当前线程进入阻塞状态，当另一个线程结束时当前线程才能从阻塞状态转为就绪态，等待获取CPU时间片。底层使用的是wait，也会释放锁。  

#### Lock 和 synchronized 的区别
Lock 接是 juc 包的顶层接口，基于Lock 接口，用户能够以非块结构来实现互斥同步，摆脱了语言特性束缚，在类库层面实现同步。Lock 并未用到 synchronized，而是利用了 volatile 的可见性。重入锁 ReentrantLock 是 Lock 最常见的实现，与 synchronized 一样可重入，不过它增加了一些高级功能：

* 等待可中断： 持有锁的线程长期不释放锁时，正在等待的线程可以选择放弃等待而处理其他事情。trylock函数 返回true 或 false
* 公平锁： 公平锁指多个线程在等待同一个锁时，必须按照申请锁的顺序来依次获得锁，而非公平锁不保证这一点，在锁被释放时，任何线程都有机会获得锁。synchronized 是非公平的，ReentrantLock 在默认情况下是非公平的，可以通过构造方法指定公平锁。一旦使用了公平锁，性能会急剧下降，影响吞吐量。
* 锁绑定多个条件： 一个 ReentrantLock 可以同时绑定多个 Condition。synchronized 中锁对象的wait 跟 notify 可以实现一个隐含条件，如果要和多个条件关联就不得不额外添加锁，而ReentrantLock 可以多次调用 newCondition 创建多个条件。

一般优先考虑使用 synchronized：① synchronized 是语法层面的同步，足够简单。② Lock 必须确保在 finally 中释放锁，否则一旦抛出异常有可能永远不会释放锁。使用 synchronized 可以由 JVM 来确保即使出现异常锁也能正常释放。③ 尽管 JDK5 时 ReentrantLock 的性能优于 synchronized，但在 JDK6进行锁优化后二者的性能基本持平。从长远来看 JVM 更容易针对synchronized 优化，因为 JVM 可以在线程和对象的元数据中记录 synchronized 中锁的相关信息，而使用 Lock 的话 JVM 很难得知具体哪些锁对象是由特定线程持有的。  

#### synchronized锁

* 可重入（如果不可重入，就会引起死锁）（子类不可以重写父类了）
* 不可中断：当一个线程获得锁后，另一个线程一直处于阻塞或等待状态，前一个线程不释放锁，后一个线程会一直阻塞或等待，不可被中断。
* synchronized的锁对象会关联一个monitor,这 个monitor不是我们主动创建的,是VM的线程执行到这个同步代码块，发现锁对象没有monitor就会创建monitor,monitor内部有两个重要的成员变量owner:拥有这把锁的线程,recursions会记录线程拥有锁的次数,当一个线程拥有monitor后其他线程只能等待
* 能执行monitorexit指令的线程一定是拥有对象的monitor的所有权的线程
* 执行monitorexit时会将monitor的进入数减1。当monitor的进入数减为0时， 当前线程退出monitor, 不再拥有monitor的所有权，此时其他被这个monitor阻塞的线程可以尝试去获取这个monitor的所有权。
* synchronized，代码块出异常，JVM会自动释放锁 

#### 介绍JUC下的同步锁 基于AQS

AQS 是多线程同步器，它是 J.U.C 包中多个组件的底层实现，如 Lock、 CountDownLatch、Semaphore 等都用到了 AQS. 从本质上来说，AQS 提供了两种锁机制，分别是排它锁，和共享锁。 排它锁，就是存在多线程竞争同一共享资源时，同一时刻只允许一个线程访问该 共享资源，也就是多个线程中只能有一个线程获得锁资源，比如 Lock 中的 ReentrantLock 重入锁实现就是用到了 AQS 中的排它锁功能。 共享锁也称为读锁，就是在同一时刻允许多个线程同时获得锁资源，比如 CountDownLatch 和 Semaphore 都是用到了 AQS 中的共享锁功能。AQS采用了一个int类型的互斥变量state，用来记录锁竞争状态，表示现在没有线程竞争到锁资源，大于等于1表示已经有线程正在持有锁资源，一个线程获得锁资源的时候先判断state是否为0，是否无锁状态如果是，把state更新为1，多个线程同时操作就会出现竞争问题，所以采用了CAS机制保证更新原子性，没有获得锁资源的线程根据unsafe中的park操作把阻塞线程按照先进先出的规则放入双向链表的结构中，当有线程释放锁后，在双向链表头部去唤醒下一个等待线程，再去竞争锁，锁的公平性和非公平性，公平性的锁就去查看双向链表中是否有等待线程有就等待，没有就竞争，非公平直接更改state竞争锁！

#### ReentrantLock 可重入锁

* 替代synchronized，需要unlock写在try finally里
* 可以使用tryLock（时间）可以决定要不要wait、要不要阻塞，等待可中断。synchronized不可中断
* 可以使用lockInterruptibly（）这种方式来加锁可以被打断，后用interrupt（）可以被打断
* ReentranLock(true)公平锁，线程来取锁先检查队列有没有等待，有就等，没有就去抢锁，默认非公平锁

#### CountDownLatch

CountDownLatch 是基于执行时间的同步类，允许一个或多个线程等待其他线程完成操作，构造方法接收一个 int 参数作为计数器，如果要等待 n 个点就传入 n。每次调用 countDown 方法时计数器减 1，await 方法会阻塞当前线程直到计数器变为0，由于 countDown 方法可用在任何地方，所以 n 个点既可以是 n 个线程也可以是一个线程里的 n 个执行步骤。 

#### Phaser（非the）

 分为每个阶段，每个阶段完成相应的线程才可以到达下一个阶段，必须完成那个阶段需要完成的所有线程才可以到达下一个阶段

#### ReadWriteLock

* 读就是共享锁写就是排他锁
* 大大提高了读的效率
* 读的时候不加锁，是不行的，可能写一半被读了

#### Semaphore(塞摸for)

* 限流，最多允许多少个线程同时使用
* 信号量用来控制同时访问特定资源的线程数量，通过协调各个线程以保证合理使用公共资源。信号量可以用于流量控制，特别是公共资源有限的应用场景，比如数据库连接。Semaphore 的构造方法参数接收一个 int 值，表示可用的许可数量即最大并发数。使用 acquire 方法获得一个许可证，使用 release 方法归还许可，还可以用 tryAcquire 尝试获得许可。  也可以设公平和不公平，默认不公平

#### Exchanger

* 交换者是用于线程间协作的工具类，用于进行线程间的数据交换。它提供一个同步点，在这个同步点两个线程可以交换彼此的数据。
* 两个线程通过 exchange 方法交换数据，第一个线程执行 exchange 方法后会阻塞等待第二个线程执行该方法，当两个线程都到达同步点时这两个线程就可以交换数据，将本线程生产出的数据传递给对方。应用场景包括遗传算法、校对工作等。  

#### LockSupport

* 锁的支持，park（），当前线程停止
* 解封LockSupport.unpark（），继续运行 
* 与wait区别，wait需要在锁上执行，LockSupport.park()不需要
* 可以叫醒指定的线程
* unpark可以先行park调用，不用停车了

#### 线程池

* 提高线程利用率
* 提高程序的响应速度
* 便于统一管理线程对象
* 可以控制最大并发数

#### 线程池参数

- **`corePoolSize`：线程池的核心线程数量**

  核心线程数线程数定义了最小可以同时运行的线程数量。

- **`maximumPoolSize` ：线程池的最大线程数** 

  当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。

- **`workQueue`：任务队列，用来储存等待执行任务的队列** 

  当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

- **`keepAliveTime`：线程空闲时间，当线程数大于核心线程数时，多余的空闲线程存活的最长时间**

  当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，**核心线程外**的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；

- **`unit`：时间单位**

  `keepAliveTime` 参数的时间单位。

- **`threadFactory`：线程工厂，用来创建线程，一般默认即可** 

  executor 创建新线程的时候会用到。用来生产一组相同任务的线程。可以给线程命名，有利于分析错误。  

- **`handler`：拒绝策略，当提交的任务过多而不能及时处理时，我们可以定制策略来处理任务** 

  默认使用 AbortPolicy 丢弃任务并抛出异常，CallerRunsPolicy 表示重新尝试提交该任务DiscardOldestPolicy 表示抛弃队列里等待最久的任务并把当前任务加入队列，DiscardPolicy 表示直接抛弃当前任务但不抛出异常。  

#### 线程池任务执行过程

首先任务线程过来调用execute方法，判断线程池里面的核心线程数有没有满，，没有满就创建一个线程，去执行你提交过来这个任务，核心线程满了，看看任务队列满没满，任务队列没满，放入任务队列里等待，无界队列和有界队列、满了的话，创建非核心线程，如果超过最大核心线程就采用拒绝策略。

#### 线程池提交 execute和submit

* execute没有返回值，submit有返回值
* execute只可以提交runnable,submit可以提交runnable和callable
* execute可以抛出异常，submit不会抛出异常

#### Threadlocal

ThreadLocal 提供一个线程内的局部变量，不同线程之间不会相互干扰。ThreadLocal 有一个静态内部类 ThreadLocalMap，其 Key 是 ThreadLocal 对象，值是 Entry 对象，Entry 中只有一个 Object 类的 vaule 值。ThreadLocal 是线程共享的，但 ThreadLocalMap 是每个线程私有的。ThreadLocal 主要有 set、get 和 remove 三个方法。

* 线程并发：用于多线程并发的情况下的
* 传递数据：我们可以通过ThreadLocal在同一线程，不同的组件中传递公共变量
* 线程隔离：每个线程变量都是独立的，不会互相影响

**set 方法**
		首先获取当前线程，然后再获取当前线程对应的 ThreadLocalMap 类型的对象 map。如果 map 存在就直接设置值，key 是当前的 ThreadLocal 对象，value 是传入的参数。如果 map 不存在就通过 createMap 方法为当前线程创建一个 ThreadLocalMap 对象再设置值。
**get 方法**
		首先获取当前线程，然后再获取当前线程对应的 ThreadLocalMap 类型的对象 map。如果 map 存在就以当前 ThreadLocal 对象作为 key 获取 Entry 类型的对象 e，如果 e 存在就返回它的 value 属性。如果 e 不存在或者 map 不存在，就调用 setInitialValue 方法先为当前线程创建一个ThreadLocalMap 对象然后返回默认的初始值 null。
**remove 方法**
		首先通过当前线程获取其对应的 ThreadLocalMap 类型的对象 m，如果 m 不为空，就解除ThreadLocal 这个 key 及其对应的 value 值的联系。
**存在的问题**
		线程复用会产生脏数据，由于线程池会重用 Thread 对象，因此与 Thread 绑定的 ThreadLocal 也会被重用。如果没有调用 remove 清理与线程相关的 ThreadLocal 信息，那么假如下一个线程没有调用 set设置初始值就可能 get 到重用的线程信息。
		ThreadLocal 还存在内存泄漏的问题，由于 ThreadLocal 是弱引用，但 Entry 的 value 是强引用，因此当 ThreadLocal 被垃圾回收后，value 依旧不会被释放。因此需要及时调用 remove 方法进行清理操作。  

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220629215742252.png" alt="image-20220629215742252" style="zoom:67%;" />

还要记得t1.remove()，用完要丢掉

#### ThreadLoacl 内存泄漏的问题

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220630145401305.png" alt="image-20220630145401305"  />

ThreadLocalMap中使用的key为ThreadLocal的弱引用，如下:

![image-20220630145506733](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220630145506733.png)

`弱引用:只要垃圾回收机制运行，不管JVM的内存空间是否充足，都会回收该对象占用的内存。`

弱引用比较容易被回收。因此，如果ThreadLocal (ThreadLocalMap的Key) 被垃圾回收器回收了，但是因为ThreadLocalMap生命周期和Thread是一样的，它这时候如果不被回收，就会出现这种情况:ThreadLocalMap的key没了，value还在，这就会**「造成了内存泄漏问题」**。如何「解决内存泄漏问题」?使用完ThreadLocal后，及时调用remove()方法释放内存空间。