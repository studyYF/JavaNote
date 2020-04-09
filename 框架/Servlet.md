#### 1.Servlet是什么

运行在Web服务器或应用服务器上的程序，作为来自Web浏览器或其他HTTP客户端的请求和HTTP服务器上的数据库或应用程序之间的中间层。
**CGI** common gateway interface 公共网关接口

#### 2.架构
![image-20200409152101288](/Users/yangf/Personal/Note/Resources/image-20200409152101288.png)


### 3.ServletConfig
servletConfig是加载web.xml配置文件的

```
<init-param>
    <param-name>name</param-name>
    <param-value>crossyf</param-value>
</init-param>
```

使用`servletConfig.getInitParamter("name")`可以获取到对应的值

#### 4.ServletContext
servletContext随着tomcat的启动而创建，是所有servlet共享的对象，servlet与servlet之间可以通过servletContext进行互相传值
```
ServletContext servletContext = this.getServletContext();
servletContext.setAttribute("name", "crossyf");
servletContext.getAttribute("name");
```
ServletContext还可以获取到web.xml文件定义的属性，供所有的servle进行访问
```
<context-param>
    <param-name>name</param-name>
    <param-value>crossyf</param-value>
</context-param>
```
```
ServletContext servletContext = this.getServletContext();
String value = servletContext.getParamter("name");
```