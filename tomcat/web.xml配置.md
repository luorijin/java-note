# web.xml配置

> web.xml的加载过程 `context-param` >> `listener` >> `fileter` >> `servlet`

1. 启动一个WEB项目的时候,容器(如:Tomcat)会去读它的配置文件web.xml.读两个节点: `<listener></listener>` 和 `<context-param></context-param>`
2. 紧接着,容器创建一个`ServletContext`(上下文),这个WEB项目所有部分都将共享这个上下文.
3. 容器将`<context-param></context-param>`转化为键值对,并交给ServletContext.
4. 容器创建`<listener></listener`>中的类实例,即创建监听.
5. 在监听中会有`contextInitialized(ServletContextEvent args)`初始化方法,在这个方法中获得`ServletContext = ServletContextEvent.getServletContext();`

## <context-param>配置

1.`<context-param>`配置是是一组键值对，比如：

```xml
 <context-param>
        <param-name>home-page</param-name>
        <param-value>home.jsp</param-value>
    </context-param>
```

param-name是键，相当于就是参数名，param-value是值，相当于参数值

2. 当服务器启动时，服务器会读取web.xml配置，当读到`<listener></listener>`和`<context-param></context-param>`这两个节点的时候，容器会将这两个节点set到ServletContext(上下文对象)中，这样我们在程序中就能通过这个上下文对象去取得我们这个配置值。

具体代码实现：

```java
String sHomePage = getServletContext().getInitParameter("home-page");
```

通过上面这句代码，我们就可以取得web.xml中配置的home.jsp这个值。


`<context-param>`配置和`<init-param>`的区别：

```xml
<servlet>
        <servlet-name>ServletInit</servlet-name>
        <servlet-class>com.sunrain.datalk.wserver.util.servlet.ServletInit</servlet-class>
        <init-param>
             <param-name>home-page</param-name>
             <param-value>home.jsp</param-value>
        </init-param>
  </servlet>
```

1.我们可以看到`<init-param>`是放在一个servlet内的，所以这个参数是只针对某一个servlet而言的

全局变量和和局部变量的区别：`<context-param>`是针对整个项目，所有的servlet都可以取得使用;`<init-param>`servlet范围内的参数，只能在servlet的init()方法中取得

```java
    public class MainServlet extends HttpServlet ...{
        public MainServlet() ...{
            super();
        }
        public void init() throws ServletException ...{
            System.out.println("下面的两个参数param1是在servlet中存放的");
            System.out.println(this.getInitParameter("param1"));
            System.out.println("下面的参数是存放在servletcontext中的");
            System.out.println(getServletContext().getInitParameter("context/param"));
        }
    }
```


## Listener 监听器

### Listener的分类与使用

1. ServletContextListener：用于对Servlet整个上下文进行监听（创建、销毁）
    > spring使用ContextLoaderListener加载ApplicationContext配置信息

    `ContextLoaderListener`的作用就是启动Web容器时，自动装配`ApplicationContext`的配置信息。因为它实现了`ServletContextListener`这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法

    `ContextLoaderListener`如何查找`ApplicationContext.xml`的配置位置以及配置多个xml：如果在`web.xml`中不写任何参数配置信息，默认的路径是`/WEB-INF/applicationContext.xml`，在WEB-INF目录下创建的xml文件的名称必须是`applicationContext.xml`。如果是要自定义文件名可以在web.xml里加入`contextConfigLocation`这个context参数

    ```xml
    <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext-*.xml</param-value><!-- 采用的是通配符方式，查找WEB-INF/spring目录下xml文件。如有多个xml文件，以“,”分隔。 -->
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    ```

    > Spring版本小于5时使用Log4jConfigListener配置Log4j日志,spring5自动加载

