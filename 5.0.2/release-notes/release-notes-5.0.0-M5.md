### 5.0.0-M5
**发布时间**： 2017.07.04

**范围**：JUnit 5的第5个里程碑版本，重点关注三点：动态容器、测试实例生命周期管理以及次要的API更改。

> ⚠️ 这是一次里程碑式的发布，包含重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上的JUnit仓库中的[5.0 M5](https://github.com/junit-team/junit5/milestone/8?closed=1)里程碑页面。

* 所有已发布的JAR文件现在都包含一个`Automatic-Module-Name`清单属性，其值被用作该JAR文件放置在Java 9模块路径上时所定义的自动模块的名称。

*表 2. 自动模块名称列表*

|**JAR 文件**|**Automatic-Module-Name**|
|:---|:---|
|junit-jupiter-api-<VERSION>.jar|org.junit.jupiter.api|
|junit-jupiter-engine-<VERSION>.jar|org.junit.jupiter.engine|
|junit-jupiter-migrationsupport-<VERSION>.jar|org.junit.jupiter.migrationsupport|
|junit-jupiter-params-<VERSION>.jar|org.junit.jupiter.params|
|junit-platform-commons-<VERSION>.jar|org.junit.platform.commons|
|junit-platform-console-<VERSION>.jar|org.junit.platform.console|
|junit-platform-engine-<VERSION>.jar|org.junit.platform.engine|
|junit-platform-gradle-plugin-<VERSION>.jar|org.junit.platform.gradle.plugin|
|junit-platform-launcher-<VERSION>.jar|org.junit.platform.launcher|
|junit-platform-runner-<VERSION>.jar|org.junit.platform.runner|
|junit-platform-suite-api-<VERSION>.jar|org.junit.platform.suite.api|
|junit-platform-surefire-provider-<VERSION>.jar|org.junit.platform.surefire.provider|
|junit-vintage-engine-<VERSION>.jar|org.junit.vintage.engine|

#### JUnit Platform

###### Bug修复

* 如果选择器是通过不具有显式参数类型的`DiscoverySelectors`创建的，则`MethodSelector.getMethodParameterTypes()`不会为参数类型返回`null`。 具体来说，如果参数类型没有被提供并且不能推导，则返回一个空字符串。同样，如果参数类型没有明确提供，但可以推导（例如，通过提供的`Method`引用），`getMethodParameterTypes()`现在返回一个包含推导的参数类型的字符串。
* `ConsoleLauncher`现在打印对应异常的`toString()`方法的内容作为回退机制，而非在`Details.TREE`模式下抛出`NullPointerException`。
* 现在，`DefaultLauncher`会捕获并警告某个引擎在其发现和执行阶段生成的异常。其他引擎可被正常处理。
* 当生成字符串形式的唯一ID时，`UniqueId.Segment`类型和值的字符串现在会部分进行URL编码。通过`UniqueIdFormat`所保留的字符（例如`[`，`:`，`]`和`/`）会被编码。默认解析器也已经更新，从而可以解码这样的编码段。

###### 弃用和彻底改变
* `ConsoleLauncher`中弃用的`--hide-details`选项已被移除；使用`--details none`替代。
* 下列之前弃用的方法现在都已被移除。
	* `junit-platform-engine` : `Node.execute(EngineExecutionContext)`
	* `junit-platform-commons` :  `ReflectionUtils.findAllClassesInClasspathRoot(Path, Predicate, Predicate)`
* `org.junit.platform.engine.support.hierarchical.Node`接口的 `isLeaf()`方法已被移除。
* `TestDescriptor`类中的默认方法`pruneTree()`和`hasTests()`已被移除。

###### 新特性与改进
* 当使用`DiscoverySelectors`通过完全限定的方法名称来选择一个方法，或者通过提供`methodParameterTypes`作为一个逗号分隔的字符串时，可以使用*源代码语法* 来描述数组参数类型，如基本数组`int[]`和`java.lang.String[]`作为一个对象数组。 此外，现在可以使用JVM的内部字符串表示来描述多维数组类型（例如，`[[[I`代表`int[][][]`，`[[Ljava.lang.String;`代表 `java.lang.String[][]`等）或*源代码语法*（例如，`boolean[][][]`，`java.lang.Double[][]`等）。
* `JUnit`的平台的`Gradle`插件的`junitPlatformTest`任务现在可以在构建的配置阶段直接访问。
* `JUnit Platform Gradle`插件与`Gradle Kotlin DSL`配合使用效果更佳。
* 当通过Java的`ServiceLoader`机制发现`TestEngine`时，现在将尝试确定引擎类的加载位置，如果位置URL可用，则将在配置级别进行记录。
* `mayRegisterTests()`方法可用来表示一个`TestDescriptor`将在执行过程中注册动态测试。
* 现在可以查询`TestPlan`是否包含`Test()`。 这被`Surefire` provider用来决定一个类是否是一个应该被执行的测试类。
* `ENGINE`枚举常量已从`TestDescriptor.Type`中移除。`EngineDescriptor`的默认类型现在是`TestDescriptor.Type.CONTAINER`。


#### JUnit Jupiter

###### Bug修复

* `@ParameterizedTest`参数不再为生命周期方法和测试类构造函数解析，以提高与具有不同参数列表的常规测试方法和参数化测试方法的互操作性。

###### 弃用和彻底改变

* 迁移支持模块现在名为`junit-jupiter-migrationsupport`，值得注意的是，在名称中，`migration`与`support`之间没有`-`。
* 为了确保JUnit Jupiter中所有受支持的扩展API的可组合性，现有API中的几个方法已被重命名。
	* 详细信息请参阅 [扩展API迁移](#扩展API迁移)。
* `junit-jupiter-params`模块的`ArgumentsProvider API`中的`arguments()`方法已被重命名为`provideArguments()`。
* `junit-jupiter-params`模块中的`ObjectArrayArguments`类已被删除; 通过`Arguments.of（...）`静态工厂方法现在可以创建`Arguments`实例的功能。
* `@MethodSource`的名称属性已被重命名为值。
* `estExtensionContext API`中的`getTestInstance()`方法已被移至`ExtensionContext API`。此外，签名已从`Object getTestInstance()`更改为`Optional <Object> getTestInstance()`。
* `TestExtensionContext API`中的`getTestException()`方法已移至`ExtensionContext API`并重命名为`getExecutionException()`。
* `TestExtensionContext`和`ContainerExtensionContext`接口已被删除，并且所有扩展接口已被更改为使用`ExtensionContext`。
* `TestExecutionCondition`和`ContainerExecutionCondition`已被一个用于条件测试执行的通用扩展API取代：`ExecutionCondition`。

<a id="扩展API迁移"></a>
*表 3. 扩展API迁移对应表*

|**扩展API**|**曾用名**|**新名称/定位**|
|:---|:---|:---|
|ParameterResolver|supports()|supportsParameter()|
|ParameterResolver|resolve()|resolveParameter()|
|ContainerExecutionCondition|evaluate()|evaluateExecutionCondition() in ExecutionCondition|
|TestExecutionCondition|evaluate()|evaluateExecutionCondition() in ExecutionCondition|
|TestExtensionContext|getTestException()|getExecutionException() in ExtensionContext|
|TestExtensionContext|getTestInstance()|getTestInstance() in ExtensionContext|
|TestTemplateInvocationContextProvider|supports()|supportsTestTemplate()|
|TestTemplateInvocationContextProvider|provide()|provideTestTemplateInvocationContexts()|

###### 新特性与改进
* 现在，通过使用新的类级注解`@TestInstance`，可以使测试实例的生命周期从默认`per-method`模式转换到`per-class`模式。 这使得在测试类中的测试方法之间以及测试类中的非静态`@BeforeAll`和`@AfterAll`方法之间可以共享测试实例状态。
	* 详细信息请参阅 [测试实例生命周期](#38-测试实例生命周期)。
* 如果测试类用`@TestInstance(Lifecycle.PER_CLASS)`进行注解，则`@BeforeAll`和`@AfterAll`方法可以不再是`static`的。同时启用以下新特性：
	* 在`@Nested`测试类中声明`@BeforeAll`和`@AfterAll`方法。
	* 在接口`default`方法上声明`@BeforeAll`和`@AfterAll`。
	* 用Kotlin编程语言实现的测试类中的`@BeforeAll`和`@AfterAll`方法的简化声明。
* `Assertions.assertAll()`现在将跟踪几乎所有类型的异常（而不是只跟踪`AssertionError`类型的异常），除非异常是一个被列入*黑名单* 的异常，在这种情况下，它将被立即重新抛出。
* 如果`@ParameterizedTest`接受一个数组作为参数，那么当为参数化测试的调用生成显示名称时，数组的字符串表示形式将被转换为人类可读的格式。
* `@EnumSource`现在提供了一个枚举常量选择模式，用于控制如何解释提供的名称。支持的模式包括：`INCLUDE`和`EXCLUDE`，以及基于正则表达式的模式匹配，如`MATCH_ALL`模式和`MATCH_ANY`模式。
* 扩展现在可以通过使用新引入的引擎级`ExtensionContext`的`Store`来在顶级测试类中共享状态。
* 现在，`@MethodSource`引用的参数提供方法可以直接返回`DoubleStream`，`IntStream`和`LongStream`的实例。
* `@TestFactory`现在支持任意嵌套的动态容器。详情请参阅`DynamicContainer`和抽象基类`DynamicNode`。
* 现在，`ExtensionContext.getExecutionException()`向`AfterAllCallbacks`提供`@BeforeAll`方法或`BeforeAllCallbacks`中抛出的异常。

#### JUnit Vintage

###### Bug修复

* `VintageTestEngine`不再过滤掉声明为静态成员类的测试类，因为它们是有效的JUnit 4测试类。
* `VintageTestEngine`不再尝试将抽象类当作测试类来执行。相反，现在会有一个警告记录来说明这些类被排除在外。
