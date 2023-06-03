\1.   Springboot中的注解说几个

\1.   Spring Boot和Spring的区别

\1.   怎么理解spring的微服务

\2.   为什么要用spring boot

\3.   Spring boot便捷是体现在哪

\1.   Spring Boot的优点

\2.   微服务了解吗

\3.   Spring cloud用过吗

\4.   Mybatis数据类型

\5.   Spring的核心，IOC AOP

IOC控制反转，把对象的创建和对象之间的调用过程，交给Spring进行管理，用来降低计算机代码之间的耦合度，

底层原理：1、xml解析、工厂模式、反射 

\6.   Spring MVC

\1.   Spring boot日志了解吗

\2.   Spring boot事务了解吗

\1.   IOC和AOP说一下

\1.   Spring boot的starter

#### AOP就是IOC的一个扩展实现

![image-20220709170727633](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220709170727633.png)

![image-20220723215057561](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220723215057561.png)

![image-20220723210520404](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220723210520404.png)

![image-20220724152247228](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724152247228.png)

 

![image-20220724151722306](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724151722306.png)



refresh的13个方法

* 先看有没有beanfactory对象没有的话直接创建 new一个DefaultListableBeanFactory对象叫beanFactory，有的话销毁了再创建bean工厂

* 此时xml读入后   这个BeanDefinition对象都是默认值

* 下面加载beanFactory得到BeanDefinition对象此时还是原始值${jdbc.url}，还没替换

* 此时执行invokeBeanFactoryPostProcessor增强器，完成替换值 url username都替换了

* 可以放监听器 ，监听事件 在我实例化前准备工作要做到位

* 国际化处理：切换语言的

* 初始化多播器 就是监听器

* 这时候可以单例化的实例操作了 非懒加载

* 反射开始 docreatebean方法 Constructor ctor = clazz.getDeclareConstructor();Object obj =  ctor.newInstance();

* 对象实例化好了 开始填充属性 把配置文件里的属性值填充进来 譬如你名字叫张三就填充张三

* 下面进行实现aware接口，实现容器对象的赋值

* 下面进行before增强 

* init方法

* after 增强--- AOP是一个小功能

* 结束可以直接拿对象 getbean（）方法

* 最后 结束了 init-destory销毁

  ![image-20220724151511892](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724151511892.png)

上图解析：首先xml和注解通过一系列操作得到BeanDefinition的对象此时是原始值也就是占位符未被替换，经过BeanFactoryPostProcessor将值替换得到最终的BeanDefinition对象

![image-20220723214300155](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220723214300155.png)

![image-20220723214358993](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220723214358993.png)

![image-20220723214424861](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220723214424861.png)

理解bean的生命周期

* 实例化 分配内存、属性默认值
* 用户自定义属性（set get）
* 容器对象赋值 通过实现aware接口来完成set get赋值
* 下面全是spring扩展实现 初始化前置处理方法
* 初始化（不是前一个初始化） 应该叫做执行初始化调用方法
* 后置处理方法---->AOP
* 使用对象
* 销毁对象

![image-20220723214617922](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220723214617922.png)

FactoryBean你可以按照自己想的方式去生成对象new 代理 反射生成对象，而不需要去经历Bean的生命周期

![image-20220724170511320](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724170511320.png)

#### 什么是Spring IOC容器

**IOC即控制反转，总结而言就是借助IOC容器实现对象之间的解耦并完成对象的创建、配置它们并管理它们的完整生命周期。**IOC的实现方式有依赖注入和依赖查找，依赖查找基本用的很少，所以IOC又叫依赖注入，依赖注入。依赖注入指对象被动地接受依赖类而不用自己主动去找，对象不是从容器中查找它依赖的类，而是在容器实例化对象时主动将它依赖的类注入给它。假设一个 Car 类需要一个 Engine 的对象，那么一般需要需要手动 new 一个 Engine，利用 IoC 就只需要定义一个私有的 Engine 类型的成员变量，容器会在运行时自动创建一个 Engine 的实例对象并将引用自动注入给成员变量。  

* 通过xml文件经过抽象接口（定义读取配置文件的规范）BeanDefinationReader（抽象接口）定义信息解析出BeanDefinaition

![image-20220724150311509](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724150311509.png)

BeanDefinationReader实现类，能够实现对应的功能，解析出BeanDefination

