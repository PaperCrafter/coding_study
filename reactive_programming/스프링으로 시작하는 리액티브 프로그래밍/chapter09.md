# 09 Sinks

## 9.1 Sinks란?

**Sinks**는 Processor의 기능을 개선한 API로, 3.5.0부터 Processor는 완전히 제거되었습니다. Sinks는 리액티브 스트림즈의 Signal을 프로그래밍 방식으로 푸시할 수 있는 구조이며, Flux 또는 Mono의 의미 체계를 지닌다. 전통적인 `generate()` operator와 `create()` operator는 싱글 스레드 기반에서 시그널을 전송하는 데 사용되었으나, Sinks는 멀티스레드 방식으로 동작해도 thread safe하다.

## 9.2 Sinks 종류 및 특징

### 1. Sinks.One

 Sinks.one() 메서드를 사용해서 한 건의 데이터를 전송하는 방법을 정의해 둔 기능 명세

```
public final class Sinks {

  public static <T> Sinks.One<T> one() {
    return SinksSpecs.DEFAULT_ROOT_SPEC.one();
  }
}
```
한 건의 데이터를 프로그래밍 방식으로 emit 한다.

```
public class Example9_4 {
  publicv static void main(String[] args) throws InterruptedException {
    Sinks.One<String> sinkOne = Sinks.one();
    Mono<String> mono = sinkOne.asMono():
    sinkOne.emitValue("Hello Reactor", FAIL_FAST);

    mono.subscribe(data -> log.info("# Subscriber1 ", data));
    # mono.subscribe(data -> log.info("# Subscriber2 ", data));
  }

}
```

FAIL_FAST

EmitFailureHandler 인터페이스의 구현체. 이 EmitFalureHandler객체를 통해서 emit 도중 발생한 에러에 대해 빠르게 실패 처리를 한다. 빠륵 ㅔ실패 처리한다는 의미는 에러가 발생했을 때 재시도를 하지 않고 즉시 실패 처리를 한다는 의미이다. 이렇게 빠른 실패 처리를 함으로써 스레드 간의 경합 등으로 발생하는 race condition 을 방지할 수 있고 결과적으로 스레드 안정성을 보장하기 위함이다.

Sinks.One은 처음 emit한 데이터는 정상적으로 emit되지만, 나머지 데이터들은 Drop된다.

### 2. Sinks.Many

```
public final class Sinks {

  public static ManySpec many() {
    return SinksSpecs.DEFAULT_ROOT_SPEC.many();
  }
}
```

ManySpec 인터페이스를 리턴

```
public final class Sinks {

  public interface ManySpec {
    UnicastSpec unicast();
    MulticastSpec multicast();
    MulticastReplaySpec replay();
  }
}
```


1. UnicastSpec
```
public class Example9_8 {
  public static void main(String[] args) throws InterruptedException {
    Sinks.Many<Integer> unicastSink = Sinks.many().unicast().onBackpressureBuffer();
    Flux<Integer> fluxView = unicastSink.asFlux();

    unicastSink.emitNext(1, FAIL_FAST);
    unicastSink.emitNext(2, FAIL_FAST);

    fluxView.subscribe(data -> log.info("# Subscriber1: {}", data));

    unicastSink.emitNext(3, FAIL_FAST);

    # fluxView.subscribe(data -> log.info("# Subscriber2: {}", data));
  }

}
```

UnicastSpec의 기능은 단 하나의 Subscriber에게만 데이터를 emit하는 것


2. MulticastSpec
```
public class Example9_9 {
  public static void main(String[] args) {
    Sinks.Many<Integer> multicastSink = Sinks.many().multicast().onBackpressureBuffer();
    Flux<Integer> fluxView = multicastSink.asFlux();

    multicastSink.emitNext(1, FAIL_FAST);
    multicastSink.emitNext(2, FAIL_FAST);

    fluxView.subscribe(data -> log.info("# Subscriber1: {}", data));

    multicastSink.emitNext(3, FAIL_FAST);

    fluxView.subscribe(data -> log.info("# Subscriber2: {}", data));
  }

}
```

hot publisher로 동작하며 여러개의 subscriber에 emit이 가능하다.


3. MulticastReplaySpec
```
public class Example9_9 {
  public static void main(String[] args) {
    Sinks.Many<Integer> replaySink = Sinks.many().replay().limit(2);
    Flux<Integer> fluxView = multicastSink.asFlux();

    replaySink.emitNext(1, FAIL_FAST);
    replaySink.emitNext(2, FAIL_FAST);
    replaySink.emitNext(3, FAIL_FAST);

    fluxView.subscribe(data -> log.info("# Subscriber1: {}", data));

    replaySink.emitNext(4, FAIL_FAST);

    fluxView.subscribe(data -> log.info("# Subscriber2: {}", data));
  }

}
```

emit된 데이터도 다시 replay해서 구독 전에 이미 emit된 데이터도 Subscriber가 전달받을 수 있다. 동작하며 여러개의 subscriber에 emit이 가능하다.

all(): 모든 데이터를 전부 emit한다.
limit(n): emit된 데이터 중에서 파라미터로 입력한 개수만큼 가장 나중에 emit된 데이터 부터 Subscriber에게 전달하는 기능을 한다.