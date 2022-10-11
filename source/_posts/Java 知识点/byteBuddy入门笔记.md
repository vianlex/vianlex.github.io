---
title: byteBuddy 入门笔记
---
# ByteBuddy 概述
ByteBuddy 是一个可以在运行时动态生成java class的类库。

# 依赖
在使用 ByteBuddy 之前我们需要引入对应的 Jar 到工程中，如下

```xml
<dependency>
  <groupId>net.bytebuddy</groupId>
  <artifactId>byte-buddy</artifactId>
  <version>1.12.17</version>
</dependency>
```

# 入门例子
## JVM 运行时创建 Java 类

```java
    @Test
	public void test01() throws Exception {
		ByteBuddy byteBuddy = new ByteBuddy();
		// 创建一个继承 Object 的子类
		Unloaded<Object> unloadedClass = byteBuddy.subclass(Object.class)
				// 使用 ElementMatchers 选择需要重写的父类方法
				.method(ElementMatchers.isToString())
				// 重写父类的 toString 方法，固定返回如下值
				.intercept(FixedValue.value("Hello World ByteBuddy!"))
				// 生成子类
				.make();
		// 将加载到 JVM 中, 并获取加载到 JVM 中的 子类的 Class 对象
		Class<? extends Object> loaded = unloadedClass.load(getClass().getClassLoader()).getLoaded();
		// 根据子类的 Class 对象实例化子类
		Object newInstance = loaded.newInstance();
		// 打印子类，会在终端输出  Hello World ByteBuddy!
		System.out.println(newInstance);
	}
```

## 
















## 参考连接
1、https://www.baeldung.com/byte-buddy