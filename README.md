# junit5-user-guide-cn
该工程中包含了JUnit5 User Guide文档的中文翻译。

原文链接：<http://junit.org/junit5/docs/current/user-guide/>

## 翻译说明

- 统一使用Markdown格式编辑，推荐使用[MacDown](http://macdown.uranusjr.com/)工具编辑。
- 可以只出现译文，也可以中英段落间隔开，一段英文，紧接着是一段中文翻译。建议翻译中存在意思拿不准的就保留英文方便快速校对，例如：

The **JUnit Platform** serves as a foundation for [launching testing frameworks](#) on the JVM. It also defines the [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API for developing a testing framework that runs on the platform. Furthermore, the platform provides a [Console Launcher](#) to launch the platform from the command line and build plugins for[Gradle](#) and [Maven](#) as well as a [JUnit 4 based Runner](#) for running any `TestEngine` on the platform.  

**JUnit Platform**是一个用来 [启动测试框架](#) 的基础平台，它是基于JVM的。它为开发一个运行在平台上的测试框架定义了 [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) API。除此之外，平台还提供了一个 [控制台启动器](#) ，用来从命令行或者[Gradle](#) 和 [Maven](#)插件来启动平台，就好比一个 [基于JUnit的Runner](#)在平台运行任何`TestEngine`。


- 段落中存在超链接，如果链接到外部，翻译段落中也同样链接到外部。如果是文档内部链接，预留出超链接格式，但不填写内容。例如：

```
- [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html)
- [基于JUnit的Runner]()
```

- 原文中存在的方法名和类名等专有名词，不翻译，直接保留，例如：

```
`TestEngine`
```

- 常用工具关键字不用翻译，例如`Gradle`,`Maven`
- 代码块不翻译
- 提交Git commit格式如下：

```
$ git commit -m "[chapter01] - Translate 1.1"
$ git commit -m "[chapter03] - Translate 3.11"
$ git commit -m "[chapter03] - Complete chapter03"

$ git commit -m "[chapter03] - Review chapter03"
```

## 翻译计划
- 2017.10.01 之前完成初稿翻译。
- 2017.12.05 之前完成交叉Review。
- 2018.01.01 发布v1.0。





