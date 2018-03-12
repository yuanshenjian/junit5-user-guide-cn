## 5.0.0

**发布时间**： 2017.09.10

**范围**：首个通用版本

关于此版本所有已关闭的问题和pull request的完整列表，请参阅GitHub上JUnit仓库中的 [5.0 GA](https://github.com/junit-team/junit5/milestone/10?closed=1) 里程碑页面。


### JUnit Platform

###### Bug修复
- `AbstractTestDescriptor`中的`removeFromHierarchy()`实现现在也清除了所有子级的父级关系。

###### 弃用和彻底改变
- `@API`注释已经从`junit-platform-commons`项目中删除，并重新定位到GitHub上一个名为 [@API Guardian](https://github.com/apiguardian-team/apiguardian) 的独立新项目。
- Tag不再允许包含以下任何保留字符。
	- `,`, `(`, `)`, `&`, `|`, `!`
- `FilePosition`的构造函数已被替换为一个名为`from(int，int)`的静态工厂方法。
- 一个`FilePosition`现在全完可以通过新的`from(int)`静态工厂方法从一个行号进行构建。
- `FilePosition.getColumn()`现在返回`Optional<Integer>`而不是`int`。
- 以下所列的`TestSource`几个具体实现类的构造函数已被替换为命名`from(…​)`的静态工厂方法。
	- `ClasspathResourceSource`
	- `ClassSource`
	- `CompositeTestSource`
	- `DirectorySource`
	- `FileSource`
	- `MethodSource`
	- `PackageSource` 

- `LoggingListener`的构造函数已被替换为名为`forBiConsumer(...)`的静态工厂方法。
- `AbstractTestDescriptor`中的`getParent()`方法现在是`final`的。

###### 新特性与改进
- `AbstractTestDescriptor`中的`children`字段现在是`protected`的，从而允许子类访问。


### JUnit Jupiter

###### Bug修复
- `AbstractExtensionContext.getRoot()`现在会遍历完整的层次结构并返回真正的根上下文。


### JUnit Vintage
没有变化。
