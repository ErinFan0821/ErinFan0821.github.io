---
layout: post
title: "Web Service In Spring Demo"
date: 2014-06-03 23:08:21 +0800
comments: true
categories: 
---
1. 什么是webService?

   Web Service技术存在的价值是：可以使运行在不同机器上的不同应用无需借助附加的，专门的第三方软件或硬件就可以相互交换数据或集成。在web Service架构中，存在着客户端，服务端。服务端通过Web Service描述语言(WSDL)来把自己提供什么样的服务发布出来，客户端通过这个wsdl就可以知道服务端提供了什么样的服务，就可以向服务端发request，服务端返回response来完成一次交互。
2. 什么是WSDL?为什么要存在WSDL?

   假如你写了一个Service, 那么你怎么向别人介绍说你的Service有什么功能，以及每个函数调用时的参数是什么呢？两种方式： 写文档，口头告诉使用的的service的人。但是，当使用这个service的人真正开始想要去调用你的服务的时候，他们使用的开发工具没法帮助开发人员快速的构建出来他们要访问的服务的代码。那么怎么办呢？这就需要一个正式的描述性文档，机器能够阅读，人也能够阅读的文档。 Web Service描述语言（Web Service Description Language）WSDL 就指这样一个基于XML的语言，用于描述Web Service及其函数，参数和返回值。机器可以直接根据WSDL生成相应的克调用的Web Service代码。
3. 客户端和服务端通过什么来进行消息的传递？

   SOAP(Simple Object Access Protocol)简单对象访问协议，它使用应用层协议作为其传输协议，大多数情况下是使用http来协助传输，也就是说，客户端发送的请求被转成soap格式之后，通过http来进行传输。它也是基于XML的。
大致了解了webService之后，可以试着去实现一个简单的webService, 包括服务端，客户端，以及服务端暴露出来的wsdl. 我们不需要去专门的写一个wsdl, 可以XSD（XML Schema Definition) 来描述我们的服务, client只要能拿到这个xsd便可知道服务端的服务长什么样子。

接下来是代码实现，可以分五步走：
Step1:写描述我们服务的XSD。 
Step2:通过jaxb框架把xsd转化成相应的java domain类。（服务端和客户端都需要做这个事情）
Step3:Server端写Endpoint来提供相应的服务。
Step4:动态生成wsdl, 可在soap ui里面测试写的server是否正确，
Step5:写client端代码，去调用之前写的server

Server 和 Client 应该放在两个不同的工程下面，先看看Server工程的目录结构：

{% img  /images/d.png  300 300%}

看起来还是比较的清晰，我是把xsd直接放在webapp这个目录下的，spring-ws-servlet.xml和web.xml是我的配置文件

Step1: 写描述我们服务的XSD。在这里是AccountDetail.xsd, 可以看看代码：
    <?xml version="1.0" encoding="UTF-8"?>
    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="http://erinfan.webservices.com"
           xmlns="http://erinfan.webservices.com"
           elementFormDefault="qualified"
           attributeFormDefault="unqualified">
    <xs:element name="Account" type="Account"/>
    <xs:complexType name="Account">
        <xs:sequence>
            <xs:element name="AccountName" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>

    <xs:element name="AccountDetailsRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="accountName" type="xs:string" />
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="AccountDetailsResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="AccountDetails" type="Account"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    </xs:schema>

这里面需要看的几个地方：

1) xmlns:xs="http://www.w3.org/2001/XMLSchema" 这里我们使用了标准的命名空间(xs), 和这个命名空间相关联的URI是Schema的语言定义，其标准值是http://www.w3.org/2001/XMLSchema, 换句话说，下面所使用的schema的默认的元素如element, sequence, type这些都是来自这个标准。

2) targetNamespace="http://erinfan.webservices.com" 这句话是指我们下面自己定义的一些类型是来自于这个命名空间。

3)  <xs:element name="Account" type="Account"/> 定义了一个名字为Account的type，对应到我们的代码里面就是一个java domain类Account

4)  
    <xs:complexType name="Account">
        <xs:sequence>
            <xs:element name="AccountName" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>

 这里定义了Account里面有类型为string的属性accountName

 5) 
    <xs:element name="AccountDetailsRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="accountName" type="xs:string" />
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="AccountDetailsResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="AccountDetails" type="Account"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

 第一个element描述了我们的server可以接收的request的name，AccountDetailRequest也可以理解为我们的请求的根标记。另外里面还描述了request里面可以传的值，在这是一个string类型的值。

 第二个element描述了我们的server会返回的的response的name，以及返回来的类型，在这是把Account类型的对象返回来。

Step2: 通过jaxb框架把xsd转化成相应的java domain类。
我在这是直接使用的intellij里面的jaxb插件来转换的,也还有其他的办法，比如用gradle.

Step3:Server端写Endpoint来提供相应的服务。

     @Endpoint
    public class AccountServiceEndpoint {
    	private static final String TARGET_NAMESPACE = "http://erinfan.webservices.com";

    	@PayloadRoot(localPart = "AccountDetailsRequest", namespace = TARGET_NAMESPACE)
    	public
    	@ResponsePayload
    	AccountDetailsResponse getAccountDetails(@RequestPayload AccountDetailsRequest request) {
        	AccountDetailsResponse response = new AccountDetailsResponse();
        	Account account = new Account();
        	account.setAccountName(request.getAccountName());
        	response.setAccountDetails(account);
        	return response;

    	}
	}

注解@Endpoint是spring提供的，用来表示这是一个server端的endpoint，我们只需要在配置文件里面加上自动扫描的配置，spring就可以帮我们找到这个endpoint来处理client端发过来的请求，这就是使用spring框架的好处，很多东西它都帮你处理了，只需要一个简单的注解便可。