**过程**

* xml配置文件，配置创建的对象（在spring配置文件中，使用bean标签，标签里面添加对应属性，就可以实现对象的创建，默认是无参构造函数）
  * bean中的属性
    * id属性：唯一标识（不允许重复）
    * class属性：类的全路径（包类路径）
  
  ~~~~java
  <bean id = "userdao" class = "com.atguigu.UserDao"></bean>
  ~~~~
  
* 创建工厂类（直接当作容器 直接通过beanfactory获得对象，ApplicationContext继承beanfactory）

![image-20220718140008187](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220718140008187.png)

* 通过反射读取到字节码，也就是.class文件，进行对象的创建 先获取class对象 Class.forName("完全限定名");或者对象.getClass()或者类.class             下面是第一种                  clazz.newInstance();还有    Constructor ctor = clazz.getDeclareConstructor();Object obj =  ctor.newInstance();

  ![image-20220724151511892](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724151511892.png)

~~~java
class UserFactory{
	public static UserDao getDao()
    {
    	String classValue = class属性值//1.xml解析
        Class clazz = Class.forName(classValue);
        return (UserDao)clazz.newInstance();
    }
}
~~~

**接口**

IOC思想基于IOC容器完成，IOC容器的底层就是对象工厂（beanfactory里有很多抽象类和接口）

Spring提供IOC容器实现的两个方式（接口）

* BeanFactory：IOC容器基本实现，是Spring内部的使用接口，不提供开发人员使用
* ApplicationContext：BeanFactory的子接口，提供更多更强大的功能，由开发人员使用

Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

#### 什么是依赖注入？可以通过多少种方式完成依赖注入？

在依赖注入中，您不必创建对象，但必须描述如何创建它们。您不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。由loC容器将它们装配在一起。通常，依赖注入可以通过三种方式完成，即: 

解决的问题就是：业务层需要调用持久层的方法，不想编写程序来写出依赖关系，而想通过Spring来维护，通过spring自动把持久层对象传入业务层，不用我们自己去获取，去把UserDao注入到UserService

* 构造函数注入（有参构造）
* setter注入

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220718143442348.png" alt="image-20220718143442348" style="zoom:80%;" />

![image-20220718143519150](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220718143519150.png)

* 接口注入
* 在SpringFramework中，仅使用构造函数和setter注入。

#### 区分BeanFactory和ApplicationContext？

| BeanFactory                | ApplicationContext       |
| -------------------------- | ------------------------ |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显示提供资源对象 | 它自己创建和管理资源对象 |
| 不支持基于依赖的注解       | 支持依赖的注解           |

BeanFactory和ApplicationContext的优缺点分析:
**BeanFactory的优缺点:**

* 优点:应用启动的时候占用资源很少，对资源要求较高的应用，比较有优势;
* 缺点:运行速度会相对来说慢一些。而且有可能会出现空指针异常的错误，而且通过Bean工厂创建的Bean生命周期会简单一些。

**ApplicationContext的优缺点:**（ClassPathXmlApplication实现类，getbean（））

* 优点:所有的Bean在启动的时候都进行了加载，系统运行的速度快;在系统启动的时候，可以发现系统中的配置问题。
* 缺点:把费时的操作放到系统启动中完成，所有的对象都可以预加载，缺点就是内存占用较大。

####  Bean 的作用范围

​	通过 scope 属性指定 bean 的作用范围，包括：
​		① singleton：单例模式，是默认作用域，不管收到多少 Bean 请求每个容器中只有一个唯一的 Bean实例。
​		② prototype：原型模式，和 singleton 相反，每次 Bean 请求都会创建一个新的实例，多例的。
​		③ request：每次 HTTP 请求都会创建一个新的 Bean 并把它放到 request 域中，在请求完成后 Bean会失效并被垃圾收集器回收。
​		④ session：和 request 类似，确保每个 session 中有一个 Bean 实例，session 过期后 bean 会随之失效。
​		⑤ global session：当应用部署在 Portlet 容器时，如果想让所有 Portlet 共用全局存储变量，那么该变量需要存储在 global session 中。

#### Bean 的生命周期

