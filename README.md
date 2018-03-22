# 面试常见问题

## 1. 网络相关问题

101. 讲下tcp协议？滑动窗口作用？拥塞处理？调优参数有哪些？time_out数量过多是因为什么，如何解决？

    tcp属于tcp/ip协议中的传输层协议，是一种有状态、可靠的传输协议。
    头格式：源端口、目标端口、发送序号、确认序号、偏移量、保留位、tcp flags、滑动窗口、校验和、紧急指针、tpc options、data
    通过三次握手创建连接，四次挥手断开连接。

    三次握手过程

    1. 客户端发送seq1给服务端(客户端从close->syn_sent,服务端处于listen状态->syn_revd)
    2. 服务端返回ack1=seq1+1和seq2(客户端处于syn_sent->established)
    3. 客户端发送ack2=seq+2(服务端从syn_revd->established)

    四次挥手过程，假设是由客户端发起

    1. 客户端发送fin=1，seq1，ack2(客户端从write->fin_wait_1)
    2. 服务端返回ack1=seq1+1(服务端从read->close_wait,客户端从fin_wait_1->fin_wait_2)
    3. 服务端发送fin=1，seq2(如果当前连接处于空闲，会马上发送fin，如果还有未发完的数据，先把数据发完在发送fin)(服务端从close_wait->closing,客户端从fin_wait_2->time_wait)
    4. 客户端发送ack2=seq2+1(服务端从close->time_wait)

    重传机制

    1. 超时重传
    2. 快速重传
    3. sack方法

102. 讲下http协议

    超文本传输协议，属于tcp/ip协议中的应用层协议，是基于tcp实现，是无状态的，request response模式
    request请求数据格式：请求行(method、url、协议版本)、header(host、referer、cookie、context-type等)、空白行、请求正文
    response响应数据格式: 状态行(协议版本、状态码、状态码简短说明)、header(location、server、cookie、accept-ranges)、空白行、响应正文
    状态码 1xx-消息已收到，正在处理 2xx-成功 3xx-重定向 4xx-客户端的错误，资源不存在或者数据错误 5xx-服务端错误

103. 讲下websocket协议

    建立在tcp协议之上，握手阶段采用http协议，双向平等对话，客户端可以主动推消息给服务端，服务端也可以主动推消息给客户端，可以发送文本也可以发送二进制，
    没有同源限制，客户端可以与任意服务器通信，协议标识是ws，如果加密为wss
    数据帧格式：fin、rsv1、rsv2、rsv3、opcode、mask、playload length、mask-key、playload data、extension data、application data

104. Tcp连接，建立连接是三次握手，而断开连接却是四次挥手

    建连接时主要是交换两端的序列号，没有中间任何状态
	而关闭连接时，有可能还有数据未发完，所以无法把ack和fin在同一次中返回

105. socket编程

    是应用层和传输层的中间软件抽象层，把复杂的tcp/ip协议族隐藏在一组接口后面。
    Client：创建一个socket->连接服务端(connect)->发送/接收数据(write/read)->关闭连接(close)
    server：创建一个socket->绑定一个端口(bind)->等待请求(listen)->允许连接请求(accept)->接收/发送数据(write/read)->关闭连接(close)

106. select、poll、epoll

### 1.1 值得一看

