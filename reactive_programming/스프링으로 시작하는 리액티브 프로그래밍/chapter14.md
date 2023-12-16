# Operator란?

## 14.1 Operator란?


## 14.2 Sequence 생성을 위한 Operator

### justOrEmtpy

just와 동일하게 동작, emit할 데이터가 null인 경우 NPE 대신 onComplete 시그널을 전송

```
@Sl4j
public class Example14_1 {
  public static void main(String[] args) {
    Mono.justOrEmpty(null)
    .subscribe(data -> {},
    error -> {},
    () -> log.info("# onComplete"));
  }
}
```


### fromIterable

Iterable에 포함된 데이터를 emit하는 Flux를 생성한다.

```
@Sl4j
public class Example14_2 {
  public static void main(String[] args) {
    Flux.fromIterable(Sampledata.coins)
    .subscribe(coin -> log.info("coin 명: {} 현재가", coin.getT1(), coin.T2()"));
  }
}
```

### fromStream

Stream에 포함된 데이터를 emit하는 Flux 생성

```
@Sl4j
public class Example14_3 {
  public static void main(String[] args) {
    Flux.fromstream(() -> SampleData.cinNames.stream())
    .subscribe(coin -> log.info("coin 명: {} 현재가", coin.getT1(), coin.T2()));
  }
}
```

### range

n부터 1씩 증가한 연속된 수를 m개 emit 하는 Flux 생성

```
@Sl4j
public class Example14_4 {
  public static void main(String[] args) {
    Flux.range(5, 10)
    .subscribe(data -> log.info("{}", data));
  }
}
```


### defer

Operator를 선언한 시점에 데이터를 emit하는 것이 아니라 구독하는 시점에 데이터를 emit 하는 Flux 또는 Mono를 생성한다. 데이터 emit을 지연시키기 때문에 꼭 필요한 시점에 데이터를 emit하여 불필요한 프로세스를 줄일 수 있다. (원본 just는 hot publisher라 데이터를 생성하는 시점에 구독 여부와 관계없이 데이터를 emit한다.)

```
@Sl4j
public class Example14_6 {
  public static void main(String[] args) {
    Mono<LocalDateTime> justMono = Mono.just(LocalDateTime.now());
    Mono<LocalDateTime> deferMono = Mono.defer(() -> Mono.just(LocalDateTime.now()));
    Thread.sleep(2000);
    
    justMono.subscribe(data -> log.info(" onNextJust1: {}", data)); //0
    deferMono.subscribe(data -> log.info(" onNextDefer1: {}", data)); //2

    Thread.sleep(2000);
    
    justMono.subscribe(data -> log.info(" onNextJust1: {}", data)); //0
    deferMono.subscribe(data -> log.info(" onNextDefer1: {}", data)); // 4
  }
}
```


 
### using

파라미터로 전달받은 resource를 emit하는 Flux를 생성한다.

```
@Sl4j
public class Example14_8 {
  public static void main(String[] args) {
    Path path = Paths.get("test.txt");
    Flux.using(() -> Files.lines(path), Flux::fromStream, Stream::close)
      .subscribe(log:info);
  }
}
```

 
### using

파라미터로 전달받은 resource를 emit하는 Flux를 생성한다.

```
@Sl4j
public class Example14_8 {
  public static void main(String[] args) {
    Path path = Paths.get("test.txt");
    Flux.using(() -> Files.lines(path), Flux::fromStream, Stream::close)
      .subscribe(log:info);
  }
}
```

### generate

동기적으로 데이터를 하나씩 emit할 경우 사용.
세 번째 파라미터로는 Consumer를 통해 후처리를 할 수 있다.

```
@Sl4j
public class Example14_9 {
  public static void main(String[] args) {
    Flux.generate(() -> 0, (state, sink) -> {
      sink.next(state);
      if (state == 10) sink.complete();
      return ++state;
    })
      .subscribe(data -> log.info('# doNext: {}', data));
  }
}
```

### create

generate Operator처럼 프로그래밍 방식으로 Signal 이벤트를 발생시키지만 generate() Operator와 몇 가지 차이점이 존재. generate()는 Operator는 데이터를 동기적으로 한 번에 한 건씩 emit할 수 있는 반면에, create() Opreator는 한번에 여러 건의 데이터를 비동기적으로 emit할 수 있다.


