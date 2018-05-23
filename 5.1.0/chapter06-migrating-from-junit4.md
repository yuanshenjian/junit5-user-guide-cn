## 6. 从JUnit4迁移
虽然JUnit Jupiter编程模型和扩展模型本身不支持JUnit 4中的`Rules`和`Runners`等特性，但我们不期望源码维护者为了迁移到JUnit Jupiter，而必须更新其现有的所有测试、测试扩展以及自定义构建测试基础设施。

然而，JUnit通过一个*JUnit Vintage测试引擎* 提供了一条平缓的迁移路径，该引擎允许使用JUnit Platform基础设施执行基于JUnit 3和JUnit 4的现有测试。由于所有JUnit Jupiter特有的类和注解都位于新的`org.junit.jupiter`基础包中，因此在类路径中同时使用JUnit 4和JUnit Jupiter不会导致任何冲突。所以，保持现有的JUnit 4测试和JUnit Jupiter测试是安全的。除此之外，JUnit团队会持续为JUnit 4.x 基线提供维护和错误修复的版本，所以开发人员有足够的时间按照自己的计划迁移到JUnit Jupiter。

### 6.1. 在 JUnit Platform 上运行JUnit4 测试
只要确保`junit-vintage-engine`包存在于你的测试运行时路径下，基于JUnit 3和 JUnit 4的测试将自动被JUnit Platform启动器拾取。

要想了解如何使用Gradle和Maven完成此操作，请参阅示例工程 [junit5-samples]({{junit5-samples-repo}})。

#### 6.1.1. 类别支持
对于使用`@Category`注解的测试类或方法，*JUnit Vintage* 测试引擎将该类别的完全限定类名作为相应测试标识符的标记。例如，如果一个测试方法使用了`@Category(Example.class)`注解，它将被标记为`"com.acme.Example"`。与JUnit 4中的`Categories` runner类似，我们可以使用该信息在执行发现的测试之前对其进行过滤（详细信息请参阅 [运行测试](#4-运行测试)）。

### 6.2. 迁移技巧
以下是在将现有JUnit 4测试迁移到JUnit Jupiter时必须注意的事项。

* `org.junit.jupiter.api`包中的注解。

* `org.junit.jupiter.api.Assertions`类中的断言。

* `org.junit.jupiter.api.Assumptions`类中的假设。

* `@Before`和`@After`已经不存在；取而代之的是`@BeforeEach`和`@AfterEach`。

* `@BeforeClass`和`@AfterClass`已经不存在；取而代之的是`@BeforeAll`和`@AfterAll`。

* `@Ignore` 已经不存在：取而代之的是 `@Disabled`。
* `@Category` 已经不存在：取而代之的是 `@Tag`。
* `@RunWith` 已经不存在：取而代之的是`@ExtendWith`。
* `@Rule`和 `@ClassRule`已经不存在；取而代之的是`@ExtendWith`；关于部分规则的支持请参阅后续章节。

### 6.3. 对JUnit4规则的有限支持
如前文所述，JUnit Jupiter本身不支持JUnit 4的`Rule`。然而，JUnit团队也意识到：很多组织，尤其是大型组织，很可能拥有使用自定义规则的大型JUnit 4代码库。为了给这些组织提供服务并实现平缓地迁移，JUnit团队决定在JUnit Jupiter中逐步地支持部分JUnit 4的`Rule`。这种支持是基于适配器的，并且仅限于那些与JUnit Jupiter扩展模型在语义上兼容的`Rule`，即那些不会完全改变测试总体执行流程的`Rule`。

JUnit Jupiter中的`junit-jupiter-migrationsupport`模块目前支持以下三种`Rule`类型以及它们的子类。

* `org.junit.rules.ExternalResource` (包含 `org.junit.rules.TemporaryFolder`)
* `org.junit.rules.Verifier` (包含`org.junit.rules.ErrorCollector`)
* `org.junit.rules.ExpectedException`

跟在JUnit 4中一样，规则注解的字段跟方法一样是被支持的。通过在测试类使用这些类级别的扩展，可以*保留* 遗留代码库中的规则实现，其中包括JUnit4规则导入语句。

这种有限的`Rule`支持形式可以通过类级的注解`org.junit.jupiter.migrationsupport.rules.EnableRuleMigrationSupport`来开启。该注解是一个组合注解，它会启用所有支持迁移的扩展：`VerifierSupport`、`ExternalResourceSupport`和`ExpectedExceptionSupport`。

然而，如果你打算开发一个新的JUnit 5扩展，请使用JUnit Jupiter的新扩展模型，而不要再去使用JUnit 4中基于`Rule`的模型。

> ⚠️ JUnit Jupiter中的JUnit 4 `Rule`支持目前是一个实验性功能。详细信息请参阅 [试验性API](#82-试验性api)
