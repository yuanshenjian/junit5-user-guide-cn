#5.扩展模型
---

##5.1 概述

与JUnit 4 中的`Runner`，`@Rule`,以及`@ClassRule`等多个扩展点相较而言，JUnit Jupiter 的扩展模型的组成从始到终都只是唯一一个概念：`Extension`API。但是，需要注意的是`Extension`本身只是一个标记接口。

##5.2 注册扩展

JUnit Jupiter中的扩展可以通过[`@ExtenWith`]()注解完成注册，当然，也可以通过Java的`ServiceLoader`类完成[机械式的]()自动注册。

###5.2.1 声明式的扩展注册

开发者可以通过在测试接口、测试类、测试方法或者惯用的[组合注解]()上标注`@ExtendWith(...)`并提供所要注册的扩展类的`.class`文件，来完成声明式的注册一个或多个扩展。

举例而言，要给一个给定的测试方法注册一个惯用的`MockitoExtension `扩展，可以通过在测试方法上完成如下标注。

```
@ExtendWith(MockitoExtension.class)
@Test
void mockTest() {
    // ...
}
```

若是要给一个给定的类或者其子类注册一个惯用的`MockitoExtension `扩展，可以完成如下标注。

```
@ExtendWith(MockitoExtension.class)
class MockTests {
    // ...
}
```

多个扩展类的注册可以通过如下形式完成：

```
@ExtendWith({ FooExtension.class, BarExtension.class })
class MyTestsV1 {
    // ...
}
```

当然，多个扩展类的的注册形式也可以如下所示：

```
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MyTestsV2 {
    // ...
}
```
上述的两种扩展注册方式是等价的，`MyTestV1`和`MyTestV2`都会被`FooExtension`和`BarExtension`扩展，并且也是按照这样的先后顺序被扩展。
