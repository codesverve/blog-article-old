# MyBatis分页配置

### 依赖

```
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>4.1.3</version>
</dependency>
```

### 配置

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="typeAliasesPackage" value="com.xxx.xxx.vo"/>
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageHelper">
                    <property name="properties">
                        <value>
                            dialect=mysql
                            offsetAsPageNum=true
                            rowBoundsWithCount=true
                        </value>
                    </property>
                </bean>
            </array>
        </property>
    </bean>
```

### 使用

```
PageHelper.startPage(pageNum, pageSize); // pageNum 下标从1开始
Page<Data> page = dao.searchList(form); // sql中不写limit

```

