---
title:  "[java]java11 http client 대충 간략정리 "
excerpt: "java11에 추가된 http client를 사용해볼까?"

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2021-04-20
last_modified_at: 2021-04-20

published: false
---

* JDK9부터 인큐베이팅 API로 추가되었던 것이 Java11부터 정식으로 들어갔음.
* JDK 1.1 이후에 http client인 HttpURLConnection([https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html](https://docs.oracle.com/javase/8/docs/api/java/net/HttpURLConnection.html) 가 있었으나, 다들 Apache의 HttpComponents client([https://hc.apache.org/httpcomponents-client-4.5.x/index.html](https://hc.apache.org/httpcomponents-client-4.5.x/index.html)를 사용했었어 이제 java 11에 들어있는것을 사용하는것도 괜찮지 않나? 싶음

## http client 소개

* HTTP/1.1 와 HTTP/2 를 지원하고, 동기, 비동기 프로그래밍이 가능

### GET 방식 호출 예제

``` java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create("http://openjdk.java.net/"))
      .build();
client.sendAsync(request, BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println)
      .join();
```

### HttpClient

* 요청을 보내려면 빌더 방식으로 HttpClient 를 생성해야한다. 빌더방식으로 할시 설정정보는 아래와 같은 것들이 있다.
    * protocol version ( HTTP/1.1 or HTTP/2 )
    * redirects
    * A proxy
    * An authenticator

``` java
HttpClient client = HttpClient.newBuilder()
      .version(Version.HTTP_2)
      .followRedirects(Redirect.SAME_PROTOCOL)
      .proxy(ProxySelector.of(new InetSocketAddress("www-proxy.com", 8080)))
      .authenticator(Authenticator.getDefault())
      .build();
```

* 일단 한번 생성되면 여러요청을 보낼수 있다.

### HttpRequest

* HttpRequest도 역시 builder방식으로 생성하고 아래와 같은 정보를 세팅할수 있다.
    * The request URI
    * The request method ( GET, PUT, POST )
    * The request body ( if any )
    * A timeout
    * Request headers

``` java
HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create("http://openjdk.java.net/"))
      .timeout(Duration.ofMinutes(1))
      .header("Content-Type", "application/json")
      .POST(BodyPublishers.ofFile(Paths.get("file.json")))
      .build()
```

* 일단 build되면 HttpRequest는 변경이 불가능하고 여러번 보낼수 있다.

### Synchronous or Asynchronous

* 동기 방식 호출
    * 당연히 HttpResponse 가 가능할때까지 block 된다.

``` java
HttpResponse<String> response =
      client.send(request, BodyHandlers.ofString());
System.out.println(response.statusCode());
System.out.println(response.body());
```

* 비동기 방식 호출
    * HttpResponse 가 사용 가능해지면 CompletableFuture와 함께 즉시 반환됨 CompletableFuture는 java8에 추가되었으면 비동기프로그래밍을 지원한다.
    * [게임플랫폼서버팀-스터디관련/31 (몰라도 상관없지만, 알면 매우 좋은) CompletableFuture 의 실전 사용법](dooray://1387695619080878080/tasks/2349067407574791943 "registered")내용 알아야 이해 가능함.

``` java
client.sendAsync(request, BodyHandlers.ofString())
      .thenApply(response -> { System.out.println(response.statusCode());
                               return response; } )
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

### Data as reactive-streams

* (언젠가 내용 넣기)

### HTTP/2

* Java HTTP Client는 HTTP / 1.1 및 HTTP / 2를 모두 지원하고 기본값은 http/2, 서버에서 http/2를 지원하지 않으면 자동으로 http / 1.1로 전환된다.
* http/2 가 제공하는 주요 개선사항
    * Header Compression. HTTP/2 uses HPACK compression, which reduces overhead.
    * Single Connection to the server, reduces the number of round trips needed to set up multiple TCP connections.
    * Multiplexing. Multiple requests are allowed at the same time, on the same connection.
    * Server Push. Additional future needed resources can be sent to a client.
    * Binary format. More compact.

## 관련 예제들

### Get 방식으로 동기 호출

Response body as a String

``` java
public void get(String uri) throws Exception {
    HttpClient client = HttpClient.newHttpClient();
    HttpRequest request = HttpRequest.newBuilder()
          .uri(URI.create(uri))
          .build();

    HttpResponse<String> response =
          client.send(request, BodyHandlers.ofString());

    System.out.println(response.body());
}
```

* 위의 예제는 ``BodyHandlers.ofString()``를 이용해서 응답 body bytes를 String 형태로 변환한다. BodyHandler는 반드시 요청보낸 HttpRequest 들에 대해서 각각 따로? 제공되어야 한다.
* The BodyHandler is invoked once the response status code and headers are available, but before the response body bytes are received. The BodyHandler is responsible for creating the BodySubscriber which is a reactive-stream subscriber that receives streams of data with non-blocking back pressure. The BodySubscriber is responsible for, possibly, converting the response body bytes into a higher-level Java type.
* The HttpResponse.BodyHandlers class provides a number of convenience static factory methods for creating a BodyHandler. A number of these accumulate the response bytes in memory until it is completely received, after which it is converted into the higher-level Java type, for example, ofString, and ofByteArray. Others stream the response data as it arrives; ofFile, ofByteArrayConsumer, and ofInputStream. Alternatively, a custom written subscriber implementation can be provided.


### Get 방식으로 비동기 호출

* 비동기 방식 예제

``` java
public CompletableFuture<String> get(String uri) {
    HttpClient client = HttpClient.newHttpClient();
    HttpRequest request = HttpRequest.newBuilder()
          .uri(URI.create(uri))
          .build();

    return client.sendAsync(request, BodyHandlers.ofString())
          .thenApply(HttpResponse::body);
}
```

### Post

* post 방식 예제

``` java
public void post(String uri, String data) throws Exception {
    HttpClient client = HttpClient.newBuilder().build();
    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(uri))
            .POST(BodyPublishers.ofString(data))
            .build();

    HttpResponse<?> response = client.send(request, BodyHandlers.discarding());
    System.out.println(response.statusCode());
}
```

* The BodyPublisher is a reactive-stream publisher that publishes streams of request body on-demand. HttpRequest.Builder has a number of methods that allow setting a BodyPublisher; Builder::POST, Builder::PUT, and Builder::method. The HttpRequest.BodyPublishers class has a number of convenience static factory methods that create a BodyPublisher for common types of data; ofString, ofByteArray, ofFile.
* The discarding BodyHandler can be used to receive and discard the response body when it is not of interest.


### Concurrent Requests

It's easy to combine Java Streams and the CompletableFuture API to issue a number of requests and await their responses. The following example sends a GET request for each of the URIs in the list and stores all the responses as Strings.

``` java
public void getURIs(List<URI> uris) {
    HttpClient client = HttpClient.newHttpClient();
    List<HttpRequest> requests = uris.stream()
            .map(HttpRequest::newBuilder)
            .map(reqBuilder -> reqBuilder.build())
            .collect(toList());

    CompletableFuture.allOf(requests.stream()
            .map(request -> client.sendAsync(request, ofString()))
            .toArray(CompletableFuture<?>[]::new))
            .join();
}
```

### Get JSON

In many cases the response body will be in some higher-level format. The convenience response body handlers can be used, along with a third-party library to convert the response body into that format.

The following example demonstrates how to use the Jackson library, in combination with BodyHandlers::ofString to convert a JSON response into a Map of String key/value pairs.

``` java
public CompletableFuture<Map<String,String>> JSONBodyAsMap(URI uri) {
    UncheckedObjectMapper objectMapper = new UncheckedObjectMapper();

    HttpRequest request = HttpRequest.newBuilder(uri)
          .header("Accept", "application/json")
          .build();

    return HttpClient.newHttpClient()
          .sendAsync(request, BodyHandlers.ofString())
          .thenApply(HttpResponse::body)
          .thenApply(objectMapper::readValue);
}

class UncheckedObjectMapper extends com.fasterxml.jackson.databind.ObjectMapper {
    /** Parses the given JSON string into a Map. */
    Map<String,String> readValue(String content) {
    try {
        return this.readValue(content, new TypeReference<>(){});
    } catch (IOException ioe) {
        throw new CompletionException(ioe);
    }
}
```

The above example uses ofString which accumulates the response body bytes in memory. Alternatively, a streaming subscriber, like ofInputStream could be used.

### Post JSON

In many cases the request body will be in some higher-level format. The convenience request body handlers can be used, along with a third-party library to convert the request body into that format.

The following example demonstrates how to use the Jackson library, in combination with the BodyPublishers::ofString to convert a Map of String key/value pairs into JSON.

``` java
public CompletableFuture<Void> postJSON(URI uri,
                                        Map<String,String> map)
    throws IOException
{
    ObjectMapper objectMapper = new ObjectMapper();
    String requestBody = objectMapper
          .writerWithDefaultPrettyPrinter()
          .writeValueAsString(map);

    HttpRequest request = HttpRequest.newBuilder(uri)
          .header("Content-Type", "application/json")
          .POST(BodyPublishers.ofString(requestBody))
          .build();

    return HttpClient.newHttpClient()
          .sendAsync(request, BodyHandlers.ofString())
          .thenApply(HttpResponse::statusCode)
          .thenAccept(System.out::println);
}
```

### Setting a proxy

A ProxySelector can be configured on the HttpClient through the client's Builder::proxy method. The ProxySelector API returns a specific proxy for a given URI. In many cases a single static proxy is sufficient. The ProxySelector::of static factory method can be used to create such a selector.

Response body as a String with a specified proxy

``` java
public CompletableFuture<String> get(String uri) {
    HttpClient client = HttpClient.newBuilder()
          .proxy(ProxySelector.of(new InetSocketAddress("www-proxy.com", 8080)))
          .build();

    HttpRequest request = HttpRequest.newBuilder()
          .uri(URI.create(uri))
          .build();

    return client.sendAsync(request, BodyHandlers.ofString())
          .thenApply(HttpResponse::body);
}
```

Alternatively, the system-wide default proxy selector can be used, which is the default on macOS.

``` java
HttpClient.newBuilder()
      .proxy(ProxySelector.getDefault())
      .build();
```

## TODO

* client side 인증서는 어떻게 설정하는지 확인 필요함
* header 정보 세팅하는 예제 넣기
* cookie 다루는 법 넣기
* connection, read timeout 설정하는 예제 찾아보기.
* 성능은?

* 참고자료
  * [https://openjdk.java.net/groups/net/httpclient/](https://openjdk.java.net/groups/net/httpclient/)
  * [https://openjdk.java.net/groups/net/httpclient/intro.html](https://openjdk.java.net/groups/net/httpclient/intro.html)
  * [https://openjdk.java.net/groups/net/httpclient/recipes.html](https://openjdk.java.net/groups/net/httpclient/recipes.html)

