# JUnit 5 User Guide
`Stefan Bechtold•Sam Brannen•Johannes Link•Matthias Merdes•Marc Philipp•Christian Stein•Version 5.0.0-M4`


## 1. Overview
## 1. 概述

The goal of this document is to provide comprehensive reference documentation for programmers writing tests, extension authors, and engine authors as well as build tool and IDE vendors.

本文档的目标是为编写测试、扩展作者和引擎作者以及构建工具和IDE供应商的程序员提供全面的参考文档。

---

### 1.1. What is JUnit 5?
### 1.1. JUnit 5 是什么?
Unlike previous versions of JUnit, JUnit 5 is composed of several different modules from three different sub-projects.  
跟jUnit之前的版本不一样的是，jUnit 5由几个不同的模块组成，它们分别来自于三个不同的子项目。


***JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage***  

***JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage***


The **JUnit Platform** serves as a foundation for [launching testing frameworks](#) on the JVM. It also defines the [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API for developing a testing framework that runs on the platform. Furthermore, the platform provides a [Console Launcher](#) to launch the platform from the command line and build plugins for[Gradle](#) and [Maven](#) as well as a [JUnit 4 based Runner](#) for running any `TestEngine` on the platform.  

**JUnit Platform**是一个用来 [启动测试框架](#) 的基础平台，它是基于JVM的。它为开发一个运行在平台上的测试框架定义了 [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API。除此之外，平台还提供了一个 [控制台启动器](#) ，用来从命令行或者[Gradle](#) 和 [Maven](#)插件来启动平台，就好比一个 [基于JUnit的Runner](#)在平台运行任何`TestEngine`。


**JUnit Jupiter** is the combination of the new [programming model](#) and [extension model](#) for writing tests and extensions in JUnit 5. The Jupiter sub-project provides a `TestEngine` for running Jupiter based tests on the platform.  

**JUnit Jupiter** 组合了新的 [编程模型](#) 和 [扩展模型](#)，我们可以在JUnit 5中使用它来编写测试及扩展。
另外，Jupiter子项目还提供了一个`TestEngine`，用于在平台上运行基于Jupiter编写的测试。


**JUnit Vintage** provides a `TestEngine` for running JUnit 3 and JUnit 4 based tests on the platform.  

**JUnit Vintage** 提供了一个`TestEngine`，用于在平台上运行基于JUnit 3和JUnit 4 编写的测试。

---

### 1.2. Supported Java Versions
### 1.2. 所支持的Java版本

JUnit 5 requires Java 8 at runtime. However, you can still test code that has been compiled with previous versions of the JDK.

JUnit 5需要Java 8的运行时环境。然而，你仍然可以测试那些由老版本JDK所编译的代码。

