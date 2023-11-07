### 7. Cold Sequence vs Hot Sequence

#### 7.1 Cold와 Hot의 의미

**cold와 hot이 들어가는 다양한 예시**
- hot swap: 전원이 켜진 상태에서 디스크 등의 장비 교체
- hot deploy: 서버를 재시작 하지 않고 애플리케이션 배포
- cold wallet: 안호화폐에서 인터넷에 다시 연결해야 하는 지갑

**cold와 hot의 의미**
- cold: 무언가를 새로 시작한다
- hot: 무언가를 샐호 시작하지 않는다

#### 7.2 Cold Sequence

Subscribe 시에 기존에 emit했던 데이터들을 새로운 subscriber에게 모두 emit합니다.

**일반 Mono**
**일반 Flux**

#### 7.3 Hot Sequence

Subscribe 시에 subscribe 시점부터 emit한 데이터들만 받을 수 있습니다.

**Flux.share** 

#### 7.4 Http요청과 응답에서 Cold Sequence와 Hot Sequence의 동작 흐름

**Mono.cache** - 매번 새로 생성하지 않고 캐싱된 것을 emit합니다. 

cold sequence -> hot sequence로 전환시켜줍니다. 처음부터 받을 수 있다는 점에서 cold 같지만, 미리 캐싱된 데이터를 사용한다는 점에서 hot이다.

