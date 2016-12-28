##总结 550,535,553 Mail from must equal authorized user— jenkinshudson email163邮箱和26邮箱成功配置总结
###553 mail from must equal authorized user
上面的问题搞了好长时间，很多网上答案都是错的，这里非常感谢，http://www.zhihu.com/question/21774292
system admin e-mail address  这里设置要和发件人邮箱保持一致。

1.如果不保持一致报错如下图所示：

![image](http://images.cnitblog.com/blog/534352/201402/141716578179854.png)

2.邮件保持一致的配置如下图所示：
![img](http://images.cnitblog.com/blog/534352/201402/141717493843979.png)
![img](http://images.cnitblog.com/blog/534352/201402/141718440819418.png)
![img](http://images.cnitblog.com/blog/534352/201402/141718568406884.png)

3.如果smtp服务器开发配置不正确会报如下错误
authenticationfailedexception: 550 
比如163的邮箱配置成126的服务器开发smtp.126.com，那么将会报错。
###.javax.mail.authenticationfailedexception: 535 error: authentication failed  535的错误是由于密码错误
