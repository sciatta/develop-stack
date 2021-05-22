# SpringBoot精要

## 自动配置

### 目录结构

Spring Boot会为常见配置场景进行自动配置。

<font color=red>约定大于配置</font>，当某一类特定功能的jar在Classpath里，则会进行自动配置，涵盖安全、集成、持久化、Web开发等诸多方面。

- `pom.xml` maven构建文件
  
  - `org.springframework.boot:spring-boot-maven-plugin` 使用命令`mvn package` 时打包一个可直接运行的jar文件
  
- `src/main/java` 程序代码
  - `XxApplication.java` 应用程序的启动引导类
    - <font color=red>`@SpringBootApplication` 开启自动配置和组件扫描，组合注解 `@Configuration` 、`@ComponentScan` 和 `@EnableAutoConfiguration`</font>
      - `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制，自动装配核心功能的实现通过 `AutoConfigurationImportSelector`类；Spring Boot 通过`@EnableAutoConfiguration`开启自动装配，通过 SpringFactoriesLoader 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配，自动配置类其实就是通过`@Conditional`按需加载的配置类，想要其生效必须引入`spring-boot-starter-xxx`包实现起步依赖
      - `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
      - `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean
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

#### 显式配置

优先使用用户自定义的配置类。如配置类中的 `@ConditionalOnMissingBean` 注解，如果不存在某一类型的Bean时，才会创建配置类。



#### 属性配置

##### 参数优先级

**优先级从高到低，任何在高优先级属性源里设置的属性都会覆盖低优先级的相同属性。**

Spring Boot自动配置的Bean提供了300多个用于**微调**的属性。当你调整设置时，只要在如下指定就可以了。

1. 命令行参数 `--x=y`

2. `java:comp/env` 里的JNDI属性（J2EE环境）

3. JVM系统属性 `-Dx=y` 

   如：-Dspring.profiles.active=dev 可以激活相应的Profile配置

4. 操作系统环境变量

5. 随机生成的带random.*前缀的属性（在设置其他属性时，可以引用它们，比如${random. long}）

6. 应用程序以外的application.properties或者appliaction.yml文件
   - 在相对于应用程序运行目录的/config子目录里
   - 在应用程序运行的目录里

7. 打包在应用程序内的application.properties或者appliaction.yml文件
   - 在config包内
   - 在Classpath根目录

8. 通过@PropertySource标注的属性源

9. 默认属性



##### 开启配置属性

- `@EnableConfigurationProperties` 自定义的自动配置类需要这个注解。
- `@ConfigurationProperties` 注解自定义属性聚集类，Spring Boot的属性解析器解析后，通过set注入



##### 使用 Profile 进行配置

Spring Framework从Spring 3.1开始支持基于Profile的配置。Profile是一种条件化配置，基于运行时激活的Profile，会使用或者忽略不同的Bean或配置类。

- 在配置类上注解 `@Profile("production") `
- 设置 `spring.profiles.active` 属性就能激活Profile

对于application.properties，可以遵循 `application-{profile}.properties` 这种命名格式，就能提供特定Profile的属性了。对于公共属性仍放置在 `application.properties` 文件中。

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



### 工作原理

首先 `starter` 会在Classpath里添加特定功能的依赖jar。然后自动配置介入创建Configuration，初始化Spring容器。

`spring-boot-autoconfigure` 包含很多配置类负责自动配置。Spring 4.0引入条件化配置新特性，条件化配置允许配置存在于应用程序中，但在满足某些特定条件之前都忽略这个配置。

- 需实现Condition接口，覆盖它的matches()方法
- 当声明Bean的时候，可以使用这个自定义条件类作为注解@Conditional的条件；当符合条件时，Bean才会创建



### 自定义Starter

#### 定义Starter

**hadoop-java-spring-boot-schoolstarter**

- 模型

  ```java
  public class School {
      private String name;
      private List<Klass> klasses = new ArrayList<>();
      
      public String getName() {
          return name;
      }
      
      public void setName(String name) {
          this.name = name;
      }
      
      public List<Klass> getKlasses() {
          return klasses;
      }
      
      public void setKlasses(List<Klass> klasses) {
          this.klasses = klasses;
      }
  }
  
  public class Klass {
      private String Name;
      private List<Student> students = new ArrayList<>();
      
      public String getName() {
          return Name;
      }
      
      public void setName(String name) {
          Name = name;
      }
      
      public List<Student> getStudents() {
          return students;
      }
      
      public void setStudents(List<Student> students) {
          this.students = students;
      }
  }
  
  public class Student {
      private String name;
      private Integer age;
      
      public String getName() {
          return name;
      }
      
      public void setName(String name) {
          this.name = name;
      }
      
      public Integer getAge() {
          return age;
      }
      
      public void setAge(Integer age) {
          this.age = age;
      }
  }
  ```

- 配置属性聚集

  负责读取application.properties属性

  ```java
  @ConfigurationProperties(prefix = "schoolconfig")
  public class SchoolConfig {
      private School school;
      
      public School getSchool() {
          return school;
      }
      
      public void setSchool(School school) {
          this.school = school;
      }
  }
  ```

- application.yaml

  application.properties

  ```properties
  schoolconfig.school.name=defaultSchool
  schoolconfig.school.klasses[0].name=maths
  schoolconfig.school.klasses[0].students[0].name=rain
  schoolconfig.school.klasses[0].students[0].age=19
  schoolconfig.school.klasses[0].students[1].name=yoyo
  schoolconfig.school.klasses[0].students[1].age=18
  schoolconfig.school.klasses[1].name=english
  schoolconfig.school.klasses[1].students[0].name=domi
  schoolconfig.school.klasses[1].students[0].age=1
  schoolconfig.enable=true
  ```

  application-dev.properties

  ```properties
  schoolconfig.school.name=devSchool
  schoolconfig.school.klasses[0].name=maths
  schoolconfig.school.klasses[0].students[0].name=rain
  schoolconfig.school.klasses[0].students[0].age=19
  schoolconfig.school.klasses[0].students[1].name=yoyo
  schoolconfig.school.klasses[0].students[1].age=18
  schoolconfig.school.klasses[1].name=english
  schoolconfig.school.klasses[1].students[0].name=domi
  schoolconfig.school.klasses[1].students[0].age=1
  schoolconfig.enable=true
  ```

- 自定义自动配置类

  当某一条件成立时才会向容器注册相应的Bean

  ```java
  @EnableConfigurationProperties(SchoolConfig.class)
  @ConditionalOnClass(School.class)
  public class SchoolAutoConfig {
      @Autowired
      private SchoolConfig schoolConfig;
      
      @Bean("mySchool")
      @ConditionalOnProperty(prefix = "schoolconfig", name = "enable", havingValue = "true")
      public School school() {
          return schoolConfig.getSchool();
      }
  }
  ```

- META-INF\spring.factories

  映射自定义自动配置类

  ```pro
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    com.sciatta.hadoop.java.spring.boot.schoolstarter.config.SchoolAutoConfig
  ```

- 测试

  `@ActiveProfiles` 可以激活待测试的profile。如果不加的话，默认读取的是默认的配置文件application.properties

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest(classes = {SchoolAutoConfig.class})
  @ActiveProfiles("dev")
  public class SchoolStarterDevProfileTests {
      @Autowired
      private SchoolAutoConfig schoolAutoConfig;
      
      @Autowired
      private School school;
      
      @Autowired
      private ApplicationContext context;
      
      @Test
      public void testSchoolAutoConfig() {
          School school = schoolAutoConfig.school();
          assertNotNull(school);
          
          assertEquals("devSchool", school.getName());
          assertEquals(2, school.getKlasses().size());
      }
      
      @Test
      public void testSchool() {
          assertEquals("devSchool", school.getName());
          assertEquals(2, school.getKlasses().size());
      }
  }
  ```



