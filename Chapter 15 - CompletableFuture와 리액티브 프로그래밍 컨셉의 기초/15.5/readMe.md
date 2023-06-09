# 15.5

Future와 CompletableFuture는 독립적 실행과 병렬성이라는 정식적 모델에 기반한다.

따라서 Future는 한번만 실행해 결과를 제공.

스트림은 선형적 파이프라인 처리 기법에 알맞다.

자바9 에서는 java.util.concurrent.Flow 의 인터페이스에 발행-구독 모델을 적용해 리액티브 프로그래밍을 제공한다.

- 구독자가 구독할 수 있는 발행자
- 이 연결을 구독 subscription 이라 한다.
- 이 연결을 이용해 메세지를 전송

발행자-구독자 모델

**1 - 두 플로를 합치는 예제**

ex) 두 정보 소스로부터 발생하는 이벤트를 합쳐서 다른 구독자가 볼 수 있도록 발행하는 예

수식을 포함하는 스프레드 시트의 셀에서 흔히 제공하는 동작

"=C1+C2" 공식을 포함하는 스프레드시트 셀 C3

```java
privateclassSimpleCell {
privateint value = 0;
private String name;
publicSimpleCell(String name) {
this.name = name;
    }
}
```

c1,c2에 이벤트가 발생했을때 c3를 구독하도록 하는 인터페이스 Publisher<T>

```java
// 통신할 수독자 수를 인수로 받는다interfacePublisher<T> {
voidsubscribe(Subscriber<?super T> subscriber);
}
// 정보 전달할 단순메서드 포함interfaceSubscriber<T> {
voidonNext(T t);
}
```

Cell 은 Publisher이면서 Subscriber이다

```java
privateclassSimpleCellimplementsPublisher<Integer>,Subscriber<Integer> {
privateint value = 0;
private String name;
private List<Subscriber> subscribers =new ArrayList<>();

publicSimpleCell(String name) {
this.name = name;
 }
 @Override
publicvoidsubscribe(Subscriber<?super Integer> subscriber) {
  subscribers.add(subscriber);
 }
privatevoidnotifyAllSubscribers() {//새 값이 있음을 알림
  subscribers.forEach(subscriber -> subscriber.onNext(this.value));
 }
 @Override
publicvoidonNext(Integer newValue) {
this.value = newValue;//값을 갱신
  System.out.println(this.name + ":" +this.value);
  notifyAllSubscribers();// 구독자에 갱신되었음을 알림
 }
}
```

다음과 같이 시도

```java
Simplecell c3 =new SimpleCell("C3");
SimpleCell c2 =new SimpleCell("C2");
SimpleCell c1 =new SimpleCell("C1");
c1.subscribe(c3);
c1.onNext(10);// C1의 값 10으로 갱신
c2.onNext(20);
```

계산 구현 클래스

```java
publicclassArithmeticCellextendsSimpleCell {
privateint left;
privateint right;
publicArithmeticCell(String name) {
super(name);
           }
publicvoidsetLeft(int left) {
this.left = left;
           onNext(left +this.right);
      }

publicvoidsetRight(int right) {
this.right = right;
           onNext(right +this.left);
      }
 }
```

다음과 같이 시도

```java
ArithmeticCell c3 =new ArithmeticCell("C3");
SimpleCell c2 =new SimpleCell("C2");
SimpleCell c1 =new SimpleCell("C1");
c1.subscribe(c3::setLeft);
c2.subscribe(c3::setRight);
c1.onNext(10);
c2.onNext(20);
c1.onNext(15);
```

출력결과

C1: 10

C3: 10

C2: 20

C3: 30

C1: 15

C3: 35

```java
// C5=C3+C4, 의존하는 새로운 셀 C5

ArithmeticCell c5 =new ArithmeticCell("C5");
ArithmeticCell c3 =new ArithmeticCell("C3");
SimpleCell c4 =new SimpleCell("C4");
SimpleCell c2 =new SimpleCell("C2");
SimpleCell c1 =new SimpleCell("C1");
c1.subscribe(c3::setLeft);
c2.subscribe(c3::setRight);
c3.subscribe(c5::setLeft);
c4.subscribe(c5::setRight);

c1.onNext(10);
c2.onNext(20);
c1.onNext(15);
c4.onNext(1);
c4.onNext(3);

```

출력결과

C1:10

C3:10

C5:10

C2:20

C3:30

C5:30

C1:15

C3:35

C5:35

C4:1

C5:36

C4:3

C5:38

**2 - 역압력**

정보의 흐름 속도를 역압력(흐름제어)으로 제어

Subscriber 에서 Publisher로 정보를 요청해야 할 필요가 있다.

Publisher는 여러 Subscriber를 갖고 있어 역압력 요청이 한 연결에만 영향을 미쳐야 한다는게 문제가 될수 있다.

자바9 플로 API의 Subscriber 인터페이스의 네번째 메서드

void onSubscribe(Subscription subscription);

둘 사이 채널이 연결되면 첫이벤트로 이 메서드가 호출된다.

Subscrption객체는 다음처럼 서로 통신할 수 있는 메서드를 포함한다.

```java
interfaceSubscription {
voidcancel();
voidrequest(long n);
}
```

콜백을 통한 '역방향' 소통으로

Publisher는 Subscription 객체를 만들어 Subscriber로 전달하면

Subscriber는 이를 이용해 Publisher로 정보를 보낼 수 있다.

**3 - 실제 역압력의 간단한 형태**

한번에 한개의 이벤트를 처리하도록 발행-구독 연결을 구성하려면?

- Subscriber가 OnSubscribe 로 전달된 Subscription 객체를 subscription 같은 필드에 로컬로 저장
- Subscriber가 수많은 이벤트를 받지 않도록 onSubscribe, onNext, onError의 마지막 동작에channel, request(1)을 추가해 오직 한 이벤트만 요청
- 요청을 보낸 채널에만 onNext, onError 이벤트를 보내도록 Publisher의 notifyAllSubscribers 코드를 바꿈(보통 여러 Subscriber가 자신만의 속도를 유지할 수 있도록 Publisher는 새 Subscription을 만들어 각 Subscriber와 연결)

역압력을 구현하는데 고려해야할 장단점

- 여러 Subscriber가 있을때 이벤트를 가장 느린속도로 보낼 것인가?각 Subscriber에게 보내지 않은 데이터를 저장할 별도의 큐를 가질 것인가?
- 큐가 너무 커지면 어떻게 해야할까?
- Subscriber가 준비가 안 되었다면 큐의 데이터를 폐기할 것인가?