## 14.3 Sequence 필터링 Operator

### filter

Upstream에서 emit된 데이터 중에서 조건에 일치하는 데이터만 Downstream으로 emit한다.

```
@Sl4j
public class Example14_15 {
  public static void main(String[] args) {
    Flux.range(1, 20 )
      .filter(num -> num %2 != 0)
      .subscribe(data -> log.info('# onNext: {}', data));
  }
}
```

### skip

파라미터로 입력받는 숫자만큼 건너뛴 후, 나머지 데이터를 Downstream으로 emit한다.

```
@Sl4j
public class Example14_18 {
  public static void main(String[] args) {
    Flux.interval(Duration.ofSeconds(1))
      .skip(2)
      .subscribe(data -> log.info('# onNext: {}', data));
  }
  Thread.sleep(5500L)
}
```


### take

upstream에서 emit되는 데이터 중에서 파라미터로 입력한 데이터 숫자만큼 DownStream으로 emit한다.

```
@Sl4j
public class Example14_21 {
  public static void main(String[] args) {
    Flux.interval(Duration.ofSeconds(1))
      .take(3)
      .subscribe(data -> log.info('# onNext: {}', data));
  }
  Thread.sleep(4000L)
}
```

takeWhile

람다식이 true가 되는 동안에만 Upstream에서 emit된 데이터를 Downstream으로 emit한다. Upstream에서 emit된 데이터에는 Predicate을 평가할 때 사용한 데이터가 포함된다.

takeUntil

람다식이 true가 될 때까지 Upstream에서 emit된 데이터를 Downstream으로 emit한다. Upstream에서 emit된 데이터에는 Predicate을 평가할 때 사용한 데이터가 포함된다.

### next

emit되는 데이터 중에서 첫 번쨰 데이터만 Donwstream으로 emit한다.

```
@Sl4j
public class Example14_21 {
  public static void main(String[] args) {
    Flux.interval(SampleData.btcTopPricesPerYear)
      .next()
      .subscribe(tuple -> log.info("# onNext: {}, {}", tuple.getT1(), tuple.getT2()));
  }
}
```


## 14.4 Sequence 변환 Operator

### map

mapper functio을 사용하여 변환, downstream으로 emit한다. map 내부에서 에러 발생 시 Sequence가 종료되지 않고 계속 진행되도록 하는 기능도 지원한다.

```
@Sl4j
public class Example14_28 {
  public static void main(String[] args) {
    final double buyPrice = 50_000_000;
    Flux.fromIterable(SampleData.btcTopPricesPerYear)
      .map(tuple -> calculateProfitRate(buyPrice, tuple.getT2()))

      .subscribe(tuple -> log.info("# onNext: {}", data));
  }
}
```

### flatMap

upstream에서 emit된 데이터가 innersequece에서 평탄화 작업을 거치면서 하나의 Sequence로 병합되어 Downstream으로 emit된다.

```
@Sl4j
public class Example14_29 {
  public static void main(String[] args) {
    Flux
      .just("Good", "Bad")
      .flatMap(feeling -> Flux
        .just("Morning", "Afternoon", "Evening"))
        .map(time -> feeling + " " + time)
      .subscribe(log::info);
  }
}
```

### concat

파라미터로 입력되는 Publisher의 Sequence를 연결해서 데이터를 순차적으로 emit한다. 먼저 입력된 Publisherdml Sequence가 종료될 때까지 나머지 Publisherdml Sequence는 subscribe 되지 않고 대기한다.

```
@Sl4j
public class Example14_31 {
  public static void main(String[] args) {
    Flux
      .concat(Flux.just(1, 2, 3), Fux.just(4, 5))
      .subscribe(data -> log.info("# onNext: {}" , data))
  }
}
```

### merge

파라미터로 입력되는 Publisher의 Sequence에서 emit된 데이터를 인터리빙 방식으로 병합한다.
concat처럼 먼저 입력된 publisher의 sequence 가 종료될 때까지 나머지 publisher의 Sequence가 subscribe되지 않고 대기하는 것이 아니라 모든 Publisherdml Sequence가 즉시 subscribe된다.

```
@Sl4j
public class Example14_33 {
  public static void main(String[] args) {
    Flux
      .concat(Flux.just(1, 2, 3), Flux.just(4, 5))
      .subscribe(data -> log.info("# onNext: {}" , data))
  }
}
```


