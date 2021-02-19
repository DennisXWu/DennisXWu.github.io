---
title: SpringCloud学习-Config
date: 2021-1-8 23:29:53
categories:
- Spring
tags:
- Spring
---

### 1、什么是SpringCloud Config？

​    Spring Cloud Config项目是一个**解决分布式系统的配置管理方案**。它包含了Client和Server两个部分，server提供配置文件的存储、以接口的形式将配置文件的内容提供出去，client通过接口获取数据、并依据此数据初始化自己的应用。可以将配置放到远程服务器，目前支持**本地存储、Git**以及**Subversion**。 

## 1.1、配置中心要解决的问题

1. 集中管理多个微服务项目的配置文件（查看、修改、通知客户端)
2. 配置文件的分层

- 按照环境分层： Test/Dev/Uat
- 按照业务分层： 内部集成 / 外部基础 / 微信相关

3. 安全性：因为有所有项目的配置，虽然运维管理很方便，但是安全性就很重要（比方说 某个项目只允许某某人修改，谁都能改而且没有修改脚印的话，都没法追溯了 、设置只读账户）

- 权限控制（比如 配置仓库 可以做权限控制 （代码仓库天生的 支持只读模式、修改模式），配置仓库可以设置访问密码 + 用户授权)
- 敏感信息加密（数据库的账号密码 不要使用明文)

4. 易用性：

- 集成和使用要方便简洁：比如Springcloud config可以使用注册中心

## 1.2、Config原理

 SpringCloud Config分服务端和客户端，服务端负责将本地，git或者svn中存储的配置文件发布成REST风格的接口，客户端可以从服务端REST接口获取配置。 **git服务器会从远程git拉取配置文件,并存入到本地git文件库,当远程git不可用时,会从本地git文件库拉取配置信息** 。

![]({{ site.url }}/assets/img/spring/10.1.png)


## 2、如何使用Config？

### 2.1、Server端

 首先在github上面创建了一个文件夹config-repo用来存放配置文件，为了模拟生产环境，我们创建以下三个配置文件： 

```xml
// 开发环境
neo-config-dev.properties
// 测试环境
neo-config-test.properties
// 生产环境
neo-config-pro.properties
```

1、添加依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
</dependencies>
```

 只需要加入spring-cloud-config-server包引用既可。 

2、配置文件

```properties
server:
  port: 8001
spring:
  application:
    name: spring-cloud-config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/ityouknow/spring-cloud-starter/     # 配置git仓库的地址
          search-paths: config-repo                             # git仓库地址下的相对地址，可以配置多个，用,分割。
          username:                                             # git仓库的账号
          password:                                             # git仓库的密码
```

   Spring Cloud Config也提供本地存储配置的方式。我们只需要设置属性`spring.profiles.active=native`，Config Server会默认从应用的`src/main/resource`目录下检索配置文件。也可以通过`spring.cloud.config.server.native.searchLocations=file:E:/properties/`属性来指定配置文件的位置。虽然Spring Cloud Config提供了这样的功能，但是为了支持更好的管理内容和版本控制的功能，还是推荐使用git的方式。 

3、启动类

 启动类添加`@EnableConfigServer`，激活对配置中心的支持 

```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

4、测试

首先我们先要测试server端是否可以读取到github上面的配置信息，直接访问：`http://localhost:8001/neo-config/dev`

返回信息如下：

```properties
{
    "name": "neo-config", 
    "profiles": [
        "dev"
    ], 
    "label": null, 
    "version": null, 
    "state": null, 
    "propertySources": [
        {
            "name": "https://github.com/ityouknow/spring-cloud-starter/config-repo/neo-config-dev.properties", 
            "source": {
                "neo.hello": "hello im dev"
            }
        }
    ]
}
```

上述的返回的信息包含了配置文件的位置、版本、配置文件的名称以及配置文件中的具体内容，说明server端已经成功获取了git仓库的配置信息。

如果直接查看配置文件中的配置信息可访问：`http://localhost:8001/neo-config-dev.properties`，返回：`neo.hello: hello im dev`

修改配置文件`neo-config-dev.properties`中配置信息为：`neo.hello=hello im dev update`,再次在浏览器访问`http://localhost:8001/neo-config-dev.properties`，返回：`neo.hello: hello im dev update`。说明server端会自动读取最新提交的内容

仓库中的配置文件会被转换成web接口，访问可以参照以下的规则：

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

以neo-config-dev.properties为例子，它的application是neo-config，profile是dev。client会根据填写的参数来选择读取对应的配置。

### 2.2、Client端

 主要展示如何在业务项目中去获取server端的配置信息 

1、添加依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
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
```

2、配置文件

需要配置两个配置文件，application.properties和bootstrap.properties

application.properties如下：

```xml
spring.application.name=spring-cloud-config-client
server.port=8002
```

 bootstrap.properties如下： 

```xml
spring.cloud.config.name=neo-config
spring.cloud.config.profile=dev
spring.cloud.config.uri=http://localhost:8001/
spring.cloud.config.label=master
```

- spring.application.name：对应{application}部分
- spring.cloud.config.profile：对应{profile}部分
- spring.cloud.config.label：对应git的分支。如果配置中心使用的是本地存储，则该参数无用
- spring.cloud.config.uri：配置中心的具体地址
- spring.cloud.config.discovery.service-id：指定配置中心的service-id，便于扩展为高可用配置集群。

> 特别注意：上面这些与spring-cloud相关的属性必须配置在bootstrap.properties中，config部分内容才能被正确加载。因为config的相关配置会先于application.properties，而bootstrap.properties的加载也是先于application.properties。

3、启动类

 启动类添加`@EnableConfigServer`，激活对配置中心的支持 

```java
@SpringBootApplication
public class ConfigClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigClientApplication.class, args);
	}
}
```

 启动类只需要`@SpringBootApplication`注解就可以 

4、Web测试

 使用`@Value`注解来获取server端参数的值 

```java
@RestController
class HelloController {
    @Value("${neo.hello}")
    private String hello;

    @RequestMapping("/hello")
    public String from() {
        return this.hello;
    }
}
```

## 3、参考资料

1、 https://www.jianshu.com/p/dbda03946dc7 

2、 http://www.glmapper.com/2019/01/05/springcloud-config-analysis/ 

3、 [http://jm.taobao.org/2018/12/12/Spring-Cloud-Config-%E8%A7%84%E8%8C%83/](http://jm.taobao.org/2018/12/12/Spring-Cloud-Config-规范/) 

4、 http://www.ityouknow.com/springcloud/2017/05/22/springcloud-config-git.html 



