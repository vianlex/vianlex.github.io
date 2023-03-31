---
title: Dom4j 入门笔记
---

## Maven 引入依赖包
```xml
<!-- https://mvnrepository.com/artifact/org.dom4j/dom4j -->
<dependency>
    <groupId>org.dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>2.1.4</version>
</dependency>
<!-- xpath  -->
<dependency>
    <groupId>jaxen</groupId>
    <artifactId>jaxen</artifactId>
    <version>2.0.0</version>
</dependency>
<!-- 测试 -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.1</version>
</dependency>

```

## 创建 XML
```java
package com.github.vianlex.xml.dom4j;

import java.net.URL;
import java.util.Iterator;
import java.util.List;

import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.DocumentHelper;
import org.dom4j.Element;
import org.dom4j.Node;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.SAXReader;
import org.dom4j.io.XMLWriter;
import org.junit.jupiter.api.Test;

public class Dom4jTest {

    /**
	 * 构建 xml 
	 * <?xml version="1.0" encoding="UTF-8"?>
	 * <beans> 
	 * 		<bean id="userService" class="com.github.vianlex.service.impl.UserServiceImpl">
	 * 			<property name="userDao" ref="userDao"/>
	 *      </bean>
	 *      <bean id="roleService" class="com.github.vianlex.service.impl.RoleServiceImpl"> 
	 *      	<property name="roleDao" ref="roleDao"></property>
	 *      </bean>
	 *      <bean id="roleDao" class="com.github.vianlex.dao.RoleDao"></bean>
	 *      <bean id="userDao" class="com.github.vianlex.dao.UserDao"></bean>
	 * <beans>
	 * 
	 */
    @Test
	public void createXml() {

		Document document = DocumentHelper.createDocument();
		// 创建 <beans> 标签
		Element root = document.addElement("beans");
		// 创建 <beans> 的子标签 <bean>
		Element userServiceBean = root.addElement("bean").addAttribute("id", "userService").addAttribute("class",
				"com.github.vianlex.service.impl.UserServiceImpl");
		// 创建 <bean> 的子标签 <property>
		userServiceBean.addElement("property").addAttribute("name", "userDao").addAttribute("ref", "userDao");

		Element roleServiceBean = root.addElement("bean").addAttribute("id", "roleService").addAttribute("class",
				"com.github.vianlex.service.impl.RoleServiceImpl");
		roleServiceBean.addElement("property").addAttribute("name", "roleDao").addAttribute("ref", "roleDao");

		Element userDao = root.addElement("bean").addAttribute("id", "roleDao").addAttribute("class",
				"com.github.vianlex.dao.RoleDao");

		Element roleDao = root.addElement("bean").addAttribute("id", "userDao").addAttribute("class",
				"com.github.vianlex.dao.UserDao");

		System.out.println(document.asXML());

		System.out.println(" ============================================== ");
		// 美化打印 xml
		try {
			OutputFormat format = OutputFormat.createPrettyPrint();
			XMLWriter writer = new XMLWriter(System.out, format);
			writer.write(document);
			
		} catch (Exception e) {
			// TODO: handle exception
		}

		System.out.println(" ============================================== ");
		
		// 紧凑型打印 xml
		try {
			// Compact format to System.out
			OutputFormat format = OutputFormat.createCompactFormat();
			XMLWriter writer = new XMLWriter(System.out, format);
			writer.write(document);
			writer.close();
		} catch (Exception e) {
			// TODO: handle exception
		}

	}
}

```

## 将文本 xml 写文件
```java
FileWriter out = new FileWriter("foo.xml");
document.write(out);
out.close();
```


