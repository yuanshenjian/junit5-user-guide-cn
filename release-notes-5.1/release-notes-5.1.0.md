## 5.1.0

**发布时间**： 2018.02.18

**范围**：

- 在Java 9模块中发现测试
- 提升Kotlin的支持
- 通过 [@RegisterExtension](https://junit.org/junit5/docs/5.1.0/api/org/junit/jupiter/api/extension/RegisterExtension.html) 的 [编程式扩展注册]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#extensions-registration-declarative' }})
- 用于过滤测试的 [标记表达式语言]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#46-标记表达式' }})
- 基于注解的 [条件测试执行]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#53-条件测试执行' }})，支持环境变量、系统属性、操作系统、JRE版本和动态脚本
- 编写 [参数化测试]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#314-参数化测试' }}) 的各种改进
- [ExtensionContext](https://junit.org/junit5/docs/5.1.0/api/org/junit/jupiter/api/extension/ExtensionContext.html) API的改进
- 支持在IDE中重新运行单个动态测试、参数化测试和测试模板调用


关于此版本所有已关闭的 问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.1 M1](https://github.com/junit-team/junit5/milestone/14?closed=1)、[5.1 M2](https://github.com/junit-team/junit5/milestone/18?closed=1)、[5.1 RC1](https://github.com/junit-team/junit5/milestone/19?closed=1)、[5.1 GA](https://github.com/junit-team/junit5/milestone/20?closed=1) 里程碑页面。本章节涵盖了版本5.0.3到5.1.0的所有变化。

### JUnit Platform

###### Bug修复

- 通过`DiscoverySelectors.selectMethod(String)`、junitPlatform Gradle插件的选择器配置的`method`或`methods`元素以及`ConsoleLauncher`的`-select-method`或`-m`命令行选项这三种方式所提供的*完全限定的方法名称*所选中的测试方法现在包含特殊字符 - 例如，JVM语言（如Kotlin和Groovy）。

###### 新特性与改进

###### Java 9模块的支持

- 新的`ModuleSelector`发现选择器，用于扫描测试类的Java 9模块。
	- 这是对现有类路径扫描支持的替代方案。
- 新的控制台启动选项`--select-module <name>`或`-o <name>` 用于选择用于测试发现的Java 9模块。
	- 这是对现有类路径扫描支持的替代方案。
- 新的控制台启动选项`--scan-modules`用于扫描引导层配置中可用的所有已解析的Java 9模块从而进行测试发现。
	- 这是对现有类路径扫描支持的替代方案。
- 在Java 9或更高版本上运行时，`TestEngine`接口中的`getVersion()`和`getArtifactId()`的`default`实现向*Java平台模块系统* 查询此信息。

###### 杂项
- 现在，使用`Launcher`、`ConsoleLauncher`、Gradle插件或Maven Surefire provider执行测试时，可以根据标记使用 [标记表达式语言]() 来包含或排除测试。
- `junit-platform-suite-api`模块中新的`@SuiteDisplayName`注解，用于声明测试套件的自定义*显示名称*。
	- 在`junit-platform-runner`模块中由JUnit 4的`JUnitPlatform`运行器支持。
- 控制台启动程序运行的摘要表现在包含最初的十个堆栈跟踪行，以更好地描述故障的位置。
- 类路径扫描期间发生的类加载错误现在记录在DEBUG级别（即`java.util.logging`中的`FINE`日志级别），而不是作为警告。


### JUnit Jupiter

###### Bug修复

- 通过`DiscoverySelectors`中的某个`selectClass()`变体、`junitPlatform` Gradle插件的选择器配置的`aClass`或`classes`元素，或者`ConsoleLauncher`的`-select-class`或`-c`命令行选项这三种方式所选中的测试类不再是允许是`private`的。这与通过包、类路径和模块路径扫描发现的测试类的行为一致。
- 现在，通过参数化测试的第一行通过`@CsvSource`在最后一列中指定的空元素现在可以正确转换为`null`而不是空`String`。

###### 新特性与改进

###### 测试发现

- `JupiterTestEngine`支持用于选择Java 9模块的新JUnit Platform `ModuleSelector`。
	- 这是对现有类路径扫描支持的替代方案。
- 现在可以单独执行选定的动态测试和测试模板调用，而无需运行完整的测试工厂或测试模板。这允许通过在随后的发现请求中选择其唯一ID来重新运行单个或选定的参数化、重复或动态测试。

###### 编程模型
- 开发人员现在可以通过使用 [`@RegisterExtension`](https://junit.org/junit5/docs/5.1.0/api/org/junit/jupiter/api/extension/RegisterExtension.html) 注解在测试类中标注字段来以编程的方式注册扩展。
	- 详情请参阅用户指南的 [编程式扩展注册]()。
- 用于声明性条件测试执行的新预定义`@Enabled*`和`@Disabled*`注解。有关详细信息，请参阅用户指南的以下部分：
	- [操作系统]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#' }}) 
	- [JRE版本]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#' }}) 
	- [系统属性]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#' }}) 
	- [环境变量]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#' }}) 
- 新的`@EnabledIf`和`@DisabledIf`注解可用于通过解析一个动态脚本语言（例如JavaScript或Groovy）的脚本来控制是否*启用* 或*禁用* 带注解的测试类或测试方法。
	- 详情请参阅用户指南的 [基于脚本的条件]()。
- 接受可执行文件集合的`Assertions`中的新`assertAll()`变体。

###### 改进的Kotlin支持
- 添加了新的Kotlin友好断言并且作为`org.junit.jupiter.api`包中的顶级函数。
	- `assertAll()`： 参数是 `Stream<() -> Unit>`、`Collection<() -> Unit>`或者 `vararg () -> Unit`。
	- `assertThrows()`：使用Kotlin具体化泛型。
	- `fail()`：删除需要明确指定泛型类型。
		- 当从Kotlin调用`Assertions.fail`方法时，编译器要求在调用它时显式声明通用返回类型的`fail`- 例如，`fail<Nothing>("Some message")`。这些新的顶层函数通过返回 [`Nothing`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-nothing.html) 来消除此要求。

###### 参数化测试
- `@CsvFileSource`现在支持一个`numLinesToSkip`属性，可用于跳过CSV文件中的标题行。
- `@ValueSource`现在还支持参数化测试的`short`、`byte`、`float`、`char`和`java.lang.Class`类型的字面值。
- `@MethodSource`的值属性不再是强制性的。如果没有值（或一个空字符串）作为方法名提供，则与当前`@ParameterizedTest`方法同名的方法将按照约定用作工厂方法。
	- 详情请参与用户指南的 [@MethodSource]()。
- 对参数化测试的新支持，用于将字符串隐式转换为任何以下常见Java类型的参数。有关示例，请参阅用户指南中的 [隐式转换表]()。
	- `java.io.File`
	- `java.math.BigDecimal`
	- `java.math.BigInteger`
	- `java.net.URI`
	- `java.net.URL`
	- `java.nio.charset.Charset`
	- `java.nio.file.Path`
	- `java.util.Currency`
	- `java.util.Locale`
	- `java.util.UUID`
- 如果目标类型只声明一个合适的工厂方法或工厂构造函数，则用于参数化测试的新回退机制用于将`String`隐式转换为给定目标类型的参数。
	- 详情请参与用户指南的 [Fallback String-to-Object Conversion]()。

###### 扩展模型

- JUnit Jupiter的扩展现在可以通过`ExtensionContext` API中新的`getConfigurationParameter(String key)`方法在运行时访问JUnit Platform配置参数。
- JUnit Jupiter的扩展现在可以通过`ExtensionContext` API中的新`getTestInstanceLifecycle()`方法访问当前测试实例的生命周期。
- `ExtensionContext.Store`中引入的新回调接口`CloseableResource`。一个`Store`被绑定到其扩展上下文的生命周期。当扩展上下文的生命周期结束时，关联的store将关闭，并且每个已保存的`ExtensionContext.Store.CloseableResource`实例都会通过调用其`close()`方法来通知。
- `ExtensionContext.Store`提供了新的便捷方法`getOrComputeIfAbsent(Class)`，它简化了扩展希望在`Store`中存储给定类型（由该类型键入）的单个对象并使用该类型的默认构造函数创建对象的使用场景。
	- 例如，`store.getOrComputeIfAbsent(X.class, key -> new X(), X.class)`等代码现在可以用`store.getOrComputeIfAbsent(X.class)`替代。


### JUnit Vintage

###### Bug修复

- 当使用标记过滤器来包含/排除表示JUnit 4类别的标签时，例如`"com.example.Integration"`，Vintage引擎不再错误地执行包含至少一种包括的测试方法的测试类的所有测试方法，例如，一个用`@Category(com.example.Integration.class)`的注解，不管它们是否属于同一个类别。

###### 新特性与改进
- `VintageTestEngine`支持用于选择Java 9模块的新JUnit Platform `ModuleSelector`。
	- 这是对现有类路径扫描支持的替代方案。
- 在此版本之前，Vintage测试引擎仅返回了一个无子的`TestDescriptor`，用于使用`@Ignore`注解的测试类。但是，像Gradle这样的构建工具需要显示准确数量的测试，即它们想要访问和统计测试类的测试方法，而不管它是否被忽略。 Jupiter引擎已经发现了跳过的容器，例如，用`@Disabled`注释的测试类，包括他们的子类。 Vintage引擎现在采用这种方法，并为那些被`@Ignore`注解标注的类返回`TestDescriptors`的完整子树。在执行期间，它只会将测试类的`TestDescriptor`报告为跳过的，这与Jupiter引擎报告跳过容器的方式一致。
