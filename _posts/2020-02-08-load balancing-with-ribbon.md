---
layout: post
title: Load balancing with Spring Cluod Ribbon
categories: [cloud, spring]
tags: [cloud, spring, eureka, netflix, ribbon, load-balance]
description: Using Spring Cloud Netflix Ribbon to load balance from client.
comments: false
---

# 1. Add api to get information
Continute previous article [Quick guide to Spring Cloud Eureka](/cloud/spring/2020/02/06/quick-guide-to-spring-eureka.html).

Add some code in Client Eureka:
{% highlight java %}
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
{% endhighlight %}

Run twice Eureka Client.

# 2. Ribbon Client without Eureka
## 2.1. Dependency
{% highlight xml %}
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
{% endhighlight %}

## 2.2. Configure
In `application.yml` file:
{% highlight yaml %}
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
{% endhighlight %}

Assume 2 Eureka Servers run at 8081 and 8082 port.

Create new config class with name `RibbonConfiguration`:
{% highlight java %}
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
{% endhighlight %}

Add a api to demo load balancing
{% highlight java %}
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

    @RequestMapping("/server-location/{service-name}")
    public Object serverLocation(@PathVariable("service-name") String serviceName) {
        return this.restTemplate.getForObject(
                "http://" + serviceName + "/info", Object.class);
    }

}
{% endhighlight %}

Testing add <a href="http://localhost:8083/server-location/spring-cloud-eureka-client-1" target="_blank">http://localhost:8083/server-location/spring-cloud-eureka-client-1</a>