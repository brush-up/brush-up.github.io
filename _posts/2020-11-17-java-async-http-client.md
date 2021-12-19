---
title:  "[java]AsyncHttpClient 대충 간략정리 "
excerpt: "AsyncHttpClient 에 대한 간단 사용법 "

categories:
  - Java
tags:
  - [Java, Programming]

toc: true
toc_sticky: true
 
date: 2020-11-17
last_modified_at: 2020-11-17
---
## 기본적인 사항

* AsyncHttpClient 는 netty 기반의 비동기 http 클라이언트
* AsyncHttpClient 객체를 얻는 가장 간단한 방법은 dsl class를 이용하는건데,

``` java
import static org.asynchttpclient.Dsl.*;
AsyncHttpClient client = Dsl.asyncHttpClient();
```
* 이렇게 해도 되고 그냥 아래처럼 해도 되지만 
``` java
AsyncHttpClient asyncHttpClient = asyncHttpClient();
```
* custom 설정이 필요로 하면 아래처럼 **DefaultAsyncHttpClientConfig.Builder**를 이용해서 asyncHttpClient 를 빌드 할수 있음
``` java
DefaultAsyncHttpClientConfig.Builder clientBuilder = Dsl.config()
```

아래는 타암아웃을 설정한 예제

``` java
DefaultAsyncHttpClientConfig.Builder clientBuilder = Dsl.config()
  .setConnectTimeout(500);
AsyncHttpClient client = Dsl.asyncHttpClient(clientBuilder);
```

>AsyncHttpClient 인스턴스는 `close`를 해줘야 memory leak 이 발생하지 않음
AsyncHttpClient 를 생성하면 항상 새로운 thread와 connectio pool이 생성되는 구조라 
이러한 이류로 매번 각 요청에 대해 새로운 인스턴스를 만들필요가 없다. 



## http 요청 준비

* asyncHttpClient 는 bound and unbound 의 2가지 api를 제공하는데, 성능상 차이는 크게 없다.
### bound 요청
    * 접두사 prepare로 시작하는 metohod 사용
    ``` java
    BoundRequestBuilder getRequest = client.prepareGet("https://www.google.com");
    ```
    * 아래처럼 사용도 가능
    ```java
    .
    .
    .
    public interface IdApi {
        Pair<HttpMethod, String> changePassword = new ImmutablePair<>(POST, "/services/{serviceCode}/v1.0/members/{uid}/change-password");
    }
    .
    .
    .
    String url = remoteAddress + changePassword.getValue();
    Request request = buildAsyncHttpRequest(changePassword.getKey(), url, new AsyncHttpRequestBody(forms), serviceCode, uid);
    return asyncHttpClient.prepareRequest(request)
                .execute()
                .toCompletableFuture()
                .handle((response, throwable) -> {
                    ...
                });
    }
    ```


### unbound 요청
* RequestBuilder class사용해서 만들거나 Dsl 클래스를 사용해서 할수 있다.
```java
Request getRequest = new RequestBuilder(HttpConstants.Methods.GET)
  .setUrl("https://www.google.com")
  .build();
// or  
Request getRequest = Dsl.get("https://www.google.com").build()
```

* 호출까지 수행하는 하는 예제

```java
import org.asynchttpclient.*;
// bound
Future<Response> whenResponse = asyncHttpClient.prepareGet("http://www.example.com/").execute();
// unbound
Request request = get("http://www.example.com/").build();
Future<Response> whenResponse = asyncHttpClient.execute(request);
```

## http 요청 실행
* asyncHttpClient는 동기 방식, 비동기 방식 모두 지원함
* http 요청 실행은 bound, unbound에 따라 각자 다름
>Executing the request depends on its type. When using a bound request we use the execute() method from the BoundRequestBuilder class and when we have an unbound request we’ll execute it using one of the implementations of the executeRequest() method from the AsyncHttpClient interface.

### request body 세팅하기
* setBody method를 이용해서 할수있고
* addBodyPart method를 이용하면 multipart 도 가능하다.

### 동기 방식 요청하기
* 비동기방식으로 디자인 되었지만 Future 객체를 블러킹해서 동기처럼 호출할수 있음
* execute()와 executeRequest() method 는 ListenableFuture 객체를 리턴하는데 이건 java Future interface를 상속받은거
* `굳이 쓸일이 없어 쓰지마`
* 쓸필요 없긴 하지만 예제는 아래처럼
```java
Future<Response> responseFuture = boundGetRequest.execute();
responseFuture.get();
// or
Future<Response> responseFuture = client.executeRequest(unboundRequest);
responseFuture.get();
```

### 비동기 방식 요청하기
* 비동기는 결과를 처리하기 위한 리스너부터 알아야해 아래의 3가지 유형의 리스너를 제공하고있어.
    * `AsyncHandler`
    * `AsyncCompletionHandler`
    * `ListenableFuture`


