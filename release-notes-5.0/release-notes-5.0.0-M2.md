### 5.0.0-M2
**发布时间**： 2016.07.23

**范围**： JUnit 5的第2个里程碑发布

#### 变更摘要
此版本主要是一个bug修复版本，修复了自`5.0.0-M1`以来所发现的bug。

以下是一个全局改变的列表。关于更改的详细信息，包括Platform、Jupiter和Vintage，请参阅下面的章节。关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0 M2](https://github.com/junit-team/junit5/milestone/4?closed=1) 里程碑页面。

* JUnit 5的Gradle构建可以正常在Microsoft Windows下运行。
* 在AppVeyor上建立了针对Microsoft Windows的持续集成构建。

#### JUnit Platform

###### 修复Bug
* 容器中的故障--例如，抛出异常的`@BeforeAll`方法--如今在使用`ConsoleLauncher`或JUnit Platform Gradle插件时会构建失败。
* JUnit Platform Surefire Provider不再默默地忽略纯动态测试类——例如只声明`@TestFactory`方法的测试类。
* 在`junit-platform-console-<release version>` TAR和ZIP发布包中包含的`junit-platform-console`和`junit-platform-console.bat` shell脚本能够正确引用`ConsoleLauncher`，而不是`ConsoleRunner`。
* 被`ConsoleLauncher`和JUnit Plaform Gradle插件使用的`TestExecutionSummary`包含了失败的错误信息。
* 类路径扫描现在可以防止类加载和处理期间遇到的问题--例如，在处理格式错误的类时，潜在的异常被吞噬并记录在有问题的文件路径中，如果这个异常是一个列入黑名单的异常，比如`OutOfMemoryError`，那么它将被重新抛出。

###### 弃用
* `DiscoverySelectors`中基于通用名称的发现选择器（即`selectName()`和`selectNames()`）已经被废弃，建议使用`selectPackage(String)`、`selectClass(String)`和`selectMethod(String)`方法。

###### 新特性
* `DiscoverySelectors`中的新方法`selectMethod(String)`支持选择*完全限定方法名*。

#### JUnit Jupiter

###### 修复Bug
* 在类级别和方法级别通过`@ExtendWith`声明的扩展实现将不再被多次注册。

#### JUnit Vintage
*自5.0.0-M1以来没有变化。*

