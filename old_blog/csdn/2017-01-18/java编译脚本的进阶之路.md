### 从零开始
因为工作的原因，需要用java写个简单的测试程序，平时自己工作主要是python，如果因为这去搭建一个eclipse的开发环境总觉得有点小题大作，于是就在配置完jdk的开发环境后，写了下面的一个脚本。

#### 目录结构
- build.sh
- lib/
 - *.jar(库文件)
- test.java

其中build.sh为编译脚本，lib/路径下面放置的是依赖的第三方提供的jar包文件，test.java即为需要编译的java文件了。
下面贴下我的build.sh脚本的写法，很简单，略带幼稚，望大家轻拍
build.sh
```
#!/bin/sh
MYCLASS=
for file in lib/*
    do
        MYCLASS=$MYCLASS:$file
    done
export CLASSPATH=$CLASSPATH:$MYCLASS
javac -Xlint *.java
```

### 初登正途
又因为机缘巧合，我要用java写一个简单的http服务(好吧，就当时开了一门副业)，上面的脚本显然需要改动了。再贴下新的目录结构吧

- build.sh
- clear.sh
- deploy.sh
- lib/
 - *.jar
- src/
- resourse/
- WebContent/
 - WEB-INF/
 
在WEB-INF目录下是classes路径和和web.xml文件，写过servlet的兄弟对这个应该不会陌生。
src路径下就是我们需要编译的java文件，一种简单的组织形式就是跟eclipse那样，按照包名的路径去组织
比如我下面这两个个servlet程序
Log4JTestServlet.java
```
package com.firstserver.server;

import java.io.IOException;  

import javax.servlet.ServletConfig;  
import javax.servlet.ServletException;  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  

import org.apache.log4j.Logger;  

/** 
 * Servlet implementation class Log4JTestServlet 
 */
public class Log4JTestServlet extends HttpServlet {  
    private static final long serialVersionUID = 1L;  
    private static Logger logger = Logger.getLogger(Log4JTestServlet.class);    

    /** 
     * @see HttpServlet#HttpServlet() 
     */  
    public Log4JTestServlet() {  
        super();  
        // TODO Auto-generated constructor stub  
    }  

    /** 
     * @see Servlet#init(ServletConfig) 
     */  
    public void init(ServletConfig config) throws ServletException {  
        // TODO Auto-generated method stub  
    }  

    /** 
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response) 
     */  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        // 记录debug级别的信息    
        logger.debug("This is debug message.");    
        // 记录info级别的信息    
        logger.info("This is info message.");    
        // 记录error级别的信息    
        logger.error("This is error message.");    
    }
```
Log4JInitServlet.java
```
package com.firstserver.server;

import java.io.File;  
import java.io.IOException;  

import javax.servlet.ServletConfig;  
import javax.servlet.ServletContext;  
import javax.servlet.ServletException;  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  

import org.apache.log4j.BasicConfigurator;  
import org.apache.log4j.PropertyConfigurator;  

/** 
 * Servlet implementation class Log4JInitServlet 
 */  
public class Log4JInitServlet extends HttpServlet {  
    private static final long serialVersionUID = 1L;  

    /** 
     * @see HttpServlet#HttpServlet() 
     */  
    public Log4JInitServlet() {  
        super();  
        // TODO Auto-generated constructor stub  
    }  

    /** 
     * @see Servlet#init(ServletConfig) 
     */  
    public void init(ServletConfig config) throws ServletException {  
        System.out.println("Log4JInitServlet 正在初始化 log4j日志设置信息");  
        String log4jLocation = config.getInitParameter("log4j-properties-location");  

        ServletContext sc = config.getServletContext();  

        if (log4jLocation == null) {  
            System.err.println("*** 没有 log4j-properties-location 初始化的文件, 所以使用 BasicConfigurator初始化");  
            BasicConfigurator.configure();  
        } else {  
            String webAppPath = sc.getRealPath("/");  
            String log4jProp = webAppPath + log4jLocation;  
            File yoMamaYesThisSaysYoMama = new File(log4jProp);  
            if (yoMamaYesThisSaysYoMama.exists()) {  
                System.out.println("使用: " + log4jProp+"初始化日志设置信息");  
                PropertyConfigurator.configure(log4jProp);  
            } else {  
                System.err.println("*** " + log4jProp + " 文件没有找到， 所以使用 BasicConfigurator初始化");  
                BasicConfigurator.configure();  
            }  
        }  
        super.init(config);  
    }  

    /** 
     * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response) 
     */  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        // TODO Auto-generated method stub  
    }  

    /** 
     * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response) 
     */  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        // TODO Auto-generated method stub  
    }  

}
```
上面的两个文件放置于src/com/firstserver路径下
lib路径下存放依赖的jar包
build.sh脚本内容
```
#!/bin/sh
MYCLASS=/usr/tomcat/apache-tomcat-7.0.67/lib/servlet-api.jar
for file in lib/*
    do
        MYCLASS=$MYCLASS:$file
    done
export CLASSPATH=$CLASSPATH:$MYCLASS
javac src/com/firstserver/server/*.java -d WebContent/WEB-INF/classes
cp resource/* WebContent/WEB-INF/classes/com/firstserver/server
cp -R lib WebContent/WEB-INF/
```
deploy.sh用于在编译完之后，将目标文件拷贝到运行目录
deploy.sh
```
#!/bin/sh
pushd /usr/tomcat/apache-tomcat-7.0.67/webapps/MyFirstServer
sudo rm -rf WEB-INF/
popd
sudo cp -R WebContent/WEB-INF /usr/tomcat/apache-tomcat-7.0.67/webapps/MyFirstServer
pushd /usr/tomcat/apache-tomcat-7.0.67/bin
sudo ./shutdown.sh
sudo ./startup.sh
popd
```
web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name></display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  <servlet>  
        <servlet-name>Log4JTestServlet</servlet-name>  
        <servlet-class>com.firstserver.server.Log4JTestServlet</servlet-class>  
  </servlet>  

  <!--用来启动 log4jConfigLocation的servlet -->  
  <servlet>  
      <servlet-name>Log4JInitServlet</servlet-name>  
      <servlet-class>com.firstserver.server.Log4JInitServlet</servlet-class>  
      <init-param>  
          <param-name>log4j-properties-location</param-name>  
          <param-value>/WEB-INF/classes/log4j.properties</param-value>  
      </init-param>  
      <load-on-startup>1</load-on-startup>  
  </servlet>  

  <servlet-mapping>  
      <servlet-name>Log4JTestServlet</servlet-name>  
      <url-pattern>/test</url-pattern>  
  </servlet-mapping>  

</web-app>
```

### 走向maven
后来这个项目需要交给qa来发布，他们说一定要是maven的工程，于是我又不得不将这个目录更改，如何搭建maven的开发环境这里就略过了，我当时就是从干java开发的同事那里拷贝了一个maven的包，然后解压，修改了其中的conf路径下的setting文件，改了下localRepository这个配置
```
build.sh
pom.xml
src/
   main/
       java/
       webapp/
             WEB-INF/
                     web.xml
```
这里的java路径下面的所放内容跟上面的src是一样的
pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.firstserver.server</groupId>
  <artifactId>MyFirstServer</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>MyFirstServer Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.4</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.5</version>
    </dependency>
  </dependencies>
  <build>
    <finalName>MyFirstServer</finalName>
  </build>
</project>

```
pom文件只是一个例子，本文例子程序所需要的依赖包没有全列在里边。
编译时执行build.sh就可以
build.sh
```
#!/bin/sh
export MAVEN_HOME=/home/myhome/javacode/apache-maven-3.2.5
export PATH=${MAVEN_HOME}/bin:${PATH}
mvn clean package
```