##### AsyncHandler 리스너는 http 호출과 관련된 웬만한 모든 이벤트를 처리할수 있다.
```java
request.execute(new AsyncHandler<Object>() {
    @Override
    public State onStatusReceived(HttpResponseStatus responseStatus)
      throws Exception {
        return null;
    }
 
    @Override
    public State onHeadersReceived(HttpHeaders headers)
      throws Exception {
        return null;
    }
 
    @Override
    public State onBodyPartReceived(HttpResponseBodyPart bodyPart)
      throws Exception {
        return null;
    }
 
    @Override
    public void onThrowable(Throwable t) {
 
    }
 
    @Override
    public Object onCompleted() throws Exception {
        return null;
    }
});
.
.
.
.
.
import static org.asynchttpclient.Dsl.*;
import org.asynchttpclient.*;
import io.netty.handler.codec.http.HttpHeaders;

Future<Integer> whenStatusCode = asyncHttpClient.prepareGet("http://www.example.com/")
.execute(new AsyncHandler<Integer>() {
	private Integer status;
	@Override
	public State onStatusReceived(HttpResponseStatus responseStatus) throws Exception {
		status = responseStatus.getStatusCode();
		return State.ABORT;
	}
	@Override
	public State onHeadersReceived(HttpHeaders headers) throws Exception {
		return State.ABORT;
	}
	@Override
	public State onBodyPartReceived(HttpResponseBodyPart bodyPart) throws Exception {
		return State.ABORT;
	}
	@Override
	public Integer onCompleted() throws Exception {
		return status;
	}
	@Override
	public void onThrowable(Throwable t) {
	}
});

Integer statusCode = whenStatusCode.get();
```

State enum는 http 요청의 진행상태를 컨트롤 할수 있도록 해줘. State.ABORT 를 돌려 주면 특정 순간에 처리 를 멈추고 State.CONTINUE 를 사용 하여 처리를 마칩니다.
AsyncHandler 는 thread safe하지 않아. concurrent 요청에 재사용안되.

##### AsyncCompletionHandler 리스너는 AsyncHandler 인터페이스 상속한거고 호출 작업이 끝났을때 호출되는 onCompleted 이 추가되었어.

``` java
request.execute(new AsyncCompletionHandler<Object>() {
    @Override
    public Object onCompleted(Response response) throws Exception {
        return response;
    }
});
```

##### ListenableFuture 리스너 인터페이스는 http 호출이 완료될때 실행되고 추가적인 리스너 등록이 가능해, 다른 thread pool을 이용해서 리스너가 수행되
* 게다가 ListenableFuture 인터페이스를 사용해서 Future 응답을 CompletableFuture 로 변환해줘?

``` java
ListenableFuture<Response> listenableFuture = client
  .executeRequest(unboundRequest);
listenableFuture.addListener(() -> {
    Response response = listenableFuture.get();
    LOG.debug(response.getStatusCode());
}, Executors.newCachedThreadPool());
```


### websocket
    * Async Http Client also supports WebSocket. You need to pass a WebSocketUpgradeHandler where you would register a WebSocketListener.

``` java
WebSocket websocket = c.prepareGet("ws://demos.kaazing.com/echo")
      .execute(new WebSocketUpgradeHandler.Builder().addWebSocketListener(
          new WebSocketListener() {

          @Override
          public void onOpen(WebSocket websocket) {
              websocket.sendTextFrame("...").sendTextFrame("...");
          }

          @Override
          public void onClose(WebSocket websocket) {
          }

    		  @Override
          public void onTextFrame(String payload, boolean finalFragment, int rsv) {
          	System.out.println(payload);
          }

          @Override
          public void onError(Throwable t) {
          }
      }).build()).get();
```

## connection pooling

* HTTP persistent connection 도 지원하는데 이건 ChannelPool 에 저장되어있어.
* The default ChannelPool is DefaultChannelPool.
* Optionally, connection pool behaviour can be configured via AsyncHttpClientConfig:

```
import static org.asynchttpclient.Dsl.*;

AsyncHttpClient http = asyncHttpClient(config()
    .setMaxConnections(500)
    .setMaxConnectionsPerHost(200)
    .setPooledConnectionIdleTimeout(100)
    .setConnectionTtl(500)
);
```

디폴트 설정은 여기에
https://github.com/AsyncHttpClient/async-http-client/blob/master/client/src/main/resources/org/asynchttpclient/config/ahc-default.properties

* 구현별 설정
    * DefaultChannelPool 은 기본적으로 last in, first out이 기본이야.
    * 하지만 엄청난 부하를 처리할수있도록 많은 연결을 유지할수도있는데 first in first out기준으로 하도록 구성가능해

