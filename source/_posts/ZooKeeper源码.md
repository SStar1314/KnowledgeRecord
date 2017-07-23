---
title: ZooKeeper源码
tags: ZooKeeper
categories: 分布式协同
---

## ZooKeeper Eclipse 编译：
1.源码下载：git clone https://github.com/apache/zookeeper.git
```bash
git checkout branch-3.4  (zookeeper版本选择3.4.8进行学习)
```

2.通过Eclipse新建一个Java project, 将project prosition 设置为source code download的位置，java project 创建成果之后，右键build.xml run as Ant build，会自动下载build project的jar包。
多个文件目录下都有build.xml文件，都可以尝试run as Ant build，下载build project所需要的jar包。
3.运行build.xml之后，jar包或许仍不完整，需要自己下载jar包，目录截图如下：
![](/images/ZooKeeper_Code_1.png)
4.至此，zookeeper java工程的编译任务完成。

### Linux 下同样需要ant编译：
```bash
apt-get install openjdk-7-jdk
cd /root/zookeeper-3.4.8
ant
```

## 源码
__RestAPI源码入门：__
RestAPI 入口main函数所在文件： org.apache.zookeeper.server.jersey.RestMain

__ZooKeeper Source Code 解析：__
1.zkServer 脚本启动命令：
```
    nohup "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
    -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=7778 \                         （此行是用作java remote debug使用的，是我后加的）
    -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
     ZOOMAIN="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=$JMXLOCALONLY org.apache.zookeeper.server.quorum.QuorumPeerMain"
```
2.zkCli 脚本启动命令：
```
    "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
     -cp "$CLASSPATH" $CLIENT_JVMFLAGS $JVMFLAGS \
     org.apache.zookeeper.ZooKeeperMain "$@"
```
由上可知， ZooKeeper的server启动入口函数为 __QuorumPeerMain__ ，而client的启动入口函数为 __ZooKeeperMain__。

