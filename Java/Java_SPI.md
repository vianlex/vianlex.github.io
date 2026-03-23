# Java SPI 机制

## 简介

SPI（Service Provider Interface），是 JDK 内置的一种服务提供发现机制，为接口寻找实现类的机制。通过这种机制可以实现插件化开发，能做到插件的热拔插。

SPI 能将服务接口和具体的服务实现分开解耦，应用层（客户端）只需要基于接口编程，然后按需引入具体接口的实现服务（接口实现 Jar），通过资源文件 META-INF/services 指定需要加载的具体实现类。

SPI 实际上是「基于接口的编程＋策略模式＋配置文件」组合实现基础接口编程，然后动态加载具体实现类的机制，即只有真正运行调用接口对象时，在去查找加载实现类和创建实例对象。


## SPI 规范约定

要使用 Java SPI，需要遵循如下约定：

1. 接口要使用的实现类，需要资源目录 META-INF/services 目录下创建一个以「接口全限定名」为命名的文件，内容为实现类的全限定名，指定多个实现需要回车换行; 
2. 引入的接口实现类 jar 包，要放到应用程序的 classpath（JVM 能找到类的路径） 中; 
3. 应用程序通过 java.util. ServiceLoder 动态装载实现类，它通过扫描 META-INF/services 目录下的配置文件找到实现类的全限定名，把类加载到 JVM; 
4. SPI 的实现类必须携带一个不带参数的构造方法。


## 简单示例-1

1. 首先需要定义一个接口

```java
package com.spi.service;
public interface SearchService {
	String searchSku(String name);
}
```

2. 根据接口编写两个实现类

```java
package com.spi.service.impl;
import com.spi.service.SearchService;
public class JdSearchServiceImpl implements SearchService{
	@Override
	public String searchSku(String name) {
		return "京东商品"+name+" sku : JD20210415001";
	}
}

package com.spi.service.impl;
import com.spi.service.SearchService;
public class TaobaoSearchServiceImpl implements SearchService {
	@Override
	public String searchSku(String name) {
		return "淘宝商品" + name + "sku : TB20210415001";
	}
}
```

3. 在 resources 目录下新建 META-INF/services/ 目录，然后创建一个名为 com.spi.service.SearchService 的文件，在文件中接口需要使用的实现类，内容如下：

```text
com.spi.service.impl.JdSearchServiceImpl
com.spi.service.impl.TaobaoSearchServiceImpl
```

4. 创建服务客户端测试类

```java 
package com.spi;
import java.util.Iterator;
import java.util.ServiceLoader;
import com.spi.service.SearchService;

public class TestJavaSPI {
	public static void main(String[] args) {
        /** ServiceLoader 会将 META-INF/services/com.spi.service.SearchService 中的指定要的实现类加载到 JVM 中。
         * 通过 SPI 机制可以做到编码代码完全基于接口编程，在编码阶段不需要引入依赖以及 import 接口的实现类，
         * 而是在 JVM 运行过程中调用该接口时， ServiceLoader 才根据 /META-IFNO/service 目录的配置，去 classpath 查找和动态加载要使用的实现类。
         * 实现类可以写在当前工程源码中，也可以作为依赖 jar 引入，按实际情况来。 
        **/
		ServiceLoader<SearchService> services = ServiceLoader.load(SearchService.class);
        // 通过迭代的方式获取已经加载到 JVM 的实现类
		Iterator<SearchService> iterator = services.iterator();
		while (iterator.hasNext()) {
            /** 当调用 next 方式，会通过 class.forName 的方式去实例化 SearchService 的实现类
             * 注意 SearchService 的实现类必须要有无参的构造方法，才能实例化，不然会报错，因为 class.forName 
             * 调用的是无参数构造方法实例化对象的
            **/
			SearchService search = iterator.next();
			String desc = search.searchSku("苹果");
			System.out.println(desc);
		}
	}
}
```

## 简单实例-2

利用 SPI + 工厂模式实现，根据不同的依赖实现动态代理

