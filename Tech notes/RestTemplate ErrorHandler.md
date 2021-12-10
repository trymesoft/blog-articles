Posted on 2018-12-04
> 在使用 Spring Boot 的 RestTemplate 调用时，发现调用成功时很好用，但是服务器返回 4xx 或者 5xx 异常时，这货直接给抛出来了，想要处理可以自定义 ErrorHandler。

### 问题
> 当服务端返回非 200 时，调用方直接抛出如下异常：

```java
org.springframework.web.client.HttpClientErrorException: 400 Bad Request

	at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:94)
	at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:79)
	at org.springframework.web.client.ResponseErrorHandler.handleError(ResponseErrorHandler.java:63)
	at org.springframework.web.client.RestTemplate.handleResponse(RestTemplate.java:766)
	at org.springframework.web.client.RestTemplate.doExecute(RestTemplate.java:724)
	at org.springframework.web.client.RestTemplate.execute(RestTemplate.java:680)
	at org.springframework.web.client.RestTemplate.postForEntity(RestTemplate.java:466)
    ...
```
为什么会抛异常呢，可以翻一下源码：
RestTemplate 中 postForEntity()方法：
```java
public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables) throws RestClientException {
        RequestCallback requestCallback = this.httpEntityCallback(request, responseType);
        ResponseExtractor<ResponseEntity<T>> responseExtractor = this.responseEntityExtractor(responseType);
        return (ResponseEntity)nonNull(this.execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables));
    }
```
进入 execute()方法：
```java
@Nullable
    public <T> T execute(String url, HttpMethod method, @Nullable RequestCallback requestCallback, @Nullable ResponseExtractor<T> responseExtractor, Object... uriVariables) throws RestClientException {
        URI expanded = this.getUriTemplateHandler().expand(url, uriVariables);
        return this.doExecute(expanded, method, requestCallback, responseExtractor);
    }
```
进入 doExecute()方法：
```java
@Nullable
    protected <T> T doExecute(URI url, @Nullable HttpMethod method, @Nullable RequestCallback requestCallback, @Nullable ResponseExtractor<T> responseExtractor) throws RestClientException {
        Assert.notNull(url, "URI is required");
        Assert.notNull(method, "HttpMethod is required");
        ClientHttpResponse response = null;

        Object var14;
        try {
            ClientHttpRequest request = this.createRequest(url, method);
            if (requestCallback != null) {
                requestCallback.doWithRequest(request);
            }

            response = request.execute();
            this.handleResponse(url, method, response);
            var14 = responseExtractor != null ? responseExtractor.extractData(response) : null;
        } catch (IOException var12) {
            String resource = url.toString();
            String query = url.getRawQuery();
            resource = query != null ? resource.substring(0, resource.indexOf(63)) : resource;
            throw new ResourceAccessException("I/O error on " + method.name() + " request for \"" + resource + "\": " + var12.getMessage(), var12);
        } finally {
            if (response != null) {
                response.close();
            }

        }

        return var14;
    }
```
进入 handleResponse()方法：
```java
protected void handleResponse(URI url, HttpMethod method, ClientHttpResponse response) throws IOException {
        ResponseErrorHandler errorHandler = this.getErrorHandler();
        boolean hasError = errorHandler.hasError(response);
        if (this.logger.isDebugEnabled()) {
            try {
                this.logger.debug(method.name() + " request for \"" + url + "\" resulted in " + response.getRawStatusCode() + " (" + response.getStatusText() + ")" + (hasError ? "; invoking error handler" : ""));
            } catch (IOException var7) {
            }
        }

        if (hasError) {
            errorHandler.handleError(url, method, response);
        }

    }
```
终于看到了想要的，可以看出 hasError 为 true，即有错误时，会走 errorHandler.handleError(url, method, response)，接下来继续查看这个 errorHandler 的源码：
```java
public interface ResponseErrorHandler {
    boolean hasError(ClientHttpResponse var1) throws IOException;

    void handleError(ClientHttpResponse var1) throws IOException;

    default void handleError(URI url, HttpMethod method, ClientHttpResponse response) throws IOException {
        this.handleError(response);
    }
}
```
这是一个接口，我们点进它的实现类 DefaultResponseErrorHandler 的 handleError()
```java
public void handleError(ClientHttpResponse response) throws IOException {
        HttpStatus statusCode = HttpStatus.resolve(response.getRawStatusCode());
        if (statusCode == null) {
            throw new UnknownHttpStatusCodeException(response.getRawStatusCode(), response.getStatusText(), response.getHeaders(), this.getResponseBody(response), this.getCharset(response));
        } else {
            this.handleError(response, statusCode);
        }
    }
```
继续往下：
```java
protected void handleError(ClientHttpResponse response, HttpStatus statusCode) throws IOException {
        switch(statusCode.series()) {
        case CLIENT_ERROR:
            throw new HttpClientErrorException(statusCode, response.getStatusText(), response.getHeaders(), this.getResponseBody(response), this.getCharset(response));
        case SERVER_ERROR:
            throw new HttpServerErrorException(statusCode, response.getStatusText(), response.getHeaders(), this.getResponseBody(response), this.getCharset(response));
        default:
            throw new UnknownHttpStatusCodeException(statusCode.value(), response.getStatusText(), response.getHeaders(), this.getResponseBody(response), this.getCharset(response));
        }
    }
```
可算找到原因了，原来在这里抛出了一个 HttpClientErrorException，既然 DefaultResponseErrorHandler实现了ResponseErrorHandler，而且从命名上看貌似是个默认的错误处理，那我们可以尝试自定义 ErrorHandler 实现 ResponseErrorHandler 来处理异常情况。

### 自定义 ErrorHandler
```java
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.web.client.DefaultResponseErrorHandler;

import java.io.IOException;

/**
 * @author tryme.wang
 * @create 2018-12-03 11:10
 * @description 远程调用自定义异常Handler
 **/
public class RestErrorHandler extends DefaultResponseErrorHandler {

    @Override
    public boolean hasError(ClientHttpResponse response) throws IOException {
        // 是否有错 返回false即手动设置了不管response是什么都没有错
        return false;
    }

    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        // 暂时空着
    }
}
```
别忘记配置进去：
```java
import com.merrichat.im.common.handler.RestErrorHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

/**
 * @author tryme.wang
 * @create 2018-11-30 10:59:48
 * @desc http请求配置 （替代httpclient）
 **/
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
        RestTemplate restTemplate = new RestTemplate(factory);
        // 使用自定义的ErrorHandler
        restTemplate.setErrorHandler(new RestErrorHandler());
        return restTemplate;
    }

    @Bean
    public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        // 单位：ms
        factory.setReadTimeout(5000);
        // 单位：ms
        factory.setConnectTimeout(15000);
        return factory;
    }
}
```
这样无论服务端返回什么状态码，客户端都会认为没错，我们就可以获取想要的 statusCode 或者 body 进而来处理各种业务了。