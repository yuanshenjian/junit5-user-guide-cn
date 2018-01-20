### 5.0.3

**发布时间**： 2018.01.15

**范围**：对`ConsoleLauncher`，Gradle插件和Maven Surefire provider的Bug修复和小的改进。


关于此版本所有*已关闭的* 问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0.3](https://github.com/junit-team/junit5/milestone/21?closed=1) 里程碑页面。

#### 整体改进

- 在5.0.1中，所有工件被更改为在其发布的 Maven POMs中的*@API Guardian* JAR中有一个可选的，而不是必须的依赖项。然而，虽然Java编译器应该忽略缺少的注解类型，但很多用户都报告说，编译测试时，如果在类路径中没有*@API Guardian* JAR，会导致`javac`发出的警告，如下所示：

```
warning: unknown enum constant Status.STABLE 
reason: class file for org.apiguardian.api.API$Status not found
```

为了避免混淆，JUnit团队决定再次*强制*依赖于*@API Guardian* JAR。

#### JUnit Platform

###### Bug修复

- 除非出现错误，否则在选择详细信息模式`NONE`时，汇总表不再通过`ConsoleLauncher `和Gradle插件进行打印。

- 当失败测试的异常消息包含XML CDATA结束标记`]]>`时，由`ConsoleLauncher`和Gradle插件生成的XML报告不再无效。

- `ConsoleLauncher`，Gradle插件，和Maven Surefire provider现在尝试在XML报告中的`<testcase />`元素的`classname`属性中写入有效的类名称。此外，动态测试和测试模板调用的`name`属性（如重复测试和参数化测试）现在后缀为调用的索引，因此它们可以通过报告工具区分。

- 当把`<testcase />`元素的`name`属性写入XML报告时， Maven Surefire提供程序现在包含方法参数类型。但是，由于 Maven Surefire的限制，而不是`methodName（type）`，它们被写为`methodName（type）`。

- 现在认为，在搜索类层次结构中的给定元注解时，被隐式地继承的非继承组合注解是用给定的`@Inherited`注解元注解的。

###### 新特性与改进

- JUnit Platform Maven Surefire provider现在在一次测试运行中运行所有指定的测试，即所有注册的`TestExecutionListeners`将收到一个单一的`TestPlan`。以前，为每个测试类别发现并执行单独的`TestPlan`。

- `Consolelauncher`和Gradle插件的新`SUMMARY`细节模式，在测试计划执行结束时打印成功和失败计数表。这个新的模式类似于之前的`NONE`详细模式的行为。

- Maven Surefire provider现在支持测试参数，它告诉Surefire只执行测试类或方法的一个子集，例如：通过在命令行指定`-Dtest=…​`（详情参考[Surefire documentation](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#test)）


#### JUnit Jupiter

###### Bug修复

- `@Tag`和`@Tags`注解现在在测试类层次结构中继承。

- 由于JUnit 平台的`AnnotationUtils`类的变化，现在被认为在搜索类层次结构中的给定元注解时被隐式地继承了非赋值注解，这些注解是用给定的`@Inherited`注解的元注解。
	- 例如，即使*构成的注解*未声明为`@Inherited`，现在也会在父类上声明的自定义*组合注解*上发现`@Testinstance `等`@Inherited`注解。


#### JUnit Vintage
没有变化。