#### 使用Starter

**hadoop-java-spring-boot**

- 引用自定义Starter模块

  以IDEA为例，Open Module Settings | Dependencies | + 依赖的Starter模块

  此时，就可以正常注入Starter自动配置生效时定义的服务Bean

  ```java
  @Autowired
  private School school;
  ```

- 如果Starter指明了多个Profile，在运行时需要指定相应的Profile，如 `-Dspring.profiles.active=dev`

- 参数覆盖

  - 不指定 `-Dspring.profiles.active=dev`

    - application.properties不存在，默认使用自定义starter的application.properties

    - application.properties存在， 设置启用profile，则读取相应的profile

      ```properties
      spring.profiles.active=dev
      ```

      - <font color=red>替换默认profile中的部分参数</font>
        - program arguments `--schoolconfig.school.name=oldSchool`
        - VM options `-Dschoolconfig.school.name=newSchool`

    - <font color=red>如果application.properties设置启用 `spring.profiles.active=dev`，但同时还存在application-dev.properties，则完全替代在Starter中application-dev.properties的配置，不会依赖默认属性</font>

  - 指定 `-Dspring.profiles.active=dev`

    - application.properties不存在，默认使用自定义starter的application-dev.properties
    - <font color=red>application.properties存在，不管是否设置default，都会完全替代Starter中application.properties的配置，不会依赖默认属性</font>
    - <font color=red>替换默认profile中的部分参数</font>
      - application-dev.properties存在
      - program arguments `--schoolconfig.school.name=oldSchool`
      - VM options `-Dschoolconfig.school.name=newSchool`



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



