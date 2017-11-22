## 10. 发布记录

### 5.0.2

**发布时间**： 2017.11.12

**范围**：自5.0.1版本以来的错误修复和小的改进。

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0.2](https://github.com/junit-team/junit5/milestone/17?closed=1) 里程碑页面。


#### JUnit Platform

##### *Bug 修复*

- 修复后，Maven Surefire对于不使用`MethodSource`的测试引擎（例如Spek）能正确地报告失败的测试。

- 修复后，当一个非零的`forkCount`与Maven Surefire一起执行时，可以正确地报告写入`System.out`或`System.err`的测试，特别是通过一个日志框架的时候。


##### *新功能和改进*

- JUnit Platform Maven Surefire提供者程序现在支持`redirectTestOutputToFile` Surefire功能。

- JUnit Platform Maven Surefire提供者程序现在会忽略通过`<includeTags/>`，`<groups/>`，`<excludeTags/>`和`<excludedGroups/>`提供的空字符串。


#### JUnit Jupiter

##### *Bug 修复*

- `@CsvSource`或`@CsvFileSource`输入行中的尾随空格不再生成空值。

- 以前，`@EnableRuleMigrationSupport`无法识别`@Rule`方法，该方法返回一个已支持的`TestRule`类型的子类型。而且，它错误地实例化了某些多次使用方法声明的规则。现在，一旦启用，它将实例化所有声明的规则（字段*和*方法），并按照JUnit 4使用的顺序来调用它们。

Previously, disabled test classes were eagerly instantiated when Lifecycle.PER_CLASS was used. Now, ExecutionCondition evaluation always takes place before test class instantiation.

- 以前，当使用`Lifecycle.PER_CLASS`时，被禁用的测试类会被迫切地实例化。现在，`ExecutionCondition`总是在测试类实例化之前就被解析。

- `unit-jupiter-migrationsupport`模块不再会错误地尝试通过`ServiceLoader`机制来注册`JupiterTestEngine`，从而允许将其用作Java 9模块路径上的模块。

##### *新功能和改进*

- 现在，`Assertions`类中的`assertTrue()`和`assertFalse()`的失败消息包含了关于预期和实际布尔值的详细信息。
	- 例如，调用`assertTrue(false)`生成的失败消息现在变成了`"expected:<true>but was: <false>"`，而不是空字符串。

- 如果参数化测试没有消费通过参数源提供给它的所有参数，那么未使用的参数将不再被包含在显示名称中。

#### JUnit Vintage
没有变化。


### 5.0.1

**发布时间**： 2017.10.03

**范围**：修复了5.0.0版的错误

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0.1](https://github.com/junit-team/junit5/milestone/16?closed=1) 里程碑页面。

#### 整体改进
- 所有的包现在都有一个`optional`的依赖，而不需要在其发布的Maven POM中强制依赖*@API Guardian* JAR包。


#### JUnit Platform
没有变化。

#### JUnit Jupiter

##### *Bug 修复*
- 如果测试类中未声明JUnit 4 `ExpectedException`规则，`junit-jupiter-migrationsupport`模块中的`ExpectedExceptionSupport`不会再吃掉异常。
	- 因此，现在可以使用`@EnableRuleMigrationSupport`和`ExpectedExceptionSupport`，而不用声明`ExpectedException`规则。


#### JUnit Vintage

##### *Bug 修复*
- `PackageNameFilters`现在应用于通过`ClassSelector`，`MethodSelector`或`UniqueIdSelector`选择的测试。



### 5.0.0

**发布时间**： 2017.09.10

**范围**：首个通用版本

关于此版本所有已关闭的问题和请求的完整列表，请参阅GitHub上的JUnit仓库中的 [5.0 GA](https://github.com/junit-team/junit5/milestone/10?closed=1) 里程碑页面。


#### JUnit Platform

##### *Bug修复*
- `AbstractTestDescriptor`中的`removeFromHierarchy()`实现现在也清除了所有子级的父级关系。

#### 启用和彻底改变
- `@API`注释已经从`junit-platform-commons`项目中删除，并重新定位到GitHub上一个名为 [@API Guardian](https://github.com/apiguardian-team/apiguardian) 的独立新项目。
- 标签不再允许包含以下任何保留字符。
	- `,`, `(`, `)`, `&`, `|`, `!`
- `FilePosition`的构造函数已被替换为一个名为`from(int，int)`的静态工厂方法。
- 一个`FilePosition`现在全完可以通过新的`from(int)`静态工厂方法从一个行号进行构建。

#### JUnit Jupiter

##### *Bug修复*
- 如果测试类中未声明JUnit 4 `ExpectedException`规则，`junit-jupiter-migrationsupport`模块中的`ExpectedExceptionSupport`不会再吃掉异常。
	- 因此，现在可以使用`@EnableRuleMigrationSupport`和`ExpectedExceptionSupport`，而不用声明`ExpectedException`规则。


#### JUnit Vintage
没有变化。