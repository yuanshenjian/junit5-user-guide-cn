### 5.0.1

**发布时间**： 2017.10.03

**范围**：修复了5.0.0版的错误

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.1](https://github.com/junit-team/junit5/milestone/16?closed=1) 里程碑页面。

#### 整体改进
- 所有的artifact现在都有一个`optional`的依赖，而不需要在其发布的Maven POM中强制依赖*@API Guardian* JAR包。

#### JUnit Platform
没有变化。

#### JUnit Jupiter

###### Bug修复
- 如果测试类中未声明JUnit 4 `ExpectedException` Rule，`junit-jupiter-migrationsupport`模块中的`ExpectedExceptionSupport`不会再吃掉异常。
	- 因此，现在可以使用`@EnableRuleMigrationSupport`和`ExpectedExceptionSupport`，而不用声明`ExpectedException` Rule。


#### JUnit Vintage

###### Bug修复
- `PackageNameFilters`现在应用于通过`ClassSelector`，`MethodSelector`或`UniqueIdSelector`选择的测试。
