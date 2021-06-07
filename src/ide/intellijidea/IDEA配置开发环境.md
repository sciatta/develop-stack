# Java环境

## 配置JDK环境变量

```bash
vi ~/.zshrc

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH

# 即时生效
source ~/.zshrc
```



## 配置JDK集成开发环境

- 项目右键 |  open module settings | sdks | + jdk 指定 jdk home path

- 项目右键 |  open module settings | project 指定sdk

- 项目右键 |  open module settings | modules | language level: 8

- Preferences | Build, Execution, Deployment | compiler | java compiler
  1. module 设置为8
  2. project bytecode version 设置为8
  
- Preferences | Editor | Inspections | SQL | Unresolved reference 取消选中

  解决 'Unable to resolve table' 问题

- Preferences | Build, Execution, Deployment | Debugger | Stepping

  do not step into the classes

  <font color=red>去掉 java.* 和 javax.*，否则无法进入JDK源码进行调试</font>
  
- Preferences | Editor | Inspections | Java | Serialization issues | Serializable class without 'serialVersionUID' 

  实现 `Serializable` 接口的类需要提供 `serialVersionUID` 并可自动生成



## 插件

- jclasslib Bytecode viewer 反编译



## 自定义文件格式

- Preferences | Editor  | File and Code Templates

  1. 新建Template，name：Schema File，extension：xsd

  2. 创建模板

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
              xmlns="http://www.sciatta.com/schema"
              targetNamespace="http://www.sciatta.com/schema"
              elementFormDefault="qualified">
     </xsd:schema>
     ```

  3. 注意修改 `xmlns` 和 `targetNamespace` 为实际的Namespace

  4. 解决问题

     - `idea  lineNumber: 1; columnNumber: 1; 前言中不允许有内容` 

       需要按照UTF-8无BOM方式编码。确认idea配置 `Preferences | Editor  | File Encodings` with NO BOM



# Scala环境

## 配置SDK环境变量

```bash
tar -zxvf scala-2.11.8.tgz -C ../install

vi ~/.zshrc

export SCALA_HOME=/Users/yangxiaoyu/work/install/scala-2.11.8
export PATH=$SCALA_HOME/bin:$PATH

# 即时生效
source ~/.zshrc
```



## 配置SDK集成开发环境

### 下载 Scala 插件

- Preferences | plugins | Scala



### 新建 pom module



### 新建 jar module



### 指定SDK

- File | Project Structure | Libraries | + | Scala SDK | 选择 scala-2.11.8
- 项目右键 |  open module settings | modules | hadoop-scala-example | Dependencies | + | Libraries | 选择 scala-sdk-2.11.8



### Java混编Scala

- src/main 新建 Directory | scala
  
- Mark Directory as | Sources Root
  
- src/test 新建 Directory | scala
  
- Mark Directory as | Test Sources Root
  
- 配置maven插件

  ```xml
  <plugin>
  	<groupId>org.apache.maven.plugins</groupId>
  	<artifactId>maven-compiler-plugin</artifactId>
  	<executions>
  		<execution>
  			<phase>compile</phase>
  				<goals>
  					<goal>compile</goal>
  				</goals>
  		</execution>
  	</executions>
  	<configuration>
  		<source>1.8</source>
  		<target>1.8</target>
  		<encoding>UTF-8</encoding>
  	</configuration>
  </plugin>
  
  <plugin>
  	<groupId>net.alchim31.maven</groupId>
  	<artifactId>scala-maven-plugin</artifactId>
  	<executions>
  		<execution>
  			<id>scala-compile-first</id>
  			<phase>process-resources</phase>
  			<goals>
  				<goal>add-source</goal>
  				<goal>compile</goal>
  			</goals>
  		</execution>
  		<execution>
  			<id>scala-test-compile</id>
  			<phase>process-test-resources</phase>
  			<goals>
  				<goal>testCompile</goal>
  			</goals>
  		</execution>
  	</executions>
  </plugin>
  ```

  

### 打包运行

- maven | package



### 下载 Scala 反编译插件

Preferences | plugins | CFR Decompile