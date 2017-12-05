title: Docker构建SpringBoot镜像
date: 2017-12-05 21:10:55
tags: docker
---
**一、使用STS构建springBoot项目**
项目结构
```
├─src
│  ├─main
│  │  ├─java
│  │  │  └─com
│  │  │      └─example
│  │  └─resources
│  │      ├─static
│  │      └─templates
│  └─test
│      └─java
│          └─com
│              └─example
└─target
```

<!-- more -->

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0</version>
    <packaging>jar</packaging>

    <name>SpringBoot</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

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
Application.java

```
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class Application {

    @RequestMapping("/")
    public String index(){

        return "Hello Docker .";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
使用maven打包项目,在target目录下得到springboot-1.0.jar
```
mvn clean pakcage //先清理再进行打包
```
run项目

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.1.RELEASE)
......
2017-02-22 13:51:45.408  INFO 1588 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-02-22 13:51:45.423  INFO 1588 --- [           main] com.example.Application                  : Started Application in 2.004 seconds (JVM running for 2.622)
2017-02-22 13:51:58.432  INFO 1588 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
2017-02-22 13:51:58.433  INFO 1588 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization started
2017-02-22 13:51:58.452  INFO 1588 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization completed in 19 ms
```
地址栏输入http://localhost:8080/, 得到如下结果,项目构建成功。
```
Hello Docker .
```

**二、构建Docker镜像,启动Docker容器**
上传springboot-1.0.jar到装有Docker服务的linux系统,这里我用的是ubutun14.04
![](http://img.blog.csdn.net/20170222142213286?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ2l0aHViXzM3NjAwMjU1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

开始编写Dockerfile文件
```
vi Dockerfile
```
Dockerfile的内容，保存退出

```
FROM hub.c.163.com/xbingo/jdk8
ADD ./springboot-1.0.jar  /springboot.jar
EXPOSE 8080
CMD ["java","-jar","/springboot.jar"]
```
ll查看
```
root@ubuntu:/data/test# ll
total 14004
drwxr-xr-x 2 root root     4096 Feb 21 22:38 ./
drwxr-xr-x 5 root root     4096 Feb 21 22:20 ../
-rw-r--r-- 1 root root      124 Feb 21 22:25 Dockerfile
-rw-r--r-- 1 root root 14326404 Feb 21 22:20 springboot-1.0.jar
root@ubuntu:/data/test#
```
构建镜像,别忘记后面的”.”,表示Dockerfile的文件位置

```
root@ubuntu:/data/test# docker build -t springboot:1.0 .
Sending build context to Docker daemon 14.33 MB
Step 1/4 : FROM hub.c.163.com/xbingo/jdk8
 ---> 3273714c9663
Step 2/4 : ADD ./springboot-1.0.jar /springboot.jar
 ---> 556a8eba0f6a
Removing intermediate container 8d6a88c466e2
Step 3/4 : EXPOSE 8080
 ---> Running in 327fc70b5fa9
 ---> e5e6c6b29983
Removing intermediate container 327fc70b5fa9
Step 4/4 : CMD java -jar /springboot.jar
 ---> Running in 4a9c41547c8c
 ---> c745ff82ccac
Removing intermediate container 4a9c41547c8c
Successfully built c745ff82ccac
root@ubuntu:/data/test# 
```
查看镜像列表,已生成springboot镜像
```
root@ubuntu:/data/test# docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
springboot                         1.0                 c745ff82ccac        5 minutes ago       182 MB
root@ubuntu:/data/test# 
```
启动容器,并查看运行容器列表
```
root@ubuntu:/data/test# docker run -d -p 8080:8080 --name springboot springboot:1.0
840e8f08bdbbf10f3050c4c7ec38ba7c1ce90378b0f31a3a8cc7a1d2ddf66119
root@ubuntu:/data/test# docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                       NAMES
840e8f08bdbb        springboot:1.0           "java -jar /spring..."   4 seconds ago       Up 3 seconds        0.0.0.0:8080->8080/tcp                      springboot

```
打开宿主主机的浏览器:输入http://localhost:8080,预览效果
![](http://img.blog.csdn.net/20170222144541412?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ2l0aHViXzM3NjAwMjU1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

要想查看Docker容器日志,显示日志结果和第一步的启动日志一致
```
docker logs springboot
```

