# Chap 01. 자바 8, 9, 10, 11 : 무슨 일이 일어나고 있는가?
## 1.1 역사의 흐름은 무엇인가?

* 자바 역사를 통틀어 **자바 8**에서 가장 큰 변화
  * 자바 1.0 : 스레드와 락, 메모리 모델 지원. 저수준 기능 활용 어렵
  * 자바 5 : 스레드 풀, 병렬 실행 컬렉션
  * 자바 7 : 병렬 실행에 도움을 줄 수 있는 포크 / 조인 프레임워크 제공.  but, 활용 어렵
  * 자바 8 : 병렬 실행을 새롭고 단순한 방식으로 접근할 수 있는 방법 제공
  * 자바 9 : RxJava으로 리액티브 프로그래밍이라는 병렬 실행 기법 지원 
* 자바 8은 **간결한 코드, 멀티 코어 프로세서의 쉬운 활용**이라는 두 가지 요구사항을 기반으로 함
* 자바 8의 새로운 기술
  * 스트림 API
  * 메서드에 코드를 전달하는 기법 (동작 파라미터화)
  * 인터페이스의 디폴트 메서드
* 자바 8 기법은 **함수형 프로그래밍**에서 위력을 발휘

## 1.2 왜 아직도 자바는 변화하는가?
### 1.2.1 프로그래밍 언어 생태계에서 자바의 위치
* 자바는 처음부터 많은 유용한 라이브러리를 포함하는 잘 설계된 객체지향 언어로 시작
* 코드를 JVM 바이트 코드로 컴파일하는 특징 및 가상 머신 코드를 지원하기 때문에 인터넷 애플릿 프로그램의 주요 언어가 됨
* 자바는 다양한 임베디드 컴퓨팅 분야를 성공적으로 장악하고 있음
* 자바 8은 더 다양한 프로그래밍 도구 그리고 다양한 프로그래밍 문제를 더욱 빠르고 정확하며 쉽게 유지보수할 수 있다는 장점 제공

### 1.2.2 스트림 처리
* 스트림이란 한번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임
* 자바 8에는 java.util.stream 패키지에 스트림 API가 추가됨
  * 스트림 API는 파이프라인을 만드는 데 필요한 많은 메서드를 제공
* 우리가 하려는 작업을 고수준으로 추상화해서 일련의 스트림으로 만들어 처리 가능
* 스레드라는 복잡한 작업을 사용하지 않으면서 공짜로 병렬성을 얻을 수 있음

### 1.2.3 동작 파라미터화로 메서드에 코드 전달하기
* 코드 일부를 API로 전달하는 기능
* 자바 8에서는 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공하며, 이것을 **동작 파라미터화** 라고 함

### 1.2.4 병렬성과 공유 가변 데이터
* 공유되지 않은 가변 데이터, 메서드, 함수 코드를 다른 메서드로 전달하는 두가지 기능은 함수형 패러다임의 핵심적인 사항
  * (명령형 프로그래밍 패러다임에서는 일련의 가변 상태로 프로그램을 정의)

### 1.2.5 자바가 진화해야 하는 이유
* 지금까지 자바는 진화해 옴
* 자바 8의 가장 큰 변화는 고전적인 객체지향에서 벗어나 함수형 프로그래밍에 다가선 것임
  * 함수형 프로그래밍에서는 우리가 하려는 작업이 최우선시되며 그 작업을 어떻게 수행하는지는 별개의 문제로 취급
* 언어는 하드웨어나 프로그램머의 기대의 변화에 부응하는 방향으로 변화해야함 

## 1.3 자바 함수
* 프로그래밍 언어에서 함수(function)라는 용어는 메서드(method) 특히 정적 메서드(static method)와 같은 의미로 사용
* 자바 8에서는 함수를 새로운 값의 형식으로 추가했는데 이는 멀티코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있도록 함수를 만들었기 때문임 
* 프로그래밍 언어의 핵심은 값을 바꾸는 것이고 이 값을 일급 값 또는 시민이라고 함
  * (일급 값이란 그 자체로 자유롭게 전달할 수 있는 형태의 값을 의미)

