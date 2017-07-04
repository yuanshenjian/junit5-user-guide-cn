# 3. 编写测试

*第一个测试用例*

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class FirstJUnit5Tests {

    @Test
    void myFirstTest() {
        assertEquals(2, 1 + 1);
    }

}
```

## 3.1. 注解
JUnit Jupiter 支持使用下面表格中所列的注解来配置测试及扩展框架。

所有的核心注解位于`junit-jupiter-api`模块的 [org.junit.jupiter.api ](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/package-summary.html) 包中。

| 注解          | 描述 |
|:--------------|:------------|
| @Test         | 表示该该方法是一个测试方法。不像JUnit4的 @Test 注解，该注解没有声明任何属性，由于JUnit Jupiter操作中，测试扩展都是基于它们自身专用的注解 |
| @RepeatedTest | 表示该方法是一个[重复测试]()的测试模板 |
| @TestFactory  | 表示该方法是一个[动态测试]()的测试工厂 |
| @DisplayName  | 为测试类或测试方法声明一个定制化的展示名字 |
| @BeforeEach   | 使用该注解的方法应该在当前类中每一个使用了`@Test`注解的方法之前执行；类似于JUnit4的 `@Before`，该方法是可被继承的 |
| @AfterEach    | 使用该注解的方法应该在当前类中每一个使用了`@Test`注解的方法之后执行；类似于JUnit4的 `@After`，该方法是可被继承的 |
| @BeforeAll    | 使用该注解的方法应该在当前类中所有使用了`@Test`注解的方法之前执行；类似于JUnit4的 `@BeforeClass`，该方法必须是 `static`方法，它也可以被继承 |
| @AfterAll     | 使用该注解的方法应该在当前类中所有使用了`@Test`注解的方法之后执行；类似于JUnit4的 `@AfterClass`，该方法必须是 `static`方法，它也可以被继承 |
| @Nested       | 使用该注解的类是一个内嵌、非静态的测试类。受限于Java语言，`@BeforeAll`和`@AfterAll`方法不能在`@Nested`测试类中使用 |
| @Tag          | 声明用于过滤测试的*tags*，该注解可以用在方法或类上；类似于TesgNG的测试组以及JUnit4的分类。
| @Disable      | 禁用一个测试类或测试方法；类似于JUnit4的`@Ignore` |
| @ExtendWith   | 注册自定义[扩展]() |


### 3.1.1. 元注解即组合注解
JUnit Jupiter注解可以被用作元注解。这意味着你可以定义你自己的组合注解，该注解会自动继承其元注解的语义。

例如，为了避免在代码库中到处复制粘贴`@Tag("fast")`（见 [标记和过滤]()），你可以创建一个名为`@Fast`自定义*组合注解*。`@Fast`就可以被用来替换`@Tag("fast")`了。如下面代码所示：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.junit.jupiter.api.Tag;

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Tag("fast")
public @interface Fast {
}
```


## 3.2. 标准测试类
*一个标准的测试用例*

```java
import static org.junit.jupiter.api.Assertions.fail;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class StandardTests {

    @BeforeAll
    static void initAll() {
    }

    @BeforeEach
    void init() {
    }

    @Test
    void succeedingTest() {
    }

    @Test
    void failingTest() {
        fail("a failing test");
    }

    @Test
    @Disabled("for demonstration purposes")
    void skippedTest() {
        // not executed
    }

    @AfterEach
    void tearDown() {
    }

    @AfterAll
    static void tearDownAll() {
    }

}
```

> Note: 测试类和测试方法都可以不是 `public`


## 3.3. 展示名字
测试类和测试方法可以声明自定义的显示名字--空格、特殊字符以及emojis表情--都可以显示在测试运行器和测试报告中。

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

@DisplayName("A special test case")
class DisplayNameDemo {

    @Test
    @DisplayName("Custom test name containing spaces")
    void testWithDisplayNameContainingSpaces() {
    }

    @Test
    @DisplayName("╯°□°）╯")
    void testWithDisplayNameContainingSpecialCharacters() {
    }

