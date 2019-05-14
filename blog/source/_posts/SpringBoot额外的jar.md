---
title: SpringBoot额外的jar
date: 2019-05-14 14:20:11
tags:
---

[SpringBoot Doc](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#executable-jar-launching)

[SpringBoot plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/)

``` xml
<!-- pom.xml -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layout>ZIP</layout>
    </configuration>
</plugin>
```

ZIP (alias to DIR): similar to the JAR layout using PropertiesLauncher.
使用 PropertiesLauncher 启动。

经过读代码分析，默认的 Jar layout 使用的 JarLauncher 无法引入外部 jar，只有使用 PropertiesLauncher 才可以通过配置 loader.path 去指定外部 jar 路径引入。

注：PropertiesLauncher 在 SpringBootLoader 中。[github](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-tools/spring-boot-loader/src/main/java/org/springframework/boot/loader/PropertiesLauncher.java)