### Tomcat分析



#### 一、目录分析



##### 1.conf目录

- `catalina.policy` tomcat安全策略
- `[]`







#### 二、部署项目

##### 1.直接把war包放在`tomcat`的webapp是目录下面



##### 2.使用`context` 标签指定目录以及访问`path`(tomcat官网不建议这么做，因为侵入性比较强，而且要重启tomcat)

在`server.xml`配置文件中添加 

 ```xml
//这里可以定义多个context
<Context docBase="/Users/yangf/Personal/Java/test/target/test.war" path="reloadable="true"/>

 ```

##### 3.在conf/Catalina/localhost下添加xxx.xml文件，xxx表示contextPath ROOT.xml表示根目录

