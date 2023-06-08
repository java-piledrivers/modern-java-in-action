# 15.2 동기 API와 비동기 API

다음은 같은 시그니처를 갖는 `f`, `g` 두 메서드의 호출을 합하는 예제이다.
    
```java
int y = f(x);
int z = g(x);
System.out.println(y + z);
```

`f`와 `g`를 실행하는데 오래 걸린다고 가정하고, 서로 상호작용을 하지 않는다면 별도의 CPU로 실행함으로써 합계를 구하는 시간을 단축시킬 수 있다.

```java
class ThreadExample {

    public static void main(String[] args) throws InterruptedException {
        int x = 1337;
        Result result = new Result();
      
        Thread t1 = new Thread(() -> { result.left = f(x); });
        Thread t2 = new Thread(() -> { result.right = g(x); });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(result.left + result.right);
    }

    private static class Result {
        private int left;
        private int right;
    }
}
```

이때 Runnable 대신 Future API 인터페이스를 이용해 코드를 더 단순화 할 수 있다.

```java
public class ExecutorServiceExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        int x = 1337;

        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Future<Integer> y = executorService.submit(() -> f(x));
        Future<Integer> z = executorService.submit(() -> g(x));
        System.out.println(y.get() + z.get());

        executorService.shutdown();
    }
}
```

- 하지만 여전히 `submit` 메서드 호출과 같은 불필요한 코드로 오염된다 
- 명시적 반복으로 병렬화를 수행하던 코드를 스트림을 이용해 내부 반복으로 바꾼 것처럼 비동기 API라는 기능으로 API를 바꿔서 해결한다

## 15.2.1 Future 형식 API
대안을 이용하면 `f`, `g`의 시그니처가 다음처럼 바뀐다
    
```java
Future<Integer> f(int x);
Future<Integer> g(int x);
```

그리고 다음처럼 호출이 바뀜
```java
Future<Integer> y = f(x);
Future<Integer> z = g(x);
System.out.println(y.get() + z.get());
```
`f`에만 Future를 적용하면서 `g`를 그대로 호출할 수도 있지만, 두 가지 이유로 이런 방식을 사용하지 않는다.
- 다른 상황에서는 `g`에도 Future 형식이 필요할 수도 있다.
- 병렬 하드웨어 프로그램 실행 속도를 높이려면 합리적인 크기의 태스크로 나누는 것이 좋다.

## 15.2.2 리액티브 형식 API
두 번째 대안은 `f`, `g` 시그니처를 바꿔서 콜백 형식의 프로그래밍 이용

```java
void f(int x, Consumer<Integer> dealWithResult);
```

`f`가 값을 반환하지 않는데 어떻게 프로그램이 동작할까?   
-> `f`에 추가 인수로 콜백(람다)을 전달해서 f의 바디에서는 return 문으로 결과를 반환하는 것이 아니라, 결과가 준비되면 이를 람다로 호출하는 태스크를 만드는 것이 비결이다

```java
public class CallbackStyleExample {
  public static void main(String[] args) {
    int x = 1337;
    Result result = new Result();
    
    f(x, (int y) -> {
      result.left = y;
      System.out.println(result.left + result.right);
    });
    
    
    g(x, (int z) -> {
      result.right = z;
      System.out.println(result.left + result.right);
    });
  }
}
```
하지만 결과가 달라졌다. 
- `f`와 `g`의 호출 합계를 정확하게 출력하지 않고 상황에 따라 먼저 계산된 결과를 출력한다
- Lock을 사용하지 않으므로 값을 두 번 호출한다
- 때로는 +에 제공된 두 피연산자가 println이 호출되기 전에 업데이트될 수도 있다.

다음 두 가지 방법으로 이 문제를 보완할 수 있다.
- if-then-else를 이용해 적절한 락을 이용해 두 콜백이 모두 호출되었는지 확인한 다음 원하는 기능을 수행한다.
- 리액티브 형식의 API는 보통 **한 결과가 아니라 일련의 이벤트에 반응하도록 설계되었으므로** Future를 이용하는 것이 더 적절하다

## 15.2.3 잠자기(그리고 기타 블록킹 동작)는 해로운 것으로 간주
- 스레드 풀에서 sleep을 호출하면 스레드가 블록되어 다른 태스크를 실행할 수 없다
- 이상적으로는 블록킹 동작을 수행하는 스레드는 없애거나 코드에서 예외를 일으켜야 한다
- 태스크를 앞과 뒤 두 부분으로 나누고 블록되지 않을 때만 뒷 부분을 자바가 스케줄링하도록 요청할 수 있다

다음은 한 개의 작업을 갖는 코드 A

```java
work1();
Thread.sleep(10000);
work2();
```

코드 B와 비교해보자
```java
public class ScheduledExecutorServiceExample {
    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executor.newScheduledThreadPool(1);

        work1();
        scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2, 10,
            timeUnit.SECONDS);

        scheduledExecutorService.shutdown();
    }

    public static void work1() {
        System.out.println("Hello from Work1!");
    }

    public static void work2() {
      System.out.println("Hello from Work2!");
    }
}
```

- 코드 A는 Sleep시 스레드가 블록되어 다른 태스크를 실행할 수 없다
- 코드 B는 스레드가 블록되지 않고 다른 태스크를 실행할 수 있다

## 15.2.4 현실성 확인
- 현실적으로 블록되는 동작을 제거하는 것은 불가능하다
- 자바의 개선된 동시성 API를 이용해 유익을 얻을 수 있는 상황을 찾아보고 사용한다
- 네트워크 서버의 블록/논블록 API를 일관적으로 제공하는 Netty 같은 새로운 라이브러리를 사용하면 블록되는 동작을 최소화할 수 있다

## 15.2.5 비동기 API에서 예외는 어떻게 처리하는가?
- 비동기 API에서 호출된 메서드의 실제 바디는 별도의 스레드에서 호출되며 이때 발생하는 에러는 이미 호출자의 실행 범위와는 상관없는 상황
- Future를 구현한 CompletableFuture에서는 런타임 get() 메서드에 예외를 처리할 수 있는 기능을 제공하며 예외에서 회복할 수 있도록 exceptionally() 같은 메서드도 제공한다
- 리액티브 형식의 비동기 API에서는 return 대신 기존 콜백이 호출되므로 예외가 발생했을 때 실행될 추가 콜백을 만들어 인터페이스를 바꿔야한다.

이후 내용은 Flow API의 예외 처리 방법입니다.   
이 부분은 그냥 보기에는 이해가 잘 안될 수도 있습니다. 17장의 Flow API를 보는 것을 추천드립니다.
