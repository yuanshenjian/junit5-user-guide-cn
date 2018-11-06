### 5.0.0-M4
**发布时间**： 2017.04.01

**范围**：JUnit 5的第4个里程碑发布，主要关注点在于测试模板、重复测试和参数化测试。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0 M4](https://github.com/junit-team/junit5/milestone/7?closed=1) 里程碑页面。

#### JUnit Platform

##### Bug修复
* 为了保证可重复的构建，JUnit Platform的Gradle插件为其依赖增加了一个修复版本（和插件版本相同），替换了默认的动态版本方案（即`1.+`）。
* JUnit平台的Gradle插件如今明确地应用在`java`插件的内置Gradle中，因为它对应用程序有一个隐含的依赖关系。
* `ReflectionUtils`中的所有`findMethods()`实现不再返回合成方法。结果中不再包含隐含的 [override-equal](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.2) 方法。
* 在`TestIdentifier`和`TestDescriptor`中引入`getLegacyReportingName()`。这使得JUnit Platform的Gradle插件和Surefire Provider能够通过`getLegacyReportingName()`解析类和方法名，而不再使用资源位置。
* 在路径中包含空格（％20）的JAR文件现在在用于类路径扫描例程之前已被正确解码。
* 更新了`ReflectionUtils`中的`findNestedClasses()`方法，以便搜索嵌套类也返回继承的嵌套类（无论它们是否为静态）。

##### 弃用和彻底改变
* 所有在`org.junit.platform.runner`包中的测试套件注解都移到了新`junit-platform-suite-api`模块中的`org.junit.platform.suite.api`包。其中包括例如`@SelectClasses`,`@SelectPackages`等注解。
* 移除了`DiscoverySelectors`中基于名字的发现选择器`selectNames()`，推荐使用方法`selectPackage(String)`,`selectClass(String)`和`selectMethod(String)`。
* `ClassNameFilter.includeClassNamePattern`被移除了，取而代之的请使用`ClassNameFilter.includeClassNamePatterns`。
* `@IncludeClassNamePattern`被移除了，请改用`@IncludeClassNamePatterns`。
* `ConsoleLauncher`的选项`--hide-details`被弃用了，请改用`--details none`
* ZIP发布包包含的控制台启动器不再提供，取代它的是一个独立的可执行的JAR发行版。更多细节见下面的"New Feature"部分。
* `ReflectionUtils`中的枚举类`MethodSortOrder`被重命名为`HierarchyTraversalMode`，产生的影响是应该使用`ReflectionSupport`来代替`ReflectionUtils`。
* `Launcher`中的方法`execute(LauncherDiscoveryRequest launcherDiscoveryRequest)`已经被弃用，并将在里程碑版本M5中删除。取代它的是新方法`execute(LauncherDiscoveryRequest launcherDiscoveryRequest, TestExecutionListener…​ listeners)`，除了已经注册的监听器之外，该方法还可以用于注册已提供的`TestExecutionListener`，但也仅限于已提供的`LauncherDiscoveryRequest `。

##### 新特性与改进
* 自定义的`TestExecutionListener`实现现在可以通过Java的`ServiceLoader`机制自动注册。
* `TestEngine `API添加了新的默认方法`getGroupId()`,`getArtifactId()`和`getVersion()`，这些方法用来调试和报告。默认情况下，包属性（一般来自JAR属性清单）用来决定发行ID和版本号；而组ID默认是空的。更多细节请参阅Javadoc的 [TestEngine](https://junit.org/junit5/docs/5.0.2/api/org/junit/platform/engine/TestEngine.html) 文档。
* 已发现的测试引擎的记录信息得到增强，包括group ID、artifact ID和每个测试引擎的版本，如果它们能够通过`getGroupId()`，`getArtifactId()`，以及`getVersion()`方法获取。
* 现在，`ConsoleLauncher` 的`--scan-classpath`允许扫描JAR文件作为显式参数提供的测试（请参阅 [Options](#431-options)）。
* 现在，`ConsoleLauncher`中新的`--details<Details>`选项允许在执行测试时选择一个输出细节的模式。可以使用`none`、`flat`、`tree`，或者`verbose`中的一个来指定该模式。如果没有指定，则只会输出汇总和测试失败信息（请参阅 [Options](#431-options)）。
* 可执行的`junit-platform-console-standalone-1.0.2.jar`包在默认的构建过程中生成，并存储于已在 [Maven 中心库](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone) 发布的 `junit-platform-console-standalone/build/libs`路径下。该jar包包含了Jupiter和Vintage测试引擎及其所有依赖。它在手动管理其依赖项的项目中提供了JUnit 5的无障碍使用，类似于JUnit 4中广为人知的 [plain-old JAR](https://github.com/junit-team/junit4/wiki/Download-and-Install#plain-old-jar) 管理。
* `ConsoleLauncher`中新增的`--exclude-classname (--N)`选项可以接受一个正则表达式来排除那些全类名匹配的类。当这个选项重复时，所有的模式将使用 "或" 语义进行组合。
* 新的JUnit Platform支持包`org.junit.platform.commons.support`包含了许多正在维护的工具方法，这些方法可用于注解、反射，以及类路径扫描任务。我们鼓励`TestEngine`和`Extension`开发人员（author）使用这些支持方法，以便与JUnit Platform的行为保持一致。
* 压缩了`TestDescriptor.isTest()`和`TestDescriptor.isContainer()`的内部逻辑，使得其返回值从两个独立的布尔属性值转变为名为`TestDescriptor.Type`的枚举类。详细信息请参阅下一小节。
* 引入了`TestDescriptor.Type`枚举类以及对应的方法`TestDescriptor.getType()`，用于定义所有可能的描述符类型。原来的`TestDescriptor.isTest（）`和`TestDescriptor.isContainer（）`现在委托给`TestDescriptor.Type`常量。
* 引入了`TestDescriptor.prune()`和`TestDescriptor.pruneTree()`方法，它们允许引擎作者自定义JUnit平台触发修剪时的处理办法。
* `TestIdentifier`现在使用新的`TestDescriptor.Type`枚举来存储底层类型。它可以通过新的`TestIdentifier.getType()`方法获取。此外，`TestIdentifier.isTest()`和`TestIdentifier.isContainer()`现在委托`TestDescriptor.Type`的常量。

#### JUnit Jupiter

###### Bug修复

* 修复了通过方法选择器选择时阻止在相同类中发现多个方法的错误。
* 当在超类中声明时，现在会检测到`@Nested`的非静态测试类。
* 现在，当在一个类层次结构中的多个级别声明时，重写的`@BeforeEach`和`@AfterEach`方法会以正确执行顺序会被强制执行，其顺序始终是`super.before`->`this.before`->`this.test`->`this.after`->`super.after`，即使编译器添加了合成（synthetic）方法。
* 现在，`TestExecutionExceptionHandlers`会被按照其注册顺序的反序执行，这类似于其他所有的'"after"`扩展。

###### 弃用和彻底改变

* 删除已弃用的`Assertions.expectThrows()`方法，`Assertions.assertThrows()`取代。
* 由相同部分组成但顺序不同的`ExtensionContext.Namespaces`不再被认为是同一个。


###### 新特性与改进

* 新增了`@ParameterizedTest `注解，从而为*参数化测试* 提供了极好的支持。请参阅 [参数化测试](#313-参数化测试)。
* 新增了`@RepeatedTest`注解和`RepetitionInfo`API,从而为*重复测试*提供了极好的支持。请参阅 [重复测试](#312-重复测试)。
* 引入了新的注解`@TestTemplate`，并附带了对应的扩展点`TestTemplateInvocationContextProvider`。
* 现在，`Assertions.assertThrows()`方法在生成断言失败的消息时，会采用规范名称作为异常类型。
* 现在，在测试方法上注册的`TestInstancePostProcessors`会被调用。
* `Asserrions.fail`现在有了新的变体：`Assertions.fail(Throwable cause)`和`Assertions.fail(String message, Throwable cause)`。
* 新的`Assertions.assertLinesMatch()`方法会比较字符串列表，应用了`Object :: equals`和正则表达式。 `assertLinesMatch()`还提供了一个快速转发机制，可以跳过每个调用中预期会更改的行，例如持续时间，时间戳，堆栈跟踪等。详细信息请参阅 [org.junit.jupiter.Assertions](https://junit.org/junit5/docs/5.0.2/api/org/junit/jupiter/api/Assertions.html) 的JavaDoc。
* 现在可以通过Java的`ServiceLoader`机制自动注册扩展。详情请参阅 [自动扩展注册](#522-自动扩展注册)。

#### JUnit Vintage

###### Bug修复

* 修复了导致只有最后一次测试失败的报告。例如，使用`ErrorCollector`规则时，只报告最后一次失败的检查。 现在，使用`org.opentest4j.MultipleFailuresError`会报告所有失败。
