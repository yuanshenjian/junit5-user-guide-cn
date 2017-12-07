## 4. 运行测试

### 4.1. IDE支持

#### 4.1.1. IntelliJ IDEA

IntelliJ IDEA 从 2016.2 版本开始支持在JUnit Platform上运行测试。详情请参阅 [IntelliJ IDEA的相关博客](https://blog.jetbrains.com/idea/2016/08/using-junit-5-in-intellij-idea/)。

###### *表格1. Junit5 版本对应的 IntelliJ IDEA*

| **IntelliJ IDEA 版本** | **捆绑的 JUnit 5 版本** |
|:--------------|:------------|
| 2016.2 | M2 |
| 2016.3.1 | M3|
| 2017.1.2 | M4|
| 2017.2.1 | M5|
| 2017.2.3 | RC2|
 
> ⚠️ IntelliJ IDEA 与 JUnit5 的特定版本绑定，也就是说，如果你使用了Jupiter API更新的里程碑版本，执行测试时可能不起作用。这种情况一致持续到JUnit 5第一个GA版本发布才得到改善。在这之前，你可以在IntelliJ IDEA中按照下面所示的方法使用JUnit 5的新版本。
 
要想使用JUnit 5的不同版本，你需要在classpath中手动添加`junit-platform-launcher`、`junit-jupiter-engine`和`junit-vintage-engine`的JAR文件。

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
Eclipse 4.7（*Oxygen*）的测试版支持JUnit Platform和Junit Jupiter。关于如何配置，请参阅 [Eclipse JDT UI/JUnit 5](https://wiki.eclipse.org/JDT_UI/JUnit_5) wiki页面。

#### 4.1.3. 其他 IDE
在本文写作之时，并没有其他任何IDE可以像IntelliJ IDEA或Eclipse的测试版一样可以直接在JUnit Platform上运行Java测试。但是，Junit团队提供了另外两种折中的方法让JUnit 5可以在其他的IDE上使用。你可以尝试手动使用 [控制台启动器](#43-控制台启动器) 或者通过 [基于JUnit 4的Runner](#44-使用junit-4运行junit-platform) 来执行测试。


### 4.2. 构建工具支持

#### 4.2.1. Gradle

JUnit开发团队已经开发了一款非常基础的Gradle插件，它允许你运行被`TestEngine`（例如，JUnit3、JUnit4、JUnit Jupiter以及 [Specsy](http://specsy.org/) 等）支持的任何种类的测试。关于插件的使用示例请参阅 [`junit5-gradle-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.2/junit5-gradle-consumer) 项目中的`build.gradle`文件。

##### 启用JUnit Gradle插件
 要使用JUnit Gradle插件，你首先要确保使用了Gradle 2.5或更高的版本，然后你可以按照下面的模板来配置项目中的`build.gradle`文件。

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

一旦应用了JUnit Gradle插件，你就可以按照下面的方式进行配置。

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

设置`logManager`会让JUnit Gradle插件将`java.util.logging.manager`系统属性设置为当前所提供的`java.util.logging.LogManager`实现类的*全类名*。上面的示例演示了如何将log4j配置为`LogManager` 。

JUnit Gradle插件在默认情况下会禁用标准的Gradle `test`任务，但可以通过`enableStandardTestTask`标志来启用。

##### 配置选择器
默认情况下，插件会扫描项目中所有测试的输出目录。不过，你可以使用一个叫`selectors`的扩展元素来显式指定要执行哪些测试。

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
⑥ 方法，全方法名（请参阅 [DiscoverySelectors中的selectMethod(String)方法](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/discovery/DiscoverySelectors.html#selectMethod-java.lang.String-)）  
⑦ 类路径资源

##### 配置过滤器
你可以使用`filters`扩展来配置测试计划的过滤器。默认情况下，所有的引擎和标记都会被包含在测试计划中。但只有默认的`includeClassNamePattern`(`^.*Tests?$`)会被应用。你可以重写默认的匹配模式，例如下面示例。当你使用了多种匹配模式时，JUnit Platform会使用逻辑 或 将它们合并起来使用。

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

如果你通过`engines {include …​}`或`engines {exclude …​}`来提供一个*测试引擎ID*，那么JUnit Gradle插件将只运行你希望运行的那个测试引擎。同样，如果你通过`tags {include …​}`或者`tags {exclude …​}`提供一个*标记*，JUnit Gradle插件将只运行相应标记的测试（例如，通过JUnit Jupiter测试的`@Tag`注解来过滤）。同理，关于包名，可以通过`packages {include …​}`或者`packages {exclude …​}`配置要包含或排除的包名。

<a id="配置参数-gradle"></a>

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
为了让JUnit Gradle插件运行所有测试，类路径中必须存在一个`TestEngine`的实现。

要支持基于JUnit Jupiter的测试，你需要配置一个JUnit Jupiter API的 `testCompile`依赖以及JUnit Jupiter `TestEngine`实现的`testRuntime`依赖，具体配置如下。

```groovy
dependencies {
    testCompile("org.junit.jupiter:junit-jupiter-api:5.0.2")
    testRuntime("org.junit.jupiter:junit-jupiter-engine:5.0.2")
}
```

只要你配置了一个JUnit 4的`testCompile`依赖以及JUnit Vintage `TestEngine`实现的`testRuntime`依赖，JUnit Gradle插件就可以运行基于JUnit 4的测试，具体配置如下。

```groovy
dependencies {
    testCompile("junit:junit:4.12")
    testRuntime("org.junit.vintage:junit-vintage-engine:4.12.2")
}
```

##### 使用JUnit Gradle插件
一旦应用并配置了JUnit Gradle插件，你就可以使用新的`junitPlatformTest`任务（在可用的Gralde task中会多出一个名为`junitPlatformTest`的Task）。

在命令行中调用`gradlew junitPlatformTest`（或者`gradlew test`）指令，项目中所有满足当前`includeClassNamePattern`（默认匹配`^.*Tests?$`）配置的测试会被执行。

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

> 📒 当任何一个容器或测试失败时，退出值为1；否则，退出值为0.

> ⚠️ **当前JUnit Gradle插件的限制**  
> 任何通过JUnit Gradle插件运行的测试结果都不会包含在Gradle生成的标准测试报告中；但通常可以在持续集成服务器上汇总测试结果。详情请参阅插件的`reportsDir`属性。


#### 4.2.2. Maven
JUnit团队已经为Maven Surefire开发了一个非常基础的provider，它允许你使用`mvn test`运行JUnit 4和JUnit Jupiter测试。[`junit5-maven-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.2/junit5-maven-consumer) 项目中的`pom.xml`文件演示了如何使用它，你可以以它作为一个起点。

> ⚠️ 由于Surefire2.20存在内存泄漏的漏洞，`junit-platform-surefire-provider`目前仅适用于Surefire 2.19.1。

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
为了让Maven Surefire运行所有测试，必须将`TestEngine`实现添加到运行时类路径中。

要支持基于JUnit Jupiter的测试，你需要配置一个JUnit Jupiter API的`test`依赖，并将JUnit Jupiter `TestEngine`的实现添加到`maven-surefire-plugin`的依赖项中，如下所示。

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

只要你配置了JUnit 4的`test`依赖，并将JUnit Vintage `TestEngine`的实现添加到`maven-surefire-plugin`的依赖项中，JUnit Platform Surefire Provider 就可以运行基于JUnit 4的测试。具体配置如下。

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

##### 按Tag过滤
使用以下配置属性，你可以通过Tag来过滤测试。

* 要包含一个 tag，可以使用`groups`或者`includeTags`
* 要排除一个 tag，可以使用`excludedGroups`或者`excludeTags`

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

<a id="配置参数-maven"></a>

##### 配置参数
你可以使用`configurationParameters`属性并以Java `Properties`文件的语法提供键值对来设置配置参数，从而影响测试发现和执行。

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
[`ConsoleLauncher`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html) 是一个Java的命令行应用程序，它允许你通过命令行来启动JUnit Platform。例如，它可以用来运行JUnit Vintage和JUnit Jupiter测试，并在控制台中打印测试结果。

`junit-platform-console-standalone-1.0.2.jar`这个包含了所有依赖的可执行的jar包已经被发布在Maven仓库中，它位于 [junit-platform-console-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone)目录下，你可以 [运行](https://docs.oracle.com/javase/tutorial/deployment/jar/run.html) 独立的ConsoleLauncher，如下所示。



java -jar junit-platform-console-standalone-1.0.2.jar<[Options](#431-options)>

如下所示为一个输出的例子。

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
> 如果任何容器或测试失败，[ConsoleLauncher](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html) 就会以状态码1退出，否则退出码为0.

#### 4.3.1. Options

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

### 4.4. 使用JUnit 4运行JUnit Platform
`JunitPlatform` 运行器是一个基于JUnit 4的`Runner`，它让你能够在一个JUnit 4环境中的JUnit Platform上运行那些编程模型被支持的任何测试。例如一个JUnit Jupiter测试类。

如果某个类被标注了`@RunWith(JUnitPlatform.class)`注解，它就可以在那些支持JUnit 4但是还不支持JUnit Platform的IDE和构建系统中直接运行。

>📒 由于JUnit Platform具备一些JUnit 4不具备的功能，因此运行器只能部分支持JUnit Platform的功能，特别是在报告方面（请参阅 [显示名称与技术名称](#442-显示名称与技术名称)）。但是就目前来说，`JUnitPlatform`运行器是一个简单的入门方式。

#### 4.4.1. 设置
你需要在类路径中添加以下的组件和它们的依赖。可以在 [依赖元数据](#21-依赖元数据) 中查看关于group ID, artifact ID 和版本的详细信息。

##### 显式依赖
* `junit-4.12.jar` 在*test* 范围内：使用JUnit 4运行测试。
* `junit-platform-runner` 在*test* 范围内：`JUnitPlatform`运行器的位置。
* `junit-jupiter-api` 在*test* 范围内：编写测试的API，包括 `@Test` 等。
* `junit-jupiter-engine` 在*test runtime* 范围内：JUnit Jupiter引擎API的实现。

##### 可传递的依赖
* `junit-platform-launcher` 在*test* 范围内
*  `junit-platform-engine` 在*test* 范围内
*  `junit-platform-commons` 在*test* 范围内
*  `opentest4j` 在*test* 范围内

#### 4.4.2. 展示名称与技术名称
默认情况下，*显示名称* 会被使用在测试产出物上，但是当`JUnitPlatform`运行器使用Gradle或者Maven等构建工具来运行测试时，生成的测试报告通常需要包含测试产出物的*技术名称*（例如，使用完整类名），而不是像测试类的简单名称或包含特殊字符的自定义显示名称这种较短的显示名称。为了在测试报告中使用技术名称，在`@RunWith(JUnitPlatform.class)`注解旁声明 `@UseTechnicalNames`注解即可。

#### 4.4.3. 单一测试类
使用`JUnitPlatform`运行器的方式之一是直接在测试类上添加 `@RunWith(JUnitPlatform.class)`注解。请注意，以下示例中的测试方法使用的注解是`org.junit.jupiter.api.Test`（JUnit Jupiter）,而不是 `org.junit.Test`(JUnit Vintage)。同时，这个类中的测试用例必须为 `public`，否则，IDE不能将其识别为一个JUnit 4的测试类。

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
如果你有多个测试类，你可以创建一个测试套件，如下例子所示。

```java
import org.junit.platform.runner.JUnitPlatform;
import org.junit.platform.suite.api.SelectPackages;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
@SelectPackages("example")
public class JUnit4SuiteDemo {
}
```

`JUnit4SuiteDemo`类会发现并运行所有位于`example`包及其子包下的测试。默认情况下，它只包含类名符合正则表达式`^.*Tests?$`的测试类。

>📒 ***附加配置选项***  
> 除了`@SelectPackages`之外，还有很多配置选项可以用来发现和过滤测试。详细内容请参考 [Javadoc](http://junit.org/junit5/docs/current/api/org/junit/platform/suite/api/package-summary.html).

### 4.5. 配置参数
除了告诉平台要包含哪些测试类、测试引擎以及要扫描哪些包等之外，有时还需要提供额外的自定义配置参数，该参数特定于特定的测试引擎。例如，JUnit Jupiter `TestEngine`支持以下用例中的*配置参数*。

* [更改默认的测试实例生命周期](#381-更改默认的测试实例生命周期)
* [启用自动扩展检测](#启用自动扩展检测)
* [禁用条件](#531-禁用条件)

*配置参数*是一种基于文本的键值对，可以通过以下任何一种机制将其提供给运行在JUnit Platform上的测试引擎。

1. `LauncherDiscoveryRequestBuilder`中的`configurationParameter()`和`configurationParameters()`方法可以用来构建提供给 [`Launcher` API](#71-junit-platform启动器api) 的请求。当使用JUnit Platform提供的某一种工具运行测试时，你可以采用如下所示的方式指定配置参数：
 * [控制台启动器](#43-控制台启动器): 使用`--config`命令行选项。
 * [Gradle插件](#配置参数-gradle): 使用`configurationParameter`或者`configurationParameters`DSL。
 * [Maven Surefire 提供者](#配置参数-maven): 使用 `configurationParameters` 属性。
2. JVM 系统属性。
3. JUnit Platform配置文件：该文件命名为`junit-platform.properties`，位于类路径根目录下，并遵循Java `Properties`文件的语法。

>📒 配置参数会按照上面定义的顺序查找。所以，直接提供给`Launcher`的配置参数优先于通过系统属性和配置文件提供的配置参数。同样，通过系统属性提供的配置参数优先于通过配置文件提供的参数。
