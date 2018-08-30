---
layout: post
title: Spring MVC 拦截器实现日志监控 
date: 2018-8-30 15:30:24.000000000 +09:00
tag: Spring
---

## 实现HandlerInterceptor接口
&emsp;&emsp;HandlerInterceptor 接口中定义了三个方法（`preHandle`、`postHandle`、`afterCompletion`），通过这三个方法对用户的请求进行拦截处理和服务端返回数据处理。

### preHandle
>&emsp;&emsp;`preHandle (HttpServletRequest request, HttpServletResponse response, Object handle)` 方法，该方法将在请求处理之前进行调用,返回值为`boolean`类型，只有当方法返回`true`时才会继续执行，否则取消当前请求。

>&emsp;&emsp;SpringMVC 中的*Interceptor* 是链式的调用的，在一个应用中或者说是在一个请求中可以同时存在多个*Interceptor* 。每个*Interceptor* 的调用会依据它的声明顺序依次执行，而且最先执行的都是*Interceptor* 中的`preHandle` 方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值是布尔值Boolean 类型的，当它返回为`false` 时，表示请求结束，后续的*Interceptor* 和*Controller*都不会再执行；当返回值为`true` 时就会继续调用下一个*Interceptor* 的`preHandle` 方法，如果已经是最后一个*Interceptor* 的时候就会是调用当前请求的*Controller* 方法。

### postHandle
>&emsp;&emsp;`postHandle (HttpServletRequest request, HttpServletResponse response, Object handle, ModelAndView modelAndView)` 方法，由`preHandle` 方法的解释我们知道这个方法包括后面要说到的`afterCompletion` *Interceptor* 的`preHandle` 方法的返回值为`true` 时才能被调用。

>&emsp;&emsp;`postHandle`在当前请求进行处理之后，也就是*Controller* 方法调用之后执行，但是它会在*DispatcherServlet* 进行视图返回渲染之前被调用，所以我们可以在这个方法中对*Controller* 处理之后的*ModelAndView* 对象进行操作。`postHandle` 方法被调用的方向跟`preHandle` 是相反的，也就是说先声明的*Interceptor* 的`postHandle` 方法反而会后执行。

### afterCompletion
>&emsp;&emsp;`afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex)` 方法，该方法也是需要当前对应的*Interceptor* 的`preHandle` 方法的返回值为`true` 时才会执行。

>&emsp;&emsp;`afterCompletion`方法将在整个请求结束之后，也就是在*DispatcherServlet* 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。

### example code

