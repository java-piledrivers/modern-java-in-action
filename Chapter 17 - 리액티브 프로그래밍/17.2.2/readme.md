

먼저 현재 보고된 온도를 전달하는 간단한 클래스를 정의한다.

- TempInfo. 원격 온도계를 흉내낸다.(0 ~ 99 사이의 화씨 온도를 임의로 만들어 연속으로 보고)
- TempSubscriber. 레포트를 관찰하면서 각 도시에 설치된 센서에서 보고된 온도 스트림을 출력한다.

```java
import java.util.Random;

@AllArgsConstructor
public class TempInfo {
  public static final Random random = new Random();

  private final String town;
  private final int temp;

  public static TempInfo fetch(String town) {
    if (random.nextInt(10) == 0)
      throw new RuntimeException("Error!");//10% 확률로 실패

    return new TempInfo(town, random.nextInt(100));
  }
}
```

Subscriber가 요청할 때마다 도시의 온도를 전송하도록 Subscription을 구현한다.

```java
import java.util.concurrent.Flow.*;

@AllArgsConstructor
public class TempSubscription implements Subscription {
  private final Subscriber<? super TempInfo> subscriber;
  private final String town;

  @Override
  public void request(long n) {
    executor.submit( () -> {//다른 스레드에서 다음 요소를 구독자에게 보낸다.
      for (long i = 0L; i < n; i++) {
        try {
          subscriber.onNext(TempInfo.fetch(town));//현재 온도를 subscriber로 전달
        } catch (Exception e) {
          subscriber.onError(e);//온도 가져오기 실패하면 Subscriber로 에러 전달break;
        }
      }
    });
  }

  @Override
  public void cancel() {
    subscriber.onComplete();//구독 취소되면 완료신호를 Subscriber로 전달
  }
}
```

새 요소를 얻을 때마다 Subscription이 전달한 온도를 출력하고 새 레포트를 요청하는 Subscriber 클래스를 구현한다.

```java
import java.util.concurrent.Flow.*;

public class TempSubscriber implements Subscriber<TempInfo> {
  private Subscription subscription;

  @Override
  public void onSubscribe(Subscription subscription) {// 구독을 저장하고 첫 번째 요청을 전달this.subscription = subscription;
    subscription.request(1);
  }

  @Override
  public void onNext(TempInfo tempInfo) {// 수신한 온도를 출력하고 다음 정보를 요청
    System.out.println(tempInfo);
    subscription.request(1);
  }

  @Override
  public void onError(Throwable t) {
    System.out.println(t.getMessage());
  }

  @Override
  public void onComplete() {
    System.out.println("done!");
  }
}
```

리액티브 애플리케이션이 실제 동작할 수 있도록 Publisher를 만들고 TempSubscriber를 이용해 Publisher에 구독하도록 Main 클래스를 구현한다.
```java
public class Main {
  public static void main(String[] args) {
//뉴욕에 새 Publisher를 만들고 TempSubscriber를 구독시킴
    getTemperatures("New York").subscribe(new TempSubscriber());
  }

  private static Publisher<TempInfo> getTemperatures(String town) {
//구독한 Subscriber에게 TempSubscription을 전송하는 Publisher를 반환return subscriber -> 
subscriber.onSubscribe(
        new TempSubscription(subscriber, town)
    );
  }
}
```