### zip

파라미터로 입력되는 Publisher의 Sequence에서 emit된 데이터를 결합하는데, 각 Publisher가 데이터를 하나씩 emit할 떄까지 기다렸다가 병합한다.

```
@Sl4j
public class Example14_35 {
  public static void main(String[] args) {
    Flux
      .zip(
        FLux.just(1,2,3).delauyElements(Duration.ofMills(300L))
        Flux.just(4,5,6).delauyElements(Duration.ofMills(500L))
      )
      .subscribe(data -> log.info("# onNext: {}" , data))

      Thread.sleep(2500L)
  }
}
```


### and

 Mono의 Complete 시그널과 파라미터로 입력된 Publisher의 complete 시그널을 결합하여 새로운 Mono<Void> 를 반환한다.


### collectList

Flux에서 emit된 데이터를 모아서 List로 변환한 후, 변환된 list를 emit하는 Mono를 반환한다. Upstream sequence가 비어있다면, 비어 있는 List를 Downstream으로 emit한다.

### collectMap

Flux에서 emit된 데이터를 기반으로 key와 value를 생성하여 Map의 Element로 추가한 후, 최종적으로 Map을 emit하는 Mono를 반환한다.

## 14.5 Sequence의 내부 동작 확인을 위한 Operator

doOnXXX()으로 싲가하는 Operator. 부수 효과만을 수행

- doOnSubscribe(): publisher가 구독 중일 때 트리거되는 동작을 추가할 수 있다.
- doOnRequest(): publisher가 요청을 수신할 떄 트리거되는 동작을 추가할 수 있다.
- doOnNext(): publisher가 데이터를 emit할 때 트리거되는 동작을 추가할 수 있다.
- doOnComplete(): publisher가 성공적으로 완료되었을 때 트리거되는 동작을 추가할 수 있다.
- doOnError(): publisher가 에러가 발생한 상태로 종료되었을 때 트리거되는 동작을 추가할 수 있다.
- doOnCancel(): publisher가 취소되었을 때 트리거는 동작을 추가할 수 있다.
- doOnTerminate(): publisher가 성공적으로 완료되었을 때 또는 에러가 발생한 상태로 종료되었을 때 트리거되는 동작을 추가할 수 있다. 
- doOnEach(): Publisher가 데이터를 emit할 떄, 성공적으로 완료되었을 때, 에러가 발생한 상태로 종료되었을 때, 트리거되는 동작을 추가할 수 있다.
- doOnDiscard(): Upstream에 있는 전체 Operator체인의 동작 중에서 Operator에 의해 폐기되는 요소를 조건부로 정리할 수 있다.
- doAlterTerminate(): Downstream을 성공적으로 완료한 직후 또는 에러가 발생하여 Publisher가 종료된 직후에 트리거되는 동작을 추가할 수 있다.
- doFirst(): publisher가 구독되기 전에 트리거되는 동작을 추가할 수 있다.
- doFinally(): 에러를 포함해서 어떤 이유이든 간에 Publisher가 종료된 후 트리거되는 동작을 추가할 수 있다.

## 14.6 예외 처리를 위한 Opreator

### error

error() Operator는 파라미터로 지정된 에러로 종료하는 Flux를 생성한다.

### onErrorReturn

에러 이벤트가 발생했을 때, 에러 이벤트를 Downstream으로 전파하지 않고 대체 값을 emit한다.

### onErrorResume

에러 이벤트가 발생했을 때, 에러 이벤트를 Downstream으로 전파하지 않고 대체 Publisher를 리턴한다.

### onErrorContinue

Reactor Sequence를 사용하다 보면 에러가 발생했을 때, Sequence가 종료되지 않고 아직 emit되지 않은 데이터를 다시 emit해야 되는 상황에서 사용. 에러가 발생했을 때, 에러 영역 내에 있는 데이터는 제거하고 Upstream에서 후속 데이터를 emit하는 방식으로 에러를 복구할 수 있도록 한다.