```java
import static org.asynchttpclient.Dsl.*;
HashedWheelTimer timer = new HashedWheelTimer();
timer.start();
ChannelPool pool =
  new DefaultChannelPool(60000, -1, DefaultChannelPool.PoolLeaseStrategy.FIFO, timer);
AsyncHttpClient httpClient = asyncHttpClient(config()
    .setNettyTimer(timer)
    .setChannelPool(pool)
);
```

* 설정 샘플 예제
	* 아래내용 참고
    * https://github.com/AsyncHttpClient/async-http-client/blob/master/client/src/main/resources/org/asynchttpclient/config/ahc-default.properties

```java
public class AsyncHttpProperties {

    private int connectTimeout = 1000 * 5;
    private int requestTimeout = 1000 * 5;
    private int pooledConnectionIdleTimeout = 1000 * 60;
    private Integer connectionPoolCleanerPeriod;
    private boolean keepAlive = true;
    private int maxConnections = -1;
    private int maxConnectionsPerHost = -1;
    private String threadPoolName = "AsyncHttpClient";
    private boolean useNativeTransport = false; // Native Epool
    private int ioThreadsCount = 0; // async worker count. 0 = cpu x 2

    private Integer connectionTtl; // millis

    public AsyncHttpProperties(AsyncHttpProperties properties) {
        this.connectTimeout = properties.getConnectTimeout();
        this.requestTimeout = properties.getRequestTimeout();
        this.keepAlive = properties.isKeepAlive();
        this.maxConnections = properties.getMaxConnections();
        this.maxConnectionsPerHost = properties.getMaxConnectionsPerHost();
        this.threadPoolName = properties.getThreadPoolName();
        this.useNativeTransport = properties.isUseNativeTransport();
        this.pooledConnectionIdleTimeout = properties.getPooledConnectionIdleTimeout();
        this.connectionPoolCleanerPeriod = properties.getConnectionPoolCleanerPeriod();
        this.ioThreadsCount = properties.getIoThreadsCount();
        this.connectionTtl = properties.getConnectionTtl();
    }
}
```

```java
public AsyncHttpClient create(AsyncHttpProperties asyncHttpProperties) {
        DefaultAsyncHttpClientConfig.Builder configBuilder = new DefaultAsyncHttpClientConfig.Builder();

        configBuilder.setFollowRedirect(true);

        //
        configBuilder.setConnectTimeout( asyncHttpProperties.getConnectTimeout() ); // 
        configBuilder.setRequestTimeout( asyncHttpProperties.getRequestTimeout() ); // 

        // keep-alvie
        configBuilder.setKeepAlive( asyncHttpProperties.isKeepAlive() );
        configBuilder.setPooledConnectionIdleTimeout( 1000 * 60 * 5 );
        configBuilder.setMaxConnections( asyncHttpProperties.getMaxConnections() );
        configBuilder.setMaxConnectionsPerHost( asyncHttpProperties.getMaxConnectionsPerHost() );

        // tuning
        configBuilder.setTcpNoDelay(true);
        configBuilder.setSoReuseAddress(true);

        // internals
        configBuilder.setThreadPoolName( asyncHttpProperties.getThreadPoolName() );
        configBuilder.setUseNativeTransport( asyncHttpProperties.isUseNativeTransport() ); // true: Linux 에서 Epoll 사용
        configBuilder.setIoThreadsCount( asyncHttpProperties.getIoThreadsCount() ); // 
        //
        configBuilder.setPooledConnectionIdleTimeout( asyncHttpProperties.getPooledConnectionIdleTimeout() );
        if (asyncHttpProperties.getConnectionPoolCleanerPeriod() != null)
            configBuilder.setConnectionPoolCleanerPeriod( asyncHttpProperties.getConnectionPoolCleanerPeriod() );

        //
        if (asyncHttpProperties.getConnectionTtl() != null)
            configBuilder.setConnectionTtl( asyncHttpProperties.getConnectionTtl() );

        log.info("Create asyncHttpClient configuration:{}", asyncHttpProperties.toString());

        return new DefaultAsyncHttpClient(configBuilder.build());
    }
```

* 아래 정보는 더 보기.
    * https://jfarcand.wordpress.com/2010/12/21/going-asynchronous-using-asynchttpclient-the-basic/
    * https://jfarcand.wordpress.com/2011/01/04/going-asynchronous-using-asynchttpclient-the-complex/
    * https://jfarcand.wordpress.com/2011/12/21/writing-websocket-clients-using-asynchttpclient/
* 참고자료
    * https://www.baeldung.com/async-http-client
    * https://github.com/AsyncHttpClient/async-http-client
    * https://github.com/AsyncHttpClient/async-http-client/wiki/Connection-pooling
