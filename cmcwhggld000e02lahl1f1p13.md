---
title: "Spring Boot API Gateway with Centralized Exception Handling and Unified Response Structure"
seoTitle: "Spring Boot API Gateway | Centralized Error Handling & Unified Respons"
seoDescription: "Learn how to build a robust API Gateway using Spring Boot with centralized exception handling and unified response formatting across all microservices."
datePublished: Wed Jul 09 2025 21:40:37 GMT+0000 (Coordinated Universal Time)
cuid: cmcwhggld000e02lahl1f1p13
slug: spring-boot-api-gateway-with-centralized-exception-handling-and-unified-response-structure
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1752097085688/e94a3b3c-ac12-4f8f-a711-4e59bbb40c1a.png
tags: microservices, springboot, api-gateway, exceptionhandling, spring-cloud-gateway, java-full-stack-development-java-backend-development-java-frontend-development-java-web-development-htmlcssjavascript

---

In this article we will be exploring how to use api gateway in our spring project and also how can we have centralized exception handling along with unified response structure. So without wasting time let’s get started.

Now what i mean by unifies response structure is this.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752088372479/fa2b1162-f2a9-4fd2-898c-56ade1c7df80.png align="center")

i.e service is not obliged to return data with other fields because that service may also be sending the response to other service instead of client directly hence we will add a transformation filter globally in the api gateway itself.

Other thing is the Exception handler

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1752092923283/2d5c0c78-7705-4ccf-a169-d376a0c7ce8d.png align="center")

Now let’s see how this can be implemented , starting with setting up the ApiService

## Creating a project

Start with creating the project wither use spring initializer (start.spring.io) or you do it manually (if you love pain) it’s up to you but ensure these dependencies in you pom.xml file

```java
   <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway-server-webflux</artifactId>

        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>
```

