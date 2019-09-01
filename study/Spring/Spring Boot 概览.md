# Spring Boot

## 简介

Spring Boot 及时一个简化 Spring 开发的框架。用来监护 Spring 应用开发，约定大于配置，去繁就简，`just run` 就能创建一个独立的、产品级别的应用。

我们在使用 Spring Boot 时只需要配置相应的 Spring Boot 就可以用所有的 Spring 组件，简单地说，Spring Boot 就是整合了很多优秀的框架，不用我们自己手动去写XML 配置文件然后进行配置。从本质上讲，Spring Boot 就是 Spring，他做了那些没有它你也会去做的 Spring 配置。

## 优点

* 快速创建独立运行的 Spring 项目以及主流框架集成
* 使用嵌入式的 Servlet 容器，应用无需达成 WAR包
* starters 自动完成依赖和版本控制
* 大量的自动配置，简化开发，也可修改默认值
* 无需配置 XML，无代码生成，开箱即用
* 准生产环境的运行时应用监控
* 与云计算天然集成

## 单体应用于微服务

单体应用是将所有的应用模块都写在一个应用中，导致项目越写越大，模块之间的耦合度也会也来越高。

## Spring Boot 核心特点

### 微服务

使用 Spring Boot 可以生成独立的微服务功能单元

### 自动配置

针对很多 Spring 应用程序常见的应用功能，Spring boot 能够自动提供相关配置

### 起步依赖

告诉 Spring Boot 需要什么功能，它就能引入需要的库。

### 命令行界面

这是 Spring Boot 的可选特性， 借此你只需要写代码就能完成完整的应用程序，无需诚通项目构建

### Actuator

让您能够深入运行中的 Spring Boot 应用程序

## 简单案例一般流程

* 创建 maven 项目
* 引入 starters
* 创建主程序
* 启动运行

$1.首先要配置 Spring Boot 依赖$

``` xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.5.9.RELEASE</version>
</parent>

<!-- Spring Boot的版本仲裁中心；以后我们导入依赖默认是不需要写版本。（没有在dependencies里面管理的依赖自然需要声明版本号）  -->

<dependencies>
   <dependency>
       <groupId>org.springframework.boot</groupId>   
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
</dependencies>
```

**spring-boot-starter-web:**
 
`spring-boot-starter` 为 Spring Boot场景启动器。帮我们导入了 web 模块正常运行所依赖的组件。

Spring Boot 将所有的功能场景都抽取出来了，做成了一个个的`starters`(启动器)，只需要在项目里面引入这些 starter， 相关场景的所有依赖都会导入进来。用什么功能就导入什么场景的启动器

$2.新建包结构$

maven 项目包结构，主程序位置高于 Java 代码一级

$3.编写相应的类代码$

**HelloController.java**

``` java

/**
* 表明是一个 Controller 类，并且类中方法支持json 格式数据输出
**/
@RestController
public class HelloController {

    // 可以通过 http://locahost:8080/sayHello 访问该方法
    @RequestMapping("/sayHello")
    public String hello() {
        return "hello world";
    }
}
```

**MainTest.java**

``` java

// 表示 Spring Boot 的启动类，用来标注主程序，说明是一个 Spring Boot 应用
@SpringBootApplication
public class MainTest {
    public static void main(String[] args) {
        // 将 Spring Boot 应用程序驱动起来
        public class MainTest {
            SpringApplication.run(MainTest.class);
        }
    }
}
```

$4.运行 main 方法$

运行 main 方法，等到自动部署服务完成，然后通过地质可以访问到相应的数据

## 案例解析--相应的问题

### 为什么只配置一个 Spring Boot 依赖

Spring Boot 为我们提供了简化企业级开发的绝大多数场景的`starters pom`(启动器)，只需要引入相应场景的 starter pom，相关技术的绝大多数配置将会消除（自动配置），从而简化我们的开发。业务中我们就会使用到Spring Boot 为我们自动配置的 bean。

### 入口类的`@SpringBootApplication`注解

>>>> 待详细了解

