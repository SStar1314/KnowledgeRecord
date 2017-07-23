---
title: Storm(流计算)介绍
tags: 大数据
categories: bigdata
---
## Storm 简介
参考： http://storm.apache.org/releases/0.10.2/index.html
Storm是一个分布式计算框架，主要由Clojure编程语言编写。最初是由Nathan Marz[1]及其团队创建于BackType，[2]该项目在被Twitter取得后开源。[3]它使用用户创建的“管（spouts）”和“螺栓（bolts）”来定义信息源和操作来允许批量、分布式处理流式数据。

### Storm 源代码导入 Eclipse
下载源代码并导入Eclipse： （可参考网页：http://ylzhj02.iteye.com/blog/2162197）
- 1.git clone git://github.com/apache/storm.git
- 2.mvn clean package install -DskipTests=true
- 3.mvn eclipse:eclipse

### Storm Topology 架构
![](/images/Storm_Topology.png)

- Topology：storm中运行的一个实时应用程序，因为各个组件间的消息流动形成逻辑上的一个拓扑结构。
- Spout：在一个topology中产生源数据流的组件。通常情况下spout会从外部数据源中读取数据，然后转换为topology内部的源数据。Spout是一个主动的角色，其接口中有个nextTuple()函数，storm框架会不停地调用此函数，用户只要在其中生成源数据即可。
- Bolt：在一个topology中接受数据然后执行处理的组件。Bolt可以执行过滤、函数操作、合并、写数据库等任何操作。Bolt是一个被动的角色，其接口中有个execute(Tuple input)函数,在接受到消息后会调用此函数，用户可以在其中执行自己想要的操作。
- Tuple：一次消息传递的基本单元。本来应该是一个key-value的map，但是由于各个组件间传递的tuple的字段名称已经事先定义好，所以tuple中只要按序填入各个value就行了，所以就是一个value list.
- Stream：源源不断传递的tuple就组成了stream。

### Storm Detail
在Storm的集群里面有两种节点： 控制节点(master node)和工作节点(worker node)。控制节点上面运行一个叫Nimbus后台程序，它的作用类似Hadoop里面的JobTracker。Nimbus负责在集群里面分发代码，分配计算任务给机器，并且监控状态。
每一个工作节点上面运行一个叫做Supervisor的节点。Supervisor会监听分配给它那台机器的工作，根据需要启动/关闭工作进程。每一个工作进程执行一个topology的一个子集；一个运行的topology由运行在很多机器上的很多工作进程组成。
![](/images/Storm1.png)  ![](/images/Storm2.jpg)  ![](/images/Storm3.png)

- Storm提供的最基本的处理stream的原语是spout和bolt。你可以实现spout和bolt提供的接口来处理你的业务逻辑。
- 消息源spout是Storm里面一个topology里面的消息生产者。一般来说消息源会从一个外部源读取数据并且向topology里面发出消息：tuple。Spout可以是可靠的也可以是不可靠的。如果这个tuple没有被storm成功处理，可靠的消息源spouts可以重新发射一个tuple， 但是不可靠的消息源spouts一旦发出一个tuple就不能重发了。
- 消息源可以发射多条消息流stream。使用OutputFieldsDeclarer.declareStream来定义多个stream，然后使用SpoutOutputCollector来发射指定的stream。
- Spout类里面最重要的方法是nextTuple。要么发射一个新的tuple到topology里面或者简单的返回如果已经没有新的tuple。要注意的是nextTuple方法不能阻塞，因为storm在同一个线程上面调用所有消息源spout的方法。
- 另外两个比较重要的spout方法是ack和fail。storm在检测到一个tuple被整个topology成功处理的时候调用ack，否则调用fail。storm只对可靠的spout调用ack和fail。
- 所有的消息处理逻辑被封装在bolts里面。Bolts可以做很多事情：过滤，聚合，查询数据库等等。
- Bolts可以简单的做消息流的传递。复杂的消息流处理往往需要很多步骤，从而也就需要经过很多bolts。比如算出一堆图片里面被转发最多的图片就至少需要两步：第一步算出每个图片的转发数量。第二步找出转发最多的前10个图片。(如果要把这个过程做得更具有扩展性那么可能需要更多的步骤)。
- Bolts可以发射多条消息流， 使用OutputFieldsDeclarer.declareStream定义stream，使用OutputCollector.emit来选择要发射的stream。
- Bolts的主要方法是execute, 它以一个tuple作为输入，bolts使用OutputCollector来发射tuple，bolts必须要为它处理的每一个tuple调用OutputCollector的ack方法，以通知Storm这个tuple被处理完成了，从而通知这个tuple的发射者spouts。 一般的流程是： bolts处理一个输入tuple, 发射0个或者多个tuple, 然后调用ack通知storm自己已经处理过这个tuple了。storm提供了一个IBasicBolt会自动调用ack。
- 定义一个topology的其中一步是定义每个bolt接收什么样的流作为输入。stream grouping就是用来定义一个stream应该如果分配数据给bolts上面的多个tasks。


### Storm里面有7种类型的stream grouping
- 　　Shuffle Grouping: 随机分组， 随机派发stream里面的tuple，保证每个bolt接收到的tuple数目大致相同。
- 　　Fields Grouping：按字段分组， 比如按userid来分组， 具有同样userid的tuple会被分到相同的Bolts里的一个task， 而不同的userid则会被分配到不同的bolts里的task。
- 　　All Grouping：广播发送，对于每一个tuple，所有的bolts都会收到。
- 　　Global Grouping：全局分组， 这个tuple被分配到storm中的一个bolt的其中一个task。再具体一点就是分配给id值最低的那个task。
- 　　Non Grouping：不分组，这个分组的意思是说stream不关心到底谁会收到它的tuple。目前这种分组和Shuffle grouping是一样的效果， 有一点不同的是storm会把这个bolt放到这个bolt的订阅者同一个线程里面去执行。
- 　　Direct Grouping： 直接分组， 这是一种比较特别的分组方法，用这种分组意味着消息的发送者指定由消息接收者的哪个task处理这个消息。 只有被声明为Direct Stream的消息流可以声明这种分组方法。而且这种消息tuple必须使用emitDirect方法来发射。消息处理者可以通过TopologyContext来获取处理它的消息的task的id (OutputCollector.emit方法也会返回task的id)。
- 　　Local or shuffle grouping：如果目标bolt有一个或者多个task在同一个工作进程中，tuple将会被随机发生给这些tasks。否则，和普通的Shuffle Grouping行为一致。
