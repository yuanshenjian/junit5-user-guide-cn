## 5.扩展模型

### 5.1 概述

相比于JUnit4中的`Runner`，`@Rule`以及`@ClassRule`等多个扩展点，JUnit Jupiter的扩展模型始于一个单一连贯概念：`Extension`API。但是，需要注意的是 `Extension` 本身也只是一个标记接口。

### 5.2 注册扩展

JUnit Jupiter中的扩展可以通过 [`@ExtenWith`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExtendWith.html) 注解显式注册，或者通过Java的 [`ServiceLoader`机制](http://junit.org/junit5/docs/current/user-guide/#extensions-registration-automatic) 自动完成。

#### 5.2.1 声明式的扩展注册
开发者可以通过在测试接口、测试类、测试方法或者自定义的[组合注解](http://junit.org/junit5/docs/current/user-guide/#writing-tests-meta-annotations) 上标注 `@ExtendWith(...)` 并提供要注册的扩展类的引用，从而以声明式的方式注册一个或多个扩展。

例如，要给某个测试方法注册一个自定义的 `MockitoExtension`，你可以参照如下的方式标注该方法:

```
@ExtendWith(MockitoExtension.class)
@Test
void mockTest() {
    // ...
}
```

若要为某个类或者其子类注册一个自定义的`MockitoExtension`，将注解添加到测试类上即可：

```
@ExtendWith(MockitoExtension.class)
class MockTests {
    // ...
}
```

多个扩展类的注册可以通过如下形式完成：

```
@ExtendWith({ FooExtension.class, BarExtension.class })
class MyTestsV1 {
    // ...
}
```

当然，另一种方式的多个扩展类的的注册形式，也可以是这样子：

```
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MyTestsV2 {
    // ...
}
```
上述的两种扩展注册方式是等价的，`MyTestV1` 和 `MyTestV2` 都会被 `FooExtension` 和 `BarExtension` 进行扩展，且扩展顺序跟声明顺序一致。

#### 5.2.2 自动扩展注册

除了 [声明式的扩展注册](http://junit.org/junit5/docs/current/user-guide/#extensions-registration-declarative) 支持使用注解外，JUnit Jupiter 同样也支持通过 Java 的`java.util.ServiceLoader` 机制来做*全局的扩展注册*。采用这种机制后自动的检测 `classpath` 下的第三方扩展，并自动完成注册。

另外，还可以通过提供自定扩展的全类名来完成注册，该扩展被定义在它所在的JAR文件中的 `/META-INF/services` 目录下的 `org.junit.jupiter.api.extension.Extension` 文件里。

##### 使用自动扩展检测

自动检测是一种高级特性，因此默认是关闭的。想要启用它，只需要在配置文件中将 `junit.jupiter.extensions.autodetection.enabled` 的配置变量设置为 `true` 即可。这一参数可以作为JVM的系统属性并作为一个`LauncherDiscoveryRequest`的配置参数传递给`Laucher`，另一种方法是通过配置 JUnit Platform 的配置文件（详情见[配置参数](http://junit.org/junit5/docs/current/user-guide/#running-tests-config-params)）。

例如，要启用扩展的自动检测，你可以通过在启动JVM时传入如下系统参数

```
-Djunit.jupiter.extensions.autodetection.enabled=true
```

当启用了自动扩展检测后，被 `ServiceLoader` 机制发现的扩展会被添加到 JUnit Jupiter 的全局扩展的扩展注册表中（例如. `TestInfo`,`TestReporter`的支持，等等）。

#### 5.2.3 扩展的继承
扩展的继承在测试类中表现为语义上自顶向下的形式。也就是说，一个类级别的注册扩展是可以被方法级的扩展所继承的。此外，一个特定的扩展实现只能在给定的扩展上下文或其父上下文中被注册一次。因此，任何重复注册的扩展实现都将会被忽略掉。

### 5.3 附加条件测试的执行

[`ExecutionCondition`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExecutionCondition.html) 定义了 `Extension` 的API编程--*附加条件测试的执行*。

每个容器（例如，测试类）都会对`ExecutionCondition` 进行解析，从而确定是否应根据提供的 `ExtensionContext` 执行其包含的所有测试。类似地，`ExecutionCondition` 会被每个测试解析，从而确定是否应该根据提供的 `ExtensionContext` 执行给定的测试方法。

当多个 `ExecutionCondition` 扩展被注册时，只要其中一个条件没有被满足，那么这个容器或者测试就会失效。由于容器可能在某个条件被解析之前就因为另一个失败的条件而失效，所以没有办法保证每个条件都被解析。也就是说，条件的解析机制类似于“短路或”(符号为`||`)操作。

可以参考 [`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.0.0-M5/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java) 和 [`@Disable`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Disabled.html)的源码来获取具体的案例。

#### 5.3.1. 停用条件

有时候，在没有明确的附加条件下运行测试集可能更有用。例如，开发者可能想要运行一些被标注了 `@Disable` 的测试，以便观察这些测试是否一直是*失败的*。为了完成这些工作，只需提供一个用于 `junit.jupiter.conditions.deactivate` 的关键词配置，以指定当前测试运行的哪些条件应该被停用（即不被解析）。该模式可以作为JVM系统属性提供，也可以作为 `LauncherDiscoveryRequest` 中的*配置参数*提供给`Launcher`, 或使用 JUnit Platform 的配置文件（详情见[配置变量](http://junit.org/junit5/docs/current/user-guide/#running-tests-config-params)）。

例如，要停用JUnit的 `@Disable` 条件，你可以在JVM启动时传入系统参数完成：
```
-Djunit.jupiter.conditions.deactivate=org.junit.*DisabledCondition
```

##### 模式匹配语法

如果 `junit.jupiter.conditions.deactivate` 模式仅由星号（`*`）组成，则所有条件都将被禁用。 否则，该模式将用于匹配每个注册的条件的完整的类名（*FQCN*）。 模式中的点（`.`）会匹配FQCN中的点（`.`）或美元符号（`$`）。 星号（`*`）匹配FQCN中的一个或多个字符。 该模式中的所有其他字符将与FQCN一对一匹配。

例如：

- `*`: 停用所有条件。
- `org.junit.*`: 停用`org.junit`基础包及子包下的所有条件。
- `*.MyCondition`: 停用`MyCondition`类中的每个条件。
- `*System*`: 停用简单类名称包含`System`类中的每个条件。
- `org.example.MyCondition`: 停用FQCN为`org.example.MyCondition`的条件。

### 5.4 测试实例的后处理

[`TestInstancePostProcessor`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestInstancePostProcessor.html) 为`Extensions`定义了测试实例后处理的API。

常用的用法涵盖了将依赖注入到测试实例中，在测试实例中调用自定义的初始化方法等。

对于具体示例，可以查看 [`MockitoExtension`](https://github.com/junit-team/junit5-samples/tree/r5.0.0-RC2/junit5-mockito-extension/src/main/java/com/example/mockito/MockitoExtension.java) 和 [`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java) 的源代码。

### 5.5 参数解析

[`ParameterResolver`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ParameterResolver.html) 定义了用于在运行时动态解析参数的 `Extension` API。

如果一个测试构造器或者 `@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll` 或者 `@AfterAll` 方法接收了一个参数，那么该参数一定会在运行时被 `ParameterResolver` *解析*。开发人员可以使用内置的 `ParameterResolver`（参考 [`TestInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.0-RC2/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)）或 [自己注册](http://junit.org/junit5/docs/current/user-guide/#extensions-registration)。一般而言，参数可能被按照其*名称*、*类型*、*注解*或在任何一种上述方式的组合所解析。具体示例可以参照 [`CustomTypeParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.0-M5/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomTypeParameterResolver.java) 和 [`CustomAnnotationParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.0-M5/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomAnnotationParameterResolver.java) 的源码。

### 5.6 测试生命周期回调

下列接口定义了用于在测试执行生命周期的不同阶段来扩展测试的API。可参考后续章节的示例，也可以查阅 [`org.junit.jupiter.api.extension`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/package-summary.html) 包中的Javadoc，获取每个接口的详细信息。

- [`BeforeAllCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeAllCallback.html)
	- [`BeforeEachCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeEachCallback.html)
		- [`BeforeTestExecutionCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeTestExecutionCallback.html)
		- [`AfterTestExecutionCallback
`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterTestExecutionCallback.html)
	- [`AfterEachCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterEachCallback.html)
- [`AfterAllCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterAllCallback.html) 

> 📒
> ###### 实现多种扩展API
> 扩展开发人员可以选择在单个扩展中实现任意数量的上述接口。参考 [`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)的源代码以获取具体示例。

#### 5.6.1 Before 和 After 的测试扩展回调

[`BeforeTestExecutionCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/BeforeTestExecutionCallback.html) 和 [`AfterTestExecutionCallback`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/AfterTestExecutionCallback.html) 分别为`Extensions`定义了用于在执行测试方法之前和之后添加立即执行行为的API。因此，这些回调非常适合于定时器、跟踪器以及其他类似的场景。如果你需要实现在`@BeforeEach`和`@AfterEach`方法下调用的回调，可以实现`BeforeEachCallback`和`AfterEachCallback`。

以下示例展示了如何使用这些回调来统计和记录测试方法的执行时间。`TimingExtension` 同时实现了 `BeforeTestExecutionCallback` 和 `AfterTestExecutionCallback` 接口从而给测试执行做时间统计和日志记录。
	
###### 一个关于测试执行的时间和日志的扩展示例：

```
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

由于 `TimingExtensionTests` 类通过 `@ExtendWith` 注册了 `TimingExtension`，所以它的测试在执行时会被计时。

###### 下面是一个测试类应用了 TimingExample 的示例：

```
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

以下是运行TimingExtensionTests时生成的日志的示例。

```
INFO: Method [sleep20ms] took 24 ms.
INFO: Method [sleep50ms] took 53 ms.
```

### 5.7 异常处理

[`TestExecutionExceptionHandler`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestExecutionExceptionHandler.html) 为`扩展`定义了异常处理的API，可以在执行测试时处理抛出的异常。

下面的例子展示了一个扩展，它将收到的 `IOException` 重新包装并抛出为其他类型的异常。

###### 一个异常处理扩展

```
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

### 5.8 为测试模板提供调用上下文

当至少有一个 [`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html) 被注册后，标注了 [`@TestTemplate`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestTemplate.html) 的方法才能被执行。每个provider都必须提供 [`TestTemplateInvocationContext`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContext.html) 实例的`Stream`。每个上下文都可以指定一个自定义的展示名称和仅用于 [`@TestTemplate`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestTemplate.html) 方法下一次调用的额外扩展列表。

以下示例展示了如何编写测试模板以及如何注册和实现一个 [`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html).

###### 一个带有附加扩展名的测试模板

```
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

在这个例子中，测试模板将被调用两次。调用的展示名称是调用上下文指定的“foo”和“bar”。每个调用都会注册一个自定义的 [`ParameterResolver`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ParameterResolver.html) 用于解析方法参数。下面输出是使用`ConsoleLauncher`时产生的。

```
└─ testTemplate(String) ✔
   ├─ foo ✔
   └─ bar ✔
```

[`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html) 扩展API主要用于实现不同类型的测试，这些测试依赖于某个类似于测试的方法重复调用，即便它们不在同一个上下文中 - 例如，使用不同的参数，通过准备不同的测试类实例，或多次调用而不修改上下文。请参阅使用 [重复测试](http://junit.org/junit5/docs/current/user-guide/#writing-tests-repeated-tests) 或 [参数化测试](http://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests) 的实现，这些章节使用了这个扩展点来提供相关的功能。

### 5.9 在扩展中保持状态

通常地，一个扩展实例只能初始化一次。那么问题来了：开发者如何能够在两次调用之间保持扩展的状态？`ExtensionContext`API提供了一个`Store`用来解决这一问题。扩展可以将值保存在Store中，以备之后的检索。查看[`TimingExtension`]()可以看到在方法级范围使用`Store`的示例。值得一提的是，在测试执行期间，被存储在一个`ExtensionContext`中的值，在其他的`ExtensionContext`中是不可用的。由于`ExtensionContexts`可能被嵌套，因此内部上下文的范围也可能受到限制。 请参阅相应的Javadoc来了解有关通过[Store](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExtensionContext.Store.html)存储和检索值的方法的详细信息。

## 5.10 扩展中支持的工具

 JUnit Platform Commons 公开了一个名为[`org.junit.platform.commons.support `](http://junit.org/junit5/docs/current/api/org/junit/platform/commons/support/package-summary.html) 的包，该包包含了用于处理注解、反射和类路径扫描任务的实用方法。`TestEngine`和`Extension`的开发者被鼓励去使用这些方法，以便与JUnit Platform的行为保持一致。
 
## 5.11 用户代码和扩展的相对执行顺序

当执行包含一个或多个测试方法的测试类时，除了用户提供的测试和生命周期方法之外，还会调用多个扩展回调。 下图说明了用户提供的代码和扩展代码的相对顺序。

![](http://junit.org/junit5/docs/current/user-guide/images/extensions_lifecycle.png)

*用户代码和扩展代码*

用户提供的测试和生命周期方法以橙色表示，扩展提供的回调代码由蓝色显示。 灰色框表示单个测试方法的执行，并将在测试类中对每个测试方法重复执行。

下表进一步说明了[用户代码和扩展代码]()图中的十二个步骤。

|步骤|接口/注解|描述|
|:---|:---|:---|
|1|接口org.junit.jupiter.api.extension.BeforeAllCallback|执行所有容器测试之前执行的扩展代码|
|2|注解 org.junit.jupiter.api.BeforeAll|执行所有容器测试之前执行的用户代码|
|3|接口org.junit.jupiter.api.extension.BeforeEachCallback|在每次执行测试之前执行的扩展代码|
|4|注解 org.junit.jupiter.api.BeforeEach|每次执行测试之前执行的用户代码|
|5|接口 org.junit.jupiter.api.extension.BeforeTestExecutionCallback|在执行测试之前立即执行扩展代码|
|6|注解org.junit.jupiter.api.Test|用户代码的真实测试方法|
|7|接口org.junit.jupiter.api.extension.TestExecutionExceptionHandler|用于处理测试期间抛出的异常的扩展代码|
|8|接口org.junit.jupiter.api.extension.AfterTestExecutionCallback|测试执行后立即执行扩展代码及其相应的异常处理程序|
|9|注解 org.junit.jupiter.api.AfterEach|每次执行测试后执行的用户代码|
|10|接口 org.junit.jupiter.api.extension.AfterEachCallback|每次执行测试后执行的扩展代码|
|11|注解org.junit.jupiter.api.AfterAll|执行所有容器测试后执行的用户代码|
|12|接口org.junit.jupiter.api.extension.AfterAllCallback|执行所有容器测试后执行的扩展代码|

上述情况在最简单的情况下，仅执行实际的测试方法（步骤6）; 所有其他步骤都是可选的，具体包含的步骤将取决于用户代码的存在或相应生命周期回调的扩展支持。 有关各种生命周期回调的更多详细信息，请参阅相应的JavaDoc以获取每个注释和扩展名。