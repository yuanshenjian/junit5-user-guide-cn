## 6. 从JUnit4迁移
虽然JUnit Jupiter编程模型和扩展模型本身不支持`Rules`和`Runners`等JUnit 4特性，但我们不期望源码维护者必须更新其现有的所有测试、测试扩展以及自定义构建测试基础设施，从而迁移到JUnit Jupiter。

然而，JUnit通过*JUnit Vintage测试引擎* 提供了一个平缓的迁移路径，该引擎允许使用JUnit Platform基础设施执行基于JUnit3和JUnit4的现有测试。由于JUnit Jupiter 特有的所有类和注解位于新的`org.junit.jupiter`基础包中，因此在类路径中同时使用JUnit 4和JUnit Jupiter不会导致任何冲突。所以，保持现有的JUnit 4测试和JUnit Jupiter测试是安全的。除此之外，JUnit团队会持续为JUnit 4.x 基线提供维护和错误修复的版本，所以开发人员有足够的时间按照自己的计划到迁移到JUnit Jupiter。

### 6.1. 在 JUnit Platform 上运行JUnit4 测试
只要确保`junit-vintage-engine`包存在于你的测试运行时路径下。在这种情况下，基于 JUnit3 和 JUnit4 的测试将自动被JUnit Platform启动器拾取。

要想了解如何使用Gradle和Maven完成此操作，请参阅示例工程 [junit5-samples](https://github.com/junit-team/junit5-samples) 。

### 6.2. 迁移技巧
以下是在将现有JUnit 4测试迁移到JUnit Jupiter时必须注意的事项。

* `org.junit.jupiter.api`包中的注解。

* `org.junit.jupiter.api.Assertions`类中的断言。

* `org.junit.jupiter.api.Assumptions`类中的假设。

* `@Before`和`@After`已经不存在; 取而代之的是`@BeforeEach`和`@AfterEach`。

* `@BeforeClass`和`@AfterClass`已经不存在; 取而代之的是`@BeforeAll`和`@AfterAll`。

* `@Ignore` 已经不存在: 取而代之的是 `@Disabled`。
* `@Category` 已经不存在: 取而代之的是 `@Tag`。
* `@RunWith` 已经不存在: 取而代之的是`@ExtendWith`。
* `@Rule`和 `@ClassRule`已经不存在; 取而代之的是`@ExtendWith`; 关于部分规则的支持请参阅后续章节。

### 6.3. 对JUnit4规则的有限支持

如前文所述，JUnit Jupiter本身不支持JUnit 4规则。然而，JUnit团队意识到：很多组织，尤其是大型组织，很可能拥有使用自定义规则的大型JUnit 4代码库。为了给这些组织提供服务并实现平缓地迁移，JUnit团队决定在JUnit Jupiter中逐步地支持JUnit 4规则。这种支持是基于适配器的，并且仅限于那些与JUnit Jupiter扩展模型在语义上兼容的规则，即那些不会完全改变测试总体执行流程的规则。

JUnit Jupiter中的`junit-jupiter-migrationsupport`模块目前支持以下三种规则类型以及它们的子类。

* `org.junit.rules.ExternalResource` (包含 `org.junit.rules.TemporaryFolder`)

* `org.junit.rules.Verifier` (包含`org.junit.rules.ErrorCollector`)

* `org.junit.rules.ExpectedException`

As in JUnit 4, Rule-annotated fields as well as methods are supported. By using these class-level extensions on a test class such Rule implementations in legacy codebases can be left unchanged including the JUnit 4 rule import statements.

跟在JUnit 4中一样，规则注解的字段跟方法一样是被支持的。通过在测试类使用这些类级别的扩展，可以*保留*遗留代码库中的规则实现，其中包括JUnit4规则导入语句。

这种有限的`Rule`支持形式可以通过类级的注解`org.junit.jupiter.migrationsupport.rules.EnableRuleMigrationSupport`来开启。该注解是一个组合注解，它会启用所有支持迁移的扩展：`VerifierSupport`、`ExternalResourceSupport` 和 `ExpectedExceptionSupport`

然而，如果你打算开发一个新的JUnit 5扩展，请使用JUnit Jupiter的新扩展模型，而不要再去使用JUnit 4中基于规则的模型。

> ⚠️ JUnit Jupiter中的JUnit 4`Rule`支持目前是一个实验性功能。详细信息请参阅 [试验性API](#82-试验性api)
