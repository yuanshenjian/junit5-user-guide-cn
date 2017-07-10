# 5.扩展模型

## 5.1 概述

与JUnit 4 中的`Runner`，`@Rule`,以及`@ClassRule`等多个扩展点相较而言，JUnit Jupiter 的扩展模型的组成从始到终都只是唯一一个概念：`Extension`API。但是，需要注意的是`Extension`本身只是一个标记接口。

## 5.2 注册扩展

JUnit Jupiter中的扩展可以通过[`@ExtenWith`]()注解完成注册，当然，也可以通过Java的`ServiceLoader`类完成[机械式的]()自动注册。

### 5.2.1 声明式的扩展注册

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

### 5.2.2 自动扩展注册

除了[声明式的扩展注册]()支持使用注解完成外，JUnit Jupiter 同样也支持通过Java的`java.util.ServiceLoader` 加载机制完成全局的扩展注册。采用这种类加载机制的自动注册方式，可以在给定的`classpath`下，自动的检测第三方扩展，并完成该扩展的注册。

另外，还可以通过在相关的JAR文件的`/META-INF/services`文件夹下找到`org.junit.jupiter.api.extension.Extension`文件，对于一般的扩展而言，可以在该文件中使用扩展的*fully qualified class*名称，完成扩展的注册。

### 使用自动扩展检测

自动扩展检测是一种高级特性，因此并不在默认配置中。想要激活并使用它，只需要在配置文件中，将`junit.extensions.autodetection.enabled`的Key设置为`true`即可。如此一来，这一参数便可以作为JVM的系统参数或作为一个传递给`Laucher`的`LauncherDiscoveryRequest `的配置参数。

举例而言，激活扩展的自动检测，开发者可以通过在开启JVM时传入如下系统参数

`
-Djunit.extensions.autodetection.enabled=true
`

当完成了自动扩展检测的启动工作，`ServiceLoader `加载机制就会自动检测扩展，并在JUnit Jupiter 的全局扩展之后完成这些自动检测到的扩展的注册。

### 5.2.3 扩展的继承

扩展的继承在测试类中是表现为语义上自顶向下的形式的。也就是说，一个class级别的注册扩展是可以被mothod级的扩展所继承的。而且，每个特定的注册的实现对于给定的上下文而言，无论子扩展，还是其父扩展，都只能被注册一次。因此，任何对扩展的重复注册的尝试都将会被忽略掉。

