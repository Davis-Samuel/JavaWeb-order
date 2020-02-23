# 1.准备

## 创建JavaWeb父工程

- 添加Servlet和servlet-jsp的依赖：

  ```xml
  <dependencies>
      <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
      </dependency>
      
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>javax.servlet.jsp-api</artifactId>
        <version>2.2.1</version>
        <scope>provided</scope>
      </dependency> 
      
  <dependency>
    <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.2</version>
      <scope>provided</scope>
  </dependency>
      
  <dependency> <!--jsp表达式的依赖-->
      <groupId>javax.servlet.jsp.jstl</groupId>
      <artifactId>jstl-api</artifactId>
      <version>1.2</version>
  </dependency>
      
  <dependency> <!--jsp表达式的依赖的-->
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
  </dependency>
      
      <dependency>
              <groupId>javax.servlet</groupId>
              <artifactId>servlet-api</artifactId>
              <version>2.5</version>
              <scope>provided</scope>
          </dependency>
  
    </dependencies>
  ```
  
  

## 创建Javaweb子工程(servlet-01)

- 添加，Java和resource文件夹。

- 添加com.it.helloservlet包，添加HelloServlet类：

  ```java
  package com.it.servlet;
  
  import javax.servlet.ServletException;
  import javax.servlet.ServletOutputStream;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  import java.io.PrintWriter;
  public class HelloServlet extends HttpServlet{
  //    由于二者只是get，post请求方式不同，所以可以互相调用，业务逻辑都一样。
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
           System.out.println("join in doGet");
  //响应流 ServletOutputStream outputStream = resp.getOutputStream();
          PrintWriter writer = resp.getWriter();  
          writer.print("hello");
      }
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doPost(req, resp);
      }
  }
  ```

