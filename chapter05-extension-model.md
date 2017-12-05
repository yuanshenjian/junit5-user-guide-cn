## 5. 扩展模型

### 5.1. 概述

相比于JUnit4中的`Runner`、`@Rule`以及`@ClassRule`等多个扩展点，JUnit Jupiter的扩展模型由一个连贯的概念组成：`Extension`API。但是，需要注意的是 `Extension`本身也只是一个标记接口。

### 5.2. 注册扩展

JUnit Jupiter中的扩展可以通过 [`@ExtenWith`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExtendWith.html) 注解显式注册，或者通过Java的 [`ServiceLoader`机制](#522-自动扩展注册) 自动注册。

#### 5.2.1. 声明式扩展注册
开发者可以通过在测试接口、测试类、测试方法或者自定义的 [*组合注解*](#311-元注解和组合注解) 上标注`@ExtendWith(...)`并提供要注册扩展类的引用，从而以*声明式*的方式注册一个或多个扩展。

例如，要给某个测试方法注册一个自定义的`MockitoExtension`，你可以参照如下的方式标注该方法。

```java
@ExtendWith(MockitoExtension.class)
@Test
void mockTest() {
    // ...
}
```

若要为某个类或者其子类注册一个自定义的`MockitoExtension`，将注解添加到测试类上即可。

```java
@ExtendWith(MockitoExtension.class)
class MockTests {
    // ...
}
```

多个扩展类的注册可以通过如下形式完成。

```java
@ExtendWith({ FooExtension.class, BarExtension.class })
class MyTestsV1 {
    // ...
}
```

还有另外一种方式来注册多个扩展类，如下面代码所示。

```java
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MyTestsV2 {
    // ...
}
```

上述两种扩展注册方式是等价的，`MyTestV1`和`MyTestV2`都会被`FooExtension`和`BarExtension`进行扩展，且扩展顺序跟声明顺序一致。


#### 5.2.2. 自动扩展注册

除了 [声明式扩展注册](#521-声明式扩展注册) 支持使用注解外，JUnit Jupiter同样也支持通过Java的`java.util.ServiceLoader`机制来做*全局的扩展注册*。采用这种机制后自动的检测`classpath`下的第三方扩展，并自动完成注册。


>Specifically, a custom extension can be registered by supplying its fully qualified class name in a file named org.junit.jupiter.api.extension.Extension within the /META-INF/services folder in its enclosing JAR file.


另外，自定义扩展可以通过提供它的全类名来完成注册，该扩展被定义在它所在的JAR文件中的`/META-INF/services`目录下的`org.junit.jupiter.api.extension.Extension`文件里。


##### 启用自动扩展检测

自动检测是一种高级特性，因此默认情况下是关闭的。要启用它，只需要在配置文件中将 `junit.jupiter.extensions.autodetection.enabled`的*配置参数*设置为 `true`即可。该参数可以作为JVM系统属性、或作为一个传递给`Launcher`的`LauncherDiscoveryRequest`中的配置参数、再或者通过JUnit Platform配置文件（详情请参阅 [配置参数](#45-配置参数)）来提供。

例如，要启用扩展的自动检测，你可以在启动JVM时传入如下系统参数。

```sh
-Djunit.jupiter.extensions.autodetection.enabled=true
```

启用自动检测功能后，通过`ServiceLoader`机制发现的扩展将在JUnit Jupiter的全局扩展（例如对`TestInfo`，`TestReporter`等的支持）之后被添加到扩展注册表中。


#### 5.2.3. 扩展继承
扩展在测试类层次结构中以自顶向下的语义被继承。同样，在类级别注册的扩展会被方法级的扩展继承。此外，特定的扩展实现只能针对给定的扩展上下文及其父上下文进行一次注册。因此，任何尝试注册重复的扩展实现都将被忽略。

### 5.3. 条件测试执行

ExecutionCondition defines the Extension API for programmatic, conditional test execution.

[`ExecutionCondition`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExecutionCondition.html) 定为程序化的条件测试执行定义了`Extension`API。

每个容器（例如，测试类）都会对`ExecutionCondition`进行解析，从而确定是否应根据提供的`ExtensionContext`执行其包含的所有测试。类似地，`ExecutionCondition`会被每个测试解析，从而确定是否应该根据提供的`ExtensionContext`执行给定的测试方法。

当多个`ExecutionCondition`扩展被注册时，只要其中一个条件返回被禁用，容器或测试就会被禁用。由于容器或测试可能在某个条件被解析之前就因为另一个扩展而被禁用，所以没有办法保证每个条件都被解析。也就是说，条件的解析机制类似于短路 或(符号为`||`)操作。

有关具体示例，请参阅 [`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.0.2/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java) 和 [`@Disable`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Disabled.html) 的源码。

#### 5.3.1. 禁用条件
有时候，在没有明确的条件被激活的情况下运行测试套件可能更有用。例如，你可能想要运行某些即便被标注了`@Disable`的测试，从而观察这些测试是否一直是*失败的*。此时只需为`junit.jupiter.conditions.deactivate`配置参数提供一个匹配模式，以指定当前测试运行应停用哪些条件（即不被解析）。该匹配模式可以作为JVM系统属性、或作为一个传递给`Launcher`的`LauncherDiscoveryRequest`中的配置参数、再或者通过JUnit Platform配置文件（详情请参阅 [配置参数](#45-配置参数)）来提供。

例如，要停用JUnit的 `@Disable` 条件，你可以在JVM启动时传入系统参数完成：

```sh
-Djunit.jupiter.conditions.deactivate=org.junit.*DisabledCondition
```

##### 模式匹配语法

如果`junit.jupiter.conditions.deactivate`模式仅由星号（`*`）组成，则所有条件都将被禁用。 否则，该模式将用于匹配每个注册的条件的完整的类名（*FQCN*）。 模式中的点（`.`）会匹配FQCN中的点（`.`）或美元符号（`$`）。 星号（`*`）匹配FQCN中的一个或多个字符。模式中的所有其他字符将与FQCN一对一匹配。

例如：

- `*`: 停用所有条件。
- `org.junit.*`: 停用`org.junit`基础包及子包下的所有条件。
- `*.MyCondition`: 停用`MyCondition`类中的每个条件。
- `*System*`: 停用简单类名称包含`System`类中的每个条件。
- `org.example.MyCondition`: 停用FQCN为`org.example.MyCondition`的条件。

### 5.4. 测试实例后处理

[`TestInstancePostProcessor`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestInstancePostProcessor.html) 为希望发布流程测试实例的`Extensions`定义了API。

常用的用法涵盖了将依赖注入到测试实例中，在测试实例中调用自定义的初始化方法等。

对于具体示例，可以查看 [`MockitoExtension`](https://github.com/junit-team/junit5-samples/tree/r5.0.0-RC2/junit5-mockito-extension/src/main/java/com/example/mockito/MockitoExtension.java) 和 [`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java) 的源代码。

### 5.5. 参数解析

[`ParameterResolver`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ParameterResolver.html) 定义了用于在运行时动态解析参数的`Extension`API。

如果测试构造器或者`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`或者`@AfterAll`方法接收参数，则必须在运行时通过`ParameterResolver`*解析*该参数。开发人员可以使用内置的`ParameterResolver`（参考 [`TestInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.0-RC2/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)）或 [自己注册](#52-注册扩展)。一般而言，参数可能被按照其*名称*、*类型*、*注解*或任何一种上述方式的组合所解析。具体示例可以参照 [`CustomTypeParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.2/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomTypeParameterResolver.java) 和 [`CustomAnnotationParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.2/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomAnnotationParameterResolver.java) 的源码。

### 5.6. 测试生命周期回调

下列接口定义了用于在测试执行生命周期的不同阶段来扩展测试的API。可参考后续章节的示例，也可以查阅 [`org.junit.jupiter.api.extension`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/package-summary.html) 包中的Javadoc，获取每个接口的详细信息。

- [`BeforeAllCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeAllCallback.html)
	- [`BeforeEachCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeEachCallback.html)
		- [`BeforeTestExecutionCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeTestExecutionCallback.html)
		- [`AfterTestExecutionCallback
`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterTestExecutionCallback.html)
	- [`AfterEachCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterEachCallback.html)
- [`AfterAllCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterAllCallback.html) 

> 📒
> ###### 实现多个扩展API
> 扩展开发人员可以选择在单个扩展中实现任意数量的上述接口。具体示例请参阅 [`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java) 的源代码。


#### 5.6.1. 测试执行之前和之后的回调

[`BeforeTestExecutionCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeTestExecutionCallback.html) 和 [`AfterTestExecutionCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterTestExecutionCallback.html) 分别为`Extensions`定义了添加行为的API，这些行为将在执行测试方法*之前*和*之后立即执行*。因此，这些回调非常适合于定时器、跟踪器以及其他类似的场景。如果你需要实现围绕`@BeforeEach`和`@AfterEach`方法调用的回调，实现`BeforeEachCallback`和`AfterEachCallback`即可。

以下示例展示了如何使用这些回调来统计和记录测试方法的执行时间。`TimingExtension`同时实现了`BeforeTestExecutionCallback`和`AfterTestExecutionCallback`接口，从而给测试执行进行计时和记录。
	
###### *一个为测试方法执行计时和记录的扩展*

```java
import java.lang.reflect.Method;
import java.util.logging.Logger;

import org.junit.jupiter.api.extension.AfterTestExecutionCallback;
import org.junit.jupiter.api.extension.BeforeTestExecutionCallback;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ExtensionContext.Namespace;
import org.junit.jupiter.api.extension.ExtensionContext.Store;

public class TimingExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private static final Logger LOG = Logger.getLogger(TimingExtension.class.getName());

    @Override
    public void beforeTestExecution(ExtensionContext context) throws Exception {
        getStore(context).put(context.getRequiredTestMethod(), System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method testMethod = context.getRequiredTestMethod();
        long start = getStore(context).remove(testMethod, long.class);
        long duration = System.currentTimeMillis() - start;

        LOG.info(() -> String.format("Method [%s] took %s ms.", testMethod.getName(), duration));
    }

    private Store getStore(ExtensionContext context) {
        return context.getStore(Namespace.create(getClass(), context));
    }

}
```

由于`TimingExtensionTests`类通过`@ExtendWith`注册了`TimingExtension`，所以，测试将在执行时应用这个时间。

###### *一个使用示例TimingExtension的测试类*

```java
@ExtendWith(TimingExtension.class)
class TimingExtensionTests {

    @Test
    void sleep20ms() throws Exception {
        Thread.sleep(20);
    }

    @Test
    void sleep50ms() throws Exception {
        Thread.sleep(50);
    }

}
```

以下是运行`TimingExtensionTests`时生成的日志记录示例。

```sh
INFO: Method [sleep20ms] took 24 ms.
INFO: Method [sleep50ms] took 53 ms.
```

### 5.7. 异常处理

[`TestExecutionExceptionHandler`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestExecutionExceptionHandler.html) 为`Extensions`定义了异常处理的API，从而可以处理在执行测试时抛出的异常。

下面的例子展示了一个扩展，它将吃掉所有的`IOException`，但会重新抛出任何其他类型的异常。

###### *一个异常处理扩展*

```java
public class IgnoreIOExceptionExtension implements TestExecutionExceptionHandler {

    @Override
    public void handleTestExecutionException(ExtensionContext context, Throwable throwable)
            throws Throwable {

        if (throwable instanceof IOException) {
            return;
        }
        throw throwable;
    }
}
```

### 5.8. 为测试模板提供调用上下文

当至少有一个 [`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html) 被注册时，标注了 [`@TestTemplate`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestTemplate.html) 的方法才能被执行。每个这样的提供者负责提供一个 [`TestTemplateInvocationContext`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContext.html) 实例的`Stream`。每个上下文都可以指定一个自定义的显示名称和一个额外的扩展名列表，这些扩展名仅用于下一次调用 [`@TestTemplate`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestTemplate.html) 方法。

以下示例展示了如何编写测试模板以及如何注册和实现一个 [`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html).

###### 一个附带扩展名的测试模板

```java
@TestTemplate
@ExtendWith(MyTestTemplateInvocationContextProvider.class)
void testTemplate(String parameter) {
    assertEquals(3, parameter.length());
}

static class MyTestTemplateInvocationContextProvider implements TestTemplateInvocationContextProvider {
    @Override
    public boolean supportsTestTemplate(ExtensionContext context) {
        return true;
    }

    @Override
    public Stream<TestTemplateInvocationContext> provideTestTemplateInvocationContexts(ExtensionContext context) {
        return Stream.of(invocationContext("foo"), invocationContext("bar"));
    }

    private TestTemplateInvocationContext invocationContext(String parameter) {
        return new TestTemplateInvocationContext() {
            @Override
            public String getDisplayName(int invocationIndex) {
                return parameter;
            }

            @Override
            public List<Extension> getAdditionalExtensions() {
                return Collections.singletonList(new ParameterResolver() {
                    @Override
                    public boolean supportsParameter(ParameterContext parameterContext,
                            ExtensionContext extensionContext) {
                        return parameterContext.getParameter().getType().equals(String.class);
                    }

                    @Override
                    public Object resolveParameter(ParameterContext parameterContext,
                            ExtensionContext extensionContext) {
                        return parameter;
                    }
                });
            }
        };
    }
}
```

在这个例子中，测试模板将被调用两次。调用的显示名称是调用上下文指定的"foo"和"bar"。每个调用都会注册一个自定义的 [`ParameterResolver`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ParameterResolver.html) 用于解析方法参数。下面是使用`ConsoleLauncher`时产生的输出信息。

```sh
└─ testTemplate(String) ✔
   ├─ foo ✔
   └─ bar ✔
```

[`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html) 扩展API主要用于实现不同类型的测试，这些测试依赖于某个类似于测试的方法的重复调用（尽管它们不在同一个上下文中）。 例如，使用不同的参数，以不同的方式准备测试类实例，或多次调用而不修改上下文。请参阅 [重复测试](#312-重复测试) 或 [参数化测试](#313-参数化测试) 的实现，它们都使用了该扩展点来提供其相关的功能。

### 5.9. 在扩展中保持状态

通常，扩展只实例化一次。随之而来的相关问题是：开发者如何能够在两次调用之间保持扩展的状态？`ExtensionContext` API提供了一个`Store`用来解决这一问题。扩展可以将值放入Store中供以后检索。请参阅 [`TimingExtension`](#一个为测试方法执行计时和记录的扩展) 了解如何使用具有方法级作用域的`Store`。要注意，在测试执行期间，被存储在一个`ExtensionContext`中的值在周围其他的`ExtensionContext`中是不可用的。由于`ExtensionContexts`可能是嵌套的，因此内部上下文的范围也可能受到限制。请参阅相应的Javadoc来了解有关通过 [Store](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExtensionContext.Store.html) 存储和检索值的方法的详细信息。

### 5.10. 在扩展中支持的实用程序

 JUnit Platform Commons公开了一个名为 [`org.junit.platform.commons.support`](http://junit.org/junit5/docs/current/api/org/junit/platform/commons/support/package-summary.html) 的包，它包含了用于处理注解、反射和类路径扫描任务且正在维护中的实用工具方法。`TestEngine`和`Extension`的作者应该被鼓励去使用这些方法，以便与JUnit Platform的行为保持一致。
 
### 5.11. 用户代码和扩展的相对执行顺序

当执行包含一个或多个测试方法的测试类时，除了用户提供的测试和生命周期方法外，还会调用大量的回调函数。 下图说明了用户提供的代码和扩展代码的相对顺序。

<a id="511-用户代码和扩展代码"></a>

![](http://junit.org/junit5/docs/current/user-guide/images/extensions_lifecycle.png)

###### 用户代码和扩展代码

用户提供的测试和生命周期方法以橙色表示，扩展提供的回调代码由蓝色显示。灰色框表示单个测试方法的执行，并将在测试类中对每个测试方法重复执行。

下表进一步解释了 [用户代码和扩展代码](#511-用户代码和扩展代码) 图中的十二个步骤。

| 步骤 | 接口/注解 |描述|
|:---|:---|:---|
| 1 |接口org.junit.jupiter.api.extension.BeforeAllCallback|执行所有容器测试之前执行的扩展代码|
| 2 |注解org.junit.jupiter.api.BeforeAll|执行所有容器测试之前执行的用户代码|
| 3 |接口org.junit.jupiter.api.extension.BeforeEachCallback|每个测试执行之前执行的扩展代码|
| 4 |注解org.junit.jupiter.api.BeforeEach|每个测试执行之前执行的用户代码|
| 5 |接口org.junit.jupiter.api.extension.BeforeTestExecutionCallback|测试执行之前立即执行的扩展代码|
| 6 |注解org.junit.jupiter.api.Test|真实测试方法的用户代码|
| 7 |接口org.junit.jupiter.api.extension.TestExecutionExceptionHandler|用于处理测试期间抛出的异常的扩展代码|
| 8 |接口org.junit.jupiter.api.extension.AfterTestExecutionCallback|测试执行后立即执行的扩展代码|
| 9 |注解org.junit.jupiter.api.AfterEach|每个执行测试之后执行的用户代码|
| 10 |接口org.junit.jupiter.api.extension.AfterEachCallback|每个执行测试之后执行的扩展代码|
| 11 |注解org.junit.jupiter.api.AfterAll|执行所有容器测试之后执行的用户代码|
| 12 |接口org.junit.jupiter.api.extension.AfterAllCallback|执行所有容器测试之后执行的扩展代码|

在最简单的情况下，只有实际的测试方法被执行（步骤6）; 所有其他步骤都是可选的，具体包含的步骤将取决于是否存在用户代码或对相应生命周期回调的扩展支持。有关各种生命周期回调的更多详细信息，请参阅每个注解和扩展各自的JavaDoc。