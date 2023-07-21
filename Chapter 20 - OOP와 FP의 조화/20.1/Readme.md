

- 객체지향과 함수형 프로그래밍을 혼합한 언어
- 스칼라는 모든 자바 라이브러리를 사용할 수 있다.
- 이 챕터를 통해 스칼라와 자바에 적용된 함수형 기능을 살펴보면서 자바의 한계가 무엇인지 확인할 수 있다.



### 20.1 스칼라 소개

- 명령형, 함수형으로 구현된 Hello World예제

- 스칼라의 자료구조 확인(List, Set, Map, Stream, Tuple, Option...)

1. 명령형 스칼라

   - 클래스 정의와 동시에 인스턴스화

   ```scala
     object Beer {
     // object 내부에 선언된 메서드는 정적 메서드로 간주되므로 static을 명시할 필요 없다.
     def main(args: Array[String]) {
       var n : Int = 2
       while (n <= 6) {
         // 스칼라의 문자열 보간법 : 문자열에 접두어s를 붙이고, ${}안에 변수 위치
         println(s"hello ${n}")
         n += 1
       }
     }
     }
   ```
2. 함수형 스칼라

     - while루프 처럼 변수를 갱신하지 않는 함수형

     ```scala
     object Beer {
     def main(args: Array[String]) {
     2 to 6 foreach { n => println(s"hello ${n}") } // 숫자 범위 표현
     // 2.to(6).foreach(n => println(s"hello ${n}"))  //위와 동일
     }
     }
     ```

3. 기본 자료구조 : 리스트, 집합, 맵, 튜플, 스트림, 옵션

   - 컬렉션 만들기

   - 간결성을 강조하는 특성 - 한 줄의 코드

     ```scala
     object Beer {
     def main(args: Array[String]) {
       // Map 선언 result : Some(23)
       var authorsToAge = Map("R" -> 23, "M" -> 40,  "A" -> 54)
       println(s"${authorsToAge.get("R")}")
       // List 선언 result : R M A 
       val authors = List("R", "M", "A")
       authors.foreach(name => print(s"${name} "))
       // Set 선언 result : 5 1 6 2 3 
       val numbers = Set(1, 1, 2, 3, 5, 6)
       numbers.foreach(number => print(s"${number} "))
     }
     }
     ```

   - 불변과 가변

     - 위 예제에서 만든 자료구조들은 불변객체

     - 갱신이 필요하면 새로운 컬렉션을 만들어야 함

       ```scala
       object Beer {
       def main(args: Array[String]) {
       val authors = Set("R", "M", "A")
       val newAuthors = authors + "B"
       println(newAuthors) // Set(R, M, A, B) -> 새로운 집합이 생성됨
       }
       }
       ```

   - 컬렉션 사용하기

   - stream API 와 비슷하다

     ```scala
     object Beer {
     def main(args: Array[String]) {
       val numbers = List("three", "four", "seven")
       val result = numbers.filter(n => n.length() > 4).map(n => n.toUpperCase())
       println(result)
       // List(THREE, SEVEN)
       val resultByUnderscore = numbers filter (_.length > 4) map (_.toUpperCase)
       println(resultByUnderscore)
       // List(THREE, SEVEN)
     }
     }
     ```

   - 튜플

     - 자바는 튜플을 지원하지 않는다. 필요하면 직접 만들어쓴다.

     - 스칼라는 튜플식으로 쓸 수 있는 record 지원

       ```scala
       object Beer {
       def main(args: Array[String]) {
       val book = (2018, "Modern Java", "Manning")
       val numbers = (2, 3, 4, 5)
       println(book) // (2018,Modern Java,Manning)
       println(numbers) // (2,3,4,5)
       }
       }
       
       println(book._1) // 2018 출력
       ```

   - 스트림

     - 스칼라의 스트림은 자바의 스트림보다 다양한 기능을 제공한다.
     - 이전 요소가 접근할 수 있도록 기존 계산 값을 기억한다.
     - 인덱스를 제공하기 때문에 리스트처럼 인덱스로 스트림 요소에 접근할 수 있다.
     - 자바의 스트림에 비해 메모리 효율성이 떨어진다. -> 이전 요소를 참조하려면 캐시가 필요하기 때문

   - 옵션

     - Optional과 비슷하다.

     - 스칼라에도 null이 존재하지만 사용하지 않는 것이 좋다.
     
       ```scala
       def getCarInsuranceName(person: Option[Person], minAge: Int) =
       person.filter(_.getAge >= minAge)
         .flatMap(_.getCar)
         .flatMap(_.getInsurance)
         .map(_.getName)
         .getOrElse("Unknown")
       ```
