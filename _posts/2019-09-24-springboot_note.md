---
layout:     post                    # 使用的布局（不需要改）
title:      2019-09-24-springboot_note  # 标题 
subtitle:    学习笔记  #副标题
date:       2019-09-24              # 时间
author:     alice                      # 作者
catalog: true                       # 是否归档
tags:                               #标签
        - note for springboot
---


###  配置文件

#### 1.配置文件

SpringBoot使用一個全局配置文件

【全局配置作用：修改SpringBoot自动配置的默认值】

application.properties

application.yml

【yml即YAML】

标记语言：

​	大多以前的配置文件使用xml，yml以数据为中心比json、xml更适合做配置文件

e.g yml文件

```yml
server:
  port: 8082
```

e.g xml文件

```xml
<server>
	<port>8082</port>
</server>
```

#### 2.YAML语法

key: value 【冒号后有空格】

```yml
server:
	port: 8082
	path: /xx
```

port 和 path左对齐，表示是同一层级的

##### **值的写法：**

<u>①字面量：普通的值（数字、字符串、布尔）</u>

​	key: value 【如果是字符串不需要加单、双引号】

​		【双引号不会转义字符串中的特殊字符原样输出，而单引号会转义按照规则显示】 

<u>②对象、Map（属性和值 键值对）</u>

​	key: value 

​	e.g 

```yml
 friends:
 		lastName:  xx
		age: 20
```

行内写法e.g：

```yml
friends{lastName: xx,age: 20}
```

<u>③数组（List、Set）</u>

用- 值表示数组中一个元素

e.g 

```yml
pets:
 - cat
 - dog
```

行内写法e.g

```yml
pets: [cat,dog]
```

##### **配置文件注入**

###### @ConfigurationProperties

配置文件 application.yml

```yml
person:
  lastName: xx
  age: 20
  boss: false
  birth: 1998/01/17
  maps: {k1: v1,k2: v2}
  lists:
    - l1
    - l2
  dog:
    name: xxx
    age: 2
```

application.properties

```properties
person.last-name=xx
person.age=20
person.birth=1998/01/17
person.boss=false
person.maps.k1=v1
person.maps.k2=v2
person.lists=a1,a2,a3
person.dog.name=name
person.dog.age=2
```

properties文件是ascii编码，idea默认utf-8，可在file encoding中修改

javaBean:

```java
/**
 * 将配置文件中属性的值映射到这个组件中，需要使用注解
 * @ConfigurationProperties 让springboot将类中的属性与配置文件中的属性进行绑定
 * perfix = "Person" 与配置文件中的Person下的值对应
 * @ConfigurationProperties 如果要正常起作用，一定要是spring容器中的组件，所以@Component把Person加入容器中
 */

@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

test:

```java

