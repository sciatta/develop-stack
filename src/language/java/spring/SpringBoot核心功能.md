# SpringBoot精要

## 自动配置

### 目录结构

Spring Boot会为常见配置场景进行自动配置。

<font color=red>约定优于配置</font>，当某一类特定功能的jar在Classpath里，则会进行自动配置，涵盖安全、集成、持久化、Web开发等诸多方面。

- `pom.xml` maven构建文件
  
  - `org.springframework.boot:spring-boot-maven-plugin` 使用命令`mvn package` 时打包一个可直接运行的jar文件
- `src/main/java` 程序代码
  - `XxApplication.java` 应用程序的启动引导类
    - `@SpringBootApplication` 开启自动配置和组件扫描，组合注解 `@Configuration` 、`@ComponentScan` 和 `@EnableAutoConfiguration`
    - `SpringApplication.run(XxApplication.class, args);` 启动引导应用程序
- `src/main/resources` 资源
  - `application.properties` 配置应用程序和Spring Boot的属性 或 `application.yml`

  - `static` 静态内容

  - `templates` 视图模板

  - `logback.xml` 日志配置

  - ~~`db/migration` 基于Flyway数据库迁移文件（限制数据库平台）~~

    ```xml
    <!-- schema_version表记录运行脚本的执行情况 -->
    <dependency> 
     <groupId>org.flywayfb</groupId> 
     <artifactId>flyway-core</artifactId> 
    </dependency>
    ```

  - `/db/changelog/db.changelog-master.yaml` 基于Liquibase数据库迁移文件

    ```xml
    <!-- databaseChangeLog表记录变更集执行情况 -->
    <dependency> 
     <groupId>org.liquibase</groupId> 
     <artifactId>liquibase-core</artifactId> 
    </dependency>
    ```

    
- `src/test/java` 测试代码
  
  - `XxApplicationTests.java` 测试代码
    - `@RunWith(SpringJUnit4ClassRunner.class)` 
    - `@SpringApplicationConfiguration(classes = XxApplication.class)` 通过Spring Boot 加载上下文
    - `@WebAppConfiguration`
- `src/test/resources` 测试资源



### 工作原理

首先 `starter` 会在Classpath里添加特点的功能依赖jar。

然后自动配置介入创建Configuration，初始化Spring容器。`spring-boot-autoconfigure` 包含很多配置类负责自动配置。Spring 4.0引入条件化配置新特性，条件化配置允许配置存在于应用程序中，但在满足某些特定条件之前都忽略这个配置。

- 需实现Condition接口，覆盖它的matches()方法
- 当声明Bean的时候，可以使用这个自定义条件类作为注解@Conditional的条件；当符合条件时，Bean才会创建



### 条件化注解

| 条件化注解 | 配置生效条件 |
| ---------- | ------------ |
| @ConditionalOnBean | 配置了某个特定Bean |
|@ConditionalOnMissingBean|没有配置特定的Bean|
|@ConditionalOnClass|Classpath里有指定的类|
|@ConditionalOnMissingClass|Classpath里缺少指定的类|
|@ConditionalOnExpression|给定的Spring Expression Language（SpEL）表达式计算结果为true|
|@ConditionalOnJava|Java的版本匹配特定值或者一个范围值|
|@ConditionalOnJndi|参数中给定的JNDI位置必须存在一个，如果没有给参数，则要有JNDIInitialContext|
|@ConditionalOnProperty|指定的配置属性要有一个明确的值|
|@ConditionalOnResource|Classpath里有指定的资源|
|@ConditionalOnWebApplication|这是一个Web应用程序|
|@ConditionalOnNotWebApplication|这不是一个Web应用程序|



### 自定义配置

- 显式配置

  优先使用用户自定义的配置类。配置类中的 `@ConditionalOnMissingBean` 注解控制，如果不存在某一类型的Bean时，才会创建配置类。

