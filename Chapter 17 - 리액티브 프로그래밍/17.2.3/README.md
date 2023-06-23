# 17.2.3 Processor로 데이터 변환하기

앞선 장에선 `Publisher`, `Subscriber`, `Subscription`을 살펴봤는데 이제는 `Processor`을 살펴볼 차례이다.  

`Processor`의 목적은 `Publisher`를 구독한 다음 수신한 데이터를 다시 제공하는 것이다.  

아래는 화씨로 제공된 데이터를 섭씨로 변환해 다시 보여주는 코드이다.  

<br/>

```java
import java.util.concurrent.Flow.Processor;
import java.util.concurrent.Flow.Subscriber;
import java.util.concurrent.Flow.Subscription;

/**
 * TempInfo 를 다른 TempInfo 로 전달하는 프로세서
 * 우리가 만든 것은 화씨를 섭씨로 변환
 */
public class TempProcessor implements Processor<TempInfo, TempInfo> {

    private Subscriber<? super TempInfo> subscriber;

    @Override
    public void subscribe(Subscriber<? super TempInfo> subscriber) {
        this.subscriber = subscriber;
    }
    
    @Override
    public void onNext(TempInfo item) {
        subscriber.onNext(new TempInfo(item.getTown(),
                (item.getTemp() - 32) * 5 / 9));  // 섭씨로 변환한 다음 TempInfo 로 다시 전송
    }

    // 다른 신호들은 업스트림 구독자에 전달
    
    @Override
    public void onSubscribe(Subscription subscription) {
        subscriber.onSubscribe(subscription);
    }

    @Override
    public void onError(Throwable throwable) {
        subscriber.onError(throwable);
    }

    @Override
    public void onComplete() {
        subscriber.onComplete();
    }
}
```

<br/>

`TempProcessor`에서 `onNext` 메서드는 화씨를 섭씨로 변환한 다음 온도를 재전송하는 역할을 한다.  

이 메서드를 제외한 다른 메서들은 모두 단순히 모든 신호를 업스트림 `Subscriber`로 전달하며 `Publisher`의 `subscribe` 

메서드는 업스트림 `Subscriber`를 `Processor`로 등록하는 동작을 수행한다.  

아래 코드는 17.2.2 에 사용했던 코드와는 달리 섭씨 온도를 보여준다. 

```java
import java.util.concurrent.Flow.Publisher;

public class Main {
    public static void main(String[] args) {
        getTemperatures("Busan").subscribe(new TempSubscriber());
    }

    private static Publisher<TempInfo> getTemperatures(String town) {
        return subscriber -> {
            TempProcessor processor = new TempProcessor();
            processor.subscribe(subscriber);
            processor.onSubscribe(new TempSubscription(processor, town));
        };
    }
}
```

<br/>

```text
현재 Busan의 온도는 35입니다.
현재 Busan의 온도는 17입니다.
현재 Busan의 온도는 9입니다.
현재 Busan의 온도는 6입니다.
현재 Busan의 온도는 -13입니다.
현재 Busan의 온도는 5입니다.
현재 Busan의 온도는 17입니다.
현재 Busan의 온도는 25입니다.
Error
```

<br/>

이렇게 플로 API에 정의된 인터페이스를 직접 구현하면서 플로 API의 핵심 사상을 구성하는 발행-구독 프로토콜을 이용해 비동기 
스트림 처리를 살펴보았다.  

하지만 왜 자바는 플로 API 구현을 제공하지 않을까?  

이유는 다음절에서 알아보자.
