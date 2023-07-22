

1. 스칼라의 일급 함수
    - Predicate함수를 저장해뒀다가 filter함수에 인수로 전달할 수 있다.
        
        ```css
        object Beer {
        defmain(args: Array[String]) {
          def isJavaMentioned(tweet: String) : Boolean = tweet.contains("Java")
          val tweets = List("Java", "Scala")
          tweets.filter(isJavaMentioned).foreach(println)
        }
        }
        ```
        
2. 익명 함수와 클로저
    - scala.Function1형식의 익명 클래스를 축약한 다음 Function1의 apply호출
        
        ```tsx
        object Beer {
        defmain(args: Array[String]) {
          val isLong : String => Boolean = (tweet : String) => tweet.length > 60
        var result = isLong.apply("short tweet")
          println(result)
        }
        }
        ```
        
    - 클로저
        - 함수의 비지역 변수를 자유롭게 참조할 수 있는 함수의 인스턴스.
        - 자바의 람다에서는 람다가 정의된 지역 변수를 final로 취급해서 고칠 수 없는 제약이 있다.
            
            ```tsx
            object Beer {
            defmain(args: Array[String]) {
            var count = 0
            // 람다 표현식인데 지역변수를 수정했다!
            val inc = () => count += 1
            inc()
            println(count)// 1
            inc()
            println(count)// 2
            }
            }
            ```
            
3. 커링
    - 스칼라에서는 커리된 함수를 직접 제공할 필요가 없다.
    - 함수가 여러 커리된 인수 리스트를 포함하고 있음을 가리키는 함수 정의 문법을 제공하기 때문이다.
        
        ```php
        object Beer {
        def main(args:Array[String]) {
        // 인수 리스트 둘을 받는 함수
          def multiply(x :Int, y :Int) = x * y
          println(multiply(2, 10))// 20// 파라미터 둘로 구성된 인수 리스트 하나를 받는 함수
          def multiplyCurry(x :Int)(y :Int) = x * y
          println(multiplyCurry(2)(10))// 20
        }
        }
        ```