- 换web.xml：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0"
    metadata-complete="true">
  
  <!--注册servlet-->
      <servlet>
          <servlet-name>hello</servlet-name>
          <servlet-class>com.it.servlet.HelloServlet</servlet-class>
      </servlet>
  <!--servlet请求路径-->
      <servlet-mapping>
          <servlet-name>hello</servlet-name >
          <!--       路径可以写为 <url-pattern>*.do</url-pattern> , do可以换成别的，固有的映射路径被访问优先级最高-->
          <url-pattern>/hello</url-pattern>
      </servlet-mapping>
  
  </web-app>
  ```

- 配置Tomcat：![image-20200216104257161](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200216104257161.png)![image-20200216104444266](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200216104444266.png)

- 如果是因为资源导出有问题，就在pom.xml中添加：

  ```xml
  <!--在build中配置resources，来防止我们资源导出失败的问题-->
      <build>
          <resources>
              <resource>
                  <directory>src/main/resources</directory>
                  <includes>
                      <include>**/*.properties</include>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>true</filtering>
              </resource>
  
              <resource>
                  <directory>src/main/java</directory>
                  <includes>
                      <include>**/*.properties</include>
                      <include>**/*.xml</include>
                  </includes>
                  <filtering>true</filtering>
              </resource>
          </resources>
      </build>
  ```

  

## 创造一个error

- com.it.servlet中，new一个ErrorServlet类：

  ```java
  package com.it.servlet;
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  import java.io.PrintWriter;
  public class ErrorServlet extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  //        修改格式为text/html的页面编码，变为utf-8
          resp.setContentType("text/html");
          resp.setCharacterEncoding("utf-8");
          PrintWriter writer = resp.getWriter();
          writer.print("<h1>Error 404 </h1>");
      }
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doPost(req, resp);
      }
  }
  ```

- web.xml中添加：

  ```xml
      <servlet>
          <servlet-name>error</servlet-name>
          <servlet-class>com.it.servlet.ErrorServlet</servlet-class>
      </servlet>
      <servlet-mapping>
          <servlet-name>error</servlet-name >
  <!--        /通配符*，后面不管是什么，都执行error-->
          <url-pattern>/*</url-pattern>
      </servlet-mapping>
  ```

# 2.ServletContext

## <u>*模块servlet-02*</u>

- 配置与模块一，一样。修改tomcat，用那个servlet就加那个servlet：![image-20200216121300790](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200216121300790.png)

- 简单使用servletContext代码:

  ```java
  package com.it.servlet;
  import javax.servlet.ServletContext;
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  public class HelloServlet extends HttpServlet {
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          ServletContext servletContext = this.getServletContext();
          String name = "111";
          //将name存进了servletContext
          servletContext.setAttribute("name",name);
          
  /*     使用  ： new一个类，编写web.xml
      ServletContext servletContext = this.getServletContext();
      由object强转为String
      String name = (String) servletContext.getAttribute(name);
      可以sout也可以resp.getWrite().print("name = " + name);
  */
      }
  }
  ```

- 获得初始化参数![image-20200219161820199](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219161820199.png)

- 请求转发![image-20200219162720165](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219162720165.png)

- 在maven中，Java文件和resource文件运行后都在target里面的classes里面，也叫classpath路径。![image-20200219163645175](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219163645175.png)

- 读资源文件文件db.properties：![image-20200219165806186](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219165806186.png)

- 看HttpServletRespones和HttpServletRequest源码，里面有很多方法，设置状态(300,404...)，设置响应头，响应状态码等等等。

# 3.Response

## <u>*模块servlet-03-response*</u>

- 新建了一个模块，需要修改tomcat的配置

## 常见应用

### 向浏览器输出信息

```Java
//字节 ServletOutputStream outputStream = resp.getOutputStream()       		  outputStream.writer("hello");
//非字节。PrintWriter writer = resp.getWriter();  writer.writer("hello");
```

### 输入路径回车下载文件。

- 图片在resource文件中![image-20200219182212528](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219182212528.png)

- ![image-20200219184154767](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219184154767.png)

- FileServlet.java：

  ```java
  package com.it.servlet;
  
  import javax.servlet.ServletException;
  import javax.servlet.ServletOutputStream;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.FileInputStream;
  import java.io.IOException;
  import java.net.URL;
  import java.net.URLEncoder;
  
  public class FileServlet extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //获取下载的路径,右键文件 → copy path → absolute path
          String realPath = "D:\\MY\\已安装\\Java开发\\Java.IDEA.Maven\\Java.IDEA.Maven程序\\JavaWeb\\servlet-03-response\\src\\main\\resources\\哈哈哈.png";
          System.out.println("下载的文件路径" + realPath);
          //下载的文件名 "获取路径之后的第一位所以+1，第一位就是文件名"
          String fileName = realPath.substring(realPath.lastIndexOf("\\") + 1);
          //将图片设置成浏览器可以支持，加入中文文件名的编码
          resp.setHeader("Content-Disposition","attachment;filename=" + URLEncoder.encode(fileName,"UTF-8"));
          //获取下载文件的输入流
          FileInputStream in = new FileInputStream(realPath);
          //创建缓冲区
          int len=0;
          byte[] buffer = new byte[1024];
          //获取输出流对象
          ServletOutputStream out = resp.getOutputStream();
          //用输入流写道缓冲区，再写入客户端
          while ((len=in.read(buffer))>0){
              out.write(buffer,0,len);
          }
          //关闭
      }
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  ```

- web.xml：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
           version="3.0"
           metadata-complete="true">
    <!--注册servlet-->
    <servlet>
      <servlet-name>down</servlet-name>
      <servlet-class>com.it.servlet.FileServlet</servlet-class>
    </servlet>
    <!--servlet请求路径-->
    <servlet-mapping>
      <servlet-name>down</servlet-name >
      <!--       路径可以写为 <url-pattern>*.do</url-pattern> , do可以换成别的。固有的映射路径被访问优先级最高-->
      <url-pattern>/down</url-pattern>
    </servlet-mapping>
  </web-app>
  ```

### 实现验证码+数字图片+随机数

- ![image-20200219190546544](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219190546544.png)
- ![image-20200219190708206](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200219190708206.png)

### 重定向

- ```java
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.sendRedirect("/down");
      }
  ```

- 在web.xml：

  ```xml
   <!--注册servlet-->
    <servlet>
      <servlet-name>down</servlet-name>
      <servlet-class>com.it.servlet.FileServlet</servlet-class>
    </servlet>
    <!--servlet请求路径-->
    <servlet-mapping>
      <servlet-name>down</servlet-name >
      <!--       路径可以写为 <url-pattern>*.do</url-pattern> , do可以换成别的。固有的映射路径被访问优先级最高-->
      <url-pattern>/down</url-pattern>
    </servlet-mapping>
  
  
  
    <servlet>
      <servlet-name>redirect</servlet-name>
      <servlet-class>com.it.servlet.FileServlet</servlet-class>
    </servlet>
    <!--servlet请求路径-->
    <servlet-mapping>
      <servlet-name>redirect</servlet-name >
      <!--       路径可以写为 <url-pattern>*.do</url-pattern> , do可以换成别的。固有的映射路径被访问优先级最高-->
      <url-pattern>/redirect</url-pattern>
    </servlet-mapping>
  ```

