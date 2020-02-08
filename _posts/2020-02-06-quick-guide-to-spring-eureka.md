---
layout: post
title: Quick guide to Spring Cloud Eureka
categories: [cloud, spring]
tags: [cloud, spring, eureka, netflix, registration, discovery]
description: Using Eureka server to manage, monitoring, discover, registry.
comments: false
---

# 1. Registeration & Discovery server 
# 2. Spring Cloud Netflix Eureka Server
## 2.1. Dependency
{% highlight xml %}
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>

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
{% endhighlight %}

# 2.2. Configure
Disable self registry in `application.yml` file:
{% highlight yaml %}
server:
  port: 8761

eureka.client:
  fetch-registry: false
  register-with-eureka: false

logging.level.com.netflix:
  eureka: OFF
  discovery: OFF
{% endhighlight %}

Add `@EnableEurekaServer` annotation in entrypoint class.

# 3. Spring Cloud Netflix Eureka Client 
## 3.1. Dependency
{% highlight xml %}
<dependencies>
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>

  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>

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
{% endhighlight %}

## 3.2. Configure
In `bootstrap.xml` file:
{% highlight yaml %}
spring:
  application:
    name: spring-cloud-eureka-client-${ID_NAME:1}
{% endhighlight %}
Variable `ID_NAME` for demonstrate

{% highlight yaml %}
server:
  port: 0
eureka:
  client:
    serviceUrl:
      defaultZone: ${EUREKA_URI:http://localhost:8761/eureka}
  instance:
    preferIpAddress: true
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
{% endhighlight %}
Value `server.port` is 0 to radom port.

## 3.3. Run Eureka Client
Open 3 prompts at this project, run twice `mvnw spring-boot:run` and once `mvnw spring-boot:run -Dspring-boot.run.arguments=--ID_NAME=2`

Result
![Eureka service with 3 clients](/assets/media/eureka-server.png)