# 5.扩展模型

## 5.1 概述

与JUnit 4 中的`Runner`，`@Rule`,以及`@ClassRule`等多个扩展点相较而言，JUnit Jupiter 的扩展模型的组成从始到终都只是唯一一个概念：`Extension`API。但是，需要注意的是`Extension`本身只是一个标记接口。

## 5.2 注册扩展

JUnit Jupiter中的扩展可以通过[`@ExtenWith`]()注解完成注册，当然，也可以通过Java的`ServiceLoader`类完成[机械式的]()自动注册。

### 5.2.1 声明式的扩展注册

开发者可以通过在测试接口、测试类、测试方法或者惯用的[组合注解]()上标注`@ExtendWith(...)`并提供所要注册的扩展类的`.class`文件，来完成声明式的注册一个或多个扩展。

举例而言，要给一个给定的测试方法注册一个惯用的`MockitoExtension `扩展，可以通过在测试方法上完成如下标注。

```
@ExtendWith(MockitoExtension.class)
@Test
void mockTest() {
    // ...
}
```

若是要给一个给定的类或者其子类注册一个惯用的`MockitoExtension `扩展，可以完成如下标注。

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

当然，多个扩展类的的注册形式也可以如下所示：

```
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MyTestsV2 {
    // ...
}
```
上述的两种扩展注册方式是等价的，`MyTestV1`和`MyTestV2`都会被`FooExtension`和`BarExtension`扩展，并且也是按照这样的先后顺序被扩展。

### 5.2.2 自动扩展注册

除了[声明式的扩展注册]()支持使用注解完成外，JUnit Jupiter 同样也支持通过Java的`java.util.ServiceLoader` 加载机制完成全局的扩展注册。采用这种类加载机制的自动注册方式，可以在给定的`classpath`下，自动的检测第三方扩展，并完成该扩展的注册。

另外，还可以通过在相关的JAR文件的`/META-INF/services`文件夹下找到`org.junit.jupiter.api.extension.Extension`文件，对于一般的扩展而言，可以在该文件中使用扩展完整的类名称名称，完成扩展的注册。

### 使用自动扩展检测

自动扩展检测是一种高级特性，因此并不在默认配置中。想要激活并使用它，只需要在配置文件中，将`junit.extensions.autodetection.enabled`的Key设置为`true`即可。如此一来，这一参数便可以作为JVM的系统参数或作为一个传递给`Laucher`的`LauncherDiscoveryRequest `的配置参数。

举例而言，激活扩展的自动检测，开发者可以通过在开启JVM时传入如下系统参数

`
-Djunit.extensions.autodetection.enabled=true
`

当完成了自动扩展检测的启动工作，`ServiceLoader `加载机制就会自动检测扩展，并在JUnit Jupiter 的全局扩展之后完成这些自动检测到的扩展的注册。

### 5.2.3 扩展的继承

扩展的继承在测试类中是表现为语义上自顶向下的形式的。也就是说，一个class级别的注册扩展是可以被mothod级的扩展所继承的。而且，每个特定的注册的实现对于给定的上下文而言，无论子扩展，还是其父扩展，都只能被注册一次。因此，任何对扩展的重复注册的尝试都将会被忽略掉。

## 5.3 附加条件测试的执行

[`ExecutionCondition`]()为附加条件的测试及其纲领定义了 `Extension` API。

对每个容器（例如，测试类）评估`ExecutionCondition`，以确定是否应根据提供的`ExtensionContext`执行其包含的所有测试. 类似地，对每个测试评估一个`ExecutionCondition`，以确定是否应该根据提供的`ExtensionContext`执行给定的测试方法。

当多个`ExecutionCondition`扩展被注册，只要其中一个条件没有被满足，那么这个容器或者测试就会失效。由于可能在某个条件被评估到之前，容器就因为另一个失败的条件而失效，因此，没有办法保证每个条件都被评估到。也就是说，对于条件的评估工作等价于“短路或”(符号为`||`)操作。

