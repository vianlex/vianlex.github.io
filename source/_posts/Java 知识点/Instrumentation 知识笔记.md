---
title: Instrumentation 知识笔记
---

## Instrumentation 介绍
Instrumentation 是从 JDK5 开始提供的一个特性，可以使用 Instrumentation 构建一个独立于应用程序的代理程序(Agent)。使用 Instrumentation 可以让开发者无需对原有应用做任何修改，就可以监测和获取 JVM 运行时状态，以及可以通过 ASM 和 Javassist 字节码类型增强修改 JVM 中运行的类。

## Instrumentation 使用
利用 JVM 提供的 Instrumentation API 来改变 JVM 中已经加载的字节代码
### 创建代理(Agent)
创建代理类，需要在代理类中定义两个名为 premain 和 agentmain 的方法，Instrumentation 实例以参数的形式传递到方法中。premain 和 agentmain 方法的区别如下：
1、premain – 在代理类中定义 premain 方法
2、agentmain – will dynamically load the agent into the JVM using the Java Attach API

```java
public class TestAgent {

    public static void premain(String agentArgs, Instrumentation inst) {
        LOGGER.info("[Agent] In premain method");
        String className = "com.baeldung.instrumentation.application.MyAtm";
        transformClass(className,inst);
    }

    public static void agentmain(String agentArgs, Instrumentation inst) {
        LOGGER.info("[Agent] In agentmain method");
        String className = "com.baeldung.instrumentation.application.MyAtm";
        transformClass(className,inst);
    }
}

```

## 参考文档
