# spring基于xml格式数据交换的前后端配置与使用之xstream方式

MarshallingHttpMessageConverter 方式解析xml（支持注解修改元素别名）配置及代码如下：

xml配置文件中添加converter

```
<bean class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
		        <constructor-arg>
		        	<bean class="org.springframework.oxm.xstream.XStreamMarshaller">
				        <property name="streamDriver">
				            <bean class="com.thoughtworks.xstream.io.xml.StaxDriver"/>
				        </property>
				        <property name="annotatedClasses">
				            <list>
				                <value>com.mufeng.dto.in.Xml</value>
				                <value>com.mufeng.dto.out.TestDtoOut</value>
				            </list>
				        </property>
				    </bean>
				    
		        </constructor-arg>
		        <property name="supportedMediaTypes">
                	<list>
                		<value>application/xml;charset=UTF-8</value>
                		<value>text/xml;charset=UTF-8</value>
                		<value>text/html;charset=UTF-8</value>
                	</list>
                </property>
			</bean>

```

pom文件


```
<!-- xstream xml start -->
		<dependency>
			<groupId>com.thoughtworks.xstream</groupId>
			<artifactId>xstream</artifactId>
			<version>1.4.10</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-oxm</artifactId>
			<version>4.3.7.RELEASE</version>
		</dependency>
		<!-- xstream xml end -->
```

controller文件

```
@RequestMapping(value = "txml", headers={"content-type=application/xml"})
	public Object testXml(@RequestBody Xml xml/*, HttpServletRequest req*/) {
                // 这部分主要打印看看，验证request xml方式入参成功
		try {
			ObjectMapper mapper = new ObjectMapper();
			mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
					.enable(JsonGenerator.Feature.IGNORE_UNKNOWN).enable(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES);
			System.out.println(mapper.writeValueAsString(xml));// 打印入参数据可以看到成功入参
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}

                // response出参数据
		TestDtoOut tdo = new TestDtoOut();
		tdo.code = 200;
		tdo.msg = "";//xml.getToUserName();
		return tdo;
	}

```
两个出入参dto（只截部分代码）

```
package com.mufeng.dto.in;

import com.thoughtworks.xstream.annotations.XStreamAlias;

@XStreamAlias(value="xml")// 给根节点别名，不然标签名会带上包名
public class Xml {
	@XStreamAlias(value="ToUserName")// 给属性别名，可以使实现xml标签名与属性名不同的需求
	String toUserName;
	String FromUserName;// 不给别名，默认标签名与属性名一致
	Long CreateTime;
	String MsgType;
	Long MsgId;
	String Content;
```
```
package com.mufeng.dto.out;

import com.thoughtworks.xstream.annotations.XStreamAlias;

@XStreamAlias(value="testDtoOut")
public class TestDtoOut {

	public Integer code;
	public String msg;
```


前端文件html

```
<html>
<head>
<script type="text/javascript">
  var xmlData = "<xml><ToUserName><![CDATA[toUser]]></ToUserName><FromUserName><![CDATA[fromUser]]></FromUserName><CreateTime>1348831860</CreateTime><MsgType><![CDATA[text]]></MsgType><Content><![CDATA[this is a test]]></Content><MsgId>1234567890123456</MsgId></xml>";

var xmlhttp;
function loadXMLDoc(url)
{
xmlhttp=null;
if (window.XMLHttpRequest)
  {// all modern browsers
  xmlhttp=new XMLHttpRequest();
  }
else if (window.ActiveXObject)
  {// for IE5, IE6
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
if (xmlhttp!=null)
  {
  xmlhttp.onreadystatechange=state_Change;
  xmlhttp.open("POST",url,true);
  xmlhttp.setRequestHeader("Accept","application/xml"); 
  xmlhttp.setRequestHeader("Content-Type","application/xml"); 
  xmlhttp.send(xmlData);
  }
else
  {
  alert("Your browser does not support XMLHTTP.");
  }
}

function state_Change()
{
if (xmlhttp.readyState==4)
  {// 4 = "loaded"
  if (xmlhttp.status==200)
    {// 200 = "OK"
    document.getElementById('p1').innerHTML="This file was last modified on: " + xmlhttp.getResponseHeader('Last-Modified');
    }
  else
    {
    alert("Problem retrieving data:" + xmlhttp.statusText);
    }
  }
}
</script>
</head>
<body>

<p id="p1">
The getResponseHeader() function returns a header from a resource.
Headers contain file information like length,
server-type, content-type, date-modified, etc.</p>

<button onclick="loadXMLDoc('http://localhost:8080/valida/webctrl/demo/txml')">Get "Last-Modified"</button>

</body>
</html>
```



另外一种使用jackson实现xml数据交换的配置，详见另一篇博客

[spring基于xml格式数据交换的前后端配置与使用之Jackson方式](./2017-08-18_spring_xml_jackson.md)