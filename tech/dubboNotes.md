# Dubbo notes

[TOC]

## Question
1. dubbo 通信机制
1) serialization bug?:  private entity id is missing if parent class id existing
2) maybe it is better dubbo service API without spring 
3) 去掉jetty端口配置 默认20880 <dubbo:protocol name="dubbo" port="20891" payload="209715200" /> log​?
4) AbstractServer doOpen() twice, Address already bind Exception?

## Code Logic

### Misc.
(1)com.alibaba.dubbo.container.Main 
(2)java.util.ServiceLoader 
(3)com.alibaba.dubbo.common.extension.ExtensionLoader 
(4)com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper 
(5)com.alibaba.dubbo.demo.consumer.DemoConsumer 
(6)com.alibaba.dubbo.demo.provider.DemoProvider 
(7)com.alibaba.dubbo.container.spring.SpringContainer 
(8)com.alibaba.dubbo.config.spring.schema.DubboNamespaceHandler 
(9)com.alibaba.dubbo.config.spring.schema.DubboBeanDefinitionParser 
(A)com.alibaba.dubbo.config.spring.ServiceBean 
(B)com.alibaba.dubbo.config.spring.ReferenceBean 
 (1)读代码第三招(debug)和一个心法(思路)。 (2)钩子 ShutdownHook，kill 和man的使用。 (3)守护线程 Thread.setDaemon(true) (4)同步块范围对象控制，CountDownLatch。 (5)SPI基本思想，DCL，动态编译，包装调用链。 (6)Dubbo RPC调用的实际动作。 (7)jenv(jevn.io)管理java环境。 (8)Spring的一点handler和listener知识

CountDownLatch

### Provider start process
com.alibaba.dubbo.config.spring.schema.DubboNamespaceHandler.init()
实现了InitializingBean接口，在设置了bean的所有属性后会调用afterPropertiesSet方法
org.springframework.beans.factory.InitializingBean.afterPropertiesSet()

com.alibaba.dubbo.config.spring.ServiceBean

``` java
[23/11/15 04:04:31:031 CST] main ERROR container.Main:  [DUBBO] Fail to start server(url: dubbo://172.27.2.27:20880/com.alibaba.dubbo.demo.DemoService?anyhost=true&application=demo-provider&channel.readonly.sent=true&codec=dubbo&dubbo=2.0.0&generic=false&heartbeat=60000&interface=com.alibaba.dubbo.demo.DemoService&loadbalance=roundrobin&methods=sayHello&owner=william&pid=24622&side=provider&timestamp=1448265869456) Failed to bind NettyServer on /172.27.2.27:20880, cause: Failed to bind to: /0.0.0.0:20880, dubbo version: 2.0.0, current host: 127.0.0.1
com.alibaba.dubbo.rpc.RpcException: Fail to start server(url: dubbo://172.27.2.27:20880/com.alibaba.dubbo.demo.DemoService?anyhost=true&application=demo-provider&channel.readonly.sent=true&codec=dubbo&dubbo=2.0.0&generic=false&heartbeat=60000&interface=com.alibaba.dubbo.demo.DemoService&loadbalance=roundrobin&methods=sayHello&owner=william&pid=24622&side=provider&timestamp=1448265869456) Failed to bind NettyServer on /172.27.2.27:20880, cause: Failed to bind to: /0.0.0.0:20880
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.createServer(DubboProtocol.java:289)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.openServer(DubboProtocol.java:266)
	at com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol.export(DubboProtocol.java:253)
	at com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper.export(ProtocolListenerWrapper.java:56)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper.export(ProtocolFilterWrapper.java:55)
	at com.alibaba.dubbo.rpc.Protocol$Adpative.export(Protocol$Adpative.java)
	at com.alibaba.dubbo.registry.integration.RegistryProtocol.doLocalExport(RegistryProtocol.java:153)
	at com.alibaba.dubbo.registry.integration.RegistryProtocol.export(RegistryProtocol.java:107)
	at com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper.export(ProtocolListenerWrapper.java:54)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper.export(ProtocolFilterWrapper.java:53)
	at com.alibaba.dubbo.rpc.Protocol$Adpative.export(Protocol$Adpative.java)
	at com.alibaba.dubbo.config.ServiceConfig.doExportUrlsFor1Protocol(ServiceConfig.java:488)
	at com.alibaba.dubbo.config.ServiceConfig.doExportUrls(ServiceConfig.java:284)
	at com.alibaba.dubbo.config.ServiceConfig.doExport(ServiceConfig.java:245)
	at com.alibaba.dubbo.config.ServiceConfig.export(ServiceConfig.java:144)
	at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:109)
	at org.springframework.context.event.SimpleApplicationEventMulticaster$1.run(SimpleApplicationEventMulticaster.java:78)
	at org.springframework.core.task.SyncTaskExecutor.execute(SyncTaskExecutor.java:49)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:76)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:274)
	at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:736)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:383)
	at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:139)
	at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:93)
	at com.alibaba.dubbo.container.spring.SpringContainer.start(SpringContainer.java:50)
	at com.alibaba.dubbo.container.Main.main(Main.java:80)
	at com.alibaba.dubbo.demo.provider.DemoProvider.main(DemoProvider.java:21)
```

