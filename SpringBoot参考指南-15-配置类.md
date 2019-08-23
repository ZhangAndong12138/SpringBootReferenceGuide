---
title: SpringBoot参考指南-15.配置类
date: 2019-08-22 07:44:17
tags: SpringBoot
---
# 15. 配置类

`SpringBoot`更喜欢基于`Java`的配置。虽然用`XML`也能配置`SpringApplication`，但是我们还是推荐你主要的配置文件是个加了`@Configuration`的类。通常，定义了`main`方法的那个类用来当做主要的配置类就挺好。

> **Tip**
>
> 网上的许多`Spring`配置的例子都是用的`XML`。如果可以的话，还是用等效的基于`Java`的配置更好。搜索一下` Enable*`这个注解会是一个好的开始。

## 15.1 引入附加的配置类

你并不需要把全部的`@Configuration`配置都放在一个类里。`@Import`注解可以用来引入附加的配置类。作为另一种选择，你也可以用`@ComponentScan`注解来自动选择所有的`Spring`组件，包括`@Configuration`配置类。

## 15.2 引入XML配置

如果你一定必须要用`XML`配置的话，我们建议你仍然从一个`@Configuration`配置类开始，接下来你可以用`@ImportResource`注解来加载`XML`配置文件。
