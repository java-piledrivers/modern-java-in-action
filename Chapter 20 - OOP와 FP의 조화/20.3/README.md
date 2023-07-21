# 20.3  클래스와 트레이트

---

자바에 클래스와 인터페이스가 있다면 스칼라에는 클래스와 트레이트가 있다. 스칼라의 클래스와 트레이트는 자바에 비해 더 유연하다.  


## 20.3.1 간결성을 제공하는 스칼라의 클래스

스칼라는 객체지향 언어이기에 클래스를 만들고 객체로 인스턴스화 할 수 있다. 스칼라의 Hello 클래스 정의 코드를 살펴보자.  

```scala
class Hello {
    def sayThankYou() {
        println("Thanks for reading our book")
    }
}
val h = new Hello()
h.sayThankYou()
```

자바처럼 클래스를 만들고 인스턴스화 하는 방법과 문맥적 비슷한 구조를 확인할 수 있다.  

<br>

### 게터와 세터

자바에서 필드 리스트만 정의하는 자바 클래스를 구현할 때에 필드의 수만큼 게터와 세터를 구현해야 한다. 그리고 이러한 클래스가 한두개가 아니라면 간단한 클래스인데도 코드의 양이 상당하게 된다.  
하지만 스칼라에서는 생성자, 게터, 세터가 암시적으로 생성되므로 코드가 훨씬 단순해진다.  


```scala
class Student(var name: String, var id: Int)
val s = new Student("Choi", 1)  // Student 객체 초기화
println(s.name)                 // Choi 출력
s.id = 1337                     // id 설정
println(s.id)                   // 1337 출력
```


<br>

## 20.3.2 스칼라 트레이트와 자바 인터페이스

스칼라의 트레이트는 자바의 인터페이스처럼 추상 기능을 제공한다. 트레이트로 추상 메서드와 기본 구현을 가진 메서드 두 가지 모두 정의할 수 있다.  
자바의 인터페이스처럼 트레이트는 다중 상속을 지원하기에 자바의 인터페이스와 디폴트 메서드 기능이 합쳐진 것으로 이해할 수 있다. 트레이트와 추상 클래스의 차이는 트레이트는 클래스와는 달리 다중 상속이 가능하다.  

Sized라는 트레이트를 정의하면서 스칼라의 트레이트가 무엇인지 확인해보자. Sized 트레이트는 size 라는 가변 필드와 기본 구현을 제공하는 isEmpty 메서드를 포함한다.  

```scala
trait Sized {
    var size : Int = 0			// size 필드
    def isEmpty() = size == 0	// 기본 구현을 제공하는 isEmpty 메서드
}
```

아래는 Sized 트레이트를 상속받은 클래스의 정의 예제이다.  

```scala
class Empty extends Sized			// 트레이트 Sized에서 상속받은 클래스

println(new Empty().isEmpty())		// true
```

자바의 인터페이스와는 달리 객체 트레이트는 **인스턴스화 과정에서도 조합**할 수 있다. 조합 결과는 런타임시 결정되는 건 아니고, 컴파일 시 결정된다.  

에를 들어 Box 클래스를 만든 다음에 어떤 Box 인스턴스는 트레이트 Sized 가 정의하는 동작을 지원하도록 결정할 수 있다.  

```scala
class Box
val b1 = new Box() with Sized 		// 객체를 인스턴스화할 때 트레이트를 조합함
println(b1.isEmpty()) 				// true
val b2 = new Box()
b2.isEmpty()						// 컴파일 에러 발생: Box 클래스 선언이
									// Sized 를 상속하지 않았기 때문
```