# 常用注解

## Spring Boot 相关

### @SpringBootApplication

- 在main类添加，Spring Boot 项目的基石
- 是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合

```java
@SpringBootApplication
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```



## Spring Bean 相关

### @Autowired

- 自动导入对象到类中，被注入进的类同样要被 Spring 容器管理。比如：Service 类注入到 Controller 类中。
- 按类型注入
- 同 @Qualifier 使用，可以按名称注入



### @Resource

- 默认按名称注入
- name 按名称注入
- type 按类型注入



### @Component

- 通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。



###@Repository

-  对应持久层即 Dao 层，主要用于数据库相关操作。



### @Service

- 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。



### @Controller

- 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。



### @RestController

- 注解是`@Controller`和`@ResponseBody`的合集，表示这是个控制器 bean，并且是将函数的返回值直接填入 HTTP 响应体中，是 REST 风格的控制器。
- `@Controller` 不加 `@ResponseBody` 的话一般使用在要返回一个视图的情况，这种情况属于比较传统的 Spring MVC 的应用，对应于前后端不分离的情况。
- `@Controller` + `@ResponseBody` 返回 JSON 或 XML 形式数据



###  @Scope

- 声明 Spring Bean 的作用域
  - singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
  - prototype : 每次请求都会创建一个新的 bean 实例。
  - request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
  - session : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。



### @Configuration

- 一般用来声明配置类，可以使用 `@Component` 注解替代，不过使用 `@Configuration` 注解声明配置类更加语义化。
  - @Configuration注解的类被CGLIB代理，此类（@Component是其元注解）和@Bean注解的方法返回的Bean都被注册为bean definition；因此，@Bean注解的不同方法调用同一个@Bean注解的方法代表相同的实例（默认是singleton，在调用方法前，先查询容器）。
  - @Component注解的类没有被CGLIB代理，@Bean注解的不同方法为工厂方法，代表不同的实例，所以方法调用不能用于内部bean的依赖（多次调用@Bean注解的方法，返回不同的实例）。



## Spring MVC 相关

### @GetMapping

- 请求从服务器获取特定资源。（查）
- @GetMapping("users") 等价于@RequestMapping(value="/users",method=RequestMethod.GET)



### @PostMapping

