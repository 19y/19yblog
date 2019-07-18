---
title: Tomcat配置虚拟路径，使上传文件与服务器分离以及<Context reloadable=true>的作用
date: 2019-07-18 23:02:02
tags:
- Context
- reloadable
- tomcat
- 配置虚拟路径
---


Tomcat配置虚拟路径，使上传文件与服务器分离2016年05月10日 15:52:16 bellus- 阅读数：20160

 版权声明：知识共享原则。 https://blog.csdn.net/xiaoyu19910321/article/details/51363679

遇到问题介绍：项目中头像上传，上传图片到服务器。如果使用tomcat下的目录作为上传图片的路径，则每次重启服务器图片将消失
遇到问题：使用服务器物理磁盘的D:\upload路径存储文件，访问请求路径的不会映射到希望到的请求。
解决：可以使用tomcat的配置文件将某个请求 映射到 物理路径下 ，完成图片的回显。
具体操作：使用Tomcat虚拟路径
1.修改tomcat的配置文件
window环境
 
首先找到tomcat目录下conf目录下的server.xml文件
 
在server.xml文件中找到<Host></Host>
 
然后在其中加上这
             <Context path="/demo/file" docBase="D:\demo\File\file"></Context>
例如：

<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">

        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                prefix="localhost_access_log." suffix=".txt"
                pattern="%h %l %u %t &quot;%r&quot; %s %b" />
  
<!-- 下面两条主要是tomcat转发图片请求到相应的电脑物理磁盘位置 -->
<Context path="/dajean/uploadpng/" docBase="D:\setup\dajean\uploadpng\"></Context>
      
      </Host>
             

 
tomcat在的请求一般为http://localhost:8080/demo/file/abc.jpg
配置完重启之后，该请求会自动跳转到物理路径D:\demo\File\file下查找。
会访问本机的D:\demo\File\file\abc.jpg

有效解决了存储路径与tomcat路径的分离。
 


<Context reloadable="true">  

为了在开发时，让tomcat能够自动重新加载，我们修改过的代码和配置，需要对Tomcat的context.xml文件进行设置。
在标签中，加上reloadable属性，并且将值设为true

<Context reloadable="true">  
    <!--注意： reloadable设为true，目的是为了方便开发阶段， 它会影响tomcat性能；当在正式部署服务时，需要改成false -->  
1
2
重启Tomcat时或（tomcat服务开启，重新部署项目时），出现如下异常：

信息: Illegal access: this web application instance has been stopped already.  Could not load java.net.BindException.  The eventual following stack trace is caused by an error thrown for debugging purposes as well as to attempt to terminate the thread which caused the illegal access, and has no functional impact.
java.lang.IllegalStateException
org.apache.catalina.loader.WebappClassLoader.loadClass(WebappClassLoader.java:1272)
 at org.apache.catalina.loader.WebappClassLoader.loadClass(WebappClassLoader.java:1232)
 at java.lang.ClassLoader.loadClassInternal(ClassLoader.java:320)
 at com.mysql.jdbc.CommunicationsException.<init>(CommunicationsException.java:161)
 at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:2759)
 at com.mysql.jdbc.MysqlIO.quit(MysqlIO.java:1410)
 at com.mysql.jdbc.Connection.realClose(Connection.java:4947)
 at com.mysql.jdbc.Connection.cleanup(Connection.java:2063)
 at com.mysql.jdbc.Connection.finalize(Connection.java:3403)
 at java.lang.ref.Finalizer.invokeFinalizeMethod(Native Method)
at java.lang.ref.Finalizer.runFinalizer(Finalizer.java:83)
at java.lang.ref.Finalizer.access$100(Finalizer.java:14)

原因是：tomcat重新装载web应用程序失败导致的。当应用程序卸载时，并不会关闭所有的线程。当tomcat已经关闭了其类加载器后，一些线程依然会继续运行，这样就导致出错。不过这个不影响正常使用，不管影响不影响，看到异常信息就不顺眼。

解决方案：修改tomcat目录下的context.xml,找到<Context>标签，把reloadble的属性值设为：reloadable=”false”，即<Context reloadable=”false”>。


---------------------
作者：kekeair-zhang
来源：CSDN
原文：https://blog.csdn.net/ke_zhang_123/article/details/78537652
版权声明：本文为博主原创文章，转载请附上博文链接！
