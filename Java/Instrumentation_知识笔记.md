---
title: Instrumentation 知识笔记
---

## Instrumentation 介绍
Instrumentation 是从 JDK5 开始提供的一个特性，可以使用 Instrumentation 构建一个独立于应用程序的代理程序(Agent)。使用 Instrumentation 可以让开发者无需对原有应用做任何修改，就可以监测和获取 JVM 运行时状态，以及可以通过 ASM 和 Javassist 字节码类型增强修改 JVM 中运行的类。

## 核心概念

### 两种加载方式
1. **premain**：在 JVM 启动时通过 `-javaagent` 参数加载代理，在 `main` 方法执行前运行
2. **agentmain**：在 JVM 运行时通过 Attach API 动态加载代理，可对已运行的程序进行增强

### 关键组件
- **Instrumentation**：核心接口，提供字节码转换、类重定义等功能
- **ClassFileTransformer**：字节码转换器接口，实现 `transform` 方法进行字节码修改
- **ClassDefinition**：用于类重定义的数据结构

## Instrumentation 使用

### 1. 创建代理(Agent)

创建代理类，需要在代理类中定义 `premain` 或 `agentmain` 方法，`Instrumentation` 实例以参数的形式传递到方法中。

```java
public class MyAgent {

    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("[Agent] premain 方法执行");
        inst.addTransformer(new MyClassTransformer());
    }

    public static void agentmain(String agentArgs, Instrumentation inst) {
        System.out.println("[Agent] agentmain 方法执行");
        inst.addTransformer(new MyClassTransformer(), true);
        try {
            Class<?> targetClass = Class.forName("com.example.TargetClass");
            inst.retransformClasses(targetClass);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 2. 实现 ClassFileTransformer

```java
public class MyClassTransformer implements ClassFileTransformer {
    
    @Override
    public byte[] transform(ClassLoader loader, String className,
            Class<?> classBeingRedefined, ProtectionDomain protectionDomain,
            byte[] classfileBuffer) {
        
        if (!"com/example/TargetClass".equals(className)) {
            return null;
        }
        
        try {
            ClassReader cr = new ClassReader(classfileBuffer);
            ClassWriter cw = new ClassWriter(cr, ClassWriter.COMPUTE_MAXS);
            ClassVisitor cv = new MyClassVisitor(cw);
            cr.accept(cv, 0);
            return cw.toByteArray();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```

### 3. 配置 MANIFEST.MF

在 JAR 包的 `META-INF/MANIFEST.MF` 中添加以下配置：

```
Premain-Class: com.example.MyAgent
Agent-Class: com.example.MyAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true
```

### 4. 使用代理

**premain 方式**：启动时加载
```bash
java -javaagent:myagent.jar -jar myapp.jar
```

**agentmain 方式**：运行时动态加载
```java
// 获取目标 JVM
VirtualMachine vm = VirtualMachine.attach(pid);
// 加载代理
vm.loadAgent("path/to/myagent.jar");
vm.detach();
```

## Instrumentation 常用 API

| 方法 | 说明 |
|------|------|
| `addTransformer()` | 注册字节码转换器 |
| `removeTransformer()` | 移除字节码转换器 |
| `retransformClasses()` | 重新转换已加载的类 |
| `redefineClasses()` | 重定义已加载的类 |
| `getAllLoadedClasses()` | 获取所有已加载的类 |
| `getObjectSize()` | 获取对象占用内存大小 |

## 应用场景

1. **APM 监控**：SkyWalking、Pinpoint 等工具通过 Instrumentation 实现方法耗时监控
2. **热部署**：JRebel 等工具实现代码热替换
3. **代码覆盖率**：JaCoCo 通过字节码插桩统计代码执行情况
4. **安全检测**：RASP 运行时应用安全防护
5. **日志增强**：自动为方法添加日志记录

## 参考文档
- [Java Instrumentation 官方文档](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/Instrumentation.html)
- [ASM 字节码框架](https://asm.ow2.io/)
- [Javassist 字节码操作库](https://www.javassist.org/)
