---
title: Tomcat 生命周期管理 及 源码分析
tags: Tomcat
categories: Tomcat
---

## 几个问题
Java 编写的 Web Application 为什么 能够监听一个port？
Web Application 都没有main函数, 那么该web从哪个入口函数启动的呢？

答案都是 依据 tomact 或 Websphere 这些server实现的， 以Tomcat为例，最重要的就是Tomcat的生命周期管理特性.

### Tomcat 生命周期管理

![](/images/tomcat_1.png)

### Tomcat startup 源代码解析

主要顺序: Bootstrap.java#main()  —> init()  —>  load()  —>  start()

```bash
init():
1. initClassLoaders
2.  Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
   Object startupInstance = startupClass.newInstance();

load():
1. Method method = catalinaDaemon.getClass().getMethod("load", paramTypes);
    method.invoke(catalinaDaemon, param);
2. Catalina.load()
    Digester digester = createStartDigester();   //very important
        inputSource.setByteStream("conf/server.xml");        在该位置解析server.xml,根据xml生成Server/Service/Listener/Context/Executor/Connector/Engine等,执行相应配置的class.
        digester.push(this);
        digester.parse(inputSource);
    getServer().init();

StandardContext  will parser web.xml and config context.

start():
1. Method method = catalinaDaemon.getClass().getMethod("start", (Class [] )null);
        method.invoke(catalinaDaemon, (Object [])null);
2.  getServer().start();    ==>  LifecycleBase.start()    or   WebappClassLoaderBase.start()
```

```java
    /**
     * Create and configure the Digester we will be using for startup.
     * @return the main digester to parse server.xml
     */
    protected Digester createStartDigester() {            // 解析server.xml并初始化的主要入口函数
        long t1=System.currentTimeMillis();
        // Initialize the digester
        Digester digester = new Digester();
        digester.setValidating(false);
        digester.setRulesValidation(true);
        Map<Class<?>, List<String>> fakeAttributes = new HashMap<>();
        List<String> attrs = new ArrayList<>();
        attrs.add("className");
        fakeAttributes.put(Object.class, attrs);
        digester.setFakeAttributes(fakeAttributes);
        digester.setUseContextClassLoader(true);

        // Configure the actions we will be using
        digester.addObjectCreate("Server",
                                 "org.apache.catalina.core.StandardServer",
                                 "className");
        digester.addSetProperties("Server");
        digester.addSetNext(“Server",
                            "setServer",
                            "org.apache.catalina.Server");

        digester.addObjectCreate("Server/GlobalNamingResources",
                                 "org.apache.catalina.deploy.NamingResourcesImpl");
        digester.addSetProperties("Server/GlobalNamingResources");
        digester.addSetNext("Server/GlobalNamingResources",
                            "setGlobalNamingResources",
                            "org.apache.catalina.deploy.NamingResourcesImpl");

        digester.addObjectCreate("Server/Listener",
                                 null, // MUST be specified in the element
                                 "className");
        digester.addSetProperties("Server/Listener");
        digester.addSetNext("Server/Listener",
                            "addLifecycleListener",
                            "org.apache.catalina.LifecycleListener");

        digester.addObjectCreate("Server/Service",
                                 "org.apache.catalina.core.StandardService",
                                 "className");
        digester.addSetProperties("Server/Service");
        digester.addSetNext("Server/Service",
                            "addService",
                            "org.apache.catalina.Service");

        digester.addObjectCreate("Server/Service/Listener",
                                 null, // MUST be specified in the element
                                 "className");
        digester.addSetProperties("Server/Service/Listener");
        digester.addSetNext("Server/Service/Listener",
                            "addLifecycleListener",
                            "org.apache.catalina.LifecycleListener");

        //Executor
        digester.addObjectCreate("Server/Service/Executor",
                         "org.apache.catalina.core.StandardThreadExecutor",
                         "className");
        digester.addSetProperties("Server/Service/Executor");

        digester.addSetNext("Server/Service/Executor",
                            "addExecutor",
                            "org.apache.catalina.Executor");


        digester.addRule("Server/Service/Connector",
                         new ConnectorCreateRule());
        digester.addRule("Server/Service/Connector", new SetAllPropertiesRule(
                new String[]{"executor", "sslImplementationName", "protocol"}));
        digester.addSetNext("Server/Service/Connector",
                            "addConnector",
                            "org.apache.catalina.connector.Connector");

        digester.addObjectCreate("Server/Service/Connector/SSLHostConfig",
                                 "org.apache.tomcat.util.net.SSLHostConfig");
        digester.addSetProperties("Server/Service/Connector/SSLHostConfig");
        digester.addSetNext("Server/Service/Connector/SSLHostConfig",
                "addSslHostConfig",
                "org.apache.tomcat.util.net.SSLHostConfig");

        digester.addRule("Server/Service/Connector/SSLHostConfig/Certificate",
                         new CertificateCreateRule());
        digester.addRule("Server/Service/Connector/SSLHostConfig/Certificate",
                         new SetAllPropertiesRule(new String[]{"type"}));
        digester.addSetNext("Server/Service/Connector/SSLHostConfig/Certificate",
                            "addCertificate",
                            "org.apache.tomcat.util.net.SSLHostConfigCertificate");

        digester.addObjectCreate("Server/Service/Connector/Listener",
                                 null, // MUST be specified in the element
                                 "className");
        digester.addSetProperties("Server/Service/Connector/Listener");
        digester.addSetNext("Server/Service/Connector/Listener",
                            "addLifecycleListener",
                            "org.apache.catalina.LifecycleListener");

        digester.addObjectCreate("Server/Service/Connector/UpgradeProtocol",
                                  null, // MUST be specified in the element
                                  "className");
        digester.addSetProperties("Server/Service/Connector/UpgradeProtocol");
        digester.addSetNext("Server/Service/Connector/UpgradeProtocol",
                            "addUpgradeProtocol",
                            "org.apache.coyote.UpgradeProtocol");

        // Add RuleSets for nested elements
        digester.addRuleSet(new NamingRuleSet("Server/GlobalNamingResources/"));
        digester.addRuleSet(new EngineRuleSet("Server/Service/"));
        digester.addRuleSet(new HostRuleSet("Server/Service/Engine/"));
        digester.addRuleSet(new ContextRuleSet("Server/Service/Engine/Host/"));
        addClusterRuleSet(digester, "Server/Service/Engine/Host/Cluster/");
        digester.addRuleSet(new NamingRuleSet("Server/Service/Engine/Host/Context/"));

        // When the 'engine' is found, set the parentClassLoader.
        digester.addRule("Server/Service/Engine",
                         new SetParentClassLoaderRule(parentClassLoader));
        addClusterRuleSet(digester, "Server/Service/Engine/Cluster/");

        long t2=System.currentTimeMillis();
        if (log.isDebugEnabled()) {
            log.debug("Digester for server.xml created " + ( t2-t1 ));
        }
        return digester;

    }
```
