### springmvc控制登录用户session失效后跳转登录页面

---

本篇文章主要介绍了springmvc控制登录用户session失效后跳转登录页面，session一旦失效就需要重新登陆，有兴趣的同学可以了解一下。

springmvc控制登录用户session失效后跳转登录页面，废话不多少了，具体如下：

第一步，配置 web.xml

```xml
 <session-config> 
  <session-timeout>15</session-timeout> 
 </session-config> 
```

第二步，配置spring-mvc.xml

```xml
<!-- Session失效拦截 --> 
  <mvc:interceptors> 
    <!-- 定义拦截器 --> 
     <mvc:interceptor>   
        <!-- 匹配的是url路径， 如果不配置或/**,将拦截所有的Controller -->  
        <mvc:mapping path="/**" />  
        <!-- 不需要拦截的地址 --> 
        <mvc:exclude-mapping path="/login.do" /> 
        <bean class="com.cm.contract.controller.annotation.GEISSSessionTimeoutInterceptor">		    
       </bean>   
    </mvc:interceptor> 
  </mvc:interceptors> 
```

第三步，写拦截器SystemSessionInterceptor 方法

```java
public class SystemSessionInterceptor implements HandlerInterceptor { 
  private static final String LOGIN_URL="/jsp/sessionrun.jsp"; 
  @Override 
  public void postHandle(HttpServletRequest request, 
      HttpServletResponse response, Object handler, 
      ModelAndView modelAndView) throws Exception { 
     
 
  } 
 
  @Override 
  public void afterCompletion(HttpServletRequest request, 
      HttpServletResponse response, Object handler, Exception ex) 
      throws Exception { 
 
  } 
 
  @Override 
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
      Object handler) throws Exception { 
    HttpSession session=request.getSession(true); 
    //session中获取用户名信息 
    Object obj = session.getAttribute(CMConstant.LOGINUSER); 
    if (obj==null||"".equals(obj.toString())) { 
      response.sendRedirect(request.getSession().getServletContext().getContextPath()+LOGIN_URL;
         return false;
      }
      return true;
   }
```

第五步，配置友情提示页面sessionrun.jsp

```html
<body>      
  <SCRIPT language="JavaScript"> 
    alert("用户已在其他地方登陆，请重新登录。"); 
    setTimeout(function () { 
      window.top.location.href="<%=path%>/index.jsp"; 
    },2000); 
  </script> 
  </body> 
```

到此 springMvc拦截session失效后处理方式结束。