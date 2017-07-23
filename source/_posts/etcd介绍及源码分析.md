---
title: etcd介绍及源码解析
tags: etcd
categories: 分布式协同
---

### etcd 相关网站
https://coreos.com/etcd/docs/latest/
https://coreos.com/etcd/docs/latest/api.html
https://coreos.com/etcd/docs/latest/libraries-and-tools.html

### Eclipse安装golang语言plugin：

Installation Requirements:

- Java VM version 8 or later.
- Eclipse 4.6 (Neon) or later.
- CDT 9.0 or later (this will be installed or updated automatically as part of the steps below).

Instructions:

- Use your existing Eclipse, or download a new Eclipse package from http://www.eclipse.org/downloads/.
    - For an Eclipse package without any other IDEs or extras (such a VCS tools), download the "Platform Runtime Binary".
- Start Eclipse, go to Help -> Install New Software...
- Click the Add... button, then enter the Update Site URL: http://goclipse.github.io/releases/ in the Location field, click OK.
- Select the recently added update site in the Work with: dropdown. Type GoClipse in the filter box. Now the Goclipse feature should appear below.
- Select the GoClipse feature, and complete the wizard.
    - Dependencies such as CDT will automatically be added during installation.
- Restart Eclipse.
- Follow the instructions from the User Guide's Configuration section to configure the required external tools. It is recommended you read the rest of the guide too.

### 源码解析

__客户端：client.go__

