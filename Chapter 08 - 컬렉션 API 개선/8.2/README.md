## 8.2 리스트와 집합 처리

새로운 결과를 만드는 stream과 달리 아래에서 살펴볼 `removeIf`, `replaceAll` 는 호출한 컬렉션 자체를 바꾼다.

### 8.2.1 removeIf

책 예제 이전에 개인적으로 더 이해하기 쉬운 코드라 먼저보면 좋을 것 같습니다.
아래의 코드는 <u>ConcurrentModificationException</u>을 발생되는 예제 코드 입니다.
(ConcurrentModificationException : for 문을 통해 순서대로 돌고 있는데, 요소가 삭제되어 인덱스가 맞지 않는 경우가 감지될 경우 생기는 오류)

```java
public static void main(String[] args){
	ArrayList<Integer> numbers = new ArrayList<Integer>(
		Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
	for (Integer num: numbers){
		if(num % 2 ==0){
			numbers.remove(num);
		}
	}
}
```
다음 코드를 removeIf로 바꾸면 아래와 같이 됩니다.
 ```java
 public static void main(String[] args){
 	ArrayList<Integer> numbers = new ArrayList<Integer>(
 			Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
 	numbers.removeIf(num -> num %2==0);
 	System.out.println(numbers);
 }
 ```
 책 예제 입니다. 이 예제 또한 ConcurrentModificationException 오류를 발생시킵니다.
```java
for (Transaction transaction : transactions) {
    if (Character.isDigit(transaction.getReferencecode().charAt(0))) {
        transactions.remove(transaction);
    }
}
```

for-each 구문은 내부적으로 Iterator 객체를 사용하기 때문에 Iterator 객체, Collection 객체 총 2개의 객체가 컬렉션을 관리하게 되고, 따라서 **반복자의 상태가 컬렉션의 상태와 서로 동기화되지 않게 됩니다.**

이를 해결해보면 아래와 같이 할 수 있는데 코드가 복잡해졌습니다.

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
    Transaction transaction = iterator.next();
    if (Character.isDigit(transaction.getReferenceCode().charAt(0))) {
        iterator.remove();
    }
}
```
이를 removeIf를 사용하면 다음과 같습니다.

```java
transactions.removeIf(transaction -> Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

### 8.2.2 replaceAll

때로는 요소를 제거하는게 아니라 바꿔야 하는 상황이 생길수도 있습니다. 

이럴 때 사용하는 `replaceAll`입니다.

List에서 이용가능하고 UnaryOperator 함수를 이용해 요소를 바꿉니다.

```java
referenceCode.stream().map(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1)) // [a12, C14, b13]
    .collect(Collectors.toList())
    .forEach(System.out::println);
```

이 요소들을 대문자를 바꿀 때, 기존 요소를 바꾸는 것이 아니라 새 문자열 컬렉션을 생성하게 됩니다.

기존 컬렉션을 바꾸는 것이 아니기 때문에 removeIf 처럼 혼용되어 사용해서 문제가 생길 수도 있습니다.

이때 replaceAll을 다음과 같이 사용하면 해결 됩니다.

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```
