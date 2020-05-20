---
title: Keycloak部署以及简单RBAC应用
date: 2020-05-20 11:38:19
tags:
---

企业级应用中，总需要一个统一的登陆。之前是apero/cas，现在有开箱即用的Keycloak

1、部署Keycloak（这里仅介绍standalone部署）

1.1、下载keycloak8.0.2安装包

https://www.keycloak.org/archive/downloads-8.0.2.html

https://downloads.jboss.org/keycloak/8.0.2/keycloak-8.0.2.tar.gz

```

tar -zxf keycloak-8.0.2.tar.gz

```

1.2、上传mariadb jdbc连接包 mariadb-java-client-2.6.0.jar到目录./keycloak-8.0.2/modules/system/layers/base/org/mariadb/main

新建文件./keycloak-8.0.2/modules/system/layers/base/org/mariadb/main/module.xml

```

<?xml version="1.0" encoding="UTF-8"?>
<module name="org.mariadb" xmlns="urn:jboss:module:1.3">
    <resources>
        <resource-root path="mariadb-java-client-2.6.0.jar"/>
    </resources>
    <dependencies>
        <module name="javax.api"/>
        <module name="javax.transaction.api"/>
    </dependencies>
</module>

```

1.3、修改配置文件./keycloak-8.0.2/standalone/configuration/standalone.xml

找到\<datasources\>中的\<drivers\>配置，在其中h2配置之前添加

```

<driver name="mariadb" module="org.mariadb">
    <xa-datasource-class>org.mariadb.jdbc.MariaDbDataSource</xa-datasource-class>
</driver>

```

往上翻一翻\<datasources\>中有\<datasource jndi-name="java:jboss/datasources/KeycloakDS"\>的配置，修改为

```

<datasource jndi-name="java:jboss/datasources/KeycloakDS" pool-name="KeycloakDS" enabled="true" use-java-context="true" statistics-enabled="${wildfly.datasources.statistics-enabled:${wildfly.statistics-enabled:false}}">
    <connection-url>jdbc:mariadb://localhost:3306/keycloak</connection-url>
    <driver>mariadb</driver>
    <security>
        <user-name>$user-name$</user-name>
        <password>$password$</password>
    </security>
</datasource>

```

1.4、登陆mariadb并创建库 keycloak

并检查mariadb配置

```

wait_timeout=10000
interactive_timeout=10000

```

1.5、配置keycloak启动参数./keycloak-8.0.2/bin/standalone.conf

```

#KeyCloak管理控制台绑定IP
JAVA_OPTS="$JAVA_OPTS -Djboss.bind.address.management=0.0.0.0"
#KeyCloak绑定IP
JAVA_OPTS="$JAVA_OPTS -Djboss.bind.address=0.0.0.0"
#KeyCloak绑定端口
JAVA_OPTS="$JAVA_OPTS -Djboss.http.port=8080"
#KeyCloak绑定SSL端口
JAVA_OPTS="$JAVA_OPTS -Djboss.https.port=8443"
#Jboss容器管理控制台绑定端口
JAVA_OPTS="$JAVA_OPTS -Djboss.management.http.port=9990"
#Jboss容器管理控制台绑定SSL端口
JAVA_OPTS="$JAVA_OPTS -Djboss.management.https.port=9993"

```

1.6、启动keycloak

```

./keycloak-8.0.2/bin/standalone.sh 


```

1.7、初始化Keycloak管理用户

```

./bin/add-user-keycloak.sh -u admin

#do next

```

1.8、重启Keycloak

2、使用Keycloak认证（RBAC体系）

2.1、访问keycloak管理控制台

访问keycloak管理控制台：https://{ip:ssl-port}/

使用admin用户名密码进入Administration Console

为了方便，可以在Master realm中修改theme，启用国际化并设置默认语言为zh-CN，然后修改admin用户语言环境为zh-CN

重新登陆admin用户

2.2、创建新域（realm）test，并切换成test域

2.3、添加组（group）实现部门管理

2.4、添加角色（role）实现角色管理

2.5、创建新客户端（client）test-client

2.6、进入test-client

添加客户端下的角色实现细粒度权限管理

添加新的有效重定向URI

比如支持本地调试 http://localhost:8080/*

添加Web-Origin（实现http跨域）: *

2.7、添加用户user

添加用户user的凭据（密码）

角色映射中可以关联角色和客户端角色（角色/权限）

组可以把用户加入某个分组（归属部门）

2.8、SpringBoot程序添加Keycloak进行安全认证和鉴权

pom.xml添加依赖

```

<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-boot-starter</artifactId>
    <version>8.0.2</version>
</dependency>

```

配置Bean

```

@Bean
public KeycloakSpringBootConfigResolver keycloakSpringBootConfigResolver() {
    return new KeycloakSpringBootConfigResolver();
}

```

配置Keycloak安全拦截代理


```

@Configuration
public class SecurityConfiguration extends KeycloakWebSecurityConfigurerAdapter {
    public SecurityConfiguration() {
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        KeycloakAuthenticationProvider keycloakAuthenticationProvider = this.keycloakAuthenticationProvider();
        SimpleAuthorityMapper grantedAuthorityMapper = new SimpleAuthorityMapper();
        grantedAuthorityMapper.setConvertToLowerCase(true);
        grantedAuthorityMapper.setPrefix("");
        keycloakAuthenticationProvider.setGrantedAuthoritiesMapper(grantedAuthorityMapper);
        auth.authenticationProvider(keycloakAuthenticationProvider);
    }

    public SessionAuthenticationStrategy sessionAuthenticationStrategy() {
        return new NullAuthenticatedSessionStrategy();
    }

    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
    }
}

```

配置application.yml

```

keycloak:
  enabled: true
  disable-trust-manager: true
  auth-server-url: https://{ip:ssl-port}/auth
  realm: test
  resource: test-client
  ssl-required: none
  public-client: true
  bearer-only: false
  use-resource-role-mappings: true

```