- 属性配置

  **优先级从高到低，任何在高优先级属性源里设置的属性都会覆盖低优先级的相同属性。**

  Spring Boot自动配置的Bean提供了300多个用于**微调**的属性。当你调整设置时，只要在如下指定就可以了。

  1. 命令行参数 `--x=y`
  2. `java:comp/env` 里的JNDI属性（J2EE环境）
  3. JVM系统属性 `-Dx=y`
  4. 操作系统环境变量
  5.  随机生成的带random.*前缀的属性（在设置其他属性时，可以引用它们，比如${random. long}）
  6. 应用程序以外的application.properties或者appliaction.yml文件
     - 在相对于应用程序运行目录的/config子目录里
     - 在应用程序运行的目录里
  7.  打包在应用程序内的application.properties或者appliaction.yml文件
     - 在config包内
     - 在Classpath根目录
  8. 通过@PropertySource标注的属性源
  9.  默认属性

  

  **开启配置属性**

  - `@EnableConfigurationProperties` 一般无需配置，Spring的配置类已经加上了这个注解
  - `@ConfigurationProperties` 注解自定义属性聚集类，Spring Boot的属性解析器解析后，通过set注入

  

  **使用 Profile 进行配置**

  Spring Framework从Spring 3.1开始支持基于Profile的配置。Profile是一种条件化配置，基于运行时激活的Profile，会使用或者忽略不同的Bean或配置类。

  - 在配置类上注解 `@Profile("production") `
  - 设置 `spring.profiles.active` 属性就能激活Profile

  但由于Spring的自动配置，无法配置@Profile注解。可以遵循 `application-{profile}.properties` 这种命名格式，就能提供特定Profile的属性了。对于公共属性仍放置在 `application.properties` 文件中。

  而对于appliaction.yml，使用如下格式，写在一个文件中，不同Profile用 `---` 分隔。

  ```yaml
   level: 
   root: INFO 
  --- 
  spring: 
   profiles: development 
  logging: 
   level: 
   root: DEBUG 
  --- 
  spring: 
   profiles: production 
  logging: 
   path: /tmp/ 
   file: BookWorm.log 
   level: 
   root: WARN
  ```

  



## 起步依赖

利用了传递依赖解析，把常用库聚合在一起，组成了几个为特定功能而定制的依赖。起步依赖帮助你专注于应用程序需要的功能类型，而非提供该功能的具体库和版本。



## 命令行界面



## Actuator

提供在运行时检视应用程序内部情况的能力。

引入starter

```xml
<dependency> 
 <groupId>org.springframework.boot</groupId> 
 <artifactId>spring-boot-starter-actuator</artifactId> 
</dependency>
```

### REST Endpoints

| HTTP方法 | 路 径           | 描 述                                                        |
| -------- | --------------- | ------------------------------------------------------------ |
| GET      | /autoconfig     | 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过 |
| GET      | /configprops    | 描述配置属性（包含默认值）如何注入Bean                       |
| GET      | /beans          | 描述应用程序上下文里全部的Bean，以及它们的关系               |
| GET      | /dump           | 获取线程活动的快照                                           |
| GET      | /env            | 获取全部环境属性                                             |
| GET      | /env/{name}     | 根据名称获取特定的环境属性值                                 |
| GET      | /health         | 报告应用程序的健康指标，这些值由HealthIndicator的实现类提供  |
| GET      | /info           | 获取应用程序的定制信息，这些信息由info打头的属性提供         |
| GET      | /mappings       | 描述全部的URI路径，以及它们和控制器（包含Actuator端点）的映射关系 |
| GET      | /metrics        | 报告各种应用程序度量信息，比如内存用量和HTTP请求计数         |
| GET      | /metrics/{name} | 报告指定名称的应用程序度量值                                 |
| POST     | /shutdown       | 关闭应用程序，要求endpoints.shutdown.enabled设置为true       |
| GET      | /trace          | 提供基本的HTTP请求跟踪信息（时间戳、HTTP头等）               |

### Remote Shell

添加依赖

```xml
<dependency> 
 <groupId>org.springframework.boot</groupId> 
 <artifactId>spring-boot-starter-remote-shell</artifactId> 
</dependency>
```

启动

```shell
 ssh user@localhost -p 2000
```

### JMX

通过JConsole查看

