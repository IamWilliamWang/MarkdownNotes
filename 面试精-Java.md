<span id="top">**目录**</span>
[Java、GC](#java)
[JVM、内存管理](#jvm)
[Java锁](#lock)
[Java线程、线程池](#threadpool)
[IO多路复用](#nio)
[Java框架](#ssm)
[数据库基础](#db)
[MySql](#mysql)
[Redis](#redis)
[操作系统](#os)
[计算机网络](#network)
[C++](#cpp)
[其他](#others)

<span id="java">**Java、GC**</span>
[Java GC过程、收集器](https://www.cnblogs.com/dmzna/archive/2020/05/18/12913458.html)
[<font size=2>G1回收过程{0.3}</font>](https://my.oschina.net/u/3159571/blog/3021068)
[<font size=2><u>元空间{0.6}</u></font>](https://www.bilibili.com/read/cv5021445/)
<font size=2>元空间不与堆连续，存在于本地内存</font>
[<u>线程传递数据的三种方式</u>](https://www.cnblogs.com/kuyuyingzi/p/4266272.html)
<font size=2>构造方法、变量和方法、回调函数</font>
[<u>ConcurrentHashMap简单原理</u>](https://www.cnblogs.com/heqiyoujing/p/11143525.html)
<font size=2>ReentrantLock+Segment+HashEntry，synchronized+CAS+HashEntry+红黑树</font>
[ConcurrentHashMap扩容原理{0.6}](https://www.cnblogs.com/lfs2640666960/p/9621461.html)
[<u>Java反射的三种方式</u>](https://www.cnblogs.com/Zombie-Xian/p/6236072.html)
<font size=2>obj.getClass、Object.class、Class.forName</font>
[红黑树](https://blog.csdn.net/li1914309758/article/details/80997342)
[<u>红黑树和AVL的区别{0.4}</u>](https://blog.csdn.net/21aspnet/article/details/88939297)
根到叶子红黑2:1，AVL 1；红黑树插入和删除需要的旋转较少，AVL树查找更快
[<u>常见RuntimeException</u>](https://zhidao.baidu.com/question/617730779498831812.html)
<font size=2>NullPointer、NumberFormat、ClassCst、IllegalArgument、IndexOutOfBounds（SQL、IO、Socket是检查型）</font>
[常见缓存技术{0.3}](https://www.zhihu.com/question/41377757)
[强引用、弱引用、软引用和虚引用](https://blog.csdn.net/baidu_22254181/article/details/82555485)
[GC root](https://wenchao.ren/2019/09/gc-Roots%E5%AF%B9%E8%B1%A1%E6%9C%89%E5%93%AA%E4%BA%9B/)
[Java Socket简单例子](https://www.cnblogs.com/Cavalry-/p/11233862.html)
[<u>Object类</u>](https://www.cnblogs.com/fnlingnzb-learner/p/7263947.html)
clone equals finalize getClass hashCode notify notifyAll toString wait
[<u>BlockingQueue</u>](https://www.cnblogs.com/200911/p/5994044.html)
[<u>字节流和字符流</u>](https://blog.csdn.net/chenkaibsw/article/details/81606722)
<font size=2>字节流不使用缓冲区</font>
[重载和重写](https://www.runoob.com/java/java-override-overload.html)
[Class Loader](https://developer.ibm.com/zh/articles/j-lo-classloader/)
[HashMap和HashSet区别{0.8}](https://blog.csdn.net/chen213wb/article/details/84647179)
[HashMap负载因子为什么选择0.75](http://baijiahao.baidu.com/s?id=1656137152537394906)
[<u>序列化ID的作用{0.6}</u>](https://blog.csdn.net/baidu_37107022/article/details/76860371)
<font size=2>验证版本一致性</font>
<u>lambda常用函数</u>
<font size=2>map、filter、sorted、forEach、reduce、collect、max、min、count、toArray、findFirst</font>
[返 回 目 录　　　　Back to top](#top)

<span id="jvm">**JVM、内存管理**</span>
[<u>JVM内存模型/管理</u>](https://blog.csdn.net/zengxiantao1994/article/details/89303290)
<font size=2>程序计数器，Java虚拟机栈，本地方法栈，Java堆，方法区</font>
[Java的类加载机制](https://blog.csdn.net/cnahyz/article/details/82219210)
[<u>各语言内存管理区别</u>](https://blog.csdn.net/qq_20081637/article/details/80875779)
<font size=2>C：栈、堆、静态文字、常量、程序代码区</font>
[-Xss -Xms -Xmx -Xmn 参数](https://blog.csdn.net/yrwan95/article/details/82826519)
[JVM启动过程](https://www.cnblogs.com/DDiamondd/p/11298477.html)
[<u>JVM常量池的内容，JAVA8变化</u>](https://www.cnblogs.com/tiancai/p/12674192.html)
<font size=2>静态常量池、运行时常量池、字符串常量池、整型常量池。运行时常量池搬到了元空间，字符串常量池搬到了堆</font>
[JVM内存泄漏/泄露场景](https://blog.csdn.net/smile_YangYue/article/details/80219001)
<font size=2>集合类(对象hash值被改变)、各种连接不close、不合理的作用域(把局部变量作为类变量)</font>
[JVM内存溢出OOM场景](https://www.cnblogs.com/yinbiao/p/10613585.html)
[JVM常用命令、Jstat监控堆内外内存(Jconsole、JVisualVM)](https://www.cnblogs.com/zhi-leaf/p/10629033.html)
[返 回 目 录　　　　Back to top](#top)

<span id="lock">**Java锁**</span>
[Java并发锁、synchronized、CAS、AQS](https://www.cnblogs.com/cyrbjh/p/12404794.html)
[<font size=2>CAS保证原子性</font>](https://blog.csdn.net/weixin_45097458/article/details/103222246)
[<font size=2>底层AQS实现原理(ReentrantLock){0.85}</font>](https://blog.csdn.net/qq_29373285/article/details/85164190)
[Java锁升级过程](https://blog.csdn.net/sinat_29774479/article/details/86648801)
[volatile的原则和应用场景{0.15}](https://blog.csdn.net/jinfeiteng2008/article/details/53423858)
[volatile禁用指令重排的目的{0.7}](https://blog.csdn.net/weixin_43982927/article/details/107969119)
[返 回 目 录　　　　Back to top](#top)

<span id="threadpool">**Java线程、线程池**</span>
[<u>线程的生命周期</u>](https://www.cnblogs.com/marsitman/p/11228684.html)
<font size=2>新建、就绪、运行、阻塞、销毁</font>
[线程5种状态（转换函数sleep,join,wait,notify,notifyAll,yield）](https://blog.csdn.net/qq_24692041/article/details/78468671)
[<u>线程池的状态转换函数</u>](https://www.cnblogs.com/noino/p/11265000.html)
<font size=2>Running、ShutDown（不接受）、Stop（打断所有）、Tidying（workCount==0）、Terminated（执行完terminated()）</font>
[线程池的三大方法、七大参数、四种拒绝策略](https://blog.csdn.net/abcdf123456er/article/details/107819340)
[ThreadLocal线程局部存储](https://www.jianshu.com/p/6fc3bba12f38)
[ThreadLocal内存泄漏/泄露](https://blog.csdn.net/puppylpg/article/details/80433271)
[Java线程通信](https://blog.csdn.net/wlddhj/article/details/83793709)
[<u>Java多线程的三大性质</u>](https://blog.csdn.net/a60782885/article/details/77803757)
原子性，有序性和可见性
[返 回 目 录　　　　Back to top](#top)

<span id="nio">**IO多路复用**</span>
[<u>IO模型</u>](https://blog.csdn.net/weixin_44571270/article/details/106651083)
<font size=2>同步阻塞IO、同步非阻塞、IO多路复用、异步IO</font>
[<font size=2>IO模型详解</font>](https://www.cnblogs.com/crazymakercircle/p/10225159.html)
[<u>NIO和AIO应用场景区别</u>](https://blog.csdn.net/lisha006/article/details/82856906)
<font size=2>如果NIO不适合读写时间长的通道，AIO适合</font>
[Java NIO（Channels、Buffers、Selectors）](https://blog.csdn.net/forezp/article/details/88414741)
[select、poll、epoll](https://www.jianshu.com/p/397449cadc9a)
[<font size=2>epoll的函数</font>](https://blog.csdn.net/qq_35210580/article/details/98603051)
[<font size=2>epoll ET与LT</font>](https://www.jianshu.com/p/d3442ff24ba6)
[epollin和epollout事件](https://www.cnblogs.com/anjianliang/p/4317908.html)
[返 回 目 录　　　　Back to top](#top)

<span id="ssm">**Java框架**</span>
[SSM高频考点](https://www.jianshu.com/p/231a582d2a02/)
[Spring和SpringBoot区别](https://www.jianshu.com/p/ffe5ebe17c3a)
[SpringBoot高频考点](https://blog.csdn.net/weixin_45136046/article/details/90768687)
[<u>常用web容器</u>](https://blog.csdn.net/yoyoxh/article/details/89210184)
<font size=2>个人开发Tomcat/Nginx，大项目weblgoic/webshere</font>
[<u>IOC、AOP简述</u>](https://blog.csdn.net/jaryle/article/details/52389672)
<font size=2>控制反转(类的解耦)两种实现：[依赖查找和依赖注入](https://www.jianshu.com/p/17b66e6390fd)。面向切面的编程(更清晰的逻辑，可以让业务逻辑去关注自己本身的业务，而不去关注安全，事物，日志等活动)</font>
[Spring AOP](https://blog.csdn.net/sinat_21843047/article/details/80299366)
[Spring IOC](https://www.jianshu.com/p/17b66e6390fd)
[SSM与SSH的区别](https://www.jianshu.com/p/ae1b0287cd0a)
[SSH原理](https://www.nowcoder.com/discuss/472496)
[SSH详解](https://blog.csdn.net/u014484743/article/details/53197351/)
[返 回 目 录　　　　Back to top](#top)

<span id="db">**数据库基础**</span>
<u>事务的定义</u>
<font size=2>是数据库操作的最小工作单元，是作为单个逻辑工作单元执行的一系列操作；要么都执行、要么都不执行；是一组不可再分割的工作单元</font>
[<u>ACID、如何保证</u>](https://blog.csdn.net/weixin_43326401/article/details/104003945)
<font size=2>原子性、一致性、隔离性和持久性</font>
[<u>隔离级别</u>](https://www.cnblogs.com/jian-gao/p/10795407.html)
<font size=2>读未提交、读提交、可重复读、可串行化；解决更新丢失，但会脏读。解决了脏读。解决了不可重复读，但会幻读。可串行化解决所有问题</font>
[<u>SQL优化方法</u>](https://www.cnblogs.com/xiangpeng/p/11032047.html)
<font size=2>减少返回不必要的数据、减少物理和逻辑读次数、减少计算次数</font>
[常见非关系型数据库](https://blog.csdn.net/qq_34116402/article/details/79578187)
<font size=2>Hbase、Redis、MongodDB[NoSql的分类{0.85}](https://www.runoob.com/mongodb/nosql.html)</font>
[范式](https://www.cnblogs.com/hum0ro/p/8877364.html)
[*B+和B树的区别*](https://blog.csdn.net/mine_song/article/details/63251546)
[*所有搜索树的区别、应用、复杂度*{0.8}](https://blog.csdn.net/u011109881/article/details/80344606)
[返 回 目 录　　　　Back to top](#top)

<span id="mysql">**MySql**</span>
[<u>Mysql表锁、行锁、优化方法</u>](https://www.cnblogs.com/itdragon/p/8194622.html)
<font size=2>默认采用行锁，在未使用索引字段查询（或者全表扫描效率更高）时升级为表锁（如全表更新、多表级联）</font>
[MyISAM 和 InnoDB 的区别](https://blog.csdn.net/chenkeqin_2012/article/details/53868311)
<u>联合索引</u>
<font size=2>CREATE INDEX test2_mm_idx ON test2 (major, minor)</font>
<u>mysql < > = LIKE是否索引失效</u>
<font size=2>LIKE只有在条件首部不是%时使用。如果获取的数据较少，会使用索引</font>
[<u>索引失效的场景</u>](https://blog.csdn.net/look_like/article/details/86486859)
条件中有or；不使用联合索引的第一个字段；组合索引；LIKE里%开头；全表扫描更快时
[redis+mysql缓存一致性{0.3}](https://zhuanlan.zhihu.com/p/58536781)
<u>mysql可重复读是怎么实现的?讲一下innodb的实现方案</u>
<font size=2>MVCC多版本并发控制</font>
[<u>SQL注入的防范</u>](https://www.cnblogs.com/mthp/articles/10668547.html)
<font size=2>把应用服务器的数据库权限降至最低；避免网站打印出SQL错误信息；对特殊字符进行转义处理；查询语句使用参数化的预编译语句</font>
[左连接、右连接、内连接、全外连接的区别](https://blog.csdn.net/weixin_39220472/article/details/81193617)
[返 回 目 录　　　　Back to top](#top)

<span id="redis">**Redis**</span>
[redis缓存击穿、穿透、雪崩](https://www.jianshu.com/p/b7f822935e28)
[redis一致性哈希{0.3}](https://www.jianshu.com/p/735a3d4789fc)
[redis数据结构和场景](https://blog.csdn.net/freedomfanye/article/details/79543970)
[redis的分布式锁，超时时间以及时间续约](https://blog.csdn.net/lzhcoder/article/details/88387751)
[redis的持久化RDB和AOF](https://blog.csdn.net/nxw_tsp/article/details/107966295)
[redis淘汰策略{0.4}](https://baijiahao.baidu.com/s?id=1655596263155881264)
[redis怎么实现事务](https://www.jianshu.com/p/ae4c52af3390)
[CAP](https://my.oschina.net/waylau/blog/625540)
[Jedis详解](https://www.jianshu.com/p/a1038eed6d44)
<u>主从数据库的好处</u>
<font size=2>数据安全、提升I/O性能、读写分离</font>
<u>redis数据结构</u>
<font size=2>string,hash,list,set及sorted set</font>
[返 回 目 录　　　　Back to top](#top)

<span id="os">**操作系统**</span>
<u>五大模块</u>
<font size=2>处理器管理、存储器管理、设备管理、文件管理、作业管理</font>
[线程通信、进程通信](https://blog.csdn.net/J080624/article/details/87454764)
[<u>进程和线程占有的资源</u>](https://blog.csdn.net/WangQYoho/article/details/52598859)
<font size=2>进程：方法区、堆、进程打开的文件描述符、进程用户ID与进程组ID；线程：栈、寄存器、程序计数器、线程ID、优先级</font>
[进程的五种状态](http://ddrv.cn/a/89510)
[<u>用户态切到内核态</u>](https://blog.csdn.net/ddna/article/details/4941373)
三种方式：系统调用、异常、外围设备的中断
[内存伪共享](https://blog.csdn.net/rrrfff/article/details/44993183)
[共享内存mmap使用方法](https://www.cnblogs.com/gdk-0078/p/5165242.html)
[Linux内存管理](https://blog.csdn.net/jasonchen_gbd/article/details/79461385)
[Linux中进程内存结构](https://blog.csdn.net/LZUwujiajun/article/details/82781918)
[Cache 和主存的映射方法](https://blog.csdn.net/qq_25406563/article/details/85011454)
[<u>Linux常用命令</u>](https://blog.csdn.net/luansj/article/details/97272672)
<font size=2>top(查看CPU、内存、进程、交换区信息) ps(进程简况) df(磁盘全部) du(磁盘文件夹) netstat(端口号) chmod(读写权限设置) more less ifconfig(查看网络配置) pwd(显示工作路径)</font>
[返 回 目 录　　　　Back to top](#top)

<span id="network">**计算机网络**</span>
[<u>应用层、运输层、网络层、数据链路层和物理层</u>](https://blog.csdn.net/qq_22238021/article/details/80279001)
<font size=2>HTTP/FTP/DNS/SMTP、TCP/UDP、IP/ICMP</font>
[TCP三次握手、四次挥手](https://blog.csdn.net/qq_40783848/article/details/96153619)
[为什么TCP握三次](https://www.zhihu.com/question/24853633)
[<u>TCP头部内容</u>](https://blog.csdn.net/qq_25948717/article/details/80382766)
源端口号、目标端口号、序列号、确认号、头部长度、保留位、tcp标志位、窗口大小、校验和、紧急指针、可选项
[输入URL到显示的过程(DNS解析过程)](https://blog.csdn.net/didudidudu/article/details/80181505)
[TCP滑动窗口（发送窗口和接受窗口）](https://www.cnblogs.com/hongdada/p/11171068.html)
[TCP流量控制算法{0.2}](https://blog.csdn.net/gengzhikui1992/article/details/89141184)
[TCP挥手Time-Wait过多的原因、解决办法](https://blog.csdn.net/twt936457991/article/details/90574284)
[<u>TCP如何检测客户端是否断开</u>](https://blog.csdn.net/rediculous/article/details/46005217)
<font size=2>Java:sendUrgentData发送一个字节,关闭状态下SO_OOBINLINE属性没有打开,会产生异常。AIO直接channel会抛出异常</font>
[http协议报文和状态码{0.2/0.6}](https://www.cnblogs.com/zhangmumu/p/9213871.html)
[HTTP 2.0与HTTP 1.1区别](https://www.cnblogs.com/frankyou/p/6145485.html)
[对称加密和非对称加密常用算法{0.4}](https://www.jianshu.com/p/3261a051a2fa)
[负载均衡的实现](https://blog.csdn.net/weixin_41440282/article/details/81141609)
[GET和POST区别](https://segmentfault.com/a/1190000018129846)
<u>为什么要建立TCP连接</u>
<font size=2>只有端到端连接是可靠性连接，连接目的是记录两个端口间的通信状态，否则丢包、重复传无从知晓</font>
[cookie和session机制](https://www.cnblogs.com/onefine/p/10499344.html)
[集线器、交换机、路由器、网桥、网关之间的区别](https://www.cnblogs.com/imapla/archive/2013/03/12/2955931.html)
[返 回 目 录　　　　Back to top](#top)

<span id="cpp">**C++**</span>
[C++面试题汇总](https://www.cnblogs.com/inception6-lxc/p/8686156.html)
[返 回 目 录　　　　Back to top](#top)

<span id="others">**其他**</span>
[Python字典实现原理](https://www.cnblogs.com/nelsen-chen/p/9073004.html)
[go协程原理](https://blog.csdn.net/bluehawksky/article/details/84315400)
[go协程调度机制](https://blog.csdn.net/weixin_38054045/article/details/104098139)
[函数式编程的好处](https://baike.baidu.com/item/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B/4035031)
[图形学基础知识](https://www.zhihu.com/question/27544895)
[代码重构](https://www.jianshu.com/p/f63622c1232e)
[返 回 目 录　　　　Back to top](#top)