注解@PayloadRoot(localPart = "AccountDetailsRequest", namespace = TARGET_NAMESPACE) 这个注解表示只要发过来的request满足这两个条件，便交给getAccountDetails()这个函数来处理，其中localPart里面的值需要与client端请求文件里面的根标记一致。namespace需要与Client端请求文件里面的targetNamespace一致.

注解@ResponsePayload和注解@RequestPayload 用来引入响应和请求的对象，也就是我们通过jaxb解析出来的两个对象AccountDetailsResponse和AccountDetailsRequest
实现部分的代码还是比较简单，把request里传递过来的accountName设置进入到new出来的Account对象里面，然后把这个对象set到response，返回response. Client端那边获取到的response就是一个包含着accountName的Account对象。

写完了Endpoint了，服务端的事情并没有结束，接下来是配置，需要配置什么呢？

第一是配置spring, 之前在endpoint里面写了注解@Endpoint, spring怎么去识别这个注解的呢？这就需要我们在spring-ws-servlet.xml这个文件里面去配置。 

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:sws="http://www.springframework.org/schema/web-services"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/web-services
                           http://www.springframework.org/schema/web-services/web-services.xsd">

    <context:component-scan base-package="com.erinfan"/>
    <sws:annotation-driven/>

    <sws:dynamic-wsdl id="account" portTypeName="Account"
                      locationUri="http://localhost:8080/webServiceAccount/accountService">
        <sws:xsd location="AccountDetails.xsd"/>
    </sws:dynamic-wsdl>

    </beans>

 其中 
    <context:component-scan base-package="com.erinfan"/>
    <sws:annotation-driven/>
 就是添加 spring的自动扫描annotation，扫描的目标是com.erinfan，也就是我的工程里面的package。

     <sws:dynamic-wsdl id="account" portTypeName="Account"
                      locationUri="http://localhost:8080/webServiceAccount/accountService">
        <sws:xsd location="AccountDetails.xsd"/>
    </sws:dynamic-wsdl>
而这段话是方便生成wsdl而添加的，只需要在浏览器里面输入http://localhost:8080/webServiceAccount/accountService/account.wsdl 便可得到通过AccountDetails.xsd转换出来的wsdl.这个wsdl之所以要生成，是为了能够使用soap ui来测试server是否work.

除了配置spring外，还需把spring-ws-servlet配置到web.xml里面。

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
         http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

    <servlet>
        <servlet-name>spring-ws</servlet-name>
        <servlet-class>org.springframework.ws.transport.http.MessageDispatcherServlet</servlet-class>
        <init-param>
            <param-name>transformWsdlLocations</param-name>
            <param-value>true</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring-ws</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    </web-app>

Step4: 动态生成wsdl, 可在soap ui里面测试写的server是否正确，

把第三步中生成的wsdl导入到soap ui里面，在soap ui自动生成的soap request里面填入参数，便可以发消息了，如果写的server没有问题，便会给返回相应的response.

Step5: 写client端代码，去调用之前写的server。

这里面需要做三件事，第一是导入server端提供的AccountDetail.xsd, 然后通过jaxb生成相应的java domain类。 第二是写client端代码来调用server. 第三是配置applicationContext.xml。
工程目录如下图：

{% img  /images/clientD.png  300 300%}

第一步很简单，不再累赘，看server里面怎么整的便可。

第二步可以看看代码先：

    package com.erinfan.client;

    import com.erinfan.domain.AccountDetailsRequest;
    import com.erinfan.domain.AccountDetailsResponse;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.FileSystemXmlApplicationContext;
    import org.springframework.ws.client.core.WebServiceTemplate;

    public class AccountClient {
    	public static void main(String[] args) {
        	ApplicationContext context = new FileSystemXmlApplicationContext("classpath:applicationContext.xml");
        	WebServiceTemplate webServiceTemplate = (WebServiceTemplate) context.getBean("webServiceTemplate");
        	AccountDetailsRequest request = new AccountDetailsRequest();
        	request.setAccountName("ErinFan");
        	AccountDetailsResponse response = (AccountDetailsResponse) webServiceTemplate.marshalSendAndReceive(request);

        	System.out.println(response.getAccountDetails().getAccountName());
    	}
	}
最主要的东西就是使用了spring提供的WebServiceTemplate去帮我们做了发消息和接收消息的工作，这又体现了框架的好处，帮我们做了封装，方便快捷，省去了很多麻烦.

想要使用WebServiceTemplate，就需要做相应的配置，也就是我们的第三步，配置applicationContext.xml,代码如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="webServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
        <property name="defaultUri" value="http://localhost:8080/webServiceAccount/defaultUri"/>
        <property name="marshaller" ref="marshaller"/>
        <property name="unmarshaller" ref="unmarshaller"/>

    </bean>

    <bean id="marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
        <property name="classesToBeBound">
            <list>
                <value>com.erinfan.domain.AccountDetailsRequest</value>
            </list>
        </property>
    </bean>

    <bean id="unmarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
        <property name="classesToBeBound">
            <list>
                <value>com.erinfan.domain.AccountDetailsResponse</value>
            </list>
        </property>
    </bean>
	</beans>

这里面主要的东西就是marshaller和unmarshaller了。
WebServiceTemplate 会帮我们把我们发出去的request做一个marshaller的工作，转换成soap消息发送出去。unmarshaller是将server端的soap response转换成我们在client端可识别和操作的对象.

到这里为止就完成了一个简单的spring web service的demo的全部代码工作了。

接下来只需要将服务部署，然后通过客户端去访问就可以了。我在这里使用的是jetty容器来部署我的服务，然后客户端直接运行main函数便可调用server。

