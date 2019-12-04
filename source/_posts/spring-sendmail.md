---
title: 通过Spring发送邮件
category:
  - Java
tags:
  - spring
  - java
abbrlink: 390cf6e4
date: 2017-09-19 00:00:00
---
记录一次在工作使用Spring框架，发送邮件的demo。
## 依赖
注意一下，我使用的是gradle，使用maven的需要通过maven的方式引入jar包依赖。
```groovy
compile group: 'com.sun.mail', name: 'javax.mail', version: '1.6.0'
compile group: 'org.apache.velocity', name: 'velocity', version: '1.7'
compile 'org.springframework:spring-context-support'
```
## 具体实现
```java
@Service
public class MailServiceImpl implements MailService {
    @Autowired
    private JavaMailSender mailSender;

    public VelocityEngine getVelocityEngine() {
        VelocityEngine velocityEngine = new VelocityEngine();
        velocityEngine.setProperty("resource.loader", "class");
        velocityEngine.setProperty("class.resource.loader.class", "org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader");
        velocityEngine.setProperty("input.encoding", "UTF-8");
        velocityEngine.setProperty("output.encoding", "UTF-8");
        velocityEngine.init();
        return velocityEngine;
    }

    private static MimeMessageHelper helper(MimeMessage mimeMessage)throws Exception{
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
        List<String> tos = new ArrayList<>();
		// 接收邮件的地址 集合 可以为多个
//        tos.add("service@*.com");
        tos.add("service2@*.com");
        helper.setFrom("service@*.com");
        String[] strings = (String[])tos.toArray(new String[tos.size()]);
        helper.setTo(strings);
        return helper;
    }
    @Override
    public void sendMail(Map<String, Object> map) throws Exception {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = helper(mimeMessage);
		// 设置邮件主题
        helper.setSubject("overdue");
		// 需要填充到模板中的内容
        Map<String, Object> model = new HashedMap();
        model.put("hospital", map.get("hospital"));
		
        String template = (String) map.get("template");
		// 得到模板 并填充内容，模板地址根据自己的实际情况所定
        String text = VelocityEngineUtils.mergeTemplateIntoString(
                getVelocityEngine(), "template/" + template, "UTF-8", model);
        helper.setText(text, true);
		//  发送邮件
        mailSender.send(mimeMessage);
    }
}

```

## 模板实例
在resources目录下，新建一个template文件夹，新建一个 *.vm 文件，注意文件后缀为 ``.vm``
如图所示：
![](http://www.myluffy.com/wp-content/uploads/2018/06/2c30c54a83059f6cde42ee9e04a56a2e.png)

模板中内容为：
```html
<html>
<body>
<div>
    test
    <p>hospital: ${hospital}</p>
</div>
</body>
</html>
```
** 注意 ``${hospital}`` 和service中model字段相对应。

## 配置发送邮件的密码
基本代码写好之后，我们需要在 ``application.yml`` 中配置一下邮件的用户名密码等；
```groovy
spring:
  mail:
    default-encoding: utf-8
    host: smtp.exmail.qq.com # 邮箱host，不同的邮箱host是不一样的，这个列子是腾讯企业邮的，具体可以google。
    username: service@*.com # 发送邮箱的用户名， 替换为自己的
    password: 123456 # 填写自己邮箱的密码，如果是QQ邮箱，应该是需要填写授权码
```

## 测试用例
现在基本代码洗好了，我们接着写一个测试用例
我的项目所用的是springboot，具体情况视自己项目的情况而定。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
@SpringBootTest(classes = OverdueApplication.class)
@ActiveProfiles("ubuntu")
@Ignore
public class MailTest {

    @Autowired
    private MailService mailService;

    @Test
    public void testSendMail() {
        Map<String, Object> map = new HashMap<>();
		// 模板中的变量
        map.put("hospital", "test hospital");
		// 对应的邮件模板 文件名
        map.put("template", "overdueMail.vm");
        try {
            mailService.sendMail(map);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

```

## 结果
![](http://www.myluffy.com/wp-content/uploads/2018/06/38dc06f8e7ed4df81bd4867d54192dd0.png)

这样，一件简单的邮件模板发送就成功了。