---
author: ["Tony Nguyen"]
title: Quick guide to Spring Cloud Config Server
date: "2022-02-04"
description: Using Config server to distribute configurations.
summary: Using Config server to distribute configurations.
categories: ["spring-cloud", "spring"]
tags: ["cloud", "spring", "spring-cloud"]
ShowToc: true
TocOpen: true

cover:
  image: "images/spring-cloud.png"
  alt: "Spring Cloud"
  caption: "Spring Cloud"
  relative: true
---

# 1. Configuration server
In distrubuted system, every service has particular config like yml, json,... Configuration server used for externalizing configuration và centralizing configuration.

![Config server](/images/config-server-architecture.png)

# 2. Spring Cloud Config Server
## 2.1. Dependency
```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
  </dependency>
</dependencies>
```

## 2.2. Enabling config server
Add `@EnableConfigServer` annotation in entrypoint class
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
  public static void main(String[] args) {
    SpringApplication.run(ConfigServer.class, args);
  }
}
```

Configure port to start Config server in `application.yml` file
```yml
server:
  port: 8888
```

## 2.3. Configure repository to store config files
In `application.yml` file
```yml
server.port: 8888
# fetch config from git
spring:
  profiles.active: git

# git repo
  cloud.config.server:
    git:
      uri: https://github.com/john/configs.git 
#     username: username    # if private repo
#     password: password    # if private repo
#     search-paths: foo,bar* # search file in folders /foo or /bar*
      timeout: 10
```

# 2.4. How to use Config server?
Serving at `http://localhost:8888` with rule api:
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
Where:
* `{application}`: service name configurated in Config client (e.g `bootstrap.yml` file) with `spring.application.name` property
* `{profile}`: profile name used in Config client (e.g `bootstrap.yml` file) with `spring.profiles.active` property. `default` if it is empty
* `{lable}` (optinal): git label. default is `master`


# 3. Spring Cloud Config Client
## 3.1. Dependency
```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>${spring-cloud.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
</dependencies>
```

## 3.2. Connect to Config Server
In `bootstrap.yml` file
```yml
spring:
  application:
    name: client-service
  profiles:
    active: default
  cloud:
    config:
      uri: http://localhost:8888
```

With this config, service will get to `http://localhost:8888/client-service/default` then Config Server will look at `https://github.com/john/configs.git` with file name `client-service.yml`.


