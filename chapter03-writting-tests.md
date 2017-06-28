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
