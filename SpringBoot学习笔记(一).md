# 一. Spring Boot入门

##  1.启动器

pom.xml配置信息

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring-boot-starter-web:

  Spring-boot-starter-web，他是spring-boot场景启动器，帮我们导入了web模块组成正常运行需要的所有依赖组件。

Spring Boot 将所有的功能场景都抽取出来，封装成各种starter（启动器）, 只需要在项目中更具需求引入相关starter，spring boot将会自动导入所有相关所需依赖。



## 2.主程序类，主入口类

```java
@SpringBootApplication
public class SellApplication {

    public static void main(String[] args) {
        SpringApplication.run(SellApplication.class, args);
    }

}
```

@SpringBootApplication: spring boot 应用标注在某个类上，说明该类是SpringBoot的主配置类，springBoot就应该运行这个类的main方法来启动springBoot应用。

@SpringBootApplication，该注解是个组合注解，我们可以查看器源码如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
  ......
}
```

@**SpringBootConfiguration**：  SpringBoot的配置类，当它标注在某个类上，标识该类是一个SpringBoot的配置类。

​	@**Configuration**：配置类上来标注这个注解，也就是以前Spring项目的配置文件，他实际上就是容器中的一个组件。@Component。

**@EnableAutoConfiguration**：开启自动配置功能。以前我们需要配置的功能，SpringBoot帮我们自动配置。@EnableAutoConfiguration这个注解就是告诉SpringBoot项目，需要开启自动配置功能。



**@EnableAutoConfiguration**: 他本是也是个组合注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
  ......
}
```

**@AutoConfigurationPackage**: 自动配置包

​		**@Import({Registrar.class})**：Spring的底层注解@Import，它可以给容器中导入一个组件，需要导入的组件就是括号内的类Registrar.class。 <u>这个类中有个方法registerBeanDefinitions(), 该方法的作用就是，将住配置类（@SpringBootApplication标注的类）所在的包及下面所有的子包里面的所有组件都扫描到Spring容器中。</u>



在**@EnableAutoConfiguration**这个组合注解中，我们可以看到，他还有一个@Import

@Import({AutoConfigurationImportSelector.class})， 他是自动导入了一些映入选择器，什么意思呢？ 他的意思就是自动导入所有所需的组件，已字符串数组类型返回，数组中存放的就是所需组件的全类名。然后这些组件就会自动的添加到我们的Spring容器中。其实这些组件就是一个个的自动配置类(***AutoConfiguration)。

**综上，SpringBoot在启动的时候，会从类路径下的META-INF/spring.factories中获取到EnableAutoConfiguration指定的值，然后将这些值作为自动配置类导入到如期中，自动配置类因此生效，帮我们完成项目配置工作。**

J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-*. * .*. RELEASE.jar这个jar包中。



## 3. 使用Spring Initializer快速创建Spring Boot项目

我们使用IDE快速创建一个SpringBoot项目时候，我们需要选择创建Spring Initializer项目。

然后根据我们的需求选择我们所需的功能模块，然后创建Spring Boot项目。

默认生成的Spring Boot项目的结构：

- 主程序已经自动生成，我们只需要编写我们自己的业务逻辑代码即可。

- resources文件夹目录结构：

  - static: 保存所有的静态资源，例如js、css、images等静态资源文件。
  - templates: 保存虽有的模板页面，（Spring Boot默认jar包使用嵌入式的tomact， 默认情况下是不支持JSP页面的。）
  - application.properties： Spring Boot应用的配置文件。在该文件中，我们可以进行自定义配置。

  

# 二.  配置文件

## 1.配置文件说明：

Spring Boot 使用一个全局配置文件，其文件名是固定的。(文件后缀，可以是``.properties`` , 也可以是``.yml``, 我们可以自动修改其后缀名)

- Application.properties
- Application.yml

配置文件的作用：修改Spring Boot自动配置的默认值，Spring Boot通过自动配置文件，在底层就一些配置项都进行了自动默认配置，如果我们需要自定义一些默认配置，我们就可以在application.yml文件中进行自定义配置。

例如我们配置默认的启动端口号：

```yaml
server:
	port: 8081
```



## 2.YAML语法

### 2.1基本语法

