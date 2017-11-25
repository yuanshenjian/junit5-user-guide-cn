## 4.运行测试

### 4.1. IDE支持

#### 4.1.1. IntelliJ IDEA

IntelliJ IDEA 从 2016.2 版本开始支持在JUnit Platform上运行测试。更多的细节参考 [IntelliJ IDEA的相关博客](https://blog.jetbrains.com/idea/2016/08/using-junit-5-in-intellij-idea/)。

###### *表格1. Junit5 版本对应的 IntelliJ IDEA*

| **IntelliJ IDEA 版本** | **捆绑的 JUnit 5 版本** |
|:--------------|:------------|
| 2016.2 | M2 |
| 2016.3.1 | M3|
| 2017.1.2 | M4|
| 2017.2.1 | M5|
| 2017.2.3 | RC2|
 
> ⚠️ IntelliJ IDEA 与 JUnit5 的特定版本绑定，也就是说，如果你想使用一个更新的里程碑版本的 Jupiter API，运行测试时可能会失败。直到 JUnit5 第一个GA 版本发布，这种情况才有所改善。在这之前，你可以在IntelliJ IDEA中按照下面所示的方法使用 JUnit5 的新版本。
 
要想使用JUnit5的不同版本，你需要在classpath中手动添加`junit-platform-launcher`,`junit-jupiter-engine`,`junit-vintage-engine`的JAR文件。

###### *添加Gradle依赖*

```java
// Only needed to run tests in an IntelliJ IDEA that bundles an older version
testRuntime("org.junit.platform:junit-platform-launcher:1.0.2")
testRuntime("org.junit.jupiter:junit-jupiter-engine:5.0.2")
testRuntime("org.junit.vintage:junit-vintage-engine:4.12.2")
```

###### *添加Maven依赖*

```xml
!-- Only required to run tests in an IntelliJ IDEA that bundles an older version -->
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.0.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.0.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>4.12.2</version>
    <scope>test</scope>
</dependency>
```


 
#### 4.1.2. Eclipse 测试版支持

Eclipse 4.7（*Oxygen*）的测试版支持JUnit Platform和Junit Jupiter。关于如何配置，请参考 [Eclipse JDT UI/JUnit 5](https://wiki.eclipse.org/JDT_UI/JUnit_5) wiki页面。

#### 4.1.3. 其他 IDE

在本文写作之时，并没有其他任何IDE可以像IntelliJ IDEA或Eclipse的测试版一样可以直接在JUnit Platform上运行Java测试。但是，Junit团队提供了另外两种折中的方法让JUnit 5可以在其他的IDE上使用。你可以尝试手动使用 [Console Launcher](#) 或者通过 [ 基于JUnit的Runner](#) 来执行测试。


### 4.2. 构建工具支持

#### 4.2.1. Gradle

JUnit开发团队已经开发了一款非常基础的Gradle插件，它允许你运行被`TestEngine`（例如，JUnit3、JUnit4、JUnit Jupiter以及 [Specsy](http://specsy.org/) 等）支持的任何种类的测试。关于插件的使用示例请参阅 [`junit5-gradle-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.2/junit5-gradle-consumer) 项目中的`build.gradle`文件。

##### 启用JUnit Gradle插件

要使用 JUnit Gradle插件，你首先要确保使用的是Gradle 2.5或更高的版本。若满足条件，你可以按照下面的模板来配置项目中的`build.gradle`文件。

```groovy
buildscript {
    repositories {
        mavenCentral()
        // The following is only necessary if you want to use SNAPSHOT releases.
        // maven { url 'https://oss.sonatype.org/content/repositories/snapshots' }
    }
    dependencies {
        classpath 'org.junit.platform:junit-platform-gradle-plugin:1.0.2'
    }
}

apply plugin: 'org.junit.platform.gradle.plugin'
```

##### 配置JUnit Gradle插件

一旦应用了JUnit Gradle插件，你可按照下面的方式进行配置。

```groovy
junitPlatform {
    platformVersion '1.0.2' // optional, defaults to plugin version
    logManager 'org.apache.logging.log4j.jul.LogManager'
    reportsDir file('build/test-results/junit-platform') // this is the default
    // enableStandardTestTask true
    // selectors (optional)
    // filters (optional)
}
```

设置`logManager`会让JUnit Gradle插件将`java.util.logging.manager`系统属性设置为要使用的`java.util.logging.LogManager`实现提供的全类名。上面的示例演示了如何将log4j配置为`LogManager` 。

JUnit Gradle插件在默认情况下会禁用标准的Gradle `test`任务，但可以通过`enableStandardTestTask`标记来启用。

##### 配置选择器

默认情况下，插件将会扫描项目中所有测试的输出目录。不过，你可以使用一个叫`selectors`的扩展元素来显式指定要执行哪些测试。

```groovy
junitPlatform {
    // ...
    selectors {
        uris 'file:///foo.txt', 'http://example.com/'
        uri 'foo:resource'  ①
        files 'foo.txt', 'bar.csv'
        file 'qux.json'  ②
        directories 'foo/bar', 'bar/qux'
        directory 'qux/bar'  ③
        packages 'com.acme.foo', 'com.acme.bar'
        aPackage 'com.example.app'  ④
        classes 'com.acme.Foo', 'com.acme.Bar'
        aClass 'com.example.app.Application' ⑤ 
        methods 'com.acme.Foo#a', 'com.acme.Foo#b'
        method 'com.example.app.Application#run(java.lang.String[])'  ⑥
        resources '/bar.csv', '/foo/input.json'
        resource '/com/acme/my.properties'  ⑦
    }
    // ...
}

```

① URIs  
② 本地文件  
③ 本地目录  
④ 包  
⑤ 类，全类名  
⑥ 方法，全方法名（参阅 [DiscoverySelectors中的selectMethod(String)方法](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/discovery/DiscoverySelectors.html#selectMethod-java.lang.String-)）  
⑦ 类路径资源

##### 配置过滤器

你可以使用 `Filter` 扩展来配置测试计划的过滤器。默认情况下，所有的引擎和标签都会被包含在测试计划中。但只有默认的`includeClassNamePattern 
(^.*Tests?$)`被应用。你可以重写默认的匹配模式，例如下面示例。当你使用了多种匹配模式时，JUnit Platform会使用逻辑 或 将它们合并起来使用。

```groovy
junitPlatform {
    // ...
    filters {
        engines {
            include 'junit-jupiter'
            // exclude 'junit-vintage'
        }
        tags {
            include 'fast', 'smoke'
            // exclude 'slow', 'ci'
        }
        packages {
            include 'com.sample.included1', 'com.sample.included2'
            // exclude 'com.sample.excluded1', 'com.sample.excluded2'
        }
        includeClassNamePattern '.*Spec'
        includeClassNamePatterns '.*Test', '.*Tests'
    }
    // ...
}
```

如果你通过 `engines {include …​}` 或 `engines {exclude …​}`来提供一个提供*测试引擎ID*，那么JUnit Gradle插件将会只运行你希望运行的那个测试引擎。同样，如果你通过 `tags {include …​}` 或者 `tags {exclude …​}` 提供一个*标签*，JUnit Gradle插件将只会处理相应标签的测试（例如，通过一个 `@Tag`注解标注的基于JUnit Jupiter测试）。同理，关于包名，可以通过`packages {include …​}`或者`packages {exclude …​}`配置要包含或排除的包名。

##### 配置参数
你可以使用`configurationParameter`或者`configurationParameters` DSL来设置配置参数，从而影响测试发现和执行。前者可以配置单独的配置参数，后者可以使用一个配置参数的map来一次性配置多个键-值对。所有的key和value都必须是`String`类型。


```groovy
junitPlatform {
    // ...
    configurationParameter 'junit.jupiter.conditions.deactivate', '*'
    configurationParameters([
        'junit.jupiter.extensions.autodetection.enabled': 'true',
        'junit.jupiter.testinstance.lifecycle.default': 'per_class'
    ])
    // ...
}
```

##### 配置测试引擎
为了能够使JUnit Gradle插件运行任何测试，类路径中必须存在一个`TestEngine`的实现。

要配置对基于JUnit Jupiter测试的支持，需要配置一个JUnit Jupiter API的 `testCompile`依赖以及JUnit Jupiter `TestEngine`实现的 `testRuntime`依赖，具体配置如下。

```groovy
dependencies {
    testCompile("org.junit.jupiter:junit-jupiter-api:5.0.2")
    testRuntime("org.junit.jupiter:junit-jupiter-engine:5.0.2")
}
```

只要你配置了一个JUnit4的`testCompile`依赖以及JUnit Vintage `TestEngine`实现的`testRuntime`依赖，JUnit Gradle插件就可以运行基于JUnit 4的测试，具体配置如下。

```groovy
dependencies {
    testCompile("junit:junit:4.12")
    testRuntime("org.junit.vintage:junit-vintage-engine:4.12.2")
}
```

##### 使用JUnit Gradle插件
一旦应用并配置了JUnit Gradle插件，你就可以使用新的`junitPlatformTest`任务（在可用的Gralde task中会多出一个叫`junitPlatformTest`task）。

在命令行中调用`gradlew junitPlatformTest` (or `gradlew test` )指令，项目中所有满足当前`includeClassNamePattern`（默认匹配`^.*Tests?$`）配置的测试会被执行。

在 [`junit5-gradle-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.2/junit5-gradle-consumer) 项目中执行 `junitPlatformTest`任务会看到类似下面的输出。

```sh
:junitPlatformTest

Test run finished after 93 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         3 tests found           ]
[         1 tests skipped         ]
[         2 tests started         ]
[         0 tests aborted         ]
[         2 tests successful      ]
[         0 tests failed          ]

BUILD SUCCESSFUL
```

如果测试失败，build会失败，并且会输出类似下面的信息。

```sh
:junitPlatformTest

Test failures (1):
  JUnit Jupiter:SecondTest:mySecondTest()
    MethodSource [className = 'com.example.project.SecondTest', methodName = 'mySecondTest', methodParameterTypes = '']
    => Exception: 2 is not equal to 1 ==> expected: <2> but was: <1>

Test run finished after 99 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         3 tests found           ]
[         0 tests skipped         ]
[         3 tests started         ]
[         0 tests aborted         ]
[         2 tests successful      ]
[         1 tests failed          ]

:junitPlatformTest FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':junitPlatformTest'.
> Process 'command '/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/bin/java'' finished with non-zero exit value 1
```

> 📒 当任何一个容器或测试失败时，退出值为1；否则，值为0.

> ⚠️ **当前JUnit Gradle插件的限制**
> 任何通过JUnit Gradle插件运行的测试结果都不会包含在Gradle生成的标准测试报告中；但通常可以在持续集成服务器上汇总测试结果。详情请参阅插件的`reportsDir`属性。


#### 4.2.2. Maven
为了能够通过`mvn test`运行 JUnit4 和 JUnit Jupiter，JUnit 团队为 Maven Surefire 提供了基础的支持保证。项目 [`junit5-maven-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.0-M4/junit5-maven-consumer) 中的 `pom.xml` 文件展示了如何作为一个开始并使用的其的描述。

> ⚠️ 由于 Surefire2.20 中的内存泄漏，`junit-platform-surefire-provider` 仅仅在Surefire 2.19.1 中可用。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.19</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.0.2</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

##### 配置测试引擎
为了使 Maven Surefire 能够运行所有的测试，`TestEngine`的实现必须加到运行时路径中。

要配置针对 JUnit Jupiter 测试的支持，你需要为JUnit Jupiter API配置 `test` 依赖，为 `maven-surefire-plugin` 增加JUnit Jupiter的 `TestEngine` 实现的依赖。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.19</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.0.2</version>
                </dependency>
                <dependency>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter-engine</artifactId>
                    <version>5.0.2</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
<dependencies>
    ...
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.0.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
...
```

只要你配置了 JUnit4 的 `test` 依赖，并增加 `maven-surefire-plugin` 的 JUnit Vintage `TestEngine` 实现的依赖，Unit Platform Surefire Provider 就可以运行基于JUnit4 的测试。具体配置如下：

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.19</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.0.2</version>
                </dependency>
                ...
                <dependency>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                    <version>4.12.2</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
<dependencies>
    ...
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
...
```

##### tag过滤测试
使用以下配置属性，你可以通过tag来过滤测试：

* 为了包含一个 tag，可以使用 `groups` 或者 `includeTags`
* 为了排除一个 tag，可以使用 `excludedGroups` 或者 `excludeTags`

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.19</version>
            <configuration>
                <properties>
                    <includeTags>acceptance</includeTags>
                    <excludeTags>integration, regression</excludeTags>
                </properties>
            </configuration>
            <dependencies>
                ...
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

##### 配置参数
通过配置参数可以影响测试路径和执行，使用属性 `configurationParameters` 并在 Java 的 `Properties` 文件中提供键值对。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.19</version>
            <configuration>
                <properties>
                    <configurationParameters>
                        junit.jupiter.conditions.deactivate = *
                        junit.jupiter.extensions.autodetection.enabled = true
                        junit.jupiter.testinstance.lifecycle.default = per_class
                    </configurationParameters>
                </properties>
            </configuration>
            <dependencies>
                ...
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

### 4.3. 控制台启动器
[ConsoleLauncher](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html) 是一个Java的命令行应用程序，它允许你通过命令行来启动JUnit平台。例如，它可以用来运行 JUnit Vintage 和 JUnit Jupiter 测试，并在命令行中打印测试结果。

`junit-platform-console-standalone-1.0.0-M4.jar`这个可执行的jar包，包括了所有的依赖，它已经被发布在 Maven 中心库中了，路径是 [junit-platform-console-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone/)，可以通过以下命令[运行](https://docs.oracle.com/javase/tutorial/deployment/jar/run.html) 单机版的 `ConsoleLauncher `

```sh
java -jar junit-platform-console-standalone-1.0.2.jar <Options>
```
如下所示为一个输出的例子：

```sh
├─ JUnit Vintage
│  └─ example.JUnit4Tests
│     └─ standardJUnit4Test ✔
└─ JUnit Jupiter
   ├─ StandardTests
   │  ├─ succeedingTest() ✔
   │  └─ skippedTest() ↷ for demonstration purposes
   └─ A special test case
      ├─ Custom test name containing spaces ✔
      ├─ ╯°□°）╯ ✔
      └─ 😱 ✔

Test run finished after 64 ms
[         5 containers found      ]
[         0 containers skipped    ]
[         5 containers started    ]
[         0 containers aborted    ]
[         5 containers successful ]
[         0 containers failed     ]
[         6 tests found           ]
[         1 tests skipped         ]
[         5 tests started         ]
[         0 tests aborted         ]
[         5 tests successful      ]
[         0 tests failed          ]
```

>📒 ***退出码***  
> 如果 [ConsoleLauncher](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html) 的返回的状态值为1，则代表有容器或测试运行失败，否则返回0.

#### Options

```sh
Option                                        Description
------                                        -----------
-h, --help                                    Display help information.
--disable-ansi-colors                         Disable ANSI colors in output (not
                                                supported by all terminals).
--details <[none,flat,tree,verbose]>          Select an output details mode for when
                                                tests are executed. Use one of: [none,
                                                flat, tree, verbose]. If 'none' is
                                                selected, then only the summary and test
                                                failures are shown. (default: tree)
--details-theme <[ascii,unicode]>             Select an output details tree theme for
                                                when tests are executed. Use one of:
                                                [ascii, unicode] (default: unicode)
--class-path, --classpath, --cp <Path:        Provide additional classpath entries --
  path1:path2:...>                              for example, for adding engines and
                                                their dependencies. This option can be
                                                repeated.
--reports-dir <Path>                          Enable report output into a specified
                                                local directory (will be created if it
                                                does not exist).
--scan-class-path, --scan-classpath [Path:    Scan all directories on the classpath or
  path1:path2:...]                              explicit classpath roots. Without
                                                arguments, only directories on the
                                                system classpath as well as additional
                                                classpath entries supplied via -cp
                                                (directories and JAR files) are scanned.
                                                Explicit classpath roots that are not on
                                                the classpath will be silently ignored.
                                                This option can be repeated.
-u, --select-uri <URI>                        Select a URI for test discovery. This
                                                option can be repeated.
-f, --select-file <String>                    Select a file for test discovery. This
                                                option can be repeated.
-d, --select-directory <String>               Select a directory for test discovery.
                                                This option can be repeated.
-p, --select-package <String>                 Select a package for test discovery. This
                                                option can be repeated.
-c, --select-class <String>                   Select a class for test discovery. This
                                                option can be repeated.
-m, --select-method <String>                  Select a method for test discovery. This
                                                option can be repeated.
-r, --select-resource <String>                Select a classpath resource for test
                                                discovery. This option can be repeated.
-n, --include-classname <String>              Provide a regular expression to include
                                                only classes whose fully qualified names
                                                match. To avoid loading classes
                                                unnecessarily, the default pattern only
                                                includes class names that end with
                                                "Test" or "Tests". When this option is
                                                repeated, all patterns will be combined
                                                using OR semantics. (default: ^.*Tests?$)
-N, --exclude-classname <String>              Provide a regular expression to exclude
                                                those classes whose fully qualified
                                                names match. When this option is
                                                repeated, all patterns will be combined
                                                using OR semantics.
--include-package <String>                    Provide a package to be included in the
                                                test run. This option can be repeated.
--exclude-package <String>                    Provide a package to be excluded from the
                                                test run. This option can be repeated.
-t, --include-tag <String>                    Provide a tag to be included in the test
                                                run. This option can be repeated.
-T, --exclude-tag <String>                    Provide a tag to be excluded from the test
                                                run. This option can be repeated.
-e, --include-engine <String>                 Provide the ID of an engine to be included
                                                in the test run. This option can be
                                                repeated.
-E, --exclude-engine <String>                 Provide the ID of an engine to be excluded
                                                from the test run. This option can be
                                                repeated.
--config <key=value>                          Set a configuration parameter for test
                                                discovery and execution. This option can
                                                be repeated.
```

### 4.4. 使用JUnit4运行JUnit Platform
`JunitPlatform` 运行器是一个基于 JUnit4 的`Runner`，它可以运行任何在 JUnit Platform 上以JUnint4 环境所支持的编程模型的测试，例如，JUnit Jupiter 测试类。

如果某个类上标注了 `@RunWith(JUnitPlatform.class)` 注解，它就可以在那些支持 JUnit4 但是还不支持JUnit Platform 的 IDE 和构建系统中上直接运行。

>📒 由于 JUnit Platform 具备一些 JUnit4 不具备的功能，运行器只能部分支持 JUnit Platform 的功能，尤其针对报告中的一些内容（见 [命名显示 vs 科学命名](http://junit.org/junit5/docs/current/user-guide/#running-tests-junit-platform-runner-technical-names)）。但是就目前来说，`JUnitPlatform` 运行器是一个开启学习的简单方式。

#### 4.4.1. 设置
你需要在类路径中添加以下的组件和它们的依赖。可以在 [依赖元数据](http://junit.org/junit5/docs/current/user-guide/#dependency-metadata) 中查看关于group ID, artifact ID 和版本的细节信息。

##### 显式依赖
* `junit-4.12.jar` 在*test*范围内：使用 JUnit4 运行测试
* `junit-platform-runner` 在*test*范围内：`JUnitPlatform`运行器的位置
* `junit-jupiter-api` 在*test*范围内：使用API编写测试，包括 `@Test` 等
* `junit-jupiter-engine` 在*test*运行范围内：为JUnit Jupiter 实现 Engine 的 API 方法

##### 传递的依赖
* `junit-platform-launcher` 在*test*范围内
*  `junit-platform-engine` 在*test*范围内
*  `junit-platform-commons` 在*test*范围内
*  `opentest4j` 在*test*范围内

#### 4.4.2. 展示名称 vs 技术名称
默认情况下，*展示名称*会被使用在测试产出物上，但是当`JUnitPlatform` 运行器使用 Gradle 或者 Maven 等编译工具来运行测试时，生成的测试报告需要使用测试产出物的*技术名称*，例如，使用完整类名，而不是使用简写类名，或者自定义的包含特殊字符的展示名称。为了在测试报告中使用技术名称，在 `@RunWith(JUnitPlatform.class)` 注解旁边声明 `@UseTechnicalNames` 注解即可。

#### 4.4.3. Single测试类
使用 `JUnitPlatform` 运行器的方式之一是直接在测试类上添加 `@RunWith(JUnitPlatform.class)` 注解。注意下面例子中的测试方法使用的注解是 `org.junit.jupiter.api.Test`（JUnit Jupiter）,而不是 `org.junit.Test`(JUnit Vintage)。同时，这个类中的测试用例必须为 `public`，否则，IDE不能将其识别为JUnit4 的测试类。

```java
import static org.junit.jupiter.api.Assertions.fail;

import org.junit.jupiter.api.Test;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit4ClassDemo {

    @Test
    void succeedingTest() {
        /* no-op */
    }

    @Test
    void failingTest() {
        fail("Failing for failing's sake.");
    }

}
```

#### 4.4.4. 测试套件
如果有多个测试类，可以通过创建测试套件完成测试，如下例子所示：

```java
import org.junit.platform.runner.JUnitPlatform;
import org.junit.platform.suite.api.SelectPackages;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
@SelectPackages("example")
public class JUnit4SuiteDemo {
}
```

`JUnit4SuiteDemo` 类会寻找并运行所有在 `example` 包及其子包下的测试。默认情况下，它只包含类名符合正则表达式 `^.*Tests?$` 的测试类。

>📒 ***附加配置选项***  
> 相比于值使用 `@SelectPackages` 注解，还有很多配置选项可以用来寻找和过滤测试。详细内容参考 [Javadoc](http://junit.org/junit5/docs/current/api/org/junit/platform/suite/api/package-summary.html).

### 4.5. 配置参数
为了给操作平台指明包含哪些测试类、测试引擎和需要扫描哪些包，需要额外添加自定义的配置参数来详细说明测试引擎。例如，JUnit Jupiter `TestEngine` 支持以下用例的配置参数。

* [Changing the Default Test Instance Lifecycle](http://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle-changing-default)
* [Enabling Automatic Extension Detection](http://junit.org/junit5/docs/current/user-guide/#extensions-registration-automatic-enabling)
* [Deactivating Conditions](http://junit.org/junit5/docs/current/user-guide/#extensions-conditions-deactivation)

配置参数是一种基于文本的键值对，可以通过以下任一种机制应用于 JUnit Platform 的测试引擎：

1. `LauncherDiscoveryRequestBuilder ` 中的方法 `configurationParameter()` 和 `configurationParameters()` 可以用来为 [Launcher](http://junit.org/junit5/docs/current/user-guide/#launcher-api) [API](http://junit.org/junit5/docs/current/user-guide/#launcher-api) 构建请求。当使用 JUnit Platform 提供的某一种测试工具，可以配置详细的配置参数，如下所示：
 * [Console Launcher](http://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher): 使用 `--config` 命令行选项
 * [Gradle plugin](http://junit.org/junit5/docs/current/user-guide/#running-tests-build-gradle-config-params): 使用 `configurationParameter` 或者 `configurationParameters` DSL
 * [Maven Surefire provider](http://junit.org/junit5/docs/current/user-guide/#running-tests-build-maven-config-params): 使用 `configurationParameters ` 属性
2. JVM 系统属性
3. JUnit Platform 配置文件：该文件命名为 `junit-platform.properties`，在根目录路径下，并遵循 Java `Properties` 文件的语法

> 📒 配置参数按照上面的定义顺序进行查找，所以，在 'Launcher' 中的配置参数优先级高于在系统属性中或配置文件中的设置。同样的，通过系统属性应用的应用变量优先级高于配置文件。






