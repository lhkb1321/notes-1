# Java Notes

## Recent

锁降级
JEP draft: Concurrent Monitor Deflation http://openjdk.java.net/jeps/8183909
不可不说的Java“锁”事 https://tech.meituan.com/2018/11/15/java-lock.html
Java偏向锁是如何撤销的？ https://www.zhihu.com/question/57774162
Java Language Specification Chapter 17. Threads and Locks https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html
Synchronization and Object Locking https://wiki.openjdk.java.net/display/HotSpot/Synchronization

对象的内置锁? 每个对象都有一个内置锁?
`java.util.concurrent.CopyOnWriteArrayList`
`AtomicInteger`底层实现机制
SpringBoot和Swagger结合提高API开发效率  [URL](http://localhost:8080/swagger-ui.html)

concurrent: 主内存.寄存器是是运行时?

并发Concurrent 并发指能够让多个任务在逻辑上交织执行的程序设计
          --  --  --
        /              \
>---- --    --  --  -- ---->>

并行Parallel 并行指物理上同时执行
     ------
    /      \
>-------------->>

### 常用参数

`-verbose:gc -Xloggc:/path/to/gc.pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps`  
`-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path/to/hprof -XX:ErrorFile=/path/to/hs_err_pid%p.log`
-XX:+Pringflagsfinal 打印平台默认值  
[HotSpot VM Command-Line Options](https://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/clopts.html)  

### [Arthas](https://github.com/alibaba/arthas)

[Quick Start](https://alibaba.github.io/arthas/en/quick-start.html)
download: `wget https://alibaba.github.io/arthas/arthas-boot.jar`
startup: `java -jar arthas-boot.jar`

[watch](https://alibaba.github.io/arthas/en/quick-start.html#watch)
watch Demo$Counter method {params,returnObj}
watch com.company.EnvironmentEnum isAbroad {params,returnObj} -x 3
watch com.company.Service method {params,returnObj} "params[0]==1 && params[1]==new java.sql.Timestamp(1572424327000L)" -x 3

[trace](https://alibaba.github.io/arthas/trace.html) 方法内部调用路径，并输出方法路径上的每个节点上耗时
trace Demo$Counter getFactoryInfo #cost>10 -n 1
`#cost > 10` 只会展示耗时大于10ms的调用路径
`-n` 参数指定捕捉结果的次数

`trace -E com.test.ClassA|org.test.ClassB method1|method2|method3` 用正则表匹配路径上的多个类和函数，一定程度上达到多层trace的效果

sc Demo$Counter

[Arthas的一些特殊用法文档说明 #71](https://github.com/alibaba/arthas/issues/71)

#### Demo

[阿里巴巴问题排查神器Arthas使用实践](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650122813&idx=1&sn=3e419623e06cfc7900929bb6e88bcd24&chksm=f36bb71cc41c3e0a4fb7420498839af553db3073a4e02c5c67eca8d6e0f5d1f916673cb41d37&mpshare=1&scene=23&srcid=1216OroF3DROBapRZwxoMpkF#rd)

#### 原理

attach：jdk1.6新增功能，通过attach机制，可以在jvm运行中，通过pid关联应用

instrument：jdk1.5新增功能，通过instrument俗称javaagent技术，可以修改jvm加载的字节码

然后 arthas 和其他诊断工具一样，都是先通过attach链接上目标应用，通过instrument动态修改应用程序的字节码达到不重启应用而监控应用的目的

## Java 8

method reference :: syntax (meaning “use this method as a value”
`File[] hiddenFiles = new File(".").listFiles(File::isHidden);`

## Java问题排查工具箱

### java.net.SocketException 异常

[understanding_some_common_socketexceptions_in_java3](https://www.ibm.com/developerworks/community/blogs/738b7897-cd38-4f24-9f05-48dd69116837/entry/understanding_some_common_socketexceptions_in_java3?lang=en)  
[java.net.SocketException 异常](http://developer.51cto.com/art/201003/189724.htm)  

1. java.net.SocketException: Broken pipe (UNIX)
 A broken pipe error is seen when the remote end of the connection is closed gracefully.
Solution: This exception usually arises when the socket operations performed on either ends are not sync'ed.
2. java.net.SocketException: reset by peer.  This error happens on server side
3. java.net.SocketException: Connection reset. This error happens on client side
 This exception appears when the remote connection is unexpectedly and forcefully closed due to various reasons like application crash, system reboot, hard close of remote host. Kernel from the remote system sends out a packets with RST bit to the local system. The local socket on performing any SEND (could be a Keep-alive packet) or RECEIVE operations subsequently fail with this error. Certain combinations of linger settings can also result in packets with RST bit set.

### [高CPU占用分析](http://www.blogjava.net/hankchen/archive/2012/05/09/377735.html)

一个应用占用CPU很高，除了确实是计算密集型应用之外，通常原因都是出现了死循环  

1. `top -H`
2. 找到具体是CPU高占用的线程 `ps -mp <PID> -o THREAD,tid,time,rss,size,%mem`
3. 将需要的线程ID转换为16进制格式 `printf "%x\n" tid`
4. 打印线程的堆栈信息 `jstack PID | grep tid -A 30`  

#### checklist from linux to application process

1. `top` 看出pid为 12666 的java进程占用了较多的cpu资源
2. `top -Hp 12666` 查看该进程下各线程的CPU资源, 可以找到占资源较多的线程pid 为 12666 (12666 用 16 进制表示为 0x321e)
3. `jstack 12666 | grep nid=0x321e` 查看当前java进程的堆栈状态

### 内存检查步骤

查看java线程在内存增长时线程数 `jstack PID | grep 'java.lang.Thread.State' | wc -l` 或者 `cat /proc/pid/status | grep Thread`

用pmap查看进程内的内存 `RSS` 情况，观察java的heap和stack大小 `pmap -x pid |less`

使用gdb观察内存块里的内容，发现里面有一些接口的返回值、mc的返回值、还有一些类名等等 `gdb: dump memory /tmp/memory.bin 0x7f6b38000000 0x7f6b38000000+65535000`
`hexdump -C /tmp/memory.bin` 或 `strings /tmp/memory.bin |less`

用strace和ltrace查找malloc调用

#### Native Memory Tracking (NMT)

* java8 给HotSpot VM引入了 NMT 特性，可以用于追踪JVM的内部内存使用
* Enable NMT `-XX:NativeMemoryTracking=[off | summary | detail]`
* Use jcmd to Access NMT Data `jcmd <pid> VM.native_memory [summary | detail | baseline | summary.diff | detail.diff | shutdown] [scale= KB | MB | GB]`
* [Oracle Technology Network - Native Memory Tracking](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/nmt-8.html)
* [Java Platform, Standard Edition Troubleshooting Guide - 2.7 Native Memory Tracking](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html)

* 使用`-XX:NativeMemoryTracking=summary`可以用于开启NMT，其中该值默认为off，可以设置summary、detail来开启；开启的话，大概会增加5%-10%的性能消耗；使用-XX:+UnlockDiagnosticVMOptions -XX:+PrintNMTStatistics可以在jvm shutdown的时候输出整体的native memory统计；其他的可以使用`jcmd pid VM.native_memory`相关命令进行查看、diff、shutdown等
* 整个memory主要包含了Java Heap、Class、Thread、Code、GC、Compiler、Internal、Other、Symbol、Native Memory Tracking、Arena Chunk这几部分；其中reserved表示应用可用的内存大小，committed表示应用正在使用的内存大小
* [聊聊HotSpot VM的Native Memory Tracking](https://cloud.tencent.com/developer/article/1406522)

#### jemalloc 查看堆外内存 anon

[native-jvm-leaks](https://github.com/jeffgriffith/native-jvm-leaks )  
[Use Case: Leak Checking](https://github.com/jemalloc/jemalloc/wiki/Use-Case:-Leak-Checking )  
[Debugging Java Native Memory Leaks](http://www.evanjones.ca/java-native-leak-bug.html )  
[Using jemalloc to get to the bottom of a memory leak](https://gdstechnology.blog.gov.uk/2015/12/11/using-jemalloc-to-get-to-the-bottom-of-a-memory-leak/ )

1. Starting your JVM with jemalloc `export LD_PRELOAD=/usr/local/lib/libjemalloc.so`
2. Configuring the profiler `export MALLOC_CONF=prof:true,lg_prof_interval:30,lg_prof_sample:17,prof_prefix:/path/to/jeprof/output/jeprof`
3. Run your program
4. Finding the needle `jeprof --show_bytes --gif /usr/bin/java /path/to/jeprof/output/jeprof.*.heap > out.gif`

#### gperf 查看堆外内存

[Work with Google performance tools](http://alexott.net/en/writings/prog-checking/GooglePT.html )  
[perftools查看堆外内存并解决hbase内存溢出](http://koven2049.iteye.com/blog/1142768 )  
[Gperftools Heap Leak Checker](https://gperftools.github.io/gperftools/heap_checker.html )  

1. `export GPERF_HOME=/path/to/gperftools-2.7`
2. `export LD_PRELOAD=$GPERF_HOME/lib/libtcmalloc.so HEAPCHECK=normal`
3. `export HEAPPROFILE=/path/to/gperf/output.heap`
4. Run program `$GPERF_HOME/bin/pprof --text /usr/bin/java $HEAPPROFILE.*.heap > gperf.output.text`

#### Valgrind

[The Valgrind Quick Start Guide](http://valgrind.org/docs/manual/quick-start.html )  
`valgrind ls -l >& valgrind.log`
`valgrind --leak-check=yes myprog arg1 arg2`

#### perf

[perf Examples](http://www.brendangregg.com/perf.html )
[brendangregg/perf-tools](https://github.com/brendangregg/perf-tools )

1. `perf mem record sh jni.sh 1000000 10 leak`  or attach a running process with `sudo perf record -g -p <PID>`
2. `perf mem report`

### CPU相关工具 bluedavy

[Java问题排查工具箱](https://mp.weixin.qq.com/s?__biz=MjM5MzYzMzkyMQ==&mid=2649826312&idx=1&sn=d28b3c91ef25a281256c6ccd2fafe0d3&mpshare=1&scene=23&srcid=1031nCrOjtP6QtlUYAL6QWso#rd )

碰到一些CPU相关的问题时，通常需要用到的工具：

`top (-H)` top可以实时的观察cpu的指标状况，尤其是每个core的指标状况，可以更有效的来帮助解决问题，-H则有助于看是什么线程造成的CPU消耗，这对解决一些简单的耗CPU的问题会有很大帮助。  

`sar` sar有助于查看历史指标数据，除了CPU外，其他内存，磁盘，网络等等各种指标都可以查看，毕竟大部分时候问题都发生在过去，所以翻历史记录非常重要。  

`jstack` jstack可以用来查看Java进程里的线程都在干什么，这通常对于应用没反应，非常慢等等场景都有不小的帮助，jstack默认只能看到Java栈，而jstack -m则可以看到线程的Java栈和native栈，但如果Java方法被编译过，则看不到（然而大部分经常访问的Java方法其实都被编译过）。  

`pstack` pstack可以用来看Java进程的native栈。  

`perf` 一些简单的CPU消耗的问题靠着 `top -H` + `jstack` 通常能解决，复杂的话就需要借助perf这种超级利器了。  

`cat /proc/interrupts` 之所以提这个是因为对于分布式应用而言，频繁的网络访问造成的网络中断处理消耗也是一个关键，而这个时候网卡的多队列以及均衡就非常重要了，所以如果观察到cpu的si指标不低，那么看看interrupts就有必要了。

### 内存相关工具 bluedavy

碰到一些内存相关的问题时，通常需要用到的工具：  

`jstat` `jstat -gcutil`或`-gc`等等有助于实时看gc的状况，不过我还是比较习惯看gc log。  

`jmap` 在需要dump内存看看内存里都是什么的时候，`jmap -dump`可以帮助你；在需要强制执行fgc的时候（在CMS GC这种一定会产生碎片化的GC中，总是会找到这样的理由的），`jmap -histo:live`可以帮助你（显然，不要随便执行）。

`gcore` 相比`jmap -dump`，其实我更喜欢gcore，因为感觉就是更快，不过由于某些jdk版本貌似和gcore配合的不是那么好，所以那种时候还是要用jmap -dump的。  

`mat` 有了内存dump后，没有分析工具的话然并卵

`btrace` 少数的问题可以mat后直接看出，而多数会需要再用btrace去动态跟踪，btrace绝对是Java中的超级神器，举个简单例子，如果要你去查下一个运行的Java应用，哪里在创建一个数组大小>1000的ArrayList，你要怎么办呢，在有btrace的情况下，那就是秒秒钟搞定的事，:)  

`gperf` Java堆内的内存消耗用上面的一些工具基本能搞定，但堆外就悲催了，目前看起来还是只有gperf还算是比较好用的一个，或者从经验上来说Direct ByteBuffer、Deflater/Inflater这些是常见问题。  

除了上面的工具外，同样内存信息的记录也非常重要，就如日志一样，所以像GC日志是一定要打开的，确保在出问题后可以翻查GC日志来对照是否GC有问题，所以像 `-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:<gc log file>` 这样的参数必须是启动参数的标配。

#### Eclipse Memory Analyzer Tool (mat)

**Dominate**: An object x dominates an object y if every path in the object graph from the start (or the root) node to y must go through x.

**Dominator Tree** : A dominator tree is built out of the object graph. In the dominator tree each object is the immediate dominator of its children, so dependencies between the objects are easily identified.

**Shallow heap** is the memory consumed by one object.

**Retained set** of X is the set of objects which would be removed by GC when X is garbage collected.

**Retained heap** of X is the sum of shallow sizes of all objects in the retained set of X, i.e. memory kept alive by X.

### jvm log 时间格式

打印绝对时间 `-XX:+PrintGCDetails -XX:+PrintGCDateStamps`  
打印相对时间 `-XX:+PrintGCDetails -XX:+PrintGCTimeStamps`  
`-Xloggc` 需要使用绝对路径  
`-verbose:gc -Xloggc:/path/to/gc.pid%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps`  
`-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path/to/hprof -XX:ErrorFile=/path/to/hs_err_pid%p.log`  
[Fatal Error Log](http://www.oracle.com/technetwork/java/javase/felog-138657.html#gbwcy)  

### ClassLoader相关工具

作为Java程序员，不碰到ClassLoader问题那基本是不可能的，在排查此类问题时，最好办的还是`-XX:+TraceClassLoading`，或者如果知道是什么类的话，我的建议就是把所有会装载的lib目录里的jar用`jar -tvf *.jar`这样的方式来直接查看冲突的class，再不行的话就要呼唤btrace神器去跟踪Classloader.defineClass之类的了。  

### 其他工具

`jinfo` Java有N多的启动参数，N多的默认值，而任何文档都不一定准确，只有用jinfo -flags看到的才靠谱，甚至你还可以看看jinfo -flag，你会发现更好玩的。  

`dmesg` 你的java进程突然不见了？ 也许可以试试dmesg先看看。  

`systemtap` 有些问题排查到java层面是不够的，当需要trace更底层的os层面的函数调用的时候，systemtap神器就可以派上用场了。  

`gdb` 更高级的玩家们，拿着core dump可以用gdb来排查更诡异的一些问题。  

## Java Mermory check

### 虚拟机监控工具

[Troubleshooting Guide for Java SE 6 with HotSpot VM](http://www.oracle.com/technetwork/java/javase/memleaks-137499.html#gdysp)
jps: 虚拟机进程状况工具  (Java Virtual Machine Process Status Tool)
jstat: 虚拟机统计信息工具  (Java Virtual Machine Statistics Monitoring Tool)
jinfo: Java配置信息工具  
jmap: Java内存映像工具  (Memory Map)
jhat: 虚拟机堆转储快照分析工具  (Java Heap Analysis Tool)
jstack: Java堆栈跟踪工具  (Stack Trace)
HSDIS: JIT生成代码反汇编  
`jcmd 4874 VM.command_line` 打印指定线程的启动参数  
HSDB: `java -cp sa-jdi.jar sun.jvm.hotspot.HSDB`  
[Serviceability in HotSpot](http://openjdk.java.net/groups/hotspot/docs/Serviceability.html)

#### jmap

排查GC问题必然会用到的工具，jmap可以告诉你当前JVM内存堆中的对象分布及其关系，当你dump堆之后可以用内存分析工具, 例如：Eclipse Memory Analyzer Tool（MAT）、VisualVM、jhat、jprofile等工具查看分析，看看有哪些大对象，或者哪些类的实例特别多。

常用用法：
强制FGC：-histo:live
dump堆：-dump:[live],format=b,file=dump.bin
`$(ps -ef | grep applicationName | grep -v grep | awk '{print $2}')` 获取pid  

查看各代内存占用情况：

* `jmap -heap [pid]`
* `jmap [pid]`
* `jmap -dump:format=b,file=/tmp/java_pid.hprof [pid]` 可以将当前Java进程的内存占用情况导出来
* `jmap -dump:live,format=b,file=/tmp/java_pid.hprof [pid]` 可以将当前Java进程存活对象在内存中占用情况导出来, `:live` 会触发一次Full GC  
* `jmap -histo:live [pid] >a.log` 查看当前Java进程创建的活跃对象数目和占用内存大小
* `jmap -histo $(ps -ef | grep applicationName | grep -v grep | awk '{print $2}') | head -20` top 20 内存占用  

浅堆（Shallow Heap）: 对象的浅堆指它在内存中的大小
保留堆（Retained Heap）: 指当特定对象被垃圾回收后即将释放的内存大小, 因为 保留堆=内存中的大小 + 持有对象的引用大小

#### jcmd

在JDK 1.7之后，新增了一个命令行工具jcmd。它是一个多功能工具，可以用来导出堆，查看java进程，导出线程信息，执行GC等。
jcmd拥有jmap的大部分功能，Oracle官方建议使用jcmd代替jmap

* `jcmd pid help` 列出该虚拟机支持的所有命令  

#### jstack

jstack 可以告诉你当前所有JVM线程正在做什么，包括用户线程和虚拟机线程，你可以用它来查看线程栈，并且结合Lock信息来检测是否发生了死锁和死锁的线程。  
另外在用top -H看到占用CPU非常高的pid时，可以转换成16进制后在jstack dump出来的文件中搜索，看看到底是什么线程占用了CPU。

#### jhat

`jhat -port 8080 /tmp/java_11211.hprof` 服务启动后，访问 [localhost](http://localhost:8080/)

#### jstat

可以告诉你当前的GC情况，包括GC次数、时间，具体的GC还可以结合gc.log文件去分析。  
`jstat -gc pid 250 10`  
`jstat -gcutil`  
根据JVM的内存布局  

* 堆内存 = 年轻代 + 年老代 + 永久代
* 年轻代 = Eden区 + 两个Survivor区（From和To）

以上统计数据各列的含义为  
    S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）  
    EC、EU：Eden区容量和使用量  
    OC、OU：年老代容量和使用量  
    PC、PU：永久代容量和使用量  
    YGC、YGT：年轻代GC次数和GC耗时  
    FGC、FGCT：Full GC次数和Full GC耗时  
    GCT：GC总耗时  

#### jVisualVM

[VisualVM CPU Sampling](http://greyfocus.com/2016/05/visualvm-sampling/)

##### Sampling

Sampling on the other side works by periodically retrieving thread dumps from the JVM. In this case, the performance impact is minor (and constant since the thread dumps are retrieved using a fixed frequency) and there’s no risk of introducing side effects. This process is a lot less intrusive and can also be performed quite reliably on remote applications (i.e. it could even be applied to production instances).

##### Profiling

Profiling involves instrumenting the entire application code or only some classes in order to provide runtime performance metrics to the profiler application. Since this involves changes to the application code, which are applied automatically by the profiler, it also means that there is a certain performance impact and risk of affecting the existing functionality.

##### Difference between “Self Time” and “Self Time (CPU)”

VisualVM reports two metrics related to the duration, but there is a significant difference between them:

* self time - counts the total time spent in that method, including the amount of time spent on locks or other blocking behaviour
* self time (cpu) - counts the total time spent in that method, excluding the amount of time the thread was blocked

#### [Interpretation of FieldType characters](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3)

[Z = boolean
[B = byte
[S = short
[I = int
[J = long
[F = float
[D = double
[C = char
[L = any non-primitives(Object)

### JVM内存分配

在Java虚拟机中，内存分为三个代：新生代（New）、老生代（Old）、永久代（Perm）。
（1）新生代New：新建的对象都存放这里
（2）老生代Old：存放从新生代New中迁移过来的生命周期较久的对象。新生代New和老生代Old共同组成了堆内存。
（3）永久代Perm：是非堆内存的组成部分。主要存放加载的Class类级对象如class本身，method，field等等。

* 堆内存 = 年轻代 + 年老代 + 永久代
* 年轻代 = Eden区 + 两个Survivor区（From和To）

如果出现java.lang.OutOfMemoryError: Java heap space异常，说明Java虚拟机的堆内存不够。原因有二：
（1）Java虚拟机的堆内存设置不够，可以通过参数-Xms1g、-Xmx2g来调整。
（2）代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）。

如果出现java.lang.OutOfMemoryError: PermGen space，说明是Java虚拟机对永久代Perm内存设置不够。
一般出现这种情况，都是程序启动需要加载大量的第三方jar包。例如：在一个Tomcat下部署了太多的应用。

从代码的角度，软件开发人员主要关注java.lang.OutOfMemoryError: Java heap space异常，减少不必要的对象创建，同时避免内存泄漏。

#### Guidelines for Calculating Java Heap Sizing

Refer to Java Performance

##### Table 7-3 Guidelines for Calculating Java Heap Sizing

Space                     | Command Line Option             | Occupancy Factor
----                      |---                              |---
Java heap                 | -Xms and -Xmx                   | 3x to 4x old generation space occupancy after full garbage collection
Permanent Generation      | -XX:PermSize -XX:MaxPermSize    | 1.2x to 1.5x permanent generation space occupancy after full garbage collection
Young Generation          | -Xmn 1x to 1.5x                 | old generation space occupancy after full garbage collection
Old Generation            | Implied from overall Java heap size minus the young generation size | 2x to 3x old generation space occupancy after full garbage collection

##### Refine Young Generation Size

* The old generation space size should be not be much smaller than 1.5x the live data size
* Young generation space size should be at least 10% of the Java heap size, the value specified as -Xmx and -Xms.
* When increasing the Java heap size, be careful not to exceed the amount of physical memory available to the JVM

### Other tools

#### [GCViewer](https://github.com/chewiebug/GCViewer)

GCViewer is a little tool that visualizes verbose GC output generated by Sun / Oracle, IBM, HP and BEA Java Virtual Machines.

#### [greys-anatomy](https://github.com/oldmanpushcart/greys-anatomy/wiki)

Greys是一个JVM进程执行过程中的异常诊断工具，可以在不中断程序执行的情况下轻松完成问题排查工作。

#### HouseMD

一个类似于BTrace的工具，用于对JVM运行时的状态进行追踪和诊断，作者是中间件团队的聚石。

通常我们排查问题很多时候都在代码中加个日志，看看方法的参数、返回值是不是我们期望的，然后编译打包部署重启应用，十几分钟就过去了。HouseMD可以直接让你可以追踪到方法的返回值和参数，以及调用次数、调用平均rt、调用栈。甚至是类的成员变量的值、Class加载的路径、对应的ClassLoader，都可以用一行命令给你展现出来，堪称神器。

更多的用法可以参考详细的[WiKi](https://github.com/CSUG/HouseMD)

再偷偷告诉你，因为HouseMD是基于字节码分析来做的，所以理论上运行在JVM的语言都可以用它，包括Groovy，Clojure都可以。

## JVM

摘自: 周志明 深入理解Java虚拟机-JVM高级特性与最佳实践

### 内存区域分析

运行时数据区分两部分  

1. 线程共享: 方法区(Method Area), 堆(Heap)  
2. 线程隔离: 虚拟机栈(JVM Stack), 本地方法栈(Native Method Stack), 程序计数器(Program Counter Register)

线程    |     名字         | 作用
--- |    ---         | ---
共享    |    Java堆        |    存放对象实例
共享    |    方法区        |    存储被虚拟机加载的类信息,常量,静态变量等.
隔离    |    程序计数器    |    当前线程所执行的字节码行号指示器,线程切换时可以快速切换位置
隔离    |    虚拟机栈        |    创建栈桢用于存储局部变量表,操作数栈,动态链接,方法出口等信息. 局部变量表
隔离    |    本地方法栈    |    同虚拟机栈

程序计数器存储两种: 1.Java方法,则保存虚拟机字节码指令地址; 2,Native方法,为空(Undefined)

#### 分区

##### 2.5.5 方法区 Method Area, 永生代

与Java堆一样,是各线程共享的内存区域,虽然Java虚拟机堆规范把方法区描述为堆的一个逻辑部分,但是它别名叫做Non-Heap(非堆),目的应该是与Java堆区分开. 在使用HotSpot时,很多人把方法区称为"永生代",本质上两者并不等价,仅仅是因为HotSpot虚拟机的设计团队把GC分代收集扩展至方法区,或者说使用永久代来实现方法区

在JDK8之前的HotSpot虚拟机中,类的"永久的"数据存放在永久代,通过‑XX:MaxPermSize 设置永久代的大小.它的垃圾回收和老年代的垃圾回收是绑定的,一旦其中一个被占满,则两个都要进行回收.但一旦元数据(类的层级信息,方法数据,方法信息如字节码,栈和变量大小,运行时常量池,已确定的符号引用和虚方法表)超过了永久代的大小,程序就会出现内存溢出OOM.

**JDK8移除了永生代**,类的元数据保存到了一个与堆不相连的本地内存区域--**元空间**,它的最大可分配空间就是系统可用的内存空间.  
元空间的内存管理: 每一个类加载器的存储区域都称作一个元空间,所有的元空间合在一起.当一个类加载器不再存活,其对应的元空间就会被回收.

#### 2.3 HotSpot虚拟机

##### 2.3.1　对象的创建

1. **指针碰撞 Bump the Pointer**:已使用的内存放一边,空闲的放另一边,建立新对象分配内存仅仅是把指针向空闲空间那边挪动一段与对象大小相等的距离
2. **空闲列表 Free List**: 已使用和空闲的内存相互交错,虚拟机必须维护一个列表,记录可用内存块,建立新对象分配内存时要从列表中找到一块足够大的空间划分给对象实例,并更新列表上的记录.

**分配空间的并发问题**:并发情况下,可能出现正在给对象A分配内存,指针还没来得及修改,对象B又同时使用了原来的指针来分配内存的情况.解决这个问题有两种方案:  
一种是对分配内存空间的动作进行同步处理--实际上虚拟机采用CAS配上失败重试的方式保证更新操作的原子性;  
另一种是把内存分配的动作按线程划分在不同的空间中进行,即每个线程在Java堆中预先分配一小块内存,称为本地线程分配缓冲(Thread Local Allocation Buffer, TLAB).

##### 2.3.2　对象内存布局

在HotSpot虚拟机中,对象在内存中存储的布局分３块区域:对象头(Header),实例数据(Instance Data)和对齐填充(Padding)  
对象头包括两部分

1. 存储对象自身的运行时数据,如HashCode,GC分代年龄,锁状态标志,线程持有的锁,偏向线程ID,偏向时间戳等.官方称为 Mark Word  
2. 类型指针,即对象指向它的类元数据的指针,虚拟机通过它来确定这个对象是哪个类的实例.

##### 2.3.3　对象访问定位

1. 使用句柄: Java堆中会划分出一块内存作为句柄池  
栈中的reference存储对象句柄地址,句柄包含对象实例数据与类型数据地址信息  
优点:在对象移动(垃圾回收)时,只会改变句柄中实例数据指针,而reference本身不需修改  
2.　使用直接指针访问:　reference存储的就是对象地址. HotSpot使用此种方式  
优点:访问速度快,节省一次指针定位的时间开销  

### 第3章 垃圾收集器与内存分配策略

* 哪些内存需要回收?
* 什么时候回收?
* 如何回收?

#### 3.2 判断对象是否在使用

##### 3.2.1 引用计数算法 Reference Counting

优点:简单  
缺点:难以解决循环引用

##### 3.2.2 可达性分析算法 Reachability Analysis

可作为GC Roots的对象包括:

* 虚拟机栈(栈桢中的本地变量表)中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法中JNI(即一般说的Native方法)引用的对象

#### 3.3 垃圾收集算法

##### 3.3.1 标记-清除算法 Mark-Sweep

首先标记出需要回收的对象,标记完成后统一回收所有被标记的对象.它是最基础的收集算法.

缺点:  
1.效率问题:标记和清除两个过程效率都不高;  
2.空间问题:会产生大量不连续的内存碎片,导致最后无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作.

##### 3.3.2 复制算法 Copying

将内存分为大小相等两块,每次只使用一块.当一块用完了,就将活着的对象复制到另外一块上,然后把已使用过的内存空间一次清理掉.  
现代商业虚拟机都采用这种算法:将内存分为一块较大的Eden空间,两块较小的Survivor空间,每次使用Eden和其中一块Survivor

优点:实现简单,运行高效  
缺点:内存缩小了一半

##### 3.3.3 标记整理算法 Mark-Compact

根据老年代特点,提出此算法,标记过程与"标记-清除"算法一样,但后续步骤不是直接清理,而是让所有存活对象都向一端移动,然后直接清理掉边界以外的内存

##### 3.3.4 分代收集 Generational Collection

根据对象存活周期不同,将内存分为几块:新生代和老年代,这样就根据各个年代特点采用最适当的收集算法.
在新生代使用复制算法,只需付出少量存活对象的复制成本就可以完成收集.
老年代中因为对象存活率高,没有额外空间对它进行分配担保,就必须使用"标记-清理"或者"标记-整理"算法

#### 3.5 垃圾收集器

**Young generation**: Serial, ParNew, Parallel Scavenge, G1(Garbage First)  
**Tenured generation**: CMS(Concurrent Mark Sweep), Serial Old(MSC), Parallel Old, G1(Garbage First)

![HostSpot垃圾回收器](image/HostSpot垃圾回收器.png "HostSpot垃圾回收器")

algorithm combinations cheat sheet

Young               |    Tenured     |    JVM options
---                 |    ---         |        ---
Incremental         |    Incremental |    -Xincgc
Serial              |    Serial      |    -XX:+UseSerialGC
Parallel Scavenge   |    Serial      |    -XX:+UseParallelGC -XX:-UseParallelOldGC
Parallel New        |    Serial      |    N/A
Serial              |    Parallel Old|     N/A
Parallel Scavenge   |    Parallel Old|    -XX:+UseParallelGC -XX:+UseParallelOldGC
Parallel New        |    Parallel Old|     N/A
Serial              |    CMS         |    -XX:-UseParNewGC -XX:+UseConcMarkSweepGC
Parallel Scavenge   |    CMS         |    N/A
Parallel New        |    CMS         |    -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
G1                  |                |    -XX:+UseG1GC

Note that this stands true for Java 8, for older Java versions the available combinations might differ a bit.  
The table is from [GC Algorithms: Implementations](https://plumbr.eu/handbook/garbage-collection-algorithms-implementations )

##### 3.5.1 Serial收集器

最基本最悠久的单线程收集器,在它进行时,必须暂停其他所有的工作线程,直到它结束. Stop the World.  
优点:简洁高效(与其他收集器的单线程比)  
应用场景:运行在Client模式下的默认新生代收集器

##### 3.5.2 ParNew收集器

是Serial收集器的多线程版本  
应用场景：运行在Server模式下的虚拟机中首选的新生代收集器  

很重要的原因是：除了Serial收集器外，目前只有它能与CMS收集器配合工作。不幸的是，CMS作为老年代的收集器，却无法与JDK 1.4.0中已经存在的新生代收集器Parallel Scavenge配合工作，所以在JDK 1.5中使用CMS来收集老年代的时候，新生代只能选择ParNew或者Serial收集器中的一个。

##### 3.5.3 Parallel Scavenge 收集器

是一个新生代收集器,使用复制算法的并行多线程收集器  
优点:GC自适应的调节策略(GC Ergonomics) 不需要手工指定新生代的大小、Eden与Survivor区的比例、晋升老年代对象年龄等细节参数

应用场景：停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验，而高吞吐量则可以高效率地利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

**Parallel Scavenge VS CMS收集器**
CMS收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间;  
Parallel Scavenge收集器的目标是达到一个可控制的吞吐量.吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)  
提供了两个参数用于精确控制吞吐量,分别是 控制最大垃圾收集停顿时间 和 直接设置吞吐量大小 的参数.

**Parallel Scavenge收集器 VS ParNew收集器**
Parallel Scavenge收集器与ParNew收集器的一个重要区别是它具有自适应调节策略。

##### 3.5.4 Serial Old 收集器

是Serial收集器的老年代版本,单线程,使用"标记-整理"算法

应用场景：

* Client模式: Serial Old收集器的主要意义也是在于给Client模式下的虚拟机使用。
* Server模式: 有两大用途：一种用途是在JDK 1.5以及之前的版本中与Parallel Scavenge收集器搭配使用，另一种用途就是作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用。

##### 3.5.5 Parallel Old 收集器

是Parallel Scavenge收集器的老年代版本,使用多线程和“标记－整理”算法。

应用场景：在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

##### 3.5.6 CMS(Concurrent Mark Sweep)收集器

以获取最短回收停顿时间为目标,基于"标记-清除"算法  过程

1. 初始标记(CMS initial mark):仅标记GC Roots能直接关联到的对象，速度很快，需要“Stop The World”  
2. 并发标记(CMS concurrent mark)  
3. 重新标记(CMS remark): 修正并发标记期间产生变动的对象的标记记录，停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短，仍然需要“Stop The World”  
4. 并发清除(CMS concurrent sweep)

优点： 并发收集、低停顿  
缺点

1. CMS收集器对CPU资源非常敏感  
2. 无法处理浮动垃圾(可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生)  
3. 产生大量空间碎片  

##### 3.5.7 G1(Garbage-First) 收集器

可以做到基本不牺牲吞吐率的前提下完成低停顿的回收工作  
面向服务端.特点如下

* 并行与并发
* 分代收集
* 空间整合:从整体看是基于"标记-整理",从局部看(两个Region之间)是基于"复制"算法,不会产生内存空间碎片
* 可预测的停顿:除了追求低停顿,还能建立可预测的停顿时间模型,能指定在垃圾收集上的时间不得超过N毫秒

就目前而言、CMS还是默认首选的GC策略、可能在以下场景下G1更适合：  

* 服务端多核CPU、JVM内存占用较大的应用（至少大于4G）
* 应用在运行过程中会产生大量内存碎片、需要经常压缩空间
* 想要更可控、可预期的GC停顿周期；防止高并发下应用雪崩现象

##### ZGC - JDK 11

[Z Garbage Collector](https://wiki.openjdk.java.net/display/zgc/Main)

goals
    Pause times do not exceed 10ms
    Pause times do not increase with the heap or live-set size
    Handle heaps ranging from a few hundred megabytes to multi terabytes in size

At a glance, ZGC is:

    Concurrent
    Region-based
    Compacting
    NUMA-aware
    Using colored pointers
    Using load barriers

江南白衣本衣 春天的旁边 [Java程序员的荣光，听R大论JDK11的ZGC](https://mp.weixin.qq.com/s/KUCs_BJUNfMMCO1T3_WAjw)  
> R大: 与标记对象的传统算法相比，ZGC在指针上做标记，在访问指针时加入Load Barrier（读屏障），比如当对象正被GC移动，指针上的颜色就会不对，这个屏障就会先把指针更新为有效地址再返回，也就是，永远只有单个对象读取时有概率被减速，而不存在为了保持应用与GC一致而粗暴整体的Stop The World。

ZGC的八大特征

1. 所有阶段几乎都是并发执行的
 这里的并发(Concurrent)，说的是应用线程与GC线程齐头并进，互不添堵。
说几乎，就是还有三个非常短暂的STW的阶段，所以ZGC并不是Zero Pause GC啦
2. 并发执行的保证机制，就是Colored Pointer 和 Load Barrier
 Colored Pointer 从64位的指针中，借了几位出来表示Finalizable、Remapped、Marked1、Marked0。 所以它不支持32位指针也不支持压缩指针， 且堆的上限是4TB
3. 像G1一样划分Region，但更加灵活
4. 和G1一样会做Compacting－压缩
 粗略了几十倍地过一波回收流程，小阶段都被略过了哈:
 4.1. Pause Mark Start －初始停顿标记
 4.2. Concurrent Mark －并发标记
 4.3. Relocate － 移动对象
 4.4. Remap － 修正指针
 上一个阶段的Remap，和下一个阶段的Mark是混搭在一起完成的，这样非常高效，省却了重复遍历对象图的开销。
5. 没有G1占内存的Remember Set，没有Write Barrier的开销
6. 支持Numa架构
 现在多CPU插槽的服务器都是Numa架构
7. 并行
8. 单代
 没分代，应该是ZGC唯一的弱点了. 所以R大说ZGC的水平，处于AZul早期的PauselessGC  与 分代的C4算法之间 － C4在代码里就叫GPGC，Generational Pauseless GC。
 分代原本是因为most object die young的假设，而让新生代和老生代使用不同的GC算法。但C4已经是全程并发算法了，为什么还要分代呢？
 R大说：因为分代的C4能承受的对象分配速度(Allocation Rate)， 大概是原始PGC的10倍。

##### 3.5.8 理解GC日志

    33.125: [GC [DefNew: 3324K->152K(3712K), 0.0025925 secs] 3324K->152K(11904K), 0.0031680 secs]
    100.667: [Full GC [Tenured: 0K->210K(10240K), 0.0149142 secs] 4603K->210K(19456K), [Perm : 2999K->2999K(21248K)], 0.0150007 secs] [Times: user=0.01 sys=0.00, real=0.02 secs]

最前面的数字"33.125："和"100.667："：代表了GC发生的时间，这个数字的含义是从Java虚拟机启动以来经过的秒数.

GC日志开头的"［GC"和"［Full GC"：说明了这次垃圾收集的停顿类型，而不是用来区分新生代GC还是老年代GC的.如果有"Full"，说明这次GC是发生了Stop-The-World的，例如下面这段新生代收集器ParNew的日志也会出现"［Full GC"(这一般是因为出现了分配担保失败之类的问题，所以才导致STW).如果是调用System.gc()方法所触发的收集，那么在这里将显示"［Full GC (System)".

    [Full GC 283.736: [ParNew: 261599K->261599K(261952K), 0.0000288 secs]

接下来的"［DefNew"、"［Tenured"、"［Perm"：表示GC发生的区域，这里显示的区域名称与使用的GC收集器是密切相关的，例如上面样例所使用的Serial收集器中的新生代名为"Default New Generation"，所以显示的是"［DefNew".如果是ParNew收集器，新生代名称就会变为"［ParNew"，意为"Parallel New Generation".如果采用Parallel Scavenge收集器，那它配套的新生代称为"PSYoungGen"，老年代和永久代同理，名称也是由收集器决定的.

后面方括号内部的"3324K->152K(3712K)"：含义是"GC前该内存区域已使用容量-> GC后该内存区域已使用容量 (该内存区域总容量)".而在方括号之外的"3324K->152K(11904K)"：表示"GC前Java堆已使用容量 -> GC后Java堆已使用容量 (Java堆总容量)".

再往后，"0.0025925 secs"表示该内存区域GC所占用的时间，单位是秒.有的收集器会给出更具体的时间数据，如"［Times： user=0.01 sys=0.00， real=0.02 secs］"，这里面的user、sys和real与Linux的time命令所输出的时间含义一致，分别代表用户态消耗的CPU时间、内核态消耗的CPU事件和操作从开始到结束所经过的墙钟时间(Wall Clock Time).CPU时间与墙钟时间的区别是，墙钟时间包括各种非运算的等待耗时，例如等待磁盘I/O、等待线程阻塞，而CPU时间不包括这些耗时，但当系统有多CPU或者多核的话，多线程操作会叠加这些CPU时间，所以读者看到user或sys时间超过real时间是完全正常的.

#### 3.6 内存分配策略与回收策略

* 对象优先在Eden分配
* 大对象直接进入老年代
* 长期存活的对象将进入老年代
* 动态对象年龄判定
* 空间分配担保

    Minor GC之前,虚拟机会先检查老年代最大可用连续空间是否大于新生代对象总空间,  
    如果大于,则Minor GC是安全的  
    如果不大于,则会查看HandlePromotionFailure设置值是否允许担保失败,  
        如果允许,则检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小  
            如果大于,将进行一次Minor GC,尽管这次是有风险的
            如果小于,那要改为进行Full GC.  
        如果不允许冒险,那要改为进行Full GC.  

### 第４章 虚拟机监控工具

jps: 虚拟机进程状况工具  
jstat: 虚拟机统计信息工具  
jinfo: Java配置信息工具  
jmap: Java内存映像工具  
jhat: 虚拟机堆转储快照分析工具  
jstack: Java堆栈跟踪工具  
HSDIS: JIT生成代码反汇编  

### 第七章 虚拟机类加载机制

7个阶段:
加载 Loading,  
验证 Verification,准备 Preparation,解析 Resolution,  
初始化 Initialization,使用 Using,卸载 Unloading

其中,验证,准备,解析三个部分称为连接Linking

#### 7.4 类加载器

两个类相等:必须是由同一个类加载器加载的同一个Class文件来的

##### 7.4.2 双亲委派模型

从Java开发人员的角度来看,主要有三种系统提供的类加载器

* 启动类加载器(Bootstrap ClassLoader):负责加载$JAVA_HOME/lib目录中的或者由-Xbootclasspath指定路径,它不能被Java程序直接引用
* 扩展类加载器(Extension ClassLoader):由sun.misc.Lanuncher$ExtClassLoader实现,负责加载$JAVA_HOME/lib/ext目录中,或者由java.ext.dirs系统变量指定,开发者可以使用
* 应用程序类加载器(Application ClassLoader),也叫系统类加载器:由sun.misc.Lanuncher$AppClassLoader实现.负责加载用户路径(Classpath)上所指定的类库,开发者可以使用.

##### 7.4.3 破坏双亲委派模型

1. 在JDK1.2双亲委派模型引入之前
2. 自身缺陷导致,基础类需要调用用户代码.  
如JNDI的代码由启动类加载器加载,但它需要调用独立实现并部署在应用程序的ClassPath下的JNDI接口提供者(SPI, Service Provider Interface)的代码,但启动类不"认识".
为了解决这个问题,引入了线程上下文类加载器(Thread Context ClassLoader)
3. 为了追求程序的动态性:代码热替换 HotSwap,模块热部署 Hot Deployment. OSGi实现模块化热部署,它的类加载器是网状结构,可以在平级的类加载器中进行

#### 类加载及执行子系统实例

##### Tomcat

[Tomcat 5.5](https://tomcat.apache.org/tomcat-5.5-doc/class-loader-howto.html)

            Bootstrap
              |
           Extension ClassLoader
              |
           System
              |
           Common
          /      \
     Catalina   Shared
                 /   \
            Webapp1  Webapp2 ...

[Tomcat 6.0 remove Shared ClassLoader](https://tomcat.apache.org/tomcat-6.0-doc/class-loader-howto.html)

          Bootstrap
              |
           System
              |
           Common
           /     \
      Webapp1   Webapp2 ...

##### OSGi

[Classloading](http://moi.vonos.net/java/osgi-classloaders/)

    bootstrap ClassLoader (includes Java standard libraries from jre/lib/rt.jar etc)
       ^
    extension ClassLoader
       ^
    system ClassLoader (i.e. stuff on $CLASSPATH, including OSGi core code)
       ^
    OSGi environment ClassLoader
       ^    (** Note: OSGi ClassLoaders forward lookups to parent ClassLoader only for some packages, e.g. java.*)
       \
        \   |-- OSGi ClassLoader for "system bundle"  -> (map of imported-package->ClassLoader)
         \--|-- OSGi ClassLoader for bundle1    -> (map of imported-package->ClassLoader)
            |-- OSGi ClassLoader for bundle2    -> (map of imported-package->ClassLoader)
            |-- OSGi ClassLoader for bundle3    -> (map of imported-package->ClassLoader)
                                         /
                                        /
          /========================================================================================\
          |  shared bundle registry, holding info about all bundles and their exported-packages  |
          \========================================================================================/

### 第12章 java内存模型与线程

#### 12.3 java 内存模型

原子性,可见性,有序性  
先行发生原则

##### 12.3.1 主内存与工作内存

Java内存模型的主要目标是定义虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节. 它规定了所有的变量都存储在主内存(Main Memory)中,每条线程有自己的工作内存(Working Memory), 线程工作内存保存了使用到的变量的主内存副本拷贝,线程对变量的操作(读取,赋值等)都必须在工作内存中进行,而不能直接读写主内存中的变量. 不同的线程之间也不能互相访问. 线程间变量值的传递均需要通过主内存来完成.

##### 12.3.2 内存间交互操作

关于主内存与工作内存之间的交互协议,Java内存模型中定义了以下8种操作来完成,虚拟机实现必须保证每一种操作都是原子的,不可再分的(对于double和long类型的变量来说,load,store,read和write操作在某些平台上允许例外)

**注**:JSR-133文档已经放弃采用这8种操作来描述Java内存模型的访问协议.

1. lock: 作用于主内存变量,它把一个变量标识为一条线程独占状态
2. unlock: 作用于主内存变量,它把一个处于锁定状态的变量释放,释放后的变量才可以被其他线程锁定
3. read: 作用于主内存变量,它把一个变量的值从主内存传输到线程的工作内存,以全 load 使用
4. load: 作用于工作内存变量,它把read操作的值放入工作内存的变量副本中
5. use: 作用于工作内存变量,它把工作内存中一个变量的值传递给执行引擎,每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作
6. assign: 作用于工作内存变量,它把一个从执行引擎收到的值赋给工作内存变量每当虚拟机遇到一个需要给变量赋值的字节码指令时执行这个操作
7. store:　作用于工作内存变量,它把工作内存中一个变量的值传送到主内存中,以便随后的write使用
8. write: 作用于主内存变量,它把store操作从工作内存中得到的变量的值放入主内存的变量中

如果要把一个变量从主内存复制到工作内存,那就要顺序执行read,load操作.
注意,Java内存模型只要求顺序执行,而不保证是连续执行.

##### 12.3.3 volatile

1. 保证此变量对所有线程的可见性
2. 禁止指令重排序优化
volatile一般情况下不能代替sychronized，因为volatile不能保证操作的原子性，即使只是i++，实际上也是由多个原子操作组成

#### 12.4 java与线程

#####　12.4.3 状态 java.lang.Thread.State

1. New
2. Runnable
3. Waiting
4. Timed Waiting
5. Blocked
6. Terminated

### Java™ Tutorials

[Java Documentation](http://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)

#### Pausing Execution with `sleep`

`Thread.sleep` causes the current thread to suspend execution for a specified period

#### Interrupts

An interrupt is an indication to a thread that it should stop what it is doing and do something else

#### Joins

The join method allows one thread to wait for the completion of another. If `t` is a `Thread` object whose thread is currently executing, `t.join();` causes the current thread to pause execution until t's thread terminates.

#### `wait`

Always invoke wait inside a loop that tests for the condition being waited for. Don't assume that the interrupt was for the particular condition you were waiting for, or that the condition is still true.

Item 50 "Never invoke wait outside a loop" in Joshua Bloch's "Effective Java Programming Language Guide" (Addison-Wesley, 2001).

该方法属于Object的方法，wait方法的作用是**使当前调用wait方法所在部分(代码块)的线程**停止执行，并释放当前获得的调用wait所在的代码块的锁，并在其他线程调用notify或者notifyAll方法时恢复到竞争锁状态(一旦获得锁就恢复执行).

#### Concurrent Collections

`BlockingQueue`, `ConcurrentMap`, `ConcurrentHashMap`, `ConcurrentNavigableMap`, `ConcurrentSkipListMap`

#### Atomic Variables

`AtomicInteger`

### [理解Java中的四种引用](http://www.importnew.com/17019.html)

Java中实际上有四种强度不同的引用，从强到弱它们分别是，强引用，软引用，弱引用和虚引用

#### 强引用(Strong Reference)

就是我们经常使用的引用，其写法如下 `StringBuffer buffer = new StringBuffer();`

#### 软引用（Soft Reference）

当内存不足时垃圾回收器才会回收这些软引用可到达的对象。

#### 弱引用(Weak Reference)

弱引用简单来说就是将对象留在内存的能力不是那么强的引用。使用WeakReference，垃圾回收器会帮你来决定引用的对象何时回收并且将对象从内存移除。创建弱引用如下
`WeakReference<Widget> weakWidget = new WeakReference<Widget>(widget)`

#### 虚引用 （Phantom Reference）

我们不可以通过get方法来得到其指向的对象。它的唯一作用就是当其指向的对象被回收之后，自己被加入到引用队列，用作记录该引用指向的对象已被销毁。

虚引用使用场景主要由两个。它允许你知道具体何时其引用的对象从内存中移除。而实际上这是Java中唯一的方式。这一点尤其表现在处理类似图片的大文件的情况。当你确定一个图片数据对象应该被回收，你可以利用虚引用来判断这个对象回收之后在继续加载下一张图片。这样可以尽可能地避免可怕的内存溢出错误。

第二点，虚引用可以避免很多析构时的问题。finalize方法可以通过创建强引用指向快被销毁的对象来让这些对象重新复活。
