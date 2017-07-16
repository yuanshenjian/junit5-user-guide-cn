# 4.运行测试

## 4.1 IDE支持

### 4.1.1 IntelliJ IDEA

IntelliJ IDEA 从 2016.2 版本开始支持在JUnit平台上运行Java测试。更多的相关细节可以参考[IntelliJ IDEA的相关博客](https://blog.jetbrains.com/idea/2016/08/using-junit-5-in-intellij-idea/)。

### 4.1.2 Eclipse 测试版支持

Eclipse 4.7（Oxygen）的测试版支持JUnit平台和Junit Jupiter。而关于如何让 JUnit 5 在Eclipse上运行起来，可以参考[Eclipse JDT UI/JUnit 5](https://wiki.eclipse.org/JDT_UI/JUnit_5)的wiki页面。

### 4.1.3 其他IDE

在本文写作之时，并没有其他任何IDE可以像 IntelliJ IDEA 和 Eclipse 的测试版一样可以直接通过 JUnit 平台运行 Java 测试。但是，Junit 团队提供了另外两种折中的方法让 JUnit 5 可以在其他的 IDE 上使用。用户可以尝试通过手动的 [Console Launcher](http://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher) 或者通过[ JUnit 4 based Runner](http://junit.org/junit5/docs/current/user-guide/#running-tests-junit-platform-runner) 执行测试。

## 4.2 构建工具支持

### 4.2.1 Gradle

JUnit 开发团队已经开发了一款基于Gradle的Junit 5 插件，它可以让使用者运行任何一种已经被支持的`TestEngine`（例如：JUnit 3、JUnit 4、JUnit Jupiter以及[Specsy](http://specsy.org/)等）。可以通过查看[`junit5-gradle-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.0-M4/junit5-gradle-consumer)中的`build.gradle`文件，将其作为一个使用插件的例子学习。

### 使用JUnit Gradle插件

要想使用 JUnit Gradle 插件，开发者首先需要确认当前的 Gradle 版本是2.5或更高。只要确认无误，就可以按以下模板配置项目中的`build.gradle`文件了。

```
buildscript {
    repositories {
        mavenCentral()
        // The following is only necessary if you want to use SNAPSHOT releases.
        // maven { url 'https://oss.sonatype.org/content/repositories/snapshots' }
    }
    dependencies {
        classpath 'org.junit.platform:junit-platform-gradle-plugin:1.0.0-M4'
    }
}

apply plugin: 'org.junit.platform.gradle.plugin'
```

### 配置 JUnit Gradle 插件

只要 JUnit Gradle 插件被使用，开发者就可以按照如下方式进行配置。

>这里配置的的选项在开发工作的过程中，很有可能是需要不断变更的。

```
junitPlatform {
    platformVersion 1.0
    logManager 'org.apache.logging.log4j.jul.LogManager'
    reportsDir file('build/test-results/junit-platform') // this is the default
    // enableStandardTestTask true
    // selectors (optional)
    // filters (optional)
}
```

设置`logManager`可以使得JUnit Gradle 插件去设置`java.util.logging.manager`的系统参数以便确保使用`java.util.logging.LogManager`的实现的*fully qualified class*名称。如上述例子就展示了如何将log4j配置为`LogManager`。

JUnit Gradle 插件在默认情况下是无法使用标准的 Gradle task 指令`test`的，但是可以通过重写`enableStandardTestTask `任务标志来修改。

### 配置Selectors

默认情况下，插件将会扫描项目中所有的测试输出文件夹。但是，开发者可以通过使用`selectors`的扩展元素来明确指定哪些测试应该被执行。

```
junitPlatform {
    // ...
    selectors {
        uris 'file:///foo.txt', 'http://example.com/'
        uri 'foo:resource' //URIs
        files 'foo.txt', 'bar.csv'
        file 'qux.json' //本地文件
        directories 'foo/bar', 'bar/qux'
        directory 'qux/bar' //本地文件夹
        packages 'com.acme.foo', 'com.acme.bar'
        aPackage 'com.example.app' //包名
        classes 'com.acme.Foo', 'com.acme.Bar'
        aClass 'com.example.app.Application' //类，完整类名称 
        methods 'com.acme.Foo#a', 'com.acme.Foo#b'
        method 'com.example.app.Application#run(java.lang.String[])' //方法，完整的方法名称
        resources '/bar.csv', '/foo/input.json'
        resource '/com/acme/my.properties' //资源路径
    }
    // ...
}

```

### 配置Filters

开发者可以通过使用`Filter`扩展完成对测试计划的过滤配置。默认情况下，所有的引擎和标签都会被包含在测试计划中。这时，只有默认的`includeClassNamePattern 
(^.*Tests?$)`被使用。开发者可应通过重写默认的匹配器就像下面的示例一样。当开发者表明要使用多种匹配器时，可以通过逻辑或将它们合并使用。

```
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

如果开发者为`engines {include …​} `或`engines {exclude …​}`的提供了*测试引擎ID*，那么JUnit Gradle插件将会只去运行开发者所希望运行的那个测试引擎。相似地，如果开发者为`tags {include …​}`或者`tags {exclude …​}`提供了*标签*，JUnit Gradle插件将只会处理含有这个标签的测试（例如，通过一个`@Tag`注解标注基于JUnit Jupiter测试）。相同的使用方式还可以应用在package名称上，例如`packages {include …​}`或者`packages {exclude …​}`。

### 配置测试引擎

为了能够使 JUnit Gradle 插件运行任何一个测试，必须给出`TestEngine`的实现的classpath。

要配置支持基于JUnit Jupiter的测试的应用，就需要在`testCompile `完成JUnit Jupiter API的项目构建依赖配置以及在`testRuntime `完成对 JUnit Jupiter `TestEngine`实现的运行时依赖配置.具体如下：

```
dependencies {
    testCompile("org.junit.jupiter:junit-jupiter-api:5.0.0-M4")
    testRuntime("org.junit.jupiter:junit-jupiter-engine:5.0.0-M4")
}
```

JUnit Gradle 插件可以运行基于JUnit 4 的测试就需要开发者配置`testCompile`依赖为JUnit 4 以及 `testRuntime`依赖为JUnit Vintage 的`TestEngine`实现。具体代码如下：

```
dependencies {
    testCompile("junit:junit:4.12")
    testRuntime("org.junit.vintage:junit-vintage-engine:4.12.0-M4")
}
```

### 使用 JUnit Gradle 插件

只要 JUnit Gradle插件被应用且配置完毕，
在Gralde的task中，就会多出一个`junitPlatformTest `task。

通过在命令行调用`gradlew junitPlatformTest` (or `gradlew test`)指令，可以执行项目中所有满足当前`includeClassNamePattern `配置的测试。（默认匹配`^.*Tests?$`）

在[`junit5-gradle-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.0-M4/junit5-gradle-consumer)项目中，执行`junitPlatformTest`任务的输出结果如下：

```
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

如果测试包含不通过的情况，那么build会失败，并且输出会如下所示：

```
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

> 当任何一个容器有误或者测试失败时，退出值为1；否则，值为0.

> **目前JUnit Gradle插件的限制**
> 目前所有通过JUnit Gradle插件完成的测试结果都无法被包含在标准生成的的Gradle测试报告中；但是，这些测试结果可以像以往一样，被记录于持续集成的服务器上。通过`reportsDir `插件的属性可以找到报告。
> 

### 4.2.2. Maven
为了能够通过`mvn test`运行JUnit 4和 JUnit Jupiter，JUnit团队为Maven Surefire提供了基础的支持保证。项目[`junit5-maven-consumer`](https://github.com/junit-team/junit5-samples/tree/r5.0.0-M4/junit5-maven-consumer)中的`pom.xml`文件展示了如何作为一个开始并使用的其的描述。

```
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
                    <version>1.0.0-M4</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

### 配置测试引擎
为了使 Maven Surefire 能够运行所有的测试，`TestEngine`的实现必须加到运行的路径中。

如下所示演示了以下内容，包括：配置并为JUnit Jupiter的基本测试提供支持，为JUnit Jupiter API配置`test`依赖，为`maven-surefire-plugin`增加JUnit Jupiter的`TestEngine`实现依赖。

```
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
                    <version>1.0.0-M4</version>
                </dependency>
                <dependency>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter-engine</artifactId>
                    <version>5.0.0-M4</version>
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
        <version>5.0.0-M4</version>
        <scope>test</scope>
    </dependency>
</dependencies>
...
```

为了使 JUnit Platform Surefire Provider 运行 JUnit4 的基本测试，可以按照如下方法在JUnit4上配置`test`依赖，并增加`maven-surefire-plugin`的JUnit Vintage `TestEngine`实现依赖。

```
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
                    <version>1.0.0-M4</version>
                </dependency>
                ...
                <dependency>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                    <version>4.12.0-M4</version>
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

### tag过滤测试
使用以下结构属性，可以通过 tag 过滤测试方法：

* 为了包含一个 tag，可以使用`groups`或者`includeTags`
* 为了不包含一个 tag，可以使用`excludedGroups`或者`excludeTags`

```
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

## 4.3 运行控制台
[ConsoleLauncher](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)是一个命令行的Java应用程序，它能使得JUnit平台在命令行启动。例如，它可以用来运行 JUnit Vintage 和 JUnit Jupiter 测试，并在命令行输入测试结果。

可运行的 `junit-platform-console-standalone-1.0.0-M4.jar`，包括所有的依赖，已经在 central Maven 库中发布了，路径是[junit-platform-console-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone/),可以通过以下命令单独[运行](https://docs.oracle.com/javase/tutorial/deployment/jar/run.html)`ConsoleLauncher `

```
java -jar junit-platform-console-standalone-1.0.0-M4.jar <Options>
```
如下所示为一个输出的例子：

```
├─ JUnit Vintage
│  ├─ example.JUnit4Tests
│  │  ├─ standardJUnit4Test ✔
├─ JUnit Jupiter
│  ├─ StandardTests
│  │  ├─ succeedingTest() ✔
│  │  ├─ skippedTest() ↷ for demonstration purposes
│  ├─ A special test case
│  │  ├─ Custom test name containing spaces ✔
│  │  ├─ ╯°□°）╯ ✔
│  │  ├─ 😱 ✔

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

> ##### 返回值
> 如果[ConsoleLauncher](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)的返回的状态值为1，则代表有容器或测试运行失败，否则返回0.

### Options

为了最终的发布成功，options经常需要改变。

```
Option                                        Description
------                                        -----------
-h, --help                                    Display help information.
--disable-ansi-colors                         Disable ANSI colors in output (not
                                                supported by all terminals).
--hide-details                                @Deprecated. Use '--details none' instead.
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
```

## 4.4 使用JUnit4运行JUnit Platform
`JunitPlatform`是一个基于JUnit4的运行器，它可以运行任何在JUnit4环境中使用JUnit Platform编写的程序，例如，JUnit Jupiter测试类。

对一个类使用`@RunWith(JUnitPlatform.class)`注释，就可以使支持JUnit4但是还不支持JUnit Platform的程序直接在IDE中编译并运行。

> 由于JUnit Platform的一些功能JUnit4没有，运行器只能部分支持JUnit Platform的功能，尤其针对报告中的一些内容（见[命名显示 vs 科学命名](http://junit.org/junit5/docs/current/user-guide/#running-tests-junit-platform-runner-technical-names)）但对于刚开始启动`JUnitPlatform`运行器，还是比较容易的。

### 4.4.1 启动
可以在项目的路径中设置artifacts和其相关依赖，可以在[依赖元数据](http://junit.org/junit5/docs/current/user-guide/#dependency-metadata)中查看关于group ID, artifact ID 和版本的细节问题。

### 详细依赖
* `junit-4.12.jar` 在测试范围内：使用JUnit4运行测试
* `junit-platform-runner` 在测试范围内：定位于`JUnitPlatform`运行器
* `junit-jupiter-api`在测试范围内：使用API编写测试用力，包括`@Test`等
* `junit-jupiter-engine`在测试运行范围内：为JUnit Jupiter实现Engine的API方法

### 传递依赖关系
* `junit-platform-launcher` 在测试范围内
*  `junit-platform-engine` 在测试范围内
*  `junit-platform-commons` 在测试范围内
*  `opentest4j` 在测试范围内


### 4.4.2 命名显示 vs 科学命名
默认情况下，命名会被使用在test artifacts上，但是当`JUnitPlatform`运行器使用Gradle或者Maven等编译工具来运行测试，生成的测试报告需要使用test artifacts的科学命名方式，例如，使用完整类名，而不是使用缩写类名，或者自定义的包含特殊字符的类名。为了达到测试报告的科学命名，可以在`@RunWith(JUnitPlatform.class)`注释旁边简单的声明`@UseTechnicalNames`注释。


### 4.4.3 Single测试类
使用`JUnitPlatform`运行器的方法之一是，直接用`@RunWith(JUnitPlatform.class)`注释测试类，注意下面例子中的测试类的注释使用`org.junit.jupiter.api.Test`（JUnit Jupiter）,而不是`org.junit.Test`(JUnit Vintage)。同时，这个类中的测试用例必须是`public`，否则，IDE不能将其识别为JUnit4的测试类。

```
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

### 4.4.4 测试套件
如果有多个测试类，可以通过创建测试套件完成测试，如下例子所示：

```
import org.junit.platform.runner.JUnitPlatform;
import org.junit.platform.suite.api.SelectPackages;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
@SelectPackages("example")
public class JUnit4SuiteDemo {
}
```

`JUnit4SuiteDemo`类会寻找并运行所有在`example`包及其子包下的测试。默认情况下，它只包含名字符合正则表达式的`^.*Tests?$`测试类。

>### 附加配置选项
> 注释 `@SelectPackages`可以用来额外配置更多的选项，用来寻找和过滤测试。更多相关介绍见[Javadoc](http://junit.org/junit5/docs/current/api/org/junit/platform/suite/api/package-summary.html).









