---
author: ["Tony Nguyen"]
title: "Load balancing with Spring Cloud Ribbon"
date: "2022-02-08"
description: "Using Spring Cloud Netflix Ribbon to load balance from client."
summary: "Using Spring Cloud Netflix Ribbon to load balance from client."
categories: ["spring-cloud", "spring"]
tags: ["cloud", "spring", "spring-cloud", "eureka", "netflix", "ribbon", "load-balance"]
ShowToc: true
TocOpen: true
---

# 1. Add api to get information
Continue previous article [Quick guide to Spring Cloud Eureka](/cloud/spring/2020/02/06/quick-guide-to-spring-eureka.html).

Add some code in Client Eureka:
```java
@SpringBootApplication
@RestController
public class SpringEurekaClientApplication {

    Logger logger = Logger.getLogger(this.getClass().getSimpleName());

    public static void main(String[] args) {
        SpringApplication.run(SpringEurekaClientApplication.class, args);
    }

    @Value("${spring.application.name}")
    private String appName;

    @Autowired
    private EurekaDiscoveryClient eurekaClient;

    @GetMapping("/instances")
    public List<ServiceInstance> instances() {
        logger.info("GET instances");
        return this.eurekaClient.getInstances(appName);
    }

    @GetMapping("/apps")
    public List<String> list() {
        logger.info("GET apps");
        return this.eurekaClient.getServices();
    }

    @GetMapping("/info")
    public ServiceInstance info() {
        logger.info("GET this instance's information");
        return this.eurekaClient.getInstances(appName).get(this.eurekaClient.getOrder());
    }

}
```

Run twice Eureka Client.

# 2. Ribbon Client without Eureka
## 2.1. Dependency
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
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
```

## 2.2. Configure
In `application.yml` file:
```yml
spring:
  application:
    name: spring-cloud-ribbon

server:
  port: 8083

spring-cloud-eureka-client-config:
  ribbon:
    eureka:
      enabled: false
    listOfServers: localhost:8081,localhost:8082
    ServerListRefreshInterval: 15000
```

Assume 2 Eureka Servers run at 8081 and 8082 port.

Create new config class with name `RibbonConfiguration`:
```java
public class RibbonConfiguration {

    @Autowired
    IClientConfig ribbonClientConfig;
    
    @Bean
    public IPing ribbonPing(IClientConfig config) {
        return new PingUrl();
    }

    @Bean
    public IRule ribbonRule(IClientConfig config) {
        return new WeightedResponseTimeRule();
    }
}
```

Add a api to demo load balancing
```java
@SpringBootApplication
@RestController
@RibbonClient(name = "spring-cloud-eureka-client-config",
        configuration = RibbonConfiguration.class)
public class SpringRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringRibbonApplication.class, args);
    }

    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/info/{service-name}")
    public Object serverLocation(@PathVariable("service-name") String serviceName) {
        return this.restTemplate.getForObject(
                "http://" + serviceName + "/info", Object.class);
    }

}
```

Testing add <a href="http://localhost:8083/info/spring-cloud-eureka-client-1" target="_blank">http://localhost:8083/info/spring-cloud-eureka-client-1</a>

# 3. Ribbon Client with Eureka and Feign
## 3.1. Dependency
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
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
```

## 3.2. Configure
In `application.yml` file:
```yml
spring:
  application:
    name: spring-cloud-ribbon

server:
  port: 8083

eureka:
  client:
    serviceUrl:
      defaultZone: ${EUREKA_URI:http://localhost:8761/eureka}
  instance:
    preferIpAddress: true
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```

Add a api to demo load balancing
```java
@SpringBootApplication
@RestController
@EnableFeignClients
public class SpringRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringRibbonApplication.class, args);
    }

    @Autowired
    private  SpringCloudEurekaClient springCloudEurekaClient;

    @RequestMapping("/info")
    public Object serverLocation() {
        return this.springCloudEurekaClient.getInfo();
    }

    @FeignClient("spring-cloud-eureka-client-1")
    interface SpringCloudEurekaClient {
        @RequestMapping(value = "/info", method = GET)
        Object getInfo();
    }
}
```

Testing add <a href="http://localhost:8083/info" target="_blank">http://localhost:8083/info</a>
