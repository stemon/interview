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

107. p2p协议

108. ARP

109. L2转发过程

    假设PC向设备R发送数据，中间经过SW1和SW2路由器

    1. PC

### 1.1 值得一看

* [TCP 的那些事儿（上）](https://coolshell.cn/articles/11564.html)
* [TCP 的那些事儿（下）](https://coolshell.cn/articles/11609.html)
* [websocket 数据帧](https://coolshell.cn/articles/11564.html)
* [大话 Select、Poll、Epoll](https://cloud.tencent.com/developer/article/1005481)
* [Linux TCP/IP调优](https://tonydeng.github.io/2015/05/25/linux-tcpip-tuning/)

## 2. redis相关问题

201. redis的集群是怎么搭建的？redis cluster是怎么分布key的？如果集群的新增或者减少一个节点的时候会发生什么？

    redis中有个slot东西，在建集群的时候就先把slot数量设定好，然后key是和slot绑定的，slot在哪个节点上key就哪个节点上。
    当redis有挂掉的时候，从节点会自动变成主节点，如果没有从节点那么在挂的这段时间，访问的key在该节点上，那么请求会失败，必须通过人工干预重启该节点。
    如果新增一个节点，也必须人工去重建slot分布。

202. redis的sorted set是怎么实现

203. Redis的使用过程中一般遇到过什么问题

    * redis是一个单线程操作，如果某个操作比较耗时，会导致后续的请求都变得耗时。例如keys、hgetall、hmget等操作
    * 主从模式改成cluster，需要重新考虑key的分布，例如会把一个大的hash拆成string
    * 使用scan的时候可能会返回重复的数据，需要做过滤

204. memcache和redis区别

    https://mp.weixin.qq.com/s/vJ5KKhns3eBTsrXCttvZow

205. redis的两种持久化方式

    RDB和AOF
    RDB定时全量备份，如果服务挂掉，会丢失一段时间的数据，对redis影响不大
    AOF增量备份，可以通过配置来指定有多少个key变化备份还是实时备份，会消耗一定的性能

### 2.1 值得一看

* [redis sorted set内部实现](https://zsr.github.io/2017/07/03/redis-zset%E5%86%85%E9%83%A8%E5%AE%9E%E7%8E%B0/)
* [Redis内部数据结构详解](http://zhangtielei.com/posts/server.html)
* [Redis Cluster 实现](http://catkang.github.io/2016/05/08/redis-cluster-source.html)
* [Redis迁移工具](https://github.com/vipshop/redis-migrate-tool)

## 3. mysql相关问题

301. Mysql的查询优化

302. Mysql索引的数据结构和最左匹配

303. Mysql的集群搭建

304. MySQL超时配置多久合适、链接数配置多少合适？

305. mysql binlog作用

    查看数据库的变更历史(具体时间点所有的mysql操作)、数据库增量备份和恢复(增量备份和基于时间点的恢复)、mysql复制(主主数据库的复制、主从数据库的复制)

### 3.1 值得一看

* [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

## 4. Go相关问题

400. channel的实现原理？channel存储的是副本还是引用

    int、bool、string等值类型是副本，slice、map等是引用

    [Go Channel 源码剖析](http://legendtkl.com/2017/08/06/golang-channel-implement/)
    [深入理解 Go Channel](http://legendtkl.com/2017/07/30/understanding-golang-channel/)

401. Goroutine是怎么实现的(也就是PMG)

402. go nil

    [Go语言中的nil](http://nanxiao.me/golang-nil/)

403. go interface的使用？interface的数据结构

    [Go Data Structures: Interfaces](https://research.swtch.com/interfaces)

    [InterfaceSlice](https://github.com/golang/go/wiki/InterfaceSlice)

    [理解 go interface 的 5 个关键点](http://sanyuesha.com/2017/07/22/how-to-understand-go-interface/)

    [Go Interface 源码剖析](http://legendtkl.com/2017/07/01/golang-interface-implement/)

404. 数组和切片的区别，数组底层的数据结构，slice底层的数据结构,append slice会有哪些情况发生？

405. make和new的区别

    new是创建一个类型指针,并将其初始化为零值，如果是一个结构体，将结构体的field都初始化为零值
    make主要是用来创建channel、slice、map，仅仅返回一个类型指针，不会做初始化。针对channel可以指定channel长度；针对slice可以指定长度(len)和容量(cap)；针对map可以指定初始的大小。

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

422. 协程和线程的区别

    协程是应用调度
	线程是系统调度

423. go 1.8 gc

    使用三色标记法+写屏障(write barrier)+STW
    辅助回收功能

    三色标记法

    0. 创建三个集合：黑、灰、白
    1. 将所有对象放入白色集合中
    2. 从根对象开始遍历(不是递归遍历)，把遍历到的对象从白色集合放入灰色集合
    3. 第一次标记，开启写屏障，遍历灰色集合，将灰色对象引用的对象从白色集合放入灰色集合，将灰色对象从灰色集合放入黑色集合.
    4. 重复上一步，直至灰色集合中无任何对象（写屏障的作用：如果这个过程中有黑色对象引用了白色对象，那么就把白色对象置为灰色）
    5. 第二次标记，开启STW，重复2-4
    6. 结束STW，回收所有白色对象

    第二次标记之前是gc和用户代码并发运行，第二次标记是只运行gc、暂停用户代码

    问题

    1. 如果白色对象产生的速度大于垃圾回收的速度，就会导致内存泄漏。

        主要发生在第一次标记，可以使用对象池来优化或者强制执行gc

    [Go语言的实时GC——理论与实践](https://segmentfault.com/a/1190000010753702)
    [Golang 垃圾回收剖析](http://legendtkl.com/2017/04/28/golang-gc/)
    [Go的三色标记GC](https://segmentfault.com/a/1190000012597428)

424. map的实现？自动扩容是怎么实现的？

    m:=make(map[int]struct{},100)//预先分配map的内存

425. [copy内置函数的使用](https://studygolang.com/articles/6160)

    签名类似: copy([]interface{},[]interface{})
    只能用于切片，将第二个slice的元素拷贝到第一个slicel里，拷贝的长度为两个slice里面长度较小的长度值。
    还有一种用法是将字符串当成[]byte类型的slice

426. 多大对象可以定义为大对象

427. go 版本更新历史

[Go 1.10中值得关注的几个变化](https://tonybai.com/2018/02/17/some-changes-in-go-1-10/)

428. csp(communication sequential process)原理

429. 进程、线程、协程和goroutine的关系和区别

### 4.1 值得一看

* [50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)
* [golang 面试题(看过表示很经典)](https://zhuanlan.zhihu.com/p/26972862) [答案](https://yushuangqi.com/blog/2017/golang-mian-shi-ti-da-an-yujie-xi.html?from=groupmessage&isappinstalled=0)
* [实效编程-官方英文版](https://golang.org/doc/effective_go.html)

## 5. 数据结构和算法相关问题

501. 手写算法一个单项链表 1234567，通过对折，变成 1726354

502. 假设有1000w个邮箱地址，怎么去重

503. 合并两个数组，去重并排序

504. 优先级队列

505. 从n个数据中取出x个，列出所有可能的组合

```go
func getN(n int, prefix string, players []string) []string {
    count := len(players)
    //TODO 如何计算出所以可能的数量？
	result := make([]string, 0)
	for i := 0; i < count; i++ {
		if n > 1 {
			array := getN(n-1, prefix+players[i], players[i+1:])
			result = append(result, array...)
		} else {
			result = append(result, prefix+players[i])
		}
	}
	return result
}
```

## 6. linux相关问题

601.  Linux下如何查看进程有问题是因为什么导致的？

602. shell脚本，单引号和双引号的区别

    单引号用于保持引号内所有字符的字面意思
	双引号除了$加变量可以取变量的值，反引号表示命令替换，加\可以对$、反引号、双引号、\表示字面意思外，和单引号一样

603. Linux环境变量的作用

604. Linux /proc

## 7. 其他问题

701. 业务场景中计数器如何实现

702. ssh怎么实现登录

703. 对grpc的了解？或者说讲讲grpc

704. LRU算法

705. 讲讲微服务

706. 讲讲restfull

707. 一致性hash实现

708. 堆和栈的区别

709. mongodb和redis的区别

710. 客户端上报日志->http 接口->kafka，如何保证http接口高吞吐、低延迟，即使kafka节点挂掉也不丢数据

711. git merge和rebase的区别
712. CAP
713. Quorum
714. Consistent Hashing

## 8. kafka相关问题

801. Kafka数据是怎么分布

## 9. 以太坊

1. 一个交易过程

2. 怎么说是去中心化的，因为还有账号啥的

3. 怎么实现让用户选择密码学签名算法

4. trie数据结构(patricia tree,merkle patricia tree)

5. sha3和RLP

6. 椭圆曲线数字签名

7. [casper](https://ethfans.org/posts/understanding-serenity-part-ii-casper)

    [干货 | 权益证明 FAQ（完整版）](https://ethfans.org/posts/Proof-of-Stake-FAQ-new-2018-3-15)
    [Casper 概念原型 乙](https://ethfans.org/posts/casper-poc2)
    [Proof of Stake - 股权证明 系列1](https://ethfans.org/posts/222)

8. 谁没收了验证人的保证金，谁收取验证人的罚金

9. 拜占庭容错研究

10. [干货 | 以太坊Sharding FAQ](https://ethfans.org/posts/Sharding-FAQ)

11. [干货 | STARKs, Part I: 多项式证明](https://ethfans.org/posts/starks_part_1)

12. [白皮书 | zeppelin_os](https://ethfans.org/posts/zeppelin_os_whitepaper)

13. Truebit

14. [科普 | Raiden Network — Ethereum 区块链支付通道](https://ethfans.org/posts/raiden-network-ethereum-17-12-23)

15. [干货 | zkSNARKs（零知识证明）简述](https://ethfans.org/posts/zksnarks-in-a-nutshell)

16. 侧链

17. [干货 | 什么是加密经济学？ 初学者终极指南](https://ethfans.org/posts/what-is-cryptoeconomics-beginners-guide)