2. HttpSessionListener接口：对Session的整体状态的监听
    > 利用HttpSessionListener统计最多在线用户人数
    ```java
    import java.text.DateFormat;
    import java.text.SimpleDateFormat;
    import java.util.Date;
    import javax.servlet.ServletContext;
    import javax.servlet.http.HttpSessionEvent;
    import javax.servlet.http.HttpSessionListener;

    public class HttpSessionListenerImpl implements HttpSessionListener {

        public void sessionCreated(HttpSessionEvent event) {
            ServletContext app = event.getSession().getServletContext();
            int count = Integer.parseInt(app.getAttribute("onLineCount").toString());
            count++;
            app.setAttribute("onLineCount", count);
            int maxOnLineCount = Integer.parseInt(app.getAttribute("maxOnLineCount").toString());
            if (count > maxOnLineCount) {
                //记录最多人数是多少
                app.setAttribute("maxOnLineCount", count);
                DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                //记录在那个时刻达到上限
                app.setAttribute("date", df.format(new Date()));
            }
        }
        //session注销、超时时候调用，停止tomcat不会调用
        public void sessionDestroyed(HttpSessionEvent event) {
            ServletContext app = event.getSession().getServletContext();
            int count = Integer.parseInt(app.getAttribute("onLineCount").toString());
            count--;
            app.setAttribute("onLineCount", count);    
            
        }
    }
    ```
3. ServletRequestListener：用于对Request请求进行监听（创建、销毁）。

## Filter 过滤器

Servlet中的过滤器Filter是实现了javax.servlet.Filter接口的服务器端程序，主要的用途是过滤字符编码、做一些业务逻辑判断等。其工作原理是，只要你在web.xml文件配置好要拦截的客户端请求，它都会帮你拦截到请求，此时你就可以对请求或响应(Request、Response)统一设置编码，简化操作；同时还可进行逻辑判断，如用户是否已经登陆、有没有权限访问该页面等等工作。它是随你的web应用启动而启动的，只初始化一次，以后就可以拦截相关请求，只有当你的web应用停止或重新部署的时候才销毁，以下通过过滤编码的代码示例来了解它的使用： AOP作用

```java
package com.hello.web.listener;  
  
import java.io.IOException;  
  
import javax.servlet.*;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
  
// 主要目的：过滤字符编码；其次，做一些应用逻辑判断等.  
// Filter跟web应用一起启动  
// 当web应用重新启动或销毁时，Filter也被销毁  
public class MyCharsetFilter implements Filter  
{  
    private FilterConfig config = null;  
      
    public void destroy()  
    {  
        System.out.println("MyCharsetFilter准备销毁...");  
    }  
  
    public void doFilter(ServletRequest arg0, ServletResponse arg1,FilterChain chain) throws IOException, ServletException  
    {  
        // 强制类型转换  
        HttpServletRequest request = (HttpServletRequest) arg0;  
        HttpServletResponse response = (HttpServletResponse) arg1;  
        // 获取web.xm设置的编码集，设置到Request、Response中  
        request.setCharacterEncoding(config.getInitParameter("charset"));  
        response.setContentType(config.getInitParameter("contentType"));  
        response.setCharacterEncoding(config.getInitParameter("charset"));  
        // 将请求转发到目的地  
        chain.doFilter(request, response);  
    }  
  
    public void init(FilterConfig arg0) throws ServletException  
    {  
        this.config = arg0;  
        System.out.println("MyCharsetFilter初始化...");  
    }  
}  
```
web.xml配置

```xml
    <filter>  
      <filter-name>filter</filter-name>  
      <filter-class>dc.gz.filters.MyCharsetFilter</filter-class>  
      <init-param>  
          <param-name>charset</param-name>  
          <param-value>UTF-8</param-value>  
      </init-param>  
      <init-param>  
          <param-name>contentType</param-name>  
          <param-value>text/html;charset=UTF-8</param-value>  
      </init-param>  
  </filter>  
  <filter-mapping>  
      <filter-name>filter</filter-name>  
      <!-- * 代表截获所有的请求  或指定请求/test.do  /xxx.do -->  
      <url-pattern>/*</url-pattern>  
  </filter-mapping>  
```