/**
 * springboot 单元测试
 * @RunWith(SpringRunner.class) 用spring的驱动器跑
 *
 * 可以在测试期间类似编码一样进行自动注入到容器的功能
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot02ApplicationTests {

    @Autowired
    Person person;

    @Test
    public void contextLoads() {
        System.out.println(person);
    }

}

```

导入配置文件处理器【导入后编译器会有提示】：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

###### @Value

除了使用@ConfigurationProperties(prefix = "xxx")来获取配置文件中的数据，还可以使用@Value

```java
@Component
public class Person {
	@Value("${person.last-name}")
    private String lastName;
    @Value("#{11*2}")
    private Integer age;
    @Value("true")
    private boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

**@Value的使用**

①<u>@Value("#{}")  Spring 表达式语言(简称SpEL)</u>

SpEL 字面量：

整数：#{8}
小数：#{8.8}
科学计数法：#{1e4}
String：可以使用单引号或者双引号作为字符串的定界符号。
Boolean：#{true}
SpEL引用bean , 属性和方法：

引用其他对象:#{car}
引用其他对象的属性：#{car.brand}
调用其它方法 , 还可以链式操作：#{car.toString()}
调用静态方法静态属性：#{T(java.lang.Math).PI}
 SpEL支持的运算符号：

算术运算符：+，-，*，/，%，^(加号还可以用作字符串连接)
比较运算符：< , > , == , >= , <= , lt , gt , eg , le , ge
逻辑运算符：and , or , not , |
if-else 运算符(类似三目运算符)：？:(temary), ?:(Elvis)
正则表达式

②<u>@Value("true") 直接赋值</u>

③<u>@Value("${person.name}")</u> //从配置文件application.properties

中取值

###### **@Value和@ConfigurationProperties的获取值区别**

|                | @ConfigurationProperties | @Value |
| -------------- | ------------------------ | ------ |
| 功能使用       | 批量注入                 | 单个   |
| 松散绑定       | 支持                     | 不支持 |
| JSR303数据校验 | 支持                     | 不支持 |
| spEL           | 不支持                   | 支持   |
| 复杂类型封装   | 支持                     | 不支持 |

- 松散绑定：

  标准：person.firstName

  法一：person.first_name [大写用 - ]

  法二：person.first-name [大写用 _ ]

  法三：PERSON_FIRST_NAME

- JSR303数据校验

  可以在Person前加@Vaildated 表示需要校验

- 复杂类型封装

  ```java
      @Value("${person.maps}")
      private Map<String,Object> maps;
  ```

  此时@Value无法获取到值

  

  ☆在单独获取配置文件某一个值的时候 可以使用@Value

  e.g helloController

  ```java
  @RestController
  public class helloController {
      @Value("${person.last-name}")
      private String name;
  
      @RequestMapping("/hello")
      public String hello(){
          return "hello"+name;
      }
  }
  ```

  如果专门写一个javabean来与配置文件映射 使用@ConfigurationPreperties



**@PropertySource与@ImportResource**

- @PropertySource:加载指定的配置文件

【@ConfigurationPreperties默认从全局配置文件中获取值】

javabean：

```java

/**
 * 将配置文件中属性的值映射到这个组件中，需要使用注解
 * @ConfigurationProperties 让springboot将类中的属性与配置文件中的属性进行绑定
 * perfix = "Person" 与配置文件中的Person下的值对应
 * @ConfigurationProperties 如果要正常起作用，一定要是spring容器中的组件，所以@Component加入容器中
 */

@Component
@ConfigurationProperties(prefix = "person")
@PropertySource(value = {"classpath:person.properties"})
public class Person {

    private String lastName;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

person.properties

```properties
person.last-name=alice
person.age=20
person.birth=1998/01/17
person.boss=false
person.maps.k1=v1
person.maps.k2=v2
person.lists=a1,a2,a3
person.dog.name=name
person.dog.age=2
```

- @ImportResource:导入Spring的配置文件，让配置文件里面的内容生效

  Springboot没有Spring的配置文件，自己编写的配置文件不能自动识别

  让Spring的配置文件生效，将该注解标注在配置类上

  beans.xml【现在不来编写Spring的配置文件】

```xml
  <bean class="com.xx.springboot02.service.helloService" id="helloService"></bean>
```

​		test：

```java
   @Autowired
    ApplicationContext ioc;

    @Test
    public void testHelloServide(){
        boolean b = ioc.containsBean("helloService");
        System.out.println(b);
    }

```

true

主配置类：

```java
@ImportResource(locations = {"classpath:beans.xml"})
```



###### **SpringBoot推荐给容器中添加组件的方式：**

**（推荐全注解的方式）**

1.配置类【Spring配置文件】

2.使用@Bean给容器中添加组件

```java
/**
 * @Configuration 指明当前类是一个配置类，用来代替之前的spring配置 beans。xml
 *
 * 在配置文件中用<bean></bean>标签添加组件
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中，容器中这个组件默认的id就是方法名
    //把helloService添加到容器中，组件的名字为hs
    @Bean
    public helloService hs(){
        System.out.println("xxtrue");
        return new helloService();
    }
}
```

test：

```java
    @Test
    public void testHelloServide(){
        boolean b = ioc.containsBean("hs");
        System.out.println(b);
    }

```

true

###### 配置文件占位符

**1、随机数**

```properties
person.last-name=xx${random.uuid}
person.age=${random.int}
person.birth=1998/01/17
person.boss=false
person.maps.k1=v1
person.maps.k2=v2
person.lists=a1,a2,a3
person.dog.name=name${person.last-name}
person.dog.age=2
```

${random.int} ${random.value} 

**2、占位符获取之前配置的值，没有则可以为其指定默认值**

```properties
person.last-name=xx${random.uuid}
person.age=${random.int}
person.birth=1998/01/17
person.boss=false
person.maps.k1=v1
person.maps.k2=v2
person.lists=a1,a2,a3
person.dog.name=name${person.hello:hello}
person.dog.age=2
```

如果是person.dog.name=name${person.hello}

输出dog{name=‘${person.hello}’，age=2}

如果是person.dog.name=name${person.hello:hello}

输出dog{name=‘hello’，age=2}



###### Profile

**1、多profile文件**

在主配置文件编写的时候，文件名可以是 application-{profile}.properites/yml

创建多个配置文件时，默认使用application.properties的配置

**2、yml支持多文档块方式**

```yml
server:
  port: 8082
spring:
  profiles:
    active: dev
---
server:
  port: 8083
spring:
  profiles: dev
---
server:
  port: 8084
  spring:
    profiles: prod
```

**3、激活指定profile**

- 在配置文件application.properties中，指定spring.profiles.active=dev （application-dev.properties）

- 命令行的方式：