开发者可以参考[`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.0.0-M5/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java)和[`@Disable`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Disabled.html)的源码来获取具体的案例。

### 5.3.1 停用条件

有时，在没有明确的附加条件下运行测试集可能更有用。例如，有时开发者可能想要运行一些被标注了`@Disable`的测试，以便观察这些测试是否真的*挂了*。为了完成这些工作，只需提供一个用于junit.conditions.deactivate的关键词配置，以指定当前测试运行的哪些条件应该被停用（即不被评估）。该模式可以作为JVM系统属性提供，也可以作为`LauncherDiscoveryRequest`中的配置参数提供给`Launcher`。

举例而言，停用JUnit的 `@Disable`条件，开发者可以通过在JVM启动时传入系统参数完成：
`-Djunit.conditions.deactivate=org.junit.*DisabledCondition`

### 句法模式匹配

如果`junit.conditions.deactivate`模式仅由星号（`*`）组成，则所有条件都将被禁用。 否则，该模式将用于匹配每个注册条件的完整的类名称（*FQCN*）。 模式中的任何点（`.`）将与FQCN中的点（`.`）或美元符号（`$`）匹配。 任何星号（`*`）将与FQCN中的一个或多个字符匹配。 该模式中的所有其他字符将与FQCN一对一匹配。

举例：

- `*`: 停用所有条件。
- `org.junit.*`: 停用org.junit基础包及其任何子包下的每个条件。
- `*.MyCondition`: 停用其简单类名称完全是MyCondition的每个条件。
- `*System*`: 停用其简单类名称包含`System`的每个条件。
- `org.example.MyCondition`: 停用FQCN为`org.example.MyCondition`的条件。

## 5.4 测试实例的后期处理

[`TestInstancePostProcessor`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestInstancePostProcessor.html) 定义需要后期处理的`Extensions`API。

通常的用例包括注入依赖到测试实例中，在测试实例中调用自定义的初始化方法等。

对于具体事例，可以参考[`MockitoExtension`]()和[`SpringExtension`]()的源代码.

## 5.5 参数解析

[`ParameterResolver`]()定义了用于在运行时动态解析参数的`Extension`API。

如果一个测试构造器或者`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`或者`@AfterAll`方法接收了一个参数，那么这个参数一定会在运行时被`ParameterResolver `所解析。`ParameterResolver`可以被开发者构建（参考[`TestInfoParameterResolver`]()）或注册。一般而言，参数可能被按照其*名称*、*类型*、*注解*或在任何一种上述方式的组合所解析。具体示例，可以参照[`CustomTypeParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.0-M5/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomTypeParameterResolver.java)和[`CustomAnnotationParameterResolver`](https://github.com/junit-team/junit5/tree/r5.0.0-M5/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomAnnotationParameterResolver.java)的源码。

## 5.6 测试生命周期回调

下列接口定义了用于在测试执行的不同生命周期的时间点完成测试扩展的API。可以参考下一小节的示例，也可以通过官方文档中的[`org.junit.jupiter.api.extension`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/package-summary.html)包，获取每个接口的详细信息。

- [`BeforeAllCallback`]()
	- [`BeforeEachCallback`]()
		- [`BeforeTestExecutionCallback`]()
		- [`AfterTestExecutionCallback
`]()
	- [`AfterEachCallback`]()
- [`AfterAllCallback`]() 

> 实现多种扩展API
> 扩展开发人员可以选择在单个扩展中实现任意数量的这些接口。参考[`Consult the source code of the SpringExtension for a concrete example.`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)的源代码以获取具体示例。

### 5.6.1 Before和After的测试扩展回调

[`BeforeTestExecutionCallback`]()和[`AfterTestExecutionCallback`]()分别定义了希望添加将在执行测试方法之前和之后立即执行的行为的扩展API。因此，这些回调非常适合于定时，跟踪以及其他类似的用例。如果需要实现在`@BeforeEach`和`@AfterEach`方法下调用的回调，请改用`BeforeEachCallback`和`AfterEachCallback`。

以下示例显示如何使用这些回调来计算和记录测试方法的执行时间。 TimingExtension实现了BeforeTestExecutionCallback和AfterTestExecutionCallback之间测试执行的时间和日志。
	
*一个关于测试执行的时间和日志的扩展示例：*

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
        getStore(context).put(context.getTestMethod().get(), System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method testMethod = context.getTestMethod().get();
        long start = getStore(context).remove(testMethod, long.class);
        long duration = System.currentTimeMillis() - start;

        LOG.info(() -> String.format("Method [%s] took %s ms.", testMethod.getName(), duration));
    }

    private Store getStore(ExtensionContext context) {
        return context.getStore(Namespace.create(getClass(), context));
    }

}
```

由于TimingExtensionTests类通过`@ExtendWith`注册了TimingExtension，所以它的测试在执行时会应用计时。

*下面是一个测试类应用了 TimingExample 的示例：*

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

以下是运行TimingExtensionTests时生成的日志记录的示例。

```
INFO: Method [sleep20ms] took 24 ms.
INFO: Method [sleep50ms] took 53 ms.
```

## 5.7 异常处理

[`TestExecutionExceptionHandler`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestExecutionExceptionHandler.html)为扩展定义了异常处理API，可以在执行测试时处理抛出的异常。

下面的例子将展示一个扩展，它将收到的`IOException`重新包装并抛出为其他类型的异常。

*一个异常处理扩展*

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

## 5.8 为测试模板提供调用上下文

只有当至少有一个[`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)被注册后，被[`@TestTemplate`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestTemplate.html)标注的方法才能执行。每个provider都必须提供一系列的[`TestTemplateInvocationContext`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContext.html)实例。每个上下文都可以指定自定义显示名称和仅用于下一次调用[`@TestTemplate`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestTemplate.html)方法的其他扩展名列表。