### Consumer start process
com.alibaba.dubbo.config.spring.ReferenceBean

通观全部Dubbo代码，有两个很重要的对象就是Invoker和Exporter，Dubbo会根据用户配置的协议调用不同协议的Invoker，再通过ReferenceFonfig将Invoker的引用关联到Reference的ref属性上提供给消费端调用。当用户调用一个Service接口的一个方法后由于Dubbo使用javassist动态代理，会调用Invoker的Invoke方法从而初始化一个RPC调用访问请求访问服务端的Service返回结果
Spring在初始化IOC容器时会利用这里注册的BeanDefinitionParser的parse方法获取对应的ReferenceBean的BeanDefinition实例，由于ReferenceBean实现了InitializingBean接口，在设置了bean的所有属性后会调用afterPropertiesSet方法
com.alibaba.dubbo.config.spring.schema.DubboNamespaceHandler.init()
com.alibaba.dubbo.config.spring.ReferenceBean.afterPropertiesSet()
com.alibaba.dubbo.config.ReferenceConfig.init()
com.alibaba.dubbo.config.ReferenceConfig.createProxy(Map<String, String>)

### Invocation
1. Invocation，一次具体的调用，包含方法名、参数类型、参数
2. Result，一次调用结果，包含value和exception
3. Invoker，调用者，对应一个服务接口，通过invoke方法执行调用，参数为Invocation，返回值为Result

#### DubboInvoker
通过ExchangeClient发送调用请求(Invocation)
doInvoke()分为oneWay、async、sync调用
对client的选择采用轮询的方式