### QuorumPeerMain解析源码：
main函数：
``` bash
—> QuorumPeerMain main = new QuorumPeerMain()
—> main.initializeAndRun(args)
          —> QuorumPeerConfig config = new QuorumPeerConfig()
          —> config.parse(args)
                    —> parseProperties(cfg)
          —> DatadirCleanupManager purgeMgr = new DatadirCleanupManager(config)
          —> purgeMgr.start()
                    —> timer = new Timer(“PurgeTask")
                    —> TimerTask task = new PurgeTask(dataLogDir, snapDir)     —> PurgeTxnLog.purge(dataLogDir, snapDir)
                    —> timer.scheduleAtFixedRate(task)
          —> runFromConfig(config)
                    —> ServerCnxnFactory cnxnFactory =  ServerCnxnFactory.createFactory()               NIOServerCnxnfactory
                    —> cnxnFactory.configure
                              —> thread = new ZooKeeperThread (Runnable)     参数Runnable是一个线程，传入的是cnxnFactory
                              —> ServerSocketChannel.open()
                    —> quorumPeer = new QuorumPeer()          是一个线程
                    —> quorumPeer.setZKDatabase (new ZKDatabase(quorumPeer.getTxnFactory()) )
                              —> new ZKDatabase()
                                        —> dataTree = new DataTree()
                    —> quorumPeer.start()
                              —> loadDataBase()
                                        —> zkDb.loadDataBase()
                                                  —> zxid = snapLog.restore(dataTree, listener)
                                                            —> processTransaction (hdr, dt, txnLog.itor)
                                                                      —> switch (hdr)
                                                                                —> OpCode.createSession
                                                                                          —> dt.processTxn (itor.getTxn)
                                                                                                    —> switch (header.getType())
                                                                                                              —> OpCode.create
                                                                                                              —> OpCode.delete
                                                                                                              —> OpCode.setData
                                                                                                              —> OpCode.setACL
                                                                                                              —> OpCode.closeSession
                                                                                                              —> OpCode.error
                                                                                                              —> OpCode.check
                                                                                                              —> OpCode.multi
                                                                                —> OpCode.closeSession
                                                                                          —> dt.processTxn (itor.getTxn)
                                        —> currentEpoch = readLongFromFile(“currentEpoch")
                                        —> acceptedEpoch = readLongFromFile(“acceptEpoch")
                              —> cnxnFactory.start()
                                        —> thread.start()
                              —> startLeaderElection()
                              —> super.start()

NIOServerCnxnfactory.run()
     —> while (! ServerSockChannel.socket().isClosed())
               —> selector.select(1000)
               —> selectedList = selector.selectedKeys()
               —> for (SelectedKey k : selectedList)
                         —> if ( k.readyOps() & SelectionKey.OP_ACCEPT != 0)
                                   —> socketChannel = ((ServerSocketChannel) key.channel()).accept()
                                   —> NIOServerCnxn cnxn =  createConnection(socketChannel, k)
                         —> else if (k.readyOps() & (SelectionKey.OP_READ | SelectionKey.OP_WRITE) != 0)
                                   —> NIOServerCnxn cnxn = (NIOServerCnxn) k.attachment()
                                   —> cnxn.doIO(k)
                                             —> if (k.isReadable())
                                                       —> socketChannel.read (incomingBuffer)
                                                       —> zkServer.processPacket (incomingBuffer)
                                                                 —> OpCode.auth                                             解析incomingBuffer，判断请求类型
                                                                 —> OpCode.sasl
                                                                 —> req = new Request (incomingBuffer, ServerCnxn)
                                                                 —> processRequest(req)
                                                                           —> submittedRequest.add(req)
                                             —> if (k.isWritable())
                                                       —> if (outgoingBuffer.size() > 0)
                                                                 —> sockChannel.write(outgoingBuffer)
               —> selector.clear()

quorumPeer.run()
     —> jmxQuorumBean = new QuorumBean(this)
     —> MBeanRegistry.getInstance().register(jmxQuorumBean)
     —> while (running)
               —> switch (peerState)
                         —> case LOOKING:
                                   —> if (readOnly)
                                             —> roZk = new ReadOnlyZooKeeperServer (logFactory, this, new ZooKeeperServer.BasicDataTreeBuilder(), this.zkDb)
                                             —> roZkMgr = new Thread()
                                             —> roZkMgr.start()
                                                       —> roZk.startup()
                                                                 —> registerJMX()
                                                                 —> startSessionTracker()
                                                                 —> setupRequestProcessors()
                                                                 —> state = RUNNING
                                   —> setBCVote
                                   —> setCurrentVote
                         —> case OBSERVING:
                                   —> setObserver (makeObserver (logFactory) )
                                             —> new ZooKeeperServer.BasicDataTreeBuilder()
                                             —> new ObserverZooKeeperServer(logFactory, zkDatabase)
                                             —> new Observer()
                                   —> observer.observeLeader()
                                             —> zk.registerJMX
                                             —> addr = findLeader()
                                             —> connectToLeader (addr)          socket连接leader
                                             —> syncWithLeader()
                                                       —> synchronized (zk)
                                                                 —> zk.getZKDatabase.deserializeSnapshot(leaderIs)
                                             —> while (self.isRunning())
                                                       —> readPacket(qp)
                                                       —> processPacket(qp)
                                                                 —> switch (qp.getType())
                                                                           —> case Leader.PING
                                                                           —> case Leader.PROPOSAL
                                                                           —> case Leader.COMMIT
                                                                           —> …...
                         —> case FOLLOWING:
                                   —> setFollower( makeFollower(logFactory) )
                                   —> follower.followLeader()
                                             —> connectToLeader()
                                             —> syncWithLeader()
                                             —> while (self.isRunning())
                                                       —> readPacket(qp)
                                                       —> processPacket(qp)
                                                                 —> switch (qp.getType())
                                                                           —> case Leader.PING
                                                                           —> case Leader.PROPOSAL
                                                                           —> case Leader.COMMIT
                                                                           —> …...
                         —> case LEADING:
                                   —> setLeader ( makeLeader(logFactory) )
                                   —> leader.lead()
                                             —> zk.registerJMX()
                                             —> cnxAccepter = new LearnerAcceptor()
                                             —> cnxAccepter.run()
                                                       —> LearnerHandler.run()
                                             —> zk.setZxid(ZxidUtils.makeZxid(epoch))
                                             —> startZkServer()
                                                       —> zk.startup ()
                                                                 —> CommitProcessor.run()                     处理已完成的客户端请求
                                                                 —> PrepRequestProcessor.run()               处理未完成客户端请求
                                                                           —> while (true)
                                                                                     —> request = submittedRequests.take()
                                                                                     —> pRequest (request)
                                                                                               —> switch (request.type)
                                                                                                         —> case OpCode.create:          pRequest2Txn(处理所有的请求)
                                                                                                         —> case OpCode.delete:          pRequest2Txn
                                                                                                         —> case OpCode.setData:       pRequest2Txn
                                                                                                         —> case OpCode.setACL:        pRequest2Txn
                                                                                                         —> case OpCode.check:           pRequest2Txn
                                                                                                         —> case OpCode.multi:           pRequest2Txn
                                                                                                         —> …...
                                   —> setLeader(null)

QuorumPeer :
    public Follower follower;          Follower extends Leader
    public Leader leader;
    public Observer observer;          Obsearver extends Learner

class Leader {
    final LeaderZooKeeperServer zk;
    final QuorumPeer self;
    private boolean quorumFormed = false;
    // the follower acceptor thread
    LearnerCnxAcceptor cnxAcceptor;
    // list of all the followers
    private final HashSet<LearnerHandler> learners = new HashSet<LearnerHandler>();
}
class Follower extends Learner{
    private long lastQueued;
    // This is the same object as this.zk, but we cache the downcast op
    final FollowerZooKeeperServer fzk;
}
class Learner {
    QuorumPeer self;
    LearnerZooKeeperServer zk;
    protected BufferedOutputStream bufferedOutput;
    protected Socket sock;
    protected InputArchive leaderIs;
    protected OutputArchive leaderOs;
}
```
＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
__ZooKeeperMain解析源码：__
main函数：
``` bash
     —> main = new ZooKeeperMain
                  —> parseOptions (-server, -timeout, -readonly)
                  —> connectToZk
                              —> zk = new ZooKeeper
                                             —> new ConnectStringParser (属性：chrootPath, serverAddress)
                                             —> new StaticHostProvider (属性：serverAddress)
                                             —> getClientCnxnSocket (返回 ClientCnxnSocket的子类ClientCnxnSocketNIO对象 (属性：selector, sockKey) )
                                             —> new ClientCnxn (参数：chrootPath, hostProvider, zookeeper, watchManager, clientCnxnSocket, 属性：authInfo, pendingQueue, outgoingQueue, zookeeper, clientWatchManager)
                                                             —>  SendThread (参数：ClientCnxnSocketNIO对象，属性：clientCnxnSocket, lastPingSentNs, isFirstConnect)
                                                             —>  EventThread (属性： waitingEvents, sessionState)
                                             —> ClientCnxn.start()
     —> main.run()
               —> processCmd (cl)
                         —> processZKCmd (co)
                                   —> cmd equals create/delete/rmr/set/aget/get/ls/ls2/getAcl/setAcl/stat/listquota/setquota/delquota
                                   —> zk.  create/delete/rmr/set/aget/get/ls/ls2/getAcl/setAcl/stat/listquota/setquota/delquota
                                             —> cnxn. submitRequest (request, response)
                                                       —> queuePacket (request, response)
                                                                 —> synchronized (outgoingQueue)
                                                                           —> outgoingQueue.add ( new Packet(request, response) )
                                                                 —> sendThread.getClientCnxnScoket().wakeupCnxn()
```
==============================================================================================================================
__SendThred run函数：__
属性： state: socket连接状态，isFirstConnect: socket连接是否为第一次初始化连接，zookeeerSaslClient: SASL用户认证机制
``` bash
run
     —> while (state.isAlive)
                    |—> if (! clientCnxnSocket.isConnected)
                    |           —>  startConnect
                    |                    —>  state = CONNECTING
                    |                    —> zookeeerSaslClient 认证
                    |                    —> ClientCnxnSocketNIO.connect
                    |                              —> SocketChannel.register(selector, OP_CONNECT)
                    |                              —> sock.connect(addr)
                    |                              if (immediateConnect)
                    |                                        —> sendThread.primeConnection
                    |                                                  —> new ConnectRequest(lastZxid)
                    |                                                  —> synchronized (outgoingQueue)
                    |                                                            if (! disabledAutoWatchReset)
                    |                                                                      —> 将zookeeper 的属性 dataWatches, existWatchs, childWatchs 组装成 SetWatchs (递增Zxid)
                    |                                                                      —> new RequestHeader (Xid=-8, OpCode.setWatches)
                    |                                                                      —> new Packet, 将requestHeader和setWatchs 一起组装成Packet
                    |                                                                      —> outgoingQueue.addFirst (packet)
                    |                                                            for (id : authInfo)
                    |                                                                      —> outgoingQueue.addFirst (new Packet(new RequestHeader(Xid=-4, OpCode.auth), new AuthPacket(id.schema, id.data)))
          |                               outgoingQueue.addFirst (new Packet(ConnectRequest))
                    |                                                            clientCnxnSocket.enableReadWriteOnly()
                    |—> if (state.isConnected)
                    |          —> if (zookeeperSaslClient != null)
                    |                    —> if (zookeeperSaslClient.getSaslState == ZookeeperSaslClient.SaslState.INITIAL)
                    |                               —> zookeeperSaslClient.initialize
                    |                    —> if (zookeeperSaslClient.getKeeperState == KeeperState.AuthFailed)
                    |                               —> state = States.AUTH_FAILED
                    |                               —> sendAuthEvent = true
                    |                    —> else if (zookeeperSaslClient.getKeeperState == KeeperState.SaslAuthenticated)
                    |                               —> sendAuthEvent = true
                    |                    —> if (sendAuthEvent == true)
                    |                               —> eventThread.queueEvent (new WatchedEvent(Watcher.Event.EventType.None, zookeeperSaslClient.getKeeperState) )
                    |—> if (state.isConnected)
                    |          —> if (clientCnxnSocket.getIdleSend  >  MAX_SEND_PING_INTERVAL)
                    |                    —> sendPing
                    |                              —>  queuePacket (new RequestHeader (Xid=-2, OpCode.ping) )
                    |                                        —> synchronized (outgoingQueue)
                    |                                                  —> new Packet (RequestHeader)
                    |                                                  —> if (RequestHeader != OpCode.closeSession)
                    |                                                            —> outgoingQueue.add(packet)
                    |                                        —> sendThread.getClientCnxnSocket().wakeupCnxn()
                    |—> if (state == States.CONNECTEDREADONLY)
                    |          —> pingRwServer()
                    |                    —> new Socket
                    |                    —> new BufferedReader (new InputStreamReader(socket.getInputStream() ) )
                    |                    —> BufferedReader.readLine()
                    |—> clientCnxnSocket.doTransport (pendingQueue, outgoingQueue)
                    |          —> synchronize (clientCnxnSocket)
                    |                    —> selected = selector.selectedKeys()
                    |          —> for (SelectionKey key : selected)
                    |                     —> if ((key.readyOps() & SelectionKey.OP_CONNECT) != 0)
                    |                              —> if ((SocketChannel)key.channel().finishConnect() )
                    |                                         —> sendThread.primeConnection
                    |                    —> else if ((key.readyOps() & (SelectionKey.OP_READ | SelectionKey.OP_WRITE)) != 0)
                    |                              —> doIO (pendingQueue, outgoingQueue)
                    |                                        —> socketChannel = socketKey.channel()
                    |                                        —> if (socketKey.isReadable())
                    |                                                  —> socketChannel.read(incomingBuffer)
                    |                                                            —> if (! initialized)
                    |                                                                      —> readConnectResult()
                    |                                                                                —> ConnectResponse response = new ConnectResponse()
                    |                                                                                —> response.deserialize (BinaryInputArchive.getArchive(new ByteBufferInputStream(incomingBuffer)) , “connect")
                    |                                                                                —> sendThread.onConnectd(response.getTimeout, response.getSessionId, response.getPasswd)
                    |                                                                                          —> eventThread.queueEvent(new WatchedEvent(Watcher.Event.EventType.None, eventState)  )
                    |                                                                      —> enableRead()
                    |                                                            —> else
                    |                                                                      —> sendThread.readResponse(incomingBuffer)
                    |                                                                                —> ReplyHeader replyHeader = new ReplyHeader()
                    |                                                                                —> replyHeader.deserialize (BinaryInputArchive.getArchive(new ByteBufferInputStream(incomingBuffer)), “header")
                    |                                                                                —> if (replyHeader.getXid == -2)    pings
                    |                                                                                          —> debug
                    |                                                                                —> if (replyHeader.getXid == -4)    AuthPacket
                    |                                                                                          —> if (replyHeader.getErr() == KeeperException.Code.AUTHFAILED)
                    |                                                                                                    —> eventThread.queueEvent (new WatchedEvent(Watcher.Event.EventType.None, Watcher.Event.KeeperState.AuthFailed) )
                    |                                                                                —> if (replyHeader.getXid == -1)    notification
                    |                                                                                          —> WatcherEvent event = new WatcherEvent()
                    |                                                                                          —> event.deserialize(BinaryInputArchive.getArchive(new ByteBufferInputStream(incomingBuffer)), “response")
                    |                                                                                          —> event.setPath (chrootPath)
                    |                                                                                          —> WatchedEvent we = new WatchedEvent(event)
                    |                                                                                          —> eventThread.queueEvent (we)
                    |                                                                      —> synchronized (pendingQueue)
                    |                                                                                —> packet = pendingQueue.remove()
                    |                                                                      —> packet.replyHeader.setXid/setErr/setZxid
                    |                                                                      —> finishPacket (packet)
                    |                                                                                —> eventThread.queuePacket(packet)
                    |                                                                                          —> waitingEvents.add (packet)
                    |                                        —> if (socketKey.isWritable())
                    |                                                  —> Packet packet = findSendablePacket(outgoingQueue, cnxn.sendThread.clientTunneledAuthenticationInProgress)
                    |                                                  —> packet.createBB          byteBuffer
                    |                                                  —> socketChannel.write(packet.bb)
                    |                                                  —> outgoingQueue.removeFirstOccurrence(packet)
                    |                                                  —> if (packet.hasRemaining)
                    |                                                            —> synchronized (pendingQueue)
                    |                                                                      —> pendingQueue.add(packet)
                    |          —> if (sendThread.getZkState().isConnected)
                    |                    —> synchronized (outgoingQueue)
                    |                              —> if (findSendablePacket(outgoingQueue, cnxn.sendThread.clientTunneledAuthenticationInProgress()) != null )
                    |                                        —> enableWrite()
```
==============================================================================================================================
__EventThred run函数：__
属性： waitingEvents
``` bash
run
      —> isRunning = true
      —> while (true)
               —> event = waitingEvents.take()
                         —> if (event == eventOfDeath)
                                   —> wasKilled = true
                         —> else
                                   —> processEvent(event)
                                        —> if (event instanceof WatcherSetEventPair)
                                                       —> for (Watcher watcher : ((WatcherSetEventPair)event).watchers )
                                                                 —> watcher.process( ((WatcherSetEventPair)event).event )
                                             —> else
                                                       —> Packet p = (Packet) event
                                                       —> if (p.response instanceof ExistsResponse || p.response instanceof SetDataResponse || p.response instanceof SetACLResponse)
                                                                 —> StatCallback cb = (StatCallback) p.cb
                                                                 —> cb.processResult ( clientPath, ((ExistsResponse/SetDataResponse/SetACLResponse) p.response).getStat )
                                                                           —> decOutstanding ()
                                                       —> else if (p.response instanceof GetDataResponse)
                                                                 —> DataCallback cb = (DataCallback) p.cb
                                                                 —> GetDataResponse response = (GetDataResponse) p.response
                                                                 —> cb.processResult (clientPath, response.getData(), response.getStat())
                                                       —> else if (p.response instanceof GetACLResponse)
                                                                 —> ACLCallback cb = (ACLCallback) p.cb
                                                                 —> GetACLResponse response = (GetACLResponse) p.response
                                                                 —> cb.processResult (clientPath, response.getAcl(), response.getStat())
                                                       —> else if (p.response instanceof GetChildrenResponse)
                                                                 —> ChildrenCallback cb = (ChildrenCallback) p.cb
                                                                 —> GetChildrenResponse response = (GetChildrenResponse) p.response
                                                                 —> cb.processResult (clientPath, response.getChildren())
                                                       —> else if (p.response instanceof GetChildren2Response)
                                                                 —> Children2Callback cb = (Children2Callback) p.cb
                                                                 —> GetChildren2Response response = (GetChildren2Response) p.response
                                                                 —> cb.processResult (clientPath, response.getChildren())
                                                       —> else if (p.response instanceof CreateResponse)
                                                                 —> StringCallback cb = (StringCallback) p.cb
                                                                 —> CreateResponse response = (CreateResponse) p.response
                                                                 —> cb.processResult (clientPath, response.getPath())
                                                       —> else if (p.response instanceof MultiResponse)
                                                                 —> MultiCallback cb = (MultiCallback) p.cb
                                                                 —> MultiResponse response = (MultiResponse) p.response
                                                                 —> cb.processResult (clientPath, response.getResultList())
                                                       —> else if (p.cb instanceof VoidCallback)
                                                                 —> VoidCallback cb = (VoidCallback) p.cb
                                                                 —> cb.processResult (clientPath)
```

