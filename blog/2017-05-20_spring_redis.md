# 关于spring redis 缓存配置错误的问题

按照网上提供的配置spring redis缓存配置问题，启动后出现如下错误

```
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'cacheManager' defined in class path resource [spring/spring-redis.xml]: Unsatisfied dependency expressed through constructor parameter 0: Ambiguous argument values for parameter of type [org.springframework.data.redis.core.RedisOperations] - did you specify the correct bean references as arguments?
Related cause: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'cacheManager' defined in class path resource [spring/spring-redis.xml]: Unsatisfied dependency expressed through constructor parameter 0: Ambiguous argument values for parameter of type [org.springframework.data.redis.core.RedisOperations] - did you specify the correct bean references as arguments?
at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:736)
at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:189)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1193)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1095)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513)
at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:761)
at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:866)
at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:542)
at org.springframework.web.servlet.FrameworkServlet.configureAndRefreshWebApplicationContext(FrameworkServlet.java:668)
at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:634)
at org.springframework.web.servlet.FrameworkServlet.createWebApplicationContext(FrameworkServlet.java:682)
at org.springframework.web.servlet.FrameworkServlet.initWebApplicationContext(FrameworkServlet.java:553)
at org.springframework.web.servlet.FrameworkServlet.initServletBean(FrameworkServlet.java:494)
at org.springframework.web.servlet.HttpServletBean.init(HttpServletBean.java:138)
at javax.servlet.GenericServlet.init(GenericServlet.java:160)
at org.apache.catalina.core.StandardWrapper.initServlet(StandardWrapper.java:1280)
at org.apache.catalina.core.StandardWrapper.loadServlet(StandardWrapper.java:1193)
at org.apache.catalina.core.StandardWrapper.load(StandardWrapper.java:1088)
at org.apache.catalina.core.StandardContext.loadOnStartup(StandardContext.java:5033)
at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5317)
at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1559)
at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1549)
at java.util.concurrent.FutureTask.run(FutureTask.java:266)
at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
at java.lang.Thread.run(Thread.java:745)
```


翻看源码可以发现是jar包版本升级之后，该类改命名引起的错误，为了解决该问题配置文件应更改如下：


```
<bean id="connectionFactory"
class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
p:host-name="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}"
p:pool-config-ref="poolConfig" />


<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
<property name="connectionFactory" ref="connectionFactory" />
</bean>

<!-- redis缓存管理器 -->
<bean id="cacheManager" class="org.springframework.data.redis.cache.RedisCacheManager"
c:redis-operations-ref="redisTemplate" />
```


将template-ref改为redis-operations-ref即可，或者改cacheManager改成如下也是可以的：


```
<!-- redis缓存管理器 -->
<bean id="cacheManager" class="org.springframework.data.redis.cache.RedisCacheManager"
c:_0-ref="redisTemplate" />

```

相比之下后面那种方式不受改名影响
