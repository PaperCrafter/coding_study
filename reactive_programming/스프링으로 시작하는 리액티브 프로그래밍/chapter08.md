### 8. Backpressure

#### 8.1 Backpressure 란?

**일반적인 의미:** 가스나 액체 등의 흐름을 제어하기 위해 역으로 가해지는 압력

**리액티브 프로그래밍적 의미:** publisher가 끊임없이 emit하는 무수히 많은 데이터를 적절하게 제어하여 데이터 처리에 과부하가 걸리지 않도록 제어하는것

#### 8.2 Reactor에서의 Backpressure 처리 방식

1. **데이터 개수 제어**

   Subscriber의 request를 통해서 적절한 개수를 요청

   - `hookOnSubscribe()`: 메서드를 대신해 구독 시점에 `request` 메서드를 호출해서 최초 데이터 요청 개수를 제어한다.
   - `hookOnNext()`: emit한 데이터를 전달받아 publisher에게 데이터를 요청하는 역할을 한다.

2. **BackPressure 전략 사용.**

   - **IGNORE:** BackPressure를 적용하지 않는다.
   - **ERROR:** DownStream으로 전달한 데이터가 버퍼에 가득 찰 경우 Exception 발생 (IllegalStateException).
   - **DROP:** Downstream으로 전달할 데이터가 버퍼에 가득 찰 경우, 버퍼밖에 대기중인 먼저 emit된 데이터부터 drop 시키는 전략.
   - **LATEST:** 버퍼 밖에서 대기하는 가장 최근에 emit된 데이터부터 버퍼에 채우는 전략 (emit된 데이터가 너무 많을 경우, 버퍼에 들어가지 않은 가장 처음에 emit한 데이터부터 버린다).
   - **BUFFER:** 버퍼 안에 있는 데이터부터 drop.
   - **DROP_LATEST:** 가장 최근에 버퍼 안에 채워진 데이터를 Drop하여 폐기한 후, 확보된 공간에 emit된 데이터를 채우는 전략.
   - **DROP_OLDEST:** 가장 오래된 데이터를 Drop하여 폐기한 후, 확보된 공간에 emit된 데이터를 채우는 전략.