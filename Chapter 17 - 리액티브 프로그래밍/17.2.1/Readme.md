**17.2.1 Flow 클래스 소개**

- 자바 9에서는 리액티브 프로그래밍을 제공하는 클래스 java.util.concurrent.Flow를 추가했다.
- 이 클래스는 정적 컴포넌트 하나만 포함하고 있으며 인스턴스화할 수 없다.
- Flow 클래스의 인터페이스
  - Publisher
  - Subscriber
  - Subscription
  - Processor
- Publisher가 항목을 발행하면 Subscriber가 한 개 또는 여러개씩 항목을 소비하는데 Subscription이 이 과정을 관리할 수 있도록 FLow 클래스는 관련된 인터페이스와 정적 메서드를 제공한다.

```
//Publisher가 발행한 리스너로 Subscriber에 등록할 수 있다.
@FunctionalInterface
public interface Publisher<T> {
  void subscribe(Subscriber<? super T> s);
}

//Publisher가 관련 이벤트를 발행할 때 호출할 수 있도록 콜백 메서드 네 개를 정의
public interface Subscriber<T> {
  void onSubscribe(Subscription s);
  void onNext(T t);
  void onError(Throwable t);
  void onComplete();
}

public interface Subscription {
  void request(long n); //publisher에게 이벤트를 처리할 준비가 되었음을 알림
  void cancel(); //publisher에게 이벤트를 받지 않음을 통지
}

//리액티브 스트림에서 처리하는 이벤트의 변환단계를 나타냄
//에러나 Subscription 취소 신호 등을 전파
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> { }
```

인터페이스 구현 규칙

- Publisher는 반드시 Subscription의 request 메서드에 정의된 개수 이하의 요소만 Subscriber에 전파해야 한다.
- Subscriber는 요소를 받아 처리할 수 있음을 Publisher에 알려야 한다. 이를 통해 역압력을 행사할 수 있다.
- Publisher와 Subscriber는 정확하게 Subscription을 공유해야한다. 그러려면 onSubscribe와 onNext 메서드에서 Subscriber는 request 메서드를 동기적으로 호출할 수 있어야 한다.
