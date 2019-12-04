---
title: exchange邮箱类型使用Java发送邮件
category: 技术
tags:
  - java
abbrlink: b3686e94
date: 2018-02-11 00:00:00
---
# 需求
使用Java实现自动发送exchange类型邮箱的功能，本文不会使用SMTP。
# 添加dependencie
使用gradle，代码如下：
```groovy
compile 'com.microsoft.ews-java-api:ews-java-api:2.0'
```
使用maven，代码如下：
```xml
<dependency>
	<groupId>com.microsoft.ews-java-api</groupId>
	<artifactId>ews-java-api</artifactId>
	<version>2.0</version>
</dependency>
```
或者直接导入`ews-java-api-2.0.jar`包

# 实现
```java
package com.myluffy.springboot.chapter33;

import microsoft.exchange.webservices.data.core.ExchangeService;
import microsoft.exchange.webservices.data.core.enumeration.misc.ExchangeVersion;
import microsoft.exchange.webservices.data.core.enumeration.property.BodyType;
import microsoft.exchange.webservices.data.core.service.item.EmailMessage;
import microsoft.exchange.webservices.data.credential.ExchangeCredentials;
import microsoft.exchange.webservices.data.credential.WebCredentials;
import microsoft.exchange.webservices.data.property.complex.MessageBody;

import java.net.URI;


public class ExchangeSendMail {
    public void sendMail() throws Exception{
        ExchangeService service = new ExchangeService(ExchangeVersion.Exchange2010_SP1);
        ExchangeCredentials credentials = new WebCredentials("username", "password");
        service.setCredentials(credentials);
        // outlook.com 改为自己的邮箱服务器地址
        service.setUrl(new URI("https://outlook.com/EWS/Exchange.asmx"));
        EmailMessage msg = new EmailMessage(service);
        // 主题
        msg.setSubject("subject");
        // 内容
        // MessageBody 默认是发送html格式的内容
        MessageBody content = new MessageBody(BodyType.Text, "content");
        msg.setBody(content);
//        msg.setBody(MessageBody.getMessageBodyFromText("content")); // 内容为html格式
        // 收件人
        msg.getToRecipients().add("xx@gmail.com");
        // 抄送人
        msg.getCcRecipients().add("xx2@gmail.com");
        // 暗送
        msg.getBccRecipients().add("xx3@gmail.com");
        // 附件
        msg.getAttachments().addFileAttachment("/Users/xx/Downloads/demo.text");
        msg.send();
    }
}

```
如有问题，可联系我。

参考文档
[ews-java-api](https://github.com/OfficeDev/ews-java-api/wiki/Getting-Started-Guide "ews-java-api")