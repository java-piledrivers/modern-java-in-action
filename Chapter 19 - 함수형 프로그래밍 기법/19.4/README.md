# 19.4 패턴 매칭
- 함수형 프로그래밍을 구분하는 중요한 특징
- 정규표현식 그리고 정규표현식과 관련된 패턴 매칭과는 다르다

## 19.4.1 방문자 디자인 패턴
- 방문자 디자인 패턴을 사용하면 자료형을 언랩할 수 있으며, 특정 데이터 형식을 '방문'하는 알고리즘을 캡슐화할 수 있다. 
- 방문자 클래스는 지정된 데이터 형식의 인스턴스를 입력으로 받고, 인스턴스의 모든 멤버에 접근한다.



```java
class BinOp extends Expr {
  ...
  public Expr accept(SimpiftExprVisitor v) {
    return v.visit(this);
  }
}

// 이제 SimplifyExprVisitor는 BinOp 객체를 언랩할 수 있다.
public class SimplifyExprVisitor {
   ...
   public Expr visit(BinOp e) {
     if("+".equal(e.opname) && e.right instanceof Number && ...) {
       return e.left;
     }
     return e;
   }
 }
```

## 19.4.2 패턴 매칭의 힘
- 자바는 패턴 매칭을 지원하지 않지만 스칼라 언어에서는 패턴 매칭을 사용해 간결하고 명확한 코드를 구현할 수 있다.

```scala
def simplifyExpression(expr: Expr): Expr = expr match {
  case BinOp("+", e, Number(0)) => e  //0 더하기
  case BinOp("*", e, Number(1)) => e  //1 곱하기
  case BinOp("/", e, Number(1)) => e  //1 나누기
  case _ => expr //expr을 단순화할 수 없다.
}
```
- 패턴 매칭을 지원하는 언어의 가장 큰 실용적인 장점은 커다란 switch 문이나 if-then-else 문을 피할 수 있다는 것이다.

자바로 패턴 매칭 흉내내기
```java
def simplifyExpression(expr: Expr): Expr = expr match {
  case BinOp("+", e, Number(0)) => e  //0 더하기
  ...
```

- 위 스칼라 코드는 expr이 BinOp인지 확인하고 expr에서 세 컴포넌트(opname, left, right)를 추출한 다음에 각각의 컴포넌트에 패턴 매칭을 시도한다. 즉 스칼라의 패턴매칭은 다수준이다. 
- 자바 8의 람다를 이용한 패턴 매칭 흉내 내기는 단일 수준의 패턴 매칭만 지원한다. 
- 먼저 규칙을 정하자. 람다를 이용하며 코드에 if-then-else가 없어야 한다.

```java
myIf(conditition, () -> e1, () -> e2);

static <T> T myIf(boolean b, Supplier<T> truecase, Supplier<T> falsecase) {
  return b ? truecase.get() : falsecase:get();
}
```

BinOp와 Number 두 서브클래스를 포함하는 Expr 클래스의 패턴 매칭 값으로 돌아와서 patternMatchExpr이라는 메서드를 정의할 수 있다.

```java
interface TriFunction<S, T, U, R> {
  R apply(S s, T t, U u);
}

static <T> T patternMatchExpr(
    Expr e,
    TriFunction<String, Expr, Expr, T> binopcase,
    Function<Integer, T> numcase,
    Supplier<T> defaultCase) {
  return
    (e instanceof Binop) ? 
      binopcase.apply(((BinOp)e).opname, ((BinOp)e).left,((Binop)e).right) : 
    (e instanceof Number) ?
      numcase.apply(((NUmber)e).val) : defaultcase.get();
}

```

다음 코드를 살펴보자
```java
patternMatchExpr(e, (op, 1, r) -> {return binopcode;},
  (n) -> {return numcode;},
  () -> {return defaultcode;});
```
- e가 BinOp인지(식별자 Op, l, r로 BinOp에 접근할 수 있는 binopcode를 실행)
- 아니면 Number인지(n 값에 접근할 수 있는 numCode를 실행) 확인한다. 
- 둘다 아닐 경우 실행되는 defaultCode도 존재한다.


다음으로 patternMatchExpr을 이용해서 덧셈과 곱셈 표현식을 단순화하는 방법이다.
```java
public static Expr simplify(Expr e) {
  TriFunction<String, Expr, Expr, Expr> binopcase =
    (opname, left, right) -> {
      if ("+".equals(opname)) {
        if (left instanceof Number && ((Number) left).val == 0) {
          return right;
        }
        if (right instanceof Number && ((Number) right).val == 0) {
          return left;
        }
      }
      if ("*".equals(opname)) {
        if (left instanceof Number && ((Number) left).val == 1) {
          return right;
        }
        if (right instanceof Number && ((Number) right).val == 1) {
          return left;
        }
      }
      return new BinOp(opname, left, right);
    };
  Function<Integer, Expr> numcase = val -> new Number(val); //숫자 처리
  Supplier<Expr> defaultcase = () -> new Number(0); // 수식을 인식할 수 없을 때 기본 처리
  
  return patternMatchExpr(e, binopcase, numcase, defaultcase); // 패턴 매칭 적용
}

// 아래처럼 simplify 메서드를 호출할 수 있다.
Expr e = new BinOp("+", new Number(5), new Number(0));
Expr match = simplify(e);
```