  --spring.profiles.active=dev

(edit---program arguments中设置)

- 虚拟机参数

  -Dspring.profiles.active=dev

(edit---VM options中设置)



###### 配置文件加载位置

application.properties加载顺序（优先级从高到低）：

file:../config/

file:../

classpath:/config

classpath:/

springboot会全部加载这四个位置，优先级高覆盖低

**互补配置**：高优先级的用高优先级，没有的就用低优先级

spring.config.location可以改变默认的配置文件位置

项目打包后，使用命令行参数，启动项目时指定配置文件的新位置，指定的配置文件和默认加载的配置文件共同起作用，形成互补配置

java -jar xxxxx.jar --spring.config.location=xxx路径



###### 外部配置加载顺序

springboot也可以从以下位置加载配置（优先级从高到低）

互补配置

**1.命令行参数**

e.g  java -jar xxxxx.jar --server.port=xxx --xxx

多个配置空格分开

【优先加载带profile】

**2.jar包外部application-(profile).properties或application.yml(带spring.profile)配置文件**

**3.jar包内部application-(profile).properties或application.yml(带spring.profile)配置文件**

【再加载不带profile】

**4.jar包外部application.properties或application.yml(不带spring.profile)配置文件**

**5.jar包内部application.properties或application.yml(不带spring.profile)配置文件**



###### 自动配置原理

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/1.5.22.RELEASE/reference/html/common-application-properties.html)

自动配置原理：

1、springboot启动时加载主配置类，开启自动配置功能@EnableAutoConfiguration

2、@EnableAutoConfiguration：

​		利用AutoConfigurationImportSelector给容器中导入一些组件

​		查看selectImports()方法的内容：

```java
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
//获取候选的配置
```

- ```java
  SpringFactoriesLoader.loadFactoryNames() 
  ```

    扫苗所有jar包类路径下的META-INF/spring.factories
    把扫描到的这些文件的内容包装成properties对象
    EnableAutoConfiguration.class 从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中
将类路径下META-INF/spring.factories里面配置的所有EnableAutoConfiguration的值加入到容器中

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudServiceConnectorsAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
。。。
```

每一个xxxAutoConfiguration类都是容器中的一个组件都加入到容器中，用来做自动配置

3、每一个自动配置类进行自动配置功能

4、以HttpEncodingAutoConfiguration为例解释自动配置

```java
@Configuration  //表示这是一个配置类 类似配置文件 可以给容器中添加组件
@EnableConfigurationProperties({HttpProperties.class})  //启用指定类ConfigurationProperties功能，将配置文件中对应的值和HttpProperties绑定
@ConditionalOnWebApplication(type = Type.SERVLET)
//spring底层有@Conditional注解，作用：根据不同条件，如果满足指定条件整个配置类中的配置才会生效。该注解判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnClass({CharacterEncodingFilter.class})
//判断当前项目有没有这个类。CharacterEncodingFilter是springmvc中进行解决乱码的过滤器
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true
)
//判断配置文件中是否存在某个配置。存在spring.http.encoding.enabled配置，如果不存在，认为该判断也是成立的 -----即使配置文件中不配置enabled也是默认生效的
public class HttpEncodingAutoConfiguration {

    @Bean //给容器中添加一个组件 filter 这个组件的某些值需要从properties中获取
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.autoconfigure.http.HttpProperties.Encoding.Type.RESPONSE));
        return filter;
    }

