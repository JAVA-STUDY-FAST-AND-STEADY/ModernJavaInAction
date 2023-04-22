# Chap 02. 동작 파라미터화 코드 전달하기
- 소비자 요구사항은 항상 바뀜. 변화하는 요구사항은 소프트웨어 엔지니어링에서 피할 수 없음.
- 동작 파라미터화 (behavior parameterization)를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응 가능
  - 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미

## 2.1 변화하는 요구사항에 대응하기
- 예제 : 농장 재고 목록 리스트 중 녹색 사과만 필터릴

### 2.1.1 첫 번째 시도 : 녹색 사과 필터링
- Color Enum이 존재한다고 가정
```java
enum Color {
	RED, GREEN
}
```
- 첫 번째 시도 
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

### 2.1.2 두 번째 시도 : 색을 파라미터화
- 농부의 변심으로 녹색 사과 말고 빨간 사과도 필터링해야 함
  - 거의 비슷한 코드가 반복 존재하면 그 코드를 추상화함

- 기존과는 다르게 메서드화 시키고 Color 값을 파라미터화 시킴.
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getColor().equals(color)) {
        result.add(apple);
      }
    }
    return result;
}
```
- 다음과 같이 호출
```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

- 이후 농부가 무게가 150 그램 이상인 사과만 필터링 하는 것으로 요구사항을 바꾼다면?
```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getWeight() > weight) {
        result.add(apple);
      }
    }
    return result;
}
```
- 위 코드도 좋은 해결책이지만 목록을 검색하고 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복됨.
- 이는 소프트웨어 공학의 DRY(Don't repeat yourself), 같은 것을 반복하지 말 것 원칙을 어기는 것.

### 2.1.3 세 번째 시도 : 가능한 모든 속성으로 필터링
- 모든 속성을 메서드 파라미터로 추가한 코드
```java
public static List<Apple> filterApplesByColor(
    List<Apple> inventory, Color color, int weight, boolean flag
) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if ((flag && apple.getColor().equals(color)) ||
        (!flag && apple.getWeight() > weight) ) {
        result.add(apple);
      }
    }
    return result;
}
```
- 다음처럼 위 메서드 사용
```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, true);
```
- 형편없는 코드다. true, false는 의미하는 바가 명확하지 않고 의미없는 값이 추가되며 요구 사항이 바뀌면 유연하게 대응도 불가
  -  내가 만약 color 만 필터링하고 싶다면? appleFilter(apples, Apple.Color.Green, 0) 이렇듯 의미없는 0이 들어가버림
- 이와 같은 문제는 동작 파라미터화를 이용해서 유연성을 얻는 방법으로 해결.

## 2.2 동작 파라미터화
- 참 또는 거짓을 반환하는 함수를 프레디케이트라고 한다. 선택 조건을 결정하는 인터페이스를 정의
- 선택 조건을 결정하는 인터페이스 정의

### 2.2.1 네 번째 시도 : 추상적 조건으로 필터링
```java
public interface ApplePredicate{
     boolean filter(Apple apple);
}

/*컬러 필터*/
class AppleColorFilter implements ApplePredicate{
    @Override
    public boolean filter(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}

/*무게 필터*/
class AppleWeightFilter implements ApplePredicate{
    @Override
    public boolean filter(Apple apple) {
        return apple.getWeight() >= 150;
    }
}
```
```java
public static List<Apple> appleFilter(List<Apple> apples, ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for (Apple apple : Apples){
        if (p.filter(apple)){
            result.add(apple);
        }
    }
    return result;
}
```
```java
public static void main(String[] args) {
    appleFilter(Apples,new AppleColorFilter())
            .stream().forEach(apple -> System.out.println(apple.color));
    appleFilter(Apples,new AppleWeightFilter())
            .stream().forEach(apple -> System.out.println(apple.color));
}
```
- 위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있다. 이를 전략 디자인 패턴 (strategy design pattern)이라고 함
- 전략 디자인 패턴은 각 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법
- 이제 필요한 대로 다양한 ApplePredicate를 만들어서 filterApples 메서드로 전달할 수 있다.
  - 예를 들어 무게 150 초과, 색깔이 빨간색인 사과만 검색해달라고 요구받으면, 다음과 같은 ApplePredicate 를 구현하는 클래스만 만들면 된다.
- 아까의 문제 사항이 파라미터화를 통해서 해결된 것을 알수 있고 코드가 더욱 명시적이게 되고 간결해짐.

![스크린샷 2023-04-15 오후 7 19 18](https://user-images.githubusercontent.com/87894946/232208188-ea361852-cc8f-4656-85bc-1b2d0de2e438.png)
<br/>
이처럼 각 항목에 적용할 동작을 분리할 수 있다는 것은 동작 파라미터화의 강점이다.
<br/>
<br/>
하지만 이러한 파라미터화에 대한 문제도 있기 마련이다.<br/> 여러 클래서를 구현해서 인스터스화하는 과정이 거추장스럽게 느껴질 수 있다.
그렇다면 이를 어떻게 개선할 수 있을까?

## 2.3 복잡한 과정 간소화
```java
/*무거운 사과만 선택*/
public class AppleHeavyWeightPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return apple.getWeight() > 150;
    }
}

/*녹색 사과만 선택*/
public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}

public class FilteringApples {
	public static void main(String.. args) {
		List<Apple> inventory = Arrays.asList(new Apple(80, GREEEN),
                                              new Apple(155, GREEEN),
                                              new Apple(120, GREEEN));
        List<Apple> heavyApples =
                filterApples(inventory, new AppleHeavyWeightPredicate());
        List<Apple> greenApples =
                filterApples(inventory, new AppleGreenColorPredicate());
    }
	
	public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
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
- 로직과 관련 없는 코드가 많이 추가되었다, 이를 익명 클래스 (anonymous class) 기법으로 간단하게 할 수 있다.

### 2.3.1 익명 클래스
- 익명 클래스는 자바의 지역 클래스 (local class, 블록 내부에 선언된 클래스)와 비슷한 개념
- 클래스 선언과 인스턴스화를 동시에 할 수 있다. -> 즉석에서 필요한 구현을 만들어 사용 가능

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용
```java
interface ApplePredicate{
    public boolean filter(Apple apple);
}
```
```java
public static List<Apple> appleFilter(List<Apple> apples, ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for (Apple apple : Apples){
        if (p.filter(apple)){
            result.add(apple);
        }
    }
    return result;
}
```
```java
public static void main(String[] args) {

    appleFilter(Apples, new ApplePredicate() { // 익명 객체 활용
        @Override
        public boolean filter(Apple apple) {
            return apple.color.equals(Apple.Color.GREEN);
        }
    }).stream().forEach(apple -> System.out.println(apple.color));
}
```
익명클래스는 여전히 많은 공간을 차지하고, 많은 프로그래머가 익명 클래스의 사용에 익숙하지 않다는 단점이 있다.
람다 표현식으로 코드를 더 간결히 정리할 수 있다.

### 2.3.3 여섯 번째 시도 : 람다 표현식 사용
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```
### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화
- 제네릭을 이용해 추상화 가능
```java
public interface Predicate<T> {
	boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> result = new ArrayList<>();
	for (T e : list) {
		if (p.test(e)) {
			result.add(e);
        }
    }
	return result;
}
```
간단히 말해 사과 뿐 아니라 바나나, 두리안 등 다양한 과일을 동일한 필터를 거쳐야 한다면 그에 맞는 메서드를 전부 만들어야 할 것이다. 하지만 추상화 과정을 통하여 이를 해결할 수 있다.




