# 17. 함수형 엔드포인트

## 17.1 HandlerFunction을 사용한 request처리 

```
@FunctionalInterface
public interface HandlerFunction<T extends ServerResponse> {
  Mono<T> handle(ServerRequest request);
}
```


 HandlerFUnction은 요청 어리를 위한 ServerRequest하나만 handle() 메서드의 파라미터로 전달받으며, 요청 처리에 대한 응답은 Mono< ServerResponse > 의 형태로 리턴된다.

 - ServerRequest

 ServerRequest는 HandlerFunction에 의해 처리되는 HTTP requestfmf vygusgksek.
 ServerRequest는 객체를 통해 HTTP headers, method, URl, query parameters에 접근할 수 있는 메서드를 제공하며, HTTP body 정보에 접근하기 위해 body(), bodyToMono(), bodyToFlux()같은 별도의 메서드를 제공한다.

 - ServerResposne

 ServerResponse는 HandlerFunction 또는 HandlerFilterFuncion에서 리턴되는 HTTP response를 표현한다.
 ServerResponse는 BodyBuilder와 HeadersBuilder를 통해 HTTP response body와 header 정보를 추가할 수 있다.


## 17.2 request 라우팅을 위한 RouterFunction

RouterFunction은 들어오는 요청을 해당 HandlerFunction으로 하우팅해 주는 역할을 한다. (@RequestMappimng과 동일한 기능을 한다.)

RouterFunction은 요청을 위한 데이터 뿐만 아니라 요청 처리를 위한 동작(HandlerFunction) 까지 RouterFunction의 파라미터로 제공한다는 차이가 존재한다.

```
@FunctionalInterface
public interface RouterFunction<T extends ServerResponse> {
  Optional<HandlerFunction<T>> route(ServerRequest request);
}
```