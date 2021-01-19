# SpringBoot2基础入门



## Spring能做什么

### Spring的能力

MicroServices、Reactive、Cloud、webapp、serverless、enentDriven、batch

### Spring的生态

SpringFramework

https://spring.io/projects/spring-boot

## 为什么用SpringBoot

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".

### SpringBoot的优点

- Create stand-alone Spring applications

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)

- Provide opinionated 'starter' dependencies to simplify your build configuration

- Automatically configure Spring and 3rd party libraries whenever possible

- Provide production-ready features such as metrics, health checks, and externalized configuration

- Absolutely no code generation and no requirement for XML configuration

### SpringBoot的缺点

​	版本迭代快，封装深，内部原理复杂，不易精通

## 时代背景

### 微服务

[James Lewis and Martin Fowler (2014)](https://martinfowler.com/articles/microservices.html)  提出微服务完整概念。https://martinfowler.com/microservices/

> In short, the **microservice architectural style** is an approach to developing a single application as a **suite of small services**, each **running in its own process** and communicating with **lightweight** mechanisms, often an **HTTP** resource API. These services are **built around business capabilities** and **independently deployable** by fully **automated deployment** machinery. There is a **bare minimum of centralized management** of these services, which may be **written in different programming languages** and use different data storage technologies.-- [James Lewis and Martin Fowler (2014)](https://martinfowler.com/articles/microservices.html)

- 微服务是一种架构风格
- 一个应用拆分为一组小型服务
- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

### 分布式

**分布式的困难**

- 远程调用
- 服务发现
- 负载均衡
- 服务容错
- 配置管理
- 服务监控
- 链路追踪
- 日志管理
- 任务调度
- ......

**分布式的解决**

- SpringBoot + SpringCloud

  ![image-20210116165007430](img/img/image-20210116165007430.png)

**云原生**

## 官方文档架构

Spring Boot Reference Documentation：https://docs.spring.io/spring-boot/docs/current/reference/html/

查看版本新特性；

https://github.com/spring-projects/spring-boot/wiki#release-notes

## SpringBoot2入门

### 环境要求

Java8及以上

Maven 3.3及以上

配置要求：https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-system-requirements

源代码：https://gitee.com/leifengyang/springboot2

**maven设置**

```xml
阿里云的镜像
<mirrors>
      <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
      </mirror>
  </mirrors>
 
使用jdk1.8
  <profiles>
         <profile>
              <id>jdk-1.8</id>
              <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
              </activation>
              <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
              </properties>
         </profile>
  </profiles>
```

### Helloworld

1. #### 创建Maven工程

   记得看idea setting，build->build tools->maven 的设置

2. #### 引入依赖

   在pom.xml文件中引入依赖

   ```xml
   <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.3.4.RELEASE</version>
       </parent>
   
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
       </dependencies>
   ```

   

3. 创建主程序

   ```java
   /**
    * 主程序类
    * @SpringBootApplication：这是一个SpringBoot应用
    */
   @SpringBootApplication
   public class MainApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(MainApplication.class,args);
       }
   }
   ```

   

4. 编写业务 

   ```java
   @RestController
   public class HelloController {
       @RequestMapping("/hello")
       public String handle01(){
           return "Hello, Spring Boot 2!";
       }
   }
   ```

   

5. 测试
   直接运行main方法

6. 简化配置
   resources 下 application.properties

7. 简化部署

   添加依赖

   ```xml
    <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   
   ```

   maven->lifecycle->package

   打包成jar包，服务器可以直接用

## 自动配置原理入门

### 1.项目依赖

依赖管理  

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
</parent>

 <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
  </parent>
<--几乎声明了所有开发中常用的依赖的版本号,自动版本仲裁机制-->
```

导入starter
只要引入starter，这个场景的所有常规需要的依赖我们都自动引入
SpringBoot所有支持的场景
https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>2.3.4.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

**无需关注版本号，自动版本仲裁**
引入的依赖默认都可以不写版本
引入非版本总裁的JAR，需要写版本号

**可以修改默认版本号**
查看spring-boot-dependencies里面规定当前依赖的版本 用的 key。
或者在当前项目里重写配置

```xml
    <properties>
        <mysql.version>5.1.43</mysql.version>
    </properties>
```

### 2.自动配置

自动配置好tomcat


# SpringBoot2核心功能

