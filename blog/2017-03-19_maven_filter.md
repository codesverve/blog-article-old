# 使用jks文件，本地运行没问题，打包到服务器出现java.io.IOException: Invalid keystore format

错误信息：
```
java.io.IOException: Invalid keystore format
	at sun.security.provider.JavaKeyStore.engineLoad(JavaKeyStore.java:658)
	at sun.security.provider.JavaKeyStore$JKS.engineLoad(JavaKeyStore.java:56)
	at sun.security.provider.KeyStoreDelegator.engineLoad(KeyStoreDelegator.java:224)
	at sun.security.provider.JavaKeyStore$DualFormatJKS.engineLoad(JavaKeyStore.java:70)
	at java.security.KeyStore.load(KeyStore.java:1445)
	at com.csii.payment.client.util.Util.getKeyStore(Util.java:446)
	at com.csii.payment.client.key.KeyManager.getKeyStore(KeyManager.java:185)
	at com.csii.payment.client.key.KeyManager.initMerchantJKS(KeyManager.java:144)
	at com.csii.payment.client.key.KeyManager.initJKSInfoUseNewMerchantProperties(KeyManager.java:109)
	at com.csii.payment.client.key.KeyManager.<clinit>(KeyManager.java:85)
	at com.csii.payment.client.core.MerchantSignTool.checkSignParam(MerchantSignTool.java:571)
	at com.csii.payment.client.core.MerchantSignTool.sign(MerchantSignTool.java:89)
	at com.csii.payment.client.core.MerchantSignTool$sign.call(Unknown Source)
	at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:48)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:113)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:125)
	at com.yoochatting.market.v5.service.impl.PayServiceImpl.checkPayGuangda(PayServiceImpl.groovy:680)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:317)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:99)
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:281)
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:96)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:207)
	at com.sun.proxy.$Proxy92.checkPayGuangda(Unknown Source)
	at com.yoochatting.market.v5.service.PayService$checkPayGuangda.call(Unknown Source)
	at org.codehaus.groovy.runtime.callsite.CallSiteArray.defaultCall(CallSiteArray.java:48)
	at org.codehaus.groovy.runtime.callsite.AbstractCallSite.call(AbstractCallSite.java:113)
	at com.yoochatting.market.ctrl.AndroidBuyerbV5Controller.guangdaPayOrder(AndroidBuyerbV5Controller.groovy:618)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.bind.annotation.support.HandlerMethodInvoker.invokeHandlerMethod(HandlerMethodInvoker.java:177)
	at org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter.invokeHandlerMethod(AnnotationMethodHandlerAdapter.java:446)
	at org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter.handle(AnnotationMethodHandlerAdapter.java:434)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:943)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:877)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:966)
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:857)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:624)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:842)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
	at org.tuckey.web.filters.urlrewrite.RuleChain.handleRewrite(RuleChain.java:176)
	at org.tuckey.web.filters.urlrewrite.RuleChain.doRules(RuleChain.java:145)
	at org.tuckey.web.filters.urlrewrite.UrlRewriter.processRequest(UrlRewriter.java:92)
	at org.tuckey.web.filters.urlrewrite.UrlRewriteFilter.doFilter(UrlRewriteFilter.java:394)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
	at org.apache.catalina.core.ApplicationDispatcher.invoke(ApplicationDispatcher.java:747)
	at org.apache.catalina.core.ApplicationDispatcher.processRequest(ApplicationDispatcher.java:485)
	at org.apache.catalina.core.ApplicationDispatcher.doForward(ApplicationDispatcher.java:410)
	at org.apache.catalina.core.ApplicationDispatcher.forward(ApplicationDispatcher.java:337)
	at org.tuckey.web.filters.urlrewrite.NormalRewrittenUrl.doRewrite(NormalRewrittenUrl.java:213)
	at org.tuckey.web.filters.urlrewrite.RuleChain.handleRewrite(RuleChain.java:171)
	at org.tuckey.web.filters.urlrewrite.RuleChain.doRules(RuleChain.java:145)
	at org.tuckey.web.filters.urlrewrite.UrlRewriter.processRequest(UrlRewriter.java:92)
	at org.tuckey.web.filters.urlrewrite.UrlRewriteFilter.doFilter(UrlRewriteFilter.java:394)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:88)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:218)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:169)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)
	at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:958)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:452)
	at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1087)
	at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:637)
	at org.apache.tomcat.util.net.AprEndpoint$SocketProcessor.doRun(AprEndpoint.java:2517)
	at org.apache.tomcat.util.net.AprEndpoint$SocketProcessor.run(AprEndpoint.java:2506)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.lang.Thread.run(Thread.java:748)
```


出现该错误，大都是因为maven将resources下文件拷到classes下的时候导致的，当filtering为true时，maven会在拷贝文件过程中进行变量替换，这导致原文件格式发生变化。而对于jks、so等二进制文件，文件内任意的字节变化都有可能导致文件不符合格式规范，所以会出现报错情况。

filtering参数官方的解释如下
```
filtering: is true or false, denoting if filtering is to be enabled for this resource. Note, that filter *.properties files do not have to be defined for filtering to occur - resources can also use properties that are by default defined in the POM (such as ${project.version}), passed into the command line using the "-D" flag (for example, "-Dname=value") or are explicitly defined by the properties element. Filter files were covered above.
```
意思是，maven会根据`<build><filters> <filter>filters/filter1.properties</filter> </filters></build>`下所定义的变量值、pom文件内默认的诸如`project.version`之类的变量，以及运行命令中-D参数定义的变量，对文件中的${}位置进行变量的替换，该机制在某些情景中应用很有意义，例如：想要在项目中获取pom文件中规定的项目版本号，可以在config.properties中增加一行：`version=${project.version}`，即可使config.properties中的版本号和pom文件保持一致。即使没有匹配的变量，filtering为true时也不可避免的对文件中的某些字节进行了修改。



解决办法是，如果没有需要变量替换的情况，可以最简单的将filtering设为false，否则需将二进制文件单独定义拷贝，并将filtering参数设置为false：
```
<resource>
	<directory>${basedir}/src/main/resources</directory>
	<includes>
		<include>abcd.so</include>
	</includes>
	<filtering>false</filtering>
</resource>
```
在pom文件中指定该文件拷贝规则filtering为false