    @Test
    @DisplayName("😱")
    void testWithDisplayNameContainingEmoji() {
    }

}
```

## 3.4. 断言
JUnit Jupiter自带了很多JUnit4就已经存在的断言方法，以及添加了一些在Java8中更好用的断言。JUnit Jupiter中所有断言都是`static`方法，它们存在 [org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html)于类中。

```java
import static java.time.Duration.ofMillis;
import static java.time.Duration.ofMinutes;
import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTimeout;
import static org.junit.jupiter.api.Assertions.assertTimeoutPreemptively;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;

class AssertionsDemo {

    @Test
    void standardAssertions() {
        assertEquals(2, 2);
        assertEquals(4, 4, "The optional assertion message is now the last parameter.");
        assertTrue(2 == 2, () -> "Assertion messages can be lazily evaluated -- "
                + "to avoid constructing complex messages unnecessarily.");
    }

    @Test
    void groupedAssertions() {
        // In a grouped assertion all assertions are executed, and any
        // failures will be reported together.
        assertAll("address",
            () -> assertEquals("John", address.getFirstName()),
            () -> assertEquals("User", address.getLastName())
        );
    }

    @Test
    void exceptionTesting() {
        Throwable exception = assertThrows(IllegalArgumentException.class, () -> {
            throw new IllegalArgumentException("a message");
        });
        assertEquals("a message", exception.getMessage());
    }

    @Test
    void timeoutNotExceeded() {
        // The following assertion succeeds.
        assertTimeout(ofMinutes(2), () -> {
            // Perform task that takes less than 2 minutes.
        });
    }

    @Test
    void timeoutNotExceededWithResult() {
        // The following assertion succeeds, and returns the supplied object.
        String actualResult = assertTimeout(ofMinutes(2), () -> {
            return "a result";
        });
        assertEquals("a result", actualResult);
    }

    @Test
    void timeoutNotExceededWithMethod() {
        // The following assertion invokes a method reference and returns an object.
        String actualGreeting = assertTimeout(ofMinutes(2), AssertionsDemo::greeting);
        assertEquals("hello world!", actualGreeting);
    }

    @Test
    void timeoutExceeded() {
        // The following assertion fails with an error message similar to:
        // execution exceeded timeout of 10 ms by 91 ms
        assertTimeout(ofMillis(10), () -> {
            // Simulate task that takes more than 10 ms.
            Thread.sleep(100);
        });
    }

    @Test
    void timeoutExceededWithPreemptiveTermination() {
        // The following assertion fails with an error message similar to:
        // execution timed out after 10 ms
        assertTimeoutPreemptively(ofMillis(10), () -> {
            // Simulate task that takes more than 10 ms.
            Thread.sleep(100);
        });
    }

    private static String greeting() {
        return "hello world!";
    }

}
```

### 3.4.1. 第三方断言类库
虽然JUnit Jupiter提供的断言工具包已经满足了很多测试场景，但有时候我们会遇到需要更加强大且具备例如*匹配器*功能的场景。这个时候，JUnit团队推荐使用第三方断言类库，例如：[AssertJ](http://joel-costigliola.github.io/assertj/)、[Hamcrest](http://hamcrest.org/JavaHamcrest/)、[Truth](http://google.github.io/truth/) 等等。所以说，使用哪个断言类库完全取决于开发人员自己的喜好。

举个例子，*匹配器*和一个流式调用的API的组合可以使得断言更加具有描述性和可读性。然而，JUnit Jupiter的 [org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html)类没有提供类似于JUnit4`org.junit.Assert`类中的 [assertThat()](http://junit.org/junit4/javadoc/latest/org/junit/Assert.html#assertThat) 方法，我们知道该方法能够接受一个 Hamcrest [Matcher](http://junit.org/junit4/javadoc/latest/org/hamcrest/Matcher.html)。所以，我们鼓励开发人员去使用第三方断言类库提供的內建匹配器。

下面的例子演示如何在JUnit Jupiter中使用Hamcrest提供的`assertThat()`。只要Hamcrest库已经被添加到classpath中，你就可以静态导入诸如`assertThat()`、`is()`以及`equalTo()`方法，然后在测试方法中使用它们，如下面代码所示的`assertWithHamcrestMatcher()`方法：

```java
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

import org.junit.jupiter.api.Test;

class HamcrestAssertionDemo {

