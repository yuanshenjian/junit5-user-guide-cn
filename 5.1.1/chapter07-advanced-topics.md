## 7. 高级主题

### 7.1 JUnit Platform启动器API
JUnit 5的主要目标之一是让JUnit与其编程客户端（构建工具和IDE）之间的接口更加强大和稳定。目的是将发现和执行测试的内部构件和外部必需的所有过滤和配置分离开来。

JUnit 5引入了`Launcher`的概念，它可以被用来发现、过滤和执行测试。此外，诸如 Spock、Cucumber和FitNesse等第三方测试库都可以通过提供自定义的 [`TestEngine`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/engine/TestEngine.html) 来集成到JUnit 5平台的启动基础设施中。

启动API在 [`junit-platform-launcher`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/launcher/package-summary.html) 模块中。 

[`junit-platform-console`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/console/package-summary.html)项目中的[`ConsoleLauncher`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/console/ConsoleLauncher.html)就是一个具体的使用例示。


#### 7.1.1 发现测试

将*测试发现* 作为平台本身的一个专用功能而引入，会（希望能够）在很大程度上解决过去IDE和构建工具难以识别测试类和测试方法的问题。

使用示例：

```java
import static org.junit.platform.engine.discovery.ClassNameFilter.includeClassNamePatterns;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectClass;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectPackage;

import org.junit.platform.launcher.Launcher;
import org.junit.platform.launcher.LauncherDiscoveryRequest;
import org.junit.platform.launcher.TestExecutionListener;
import org.junit.platform.launcher.TestPlan;
import org.junit.platform.launcher.core.LauncherDiscoveryRequestBuilder;
import org.junit.platform.launcher.core.LauncherFactory;
import org.junit.platform.launcher.listeners.SummaryGeneratingListener;
```

```java
LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
    .selectors(
        selectPackage("com.example.mytests"),
        selectClass(MyTestClass.class)
    )
    .filters(
        includeClassNamePatterns(".*Tests")
    )
    .build();

Launcher launcher = LauncherFactory.create();

TestPlan testPlan = launcher.discover(request);
```


目前，测试发现的搜索范围涵盖了类、方法、包中的所有类，甚至所有类路径中的测试。测试发现会贯穿所有参与的测试引擎。

生成的`TestPlan`是符合`LauncherDiscoveryRequest`对象的所有引擎、类、和测试方法的结构化（只读）描述。客户端可以遍历树，检索节点的详细信息，并获取到原始源的链接（如类，方法或文件位置）。测试计划中的每个节点都有一个*唯一的ID*，可以用它来调用特定的测试或一组测试。


#### 7.1.2 执行测试
要执行测试，客户端可以使用与发现阶段相同的`LauncherDiscoveryRequest`，或者创建一个新的请求。测试进度和报告可以通过使用`Launcher`注册一个或多个[`TestExecutionListener`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/launcher/TestExecutionListener.html)实现来获取，如下面例子所示。


```java
LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
    .selectors(
        selectPackage("com.example.mytests"),
        selectClass(MyTestClass.class)
    )
    .filters(
        includeClassNamePatterns(".*Tests")
    )
    .build();

Launcher launcher = LauncherFactory.create();

// 注册一个你选择的监听器
TestExecutionListener listener = new SummaryGeneratingListener();
launcher.registerTestExecutionListeners(listener);

launcher.execute(request);
```

`execute()`方法没有返回值，但你可以轻松地使用监听器将最终结果聚合到你自己的对象中。相关示例请参阅 [`SummaryGeneratingListener`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/launcher/listeners/SummaryGeneratingListener.html)。


#### 7.1.3 插入你自己的测试引擎

Junit 目前提供了两种开箱即用的 [`TestEngine`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/engine/TestEngine.html) ：

- [`junit-jupiter-engine`](http://junit.org/junit5/docs/5.1.1/api/org/junit/jupiter/engine/package-summary.html): JUnit Jupiter的核心。

- [`junit-vintage-engine`](http://junit.org/junit5/docs/5.1.1/api/org/junit/vintage/engine/package-summary.html): JUnit 4之上的一个薄层，它允许使用启动器基础设施来运行`老版本`的测试。


第三方也可以通过在 [`junit-platform-engine`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/engine/package-summary.html) 模块中实现接口并*注册* 引擎来提供他们自己的`TestEngine`。 目前Java的`java.util.ServiceLoader`机制支持引擎注册。 例如，`junit-jupiter-engine`模块将其`org.junit.jupiter.engine.JupiterTestEngine`注册到一个名为`org.junit.platform.engine.TestEngine`的文件中，该文件位于`junit-jupiter-engine`JAR包中的`/META-INF/services`目录。


#### 7.1.4 插入你自己的测试执行监听器
除了以编程方式来注册测试执行监听器的公共 [`Launcher`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/launcher/Launcher.html)  API方法之外，在运行时由Java的`java.util.ServiceLoader`发现的自定义 [`TestExecutionListener`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/launcher/TestExecutionListener.html) 实现会被自动注册到`DefaultLauncher`。 例如，一个实现了 [`TestExecutionListener`](http://junit.org/junit5/docs/5.1.1/api/org/junit/platform/launcher/TestExecutionListener.html) 并声明在`/META-INF/services/org.junit.platform.launcher.TestExecutionListener`文件中的`example.TestInfoPrinter`类会被自动加载和注册。
