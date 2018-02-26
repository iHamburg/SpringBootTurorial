# 【Spring Boot】02 - 用Spring Boot创建Java应用



本教程的目的是用Spring Boot创建一个最简单的Java应用。这个教程可以是我们学习使用Spring Boot的第一步



## 本教程实现了什么

完成本教程后，我们就能用Spring Boot搭建一个最简单的Web应用，并且能够访问。



## 本教程的准备

- 大约15分钟时间
- JDK 1.8及以上
- Maven 3.0+
- IDE： STS（Spring Tool Suite）



## 建立一个Java项目 [gs-spring-boot]

一个最简单的Java项目只需要一个文件夹， 用`mkdir -p src/main/java/hello`就能创建

```
└── src
    └── main
        └── java
            └── hello
```

在 `src/main/java/hello`文件夹中，你可以创建Java类文件。

## 加入Maven配置

首先在`src`文件夹同级新建文件 `pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-spring-boot</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

Spring Boot Maven plugin 实现了：

- 把classpath上所有的jar包打成了一个jar包，这样方便部署和执行
- 会自动搜索``public static void main()` 方法，并标识成一个可执行的入口



## 创建简单的Web应用

现在让我们来创建一个view controller吧

`src/main/java/hello/HelloController.java`

```
package hello;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
public class HelloController {

    @RequestMapping("/")
    public String index() {
        return "Greetings from Spring Boot!";
    }

}
```

HelloController用@RestController来标识，表明会用Spring MVC来处理网络请求。 @RequestMapping负责路由。



## 创建应用路口

现在需要为Web应用创建一个入口类

`src/main/java/hello/Application.java`



```
package hello;

import java.util.Arrays;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
        return args -> {

            System.out.println("Let's inspect the beans provided by Spring Boot:");

            String[] beanNames = ctx.getBeanDefinitionNames();
            Arrays.sort(beanNames);
            for (String beanName : beanNames) {
                System.out.println(beanName);
            }

        };
    }

}
```



## 运行Web应用

用Maven来构建并运行

```
mvn package && java -jar target/gs-spring-boot-0.1.0.jar
```



查看Web应用结果

~~~
➜  02-用SpringBoot创建应用 git:(master) ✗ curl localhost:8080
Greetings from Spring Boot!%

~~~

或者可以直接用浏览器访问 `http://localhost:8080`



## 添加单元测试

我们可以为我们的Web应用添加单元测试

`pom.xml`中添加依赖

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
```

### MVC测试

src/test/java/hello/HelloControllerTest.java`

```
package hello;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string(equalTo("Greetings from Spring Boot!")));
    }
}
```



### 集成测试

也能mock HTTP请求来进行一个简单的full-stack的集成测试

`src/test/java/hello/HelloControllerIT.java`

```
package hello;

import static org.hamcrest.Matchers.equalTo;
import static org.junit.Assert.assertThat;

import java.net.URL;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.embedded.LocalServerPort;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloControllerIT {

    @LocalServerPort
    private int port;

    private URL base;

    @Autowired
    private TestRestTemplate template;

    @Before
    public void setUp() throws Exception {
        this.base = new URL("http://localhost:" + port + "/");
    }

    @Test
    public void getHello() throws Exception {
        ResponseEntity<String> response = template.getForEntity(base.toString(),
                String.class);
        assertThat(response.getBody(), equalTo("Greetings from Spring Boot!"));
    }
}
```



## 添加生产环境服务

要在生产环境使用Web应用服务的话，我们还会需要一些管理服务来进行监控。 Spring Boot 提供了一些例如 [actuator module](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#production-ready)等服务

在 `pom.xml`中添加依赖

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

再次编译执行应用

```
mvn package && java -jar target/gs-spring-boot-0.1.0.jar
```

在启动信息中就能看到以下信息

```
2014-06-03 13:23:28.119  ... : Mapped "{[/error],methods=[],params=[],headers=[],consumes...
2014-06-03 13:23:28.119  ... : Mapped "{[/error],methods=[],params=[],headers=[],consumes...
2014-06-03 13:23:28.136  ... : Mapped URL path [/**] onto handler of type [class org.spri...
2014-06-03 13:23:28.136  ... : Mapped URL path [/webjars/**] onto handler of type [class ...
2014-06-03 13:23:28.440  ... : Mapped "{[/info],methods=[GET],params=[],headers=[],consum...
2014-06-03 13:23:28.441  ... : Mapped "{[/autoconfig],methods=[GET],params=[],headers=[],...
2014-06-03 13:23:28.441  ... : Mapped "{[/mappings],methods=[GET],params=[],headers=[],co...
2014-06-03 13:23:28.442  ... : Mapped "{[/trace],methods=[GET],params=[],headers=[],consu...
2014-06-03 13:23:28.442  ... : Mapped "{[/env/{name:.*}],methods=[GET],params=[],headers=...
2014-06-03 13:23:28.442  ... : Mapped "{[/env],methods=[GET],params=[],headers=[],consume...
2014-06-03 13:23:28.443  ... : Mapped "{[/configprops],methods=[GET],params=[],headers=[]...
2014-06-03 13:23:28.443  ... : Mapped "{[/metrics/{name:.*}],methods=[GET],params=[],head...
2014-06-03 13:23:28.443  ... : Mapped "{[/metrics],methods=[GET],params=[],headers=[],con...
2014-06-03 13:23:28.444  ... : Mapped "{[/health],methods=[GET],params=[],headers=[],cons...
2014-06-03 13:23:28.444  ... : Mapped "{[/dump],methods=[GET],params=[],headers=[],consum...
2014-06-03 13:23:28.445  ... : Mapped "{[/beans],methods=[GET],params=[],headers=[],consu...
```



### 查看结果

我们能用直接访问 `http://localhost:8080/info`的方式来访问

```
$ curl localhost:8080/health
{"status":"UP","diskSpace":{"status":"UP","total":397635555328,"free":328389529600,"threshold":10485760}}}
```

errors, environment, health, beans, info, metrics, trace, configprops和 dump

也能直接关闭服务

```
$ curl -X POST localhost:8080/shutdown
{"timestamp":1401820343710,"error":"Method Not Allowed","status":405,"message":"Request method 'POST' not supported"}
```



## 总结

以上我们就用Spring Boot完成了一个最简单的Web应用。



## 参考资料

- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)