```

根据当前不同的条件判断，决定这个配置类是否生效

一旦这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从properties中获取的，这些类里面的每一个属性又是和配置文件绑定的

5、所有在配置文件中能配置的属性都是在xxxConfigurationProperties类中封装

配置文件能配置什么就可以参照某个功能对应的属性类

```java
@ConfigurationProperties(prefix = "spring.http")  //从配置文件中获取指定的值和bean的属性绑定
public class HttpProperties {
```





**springboot：**

​	**1.启动加载大量的自动配置类**

​	**2.我们需要的功能有没有springboot默认写好的自动配置类**

​	**3.这个自动配置类中到底配置了哪些组件，只要有需要的组件 就不需要配置了**

​	**4.给容器中自动配置类添加组件的时候，会从properties类中获取某些属性，于是就可以在配置文件中指定这些属性的值。**

 xxxAutoConfiguration：自动配置类

给容器中添加组件

xxxProperties：封装配置文件中相关属性



###### 自动配置原理细节

1.@Conditional派生注解

作用：必须是@Conditional指定的条件成立，才能给容器添加组件，配置里的内容才生效



自动配置类必须在一定的条件下生效，如何知道哪些自动配置类生效了？

通过在配置文件中写：debug=true 【开启springboot的debug模式】，控制台打印自动配置报告，可以知道哪些自动配置类生效

e.g  Positive matches:....【自动配置类启用的】

​		Negative matches....【没有自动，没有匹配成功的自动配置类】 



### 日志

| 日志门面（日志的抽象层）   | 日志实现                    |
| -------------------------- | --------------------------- |
| JCL  SLF4j   Jboss-Logging | Log4j  JUL  Log4j2  Logback |

左边选一个门面，右边选一个实现

日志门面：SFL4j

日志实现：Logback



Springboot：底层是spring框架，spring选用JCL，springboot选用SFL4j和Logback



#### SLF4j的使用

**如何在系统中使用SLF4j**

开发的时候，，日志记录方法的调用，不应该直接调用日志的实现类，而是调用日志抽象层里面的方法

给系统中导入SLF4j 和 logback的jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

![](.\img\legacy.png)

每一个日志的实现框架都有自己的配置文件。使用SLF4j以后，配置文件还是做成日志实现框架自己本身的配置文件

**遗留问题**

a系统（SLF4j+logback），还包含spring hibernate等使用其他日志框架的框架

统一日志记录，即使是其它框架也一起使用SLF4j进行输出

![](.\img\legacy.png)

<u>如何让系统中所有的日志都统一到SLF4j：</u>

1.将系统中其它日志框架先排除，还是排除那某些框架无法运行

2.用中间包替换原有的日志框架

3.再导入SLF4j其他的实现



#### Springboot日志关系

springboot使用spring-boot-starter-logging来做日志功能

1. springboot底层也是使用SLF4j+Logback的方式进行日志记录
2. springboot也把其它日志都替换成了SLF4j
3. 中间替换包

![](.\img\TIM图片20190906163252.png)

4.如果要引入其它框架，一定要把这个框架的默认日志依赖移除

 springboot能自动适配所有的日志，底层使用slf4j+logback的方式记录日志，引入其它框架就要把这个框架以来的日志框架排除



#### 日志使用

1.默认配置

springboot默认帮我们配置好了日志

test：

```java
public class SpringBoot03ApplicationTests {
    //记录器
    Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    public void contextLoads() {
        //日志的级别：由低到高
        //可以调整输出的日志级别：日志就只会在这个级别及以后的高级别生效
        //springboot默认使用info级别
        logger.trace("trace日志");
        logger.debug("debug日志");
        logger.info("info日志");
        logger.warn("warn日志");
        logger.error("error日志");
    }
}

```

application.properties

```properties
logging.level.com.xx=trace;
logging.path=/spring/log、
logging.file=C:/springboot.log
```

logging.path=/spring/log：在当前磁盘的根路径下创建spring文件夹 和里面的log文件夹；使用spring.log作为默认文件

logging.file=C:/springboot.log 当前项目下（默认路径）生成springboot.log,也可以指定路径

path和file都设置了 听从file



### WEB开发

#### 使用SpringBoot

1.创建springboot项目，选中需要的模块

2.springboot已经默认将这些场景配置完成，只需要在配置文件中指定少量配置就可运行

3.编写业务代码



自动配置原理

这个场景配置了什么，能否修改，可修改哪些配置 能否扩展

xxxAutoConfiguration ：给容器中自动配置组件

xxxProperties：配置类封装配置文件的内容



#### springboot对静态资源的映射规则

相关代码：

```
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
```

可以设置和静态资源有关的参数，缓存时间等

```java
 public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }
        }

     @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext) {
            WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
            welcomePageHandlerMapping.setInterceptors(this.getInterceptors());
            return welcomePageHandlerMapping;
        }