* 程序充 main 方法开始运行
* 使用 SpringApplication.run()加载主程序类
* 主程序类需要标注 `@SpringBootApplication`
* `@EnableAutoConfiguration`是核心注解
* `@Import`导入所有自动配置场景
* `@AutoConfigurationPackage`定义默认的包扫描规则
* 程序启动扫描加载主程序类所在的包以及下面所有子包的组件
* `ComponentScan` `Filter` `Configuration` `Component`

### AutoConfigurationPackage

自动扫描`@SpringBootApplication`标记的主类，在主类所在的包或者所在的子包下面找相应的控制类。然后根据相应的注解自动配置相应的项目所需的 bean。

**xxxAutoConfiguration**

>>>> 待详细了解

* Spring Boot 中存在大量的这些类，这些类的作用就是帮我们进行自动配置
* 它会将这个场景需要的所有组件都注册到容器中，并配置好
* 它们在类路径下的 `META-INF/Spring.factories`文件中
* `Spring-boot-autoconfiguration-1.5.9.RELEASE.jar` 中包含了所有场景的自动配置类代码
* 这些自动配置类是 Spring Boot 进行自动配置的精髓

## Spring Boot配置文件

使用 Spring Boot 配置文件首先需要 Javabean 类，然后才能在相应的配置文件中为 Javabean 中的属性赋值

想要在 Javabean类中赋值，需要在 Javabean 的类中添加相应的注解，比如：`@Component` 和 `ConfigurationProperties(prefix = "person")` ，前者的作用是将类添加到容器，只有添加到容器中才能成为容器的组件，才能使用容器提供的`@ConfigurationProperties` 关联的功能，后者的作用是将配置文件中的数据注入类中，即告知 Spring Boot 将本类中的所有属性与配置文件中的相关内容配置进行绑定，括号内的 `prefix = "person"`用来筛选和映射配置文件中相关属性设置

$配置文件$

配置文件名是固定的 application.properties 或 application.yml

$配置文件的作用$

修改 Spring Boot 自动配置的默认值，SpringBoot 在地层都给我们自动配置好

### application.properties 详解

$1.修改默认8080端口$

``` xml
server.port = 8989
```

$2.其他$

``` xml
person.age = 80
person.birthday = 2019/08/01
person.list = a, b, c
person.name = huanyu
person.sex = male
person.map.v1 = k1
person.map.v2 = k2
person.dog.name = jjj
person.dog.age = 7
```

> 注意：在使用 `application.properties` 配置文件时，可能会出现乱码，注意改变编码格式为 UTF-8

### yaml 文件详解

![Xnip2019-08-01_16-48-32](/assets/Xnip2019-08-01_16-48-32.png)

## Spring Boot 的特殊注解

$@SpringBootApplicationn$

是一个符合注解，包括 `@ComponentScan` `@SpringBootConfiguration` `@EnableAutoConfiguration`

1. `@ComponentScan` 作用为扫描当前包及其子包中被 `@Component, @Controller, @Service, @Repository`注解标记的类，并将其纳入 Spring 容器中进行管理。在使用 Spring 进行组件管理时，使用了 `<context: component-scan>` xml 标签。
2. `SpringBootConfiguration` 继承自 `@Configuration`，二者功能也一致，标注当前类是**配置类**，并且会将在当前类中声明的一个或多个以 `@Bean` 注解标记的方法实例纳入到 Spring容器中，并且实例名称就是方法名。
3. `@EnableAutoConfiguration` 作用是启动自动配置，即就是 Spring Boot 会根据你添加的 jar 包来设置你项目中的默认配置，比如根据 Spring-boot-starter-web 来判断你的项目是否需要添加 `webmvc`， `tomcat`，如需要，则会自动为你添加 web 项目所需的默认配置。

$@ResponseBody$

表示被该注解标注的方法的返回结果直接写入 HTTP Response Body中，用于构建 RESTFUL 的 API，一般在异步获取数据时使用。

该注解通常会配合 `@RequestMapping` 一起使用，一般在使用 `RequestMapping`后，返回值通常会解析为跳转路径，加上 `@ResponseBody` 后，返回值不会被解析为跳转路径，而是直接写入 HTTP Response Body 中。比如异步获取 json 数据，在加上 `@ResponseBody` 后，会直接返回json 数据。

$@Controller$

