# 11. Context

## 11.1 Context란?

어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보

구독이 발생할 떄마다 해당 구독과 연결된 하나의 Context가 생긴다.


```
public class Example11_1 {
  public static void main(String[] arfs) throws InterruptedException {
    Mono.deferContextual(ctxx -> 
      Mono
        .just("Hello" + " " + ctx.get("firstName")) //데이터 읽기
        .doOnNext(data -> log.info(" # just doOnNext : {}", data))
      )
      .subscribeon(Schedulers.boundedElastic())
      .publishOn(Schedulers.parallel())
      .transformDeferredContextual(
        (mono, ctx) -> mono.map(data -> data +  " " + ctx.get("lastName"))
      )
      .contextWrite(context -> context.put("lastName", "Jobs")) //쓰기
      .contextWrite(context -> context.put("firstName", "Steve"))
      .subscribe(data -> log.info("# onNext: {}", data));


    Thread.sleep(10L)
  }
}

```


- 컨텍스트에 데이터 쓰기

 contextWrite() operator
실제로 데이터를 쓰는 작업은 Context API 중 하나인 put을 사용해서 가능하다.

- 컨텍스트에 쓰인 데이터 읽기

 - 원본 데이터 소스 레벨에서 읽는 방식: deferContextual() Operator를 사용한다. deferContextual의 파라미터로 정의된 람다 파라미터는 COntext타입의 객체가 아니라 ContextView 타입의 객체이다. (쓸 떄는 Context, 읽을 때는 ContextView)
 - Operator 체인의 중간에서 읽는 방식: 


## 11.2 자주 사용되는 Context 관련 API

### 자주 사용하는 Context API
- put(key, value): key value 형태로 값을 쓴다.
- of(key1, value1, key2, value2, ...): key/value 형태로 여러 개의 값을 쓴다. (최대 5개)
Context.of() 는 Context 타입의 객체를 리턴해주는데, putAll로 데이터를 쓰기 위해서는 readonly 를 호출하여 ConextView로 전환해 주어야 한다.
- putAll(ContextView): 현재 Context와 파라미터로 입력된 ContextView를 merge 한다.
- delete(key): Context에서 key에 해당하는 값ㅇ르 삭제한다.

```
public class Example11_3 {
  public static void main(String[] arfs) throws InterruptedException {
    Mono.deferContextual(ctxx -> 
      Mono
        .just("Hello" + " " + ctx.get("firstName"))
      )
      .publishOn(Schedulers.parallel())
      .contextWrite(context -> 
        context.putAll(Context.of("lastName", "Jobs").readOnly())
      ) //쓰기
      .subscribe(data -> log.info("# onNext: {}", data));


    Thread.sleep(10L)
  }
}
```

### 자주 사용하는 ContextView API
- get(key): ContextView에서 key에 해당하는 value를 반환한다.
- getOrEmpty(key): ContextView에서 key에 해당하는 value를 Optional로 래핑해서 반환한다.
- getOrDefault(key, defaultValue): ContextView에 해당하는 value를 가져온다. key에 해당하는 value가 없으면 defaultValue를 가져온다.
- hasKey(key): ContextView에서 특정 key가 존재하는지 확인한다.
- isEmpty(): Context가 비어있는지 확인한다.
- size(): Context 내에 있는 key/value의 개수를 반환한다.


## 11.3 Context의 특징

- Context는 구독이 발생할 때마다 하나의 Context가 해당 구독에 연결된다.
- Context는 Operator 체인의 아래에서 위로 전파된다. 따라서 일반적으로는 모든 Operator에서 Context에 저장된 데이터를 읽을 수 있도록 contextWrite()를 Operator 체인의 맨 마지막에 둔다.
- 동일한 키에 대한 값을 중복해서 저장하면 Operator 체인에서 가장 위쪽에 위치한 contextWrite()이 저장한 값으로 덮어쓴다.

```
public class Example11_5 {
  public static void main(String[] arfs) throws InterruptedException {
    final String key1 = "company";
    Mono<String> mono = Mono.deferContextual(ctx -> 
        Mono.just("Company: " + " " + ctx.get(key1))
      )
      .publishOn(Schedulers.parallel());

    mono.contextWrite(context -> context.put(key1, "Apple"))
      .subscribe(data -> log.info("# subscribe1 onNext: {}", data));    

    mono.contextWrite(context -> context.put(key1, "Microsoft"))
      .subscribe(data -> log.info("# subscribe1 onNext: {}", data));    
  }
}
```

- Inner Sequence 내부에서는 외부 Context에 저장된 데이터를 읽을 수 있다.
- Inner Sequence 외부에서는 Inner Sequecne 내부 Context에 저장된 데이터를 읽을 수 없다.

Context는 인증 정보 같은 직교성(독립성)을 가지는 정보를 전송하는데 적합하다.