以下示例显示了如何编写测试模板以及如何注册和实现[`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html).

*一个带有附带扩展名的测试模板*

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

在这个例子中，测试模板将被调用两次。 调用的显示名称是调用上下文指定的“foo”和“bar”。 每个调用都会注册一个自定义的[`ParameterResolver`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ParameterResolver.html)，用于解析方法参数。 使用ConsoleLauncher时的输出如下。

```
└─ testTemplate(String) ✔
   ├─ foo ✔
   └─ bar ✔
```

[`TestTemplateInvocationContextProvider`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)扩展API主要用于实现不同类型的测试，这些测试依赖于重复调用在不同的上下文中类似于测试的方法 - 例如，使用不同的参数，通过不同的准备测试类实例，或多次调用而不修改上下文。 请参阅使用此扩展点的[重复测试](http://junit.org/junit5/docs/current/user-guide/#writing-tests-repeated-tests)或[参数化测试](http://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests)的实现来提供其功能。

## 5.9 在扩展中保持状态

通常地，一个扩展实例只能初始化一次。那么问题来了：开发者如何能够在两次调用之间保持扩展的状态？`ExtensionContext`API提供了一个`Store`用来解决这一问题。扩展可以将值保存在Store中，以备之后检索。查看[`TimingExtension`](http://junit.org/junit5/docs/current/user-guide/#extensions-lifecycle-callbacks-timing-extension)可以看到在方法级范围使用`Store`的示例。值得一提的是，在测试执行期间，被存储在一个`ExtensionContext`中的值，在其他的`ExtensionContext`中是不可用的。由于ExtensionContexts可能嵌套，因此内部上下文的范围也可能受到限制。 请参阅相应的Javadoc，了解有关通过[Store](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ExtensionContext.Store.html)存储和检索值的方法的详细信息。

## 5.10 扩展中支持的工具

 JUnit Platform Commons 人为公开了一个名为[`org.junit.platform.commons.support `](http://junit.org/junit5/docs/current/api/org/junit/platform/commons/support/package-summary.html) 的包，该包包含了用于处理注释，反射和类路径扫描任务的维护实用方法。`TestEngine`和`Extension`的开发者被鼓励去使用这些支持的方法，以便与JUnit Platform的行为保持一致。
 
## 5.11 用户代码和扩展的相对执行顺序

当执行包含一个或多个测试方法的测试类时，除了用户提供的测试和生命周期方法之外，还会调用多个扩展回调。 下图说明了用户提供的代码和扩展代码的相对顺序。

![](http://junit.org/junit5/docs/current/user-guide/images/extensions_lifecycle.png)

*用户代码和扩展代码*

用户提供的测试和生命周期方法以橙色显示，回调代码由蓝色显示。 灰色框表示单个测试方法的执行，并将在测试类中对每个测试方法重复执行。

下表进一步说明了[用户代码和扩展代码]()图中的十二个步骤。

|步骤|接口/注解|描述|
|:---|:---|:---|
|1|接口org.junit.jupiter.api.extension.BeforeAllCallback|执行所有容器测试之前执行的扩展代码|
|2|注解 org.junit.jupiter.api.BeforeAll|执行所有容器测试之前执行的用户代码|
|3|接口org.junit.jupiter.api.extension.BeforeEachCallback|在每次执行测试之前执行的扩展代码|
|4|注解 org.junit.jupiter.api.BeforeEach|每次执行测试之前执行的用户代码|
|5|接口 org.junit.jupiter.api.extension.BeforeTestExecutionCallback|在执行测试之前立即执行扩展代码|
|6|注解org.junit.jupiter.api.Test|用户代码的实际测试方法|
|7|接口org.junit.jupiter.api.extension.TestExecutionExceptionHandler|用于处理测试期间抛出的异常的扩展代码|
|8|接口org.junit.jupiter.api.extension.AfterTestExecutionCallback|测试执行后立即执行扩展代码及其相应的异常处理程序|
|9|注解 org.junit.jupiter.api.AfterEach|每次执行测试后执行的用户代码|
|10|接口 org.junit.jupiter.api.extension.AfterEachCallback|每次执行测试后执行的扩展代码|
|11|注解org.junit.jupiter.api.AfterAll|执行所有容器测试后执行的用户代码|
|12|接口org.junit.jupiter.api.extension.AfterAllCallback|执行所有容器测试后执行的扩展代码|

上述情况在最简单的情况下，仅执行实际的测试方法（步骤6）; 所有其他步骤都是可选的，具体包含的步骤将取决于用户代码的存在或相应生命周期回调的扩展支持。 有关各种生命周期回调的更多详细信息，请参阅相应的JavaDoc以获取每个注释和扩展名。