* (1)通过构造器创建bean实例(无参数构造) 
* (2)为bean的属性设置值和对其他bean引用(调用set方法)。
* (3)调用bean的初始化的方法(在xml中需要进行配置初始化的方法init-Method)。
* (4) bean可以使用了(对象获取到了)
* (5)当容器关闭时候，调用bean的销毁的方法(需要进行配置销毁的方法)destroy-method属性 
* ![image-20220718141756474](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220718141756474.png)

#### 数据源（Druid）

* 提高程序性能
* 实例化数据源    初始化部分连接资源
* 使用获取
* 用完还回去
* 步骤
  * 导入数据源的坐标和数据库驱动 坐标
  * 创建数据源对象
  * 设置数据源

#### Spring如何解决循环依赖

![image-20220724182955667](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724182955667.png)

* 三级缓存，提前暴露对象 AOP

* 总：循环依赖问题 A依赖B B依赖A

* 分：说明bean的创建过程：实例化 初始化  （属性赋值）

* 打断闭环操作 1先创建A对象 实例化A对象，A对象中的B属性为空，填充属性B

* 从容器中查找B对象，找到了直接赋值，不存在循环依赖的问题（不通），找不到直接创建B对象

* 实例化B对象，此时B对象中的A属性为空，填充属性A

* 从容器中查找A对象，找不到直接创建，上述形成闭环原因

* 此时仔细琢磨 A对象存在，此时A对象不是一个完整状态，只完成实例化未完成初始化 也就是半初始化状态如果在程序调用过程中

  拥有了某个对象的引用，能否后期给他完成赋值操作，可以优先把非完整状态的对象优先赋值，等待后序操作来完成赋值，相当于提前暴露了某个不完整对象的引用，所以解决问题的核心就在于实例化和初始化分开操作，所有对象都完成实例化和初始化操作后，还要把完整对象放到容器中，此时在容器中存在对象的几个状态，完成实例化未完成初始化、完整状态，因为在容器中所以就要采取不同的容器存储，此时就有了一级缓存和二级缓存。如果一级缓存中有 二级缓存就不会存在同名对象，因为他们查找顺序是123，这样的方式来查找的 一级缓存中存放的完整对象 二级缓存中放的是非完整对象，半成品

  为什么需要三级缓存 三级缓存的value类型是ObjectFactory，是一个函数式接口，存在的意义是保证在整个容器的运行过程中只能有一个同名bean的对象

  如果一个对象被代理，是要生成一个普通对象的。

  普通对象和代理对象是不能同时出现在容器中的，当一个对象需要被代理的时候，就需要代理对象覆盖掉之前的普通对象，实际的调用过程中，是没有办法确定什么时候对象被使用的，所以就要求某个对象被调用的时候，优先判断此对象是否需要被代理，类似回调机制的实现，因此传入lambda表达式，可以通过lambda表达式来执行对象的覆盖过程，getEarlyBeanReference（）

  因此所有的bean的对象优先放到三级缓存中，如果需要被代理，则返回代理对象，不需要被代理返回普通对象

  ![image-20220724192216410](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724192216410.png)

![image-20220724192811320](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220724192811320.png)

#### 通过注解创建 Bean

​		@Component 把当前类对象存入 Spring 容器中，相当于在 xml 中配置一个 bean 标签。value 属性指定 bean 的 id，不加的话，默认使用当前类的首字母小写的类名。

~~~java
@Component(value = "userService")
public class UserService
{
    public void add(){
    	System.out.println("add.....")
     }
}
~~~

​		@Controller ， @Service ， @Repository 三个注解都是 @Component 的衍生注解，作用及属性都是一模一样的。只是提供了更加明确语义， @Controller 用于表现层， @Service 用于业务层，@Repository 用于持久层。如果注解中有且只有一个 value 属性要赋值时可以省略 value。把类交给spring管理

* @Controller 注解主要声明将控制器类配置给Spring管理，例如Servlet
* @Service 注解主要声明业务处理类配置Spring管理，Service接口的实现类
* @Repository直接主要声明持久化类配置给Spring管理，DAO接口
* @Component除了控制器、service和DAO之外的类一律使用此注解声明

