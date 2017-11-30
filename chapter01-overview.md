# JUnit 5 User Guide
`Stefan Bechtold•Sam Brannen•Johannes Link•Matthias Merdes•Marc Philipp•Christian Stein - Version 5.0.2`


## 1. 概述
本文档的目标是为那些编写测试、扩展作者和引擎作者以及构建工具和IDE供应商的程序员提供综合全面的参考。

本文档的PDF格式[下载链接](http://junit.org/junit5/docs/current/user-guide/index.pdf)。

### 1.1. JUnit 5 是什么?
与以前版本的JUnit不同，JUnit5由几个不同的模块组成，它们分别来自于三个不同的子项目。

***JUnit 5*** = ***JUnit Platform*** + ***JUnit Jupiter*** + ***JUnit Vintage***

**JUnit Platform**是在JVM上 [启动测试框架](#71-junit-platform启动器api) 的基础平台。它还定义了[TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API，该API可用于开发运行在平台上的测试框架。此外，平台还提供了一个从命令行或者 [Gradle](#421-gradle) 和 [Maven](#422-maven) 插件来启动平台的 [控制台启动器](#43-控制台启动器) ，它就好比一个 [基于JUnit 4的Runner](#44-使用junit-4运行junit-platform) 在平台上运行任何`TestEngine`。

**JUnit Jupiter** 是一个组合体，由在JUnit5中编写测试和扩展的新 [编程模型](#3-编写测试) 和 [扩展模型](#5-扩展模型) 组成。另外，Jupiter子项目还提供了一个`TestEngine`，用于在平台上运行基于Jupiter的测试。

**JUnit Vintage** 提供了一个`TestEngine`，用于在平台上运行基于JUnit 3和JUnit 4 的测试。

### 1.2. 支持的Java版本
JUnit 5需要Java 8（或更高）的运行时环境。不过，你仍然可以测试那些经由老版本JDK编译的代码。

### 1.3. 获取帮助
与JUnit 5相关问题，可以在 [Stack Overflow](https://stackoverflow.com/questions/tagged/junit5)
进行提问，或者在 [Gitter](https://gitter.im/junit-team/junit5) 上跟我们进行交流。

