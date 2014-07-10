---
layout: post
title: "Jstl core和fmt标签的使用"
date: 2014-06-08 17:49:45 +0800
comments: true
categories: 
---
1. 什么是JSTL?

   JSP标准标记库(JSP Standard Tag Library, JSTL)是一个实现Web应用程序中常见的通用功能的定制标记库集，这些功能包括迭代和条件判断，书库管理格式化，XML操作以及数据库的访问。
   
2. JSTL帮我们解决了什么问题？

   JSTL帮组我们把底层内容模型与表示进行了分离。
   在多数大型开发工程中，程序员负责实现后端，表示则留给一名或多名 Web 页面设计人员去做。在引入 JavaServer Pages（JSP）规范的版本 1.1 中的定制标记库之前，必须使用 JSP 脚本元素来在 JSP 页面内提供任意的定制功能性。显式地使用脚本元素违背了模型与表示相分离的原则。显式地使用脚本元素还要求，如果 Web 页面设计人员要做任何“从 JavaBean 组件中检索属性”之外的事情，他就要具有 Java 编程经验。这引起了人们对 JSP 页面内脚本元素的使用的广泛关注，这也就是JSTL存在的目的，我们的web开发人员不需要有java编程经验也可以写前端代码。

3. JSTL标签库有哪些？

   在第一点里面说过JSTL是实现常见功能的定制标记库集，这些功能包括迭代和条件判断，书库管理格式化，XML操作以及数据库的访问。JSTL1.0 发布了四个定制标记库（core、format、xml 和 sql）和一堆通用标记库验证器（ScriptFreeTLV 和 PermittedTaglibsTLV）。 四个定制标记库的作用分别是： 1. core标记库提供了定制操作，通过限制了作用域的变量管理数据，以及执行页面内容的迭代和条件操作。它还提供了用来生成和操作URL的标记。2. format标记库定义了用来格式化数据（特别是数字和日期）的操作。它还支持使用本地化资源束来进行JSP页面的国际化。3. xml库包含一些标记，这些标记用来操作通过xml表示的数据。4. sql库定义了用来查询关系数据库的操作。两个JSTL标记库验证器允许开发人员在其jsp应用程序中强制的使用标准编码。1. 通过配置ScripFreeTLV验证器可以在JSP页面中禁用各种类型的JSP脚本元素--scriptlet,表达式和声明。 2. PermittedTaglibsTLV 验证器可以用来限制可能由应用程序的jsp页面访问的定制标记库集(包括JSTL标记库)。
   以上是对jstl标记里面包含的内容做简单的介绍，由于在项目开发过程中使用较多的是core和fmt，所以这里的demo主要是core和fmt.

4. JSTL Demo

   我在这是使用了spring MVC来搭建了一个简单的项目来做页面的demo，可以看一下我的目录结构。
   
   {% img  /images/dd.png  400 400%}
   
其中jstlTestPage.jsp是用来demo显示的页面
jstlLearningController里面可以构造一些java对象，通过Modle来往jsp页面传值。
整个项目通过gradle来构建，jetty来部署。

这个demo里面我们主要关注jsp页面。

4.1 JSTL core 标签Demo(JSTL c)
   想要使用jstl需要在我们的工程文件里面引入相应的包, 我在这使用的是1.2的版本： jstl-1.2.jar。
   想要使用core标签，还需要在这个jsp页面加入：
   
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   
   1> c:out 用于显示数据对象（字符串、表达式）的内容或结果
   
   使用java脚本的方式为: <% out.println("hello") %> <% =表达式>
   使用JSTL标签: <c:out value="字符串">
   
   JSP页面代码：
   
	<c:out value="I am a nomal string" default="defaultValue"></c:out><br>
	<c:out value="I am a string with special char: &lt&lt&lt&lt" escapeXml="true"></c:out><br>
	<c:out value="I am a string with special char: &lt&lt&lt&lt"></c:out><br>
	<c:out value="I am a string with special char: &lt&lt&lt&lt" escapeXml="false"><、c:out><br>
	<c:out value="${null}">default string</c:out>
	
   第1行代码也等同于<c:out value="I am a nomal string">defaultValue</c:out><br>
   value：表示要输出的值，
   default：表示如果value为null，则输出defaultValue, 参考第5行代码。
   第2，3，4行主要目的是显示escapeXml的用途，表示是否转换特殊字符，为true不转换，为false则转换，默认值是true.
   下面是页面输出的样子：
   
   {% img  /images/cout.png  500 600%}
   
   2> c:set 用于保存数据。
   
	<c:set value="ErinFan" var="myName" scope="session"/>
	<c:set var="name2" scope="session">Reg</c:set>
	<c:set var="name2" scope="request">Reg</c:set>
	<li>get name from session: ${sessionScope.myName}</li>
	<li>get second name from the session: ${sessionScope.name2}</li>
	
   value: 要被存储的值
   var: value存入的变量的名字
   scope:var变量的jsp范围，在这里为session, 另外还有application, page.
   
   页面：
   
   {% img  /images/cset.png  500 600%}
   
   3> c:remove 用来移除范围变量
   
	<c:out value="c:remove demo"/><br>
	<c:remove var="name2" scope="session"/>
	<c:out value="${sessionScope.name2}">name2 already been removed from session</c:out><br>
	
   可以看到name2已经被移除，所以会输出default值
   
   {% img  /images/cremove.png  500 600%}
   
   4> c:catch 用来捕获嵌套在这个标签里面的操作所抛出来的异常对象，并将异常信息保存到变量中。
   
   5> c:if 用于实现java中的if语句的功能
   
	<c:set value="10000" var="account1"/>
	<c:set value="20000" var="account2"/>
	<c:if test="${account1 == '10000'}" var="account1TestResult"></c:if>
	<c:out value="account1 test value is: ${account1TestResult}"></c:out><br>
	<c:if test="${account2 == '10000'}" var="account2TestResult"></c:if>
	<c:out value="account2 test value is: ${account2TestResult}"></c:out><br>
	
   第1，2句设置了两个变量account1, account2, 第3句用if做了判断，判断的结果放入变量account1TestResult里，第4句把上面判断的结果输出页面，第5句和第6句也是判断和输出，这个和java代码很像，应该很容易看懂。
   
   {% img  /images/cif.png  400 500%}
   
   
   6> c:choose  和 <c:when> 、 <c:otherwise> 一起实现互斥条件执行，类似于 java 中的 if else.