```



1.所有"/webjars/**"【webjars：以jar包的方式引入静态资源】下的请求，

都去"classpath:/META-INF/resources/webjars/"找资源

[https://www.webjars.org](https://www.webjars.org/)可以显示许多框架的依赖

![](.\img\TIM图片20190910103401.png)

localhost:8080/webjars/jquery/3.3.1/jquery.js

2.  /** 访问当前项目的任何资源

```
classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
/ [当前项目的根路径]
```

以上都是静态资源文件夹，保存其中，访问时默认会访问这些路径

localhost:port/xx == 去静态资源资源文件夹中找xx

e.g http://localhost:8084/asserts/js/Chart.min.js



3.欢迎页：静态资源文件夹下的所有index.html页面，被“/**”映射

访问localhost:port/  (这样也满足/**)  找index



4.所有的**/favicon.ico都是在静态资源文件下找



#### 模板引擎

JSP、Velocity、Freemarker、**Thymeleaf【springboot推荐】**

用法：

**1.引入thymeleaf**

可去thymeleaf官网设置新版版本

**2.thymeleaf使用&语法**

```java
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    // 只要我们把html页面放在"classpath:/templates/"，thymeleaf就能自动渲染
```

任何都不用配置，只需要把html放在classpath:/templates/下即可

使用：

1.导入thymeleaf 的名称空间

```html
<html lang="en"  xmlns:th="http://www.thymeleaf.org">
```

2.使用thymeleaf语法

th:text 改变当前元素里面的文本内容

th：任意html属性  来替换原属性的值

 表达式：

1. ${...}:

   - 获取对象的属性 调用方法

   - 使用内置的基本对象 

     （#ctx #locale  #request  #response  #session  #servletContext）

     ${#local.countey} 区域信息的国家代号

   - 使用内置的工具对象 （#message #number #string 。。。）

​	2.*{...} 【选择表达式 使用上和${...}效果是一样的】

 配合 th:object="${session.user}">使用

```xml
 <div th:object="${session.user}">    
 	<p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>    
 	<p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>    
 <div>
```
等价
```xml
<div>  
<p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p> <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p> 
</div>
```

3.#{...} 获取国际化内容

4.@{...} 定义url  th:href=@{...}

@{/order/process(execId=${execId},execType='FAST')}  key=${value}

5.~{...}  片段引用表达式



th：text  不会识别标签

th：utext 识别标签



#### SpringMVC自动配置

**①Spring Boot 自动配置springmvc**

默认配置：

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  自动配置<u>ViewResolver</u>【视图解析器是 根据方法的返回值得到视图对象view，视图对象决定转发/重定向】

  ContentNegotiatingViewResolver:组合所有的视图解析器

  如何定制：给容器中添加一个视图解析器；自动将其组合进来

- Support for serving static resources, including support for WebJars (see below).

  静态资源文件夹路径 webjars

- Automatic registration of `Converter`, `GenericConverter`, `Formatter` beans.

  自动注册了converter这些组件

  `Converter`：转换器，类型转换（如文本→int）

  `Formatter` :格式化器 如2019-9-1转换成日期类型

  **自己添加的格式化器和转换器，只需要放在容器中即可**

- Support for `HttpMessageConverters` (see below).

  `HttpMessageConverters` ：springmvc用来转换HTTP请求和响应的；

  比如一个方法返回了User类型的对象，以json方式写出去

  `HttpMessageConverters` 是从容器中确定的，获取所有的HttpMessageConverter**（@Bean @Component）**

  **自己给容器中添加HttpMessageConverters，只需要将自己的组件注册在容器中**

- Automatic registration of `MessageCodesResolver` (see below).

  定义错误代码生成规则

- Static `index.html` support.

  静态首页访问

- Custom `Favicon` support (see below).

- Automatic use of a `ConfigurableWebBindingInitializer` bean (see below).

  **可配置一个`ConfigurableWebBindingInitializer` 来替换默认的，需要添加到容器中**

  - 作用：初始化WebDateBinder，把请求数据绑定到javabean中

**org\springframework\boot\autoconfigure\web：web的所有自动场景**

If you want to keep Spring Boot MVC features, and you just want to add additional [MVC configuration](https://docs.spring.io/spring/docs/4.3.25.RELEASE/spring-framework-reference/htmlsingle#mvc) (interceptors, formatters, view controllers etc.) you can add your own `@Configuration` class of type `WebMvcConfigurerAdapter`, but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter` or `ExceptionHandlerExceptionResolver` you can declare a `WebMvcRegistrationsAdapter` instance providing such components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.



