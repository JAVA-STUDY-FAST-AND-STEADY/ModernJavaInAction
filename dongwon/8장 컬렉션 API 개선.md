# 컬렉션 API 개선

## 기존 형식의 자바 코드 리팩터링 하는 기법들

<br>

자바 개발을 도와주는 컬렉션 API. 자바 9에서 컬렉션 API의 단점을 보완하고, 추가된 새로운 컬렉션 API 의 기능들이 있다. 

크게 3가지의 기능으로 나누어 설명한다

1. 작은 리스트, 집합, 맵을 쉽게 만들수 있도록 한 컬렉션 팩토리
2. 리스트와 집합에서 요소를 삭제하거나 바꾸는 관용 패턴을 적용하는 방법
3. 맵 작업과 관련하여 추가된 편리한 새로운 기능

<br>

## 컬렉션 팩토리

작은 컬렉션 객체를 쉽게 만들 수 있는 방법

```java
기존

List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");

Arrays.asList() 팩토리 메서드 사용하기

List<String> friends = Arrrays.asList("Raphael", "Olivia", "Thibaut");
```

고정 크기의 리스트를 만들어서 요소를 갱신할 순 있지만 새 요소를 추가하거나 삭제할 수는 없다

<br>

요소를 추가하려 하면 `UnsupportedOperationException`이 발생한다

```java
List<String> friends = Arrrays.asList("Raphael", "Olivia");
friends.set(0, "Richard");
friends.add("Thibaut");
```

<img src="image\UnsupportedOperationException 예외발생.png" width="80%" height="80%">

<br>

### UnsupportedOperationException 예외 발생

내부적으로 고정된 크기의 변환할 수 있는 배열로 구현되어서 예외가 발생하는 것

집합에서는 `Arrays.asSet()` 의 팩토리 메서드가 없어서 `Hashset` 생성자를 사용할 수 있다

```java
HashSet 생성자 사용

Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", Thibaut"));

스트림 API 사용

Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut")
							.coolect(Collectors.toSet());
```

두 방법 모두 매끄럽지 못하며 내부적으로 불필요한 객체 할당을 필요로 한다. 또한 결과는 변환할 수 있는 집합이다

맵은 작은 맵 만드는 메서드 따로 없지만 자바9에서는 작은 리스트, 집합, 맵 쉽게 만드는 팩토리 메서들르 제공한다

<br>
<hr>
<aside>
💡 컬렉션 리터럴

파이선, 그루비등을 포함한 일부 언어는 컬렉션 리터럴 (ex. [42, 1, 5]) 와 같은 특별한 문법을 이용해 컬렉션을 만드는 기능을 지원한다. 자바에선 너무 큰 언어 변화와 관련된 비용이 들어서 이와 같은 기능을 지원하지는 않는다. 

대신 자바9에서 컬렉션 API를 개선했다!

</aside>
<hr>
<br>

### 리스트 팩토리

List.of 팩토리 메소드 사용해 간단한 리스트 만들기

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends); // [Raphael, Olivia, Thibaut]

여기 friends 리스트에 요소를 추가하면 
friends.add("Chih_chun");
java.lang.UnsupportedOperationException 에러가 발생한다

