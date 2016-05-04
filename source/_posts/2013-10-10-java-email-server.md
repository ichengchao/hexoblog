---
layout: post
title: Java实现的邮件服务器
date: 2013-10-10 09:11
author: charles
comments: true

---

由于项目要求,需要实现一个邮箱服务器,原本打算自己实现一个,google了一下发现一个很好用的.  
项目名称是:subethasmtp,地址:<https://code.google.com/p/subethasmtp/>  
用起来也非常方便.  

```java
//启动邮件服务器
public void startMailServer() {
        try {
            MySimpleMessageListener myListener = new MySimpleMessageListener();
            SMTPServer smtpServer = new SMTPServer(new SimpleMessageListenerAdapter(myListener));
            smtpServer.setPort(25);
            smtpServer.start();
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
    }
//MySimpleMessageListener.java
public class MySimpleMessageListener implements SimpleMessageListener {
    private Log          logger = LogFactory.getLog(MySimpleMessageListener.class);
    @Override
    public boolean accept(String from, String recipient) {
        // System.out.println(from);
        // System.out.println(recipient);
        // true表示需要投递
        return true;
    }
    @Override
    public void deliver(String from, String recipient, InputStream data) throws Exception {
        // System.out.println(from);
        // System.out.println(recipient);
        // System.out.println("===================================");
        // String body = this.convertStreamToString(data);
        Session session = Session.getInstance(new Properties(), null);
        try {
            MimeMessage mm = new MimeMessage(session, data);
            // System.out.println(mm.getContentType());
            // System.out.println(mm.getContent());
            String msg = mm.getContent().toString();
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
    }
}
```


如何测试:  

下载foxmail之类的客户端,或者直接用outlook.  
把stmp的地址改成localhost,并且把邮件的格式设置成纯文本.  
ok,现在就写封邮件试试吧.你的邮件内容就可以在你的控制台打印出来了.  

