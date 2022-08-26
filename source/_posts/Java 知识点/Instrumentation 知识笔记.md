---
title: Instrumentation 知识笔记
---

## Instrumentation 介绍
Instrumentation 是从 JDK5 开始提供的一个特性，可以使用 Instrumentation 构建一个独立于应用程序的代理程序(Agent)。使用 Instrumentation 可以让开发者无需对原有应用做任何修改，就可以监测和获取 JVM 运行时状态，以及可以通过 ASM 和 Javassist 字节码类型增强修改 JVM 中运行的类。

## Instrumentation 使用
### 创建 Agent 代理类
想要利用 JVM 提供的 Instrumentation API 来改变 JVM 中已经加载的字节代码，要创建一个

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