__ClinetCnxn是核心函数：__
``` java
   public ClientCnxn(String chrootPath, HostProvider hostProvider, intsessionTimeout, ZooKeeper zooKeeper,
            ClientWatchManager watcher, ClientCnxnSocket clientCnxnSocket,
            long sessionId, byte[] sessionPasswd, boolean canBeReadOnly) {
        this.zooKeeper = zooKeeper;
        this.watcher = watcher;
        this.hostProvider = hostProvider;
        this.chrootPath = chrootPath;

        sendThread = new SendThread(clientCnxnSocket);
        eventThread = new EventThread();
    }

   public enum States {
        CONNECTING, ASSOCIATING, CONNECTED, CONNECTEDREADONLY,
        CLOSED, AUTH_FAILED, NOT_CONNECTED;

        public boolean isAlive() {
            return this != CLOSED && this != AUTH_FAILED;
        }
        public boolean isConnected() {
            return this == CONNECTED || this == CONNECTEDREADONLY;
        }
    }

        Packet(RequestHeader requestHeader, ReplyHeader replyHeader,
               Record request, Record response,
               WatchRegistration watchRegistration, boolean readOnly) {
            this.requestHeader = requestHeader;
            this.replyHeader = replyHeader;
            this.request = request;
            this.response = response;
            this.readOnly = readOnly;
            this.watchRegistration = watchRegistration;
        }

    public DataTree() {
        /* Rather than fight it, let root have an alias */
        nodes.put("", root);
        nodes.put(rootZookeeper, root);

        /** add the proc node and quota node */
        root.addChild(procChildZookeeper);
        nodes.put(procZookeeper, procDataNode);

        procDataNode.addChild(quotaChildZookeeper);
        nodes.put(quotaZookeeper, quotaDataNode);
    }

   protected void pRequest2Txn(int type, long zxid, Request request, Record record, boolean deserialize)
        throws KeeperException, IOException, RequestProcessorException
    {
        request.hdr = new TxnHeader(request.sessionId, request.cxid, zxid,
                                    zks.getTime(), type);

        switch (type) {
            case OpCode.create:
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                CreateRequest createRequest = (CreateRequest)record;
                if(deserialize)
                    ByteBufferInputStream.byteBuffer2Record(request.request, createRequest);
                String path = createRequest.getPath();
                int lastSlash = path.lastIndexOf('/');
                if (lastSlash == -1 || path.indexOf('\0') != -1 || failCreate) {
                    LOG.info("Invalid path " + path + " with session 0x" +
                            Long.toHexString(request.sessionId));
                    throw new KeeperException.BadArgumentsException(path);
                }
                List<ACL> listACL = removeDuplicates(createRequest.getAcl());
                if (!fixupACL(request.authInfo, listACL)) {
                    throw new KeeperException.InvalidACLException(path);
                }
                String parentPath = path.substring(0, lastSlash);
                ChangeRecord parentRecord = getRecordForPath(parentPath);

                checkACL(zks, parentRecord.acl, ZooDefs.Perms.CREATE,
                        request.authInfo);
                int parentCVersion = parentRecord.stat.getCversion();
                CreateMode createMode =
                    CreateMode.fromFlag(createRequest.getFlags());
                if (createMode.isSequential()) {
                    path = path + String.format(Locale.ENGLISH, "%010d", parentCVersion);
                }
                validatePath(path, request.sessionId);
                try {
                    if (getRecordForPath(path) != null) {
                        throw new KeeperException.NodeExistsException(path);
                    }
                } catch (KeeperException.NoNodeException e) {
                    // ignore this one
                }
                boolean ephemeralParent = parentRecord.stat.getEphemeralOwner() != 0;
                if (ephemeralParent) {
                    throw new KeeperException.NoChildrenForEphemeralsException(path);
                }
                int newCversion = parentRecord.stat.getCversion()+1;
                request.txn = new CreateTxn(path, createRequest.getData(),
                        listACL,
                        createMode.isEphemeral(), newCversion);
                StatPersisted s = new StatPersisted();
                if (createMode.isEphemeral()) {
                    s.setEphemeralOwner(request.sessionId);
                }
                parentRecord = parentRecord.duplicate(request.hdr.getZxid());
                parentRecord.childCount++;
                parentRecord.stat.setCversion(newCversion);
                addChangeRecord(parentRecord);
                addChangeRecord(new ChangeRecord(request.hdr.getZxid(), path, s,
                        0, listACL));
                break;
            case OpCode.delete:
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                DeleteRequest deleteRequest = (DeleteRequest)record;
                if(deserialize)
                    ByteBufferInputStream.byteBuffer2Record(request.request, deleteRequest);
                path = deleteRequest.getPath();
                lastSlash = path.lastIndexOf('/');
                if (lastSlash == -1 || path.indexOf('\0') != -1
                        || zks.getZKDatabase().isSpecialPath(path)) {
                    throw new KeeperException.BadArgumentsException(path);
                }
                parentPath = path.substring(0, lastSlash);
                parentRecord = getRecordForPath(parentPath);
                ChangeRecord nodeRecord = getRecordForPath(path);
                checkACL(zks, parentRecord.acl, ZooDefs.Perms.DELETE,
                        request.authInfo);
                int version = deleteRequest.getVersion();
                if (version != -1 && nodeRecord.stat.getVersion() != version) {
                    throw new KeeperException.BadVersionException(path);
                }
                if (nodeRecord.childCount > 0) {
                    throw new KeeperException.NotEmptyException(path);
                }
                request.txn = new DeleteTxn(path);
                parentRecord = parentRecord.duplicate(request.hdr.getZxid());
                parentRecord.childCount--;
                addChangeRecord(parentRecord);
                addChangeRecord(new ChangeRecord(request.hdr.getZxid(), path,
                        null, -1, null));
                break;
            case OpCode.setData:
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                SetDataRequest setDataRequest = (SetDataRequest)record;
                if(deserialize)
                    ByteBufferInputStream.byteBuffer2Record(request.request, setDataRequest);
                path = setDataRequest.getPath();
                validatePath(path, request.sessionId);
                nodeRecord = getRecordForPath(path);
                checkACL(zks, nodeRecord.acl, ZooDefs.Perms.WRITE,
                        request.authInfo);
                version = setDataRequest.getVersion();
                int currentVersion = nodeRecord.stat.getVersion();
                if (version != -1 && version != currentVersion) {
                    throw new KeeperException.BadVersionException(path);
                }
                version = currentVersion + 1;
                request.txn = new SetDataTxn(path, setDataRequest.getData(), version);
                nodeRecord = nodeRecord.duplicate(request.hdr.getZxid());
                nodeRecord.stat.setVersion(version);
                addChangeRecord(nodeRecord);
                break;
            case OpCode.setACL:
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                SetACLRequest setAclRequest = (SetACLRequest)record;
                if(deserialize)
                    ByteBufferInputStream.byteBuffer2Record(request.request, setAclRequest);
                path = setAclRequest.getPath();
                validatePath(path, request.sessionId);
                listACL = removeDuplicates(setAclRequest.getAcl());
                if (!fixupACL(request.authInfo, listACL)) {
                    throw new KeeperException.InvalidACLException(path);
                }
                nodeRecord = getRecordForPath(path);
                checkACL(zks, nodeRecord.acl, ZooDefs.Perms.ADMIN,
                        request.authInfo);
                version = setAclRequest.getVersion();
                currentVersion = nodeRecord.stat.getAversion();
                if (version != -1 && version != currentVersion) {
                    throw new KeeperException.BadVersionException(path);
                }
                version = currentVersion + 1;
                request.txn = new SetACLTxn(path, listACL, version);
                nodeRecord = nodeRecord.duplicate(request.hdr.getZxid());
                nodeRecord.stat.setAversion(version);
                addChangeRecord(nodeRecord);
                break;
            case OpCode.createSession:
                request.request.rewind();
                int to = request.request.getInt();
                request.txn = new CreateSessionTxn(to);
                request.request.rewind();
                zks.sessionTracker.addSession(request.sessionId, to);
                zks.setOwner(request.sessionId, request.getOwner());
                break;
            case OpCode.closeSession:
                // We don't want to do this check since the session expiration thread
                // queues up this operation without being the session owner.
                // this request is the last of the session so it should be ok
                //zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                HashSet<String> es = zks.getZKDatabase()
                        .getEphemerals(request.sessionId);
                synchronized (zks.outstandingChanges) {
                    for (ChangeRecord c : zks.outstandingChanges) {
                        if (c.stat == null) {
                            // Doing a delete
                            es.remove(c.path);
                        } else if (c.stat.getEphemeralOwner() == request.sessionId) {
                            es.add(c.path);
                        }
                    }
                    for (String path2Delete : es) {
                        addChangeRecord(new ChangeRecord(request.hdr.getZxid(),
                                path2Delete, null, 0, null));
                    }

                    zks.sessionTracker.setSessionClosing(request.sessionId);
                }

                LOG.info("Processed session termination for sessionid: 0x"
                        + Long.toHexString(request.sessionId));
                break;
            case OpCode.check:
                zks.sessionTracker.checkSession(request.sessionId, request.getOwner());
                CheckVersionRequest checkVersionRequest = (CheckVersionRequest)record;
                if(deserialize)
                    ByteBufferInputStream.byteBuffer2Record(request.request, checkVersionRequest);
                path = checkVersionRequest.getPath();
                validatePath(path, request.sessionId);
                nodeRecord = getRecordForPath(path);
                checkACL(zks, nodeRecord.acl, ZooDefs.Perms.READ,
                        request.authInfo);
                version = checkVersionRequest.getVersion();
                currentVersion = nodeRecord.stat.getVersion();
                if (version != -1 && version != currentVersion) {
                    throw new KeeperException.BadVersionException(path);
                }
                version = currentVersion + 1;
                request.txn = new CheckVersionTxn(path, version);
                break;
        }
    }

    protected void processPacket(QuorumPacket qp) throws IOException{
        switch (qp.getType()) {
        case Leader.PING:
            ping(qp);
            break;
        case Leader.PROPOSAL:
            LOG.warn("Ignoring proposal");
            break;
        case Leader.COMMIT:
            LOG.warn("Ignoring commit");
            break;
        case Leader.UPTODATE:
            LOG.error("Received an UPTODATE message after Observer started");
            break;
        case Leader.REVALIDATE:
            revalidate(qp);
            break;
        case Leader.SYNC:
            ((ObserverZooKeeperServer)zk).sync();
            break;
        case Leader.INFORM:
            TxnHeader hdr = new TxnHeader();
            Record txn = SerializeUtils.deserializeTxn(qp.getData(), hdr);
            Request request = new Request (null, hdr.getClientId(),
                                           hdr.getCxid(),
                                           hdr.getType(), null, null);
            request.txn = txn;
            request.hdr = hdr;
            ObserverZooKeeperServer obs = (ObserverZooKeeperServer)zk;
            obs.commitRequest(request);
            break;
        }
    }

switch (packetType) {
        case DIFF:
            return "DIFF";
        case TRUNC:
            return "TRUNC";
        case SNAP:
            return "SNAP";
        case OBSERVERINFO:
            return "OBSERVERINFO";
        case NEWLEADER:
            return "NEWLEADER";
        case FOLLOWERINFO:
            return "FOLLOWERINFO";
        case UPTODATE:
            return "UPTODATE";
        case LEADERINFO:
            return "LEADERINFO";
        case ACKEPOCH:
            return "ACKEPOCH";
        case REQUEST:
            return "REQUEST";
        case PROPOSAL:
            return "PROPOSAL";
        case ACK:
            return "ACK";
        case COMMIT:
            return "COMMIT";
        case PING:
            return "PING";
        case REVALIDATE:
            return "REVALIDATE";
        case SYNC:
            return "SYNC";
        case INFORM:
            return "INFORM";
        default:
            return "UNKNOWN";
        }
    }
```
