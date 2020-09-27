# 一、SpringBoot入门

## 1、SpringBoot简介

>简化Spring应用开发的一个框架
>
>整个Spring技术栈的一个大整合；
>
>J2EE开发的一站式解决方案



## 2、微服务

2014，martin fowler

微服务：架构风格

一个应用应该是一组小型服务，可以通过HTTP的方式进行互通



每一个功能元素最终都是一个可独立替换和独立升级的软件单元

详细参照微服务文档https://martinfowler.com/microservices/



## 3、Spring Boot HelloWorld

#### 3.1 创建一个maven工程

#### 3.2 导入依赖spring boot相关依赖

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
</dependency>
```

#### 3.3 编写主程序

```java
/**
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot 应用
 */
@SpringBootApplication
public class HelloworldApplication {

    public static void main(String[] args) {
        //启动Spring应用
        SpringApplication.run(HelloworldApplication.class, args);
    }
}
```

#### 3.4 编写相关的Controller、Service.....

```java
@Controller
public class HelloController {
    
    @ResponseBody
    @RequestMapping("hello")
    public String hello(){
        return "Hello World";
    }
}
```

#### 3.5 运行主程序测试

#### 3.6 简化部署

```xml
<!--这个插件，可以将应用打包成一个可执行的jar包-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

将这个应用打成jar包，直接使用java -jar的命令运行



## 4、Hello World探究

#### 4.1 POM文件

```xml
<parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>2.1.9.RELEASE</version>
     <relativePath/>
</parent>

他的父项目是
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.9.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
他来真正管理Spring Boot应用里面的所有版本依赖
```

Spring  Boot的版本仲裁中心

以后我们导入依赖默认是不需要写版本（没有在dependencies里面管理的依赖自然是需要声明版本）

#### 4.2 启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring-boot-starter-*==web==*

​		**spring-boot-starter**:spring-boot场景启动器；帮我们导入web模块正常运行所依赖的组件

Spring Boot将所有的功能场景都抽取出来，做成一个个都Starters（启动器），只需要在项目中引入这些starter相关场景都所有依赖都会导入进来，要用什么功能就导入什么场景都启动器；

#### 4.3 主程序类，入口类

```java
/**
 * @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot 应用
 */
@SpringBootApplication
public class HelloworldApplication {

    public static void main(String[] args) {
        //启动Spring应用
        SpringApplication.run(HelloworldApplication.class, args);
    }
}
```

**@SpringBootApplication**:Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法启动SpringBoot应用

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

**@SpringBootConfiguration**:Spring Boot的配置类；

​					标注在某个类上，表示这是一个Spring Boot的配置类；

​					**@Configuration**：配置类上来标注这个注解；配置类就相当于配置文件，配置类也是容器中的一个组					件；@Component

**@EnableAutoConfiguration**：开启自动配置功能；

​					以前我们需要配置的东西，Spring Boot帮我们自动配置；**@EnableAutoConfiguration**告诉SpringBoot开启自动配置功能；这样自动配置才能生效；

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

**@AutoConfigurationPackage**:自动配置包

​		**@Import(AutoConfigurationPackages.Registrar.class)**：

​		Spring的底层注解@Import，给容器中导入一个组件，导入的组AutoConfigurationPackages.Registrar.class

​		`将主配置类（@SpringBootApplication）的所在包及下面所有子包里面的所有组件扫描到Spring容器中`

**@Import(AutoConfigurationImportSelector.class)**:给容器中导入AutoConfigurationImportSelector组件

​		AutoConfigurationImportSelector：导入那些组件的选择器；

​		将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中

​		会给容器中导入非常多的自动配置类（xxxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件;

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

​		SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,ClassLoader)

**Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类生效，帮我们进行自动配置工作；**以前我们需要自己配置的东西，自动配置类都帮我们配置好了；

J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-2.1.9.RELEASE.jar；



## 5、使用Spring Initializer快捷创建Spring Boot项目

IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目

选择我们需要的模块，向导会联网创建Spring Boot项目；

默认生成的Spring Boot项目;

> 主程序已经生成好了，我们自需要实现我们

> 主程序已经生成好了，我们只需要实现我们自己的逻辑
>
> resources文件夹中的目录结构
>
> ​		static : 保存所有的静态资源; js css images;
>
> ​		templates: 保存所有的模版页面；（Spring Boot 默认jar包使用嵌入式的Tomcat，默认不支持jsp页面）；可以使用模版引擎（freemarker、thymeleaf）;
>
> ​		application.properties: Spring Boot应用的配置文件，可以修改一些默认的配置

​		



# 二、SpringBoot配置文件

## 1、配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

> application.properties
>
> Application.yml

配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层都给我们自动配置好；

YAML(YAML Ain't Markup Language)

​		YAML A Markup Language:是一个标记语言

​		YAML isn't Markup Language:不是一个标记语言

标记语言：

​		以前是配置文件；大多都使用的是**xxx.xml**文件；

​		YAML : 以数据为中心，比json、xml等更适合做配置文件；

​		YAML: 配置实例

```yaml
server:
  port: 8081
```

​		XML:

```xml
<server>
	<port>8081</port>
</server>
```