<c:choose> 一般作为 <c:when> 、 <c:otherwise> 的父标签。

	<c:set var="scope" value="90"></c:set>
	<c:choose>
    	<c:when test="${scope >= 90}">
        	Congratulations!
    	</c:when>
    	<c:when test="${scope >= 80 && scope < 90}">
        	Just So So!
    	</c:when>
    	<c:when test="${scope >= 70 && scope < 80}">
        	Need more hard work!
    	</c:when>
    	<c:otherwise>
        	Sorry, you do not pass the exam.
    	</c:otherwise>
	</c:choose>
	
   第一句话设置了scope变量值为90，然后进入choose里面去做判断。最后输出结果如下图：

   {% img  /images/cchoose.png  500 600%}
   
   7> c:forEach 迭代标签
   
	<c:forEach var="account" items="${accountList}">
    	<c:out value="${account}"/>
	</c:forEach>
	<br><br>
	<c:out value="Use begin and end to do foreach"/><br>
	<c:forEach var="account" items="${accountList}" begin="1" end="2" step="1">
    	<c:out value="${account}"/>
	</c:forEach>
   forEach可以使用两种不同的写法，第一个forEach就是第一种，默认的会从头到尾挨个循环输出。
   第二个foreach是使用了begin，end, step属性，begin表示开始循环的位置，end表示结束循环的位置， step表示循环一次跳跃多少个index. 
   这两个的输出如下图：
   
   
   {% img  /images/cforeach1.png  300 400%}
   
   c:foreach标签里面有个属性varStatus，用来表示当前foreach的状态，可以看下下面的例子：
   
	<c:forEach var="account" items="${accountList}" begin="0" end="3" step="2" varStatus="attribute">
    	<br>
    	Current value: <c:out value="${account}"/><br>
    	Current Index: <c:out value="${attribute.index}"/><br>
    	Foreach times: <c:out value="${attribute.count}"/><br>
    	Is the first index: <c:out value="${attribute.first}"/><br>
    	Is the last index: <c:out value="${attribute.last}"/><br>
	</c:forEach>
	
   这个循环开始的位置是0，结束的位子是3，一次跳2步，varStatus变量名字为attribute,用这个变量名就可以调用这个属性的值：index表示当前循环的index; count表示当前循环是第几次循环; first表示当前循环是否是第一次，如果不是返回false,是返回true; last表示的意思和first正好相反，表示判断当前循环是否是最后一次。