etcdmain/main.go  Main():
``` golang
     —> etcdmain/etcd.go  startEtcdOrProxyV2():
               —> etcdmain/config.go  cfg := newConfig():               etcd初始化对象配置
               —> etcdmain/config.go  cfg.parse(Args[1:]):               解析etcd启动参数
               —> etcdmain/etcd.go  setupLogging(cfg)
               —> etcdmain/etcd.go  which := identifyDataDirOrDie(cfg.Dir)       分析Dir参数，解析etcd启动方式，如果Dir参数为空，也会走下面2个分支
                         —> case dirMember:
                                   —> etcdmain/etcd.go  startEtcd(&cfg.Config)
                                             —> embed/etcd.go  e, err := embed.StartEtcd(cfg)                         启动etcd服务
                                                       —> embed/config.go  cfg.Validate()                                             验证config值
                                                       —> embed/etcd.go  e.Peers := startPeerListeners(cfg)
                                                                 —> plns = make([]net.Listener, len(cfg.LPUrls))
                                                                 —> for i, u := range cfg.LPUrls
                                                                           —> pkg/transport/listener.go  tlscfg = cfg.PeerTLSInfo.ServerConfig()               CA file config
                                                                           —> rafthttp/util.go  plns[i] = rafthttp.NewListener(u, tlscfg)
                                                                                     —> transport/timeout_listener.go  transport.NewTimeoutListener(u.Host, u.Scheme, tlscfg, ConnReadTimeout, ConnWriteTimeout)
                                                                                               —> pkg/transport/listener.go  ln := newListener(u.Host, u.Scheme)
                                                                                                         —> unix_listener.go       if (u.Scheme == “unix” | “unixs”)     return NewUnixListener(u.Host)               unix socket via unix://laddr
                                                                                                         —> dial.go                         else      return net.Listen(“tcp”, u.Host)                                        tcp socket
                                                                                               —> transport/timeout_listener.go  ln = &rwTimeoutListener{}
                                                                                               —> pkg/transport/listener.go  ln = wrapTLS(u.Host, u.Scheme, tlscfg, ln)
                                                                                                         —> tls.go  tlscfg.NewListener(ln, tlscfg)                                                  创建listener，该listener接受inner连接               tls：Transport Layer Security 传输层安全协议
                                                       —> embed/etcd.go  e.sctxs := startClientListeners(cfg)
                                                                 —> if  cfg.ClientAutoTLS && cfg.ClientTLSInfo.Empty()     cfg.ClientTLSInfo = transport.SelfCert(path.Join(cfg.Dir, “fixtures/client"), cfg.LCurls)               配置tls信息
                                                                 —> sctxs = make(map[string] *serveCtx)
                                                                 —> sctx.l = net.Listen(proto, cfg.LCurls)                         proto=“tcp|unix|unixs"      创建socket 进行listen
                                                                 —> sctx.l = transport.LimitListener(sctx.l, int(fdLimit - reservedInternalFDNum))                    设置listen limit上限值
                                                                 —> pkg/transport/keepalive_listener.go  sctx.l = transport.NewKeepAliveListener(sctx.l, “tcp”, cfg)                                             创建socket
                                                                           —> newTLSKeepaliveListener(sctx.l, cfg)
                                                       —> e.Clients = append(e.Clients, e.sctxs.l)
                                                       —> srvcfg := &etcdserver.ServerConfig{cfg}
                                                       —> etcdserver/server.go  e.Server = etcdserver.NewServer(srvcfg)
                                                                 —> store/store.go  st := store.New(StoreClusterPrefix, StoreKeysPrefix)
                                                                           —> store/store.go  s := newStore(namespaces ...)
                                                                                     —> s := new(store)
                                                                                     —> s.Root = newDir(s, “/“, s.CurrentIndex, Permanent)
                                                                                     —> for namespace := range namespaces        s.Root.Add(newDir(s, namespace, s.CurrentIndex, s.Root, Permanent))
                                                                                     —> s.Stats = newStats()
                                                                                     —> s.WatcherHub = newWatchHub(1000)
                                                                                     —> s.ttlKeyHeap = newTtlKeyHeap()
                                                                                     —> s.readonlySet =  types.NewUnsafeSet(append(namespaces, “/“) ...)
                                                                           —> s.clock = clockwork.NewRealClock()
                                                                 —> haveWAL := wal.Exist(cfg.WALDir())
                                                                 —> ss := snap.New(cfg.SnapDir())
                                                                 —> bepath := path.Join(cfg.SnapDir(), databaseFilename)
                                                                 —> mvcc/backend/backend.go  be := backend.NewDefaultBackend(bepath)
                                                                           —> newBackend(bepath, defaultBatchInterval, defaultBatchLimit)
                                                                                     —> db := bolt.Open(bepath, 0600, boltOpenOptions)
                                                                                     —> b := &backend{}
                                                                                     —> mvcc/backend/batch_tx.go  b.batchTx = newBatchTx(b)
                                                                                               —> tx := &batchTx{b}
                                                                                               —> tx.Commit()
                                                                                     —> go b.run()
                                                                                               —> t := time.NewTimer(b.batchInterval)
                                                                                               —> for
                                                                                                         —> select
                                                                                                                   —> case <- t,C:
                                                                                                                   —> case <- b.stopc:
                                                                                                                             —> b.batchTx.CommitAndStop()
                                                                                                         —> b.batchTx.Commit()
                                                                                                         —> b.Reset(b.batchInterval)
                                                                 —> rafthttp/util.go  prt := rafthttp.NewRoundTripper(cfg.PeerTLSInfo, cfg.peerDialTimeout())
                                                                           —> pkg/transport/timeout_transport.go  transport.NewTimoutTransport(cfg.PeerTLSInfo, cfg.peerDialTimeout())
                                                                                     —> tr := NewTransport(info, dialtimeoutd)
                                                                                     —> tr.Dial =(&rwTimeoutDialer{}).Dial
                                                                 —> swith
                                                                           —> case !haveWAL && !cfg.NewCluster
                                                                                     —> cfg.VerifyJoinExisting()
                                                                                     —> cl = membership.NewClusterFromURLsMap(cfg.InitialClusterToken, cfg.InitialPeerURLsMap)
                                                                                     —> existingCluster = GetClusterFromRemotePeers(getRemotePeerURLs(cl, cfg.Name), prt)
                                                                                     —> membership.ValidateClusterAndAssignIDs(cl, existingCluster)
                                                                                     —> isCompatibleWithCluster(cl, cl.MemberByName(cfg.Name).ID, prt)
                                                                                     —> remotes = existingCluster.Members()
                                                                                     —> cl.SetID(existingCluster.ID())
                                                                                     —> cl.SetStore(st)
                                                                                     —> cl.SetBackend(be)
                                                                                     —> cfg.Print()
                                                                                     —> id, n, s, w = startNode(cfg, cl)
                                                                           —> case !haveWAL && cfg.NewCluster
                                                                                     —> cfg.VerifyBootstrape()
                                                                                     —> cl = membership.NewClusterFromURLsMap(cfg.InitialClusterToken, cfg.InitialPeerURLsMap)
                                                                                     —> m := cl.MemberByName(cfg.Name)
                                                                                     —> isMemberBootstrapped(cl, cfg.Name, prt, cfg.bootstrapTimeout())
                                                                                     —> if cfg.ShouldDiscover()
                                                                                               —> str = discovery.JoinCluster(cfg.DiscoveryURL, cfg.DiscoveryProxy, m.ID, cfg.InitialPeerURLsMap.String())
                                                                                               —> urlsmap = types.NewURLsMap(str)
                                                                                               —> checkDuplicateURL(urlsmap)
                                                                                               —> cl = membership.NewClusterFromURLsMap(cfg.InitoalClusterToken, urlsmap)
                                                                                     —> cl.SetStore(st)
                                                                                     —> cl.SetBackend(be)
                                                                                     —> cfg.PrintWithInitial()
                                                                                     —> id, n, s, w = startNode(cfg, cl, cl.MemberIDs())
                                                                           —> case haveWAL:
                                                                                     —> fileutil.IsDirWriteable(cfg.MemberDir())
                                                                                     —> fileutil.IsDirWriteable(cfg.WALDir())
                                                                                     —> snapshot = ss.Load()
                                                                                     —> st.Recovery(snapshot.Data)
                                                                                     —> cfg.Print()
                                                                                     —> if !cfg.ForceNewCluster
                                                                                               —> id, cl, n, s, w = restartNode(cfg, snapshot)
                                                                                     —> else
                                                                                               —> id, cl, n, s, w = restartAsStandaloneNode(cfg, snapshot)
                                                                                     —> cl.SetStore(st)
                                                                                     —> cl.SetBackend(be)
                                                                                     —> cl.Recover()
                                                                 —> sstats := &stats.ServerStats{}
                                                                 —> sstats.Initialize()
                                                                 —> lstats := stats.NewLeaderStats(id.String())
                                                                 —> srv = &EtcdServer{}
                                                                 —> srv.applyV2 = &applierV2store{}
                                                                 —> srv.be = be
                                                                 —> srv.lessor = lease.NewLessor(srv.be)
                                                                 —> srv.kv = mvcc.New(src.be, srv.lessor, &srv.consistIndex)
                                                                 —> srv.consistIndex.setConsistentIndex(srv.kv.ConsistentIndex())
                                                                 —> srv.authStore = auth.NewAuthStore(srv.be)
                                                                 —> h := cfg.AutoCompactionRetention
                                                                 —> srv.compactor = compactor.NewPeriodic(h, srv,kv, srv)
                                                                 —> tr := &rafthttp.Transport{}
                                                                 —> tr.Start()
                                                                 —> for m := range  remotes      tr.AddRemote(m.ID, m.PeerURLs)
                                                                 —> for m := range  cl.Members()      tr.AddPeer(m.ID, m.PeerURLs)
                                                                 —> srv.r.transport = tr
                                                       —> etcdserver/server.go  e.Server.Start()
                                             —> interrupt_unix.go  osutil.RegisterInterruptHandler(e.Server.Stop)               向系统注册中断事件
                         —> case dirProxy:
                                   —> etcdmain/etcd.go  startProxy(&cfg.Config)
```
