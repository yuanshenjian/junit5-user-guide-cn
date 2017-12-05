## 10. 发布记录

### 5.0.2

**发布时间**： 2017.11.12

**范围**：自5.0.1版本以来的错误修复和小的改进。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.2](https://github.com/junit-team/junit5/milestone/17?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复

- 修复后，Maven Surefire对于不使用`MethodSource`的测试引擎（例如Spek）能正确地报告失败的测试。

- 修复后，当一个非零的`forkCount`与Maven Surefire一起执行时，可以正确地报告写入`System.out`或`System.err`的测试，特别是通过一个日志框架的时候。


###### 新特性与改进

- JUnit Platform Maven Surefire提供者程序现在支持`redirectTestOutputToFile` Surefire功能。

- JUnit Platform Maven Surefire提供者程序现在会忽略通过`<includeTags/>`，`<groups/>`，`<excludeTags/>`和`<excludedGroups/>`提供的空字符串。


#### JUnit Jupiter

###### Bug修复

- `@CsvSource`或`@CsvFileSource`输入行中的尾随空格不再生成空值。

- 以前，`@EnableRuleMigrationSupport`无法识别`@Rule`方法，该方法返回一个已支持的`TestRule`类型的子类型。而且，它错误地实例化了某些多次使用方法声明的规则。现在，一旦启用，它将实例化所有声明的规则（字段*和*方法），并按照JUnit 4使用的顺序来调用它们。

> - Previously, disabled test classes were eagerly instantiated when Lifecycle.PER_CLASS was used. Now, ExecutionCondition evaluation always takes place before test class instantiation.

- 以前，当使用`Lifecycle.PER_CLASS`时，被禁用的测试类会被迫切地实例化。现在，`ExecutionCondition`总是在测试类实例化之前就被解析。

- `unit-jupiter-migrationsupport`模块不再会错误地尝试通过`ServiceLoader`机制来注册`JupiterTestEngine`，从而允许将其用作Java 9模块路径上的模块。

###### 新特性与改进

- 现在，`Assertions`类中的`assertTrue()`和`assertFalse()`的失败消息包含了关于预期和实际布尔值的详细信息。
	- 例如，调用`assertTrue(false)`生成的失败消息现在变成了`"expected:<true>but was: <false>"`，而不是空字符串。

- 如果参数化测试没有消费通过参数源提供给它的所有参数，那么未使用的参数将不再被包含在显示名称中。

#### JUnit Vintage
没有变化。


### 5.0.1

**发布时间**： 2017.10.03

**范围**：修复了5.0.0版的错误

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.1](https://github.com/junit-team/junit5/milestone/16?closed=1) 里程碑页面。

#### 整体改进
- 所有的包现在都有一个`optional`的依赖，而不需要在其发布的Maven POM中强制依赖*@API Guardian* JAR包。

#### JUnit Platform
没有变化。

#### JUnit Jupiter

###### Bug修复
- 如果测试类中未声明JUnit 4 `ExpectedException`规则，`junit-jupiter-migrationsupport`模块中的`ExpectedExceptionSupport`不会再吃掉异常。
	- 因此，现在可以使用`@EnableRuleMigrationSupport`和`ExpectedExceptionSupport`，而不用声明`ExpectedException`规则。


#### JUnit Vintage

###### Bug修复
- `PackageNameFilters`现在应用于通过`ClassSelector`，`MethodSelector`或`UniqueIdSelector`选择的测试。



### 5.0.0

**发布时间**： 2017.09.10

**范围**：首个通用版本

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 GA](https://github.com/junit-team/junit5/milestone/10?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复
- `AbstractTestDescriptor`中的`removeFromHierarchy()`实现现在也清除了所有子级的父级关系。

###### 弃用和彻底改变
- `@API`注释已经从`junit-platform-commons`项目中删除，并重新定位到GitHub上一个名为 [@API Guardian](https://github.com/apiguardian-team/apiguardian) 的独立新项目。
- Tag不再允许包含以下任何保留字符。
	- `,`, `(`, `)`, `&`, `|`, `!`
- `FilePosition`的构造函数已被替换为一个名为`from(int，int)`的静态工厂方法。
- 一个`FilePosition`现在全完可以通过新的`from(int)`静态工厂方法从一个行号进行构建。
- `FilePosition.getColumn()`现在返回`Optional<Integer>`而不是`int`。
- 以下所列的`TestSource`几个具体实现类的构造函数已被替换为命名`from(…​)`的静态工厂方法。
	- `ClasspathResourceSource`
	- `ClassSource`
	- `CompositeTestSource`
	- `DirectorySource`
	- `FileSource`
	- `MethodSource`
	- `PackageSource` 

- `LoggingListener`的构造函数已被替换为名为`forBiConsumer(...)`的静态工厂方法。
- `AbstractTestDescriptor`中的`getParent()`方法现在是`final`的。

###### 新特性与改进
- `AbstractTestDescriptor`中的`children`字段现在是`protected`的，从而让子类能够访问。


#### JUnit Jupiter

###### Bug修复
- `AbstractExtensionContext.getRoot()`现在会遍历完整的层次结构并返回真正的根上下文。


#### JUnit Vintage
没有变化。



### 5.0.0-RC3
**发布时间**： 2017.08.23

**范围**：配置参数和错误修复。

>⚠️ 这是一个预发行版，包含一些重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 RC3](https://github.com/junit-team/junit5/milestone/13?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复
- 源JAR包不再包含每个源文件两次。
- The Maven Surefire provider now reports a failed test with a cause that is not an instance of AssertionError as an error instead of a failure for compatibility reasons.
- Maven Surefire提供者程序现在报告一个失败的测试，其原因不是`AssertionError`的一个实例就是一个*错误*，而不是因为兼容性导致的*失败*。

###### 新特性与改进
- 现在可以通过许多新的方式提供`配置参数`：
	- 通过类路径根目录下的`junit-platform.properties`文件。详情请参阅 [配置参数](#45-配置参数)。
	- 通过 [控制台启动器](#43-控制台启动器) 中的`--config`命令行选项。
	- 通过Gradle插件的`configurationParameter`或`configurationParameters` DSL。
	- 通过Maven Surefire提供这程序的`configurationParameters`属性。
	
#### JUnit Jupiter

###### Bug修复
- 源JAR包不再包含每个源文件两次。
- `ExecutionContext.Store.getOrComputeIfAbsent`现在在计算值之前会在其祖父级上下文中查找值（并在其父级中递归）。
- `ExecutionContext.Store.getOrComputeIfAbsent()`现在是现成安全的。
- 如果唯一ID属于不同的测试引擎，`JupiterTestEngine`就不会再尝试解析通过其中一个`DiscoverySelectors.selectUniqueId()`方法选择的唯一ID。

###### 弃用和彻底改变
- 恢复RC1中引入的更改：现在使用与Java类相同的默认测试实例生命周期模式（即"per-method"）执行使用Kotlin编程语言编写的测试类。
- `junit.conditions.deactivate` 配置参数已被重命名为` junit.jupiter.conditions.deactivate`。
- `junit.extensions.autodetection.enabled`配置参数已被重命名为` junit.jupiter.extensions.autodetection.enabled`。
- `ExtensionContext`中的默认全局扩展名称空间常量已从`Namespace.DEFAULT`重命名为`Namespace.GLOBAL`。
- 默认的`getStore()`方法已经从`ExtensionContext`接口中移除。要访问全局存储，需要显式调用`getStore(Namespace.GLOBAL)`方法。

###### 新特性与改进
- 现在可以通过名为`junit.jupiter.testinstance.lifecycle.default`的配置参数或JVM系统属性来设置*默认*的测试实例生命周期模式。详情请参阅 [更改默认的测试实例生命周期](#381-更改默认的测试实例生命周期)。
- 在参数化测试中使用`@CsvSource`或`@CsvFileSource`时，如果CSV解析器没有从输入中读取到任何字符，并且输入位于引号内，则返回空字符串`""`而不是`null`。


#### JUnit Vintage

###### Bug修复
- 源JAR包不再包含每个源文件两次。
- 现在可以通过`DiscoverySelectors`中的`selectMethod()`变体在JUnit 4参数化测试类中选择单个方法。



### 5.0.0-RC2
**发布时间**： 2017.07.30


**范围**：修复`junit-jupiter-engine`的Gradle消耗

> ⚠️ 这是一个预发行版，包含一些重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。


关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 RC2](https://github.com/junit-team/junit5/milestone/12?closed=1)里程碑页面。


#### JUnit Platform
没有变化。

#### JUnit Jupiter

###### Bug修复
- 修正`junit-jupiter-engine`的无效POM，排除`test`作用域依赖。

#### JUnit Vintage
没有变化.


### 5.0.0-RC1
**发布时间**： 2017.07.30

**范围**：5.0 GA之前的错误修复和文档改进

> ⚠️ 这是一个预发行版，包含一些重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 RC1](https://github.com/junit-team/junit5/milestone/9?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复
- 现在可以通过类，方法名，参数类型或完全限定的方法名来*选择*未被实现类覆盖的通用接口`default`方法。这适用于`DiscoverySelectors`中的方法选择器以及`ReflectionSupport`中的`findMethod()`变体。
- 在使用`ReflectionSupport`中的`findMethods()`搜索类层次结构中的方法时，现在可以正确发现一个非重写的接口`default`方法，其方法签名被本地声明的方法重载。
- 在使用`ReflectionSupport`中的`findMethods()`搜索类层次结构中的方法时，不再发现重写的接口`default`方法。


###### 弃用和彻底改变
- 从`Launcher`类中删除了已弃用的方法`execute(LauncherDiscoveryRequest)`。已删除的方法在里程碑4中被`execute(LauncherDiscoveryRequest，TestExecutionListener ...)`方法替换。



#### JUnit Jupiter

###### Bug修复
- 与`@BeforeAll`，`@AfterAll`，`@BeforeEach`或`@AfterEach`注解标注的生命周期方法有关的配置错误在测试发现阶段不再停止执行整个测试计划。相反，现在在执行受影响的测试类时就会报告这样的错误。
- 测试计划中已经正确地包含了一个未覆盖的接口默认方法，其方法签名被本地声明的方法重载。这适用于使用了Jupiter注解（如`@Test`，`@BeforeEach`等）的`default`方法。
- 测试计划中不再包含重写的接口`default`方法。这适用于使用了Jupiter注解（如`@Test`，`@BeforeEach`等）的`default`方法。

###### 新特性与改进
- `Assertions.assertThrows()`接受自定义失败消息的新变体作为`String`或`Supplier<String>`。
- 如果测试类使用了`@TestInstance(Lifecycle.PER_CLASS)`，则`@MethodSource`引用的方法不再必须是`static`的。
- 使用Kotlin编程语言编写的测试类现在默认使用`@TestInstance(Lifecycle.PER_CLASS)`语义来执行。


#### JUnit Vintage
除了内部重构之外没有变化。


### 5.0.0-M6
**发布时间**： 2017.07.18

**范围**：JUnit 5的第六次发布，主要解决Java 9的兼容性、验证（例如：Tag语法规则）和修复bug。

> ⚠️ 这是一次里程碑式的发布，包含重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的  [5.0 M6](https://github.com/junit-team/junit5/milestone/11?closed=1) 里程碑页面。
 
#### 兼容Java 9
JUnit 5的运行时环境的主要目标是Java 8，因此，JUnit 5的发布版本不能描述Java 9的编译模块。然而，由于 [第5次发布](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m5) 的每个发布包在其JAR清单中声明了稳定的`Automatic-Module-Name`，这使得可以在测试模块中包含著名的JUnit模块名称，如下所示。

```java
module foo.bar {
  requires org.junit.jupiter.api;
}
```

通常在类路径上就可以运行测试，这方面Java 8和Java 9没什么区别。只要支持JUnit平台，大多数的命令行工具和IDE都可以*开箱即用*JUnit 5。如果你选择的开发工具不支持JUnit平台，可以使用`ConsoleLauncher`，甚至可以使用可执行的`junit-platform-console-standalone`一站式jar包。

要在模块路径中运行JUnit Jupiter测试，可以通过一个Java 9兼容的构建工具 [pro](https://github.com/forax/pro) 来实现。

**pro** 支持黑盒测试和白盒测试。前者用来测试模块表面，只能访问应用程序模块的导出位。后者使用合并的模块描述符技术，允许访问`protected`和包私有类型以及非导出包。

测试模块的例子可以查看 [pro的GitHub仓库](https://github.com/forax/pro/tree/master/src/test/java)：`integration.pro`是一个黑盒测试模块；但是`com.github.forax.pro.api`和`com.github.forax.pro.helper`是白盒测试模块。

#### JUnit Platform

###### Bug修复
* 为了删除前导和尾随的空白，所有的Tag都被*修剪*了。这适用于任何直接通过`TagFilter.includeTags()`和`TagFilter.excludeTags()`或者间接通过`@IncludeTags`,`@ExcludeTags`，JUnit Platform控制台启动器，JUnit Platform的Gradle插件和JUnit平台的Maven Surefire provider所提供的任何Tag。

###### 弃用和彻底改变
* 所有的Tag现在都要满足以下语法规则：
	* 标签不能是`null`或者*空白*。
	* *修剪*的标签不能包含空白。
	* *修剪*的标签不能包含ISO控制字符。
* 如果提供的Tag在语法上是无效的，`TagFilter.includeTags()`,`TagFilter.excludeTags()`和`TestTag.create()`工厂方法现在会抛出一个`PreconditionViolationException`异常。
* `EngineDiscoveryRequest`的方法`getDiscoveryFiltersByType`已经被改名为`getFiltersByType`。
* `UniqueId`的方法`getSegments()`现在返回一个不可变的列表。
* `AbstractTestDescriptor`的方法`setSource`被删除，替代它的是一个包含`source`变量的构造函数。

###### 新特性与改进
* 为语法有效的Tag增加了一个新的检查方法`TestTag.isValid(String)`
* 如果应用的元素是类，`AnnotationSupport`的方法`findAnnotation()`现在可以被一个类实现的接口中搜索
* `org.junit.platform.commons.util.ReflectionUtils`的下面这些方法现在通过`org.junit.platform.commons.support.ReflectionSupport`向外暴露：
	* `public static Optional<Class<?>> loadClass(String name)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, String parameterTypeNames)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, Class<?>…​ parameterTypes)`
	* `public static <T> T newInstance(Class<T> clazz, Object…​ args)`
	* `public static Object invokeMethod(Method method, Object target, Object…​ args)`
	* `public static List<Class<?>> findNestedClasses(Class<?> clazz, Predicate<Class<?>> predicate)`

#### JUnit Jupiter

###### Bug修复
* 为了删除前导和尾随的空白，所有通过`@Tag`声明的标记都被修剪了。
* 现在支持所有基本数组类型（从`boolean[]`到`short[]`）作为参数化测试的静态参数的返回类型。

###### 弃用和彻底改变
* `@Test`和有生命周期的方法现在都强制返回`void`类型。

###### 新特性与改进
* 现在可以使用`Assertions `中的所有`fail(...)`方法来实现单语句lambda表达式，从而避免需要实现具有显式返回值的代码块。
* `ExtensionContext`中的新方法`getRoot()`能够容易的获得最高的，*root*扩展上下文。
* `ExtensionContext` API中的新方法`getRequiredTestClass()`、`getRequiredTestInstance()`和`getRequiredTestMethod()`更加便捷，可用于
在需要这些元素的用例中检索测试类，测试实例和测试方法。
* 类似`@TestInstance`和`@Disabled`的类级注解现在可以被声明与测试接口上（也称为测试特征）。
* 如果针对单个方法解析了多个`TestDescriptor`，则现在会记录一条警告。 这有助于调试由多个竞争注解（例如`@Test`，`@RepeatedTest`，`@ParameterizedTest`，`@TestFactory`等）同时注解的方法导致的错误。
* 包含无效标记语法的`@Tag`声明现在将被记录为警告，但会被有效地忽略掉。

#### JUnit Vintage

###### Bug修复

* 与`@Unroll`一起使用时，添加了对`Runners`的支持，这些`Runners`报告不属于`Description`树的测试事件，例如`Spock`的`Sputnik`。 以前，这样的测试根本没有报告; 现在它们被作为动态测试而报告。


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

####### Bug修复

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
* 当使用`DiscoverySelectors`通过完全限定的方法名称来选择一个方法，或者通过提供`methodParameterTypes`作为一个逗号分隔的字符串时，可以使用*源代码语法*来描述数组参数类型，如基本数组`int[]`和`java.lang.String[]`作为一个对象数组。 此外，现在可以使用JVM的内部字符串表示来描述多维数组类型（例如，`[[[I`代表`int[][][]`，`[[Ljava.lang.String;`代表 `java.lang.String[][]`等）或*源代码语法*（例如，`boolean[][][]`，`java.lang.Double[][]`等）。
* `JUnit`的平台的`Gradle`插件的`junitPlatformTest`任务现在可以在构建的配置阶段直接访问。
* `JUnit Platform Gradle`插件与`Gradle Kotlin DSL`配合使用效果更佳。
* 当通过Java的`ServiceLoader`机制发现`TestEngine`时，现在将尝试确定引擎类的加载位置，如果位置URL可用，则将在配置级别进行记录。
* `mayRegisterTests()`方法可用来表示一个`TestDescriptor`将在执行过程中注册动态测试。
* 现在可以查询`TestPlan`是否包含`Test()`。 这被`Surefire`提供者用来决定一个类是否是一个应该被执行的测试类。
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
* `Assertions.assertAll()`现在将跟踪几乎所有类型的异常（而不是只跟踪`AssertionError`类型的异常），除非异常是一个被列入*黑名单*的异常，在这种情况下，它将被立即重新抛出。
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


### 5.0.0-M4
**发布时间**： 2017.04.01

**范围**：JUnit 5的第4个里程碑发布，主要关注点在于测试模板、反复测试和参数化测试。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0 M4](https://github.com/junit-team/junit5/milestone/7?closed=1) 里程碑页面。

#### JUnit Platform

##### Bug修复
* 为了保证可重复的构建，JUnit Platform的Gradle插件为其依赖增加了一个修复版本（和插件版本相同），替换了默认的动态版本方案（即`1.+`）。
* JUnit平台的Gradle插件如今明确地应用在`java`插件的内置Gradle中，因为它对应用程序有一个隐含的依赖关系。
* `ReflectionUtils`中的所有`findMethods()`实现不再返回合成方法。结果中不再包含隐含的 [override-equal](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.4.2)方法。
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
* `Launcher`中的方法`execute(LauncherDiscoveryRequest launcherDiscoveryRequest)`已经被弃用，且在M5发布将会被移除。Instead use the following new method that registers supplied TestExecutionListeners in addition to already registered listeners but only for the supplied LauncherDiscoveryRequest: execute(LauncherDiscoveryRequest launcherDiscoveryRequest, TestExecutionListener…​ listeners)

##### 新特性与改进
* 自定义的`TestExecutionListener`实现现在可以通过Java的`ServiceLoader`机制自动注册。
* `TestEngine `API添加了新的默认方法`getGroupId()`,`getArtifactId()`和`getVersion()`，这些方法用来调试和报告。默认情况下，包属性（一般来自JAR属性清单）用来决定发行ID和版本号；而组ID默认是空的。更多细节请参阅Javadoc的 [TestEngine](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) 文档。
* 已发现的测试引擎的记录信息得到增强，包括组ID，工件ID和每个测试引擎的版本，如果它们能够通过`getGroupId()`，`getArtifactId()`，以及`getVersion()`方法获取。
* 现在，`ConsoleLauncher` 的`--scan-classpath`允许扫描JAR文件作为显式参数提供的测试（请参阅 [Options](#431-options)）。
* 现在，`ConsoleLauncher`中新的`--details<Details>`选项允许在执行测试时选择一个输出细节的模式。可以使用`none`、`flat`、`tree`，或者`verbose`中的一个来指定该模式。如果没有指定，则只会输出汇总和测试失败信息（请参阅 [Options](#431-options)）。
* 可执行的`junit-platform-console-standalone-1.0.2.jar`包在默认的构建过程中生成，并存储于已在**Maven Central**中发布的 `junit-platform-console-standalone/build/libs`路径下。该jar包包含了Jupiter和Vintage测试引擎及其所有依赖。它在手动管理其依赖项的项目中提供了JUnit 5的无障碍使用，类似于JUnit 4中广为人知的简单JAR包管理。
* `ConsoleLauncher`中新增的`--exclude-classname (--N)`选项可以接受一个正则表达式来排除那些全类名匹配的类。当这个选项重复时，所有的模式将使用 "或" 语义进行组合。
* 新的JUnit Platform支持包`org.junit.platform.commons.support`包含了许多正在维护的工具方法，这些方法可用于注解、反射，以及类路径扫描任务。我们鼓励`TestEngine`和`Extension`作者使用这些支持方法，以便与JUnit Platform的行为保持一致。
* 压缩了`TestDescriptor.isTest()`和`TestDescriptor.isContainer()`的内部逻辑，使得其返回值从两个独立的布尔属性值转变为名为`TestDescriptor.Type`的枚举类.详细信息请参阅下一小节。
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

* 新增了`@ParameterizedTest `注解，从而为*参数化测试*提供了极好的支持。请参阅 [参数化测试](#313-参数化测试)。
* 新增了`@RepeatedTest`注解和`RepetitionInfo`API,从而为*重复测试*提供了极好的支持。请参阅 [重复测试](#312-重复测试)。
* 引入了新的注解`@TestTemplate`，并附带了对应的扩展点`TestTemplateInvocationContextProvider`。
* 现在，`Assertions.assertThrows()`方法在生成断言失败的消息时，会采用规范名称作为异常类型。
* 现在，在测试方法上注册的`TestInstancePostProcessors`会被调用。
* `Asserrions.fail`现在有了新的变体：`Assertions.fail(Throwable cause)`和`Assertions.fail(String message, Throwable cause)`。
* 新的`Assertions.assertLinesMatch()`方法会比较字符串列表，应用了`Object :: equals`和正则表达式。 `assertLinesMatch()`还提供了一个快速转发机制，可以跳过每个调用中预期会更改的行，例如持续时间，时间戳，堆栈跟踪等。详细信息请参阅 [org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html) 的JavaDoc。
* 现在可以通过Java的`ServiceLoader`机制自动注册扩展。详情请参阅 [自动扩展注册](#522-自动扩展注册)。

#### JUnit Vintage

###### Bug修复

* 修复了导致只有最后一次测试失败的报告。例如，使用`ErrorCollector`规则时，只报告最后一次失败的检查。 现在，使用`org.opentest4j.MultipleFailuresError`会报告所有失败。


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
* 现在，[`ConsoleLauncher`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)、Gradle插件和[`JUnitPlatform`](http://junit.org/junit5/docs/current/api/org/junit/platform/runner/JUnitPlatform.html) 运行器使用`^.* Tests？$`正则表达式作为测试运行中所包含的类名的默认匹配模式。
* Gradle插件现在允许明确地选择应该执行哪些测试（请参阅 [配置选择器](#配置选择器)）。
* 新的`@Testable`注解可以用来向IDE和开发工具供应商传递信息，用以说明被此注解所标注的元素是可测试的（即，它可以在JUnit Platform上作为测试被执行）。
* 现在，在使用`ClassNameFilter.includeClassNamePatterns`时，可以传递多个正则表达式，它们之间通过`"或"`语义组合。
* 现在，`@IncludeClassNamePatterns`注解可以将多个正则表达式以`"或"`的语义组合，并传递到`JUnitPlatform`运行器中。
* 现在，多个正则表达式可以通过`"或"`的语义，进行组合，并被传递给JUnit Platform Gradle 插件（参阅 [配置过滤器](#配置过滤器)）以及`ConsoleLauncher`（请参阅 [Options](#431-options)）。
* 现在，可以通过使用`PackageNameFilter.includePackageNames`来指定被包含的包名或使用`PackageNameFilter.excludePackageNames`来指定被排除的包名。
* 现在，`JUnitPlatform`运行器可以结合`@IncludePackages`注解指定希望包含的包名，也可以结合`@ExcludePackages`来指定希望排除在外的包名。
* 现在，对于使用JUnit Platform Gradle插件和`ConsoleLauncher`的用户，可应通过配置过滤器完成包名的添加与排除。
* 包名现在可以通过JUnit Platform Gradle插件（请参阅 [配置过滤器](#配置过滤器)）和[`ConsoleLauncher`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)（请参阅 [Options](#431-options)）的过滤器配置来包含或排除。
* 现在，`junit-platform-console`不再强依赖于[JOptSimple](https://pholser.github.io/jopt-simple/)。因此，现在可以测试那些使用该库的不同版本的自定义代码。
* 现在，包选择器的解析会扫描JAR文件。
* Surefire供应商现在支持forking。
* Surefire供应商想在支持使用标记过滤，可以通过以下方法：
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

* 支持`Assertions`中的lambda表达式的延迟和抢占超时。请参阅 [`AssertionsDemo`](#34-断言) 中的示例，更多详细信息请参阅[org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html) JavaDoc。
* 新的`assertIterableEquals()`断言会校验两个可迭代对象是否深度一致（查阅Java文档获取细节）。
* `Assertions.assertAll()`的新变体接收可执行流（即，`Stream<Executable>`）。
* 现在，`Assertions.assertThrows()`方法将返回抛出的异常。
* 现在，`@BeforeAll`与`@AfterAll`可以被声明在接口中的静态方法上。
* JUnit Jupiter现在支持JUnit 4中的`org.junit.rules.ExternalResource`、`org.junit.rules.Verifier`、`org.junit.rules.ExpectedException`规则及其子类，这更有利于JUnit 4代码库的迁移。

#### JUnit Vintage
*自5.0.0-M2以来没有变化。*


### 5.0.0-M2
**发布时间**： 2016.07.23

**范围**： JUnit 5的第2个里程碑发布

#### 变更摘要
此版本主要是一个bug修复版本，修复了自`5.0.0-M1`以来所发现的bug。

以下是一个全局改变的列表。关于更改的详细信息，包括Platform、Jupiter和Vintage，请参阅下面的章节。关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0 M2](https://github.com/junit-team/junit5/milestone/4?closed=1) 里程碑页面。

* JUnit 5的Gradle构建可以正常在Microsoft Windows下运行。
* 在AppVeyor上建立了针对Microsoft Windows的持续集成构建。

#### JUnit Platform

###### 修复Bug
* 容器中的故障--例如，抛出异常的`@BeforeAll`方法--如今在使用`ConsoleLauncher`或JUnit Platform Gradle插件时会构建失败。
* JUnit Platform Surefire Provider不再默默地忽略纯动态测试类——例如只声明`@TestFactory`方法的测试类。
* 在`junit-platform-console-<release version>` TAR和ZIP发布包中包含的`junit-platform-console`和`junit-platform-console.bat` shell脚本能够正确引用`ConsoleLauncher`，而不是`ConsoleRunner`。
* 被`ConsoleLauncher`和JUnit Plaform Gradle插件使用的`TestExecutionSummary`包含了失败的错误信息。
* 类路径扫描现在可以防止类加载和处理期间遇到的问题--例如，在处理格式错误的类时，潜在的异常被吞噬并记录在有问题的文件路径中，如果这个异常是一个列入黑名单的异常，比如`OutOfMemoryError`，那么它将被重新抛出。

###### 弃用
* `DiscoverySelectors`中基于通用名称的发现选择器（即`selectName()`和`selectNames()`）已经被废弃，建议使用`selectPackage(String)`、`selectClass(String)`和`selectMethod(String)`方法。

###### 新特性
* `DiscoverySelectors`中的新方法`selectMethod(String)`支持选择 *全限定方法名*。

#### JUnit Jupiter

###### 修复Bug
* 在类级别和方法级别通过`@ExtendWith`声明的扩展实现将不再被多次注册。

#### JUnit Vintage
*自5.0.0-M1以来没有变化。*



### 5.0.0-M1
**发布时间**： 2016.07.07

**范围**： JUnit 5的第一个里程碑发布

##### 变更摘要
下面首先给出整体改动。而有关Platform、Jupiter，以及Vintage的细节变动，可以查阅后面的专项信息。对于本次发布相关的 commit 信息，可以通过查看 [5.0 M1](https://github.com/junit-team/junit5/milestone/2?closed=1)里程碑页面在GitHub上的JUnit代码库了解。

* 在已发布的JAR清单中，包含了额外的元数据，例如`Create-By`，`Built-By`，`Build-Date`，`Build-Time`，`Build-Revision`，`Implementation-Title`，`Implementation-Version`，`Implementation-Vendor`等。
* 当前发布的工件中，包含了`LICENSE.md`和`META-INF`。
* 现在，JUnit参与到 [Up For Grabs](http://up-for-grabs.net/#/tags/junit) 运动中，以便为开源做贡献。
	* 点击这里了解更多关于 [up-for-grabs](https://github.com/junit-team/junit5/labels/up-for-grabs) 的信息。
* 对于已发布的工件，其组ID,工件ID,以及版本都已改变。
	* 查看 [工件迁移](#工件迁移) 以及 [依赖元数据](#21-依赖元数据)。
* 所有的基础包都被重命名了。
	* 查看 [包迁移](#包迁移)

<a id="工件迁移"></a>
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

<a id="包迁移"></a>
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

#### JUnit Platform
* 重命名`ConsoleRunner`为`ConsoleLauncher`。
* 现在，`ConsoleLauncher`在退出时总会返回状态码，并且已经移除了*激活退出码*的标志位。
* `junit-platform-console`不再在`junit-platform-runner`，`junit-jupiter-engine`，或`junit-vintage-engine`上定义传递依赖。
* 现在，`JUnit5`运行器已经被重命名为`JUnitPlatform`。
	* `@Packages`已经被重命名为`@SelectPackages`。
	* `@Classes`已经被重命名为`@SelectClasses`。
	* `@UniqueIds`已经被移除。
	* 引入了新的注解`@UseTechnicalNames`。
		* 详情请参阅 [显示名称与技术名称](#442-显示名称与技术名称)
* Gradle的JUnit Platform插件已经全面修订。
	* 现在，JUnit Platform的 Gradle 插件要求 Gradle 版本在2.5或更高版本。
	* Gradle中名为`junit5Test`的任务已经被重命名为`junitPlatformTest`。
	* Gradle中名为`junit5`的配置已经被重命名为`junitPlatform`。
		* `runJunit4`已经被`enableStandradTestTask`替换。
		* `version`已经被`platformVersion`替换。
	* 详情请查阅 [Gralde](#421-gradle)。
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

#### JUnit Jupiter
* `junit5`引擎ID已经被重命名为`junit-jupiter`。
* `Junit5TestEngine`已经被重命名为`JupiterTestEngine`。
* 现在，`Assertions`提供了以下支持：
	* `assertEquals()`方法可以对基本类型使用
	* `assertEquals()`方法可以对包含增量的double类型和floats类型的值使用
	* `assertArrayEquals()`
	* 现在期望值与实际值都被提供给`AssertionFailedError`。
* [动态测试](#315-动态测试):现在，测试可以通过lambda表达式在运行时被动态注册。
* 现在，`TestInfo`通过`getTags()`方法提供了获取标签的方法。
* 现在，`@AfterEach`注解所标注的方法以及*after*回调会在被`@Test`、`@BeforeEach`注解所标注的方法或*before*回调抛出异常后被调用。
* 现在，`@AfterAll`注解所标注的方法以及*after all*回调一定会被调用。
* 现在，在测试类的层级结构中，父类或父级接口上使用的可重复注解，例如`@ExtendWith`和`@Tag`,会被发现。
* 现在，在测试类或接口的层级结构中，扩展将会被*自上而下*地注册。
* 现在，测试和容器的 [执行条件可以被禁用](#531-停用条件)。
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

