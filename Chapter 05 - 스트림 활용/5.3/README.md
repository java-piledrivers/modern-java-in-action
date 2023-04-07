# 매핑

자료형을 변환시키거나 데이타를 처리할 때에도 스트림의 매핑을 활용할 수 있다.  

---

## 스트림의 각 요소에 함수 적용하기

스트림 인터페이스의 map 메소드가 바로 그 기능을 해주는데 뜯어보면 이렇게 생겼다.

<br>

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);

```
<br>


함수를 인수로 받고, 각 요소에 적용시켜 새로운 요소로 변환한다.  


아래의 예제는 Dish::getName을 map 메서드로 전달해서 스트림의 요리명을 추출하는 코드다.

<br>


```java
List<String> dishNames = menu.stream()
                              .map(Dish::getName)
                              .collect(toList());

```
<br>


map이 Dish 객체에서 getName 이라는 메소드를 사용하여 String으로 변환시켜주는 역활을 하였다.  

단어가 주어졌을때 그 단어에 대한 길이를 알고싶을때에도 스트림의 map을 사용하면 된다.

```java
List<Integer> dishNames = menu.stream()
                              .map(Dish::getName)
                              .map(String::length)
                              .collect(toList());

```

<br>

이전 코드에서 map(String::length)가 추가 되었는데, Dish -> String -> Integer 순으로 map이 getName 메소드와 length 메소드 를 
적용하여 새로운 요소로 매핑한 것이다.  

<br>


## 5.3.2 스트림 평면화


단어들 사이에 사용된 고유 문자를 반환하고자 할 때에도, 스트림을 활용할 수 있다.  


<br>

```java

words = ["Hello", "World"];

// 얻고자 하는 결과값
// ["H", "e", "l", "o", "W", "r", "d"]

```



### 첫번째 시도

```java
words.stream()
      .map(word -> word.split(""))
      .distinct()
      .collect(toList());
```

<br>

위의 코드는 String을 String[] 으로 반환하기 때문에 그 뒤에 오는 distinct()가 별 역활을 해주지 못한다.  


### 두번째 시도

배열 스트림 대신에 문자열 스트림을 활용해보자. 문자열을 받아 스트림으로 제공하는 메소드가 있다.  


```java

String[] arrayOfWords = {"Goodbye", "World"};
Stream<String> streamOfwords = Arrays.stream(arrayOfWords);



words.stream()
      .map(word -> word.split(""))
      .map(Arrays::stream)
      .distinct()
      .collect(toList());
```

하지만 위의 List<Stream<String>> 와 같은 형태가 되므로 문제는 해결되지 않았다.  
  문제를 해결하려면 각 단어를 개별 문자열로 이루어진 배열로 만든 다음에 각 배열을 별도의 스트림으로 반환하여야 한다.  
  

  
  ### 세번째 시도

    flatMap을 활용하면 문제 해결이 가능하다.  
   
  
```java
List<String> uniqueCharacters = words.stream()
                                     .map(word -> word.split(""))   // Stream<String[]>
                                     .flafMap(Arrays::stream)       // Stream<String>
                                     .distinct()                    // Stream<String>
                                     .collect(toList());            // List<String>

```
    
    <br>
    
flatMap 메소드가 스트링 배열에서 개별 스트링으로 반환해주는 역활을 하였다.  
