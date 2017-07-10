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