## 解析 XML 
1. 解析文本 xml
```java
@Test
public void getDocumentByText() {
    String xml = "<beans><bean id=\"userService\" class=\"com.github.vianlex.service.impl.UserServiceImpl\"><property name=\"userDao\" ref=\"userDao\"/></bean><bean id=\"roleService\" class=\"com.github.vianlex.service.impl.RoleServiceImpl\"><property name=\"roleDao\" ref=\"roleDao\"/></bean><bean id=\"roleDao\" class=\"com.github.vianlex.dao.RoleDao\"/><bean id=\"userDao\" class=\"com.github.vianlex.dao.UserDao\"/></beans>";
    try {
        Document document =  DocumentHelper.parseText(xml);
        System.out.println(document.asXML());
    } catch (DocumentException e) {
        throw new RuntimeException("解析文本 xml 出错", e);
    }
}

```

2. 解析文件中的 xml
```java
@Test
public void getDocumentByXmlFile() {
    String fileName = "spring-context.xml";
    SAXReader saxReader = new SAXReader();
    try {
        Document document = saxReader.read(this.getClass().getResourceAsStream(fileName));
        System.out.println(document.asXML());
    } catch (DocumentException e) {
        throw new RuntimeException("解析 xml 文件出错", e);
    }

}
```

## 解析 document
```java

/**
	 * 迭代方式解析 document
	 */
@Test
public void parseDocumentByIter() throws Exception{
    
    String xml = "<beans id=\"helloRoot\"><bean id=\"userService\" class=\"com.github.vianlex.service.impl.UserServiceImpl\"><property name=\"userDao\" ref=\"userDao\"/></bean><bean id=\"roleService\" class=\"com.github.vianlex.service.impl.RoleServiceImpl\"><property name=\"roleDao\" ref=\"roleDao\"/></bean><bean id=\"roleDao\" class=\"com.github.vianlex.dao.RoleDao\"/><bean id=\"userDao\" class=\"com.github.vianlex.dao.UserDao\"/></beans>";
    Document document =  DocumentHelper.parseText(xml);
    
    // root elements
    Element root = document.getRootElement();

    // iterate through child elements of root
    for (Iterator<Element> it = root.elementIterator(); it.hasNext();) {
        Element element = it.next();
        System.out.println("元素标签名："+element.getName()+"，标签属性：(id = '"+element.attributeValue("id") +"', class='"+element.attributeValue("class")+"')" );
    }

    System.out.println(" ================================================= ");
    // iterate through child elements of root with element name "bean"
    for (Iterator<Element> it = root.elementIterator("bean"); it.hasNext();) {
        Element element = it.next();
        System.out.println("元素标签名："+element.getName()+"，标签属性：(id = '"+element.attributeValue("id") +"', class='"+element.attributeValue("class")+"')" );
    }

    System.out.println(" ====================================== ");
    // iterate through attributes of root
    for (Iterator<Attribute> it = root.attributeIterator(); it.hasNext();) {
        Attribute attribute = it.next();
        System.out.println(attribute.getName() + " = " + attribute.getValue());
    }
}



/**
    * xpath 方式解析 document
    * @throws Exception 
    */
@Test
public void parseDocumentByXpath() throws Exception {
    
    String xml = "<beans><bean id=\"userService\" class=\"com.github.vianlex.service.impl.UserServiceImpl\"><property name=\"userDao\" ref=\"userDao\"/></bean><bean id=\"roleService\" class=\"com.github.vianlex.service.impl.RoleServiceImpl\"><property name=\"roleDao\" ref=\"roleDao\"/></bean><bean id=\"roleDao\" class=\"com.github.vianlex.dao.RoleDao\"/><bean id=\"userDao\" class=\"com.github.vianlex.dao.UserDao\"/></beans>";
    Document document =  DocumentHelper.parseText(xml);
    // 查找 <bean> 元素
    List<Node> beanNodeList = document.selectNodes("//bean");
    for (Node node : beanNodeList) {
        Element element = (Element) node;
        System.out.println(element.getName()+ ", " + element.attributeValue("id") + element.attributeValue("class"));
    }
    // 查找 <property> 元素
    List<Node> propertyNodeList = document.selectNodes("//property");
    for (Node node : propertyNodeList) {
        Element element = (Element) node;
        System.out.println(element.getName()+ ", " + element.attributeValue("name") + element.attributeValue("ref"));
    }

}

```


## 参考链接
1. https://dom4j.github.io/#top