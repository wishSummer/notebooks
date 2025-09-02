# Http发送请求

* http、get 请求、post 请求

```java
/**
 * wesoft.com Inc.
 * Copyright (c) 2005-2016 All Rights Reserved.
 */
package com.wesoft.util;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import javax.validation.constraints.NotNull;

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
import com.fasterxml.jackson.databind.ObjectMapper;

public final class HttpRequest {

    private static ObjectMapper om = new ObjectMapper();

    /**
     * 通过GET方式发起http请求
     */
    public static String sendGet(String url) throws IOException {
        CloseableHttpClient httpClient = getHttpClient();
        try {
            HttpGet get = new HttpGet(url);
            CloseableHttpResponse httpResponse;
            httpResponse = httpClient.execute(get);
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            } else {
                return null;
            }
        } finally {
            closeHttpClient(httpClient);
        }
    }

    public static String sendPost(String url, @NotNull Map<String, String> parameters) throws IOException {
        return sendPost(url, parameters, "UTF-8");
    }

    /**
     *
     * @param url
     * @param parameters
     * @param code 编码方式
     * @return
     */
    public static String sendPost(String url, @NotNull Map<String, String> parameters,
                                  String code) throws IOException {
        CloseableHttpClient httpClient = getHttpClient();
        try {
            HttpPost post = new HttpPost(url);
            List<NameValuePair> list = new ArrayList<>();

            parameters.forEach((key, val) -> list.add(new BasicNameValuePair(key, val)));

            UrlEncodedFormEntity uefEntity = new UrlEncodedFormEntity(list, code);
            post.setEntity(uefEntity);

            CloseableHttpResponse httpResponse = httpClient.execute(post);
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            } else {
                return null;
            }

        } finally {
            closeHttpClient(httpClient);
        }

    }

    public static String sendPostByJson(String url,
                                        @NotNull Map<String, String> parameters) throws IOException {
        CloseableHttpClient httpClient = getHttpClient();
        HttpPost httpPost = new HttpPost(url);
        StringEntity stringEntity = new StringEntity(om.writeValueAsString(parameters),
            StandardCharsets.UTF_8);

        httpPost.setHeader("content-type", "application/json;charset=utf-8");
        httpPost.setEntity(stringEntity);

        try (CloseableHttpResponse httpResponse = httpClient.execute(httpPost);) {
            HttpEntity entity = httpResponse.getEntity();
            if (entity != null) {
                return EntityUtils.toString(entity);
            } else {
                return null;
            }
        } finally {
            closeHttpClient(httpClient);
        }

    }

    private static CloseableHttpClient getHttpClient() {
        return HttpClients.createDefault();
    }

    private static void closeHttpClient(CloseableHttpClient client) throws IOException {
        if (client != null) {
            client.close();
        }
    }

}

```

* http post 请求附带文件

```java

import java.io.File;
import java.io.IOException;
import java.util.Map;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.wesoft.cloud.platform.publish.infra.jpa.domain.ResponseData;

import okhttp3.*;

public class FileUploadUtil {

    private static final ObjectMapper objectMapper = new ObjectMapper();

    private static final OkHttpClient client       = new OkHttpClient();

    /**
     * @param url     请求地址
     * @param headers headers
     * @param file    文件
     * @return response
     * @throws IOException
     */
    public static ResponseData uploadFile(String url, File file,
                                          Map<String, String> headers) throws Exception {

        // Create a request body with the file and other parameters
        MultipartBody.Builder builder = new MultipartBody.Builder().setType(MultipartBody.FORM)
            .addFormDataPart("file", file.getName(),
                RequestBody.create(file, MediaType.parse("multipart/form-data")));

        RequestBody requestBody = builder.build();

        // Build the request
        Request.Builder requestBuilder = new Request.Builder().url(url).post(requestBody);

        // Add headers if needed
        if (headers != null) {
            for (Map.Entry<String, String> entry : headers.entrySet()) {
                requestBuilder.addHeader(entry.getKey(), entry.getValue());
            }
        }

        Request request = requestBuilder.build();

        // Execute the request and get the response
        try (Response response = client.newCall(request).execute()) {
            if (!response.isSuccessful()) {
                throw new IOException("Unexpected code " + response);
            }

            String res = response.body().string();
            ResponseData responseData = objectMapper.readValue(res, ResponseData.class);

            return responseData;
        } catch (Exception e) {
            throw new IOException(e);
        }
    }
}

```
