# null 대신 Optional 클래스

- [x] 1. 값이 없는 상황을 어떻게 처리할까?
- [x] 2. Optional 클래스 소개
- [x] 3. Optional 적용 패턴
- [x] 4. Optional을 사용한 실용 예제
- [x] 5. 마치며
- [x] 6. 배우고 느낀점

<br><br>

## 1. 값이 없는 상황을 어떻게 처리할까?

---

<br>

 1965년 토니 호어라는 영국 컴퓨터 과학자가 힙에 할당되는 레코드를 사용하며 형식을 갖는 최초의 프로그래밍언어 중 하나인 알골을 설계하면서 처음 null이 등장했다. 그는 구현하기가 쉬웠기에 null를 도입했고, 자바를 포함해서 최근 수십 년간 탄생한 대부분의 언어 설계에는 null 참조 개념을 포함하게 되었다.

 그렇다면 우리는 값이 없는 상황을 어떻게 처리할까?

```java
public class Person {
    private Car car;
    public Car getCar() { return car; }
}

public class Car {
    private Insurance insurance;
    public Insurance getInsurance() { return insurance; }
}

public class Insurance {
    private String name;
    public String getName() { return name; }
}

public String getCarInsuranceName(Person person) {
    return person.getCar().getInsurance().getName();
}
```

 다음과 같은 코드를 볼 때 우리는 직감적으로 문제가 있음을 알 수 있다. 만약 사람이 차를 가지지 않는다면 NullPointerException이 발생하게 될 것이다. 또한 person 그 자체가 null인 경우도, Insurance가 null인 경우도 마찬가지다. 우리는 어떻게 null에 대응했었을까?

<br>

### 1.1 보수적인 자세로 NullPointerException 줄이기

<br>

 우리는 NullPointerException을 피하기 위해 본능적으로 if문을 이용해 null을 구분해갔다.

```java
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "UnKnown";
}
```

 우리는 null을 확인하는 if 문 코드로 인해 나머지 호출 체인의 들여쓰기 수준이 증가함을 볼 수 있다. 그리고 이와 같은 **반복 패턴**코드를 **깊은 의심**이라고 부른다. 이를 반복하다 보면, 코드의 구조가 엉망이 되고 가독성도 떨어진다. 그렇다면 우리는 또다른 방법으로 해결을 해보자.

```java
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "UnKnown";
    }
    if (car == null) {
        return "UnKnown";
    }
    if (insurance == null) {
        return "UnKnown";
    }
    return insurance.getName();
}
```

 위 코드는 조금 다른 방법으로 중첩 if 블록을 없앴다. 즉, null 변수가 있으면 즉시 "Unknown"을 반환한다. 하지만 위 코드는 좋은 코드는 절대 아니다. 하나의 메서드에 탈출구가 4개가 생겼기 때문이다. 결국 출구 때문에 유지보수가 더욱 힘들어 지고, return의 문자열을 반복해서 사용할 경우, 오타 또는 실수가 생길 수 있다.

<br>

### 1.2 null 때문에 발생하는 문제

<br>

 - **에러의 근원이다** : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.

 - **코드를 어지럽힌다** : 때로는 중첩된 null 확인 코드를 추가해야하 하므로 null 때문에 코드 가독성이 떨어진다.

 - **아무 의미가 없다** : null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.

 - **자바 철학에 위배된다** : 자바는 개발자로 부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null포인터다.

 - **형식 시스템에 구멍을 만든다** : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때, 애초에 null이 어떤 의미로 사용되었는지 알 수 없다. (~~격하게 공감한다..~~)


<br>

### 1.3 다른 언어는 null 대신 무얼 사용하나?

<br>

 그루비 같은 언어는 안전 네비게이션 연산자 (?.)를 도입해서 null 문제를 해결했다.

```groovy
def carInsuranceName = person?.car?.insurance?.name
```

 그루비는 안전 네비게이션 연산자를 이용하면, null 참조 예외 걱정 없이 객체에 접근할 수 있다. 또한 하스켈은 Maybe, 스칼라는 Option[T] 제공해 대응했다.

 자바는 선택형 값 개념에 영향을 받아 Optional 클래스를 만들게 되었다.

<br><br>

