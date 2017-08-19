# 7. 高级特性

## 7.1 JUnit平台加载器API

JUnit 5最突出的一个目标是提升JUnit平台和编程客户端--项目构建工具以及IDE--之间的接口功能和稳定性。这么做的目的是将发现和执行测试的内部构件从外部所有必要的过滤和配置中解耦出来。

JUnit 5 引入了`Launcher`的概念，它可以被用来发现、过滤，以及执行测试。除此以外，诸如 Spock、Cucumber 和 FitNesse 等第三方测试库都可以通过提供自定义的[`TestEngine`](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) 方式被集成到JUnit 5平台的启动基础设施中。

Launching API在[`junit-platform-launcher`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/package-summary.html)模块中。 

[`junit-platform-console`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/package-summary.html)项目中的[`ConsoleLauncher`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)就是一个具体示的例示。

### 7.1.1 测试发现

引入*测试发现*作为平台本身的一个重要特性，以便（希望）能够从过去识别测试类和测试方法时遇到的大多数困难中，将IDE和项目构建工具解放出来。

使用示例：

```java
import static org.junit.platform.engine.discovery.ClassNameFilter.includeClassNamePatterns;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectClass;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectPackage;

import org.junit.jupiter.api.Test;
import org.junit.platform.launcher.Launcher;
import org.junit.platform.launcher.LauncherDiscoveryRequest;
import org.junit.platform.launcher.TestExecutionListener;
import org.junit.platform.launcher.TestPlan;
import org.junit.platform.launcher.core.LauncherDiscoveryRequestBuilder;
import org.junit.platform.launcher.core.LauncherFactory;
import org.junit.platform.launcher.listeners.SummaryGeneratingListener;

LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
    .selectors(
        selectPackage("com.example.mytests"),
        selectClass(MyTestClass.class)
    )
    .filters(includeClassNamePatterns(".*Test"))
    .build();

TestPlan plan = LauncherFactory.create().discover(request);

```

目前，搜索范围可能涵盖了类、方法、包中的所有类，甚至所有路径中的测试。测试发现会贯穿所有参与的测试引擎。

产生的测试计划是所有符合`specification`对象的引擎、类、和测试方法的结构化（只读）描述。客户端可以遍历树，检索节点的信息，并获取到原始源的链接（如类，方法或文件位置）。测试计划树中的每个节点都有一个*唯一的ID*，可以用于调用特定的测试或一组测试。

### 7.1.2 执行测试

执行测试有两种方法。一种是使用与发现步骤中相同的测试规范对象，或者使用另一种稍快一些的方式，通过先前发现步骤中准备好的`TestPlan`对象完成。测试的进度和结果报告可以通过[`TestExecutionListener`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/TestExecutionListener.html)获取：

```java
LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
    .selectors(
        selectPackage("com.example.mytests"),
        selectClass(MyTestClass.class)
    )
    .filters(includeClassNamePatterns(".*Test"))
    .build();

Launcher launcher = LauncherFactory.create();

// Register a listener of your choice
TestExecutionListener listener = new SummaryGeneratingListener();
launcher.registerTestExecutionListeners(listener);

launcher.execute(request);
```

目前没有结果对象，但是你可以轻松地使用监听器将最终结果聚合到你自己的对象中。 相关示例，请参阅[`SummaryGeneratingListener`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/listeners/SummaryGeneratingListener.html)。

### 7.1.3 集成自定义测试引擎插件

Junit 目前提供了两种开箱即用的 [`TestEngine`](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) 的：

- [`junit-jupiter-engine`](http://junit.org/junit5/docs/current/api/org/junit/jupiter/engine/package-summary.html): JUnit Jupiter 的核心。

- [`junit-vintage-engine`](http://junit.org/junit5/docs/current/api/org/junit/vintage/engine/package-summary.html): 在JUnit 4之上的一个薄层，它允许在启动基础架构中运行`旧版本`的测试。

第三方也可以通过在[`junit-platform-engine`](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/package-summary.html)模块中实现接口并*注册*引擎来提供自己的`TestEngine`。 目前Java的`java.util.ServiceLoader`机制支持引擎注册。 例如，`junit-jupiter-engine`模块将`org.junit.jupiter.engine.JupiterTestEngine`注册到`junit-jupiter-engine`JAR中的`/ META-INF / services`内的名为`org.junit.platform.engine.TestEngine`的文件中。

### 7.1.4 引入自定义的测试执行监听器

除了以编程方式注册测试执行监听器的公共[`Launcher`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/Launcher.html) API方法外，被Java`java.util.ServiceLoader`工具在运行时发现的自定义[`TestExecutionListener`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/TestExecutionListener.html)实现也会以`DefaultLauncher`的形式被自动注册。 例如，一个实现[`TestExecutionListener`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/TestExecutionListener.html)并在`/META-INF/services/org.junit.platform.launcher.TestExecutionListener`文件中声明的`example.TestInfoPrinter`类会被自动加载和注册。
