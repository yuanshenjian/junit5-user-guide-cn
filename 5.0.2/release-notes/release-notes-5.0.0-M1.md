### 5.0.0-M1
**发布时间**： 2016.07.07

**范围**： JUnit 5的第一个里程碑发布

##### 变更摘要
下面首先给出整体改动。而有关Platform、Jupiter，以及Vintage的细节变动，可以查阅后面的专项信息。对于本次发布相关的commit信息，可以通过查看 [5.0 M1](https://github.com/junit-team/junit5/milestone/2?closed=1)里程碑页面在GitHub上的JUnit代码库了解。

* 在已发布的JAR清单中，包含了额外的元数据，例如`Create-By`、`Built-By`、`Build-Date`、`Build-Time`、`Build-Revision`、`Implementation-Title`、`Implementation-Version`、`Implementation-Vendor`等。
* 当前发布的工件中，包含了`LICENSE.md`和`META-INF`。
* 现在，JUnit参与到 [Up For Grabs](http://up-for-grabs.net/#/tags/junit) 运动中，以便为开源做贡献。
	* 请参阅GitHub上的 [up-for-grabs](https://github.com/junit-team/junit5/labels/up-for-grabs) 标签。
* 对于已发布的Artifact，其Group ID，Artifact ID，以及版本都已改变。
	* 查看 [工件迁移](#工件迁移) 以及 [依赖元数据](#21-依赖元数据)。
* 所有的基础包都被重命名了。
	* 查看 [包迁移](#包迁移)

<a id="工件迁移"></a>
*表 4. 工件迁移信息表*

|旧的Group ID|旧的Artifact ID|新的Group ID|新的Artifact ID|新的Base版本|
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

|旧基础包名|新基础包名|
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
* 现在，`ConsoleLauncher`在退出时总会返回状态码，并且已经移除了*激活退出码* 的标志位。
* `junit-platform-console`不再在`junit-platform-runner`、`junit-jupiter-engine`或`junit-vintage-engine`上定义传递依赖。
* 现在，`JUnit5`运行器已经被重命名为`JUnitPlatform`。
	* `@Packages`已经被重命名为`@SelectPackages`。
	* `@Classes`已经被重命名为`@SelectClasses`。
	* `@UniqueIds`已经被移除。
	* 引入了新的注解`@UseTechnicalNames`。
		* 详情请参阅 [显示名称与技术名称](#442-显示名称与技术名称)
* Gradle的JUnit Platform插件已经全面修订。
	* 现在，JUnit Platform的Gradle插件要求Gradle版本在2.5或更高版本。
	* Gradle中名为`junit5Test`的任务已经被重命名为`junitPlatformTest`。
	* Gradle中名为`junit5`的配置已经被重命名为`junitPlatform`。
		* `runJunit4`已经被`enableStandradTestTask`替换。
		* `version`已经被`platformVersion`替换。
	* 详情请查阅 [Gralde](#421-gradle)。
* XML测试报告已被全面修正。
	* 现在，XML报告包含换行。
	* 现在，在JUnit Platform中明确定义的，但是在标准XML文档属性中不包含的属性元素，会被包含在`<system-out>`元素中的`CDATA`块中。
	* 现在，测试报告中会用全限定类名和真正的方法名来取代先前的显示名称。
* 现在，`Testidentifier`中的唯一标识符是`String`类型。
* 现在，`TestSource`是有着专用层级结构的接口，它由`CompositeTestSource`、 `JavaSource`、`JavaPackageSource`、`JavaClassSource`、`JavaMethodSource`、`UriSource`、`FileSystemSource`、`DirectorySource`和`FileSource`构成。
* 新增了`DiscoverySelectors`类，用于集中管理所有的*select* 方法，同时将所有`DiscoverySelector `工厂方法转移到新增类中。
* `Test.filter()`已经被重命名为`Filter.apply()`。
* `TestTag.of()`已被重命名为`TestTag.create()`。
* `TestDiscoveryRequestBuilder`已被重命名为`LauncherDiscoveryRequest`。
* 现在，`LauncherDiscoveryRequest`是不可变的。
* `TestDescriptor.allDescendants()`已经被重命名为`TestDescriptor.getAllDescendants()`。
* `TestEngine#discover(EngineDiscoveryRequest)`已经被`TestEngine#discover(EngineDiscoveryRequest, UniqueId)`替换。
* 引入了`ConfigurationParameters`，`Laucher`可以通过`EngineDiscoveryRequest`和`ExecutionRequest`将配置参数传递给引擎。
* `Container`和`Leaf`抽象类已经从`HierarchicalTestEngine`中移除。
* `getName()`方法已经被从`TestIdentifier`和`TestDescriptor`中移除，取而代之的是通过`TestSource`获取一个实现类的具体名称。
* 现在，测试引擎在本质上是完全动态的。也就是说，`TestEngine`无需在*发现阶段* 创建`TestDescriptor`条目；现在，TestEngine现在可以选择在执行阶段注册容器和测试。
* 包括和排除对引擎和Tag的支持已经完全修改。
	* 引擎和Tag不再是`required`，而是`included`。
	* 现在，`ConsoleLauncher`支持以下选项：`t`/`include-tag`、`T`/`exclude-tag`、`e`/`include-engine`、`E`/`exclude-engine`。
	* 现在，Gradle插件支持嵌套在`include`和`exclude`实体中的`engines`和`tags`配置块。
	* 现在，`EngineFilter`支持`includeEngines()`和`exludeEngines()`工厂方法。
	* 现在，`JUnitPlatform`运行器支持`@IncludeTags`、`@ExcludeTags`、`@IncludeEngines`以及`@ExcludeEngines`。

#### JUnit Jupiter
* `junit5`引擎ID已经被重命名为`junit-jupiter`。
* `Junit5TestEngine`已经被重命名为`JupiterTestEngine`。
* 现在，`Assertions`提供了以下支持：
	* `assertEquals()`方法可以对基本类型使用
	* `assertEquals()`方法可以对包含增量的double类型和float类型的值使用
	* `assertArrayEquals()`
	* 现在期望值与实际值都被提供给`AssertionFailedError`。
* [动态测试](#315-动态测试)：现在，测试可以通过lambda表达式在运行时被动态注册。
* 现在，`TestInfo`通过`getTags()`方法提供了获取Tag的方法。
* 如果`@Test`、`@BeforeEach`或*before* 回调引发异常，现在将调用`@AfterEach`方法和*after* 回调函数。
* 现在，`@AfterAll`注解所标注的方法以及*after all* 回调一定会被调用。
* 现在，可以在测试类层次结构中的超类以及接口中发现可重复的注解，例如`@ExtendWith`和`@Tag`。
* 现在，在测试类或接口的层级结构中，扩展将会被*自上而下* 地注册。
* 现在，测试和容器的 [执行条件可以被禁用](#531-禁用条件)。
* `InstancePostProcessor`已被重命名为`TestInstancePostProcessor`。
	* 现在，`TestInstancePostProcessor`实现正确地应用在`@Nested`测试类层次结构中。 
* `MethodParameterResolver`已被重命名为`ParameterResolver`。
	* 现在，`ParameterResolver`API是基于`java.lang.reflect.Executable`的，因此可以被用来解析方法*和构造器* 的参数。
	* 新的`ParameterContext`用来作为参数传递给`ParameterResolver`扩展的`supports()`和`resolve()`方法。
	* 现在，`ParameterResolver`扩展支持基础类型的解析。
* `ExtensionPointRegistry`和`ExtensionRegistrar`已经被移除，现在通过`@ExtendWith`注解完成声明式注册。
* `BeforeAllExtensionPoint`已经被重命名为`BeforeAllCallback`。
* `AfterAllExtensionPoint`已经被重命名为`AfterAllCallback`。
* `BeforeEachExtensionPoint`已经被重命名为`BeforeEachCallback`。
* `BeforeAllExtensionPoint`已经被重命名为`BeforeAllCallback`。
* 新增了`BeforeTestExecutionCallback`与`AfterTestExecutionCallback`扩展API。
* `ExceptionHandlerExtensionPoint`已经被重命名为`TestExecutionExceptionHandler`。
* 现在，测试异常通过`TestExtensionContext`被提供给扩展。
* `ExtensionContext.Store`现在支持许多方法的类型安全变体。
* 现在，`ExtensionContext.getElement()`方法返回`Optional`类型。
* `Namespace.of()`已经被重命名为`Namespace.create()`。
* `TestInfo`和`ExtensionContext`新增了`getTestClass()`和`getTestMethod()`方法。
* 移除了`TestInfo`和`ExtensionContext`中的`getName()`方法，现在通过当前的测试类或测试方法来获取上下文特定的名称。

#### JUnit Vintage
* `junit4`引擎ID已经被重命名为`junit-vintage`。
* `Junit4TestEngine`已经被重命名为`VintageTestEngine`。

