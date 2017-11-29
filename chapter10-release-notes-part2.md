## 10. 发布记录（Part2）

### 5.0.0-M6
**发布时间**： 2017.07.18

**范围**：JUnit 5的第六次发布，主要解决Java 9的兼容性、验证（例如：tag语法规则）和修复bug。

 > ⚠️ 这是一次里程碑式的发布，并且包含了一些突破性变化。如果想在Intellij IDEA中使用更早版本的发布，请参考以上 [说明](http://junit.org/junit5/docs/current/user-guide/#running-tests-ide-intellij-idea)。
 
 若要查看这个发布全部的已解决问题和pull request，可以查看GitHub上的 [5.0 M6](https://github.com/junit-team/junit5/milestone/11?closed=1) JUnit仓库。
 
#### 兼容Java 9
JUnit 5的运行时环境的主要目标是Java 8，因此，JUnit 5的发布版本不能描述Java 9的编译模块。然而，由于 [第5次发布](http://junit.org/junit5/docs/current/user-guide/#release-notes-5.0.0-m5) 的每个发布包在其JAR清单中声明了稳定的`Automatic-Module-Name`，这使得可以在测试模块中签名包含著名的JUnit模块名称，如下所示：

```java
module foo.bar {
  requires org.junit.jupiter.api;
}
```

通常在类路径上就可以运行测试，这方面Java 8和Java 9没什么区别。只要支持JUnit平台，大多数的命令行工具和IDE都可以*开箱即用*JUnit 5。如果开发工具不支持JUnit平台，可以使用`ConsoleLauncher`或者甚至执行`junit-platform-console-standalone`，它们都在同一个jar包中。

要在模块路径中运行JUnit Jupiter测试，可以通过一个Java 9兼容的构建工具 [pro](https://github.com/forax/pro) 来实现。

**pro**能够支持黑盒测试和白盒测试。前者用来测试模块表面，只对输出应用模块有权限。后者使用合并的模块描述技术，对`protedted`，私有的包和非输出的包都有权限。

测试模块的例子可以查看 [pro的GitHub仓库](https://github.com/forax/pro/tree/master/src/test/java)：`integration.pro`是一个黑盒测试模块；但是`com.github.forax.pro.api`和`com.github.forax.pro.helper`是白盒测试模块。

#### JUnit Platform
##### 修复Bug
* 为了删除前置的和后置的空白，所有的标签都被*修剪*了。可以应用于以下几个地方，任何直接通过`TagFilter.includeTags()`和`TagFilter.excludeTags()`或者间接通过`@IncludeTags`,`@ExcludeTags`应用的`Launcher`，JUnit平台的控制台启动器，JUnit平台的Gradle插件和JUnit平台的Maven Surefire provider。

##### 弃用和突破性改变
* 所有的标签现在都要满足以下语法规则：
	* 标签不能是`null`或者*空白*
	* *修剪*的标签不能包含空白
	* *修剪*的标签不能包含ISO控制字符
* 如果一个被应用的标签语法上无效，`TagFilter.includeTags()`,`TagFilter.excludeTags()`和`TestTag.create()`工厂方法现在会抛出一个`PreconditionViolationException`异常。
* `EngineDiscoveryRequest`的方法`getDiscoveryFiltersByType`已经被改名为`getFiltersByType`。
* `UniqueId`的方法`getSegments()`现在返回一个不变的列表。
* `AbstractTestDescriptor`的方法`setSource`被删除，且替代它的是一个包含`source`变量的构造函数。

##### 新特性和改进
* 为语法有效的标签增加了一个新的检查方法`TestTag.isValid(String)`
* 如果应用的元素是类，`AnnotationSupport`的方法`findAnnotation()`现在可以被一个类实现的接口中搜索
* `org.junit.platform.commons.util.ReflectionUtils`的下列方法现在通过`org.junit.platform.commons.support.ReflectionSupport`向外暴露：
	* `public static Optional<Class<?>> loadClass(String name)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, String parameterTypeNames)`
	* `public static Optional<Method> findMethod(Class<?> clazz, String methodName, Class<?>…​ parameterTypes)`
	* `public static <T> T newInstance(Class<T> clazz, Object…​ args)`
	* `public static Object invokeMethod(Method method, Object target, Object…​ args)`
	* `public static List<Class<?>> findNestedClasses(Class<?> clazz, Predicate<Class<?>> predicate)`

#### JUnit Jupiter
##### 修复Bug
* 为了去掉前置和后置的空白，所有通过`@Tag`申明的标签都被修剪了。
* 参数化测试中所有的原始数组类型（从`boolean[]`到`short[]`）现在都被作为返回类型为静态的方法支持。

##### 弃用和突破性改变
* `@Test`和有生命周期的方法现在都强制返回`void`类型。

##### 新特性和改进
* `断言`中所有的`fail(…​)
* `方法现在都可以用来实现single的lambda表达式，因此避免了用一个明确的返回类型来实现代码块。
* `ExtensionContext`中的新方法`getRoot()`能够容易的获得最高的，*root*扩展内容的权限。
* `ExtensionContext` API中的新方法`getRequiredTestClass()`,`getRequiredTestInstance()`和`getRequiredTestMethod()`，更加便捷，可用于在需要对应元素的用例中获取测试类、实例以及测试方法。
* 类似`@TestInstance`和`@Disabled`的类级注解现在可以被声明与测试接口上（如测试特性一样）。
* 如果针对单个方法解析了多个`TestDescriptor`，则现在会记录一条警告。 这有助于调试由多个竞争注释（例如`@Test`，`@RepeatedTest`，`@ParameterizedTest`，`@TestFactory`等）同时注释的方法导致的错误。
* 包含无效标记语法的`@Tag`声明现在将被记录为警告，而不是被有效忽略。

#### JUnit Vintage
##### 修复Bug

* 与`@Unroll`一起使用时，添加了对Runners的支持，这些Runners报告不属于`Description`树的测试事件，例如`Spock`的`Sputnik`。 以前，这样的测试根本没有报告; 现在它们被作为动态测试而报告。


### 5.0.0-M5
**发布时间**： 2017.07.04

**范围**：


### 5.0.0-M4
**发布时间**： 2017.04.01

**范围**：


### 5.0.0-M3
**发布时间**： 2016.11.30

**范围**：

### 5.0.0-M2
**发布时间**： 2016.07.23

**范围**：

### 5.0.0-M1
**发布时间**： 2016.07.07

**范围**：


### 5.0.0-ALPHA
**发布时间**： 2016.02.01

**范围**：


