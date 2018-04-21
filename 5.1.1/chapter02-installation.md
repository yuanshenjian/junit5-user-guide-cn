## 2. 安装
最终版本和里程碑版本已经被部署到Maven仓库中心。

快照版本被部署到 [Sonatype 快照库](https://oss.sonatype.org/content/repositories/snapshots) 中的 [/org/junit](https://oss.sonatype.org/content/repositories/snapshots/org/junit/)目录下。

### 2.1. 依赖元数据

#### 2.1.1. JUnit Platform

* **Group ID**: `org.junit.platform`

* **Version**: `{{ platform_version }}`

* **Artifact IDs:**

`junit-platform-commons`  

JUnit 内部通用类库/实用工具，它们仅用于JUnit框架本身，*不支持任何外部使用*，外部使用风险自负。

`junit-platform-console`  

支持从控制台中发现和执行JUnit Platform上的测试。详情请参阅 [控制台启动器](#43-控制台启动器)。
	

`junit-platform-console-standalone`  

一个包含了Maven仓库中的 [junit-platform-console-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone) 目录下所有依赖项的可执行JAR包。详情请参阅 [控制台启动器](#43-控制台启动器)。


`junit-platform-engine`  	

测试引擎的公共API。详情请参阅 [插入你自己的测试引擎](#713-插入你自己的测试引擎)。


`junit-platform-gradle-plugin`  	

支持使用 [Gralde](#421-gradle) 来发现和执行JUnit Platform上的测试。


`junit-platform-launcher`	

配置和加载测试计划的公共API -- 典型的使用场景是IDE和构建工具。详情请参阅 [JUnit Platform启动器API](#71-junit-platform启动器api)。


`junit-platform-runner`

在一个JUnit 4环境中的JUnit Platform上执行测试和测试套件的运行器。详情请参阅 [使用JUnit 4运行JUnit Platform](#44-使用junit-4运行junit-platform)。
   
   
`junit-platform-suite-api`
	
在JUnit Platform上配置测试套件的注解。被 [JUnit Platform运行器](#44-使用junit-4运行junit-platform) 所支持，也有可能被第三方的`TestEngine`实现所支持。 


`junit-platform-surefire-provider`

支持使用 [Maven Surefire](#422-maven) 来发现和执行JUnit Platform上的测试。


#### 2.1.2. JUnit Jupiter
* **Group ID**: `org.junit.jupiter`

* **Version**: `{{ jupiter_version }}`

* **Artifact IDs**:

`junit-jupiter-api`

[编写测试](#3-编写测试) 和 [扩展](#5-扩展模型) 的JUnit Jupiter API。


`junit-jupiter-engine`

JUnit Jupiter测试引擎的实现，仅仅在运行时需要。


`junit-jupiter-params`

支持JUnit Jupiter中的 [参数化测试](#313-参数化测试)。


`junit-jupiter-migration-support`

支持从JUnit 4迁移到JUnit Jupiter，仅在使用了JUnit 4规则的测试中才需要。


#### 2.1.3. JUnit Vintage

* **Group ID**: `org.junit.vintage`

* **Version**: `{{ vintage_version }}`

* **Artifact ID**:

`junit-vintage-engine`

JUnit Vintage测试引擎实现，允许在新的JUnit Platform上运行低版本的JUnit测试，即那些以JUnit 3或JUnit 4风格编写的测试。


#### 2.1.4. 依赖
以上所有artifacts在它们已发布的Maven POM中都依赖了下面的*@API Guardian* JAR文件。

* **Group ID**: `org.apiguardian`

* **Artifact ID**: `apiguardian-api`

* **Version**: `{{ apiguardian_version }}`

此外，上面大部分artifacts都对下面的*OpenTest4J* JAR文件有直接或传递的依赖关系。

* **Group ID**: `org.opentest4j`

* **Artifact ID**: `opentest4j`

* **Version**: `{{ ota4j_version }}`


### 2.2. 依赖关系图

![](https://junit.org/junit5/docs/5.1.0/user-guide/images/component-diagram.svg)


### 2.3 JUnit Jupiter示例工程
[junit5-samples](https://github.com/junit-team/junit5-samples) 代码库中包含了一系列基于JUnit Jupiter和JUnit Vintage的示例工程。你可以在下面的项目中找到相应的`build.gradle`和`pom.xml`文件：

- Gradle工程：[junit5-gradle-consumer](https://github.com/junit-team/junit5-samples/tree/r5.0.3/junit5-gradle-consumer).

- Maven工程：[junit5-maven-consumer](https://github.com/junit-team/junit5-samples/tree/r5.0.3/junit5-maven-consumer).
