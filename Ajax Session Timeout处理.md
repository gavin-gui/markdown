### Ajax Session Timeout处理

---

对于Session过期跳转问题，对于一般的请求处理很简单，就是使用一个拦截器或过滤器，当请求过来时判断Session是否过期，过期则跳转，否则继续。

但是对于Ajax请求，需要做特殊处理。Ajax请求的时候，请求头是：X-Requested-With: XMLHttpRequest，所以我们可以根据请求头做特殊处理。处理逻辑如下：

```java
String contextPath = request.getContextPath();
//获取请求头X-Requested-With的值
String requestType = request.getHeader("X-Requested-With");
//请求头为XMLHttpRequest表示为Ajax请求
if(StringUtils.isNotBlank(requestType) && requestType.equals("XMLHttpRequest")) {
  	//设置过期状态码为911
    response.setStatus(911);
  	//设置Session状态为timeout
  	response.addHeader("sessionstatus", "timeout");
  	//设置重新登录路径
  	response.addHeader("loginPath", contextPath + "/login.do")；
} else { //如果不是Ajax请求
    this.forward(request, response, "/login.do");
}
```

页面上的处理，使用了jquery的全局事件处理机制：

```javascript
$(function() {
    $.ajaxSetup({
        contentType: 'application/x-www-form-urlencoded;charset=utf-8',
      	cache: false,
      	complete: function(XHR, TS) {
            var responseText = XHR.responseText;
          	//获取响应头中的sessionstatus,loginPath，以及响应状态码
          	var sessionstatus = XHR.getResponseHeader('sessionstatus');
          	var loginPath = XHR.getResponseHeader('loginPath');
          	var status = XHR.status;
          
          	//当 (status=911 && sessionstatus=timeout)时，表明Session过期
          	if(911 == status && 'timeout' == sessionstatus) {
                layer.alert('你的会话已经过期，请重新登录后继续操作！', function(index) {
                    layer.close(index);
                  	//跳转到登录页面
                  	window.location.replace(loginPath);
                })
            }
        }
    });
});
```

参考：[Ajax Session Timeout处理](lgscofield.iteye.com/blog/2087032)



