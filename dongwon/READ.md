## 자바 8, 9, 10, 11

</br>

### 자바 8은 왜 생겼나?

</br>

현재 사용하는 laptop, desktop 에서는 듀얼, 쿼드 코어이상을 지원하는 CPU가 내장되어 있지만 

자바에서는 8을 쓰기 전까지 코어 중 하나만 사용했다

(나머지 코어는 idle 상태나, 운영체제, 바이러스 검사 프로그램과 함께 파워를 나눠 사용)

스레드를 사용해 관리하기어려운 문제를 쉽게 관리하고 에러가 덜 발생하는 방향으로 발전함

</br>

        자바 1.0   스레드, 락, 메모리 모델 지원(동시성 지원)

        자바 5      스레드 풀, 병렬 실행 컬렉션

        자바 7      포크 / 조인 프레임워크 제공(병렬실행에 도움을 주는)

        자바 8      병렬 실행을 단순한 방식으로 접근하는 방법 제공

        (자바 9     리액티브 프로그래밍 지원 - 병렬 실행 기법)

        자바 8 새로운 기능(간결한 코드와 멀티코어 프로세서의 쉬운 활용)

</br>

- **스트림 API**
    
    <aside>
    🎃 데이터베이스의 언어에서 동작을 표현하면
    구현에서(자바는 스트림 라이브러리) 최적의 저수준 실행방법 선택해 동작함
    
    stream을 사용함으로 비용이 비싼 동기화**(Synchronized)** 키워드를 사용하지 않아도됨
    
    </aside>

</br>

- **메서드에 코드를 전달하는 기법**(메서드 참조와 람다)
    
    ```java
    
    List<String> strings = Arrays.asList("apple", "banana", "cherry");
    strings.stream()
           .map(String::toUpperCase)
           .forEach(System.out::println);
    ```

</br>

- **인터페이스의 디폴트 메서드**
    
    ```java
    public interface MyInterface {
        void abstractMethod();
    
        default void defaultMethod() {
            System.out.println("This is a default method in MyInterface");
        }
    }
    ```
</br>

### 왜 자바는 왜 변화할까(어떻게 살아남았을까?)

객체지향의 각광

1. 캡슐화 ( 관련있는 변수와 함수를 묶어 외부에 쉽게 접근하지  못하게 함)
2. 객체지향의 정신적 모델(윈도우 95, WIMP모델에 쉽게 대응할수 있었음)

</br>

C / C++ 에 비해 추가 비용이 있었지만 하드웨어의 발전으로 프로그래머의 시간이 더 중요해짐

</br>

**스트림 처리**

스트림 : 한 번에 한 개씩 만들어지는 연속적인 데이터 항목의 모임

이론적으론 입력스트림에서 데이터 한개 읽어서 출력으르림으로 데이터 한개씩 기록

(ex. 자동자 공장에서 각 공정마다 하나씩 처리하지만 공장전체는 다 돌아가고 있다)

![출처 : [https://yooniron.tistory.com/36](https://yooniron.tistory.com/36)](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b18b14f-ecd4-426a-b996-a21c32aba069/Untitled.png)

출처 : [https://yooniron.tistory.com/36](https://yooniron.tistory.com/36)

자바 패키지의 `Stream<T>` 는 T형식으로 구성된 일련의 항목을 의미한다
( 조립 라인과 같이 어떠한 항목을 연속으로 제공하는 기능)

**핵심은** 기존에는 한 번에 한 항목을 처리했지만, 하려는 작업을 (데이터베이스 질의와 같이) 고수준 추상화해 일련의 스트림으로 처리할수 있게됨 + 입력부분 여러 CPU 코어에 쉽게 할당할 수 있게 됨(병렬성)

**동작 파라미터화**

메서드(내 코드)를 다른 메서드의 인수로 넘겨주는 기능

```java
public int compareUsingCustomerId(String inv1, String inv2) {
		...
}
 
-> sort 동작 파라미터와 같이 **스트림 API는 연산의 동작을 파라미터화 한다는 개념**
```

**병렬성과 공유 가변 데이터**

스트림은 병렬성을 얻는 대신에 코드 동작방식을 조금 바꾼다(다른 코드와 동시 실행에도 안전할수 있게 → 필요조건. 공유된 가변 데이터에 접근 하지 않아야 )

함수형 프로그래밍(어떻게 할 건지 나타내기보단 무엇을 할건지 설명)

공유되지 않은 가변 데이터, 메서드, 함수 코드를 다른 메서드로 전달, 순수함수 조합해 소프트웨어 만듬

ex) 클로저, 하스켈, 리스프

명령형 프로그래밍(무엇을 할 것인지 나타내기보다 어떻게 할 것인지 설명하는 방식)

인수를 결과로 변환, 일련의 가변 상태로 프로그램 정의, 수학적 함수와 같이 정해진 기능만 수행, 다른 부작용 일으키지 않음

ex) C++, Java, C#

**자바의 진화**

고전적 객체지향에서 벗어나 함수형 프로그래밍으로 다가감

함수형 프로그래밍 특

ex) A → B이동 