{% highlight java %}
import java.io.PrintWriter;
import java.io.StringWriter;
import java.lang.reflect.Field;
import java.text.SimpleDateFormat;
import java.util.Arrays;
import java.util.Map;
import java.util.Map.Entry;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.catalina.connector.CoyoteOutputStream;
import org.apache.catalina.connector.OutputBuffer;
import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.apache.tomcat.util.buf.ByteChunk;
import org.springframework.core.NamedThreadLocal;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class LogInterceptor implements HandlerInterceptor {

	private Logger logger = Logger.getLogger(LogInterceptor.class);

	private static final ThreadLocal<Long> startTimeThreadLocal = new NamedThreadLocal<Long>("ThreadLocal StartTime");

	/**
	 * getParamString:get param string. <br/>
	 * 
	 * @author idaidai 
	 * @ate:Aug 30, 201812:19:24 PM 
	 * @return String
	 */
	private String getParamString(Map<String, String[]> map) {
		StringBuilder sb = new StringBuilder();
		for (Entry<String, String[]> e : map.entrySet()) {
			sb.append(e.getKey()).append("=");
			String[] value = e.getValue();
			if (value != null && value.length == 1) {
				sb.append(value[0]).append("\t");
			} else {
				sb.append(Arrays.toString(value)).append("\t");
			}
		}
		return sb.toString();
	}

	/**
	 * getStackTraceAsString:将ErrorStack转化为String. <br/>
	 * 
	 * @author idaidai 
	 * @date:Aug 30, 201812:19:43 PM 
	 * @return String
	 */
	public static String getStackTraceAsString(Throwable e) {
		if (e == null) {
			return "";
		}
		StringWriter stringWriter = new StringWriter();
		e.printStackTrace(new PrintWriter(stringWriter));
		return stringWriter.toString();
	}

	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
		logger.setLevel(Level.DEBUG);
        if (logger.isDebugEnabled()){
        	
            long beginTime = startTimeThreadLocal.get();//得到线程绑定的局部变量（开始时间）
            long endTime = System.currentTimeMillis(); 	//2、结束时间
 
            //如果controller报错，则记录异常错误
            if(ex != null){
            	logger.info("Controller异常: " + getStackTraceAsString(ex));
            }
 
            logger.info("计时结束："+ new SimpleDateFormat("hh:mm:ss.SSS").format(endTime) + " 耗时：" + (endTime - beginTime) + "ms URI:" +
                    request.getRequestURL()+ " 最大内存: " +Runtime.getRuntime().maxMemory()/1024/1024+ "m  已分配内存: " +Runtime.getRuntime().totalMemory()/1024/1024+ "m  已分配内存中的剩余空间: " +Runtime.getRuntime().freeMemory()/1024/1024+ "m  最大可用内存: " +
                    (Runtime.getRuntime().maxMemory()-Runtime.getRuntime().totalMemory()+Runtime.getRuntime().freeMemory())/1024/1024 + "m");
            startTimeThreadLocal.remove();
        }
	}

	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
		long startTime = (Long) request.getAttribute("startTime");
        long endTime = System.currentTimeMillis();
        long executeTime = endTime - startTime;
        if(handler instanceof HandlerMethod){
            StringBuilder sb = new StringBuilder(1000);
            sb.append("CostTime  : ").append(executeTime).append("ms").append("\n");
            sb.append("-------------------------------------------------------------------------------");
            logger.info(sb.toString());
            
            if (modelAndView == null) {
            	String returnVal = getReturnValueFromStream((CoyoteOutputStream) response.getOutputStream());
            	logger.info("返回值：" + returnVal);
			}
        }
	}

	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		long startTime = System.currentTimeMillis();
		request.setAttribute("startTime", startTime);
		startTimeThreadLocal.set(startTime); // 线程绑定变量（该数据只有当前请求的线程可见）
		if (handler instanceof HandlerMethod) {
			StringBuilder sb = new StringBuilder(1000);
			sb.append("-----------------------开始计时:").append(new SimpleDateFormat("hh:mm:ss.SSS").format(startTime))
					.append("-------------------------------------\n");
			HandlerMethod h = (HandlerMethod) handler;
			sb.append("Controller: ").append(h.getBean().getClass().getName()).append("\n");
			sb.append("Method    : ").append(h.getMethod().getName()).append("\n");
			sb.append("Params    : ").append(getParamString(request.getParameterMap())).append("\n");
			sb.append("URI       : ").append(request.getRequestURL()).append("\n");
			logger.info(sb.toString());
		}
		return true;
	}
}
{% endhighlight %}

## 读取HttpServletResponse流
>&emsp;&emsp;使用拦截器来做日志监控会遇到一个问题，如何把接口返回的数据取出来保存。如果返回的是`ModelAndView`类型，可以直接在`postHandle`方法中操作`ModelAndView`对象，但如果返回的是`@ResponseBody`,则需要通过反射读取HttpServletResponse输出至客户端流的方式取出接口所返回的数据。

![debug](/assets/images/2018/2018-08-30-01.png)

>&emsp;&emsp;通过调试我们可以看到`response`里面是有我们想取的返回值，并没有网上讲的那么麻烦通过实现别的接口去截取。


### key code
{% highlight Java %}
    String returnStr = "";

    // 截取响应流
    CoyoteOutputStream os = (CoyoteOutputStream) response.getOutputStream();
    // 取到流对象对应的Class对象
    Class<CoyoteOutputStream> c = CoyoteOutputStream.class;
    // 取出流对象中的OutputBuffer对象，该对象记录响应到客户端的内容
    Field fs = c.getDeclaredField("ob");
    if (fs.getType().toString().endsWith("OutputBuffer")) {
        fs.setAccessible(true);// 设置访问ob属性的权限
        OutputBuffer ob = (OutputBuffer) fs.get(os);// 取出ob
        Class<OutputBuffer> cc = OutputBuffer.class;
        Field ff = cc.getDeclaredField("outputChunk");// 取到OutputBuffer中的输出流
        ff.setAccessible(true);
        if (ff.getType().toString().endsWith("ByteChunk")) {
            ByteChunk bc = (ByteChunk) ff.get(ob);// 取到byte流
            returnStr = new String(bc.toString().getBytes(bc.getCharset()), "UTF-8");// 最终的值
        }
    }
{% endhighlight %}

## 附件 (完整代码)

>[LogInterceptor.java](https://pan.baidu.com/s/1-aT_eT6jNZ99W1O_Ji2XbA)