the \``spring-cloud-starter-gateway-server-webflux`\` is the core dependency of the spring cloud gateway and the `spring-cloud-starter-netflix-eureka-client` is dependency for creating eureka client.

Now after the project is created and all dependencies are loaded mandatorily annotate the main entry point (i.e class Annotated with `@SpringBootApplication`) with the annotation `@EnableDiscoveryClient`

And it is obvious that you must have eureka server running. with this url <mark>http://localhost:8761/eureka</mark>

Now we must configure the our ApiGateway we can do it through application.yaml file

```yaml
eureka:
  instance:
    prefer-ip-address: true
  client:
    fetch-registry: true
    serviceUrl:
      defaultZone: "${EUREKA_CLIENT_SERVICEURL_DEFAULTZONE:http://localhost:8761/eureka}"
    register-with-eureka: true
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      server:
        webflux:
          discovery:
            locator:
              enabled: true
          routes:
            - id: user-service
              uri: lb://user-service
              filters:
                - AuthenticationFilter
                - ApiResponseTransformFilter
                - RewritePath=/api/v1/user/(?<segment>.*), /v1/$\{segment}
              predicates:
                - Path=/api/v1/user/**
server:
  port: 8005
```

This configuration sets up a Spring Cloud Gateway application integrated with Eureka Service Discovery. It defines how the gateway registers with Eureka, discovers services, routes requests, and applies filters for authentication and response transformation.

---

### 🔹 Eureka Configuration

```yaml
eureka:
  instance:
    prefer-ip-address: true
```

* `prefer-ip-address: true`: Configures the service to register its IP address instead of the hostname with Eureka. This is useful in environments where DNS resolution may be unreliable or unavailable (e.g., Docker or local development networks).
    

```yaml
  client:
    fetch-registry: true
```

* `fetch-registry: true`: Enables the gateway to fetch the list of registered services from the Eureka server. This is essential for enabling service discovery and client-side load balancing.
    

```yaml
    serviceUrl:
      defaultZone: "${EUREKA_CLIENT_SERVICEURL_DEFAULTZONE:http://localhost:8761/eureka}"
```

* `defaultZone`: Specifies the Eureka server endpoint. It uses a placeholder to support external configuration via environment variables. If not set, it defaults to [`http://localhost:8761/eureka`](http://localhost:8761/eureka).
    

```yaml
    register-with-eureka: true
```

* `register-with-eureka: true`: Registers the API gateway itself as a service in Eureka. This is optional but helpful if other services need to discover and communicate with the gateway.
    

---

### 🔹 Spring Application Configuration

```yaml
spring:
  application:
    name: api-gateway
```

* `name: api-gateway`: Sets the logical name of the application. This name will be used when the service registers itself in Eureka.
    

---

### 🔹 Gateway Configuration

```yaml
  cloud:
    gateway:
      server:
        webflux:
```

* This section initiates the WebFlux-based reactive server used by Spring Cloud Gateway.
    

---

### 🔹 Discovery Locator

```yaml
          discovery:
            locator:
              enabled: true
```

* Enables dynamic route registration for services discovered via Eureka. When enabled, routes are automatically created based on service names. For example, if a service named `user-service` is registered, it can be accessed via `/user-service/**` without defining explicit routes.
    

---

### 🔹 Custom Route Definition

```yaml
          routes:
            - id: user-service
```

* Defines a custom route with the ID `user-service`.
    

```yaml
              uri: lb://user-service
```

* `lb://user-service`: Routes the request to the logical service name `user-service`, using client-side load balancing provided by Spring Cloud LoadBalancer.
    

---

### 🔹 Filters

```yaml
              filters:
                - AuthenticationFilter
                - ApiResponseTransformFilter
```

* `AuthenticationFilter`: A custom filter that handles authentication logic such as verifying JWT tokens or validating request headers.
    
* `ApiResponseTransformFilter`: A custom post-filter that transforms the service response into a standardized structure (e.g., wrapping it in a common response object).
    

```yaml
                - RewritePath=/api/v1/user/(?<segment>.*), /v1/${segment}
```

* `RewritePath`: Uses a regular expression to rewrite incoming request paths. For example, `/api/v1/user/register` becomes `/v1/register` before it is forwarded to the downstream service.
    

---

### 🔹 Predicates

```yaml
              predicates:
                - Path=/api/v1/user/**
```

* `Path`: Defines the URL pattern that should match this route. Requests starting with `/api/v1/user/` will be forwarded to `user-service`.
    

---

### 🔹 Server Port

```yaml
server:
  port: 8005
```

* Runs the API Gateway on port `8005`. Clients can send requests to [`http://localhost:8005`](http://localhost:8005).
    

---

## Creating filters

Now we will require some custom filters as mentioned above AuthenticationFilter for authenticating requests and ApiResponseTransformFilter for transforming the response to send back the unified response.

### AuthenticationFilter

```java
package com.lms.api_gateway.config.filters;
import com.lms.api_gateway.config.RouteValidator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

@Component
public class AuthenticationFilter extends AbstractGatewayFilterFactory<AuthenticationFilter.Config> {

    public AuthenticationFilter() {
        super(Config.class);
    }

    @Autowired
    private RouteValidator routeValidator; // used for checking wether endpoint is open or secure
    @Autowired
    RestTemplate restTemplate;
    @Override
    public GatewayFilter apply(Config config) {
        return (((exchange, chain) -> {



                ServerHttpRequest request = null;
                if(routeValidator.isSecured.test(exchange.getRequest())) {

                    if (!exchange.getRequest().getHeaders().containsKey(HttpHeaders.AUTHORIZATION)) {
                        throw new RuntimeException("missing authorization header");
                    }
                    String authHeader = exchange.getRequest().getHeaders().get(HttpHeaders.AUTHORIZATION).get(0);
                    System.out.println("Token"+authHeader);

                    if (authHeader != null && authHeader.startsWith("Bearer ")) {
                        authHeader = authHeader.substring(7);
                    }
                    else{
                        throw new RuntimeException("missing authorization header");
                    }
                    try {
//                    REST call to AUTH service

                        HttpHeaders headers = new HttpHeaders();
                        headers.set("Authorization", "Bearer "+authHeader);
                        HttpEntity<String> entity = new HttpEntity<>(headers);
                        ResponseEntity<Map> response = restTemplate.exchange( "http://localhost:8001/api/v1/user/auth/authenticate",
                                HttpMethod.GET,
                                entity,
                                Map.class);

                        String userId = (String) response.getBody().get("id");
                        request= exchange.getRequest().mutate().header("X-USER-ID", userId).build();
                        exchange = exchange.mutate().request(request).build();
                    } catch (Exception e) {
                        e.printStackTrace();
                        System.out.println("invalid access...!");
                        throw new RuntimeException("un-authorized access to application");
                    }
                }
                return chain.filter(exchange);


        }));
    }

    public static class Config {}
}
```

In this filter what we will be doing is that for each request we first check wether the endpoint is secured or open with the help of `RouteValidator`

```java
package com.lms.api_gateway.config;

import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.function.Predicate;

@Component
public class RouteValidator {

    public static final List<String> openApiEndpoints = List.of(
            "/api/v1/user/auth/register",
            "/api/v1/user/auth/login",
            "/api/v1/user/token/refresh-token",

            "/eureka"
    );

    public Predicate<ServerHttpRequest> isSecured =
            request -> openApiEndpoints
                    .stream()
                    .noneMatch(uri -> request.getURI().getPath().contains(uri));

}
```

And then if endpoint is secured sending a verification request to the UserService along with the token (access\_token) extracted from headers , then if response is successfull then it means the request is authenticated and request pass the filter.

And I think rest is pretty easy stuff like creating API call with RestTemplate.

### ApiResponseTransformFilter

Now coming to our star of the show i.e ApiResponseTransformFilter which will transform the response from downstream into our unified structure

Now first we define a class of that standard structure

```java
package com.lms.api_gateway.config;

import java.util.Map;

public class APIResponse {
    private boolean success;
    private Map<String,Object> data ;
    private String message;
    private String error;
    private int code;
    @Override
    public String toString() {
        return "APIResponse{" +
                "success=" + success +
                ", data=" + data +
                ", message='" + message + '\'' +
                ", error='" + error + '\'' +
                '}';
    }

    public APIResponse() {
    }

    public APIResponse(boolean success, Map<String,Object> data, String message, String error) {
        this.success = success;
        this.data = data;
        this.message = message;
        this.error = error;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public Map<String,Object> getData() {
        return data;
    }

    public void setData(Map<String,Object> data) {
        this.data = data;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getError() {
        return error;
    }

    public void setError(String error) {
        this.error = error;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }
}
```

So this is our APIResponse which we will send to the client

In case of error data will be null and in case of successfull request error will be null.

Now how our ApiResponse transformation will work is we will create a simple Filter class Extending AbstractGatewayFilterFactory and then while filtering the request we will mutate the response body with the help of our custom `ServerHttpResponseDecorator`

### ServerHttpResponseDecorator

`ServerHttpResponseDecorator` is a class provided by **Spring WebFlux** that allows developers to **intercept, inspect, modify, or wrap** the outgoing HTTP response **before it's sent to the client**.

It acts as a **wrapper** around the original `ServerHttpResponse` and is typically used in **custom filters** in Spring Cloud Gateway to:

* Transform or modify the response body
    
* Change response headers
    
* Log or audit the response
    
* Standardize API response formats
    
* Modify HTTP status codes (with caution)
    

### Creating a ServerHttpResponseDecorator

```java
package com.lms.api_gateway.config;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.lms.api_gateway.utils.JsonUtil;

import org.reactivestreams.Publisher;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferFactory;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.HttpStatusCode;
import org.springframework.http.MediaType;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.http.server.reactive.ServerHttpResponseDecorator;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import reactor.core.scheduler.Schedulers;

import java.nio.charset.StandardCharsets;
import java.util.Map;

/**
 * This class decorates the original ServerHttpResponse to intercept and transform the response body.
 * It wraps the downstream service response in a standardized APIResponse structure.
 */