    @Test
    void assertWithHamcrestMatcher() {
        assertThat(2 + 1, is(equalTo(3)));
    }

}
```
当然，那些基于JUnit4编程模型的遗留测试可以继续使用`org.junit.Assert#assertThat`。


## 3.5. 假设
JUnit Jupiter自带了JUnit4所提供的假设方法的一个子集，以及添加了一些在Java8中更好用的假设。JUnit Jupiter中所有假设都是静态方法，它们存在于 [org.junit.jupiter.Assumptions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assumptions.html) 类中。

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

import org.junit.jupiter.api.Test;

public class AssumptionsDemo {

    @Test
    void testOnlyOnCiServer() {
        assumeTrue("CI".equals(System.getenv("ENV")));
        // remainder of test
    }

    @Test
    void testOnlyOnDeveloperWorkstation() {
        assumeTrue("DEV".equals(System.getenv("ENV")),
            () -> "Aborting test: not on developer workstation");
        // remainder of test
    }

    @Test
    void testInAllEnvironments() {
        assumingThat("CI".equals(System.getenv("ENV")),
            () -> {
                // perform these assertions only on the CI server
                assertEquals(2, 2);
            });

        // perform these assertions in all environments
        assertEquals("a string", "a string");
    }

}
```
## 3.6. 禁用测试
下面是一个被禁用的测试用例：

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

@Disabled
class DisabledClassDemo {
    @Test
    void testWillBeSkipped() {
    }
}
```

下面是一个包含被禁用测试方法的测试用例：

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class DisabledTestsDemo {

    @Disabled
    @Test
    void testWillBeSkipped() {
    }

    @Test
    void testWillBeExecuted() {
    }
}
```

## 3.7. 标记和过滤
测试类和测试方法可以被标记。那些标签可以在后面被用来过滤 [测试发现和执行]()

```java
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

@Tag("fast")
@Tag("model")
class TaggingDemo {

    @Test
    @Tag("taxes")
    void testingTaxCalculation() {
    }

}
```

## 3.8. 内嵌测试
内嵌测试使得测试编写者能够表示出几组测试用例之间的关系。下面来看一个精心设计的例子。

*一个用于测试栈的内嵌测试套件*

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.EmptyStackException;
import java.util.Stack;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, () -> stack.pop());
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, () -> stack.peek());
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

> Note: 用作`@Nested`的测试只能是非静态的内嵌类（i.e. 内部类）。内嵌可以是任意深度，那些内部类会被认为是一个该测试类家庭中的成员，但有一种特殊情况：`@BeforeAll`和`@AfterAll`，因为Java不允许内部类中存在`static`成员。

## 3.9. 构造器和方法的依赖注入
JUnit之前所有的版本中，测试构造器和方法是不允许传入参数的（至少标准的`Runner`实现是不允许的）。JUnit Jupiter一个主要的改变是：测试类的构造器和方法都允许传入参数了。这带来了更大的灵活性，并且可以在构造器和方法上使用`依赖注入`。

[ParameterResolver](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/extension/ParameterResolver.html) 为测试扩展定义了API，它可以在运行时*动态*解析参数。如果一个测试的构造器或者`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`或者 `@AfterAll`方法接收一个参数，这个参数就必须在运行时被一个已注册的`ParameterResolver`所解析。

目前有三种被自动注册的內建的解析器。

* [TestInfoParameterResolver](https://github.com/junit-team/junit5/tree/r5.0.0-M4/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)：如果一个方法参数的类型是 [TestInfo](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestInfo.html)，`TestInfoParameterResolver`将根据当前的测试提供一个`TestInfo`的实例用于填充参数的值。然后`TestInfo`就可以被用来检索关于当前测试的信息，例如：展示名、测试类、测试方法或相关的标记。展示名要么是一个类似于测试类或测试方法的技术名称，要么是一个通过`@DisplayName`配置的自定义名称。

 [TestInfo](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestInfo.html)就像JUnit4规则中`TestName`规则的代替者。下面是来看一个示例：

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;

class TestInfoDemo {

    @BeforeEach
    void init(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
    }

    @Test
    @DisplayName("TEST 1")
    @Tag("my tag")
    void test1(TestInfo testInfo) {
        assertEquals("TEST 1", testInfo.getDisplayName());
        assertTrue(testInfo.getTags().contains("my tag"));
    }

    @Test
    void test2() {
    }

}
```

