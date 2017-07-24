# 7. 高级特性

## 7.1 JUnit平台加载器API

JUnit 5 的目标之一就是为用户提供其项目构建工具、IDE和JUnit平台间更加强大和稳健的接口。如此一来，就可以从外部所有的过滤文件和一些必须的配置文件中将发现和执行测试的内部解耦出来。

JUnit 5 引入了`Launcher`的概念，它可以被用来发现、过滤，以及执行测试。除此以外，诸如 Spock、Cucumber 和 FitNesse 的第三方测试库都可以通过提供定制的[`TestEngine`](http://junit.org/junit5/docs/current/api/org/junit/platform/engine/TestEngine.html) 来插入 JUnit 5 平台的加载基础设施。

加载API在[`junit-platform-launcher`](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/package-summary.html)的标准组件中。 

加载API的一个具体示例示[`junit-platform-console`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/package-summary.html)项目中的[`ConsoleLauncher`](http://junit.org/junit5/docs/current/api/org/junit/platform/console/ConsoleLauncher.html)

### 7.1.1 发现测试

引入 *测试发现* 作为平台本身的一个重要特性（希望）以便从过去确定测试类和测试方法时遇到的大多数困难中，将 IDE 和 项目构建工具解放出来。

使用示例：

```
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

