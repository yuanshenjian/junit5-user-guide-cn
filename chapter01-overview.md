## 1. 概述
本文档的目标是为那些编写测试的程序员、扩展开发人员（extension authors）和引擎开发人员（engine authors）以及构建工具和IDE供应商提供综合全面的参考。

本文档PDF格式 [下载](https://github.com/sjyuan-cc/sjyuan-cc.github.io/raw/master/assets/documents/junit5-user-guide-cn.pdf)。

### 1.1. JUnit 5 是什么?
JUnit 5跟以前的JUnit版本不一样，它由几大不同的模块组成，这些模块分别来自三个不同的子项目。

***JUnit 5*** = ***JUnit Platform*** + ***JUnit Jupiter*** + ***JUnit Vintage***

**JUnit Platform**是在JVM上 [启动测试框架](#71-junit-platform启动器api) 的基础平台。它还定义了 [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API，该API可用于开发在平台上运行的测试框架。此外，平台还提供了一个从命令行或者 [Gradle](#421-gradle) 和 [Maven](#422-maven) 插件来启动的 [控制台启动器](#43-控制台启动器) ，它就好比一个 [基于JUnit 4的Runner](#44-使用junit-4运行junit-platform) 在平台上运行任何`TestEngine`。

**JUnit Jupiter** 是一个组合体，它是由在JUnit 5中编写测试和扩展的新 [编程模型](#3-编写测试) 和 [扩展模型](#5-扩展模型) 组成。另外，Jupiter子项目还提供了一个`TestEngine`，用于在平台上运行基于Jupiter的测试。

**JUnit Vintage** 提供了一个`TestEngine`，用于在平台上运行基于JUnit 3和JUnit 4的测试。

### 1.2. 支持的Java版本
JUnit 5需要Java 8（或更高）的运行时环境。不过，你仍然可以测试那些由老版本JDK编译的代码。

### 1.3. 获取帮助
与JUnit 5相关问题，可以在 [Stack Overflow](https://stackoverflow.com/questions/tagged/junit5)
进行提问，或者在 [Gitter](https://gitter.im/junit-team/junit5) 上跟我们交流。