**②扩展SpringMvc**

**编写一个配置类使用`@Configuration`，类型为`WebMvcConfigurerAdapter`，并且不能使用@`EnableWebMvc`**

既保留了所有的自动配置，也能用我们扩展的配置

```java
//使用WebMvcConfigurerAdapter可以来扩展springmvc的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //super.addViewControllers(registry);
        //浏览器发送/xx 请求来到success页面
        registry.addViewController("/xx").setViewName("success");
    }
}
```

原理：

1.WebMvcConfigureration是SpringMVC的自动配置类

2.在做其他自动配置时会导入：@import（EnableWebConfiguration.class）

3.容器中所有的WebMvcConfigurer都会一起起作用

4.我们的配置类也会被调用

效果:

springmvc的自动配置和我们的扩展配置都会起作用

**③全面接管springmvc**

不用springboot对springmvc的自动配置，所有都是自己配置，所有的springmvc自动配置都失效了

只需要@EnableWebMvc添加在配置类中

#### 如何修改Springboot的默认配置

模式：

1.springboot在自动配置很多组件的时候，先看有没有用户自己配置的@Bean@Component，如果没有就使用自动配置；如果有些组件可以有多个（如ViewResolver），将用户自己配置的和默认配置结合起来

2.springboot会有很多的xxxconfigurer来进行扩展配置

#### RestfulCRUD