​		如果想将第三方的类变成组件又没有源代码，也就没办法使用 @Component 进行自动配置，这种时候就要使用 @Bean 注解。被 @Bean 注解的方法返回值是一个对象，将会实例化，配置和初始化一个新对象并返回，这个对象由 Spring 的 IoC 容器管理。name 属性用于给当前 @Bean 注解方法创建的对象指定一个名称，即 bean 的 id。当使用注解配置方法时，如果方法有参数，Spring 会去容器查找是否有可用 bean对象，查找方式和 @Autowired 一样。

#### 通过注解方式实现属性注入

~~~~java
//@Autowired   根据类型装配 UserDao
//第一步 把service和dao对象创建，在service和dao类添加对象注解
//第二步 在service注入dao对象，在service类添加dao类型属性，在属性上面使用注解。
@Service
public class UserService{
    //定义dao类型属性。
	//不需要添加set方法
	//添加注入属性注解。
	@Autowired
    private UserDao userdao;
    public void add(){
    	System.out.println("service add.......");
        userDao.add();
    }
}
//@Qualifier 根据名称进行注入 搭配Autowired一起使用
@Service
public class UserService{
    //定义dao类型属性。
	//不需要添加set方法
	//添加注入属性注解。
	@Autowired
    @Qualifier(value = "userdao1")//这里的目的就是UserDao接口有很多实现类，你可以指定注入具体的实现类，通过名称 以防你有多个子类继承userDao
    private UserDao userdao;
    public void add(){
    	System.out.println("service add.......");
        userDao.add();//多态父类的引用指向子类的对象，调用的方法是子类重写之后的方法
    }
}
// 上面两个都是spring里的包 Resource是javax扩展包里的 不建议但是功能可以的
@Service
public class UserService{
	@Resource //先byname找userdao  没找到就根据类型UserDao进行注入
    private UserDao userdao;
    public void add(){
    	System.out.println("service add.......");
        userDao.add();
    }
}
@Service
public class UserService{
	@Resource(name = "userDaoImpl1") //根据类型进行注入
    private UserDao userdao;
    public void add(){
    	System.out.println("service add.......");
        userDao.add();
    }
}
~~~~

![image-20220718153000718](C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220718153000718.png)

#### AOP

​		AOP 即面向切面编程，简单地说就是将代码中重复的部分抽取出来，在需要执行的时候使用动态代理技术，在不修改源码的基础上对方法进行增强，将切面织入到主业务中。
​		Spring 根据类是否实现接口来判断动态代理方式，如果实现接口会使用 JDK 的动态代理，核心是InvocationHandler 接口和 Proxy 类，**用proxy里面的newProxyInstance创建接口实现类的代理对象**。如果没有实现接口会使用 CGLib 动态代理，CGLib 是在运行时动态生成某个类的子类，如果某个类被标记为 final，不能使用 CGLib 。
​		JDK 动态代理主要通过重组字节码实现，首先获得被代理对象的引用和所有接口，生成新的类必须实现被代理类的所有接口，动态生成Java 代码后编译新生成的 .class 文件并重新加载到 JVM 运行。JDK代理直接写 Class 字节码，CGLib 是采用 ASM 框架写字节码，生成代理类的效率低。但是 CGLib 调用方法的效率高，因为 JDK 使用反射调用方法，CGLib 使用 FastClass 机制为代理类和被代理类各生成一个类，这个类会为代理类或被代理类的方法生成一个 index，这个 index 可以作为参数直接定位要调用的方法。
​		常用场景包括权限认证、自动缓存、错误处理、日志、调试和事务等。

**总结：AOP是面向切面编程，AOP关注的不再是程序代码中某个类，某些方法，考虑的是面到面，层与层之间的切入；AOP通常用来做一 些公共性的重复功能，比如安全控制、性能统计、日志记录；AOP作用是降低模块之间的耦合度，提高业务代码的聚合度，实现高内聚低耦合，提高代码的复用性，在不影响原有代码的情况下添加一些额外的功能；AOP的底层是动态代理实现的。**

AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

~~~~java
UserDao dao = (UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(),interfaces,new UserDao(userDao));
~~~~

#### 连接点 切入点  通知（增强）

* 连接点：类里面哪些方法可以被增强，这些方法称为连接点

* 切入点：实际被真正增强的方法被称为切入点

* 通知（增强）：实际增强逻辑部分就叫通知或者增强

  ​                                                         JDK动态代理：创建接口实现类代理对象，增强类的方法

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220629181741418.png" alt="image-20220629181741418" style="zoom: 80%;" />

