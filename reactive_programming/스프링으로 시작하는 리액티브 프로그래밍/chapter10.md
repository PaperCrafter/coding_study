# Scheduler

## 10.1 스레드(Thread)의 개념 이해

- **Reactor**에서 사용되는 **Schedulers**는 Reactor Sequence에서 사용되는 스레드를 관리하는 역할을 합니다.

### 물리적 스레드 VS 논리적 스레드
- **물리적 스레드**: 하드웨어에 존재하는 실제 가용한 스레드로, 병렬성을 갖추어 실제 동시 실행이 가능합니다.
  
- **논리적 스레드**: Thread Pool과 같이 사용할 수 있는 스레드로, 프로그램 내에서 세부 작업의 단위로 동시성을 제공하며, 동시에 실행되는 것처럼 보입니다.


## 10.2 Scheduler란?

- **운영체제 레벨에서의 Scheduler**: 실행 중인 프로세스를 선택하고 실행하여 프로세스의 라이프사이클을 관리합니다.

- **Reactor의 Scheduler**: 어떤 스레드에서 무엇을 처리해야 할 지를 제어합니다.


## 10.3 Scheduler를 위한 전용 Operator

Scheduler 전용 Operator에 파라미터로 적절한 Scheduler를 전달하여 사용 가능합니다.

### Operator 종류
- **subscribeOn()**: 구독이 발생한 직후 실행될 스레드를 지정하는 Operator로, 원본 Publisher의 동작을 수행하기 위한 스레드를 지정합니다.

- **publishOn()**: 코드에서 publishOn을 기준으로 아래쪽인 downstream의 실행 스레드를 변경합니다.

- **parallel()**: 물리적인 스레드에서 병렬성을 가지며, CPU 코어 수만큼의 스레드를 병렬로 실행합니다.


## 10.4 publishOn()과 subscribeOn()의 동작 이해

- **publishOn**: 한 개 이상의 publishOn을 사용하여 실행 스레드를 목적에 맞게 분리할 수 있습니다.

- **subscribeOn**: 구독 시점 직후에 실행되기 때문에, 구독된 스레드를 지정합니다.

## 10.5 Scheduler의 종류

- **Schedulers.immediate()**: 현재 스레드에서 작업을 처리하고자 할 때 사용되며, 별도의 스레드를 추가로 생성하지 않습니다.

- **Schedulers.single()**: 하나의 스레드만 생성하여 Scheduler가 제거되기 전까지 재사용합니다.

- **Schedulers.newSingle()**: 호출될 때마다 새로운 스레드를 생성하며, 데몬 스레드 여부를 선택할 수 있습니다.

- **Schedulers.boundedElastic()**: ExecutorService 기반의 스레드풀을 사용하며, 정해진 수의 스레드로 작업을 처리하고 재사용합니다.

- **Schedulers.parallel()**: Non-Blocking I/O 작업에 최적화되며, CPU 코어 수만큼의 스레드를 생성합니다.

- **Schedulers.fromExecutorService()**: 기존의 ExecutorService로부터 Scheduler를 생성하는 방식입니다.

- **Schedulers.newXXX()**: Reactor에서 제공하는 디폴트 Scheduler 인스턴스를 사용하지 않고, 새로운 Scheduler 인스턴스를 생성할 수 있습니다. 커스텀 스레드 풀을 생성할 때 사용됩니다.