用于定义控制器类，在 Spring 项目中，控制器负责江永胡发过来的 URL 请求转发到对应的服务接口（service 层），一般这个注解在类上，通常需要配合 `@RequestMapping`使用。

$@RestController$

用于标注控制层组件，是 `@ResponseBody` 和 `@Controller` 的合集，表明该类为 Controller 类，旗下方法都支持 json 格式数据输出。

$@Service$

一般用于修饰 Service 层的组件

$Repository$

>>>> 待详细了解

使用 `@Repository` 注解可以确保为 Dao类 或者 repositories类 提供异常转译，这个注解修饰的 Dao类 或者 repositories 类会被 ComponentScan 发现并配置，同时也不需要为他们提供 xml 配置项。

$Bean$

使用 @Bean 注解标注的方法等价于 xml 中配置的 bean

$Value$

注入 Spring Boot 的 application.properties 配置的属性值。

## 开发环境的调试

热启动在正常开发项目中已经很常见，虽然平时在开发 web 项目过程中，改动项目后重启总是报错，但是 Spring Boot 对调试支持很好，修改之后就可以实时生效，为此，一般需要添加如下配置：

``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

该模块在完整的打包环境下运行的时候会被禁用。如果您使用 `java -jar` 启动应用或者用一个特定的 `classloader` 启动，它会认为这是一个 “生产环境”。

## Web 开发

### json 接口设置

使用 `@TestController` 注解

### 自定义 Filter

### 自定义 Property

### log 配置

### 数据库操作

### Thymeleaf 模板

Java领域的表现层技术：JSP、freemarker、Velocity、Thymeleaf

Spring Boot 推荐使用 Thymeleaf 来代替 JSP， Thymeleaf 是一款用于渲染 XML/XHTML/HTML5 内容的模板引擎。类似 JSP、Velocity、FreeMaker 等，它也可以轻易地与 Spring MVC 等 web 框架进行集成作为 web应用的模板引擎。与其他模板引擎相比， T话要么leaf 最大的特点是能够直接在浏览器中打开并正确显示模板页面，而不需要启动整个 web 应用。

Thymeleaf 是与众不同的，因为它使用了自然的模板技术。这意味着 Thymeleaf 的模板语法并不会破坏文档的结构，模板依旧是有效的XML文档。模板还可以用作工作原型，Thymeleaf 会在运行期替换掉静态值。Velocity 与 FreeMarke r则是连续的文本处理器。

>> 注意，由于 Thymeleaf 使用了 XML DOM 解析器，因此它并不适合于处理大规模的 XML 文件。

>>> Html 表现： Hogan.js is a compiler for the Mustache templating language

$URL$

URL 在 web 应用模板中占据着十分重要的地位，需要特别注意的是 Thymeleaf 对于 URL 的处理是通过语法 `@{...}` 来处理。Thymeleaf 支持绝对 URL：

``` xml
<a th:href="@{http://www.thymeleaf.org}">Thymeleaf</a>
```

$条件求值$

``` xml
<a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
```

$for循环$

``` xml
<tr th:each="prod : ${prods}">
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```

### 页面即原型

>>>> 待续 ❓

### WebJars

WebJars 是一个很神奇的东西，可以让大家以 Jar 包的形式来使用前端的各种框架、组件。

什么是 WebJars

WebJars 是将客户端（浏览器）资源（JavaScript，Css等）打成 Jar 包文件，以对资源进行统一依赖管理。WebJars 的 Jar 包部署在 Maven 中央仓库上。

为什么使用

我们在开发 Java web 项目的时候会使用像 Maven，Gradle 等构建工具以实现对 Jar 包版本依赖管理，以及项目的自动化管理，但是对于 JavaScript，Css 等前端资源包，我们只能采用拷贝到 webapp 下的方式，这样做就无法对这些资源进行依赖管理。那么 WebJars 就提供给我们这些前端资源的 Jar 包形势，我们就可以进行依赖管理。

如何使用
1、 WebJars主官网 查找对于的组件，比如 Vuejs

``` xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>vue</artifactId>
    <version>2.5.16</version>
</dependency>
```

2、页面引入

``` xml
<link th:href="@{/webjars/bootstrap/3.3.6/dist/css/bootstrap.css}" rel="stylesheet"></link>
```

## Thymeleaf 详解



