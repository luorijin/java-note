# spring拦截器

`Spring MVC`中的拦截器`Interceptor）`类似于`Servlet`中的过滤器`Filter`，它主要用于拦截用户请求并作相应的处理。例如通过拦截器可以进行权限验证、记录请求信息的日志、判断用户是否登录等。

要使用Spring MVC中的拦截器，就需要对拦截器类进行定义和配置。通常拦截器类可以通过两种方式来定义。

1. 通过实现HandlerInterceptor接口，或继承`HandlerInterceptor`接口的实现类（如HandlerInterceptorAdapter）来定义。

2. 通过实现WebRequestInterceptor接口，或继承`WebRequestInterceptor`接口的实现类来定义。

```java
package com.luo.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.log4j.Logger;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import com.luo.entity.UserAdmin;

public class LoginInterceptor implements HandlerInterceptor {
	static Logger log = Logger.getLogger(LoginInterceptor.class);
    //重写了HandlerInterceptor的接口，三个方法，这里只用preHandle()方法，
    //preHandle()方法，boolean布尔类型，false表示请求结束，true代表继续执行（如果是最后一个拦截器那么就会调用当前controller的方法
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取session，有就是说明已经登录，没有就是拦截访问
        HttpSession session = request.getSession();
        UserAdmin useradmin = (UserAdmin) session.getAttribute("useradmin");
        if (useradmin != null){
        	log.info("你已登录");
            return true;
        }
        log.info("你未登录");

        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

上述代码中，自定义拦截器实现了HandlerInterceptor接口，并实现了接口中的三个方法：

1. preHandle() 方法：该方法会在控制器方法前执行，其返回值表示是否中断后续操作。当其返回值为true时，表示继续向下执行；当其返回值为false时，会中断后续的所有操作（包括调用下一个拦截器和控制器类中的方法执行等）。

2. postHandle()方法：该方法会在控制器方法调用之后，且解析视图之前执行。可以通过此方法对请求域中的模型和视图做出进一步的修改。

3. afterCompletion()方法：该方法会在整个请求完成，即视图渲染结束之后执行。可以通过此方法实现一些资源清理、记录日志信息等工作。

## 拦截器的配置

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/login"/>
        <!-- 用户是否已经登录的检查bean -->
        <bean class="com.luo.controller.LoginInterceptor"/>
            
    </mvc:interceptor>
</mvc:interceptors>
```