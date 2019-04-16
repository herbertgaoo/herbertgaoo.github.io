---
layout: post
title: HttpClient Post请求服务端获取不到参数
date: 2019-04-16 17:30:24.000000000 +09:00
tag: httpclient post
---

&emsp;&emsp;使用HttpClient发送post请求时会遇到服务端收不到参数的情况，查找了一下原因是因为大部分使用Httpclient模拟post请求时都会把参数放在Entity里面，示例如下：

``` java
// 创建Http Post请求
HttpPost httpPost = new HttpPost(url);
// 创建参数列表
if (param != null) {
    List<NameValuePair> paramList = new ArrayList<>();
    for (String key : param.keySet()) {
        paramList.add(new BasicNameValuePair(key, param.get(key)));
    }
    // 模拟表单
    UrlEncodedFormEntity entity = new UrlEncodedFormEntity(paramList);
    httpPost.setEntity(entity);
}
// 执行http请求
response = httpClient.execute(httpPost);
resultString = EntityUtils.toString(response.getEntity(),"utf-8");
```
&emsp;&emsp;上面的这种设置参数的方式通过`request.getParameter("key")`的方式获取不到参数，所以如果想通过`request.getParameter("key")`获取参数，则需要修改HttpClient模拟post请求时设置参数的方式，示例如下：

``` java
private static HttpPost postForm(String url, Map<String, String> params) {
		URIBuilder builder;
		HttpPost httpost = null;
		try {
			builder = new URIBuilder(url);
			
			List<NameValuePair> nvps = new ArrayList <NameValuePair>();
			Set<String> keySet = params.keySet();
			for(String key : keySet) {
				nvps.add(new BasicNameValuePair(key, params.get(key)));
			}
			
			builder.setCharset(Consts.UTF_8);
			builder.setParameters(nvps);
			httpost = new HttpPost(builder.build());
		} catch (URISyntaxException e) {
			e.printStackTrace();
		}
		return httpost;	
	}
```
&emsp;&emsp;使用`URIBuilder`将参数加到parameters里面，这样就可以通过`request.getParameter("key")`的方式获取到参数，或者是Spring框架里面直接使用实体类来接收参数。

&emsp;&emsp;最后附上完整的HttpUtils类

``` java
package com.jeeplus.modules.tools.utils;


import java.io.IOException;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.apache.http.Consts;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.ParseException;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.methods.HttpUriRequest;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.entity.ContentType;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.apache.log4j.Logger;

/**
 * ClassName: HttpPostTest <br/> 
 * Function: update CloseableHttpClient <br/> 
 * date: Mar 20, 2019 11:09:00 AM <br/> 
 * 
 * @author idaidai 
 * @version 2.0
 * @since JDK 1.8
 */
public class HttpPostTest {
	private static Logger log = Logger.getLogger(HttpPostTest.class);
	Map<String, String> params;
	String url;
	static String code;
	public static String post(String url, Map<String, String> params) {
		CloseableHttpClient httpclient = HttpClientBuilder.create().build();
		String body = null;
		
		log.info("create httppost:" + url);
		HttpPost post = postForm(url, params);
		
		body = invoke(httpclient, post);
		
		try {
			httpclient.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		return body;
	}
	
	 public HttpPostTest(String url, 	Map<String, String> params){
		this.url = url;
		this.params = params;
	}
	public static String get(String url) {
		CloseableHttpClient httpclient = HttpClientBuilder.create().build();
		String body = null;
		
		log.info("create httppost:" + url);
		HttpGet get = new HttpGet(url);
		body = invoke(httpclient, get);
		
		try {
			httpclient.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		return body;
	}
		
	
	private static String invoke(CloseableHttpClient httpclient,
			HttpUriRequest httpost) {
		
		HttpResponse response = sendRequest(httpclient, httpost);
		String body = paseResponse(response);
		
		return body;
	}

	private static String paseResponse(HttpResponse response) {
		log.info("get response from http server..");
		HttpEntity entity = response.getEntity();
		
		log.info("response status: " + response.getStatusLine());
		String charset = ContentType.get(entity).getCharset().name();
		log.info(charset);
		
		String body = null;
		try {
			body = EntityUtils.toString(entity);
			log.info(body);
		} catch (ParseException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		code = "response status: " + response.getStatusLine() +"\n";
		
		return body;
	}

	private static HttpResponse sendRequest(CloseableHttpClient httpclient,
			HttpUriRequest httpost) {
		log.info("execute post...");
		HttpResponse response = null;
		
		try {
			response = httpclient.execute(httpost);
		} catch (ClientProtocolException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		return response;
	}

	private static HttpPost postForm(String url, Map<String, String> params) {
		URIBuilder builder;
		HttpPost httpost = null;
		try {
			builder = new URIBuilder(url);
			
			List<NameValuePair> nvps = new ArrayList <NameValuePair>();
			Set<String> keySet = params.keySet();
			for(String key : keySet) {
				nvps.add(new BasicNameValuePair(key, params.get(key)));
			}
			
			builder.setCharset(Consts.UTF_8);
			builder.setParameters(nvps);
			httpost = new HttpPost(builder.build());
		} catch (URISyntaxException e) {
			e.printStackTrace();
		}
		return httpost;
		
	}
	
	public  String postWithCode(){
		String xml = HttpPostTest.post(url, params);
	  return code + xml;
	}
}

```

