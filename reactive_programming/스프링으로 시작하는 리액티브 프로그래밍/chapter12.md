# Debugging

## 12.1 Reactor에서의 디버깅 방법

### 12.1.1 Debug Mode를 사용한 디버깅

Hooks.onOperatorDebug() 로 Debug 모드를 활성화 할 수 있다. 하지만 이는 다음과 같은 한계를 가진다.
- 애플리케이션 내에 있는 모든 Operator의 스택트레이스를 캡쳐한다.
- 에러가 발생하면 캡쳐한 정보를 기반으로 에러가 발생한 Assembly(Operator에서 리턴하는 새로운 Mono 또는 Flux가 선언된 지점)의 스택트에리스를 원본 스택트레이스 중간에 끼워넣는다.

Traceback: 디버그 모드 활성화 후, 에러가 발생한 Operator의 스택트레이스를 캡쳐한 Assembly 정보를 Traceback이라고 한다.
프로덕션 환경에서의 디버깅 설정: io.projectreactor:reactor-tools 를 추가하여 ReactorDebugAgent를 활성화 시킬 수 있다.


### 12.1.2 Checkpoint() Operator를 사용한 디버깅

checkPoint() Operator를 사용하면 특정 Operator 체인 내의 스택트레이스만 캡쳐한다.

- Traceback을 출력하는 방법: checkpoint()
- Traceback 출력 없이 식별자를 포함한 Description을 출력해서 에러 발생 지점을 예상하는 방법: checkpoint("description") 으로 에러가 발생할 경우 stacktrace에 description이 나타난다.
- Traceback과 Description을 모두 출력하는 방법: checkpoint("description", true)


### 12.1.3 log() operator를 사용한 디버깅

log() Operator는 Reactor Sequence의 동작을 로그로 출력하는데, 이 로그를 통해 디버깅이 가능하다.