- 在服务器上创建一个新的资源。（增）
- @PostMapping("users") 等价于@RequestMapping(value="/users",method=RequestMethod.POST)



### @PutMapping

- 更新服务器上的资源（客户端提供更新后的整个资源）。（改 / 全部更新）
- @PutMapping("/users/{userId}") 等价于@RequestMapping(value="/users/{userId}",method=RequestMethod.PUT)



### @DeleteMapping

- 从服务器删除特定的资源。（删）
- @DeleteMapping("/users/{userId}")等价于@RequestMapping(value="/users/{userId}",method=RequestMethod.DELETE)



### @PatchMapping

- 更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新）。（改 / 部分更新）



### @PathVariable

- 用于获取路径参数



### @RequestParam

- 用于获取查询参数

```java
// 请求的 url 是：/klasses/{123456}/teachers?type=web

@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```



###  @RequestBody

- 用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且**Content-Type 为 application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象。
- 一个请求方法只可以有一个`@RequestBody`，但是可以有多个`@RequestParam`和`@PathVariable`

```java
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```



### @ControllerAdvice

- 注解定义全局异常处理类

### @ExceptionHandler

- 注解声明异常处理方法

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * 请求参数异常处理
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       ......
    }
}
```



### @JsonIgnore

- @JsonIgnoreProperties 作用在类上用于过滤掉特定字段不返回或者不解析
- @JsonIgnore 一般用于类的属性上



### @JsonFormat

- 一般用来格式化 json 数据



### @JsonUnwrapped

- 扁平对象



## Spring Config 相关

### @Value

- 使用 `@Value("${property}")` 读取比较简单的配置信息‘
- `@value` 这种方式不被推荐



### @ConfigurationProperties

- 通过`@ConfigurationProperties`读取配置信息并与 bean 绑定
- <font color=red>自动注入配置类</font>
  - 属性类使用 @Component + @ConfigurationProperties 搭配使用，这样LibraryProperties作为普通Bean就可以被注入到其他类中
  - 属性类使用 @ConfigurationProperties("my-profile") ，需要使用@EnableConfigurationProperties(ProfileProperties.class)注册配置类

```java
// @Component + @ConfigurationProperties 搭配使用，这样LibraryProperties就可以被注入到其他类中
@Component
@ConfigurationProperties(prefix = "library")	// 读取application.yml
@Setter
@Getter
@ToString
class LibraryProperties {
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
}

// application.yml
library:
  location: X
  books:
    - name: A
      description: 1
    - name: B
      description: 2
    - name: C
      description: 3
```



### @PropertySource

- `@PropertySource`读取指定 properties 文件，如 @PropertySource("classpath:website.properties")



## Spring Validation 相关

**JSR(Java Specification Requests）**是一套 JavaBean 参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在 JavaBean 的属性上面，这样就可以在需要校验的时候进行校验了。

校验的时候我们实际用的是 **Hibernate Validator** 框架。Hibernate Validator 是 Hibernate 团队最初的数据校验框架，Hibernate Validator 4.x 是 Bean Validation 1.0（JSR 303）的参考实现，Hibernate Validator 5.x 是 Bean Validation 1.1（JSR 349）的参考实现，目前最新版的 Hibernate Validator 6.x 是 Bean Validation 2.0（JSR 380）的参考实现。

**推荐使用 JSR 注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`**



- 验证@RequestBody

  ```java
  // 验证的请求体
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class Person {
  
      @NotNull(message = "classId 不能为空")
      private String classId;
  
      @Size(max = 33)
      @NotNull(message = "name 不能为空")
      private String name;
  
      @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
      @NotNull(message = "sex 不能为空")
      private String sex;
  
      @Email(message = "email 格式不正确")
      @NotNull(message = "email 不能为空")
      private String email;
  
  }
  
  // 在需要验证的参数上加上了@Valid注解，如果验证失败，它将抛出MethodArgumentNotValidException
  @RestController
  @RequestMapping("/api")
  public class PersonController {
  
      @PostMapping("/person")
      public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
          return ResponseEntity.ok().body(person);
      }
  }
  ```