set() 메서드로 아이템 바꾸려 해도 비슷한 예외가 발생한다 -> set 메서드로도 리스트를 바꿀 수 없다
```

바꿀수 없다는 것이 단점이 될 수도 있지만. 컬렉션이 의도치 않게 변하는 것을 막을 수도 있다. [쓰기 나름이라는 것]

하지만?! 요소 자체가 변하는 것을 막을 수 있는 방법은 없다. 리스트를 바꿔야 한다면 직접 리스트를 만들면 된다

또한 null 요소를 금지하여 의도치 않은 버그를 방지하고, 조금 더 간결한 내부 구현을 달성했다

> 278.p 요소 자체 변하는 경우???

<img src="image\List.of 특징.png" width="70%" height="70%" >

<br>

<hr>

<aside>
💡 오버로딩 vs 가변 인수

List 인터페이스에는 List.of의 다양한 오버로드 버전이 있다
static <E> List<E> of(E e1, E e2, E e3, E e4)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5)
<hr>
<br>

	왜 밑에서와 같이. 다중 요소를 받을 수 있게 자바API를 만들지 않았을까?
	 static <E> List<E> of(E… elements)
	내부적으로 가변 인수 버전은 추가 배열을 할당해서 리스트로 감싼다.

	따라서 배열을 할당하고 초기화하며 나중에 가비지 컬렉션을 하는 비용을 지불해야 한다. → 고정된 숫자의 요소(최대 10개 까지)를 API로 정의하므로 비용을 줄일수 있다. List.of로 열 개 이상의 요소를 가진 리스트를 만들 수도 있지만 이때는 가변 인수를 이용하는 메서드가 사용된다. 
	
	Set.of와 Map.of에서도 같은 패턴이 등장한다

</aside>

<br>
<b>새롭게 컬렉션 팩토리 메서드를 배워봤는데 언제 팩토리 메서드 대신 스트림 API를 사용해 리스트를 만들어야 하나?</b>

<br>

	요소 변환 또는 필터링 할때, 복잡한 데이터 처리할때, 지연 평가, 기존 코드와 통합할때 스트림API를 이용하고

	데이터 처리 형식을 설정하거나, 데이터를 변환할 필요 없으면 사용하기 간편한 팩토리 메서드를 이용하기 권장한다. 팩토리 메서드 구현이 더 단순하고 목적 달성에 충분하다

	### Collectors.toList()를 사용하여 스트림을 목록으로 변환하면 목록 인터페이스에서 제공하는 다양한 메서드에 액세스할 수 있습니다

<br>

### 집합 팩토리

 List.of 와 같이 바꿀 수 없는 집합 만들기

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends); // [Raphael, Olivia, Thibaut]

중복된 요소 제공해 집합 만들려 하면 Olivia 요소가 중복되어 있다는 말과 함께 IllegalArgumentExcption이 발생한다.(집합은 고유의 요소 포함하는 특성으로 그렇다)
```
<br>

### 맵 팩토리

맵을 만들때는 키와 값이 필요해서 조금 더 복잡하다. 
두가지 방법으로 바꿀 수 없는 맵을 초기화 할수 있는데  Map.of 팩토리 메서드의 키와 값을 번갈아 제공하는 방법으로 맵을 만들수 있다

```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
System.out.println(ageOfFriends); // <- {Ollivia=25, Raphael=30, Thibaut=26}

10개 이하의 키와 값 쌍을 가진 작은 맵 만들 때는 이 메소드가 유용하다. 
그 이상의 맵에서는 Map.Entry<K,Y> 객채를 인수로 받으며 가변 인수로 구현된 Map.ofEntries 팩토리 메서드를 이용하는 것이 좋다. 이 메서드는 키와 값을 감쌀 추가 객체 할당을 필요로 한다.

10개 이상의 값을 가질때
Map<String, Integer> ageOfFreinds = Map.ofEntres(entry("Raphael", 30), 
										entry("Olivia", 25),
										entry("Thibaut", 26));
System.out.println(ageOfFriends); // <- {Olivia=25, Raphael=30, Thibaut=26}

Map.entry는 Map.Entry 객체를 만드는 새로운 팩토리 메서드다
```

팩토리 메서드를 이용해서 컬렉션을 좀더 쉽게 만드는 방법을 봤는데. 자바 9가 있기전에는 컬렉션을 만들때 어떤 문제가 있었나???

	문제점
	1. 컬렉션을 초기화 해야하는데 요소가 많을시 코드가 길어져 가독성을 해쳤다.
	2. 안전성과 스레드의 안정성을 보장하는 수정 불가능 컬렉션 만들기가 어려웠다

<br>

### 리스트와 집합 처리

자바8 에서는  List, Set 인터페이스에 다음과 같은 메서드를 추가했다

- removeIf : 프리티케이트를 만족하는 요소를 제거한다.  List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있다
- replaceAll : 리스트에서 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꾼다
- sort : List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다

이 메서드들은 호출한 컬렉션 자체를 바꾼다. 새로운 결과를 만드는 스트림 동작과 달리 이들 메서드는 기존 컬렉션을 바꾼다. 컬렉션을 바꾸는 동작은 에러를 유발하며 복잡함을 더해서, 자바8에 removeIf와 replaceAll을 추가한 이유다

<br>

### removeIf 메서드

숫자로 시작되는 참조 코드 가진 트랜잭션 삭제 코드

```java
for (Transaction transaction : transactions) {
		if(Character.isdDigit(transaction.getReferenceCode().charAt(0))) {
				transactions.remove(transaction);
		}
}
```
<br>

위의 코드는 `ConcurrentMOdificationException` 을 일으키는데 내부적으로 for-each 루프는 Iterator 객체를 사용해서 위의 코드를 풀어서 보면


```java
for (Iterator<Transaction> iterator = transactions.iterator();
		 iterator.hasNext(); ) {
		Transaction transaction = iterator.next();
		if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
				transactions.remove(transaction);
		// 반복하면서 별도의 두 객체를 통해 컬렉션을 바꾸고 있는 문제가 있다
		}
}
```

