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

**范围**：


### 5.0.0-M3
**发布时间**： 2016.11.30

**范围**：

### 5.0.0-M2
**发布时间**： 2016.07.23

**范围**：

### 5.0.0-M1
**发布时间**： 2016.07.07

**范围**：


### 5.0.0-ALPHA
**发布时间**： 2016.02.01

**范围**：


