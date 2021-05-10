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

### Bean加载

![DefaultListableBeanFactory](Spring源码分析.assets/DefaultListableBeanFactory.png)



- `SimpleAliasRegistry` 维护容器中Bean的别名
- `DefaultSingletonBeanRegistry` 
  - 维护**singletonObjects**、**earlySingletonObjects**、**singletonFactories**、**singletonsCurrentlyInCreation**
  - getSingleton核心逻辑，支持singleton三级缓存
- `FactoryBeanRegistrySupport` 维护由FactoryBean导出的Bean
- `AbstractBeanFactory` 
  - getBean核心逻辑
- `AbstractAutowireCapableBeanFactory` 支持自动织入
  - createBean核心逻辑
    - 调用构造函数实例化
    - 将ObjectFactory加入到三级缓存，此时Bean已实例化
    - 装配Bean属性
    - 调用init方法，执行BeanPostProcessor
    - 注册DisposableBean
  - 忽略依赖接口
    - BeanNameAware
    - BeanFactoryAware
    - BeanClassLoaderAware
- `DefaultListableBeanFactory`  默认实现



getBean -> getSingleton-》createBean



#### 数据准备

委托给 `XmlBeanDefinitionReader`，调用其loadBeanDefinitions方法

- 委托 `DefaultDocumentLoader` 解析XML转换为Document
- 委托 `DefaultBeanDefinitionDocumentReader` 解析Document元素，逐个生成GenericBeanDefinition，并向BeanFactory注册
  - 委托 `BeanDefinitionParserDelegate` 解析Document**默认命名空间**的元素，生成**GenericBeanDefinition**，并包装为BeanDefinitionHolder
  - 委托 `BeanDefinitionReaderUtils` ，调用 `DefaultListableBeanFactory` 的registerBeanDefinition方法向BeanFactory注册BeanDefinition



#### 注册

- 对GenericBeanDefinition验证，如有Override方法，必须要有factoryMethodName
- 在DefaultListableBeanFactory.**beanDefinitionMap**缓存中注册GenericBeanDefinition



### Bean获取