public class DecoratorImpl extends ServerHttpResponseDecorator {

    // Stores the content type of the original response
    MediaType contentType;

    // ObjectMapper for JSON serialization
    private final ObjectMapper objectMapper = new ObjectMapper();

    // Constructor: takes the original ServerHttpResponse as delegate
    public DecoratorImpl(ServerHttpResponse delegate) {
        super(delegate); // Passes the original response to the base decorator

        // Capture content type from original headers
        this.contentType = delegate.getHeaders().getContentType();

        // Default to JSON if content type is null
        if (this.contentType == null) {
            this.contentType = MediaType.APPLICATION_JSON;
        }
    }

    // Helper method to create a standardized success API response
    private APIResponse transform(Map<String, Object> object) {
        APIResponse apiResponse = new APIResponse();
        apiResponse.setData(object);
        apiResponse.setSuccess(true);
        return apiResponse;
    }

    // Buffer factory used to create the final DataBuffer after transformation
    DataBufferFactory bufferFactory = getDelegate().bufferFactory();

    // Override the writeWith method to intercept and transform the response body
    @Override
    public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {

        // Ensure the Content-Type header is set correctly
        getHeaders().setContentType(contentType);

        // Convert body Publisher into Flux for reactive processing
        Flux<? extends DataBuffer> fluxBody = Flux.from(body);

        // Capture the HTTP status code of the response
        HttpStatusCode statusCode = getStatusCode();
        HttpStatus status = HttpStatus.valueOf(statusCode.value());

        // If response is not JSON, skip transformation and write as-is
        if (!contentType.equals(MediaType.APPLICATION_JSON)) {
            return super.writeWith(body);
        }

        // Join all data buffers into one complete buffer for processing
        return DataBufferUtils.join(fluxBody)
                .publishOn(Schedulers.boundedElastic()) // Offload heavy work to boundedElastic thread pool
                .flatMap(dataBuffer -> {
                    // Read bytes from data buffer
                    byte[] content = new byte[dataBuffer.readableByteCount()];
                    dataBuffer.read(content);

                    // Release buffer memory
                    DataBufferUtils.release(dataBuffer);

                    // Convert byte[] to String (assumed to be UTF-8 JSON)
                    String responseBody = new String(content, StandardCharsets.UTF_8);

                    // Convert JSON string into Map
                    Map<String, Object> mapBody = JsonUtil.toMap(responseBody);

                    APIResponse apiResponse;

                    // If it's a 4xx client error, wrap as an error response
                    if (statusCode != null && statusCode.is4xxClientError()) {
                        apiResponse = new APIResponse();
                        apiResponse.setSuccess(false);
                        apiResponse.setCode(statusCode.value());
                        apiResponse.setError(status.getReasonPhrase());
                        apiResponse.setMessage(responseBody); // fallback
                        apiResponse.setMessage(mapBody.getOrDefault("message", "Some error occurred").toString());
                    } else {
                        // Otherwise, wrap as a success response
                        apiResponse = transform(mapBody);
                    }

                    try {
                        // Serialize the APIResponse into byte[]
                        byte[] newContent = objectMapper.writeValueAsBytes(apiResponse);

                        // Wrap the new content in a DataBuffer
                        DataBuffer buffer = bufferFactory.wrap(newContent);

                        // Write the new DataBuffer to the response
                        return super.writeWith(Mono.just(buffer));
                    } catch (JsonProcessingException e) {
                        e.printStackTrace();
                        // If serialization fails, return the error as-is
                        return super.writeWith(Flux.error(e));
                    }
                });
    }
}
```

Let me explain how this works ,

First of all we get a `ServerHttpResponse` as a constructor parameter which is the original response sent by the downstream ok .

Then we have the extract the info like ContentType and Status code from the headers.

Then override a method called **writeWith**  
`writeWith(...)` is the method used to **write the response body** to the client in a **reactive, non-blocking way**.

It's part of the `ServerHttpResponse` (and also `ServerHttpResponseDecorator`) and is **the only way** to programmatically write content into the response body when building reactive applications.

### 🔸Parameters:

* `Publisher<? extends DataBuffer> body`:
    
    * This is the **reactive stream** (Mono or Flux) of `DataBuffer` that represent the bytes of the response body.
        

### 🔸 Returns:

* `Mono<Void>`:
    
    * A reactive signal indicating **completion** (or failure) of writing the response
        

in our method implementation what we are doing is first of all check whether the response was 200 OK or it’s a error (client error) now if it’s a client error then we return different body otherwise different response.

Now for processing we first convert that Publisher to Flux for reactive stream processing and also fetch the DataBufferFactory from the original delegated response to create a new buffer as we can’t directly create the DataBuffer because DataBuffer is an interface whose implementations are internally managed.

```java
  return DataBufferUtils.join(fluxBody)
                .publishOn(Schedulers.boundedElastic()) // Offload heavy work to boundedElastic thread pool
                .flatMap(dataBuffer -> {
                    // Read bytes from data buffer
                    byte[] content = new byte[dataBuffer.readableByteCount()];
                    dataBuffer.read(content);

                    // Release buffer memory
                    DataBufferUtils.release(dataBuffer);

                    // Convert byte[] to String (assumed to be UTF-8 JSON)
                    String responseBody = new String(content, StandardCharsets.UTF_8);

                    // Convert JSON string into Map
                    Map<String, Object> mapBody = JsonUtil.toMap(responseBody);

                    APIResponse apiResponse;

                    // If it's a 4xx client error, wrap as an error response
                    if (statusCode != null && statusCode.is4xxClientError()) {
                        apiResponse = new APIResponse();
                        apiResponse.setSuccess(false);
                        apiResponse.setCode(statusCode.value());
                        apiResponse.setError(status.getReasonPhrase());
                        apiResponse.setMessage(responseBody); // fallback
                        apiResponse.setMessage(mapBody.getOrDefault("message", "Some error occurred").toString());
                    } else {
                        // Otherwise, wrap as a success response
                        apiResponse = transform(mapBody);
                    }

                    try {
                        // Serialize the APIResponse into byte[]
                        byte[] newContent = objectMapper.writeValueAsBytes(apiResponse);

                        // Wrap the new content in a DataBuffer
                        DataBuffer buffer = bufferFactory.wrap(newContent);

                        // Write the new DataBuffer to the response
                        return super.writeWith(Mono.just(buffer));
                    } catch (JsonProcessingException e) {
                        e.printStackTrace();
                        // If serialization fails, return the error as-is
                        return super.writeWith(Flux.error(e));
                    }
                });