页面输入如下图：
   
   {% img  /images/cforeach2.png  300 400%}
   
   8> c:forTokens 用于对带有相同符合格式内容进行分割输出，例如var = "1,2,3,4", 以逗号为分隔
   
	<c:forTokens items="1,2,3,4" delims="," var="result">
    	<c:out value="${result}"/>
	</c:forTokens>
	<br>
	<c:forTokens items="A#B#C" delims="#" var="result" varStatus="attribute"><br>
    	Current Token: <c:out value="${result}"/><br>
    	Current Index: <c:out value="${attribute.index}"/><br>
	    Foreach Times: <c:out value="${attribute.count}"/><br>
    	Is the first index: <c:out value="${attribute.first}"/><br>
    	Is the last index: <c:out value="${attribute.last}"/><br>
	</c:forTokens>
	
   items表示被循环遍历的对象，一般为数组和集合类
   delims表示分隔符
   这里也有begin,end,step用来控制位置和循环步长的属性。
   varStatus表示迭代当前的状态。
   
   输出如图：
   
   {% img  /images/cfortokens.png  300 400%}
   
   
   9> c:import 把其他静态或者动态文件包含到jsp页面，可以包含其他web应用中的文件，甚至是网络上的资源。
   
	<c:out value="c:import demo"/><br>
	<c:catch var="error">
		<c:import url="http://sdfsadfaf"/>
	</c:catch>
	<c:out value="${error}"/><br>
	
   这里可以看到c:catch的用法，如果页面没法打开，会抛出异常，最后一句话用于输出异常信息，页面如下图：
   
   {% img  /images/ccatch.png  300 400%}
   
   如果url正确，则会打开url，如下面直接就打开了百度的页面。
   
	<c:out value="c:import demo"/><br>
	<c:catch var="error">
		<c:import url="http://www.baidu.com"/>
	</c:catch>
	<c:out value="${error}"/><br>
	
   c:import还可以通过本地的url打开相应的文件：
   
	<c:catch var="error">
    	First import: <c:import url="test.txt"/>
	</c:catch>
	<c:out value="${error}"/><br>
	<c:catch var="error">
    	<c:import url="test.txt" var="myurl" scope="session" charEncoding="gbk"/><br>
    	Second import: <c:out value="${myurl}"/>
	</c:catch>
	
   test.txt是我放在和当前jsp同一目录下的文件，通过上面的代码，直接就帮我把这个文件里面的内容输出到页面上来了，也有两种不同的输入方式，可以放到变量里面，也可是直接输出。如下图：
   
   {% img  /images/cimport.png  300 400%}
   
   
   10> c:redirect 该标签用来实现请求的重定向。例如，对用户输入的用户名和密码进行验证，不成功则重定向到登录页面。或者实现Web应用不同模块之间的衔接
   
	<c:out value="c:redirect demo"/><br>
	<c:redirect url="http://localhost:8080">
    	<c:param name="username" value="fakeUser"/>
    	<c:param name="password" value="fakePsw"/>
	</c:redirect>
	
   上面这段代码运行后悔跳转为： http://localhost:8080/?username=fakeUser&password=fakePsw
	
   11> c:url 用于动态生成一个 String 类型的URL，可以同上个标签共同使用，也可以使用HTML的a标签实现超链接
   
	<c:url value="http://localhost:8080/jstlLearning" var="url" scope="session"/>
	<a href="${url}">Open Url</a><br>
	
   页面显示如下，点击Open Url会跳转到http://localhost:8080/jstlLearning页面
   
   {% img  /images/openurl.png  300 400%}
   
   
4.2  JSTL format标签demo(JSTL fmt)
   fmt标签主要就是用来格式化。
   1> fmt:formatDate value表示要格式化的值， pattern表示要格式化的格式
   
	<fmt:formatDate value="${date}" pattern="yyyy-MM-dd HH:mm:ss"/><br>
    
   2> fmt:formatNumber 有三种格式化的type，number(数字), percent(百分比), currency(货币) 
    
    <fmt:formatNumber value="145.65464546" pattern="0.00"/><br>
	<fmt:formatNumber type="number" value="4534353.45353"/><br>
	<fmt:formatNumber type="percent" value="0.6789"/><br>
	
	
   {% img  /images/fmt.png  300 400%}
   
   
   在这里面其实还用了jsp EL(Expression Language).
   表达式语言的灵感来自于 ECMAScript 和 XPath 表达式语言，它提供了在 JSP 中简化表达式的方法。它是一种简单的语言，基于可用的命名空间（PageContext 属性）、嵌套属性和对集合、操作符（算术型、关系型和逻辑型）的访问符、映射到 Java 类中静态方法的可扩展函数以及一组隐式对象。EL 提供了在 JSP 脚本编制元素范围外使用运行时表达式的功能。脚本编制元素是指页面中能够用于在 JSP 文件中嵌入 Java 代码的元素。它们通常用于对象操作以及执行那些影响所生成内容的计算。JSP 2.0 将 EL 表达式添加为一种脚本编制元素。
   我们上面的demo中类似${expression}这样的表达就是使用的EL，将会在下一篇博客中详细的介绍。
   
   本文章参考url: 
   
   http://www.ibm.com/developerworks/cn/java/j-jsptags/
   
   http://www.cnblogs.com/cliffever/archive/2008/11/13/1333025.html
   
   http://elf8848.iteye.com/blog/245559
   
   源码下载地址：
   
   https://github.com/ErinFan0821/jstlLearning.git
   
	 


