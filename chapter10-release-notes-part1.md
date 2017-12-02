## 10. 发布记录

### 5.0.2

**发布时间**： 2017.11.12

**范围**：自5.0.1版本以来的错误修复和小的改进。

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.2](https://github.com/junit-team/junit5/milestone/17?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复

- 修复后，Maven Surefire对于不使用`MethodSource`的测试引擎（例如Spek）能正确地报告失败的测试。

- 修复后，当一个非零的`forkCount`与Maven Surefire一起执行时，可以正确地报告写入`System.out`或`System.err`的测试，特别是通过一个日志框架的时候。


###### 新功能和改进

- JUnit Platform Maven Surefire提供者程序现在支持`redirectTestOutputToFile` Surefire功能。

- JUnit Platform Maven Surefire提供者程序现在会忽略通过`<includeTags/>`，`<groups/>`，`<excludeTags/>`和`<excludedGroups/>`提供的空字符串。


#### JUnit Jupiter

###### Bug修复

- `@CsvSource`或`@CsvFileSource`输入行中的尾随空格不再生成空值。

- 以前，`@EnableRuleMigrationSupport`无法识别`@Rule`方法，该方法返回一个已支持的`TestRule`类型的子类型。而且，它错误地实例化了某些多次使用方法声明的规则。现在，一旦启用，它将实例化所有声明的规则（字段*和*方法），并按照JUnit 4使用的顺序来调用它们。

> - Previously, disabled test classes were eagerly instantiated when Lifecycle.PER_CLASS was used. Now, ExecutionCondition evaluation always takes place before test class instantiation.

- 以前，当使用`Lifecycle.PER_CLASS`时，被禁用的测试类会被迫切地实例化。现在，`ExecutionCondition`总是在测试类实例化之前就被解析。

- `unit-jupiter-migrationsupport`模块不再会错误地尝试通过`ServiceLoader`机制来注册`JupiterTestEngine`，从而允许将其用作Java 9模块路径上的模块。

###### 新功能和改进

- 现在，`Assertions`类中的`assertTrue()`和`assertFalse()`的失败消息包含了关于预期和实际布尔值的详细信息。
	- 例如，调用`assertTrue(false)`生成的失败消息现在变成了`"expected:<true>but was: <false>"`，而不是空字符串。

- 如果参数化测试没有消费通过参数源提供给它的所有参数，那么未使用的参数将不再被包含在显示名称中。

#### JUnit Vintage
没有变化。


### 5.0.1

**发布时间**： 2017.10.03

**范围**：修复了5.0.0版的错误

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.1](https://github.com/junit-team/junit5/milestone/16?closed=1) 里程碑页面。

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

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 GA](https://github.com/junit-team/junit5/milestone/10?closed=1) 里程碑页面。


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

###### 新功能和改进
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

关于此版本所有已关闭问题和请求的完整列表，请参阅GitHub上JUnit存储库中的 [5.0 RC3](https://github.com/junit-team/junit5/milestone/13?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复
- 源JAR包不再包含每个源文件两次。
- The Maven Surefire provider now reports a failed test with a cause that is not an instance of AssertionError as an error instead of a failure for compatibility reasons.
- Maven Surefire提供者程序现在报告一个失败的测试，其原因不是`AssertionError`的一个实例就是一个*错误*，而不是因为兼容性导致的*失败*。

###### 新功能和改进
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

###### 新功能和改进
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


关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 RC2](https://github.com/junit-team/junit5/milestone/12?closed=1)里程碑页面。


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

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 RC1](https://github.com/junit-team/junit5/milestone/9?closed=1)里程碑页面。


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

###### 新功能和改进
- `Assertions.assertThrows()`接受自定义失败消息的新变体作为`String`或`Supplier<String>`。
- 如果测试类使用了`@TestInstance(Lifecycle.PER_CLASS)`，则`@MethodSource`引用的方法不再必须是`static`的。
- 使用Kotlin编程语言编写的测试类现在默认使用`@TestInstance(Lifecycle.PER_CLASS)`语义来执行。


#### JUnit Vintage
除了内部重构之外没有变化。

