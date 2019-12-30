## 引入pom

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.2</version>
</dependency>
```

## 编写HttpClient发送请求的步骤

1. 创建HttpClient对象
2. 创建Http[HttpMethod]具体发送http方式对象
3. 设置http方法对象
4. 使用HttpClient执行http方法对象
5. 接受HttpClient方法返回值

## 案例

### 处理url

```java
    /**
     * 构造请求 url
     * @param url 源url 
     * @param param 键值对参数
     * @return 带参数的url
     */
    private static String buildUrl(String url, Map<String, String> param) {
        if (url == null || "".equals(url = url.trim())) {
            throw new IllegalArgumentException("Http request url must not null");
        }
        StringBuilder requestUrl = new StringBuilder(url);
        if (param != null && param.size() > 0) {
            boolean first = true;
            for (Map.Entry<String, String> entry : param.entrySet()) {
                if (first) {
                    // 不存在 ? 添加一个
                    if (!url.contains("?")) {
                        requestUrl.append("?");
                    }
                    if (url.contains("&") && url.indexOf("&") < url.length() - 1) {
                        requestUrl.append("&");
                    }
                    first = !first;
                } else {
                    requestUrl.append("&");
                }
                requestUrl.append(entry.getKey()).append("=").append(entry.getValue());
            }
        }
        return requestUrl.toString();
    }
```

### get 请求

```java
    /**
     * 发送get请求
     * @param url 请求地址
     * @param headers 头部信息
     * @param params 请求参数
     * @return 返回值
     */
    public static String get(String url, Map<String, String> headers, Map<String, String> params) {
        String result = null;
        // http 客户端
        HttpClient httpClient = new DefaultHttpClient();
        try {
            // 执行get请求
            HttpGet httpGet = new HttpGet(buildUrl(url, params));
            if (headers != null && headers.size() > 0) {
                headers.forEach(httpGet::setHeader);
            }
            HttpResponse response = httpClient.execute(httpGet);
            HttpEntity httpEntity = response.getEntity();
            if (httpEntity != null) {
                result = EntityUtils.toString(httpEntity);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            httpClient.getConnectionManager().shutdown();
        }
        return result;
    }
```

### post 请求

```java
    /**
     * 发送get请求
     * @param url 请求地址
     * @param headers 头部信息
     * @param params 请求参数 (普通的键值对参数)
     * @param body 内容体，可以是任何字符串形式
     * @return 返回值
     */
    public static String post(String url, Map<String, String> headers, Map<String, String> params, String body) {
        String result = null;
        // http 客户端
        HttpClient httpClient = new DefaultHttpClient();
        try {
            HttpPost httpPost = new HttpPost(buildUrl(url, params));
            if (headers != null && headers.size() > 0) {
                headers.forEach(httpPost::setHeader);
            }
            if (body != null && !"".equals(body.trim())) {
                HttpEntity httpEntity = new StringEntity(body, Charset.forName("UTF-8"));
                httpPost.setEntity(httpEntity);
            }

            HttpResponse response = httpClient.execute(httpPost);
            HttpEntity httpEntity = response.getEntity();
            if (httpEntity != null) {
                result = EntityUtils.toString(httpEntity);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            httpClient.getConnectionManager().shutdown();
        }

        return result;
    }
```