두 개의 개별 객체가 컬렉션을 관리하게 된다
- Iterator 객체, next(), hasNext() 를 이용해서 소스를 질의한다
- Collection 객체 자체, remove() 를 호출해 요소를 삭제한다

**결론적으로 반복자의 상태는 컬렉션의 상태와 서로 동기화되지 않는다. Iterator 객체를 명시적으로 사용하고 그 객체의 remove() 메서드를 호출함으로 문제를 해결할 수 있다**

<br>

```java
for (Iterator<Transaction> iterator = transactions.iterator();
		 iterator.hasNext(); ) {
		Transaction transaction = iterator.next();
		if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
				transactions.remove();
		// 반복하면서 별도의 두 객체를 통해 컬렉션을 바꾸고 있는 문제가 있다
		}
}
```

좀 복잡한 코드를 자바8의  removeIf 메서드로 바꿀 수 있다. removeIf 메서드는 삭제할 요소를 가리키는 프레디케이트를 인수로 받는다

```java
transactions.removeIf(transaction -> 
		Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

요소를 제거하는것이 아닌 바꿔야 하는 상황에서는 repalceAll을 사용하면 된다

<br>

### repalceAll 메서드

리스트의 각 요소를 새로운 요소로 바꿀 때

```java
referencecodes.stram() <- [a12, c14, b13]
							.map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
							.collect(Collectors.toList())
							.forEach(System.out::println); <- outputs A12, C14, B13
```

스트림API 는 요소를 바꾸긴 하지만 새 문자열 컬렉션을 만든다. 원하는 바는 기존 컬렉션을 바꾸는 것이니 

ListIterator 객체(요소를 바꾸는  set() 메서드 지원)를 이용할 수 있다

```java
for (ListIterator<String> iterator = referenceCodes.listIterator();
		iterator.hasNext(); ) {
		String code = iterator.next();
		iterator.set(Character.toUpperCase(code.charAt(0)) + code.substring(1));
}
```

코드가 좀 복잡해 졌으며, 컬렉션 객체를 Iterator 객체와 혼용하면 반복과 컬렉션 변경이 동시에 이루어지며 쉽게 문제가 일어난다. 자바 8의 기능 repalceAll 이용하면 좀더 보기 좋다

```java
referenceCodes.repalceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```
<br>

### 맵 처리

자바8 에서 Map 인터페이스에 몇 가지 디폴트 메서드가 추가되었다. 간단히 기본적인 구현을 인터페이스에 제공하는 기능 정도로 생각하자

<br>

### forEach 메서드

맵에서 키와 값을 반복하며 확인하는 작업. 실제  Map.Entry<K, V> 반복자 이용해 맵의 항목 집합을 반복할 수 있다

```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
		String friend = entry.getKey();
		Integer age = entry.getValue();
		System.out.println(friend + " is " + age + " years old");
}
```

자바8 부터는 Map 인터페이스는 BigConsumer(키와 값을 인수로 받는)를 인수로 받는  forEach 메서드를 지원해서 코드를 좀더 간단히 구현할 수 있다

```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

자바 8에서 맵의 항목을 쉽게 비교할 수 있는 방법들

<br>

**정렬 메서드**

맵의 항목을 값 또는 키 기준으로 정렬

- Entry.comparingByValue
- Entry.comparingByKey

```java
Map<String, String> favoriteMovies = 
Map.ofEntries(entry("Raphael", "Star Wars"), 
							entry("Cristina", "Matrix"), 
							entry("Olivia", "James Bond"));

favouriteMovies
		.entrySet()
		.stream()
		.sorted(Entry.comparingByKey())
		.forEachOrdered(System.out::println);
		// 사람의 이름을 알파벳 순으로 스트림 요소를 처리한다

--------------------------------------------------------------------------------

결과
Cristina = Matrix
Olivia = James Bond
Raphael = Star Wars
```

<aside>
<hr>
💡 HashMap  성능

자바 8에서 HashMap의 내부 구조를 바꿔 성능을 개선했다. 

기존에 맵의 항목은 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장했었다. If 많은 키가 같은 해시코드를 반환하는 상황이 된다면 O(n)의 시간이 걸려서 LinkedList로 버킷을 반환해야 해서 성능이 저하 되었다. 최근 버킷이 너무 커지면 이를 O(log(n))의 시간이 소요되는 정렬 트리를 이용해 동적으로 치환해 충돌이 일어나는 요소 반환 성능을 개선했다.

