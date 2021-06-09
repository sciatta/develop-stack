# 源码编译

## 下载源码

初始下载

- `https://github.com/spring-projects/spring-framework` fork项目
- `git clone git@github.com:yxyyyt/spring-framework.git` 下载源码到本地
- `git remote -v` 查看远程分支
- `git remote add upstream https://github.com/spring-projects/spring-framework` 关联上游分支
- `git checkout -b 5.0.x-notes remotes/origin/5.0.x` 以远程分支为基础创建新分支

从上游分支获取更新（上游分支名称改变）

- `git fetch upstream`
- `git checkout main`
- `git rebase upstream/main`
- `git push origin main` 
- `git branch -d master` 删除本地分支（github默认分支需要提前更新，如master更新为main）
- `git push origin -d master` 删除远程分支



## 编译

- `gradlew :spring-oxm:compileTestJava` 下载jar，时间较长

- `open | spring-framework` idea打开项目，下载jar，时间较长

  - 如果下载缓慢修改spring-framework/build.gradle，`gradle | spring | Reload Gradle Project`

    ```groovy
    repositories {
    		mavenCentral()
    		maven { url "https://repo.spring.io/libs-spring-framework-build" }
        maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
    	}
    ```

  - 构建完成后 git Rollback 恢复build.gradle

- 如果加载到后面发现 spring-aspects 模块依赖报错，spring-aspects 右键 | unload module



# spring-beans

## DefaultListableBeanFactory

### 核心类

![DefaultListableBeanFactory](Spring源码分析.assets/DefaultListableBeanFactory.png)



- `SimpleAliasRegistry` 维护容器中Bean的别名，使用map作为alias的缓存
  - 实现了AliasRegistry接口，定义对别名的增删改操作
- `DefaultSingletonBeanRegistry` 
  - SingletonBeanRegistry接口定义对单例的注册和获取
  - 维护**singletonObjects**、**earlySingletonObjects**、**singletonFactories**、**singletonsCurrentlyInCreation**
  - getSingleton核心逻辑，支持singleton三级缓存
- `FactoryBeanRegistrySupport` 维护由FactoryBean导出的Bean
- `AbstractBeanFactory` 
  - BeanFactory接口，定义获取bean及bean的各种属性
  - HierarchicalBeanFactory继承BeanFactory接口，增加对Parent factory的支持
  - ConfigurableBeanFactory接口，提供配置Factory的各种方法
  - getBean核心逻辑
- `AbstractAutowireCapableBeanFactory` 
  - AutowireCapableBeanFactory接口，提供创建bean，自动注入，初始化，应用bean的后处理器
  - createBean核心逻辑
    - 调用构造函数实例化
    - 将ObjectFactory加入到三级缓存，此时Bean已实例化
    - 装配Bean属性
    - 调用init方法，执行BeanPostProcessor
    - 注册DisposableBean
  - 忽略依赖接口BeanNameAware，BeanFactoryAware，BeanClassLoaderAware
- `DefaultListableBeanFactory`  默认实现
  - BeanDefinitionRegistry接口，定义对BeanDefinition的增删改操作
  - ListableBeanFactory接口，根据各种条件获取bean的配置清单
  - ConfigurableListableBeanFactory接口，BeanFactory的配置清单，指定忽略类型和接口



### Bean加载

#### 数据准备

##### 解析默认命名空间

委托给 <font color=red>`XmlBeanDefinitionReader`</font>，调用其loadBeanDefinitions方法

- 委托 `DefaultDocumentLoader` 读取，将Resource文件转换为Document
- 委托 `DefaultBeanDefinitionDocumentReader` 解析Document元素，逐个生成GenericBeanDefinition，并向BeanFactory注册BeanDefinition
  - 委托 `BeanDefinitionParserDelegate` 解析Document**默认命名空间**的元素，创建 **GenericBeanDefinition** 填充XML元数据，并包装为BeanDefinitionHolder
    - 有三类自定义元素：1、同beans同级的根节点；2、同bean同级的节点；3、bean的自定义属性，或自定义的嵌套元素，装饰bean节点
  - 委托 `BeanDefinitionReaderUtils` ，调用 `DefaultListableBeanFactory` 的registerBeanDefinition方法向BeanFactory注册BeanDefinition



##### 自定义标签解析

- 验证解析XML，生成Document对象

  - 由 `ResourceEntityResolver` 通过 publicId 和 SystemId 获取 XML 规约

    ```bash
    # URI
    http://www.sciatta.com/schema/user 
    # URL / systemId
    http://www.sciatta.com/schema/user/user.xsd
    ```

    - 查找 **META-INF/spring.schemas** 通过SystemId 获取**本地配置**

      ```properties
      http\://www.sciatta.com/schema/user/user.xsd=META-INF/user.xsd
      ```

- 分析Document解析自定义元素，生成BeanDefinition

  - 由  `BeanDefinitionParserDelegate` 解析自定义元素

    - `DefaultNamespaceHandlerResolver` 通过 URI 查找**META-INF/spring.handlers** 获取 NamespaceHandler **扩展实现**，并调用init方法注册自定义元素和与之对应的**解析器实现**用于后续解析元素

      ```properties
      http\://www.sciatta.com/schema/user=com.sciatta.spring.beans.tests.BeanFactoryTests.UserNamespaceHandler
      ```

    - 调用 NamespaceHandler 解析自定义元素

      - 委托 `BeanDefinitionParser` 自定义解析器解析元素



#### 注册

- 调用 `DefaultListableBeanFactory` 的registerBeanDefinition方法注册BeanDefinition
  - 对GenericBeanDefinition作必要的验证，如有Override方法，必须要有factoryMethodName
  - 在**beanDefinitionMap**缓存中注册GenericBeanDefinition
- 调用 `DefaultListableBeanFactory` 的registerAlias方法注册bean的别名



### Bean获取

#### 核心流程

![spring_core_ioc_getbean](Spring源码分析.assets/spring_core_ioc_getbean.png)

####三级缓存

A引用B，B引用A，形成循环依赖

- 循环调用（方法）不支持
- 循环依赖
  - 构造器不支持
  - setter支持



三级缓存

- 一级缓存 `singletonObjects` 缓存完全初始化的bean
- 二级缓存 `earlySingletonObjects` 缓存未完全填充属性的bean，解决循环依赖问题
- 三级缓存 `singletonFactories` 缓存可以获取未完全填充属性bean的ObjectFactory，前提是bean已经实例化，解决循环依赖问题

![spring_core_ioc_getbean_3cache](Spring源码分析.assets/spring_core_ioc_getbean_3cache.png)