​                                                               CGLIB动态代理:创建子类的代理对象，增强类的方法

<img src="C:\Users\JH\AppData\Roaming\Typora\typora-user-images\image-20220629181854744.png" alt="image-20220629181854744" style="zoom:80%;" />

#### 谈谈IOC和AOP的理解

IOC控制反转 AOP面向切面编程

Spring要自己控制和管理Bean，就需要从配置到实例化设置属性初始化这一套都要自己做，自己要操作bean，因此叫做控制反转
AOP叫做切面编程统一做局部功能的加强，Spring想要通过ioc控制反转来创建bean，就需要自己从xml和注解中读取资源文件，把bean对象通过BeanDefineReader，把bean对象读取为BeanDefinition，然后进行实例化，属性填充
AOP实现的是使用JDK动态代理和CGlib动态代理实现的功能增加，譬如日志的功能

#### Bean的生命周期

在 IoC 容器的初始化过程中会对 Bean 定义完成资源定位，加载读取配置并解析，最后将解析的 Bean信息放在一个 HashMap 集合中。当 IoC 容器初始化完成后，会进行对 Bean 实例的创建和依赖注入过程，注入对象依赖的各种属性值，在初始化时可以指定自定义的初始化方法。经过这一系列初始化操作后 Bean 达到可用状态，接下来就可以使用 Bean 了，当使用完成后会调用 destroy 方法进行销毁，此
时也可以指定自定义的销毁方法，最终 Bean 被销毁且从容器中移除。
XML 方式通过配置 bean 标签中的 init-Method 和 destory-Method 指定自定义初始化和销毁方法。
注解方式通过配置 @Bean 注解中的 init-Method 和 destory-Method 指定自定义初始化和销毁方法。  

#### BeanFactory、FactoryBean 和 ApplicationContext 的区别
* BeanFactory 是一个 Bean 工厂，使用简单工厂模式，是 Spring IoC 容器顶级接口，可以理解为含有Bean 集合的工厂类，作用是管理 Bean，包括实例化、定位、配置对象及建立这些对象间的依赖。BeanFactory 实例化后并不会自动实例化 Bean，只有当 Bean 被使用时才实例化与装配依赖关系，属于延迟加载，适合多例模式。简而言之**创建一系列的类似对象**
* FactoryBean 是一个工厂 Bean，使用了工厂方法模式，作用是生产其他 Bean 实例，可以通过实现该接口，提供一个工厂方法来自定义实例化 Bean 的逻辑。FactoryBean 接口由 BeanFactory 中配置的对象实现，这些对象本身就是用于创建对象的工厂，如果一个 Bean 实现了这个接口，那么它就是创建对象的工厂 Bean，而不是 Bean 实例本身。简而言之**生成一个有很复杂属性的对象**
* ApplicationConext 是 BeanFactory 的子接口，扩展了 BeanFactory 的功能，提供了支持国际化的文本消息，统一的资源文件读取方式，事件传播以及应用层的特别配置等。容器会在初始化时对配置的Bean 进行预实例化，Bean 的依赖注入在容器初始化时就已经完成，属于立即加载，适合单例模式，一般推荐使用  

#### Springboot 事务

* Spring事务机制根据数据库事务和AOP机制的
* 首先对使用@Transctional注解的Bean，Spring会创建一个代理对象作为Bean
* 利用数据库管理器创建一个数据库连接
* 修改数据库连接的auto commit为false，禁止连接的自动提交
* 没出现异常就提交
* 出现异常就回滚

* 编程式事务管理 这意味着你可以通过编程的方式管理事务，这种方式带来了很大的灵活性，但很难维护。
* 声明式事务管理，基于AOP原理的注解@Transcational 这种方式意味着你可以将事务管理和业务代码分离。你只需要通过注解或者XML配置管理事务

#### Springboot Starter的工作原理

总结一下，其实就是Spring Boot在启动的时候，按照约定去读取Spring Boot Starter的配置信息，再根据配置信息对资源进行初始化，并注入到Spring容器中。这样Spring Boot启动完毕后，就已经准备好了一切资源，使用过程中直接注入对应Bean资源即可

#### Spring优势

* 方便解耦，简化开发
* AOP编程支持
* 声明式事务的支持
* 方便程序的测试
* 方便集成优秀的框架