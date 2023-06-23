# 17.3 리액티브 라이브러리 RxJava 사용하기
RxJava는 자바로 리액티브 애플리케이션을 구현하는 데 사용하는 라이브러리다.<br>
<br>
RxJava는 Flow.Publisher를 구현하는 두 클래스를 제공한다.<br>
역압력 기능이 있는 Flow를 포함하는 io.reactivex.Flowable 클래스,<br>
역압력을 지원하지 않는 io.reactivex.Observable 클래스이다.<br>
<br>
역압력은 Publisher가 너무 빠른 속도로 데이터를 발행하면서 <br>
Subscriber가 이를 감당할 수 없는 상황에 이르는 것을 방지하는 기능이다.<br>
<br>
## 17.3.1 Observable 만들고 사용하기
Observable, Flowable 클래스는 여러 팩토리 메서드를 제공한다. 아래는 몇가지 예제이다.<br>
<br>
```java
    Observable<String> strings = Observable.just("first", "second");
```
just() 팩토리 메서드는 한 개 이상의 요소를 이용해 이를 방출하는 Observable로 변환한다.<br>
위 Observable은 onNext("first"), onNext("second"), onComplete()의 순서로 방출한다.<br>
<br>
```java
    Observable<Long> onePerSec = Observable.interval(1, TimeUnit.SECONDS);
```
interval() 팩토리 메서드는 onePerSec라는 변수로 Observable을 반환해 할당한다.<br>
위 Observable은 0에서 시작해 1초 간격으로 long 형식 값을 무한히 방출한다.<br>
<br>
다음은 Observer 인터페이스 코드다.
```java
    public interface Observer<T> {
	void onSubscribe(Disposable d);
	void onNext(T t);
	void onError(Throwable t);
	void onComplete();
}
```
RxJava에선 Observable이 Publisher 역할, Observer가 Subscriber 역할을 한다.<br>
RxJava의 API는 자바 9 네이티브 플로 API와 동일한 역할을 하지만, 많은 오버로드된 기능을 제공하여 더 유연하다.<br>
예를 들어 네 개의 메서드를 모두 구현해야 하는 Flow.Subscriber와 달리, <br>
다른 세 메서드는 생략하고 onNext의 시그니처에 해당하는 람다 표현식만 전달해 Observable을 구독할 수 있다.<br>
이때 완료, 에러 처리 메서드는 아무것도 하지 않는 기본 동작을 가진다.<br>
아래는 작성 예제이다.
```java
    Observable<Long> onePerSec = Observable.interval(1, TimeUnit.SECONDS);

    onePerSec.subscribe(i -> System.out.println(TempInfo.fetch("New York"));
```
---
<br>이제 RxJava의 리액티브 스트림의 구현을 이용해 온도 보고 시스템을 정의한다.<br><br>
아래는 5번 온도를 방출하고 종료시키는 Observable을 반환하는 예제이다.<br>
emitter는 새 Disposable을 설정하는 메서드와 시퀀스가 이미 다운스트림을 폐기했는지 확인하는 메서드 등을 제공한다.
```java
private static Observable<TempInfo> getTemperature(String town) {
    return Observable.create(emitter -> //Observer를 소비하는 함수로부터 Observable 만들기
            Observable.interval(1, TimeUnit.SECONDS) //매 초마다 무한으로 증가하는 long값을 방출하는 Observable
            .subscribe(i -> {
                if(!emitter.isDisposed()) { //소비된 Observer가 아직 폐기되지 않았으면 작업 수행
                if (i >= 5) { // 5번 온도보고했으면
                    emitter.onComplete(); //옵저버를 완료하고 스트림 종료
                } else {
                    try {
                    emitter.onNext(TempInfo.fetch(town)); //아니면 온도를 보고
                    } catch (Exception e) {
                    emitter.onError(e); //에러 발생하면 Observer에 알림
                    }
                }
                }
            })
    );
}
```

<br>아래는 getTemperature 메서드가 반환하는 Observable을 구독할 Observer 작성 예제이다.<br>
TempSubscriber() 클래스-책546p-와 비슷하지만 더 단순하다.<br>
RxJava의 Observable은 역압력을 지원하지 않으므로, 요소 처리 후 추가 요소를 요청하는 request() 메서드가 필요 없기 때문이다.
```java
    public class TempObserver implements Observable<TempInfo> {

        @Override
        public void onSubscribe(Disposable disposable) {
        }

        @Override
        public void onNext(TempInfo tempInfo) {
            System.out.println(tempInfo);
        }

        @Override
        public void onError(Throwable throwable) {
            System.out.println("Got problem: " + throwable.getMessage());
        }

        @Override
        public void onComplete() {
            System.out.println("Done!");
        }
    }
```
<br>이제 메인 프로그램을 만들어 위 예제의 Observer로, 위위 예제의 getTemperature 메서드가 반환하는 Observable을 구독한다.
```java
    public class Main{
        public static void main(String[] args) {
            Observable<TempInfo> observable = getTemperature("New York"); //매 초마다 뉴욕 온도를 방출하는 Observable
            observable.blockingSubscribe(new TempObserver()); //단순 Observer로 이 Observable을 구독해 온도 출력
        }
    }

    New York: 69
    New York: 26
    New York: 85
    New York: 94
    New York: 29
    Done!
```
