---
title: Java Instrumentation 知识笔记
---

## Java Instrumentation 介绍
Instrumentation 是从 JDK5 开始提供的一个特性，可以使用 Instrumentation 构建一个独立于应用程序的代理程序(Agent)。使用 Instrumentation 可以让开发者无需对原有应用做任何修改，就可以监测和获取 JVM 运行时状态，以及可以通过 ASM 和 Javassist 字节码类型增强修改 JVM 中运行的类。

## Java Instrumentation 使用
###  