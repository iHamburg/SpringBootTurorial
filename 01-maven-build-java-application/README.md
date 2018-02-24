

#用Maven来编译Java项目

本教程的目的是用Maven来创建一个最简单的Java项目



## 教程实现了什么

你会用Maven创建一个Java应用，能显示当前的日期时间



## 教程的准备

- 大约15分钟时间
- JDK6以上




##安装Maven

可以直接下载binary包

```
https://maven.apache.org/download.cgi
```

macOS 也可用 homebrew下载

```
brew install maven
```

测试是否安装成功

```
` mvn -v

Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
Maven home: /usr/local/Cellar/maven/3.5.2/libexec
```



## 建立一个Java项目

一个最简单的Java项目只需要一个文件夹， 用`mkdir -p src/main/java/hello`就能创建

```
└── src
    └── main
        └── java
            └── hello
```

在 `src/main/java/hello`文件夹中，你可以创建Java类文件。

我们就先创建两个类： `HelloWorld.java`和`Greeter.java`

`src/main/java/hello/HelloWorld.java`

```
package hello;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        System.out.println(greeter.sayHello());
    }
}
```

`src/main/java/hello/Greeter.java`

```
package hello;

public class Greeter {
    public String sayHello() {
        return "Hello world!";
    }
}
```

以上我们已经创建好了一个最简单的Java项目，接下去我们就能用maven来编译了。



## 加入Maven配置

首先在`src`文件夹同级新建文件 `pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

以上我们就建立好了一个可以编译的项目

## 编译Java代码



```
mvn compile
```

执行完compile后，maven会生成target/classes文件夹和class文件

```
mvn package
```

maven会在target文件夹下生成可执行的jar文件， 文件名由<artifactId>和<version>拼接而成： *gs-maven-0.1.0.jar*

如果你想把这个jar包让其他项目引入，执行

```
mvn install
```

maven会将jar包复制到*.m2/repository* 中去



执行Jar文件

```
java -jar target/gs-maven-0.1.0.jar
```



### 重新编译并执行

在修改了代码之后，需要重新编译并执行

```
mvn package && java -jar target/gs-maven-0.1.0.jar
```



## 依赖其他模块

上面的这个项目没有引入任何第三方包，如果你想引入第三方的包的话，也可以在`pom.xml`中进行配置。

下面我们在`HelloWorld.java`中引入`Joda Time`包来打印时间

``src/main/java/hello/HelloWorld.java``

```
package hello;

import org.joda.time.LocalTime;

public class HelloWorld {
	public static void main(String[] args) {
		LocalTime currentTime = new LocalTime();
		System.out.println("The current local time is: " + currentTime);
		Greeter greeter = new Greeter();
		System.out.println(greeter.sayHello());
	}
}
```

在重新编译运行时会出错，因为系统不认识org.joda.time.LocalTime，所以需要在pom.xml 中添加依赖

```
<dependencies>
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.2</version>
		</dependency>
</dependencies>
```

这时我们再编译执行就没有问题了

## 总结

恭喜！ 我们已经成功的创建了一个Java项目，并用Maven编译完成了。



这是我们SpringBoot教学系列的第一篇文章，相信即使没有Java编程基础的同学也能迅速上手开始我们的第一个Java项目。



## 参考资料

- [Building Java Projects with Maven](https://spring.io/guides/gs/maven/)









