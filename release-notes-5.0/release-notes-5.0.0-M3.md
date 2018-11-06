### 5.0.0-M3
**发布时间**： 2016.11.30

**范围**：JUnit 5 第三次发布，重点是JUnit 4的互操作性、附加的发现选择器以及文档。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0 M3](https://github.com/junit-team/junit5/milestone/6?closed=1) 里程碑页面。

#### JUnit Platform

###### Bug修复

* 现在，在打印异常信息时，`ConsoleLauncher`所使用的`ColoredPrintingTestListener`输出实际异常类型以及堆栈跟踪。
* 在扫描classpath根目录时，现在通过类路径扫描来获取*默认*包中的测试类。例如，在没有选择显式包的情况下，会使用JUnit Platform Gradle插件。
* 类路径扫描不再加载类名过滤器排除的类。
* 类路径扫描不再尝试加载Java 9的`module-info.class`文件。

###### 弃用与彻底改变

* 重命名`ClasspathSelector`为`ClasspathRootSelector`，以避免与`ClasspathResourceSelector`混淆。
* 重命名`JavaPackageSource`为`PackageSource`，以便于`PackageSelector`的命名方式保持一致。
* 重命名`JavaMethodSource`为`MethodSource`，以便于`MethodSelector`的命名方式保持一致。
* 现在，`PackageSource`，`ClassSource`，以及`MethodSource`直接实现`TestSource`接口，而不是已经移除的`JavaSource`接口。
* `DiscoverySelectors`中，基于名称的通用发现选择器（例如`selectName()`和`selectNames()`）已经被弃用，取而代之的是更加专用的`selectPackage(String)`，`selectClass(String)`，以及`selectMethod(String)`方法。
* 现在，`ClassFliter`已经被重命名为`ClassNameFilter`，并且实现了`DiscoveryFilter<String>`，而非之前的`DiscoveryFilter<Class<?>>`。因此可以在类路径扫描的期间，在类加载之前应用它。
* 现在，`ClassNameFilter.includeClassNamePattern`被弃用，并以`ClassNameFilter.includeClassNamesPatterns`取而代之。
* 现在，`@IncludeClassNamePattern`被弃用，并以`@IncludeClassNamePatterns`取而代之。
* 用于配置`ConsoleLauncher`其他类路径条目的命令行选项`-p`已被重命名为`-cp`，以便与`java`其他可执行执行选项保持一致。另外，为`--classpath`命令行选项引入了一个新的别名`--class-path`，而原有命令行保持不变。
* 对`ConsoleLauncher`的命令行选项`-a`和`--all`已被重命名为`--scan-class-path`。
* 对`ConsoleLauncher`的短命令行选项`-C`，`-D`，以及`-r`都被弃用，取而代之的是对应的长命令行选项`--disable-ansi-colors`，`--hide-details`，以及`--reports-dir`，分别与之等价。
* `ConsoleLauncher`不再支持无选择参数的使用方式。请使用新的显式选择器选项以明确包名、类名或方法名。另外，`--scan-class-path`现在可以接受一个可选参数，该参数可被用来选择需要被扫描的类路径。涉及多条类路径时，可以使用系统自带的路径分隔符将之隔离（Windows使用`;`，Unix使用`:`）。
* `LauncherDiscoveryRequestBuilder`不再接收`selectors`，`filters`或配置参数映射的`null`值。
* 现在需要将Gradle插件配置中的类名称、引擎ID和标记过滤器包装在`filters`扩展元素中（请参阅 [配置过滤器](#配置过滤器)）。

#### 新特性

* `DiscoverySelectors`中新增的`selectUri（...）`方法用于选择URI。 `TestEngine`可以通过查询`UriSelector`的注册实例来检索这些值。
* 在`DiscoverySelectors`中新增的`selectFile（...）`和`selectDirectory（...）`方法可以用于选择文件系统中的文件和目录。`TestEngine`可以通过查询`FileSelector`和`DirectorySelector`的已注册实例来检索这些值。
* `DiscoverySelectors`中的新的`selectClasspathResource（String）`方法，用于按名称选择类路径资源（如XML或JSON文件），其中名称是当前类路径中资源的分离路径名。`TestEngine`可以通过查询`ClasspathResourceSelector`的已注册实例来获取这些值。此外，可以通过新的`ClasspathResourceSource`将类路径资源作为`TestIdentifier`的`TestSource`。
* 现在，`DiscoverySelectors`中的`selectMethod(String)`方法支持*完全限定的方法名*作为参数的选择方法（例如`"org.example.TestClass＃testMethod(org.junit.jupiter.api.TestInfo)"`）。
* 现在，`ConsoleLauncher`和`JUnit Platform Gradle`插件使用的`TestExecutionSummary`除了包含测试事件外，还包含所有容器事件的统计信息。
* 现在，`TestExecutionSummary`可以用来获取所有失败的测试用例列表。
* 现在，[`ConsoleLauncher`](https://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)、Gradle插件和[`JUnitPlatform`](https://junit.org/junit5/docs/current/api/org/junit/platform/runner/JUnitPlatform.html) 运行器使用`^.* Tests？$`正则表达式作为测试运行中所包含的类名的默认匹配模式。
* Gradle插件现在允许明确地选择应该执行哪些测试（请参阅 [配置选择器](#配置选择器)）。
* 新的`@Testable`注解可以用来向IDE和开发工具供应商传递信息，用以说明被此注解所标注的元素是可测试的（即，它可以在JUnit Platform上作为测试被执行）。
* 现在，在使用`ClassNameFilter.includeClassNamePatterns`时，可以传递多个正则表达式，它们之间通过`"或"`语义组合。
* 现在，`@IncludeClassNamePatterns`注解可以将多个正则表达式以`"或"`的语义组合，并传递到`JUnitPlatform`运行器中。
* 现在，多个正则表达式可以通过`"或"`的语义，进行组合，并被传递给JUnit Platform Gradle 插件（参阅 [配置过滤器](#配置过滤器)）以及`ConsoleLauncher`（请参阅 [Options](#431-options)）。
* 现在，可以通过使用`PackageNameFilter.includePackageNames`来指定被包含的包名或使用`PackageNameFilter.excludePackageNames`来指定被排除的包名。
* 现在，`JUnitPlatform`运行器可以结合`@IncludePackages`注解指定希望包含的包名，也可以结合`@ExcludePackages`来指定希望排除在外的包名。
* 现在，对于使用JUnit Platform Gradle插件和`ConsoleLauncher`的用户，可应通过配置过滤器完成包名的添加与排除。
* 包名现在可以通过JUnit Platform Gradle插件（请参阅 [配置过滤器](#配置过滤器)）和[`ConsoleLauncher`](https://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)（请参阅 [Options](#431-options)）的过滤器配置来包含或排除。
* 现在，`junit-platform-console`不再强依赖于[JOptSimple](https://pholser.github.io/jopt-simple/)。因此，现在可以测试那些使用该库的不同版本的自定义代码。
* 现在，包选择器的解析会扫描JAR文件。
* Surefire provider现在支持forking。
* Surefire provider现在支持使用标记过滤，可以通过以下方法：
	* 包含: `groups/includeTags`
	* 排除：`excludedGroups/excludeTags`
* Surefire Provider现在获得了Apache License v2.0许可。


#### JUnit Jupiter

###### Bug修复

* 现在，`@AfterEach`标注的方法在类的层级结构中，将以*自下而上*的语义顺序执行。
* 现在，`DynamicTest.stream()`会为*测试执行器*接收一个`ThrowingConsumer`，而非以往的`Consumer`，从而允许定制动态测试流可能会抛出的检查异常。
* 现在，当为对应的测试方法调用`@BeforeEach`和`@AfterEach`方法时，该测试方法级的扩展注册会被使用。
* 现在，`JupiterTestEngine`支持通过接受数组或基本类型参数的唯一标识（方法的ID）来选择测试方法。
* 现在，`ExtensionContext.Store`是线程安全的。

###### 弃用和彻底改变

* `Executable`函数式接口已经被搬移到了新的专用包`org.junit.jupiter.api.function`中。
* `Assertions.expectThrows()` 已经被弃用，并以`Assertions.assertThrows()`取而代之。

###### 新特性与改进

* 支持`Assertions`中的lambda表达式的延迟和抢占超时。请参阅 [`AssertionsDemo`](#34-断言) 中的示例，更多详细信息请参阅 [org.junit.jupiter.Assertions](https://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html) JavaDoc。
* 新的`assertIterableEquals()`断言会校验两个可迭代对象是否深度一致（查阅Java文档获取细节）。
* `Assertions.assertAll()`的新变体接收可执行流（即，`Stream<Executable>`）。
* 现在，`Assertions.assertThrows()`方法将返回抛出的异常。
* 现在，`@BeforeAll`与`@AfterAll`可以被声明在接口中的静态方法上。
* JUnit Jupiter现在支持JUnit 4中的`org.junit.rules.ExternalResource`、`org.junit.rules.Verifier`、`org.junit.rules.ExpectedException`规则及其子类，这更有利于JUnit 4代码库的迁移。

#### JUnit Vintage
*自5.0.0-M2以来没有变化。*
