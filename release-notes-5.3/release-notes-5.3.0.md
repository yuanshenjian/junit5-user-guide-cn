## 5.3.0

**发布时间**： 2018.09.03

**范围**：

并行测试执行，`System.out`和`System.err`的输出捕获，新的`TestInstanceFactory`扩展API，动态测试的自定义测试源，从实验到维护状态的动态测试API的推广，`junit-platform-gradle-plugin`的终止， 对`junit-platform-surefire-provider`的弃用，以及各种小改进和错误修复。

关于此版本所有已关闭的 问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.3 M1](https://github.com/junit-team/junit5/milestone/23?closed=1)、[5.3 RC1](https://github.com/junit-team/junit5/milestone/27?closed=1) 和 [5.3 GA](https://github.com/junit-team/junit5/milestone/28?closed=1) 里程碑页面。

### JUnit Platform

###### Bug修复
- 现在，在`--details`详细模式下运行`ConsoleLauncher`时，完整的堆栈跟踪将打印到控制台。
- 如果嵌套类或嵌套接口具有无效的类文件，则`ReflectionUtils.findNestedClasses()`和`ReflectionSupport.findNestedClasses()`不再允许抛出`NoClassDefFoundError`。 相反，错误现在将被吞下并记录在警告级别。
- 所有`DiscoverySelector`实现（例如，`PackageSelector`、`ClassSelector`、`MethodSelector`等）现在实现了`equals()`和`hashCode()`，以便在存储在集合中时获得正确的行为。
- 修改了`ClassSource`，以便`equals()`和`hashCode()`现在正确地基于所需的类名而不是可选的`Class`引用。此外，现在强制执行类名的非空前置条件。

###### 弃用及重大变化
- `junit-platform-gradle-plugin`已经停止使用，不再作为JUnit 5的一部分发布。请使用 [Gradle的原生支持]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#42-构建工具支持' }}) 在JUnit平台上运行测试（需要Gradle 4.6或更高版本）。
- 现在不推荐使用JUnit Platform Surefire Provider（`junit-platform-surefire-provider`），推荐使用Maven Surefire 2.22.0及更高版本提供的对JUnit平台的原生支持。
- JUnit Platform Launcher现在强制执行JUnit Team发布的只有TestEngine实现可以使用`junit-`前缀作为其TestEngine ID。
	- 详情请参阅 [用户指南]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#prefix-is-reserved-for-test-engines'}}) 
- `AnnotationSupport`和`AnnotationUtils`中的`findAnnotation()`方法不再缓存注解查找。但是请注意，它的算法保持不变，因此在语义上与先前的行为相同。

###### 新特性与改进
- 对可扩展`HierarchicalTestEngine`的测试引擎的并行测试执行的可重用支持。
	- `HierarchicalTestEngine`实现现在可以指定`HierarchicalTestExecutorService`。
	- 默认情况下，使用`SameThreadHierarchicalTestExecutorService`。
	- 测试引擎可以使用`ForkJoinPoolHierarchicalTestExecutorService`来支持基于Java的Fork/Join框架的并行测试执行。
	- `Node`实现可以提供一组`ExclusiveResources`和一个由`ForkJoinPoolHierarchicalTestExecutorService`使用的`ExecutionMode`。
- 实验性支持捕获在测试执行期间打印到`System.out`和`System.err`的输出。
	- 默认情况下禁用此功能，但可以使用配置参数启用此功能（有关详细信息，请参阅 [用户指南]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#47-捕获标准输出错误'}})）。
	- 如果启用了新的实验功能，则`System.out`和`System.err`捕获的输出将分别写入`ConsoleLauncher`生成的XML报告中的专用`system-out`和`system-err`元素。
- 新的`UriSource.from(URI)`静态工厂方法，允许从`URI`创建`TestSource`。 如果`URI`引用本地文件系统中的文件或目录，则将创建`FileSource`或`DirectorySource`; 否则，将创建默认`UriSource`实现的实例。
- 新的`MethodSource.from(Class，Method)`静态工厂方法，用于从特定的类和方法创建`MethodSource`。 当测试方法从超类继承或作为接口默认方法存在时，应使用此方法以支持`MethodSource.from(Method)`。
- 如果URI使用类路径方案，现在可以通过新的`from(URI)`静态工厂方法从`URI`创建`ClasspathResourceSource`。
- `AnnotationSupport`中`isAnnotated()`的新重载变体，它接受`Optional<？extends AnnotatedElement>`而不是`AnnotatedElement`。
- 用于配置`LauncherFactory`的新`LauncherConfig`和关联的构建器。 具体而言，现在可以禁用测试引擎和测试执行侦听器的自动注册，并且可以以编程方式注册其他引擎和侦听器。
- 用于`ConsoleLauncher`的新的`--fail-if-no-tests`命令行选项
	- 启用此选项并且未发现任何测试时，启动程序将失败并退出，状态代码为`2`。
- `ConsoleLauncher`现在使用底层的 [picocli](https://github.com/remkop/picocli) 库来解析命令行并生成使用帮助。
	- 用户现在可以通过参数文件（`@-file`）将命令行参数传递给控制台启动器。参数文件允许用户在创建具有大量选项的命令行或使用长参数选项时解决命令行长度的系统限制。
	- 现在，在支持的平台上使用ANSI颜色显示使用帮助。


### JUnit Jupiter

###### Bug修复
- 使用`@TestInstance(Lifecycle.PER_CLASS)`语义时，如果测试类构造函数抛出异常，则不再调用已注册的`AfterAllCallback`扩展。因此，现在仅在调用`BeforeAllCallback`扩展时才调用`AfterAllCallback`扩展。
- `@After`和`@AfterAll`生命周期方法中抛出的异常现在优先于测试或以前生命周期方法中违反的假设（即`TestAbortedExceptions`）。
- 继承的`@Test`方法的`MethodSource`现在可以正确引用当前的测试类，而不是声明`@Test`方法的类或接口。这允许诸如Maven Surefire之类的构建工具在执行单个测试类或基于过滤器的特定测试类时正确地包含继承的`@Test`方法 -- 例如，通过`mvn test -Dtest=SubclassTests`）。
- 如果测试类中的嵌套类或嵌套接口具有无效的类文件，则测试发现不会提前中止。
- 在测试发现阶段遇到的某些类别的错误不再导致JUnit Jupiter提前中止整个发现过程。
	- 现在记录了这样的错误，从而使JUnit Jupiter能够发现并执行尽可能多的测试，同时仍然告知用户无法正确发现的容器和测试。

######  自5.3 RC1以来的错误修复
- `junit-jupiter-params`JAR文件缺少`getAs<Class>(index)` Kotlin扩展方法所需的Kotlin元数据。底层打包问题已经解决，扩展方法已经重命名为`get<Class>(index)`，吻合了5.3.0-M1最初的设计。
- `TestInstanceFactory`扩展在`@Nested`测试类层次结构中正确地被继承。

######  自5.3 RC1以来的重大变化
- `@Nested`测试类不能再覆盖在封闭类上注册的`TestInstanceFactory`扩展。这与在传统测试类层次结构中注册的`TestInstanceFactory`扩展的行为一致。

###### 新特性与改进
- 对并行测试执行的实验性支持。默认情况下，测试仍按顺序执行，但可以使用配置参数启用并行性（有关示例和配置选项，请参阅 [用户指南]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#317-并行执行'}}) ）。
- 如果提供的lambda表达式或方法引用返回结果而不是抛出异常，则`Assertion`的新`assertThrows`方法会提供更具体的失败消息。
- 如果提供给断言的对象的`toString()`实现引发异常，则为失败的断言生成详细的失败消息不再失败。相反，具有断开的`toString()`实现的对象将通过基于对象的完全限定类名和系统哈希码的默认String表示来引用，由`@`符号分隔。
- 尽管非常不鼓励，但现在可以扩展 {{Assertions}} 和 {{Assumptions}} 类以用于特殊用例。
- `TestReporter`中的新的`publishEntry(String)`方法，可以更轻松地仅基于值发布报表条目，而无需指定键（如现有的`publishEntry()`变量所需）。
- `@EnabledOnOs`和`@DisabledOnOs`中对IBM AIX操作系统的新支持。
- 动态测试API已从实验状态升级到维护状态。这会影响`@TestFactory`注解以及`org.junit.jupiter.api`包中的`DynamicTest`，`DynamicContainer`和`DynamicNode`类型。
- 在创建动态容器或测试时提供自定义测试源`URI`的新支持。
	- 如果`URI`使用类路径方案，则动态容器或动态测试的自定义测试源`URI`将注册为`ClasspathResourceSource`；否则，这样的`URI`将根据需要注册为`FileSource`，`DirectorySource`或`UriSource`。
	- 有关详细信息，请参阅`DynamicTestiner`中的新工厂方法`dynamicContainer(String, URI, ...)`和`DynamicTest`中的`dynamicTest(String，URI，Executable)`。
- `@ParameterizedTest`中`name`属性的新`{displayName}`占位符，允许开发人员在自定义显示名称中包含`@ParameterizedTest`方法的显示名称，以便调用该参数化测试。
	- 这与`@RepeatedTest`的现有`{displayName}`占位符支持一致。
- 如果参数化测试的参数的`toString()`实现引发异常，则生成`@ParameterizedTest`的显示名称不再失败。相反，具有断开的`toString()`实现的对象将通过基于对象的完全限定类名和系统哈希码的默认字符串表示来引用，由`@`符号分隔。
- 参数化测试的隐式参数转换现在可以将诸如`"java.lang.Integer"`，`"long"`和`"byte []"`之类的字符串转换为`Class`实例。
- `Arguments`接口中的新`arguments()`静态工厂方法，用作`Arguments.of()`的别名。 `arguments()`可以通过`import static`来使用。
- 新的`get<Class>(index)` Kotlin扩展方法，使`ArgumentsAccessor`更友好地使用Kotlin。
- 分别使用`@ConvertWith`和`@AggregateWith`注册的`ArgumentConverters`和`ArgumentsAggregators`，现在每个`@ParameterizedTest`仅实例化一次，而不是每次调用一次。
- 执行参数化测试的性能改进，特别是当方法声明了多个参数时。
- 新的`TestInstanceFactory`扩展API，支持自定义创建测试类实例。
- 有关详细信息，请参阅用户指南中的 [测试实例工厂]({{ site.url }}{{ '/junit5/' }}{{ page.version }}{{ '/user-guide-cn/#54-测试实例工厂'}})。


### JUnit Vintage

###### Bug修复
继承的`@Test`方法的`MethodSource`现在可以正确引用当前的测试类，而不是声明`@Test`方法的类或接口。这允许诸如Maven Surefire之类的构建工具在执行单个测试类或基于过滤器的特定测试类时正确地包含继承的`@Test`方法 -- 例如，通过`mvn test -Dtest=SubclassTests`。

###### 新特性与改进
`VintageTestEngine`现在使用测试类的简单名称作为显示名称，而不是完全限定的类名。 这与`JupiterTestEngine`的行为一致。

