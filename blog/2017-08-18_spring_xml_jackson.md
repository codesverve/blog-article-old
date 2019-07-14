# spring基于xml格式数据交换的前后端配置与使用之Jackson方式

MappingJackson2XmlHttpMessageConverter方式解析xml（支持注解修改元素别名）配置及代码如下：

xml配置文件中添加converter

```
<bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
                <property name="supportedMediaTypes">
                	<list>
                		<value>application/xml;charset=UTF-8</value>
                	</list>
                </property>
                <property name="prettyPrint" value="false"/>
            </bean>
```

pom 文件（在原来的基础上添加了jackson-dataformat-xml的依赖，这里jackson为2.8.7版本）


```
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-annotations</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.dataformat</groupId>
			<artifactId>jackson-dataformat-xml</artifactId>
			<version>${jackson.version}</version>
		</dependency>
<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-annotations</artifactId>
			<version>${jackson.version}</version>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.dataformat</groupId>
			<artifactId>jackson-dataformat-xml</artifactId>
			<version>${jackson.version}</version>
		</dependency>
```

controller文件（这个controller本就是@restcontroller）

```
@RequestMapping(value = "txml", headers = { "content-type=application/xml;charset=UTF-8"})
	public Object testXml(@RequestBody Xml xml) {

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
		Test_DtoOut tdo = new Test_DtoOut();
		tdo.code = 200;
		tdo.msg = xml.getToUserName();
		return tdo;
	}
```


两个出入参dto（只截部分代码）

```
package com.mufeng.dto.in;

import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonRootName;

@JsonRootName(value="out") // 给xml根节点别名，不然会带包名
public class Xml {
	@JsonProperty(value="toUserName")// 变量对应的入参数据标签名，这里可以使实现与入参数据标签名不同
	String toUserName;
	@JsonProperty(value="fromUserName")
	String fromUserName;
	@JsonProperty(value="CreateTime")
	Long createTime;
	@JsonProperty(value="MsgType")
	String msgType;
	@JsonProperty(value="Event")
	String event;
```


```
package com.mufeng.dto.out;

import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonRootName;
import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlRootElement;

//@JacksonXmlRootElement(localName="out")
@JsonRootName(value="out")
public class Test_DtoOut {

	@JsonProperty(value="codes")
	public Integer code;
        // 不给别名xml数据默认的标签名与属性名一致
	public String msg;
```

前端文件html（注意前端代码要放到服务器上，浏览器输入url，否则contentType不会起作用）
```
<html>  
<head>  
<meta http-equiv="Content-Type" content="application/xml; charset=UTF-8">  
<title>测试接收XML格式的数据</title>  
<script type="text/javascript" src="https://code.jquery.com/jquery-1.12.4.js"></script>  
<!-- <script type="text/javascript" src="js/json2.js"></script>  -->
<script type="text/javascript">  
$(document).ready(function(){  
    sendxml();  
});  
  
function sendxml(){  
    var xmlData = "<out><toUserName>toUser</toUserName><fromUserName>FromUser</fromUserName><CreateTime>123456789</CreateTime><MsgType>event</MsgType><Event>subscribe</Event></out>";
   
	$.ajax({
            type: "POST",
            url: "http://localhost:8080/valida/webctrl/demo/txml",
                
                // data sent is xml
				contentType: "application/xml; charset=utf-8",
                data: xmlData, 
                // data in response will expected xml
                dataType: "xml",
				processData: false,
                anysc: false,
				beforeSend: function(request) {
						
                       request.setRequestHeader('Access-Control-Allow-Origin: *');
					   request.setRequestHeader('Access-Control-Allow-Methods: POST,GET,OPTIONS');
					   request.overrideMimeType('application/xml; charset=utf-8')
					   console.log(JSON.stringify(request));
                 },
                success: function (result) { 
					console.log(result);
                },
                error: function (XMLHttpRequest, textStatus, errorThrown) {
                    alert(errorThrown + ':' + textStatus); // 错误处理
                }
            });
		
}  
</script>  
</head>  
<body>  
</body>  
</html>  
```


后端控制台打印
```
{"toUserName":"toUser","fromUserName":"FromUser","CreateTime":123456789,"MsgType":"event","Event":"subscribe"}
```
前端接收到返回值

```
<out><msg>toUser</msg><codes>200</codes></out>
```



另外一种使用xstream实现xml数据交换的配置，详见另一篇博客
[spring基于xml格式数据交换的前后端配置与使用之xstream方式](./2017-08-18_spring_xml_xstream.md)

