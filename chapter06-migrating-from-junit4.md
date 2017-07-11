# 6. 从JUnit4迁移
虽然JUnit Jupiter编程模型以及扩展模型本身不支持JUnit4的某些特性，诸如：`Rules`和`Runners`，但我们也不期望源码维护者就必须将它们所有的测试、测试扩展以及定制化构建测试基础设施全部迁移到JUnit Jupiter。

然而，JUnit通过*JUnit Vintage测试引擎*提供了一个平缓的迁移路径，该引擎能够允许那些基于JUnit3和JUnit4的已存在测试可以在JUnit平台下执行。由于JUnit Jupiter所有类和注解的规范存在于`org.junit.jupiter`基础包中，JUnit4和JUnit Jupiter的测试同时存在类路径中就不会产生冲突了。因此，维护已存在的JUnit4测试和JUnit Jupiter测试是安全的。除此之外，JUnit团队会继续为JUnit4.x基线提供维护和bug修复的版本发布，所以开发人员将有大量的时间可以按照自己的进程去完成到JUnit Jupiter的迁移。


## 6.1. Running JUnit 4 Tests on the JUnit Platform
Just make sure that the junit-vintage-engine artifact is in your test runtime path. In that case JUnit 3 and JUnit 4 tests will automatically be picked up by the JUnit Platform launcher.

See the example projects in the junit5-samples repository to find out how this is done with Gradle and Maven.

## 6.2. Migration Tips
The following are things you have to watch out for when migrating existing JUnit 4 tests to JUnit Jupiter.

Annotations reside in the org.junit.jupiter.api package.

Assertions reside in org.junit.jupiter.api.Assertions.

Assumptions reside in org.junit.jupiter.api.Assumptions.

@Before and @After no longer exist; use @BeforeEach and @AfterEach instead.

@BeforeClass and @AfterClass no longer exist; use @BeforeAll and @AfterAll instead.

@Ignore no longer exists: use @Disabled instead.

@Category no longer exists; use @Tag instead.

@RunWith no longer exists; superseded by @ExtendWith.

@Rule and @ClassRule no longer exist; superseded by @ExtendWith; see the following section for partial rule support.

## 6.3. Limited JUnit 4 Rule Support
As stated above, JUnit Jupiter does not and will not support JUnit 4 rules natively. The JUnit team realizes, however, that many organizations, especially large ones, are likely to have large JUnit 4 codebases including custom rules. To serve these organizations and enable a gradual migration path the JUnit team has decided to support a selection of JUnit 4 rules verbatim within JUnit Jupiter. This support is based on adapters and is limited to those rules that are semantically compatible to the JUnit Jupiter extension model, i.e. those that do not completely change the overall execution flow of the test.

JUnit Jupiter currently supports the following three Rule types including subclasses of those types:

org.junit.rules.ExternalResource (including org.junit.rules.TemporaryFolder)

org.junit.rules.Verifier (including org.junit.rules.ErrorCollector)

org.junit.rules.ExpectedException

As in JUnit 4, Rule-annotated fields as well as methods are supported. By using these class-level extensions on a test class such Rule implementations in legacy codebases can be left unchanged including the JUnit 4 rule import statements.

This limited form of Rule support can be switched on by the class-level annotation org.junit.jupiter.migrationsupport.rules.EnableRuleMigrationSupport. This annotation is a composed annotation which enables all migration support extensions: VerifierSupport, ExternalResourceSupport, and ExpectedExceptionSupport.

However, if you intend to develop a new extension for JUnit 5 please use the new extension model of JUnit Jupiter instead of the rule-based model of JUnit 4.