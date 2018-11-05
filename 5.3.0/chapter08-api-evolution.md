## 8. API演变
JUnit 5的主要目标之一是提高维护者发展演进JUnit的能力，尽管它在许多项目被使用。在JUnit 4中，起初作为内部构造而被添加的大量内容只能被外部扩展编写器和工具构建器使用。这就使得改变JUnit 4异常困难，甚至有时是不可能的。

这就是为什么JUnit 5为所有公开的接口、类和方法引入了一个明确的生命周期。


### 8.1. API 版本和状态
每个已发布的artifact都有一个版本号`<major>.<minor>.<patch>`，所有公开的接口、类和方法都使用了 {{API_Guardian}}项目中的 {{API}} 注解进行标注。`@API`注解的`status`属性可以被赋予下面表格中的值。

| 状态 | 描述 |
|:---|:---|
| INTERNAL | 只能被JUnit自身使用，可能会被删除，但不事先另行通知。 |
| DEPRECATED | 不应该再使用；可能会在下一个小版本中消失。 |
| EXPERIMENTAL | 用于我们正在收集反馈的新的试验性功能。谨慎使用这个元素；它可能会在未来被提升为`MAINTAINED`或`STABLE`，但也可能在没有事先通知的情况下被移除，即使在一个补丁中。 |
| MAINTAINED | 用于*至少* 在当前主要版本的下一个次要版本中不会以反向不兼容的方式更改的功能。如果计划删除，则会首先将其降为`DEPRECATED`。 |
| STABLE | 用于在当前主版本（5. *）中不会以反向不兼容的方式更改的功能。 |

如果`@API`注解出现在某个类型上，则认为它也适用于该类型的所有公共成员。一个成员可以声明一个稳定性更低的`status`值。

### 8.2. 试验性API
下表列出了那些通过`@API(status = EXPERIMENTAL)`来标记为*试验性的*的API。使用这样的API时应该谨慎。

| 包名 | 类型 | 起始版本 |
|:---|:---|:---|
|org.junit.jupiter.api|AssertionsKt*（类）*| 5.1 |
|org.junit.jupiter.api.condition|DisabledIf*（注解）*| 5.1 |
|org.junit.jupiter.api.condition|EnabledIf*（注解）*| 5.1 |
|org.junit.jupiter.api.extension|ScriptEvaluationException*（类）*|5.1|
|org.junit.jupiter.api.extension|TestInstanceFactory *(接口)*|5.3|
|org.junit.jupiter.api.extension|TestInstanceFactoryContext *(接口)*|5.3|
|org.junit.jupiter.api.extension | TestInstantiationException *(类)* | 5.3| 
|org.junit.jupiter.api.parallel|Execution *(注解)*|5.3|
|org.junit.jupiter.api.parallel|ExecutionMode *(枚举)*|5.3|
|org.junit.jupiter.api.parallel|ResourceAccessMode *(枚举)*|5.3|
|org.junit.jupiter.api.parallel|ResourceLock *(注解)*|5.3|
|org.junit.jupiter.api.parallel|ResourceLocks *(注解)*|5.3|
|org.junit.jupiter.api.parallel|Resources *(类)*|5.3|
|org.junit.jupiter.params|ParameterizedTest *(注解)*|5.0|
|org.junit.jupiter.params.aggregator|AggregateWith *(注解)*|5.2|
|org.junit.jupiter.params.aggregator|ArgumentAccessException *(类)*|5.2|
|org.junit.jupiter.params.aggregator|ArgumentsAccessor *(接口)*|5.2|
|org.junit.jupiter.params.aggregator|ArgumentsAggregationException *(类)*|5.2|
|org.junit.jupiter.params.aggregator|ArgumentsAggregator *(接口)*|5.2|
|org.junit.jupiter.params.converter|ArgumentConversionException *(类)*|5.0|
|org.junit.jupiter.params.converter|ArgumentConverter *(接口)*|5.0|
|org.junit.jupiter.params.converter|ConvertWith *(接口)*|5.0|
|org.junit.jupiter.params.converter|JavaTimeConversionPattern *(接口)*|5.0|
|org.junit.jupiter.params.converter|SimpleArgumentConverter *(类)*|5.0|
|org.junit.jupiter.params.provider|Arguments *(接口)*|5.0|
|org.junit.jupiter.params.provider|ArgumentsProvider *(接口)*|5.0|
|org.junit.jupiter.params.provider|ArgumentsSource *(接口)*|5.0|
|org.junit.jupiter.params.provider|ArgumentsSources *(接口)*|5.0|
|org.junit.jupiter.params.provider|CsvFileSource *(接口)*|5.0|
|org.junit.jupiter.params.provider|CsvParsingException *(类)*|5.3|
|org.junit.jupiter.params.provider|CsvSource *(接口)*|5.0|
|org.junit.jupiter.params.provider|EnumSource *(接口)*|5.0|
|org.junit.jupiter.params.provider|MethodSource *(接口)*|5.0|
|org.junit.jupiter.params.provider|ValueSource *(接口)*|5.0|
|org.junit.jupiter.params.support|AnnotationConsumer*（接口）*|5.0|
|org.junit.platform.engine.support.config|PrefixedConfigurationParameters *(类)*|1.3|
|org.junit.platform.engine.support.hierarchical|DefaultParallelExecutionConfigurationStrategy *(枚举)*|1.3|
|org.junit.platform.engine.support.hierarchical|ExclusiveResource *(类)*|1.3|
|org.junit.platform.engine.support.hierarchical|ForkJoinPoolHierarchicalTestExecutorService *(类)*|1.3|
|org.junit.platform.engine.support.hierarchical|HierarchicalTestExecutorService *(接口)*|1.3|
|org.junit.platform.engine.support.hierarchical|ExecutionMode *(枚举)*|1.3|
|org.junit.platform.engine.support.hierarchical|ParallelExecutionConfiguration *(接口)*|1.3|
|org.junit.platform.engine.support.hierarchical|ParallelExecutionConfigurationStrategy *(接口)*|1.3|
|org.junit.platform.engine.support.hierarchical|ResourceLock *(接口)*|1.3|
|org.junit.platform.engine.support.hierarchical|SameThreadHierarchicalTestExecutorService *(类)*|1.3|


### 8.3. 弃用的API
下表列出了那些通过`@API(status = DEPRECATED)`来标记为*弃用*的API。由于这些API可能在接下来的版本中会被移除，所以你应该尽可能不要使用它们。

| 包名 | 类型 | 起始版本 |
|:---|:---|:---|
|org.junit.platform.engine.support.hierarchical|SingleTestExecutor *(类)*|1.2|
|org.junit.platform.surefire.provider|JUnitPlatformProvider *(类)*|1.3|


### 8.4. @API工具支持

{{API_Guardian}} 项目计划为使用 {{API}} 注解的API的发布者和消费者提供工具支持。例如，工具支持可能会提供一种方法来检查是否按照`@API`注解声明来使用JUnit API。
