# dubbo启动时class not found org/apache/curator/RetryPolicy

```
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/curator/RetryPolicy
	at com.alibaba.dubbo.remoting.zookeeper.curator.CuratorZookeeperTransporter.connect(CuratorZookeeperTransporter.java:27)
	at com.alibaba.dubbo.remoting.zookeeper.ZookeeperTransporter$Adaptive.connect(ZookeeperTransporter$Adaptive.java)
	at com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistry.<init>(ZookeeperRegistry.java:69)
	at com.alibaba.dubbo.registry.zookeeper.ZookeeperRegistryFactory.createRegistry(ZookeeperRegistryFactory.java:38)
	at com.alibaba.dubbo.registry.support.AbstractRegistryFactory.getRegistry(AbstractRegistryFactory.java:96)
	at com.alibaba.dubbo.registry.RegistryFactory$Adaptive.getRegistry(RegistryFactory$Adaptive.java)
	at com.alibaba.dubbo.registry.integration.RegistryProtocol.getRegistry(RegistryProtocol.java:203)
	at com.alibaba.dubbo.registry.integration.RegistryProtocol.export(RegistryProtocol.java:137)
	at com.alibaba.dubbo.rpc.protocol.ProtocolFilterWrapper.export(ProtocolFilterWrapper.java:98)
	at com.alibaba.dubbo.rpc.protocol.ProtocolListenerWrapper.export(ProtocolListenerWrapper.java:55)
	at com.alibaba.dubbo.qos.protocol.QosProtocolWrapper.export(QosProtocolWrapper.java:60)
	at com.alibaba.dubbo.rpc.Protocol$Adaptive.export(Protocol$Adaptive.java)
	at com.alibaba.dubbo.config.ServiceConfig.doExportUrlsFor1Protocol(ServiceConfig.java:513)
	at com.alibaba.dubbo.config.ServiceConfig.doExportUrls(ServiceConfig.java:358)
	at com.alibaba.dubbo.config.ServiceConfig.doExport(ServiceConfig.java:317)
	at com.alibaba.dubbo.config.ServiceConfig.export(ServiceConfig.java:216)
	at com.alibaba.dubbo.config.spring.ServiceBean.export(ServiceBean.java:291)
	at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:131)
	at com.alibaba.dubbo.config.spring.ServiceBean.onApplicationEvent(ServiceBean.java:53)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:172)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:165)
	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:398)
	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:355)
	at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:882)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549)
	at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:144)
	at org.springframework.context.support.ClassPathXmlApplicationContext.<init>(ClassPathXmlApplicationContext.java:95)
	at com.uetty.dbo.service.App.main(App.java:18)
Caused by: java.lang.ClassNotFoundException: org.apache.curator.RetryPolicy
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:338)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 29 more
```

确保包含如下依赖，注意该依赖所依赖的的zookeeper的jar版本需与zookeeper服务器版本匹配，具体版本对应见zookeeper文档或者一个个版本调试一下试试（如果与zookeeper版本匹配问题会报KeeperErrorCode = Unimplemented错误）
```
<dependency>
    <groupId>org.apache.curator</groupId>
	<artifactId>curator-framework</artifactId>
</dependency>
```

