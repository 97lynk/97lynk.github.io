---
author: ["Tony Nguyen"]
title: "The Lombok Mistake That Could Make Your App Slowly"
date: "2025-04-25"
description: "Using Lombok carefully"
summary: "Using Lombok carefully"
categories: ["lombok", "java"]
tags: ["spring", "spring-boot", "java", "lombok"]
series: ["Themes Guide"]
ShowToc: true
TocOpen: true

cover:
  image: "images/cover.png"
  alt: "Project Lombok"
  caption: "Project Lombok"
  relative: true
---

Lombok is a lifesaver when it comes to eliminating boilerplate code. Annotations like `@Getter`, `@Setter`, `@Builder`, `@Data`, `@RequiredArgsConstructor` and `@AllArgsConstructor` help keep your code concise and readable. But if you’re not careful, using these annotations (especially `@RequiredArgsConstructor` and `@AllArgsConstructor`) can cause unexpected and hard-to-diagnose bugs.

Here’s one we ran into that looked harmless, but nearly derailed a production release.

# Situation
We were working with a database setup that follows a **leader-follower strategy**, where the **leader** handles **write** operations and the **followers** handle **read** operations.

Here our customize annotations:
```java
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface LeaderDb {
}

@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface FollowerDb {
}
```

and then config beans:
```java
@Bean
@Primary
@LeaderDb
public DataSource leaderDataSource(HikariConfig writeConfig) {
    return new HikariDataSource(writeConfig);
}

@Bean
@FollowerDb
public DataSource followerDataSource(HikariConfig readConfig) {
    return new HikariDataSource(readConfig);
}

@Bean
@Primary
@LeaderDb
public JdbcTemplate leaderJdbcTemplate(@LeaderDb DataSource leaderDb) {
    return new JdbcTemplate(leaderDb);
}

@Bean
@FollowerDb
public JdbcTemplate followerJdbcTemplate(@FollowerDb DataSource followerDb) {
    return new JdbcTemplate(followerDb);
}
```

Finally, we injected them into service. Here's what our service looked like:

```java
@AllArgsConstructor
public class ProductService {

    @LeaderDb
    private final JdbcTemplate writeTemplate;

    @FollowerDb
    private final JdbcTemplate readTemplate;
    
    public product getProduct(long productId) {
        return readTemplate.query("SELECT * FROM product....", productId, mapRowProduct);
    }

    public void updateProduct(product dto) {
        writeTemplate.execute("UPDATE product....");
    }
}
```
Everything compiled without issues and no errors were reported during Dev and Staging environment.

# Problem
The problem surfaced after the changes were released. Many end user want to view products that started invoking the `getProduct` method, which led to a spike in IOPS (Input/Output Operations Per Second) on the Leader database.

# What happened?
- Why did the IOPS increase on the Leader database instead of the Follower?
- Any mistake configuration for Leader / Follower?
---

# Debugging the Root Cause

We reached out to the DBA team for support on Dev environment. After enabling SQL query logging on both databases, they confirmed that all `SELECT` statements were being executed on the Leader database. What the heck in our code?

We tried to debug on both method `getProduct` and `updateProduct`, something was strange here, both `readTemplate` and `writeTemplate` showed same object address on IDE. ?? 🙂 ?? What?

We found generated class in target folder to see what was behind the scene. Please pay attention on constructor which generated by Lombok:

```java
public class ProductService {

    private final JdbcTemplate writeTemplate;

    private final JdbcTemplate readTemplate;
    
    public product getProduct(long productId) {
        return readTemplate.query("SELECT * FROM product....", productId, mapRowProduct);
    }

    public void updateProduct(product dto) {
        writeTemplate.execute("UPDATE product....");
    }
    
    public ProductService(JdbcTemplate writeTemplate, JdbcTemplate readTemplate) {
        this.writeTemplate = writeTemplate;
        this.readTemplate = readTemplate;
    }
}
```

Why Lombok removed our customize annotation? We found out somebody who has same case [Missing annotation](https://github.com/projectlombok/lombok/issues/1528), that means Lombok generated constructor without any annotation(`@RequiredArgsConstructor` or `@AllArgsConstructor` for our issue).

# Time to fix
## Solution 1
So simple, create own constructor clearly:

```java
public ProductService(@LeaderDb JdbcTemplate writeTemplate, @Follower JdbcTemplate readTemplate) {
    this.writeTemplate = writeTemplate;
    this.readTemplate = readTemplate;
}
```
However, constructor will be warning when reach 7 parameters by IDE, Sonar or any code quality tool.

## Solution 2
As you can see in [GitHub issue topic](https://github.com/projectlombok/lombok/issues/1528), the proposal solution make a config file `lombok.config` which place in base folder project flowing config:

```lombok.config
lombok.xArgsConstructor.fieldAnnotationTakeOver += com.company.LeaderDb
lombok.xArgsConstructor.fieldAnnotationTakeOver += com.company.FollowerDb
```


Certainly! Here's a concise and professional conclusion you can use in your tech blog:

---

# Conclusion
While Lombok’s `@RequiredArgsConstructor` or ``@AllArgsConstructor` is a powerful and convenient annotation for reducing boilerplate code, it should be used with care—especially when combined with other annotations. Lombok generates constructors at compile time, and if other frameworks rely on constructor behavior at runtime, conflicts or unexpected behavior can arise. Always review the generated code (e.g., using your IDE or Lombok plugin) and test carefully to ensure compatibility.