but 키가 String, Number 클래스와 같은 Comparable의 형태여야지 정렬된 트리가 지원된다

요청한 키가 맵에 존재하지 않는다면 에러가 발생한다. 이때 getOrDefault 메서드 이용하면 쉽게 해결 가능하다

</aside>
<hr>

<br>

### getOrDefault 메서드

기존에는 찾으려는 키가 없을시 null 이 반환되어. NullpointerException을 방지하기 위해서 요청결과가 null인지 확인해야 했다. 이를 기본값 반환하는 방식으로 해결할수 있었는데

getOrDefault 메서드를 이용하면 쉽게 문제를 해결할 수 있었다. 이 메서드는 첫 번째 인수로 키를, 두 번째 인수로 기본값을 받으며 맵에 키가 존재하지 않으면 두 번째 인수로 받은 기본값을 반환한다.

```java
Map<String, String> favouriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Oliivia", "James Bond"));

System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix")); 
// 키가 있을 때 : James Bonde 출력
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix")); 
// 키가 없을 때 : Matrix 출력
```

키가 존재하더라도 값이 널인 상황에서는 getOrDefault가 null을 반환할 수 있다. 

<br>

### 계산 패턴

키의 존재 여부에 따라 동작을 실행하고 결과를 저장하는 상황에 도움을 주는 연산

- computeIfAbsent : 제공된 키에 해당하는 값이 없으면(값이 없거나 null), 키를 이용해 새 값을 게산하고 맵에 추가한다
- computeIfPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다
- compute : 제공된 키로 새 값을 계산하고 맵에 저장한다

정보를 캐시할 때 computeIfAbsent활용가능, 기존 데이터를 처리했다면 다시 계산할 필요 없다

<br>

맵 이용해 캐시 구현

```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");

lines.forEach(line -> 
		dataToHash.computeIfAbsent(line, this::calculateDigest));
		// 키가 존재하지 않으면 동작을 실행한다
		// line은 맵에서 찾을 키다
		
private byte[] calculateDigest(String key) {
		return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}
```
<br>

여러 값 저장하는 맵 처리할 때도 유용. Map<K, List<V>>에서 요소를 추가하려면 항목이 초기화 되어 있는지 확인해야 한다

```java
String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);
if(movies == null) { // 리스트가 초기화 되어 있는지 확인
		movies = new ArrayLisst<>();
		friendsToMovies.put(friend, movies);
}

movies.add("Star Wars"); // 영화 추가

System.out.println("Star Wars"); // <- {Raphael:[Star Wars]}

computeIfAbsent 활용

friendstoMovies.computeIfAbsent("rphael", name -> new ArrayList<>())
							 .add("Star Wars"); <- {Raphael:[Star Wars]}
```

`computeIfPresent` 메서드는 현재 키와 관련된 값이 맵에 존재하며 null이 아닐 때만 새 값을 계산. 이 메서드의 실행과정은. 값을 만드는 함수가 null을 반환하면 현제 매핑을 맵에서 제거. but 매핑을 제거할 때 remove 메서드를 오버라이드 하는게 더 적합하다

<br>

### 삭제 패턴

remove 메서드를 통해 삭제하는데. 자바 8에서는 키가 특정한 값과 연관되었을 때만 항목을 제거하는 오버로드 버전 메서드를 제공한다.

```java
기존코드

String key = "Raphael";
String value = "Jack Reacher 2";
if (favouriteMovies.containsKey(key) && Objects.equals(favouriteMovies.get(key), value)) {
		favouriteMovies.remove(key);
		return true;
}
else {
		return false;
}

간결하게 구현
favouriteMovies.remove(key, value);
```
<br>

### 교체 패턴

맵 항목 바꾸는 데 사용할 수 있는 메서드 2개 추가됨

- replaceall : BiFunction을 적용한 결과 각 항목의 값을 교체한다. 이 메서드는 이전에 살펴본 List의 repalce와 비슷한 동작을 한다
- Replace : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다

```java
favorutieMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

replace는 한개의 맵에만 적용할 수 있다.  두 개이상의 맵에서 값을 합치거나 바꾸려 한다면 merge 메서드를 사용하면 된다

<br>

### 합침

putAll을 사용해서 합치면 된다

```java
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends); // friends의 모든 항목을 everyone으로 복사
```

중복된 키가 없다면 키, 값 으로 잘 합쳐서 동작할 것이다. 값을 좀더 유연하게 합치려 한다면 merge메서드를 이용할 수 있다. 

merge는 중복된 키를 어떻게 합칠지 결정하는 BiFunction을 인수로 받는다. 같은 키에 다른 값이 존재한다면 forEach와 merge메서드를 이용해 해결할 수 있다. 

```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) ->
		everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
		// 중복된 키가 있으면 두 값을 연결한다
