### 5.0.0-RC1
**发布时间**： 2017.07.30

**范围**：5.0 GA之前的错误修复和文档改进

> ⚠️ 这是一个预发行版，包含一些重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 RC1](https://github.com/junit-team/junit5/milestone/9?closed=1) 里程碑页面。


#### JUnit Platform

###### Bug修复
- 现在可以通过类，方法名，参数类型或完全限定的方法名来*选择* 未被实现类覆盖的通用接口`default`方法。这适用于`DiscoverySelectors`中的方法选择器以及`ReflectionSupport`中的`findMethod()`变体。
- 在使用`ReflectionSupport`中的`findMethods()`搜索类层次结构中的方法时，现在可以正确发现一个非重写的接口`default`方法，其方法签名被本地声明的方法重载。
- 在使用`ReflectionSupport`中的`findMethods()`搜索类层次结构中的方法时，不再发现重写的接口`default`方法。


###### 弃用和彻底改变
- 从`Launcher`类中删除了已弃用的方法`execute(LauncherDiscoveryRequest)`。已删除的方法在里程碑4中被`execute(LauncherDiscoveryRequest，TestExecutionListener ...)`方法替换。



#### JUnit Jupiter

###### Bug修复
- 与`@BeforeAll`、`@AfterAll`、`@BeforeEach`或`@AfterEach`注解标注的生命周期方法有关的配置错误在测试发现阶段不再停止执行整个测试计划。相反，现在在执行受影响的测试类时就会报告这样的错误。
- 测试计划中已经正确地包含了一个未覆盖的接口默认方法，其方法签名被本地声明的方法重载。这适用于使用了Jupiter注解（如`@Test`，`@BeforeEach`等）的`default`方法。
- 测试计划中不再包含重写的接口`default`方法。这适用于使用了Jupiter注解（如`@Test`，`@BeforeEach`等）的`default`方法。

###### 新特性与改进
- `Assertions.assertThrows()`接受自定义失败消息的新变体作为`String`或`Supplier<String>`。
- 如果测试类使用了`@TestInstance(Lifecycle.PER_CLASS)`，则`@MethodSource`引用的方法不再必须是`static`的。
- 使用Kotlin编程语言编写的测试类现在默认使用`@TestInstance(Lifecycle.PER_CLASS)`语义来执行。


#### JUnit Vintage
除了内部重构之外没有变化。
