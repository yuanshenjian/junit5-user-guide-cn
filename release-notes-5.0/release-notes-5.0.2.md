### 5.0.2

**发布时间**： 2017.11.12

**范围**：自5.0.1版本以来的错误修复和小的改进。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.2](https://github.com/junit-team/junit5/milestone/17?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复

- 修复后，Maven Surefire对于不使用`MethodSource`的测试引擎（例如Spek）能正确地报告失败的测试。

- 修复后，当一个非零的`forkCount`与Maven Surefire一起执行时，可以正确地报告写入`System.out`或`System.err`的测试，特别是通过一个日志框架的时候。


###### 新特性与改进

- JUnit Platform Maven Surefire provider现在支持`redirectTestOutputToFile` Surefire功能。

- JUnit Platform Maven Surefire provider现在会忽略通过`<includeTags/>`，`<groups/>`，`<excludeTags/>`和`<excludedGroups/>`提供的空字符串。


#### JUnit Jupiter

###### Bug修复

- `@CsvSource`或`@CsvFileSource`输入行中的尾随空格不再生成空值。

- 以前，`@EnableRuleMigrationSupport`无法识别`@Rule`方法，该方法返回一个已支持的`TestRule`类型的子类型。而且，它错误地实例化了某些多次使用方法声明的`Rule`。现在，一旦启用，它将实例化所有声明的`Rule`（字段*和*方法），并按照JUnit 4使用的顺序来调用它们。

- 以前，当使用`Lifecycle.PER_CLASS`时，被禁用的测试类会被迫切地实例化。现在，`ExecutionCondition`总是在测试类实例化之前就被解析。

- `unit-jupiter-migrationsupport`模块不再会错误地尝试通过`ServiceLoader`机制来注册`JupiterTestEngine`，从而允许将其用作Java 9模块路径上的模块。

###### 新特性与改进

- 现在，`Assertions`类中的`assertTrue()`和`assertFalse()`的失败消息包含了关于预期和实际布尔值的详细信息。
	- 例如，调用`assertTrue(false)`生成的失败消息现在变成了`"expected:<true>but was: <false>"`，而不是空字符串。

- 如果参数化测试没有消费通过参数源提供给它的所有参数，那么未使用的参数将不再被包含在显示名称中。

#### JUnit Vintage
没有变化。