```java

// 要代理的对象接口
package com.github.vianlex.service;
public interface UserService {
	public void sayHello();
}

// 要代理的对象实现类
package com.github.vianlex.service.impl;
import com.github.vianlex.service.UserService;
public class UserServiceImpl implements UserService {
	public void sayHello() {
		System.out.println("Hello I am user service!");
	}
}

// 代理抽象工厂
package com.github.vianlex.proxy;
import java.util.Iterator;
import java.util.ServiceConfigurationError;
import java.util.ServiceLoader;

public abstract class ProxyFactory {

	public abstract <T> T createProxy(T target);

	public static ProxyFactory create() {
        // 通过 SPI 在创建代理工厂时，不需要硬编码 import 具体的实现类，而是通过 ServiceLoader 去加载和实例化具体的实现类
		ServiceLoader<ProxyFactory> serviceLoader = ServiceLoader.load(ProxyFactory.class);
		Iterator<ProxyFactory> iterator = serviceLoader.iterator();
		while (iterator.hasNext()) {
			try {
				// 如果实例化成功直接返回第一个实现类
				return iterator.next();
			} catch (ServiceConfigurationError ignore) {
				// ignore
			}
		}
		return null;
	}
}

// 代理测试客户端
package com.github.vianlex.proxy;
import com.github.vianlex.service.UserService;
import com.github.vianlex.service.impl.UserServiceImpl;

public class HelloProxyClient {
	public static void main(String[] args) {
		UserService userService = new UserServiceImpl();
		UserService userService2 =  ProxyFactory.create().createProxy(userService);
		userService2.sayHello();
	}
}

```

在资源目录下，创建 SPI 约定的配置文件 META-INF/services/com.github.vianlex.proxy.ProxyFactory 文件内容如下：

```text
com.github.vianlex.proxy.impl.JdkProxyFactory
com.github.vianlex.proxy.impl.CglibProxyFactory
```

ProxyFactory 接口的具体实现类（实现类编码阶段引入全部依赖的jar，打包给其他应用使用时，可以按需引入依赖的jar），如下

```java

package com.github.vianlex.proxy.impl;
import java.lang.reflect.Method;
import com.github.vianlex.proxy.ProxyFactory;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class CglibProxyFactory extends ProxyFactory implements MethodInterceptor {

	/**
	 * 被代理的对象
	 */
	private Object target;

	@Override
	@SuppressWarnings("unchecked")
	public <T> T createProxy(T target) {
		this.target = target;
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(target.getClass());
		enhancer.setCallback(this);
		return (T) enhancer.create();

	}

	@Override
	public Object intercept(Object obj, Method method, Object[] params, MethodProxy proxy) throws Throwable {
		System.out.println("调用前" + obj.getClass());
		System.out.println(method.getName() + " : " + proxy.toString());
		Object result = method.invoke(target, params);
		System.out.println(" 调用后" + result);
		return result;
	}

}


package com.github.vianlex.proxy.impl;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

import com.github.vianlex.proxy.ProxyFactory;

public class JdkProxyFactory extends ProxyFactory {

	@Override
	@SuppressWarnings("unchecked")
	public <T> T createProxy(T target) {
		// 获取被代理类的接口
		Class<?>[] clz = target.getClass().getInterfaces();
		return (T) Proxy.newProxyInstance(this.getClass().getClassLoader(), clz, new InvocationHandler() {
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				System.out.println("============进入代理类的方法===============");
				System.out.println("我代理的方法是：" + method.getName());
				// 调用被代理对象的方法
				Object result = method.invoke(target, args);
				System.out.println("============被代理的方法已经执行==========");
				return result;
			}
		});

	}

}


```


## SPI 的优缺点

优点：

-  使用 Java SPI 机制的优势是实现解耦，使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离，而不是耦合在一起。应用程序可以根据实际业务情况启用框架扩展或替换框架组件。

缺点：

- 颗粒度不够细，不能按需加载需要的实现类，只能遍历所有实现类，才能找到我们需要的实现类。如果你并不想用某些实现类，它也被加载并实例化了，这就造成了浪费。
- 获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。
- 多个并发多线程使用 ServiceLoader 类的实例是不安全的。



## ServiceLoader 加载实现类的原理

1. 通过 URL 工具类从 jar 包的 /META-INF/services 目录下面找到对应的文件，
2. 读取这个文件的名称找到对应的 spi 接口，
3. 通过 InputStream 流将文件里面的具体实现类的全类名读取出来，
4. 根据获取到的全类名，先判断跟 spi 接口是否为同一类型，如果是的，那么就通过反射的机制构造对应的实例对象，
5. 将构造出来的实例对象添加到 Providers 的列表中。