```

Now this part of code is what which do all the processing on the stream.

First, it joins all incoming `DataBuffer` chunks from the `fluxBody` into a single `DataBuffer` using `DataBufferUtils.join(...)`, ensuring the full response body is available at once for processing.

It then switches execution to the `boundedElastic` scheduler using `publishOn(...)` to safely handle blocking operations like JSON parsing off the main event loop (i.e in a new Thread)

Inside the `flatMap`, it reads the raw byte content from the buffer, releases the buffer memory to prevent leaks, and converts the byte array into a UTF-8 encoded string, assuming it's JSON. This JSON string is parsed into a `Map<String, Object>` using a custom utility `JsonUtil.toMap(...)`.

Based on the HTTP status code (specifically if it's a 4xx client error), the response is either wrapped into an error format with metadata like status code, error reason, and a meaningful message, or transformed into a standardized success wrapper using the `transform(...)` method.

Finally, the new structured response (`APIResponse`) is serialized back into a JSON byte array, wrapped in a `DataBuffer`, and written to the response using `super.writeWith(...)`.

In case of any serialization errors, a fallback error response is triggered using `Flux.error(...)`. This flow ensures consistent, structured API responses without blocking the reactive pipeline.

## ApiResponseTransformFilter

```java
package com.lms.api_gateway.config.filters;