## 2. Optional 클래스 소개

---

<br>

 Optional은 선택형 값을 캡슐화하는 클래스다. Optional`<T>`의 형식을 가지며, 만약 값이 존재하면 Optional 클래스는 값을 감싸고, 없다면 Optional.empty(싱글턴 인스턴스를 반환하는 정적 팩토리 메서드) 메서드로 Optional을 반환한다. 

 다음은 Optional로 Car, Insurance, Person 클래스를 재정의한 코드다.

```java
public class Person {
    private Optional<Car> car;
    public Optional<Car> getCar() { return car; }
}

public class Car {
    private Optional<Insurance> insurance;
    public Optional<Insurance> getInsurance() { return insurance; }
}
```

 Optional을 이용해 재정의를 하니 조금 더 모델의 의미(sementic)가 더 명확해졌음을 알 수 있다. Optional로 감싼 클래스는 있을 수도 있고 없을 수 도 있지만, 감싸지 않은 클래스는 무조건 존재해야한다. 

<br>

#### Optional 클래스의 메서드

|메서드  | 설명                                               |
|------|---------------------------------------------------|
|empty|빈 Optional 인스턴스 반환|
|filter|값이 존재하며 프레디케이트와 일치하면 값을 포함하는 Optional을 반환하고, 값이 없거나 프레디 케이트와 일치하지 않으면 빈 Optional을 반환함|
|flatMap|값이 존재하면 인수로 제공된 함수를 적용한 결과 Optional을 반환하고, 값이 없으면 빈 Optional을 반환함|
|get|값이 존재하면 Optional이 감싸고 있는 값을 반환, 없으면 NoSuchElementException이 발생|
|ifPresent|값이 존재하면 지정된 Consumer 실행, 값이 없으면 아무일도 일어나지 않음|
|ifPresentOrElse|값이 존재하면 지정된 Consumer 실행, 값이 없으면 실행할 수 있는 Runnable을 인수로 받음|
|isPresent|값이 존재하면 true를 반환하고, 값이 없으면 false를 반환함|
|map|값이 존재하면 제공된 매핑 함수를 적용함|
|of|값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 없으면 NullPointerException을 발생함|
|ofNullable|값이 존재하면 값을 감싸는 Optional 반환, 값이 null이면 빈 Optional을 반환|
|or|값이 존재하면 같은 Optional을 반환, 값이 없으면 Supplier에서 만든 Optional을 반환|
|orElse|값이 존재하면 값을 반환하고, 값이 없으면 기본값을 반환|
|orElseGet|값이 존재하면 값을 반환하고, 값이 없으면 Supplier에서 제공하는 값을 반환|
|orElseThrow|값이 존재하면 값을 반환하고, 값이 없으면 Supplier에서 생성한 예외를 발생|
|stream|값이 존재하면 존재하는 값만 포함하는 스트림을 반환하고, 값이 없으면 빈 스트림을 반환|

<br><br>

## 3. Optional 적용 패턴

---

<br>

 우리는 지금까지 Optional로 null에 대응하고, 도메인 모델의 의미를 더 명확하게 만듦을 알게 되었다. 그러면 실제로는 어떻게 활용할 수 있을까?

<br>

### 3.1 Optional 객체 만들기

<br>

 Optional을 활용하기 위해선 먼저 Optional 객체를 생성해야한다.

```java
// 빈 Optional 생성
Optional<Car> car = Optional.empty();

// null이 아닌 값으로 Optional 만들기
// 또는 정적 팩토리 메서드 Optional.of로 null이 아닌 값을 포함하는 Optional을 만들수 있다.
// 하지만 car가 null인 경우 곧 바로 NullPointerException이 발생한다.
Optional<Car> car = Optional.of(car);

// null값으로 Optional 만들기
// 정적 팩토리 메서드 Optional.ofNullable로 null값 저장할 수 있는 Optional을 만들 수 있다.
// car가 null인 경우 빈 Optional 객체가 반환된다.
Optional<Car> optCar = Optional.ofNullable(car);
```

 Optional은 값을 가져올 때 get 메서드를 이용해 값을 가져올 수 있다. 하지만 Optional이 비어있는 경우 get을 호출했을 때 예외가 발생한다. 이 점을 주의하자.

