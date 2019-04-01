---
title: SpringBootAdmin中集成SwaggerApi
date: 2019-04-01 11:45:57
tags:
---
在使用SpringBoot以及SpringCloud搭建微服务时，我们可能会用SpringBootAdmin去监控各个服务。

另外我们还会使用Springfox-Swagger去生成Swagger文档，那么可以把Swagger文档集成到SpringBootAdmin。

首先，SpringBootAdmin监听/api-docs端点，在相应配置中添加/api-docs

```yml
spring:
  boot:
    admin:
	  probed-endpoints: api-docs,health,env,metrics,httptrace:trace,threaddump:dump,jolokia,info,logfile,refresh,flyway,liquibase,heapdump,loggers,auditevents
```

然后，创建一个vuejs项目去扩展SpringBootAdmin功能，如：
[spring-boot-admin-swagger-view](https://github.com/mosence/spring-boot-admin-swagger-view)

maven打成jar包后引入SpringBootAdmin项目或者npm打包后把dist目录文件copy到/reousrces/META-INF/spring-boot-admin-server-ui/extensious/api-docs

接着在各个服务开放/api-docs端点，在SpringBoot 2.X中，访问路径为/actuator/api-docs，如：

```java
@Component
@Endpoint(id = "api-docs")
public class ServletSwaggerApiEndpoint{

    @Resource
    private HttpServletRequest servletRequest;

    private DocumentationCache documentationCache;
    private ServiceModelToSwagger2Mapper mapper;

    public ServletSwaggerApiEndpoint(DocumentationCache documentationCache,
                                      ServiceModelToSwagger2Mapper mapper) {
        this.documentationCache = documentationCache;
        this.mapper = mapper;
    }
	
    @ReadOperation(produces = "application/json")
    public Swagger loadApiDocs() {
        String groupName = Docket.DEFAULT_GROUP_NAME;
        Documentation documentation = documentationCache.documentationByGroup(groupName);
        Swagger swagger = mapper.mapDocumentation(documentation);

        UriComponents uriComponents = componentsFrom(servletRequest, swagger.getBasePath());
        swagger.basePath(Strings.isNullOrEmpty(uriComponents.getPath()) ? "/" : uriComponents.getPath());
        swagger.setHost(uriComponents.getHost()+ ":" +uriComponents.getPort());
        
        return swagger;
    }
}
```

服务被SpringBootAdmin发现后就可以在服务实例中看到Api-Docs啦！