* `RepetitionInfoParameterResolver`：如果一个位于`@RepeatedTest`、`@BeforeEach`或者`@AfterEach`方法的参数的类型是 [RepetitionInfo](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/RepetitionInfo.html)，`RepetitionInfoParameterResolver`会提供一个`RepetitionInfo`实例。然后`RepetitionInfo`就可以被用来检索对应`@RepeatedTest`方法的当前重复以及总重复次数等相关信息。注意，`RepetitionInfoParameterResolver`不是在`@RepeatedTest`方法上下文外部被注册的。参考[重复测试示例]()
* [TestReporterParameterResolver](https://github.com/junit-team/junit5/tree/r5.0.0-M4/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestReporterParameterResolver.java)：如果一个方法参数的类型是 [TestReporter](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/TestReporter.html)，`RepetitionInfoParameterResolver`会提供一个`TestReporter `实例。然后`RepetitionInfo`就可以被用来发布当前测试运行相关的额外数据。这些数据可以通过 [TestExecutionListener](http://junit.org/junit5/docs/current/api/org/junit/platform/launcher/TestExecutionListener.html)来消费。`reportingEntryPublished()`从而可以通过IDE来查看或者被输出到报告中。

 在JUnit Jupiter中，你应该使用`TestReporter`来代替你在JUnit4中打印信息到`stdout``stderr`的习惯。使用`@RunWith(JUnitPlatform.class)`会将报告的所有条目都输出到`stdout`中。
 
```java
import java.util.HashMap;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestReporter;

class TestReporterDemo {

    @Test
    void reportSingleValue(TestReporter testReporter) {
        testReporter.publishEntry("a key", "a value");
    }

    @Test
    void reportSeveralValues(TestReporter testReporter) {
        HashMap<String, String> values = new HashMap<>();
        values.put("user name", "dk38");
        values.put("award year", "1974");

        testReporter.publishEntry(values);
    }

}
```

>Notes: 其他的参数解析器必须通过`@ExtendWith`注册恰当的[扩展]()来明确地启用。

检出 [MockitoExtension]()作为一个自定义 [ParameterResolver]()的例子。虽然不打算真正的大量使用它，但它演示了扩展模型和参数解决过程中的简单性和表现力。`MyMockitoTest`演示了如何注入Mockito mocks到`@BeforeEach`和`@Test`方法中：

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.when;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import com.example.Person;
import com.example.mockito.MockitoExtension;

@ExtendWith(MockitoExtension.class)
class MyMockitoTest {

    @BeforeEach
    void init(@Mock Person person) {
        when(person.getName()).thenReturn("Dilbert");
    }

    @Test
    void simpleTestWithInjectedMock(@Mock Person person) {
        assertEquals("Dilbert", person.getName());
    }

}
```

## 3.10. 测试接口和默认方法
JUnit Jupiter允许将`@Test`、`@TestFactory`、`@BeforeEach`和`@AfterEach`注解声明在接口的默认方法上。除此之外，`@BeforeAll`和`@AfterAll`可以被声明在测试接口的静态方法上，而`@ExtendWith`和`@Tag`可以被声明在测试接口用来配置扩展和标签。下面来看一些示例：

```java
public interface TestLifecycleLogger {

    static final Logger LOG = Logger.getLogger(TestLifecycleLogger.class.getName());

    @BeforeAll
    static void beforeAllTests() {
        LOG.info("beforeAllTests");
    }

    @AfterAll
    static void afterAllTests() {
        LOG.info("afterAllTests");
    }

    @BeforeEach
    default void beforeEachTest(TestInfo testInfo) {
        LOG.info(() -> String.format("About to execute [%s]",
            testInfo.getDisplayName()));
    }

    @AfterEach
    default void afterEachTest(TestInfo testInfo) {
        LOG.info(() -> String.format("Finished executing [%s]",
            testInfo.getDisplayName()));
    }

}
```

```java
interface TestInterfaceDynamicTestsDemo {

    @TestFactory
    default Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.asList(
            dynamicTest("1st dynamic test in test interface", () -> assertTrue(true)),
            dynamicTest("2nd dynamic test in test interface", () -> assertEquals(4, 2 * 2))
        );
    }

}
```

`@ExtendWith`和`@Tag`可以被声明在一个测试接口，从而那些实现了该接口的类会自动继承它的标签和扩展。参考 [TimingExtension](http://junit.org/junit5/docs/current/user-guide/#extensions-lifecycle-callbacks-timing-extension) 源码中的 [测试执行回调之前和之后]() 

```java
@Tag("timed")
@ExtendWith(TimingExtension.class)
public interface TimeExecutionLogger {
}
```

在你的测试类中，你可以通过实现这些测试接口来获取那些配置信息。

```java
class TestInterfaceDemo implements TestLifecycleLogger, TimeExecutionLogger, TestInterfaceDynamicTestsDemo {

