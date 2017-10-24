### JSP中解决session超时跳转到登陆页面并跳出iframe框架或局部区域的方法

---

> 摘要: JSP中解决session超时跳转到登陆页面并跳出iframe框架或局部区域的方法

​       当session会话超时，页面请求被重新定位到了登陆界面。因大都采用Ajax动态局部请求，导致返回登陆页面被嵌套在系统界面的局部区域中，并非想要的效果。一般页面主体布局采用iframe框架进行分割，或者简单实用table等实现同样样式效果，在此简单介绍后台页面重新定向到登陆界面返回前台后，前台进行重新再次定向到登陆界面实现登陆界面无暇。

​        在登陆界JSP面增加以下内容：

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"%>
<%
     String path = request.getContextPath();
     String basePath = request.getScheme() + "://" + request.getServerName() + ":" 
                        + request.getServerPort() + path + "/";                        
%>

<script type="text/javascript">
    // 页面初始化，本处使用定时器处理
    // 如果使用onload或者jquery的$(document).read(function(){...});未必能达到效果。因地制宜。
    
    var initScript = setInterval(function(){
    
            // 针对iframe嵌套的情况
            if (window.top!=null && window.top.document.URL!=document.URL){
                clearInterval(initScript);
                window.top.location.href = document.URL; 
	    }
	    
	    // 针对table布局，非iframe情况
	    if(document.getElementById("你页面头区域某个元素Id")){
	        clearInterval(initScript);
		window.top.location.href = "<%=basePath%>你的登陆界面相对路径"; 
	    }
        },400);

</script>
```