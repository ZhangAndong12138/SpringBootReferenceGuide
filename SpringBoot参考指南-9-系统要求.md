---
title: SpringBoot参考指南-9.系统要求
date: 2019-08-22 03:46:55
tags: SpringBoot
---
# 9. 系统要求

`SpringBoot2.1.7`版本需要`Java8`，并且已经完全兼容`Java12`。同时，还需要`Spring Framework5.1.9`或更新的版本。此外，使用创建工具，需要`Maven3.3`+和`Gradle4.4`+。

## 9.1 Servlet 容器

`SpringBoot2.1.7`支持以下内置Servlet容器
- Tomcat 9.0 （servlet版本4.0）
- Jetty 9.4 （servlet版本3.1）
- Undertow 2.0 （servlet版本4.0）

此外，你也可以把`SpringBoot`应用部署在任何兼容Servlet 3.1+的容器里。