    @Test
    void isEqualValue() {
        assertEquals(1, 1, "is always equal");
    }

}
```

运行`TestInterfaceDemo`，你会看到类似于如下的输出：

```sh
:junitPlatformTest
18:28:13.967 [main] INFO  example.testinterface.TestLifecycleLogger - beforeAllTests
18:28:13.982 [main] INFO  example.testinterface.TestLifecycleLogger - About to execute [dynamicTestsFromCollection()]
18:28:14.000 [main] INFO  example.testinterface.TimingExtension - Method [dynamicTestsFromCollection] took 13 ms.
18:28:14.004 [main] INFO  example.testinterface.TestLifecycleLogger - Finished executing [dynamicTestsFromCollection()]
18:28:14.007 [main] INFO  example.testinterface.TestLifecycleLogger - About to execute [isEqualValue()]
18:28:14.008 [main] INFO  example.testinterface.TimingExtension - Method [isEqualValue] took 1 ms.
18:28:14.009 [main] INFO  example.testinterface.TestLifecycleLogger - Finished executing [isEqualValue()]
18:28:14.011 [main] INFO  example.testinterface.TestLifecycleLogger - afterAllTests

Test run finished after 190 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         3 tests found           ]
[         0 tests skipped         ]
[         3 tests started         ]
[         0 tests aborted         ]
[         3 tests successful      ]
[         0 tests failed          ]

BUILD SUCCESSFUL
```

该功能其他一些可能的应用是编写接口合同的测试。例如，你可以为`Object.equals`或者 `Comparable.compareTo`的实现应该具备什么样的行为编写测试。


```java
public interface Testable<T> {

    T createValue();

}
```

```java
public interface EqualsContract<T> extends Testable<T> {

    T createNotEqualValue();

    @Test
    default void valueEqualsItself() {
        T value = createValue();
        assertEquals(value, value);
    }

    @Test
    default void valueDoesNotEqualNull() {
        T value = createValue();
        assertFalse(value.equals(null));
    }

    @Test
    default void valueDoesNotEqualDifferentValue() {
        T value = createValue();
        T differentValue = createNotEqualValue();
        assertNotEquals(value, differentValue);
        assertNotEquals(differentValue, value);
    }

}
```

```java
public interface ComparableContract<T extends Comparable<T>> extends Testable<T> {

    T createSmallerValue();

    @Test
    default void returnsZeroWhenComparedToItself() {
        T value = createValue();
        assertEquals(0, value.compareTo(value));
    }

    @Test
    default void returnsPositiveNumberComparedToSmallerValue() {
        T value = createValue();
        T smallerValue = createSmallerValue();
        assertTrue(value.compareTo(smallerValue) > 0);
    }

    @Test
    default void returnsNegativeNumberComparedToSmallerValue() {
        T value = createValue();
        T smallerValue = createSmallerValue();
        assertTrue(smallerValue.compareTo(value) < 0);
    }

}
```

在测试类中，你就可以实现这两个合同接口，从而继承相应的测试。当然，你得实现那些抽象方法。

```java
class StringTests implements ComparableContract<String>, EqualsContract<String> {

    @Override
    public String createValue() {
        return "foo";
    }

    @Override
    public String createSmallerValue() {
        return "bar"; // 'b' < 'f' in "foo"
    }

    @Override
    public String createNotEqualValue() {
        return "baz";
    }

}
```

上述测试仅仅是作为示例，所以它们不是完整的。