- 在浏览器中输入/redirect，就会跳转到/down中。

# 3.Request

## *<u>模块servlet-04-request</u>*

## 常见应用

### 获取前端请求参数，并请求转发

- ​	![image-20200220191657671](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200220191657671.png)

- ![image-20200220201134603](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200220201134603.png)

# 4.Cookie

- ![image-20200220215300479](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200220215300479.png)

# 5.Session

## *<u>模块session</u>*

- 在Session类中使用保存，在GetSession类中取得。

- Session.java：

  ```Java
  package com.it.session;
  import com.it.pojo.Person;
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import javax.servlet.http.HttpSession;
  import java.io.IOException;
  public class Session extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //编码
          req.setCharacterEncoding("utf-8");
          resp.setCharacterEncoding("utf-8");
          resp.setContentType("text/html;charset=utf-8");
          //得到session
          HttpSession session = req.getSession();
          //向session存东西,可以存对象
          session.setAttribute("name",new Person("lisi",10));
          //得到session的id
          String id = session.getId();
          //判断session是不是新建的
          if (session.isNew()){
              resp.getWriter().write("创建成功" + id);
          }else {
              resp.getWriter().write("已经存在" + id);
          }
      }
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  ```

- GetSession.java：

  ```java
  package com.it.session;
  import com.it.pojo.Person;
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import javax.servlet.http.HttpSession;
  import java.io.IOException;
  public class GetSession extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  
          req.setCharacterEncoding("utf-8");
          resp.setCharacterEncoding("utf-8");
          resp.setContentType("text/html;charset=utf-8");
  
          HttpSession session = req.getSession();
          Person name = (Person)session.getAttribute("name");
          System.out.println(name.toString());
      }
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  ```