<br>

### 3.2 맵으로 Optional의 값을 추출하고 변환하기

<br>

 보통 객체의 정보를 추출할 때에는 Optional을 사용할 때가 많다. 이전에 이용했던 Insurance 클래스에서 값을 뽑는 코드를 가져와보자.

```java
String name = null;
if (insurance != null) {
    name = insurance.getName();
}
```

 우리는 null값에 대응하기 위해 지금까지 위와 같은 코드를 작성했다. Optional은 이런 유형의 패턴에 사용하게끔, map 메서드를 지원한다.

```java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
```

 Optional의 map 메서드는 스트림의 map 메서드와 개념적으로 비슷하다. 만약 Optional이 비어있는 경우 아무 일도 일어나지 않는다. 

<br>

### 3.3 flatMap으로 Optional 객체 연결

<br>

 우리는 스트림에서 배운 평면화를 통해 다음과 같은 코드가 안된다는 것을 알 수 있다.

```java
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson.map(Person::getCar)
                                 .map(Car::getInsurance)
                                 .map(Insurance::getName);
```

 이유를 간단히 설명하면 처음 map을 실행할 당시 Optional`<Person>`에서 Optional`<Car>`로 바뀌길 기대하지만, Optional`<Optional<Car>>`로 바뀌게 된다. 이후 map도 마찬가지다. 하지만 flatMap은 인수로 받은 함수를 적용해서 생성된 각각의 스트림에서 콘텐츠만 남긴다.

```java
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName)
                 .orElse("Unknown");
}
```

 우리는 null을 확인하는 기존의 방법(많은 조건 분기)를 사용하지않고, 이해할 수 있는 코드를 작성하게 되었다. 

<br>

#### 도메인 모델에 Optional을 사용했을 때, 데이터를 직렬화 할 수 없는 이유

 Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로, Serializable 인터페이스를 구현하지 않는다. 따라서 우리 도메인 모델에 Optional을 사용한다면, 직렬화 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다. 하지만 이와 같은 단점에도 불구하고 여전히 Optional을 사용해서 도메인 모델을 구성하는 것이 바람직 하다고 생각된다. 특히 객체 그래프에서 일부 또는 전체 객체가 null일 수 있는 상황이라면 더욱 그렇다. 직렬화 모델이 필요하다면 다음 예제에서 보여주는 것처럼 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.

```java
public class Car {
    private Car car;
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```


<br>

### 3.4 Optional 스트림 조작

<br>

 자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다. 다음은 Person/Car/Insurance 도메인 모델을 사용한 해 사람들이 가입한 보험 회사의 이름의 집합을 구하는 코드다.

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
    return persons.stream()
                  .map(Person::getCar) // 사람 목록을 각 사람이 보유한 자동차의 Optional<Car> 스트림으로 변환
                  .map(optCar -> optCar.flatMap(Car::getInsurance))
                  .map(optIns -> optIns.map(Insurance::getName))
                  .flatMap(Optional::stream) // Stream<Optional<String>>을 현재 이름을 포함하는 Stream<String>으로 변환
                  .collect(toSet());
}
```

 우리는 스트림 요소를 조작하려면 긴 메서드 체인이 필요함을 알고있다. 하지만 Optional으로 값을 감싸고 있어 메서드 체인이 더 길어짐을 알 수 있게 되었다. 또한 이런 변환 과정을 거치고 나서도 결과가 비어있는 경우가 존재한다. Optional 덕분에 이런 종류의 연산을 null 걱정 없이 안전하게 처리할 수 있지만, 마지막 결과를 얻으려면 빈 Optional을 제거하고 값을 언랩해야 한다. 다음의 코드와 같이 filter, map을 순서대로 이용해 결과를 얻을 수 있다.

```java
Stream<Optional<String>> stream = ...
Set<String> result = stream.filter(Optional::ifPresent)
                           .map(Optional::get)
                           .collect(toSet());
