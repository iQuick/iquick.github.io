---
layout:		post
title:		RequestManager
category:	[Android]
tags:		[Android, Volley]
published:	true
---
# 基于Volley的网络请求的简单封闭
---

最近写一直在用Google的Volley做网络请求部分，但是Volley源码只有简单的通信，如果需要上传文件、图片时就需要时就稍微复杂了，于是自己就对Volley进入了简单的封装。

## 简介
本例基于Volley库扩展，RequestManager类管理应用请求， ImageCacheManager类加载图片及图片缓存管理

结构如下：

![结构](image/volley-manager-structure.jpg)

<!--break-->

## 核心类简介
### 1.RequestManager.java
核心网络请求管理，主要用用于管理应用的发送请求及取消请求
<pre class="prettyprint linenums">
import java.io.UnsupportedEncodingException;
import cn.uue.yixiaoba.App;
import com.android.volley.Request;
import com.android.volley.Request.Method;
import com.android.volley.RequestQueue;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.Volley;

/**
 * 
 * @author Doots
 *
 */
public class RequestManager {
	
    public static RequestQueue mRequestQueue = Volley.newRequestQueue(App.getContext());

    private RequestManager() {
        // no instances
    }

    /**
     * 
     * @param <T>
     * @param url
     * @param tag
     * @param listener
     */
    public static void get(String url, Object tag, RequestListener listener) {
    	get(url, tag, null, listener);
    }

    /**
     * 
     * @param url
     * @param tag
     * @param params
     * @param listener
     */
    public static void get(String url, Object tag, RequestParams params, RequestListener listener) {
    	ByteArrayRequest request = new ByteArrayRequest(Method.GET, url, null, responseListener(listener), responseError(listener));
    	addRequest(request , tag);
    }

    /**
     * 
     * @param url
     * @param tag
     * @param listener
     */
    public static void post(String url, Object tag, RequestListener listener) {
    	post(url, tag, null, listener);
    }
    
    /**
     * 
     * @param url
     * @param tag
     * @param params
     * @param listener
     */
    public static void post(String url, Object tag, RequestParams params, RequestListener listener) {
    	ByteArrayRequest request = new ByteArrayRequest(Method.POST, url, params, responseListener(listener), responseError(listener));
    	addRequest(request , tag);
    }
    
    public static void addRequest(Request<?> request, Object tag) {
        if (tag != null) {
            request.setTag(tag);
        }
        mRequestQueue.add(request);
    }

    public static void cancelAll(Object tag) {
        mRequestQueue.cancelAll(tag);
    }
    
    
    /**
     * 成功消息监听
     * @param l
     * @return
     */
    protected static Response.Listener<byte[]> responseListener(final RequestListener l) {
    	return new Response.Listener<byte[]>() {
			@Override
			public void onResponse(byte[] arg0) {
				String data = null;
				try {
					data = new String(arg0, "UTF-8");
				} catch (UnsupportedEncodingException e) {
					e.printStackTrace();
				}
				l.requestSuccess(data);
			}
		};
    }
    
    /**
     * 返回错误监听
     * @param l
     * @return
     */
    protected static Response.ErrorListener responseError(final RequestListener l) {
    	return new Response.ErrorListener() {

			@Override
			public void onErrorResponse(VolleyError e) {
				l.requestError(e);
			}
		};
    }
}
</pre>

### 2.RequestParams.java
请求参数，装的请求参数类，用法与Map相似
<pre class="prettyprint linenums">
/**
 * @param key
 * @param value
 */
public void put(String key, String value) {
	if (key != null && value != null) {
		urlParams.put(key, value);
	}
}

/**
 * @param key
 * @param file
 */
public void put(String key, File file) {
	try {
		put(key, new FileInputStream(file), file.getName());
	} catch (FileNotFoundException e) {
		e.printStackTrace();
	}
}

/**
 * @param key
 * @param stream
 * @param fileName
 */
public void put(String key, InputStream stream, String fileName) {
	put(key, stream, fileName, null);
}

/**
 * @param key
 * @param stream
 * @param fileName
 * @param contentType
 */
public void put(String key, InputStream stream, String fileName, String contentType) {
	if (key != null && stream != null) {
		fileParams.put(key, new FileWrapper(stream, fileName, contentType));
	}
}
</pre>

### 3.RequestListener.java
请求监听接口
<pre class="prettyprint linenums">
public interface RequestListener {

	/** 成功 */
	public void requestSuccess(String json);

	/** 错误 */
	public void requestError(VolleyError e);
}
</pre>

### 4.ByteArrayRequest.java
重写了Request类的头部获取及参数获取部分
<pre class="prettyprint linenums">
@SuppressWarnings("unchecked")
protected Map<String, String> getParams() throws AuthFailureError {
	if (this.httpEntity == null && this.mPostBody != null && this.mPostBody instanceof Map<?, ?>) {
		return ((Map<String, String>) this.mPostBody);//common Map<String, String>
	}
	return null;//process as json, xml or MultipartRequestParams
}

@Override
public Map<String, String> getHeaders() throws AuthFailureError {
	Map<String, String> headers = super.getHeaders();
	if (null == headers || headers.equals(Collections.emptyMap())) {
		headers = new HashMap<String, String>();
	}
	return headers;
}

@Override
public String getBodyContentType() {
	if (httpEntity != null) {
		return httpEntity.getContentType().getValue();
	}
	return null;
}

@Override
public byte[] getBody() throws AuthFailureError {
	if (this.mPostBody != null && this.mPostBody instanceof String) {//process as json or xml
		String postString = (String) mPostBody;
		if (postString.length() != 0) {
			try {
				return postString.getBytes("UTF-8");
			} catch (UnsupportedEncodingException e) {
				e.printStackTrace();
			}
		} else {
			return null;
		}
	}
	if (this.httpEntity != null) {//process as MultipartRequestParams
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
			try {
				httpEntity.writeTo(baos);
			} catch (IOException e) {
				e.printStackTrace();
				return null;
			}
		return baos.toByteArray();
	}
	return super.getBody();// mPostBody is null or Map<String, String>
}
</pre>


Github：[https://github.com/iQuick/VolleyManager](https://github.com/iQuick/VolleyManager)
