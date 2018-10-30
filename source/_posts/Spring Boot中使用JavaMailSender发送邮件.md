---
title: Spring Boot中使用JavaMailSender发送邮件
date: 2018-01-17
category: "java" 
tags: 
  - java
  - mail
  - springboot
---

# SpringBoot中使用JavaMailSender发送邮件

## 导入依赖
在Spring Boot的工程中的 .gradle 中引入spring-boot-starter-mail依赖： 使用gradle
``` 
compile group: 'org.springframework.boot', name: 'spring-boot-starter-mail', version: '1.5.9.RELEASE'
```
<!-- more -->
## 配置文件 resource ，使用QQ邮箱为例。
```
spring.mail.host: smtp.qq.com 
spring.mail.username: 用户名
spring.mail.password: 密码  该密码是QQ邮箱的授权码，
spring.mail.properties.mail.smtp.auth: true
spring.mail.properties.mail.smtp.starttls.enable: true
spring.mail.properties.mail.smtp.starttls.required: true
```