```

<br>

### 3.5 디폴트 액션과 Optional 언랩

<br>

 우리는 빈 Optional인 상황에서 기본값을 반환하도록 orElse로 Optional을 읽었다. Optional 클래스는 이 외에도 Optional 인스턴스에 포함된 값을 읽는 다양한 방법을 제공한다.

 - get()은 값을 읽는 가장 간단한 메서드 이며, 동시에 가장 안전하지 않다. 값이 없는 경우 NoSuchElementException이 발생한다.

 - orElse 메서드를 이용하면, Optional이 값을 포함하지 않을 때 기본 값을 제공할 수 있다.

 - orElseGet(Supplier<? extends T> other)는 orElse 메서드에 대응하는 **게으른 버전**의 메서드다. Optional의 값이 없을 때만 Supplier가 실행되기 때문, 다만 디폴트 메서드를 만드는 데 시간이 걸리거나, Optional이 비어 있을 때만 기본값을 생성하고 싶다면(기본값이 반드시 필요한 상황) orElseGet(Supplier<? extends T> other)를 사용해야 한다.

 - orElseThrow(Supplier<? extends X>)는 Optional이 비어있을 때 예외를 발생시킨다는 점에서 get과 비슷하지만 던질 예외를 선택할 수 있다.

 - ifPresent(Consumer<? super T> consumer)를 이용하면, 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다. 값이 없다면 아무일도 일어나지 않는다.

 - ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction). 이 메서드는 Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPresent와 다르다. 

<br>

### 3.6 두 Optional 합치기

<br>

 이제 Person과 Car정보를 이용해 가장 저렴한 보험료를 제공하는 보험회사를 찾아보자.

```java
public Insurance findCheapestInsurance(Person person, Car car) {
    // 서비스 조회 로직
    // 로직은 중요하지않다.
    return cheapestCompany;
}

// 우리는 findCheapestInsurance에 들어간 person과 car는 둘 다 null이 되면 안된다. 따라서 nullSafeFindeCheapestInsurance 메서드를 만들고 전달해주자.

