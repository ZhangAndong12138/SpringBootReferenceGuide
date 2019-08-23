---
title: SpringBoot参考指南-14.代码结构
date: 2019-08-22 07:03:21
tags: SpringBoot
---
# 14. 代码结构

`SpringBoot`并不需要特殊的代码结构。不过，遵循这些惯例可以帮助到你。

## 14.1 “default” 包的使用

当一个类没有`package`声明的时候，它就会被放到“default package”里面。通常，我们不鼓励，也应该避免使用“default package”。它会导致`SpringBoot`应用在使用`@ComponentScan`，`@EntityScan`或`@SpringBootApplication`注解时产生问题，因为每个jar包中的每个类都会被读到。

> **Tip**
> 
> 我们建议你遵循Java推荐的包命名约定，并且使用反转领域名称（比如，`com.example.project`）。

## 14.2 定位应用的Main Class

我们通常建议你把应用的Main Class放在其它class最外层的包里。`@SpringBootApplication`注解通常放在你的Main Class里，它隐式的定义了`SpringBoot`在你项目里要从哪个包开始往下扫描。举个例子，如果你正在写一个`JPA`应用，有`SpringBootApplication`注解的类所在的包下面所有的`@Entity`注解的项都会被扫描到。把它放在包的最外层也可以让组件只扫描你的项目。

> **Tip**
>
>如果你不想用`@SpringBootApplication`注解，你可以用` @EnableAutoConfiguration`和`@ComponentScan`来代替，它们也可以用来定义扫描的位置。

典型的项目结构应该像这样：
```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

在`Application.java`这个文件里，应该声明`main`方法，并且加上`@SpringBootApplication`注解，像下面这样：

```java
package com.example.myapplication;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
 public static void main(String[] args) {
  SpringApplication.run(Application.class, args);
 }
}
```