* [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
* [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)
* [websocket 数据帧](https://coolshell.cn/articles/11564.html)
* [大话 Select、Poll、Epoll](https://cloud.tencent.com/developer/article/1005481)

## 2. redis相关问题

201. redis的集群是怎么搭建的？redis cluster是怎么分布key的？如果集群的新增或者减少一个节点的时候会发生什么？

    redis中有个slot东西，在建集群的时候就先把slot数量设定好，然后key是和slot绑定的，slot在哪个节点上key就哪个节点上。
    当redis有挂掉的时候，集群是无法自动恢复，在挂的这段时间，访问的key在该节点上，那么请求会失败。必须通过人工干预重启该节点。
    如果新增一个节点，也必须人工去重建slot分布。

202. redis的sorted set是怎么实现

203. Redis的使用过程中一般遇到过什么问题

    * redis是一个单线程操作，如果某个操作比较耗时，会导致后续的请求都变得耗时。例如keys、hgetall、hmget等操作
    * 主从模式改成cluster，需要重新考虑key的分布，例如会把一个大的hash拆成string
    * 使用scan的时候可能会返回重复的数据，需要做过滤

204. memcache和redis区别

### 2.1 值得一看

* [redis sorted set内部实现](https://zsr.github.io/2017/07/03/redis-zset%E5%86%85%E9%83%A8%E5%AE%9E%E7%8E%B0/)
* [Redis内部数据结构详解](http://zhangtielei.com/posts/server.html)

## 3. mysql相关问题

301. Mysql的查询优化

302. Mysql索引的数据结构和最左匹配

303. Mysql的集群搭建

304. MySQL超时配置多久合适、链接数配置多少合适？

### 3.1 值得一看

* [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

## 4. Go相关问题

400. channel的实现原理？channel存储的是副本还是引用

    int、bool、string等值类型是副本，slice、map等是引用

401. Goroutine是怎么实现的(也就是PMG)

402. go nil的实现

403. go interface的使用？interface的数据结构

404. 数组和切片的区别，数组底层的数据结构，slice底层的数据结构,append slice会有哪些情况发生？

405. make和new的区别

    new是创建一个类型指针,并将其初始化为零值，如果是一个结构体，将结构体的field都初始化为零值
    make主要是用来创建channel、slice、map，仅仅返回一个类型指针，不会做初始化。针对channel可以指定channel长度，针对slice可以指定长度(len)和容量(cap)。

406. sync.WaitGroup的实现原理

407. sync.Once()如果要你自己来实现这个机制？你会怎么去做？

408. struct和*struct的区别，或者说结构体方法和结构体指针方法的区别？

    无论创建的对象是不是指针，都能调用。还有就是修改field值得区别
    在给接口赋值的时候，如果有指针方法，必须用指针赋值

409. context的实现原理？使用场景？

    使用场景
    * 超时控制
    * 主动结束多个协程
    * 夸多个协程传值，例如请求者信息、trace id

    原理
    
    有三种实现cancel、timeout、value，数据结构都是树

    value：数据结构key、value和parent context，只能进行新增值和获取值，不能修改值。要修改值只能重建一个新的树

410. go中有哪些方式可以实现并发同步？eg：4件事情要完成，1 2 3完成之后4才执行

	sync.WaitGroup、channel

411. Go网络模型，select的io多路复用

412. sort接口定义？怎么实现？

	包含三个方法，len，swap，less

413. Go中的性能调优方式？pprof 、race？

414. go tools的使用，go build参数

415. go error的实现

    error接口定义了一个方法Error()string

416. go database/sql包的实现

417. select和switch的区别

    select只能接受channel参数，并且每次只有一个case会被执行，如果没case和default会被block住，如果有多个case会随机执行一个
	switch能接受所有类型参数，还能用于做数据类型判断，可能有多个case被执行

418. 从main函数里面go出去的协程panic会怎样？http handler发生panic了会不会导致进程退出？

	从main函数里面go出去的协程panic会导致进程panic。
    http handler发生panic了不会导致进程退出。

419. init的执行顺序？

    被引用包init先执行，同一个包内是无序的

420. Map底层的数据结构

421. string底层实现？为什么是固定长度？


### 4.1 值得一看

* [50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)
* [golang 面试题(看过表示很经典)](https://zhuanlan.zhihu.com/p/26972862) [答案](https://yushuangqi.com/blog/2017/golang-mian-shi-ti-da-an-yujie-xi.html?from=groupmessage&isappinstalled=0)
* [实效编程-官方英文版](https://golang.org/doc/effective_go.html)

## 5. 数据结构和算法相关问题

501. 手写算法一个单项链表 1234567，通过对折，变成 1726354

502. 假设有1000w个邮箱地址，怎么去重

503. 合并两个数组，去重并排序

## 6. linux相关问题

601.  Linux下如何查看进程有问题是因为什么导致的？

## 7. 其他问题

701. 业务场景中计数器如何实现

702. ssh怎么实现登录

703. 对grpc的了解？或者说讲讲grpc

704. LRU算法

705. 讲讲微服务

706. 讲讲restfull

707. 一致性hash实现
