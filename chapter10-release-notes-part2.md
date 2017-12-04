## 10. 发布记录（Part2）

### 5.0.0-M6
**发布时间**： 2017.07.18

**范围**：JUnit 5的第六次发布，主要解决Java 9的兼容性、验证（例如：tag语法规则）和修复bug。

 > ⚠️ 这是一次里程碑式的发布，并且包含了一些突破性变化。如果想在Intellij IDEA中使用更早版本的发布，请参考以上 [说明](http://junit.org/junit5/docs/current/user-guide/#running-tests-ide-intellij-idea)。
 
 若要查看这个发布全部的已解决问题和pull request，可以查看GitHub上的 [5.0 M6](https://github.com/junit-team/junit5/milestone/11?closed=1) JUnit仓库。
 
#### 兼容Java 9
JUnit 5的运行时环境的主要目标是Java 8，因此，JUnit 5的发布版本不能描述Java 9的编译模块。然而，由于 [第5次发布](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m5) 的每个发布包在其JAR清单中声明了稳定的`Automatic-Module-Name`，这使得可以在测试模块中签名包含著名的JUnit模块名称，如下所示：

```java
module foo.bar {
  requires org.junit.jupiter.api;
}
```

通常在类路径上就可以运行测试，这方面Java 8和Java 9没什么区别。只要支持JUnit平台，大多数的命令行工具和IDE都可以*开箱即用*JUnit 5。如果开发工具不支持JUnit平台，可以使用`ConsoleLauncher`或者甚至执行`junit-platform-console-standalone`，它们都在同一个jar包中。

要在模块路径中运行JUnit Jupiter测试，可以通过一个Java 9兼容的构建工具 [pro](https://github.com/forax/pro) 来实现。

**pro**能够支持黑盒测试和白盒测试。前者用来测试模块表面，只对输出应用模块有权限。后者使用合并的模块描述技术，对`protedted`，私有的包和非输出的包都有权限。

测试模块的例子可以查看 [pro的GitHub仓库](https://github.com/forax/pro/tree/master/src/test/java)：`integration.pro`是一个黑盒测试模块；但是`com.github.forax.pro.api`和`com.github.forax.pro.helper`是白盒测试模块。

#### JUnit Platform
##### 修复Bug
* 为了删除前置的和后置的空白，所有的标签都被*修剪*了。可以应用于以下几个地方，任何直接通过`TagFilter.includeTags()`和`TagFilter.excludeTags()`或者间接通过`@IncludeTags`,`@ExcludeTags`应用的`Launcher`，JUnit平台的控制台启动器，JUnit平台的Gradle插件和JUnit平台的Maven Surefire provider。

##### 弃用和突破性改变
* 所有的标签现在都要满足以下语法规则：
	* 标签不能是`null`或者*空白*
	* *修剪*的标签不能包含空白
	* *修剪*的标签不能包含ISO控制字符
* 如果一个被应用的标签语法上无效，`TagFilter.includeTags()`,`TagFilter.excludeTags()`和`TestTag.create()`工厂方法现在会抛出一个`PreconditionViolationException`异常。
* `EngineDiscoveryRequest`的方法`getDiscoveryFiltersByType`已经被改名为`getFiltersByType`。
* `UniqueId`的方法`getSegments()`现在返回一个不变的列表。
* `AbstractTestDescriptor`的方法`setSource`被删除，且替代它的是一个包含`source`变量的构造函数。

##### 新特性和改进
* 为语法有效的标签增加了一个新的检查方法`TestTag.isValid(String)`
* 如果应用的元素是类，`AnnotationSupport`的方法`findAnnotation()`现在可以被一个类实现的接口中搜索
* `org.junit.platform.commons.util.ReflectionUtils`的下列方法现在通过`org.junit.platform.commons.support.ReflectionSupport`向外暴露：
	* `public static Optional<Class<?>> loadClass(String name)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, String parameterTypeNames)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, Class<?>…​ parameterTypes)`
	* `public static <T> T newInstance(Class<T> clazz, Object…​ args)`
	* `public static Object invokeMethod(Method method, Object target, Object…​ args)`
	* `public static List<Class<?>> findNestedClasses(Class<?> clazz, Predicate<Class<?>> predicate)`

#### JUnit Jupiter
##### 修复Bug
* 为了去掉前置和后置的空白，所有通过`@Tag`申明的标签都被修剪了。
* 参数化测试中所有的原始数组类型（从`boolean[]`到`short[]`）现在都被作为返回类型为静态的方法支持。

##### 弃用和突破性改变
* `@Test`和有生命周期的方法现在都强制返回`void`类型。

##### 新特性和改进
* `断言`中所有的`fail(…​)
* `方法现在都可以用来实现single的lambda表达式，因此避免了用一个明确的返回类型来实现代码块。
* `ExtensionContext`中的新方法`getRoot()`能够容易的获得最高的，*root*扩展内容的权限。
* `ExtensionContext` API中的新方法`getRequiredTestClass()`,`getRequiredTestInstance()`和`getRequiredTestMethod()`，更加便捷，可用于在需要对应元素的用例中获取测试类、实例以及测试方法。
* 类似`@TestInstance`和`@Disabled`的类级注解现在可以被声明与测试接口上（如测试特性一样）。
* 如果针对单个方法解析了多个`TestDescriptor`，则现在会记录一条警告。 这有助于调试由多个竞争注释（例如`@Test`，`@RepeatedTest`，`@ParameterizedTest`，`@TestFactory`等）同时注释的方法导致的错误。
* 包含无效标记语法的`@Tag`声明现在将被记录为警告，而不是被有效忽略。

#### JUnit Vintage
##### 修复Bug

* 与`@Unroll`一起使用时，添加了对Runners的支持，这些Runners报告不属于`Description`树的测试事件，例如`Spock`的`Sputnik`。 以前，这样的测试根本没有报告; 现在它们被作为动态测试而报告。


### 5.0.0-M5
**发布时间**： 2017.07.04

**范围**：JUnit 5的第5个里程碑版本重点关注三点：动态容器，测试实例生命周期管理和少数API更改。

> ⚠️ 这是一次里程碑式的发布，包含重大更改。请在使用本版本替换IntelliJ IDEA中的旧版本`JUnit`之前，参阅[说明](http://junit.org/junit5/docs/current/user-guide/#running-tests-ide-intellij-idea)。

有关此版本的所有已关闭问题和请求的完整列表，请参阅GitHub上的JUnit存储库中的[5.0 M5](https://github.com/junit-team/junit5/milestone/8?closed=1)里程碑页面。

* 所有已发布的JAR构件现在都包含一个`Automatic-Module-Name`清单属性，当该JAR文件被放置在 Java 9 的模块路径上时，该属性的值就通过该文件被用来作为定义的自动模块的名称。

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
##### 修复Bug

* 如果选择器是通过不具有显式参数类型的`DiscoverySelectors`创建的，则`MethodSelector.getMethodParameterTypes()`不会为参数类型返回`null`。 具体来说，如果参数类型没有被提供并且不能推导，则返回一个空字符串。同样，如果参数类型没有明确提供，但可以推导（例如，通过提供的`Method`引用），`getMethodParameterTypes()`现在返回一个包含推导的参数类型的字符串。
* `ConsoleLauncher` 现在打印对应的可抛出异常的`toString()`方法的内容作为回退机制，而非在`Details.TREE`模式下抛出`NullPointerException`。
* 现在，`DefaultLauncher`会捕获并警告某个引擎在其发现和执行阶段生成的异常。其他引擎可被正常处理。
* 当生成字符串形式的唯一ID时，`UniqueId.Segment`类型和值的字符串现在会部分进行URL编码。通过`UniqueIdFormat`所保留的字符（例如[，：，]和/）会被编码。默认解析器也已经更新，从而可以解码这样的编码段。

##### 弃用和突破性改变
* `ConsoleLauncher`中弃用的`--hide-details`选项已被移除；使用`--details none`替代。
* 下列先前启用的方法现在都已被移除。
	* `junit-platform-engine` : `Node.execute(EngineExecutionContext)`
	* `junit-platform-commons` :  `ReflectionUtils.findAllClassesInClasspathRoot(Path, Predicate, Predicate)`
* `org.junit.platform.engine.support.hierarchical.Node`接口的 `isLeaf()`方法已被移除。
* `TestDescriptor`类中的默认方法`pruneTree()` 和 `hasTests()`已被移除。

#####新特性与改进
* 当使用`DiscoverySelectors`通过完全限定的方法名称来选择一个方法，或者通过提供`methodParameterTypes`作为一个逗号分隔的字符串时，可以使用源代码语法来描述数组参数类型，如基本数组`int[]`和`java.lang.String[]`作为一个对象数组。 此外，现在可以使用JVM的内部字符串表示来描述多维数组类型（例如，`[[[I`代表`int[][][]`，`[[Ljava.lang.String;`代表 `java.lang.String[][]`等）或*源代码语法*（例如，`boolean[][][]`，`java.lang.Double[][]`等）。
* `JUnit`的平台的`Gradle`插件任务`junitPlatformTest`现在可以在构建的配置阶段直接访问。
* `JUnit Platform Gradle`插件与`Gradle Kotlin DSL`配合使用,效果更佳。
* 当通过Java的`ServiceLoader`机制发现`TestEngine`时，将尝试确定引擎类的加载位置，如果位置URL可用，则将在配置级别进行记录。
* `mayRegisterTests()`方法可用于标记一个`TestDescriptor`将在执行过程中注册动态测试。
* 可以查询`TestPlan`是否包含`Test()`。 这被`Surefire`提供者用来决定一个类是否是一个应该被执行的测试类。
* `ENGINE`枚举常量已从`TestDescriptor.Type`中移除。 `EngineDescriptor`的默认类型现在是`TestDescriptor.Type.CONTAINER`。

#### JUnit Jupiter
##### 修复Bug

* `@ParameterizedTest`参数不再为生命周期方法和测试类构造函数解析，以提高与具有不同参数列表的常规和参数化测试方法的互操作性。

##### 弃用和突破性改变

* 迁移支持模块现在名为`junit-jupiter-migrationsupport`，值得注意的是，在名称中，`migration`与`support`之间没有`-`。
* 为了确保JUnit Jupiter中所有受支持的扩展API的可组合性，现有API中的几个方法已被重命名。
	* 查阅 [扩展API迁移文档](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m5-migration-extension-api)获取细节信息。
* `junit-jupiter-params`模块的`ArgumentsProvider API`中的`arguments()`方法已被重命名为`provideArguments()`。
* `junit-jupiter-params`模块中的`ObjectArrayArguments`类已被删除; 通过`Arguments.of（...）`静态工厂方法现在可以创建`Arguments`实例的功能。
* `@MethodSource`的名称属性已被重命名为值。
* `estExtensionContext API`中的`getTestInstance()`方法已被移至`ExtensionContext API`。此外，签名已从`Object getTestInstance()`更改为`Optional <Object> getTestInstance()`。
* `TestExtensionContext API`中的`getTestException()`方法已移至`ExtensionContext API`并重命名为`getExecutionException()`。
* `TestExtensionContext`和`ContainerExtensionContext`接口已被删除，并且所有扩展接口已被更改为使用`ExtensionContext`。
* `TestExecutionCondition`和`ContainerExecutionCondition`已被一个用于条件测试执行的通用扩展API取代：`ExecutionCondition`。

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

#####新特性与改进
* 现在，通过使用新的类级注解`@TestInstance`，可以使测试实例的生命周期从默认的方法模式转换到类模式。 这使得在测试类中的测试方法之间以及测试类中的非静态`@BeforeAll`和`@AfterAll`方法之间可以共享测试实例状态。
	* 通过查阅[测试实例的生命周期](http://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle)获取更多细节。
* 如果测试类用`@TestInstance`（Lifecycle.PER_CLASS）进行注释，则`@BeforeAll`和`@AfterAll`方法不再是静态的。 同时启用以下新特性：
	* 在`@Nested`测试类中声明`@BeforeAll`和`@AfterAll`方法。
	* 在接口默认方法上声明`@BeforeAll`和`@AfterAll`。
	* 用Kotlin编程语言实现的测试类中的`@BeforeAll`和`@AfterAll`方法的简化声明。
* 现在，`Assertions.assertAll()`将跟踪几乎所有类型的异常（这是与`AssertionError()相反的`），除非某个异常已被加入黑名单。在黑名单中的异常将会被立即抛出。
* 如果`@ParameterizedTest`接受一个数组作为参数，那么当为参数化测试的调用生成显示名称时，数组的字符串表示形式更具可读性。
* `@EnumSource`现在提供了一个枚举常量选择模式，用于控制如何解释提供的名称。支持的模式包括：`INCLUDE`和`EXCLUDE`，以及基于正则表达式的模式匹配，如`MATCH_ALL`模式和`MATCH_ANY`模式。
* 扩展现在可以通过使用新引入的引擎级`ExtensionContext`的`Store`来在顶级测试类中共享状态。
* 现在，提供使用`@MethodSource`引用的参数的参数可以直接返回`DoubleStream`，`IntStream`和`LongStream`的实例。
* `@TestFactory`现在支持任意嵌套的动态容器。详情请参阅`DynamicContainer`和抽象基类`DynamicNode`。
* 现在，`ExtensionContext.getExecutionException()`向`AfterAllCallbacks`提供`@BeforeAll`方法或`BeforeAllCallbacks`中抛出的异常。

#### JUnit Vintage
##### 修复Bug

* `VintageTestEngine`不再过滤掉声明为静态成员类的测试类，因为它们是有效的JUnit 4测试类。
* `VintageTestEngine`不再尝试执行抽象类作为测试类。相反，现在会有一个警告记录，说明这些类被排除在外。


### 5.0.0-M4
**发布时间**： 2017.04.01

**范围**：JUnit 5的第4个里程碑发布主要关注点在于测试模板、反复测试和参数化测试。

若要查看所有已解决的问题和pull request，可以查看Github上的 [5.0 M4](https://github.com/junit-team/junit5/milestone/7?closed=1) JUnit仓库。

#### JUnit Platform
##### 修复Bug
* 为了保证可重复的构建，JUnit平台的Gradle为其依赖增加了一个修复版本（和插件版本相同），替换了默认的动态版本方案（即`1.+`）。
* JUnit平台的Gradle插件如今明确的应用在`java`插件的内置Gradle中，因为它对应用程序有一个隐含的依赖关系。
* `ReflectionUtils`中`findMethods()`的实现不再返回合成方法。结果中不再包含隐含的 [override-equal](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.2)方法。
* 在`TestIdentifier`和`TestDescriptor`中引入`getLegacyReportingName()`。这使得JUnit平台的Gradle插件和Surefire Provider能够通过`getLegacyReportingName()`解析类和方法名，而不再使用资源位置。
* 路径中仍有空间的JAR文件（`20%`）在被扫描类路径之前要被解码。
* 更新了`ReflectionUtils`中的方法`findNestedClasses()`，因此搜索嵌套类也会返回继承的嵌套类（不管他们是不是静态的）。

##### 弃用和突破性改变
* 所有在`org.junit.platform.runner`包中的测试套件注释都移动到了新`junit-platform-suite-api`模块中的`org.junit.platform.suite.api`包。其中包括例如`@SelectClasses`,`@SelectPackages`等标签。
* 移除了`DiscoverySelectors`中基于名字的`selectNames()`搜索，推荐使用方法`selectPackage(String)`,`selectClass(String)`和`selectMethod(String)`。
* `ClassNameFilter.includeClassNamePattern`被移除了，取而代之的请使用`ClassNameFilter.includeClassNamePatterns`。
* `@IncludeClassNamePattern`被移除了，取而代之的请使用`@IncludeClassNamePatterns`。
* `ConsoleLauncher`的选项`--hide-details`被弃用了，取而代之的请使用`--details none`
* ZIP分发包含的控制台启动器不再提供，取代其的是一个独立的可执行的JAR发行版。更多细节见下面的"New Feature"部分。
* `ReflectionUtils`中的枚举类`MethodSortOrder`被重命名为`HierarchyTraversalMode`,产生的影响是应该使用`ReflectionSupport`来代替`ReflectionUtils`。
* `Launcher`中的方法`execute(LauncherDiscoveryRequest launcherDiscoveryRequest)`已经被弃用，且在M5发布将会被移除。Instead use the following new method that registers supplied TestExecutionListeners in addition to already registered listeners but only for the supplied LauncherDiscoveryRequest: execute(LauncherDiscoveryRequest launcherDiscoveryRequest, TestExecutionListener…​ listeners)

##### 新特性和改进
* 自定义的`TestExecutionListener`实现现在可以通过Java的`ServiceLoader`机制自动注册。
* `TestEngine`API 添加了默认的方法`getGroupId()`,`getArtifactId()`和`getVersion()`，这新方法用来调试和报告。默认情况下，包属性（一般来自JAR属性清单）用来决定版本号和发行ID，然而，默认时组ID是空的。更多细节查看Javadoc的 [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) 文档。
* 对于已有测试引擎的日志信息都进行了增强，若测试引擎可用，则可以通过`getGroupId()`，`getArtifactId()`，以及`getVersion()`等方法获取分组 ID， 工件ID，以及版本ID。
* 现在，`ConsoleLauncher` 的`--scan-classpath`选项允许JAR文件作为显式参数被提供给测试（详情查看[Options](http://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher-options)）。
* 现在，`ConsoleLauncher`中新的`--details<Details>`选项允许在执行测试时选择细节输出的模式。可以使用`none`、`flat`、`tree`，或者`verbose`中的一个关键字来指定该模式。如果没有指定，则只会输出汇总和测试失败信息（详情查看[Options](http://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher-options)）。
* 可执行的`junit-platform-console-standalone-1.0.2.jar` 包在默认的构建过程中生成，并存储于已在**Maven Central**中发布的 `junit-platform-console-standalone/build/libs`路径下。该 jar 包包含了 Jupiter 和 Vintage 测试引擎及其所有依赖。它为以往使用手工管理依赖的 JUnit 4 老式jar包的项目提供了与之类似的 JUnit 5 方式，使得用户无需投入精力在用法的版本差异上。
* `ConsoleLauncher` 中新增的`--exclude-classname (--N)`选项可以通过传入类的全限定名称的正则表达式来排除匹配到的对应类。当重复使用该选项时，匹配模式间的逻辑关系将通过“或”语义进行组合。
* 新的 JUnit 平台的辅助包`org.junit.platform.commons.support` 包含了许多以为维护的工具方法，这些方法可用于注解、反射，以及类路径扫描任务。官方鼓励测试引擎和扩展作者使用这些辅助方法，因为这样更有利于JUnit平台的行为保持一致。
* 压缩了`TestDescriptor.isTest()`和`TestDescriptor.isContainer()`的内部逻辑，使得其返回从两个独立的布尔属性值转变为名为`TestDescriptor.Type`的枚举类.
* 引入了`TestDescriptor.Type`枚举类以及对应的方法`TestDescriptor.getType()`，用于定义所有可能的描述符类型。原来的`TestDescriptor.isTest（）`和`TestDescriptor.isContainer（）`现在委托给`TestDescriptor.Type`常量。
* 引入了 `TestDescriptor.prune() ` 和`TestDescriptor.pruneTree() ` 方法用于引擎作者自定义JUnit平台触发修剪时的处理办法。
* `TestIdentifier`现在使用新的`TestDescriptor.Type`枚举来存储底层类型。它可以通过新的`TestIdentifier.getType()`方法获取。此外，`TestIdentifier.isTest()`和`TestIdentifier.isContainer()`现在委托`TestDescriptor.Type`的常量。

#### JUnit Jupiter
##### 修复Bug

* 修复了使用方法选择器在同一个类中进行选择时，阻止发现多个方法的错误。
* 现在，非静态测试类将被检测，当它的父类被`@Nested`注解标注后。
* 重写了`@BeforeEach`和`@AfterEach`方法的正确执行顺序。现在，当在一个测试中有多个级别声明时，即使编译器添加了（synthetic）方法，其顺序始终是`super.before`->`this.before`->`this.test`->`this.after`->`super.after`.
* 现在，`TestExecutionExceptionHandlers`会被按照其被注册顺序的反序执行，这类似于其他所有的`all`扩展。

##### 弃用和突破性改变

* 删除已弃用的Assertions.expectThrows()方法，用Assertions.assertThrows()取代。
* 组成`ExtensionContext.Namespaces`命名空间的每部分都相同，但顺序不同，现在会被认为是不同的。

##### 新特性和改进

* 新增了`@ParameterizedTest `注解，从而为*参数化测试*提供了极好的支持。
* 新增了`@RepeatedTest`注解和`RepetitionInfo`API,从而为*重复测试*提供了极好的支持。
* 引入了新的注解`@TestTemplate`，并附带了对应的扩展点`TestTemplateInvocationContextProvider`。
* 现在，`Assertions.assertThrows()`方法在生成断言失败的消息时，会采用规范名称作为异常类型。
* 在测试方法上注册的`TestInstancePostProcessors`现在会被调用。
* `Asserrions.fail`现在有了新的变体：`Assertions.fail(Throwable cause)`和`Assertions.fail(String message, Throwable cause)`。
* 新的`Assertions.assertLinesMatch()`方法会比较字符串列表，应用了`Object :: equals`和正则表达式检查。 `assertLinesMatch()`还提供了一个快速转发机制，可以跳过每个调用中预期会更改的行，例如持续时间，时间戳，堆栈跟踪等。有关详细信息，请参阅 [org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html)的文档。
* 现在可以通过 Java 的`ServiceLoader`机制自动注册扩展。详情参阅[自动扩展注册](http://junit.org/junit5/docs/current/user-guide/#extensions-registration-automatic)。

#### JUnit Vintage
##### 修复Bug

* 修复了导致只有最后一次测试失败的报告。 例如，使用`ErrorCollector`规则时，只报告最后一次失败的检查。 现在，使用`org.opentest4j.MultipleFailuresError`报告所有失败。


### 5.0.0-M3
**发布时间**： 2016.11.30

**范围**：JUnit 5 第三次发布针对改善 JUnit 4 的互操作性，新增的发现选择器以及文档。

对于本次发布相关的 commit 信息，可以通过查看[5.0 M3](https://github.com/junit-team/junit5/milestone/6?closed=1)里程碑页面在GitHub上的JUnit代码库了解。

####JUnit Platform

#####Bug 修复

* 现在，在打印异常信息时，`ConsoleLauncher`所使用的`ColoredPrintingTestListener`输出实际异常类型以及堆栈跟踪。
* 现在，在扫描类路径的根目录时，默认包中的测试类都会被提取出来。例如，在没有显式包被选择的时候，使用Unit Platform Gradle 插件。
* 类路径扫描不再加载类名过滤器排除的类。
* 类路径扫描不再尝试加载Java 9的`module-info.class`文件。

#####弃用与突破性改变

* 重命名`ClasspathSelector`为`ClasspathRootSelector`,以避免与`ClasspathResourceSelector`混淆。
* 重命名`JavaPackageSource`为`PackageSource`,以便于`PackageSelector`的命名方式保持一致。
* 重命名`JavaMethodSource`为`MethodSource`,以便于`MethodSelector`的命名方式保持一致。
* 现在，`PackageSource`，`ClassSource`，以及`MethodSource`直接实现`TestSource`接口，而非已经移除的`JavaSource`接口。
* `DiscoverySelectors`中，基于名称的通用发现选择器（例如`selectName()`和`selectNames()`）已经被弃用，取而代之的是更加专用的`selectPackage(String)`，`selectClass(String)`,以及`selectMethod(String)`方法。
* 现在，`ClassFliter`已经被重命名为`ClassNameFilter`，并且实现了`DiscoveryFilter<String>`,而非之前的`DiscoveryFilter<Class<?>>`。因此`ClassFilter`可以在类路径扫描的过程中，于类加载之前使用。
* 现在，`ClassNameFilter.includeClassNamePattern`被弃用，并以`ClassNameFilter.includeClassNamesPatterns`取而代之。
* 现在，`@IncludeClassNamePattern`被弃用，并以`@IncludeClassNamePatterns`取而代之。
* 用于对`ConsoleLauncher`配置额外类路径实体的命令行选项`-p`已被重命名为`-cp`，以便与`java`其他可执行执行选项保持一致。另外，为`--classpath`命令行选项引入了一个新的别名`--class-path`,而原有命令行保持不变。
* 对`ConsoleLauncher`的命令行选项`-a`和`--all`已被重命名为`--scan-class-path`。
* 对`ConsoleLauncher`的短命令行选项`-C`，`-D`，以及`-r`都被弃用，取而代之的是对应的长命令行选项`--disable-ansi-colors`，`--hide-details`，以及`--reports-dir`，分别与之等价。
* `ConsoleLauncher`不在支持无选择参数的使用方式。请使用新的显式选择器选项以明确包名、类名或方法名。另外，`--scan-class-path`现在可以接受一个可选参数，该参数可被用来选择需要被扫描的类路径。涉及多条类路径时，可以使用系统自带的路径分隔符将之隔离（Windows使用`;`，Unix使用`:`）。
* `LauncherDiscoveryRequestBuilder`不再接受`selectors`，`filters`或配置参数映射的空值。
* 现在需要将Gradle插件配置中的类名称，引擎ID和标签过滤器包装在filters扩展元素中（请参阅[配置过滤器](http://junit.org/junit5/docs/current/user-guide/#running-tests-build-gradle-filters)）。

####新特性

* `DiscoverySelectors`中新增的`selectUri（...）`方法用于选择URI。 `TestEngine`可以通过查询`UriSelector`的注册实例来检索这些值。
* 在`DiscoverySelectors`中新增的`selectFile（...）`和`selectDirectory（...）`方法，用于选择文件系统中的文件和目录。 `TestEngine`可以通过查询`FileSelector`和`DirectorySelector`的已注册实例来检索这些值。
* `DiscoverySelectors`中的新的`selectClasspathResource（String）`方法，用于按名称选择类路径资源（如XML或JSON文件），其中名称是当前类路径中资源的分离路径名。 `TestEngine`可以通过查询`ClasspathResourceSelector`的已注册实例来获取这些值。 此外，可以通过新的`ClasspathResourceSource`将类路径资源作为`TestIdentifier`的`TestSource`。
* 现在，`DiscoverySelectors`中的`selectMethod（String）`方法支持完全限定的方法名作为参数的选择方法（例如“org.example.TestClass＃testMethod（org.junit.jupiter.api.TestInfo）”）.
* 现在，`ConsoleLauncher`和`JUnit Platform Gradle`插件使用的`TestExecutionSummary`除了包含测试事件外，还包含所有容器事件的统计信息。
* 现在，`TestExecutionSummary`可以用来获取所有失败的测试用例列表。
* 现在，`ConsoleLauncher`，Gradle插件和`JUnitPlatform`运行器使用`^。* Tests？$`正则匹配方式作为测试运行中包含的类名的默认匹配模式。
* Gradle插件现在允许明确地选择应该执行哪些测试（请参阅[配置选择器](http://junit.org/junit5/docs/current/user-guide/#running-tests-build-gradle-selectors)）。
* 新的`@Testable`标签可以用来向IDE和开发工具供应商传递信息，用以说明被此注解所标注的元素是可测试的（即，它可以在JUnit平台上作为测试被执行）。
* 现在，在使用`ClassNameFilter.includeClassNamePatterns`时，可以传递多个正则表达式，它们之间通过“或”语义组合。
* 现在，`@IncludeClassNamePatterns`注解可以将多个正则表达式以“或”的语义组合，并传递到`JUnitPlatform`运行器中。
* 现在，多个正则表达式可以通过“或”的语义，进行组合，并被传递给JUnit Platform Gradle 插件（参阅[Configuring Filters](http://junit.org/junit5/docs/current/user-guide/#running-tests-build-gradle-filters)）以及`ConsoleLauncher`（参阅[Options](http://junit.org/junit5/docs/current/user-guide/#running-tests-console-launcher-options)）。
* 现在，可以通过使用`PackageNameFilter.includePackageNames`来指定被包含的包名或使用`PackageNameFilter.excludePackageNames`来指定被排除的包名。
* 现在，`JUnitPlatform`运行器可以结合`@IncludePackages`注解指定希望包含的包名，也可以结合`@ExcludePackages`来指定希望排除在外的包名。
* 现在，对于使用JUnit Platform Gradle插件和`ConsoleLauncher`的用户，可应通过配置过滤器完成包名的添加与排除。
* 现在，`junit-platform-console`不在强依赖与[JOptSimple](https://pholser.github.io/jopt-simple/)。因此，现在对于一般的测试而言可以使用不同版本的库。
* 现在，包选择器会扫描JAR文件。
* Surefire供应商现在支持forking。
* Surefire供应商想在支持使用标签过滤，可以通过以下方法：
	* 包含: `groups/includeTags`
	* 排除：`excludedGroups/excludeTags`
* Surefire提供商现已获得Apache License v2.0许可。

####JUnit Jupiter
#####Bug 修复

* 现在，`@AfterEach`标注的方法在类的层级结构中，将以*自下而上*的的语义顺序执行。
* 现在，`DynamicTest.stream()`会为*测试执行器*接收一个`ThrowingConsumer`，而非以往的`Consumer`,从而允许定制动态测试流可能会抛出的检查异常。
* 现在，当为对应的测试方法调用`@BeforeEach`和`@AfterEach`方法时，该测试方法级的扩展注册会被使用。
* 现在，`JupiterTestEngine`支持通过接受数组或基本类型参数的唯一标识（方法的ID）来选择测试方法。
* 现在，`ExtensionContext.Store`是线程安全的。

#####弃用与突破性变化

* `Executable`函数式接口已经被搬移到了新的专用包`org.junit.jupiter.api.function`中。
* `Assertions.expectThrows()` 已经被弃用，并以`Assertions.assertThrows()`取而代之。

#####新特性与改进

* 支持`Assertions`中的lambda表达式的延迟和抢占超时。 请参阅[`AssertionsDemo`](http://junit.org/junit5/docs/current/user-guide/#writing-tests-assertions)中的示例，并参阅[org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html) Java文档获取更多详细信息。
* 新的`assertIterableEquals()`断言会校验两个可迭代对象是否深度一致（查阅Java文档获取细节）
* `Assertions.assertAll()`的新变体可以接收可执行流（即，`Stream<Executable>`）。
* 现在，`Assertions.assertThrows()`方法将返回抛出的异常。
* 现在，`@BeforeAll`与`@AfterAll`可以被声明在接口中的静态方法上。
* JUnit Jupiter现在支持JUnit 4 中的`org.junit.rules.ExternalResource`, `org.junit.rules.Verifier`, `org.junit.rules.ExpectedException`规则，及其相关的子类，这更有利于JUnit 4代码库的版本迁移。

####JUnit Vintage
以上一版本相比暂无变化。

### 5.0.0-M2
**发布时间**： 2016.07.23

**范围**： JUnit 5的第2个里程碑发布

#### 变更摘要
此版本主要是针对自从`5.0.0-M1`发现的错误发布的修补程序版本。

以下是一个全局改变的列表。关于更改的详细信息，包括平台、Jupiter和Vintage，请参阅下面的章节。有关此版本的所有已关闭问题和请求的完整列表，请参阅GitHub上的JUnit存储库中的[5.0 M2](https://github.com/junit-team/junit5/milestone/4?closed=1)里程碑页面。

* JUnit 5的Gradle构建可以正常在Microsoft Windows下运行。
* 在AppVeyor上建立了针对Microsoft Windows的持续集成构建。

#### JUnit Platform
##### 修复Bug
* 容器中的故障——例如，抛出异常的`@BeforeAll`方法——如今在使用`ConsoleLauncher`或JUnit平台Gradle插件时会构建失败。
* JUnit Platform Surefire Provider不再静默忽略纯动态测试类——例如，只声明`@TestFactory`方法的测试类。
* 在`junit-platform-console-<release version>` TAR和ZIP分布中包含的`junit-platform-console`和`junit-platform-console.bat`shell脚本，如今能够正确引用`ConsoleLauncher`，而不是`ConsoleRunner`。
* 被`ConsoleLauncher`和JUnit平台Gradle插件使用的`TestExecutionSummary`如今包含了失败的错误信息。
* 类路径扫描现在可以防止类加载和处理期间遇到的问题——例如，在处理格式错误的类时，潜在的异常被吞噬并记录在有问题的文件路径中，如果这个异常是一个列入黑名单的异常，比如，`OutOfMemoryError`，那么它将被重新抛出。

##### 弃用
* `DiscoverySelectors`中基于通用名称的发现选择器（也就是，`selectName()`和`selectNames()`）已经被废弃，建议使用`selectPackage(String)`,`selectClass(String)`和`selectMethod(String)`方法。

##### 新特性
* `DiscoverySelectors`中的新方法`selectMethod(String)`支持选择 *fully qualified method name*。

#### JUnit Jupiter
##### 修复Bug
* 在类级别和方法级别使用`@ExtendWith`注解声明扩展实现的方法不再能够被多次注册。

#### JUnit Vintage
从*5.0.0-M1*开始没有发生变化。


### 5.0.0-M1
**发布时间**： 2016.07.07

**范围**： JUnit 5的第1个里程碑发布

####变动总结
下面首先给出整体改动。而有关Platform、Jupiter，以及Vintage的细节变动，可以查阅后面的专项信息。对于本次发布相关的 commit 信息，可以通过查看[5.0 M1](https://github.com/junit-team/junit5/milestone/2?closed=1)里程碑页面在GitHub上的JUnit代码库了解。

* 在已发布的JAR清单中，包含了额外的元数据，例如`Create-By`，`Built-By`，`Build-Date`，`Build-Time`，`Build-Revision`，`Implementation-Title`，`Implementation-Version`，`Implementation-Vendor`等。
* 当前发布的工件中，包含了`LICENSE.md`和`META-INF`。
* 现在，JUnit参与到[Up For Grabs](http://up-for-grabs.net/#/tags/junit)运动中，以便为开源做贡献。
	* 点击这里了解更多关于[up-for-grabs](https://github.com/junit-team/junit5/labels/up-for-grabs) 的信息。
* 对于已发布的工件，其组ID,工件ID,以及版本都已改变。
	* 查看[工件迁移](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m1-migration-artifacts)以及[依赖元数据](http://junit.org/junit5/docs/current/user-guide/#dependency-metadata)。
* 所有的基础包都被重命名了。
	* 查看[包迁移](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m1-migration-packages)

*表 4. 工件迁移信息表*

|Old Group ID|Old Artifact ID|New Group ID|New Artifact ID|New Base Version|
|:---|:---|:---|:---|:---|
|org.junit|junit-commons|org.junit.platform|junit-platform-commons|1.0.0|
|org.junit|junit-console|org.junit.platform|junit-platform-console|1.0.0|
|org.junit|junit-engine-api|org.junit.platform|junit-platform-engine|1.0.0|
|org.junit|junit-gradle|org.junit.platform|junit-platform-gradle-plugin|1.0.0|
|org.junit|junit-launcher|org.junit.platform|junit-platform-launcher|1.0.0|
|org.junit|junit4-runner|org.junit.platform|junit-platform-runner|1.0.0|
|org.junit|surefire-junit5|org.junit.platform|junit-platform-surefire-provider|1.0.0|
|org.junit|junit5-api|org.junit.jupiter|junit-jupiter-api|5.0.0|
|org.junit|junit5-engine|org.junit.jupiter|junit-jupiter-engine|5.0.0|
|org.junit|junit4-engine|org.junit.vintage|junit-vintage-engine|4.12.0|

*表 5. 包迁移信息表*

|旧基包名称|新基包名称|
|:---|:---|
|org.junit.gen5.api|org.junit.jupiter.api|
|org.junit.gen5.commons|org.junit.platform.commons|
|org.junit.gen5.console|org.junit.platform.console|
|org.junit.gen5.engine.junit4|org.junit.vintage.engine|
|org.junit.gen5.engine.junit5|org.junit.jupiter.engine|
|org.junit.gen5.engine|org.junit.platform.engine|
|org.junit.gen5.gradle|org.junit.platform.gradle.plugin
|org.junit.gen5.junit4.runner|org.junit.platform.runner|
|org.junit.gen5.launcher|org.junit.platform.launcher|
|org.junit.gen5.launcher.main|org.junit.platform.launcher.core|
|org.junit.gen5.surefire|org.junit.platform.surefire.provider|

####JUnit Platform
* 重命名`ConsoleRunner`为`ConsoleLauncher`。
* 现在，`ConsoleLauncher`在退出时总会返回状态码，并且已经移除了*激活退出码*的标志位。
* `junit-platform-console`不再在`junit-platform-runner`，`junit-jupiter-engine`，或`junit-vintage-engine`上定义传递依赖。
* 现在，`JUnit5`运行器已经被重命名为`JUnitPlatform`。
	* `@Packages`已经被重命名为`@SelectPackages`。
	* `@Classes`已经被重命名为`@SelectClasses`。
	* `@UniqueIds`已经被移除。
	* 引入了新的注解`@UseTechnicalNames`。
		* 详情参阅[ Display Names vs. Technical Names.](http://junit.org/junit5/docs/current/user-guide/#running-tests-junit-platform-runner-technical-names)
* Gradle的JUnit Platform插件已经全面修订。
	* 现在，JUnit Platform的 Gradle 插件要求 Gradle 版本在2.5或更高版本。
	* Gradle中名为`junit5Test`的任务已经被重命名为`junitPlatformTest`。
	* Gradle中名为`junit5`的配置已经被重命名为`junitPlatform`。
		* `runJunit4`已经被`enableStandradTestTask`替换。
		* `version`已经被`platformVersion`替换。
	* 查阅[Gralde文档](http://junit.org/junit5/docs/current/user-guide/#running-tests-build-gradle)了解更多。
* XML测试报告已被全面修正。
	* XML报告现在包含换行。
	* 现在，在JUnit Platform中明确定义的，但是在标准XML文档属性中不包含的属性元素，会被包含在`<system-out>`元素中的`CDATA`块中。
	* 现在，测试报告中会用全限定类名和真正的方法名来取代先前的显示用类名。
	* 现在，`Testidentifier`中的唯一标识符是`String`类型。
	* 现在，`TestSource`是有着专用层级结构的接口，它由`CompositeTestSource`, `JavaSource`, `JavaPackageSource`, `JavaClassSource`, `JavaMethodSource`, `UriSource`, `FileSystemSource`, `DirectorySource`, 和`FileSource`构成。
	* 新增了`DiscoverySelectors `类，用于集中管理所有的*select*方法，同时将所有`DiscoverySelector `工厂方法转移到新增类中。
	* `Test.filter()`已经被重命名为`Filter.apply()`。
	* `TestTag.of()`已被重命名为`TestTag.create()`。
	* `TestDiscoveryRequestBuilder`已被重命名为`LauncherDiscoveryRequest`。
	* 现在，`LauncherDiscoveryRequest`是不可变的。
	* `TestDescriptor.allDescendants()`已经被重命名为`TestDescriptor.getAllDescendants()`。
	* `TestEngine#discover(EngineDiscoveryRequest)`已经被`TestEngine#discover(EngineDiscoveryRequest, UniqueId)`替换。
	* 引入了`ConfigurationParameters`，`Laucher`可以通过`EngineDiscoveryRequest`和`ExecutionRequest`将配置参数传递给引擎。
	* `Container`和`Leaf`抽象类已经从`HierarchicalTestEngine `中移除。
	* `getName()`方法已经被从`TestIdentifier `和`TestDescriptor `中移除，取而代之的是通过`TestSource`获取一个实现类的具体名称。
	* 现在，测试引擎在本质上是完全动态的。换句话说，`TestEngine`无需在*发现阶段*创建`TestDescriptor`实体；现在，`TestEngine`在执行阶段，注册容器与测试都是动态可选的。
* 所支持引擎和标签d的包含和排除关系全部被修改：
		* 引擎和标签不在是`required`，而是`included`.
		* 现在，`ConsoleLauncher`支持以下选项：`t`/`include-tag`, `T`/`exclude-tag`, `e`/`include-engine`, `E`/`exclude-engine`.
		* 现在，Gradle 插件支持嵌套在`include`和`exclude`实体中的`engines`和`tags`配置块。
		* 现在，`EngineFilter`支持`includeEngines()`和`exludeEngines()`工厂方法。
		* 现在，`JUnitPlatform `运行器支持`@IncludeTags `,`@ExcludeTags `,`@IncludeEngines `，以及`@ExcludeEngines `。

####JUnit Jupiter
* `junit5`引擎ID已经被重命名为`junit-jupiter`。
* `Junit5TestEngine`已经被重命名为`JupiterTestEngine`。
* 现在，`Assertions`提供了以下支持：
	* `assertEquals()`方法可以对基本类型使用
	* `assertEquals()`方法可以对包含增量的double类型和floats类型的值使用
	* `assertArrayEquals()`
	* 现在期望值与实际值都被提供给`AssertionFailedError`。
* [动态测试](http://junit.org/junit5/docs/current/user-guide/#writing-tests-dynamic-tests):现在，测试可以通过lambda表达式在运行时被动态注册。
* 现在，`TestInfo`通过`getTags()`方法提供了获取标签的方法。
* 现在，`@AfterEach`注解所标注的方法以及*after*回调会在被`@Test`、`@BeforeEach`注解所标注的方法或*before*回调抛出异常后被调用。
* 现在，`@AfterAll`注解所标注的方法以及*after all*回调一定会被调用。
* 现在，在测试类的层级结构中，父类或父级接口上使用的可重复注解，例如`@ExtendWith`和`@Tag`,会被发现。
* 现在，在测试类或接口的层级结构中，扩展将会被*自上而下*地注册。
* 现在，测试和容器的[执行条件可以被禁用](http://junit.org/junit5/docs/current/user-guide/#extensions-conditions-deactivation)。
* `InstancePostProcessor`已被重命名为`TestInstancePostProcessor`。
	* 现在，`ParameterResolver`API是基于`java.lang.reflect.Executable`的，因此可以被用来解析一般方法和构造器的参数。
	* 新的`ParameterContext`用来作为参数传递给`ParameterResolver`扩展的方法`supports()`和`resolve()`。
	* 现在，`ParameterResolver`扩展支持基础类型的解析。
* `ExtensionPointRegistry`和`ExtensionRegistrar`已经被移除，取而代之的是通过`@ExtendWith`注解完成的声明式注册。
* `AfterAllExtensionPoint`已经被重命名为`AfterAllCallback`。
* `AfterAllExtensionPoint`已经被重命名为`AfterAllCallback`。
* `BeforeEachExtensionPoint`已经被重命名为`BeforeEachCallback`。
* `BeforeAllExtensionPoint`已经被重命名为`BeforeAllCallback`。
* 新增了`BeforeTestExecutionCallback`与`AfterTestExecutionCallback`扩展API。
* `ExceptionHandlerExtensionPoint`已经被重命名为`TestExecutionExceptionHandler`。
* 现在，测试异常通过`TestExtensionContext`被提供给扩展。
* 现在，在`ExtensionContext`中，很多方法都支持类型安全变体。
* 现在，`ExtensionContext.getElement()`方法返回`Optional`类型。
* `Namespace.of()`已经被重命名为`Namespace.create()`。
* `TestInfo`和`ExtensionContext`新增了`getTestClass()`和`getTestMethod()`方法。
* 移除了`TestInfo`和`ExtensionContext`中的`getName()`方法，取而代之的是通过当前的测试类名或方法名获取具体的名称。

####JUnit Vintage
* `junit4`引擎ID已经被重命名为`junit-vintage`。
* `Junit4TestEngine`已经被重命名为`VintageTestEngine`。

### 5.0.0-ALPHA
**发布时间**： 2016.02.01

**范围**：JUnit 5的Alpha版本


