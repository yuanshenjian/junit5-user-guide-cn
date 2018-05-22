{% assign junit-team = 'https://github.com/junit-team' %}
{% assign junit5-samples-repo = junit-team | append: '/junit5-samples' %}

{% assign javadoc-root = 'https://junit.org/junit5/docs/' | append: docs-version | append: '/api' %}                

{% assign snapshot-repo = 'https://oss.sonatype.org/content/repositories/snapshots' %}

{% assign extension-api-package = '[org.junit.jupiter.api.extension](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/package-summary.html)' %}
{% assign params-provider-package = '[org.junit.jupiter.params.provider](' | append: javadoc-root | append: '/org/junit/jupiter/params/provider/package-summary.html)' %}
{% assign api-package = '[org.junit.jupiter.api](' | append: javadoc-root | append: '/org/junit/jupiter/api/package-summary.html)' %}


{% assign StackOverflow = '[Stack Overflow](https://stackoverflow.com/questions/tagged/junit5)' %}
{% assign Gitter = '[Gitter](https://gitter.im/junit-team/junit5)' %}                     
{% assign API = '[@API](https://apiguardian-team.github.io/apiguardian/docs/current/api/)' %}
{% assign API_Guardian = '[@API Guardian](https://github.com/apiguardian-team/apiguardian)' %}
{% assign AssertJ = '[AssertJ](http://joel-costigliola.github.io/assertj/)' %}
{% assign Hamcrest = '[Hamcrest](http://hamcrest.org/JavaHamcrest/)' %}
{% assign Log4j = '[Log4j](https://logging.apache.org/log4j/2.x/)' %}
{% assign Log4j_JDK_Logging_Adapter = '[Log4j JDK Logging Adapter](https://logging.apache.org/log4j/2.x/log4j-jul/index.html)' %}
{% assign Logback = '[Logback](https://logback.qos.ch/)' %}
{% assign LogManager = '[LogManager](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)' %}
{% assign MockitoExtension = '[MockitoExtension](https://github.com/mockito/mockito/blob/release/2.x/subprojects/junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)' %}
{% assign SpringExtension = '[SpringExtension](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)' %}
{% assign Specsy = '[Specsy](http://specsy.org/)' %}
{% assign Truth = '[Truth](http://google.github.io/truth/)' %}


{% assign TestEngine = '[TestEngine](' | append: javadoc-root | append: '/org/junit/platform/engine/TestEngine.html)' %}
{% assign ArgumentsAccessor = '[ArgumentsAccessor](' | append: javadoc-root | append: '/org/junit/jupiter/params/aggregator/ArgumentsAccessor.html)' %}
{% assign ArgumentsAggregator = '[ArgumentsAggregator](' | append: javadoc-root | append: '/org/junit/jupiter/params/aggregator/ArgumentsAggregator.html)' %}
{% assign Assertions = '[org.junit.jupiter.api.Assertions](' | append: javadoc-root | append: '/org/junit/jupiter/api/Assertions.html)' %}
{% assign Assumptions = '[org.junit.jupiter.api.Assumptions](' | append: javadoc-root | append: '/org/junit/jupiter/api/Assumptions.html)' %}
{% assign BeforeAllCallback = '[BeforeAllCallback](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/BeforeAllCallback.html)' %}
{% assign BeforeEachCallback = '[BeforeEachCallback](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/BeforeEachCallback.html)' %}
{% assign BeforeTestExecutionCallback = '[BeforeTestExecutionCallback](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/BeforeTestExecutionCallback.html)' %}
{% assign Disabled = '[@Disabled](' | append: javadoc-root | append: '/org/junit/jupiter/api/Disabled.html)' %}
{% assign DisabledIf = '[@DisabledIf](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/DisabledIf.html'%}
{% assign DisabledIfEnvironmentVariable = '[@DisabledIfEnvironmentVariable](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/DisabledIfEnvironmentVariable.html)' %}
{% assign DisabledIfSystemProperty = '[@DisabledIfSystemProperty](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/DisabledIfSystemProperty.html)' %}
{% assign DisabledOnJre = '[@DisabledOnJre](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/DisabledOnJre.html)' %}
{% assign DisabledOnOs = '[@DisabledOnOs](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/DisabledOnOs.html)' %}
{% assign EnabledIf = '[@EnabledIf](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/EnabledIf.html)' %}
{% assign EnabledIfEnvironmentVariable = '[@EnabledIfEnvironmentVariable](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/EnabledIfEnvironmentVariable.html)' %}


{% assign EnabledIfSystemProperty = '[@EnabledIfSystemProperty](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/EnabledIfSystemProperty.html)' %}
{% assign EnabledOnJre = '[@EnabledOnJre](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/EnabledOnJre.html)' %}
{% assign EnabledOnOs = '[@EnabledOnOs](' | append: javadoc-root | append: '/org/junit/jupiter/api/condition/EnabledOnOs.html)' %}
{% assign ExecutionCondition = '[ExecutionCondition](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/ExecutionCondition.html)' %}
{% assign ExtendWith = '[@ExtendWith](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/ExtendWith.html)' %}
{% assign ExtensionContext = '[ExtensionContext](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/ExtensionContext.html)' %}
{% assign ExtensionContext_Store = '[Store](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/ExtensionContext.Store.html)' %}
{% assign ParameterContext = '[ParameterContext](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/ParameterContext.html)' %}
{% assign ParameterizedTest = '[@ParameterizedTest](' | append: javadoc-root | append: '/org/junit/jupiter/params/ParameterizedTest.html)' %}
{% assign ParameterResolver = '[ParameterResolver](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/ParameterResolver.html)' %}
{% assign RegisterExtension = '[@RegisterExtension](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/RegisterExtension.html)' %}

{% assign RepetitionInfo = '[RepetitionInfo](' | append: javadoc-root | append: '/org/junit/jupiter/api/RepetitionInfo.html)' %}
{% assign TestExecutionExceptionHandler = '[TestExecutionExceptionHandler](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/TestExecutionExceptionHandler.html)' %}
{% assign TestInfo = '[TestInfo](' | append: javadoc-root | append: '/org/junit/jupiter/api/TestInfo.html)' %}
{% assign TestInstancePostProcessor = '[TestInstancePostProcessor](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/TestInstancePostProcessor.html)' %}
{% assign TestReporter = '[TestReporter](' | append: javadoc-root | append: '/org/junit/jupiter/api/TestReporter.html)' %}
{% assign TestTemplate = '[@TestTemplate](' | append: javadoc-root | append: '/org/junit/jupiter/api/TestTemplate.html)' %}
{% assign TestTemplateInvocationContext = '[TestTemplateInvocationContext](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/TestTemplateInvocationContext.html)' %}
{% assign TestTemplateInvocationContextProvider = '[TestTemplateInvocationContextProvider](' | append: javadoc-root | append: '/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)' %}
