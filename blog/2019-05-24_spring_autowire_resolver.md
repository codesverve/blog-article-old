# SpringMVC自定义配置自动注入对象

1. 继承org.springframework.web.method.support.HandlerMethodArgumentResolver的类test.UserArgumentResolver
2. 配置文件配置参数

```
<mvc:annotation-driven>
    <mvc:argument-resolvers>
    	<bean class="com.xxx.UserArgumentResolver"/>
    </mvc:argument-resolvers>
</mvc:annotation-driven>
```

