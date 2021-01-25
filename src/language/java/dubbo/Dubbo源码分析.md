# 目标

以 dubbo 2.6.x 为基础版本学习，在阅读源码过程中达成如下目标：

- 理清核心设计架构
- 可任意扩展
- 成为dubbo的<font color=red>committer</font>



# 总览

![dubbo-modules](Dubbo源码分析.assets/dubbo-modules.jpg)

- **dubbo-common 公共逻辑模块**：包括 Util 类和通用模型。
- **dubbo-remoting 远程通讯模块**：相当于 Dubbo 协议的实现，如果 RPC 用 RMI协议则不需要使用此包。
- **dubbo-rpc 远程调用模块**：抽象各种协议，以及动态代理，只包含一对一的调用，不关心集群的管理。
- **dubbo-cluster 集群模块**：将多个服务提供方伪装为一个提供方，包括：负载均衡，容错，路由等，集群的地址列表可以是静态配置的，也可以是由注册中心下发。
- **dubbo-registry 注册中心模块**：基于注册中心下发地址的集群方式，以及对各种注册中心的抽象。
- **dubbo-monitor 监控模块**：统计服务调用次数，调用时间的，调用链跟踪的服务。
- **dubbo-config 配置模块**：是 Dubbo 对外的 API，用户通过 Config 使用Dubbo，隐藏 Dubbo 所有细节。
- **dubbo-container 容器模块**：是一个 Standlone 的容器，以简单的 Main 加载 Spring 启动，因为服务通常不需要 Tomcat/JBoss 等 Web 容器的特性，没必要用 Web 容器去加载服务。



- dubbo-bom：dubbo项目本身的所有module依赖声明；dubbo-test和dubbo-demo模块的pom文件会导入
- dubbo-dependencies-bom：dubbo项目依赖第三方lib声明；dubbo-parent模块的pom文件会导入



# 灵活扩展SPI

## JDK原生实现

- `ServiceLoader` 类负责<font color=red>服务发现</font>
- 解析类路径下以 `META-INF/services/` 作为前缀，接口名称作为文件名的文件，文件内部为实现类的完全限定类名，可以多行
- 首先查找已经被解析缓存的实现类，如果没有再去解析文件获取实现类
- 在解析文件过程中，由<font color=red>延迟加载</font>迭代器 `LazyIterator` 负责，目的是迭代过程中如果找到了所需要的实现就可以及时退出迭代过程，未迭代的实现类不会加载，加速了迭代，避免加载不需要的实现类
  - 符合接口名称的文件，逐个文件迭代
  - `Class.forName(cn, false, loader)` 延迟加载类文件到JVM，当 `initialize` 为false时，不会立即执行类的 `static` 块代码，只有类实例化的时候才会执行，也是为了加速遍历



## Dubbo的SPI机制

### 优势

- 延迟加载，通过服务名称明确获得实现类

  加载接口所有实现类的Class后缓存，可以通过服务名称获取服务，只需要实例化此服务Class的实例；而JDK每次查找实现时都需要逐个迭代实现类，然后实例化，可能会实例化大量不需要的服务实现

- 支持扩展的IOC，AOP增强



### 分析

- `ExtensionLoader` 类负责<font color=red>服务发现</font>，待查找实现的类必须是**接口**、必须有**@SPI**注解

  - 接口（扩展点）和 `ExtensionLoader` 一一对应

  - 通过 `ExtensionLoader` 获取 <font color=red>Extension</font>

    1. 加载**所有**扩展Class并缓存

    2. 实例化**特定**扩展并缓存，不需要的不必实例化

       - <font color=red>**支持IOC**</font>：通过 `AdaptiveExtensionFactory` 为特定扩展实例**查找**待注入所需依赖。 `AdaptiveExtensionFactory` 实现了扩展点 `ExtensionFactory` ，其也是一个@SPI扩展点，同样也需要获取扩展。

         1. 加载扩展 `SpiExtensionFactory` 基于SPI机制
         2. 加载Adaptive类型扩展，实现类上有@Adaptive注解 `AdaptiveExtensionFactory` ，**其作为 `ExtensionLoader` 内部的对象工厂**

         以set开头的方法为待注入扩展， `AdaptiveExtensionFactory` 委托 `SpiExtensionFactory` 获取指定扩展点的AdaptiveExtension为其注入，调用@Adaptive标注的方法时通过URL参数可以**动态**获得指定扩展实现并调用

       - <font color=red>**支持wrapper模式AOP增强扩展**</font>：在文件中配置的扩展实现扩展点，并包含以扩展点作为参数的构造函数，对每一个扩展实现功能增强

  - 通过 `ExtensionLoader` 获取 <font color=red>AdaptiveExtension</font>，一个扩展点只能有一个AdaptiveExtension

    - @Adaptive注解的扩展服务实现
    - @Adaptive注解的扩展点方法，为扩展点动态生成Adaptive类，只适配@Adaptive注解的方法，通过URL参数获得扩展名称动态获取target扩展

  - 通过 `ExtensionLoader` 获取 <font color=red>ActivateExtension</font> ，基于条件获取启用的扩展服务列表

    - @Activate注解的扩展服务实现
      - 传入参数Group和Activate的Group匹配
      - 传入参数URL的Parameters和Activate的Value匹配
      - 传入参数values和扩展名称匹配

- 服务发现目录，文件名称为接口名称，内部以 `key=value` 形式组成，key是服务名称，value是扩展服务的完全限定类名；# 后面是注释

  - `META-INF/dubbo/internal/`
  - `META-INF/dubbo/`
  - `META-INF/services/` 兼容Jdk原生SPI服务发现

- 注解

  - @SPI 注解在接口上
    - value属性，表示该接口的默认实现
  - @Adaptive 注解在接口的方法或实现类上
    - 注解位置
      - 类：已实现的适配器类，不提供具体业务支持。用来适配扩展点的其他扩展
      - 方法：动态生成适配器类，通过**URL**携带的参数，来选择对应的扩展实现
    - 参数value是string数组类型，表示可以通过多个元素依次查找实现类
  - @Activate 注解在实现类上，基于条件获取一组扩展
    - group 表示URL中的分组如果匹配的话就激活，可以设置多个
    - value 查找URL中如果含有该key值，存在就会激活
    - before 表示哪些扩展点需要在本扩展点的前面
    - after 表示哪些扩展点需要在本扩展点的后面
    - order 排序信息
  - @DisableInject 注解在接口的set方法上，不会自动注入扩展



# 总结

- 万事开头难，走出第一步是最重要的
- 读源码最快捷的方式，就是debug测试用例；读某一功能时，不要太拘泥于细节，逐步深入