#### Exchanger
[Source](http://gaofeihang.cn/archives/255)
在一个框架中我们通常把负责数据交换和网络通信的组件叫做Exchanger。Dubbo中每个Invoker都维护了一个ExchangeClient的引用，并通过它和远程的Server进行通信。整个与ExchangeClient相关的类图如下
![Dubbo-Exchanger](http://gaofeihang.cn/wp-content/uploads/2015/04/Dubbo-Exchanger.jpg "Dubbo-Exchanger") 
1. ExchangeClient只有一个常用的实现类，HeaderExchangeClient
2. 在Invoker需要发送数据时，单程发送使用的是ExchangeClient的send方法，需要返回结果的使用request方法
3. 最终send方法传递到channel的send，而request方法则是通过构建ResponseFuture和调用send组合实现的。接下来的重点就是这个Channel参数如何来构建
4. 它来自Transporters的connect方法，具体的Transporter来源于ExtensionLoader，默认为NettyTransporter，由它构建的是NettyClient。NettyClient再次维护了一个Channel引用，来自NettyChannel的getOrAddChannel()方法，创建的是NettyChannel。最终由基类AbstractClient实现的send方法调用了NettyChannel
5. 执行Netty的channel.write()将数据真正发送出去，也可以由此看出boolean sent;参数的含义：是否去等待发送完成、是否执行超时的判断。

本文主要以Client的角度对通信流程进行了介绍，Server端也可以遵循这样的路径去梳理，这里就不再赘述了。


### Register process

#### Provider
Provider初始化时会调用doRegister方法向注册中心发起注册

``` java
ZookeeperRegistry.doSubscribe(URL, NotifyListener) line: 114	
ZookeeperRegistry(FailbackRegistry).subscribe(URL, NotifyListener) line: 189	
RegistryProtocol.export(Invoker<T>) line: 117	
ProtocolListenerWrapper.export(Invoker<T>) line: 54	
ProtocolFilterWrapper.export(Invoker<T>) line: 53	
Protocol$Adpative.export(Invoker) line: not available	
ServiceBean<T>(ServiceConfig<T>).doExportUrlsFor1Protocol(ProtocolConfig, List<URL>) line: 488	
ServiceBean<T>(ServiceConfig<T>).doExportUrls() line: 284	
ServiceBean<T>(ServiceConfig<T>).doExport() line: 245	
ServiceBean<T>(ServiceConfig<T>).export() line: 144	
ServiceBean<T>.onApplicationEvent(ApplicationEvent) line: 109	
SimpleApplicationEventMulticaster$1.run() line: 78	
SyncTaskExecutor.execute(Runnable) line: 49	
SimpleApplicationEventMulticaster.multicastEvent(ApplicationEvent) line: 76	
ClassPathXmlApplicationContext(AbstractApplicationContext).publishEvent(ApplicationEvent) line: 274	
ClassPathXmlApplicationContext(AbstractApplicationContext).finishRefresh() line: 736	
ClassPathXmlApplicationContext(AbstractApplicationContext).refresh() line: 383	
ClassPathXmlApplicationContext.<init>(String[], boolean, ApplicationContext) line: 139	
ClassPathXmlApplicationContext.<init>(String[]) line: 93	
SpringContainer.start() line: 50	
Main.main(String[]) line: 80	
DemoProvider.main(String[]) line: 21	
```

#### Consumer
Provider初始化时会调用doRegister方法向注册中心发起注册。那么客户端又是怎么subscribe在注册中心订阅服务的呢？答案是服务消费者在初始化ConsumerConfig时会调用RegistryProtocol的refer方法进一步调用RegistryDirectory的subscribe方法最终调用ZookeeperRegistry的subscribe方法向注册中心订阅服务。
com.alibaba.dubbo.registry.support.FailBackRegistry的subscribe方法

``` java
ZookeeperRegistry.doSubscribe(URL, NotifyListener) line: 114	
ZookeeperRegistry(FailbackRegistry).subscribe(URL, NotifyListener) line: 189	
RegistryDirectory<T>.subscribe(URL) line: 133	
RegistryProtocol.doRefer(Cluster, Registry, Class<T>, URL) line: 271	
RegistryProtocol.refer(Class<T>, URL) line: 254	
ProtocolListenerWrapper.refer(Class<T>, URL) line: 63	
ProtocolFilterWrapper.refer(Class<T>, URL) line: 60	
Protocol$Adpative.refer(Class, URL) line: not available	
ReferenceBean<T>(ReferenceConfig<T>).createProxy(Map<String,String>) line: 392	
ReferenceBean<T>(ReferenceConfig<T>).init() line: 300	
ReferenceBean<T>(ReferenceConfig<T>).get() line: 138	
ReferenceBean<T>.getObject() line: 65	
FactoryBeanRegistrySupport$1.run() line: 121	
AccessController.doPrivileged(PrivilegedAction<T>, AccessControlContext) line: not available [native method]	
DefaultListableBeanFactory(FactoryBeanRegistrySupport).doGetObjectFromFactoryBean(FactoryBean, String, boolean) line: 116	
DefaultListableBeanFactory(FactoryBeanRegistrySupport).getObjectFromFactoryBean(FactoryBean, String, boolean) line: 91	
DefaultListableBeanFactory(AbstractBeanFactory).getObjectForBeanInstance(Object, String, String, RootBeanDefinition) line: 1288	
DefaultListableBeanFactory(AbstractBeanFactory).doGetBean(String, Class, Object[], boolean) line: 275	
DefaultListableBeanFactory(AbstractBeanFactory).getBean(String, Class, Object[]) line: 185	
DefaultListableBeanFactory(AbstractBeanFactory).getBean(String) line: 164	
BeanDefinitionValueResolver.resolveReference(Object, RuntimeBeanReference) line: 269	
BeanDefinitionValueResolver.resolveValueIfNecessary(Object, Object) line: 104	
DefaultListableBeanFactory(AbstractAutowireCapableBeanFactory).applyPropertyValues(String, BeanDefinition, BeanWrapper, PropertyValues) line: 1245	
DefaultListableBeanFactory(AbstractAutowireCapableBeanFactory).populateBean(String, AbstractBeanDefinition, BeanWrapper) line: 1010	
DefaultListableBeanFactory(AbstractAutowireCapableBeanFactory).doCreateBean(String, RootBeanDefinition, Object[]) line: 472	
AbstractAutowireCapableBeanFactory$1.run() line: 409	
AccessController.doPrivileged(PrivilegedAction<T>, AccessControlContext) line: not available [native method]	
DefaultListableBeanFactory(AbstractAutowireCapableBeanFactory).createBean(String, RootBeanDefinition, Object[]) line: 380	
AbstractBeanFactory$1.getObject() line: 264	
DefaultListableBeanFactory(DefaultSingletonBeanRegistry).getSingleton(String, ObjectFactory) line: 222	
DefaultListableBeanFactory(AbstractBeanFactory).doGetBean(String, Class, Object[], boolean) line: 261	
DefaultListableBeanFactory(AbstractBeanFactory).getBean(String, Class, Object[]) line: 185	
DefaultListableBeanFactory(AbstractBeanFactory).getBean(String) line: 164	
DefaultListableBeanFactory.preInstantiateSingletons() line: 429	
ClassPathXmlApplicationContext(AbstractApplicationContext).finishBeanFactoryInitialization(ConfigurableListableBeanFactory) line: 728	
ClassPathXmlApplicationContext(AbstractApplicationContext).refresh() line: 380	
ClassPathXmlApplicationContext.<init>(String[], boolean, ApplicationContext) line: 139	
ClassPathXmlApplicationContext.<init>(String[]) line: 93	
SpringContainer.start() line: 50	
Main.main(String[]) line: 80	
DemoConsumer.main(String[]) line: 21	
```

## Dubbo一些逻辑
Dubbo分为注册中心、服务提供者(provider)、服务消费者(consumer)三个部分。


### Alibaba Dubbo框架同步调用原理分析-1 - sun - 学无止境
[Source](http://sunjun041640.blog.163.com/blog/static/256268322011111874633997/)

Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。

#### Dubbo缺省协议，使用基于mina1.1.7+hessian3.2.1的tbremoting交互。
连接个数：单连接
连接方式：长连接
传输协议：TCP
传输方式：NIO异步传输
序列化：Hessian二进制序列化
适用范围：传入传出参数数据包较小(建议小于100K)，消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用dubbo协议传输大文件或超大字符串。
适用场景：常规远程服务方法调用

#### 通常，一个典型的同步远程调用应该是这样的：
1， 客户端线程调用远程接口，向服务端发送请求，同时当前线程应该处于“暂停“状态，即线程不能向后执行了，必需要拿到服务端给自己的结果后才能向后执行
2， 服务端接到客户端请求后，处理请求，将结果给客户端
3， 客户端收到结果，然后当前线程继续往后执行

Dubbo里使用到了Socket(采用Apache mina框架做底层调用)来建立长连接，发送、接收数据，底层使用Apache mina框架的IoSession进行发送消息。

查看Dubbo文档及源代码可知，Dubbo底层使用Socket发送消息的形式进行数据传递，结合了mina框架，使用IoSession.write()方法，这个方法调用后对于整个远程调用(从发出请求到接收到结果)来说是一个异步的，即对于当前线程来说，将请求发送出来，线程就可以往后执行了，至于服务端的结果，是服务端处理完成后，再以消息的形式发送给客户端的。

#### 于是这里出现了2个问题：
1. 当前线程怎么让它“暂停”，等结果回来后，再向后执行？
2. 正如前面所说，Socket通信是一个全双工的方式，如果有多个线程同时进行远程方法调用，这时建立在client server之间的socket连接上会有很多双方发送的消息传递，前后顺序也可能是乱七八糟的，server处理完结果后，将结果消息发送给client，client收到很多消息，怎么知道哪个消息结果是原先哪个线程调用的？

#### 分析源代码，基本原理如下：
1. client一个线程调用远程接口，生成一个唯一的ID(比如一段随机字符串，UUID等)，Dubbo是使用AtomicLong从0开始累计数字的
2. 将打包的方法调用信息(如调用的接口名称，方法名称，参数值列表等)，和处理结果的回调对象callback，全部封装在一起，组成一个对象object
3. 向专门存放调用信息的全局ConcurrentHashMap里面put(ID, object)
4. 将ID和打包的方法调用信息封装成一对象connRequest，使用IoSession.write(connRequest)异步发送出去
5. 当前线程再使用callback的get()方法试图获取远程返回的结果，在get()内部，则使用synchronized获取回调对象callback的锁， 再先检测是否已经获取到结果，如果没有，然后调用callback的wait()方法，释放callback上的锁，让当前线程处于等待状态。
6. 服务端接收到请求并处理后，将结果(此结果中包含了前面的ID，即回传)发送给客户端，客户端socket连接上专门监听消息的线程收到消息，分析结果，取到ID，再从前面的ConcurrentHashMap里面get(ID)，从而找到callback，将方法调用结果设置到callback对象里
7. 监听线程接着使用synchronized获取回调对象callback的锁(因为前面调用过wait()，那个线程已释放callback的锁了)，再notifyAll()，唤醒前面处于等待状态的线程继续执行(callback的get()方法继续执行就能拿到调用结果了)，至此，整个过程结束。

这里还需要画一个大图来描述，后面再补了
需要注意的是，这里的callback对象是每次调用产生一个新的，不能共享，否则会有问题；另外ID必需至少保证在一个Socket连接里面是唯一的。

#### 现在，前面两个问题已经有答案了，
1. 当前线程怎么让它“暂停”，等结果回来后，再向后执行？
 答：先生成一个对象obj，在一个全局map里put(ID,obj)存放起来，再用synchronized获取obj锁，再调用obj.wait()让当前线程处于等待状态，然后另一消息监听线程等到服务端结果来了后，再map.get(ID)找到obj，再用synchronized获取obj锁，再调用obj.notifyAll()唤醒前面处于等待状态的线程。

2. 正如前面所说，Socket通信是一个全双工的方式，如果有多个线程同时进行远程方法调用，这时建立在client server之间的socket连接上会有很多双方发送的消息传递，前后顺序也可能是乱七八糟的，server处理完结果后，将结果消息发送给client，client收到很多消息，怎么知道哪个消息结果是原先哪个线程调用的？
 答：使用一个ID，让其唯一，然后传递给服务端，再服务端又回传回来，这样就知道结果是原先哪个线程的了。


### Dubbo整体架构

#### 1、Dubbo与Spring的整合
Dubbo在使用上可以做到非常简单，不管是Provider还是Consumer都可以通过Spring的配置文件进行配置，配置完之后，就可以像使用springbean一样进行服务暴露和调用了，完全看不到dubboapi的存在。这是因为dubbo使用了spring提供的可扩展Schema自定义配置支持。在spring配置文件中，可以像、这样进行配置。META-INF下的spring.handlers文件中指定了dubbo的xml解析类：DubboNamespaceHandler。像前面的被解析成ServiceConfig，被解析成ReferenceConfig等等。

#### 2、jdkspi扩展
由于Dubbo是开源框架，必须要提供很多的可扩展点。Dubbo是通过扩展jdkspi机制来实现可扩展的。具体来说，就是在META-INF目录下，放置文件名为接口全称，文件中为key、value键值对，value为具体实现类的全类名，key为标志值。由于dubbo使用了url总线的设计，即很多参数通过URL对象来传递，在实际中，具体要用到哪个值，可以通过url中的参数值来指定。
Dubbo对spi的扩展是通过ExtensionLoader来实现的，查看ExtensionLoader的源码，可以看到Dubbo对jdkspi做了三个方面的扩展：
(1)jdkspi仅仅通过接口类名获取所有实现，而ExtensionLoader则通过接口类名和key值获取一个实现；
(2)Adaptive实现，就是生成一个代理类，这样就可以根据实际调用时的一些参数动态决定要调用的类了。
(3)自动包装实现，这种实现的类一般是自动激活的，常用于包装类，比如Protocol的两个实现类：ProtocolFilterWrapper、ProtocolListenerWrapper。

#### 3、url总线设计
Dubbo为了使得各层解耦，采用了url总线的设计。我们通常的设计会把层与层之间的交互参数做成Model，这样层与层之间沟通成本比较大，扩展起来也比较麻烦。因此，Dubbo把各层之间的通信都采用url的形式。比如，注册中心启动时，参数的url为：
registry://0.0.0.0:9090?codec=registry&transporter=netty
这就表示当前是注册中心，绑定到所有ip，端口是9090，解析器类型是registry，使用的底层网络通信框架是netty。

### 1、注册中心启动过程
注册中心的启动过程，主要看两个类：RegistrySynchronizer、RegistryReceiver，两个类的初始化方法都是start。
RegistrySynchronizer的start方法：
(1)把所有配置信息load到内存；
(2)把当前注册中心信息保存到数据库；
(3)启动5个定时器。
5个定时器的功能是：
(1)AutoRedirectTask，自动重定向定时器。默认1小时运行1次。如果当前注册中心的连接数高于平均值的1.2倍，则将多出来的连接数重定向到其他注册中心上，以达到注册中心集群的连接数均衡。
(2)DirtyCheckTask，脏数据检查定时器。作用是：分别检查缓存provider、数据库provider、缓存consumer、数据库consumer的数据，清除脏数据；清理不存活的provider和consumer数据；对于缓存中的存在的provider或consumer而数据库不存在，重新注册和订阅。
(3)ChangedClearTask，changes变更表的定时清理任务。作用是读取changes表，清除过期数据。
(4)AlivedCheckTask，注册中心存活状态定时检查，会定时更新registries表的expire字段，用以判断注册中心的存活状态。如果有新的注册中心，发送同步消息，将当前所有注册中心的地址通知到所有客户端。
(5)ChangedCheckTask，变更检查定时器。检查changes表的变更，检查类型包括：参数覆盖变更、路由变更、服务消费者变更、权重变更、负载均衡变更。
RegistryReceiver的start方法：启动注册中心服务。默认使用netty框架，绑定本机的9090端口。最后启动服务的过程是在NettyServer来完成的。接收消息时，抛开dubbo协议的解码器，调用类的顺序是
1 NettyHandler-》NettyServer-》MultiMessageHandler-》HeartbeatHandler-》AllDispatcher-》
2 DecodeHandler-》HeaderExchangeHandler-》RegistryReceiver-》RegistryValidator-》RegistryFailover-》RegistryExecutor。

### 2、provider启动过程
provider的启动过程是从ServiceConfig的export方法开始进行的，具体步骤是：
(1)进行本地jvm的暴露，不开放任何端口，以提供injvm这种形式的调用，这种调用只是本地调用，不涉及进程间通信。
(2)调用RegistryProtocol的export。
(3)调用DubboProtocol的export，默认开启20880端口，用以提供接收consumer的远程调用服务。
(4)通过新建RemoteRegistry来建立与注册中心的连接。
(5)将服务地址注册到注册中心。
(6)去注册中心订阅自己的服务。

### 3、consumer启动过程
consumer的启动过程是通过ReferenceConfig的get方法进行的，具体步骤是：
(1)通过新建RemoteRegistry来建立与注册中心的连接。
(2)新建RegistryDirectory并向注册中心订阅服务，RegistryDirectory用以维护注册中心获取的服务相关信息。
(3)创建代理类，发起consumer远程调用时，实际调用的是InvokerInvocationHandler。

### 实际调用过程
1、consumer端发起调用时，实际调用经过的类是：
InvokerInvocationHandler-》MockClusterInvoker(如果配置了Mock，则直接调用本地Mock类)-》FailoverClusterInvoker(负载均衡，容错机制，默认在发生错误的情况下，进行两次重试)-》RegistryDirectory$InvokerDelegete-》ConsumerContextFilter-》FutureFilter-&gt;DubboInvoker
2、provider:
NettyServer-》MultiMessageHandler-》HeartbeatHandler-》AllDispatcher-》DecodeHandler-》HeaderExchangeHandler-》DubboProtocol.requestHandler-》EchoFilter-》ClassLoaderFilter-》GenericFilter-》ContextFilter-》ExceptionFilter-》TimeoutFilter-》MonitorFilter-》TraceFilter-》实际service。

### Dubbo使用的设计模式

1、工厂模式
ServiceConfig中有个字段，代码是这样的：
```
private  static  final  Protocol protocol = ExtensionLoader.getExtensionLoader(Protocol. class ).getAdaptiveExtension();
```
Dubbo里有很多这种代码。这也是一种工厂模式，只是实现类的获取采用了jdkspi的机制。这么实现的优点是可扩展性强，想要扩展实现，只需要在classpath下增加个文件就可以了，代码零侵入。另外，像上面的Adaptive实现，可以做到调用时动态决定调用哪个实现，但是由于这种实现采用了动态代理，会造成代码调试比较麻烦，需要分析出实际调用的实现类。

2、装饰器模式
Dubbo在启动和调用阶段都大量使用了装饰器模式。以Provider提供的调用链为例，具体的调用链代码是在ProtocolFilterWrapper的buildInvokerChain完成的，具体是将注解中含有group=provider的Filter实现，按照order排序，最后的调用顺序是
```
EchoFilter-》ClassLoaderFilter-》GenericFilter-》ContextFilter-》ExceptionFilter-》
TimeoutFilter-》MonitorFilter-》TraceFilter。
```
更确切地说，这里是装饰器和责任链模式的混合使用。例如，EchoFilter的作用是判断是否是回声测试请求，是的话直接返回内容，这是一种责任链的体现。而像ClassLoaderFilter则只是在主功能上添加了功能，更改当前线程的ClassLoader，这是典型的装饰器模式。

3、观察者模式
Dubbo的provider启动时，需要与注册中心交互，先注册自己的服务，再订阅自己的服务，订阅时，采用了观察者模式，开启一个listener。注册中心会每5秒定时检查是否有服务更新，如果有更新，向该服务的提供者发送一个notify消息，provider接受到notify消息后，即运行NotifyListener的notify方法，执行监听器方法。

4、动态代理模式
Dubbo扩展jdkspi的类ExtensionLoader的Adaptive实现是典型的动态代理实现。Dubbo需要灵活地控制实现类，即在调用阶段动态地根据参数决定调用哪个实现类，所以采用先生成代理类的方法，能够做到灵活的调用。生成代理类的代码是ExtensionLoader的createAdaptiveExtensionClassCode方法。代理类的主要逻辑是，获取URL参数中指定参数的值作为获取实现类的key。

#### Dubbo Main启动方式浅析
服务容器是一个standalone的启动程序，因为后台服务不需要Tomcat或JBoss等Web容器的功能，如果硬要用Web容器去加载服务提供方，增加复杂性，也浪费资源。
服务容器只是一个简单的Main方法，并加载一个简单的Spring容器，用于暴露服务。
服务容器的加载内容可以扩展，内置了spring, jetty, log4j等加载，可通过Container扩展点进行扩展，参见：Container
Spring Container
    自动加载META-INF/spring目录下的所有Spring配置。
    配置：(配在java命令-D参数或者dubbo.properties中)
        dubbo.spring.config=classpath*:META-INF/spring/*.xml ----配置spring配置加载位置

Jetty Container
    启动一个内嵌Jetty，用于汇报状态。
    配置：(配在java命令-D参数或者dubbo.properties中)
        dubbo.jetty.port=8080 ----配置jetty启动端口
        dubbo.jetty.directory=/foo/bar ----配置可通过jetty直接访问的目录，用于存放静态文件
        dubbo.jetty.page=log,status,system ----配置显示的页面，缺省加载所有页面

Log4j Container
    自动配置log4j的配置，在多进程启动时，自动给日志文件按进程分目录。
    配置：(配在java命令-D参数或者dubbo.properties中)
        dubbo.log4j.file=/foo/bar.log ----配置日志文件路径
        dubbo.log4j.level=WARN ----配置日志级别
        dubbo.log4j.subdirectory=20880 ----配置日志子目录，用于多进程启动，避免冲突

容器启动

如：(缺省只加载spring)
java com.alibaba.dubbo.container.Main

或：(通过main函数参数传入要加载的容器)
java com.alibaba.dubbo.container.Main spring jetty log4j

或：(通过JVM启动参数传入要加载的容器)
java com.alibaba.dubbo.container.Main -Ddubbo.container=spring,jetty,log4j

或：(通过classpath下的dubbo.properties配置传入要加载的容器)
dubbo.properties
dubbo.container=spring,jetty,log4j 

## 分布式服务框架远程服务通讯介绍 
[Hessian 原理分析](http://blog.sina.com.cn/s/blog_56fd58ab0100mrl6.html)
[java 几种远程服务调用协议的比较](http://www.cnblogs.com/jifeng/archive/2011/07/20/2111183.html)
在分布式服务框架中，一个最基础的问题就是远程服务是怎么通讯的，在Java领域中有很多可实现远程通讯 的技术，例如：RMI、MINA、ESB、Burlap、Hessian、SOAP、EJB和JMS等
那么在了解这些远程通讯的框架或library时，会带着什么问题去学 习呢？
1、是基于什么协议实现的？
2、怎么发起请求？
3、怎么将请求转化为符合协议的格式的？
4、使用什么传输协议传 输？
5、响应端基于什么机制来接收请求？
6、怎么将流还原为传输格式的？
7、处理完毕后怎么回应？