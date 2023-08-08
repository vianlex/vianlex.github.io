# ByteBuddy 入门笔记

# ByteBuddy 概述

ByteBuddy 是一个可以在运行时动态生成 java class 的类库。

# 依赖

在使用 ByteBuddy 之前我们需要引入对应的 Jar 到工程中，如下

```xml
<dependency>
  <groupId>net.bytebuddy</groupId>
  <artifactId>byte-buddy</artifactId>
  <version>1.12.17</version>
</dependency>
```
# 根据[官网](https://bytebuddy.net)例子入门

## JVM 运行时创建 Java 类

```java
class HellWorld{
	@Test
	public void testBuddy() throws Exception {
		ByteBuddy byteBuddy = new ByteBuddy();
		// 创建一个继承 Object 的子类
		Unloaded<Object> unloadedClass = byteBuddy.subclass(Object.class)
				// 使用 ElementMatchers 选择需要重写的父类方法
				.method(ElementMatchers.named("toString"))
				// 重写父类的 toString 方法，固定返回如下值
				.intercept(FixedValue.value("Hello World ByteBuddy!"))
				// 定义子类的名称(包含 package 路径)，如果不定义 bytebuddy 会自动生成 net.bytebuddy.renamed.xxxx 名称
				.name("hello.Test")
				// 生成子类
				.make();
		// 将加载到 JVM 中, 并获取加载到 JVM 中的 子类的 Class 对象
		Class<? extends Object> loaded = unloadedClass.load(getClass().getClassLoader()).getLoaded();
		// 通过反射把子类的 Class 对象实例化
		Object newInstance = loaded.newInstance();
		// 打印子类，会在终端输出  Hello World ByteBuddy!
		System.out.println(newInstance);
		// 打印类型，因为定义名称，所以会输出 class hello.Test
		System.out.println(newInstance.getClass());
	}
}
```
## 动态类的命名说明

使用 [name|with] 方法, 可以指定动态类的名称，当两者同时设置时 with 方法的优先级更高

```java
class NameTest{
	@Test
	public void testName() {
		DynamicType.Unloaded<?> dynamicType = new ByteBuddy()
				  .with(new NamingStrategy.AbstractBase() {
				    @Override
				    protected String name(TypeDescription superClass) {
				        return "i.love.ByteBuddy." + superClass.getSimpleName();
				    }
				  })
				  .withNamingStrategy(new NamingStrategy.SuffixingRandom("suffix"));
				  .subclass(Object.class)
				  .name("hello.dynamicTest")
				  .make();
		// 输出 class net.bytebuddy.dynamic.DynamicType$Default$Unloaded
		System.out.println(dynamicType.getClass());
	}
}

```
## 动态类重写父类方法

ByteBuddy 动态生成的类, 需要重写父类方法时， 使用 method 配合 ElementMatcher 选择器去匹配父类的方法，然后通过 intercept 指定重写方法的实现，如下：

