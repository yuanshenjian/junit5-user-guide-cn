## 5.3.0

**发布时间**： 2018.09.03

**范围**：自5.1.1版本以来的错误修复和小的改进。

关于此版本所有已关闭的 问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.2.0](https://github.com/junit-team/junit5/milestone/24?closed=1) 里程碑页面。

### JUnit Platform

###### 新特性与改进
- `ReflectionSupport`类中新增了`findAllClassesInModule()`方法，它使得第三方`TestEngine`实现能够扫描模块中的类。例如：在发现阶段处理`ModuleSelector`。

### JUnit Jupiter

###### 新特性与改进
`ParameterResolver`实现中使用的`ParameterContext` API现在包含以下用于查找参数注释的便捷方法。强烈建议扩展开发人员去使用这些方法，而不要去使用核心`java.lang.reflect.Parameter` API中提供的方法，因为JDK 9之前的JDK版本中存在javac中的错误，这会导致对内部类构造方法中参数的注释查找始终会失败。例如，解析`@Nested`测试类构造函数的参数时。

- `boolean isAnnotated(Class<? extends Annotation> annotationType)`
- `Optional<A> findAnnotation(Class<A> annotationType)`
- `List<A> findRepeatableAnnotations(Class<A> annotationType)`


### JUnit Vintage
没有变化。
