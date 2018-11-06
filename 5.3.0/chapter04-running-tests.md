## 4. 运行测试

### 4.1. IDE支持

#### 4.1.1. IntelliJ IDEA

IntelliJ IDEA 从 2016.2 版本开始支持在JUnit Platform上运行测试。详情请参阅 [IntelliJ IDEA的相关博客](https://blog.jetbrains.com/idea/2016/08/using-junit-5-in-intellij-idea/)。但是请注意，我们建议使用IDEA 2017.3或更新的版本，因为这些较新版本的IDEA会根据项目中使用的API版本自动下载这些JAR文件：`junit-platform-launcher`，`junit-jupiter-engine`和`junit-vintage-engine`。

> ⚠️ IntelliJ IDEA版本在IDEA 2017.3之前捆绑了特定版本的 JUnit 5。
因此，如果你想使用更新版本的JUnit Jupiter，那么执行测试
由于版本冲突，IDE可能会失败。在这种情况下，请按照说明进行操作
下面使用比IntelliJ IDEA捆绑的更新版本的JUnit 5。
 
要想使用JUnit 5的不同版本（比如，{{ jupiter-version }}），你需要在类路径中引入相应版本的`junit-platform-launcher`、`junit-jupiter-engine`和`junit-vintage-engine` JAR文件。

###### *添加Gradle依赖*

```java
// Only needed to run tests in a version of IntelliJ IDEA that bundles older versions
testRuntime("org.junit.platform:junit-platform-launcher:{{ platform-version }}")
testRuntime("org.junit.jupiter:junit-jupiter-engine:{{ jupiter-version }}")
testRuntime("org.junit.vintage:junit-vintage-engine:{{ vintage-version }}")
```

###### *添加Maven依赖*

```xml
<!-- Only needed to run tests in a version of IntelliJ IDEA that bundles older versions -->
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>{{ platform-version }}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>{{ jupiter-version }}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>{{ vintage-version }}</version>
    <scope>test</scope>
</dependency>
```


 
#### 4.1.2. Eclipse
自从Eclipse Oxygen.1a（4.7.1a）版本发布开始，Eclipse IDE提供了对JUnit平台的支持。

有关在Eclipse中使用JUnit 5的更多信息，请参阅官方 [Eclipse Project Oxygen.1a (4.7.1a) - New and Noteworthy](https://www.eclipse.org/eclipse/news/4.7.1a/#junit-5-support) 文档中的*Eclipse support
for JUnit 5* 章节。

#### 4.1.3. 其他 IDE
在本文写作之时，并没有其他任何IDE可以像IntelliJ IDEA和Eclipse一样可以直接在JUnit Platform上运行Java测试。但是，Junit团队提供了另外两种折中的方法让JUnit 5可以在其他的IDE上使用。你可以尝试手动使用 [控制台启动器](#43-控制台启动器) 或者通过 [基于JUnit 4的Runner](#44-使用junit-4运行junit-platform) 来执行测试。


### 4.2. 构建工具支持

#### 4.2.1. Gradle
从 [4.6](https://docs.gradle.org/4.6/release-notes.html) 开始，Gradle对在JUnit Platform执行测试提供了 [本地化支持](https://docs.gradle.org/5.2.0/userguide/java_testing.html#using_junit5) 。要启用此功能，你只需要`build.gradle`文件中的`test`任务声明里指定`useJUnitPlatform()`即可：

```groovy
test {
    useJUnitPlatform()
}
```

同样支持通过tag和引擎过滤：

```groovy
test {
    useJUnitPlatform {
        includeTags 'fast', 'smoke & feature-a'
        // excludeTags 'slow', 'ci'
        includeEngines 'junit-jupiter'
        // excludeEngines 'junit-vintage'
    }
}
```

有关详细的配置选项，请参考 [官方Gradle文档](https://docs.gradle.org/5.2.0/userguide/java_plugin.html#sec:java_test)。

> ⚠️ JUnit Platform Gradle 插件被弃用了
> 
> 有JUnit team开发的基础`junit-platform-gradle-plugin`在 JUnit Platform 1.2 中被弃用，并且在1.3中不再支持。请使用标准的Gradle `test`任务代替。

##### 配置参数
标准的Gradle `test` 任务当前没有提供一个专用的DSL用来设置JUnit Platform [配置参数](#45-配置参数)来影响测试发现和执行。不过，你可以通过系统属性（如下所示）或通过`junit-platform.properties`文件在构建脚本中提供配置参数。

```groovy
test {
    // ...
    systemProperty 'junit.jupiter.conditions.deactivate', '*'
    systemProperties = [
        'junit.jupiter.extensions.autodetection.enabled': 'true',
        'junit.jupiter.testinstance.lifecycle.default': 'per_class'
    ]
    // ...
}
```

##### 配置测试引擎
为了运行任何测试，在类路径中必须存在一个`TestEngine`的实现。

要支持基于JUnit Jupiter的测试，配置一个JUnit Jupiter API的`testCompile`依赖以及JUnit Jupiter `TestEngine`的`testComiple`依赖，类似如下配置：

```groovy
dependencies {
    testCompile("org.junit.jupiter:junit-jupiter-api:5.2.0")
    testRuntime("org.junit.jupiter:junit-jupiter-engine:5.2.0")
}
```

只要你配置了一个JUnit 4的`testCompile`依赖和一个JUnit Vintage `TestEngine`的`testRuntime`依赖，JUnit Platform就可以运行基于JUnit 4的测试，类似如下配置：

```groovy
dependencies {
    testCompile("junit:junit:4.12")
    testRuntime("org.junit.vintage:junit-vintage-engine:5.2.0")
}
```

##### 配置日志记录（可选）
JUnit 使用了位于`java.util.logging`包（a.k.a.*JUL*）下的Java Logging API来发布警告和调式信息。关于配置选项，请参阅官方文档的 {{ LogManager }} 章节。

或者，你也可以将日志信息重定向到其他的日志框架，比如 {{Log4j}} 或者 {{Logback}}。要使用提供了自定义的 {{LogManager}}实现的日志框架，将`java.util.loggin.manager`系统属性设置为 {{LogManager}} *全类名* 即可。下面的例子演示了如何配置Log4j 2.x（有关详细信息，请参阅 {{ Log4j_JDK_Logging_Adapter }}）。

```groovy
test {
    systemProperty 'java.util.logging.manager', 'org.apache.logging.log4j.jul.LogManager'
}
```

其他的日志框架提供了不同的方式来重定向`java.util.logging`记录的信息。例如，对于 {{Logback}}，你可以通过向runtime类路径添加一个额外的依赖来启用{{JUL_to_SLF4J_Bridge}}。

#### 4.2.2. Maven
> 📒 最初由JUnit团队开发的自定义`junit-platform-surefire-provider`已被弃用，并计划在JUnit Platform 1.4中删除。 请改用Maven Surefire的原生支持。

从版本 [2.22.0](https://issues.apache.org/jira/browse/SUREFIRE-1330) 开始，Maven Surefire为在JUnit平台上执行测试提供 [原生支持](http://maven.apache.org/surefire/maven-surefire-plugin/examples/junit-platform.html)。 {{junit5-jupiter-starter-maven}}项目中的pom.xml文件演示了如何使用它，并可以作为配置Maven构建的起点。

##### 配置测试引擎
为了让Maven Surefire运行任何测试，必须至少将一个`TestEngine`实现添加到测试类路径中。

要配置对基于JUnit Jupiter的测试的支持，请在JUnit Jupiter API和JUnit Jupiter `TestEngine`实现上配置`test`范围依赖项，类似于以下内容。


```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>{{ surefire-version }}</version>
        </plugin>
    </plugins>
</build>
...
<dependencies>
    ...
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>{{ jupiter-version }}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine </artifactId>
        <version>{{ jupiter-version }}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
...
```

只要你在JUnit 4和JUnit Vintage `TestEngine`实现上配置`test`范围依赖项，Maven Surefire就可以运行基于JUnit 4的测试以及Jupiter测试，类似于以下配置：


```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>{{ surefire-version }}</version>
        </plugin>
    </plugins>
</build>
...
<dependencies>
    ...
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>{{ junit4-version }}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
    	  <groupId>org.junit.vintage</groupId>
        <artifactId>junit-vintage-engine</artifactId>
        <version>{{vintage-version}}</version>
        <scope>test</scope>
    </dependency>    
</dependencies>
...
```

##### 按测试类名过滤
Maven Surefire插件将扫描全类名与以下模式匹配的测试类。

- `**/Test*.java`

- `**/*Test.java`

- `**/*Tests.java`

- `**/*TestCase.java`

此外，它默认会排除所有内嵌类（包括静态成员类）。

但是请注意，你可以通过在`pom.xml`文件中配置显式`include`和`exclude`规则来覆盖其默认行为。例如，要阻止Maven Surefire排除静态成员类，你可以覆盖它的排除规则。

*覆盖Maven Surefire的排除规则*
```xml
...
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>{{ surefire-version }}</version>
            <configuration>
                <excludes>
                    <exclude/>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
...
```

有关详细信息，请参阅 [Inclusions and Exclusions of Tests](https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html) 的文档。

##### 按Tag过滤
使用以下配置属性，你可以通过Tag来过滤测试。

* 要包含一个 *tags* 或者 *tag expressions*，可以使用`groups`
* 要排除一个 *tags* 或者 *tag expressions*，可以使用`excludedGroups`

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>{{ surefire-version }}</version>
            <configuration>
                <groups>acceptance | !feature-a</groups>
                <excludedGroups>integration, regression</excludedGroups>
            </configuration>
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
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>{{ surefire-version }}</version>
            <configuration>
                <properties>
                    <configurationParameters>
                        junit.jupiter.conditions.deactivate = *
                        junit.jupiter.extensions.autodetection.enabled = true
                        junit.jupiter.testinstance.lifecycle.default = per_class
                    </configurationParameters>
                </properties>
            </configuration>
        </plugin>
    </plugins>
</build>
...
```

#### 4.2.3. Ant
[Ant](https://ant.apache.org/) 从`1.10.3`开始，引入了一个新的 [`junitlauncher`](https://ant.apache.org/manual/Tasks/junitlauncher.html) 任务来提供在JUnit Platform上加载测试的本地化支持。`junitlauncher`任务单独负责加载JUnit Platform并将选定的测试集合传递给它。然后，JUnit Platform委托已注册的测试引擎来发现和执行测试。

`junitlauncher`任务尝试尽可能地与本地Ant构造（如 [资源集合](https://ant.apache.org/manual/Types/resources.html#collection)）保持一致，以便用户选择他们想要由测试引擎执行的测试。与许多其他核心Ant任务相比，这赋予了该任务一致且自然的感觉。

>📒 Ant 1.10.3中提供的`junitlauncher`任务版本为启动JUnit平台提供了基本的最小支持。其他增强功能（包括支持在单独的JVM中分支测试）将在随后的Ant版本中提供。

{{junit5-jupiter-starter-ant}} 项目中的`build.xml`文件演示了如何使用它，你可以以它作为一个起点。

##### 基本用法
以下示例演示了如何配置`junitlauncher`任务以选择一个单独的测试类（即：`org.myapp.test.MyFirstJUnit5Test`）：

```xml
<path id="test.classpath">
    <!-- The location where you have your compiled classes -->
    <pathelement location="${build.classes.dir}" />
</path>

<!-- ... -->

<junitlauncher>
    <classpath refid="test.classpath" />
    <test name="org.myapp.test.MyFirstJUnit5Test" />
</junitlauncher>
```
`test` 元素允许你指定你想要选择和执行的单个测试类。`classpath`元素允许你指定用于启动JUnit平台的类路径。这个类路径也将用于定位属于执行部分的测试类。

以下示例演示如何配置`junitlauncher`任务以从多个位置选择测试类：

```xml
<path id="test.classpath">
    <!-- The location where you have your compiled classes -->
    <pathelement location="${build.classes.dir}" />
</path>
....
<junitlauncher>
    <classpath refid="test.classpath" />
    <testclasses outputdir="${output.dir}">
        <fileset dir="${build.classes.dir}">
            <include name="org/example/**/demo/**/" />
        </fileset>
        <fileset dir="${some.other.dir}">
            <include name="org/myapp/**/" />
        </fileset>
    </testclasses>
</junitlauncher>
```

在上面的示例中，`testclasses`元素允许你选择位于不同位置的多个测试类。

有关使用和配置选项的更多详细信息，请参阅 [`junitlauncher`任务](https://ant.apache.org/manual/Tasks/junitlauncher.html) 的官方Ant文档。

### 4.3. 控制台启动器
{{ConsoleLauncher}} 是一个Java的命令行应用程序，它允许你通过命令行来启动JUnit Platform。例如，它可以用来运行JUnit Vintage和JUnit Jupiter测试，并在控制台中打印测试结果。

`junit-platform-console-standalone-{{platform-version}}.jar`这个包含了所有依赖的可执行的jar包已经被发布在Maven仓库中，它位于 [junit-platform-console-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone)目录下，你可以 [运行](https://docs.oracle.com/javase/tutorial/deployment/jar/run.html) 独立的ConsoleLauncher，如下所示。

java -jar junit-platform-console-standalone-{{platform-version}}.jar<[Options](#431-options)>

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
> 如果任何容器或测试失败，{{ConsoleLauncher}} 将以状态码`1`退出。 如果未发现任何测试并且提供了`--fail-if-no-tests`命令行选项，则`ConsoleLauncher`将以状态代码`2`退出。否则退出代码为0。


#### 4.3.1. Options

```sh
Usage: ConsoleLauncher [-h] [--disable-ansi-colors] [--fail-if-no-tests] [--scan-modules]
                       [--scan-classpath[=PATH[;|:PATH...]]]... [--details=MODE]
                       [--details-theme=THEME] [--reports-dir=DIR]
                       [--config=KEY=VALUE]... [--exclude-package=PKG]...
                       [--include-package=PKG]... [-c=CLASS]... [-cp=PATH[;|:PATH...]]...
                       [-d=DIR]... [-e=ID]... [-E=ID]... [-f=FILE]... [-m=NAME]...
                       [-n=PATTERN]... [-N=PATTERN]... [-o=NAME]... [-p=PKG]...
                       [-r=RESOURCE]... [-t=TAG]... [-T=TAG]... [-u=URI]...
Launches the JUnit Platform from the console.
  -h, --help                 Display help information.
      --disable-ansi-colors  Disable ANSI colors in output (not supported by all terminals).
      --details=MODE         Select an output details mode for when tests are executed. Use
                               one of: none, summary, flat, tree, verbose. If 'none' is
                               selected, then only the summary and test failures are shown.
                               Default: tree.
      --details-theme=THEME  Select an output details tree theme for when tests are executed.
                               Use one of: ascii, unicode. Default: unicode.
      -cp, --classpath, --class-path=PATH[;|:PATH...]
                             Provide additional classpath entries -- for example, for adding
                               engines and their dependencies. This option can be repeated.
      --fail-if-no-tests     Fail and return exit status code 2 if no tests are found.
      --reports-dir=DIR      Enable report output into a specified local directory (will be
                               created if it does not exist).
      --scan-modules         EXPERIMENTAL: Scan all resolved modules for test discovery.
  -o, --select-module=NAME   EXPERIMENTAL: Select single module for test discovery. This
                               option can be repeated.
      --scan-classpath, --scan-class-path[=PATH[;|:PATH...]]
                             Scan all directories on the classpath or explicit classpath
                               roots. Without arguments, only directories on the system
                               classpath as well as additional classpath entries supplied via
                               -cp (directories and JAR files) are scanned. Explicit classpath
                               roots that are not on the classpath will be silently ignored.
                               This option can be repeated.
  -u, --select-uri=URI       Select a URI for test discovery. This option can be repeated.
  -f, --select-file=FILE     Select a file for test discovery. This option can be repeated.
  -d, --select-directory=DIR Select a directory for test discovery. This option can be
                               repeated.
  -p, --select-package=PKG   Select a package for test discovery. This option can be repeated.
  -c, --select-class=CLASS   Select a class for test discovery. This option can be repeated.
  -m, --select-method=NAME   Select a method for test discovery. This option can be repeated.
  -r, --select-resource=RESOURCE
                             Select a classpath resource for test discovery. This option can
                               be repeated.
  -n, --include-classname=PATTERN
                             Provide a regular expression to include only classes whose fully
                               qualified names match. To avoid loading classes unnecessarily,
                               the default pattern only includes class names that begin with
                               "Test" or end with "Test" or "Tests". When this option is
                               repeated, all patterns will be combined using OR semantics.
                               Default: [^(Test.*|.+[.$]Test.*|.*Tests?)$]
  -N, --exclude-classname=PATTERN
                             Provide a regular expression to exclude those classes whose fully
                               qualified names match. When this option is repeated, all
                               patterns will be combined using OR semantics.
      --include-package=PKG  Provide a package to be included in the test run. This option can
                               be repeated.
      --exclude-package=PKG  Provide a package to be excluded from the test run. This option
                               can be repeated.
  -t, --include-tag=TAG      Provide a tag or tag expression to include only tests whose tags
                               match. When this option is repeated, all patterns will be
                               combined using OR semantics.
  -T, --exclude-tag=TAG      Provide a tag or tag expression to exclude those tests whose tags
                               match. When this option is repeated, all patterns will be
                               combined using OR semantics.
  -e, --include-engine=ID    Provide the ID of an engine to be included in the test run. This
                               option can be repeated.
  -E, --exclude-engine=ID    Provide the ID of an engine to be excluded from the test run.
                               This option can be repeated.
      --config=KEY=VALUE     Set a configuration parameter for test discovery and execution.
                               This option can be repeated.
```

#### 4.3.2. 参数文件 (@-files)
在某些平台上，在创建包含大量选项或长参数的命令行时，可能会遇到命令行长度的系统限制。

从版本1.3开始，`ConsoleLauncher`支持参数文件，也称为*@-files*。参数文件是本身包含要传递给命令的参数的文件。当底层[picocli](https://github.com/remkop/picocli)命令行解析器遇到以字符`@`开头的参数时，它会将该文件的内容扩展到参数列表中。

文件中的参数可以用空格或换行符分隔。如果参数包含嵌入的空格，则整个参数应包含在双引号或单引号中 -- 例如，`"-f = My Files/Stuff.java"`。

如果参数文件不存在或无法读取，则参数将按字面处理，不会被删除。这可能会导致"不匹配的参数"的错误消息。你可以通过执行`picocli.trace`系统属性设置为`DEBUG`的命令来解决此类错误。

可以在命令行上指定多个*@-files*。指定的路径可以是相对于当前目录的相对路径或绝对路径。

你可以通过使用额外的`@`符号转义以`@`开始的字符的参数。例如，`@@somearg`将成为`@somearg`，不会被扩展。


### 4.4. 使用JUnit 4运行JUnit Platform
`JunitPlatform` 运行器是一个基于JUnit 4的`Runner`，它让你能够在一个JUnit 4环境中的JUnit Platform上运行那些编程模型被支持的任何测试。例如一个JUnit Jupiter测试类。

如果某个类被标注了`@RunWith(JUnitPlatform.class)`注解，它就可以在那些支持JUnit 4但是还不支持JUnit Platform的IDE和构建系统中直接运行。

>📒 由于JUnit Platform具备一些JUnit 4不具备的功能，因此运行器只能部分支持JUnit Platform的功能，特别是在报告方面（请参阅 [显示名称与技术名称](#442-显示名称与技术名称)）。但是就目前来说，`JUnitPlatform`运行器是一个简单的入门方式。

#### 4.4.1. 设置
你需要在类路径中添加以下的组件和它们的依赖。可以在 [依赖元数据](#21-依赖元数据) 中查看关于group ID, artifact ID 和版本的详细信息。

##### 显式依赖
* `junit-{{ junit4-version }}.jar` 在*test* 作用域内：使用JUnit 4运行测试。
* `junit-platform-runner` 在*test* 作用域内：`JUnitPlatform`运行器的位置。
* `junit-jupiter-api` 在*test* 作用域内：编写测试的API，包括 `@Test` 等。
* `junit-jupiter-engine` 在*test runtime* 范围内：JUnit Jupiter引擎API的实现。

##### 可传递的依赖
* `junit-platform-launcher` 在*test* 作用域内
*  `junit-platform-engine` 在*test* 作用域内
*  `junit-platform-commons` 在*test* 作用域内
*  `opentest4j` 在*test* 作用域内

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

`JUnit4SuiteDemo`类会发现并运行所有位于`example`包及其子包下的测试。默认情况下，它只包含类名以`Test`开始或者以`Test`或`Tests结束的测试类。


>📒 ***附加配置选项***  
> 除了`@SelectPackages`之外，还有很多配置选项可以用来发现和过滤测试。详细内容请参考 [Javadoc](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/suite/api/package-summary.html).

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

### 4.6. 标记表达式

标记表达式是运算符`！`，`＆`和`|`的布尔表达式。另外，`（`和`）`可用于调整运算符优先级。

*表 1. 运算符（按优先顺序降序排列）*

| **运算符** | **含义** | **关联性** |
|:-------------|:------------|:------------|
| `!` | 非 | right |
| `&` | 与 | left |
| `|` | 或 | left |

如果你在多个维度上标记测试，tag expressions 可帮助您选择要执行的测试。通过测试类型（例如，*micro*, *integration*, *end-to-end*）和特征（例如，**foo**，**bar**，**baz**）标记以下表达式可能很有用。

| **标记表达式** | **选择** |
|:-------------|:------------|
| `foo` | **foo**的所有测试 |
| `bar | baz` | **bar**和**baz**的所有测试 |
| `bar & baz` | **bar**和**baz**的测试交集 |
| `foo & !end-to-end` | **foo**的所有测试，但不是*端到端测试* |
| `(micro | integration) & (foo | baz)` | **foo**或**baz**的所有微测试或集成测试|

### 4.7. 捕获标准输出/错误
从版本1.3开始，JUnit平台提供了可选择的支持，用于捕获打印到`System.out`和`System.err`的输出。 要启用它，只需将`junit.platform.output.capture.stdout`和/或`junit.platform.output.capture.stderr` [配置参数](#45-配置参数) 设置为`true`即可。 此外，你可以使用`junit.platform.output.capture.maxBuffer`配置每个执行的测试或容器使用的最大缓冲字节数。

启用后，JUnit Platform会在报告测试或容器完成之前立即捕获相应的输出并使用`stdout`或`stderr`键将其作为报告条目发布到所有已注册的{{TestExecutionListener}}实例。

请注意，捕获的输出将仅包含用于执行容器或测试的线程发出的输出。其他线程的任何输出都将被省略，因为特别是在 [并行执行测试](#317-并行执行) 时，不可能将其归因于特定的测试或容器。

> ⚠️ 捕获输出目前是一项*实验性*功能。 你被邀请尝试并向JUnit团队提供反馈，以便他们可以 [改进](#8-api演变) 并最终推广此功能。




