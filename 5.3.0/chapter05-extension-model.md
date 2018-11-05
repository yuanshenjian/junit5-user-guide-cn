## 5. 扩展模型

### 5.1. 概述

不同于JUnit4中的`Runner`、`@Rule`以及`@ClassRule`等多个扩展点，JUnit Jupiter的扩展模型由一个连贯的概念组成：`Extension`API。但是，需要注意的是 `Extension`本身也只是一个标记接口。

### 5.2. 注册扩展
JUnit Jupiter中的扩展可以通过 [`@ExtenWith`](#extensions-registration-declarative) 注解进行声明式注册，或者通过 [`@RegisterExtension`](#extensions-registration-programmatic) 注解进行编程式注册，再或者通过Java的 [`ServiceLoader`](#extension-registration-automatic) 机制自动注册。


<a id = "extensions-registration-declarative"></a>

#### 5.2.1. 声明式扩展注册
开发者可以通过在测试接口、测试类、测试方法或者自定义的 [*组合注解*](#311-元注解和组合注解) 上标注`@ExtendWith(...)`并提供要注册扩展类的引用，从而以*声明式* 的方式注册一个或多个扩展。

例如，要给某个测试方法注册一个自定义的`RandomParametersExtension`，你可以参照如下的方式标注该方法。

```java
@ExtendWith(RandomParametersExtension.class)
@Test
void test(@Random int i) {
    // ...
}
```

若要为某个类或者其子类注册一个自定义的`MockitoExtension`，将注解添加到测试类上即可。

```java
@ExtendWith(RandomParametersExtension.class)
class MyTests {
    // ...
}
```

多个扩展类的注册可以通过如下形式完成。

```java
@ExtendWith({ FooExtension.class, BarExtension.class })
class MyFirstTests {
    // ...
}
```

还有另外一种方式来注册多个扩展类，如下面代码所示。

```java
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MySecondTests {
    // ...
}
```

> 📒 *扩展注册顺序*  
> 通过`@ExtendWith`以声明方式注册的扩展将按照它们在源代码中声明的顺序执行。例如，`MyFirstTests`和`MySecondTests`中的测试执行将按照`FooExtension`和`BarExtension`的实际顺序进行扩展。


<a id = "extensions-registration-programmatic"></a>

#### 5.2.2. 编程式扩展注册
开发人员可以通过*编程的* 方式来注册扩展，只需要将测试类中的属性字段使用 {{RegisterExtension}} 注解标注即可。

当一个扩展通过 [`@ExtenWith`](#extensions-registration-declarative) 声明式注册后，它就只能通过注解配置。相比之下，当通过`@RegisterExtension`注册扩展时，我们可以通过*编程* 的方式来配置扩展 -- 例如，将参数传递给扩展的构造函数、静态工厂方法或构建器API。

> 📒 `@RegisterExtension` 字段不能为`private`或`null` (在评估阶段) ，但可以是`static`或非静态。

#####  静态字段
如果一个`@RegisterExtension`字段是`static`的，该扩展会在那些在测试类中通过`@ExtendWith`进行注册的扩展之后被注册。这种*静态扩展* 在扩展API的实现上没有任何限制。因此，通过静态字段注册的扩展可能会实现类级别和实例级别的扩展API，例如`BeforeAllCallback`、`AfterAllCallback`和`TestInstancePostProcessor`，同样还有方法级别的扩展API，例如`BeforeEachCallback`等等。

在下面的例子中，测试类中的`server`字段通过使用`WebServerExtension`所支持的构建器模式以编程的方式进行初始化。已经配置的`WebServerExtension`将在类级别自动注册为一个扩展 - 例如，要在测试类中所有测试方法运行之前启动服务器，以及在所有测试完成后停止服务器。此外，使用`@BeforeAll`或`@AfterAll`标注的静态生命周期方法以及`@BeforeEach`、`@AfterEach`和`@Test`标注的方法可以在需要的时候通过`server`字段访问该扩展的实例。

*一个通过静态字段注册的扩展：*

```java
class WebServerDemo {

    @RegisterExtension
    static WebServerExtension server = WebServerExtension.builder()
        .enableSecurity(false)
        .build();

    @Test
    void getProductList() {
        WebClient webClient = new WebClient();
        String serverUrl = server.getServerUrl();
        // Use WebClient to connect to web server using serverUrl and verify response
        assertEquals(200, webClient.get(serverUrl + "/products").getResponseStatus());
    }
}
```

##### Kotlin中的静态字段
Kotlin编程语言没有`static`字段的概念。但我们可以使用注解来让编译器生成静态字段。如前文说述，`@RegisterExtension`标注的字段不能为`private`或`null`，因此我们不能在Kotlin中使用会生成`private`字段`@JvmStatic`注解，所以我们就只能使用`@JvmField`注解了。

以下示例是上一节中已移植到Kotlin的`WebServerDemo`的一个版本。

```java
class KotlinWebServerDemo {

    companion object {
        @JvmField
        @RegisterExtension
        val server = WebServerExtension.builder()
                .enableSecurity(false)
                .build()
    }

    @Test
    fun getProductList() {
        // Use WebClient to connect to web server using serverUrl and verify response
        val webClient = WebClient()
        val serverUrl = server.serverUrl
        assertEquals(200, webClient.get("$serverUrl/products").responseStatus)
    }
}
```


#####  实例字段
如果`@RegisterExtension`字段是非静态的（例如，一个实例字段），那么该扩展将在测试类实例化之后被注册，并且在每个已注册的`TestInstancePostProcessor`被赋予后处理测试实例的机会之后（可能给被标注的字段注入要使用的扩展实例）。因此，如果这样的*实例扩展* 实现了诸如`BeforeAllCallback`、`AfterAllCallback`或`TestInstancePostProcessor`这些类级别或实例级别的扩展API，那么这些API将不会正常执行。默认情况下，实例扩展将在那些通过`@ExtendWith`在方法级别注册的扩展之后被注册。但是，如果测试类是使用了`@TestInstance(Lifecycle.PER_CLASS)`配置，实例扩展将在它们之前被注册。


在下面的例子中，通过调用自定义`lookUpDocsDir()`方法并将结果提供给`DocumentationExtension`中的静态`forPath()`工厂方法，从而以编程的方式初始化测试类中的`docs`字段。配置的`DocumentationExtension`将在方法级别自动被注册为扩展。另外，`@BeforeEach`、`@AfterEach`和`@Test`方法可以在需要的时候通过`docs`字段访问扩展的实例。

*一个通过静态字段注册的扩展：*

```java
class DocumentationDemo {

    static Path lookUpDocsDir() {
        // return path to docs dir
    }

    @RegisterExtension
    DocumentationExtension docs = DocumentationExtension.forPath(lookUpDocsDir());

    @Test
    void generateDocumentation() {
        // use this.docs ...
    }
}
```

 
<a id = "extension-registration-automatic"></a>

#### 5.2.3. 自动扩展注册

除了 [声明式扩展注册](#extensions-registration-declarative) 和 [编程式扩展注册](#extensions-registration-programmatic) 支持使用注解，JUnit Jupiter还支持通过Java的`java.util.ServiceLoader`机制进行*全局扩展注册*，采用这种机制后会自动的检测`classpath`下的第三方扩展，并自动完成注册。

具体来说，自定义扩展可以通过在`org.junit.jupiter.api.extension.Extension`文件中提供其全类名来完成注册，该文件位于其封闭的JAR文件中的`/META-INF/services`目录下。

##### 启用自动扩展检测
自动检测是一种高级特性，默认情况下它是关闭的。要启用它，只需要在配置文件中将 `junit.jupiter.extensions.autodetection.enabled`的*配置参数* 设置为 `true`即可。该参数可以作为JVM系统属性、或作为一个传递给`Launcher`的`LauncherDiscoveryRequest`中的配置参数、再或者通过JUnit Platform配置文件（详情请参阅 [配置参数](#45-配置参数)）来提供。

例如，要启用扩展的自动检测，你可以在启动JVM时传入如下系统参数。

```sh
-Djunit.jupiter.extensions.autodetection.enabled=true
```

启用自动检测功能后，通过`ServiceLoader`机制发现的扩展将在JUnit Jupiter的全局扩展（例如对`TestInfo`，`TestReporter`等的支持）之后被添加到扩展注册表中。


#### 5.2.4. 扩展继承
扩展在测试类层次结构中以自顶向下的语义被继承。同样，在类级别注册的扩展会被方法级的扩展继承。此外，特定的扩展实现只能针对给定的扩展上下文及其父上下文进行一次注册。因此，任何尝试注册重复的扩展实现都将被忽略。

### 5.3. 条件测试执行
{{ExecutionCondition}} 定为程序化的条件测试执行定义了`Extension`API。

每个容器（例如测试类）都会对`ExecutionCondition`进行解析，从而确定是否应该根据提供的`ExtensionContext`执行其包含的所有测试。类似地，`ExecutionCondition`会被每个测试解析，从而确定是否应该根据提供的`ExtensionContext`执行给定的测试方法。

当多个`ExecutionCondition`扩展被注册时，只要有一个条件*被禁用*，容器或测试就会被禁用。所以，不能保证每个条件都会被解析，因为其中某个扩展可能已经导致容器或测试被禁用了。也就是说，条件的解析机制类似于短路 或(符号为`||`)操作。

有关具体示例，请参阅 {{DisabledCondition}} 和 {{Disabled}} 的源码。

#### 5.3.1. 禁用条件
有时候，在没有明确的条件被激活的情况下运行测试套件可能更有用。例如，你可能想要运行某些即便被标注了`@Disable`的测试，从而观察这些测试是否一直是*失败的*。此时只需为`junit.jupiter.conditions.deactivate`配置参数提供一个匹配模式，以指定当前测试运行应停用哪些条件（即不被解析）。该匹配模式可以作为JVM系统属性、或作为一个传递给`Launcher`的`LauncherDiscoveryRequest`中的配置参数、再或者通过JUnit Platform配置文件（详情请参阅 [配置参数](#45-配置参数)）来提供。

例如，要停用JUnit的`@Disable`条件，你可以在JVM启动时传入系统参数完成：

```sh
-Djunit.jupiter.conditions.deactivate=org.junit.*DisabledCondition
```

##### 模式匹配语法
如果`junit.jupiter.conditions.deactivate`模式仅由星号（`*`）组成，则所有条件都将被禁用。 否则，该模式将用于匹配每个注册的条件的完整的类名（*FQCN*）。 模式中的点（`.`）会匹配FQCN中的点（`.`）或美元符号（`$`）。 星号（`*`）匹配FQCN中的一个或多个字符。模式中的所有其他字符将与FQCN一对一匹配。

例如：

- `*`: 禁用所有条件。
- `org.junit.*`: 禁用`org.junit`基础包及子包下的所有条件。
- `*.MyCondition`: 禁用`MyCondition`类中的每个条件。
- `*System*`: 禁用其简单类名包含`System`的类中的每个条件。
- `org.example.MyCondition`: 禁用FQCN为`org.example.MyCondition`的条件。

### 5.4. 测试实例工厂
{{TestInstanceFactory}} 为希望创建测试类实例的`Extensions`定义了API。

常见用例包括从依赖注入框架获取测试实例或调用静态工厂方法来创建测试类实例。

如果没有注册任何`TestInstanceFactory`，框架将只调用测试类的`唯一`构造方法来初始化它，并可能通过已注册的`ParameterResolver`来解析构造方法参数。

实现了`TestInstanceFactory`的扩展可以在测试接口、顶级测试类或者`@Nested`测试类上被注册。

> ⚠️ 在一个特定测试类中注册多个实现了`TestInstanceFactory`的扩展将会在该类、它的子类以及它的内嵌类的所有测试方法中引发异常。请注意，在超类或者封闭类中（即，在`@Nested`测试类的情况下）注册的任何`TestInstanceFactory`也都会被继承。所以，开发者有责任去确保在一个特定的测试类中只注册一个`TestInstanceFactory`。

### 5.5. 测试实例后处理
{{TestInstancePostProcessor}} 为希望发布流程测试实例的`Extensions`定义了API。

常见的用法涵盖了诸如将依赖注入到测试实例中，在测试实例中调用自定义的初始化方法等。

关于具体示例，请查阅 {{MockitoExtension}} 和 {{SpringExtension}} 的源代码。

### 5.6. 参数解析
{{ParameterResolver}} 定义了用于在运行时动态解析参数的`Extension`API。

如果测试构造器或者`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`或者`@AfterAll`方法接收参数，则必须在运行时通过`ParameterResolver`*解析* 该参数。开发人员可以使用内置的`ParameterResolver`（参考 {{TestInfoParameterResolver}}）或 [自己注册](#52-注册扩展)。一般而言，参数可能被按照其*名称*、*类型*、*注解* 或任何一种上述方式的组合所解析。具体示例可以参照 {{CustomTypeParameterResolver}} 和 {{CustomAnnotationParameterResolver}} 的源码。

> ⚠️  由于JDK 9之前的JDK版本中，由javac生成的字节代码存在错误，直接通过核心`java.lang.reflect.Parameter` API查找参数上的注解对于内部类构造函数总是会失败（例如，一个在`@Nested`测试类中构造函数）。
> <br/>    
> 因此，提供给`ParameterResolver`实现的 {{ParameterContext}} API包含以下用于正确查找参数注释的便捷方法。强烈建议扩展开发人员使用这些方法，而不去使用`java.lang.reflect.Parameter`中提供的方法，从而避免JDK中的这个错误。
>
> - `boolean isAnnotated(Class<? extends Annotation> annotationType)`
> - `Optional<A> findAnnotation(Class<A> annotationType)`
> - `List<A> findRepeatableAnnotations(Class<A> annotationType)`
 

### 5.7. 测试生命周期回调

下列接口定义了用于在测试执行生命周期的不同阶段来扩展测试的API。关于每个接口的详细信息，可以参考后续章节的示例，也可以查阅 {{extension-api-package}} 包中的Javadoc。

- {{BeforeAllCallback}}
	- {{BeforeEachCallback}}
		- {{BeforeTestExecutionCallback}}
		- {{AfterTestExecutionCallback}}
	- {{AfterEachCallback}}
- {{AfterAllCallback}}

> 📒 ***实现多个扩展API***  
> 扩展开发人员可以选择在单个扩展中实现任意数量的上述接口。具体示例请参阅 {{SpringExtension}} 的源代码。


#### 5.7.1. 测试执行之前和之后的回调
{{BeforeTestExecutionCallback}} 和 {{AfterTestExecutionCallback}} 分别为`Extensions`定义了添加行为的API，这些行为将在执行测试方法*之前* 和*之后立即执行*。因此，这些回调非常适合于定时器、跟踪器以及其他类似的场景。如果你需要实现围绕`@BeforeEach`和`@AfterEach`方法调用的回调，实现`BeforeEachCallback`和`AfterEachCallback`即可。

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

由于`TimingExtensionTests`类通过`@ExtendWith`注册了`TimingExtension`，所以，测试将在执行时应用这个计时器。

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

### 5.8. 异常处理

{{TestExecutionExceptionHandler}} 为`Extensions`定义了异常处理的API，从而可以处理在执行测试时抛出的异常。

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

### 5.9. 为测试模板提供调用上下文

当至少有一个 {{TestTemplateInvocationContextProvider}} 被注册时，标注了 {{TestTemplate}} 的方法才能被执行。每个这样的provider负责提供一个 {{TestTemplateInvocationContext}} 实例的`Stream`。每个上下文都可以指定一个自定义的显示名称和一个额外的扩展名列表，这些扩展名仅用于下一次调用 {{TestTemplate}} 方法。

以下示例展示了如何编写测试模板以及如何注册和实现一个 {{TestTemplateInvocationContextProvider}}。


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

在这个例子中，测试模板将被调用两次。调用的显示名称是调用上下文指定的"foo"和"bar"。每个调用都会注册一个自定义的 {{ParameterResolver}} 用于解析方法参数。下面是使用`ConsoleLauncher`时产生的输出信息。

```sh
└─ testTemplate(String) ✔
   ├─ foo ✔
   └─ bar ✔
```

{{TestTemplateInvocationContextProvider}} 扩展API主要用于实现不同类型的测试，这些测试依赖于某个类似于测试的方法的重复调用（尽管它们不在同一个上下文中）。 例如，使用不同的参数，以不同的方式准备测试类实例，或多次调用而不修改上下文。请参阅 [重复测试](#312-重复测试) 或 [参数化测试](#313-参数化测试) 的实现，它们都使用了该扩展点来提供其相关的功能。

### 5.10. 在扩展中保持状态

通常，扩展只实例化一次。随之而来的相关问题是：开发者如何能够在两次调用之间保持扩展的状态？`ExtensionContext` API提供了一个`Store`用来解决这一问题。扩展可以将值放入Store中供以后检索。请参阅 [`TimingExtension`](#一个为测试方法执行计时和记录的扩展) 了解如何使用具有方法级作用域的`Store`。要注意，在测试执行期间，被存储在一个`ExtensionContext`中的值在周围其他的`ExtensionContext`中是不可用的。由于`ExtensionContexts`可能是嵌套的，因此内部上下文的范围也可能受到限制。请参阅相应的Javadoc来了解有关通过 {{ExtensionContext_Store}} 存储和检索值的方法的详细信息。

### 5.11. 在扩展中支持的实用程序
`junit-platform-commons`公开了一个名为 {{org.junit.platform.commons.support}} 的包，它包含了用于处理注解、类、反射和类路径扫描任务且正在维护中的实用工具方法。`TestEngine`和`Extension`开发人员（authors）应该被鼓励去使用这些方法，以便与JUnit Platform的行为保持一致。
 
####  5.11.1. 注解支持
`AnnotationSupport`提供对注解元素（例如包、注解、类、接口、构造函数、方法和字段）进行操作的静态实用工具方法。这些方法包括检查元素是否使用特定注释进行注解或元注解，搜索特定注解以及如何在类或界面中查找注解的方法和字段。其中一些方法搜索已实现的接口和类层次结构以查找注解。有关更多详细信息，请参阅JavaDoc的 {{AnnotationSupport}}。

####  5.11.2. 类支持
`ClassSupport`提供静态工具方法来处理类（即`java.lang.Class`的实例）。有关详细信息，请参阅JavaDoc的 {{ClassSupport}}。


####  5.11.3. 反射支持
`ReflectionSupport`提供了静态实用工具方法，以增强标准的JDK反射和类加载机制。这些方法包括扫描类路径以搜索匹配了指定谓词的类，加载和创建类的新实例以及查找和调用方法。其中一些方法可以遍历类层次结构以找到匹配的方法。有关更多详细信息，请参阅JavaDoc的 {{ReflectionSupport}}。


### 5.12. 用户代码和扩展的相对执行顺序

当执行包含一个或多个测试方法的测试类时，除了用户提供的测试和生命周期方法外，还会调用大量的回调函数。 下图说明了用户提供的代码和扩展代码的相对顺序。

<a id="511-用户代码和扩展代码"></a>

![](http://junit.org/junit5/docs/5.2.0/user-guide/images/extensions_lifecycle.png)

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