public Optional<Insurance> nullSafeFindeCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (car.isPresent() && person.isPresent()) {
        return Optional.of(findCheapestInsurance(person.get(), car.get()));
    } else {
        return Optional.empty();
    }
}
```

 위와 같이 Optional을 이용해 두 Optional을 합칠 수 있다. 하지만 if문을 사용하는 것은 곧 null 조건문으로 비교하는 것과 다른게 없다. 우리는 Optional을 언랩하지 않고 두 Optional을 합치는 코드를 구현해보자.

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

<br>

### 3.7 필터로 특정값 거르기

<br>

 우리는 종종 객체의 메서드를 호출해서, 어떤 프로퍼티를 확인해야 할 때가 있다. 예를 들어 보험회사 이름이 'CambridgeInsurance'인지 확인해야 될 때, 이 작업을 안전하게 수행하려면 다음 코드에서 보여주는 것처럼 Insurance 객체가 null인지 여부를 확인한 다음에 getName 메서드를 호출해야한다.

```java
Insurance insurance = ...;
if (insurance != null && "CambridgeInsurance".equals(insurance.getName())) {
    System.out.println("ok");
}
```

 우리는 Optional 객체에 filter 메서드를 이용해 다음과 같은 코드르 재구성할 수 있다.

```java
Optiona<Insurance> optInsurance = ...;
optInsurance.filter(p -> "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```


<br><br>

## 4. Optional을 사용한 실용 예제

---

<br>

 지금까지 배운 Optional 클래스를 효과적으로 사용하려면, 잠재적으로 존재하지 않은 값의 처리 방법을 바꿔야 한다. Optional의 기능을 최대한 활용할 수 있도록, 우리 코드에 작은 유틸리티 메서드를 추가하는 방식으로 이 문제를 해결해보자.

<br>

### 4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

<br>

 null 에러가 발생하는 자료구조 중 대표적인 예로 Map이 존재한다. 키값에 없는 값을 넣으면 NullPointerException이 발생한다. 우리는 Optional을 이용해 대응할 수 있다.

```java
Optional<Object> value = Optiona.ofNullable(map.get("key"));
```

<br>

### 4.2 예외와 Optional 클래스

<br>

 자바 API는 어떤 이유에서 값을 제공할 수 없을 때 null을 반환하는 대신 예외를 발생시킬 때도 있다. 이것에 대한 전형적인 예가 문자열을 정수로 변환하는 정적 메서드다.

```java
Integer.parseInt("oe"); -> NumberFormatExcetion

public static Optional<Integer> stringToInt(String s) {
    try {
        return Optional.of(Integer.parseInt(s));
    } catch(NumberFormatException e) {
        return Optional.empty();
    }
}
```

 이와 같이 유틸리티 클래스를 만들게 되면 복잡해 보이는 try - catch 구문을 깔끔하게 이용할 수 있다.

<br>

### 4.3 기본형 Optional을 사용하지 말아야 하는 이유

<br>

 기본형 특화 Optional인 OptionalInt, OptionalLong, OptionalDouble을 사용해서는 안된다. 왜냐하면 map, flatMap, filter 등을 지원하지 않는다.

<br>

### 4.4 응용

<br>

 Optional 클래스의 메서드를 실제 업무에서 어떻게 활용할 수 있는지 살펴보자. 예를 들어 프로그램의 설정 인수로 Properties를 전달한다고 가정하자. 그리고 다음과 같은 Properties로 우리가 만든 코드를 테스트할 것이다.

```java
    Properties props = new Properties();
    props.setProperty("a", "5");
    props.setProperty("b", "true");
    props.setProperty("c", "-3");
```

 이제 프로그램에서는 Properties를 읽어서 초 단위의 지속 지간으로 해석한다. 다음과 같은 메서드 시그니처로 지속 시간을 읽을 것이다.

```java
public int readDuration(Properties props, String name);
```

 지속 시간은 양수영 하므로 문자열이 양의 정수를 가리키면 해당 정수를 반환하지만, 그 외에는 0을 반환한다.

```java
    assertEquals(5, readDurationImperative(props, "a"));
    assertEquals(0, readDurationImperative(props, "b"));
    assertEquals(0, readDurationImperative(props, "c"));
    assertEquals(0, readDurationImperative(props, "d"));
```

 프로퍼티에서 지속 시간을 읽는 명령형 코드

```java
  public static int readDurationImperative(Properties props, String name) {
    String value = props.getProperty(name);
    if (value != null) {
      try {
        int i = Integer.parseInt(value);
        if (i > 0) {
          return i;
        }
      } catch (NumberFormatException nfe) { }
    }
    return 0;
  }
```

 if문과 try catch문이 중첩되며 코드가 복잡하고 가독성도 많이 떨어지게 되었다. 이제 Optional을 이용해 다시 구현해보자.

```java
  public static int readDurationWithOptional(Properties props, String name) {
    return ofNullable(props.getProperty(name))
            .flatMap(ReadPositiveIntParam::s2i)
            .filter(i -> i > 0)
            .orElse(0);
  }
```

<br><br>

## 5. 마치며

---

<br>

 - 역사적으로 프로그래밍 언어에서는 null 참조로 값이 없는 상황을 표현해왔다.

 - 자바 8에서는 값이 있거나 없음을 표현할 수 있는 클래스 java.util.Optional`<T>`를 제공한다.

 - 팩토리 메서드 Optional.empty, Optional.of, Optional.ofNullalbe 등을 이용해서 Optional 객체를 만들 수 있다.

 - Optional 클래스는 스트림과 비슷한 연산을 수행하는 map, flatMap, filter 등의 메서드를 제공한다.

 - Optional로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있다. 즉, Optional로 예상치 못한 null 예외를 방지할 수 있다.

 - Optional을 활용하면, 더 좋은 API를 설계할 수 있다. 즉, 사용자는 메서드의 시그니처만 보고도 Optional 값이 사용되거나 반환되는지 예측할 수 있다. (도메인 모델링이 더 명확해진다.)


<br><br>

## 6. 배우고 느낀점

---

<br>

 학교에서 보험사 프로젝트를 구현해보며, 생각보다 getter/setter를 자주사용했고, 사용자가 입력하는 과정에서 처리해야될 예외들이 너무나도 많았다. Optional을 이용한다면 더 편하게 처리할 수 있음을 프로젝트가 끝나고 알게되어 아쉽다. 개념적인 부분은 모두 이해가 되었다. 하지만 책만 보고 모든걸 할 수는 없다. 실제로 구현을 해보며 개념을 적용 시켜야겠다.

<br><br>