### 1.3.1 메서드와 람다를 일급 시민으로 
* 자바 8 이전
```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
	public boolean accept(File file) {
		return file.isHidden(); // 숨겨진 파일 필터링
    }
})
```
* 자바 8
```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
* 메서드 참조(method reference) :: ('이 메서드를 값으로 사용하라'의 의미)를 이용해서 listFiles에 직접 전달 가능
* 람다 : 익명함수
  * 예) (int x) -> x + 1, 즉 'x 라는 인수로 호출하면 x + 1을 반환'
  * 이용할 수 있는 편리한 클래스나 메서드가 없을 때 람다 문법을 이용해 더 간결하게 코드를 구현 가능

### 1.3.2 코드 넘겨주기 : 예제
* 자바 8 이전
* 녹색 사과만 선택

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();

  for (Apple apple : inventory) {
    if (GREEN.equals(apple.getColor())) {
        result.add(apple);
    }
  }
  return result;
}
```
* 누군가는 무게로 필터링 (150그램 이상)
```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
  List<Apple> result = new ArrayList<>();
  
  for (Apple apple : inventory) {
    if (apple.getWeight() > 150) {
      result.add(apple);
    }
  }
  return result;
}
```
* 자바 8 이후
```java
class a {
	public static boolean isGreenApple(Apple apple) {
		return GREEN.equals(apple.getColor());
    }

    public static boolean isHeavyApple(Apple apple) {
        return apple.getWeight() > 150;
    }
	
	public interface Predicate<T> {
		boolean test(T t);
    }
	
	static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
		List<Apple> result = new ArrayList<>();
		for (Apple apple : inventory) {
			if (p.test(apple)) {
				result.add(apple);
            }
        }
		return result;
    }
}
```
  ***predicate(프레디케이트)** : 인수로 값을 받아 true나 fasle를 반환하는 함수를 프레디케이트라고 함.*

  
* 다음처럼 메서드 호출 가능
```java
filterApples(inventory, Apple::isGreenApple);
filterApples(inventory, Apple::isHeavyApple);
```

## 1.4 스트림
* 스트림 X
```java
Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();
for (Transaction transaction : transactions) {
    if (transaction.getPrice() > 1000) {
        Currency currency = transaction.getCurrency();
        List<Transaction> transactionsForCurrency = transactionByCurrencies.get(currency);
        if (transactionsForCurrency == null) {
            transactionsForCurrency = new ArrayList<>();
            transactionByCurrencies.put(currency, transactionsForCurrency);
        }
        transactionsForCurrency.add(transaction);
    }
}
```

* 스트림 API 이용
```java
import java.util.stream.Collectors.groupingBy;

Map<Currency, List<Transaction>> transactionsByCurrencies 
        = transactions.stream()
                .filter((Transaction t) -> t.getPrice() > 1000)
                .collect(groupingBy(Transaction::getCurrency));
```

### 1.4.1 멀티스레딩은 어렵다
* 자바 8은 스트림 API로 '컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드 문제' 그리고 '멀티코어 활용 어려움' 이라는 두가지 문제를 모두 해결
* 즉, 자주 반복되는 패턴으로 주어진 조건에 따라 데이터를 필터링하거나, 데이터를 추출하거나, 데이터를 그룹화 하는 등의 기능이 있음

## 1.5 디폴티 메서드와 자바 모듈
* 자바 9 의 모듈 시스템은 모듈을 정의하는 문법을 제공하므로 이를 이용해 패키지 모음을 포함하는 모듈을 정의
* 또한 자바 8 에서는 인터페이스를 쉽게 바꿀 수 있도록 디폴트 메서드를 지원
* 디폴트 메서드를 이용하면 기존의 코드를 건드리지 않고도 원래의 인터페이스 설계를 자유롭게 확장 할 수 있음
* 예를 들어 자바 8에서는 List 에 직접 sort 메서드를 호출할 수 있음

```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```

## 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어
* 자바 8 에서는 NullPointer 예외를 피할 수 있도록 Optional 클래스 제공
* Optional은 값이 없는 상황을 어떻게 처리할 지 명시적으로 구현하는 메서드를 포함
* 패턴 매칭

## 1.7 마치며
* 언어 생태계의 모든 언어는 변화 후 살아 남거나 사라진다.
* 자바 8은 프로그램을 더 효과적이고 간결하게 구현할 수 있는 새로운 개념과 기능 제공
* 함수는 일급값
* 자바 8 스트림 일부는 컬렉션에서 가져옴
* 자바 9 부터 모듈을 이용해 시스템의 구조를 만들 수 있고 디폴트 메서드를 이용해 기존 인터페이스를 구현하는 클래스를 바꾸지 않고도 인터페이스 변경 가능
* 함수형 프로그래밍에서의 null 처리 방법

    
