# 2. 安装
最终版本和里程碑的包已经部署到Mave仓库中心了。

快照版本被部署到 [Sonatype快照库](https://oss.sonatype.org/content/repositories/snapshots) 中的 [/org/junit](https://oss.sonatype.org/content/repositories/snapshots/org/junit/)目录下。

## 2.1. 依赖元数据
### 2.1.1. JUnit Platform

* **Group ID**: `org.junit.platform`

* **Version**: `1.0.0-M4`

* **Artifact IDs:**

`junit-platform-commons`  

jUnit 内部通用类库/工具。这些工具是专门给JUnit框架自身使用的。不支持任何外部使用。使用的需要自行承担风险。

`junit-platform-console`  

支持从控制台中查找和执行JUnit平台上的测试。详情参考[控制台启动器]()
	

`junit-platform-console-standalone`  

一个包含了Maven仓库中心 [junit-platform-console-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone) 目录下所提供的所有依赖的可执行JAR包。详情参考[控制台启动器]()

`junit-platform-engine`  	

测试引擎的公共API。详情参考 [插入你自己的测试引擎]()

`junit-platform-gradle-plugin`  	

支持使用 [Gralde]() 来查找和执行JUnit平台上的测试。

`junit-platform-launcher`	

配置和加载测试计划的公共API -- 典型的使用场景是在IDE和构建工具中。详情参考 [JUnit 平台启动器API]()

`junit-platform-runner`

它是一个在JUnit平台上以JUnit 4的环境执行的测试和测试套件的运行器。详情参考 [使用JUnit 4运行JUnit平台]()
   
   
`junit-platform-suite-api`
	
在JUnit平台上配置测试套件的注解。被 [JUnit平台运行器]()所支持，也有可能被第三方`TestEngine`的实现所支持。 

`junit-platform-surefire-provider`

	
支持使用 [Maven Surefire]() 来查找和执行JUnit 平台上的测试。


### 2.1.2. JUnit Jupiter
* **Group ID**: `org.junit.jupiter`

* **Version**: `5.0.0-M4`

* **Artifact IDs**:

`junit-jupiter-api`

[编写测试]() 和 [扩展]() 的JUnit Jupiter API。

`junit-jupiter-engine`

JUnit Jupiter测试引擎的实现，仅仅用在运行时。

`junit-jupiter-params`

支持JUnit Jupiter中 [参数化的测试]()。

`junit-jupiter-migration-support`

将JUnit 4的支持迁移到JUnit Jupiter，仅仅用来运行选择了JUnit 4规则的测试。


### 2.1.3. JUnit Vintage

* **Group ID**: `org.junit.vintage`

* **Version**: `4.12.0-M4`

* **Artifact ID**:

`junit-vintage-engine`

JUnit Vintage测试引擎实现，允许在行的JUnit平台上运行古老的JUnit测试，即那些使用JUnit 3或JUnit 4风格编写的测试。

## 2.2. 依赖关系图

![](http://junit.org/junit5/docs/current/user-guide/images/component-diagram.svg)

## 2.3. JUnit Jupiter Sample Projects

## 2.3 JUnit Jupiter示例工程
[junit5-samples](https://github.com/junit-team/junit5-samples) 代码库中包含了一系列基于JUnit Jupiter和JUnit Vintage的示例工程。在下面的项目中，你会分别找到`build.gradle`和`pom.xml`文件：

- Gradle工程：[junit5-gradle-consumer](https://github.com/junit-team/junit5-samples/tree/r5.0.0-M4/junit5-gradle-consumer).

- Maven工程：[junit5-maven-consumer](https://github.com/junit-team/junit5-samples/tree/r5.0.0-M4/junit5-maven-consumer).