```java
class Foo {

	public String bar() {
		return null; 
   	}
  	public String foo() {
	 	return null; 
  	}
  	public String foo(Object o) { 
		return null; 
	}
}
 
class MethodTest {

	@Test
	public void testOverride() throws Exception {
		// 定义一个动态类继承 Foo 类
		Unloaded<Foo> unloadedClass = new ByteBuddy().subclass(Foo.class)
				/** 使用 method 重写 Foo 类中的方法，使用 ElementMatchers 选择要重写那个方法，isDeclaredBy 表示把 Foo 类的全部方法都重写
				**/
				.method(ElementMatchers.isDeclaredBy(Foo.class))
				// intercept 指定重写方法的实现，FixedValue.value 表示重写的方法全部返回 Hello Method!
				.intercept(FixedValue.value("Hello Method!"))
				.make();
		// unloadedClass 加载到 JVM 中并通过反射实例化
		Foo dynamicFoo = unloadedClass.load(getClass().getClassLoader()).getLoaded().newInstance();
		// 输出 Hello Method!
		System.out.println(dynamicFoo.bar());
		// 输出 Hello Method!
		System.out.println(dynamicFoo.foo()); 
		// 输出 Hello Method!
		System.out.println(dynamicFoo.foo("")); // 输出 One!
	}

	/**
		多个 method 是堆栈式匹配的，新的 method 在栈顶，会有优先使用 ElementMatchers 匹配方法，匹配结束则出栈，使用下一个 method 的 ElementMatchers 进行匹配，父类的方法被前一个 method 匹配到后，后面的 metod 不再匹配，只能匹配一次。
	*/
	@Test
	public void testOverride() throws Exception{
		Unloaded<Foo> unloadedClass = new ByteBuddy().subclass(Foo.class)
				.method(ElementMatchers.isDeclaredBy(Foo.class)).intercept(FixedValue.value("One!"))
				.method(ElementMatchers.named("foo")).intercept(FixedValue.value("Two!"))
				.method(ElementMatchers.named("foo").and(ElementMatchers.takesArguments(1)))
				.intercept(FixedValue.value("Three!")).make();
		// unloadedClass 加载到 JVM 中并通过反射实例化
		Foo dynamicFoo = unloadedClass.load(getClass().getClassLoader()).getLoaded().newInstance();
		// 打印重写方法返回的值
		System.out.println(dynamicFoo.bar()); // 输出 One!
		System.out.println(dynamicFoo.foo()); // 输出 Two!
		System.out.println(dynamicFoo.foo("Hello Wold")); // 输出 Three!
		
		Unloaded<Foo> unloadedClass02 = new ByteBuddy().subclass(Foo.class)
				.method(ElementMatchers.named("foo")).intercept(FixedValue.value("Two!"))
				.method(ElementMatchers.named("foo").and(ElementMatchers.takesArguments(1))).intercept
				(FixedValue.value("Three!"))
				// 该 method 会优先匹配所有方法，所以下面的打印会全部都输出 One!
				.method(ElementMatchers.isDeclaredBy(Foo.class)).intercept(FixedValue.value("One!"))
				.make();
		// unloadedClass02 加载到 JVM 中并通过反射实例化
		Foo dynamicFoo02 = unloadedClass02.load(getClass().getClassLoader()).getLoaded().newInstance();
		// 打印重写方法返回的值
		System.out.println(dynamicFoo02.bar()); // 输出 One!
		System.out.println(dynamicFoo02.foo()); // 输出 One!
		System.out.println(dynamicFoo02.foo("Hello Wold")); // 输出 One!
	}


}

```
## 代理方法的调用

```java
public class Source {

	public String hello(String name) {
		return null;
	}

}
public class Target {

	public static String intercept01(String name) {
		return "Hello " + name + "!";
	}

	public static String intercept02(int i) {
		return Integer.toString(i);
	}

	public static String intercept03(Object o) {
		return o.toString();
	}
}
public class DelegationTest {
	/**
	 * 方法委托调用
	 * 
	 * @throws Exception
	 */
	@Test
	public void testMethodDelegation() throws Exception {
		// 生成动态类
		Unloaded<Source> unloaded = new ByteBuddy()
				// 指定动态类的父类
				.subclass(Source.class)
				.method(ElementMatchers.named("hello").and(ElementMatchers.returns(String.class)))
				/**
				 *  拦截 target 类方法，delegation 会找到一个最匹配的方法，名称不要求一样，但是返回值类型和参数类型要求相匹配才行，
				 *  比如  target 类中 intercept02 方法的参数 int 类型肯定是不匹配的，因为 source 类的 hello 方法参数是 String,
				 *  target 类中剩下参数是 Object 和  String 的方法，根据最优原则，会匹配到 intercept01 方法，如果
				 *  没有匹配到就抛出异常，注意 Delegation 委托的 target 是类不是实例对象，所以只会匹配静态方法
				 */
				.intercept(MethodDelegation.to(Target.class ))
				.make();
		// 将动态类加载到 JVM 并通过反射实例化
		Source source = unloaded.load(getClass().getClassLoader()).getLoaded().newInstance();
		String result =  source.hello("World");
		// 输出  helloWorld
		System.out.println(result);
	}
}
```


