key: (space空格) value  -> 表示一对键值对，空格必须有

以空格的缩进来控制层级关系，只有左对齐的一列数据，是同一层级。例如：

```yaml
server:
	port: 8080
	path: /user
```

注意，key和value是大小写敏感的。

### 2.2 值得写法

常量： 数字， 字符串， bool类型：

key: (space 空格) value   --> 字面量直接定义。

​												 字符串

​												“xxx”: 双引号字符串， 不会转移字符串里的特殊字符，

​									      	  ’xxxx‘: 单引号字符串， 他会转移特殊字符串

对象，map(key: value) :

 Key: (space空格) value, 是以键值对形式表达。

例如： 

```yaml
user:
	name: 'taeja'
	age: 10
	objectRecord:
		Chinese: 80
		math: 90
//也可以使用行内写法
user: {name: 'taeja', age: 10, objectReocrd: {Chinese: 80, math: 90}}
```



集合类型： Array, Set

使用 ``-``来标识元素， 例如

```yaml
animals:
	- cat
	- dog
	- pig
//行内写法
animals: [cat, dog, pig]
```



## 3 读取配置文件中的值运用的方法

1.如果我们的业务逻辑只需要获取配置文件的某一个值，我们则使用``@value``注解。例如：

```java
@RestController
@ReqeustMapping("/user")
public class UserController {
	
  @Value("${user.name}")  //只需要简单的从配置文件中读取一个值，直接使用@Value
  private String name;
  
  @GetMapping("/hello")
  public String sayHello() {
    return "Hello " + name;
  }
}
```

2.如果我们单独创建一个JavaBean来和我们的配置文件进行映射，我们则使用``@ConfigurationProperties``注解。例如：

```java
@ConfigurationProperties
public class User {
  private String name;
  private Integer age;
  private ObjectRecord objectRecord;
}

@ConfigurationProperties
public class ObjectRecord {
  private Integer Chinese;
  private Integer math;
}
```

### 3.1 @Value和@ConfigurationProperties获取配置文件值的区别

|                      | @ConfigurationProperties | @Value   |
| -------------------- | ------------------------ | -------- |
| 功能                 | 批量注入配置文件中的属性 | 单个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持   |
| SpEL                 | 不支持                   | 支持     |
| JSR303校验           | 支持                     | 不支持   |
| 复杂数据类型封装     | 支持                     | 不支持   |

## 5 @PropertySource 和 @ImportResource的使用

**@PropertySource**通常的使用场景是：

如果我单独创建了一个配置文件，我们想要将该配置和我们JavaBean进行绑定，由于该配置文件是我们自定义的配置的文件，所以我们需要在的Bean类上添加@Prop ertySource注解来申明该配置文件。例如：

自定义配置信息：

```properties
//userinfo.properties

user.first-name = taeja
user.last-name = liquid
user.age = 28
user.record.English = 80
user.record.Math = 90

```

实体类：

```java
@Component
@PropertySource(value = {"classpath:userinfo.properties"})
@ConfigurationProperties(prefix = "user")
public class User {

    private String firstName;
    private String lastName;

    private Integer age;
    private Record record;
  
  	//getter and setter ...
}
  
public class Record {
  private Integer English;
  private Integer Math
}
```



@ImportResource的使用场景，导入我们自定义的xml配置文件，让其配置内容生效。

当我们自定义一个xml配置文件的时候，此时编译器是不能自动识别我们的自定义xml配置文件的，如果想让编译器识别到我们的配置文件，则我们需要在配置类上添加这个注解

```java
//导入我们的spring自定义xml配置文件，让其生效
@ImportResource(location = {"classpath: ***.xml"})
```

当时这种自定义spring的xml配置文件，在springBoot项目是不推荐使用的。



## 6.配置文件占位符

#### 6.1随机数

```java
${random.value},
${random.int},
${random.long},
${random.int(10)},
${random.int[1024, 65536]},
```

6.2占位符获取之前的配置值，如果没有改配置值，使用``:``来是设置默认值得

```properties
user.first-name = taeja
user.last-name = liquid
user.age = ${random.int}
user.pet-name = ${user.first-name:default}_pet
user.record.English = 80
user.record.Math = 90
```











