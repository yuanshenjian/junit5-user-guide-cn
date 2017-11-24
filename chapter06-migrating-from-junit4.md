## 6. 从JUnit4迁移
虽然JUnit Jupiter编程模型以及扩展模型本身不支持JUnit4 的一些特性，诸如：`Rules` 和 `Runners`，但我们也不期望源码维护者就必须将它们所有的测试、测试扩展以及定制化构建测试基础设施全部迁移到 JUnit Jupiter。

然而，JUnit通过 *JUnit Vintage测试引擎* 提供了一个平缓的迁移路径，该引擎能够允许那些基于 JUnit3 和 JUnit4 的已存在测试可以在 JUnit 平台下执行。由于 JUnit Jupiter 所有类和注解的规范存在于 `org.junit.jupiter` 基础包中，JUnit4 和 JUnit Jupiter 的测试同时存在类路径中就不会产生冲突了。因此，维护已存在的 JUnit4 测试和 JUnit Jupiter 测试是安全的。除此之外，JUnit 团队会持续为 JUnit4.x 基线提供维护和 bug 修复的版本发布，所以开发人员将有大量的时间可以按照自己的进度去完成到 JUnit Jupiter 的迁移。

### 6.1. 在 JUnit Platform 上运行JUnit4 测试
Just make sure that the junit-vintage-engine artifact is in your test runtime path. In that case JUnit 3 and JUnit 4 tests will automatically be picked up by the JUnit Platform launcher.
只要确保 `junit-vintage-engine` 包存在于你的测试运行时路径下。这样一来，基于 JUnit3 和 JUnit4 的测试将自动被 JUnit Platform 加载器加载。

具体如何实现，可以参考 [junit5-samples](https://github.com/junit-team/junit5-samples) 仓库中那些以Gradle和Maven来实现的样例工程。

### 6.2. 迁移技巧
以下是当你在将现存的 JUnit4 测试迁移到 JUnit Jupiter上的时候要注意的东西：

* `org.junit.jupiter.api` 包中的注解。

* `org.junit.jupiter.api.Assertions` 类中的断言。

* `org.junit.jupiter.api.Assumptions` 类中的假设。

* `@Before` 和 `@After` 已经不存在; 取而代之的是`@BeforeEach` 和 `@AfterEach`。

* `@BeforeClass` 和 `@AfterClass` 已经不存在; 取而代之的是 `@BeforeAll` 和 `@AfterAll`。

* `@Ignore` 已经不存在: 取而代之的是 `@Disabled`。
* `@Category` 已经不存在: 取而代之的是 `@Tag`。
* `@RunWith` 已经不存在: 取而代之的是`@ExtendWith`。
* `@Rule` 和 `@ClassRule` 已经不存在; 取而代之的是`@ExtendWith`; 请参阅后续章节关于对JUnit规则的有限支持。

### 6.3. 对JUnit4规则的有限支持

如上一章节所述，JUnit Jupiter 不再原生支持 JUnit4 的规则。然而，JUnit团队意识到一点：很多组织，尤其是一些大组织，很可能已经拥用一个包含了自定义规则且很庞大的JUnit4 代码库。为了给这些组织提供服务，并提供一个平缓的迁移路线，JUnit团队已经决定在 JUnit Jupiter 中逐步地支持 JUnit4 中的规则。这些支持是基于适配器的，并且仅限于那些语义上兼容 JUnit Jupiter 扩展模型的规则，即那些不会完全改变整个执行流的测试：

JUnit Jupiter 中的 `junit-jupiter-migrationsupport` 模块目前支持以下三种规则类型以及它们的子类：

* `org.junit.rules.ExternalResource` (包含 `org.junit.rules.TemporaryFolder`)

* `org.junit.rules.Verifier` (包含`org.junit.rules.ErrorCollector`)

* `org.junit.rules.ExpectedException`

As in JUnit 4, Rule-annotated fields as well as methods are supported. By using these class-level extensions on a test class such Rule implementations in legacy codebases can be left unchanged including the JUnit 4 rule import statements.

因为在JUnit4中，规则注解的字段跟方法一样是被支持的。通过在测试类使用这些类级别的扩展，这种在遗留代码库中规则实现*可以继续保留下来*，其中还包括JUnit4规则import语句。

这种有限形式的 `Rule` 支持通过类级别的注解`org.junit.jupiter.migrationsupport.rules.EnableRuleMigrationSupport`来开启。该注解是一个组合注解，它会开启所有支持迁移的扩展：`VerifierSupport`、`ExternalResourceSupport` 和 `ExpectedExceptionSupport`

然而，如果你想开发一个新的 JUnit5 扩展，请使用 JUnit Jupiter 中新的扩展模型，而不要去使用JUnit4中基于规则的模型。

> ⚠️ 目前，在 JUnit Jupiter 中支持 JUnit4 的 `Rule` 还在试验阶段。更多细节见 [Experimental APIs](http://junit.org/junit5/docs/current/user-guide/#api-evolution-experimental-apis)
