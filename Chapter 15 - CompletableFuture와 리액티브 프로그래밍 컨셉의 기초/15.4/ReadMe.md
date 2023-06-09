## 15.4 CompletableFuture와 콤비네티어를 이용한 동시성

자바 8에서는 CompletableFuture는 Future를 조합할 수 있는 기능이 있다.

ComposableFuture가 아닌 CompletableFuture라 부르는 이유는 실행할 코드 없이 Future를 만들 수 있고, complete() 메서드를 이용해 다른 스레드가 완료한 후에 get()으로 값을 얻을 수 있도록 허용하기 때문이다.

```
public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFiexedthreadPool(10);
    int x = 1337;
    
    CompletableFuture<Integer> a = new CompletableFuture<>();
    executorService.submit(()-> a.complete(f(x)));
    int b = g(x);
    System.out.println(a.get()+b);
    
    executorService.shutdown();
  }
}
```

f(x)와 g(x)를 동시에 실행해 합계를 구하는 위 코드에서, f(x)의 실행이 끝나지 않은 상황에서 get()을 기다리며 프로세싱 자원을 낭비할 수 있다.



ComposableFuture<T>의 thenCombine 메서드를 사용하면 연산 결과를 효과적으로 더할 수 있다.

```
ComposableFuture<V> thenCombine(CompletableFuture<U> other, Bifunction<T, U, V> fn)
```

이 메서드는 T, U 값을 받아 새로운 V값을 만든다.

```
public class CFComplete {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    ExecutorService executorService = Executors.newFiexedthreadPool(10);
    int x = 1337;
    
    CompletableFuture<Integer> a = new CompletableFuture<>();
    CompletableFuture<Integer> b = new CompletableFuture<>();
    
    CompletableFuture<Integer> c = a.thenCombine(b, (y, z) -> y + z);
    executorService.submit(()-> a.complete(f(x)));
    executorService.submit(()-> b.complete(g(x)));

    System.out.println(c.get());
    executorService.shutdown();
  }
}
```

결과를 추가하는 세 번째 연산 c는 다른 두 작업이 끝날때까지 실행되지 않으므로 먼저 시작해서 블록되지 않는다.

이전 버전의 y+z 연산은 g(x)를 실행한 스레드에서 수행되어 f(x)가 완료될 때까지 블록될 여지가 있었다.

반면 thenCombine을 이용하면 f(x)와 g(x)가 끝난 다음에 덧셈 계산(c)이 실행된다.