**1.默认访问首页**

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("templates/login.html");
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/asserts/**").addResourceLocations("classpath:/asserts/");
        registry.addResourceHandler("/templates/**").addResourceLocations("classpath:/templates/");
    }
}
```

**2.国际化**

<u>1.编写国际化配置文件</u>

2.使用ResourceBundleMessageSource管理国际化资源文件

3.在页面使用fmt:message取出国际化内容 

步骤：

1.编写国际化配置文件，抽取页面需要显示的国际化消息

![](.\img\[}UQ16K5CN_C@W~NI31EK%E.png)

2.springboot自动配置好了管理国际化资源文件的组件

```java
    @Bean
    public MessageSource messageSource(MessageSourceProperties properties) {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        if (StringUtils.hasText(properties.getBasename())) {
//设置国际化资源文件的基础名（去掉国家 语言代码）            messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(StringUtils.trimAllWhitespace(properties.getBasename())));
        }

        if (properties.getEncoding() != null) {
            messageSource.setDefaultEncoding(properties.getEncoding().name());
        }

        messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
        Duration cacheDuration = properties.getCacheDuration();
        if (cacheDuration != null) {
            messageSource.setCacheMillis(cacheDuration.toMillis());
        }

        messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
        messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
        return messageSource;
    }

```

配置文件可以直接放在类路径下叫message.properties

```
spring.messages.basename=i18n.login
```

3.在页面获取国际化的值



**3.登陆**

模板引擎页面修改之后，实时生效：

①禁用模板引擎缓存 spring.thymeleaf.cache=false

②页面修改完成之后 <u>ctrl+f9</u> ，页面会重新编译



登陆错误消息的显示：

```html
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```



拦截器进行登陆检查：

```java
public class LoginHandlerIntercept implements HandlerInterceptor {

    //目标方法执行之前 执行preHandle() 进行预检查
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
       Object user = request.getSession().getAttribute("loginUser");
        if(user == null){
            //未登录 返回登陆页面
            request.setAttribute("msg","请先登录");
            request.getRequestDispatcher("/").forward(request,response);//获取到转发器
            return false;
        }else{
            return true;
        }
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}

```

```java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginHandlerIntercept()).addPathPatterns("/**")//拦截任意多层路径下的多层请求
                                .excludePathPatterns("/login.html","/","/user/login","/asserts/**","/templates/**");
    }
```

**CRUD-员工列表**

RestfulCRUD：满足Rest风格的CRUD

URI：/资源名称/资源标识  HTTP请求方式分别对资源CRUD操作

|      | 普通CRUD（uri区分操作） | RestfulCRUD          |
| ---- | ----------------------- | -------------------- |
| 查询 | getEmp                  | emp【以GET方式发送】 |
| 添加 | addEmp？xxx             | emp【以POST】        |
| 修改 | updateEmp？id=xxx       | emp/{id}【PUT】      |
| 删除 | deleteEmp？id=2         | emp/{id}【delete】   |

请求架构：

|                                      | 请求URI  | 请求方式 |
| ------------------------------------ | -------- | -------- |
| 查询所有员工                         | emps     | GET      |
| 查询某个员工（来到修改页面）         | emp/{id} | GET      |
| 来到添加页面                         | emp      | GET      |
| 添加员工                             | emp      | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/{id} | GET      |
| 修改员工                             | emp      | PUT      |
| 删除员工                             | emp/{id} | DELETE   |

员工列表：

thymeleaf公共页面元素抽取

1.抽取公共片段

```html
<div th:fragment="copy">      
    &copy; 2011 The Good Thymes Virtual Grocery    
</div> 
```

2.引入公共片段

```html
 <div th:insert="~{footer :: copy}"></div> 
~{}可以去掉
```

~{templatename::selector} 模板名：：选择器

 ~{templatename::fragmentname} 模板名：：片段名

3.默认效果

insert的功能片段在div标签中



三种引入功能片段的th属性

th:insert：将公共片段整个插入指定元素中

th:replace：将声明引入的元素替换为公共元素

th:include：将被引入的片段的内容包含进标签中

```html
<footer th:fragment="copy">  &copy; 2011 The Good Thymes Virtual Grocery </footer>

<body>
  ...
  <div th:insert="footer :: copy"></div>
  <div th:replace="footer :: copy"></div>
  <div th:include="footer :: copy"></div>  
</body>
```

效果：

```html
 <div>    
     <footer>      
         &copy; 2011 The Good Thymes Virtual Grocery    
     </footer>  
</div>

  <footer>    
      &copy; 2011 The Good Thymes Virtual Grocery  
</footer>

  <div>    &copy; 2011 The Good Thymes Virtual Grocery  </div> 
```





#### SpringBoot错误处理机制

浏览器返回默认错误页面：

【F12后，可在Header中看到类型text/html】

Whitelabel Error Page

其他客户端 相应一个json数据：

```
{
    "timestamp": "2019-09-18T01:44:16.233+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/crud/xxxxxx"
}
```

原理：

可以参照ErrorMvcAutoConfiguration；错误处理的自动配置

一旦出现4xx或者5xx错误，ErrorPageCustomizer就会生效（定制错误的响应规则），来到/error请求，BasicErrorController处理/error请求，BasicErrorController有两种请求（一种产生html类型数据，一种json数据）

 

**定制错误响应**

①错误的页面

​		1.有模板引擎的情况下 error/状态码 【将错误页面命名为 错误码.html 放在模板引擎的error文件夹下】 发生此状态码的错误就会来到对应的页面

 		可以使用4xx和5xx作为错误页面的文件名来匹配，有精确的如404就去显示404。

页面能获取的信息：timestamp、status状态码、error状态提示、exception异常对象、message异常消息、errors JSR303数据校验

​		2.没有模板引擎（模板引擎找不到这个错误页面），在静态资源文件夹下

​		3.以上都没有错误页面，就来到默认的页面

②错误的json数据

 		1.自定义异常处理 返回定制json数据

```java

@ControllerAdvice
public class MyExceotionHandler {
    @ResponseBody
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handleException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexists");
        map.put("message",e.getMessage());
        return map;
    }
}
```

这样浏览器和客户端返回的都是json

​		2.转发到/error进行自适应响应效果处理

```java
   @ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码 不传入就默认为200
        /**
         * Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexists");
        map.put("message",e.getMessage());
        //转发到/error
        return "forward:/error";
    }
```

​		3.将我们的定制数据携带出去

出现错误后，来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）

​		1）完全来编写一个ErrorController的实现类【或编写AbstractErrorController的子类】，放在容器中

​		2）页面上能用的数据或json返回能用的数据都是通过

