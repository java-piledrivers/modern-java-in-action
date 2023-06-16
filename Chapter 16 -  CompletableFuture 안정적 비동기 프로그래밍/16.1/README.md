# 16.1 Future의 단순 활용


자바 5부터는 작업을 걸어두고 미래의 원하는 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공한다.  

**Future 인터페이스**

```java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```


시간이 걸릴 수 있는 작업을 `Future` 내부로 설정하면 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 수행할 수 있다. `Future`를 이용하려면 시간이 오래걸리는 작업을 `Callable` 객체 내부로 감싼 다음에 `ExecutorService`에 제출해야 한다.  



**Future 예시 코드**

```java
    ExecutorService executor = Executors.newCachedThreadPool();
    Future<Double> future = executor.submit(new Callable<double>() {
        public Double call() {
            return doSomeLongComputation();// 시간이 오래걸리는 작업
        }
    });
    doSomethingElse(); // 비동기 작업을 수행하는 동안 다른 작업을 수행.
    try {
    	// 비동기 작업을 가져오는데, 
        // 만약 결과가 준비되지 않았으면 호출 스레드가 블록되고, 결과를 기다린다. 
        // 최대 1초까지
        Double result = future.get(1,TimeUnit.SECONDS);
    } catch(ExecutionException ee){
        // 계산 중 예외 발생
    } catch (InterruptedException ie){
        // 현재 스레드에서 대기 중 인터럽트 발생
    } catch (TimeoutException te){
        // Future가 완료되기 전에 타임아웃 발생
    }
```

`ExecutorService`에서 제공하는 스레드가 시간이 오래 걸리는 작업을 처리하는동안 우리 스레드로 다른 작업을 동시에 실행할 수 있다. 다른 작업을 처리하다가 시간이 오래 걸리는 작업의 결과가 필요한 시점이 되었을 떄 `Future`의 `get` 메서드로 결과를 가져올 수 있다.  
`get` 메서드를 호출해 결과를 가져올 때 결과가 준비되어 있다면 바로 가져오지만 만약 결과가 준비되지 않았다면 블록 시키고 결과를 기다린다. 그런데 아무리 기다려도 작업이 안끝나고 결과가 나오지 않는 불상사가 있을 수 있기 때문에 `get` 메서드를 오버로드 하여 스레드가 대기할 때 최대 타임아웃 시간을 설정하는 것이 좋다.  


## 16.1.1 Future 제한

`Future` 인터페이스로는 비동기 계산이 끝났는지 확인 하는 `isDone` 메서드, 결과를 회수하는 `get` 메서드 그리고 도중에 작업을 중단하는 `cancel` 메서드를 제공하는데 이들 메서드 만으로는 간결한 동시 실행 코드를 구현하기가 충분하지 않다.  
예를 들면 여러 `Future` 결과가 있을 때 이들의 의존성을 표현하기가 어렵다. 다시 말해 "오래 걸리는 작업 A의 계산이 끝나고 결과가 나오면 다른 오래 걸리는 작업 B로 전달하고, B의 결과가 나오면 다른 질의의 결과와 B의 결과를 조합하시오"를 수행하기 어렵다.

아래와 같은 선언형 기능이 있다면 유용할 것이다.  

* 두 개의 비동기 계산 결과를 하나로 합친다. 두 가지 계산은 서로 독립적이거나 의존하는 상황일 수 있다.
* `Future` 집합이 실행하는 모든 작업의 완료를 기다린다. 
* `Future` 집합에서 가장 빨리 완료되는 작업을 기다렸다가 결과를 얻는다.
* 프로그램적으로 `Future`를 완료시킨다. (비동기 동작에 수동으로 결과를 제공)
* `Future` 완료 동작에 반응한다. (즉, 결과를 기다리면서 블록되지 않고 결과가 준비되었다는 알림을 받은 다음에 `Future`의 결과로 원하는 추가 동작을 수행할 수 있음)

지금까지 설명한 기능을 선언형으로 이용할 수 있도록 `Future` 인터페이스를 구현한 `CompletableFuture` 클래스를 살펴보도록 하자.  
`Stream`과 `CompletableFuture`는 비슷한 패턴, 즉 람다 파이프라이닝을 활용한다. 따라서 `Future`와 `CompletableFuture`의 관계를 `Collection` 과 `Stream`의 관계에 비유할 수 있다.


<br>


## 16.1.2 CompletableFuture로 비동기 애플리케이션 만들기

어떤 제품이나 서비스를 이용해야 하는 상황이라고 가정하자. 예산을 줄일 수 있도록 여러 온라인상점 중 가장 저렴한 가격을 제시하는 상점을 찾는 어플리케이션을 완성해가는 예제를 이용해서 `CompletableFuture`의 기능을 살펴보자. 이 어플리케이션을 만드는 동안 다음과 같은 기술을 배울 수 있다. 

1. 고객에게 비동기 API 를 제공하는 방법
2. 동기 API를 사용해야 할 때 코드를 비블록으로 만드는 방법
3. 비동기 동작의 완료에 대응하는 방법