- 验证@PathVariable和@RequestParam

  - 在类上加 `Validated` 注解校验方法参数

  ```java
  @RestController
  @RequestMapping("/api")
  @Validated
  public class PersonController {
  
      @GetMapping("/person/{id}")
      public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
          return ResponseEntity.ok().body(id);
      }
  }
  ```

  

### @NotEmpty

- 被注释的字符串的不能为 null 也不能为空



### @NotBlank

- 被注释的字符串非 null，并且必须包含一个非空白字符



### @Null

- 被注释的元素必须为 null



### @NotNull

- 被注释的元素必须不为 null



### @AssertTrue

- 被注释的元素必须为 true



### @AssertFalse

- 被注释的元素必须为 false



### @Pattern(regex=,flag=)

- 被注释的元素必须符合指定的正则表达式



### @Email

- 被注释的元素必须是 Email 格式



### @Min(value)

- 被注释的元素必须是一个数字，其值必须大于等于指定的最小值



### @Max(value)

- 被注释的元素必须是一个数字，其值必须小于等于指定的最大值



### @DecimalMin(value)

- 被注释的元素必须是一个数字，其值必须大于等于指定的最小值



### @DecimalMax(value)

- 被注释的元素必须是一个数字，其值必须小于等于指定的最大值



### @Size(max=, min=)

- 被注释的元素的大小必须在指定的范围内



### @Digits (integer, fraction)

- 被注释的元素必须是一个数字，其值必须在可接受的范围内



### @Past

- 被注释的元素必须是一个过去的日期



### @Future

- 被注释的元素必须是一个将来的日期



## Spring JPA 相关

### 创建表

- `@Entity` 声明一个类对应一个数据库实体

- `@Table` 设置表名
- `@Id` 声明一个字段为主键
- `@GeneratedValue` 指定主键生成策略
  - TABLE 持久化引擎通过关系数据库的一张特定的表格来生成主键
  - SEQUENCE 在某些数据库中，不支持主键自增长，比如Oracle、PostgreSQL其提供了一种叫做"序列(sequence)"的机制生成主键
  - IDENTITY 主键自增长
  - AUTO 把主键生成策略交给持久化引擎(persistence engine)，持久化引擎会根据数据库在以上三种主键生成策略中选择其中一种

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    省略getter/setter......
}
```



### 字段类型

- `@Column` 声明字段
- `@Transient` 声明不持久化
- `@Lob` 声明某个字段为大字段
- `@Enumerated` 枚举类型的字段

```java
@Column(name = "user_name", nullable = false, length=32)
private String userName;

// ----

@Lob
//指定 Lob 类型数据的获取策略， FetchType.EAGER 表示非延迟 加载，而 FetchType. LAZY 表示延迟加载 ；
@Basic(fetch = FetchType.EAGER)
//columnDefinition 属性指定数据表对应的 Lob 字段类型
@Column(name = "content", columnDefinition = "LONGTEXT NOT NULL")
private String content;
```



### 增加审计功能

- 继承AbstractAuditBase

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}
```



### 删除修改数据

- `@Modifying` 注解提示 JPA 该操作是修改操作

```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {

    @Modifying
    @Transactional(rollbackFor = Exception.class)
    void deleteByUserName(String userName);
}
```



### 关联关系

- `@OneToOne` 声明一对一关系
- `@OneToMany` 声明一对多关系
- `@ManyToOne`声明多对一关系
- `@ManyToMany`声明多对多关系



### 事务

- 如果不配置 `rollbackFor` 属性只会在遇到 `RuntimeException` 的时候才会回滚
- @Transactional 注解可以作用在类或者方法上

  - 作用于类：当把@Transactional 注解放在类上时，表示所有该类的 public 方法都配置相同的事务属性信息
  - 作用于方法：当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
  ......
}
```