```
@Sl4j
public class Example14_48 {
  public static void main(String[] args) {
    Flux
      .just(1, 2, 4, 0, 6, 12)
      .map(num -> 12 / num)
      .onErrorContinue((error, num) -> log.error("error: {}, num: {}", error. getMessage(), num()))
      .subscribe(data -> log.info("# onNext: {}" , data), error -> log.info("# onNext: {}", data), error -> log.error("# onError: {}", error));
  }
}
```

### retry

Publisher가 데이터를 emit하는 과정에서 에러가 발생하면 파라미터로 입력한 횟수만큼 원본 Flux의 Sequence를 다시 구돗한다. 만약 파라미터로 Long.MAX_VALUE를 입력하면 재구독을 무한 반복합니다.

```
@Sl4j
public class Example14_48 {
  public static void main(String[] args) throws InterruptedException{
    final int[] count = {1};
    Flux
      .range(1, 3)
      .delayElements(Duration.ofSeconds(1))
      .map(num -> {
        try {
          if (num == 3 && count[0] == 1) {
            count[0]++;
            Thread.sleep(1000);
          }
        } catch (InterruptedException e) {}
        return num;
      })
      .timeout(Duration.ofMills(1500))
      .retry(1)
      .subscribe(data -> log.info("# onNext: {}", data),
      (error -> log.error("# onError ", error)),
      () -> log.info("# onComplete"));

      Thread.sleep(7000);
  }
}
```


## 14.7 Sequence의 동작 시간 측정을 위한 Opreator

### elapsed

emit된 데이터 사이의 경과 시간을 측정해서 Tuble<Long, T> 형태로 Downstream에 emit한다.
```
@Sl4j
public class Example14_51 {
  public static void main(String[] args) throws InterruptedException{
    Flux
      .range(1, 5)
      .delayElements(Duration.ofSeconds(1))
      .elapsed()
      .subscribe(data -> log.info("# onNext: {}, time: {}", 
        data.getT2(), data.getT1()));

      Thread.sleep(6000);
  }
}
```

## 14.8 Flux Sequence 분할을 위한 Operator

### window

window(int maxSize)는 Upstreamdptj emit되는 첫 번째 데이터부터 maxSize 숫자만큼의 데이터를 포함하는 새로운 Flux로 분할한다. Reactor에서는 이렇게 분할된 Flux를 윈도우라고 부른다.

### buffer 

Upstream에서 emit되는 첫 번째 데이터부터 maxSize숫자만큼의 데이터를 List 버퍼로 한번에 emit한다.

### bufferTimeout

bufferTimeout(maxSize, maxTime) operator는 upstream에서 emit되는 첫 번째 데이터부터 maxSize숫자만큼의 데이터 또는 maxTime내에 emit된 데이터를 List버퍼로 한 번에 emit한다. 또한 maxSize나 maxTime중에서 먼저 조건에 부합할 때까지 emit된 데이터를 List버퍼로 emit한다.

### groupBy

groupBy(keyMapper) Operator는 emit되는 데이터를 keyMapper로 생성한 key를 기준으로 그룹화한 GroupedFlux를 리턴하며, 이 GroupedFlux를 통해서 그룹별로 작업을 수행할 수 있다.

## 14.9 다수의 Subscriber에게 Flux를 멀티캐스팅(Multicasting)하기 위한 Operator

Subscriber가 구독을 하면 Upstream에서 emit된 데이터가 구독 중인 모든 Subscriber에게 멀티캐스팅되는데, 이를 가능하게 해 주는 Operator들은 Cold Sequence를 Hot Sequence로 동작하게 한다.

### publish

구독하는 시점에 즉시 데이터를 emit하지 않고, connect()를 호출하는 시점에 비로소 데이터를 emit한다.

### autoConnect

구독이 발생하더라도 connect()를 직접 호출하기 전까지는 데이터를 emit하지 않기 때문에 코드상에서 connect()를 직접 호출해야 한다. 반면 autoConnect()는 파라미터로 지정하는 숫자만큼의 구독이 발생하는 시점에 Upstream소스에 자동으로 연결되기 때문에 connect()호출이 필요하지 않다.

### refCount

파라미터로 입력된 숫자만큼의 구독이 발생하는 시점에 upstream소스에 연결되며, 모든 구독이 취소하거나 Upstream의 데이터 emit이 종료되면 연결이 해제된다. 주로 무한 스트림 상황에서 모든 구독이 취소된 경우 연결을 해제하는 데 사용할 수 있다.