- web.xml：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
           version="3.0"
           metadata-complete="true">
    <servlet>
      <servlet-name>getSession</servlet-name>
      <servlet-class>com.it.session.GetSession</servlet-class>
    </servlet>
    <servlet-mapping>
      <servlet-name>getSession</servlet-name >
      <url-pattern>/getSession</url-pattern>
    </servlet-mapping>
  
    <servlet>
      <servlet-name>session</servlet-name>
      <servlet-class>com.it.session.Session</servlet-class>
    </servlet>
    <servlet-mapping>
      <servlet-name>session</servlet-name >
      <url-pattern>/session</url-pattern>
    </servlet-mapping>
  
  </web-app>
  ```


# 6.JSP

## *<u>模块jsp</u>*

- 表达式：

  ```jsp
  <%= 变量或者表达式%>  <%= new java.util.Date()%> 
  ```

- jsp脚本片段：![image-20200221145938749](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200221145938749.png)

  

- ![image-20200221152627647](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200221152627647.png)
- ![image-20200221155150633](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200221155150633.png)
- **${ }:也是输出**
- ![image-20200221155357569](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200221155357569.png)

- 导包：

  ```jsp
  在最上面 <%@ page import="java.util.*"%>  alt+回车
  ```

- ![image-20200221162541687](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200221162541687.png)

- ![image-20200221163844335](C:\Users\91566\AppData\Roaming\Typora\typora-user-images\image-20200221163844335.png)

## 九大内置对象四大作用域

- PageContext    存东西

- Request     存东西

- Response

- Session      存东西

- Application   【SerlvetContext】   存东西

- config    【SerlvetConfig】

- out

- page ，不用了解

- exception

- ```jsp
  pageContext.setAttribute("name1","秦疆1号"); //保存的数据只在一个页面中有效
  request.setAttribute("name2","秦疆2号"); //保存的数据只在一次请求中有效，请求转发会携带这个数据
  session.setAttribute("name3","秦疆3号"); //保存的数据只在一次会话中有效，从打开浏览器到关闭浏览器
  application.setAttribute("name4","秦疆4号");  //保存的数据只在服务器中有效，从打开服务器到关闭服务器
  ```

  request：客户端向服务器发送请求，产生的数据，用户看完就没用了，比如：新闻，用户看完没用的！

  session：客户端向服务器发送请求，产生的数据，用户用完一会还有用，比如：购物车；

  application：客户端向服务器发送请求，产生的数据，一个用户用完了，其他用户还可能使用，比如：聊天数据；

## JSP标签、JSTL标签、EL表达式

- EL表达式：获取数据，执行运算，获取web开发常用对象

- JSP标签：

  ```jsp
  <%--jsp:include--%>
  
  <%--
  相当于：http://localhost:8080/jsptag.jsp?name=kuangshen&age=12
  --%>
  
  <jsp:forward page="/jsptag2.jsp">
      <jsp:param name="name" value="kuangshen"></jsp:param>
      <jsp:param name="age" value="12"></jsp:param>
  </jsp:forward>
  ```

- JSTL表达式：

- IF语句

  ```jsp
  <head>
      <title>Title</title>
  </head>
  <body>
  
  
  <h4>if测试</h4>
  
  <hr>
  
  <form action="coreif.jsp" method="get">
      <%--
      EL表达式获取表单中的数据
      ${param.参数名}
      --%>
      <input type="text" name="username" value="${param.username}">
      <input type="submit" value="登录">
  </form>
  
  <%--判断如果提交的用户名是管理员，则登录成功--%>
  <c:if test="${param.username=='admin'}" var="isAdmin">
      <c:out value="管理员欢迎您！"/>
  </c:if>
  
  <%--如果不是管理员，自闭合标签--%>
  <c:out value="${isAdmin}"/>
  
  </body>
  ```

- choose when语句：

  ```jsp
  <body>
  
  <%--定义一个变量score，值为85--%>
  <c:set var="score" value="55"/>
  
  <c:choose>
      <c:when test="${score>=90}">
          你的成绩为优秀
      </c:when>
      <c:when test="${score>=80}">
          你的成绩为一般
      </c:when>
      <c:when test="${score>=70}">
          你的成绩为良好
      </c:when>
      <c:when test="${score<=60}">
          你的成绩为不及格
      </c:when>
  </c:choose>
  
  </body>
  ```

- foreach语句：

  ```jsp
  <%
  
      ArrayList<String> people = new ArrayList<>();
      people.add(0,"张三");
      people.add(1,"李四");
      people.add(2,"王五");
      people.add(3,"赵六");
      people.add(4,"田六");
      request.setAttribute("list",people);
  %>
  
  
  <%--
  var , 每一次遍历出来的变量
  items, 要遍历的对象
  begin,   哪里开始
  end,     到哪里
  step,   步长
  --%>
  <c:forEach var="people" items="${list}">
      <c:out value="${people}"/> <br>
  </c:forEach>
  
  <hr>
  
  <c:forEach var="people" items="${list}" begin="1" end="3" step="1" >
      <c:out value="${people}"/> <br>
  </c:forEach>
  ```

# 7.Filter

## *<u>模块filter</u>*

- 创建一个ServletFilter类：

  ```java
  package com.it.filter;
  import javax.servlet.*;
  import java.io.IOException;
  
  public class ServletFilter implements Filter {
      public void init(FilterConfig filterConfig) throws ServletException {
          System.out.println("程序创建");
      }
  
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
          request.setCharacterEncoding("utf-8");
          response.setCharacterEncoding("utf-8");
          response.setContentType("text/html;charset=UTF-8");
  
          System.out.println("doFilter执行前");
          chain.doFilter(request,response); //让请求继续往下走，必须写，如果不写就停止了
          System.out.println("doFilter执行后");
      }
  
      public void destroy() {
          System.out.println("程序摧毁");
      }
  }
  ```

- 创建一个Servlet类：

  ```java
  package com.it.servlet;
  
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  public class Servlet extends HttpServlet {
  
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.getWriter().write("你好呀世界");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  ```

- web.xml：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://java.sun.com/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
           version="3.0"
           metadata-complete="true">
    <servlet>
      <servlet-name>servlet</servlet-name>
      <servlet-class>com.it.servlet.Servlet</servlet-class>
    </servlet>
    <servlet-mapping>
      <servlet-name>servlet</servlet-name >
      <url-pattern>/servlet</url-pattern>
    </servlet-mapping>
    
    <filter>
      <filter-name>filter</filter-name>
      <filter-class>com.it.filter.ServletFilter</filter-class>
    </filter>
    <filter-mapping>
      <filter-name>filter</filter-name>
        <!--/servlet/* 下的任何请求都会经过过滤器    -->
      <url-pattern>/servlet/*</url-pattern>
    </filter-mapping> 
  </web-app>
  ```

  

# 8.small-exercises

- 见idea源码