System.out.println(everyone);
// Outputs {Raphael=Star Wars, Cristina=James Bond & Matrix, Teo=Star Wars}
```

merge 메서드는 null과 관련한 복잡한 상황도 처리할 수 있다

지정된 키와 연관된 값이 없거나 값이 null이면[merge]는 키를 null이 아닌 값과 연결한다. 아니면 [merge]는 연결된 값을 주어진 매핑 함수의 [결과] 값으로 대치하거나 결과가 널이면 [항목]을 제거한다

<br>


```java
merge를 이용한 초기화 검사 구현
영화 시청 횟수 기록맵, 영화에 대한 정보 존재하는지 확인

Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if(count == null) {
		moviesToCount.put(movieName, 1);
} else {
		moviesToCount.put(moviename, count + 1);
}

merge 사용하기

moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);

여기서 merge의 두번째 인수는 1L 이다. 이 인수는 “키와 연관된 기존 값에 합쳐질 null이 아닌, 값 또는 값이 없거나 키에 null 값이 연관되어 있다면 이 값을 키와 연결” 하는데 사용한다.
처음 코드가 실행될때 1이 사용되고 이후로 1로 초기화 되고 BiFunction을 적용해 값이 증가된다
```

<br>

### 개선된 ConcurrentHashMap

`ConcurrentHashMap` 클래스는 동시성 친화적이며 최신 기술 반영한  HashMap 버전이다 `ConcurrentHashMap` 은 내부 자료구조의 특정 부분만 잠궈서 동시 추가하며, 갱신 작업을 허용한다. 따라서 동기화된  Hashtable 버전에 비해 읽기 쓰기 연산 능력이 월등하다(표준. HashMap은 비동기로 동작한다) 

<br>

### 리듀스와 검색

`ConcurrentHashMap`은 스트림과 비슷한 종류의 세 가지 연산, 네가지 연산 형태를 지원한다

연산

- forEach : 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce :  모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합친다
- search : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용한다

연산형태

- 키, 값으로 연산(forEach, reduce, search)
- 키로 연산(forEachkey, reduceKeys, searchKeys)
- 값으로 연산(forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

이들의 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다. 그래서 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.

여기서 연산에 병렬성 기준값을 지정해야 한다. 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다. 기준값을 1로 지정할시 공통 스레드 풀을 이용해 병렬성을 극대화 한다.  Long.MAX_VALUE를 기준값으로 설정할시 한 개의 스레드로 연산을 실행한다. 

<br>

```java
reduceValues 메서드 이용한 맵의 최댓값 찾기

ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
																	// 여러 키와 값을 포함하도록 갱신될 ConcurrentHashMap
long parallelismThreshold = 1;
Optional<Integer> maxValue = 
				Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```

int, long, double 등의 기본값에는 전용 each reduce연산이 제공되어서 reduceValuesToInt, reduceKeysToLong 등을 이용하면 박싱 작업을 피해서 효율적으로 작업할 수 있다

<br>

### 계수

ConcurrentHashMap클래스는 맵의 매핑 개수 반환하는 mappingCount 메서드를 제공한다. 기존의 size메서드 대신 새 코드에서는 int를 반환하는 mappingCount메서드를 사용하는게 좋다. 그래야만 매핑의 개수가 int의 범위를 넘어서는 상황을 대처 할 수 있다(long형 반환함)

<br>

### 집합뷰

ConcurrentHashMap클래스는 ConcurrentHashMap을 집합 뷰로 반환하는 KeySet 메서드 제공.

맵 바꾸면 집합도 바뀌고 집합을 바꾸면 맵도 영향을 받는다. newKeySet 의 새 메서드 이용해 ConcurrentHashMap으로 유지되는 집합 만들수 있다

---

<br>

### 정리

- 자바 9는 원소 포함해 바꿀 수 없는 리스트, 집합, 맵을 쉽게 만드는 List.of, Set.of, Map.ofEntries등의 컬렉션 팩토리를 지원한다
- 컬렉션 팩토리가 반환한 객체는 만들고 나서 바꿀 수 없다
- List 인터페이스는 removeIf, replaceAll, sort 세 가지 디폴트 메서드를 지원한다
- Set 인터페이스는 removeIf 디폴트 메서드를 지원한다
- Map 인터페이스는 자주 쓰는 패턴과 버그를 방지 하도록 다양한 디폴트 메서드를 지원한다
- ConcurrentHashMap은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공