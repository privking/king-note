# http

```java
import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

public class HttpUtil {

    /**
     * 日志
     */
    private static final Logger log = LoggerFactory.getLogger(HttpUtils.class);

    /**
     * 编码
     */
    private static final String CHARSET = "UTF-8";

    public static String get(String url) {
        return execute(url);
    }

    public static String get(String url, Map<String, Object> params) {
        url = url + "?";
        for (Iterator<String> iterator = params.keySet().iterator(); iterator.hasNext();) {
            String key = iterator.next();
            String temp = key + "=" + params.get(key) + "&";
            url = url + temp;
        }
        url = url.substring(0, url.length() - 1);
        return execute(url);
    }

    /**
     * http get请求
     */
    private static String execute(String url) {
        try {
            CloseableHttpClient httpClient = HttpClients.createDefault();
            HttpGet httpGet = new HttpGet(url);
            CloseableHttpResponse response = httpClient.execute(httpGet);
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                String str = EntityUtils.toString(entity, CHARSET);
                return str;
            }
        } catch (Exception e) {
            log.error("请求 " + url + "失败:" + e);
        }
        return null;
    }

    /**
     * http post请求
     * @return
     */
    public static String post(String url, Map<String, Object> params) {
        try {
            CloseableHttpClient httpClient = HttpClients.createDefault();
            HttpPost httpPost = new HttpPost(url);
            List<NameValuePair> parameters = new ArrayList<NameValuePair>();
            for (Iterator<String> iterator = params.keySet().iterator(); iterator.hasNext();) {
                String key = iterator.next();
                parameters.add(new BasicNameValuePair(key, params.get(key).toString()));
            }
            UrlEncodedFormEntity uefEntity = new UrlEncodedFormEntity(parameters, CHARSET);
            httpPost.setEntity(uefEntity);
            CloseableHttpResponse response = httpClient.execute(httpPost);
            try {
                HttpEntity entity = response.getEntity();
                if (entity != null) {
                    String str = EntityUtils.toString(entity, CHARSET);
                    return str;
                }
            } finally {
                response.close();
                httpClient.close();
            }
        } catch (Exception e) {
            log.error("请求" + url + "失败:" + e);
            throw new RuntimeException(e);
        }
        return null;
    }

    /**
     * http post请求 json
     * @return
     */
    public static String post(String url, String params) {
        try {
            CloseableHttpClient httpClient = HttpClients.createDefault();
            HttpPost httpPost = new HttpPost(url);
            StringEntity sEntity = new StringEntity(params, CHARSET);
            httpPost.setEntity(sEntity);
            httpPost.setHeader("Content-Type","application/json;charset=utf-8");
            CloseableHttpResponse response = httpClient.execute(httpPost);
            try {
                HttpEntity entity = response.getEntity();
                if (entity != null) {
                    return EntityUtils.toString(entity, CHARSET);
                }
            } finally {
                response.close();
                httpClient.close();
            }
        } catch (Exception e) {
            log.error("请求" + url + "失败:" + e);
            throw new RuntimeException(e);
        }
        return null;
    }
}
```

