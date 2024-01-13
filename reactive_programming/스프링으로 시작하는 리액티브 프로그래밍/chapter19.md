# 19. 예외처리

## 19.1 onErrorResume() Operator를 사용한 예외처리

에러 이벤트가 발생했을 때 에러 이벤트를 Downstream으로 전파하지 않고, 대체 Publisher를 통해 에러 이벤트에 대한 대체 값을 emit하거나 발생한 에러 이벤트를 래핑한 후에 다시 이벤트를 발생시키는 역할을 한다.

## 19.2 ErrorWebExceptionHandler를 이용한 글로벌 예외처리
onErrorResume() 등의 Operator를 이용한 인라인 예외처리는 사용하기 간편하다. 하지만 클래스 내에 여러 개의 Sequence가 존재한다면 각 Sequence마다 onErrorResume() Operator를 일일히 추가해야 하고 중복 코드가 발생할 수 있다. 이러한 단점을 보완 가능하다.

```
public class GlobalWebExceptionHandler implements ErrorWebExceptionHandler {
  @Override
  public Mono<Void> handle(ServerWebExchange serverWebExchange, Throwable throwable) {
    ...
  }
}
```
