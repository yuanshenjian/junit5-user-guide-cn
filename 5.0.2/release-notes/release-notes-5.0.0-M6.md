### 5.0.0-M6
**发布时间**： 2017.07.18

**范围**：JUnit 5的第六次发布，主要解决Java 9的兼容性、验证（例如：Tag语法规则）和修复bug。

> ⚠️ 这是一次里程碑式的发布，包含重大更改。如果想在捆绑了旧版里程碑版本的IntelliJ IDEA中使用此版本，请参阅上面的 [说明](#411-intellij-idea)。

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的  [5.0 M6](https://github.com/junit-team/junit5/milestone/11?closed=1) 里程碑页面。
 
#### 兼容Java 9
JUnit 5的运行时环境的主要目标是Java 8，因此，JUnit 5的发布版本不能描述Java 9的编译模块。然而，由于 [第5次发布](http://junit.org/junit5/docs/current/user-guide/release-notes-5.0.0-RC3.md#release-notes-5.0.0-m5) 的每个发布包在其JAR清单中声明了稳定的`Automatic-Module-Name`，这使得可以在测试模块中包含著名的JUnit模块名称，如下所示。

```java
module foo.bar {
  requires org.junit.jupiter.api;
}
```

通常在类路径上就可以运行测试，这方面Java 8和Java 9没什么区别。只要支持JUnit平台，大多数的命令行工具和IDE都可以*开箱即用* JUnit 5。如果你选择的开发工具不支持JUnit平台，可以使用`ConsoleLauncher`，甚至可以使用可执行的`junit-platform-console-standalone`一站式jar包。

要在模块路径中运行JUnit Jupiter测试，可以通过一个Java 9兼容的构建工具 [pro](https://github.com/forax/pro) 来实现。

**pro** 支持黑盒测试和白盒测试。前者用来测试模块表面，只能访问应用程序模块的导出位。后者使用合并的模块描述符技术，允许访问`protected`和包私有类型以及非导出包。

测试模块的例子可以查看 [pro的GitHub仓库](https://github.com/forax/pro/tree/master/src/test/java)：`integration.pro`是一个黑盒测试模块；但是`com.github.forax.pro.api`和`com.github.forax.pro.helper`是白盒测试模块。

#### JUnit Platform

###### Bug修复
* 为了删除前导和尾随的空白，所有的Tag都被*修剪* 了。这适用于任何直接通过`TagFilter.includeTags()`和`TagFilter.excludeTags()`或者间接通过`@IncludeTags`,`@ExcludeTags`，JUnit Platform控制台启动器，JUnit Platform的Gradle插件和JUnit平台的Maven Surefire provider所提供的任何Tag。

###### 弃用和彻底改变
* 所有的Tag现在都要满足以下语法规则：
	* 标签不能是`null`或者*空白*。
	* *修剪* 的标签不能包含空白。
	* *修剪* 的标签不能包含ISO控制字符。
* 如果提供的Tag在语法上是无效的，`TagFilter.includeTags()`,`TagFilter.excludeTags()`和`TestTag.create()`工厂方法现在会抛出一个`PreconditionViolationException`异常。
* `EngineDiscoveryRequest`的`getDiscoveryFiltersByType`方法已经被改名为`getFiltersByType`。
* `UniqueId`的`getSegments()`方法现在返回一个不可变的列表。
* `AbstractTestDescriptor`的方法`setSource`被删除，替代它的是一个包含`source`变量的构造函数。

###### 新特性与改进
* 为语法有效的Tag增加了一个新的检查方法`TestTag.isValid(String)`
* 如果应用的元素是类，`AnnotationSupport`的方法`findAnnotation()`现在可以被一个类实现的接口中搜索
* `org.junit.platform.commons.util.ReflectionUtils`的下面这些方法现在通过`org.junit.platform.commons.support.ReflectionSupport`向外暴露：
	* `public static Optional<Class<?>> loadClass(String name)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, String parameterTypeNames)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, Class<?>…​ parameterTypes)`
	* `public static <T> T newInstance(Class<T> clazz, Object…​ args)`
	* `public static Object invokeMethod(Method method, Object target, Object…​ args)`
	* `public static List<Class<?>> findNestedClasses(Class<?> clazz, Predicate<Class<?>> predicate)`

#### JUnit Jupiter

###### Bug修复
* 为了删除前导和尾随的空白，所有通过`@Tag`声明的标记都被修剪了。
* 现在支持所有基本数组类型（从`boolean[]`到`short[]`）作为参数化测试的静态参数的返回类型。

###### 弃用和彻底改变
* `@Test`和有生命周期的方法现在都强制返回`void`类型。

###### 新特性与改进
* 现在可以使用`Assertions `中的所有`fail(...)`方法来实现单语句lambda表达式，从而避免需要实现具有显式返回值的代码块。
* `ExtensionContext`中的新方法`getRoot()`能够容易的获得最高的，*root* 扩展上下文。
* `ExtensionContext` API中的新方法`getRequiredTestClass()`、`getRequiredTestInstance()`和`getRequiredTestMethod()`更加便捷，可用于在需要这些元素的用例中检索测试类，测试实例和测试方法。
* 类似`@TestInstance`和`@Disabled`的类级注解现在可以被声明与测试接口上（也称为测试特征）。
* 如果针对单个方法解析了多个`TestDescriptor`，则现在会记录一条警告。 这有助于调试由多个竞争注解（例如`@Test`，`@RepeatedTest`，`@ParameterizedTest`，`@TestFactory`等）同时注解的方法导致的错误。
* 包含无效标记语法的`@Tag`声明现在将被记录为警告，但会被有效地忽略掉。

#### JUnit Vintage

###### Bug修复

* 与`@Unroll`一起使用时，添加了对`Runners`的支持，这些`Runners`报告不属于`Description`树的测试事件，例如`Spock`的`Sputnik`。 以前，这样的测试根本没有报告; 现在它们被作为动态测试而报告。