이동할수 있는 모든 경로 대표값 생성이 최우선이며, 그 작업을 어떻게 수행할지는 별개로 취급함

객체지향과 정반대로 보이지만 함수형을 도입하므로 두가지 이점을 쓸수있는 것

### 자바 함수

함수 = 메서드 ( 정적 메서드) 로 사용

1급 - 기본값, 객체 (객체 참조는 인스턴스 가리킴)

2급 - 구조체(메서드, 클래스) 그 자체로 값이 안됨

자바 8에서 2급을 1급으로 바꾸도록 기능 추가함(메서드를 값으로 취급할수 있게!!)

**메서드 참조**

```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
		public boolean accept(File file) {
				return file.isHidden(); // 숨겨진 파일 필터링
		}
});

File[] hiddenFiles = new File(".").listFiles(File::isHidden);
// isHidden 함수는 준비되어 있어 직접 전달
// new로 객체 참조를 -> File::isHidden 으로 메서드 참조함
```

> 메서드 참조 :: (이 메서드를 값으로 사용해라)
> 

**람다 : 익명 함수**

(int x) → x + 1

x 호출시 x+1 반환

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd6b56e9-fd90-4b33-91c2-cf00273dfe4f/Untitled.png)

<aside>
🐕‍🦺 프레디케이드(predicate)
Apple::isGreenApple 메서드로 값을 넘겨 줄대 Function<Apple, Boolean> 과 같이 코드 구현보다 Predicate<Apple> 을 사용해서 boolean 으로 변환과정 없이 true, false 반환하는것이 더 효율적이며 표준적이다

</aside>

**메서드 전달 람다**

```java
한번만 사용할 메서드를 매번 정의하지 않고 람다 (또는 익명함수)로 값을 넘김

filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()) );
filterApples(inventory, (Apple a) -> a.getWeight() > 150 );
filterApples(inventory, (Apple a) -> a.getWeight() < 80 || RED.equals(a.getColor()) );
```

**스트림**

외부 반복 : for-each 루프를 통해 각요소를 반복하며 작업수행

내부 반복 : 스프림 API를 사용해서 반복

컬랙션에 비해 처리 시간을 줄일수 있음

**멀티스레딩**

멀티스레팅 환경에서는 각각의 스레드는 동시에 공유된 데이터에 접근하며, 데이터를 갱신함

스레드를 잘 제어하지 못할시 데이터가 바뀔 수 있음

스트림 API가 해결

반복되는 패턴 **필터링**, 데이터 **추출**, **그룹화** 기능 제공

두 CPU 가진 환경에서 리스트 필터링. 이 과정을 **포킹 단계**라 함

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b6b8bc89-fa1c-4040-b767-3faff58a2a21/Untitled.png)

컬렉션과 비슷한 방식으로 동작하지만

컬렉션은 어떻게 데이터 저장하고 접근할지에 중점을 두며

스트림은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중점을 둠(스트림 내 요소 쉽게 병렬처리 가능)

<aside>
🌟 자바 8 요약

라이브러리에서의 분할 처리
큰 스트림을 병렬로 처리하도록 작은 스트림으로 분할

filter같은 라이브러리 메서드로 전달된메서드가 상호작용 하지 않으면 가변 공유 객체 통해 공짜 병렬성
(Apple::isGreenApple 컴포넌트 간 상호작용 하지 않음)

</aside>

**디폴트 메서드 자바 모듈**

인터페이스 바꿀수 있도록 디폴트 메서드 지원 → 구현하지 않다도 되는 메서드를 인터페이스에 추가 할 수 있는 기능 제공. ❗클래스 구현이 아닌 인터페이스의 일부임 

```java
자바8 인터페이스 디폴트 메서드 (정적 메서드 Collections.sort 를 호출함)

default void sort((Compaator<? super E> c) {
		Collections.sort(this, c);
}
```

> 다이아몬드 상속 문제 : 어떤 부모의 메서드를 호출해야할지 모호해지는 문제
> 

NullPointer 예외를 피하는 Optional<T> 클래스 제공

Optional<T> 는 값을 갖거나 갖지 않을수 있는 컨테이너 객체임

구조적 패턴 매칭 기법 (if - then - else)

스칼라의  expr match = switch(expr) 같은 기능