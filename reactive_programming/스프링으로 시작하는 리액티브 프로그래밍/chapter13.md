# Testing

## 13.1 StepVerifier를 사용한 테스팅

### Signal 이벤트 테스트

```
public class StepVerifierGeneralExample01Test {

  @Test
  public void sayHelloReactorTest() {
    StepVerifier
      .create(Mono.just("Hello Reactor") //sequence 생성
      .expectNext("Hello Reactor") //emit 기댓값 평가
      .expectComplete() //onComplete signal 기댓값 평가
      .verify()); //검증 실행
  }
}
```

#### expectXXX() 메서드

- expectSubscription() : 구독이 이루어짐을 기대한다.
- expectNext(T t) : onNExt Signal을 통해 전달되는 값이 파라미터로 전달된 값과 같음을 기대한다.
- expectComplete() : onComplete Signal이 전송되기를 기대한다.
- expectError() : onError Signal이 전송되기를 기대한다.
- expectNextCount(long count) : 구독 시점 또는 이전 expectNext()를 통해 기댓값이 평가된 데이터 이후부터 emit된 수를 기대한다.
- expectNoEvent(Duration duration) : 주어진 시간 동안 Signal 이벤트가 발생하지 않았음을 기대한다.
- expectAccessibleContext() : 구독 시점 이후에 Context가 전파되었음을 기대한다.
- expectNextSequence(Iterable<> extends T> iterable) : emit 된 데이터들이 파라미터로 전다로딘 Iterable의 요소와 매치됨을 기대한다.

#### verifyXXX() 메서드

- verify() : 검증을 트리거한다.
- verifyComplete() : 검증을 트리거하고, onComplete Signal을 기대한다.
- verifyError() : 검증을 트리거하고 onError Singnal을 기대한다.
- verifytTimeout(Duration duration) : 검증을 트리거하고, 주어진 시간이 초과되어도 Publisher가 종료되지 않음을 기대한다.
 

### 시간 기반(Time-based) 테스트

 StepVerifier는 가상의 시간ㅇ르 이용해 미래에 실행되는 Reactor Sequence의 시간을 앞당겨 테스트할 수 있는 기능을 지원한다.


시간 흐름 제어
```
StepVerifier
.withVirtualTime(() -> ...)

.then(() -> VirtualTimeScheduler.get().advanceTimeBy(Duration.ofHours(1))) //시간을 앞당긴다.
```


시간 제한
```
StepVerifier
.verify(Duration.ofSeconds(3))
```

### Backpressure 테스트

```
StepVerifier
.thenConsumeWhile(num -> num >= 1)
```

### Context 테스트

```
StepVerifier
.expectAccessibleContext()
.hasKey("test")
.then()
```

### Record 기반 테스트

```
StepVerifier
.recordWith() //emit 된 데이터의 기록 시작
.thenConsumeWhile() // 파라미터로 전달한 Predicate와 일치하는 데이터는 다음 단계에서 소비할 수 있도록 한다.
.consumeRecordedWith() // 컬렉션에 기록된 데이터를 소비한다.
.expectRecordedMatches() // 메서드 내에서 predicate를 사용하여 컬렉션에 기록된 데이터를 검증할 수 있다.
```


## 13.2 TestPublisher를 사용한 테스팅

reactor-test 모듈에서 지원하는 테스트 전용 Publisherdls TestPublisher를 이용해서 테스트를 진행할 수 있다.

### 정상 동작하는(Well-behaved) TestPublisher

TestPublisher를 사용하면, 개발자가 직접 프로그래밍 방식으로 Signal을 발생시키면서 원하는 상황을 미세하게 재연하메 테스트를 진행할 수 있다.

* 정상동작하는 (Well-behaved) TestPublisher의 의미: emit하는 데이터가 Null인지, 요청하는 개수보다 더 많은 데이터를 emit하는지 등의 리액티브 스트림즈 사양 위반 여부를 사전에 체크한다는 의미

TestPublisher가 발생시키는 Signal 유형
- next(T), next(T, T...): 1개 이상의 onNext Signal을 발생시킨다.
- emit(T...): 1개 이상의 onNext Singal을 발생시킨 후, onCpmplete Singnal을 발생시킵니다.
- complete(): onComplete Singnal을 발생시킨다.
- error(Throwable): onError Signal을 발생시킨다.


### 오동작하는(Misbehaved) TestPublisher

오동작 하는 TestPublisher를 사용하면 리액티브 스트림즈의 사양을 위반하는 상황이 발생하는지 테스트할 수 있다.

*오동작하는 (Well-behaved) TestPublisher의 의미: 리액티브 스트림즈 사양 위반 여부를 사전에 체크하지 않는다는 의미. 따라서 리액티브 스트림즈 사양에 위반되더라도 TestPublisher는 데이터를 emit 할 수 있다.

```
TestPublisher<Integer> source = TestPublisher.createNoncompliant(TestPublisher.Violation.ALLOW_NULL);
```

오동작 하는 TestPublisher를 생성하기 위한 위반 조건
- ALLOW_NULL: 전송할 데이터가 null이어도 NullpointerException을 발생시키지 않고 다음 호출을 진행할 수 있다.
- CLEANUP_ON_TERMINATE: onComplete, onError, emit 같은 Termninal Signal을 연달아 여러 번 보낼 수 있도록 한다.
- DEFER_CANCELLATION: cancel Signal을 무시하고 계속해서 Signal을 emit할 수 있도록 한다.
- REQUEST_OVERFLOW: 요청 개수보다 더 많은 Signal이 발생하더라도 IllegalStateException을 발생시키지 않고 다음 호출을 진행할 수 있도록 한다.


## 13.3 PublisherProbe를 사용한 테스팅

reactor-test 모듈은 PublisherProbe를 이용해 Sequencedml 실행 경로를 테스트 할 수 있다.
주로 조건에 따라 Sequence가 분기되는 경우, Sequence의 실행 경로를 추적해서 정상적으로 실행되었는지 테스트할 수 있다.