import com.fasterxml.jackson.databind.ObjectMapper;

import com.lms.api_gateway.config.APIResponse;
import com.lms.api_gateway.config.DecoratorImpl;

import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.NettyWriteResponseFilter;
import org.springframework.cloud.gateway.filter.OrderedGatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;

import org.springframework.http.server.reactive.ServerHttpResponse;

import org.springframework.stereotype.Component;

import java.util.Map;

import static org.springframework.cloud.gateway.support.ServerWebExchangeUtils.ORIGINAL_RESPONSE_CONTENT_TYPE_ATTR;

@Component
public class ApiResponseTransformFilter extends AbstractGatewayFilterFactory<ApiResponseTransformFilter.Config> {

    private final ObjectMapper objectMapper = new ObjectMapper();

    public ApiResponseTransformFilter() {
        super(Config.class);
    }

    private APIResponse transform(Map<String, Object> object, Config config) {
        APIResponse apiResponse = new APIResponse();
        apiResponse.setData(object);
        apiResponse.setSuccess(true);
        return apiResponse;
    }

    @Override
    public GatewayFilter apply(Config config) {
        return new OrderedGatewayFilter((exchange, chain) -> {
            ServerHttpResponse originalResponse = exchange.getResponse();

                       return chain.filter(exchange.mutate().response(new DecoratorImpl(originalResponse)).build());




        }, NettyWriteResponseFilter.WRITE_RESPONSE_FILTER_ORDER - 1);
    }

    public static class Config {
        // Future config options
    }
}
```

### JsonUtil

```java
package com.lms.api_gateway.utils;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.util.Map;

@Service
public  class JsonUtil {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    public static Map<String, Object> toMap(String json) {
        try {
            return objectMapper.readValue(json, new TypeReference<Map<String, Object>>() {});
        } catch (IOException e) {
            throw new RuntimeException("Failed to parse JSON to Map", e);
        }
    }
}
```

Here is the final code [Github](https://github.com/memkgupta/lms_backend/tree/main/api_gateway)

Thanx for reading will see you next time , if you have any question feel free to ask.