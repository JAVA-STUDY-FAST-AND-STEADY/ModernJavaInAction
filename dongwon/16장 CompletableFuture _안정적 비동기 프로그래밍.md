# CompletableFuture : 안정적 비동기 프로그래밍

### Future의 단순 활용

- 자바 5부터 Future인터페이스 제공(미래의 어느 시점에 결과 얻는 모델에 활용)
- 비동기 계산 모델링에 이용
- 계산이 끝났을 때 결과에 접근할 수 있는 참조 제공
- 시간이 걸리는 작업 Future내부로 설정할 시 결과 기다리며 다른 일 수행 가능.

ex) 단골 세탁소에 옷을 맡기며, 일이 언제 끝날지 알려줌. 드라이 되는 동안 나는 다른 일을 할 수가 있음

저수준의 스레드에 비해 직관적이며 이해하기 쉽다. 

<br>

`Future`이용을 위해서는 시간 오래 걸리는 작업을 `Callable`객체 내부로 감싼 다음 `ExecutorService`에 제출해야 함

`ExecutorService`에서 제공하는 스레드가 시간 오래 걸리는 작업 처리하는 동안 우리 스레드로 다른 작업 동시에 실행. 다른 작업 처리하다 오래걸리는 작업의 결과가 필요하면 Future의 get메서드 통해 결과를 가져옴. 이미 완료되 결과가 준비되있으면, 즉시 반환하지만, 준비 되지 않았으면. 작업 완료까지 스레드를 블록 시킨다

Future로 시간 오래 걸리는 작업 비동기적으로 실행

![Untitled](./image/Future%EB%A1%9C%20%EC%8B%9C%EA%B0%84%20%EC%98%A4%EB%9E%98%20%EA%B1%B8%EB%A6%AC%EB%8A%94%20%EC%9E%91%EC%97%85%20%EB%B9%84%EB%8F%99%EA%B8%B0%EB%A1%9C%20%EC%8B%A4%ED%96%89.png)

(문제)오래 걸리는 작업이 영원히 끝나지 않는다면 문제가 생긴다

(해결방법) `get`메서드 오버로드해서 스레드 대기할 최대 타임아웃 시간을 설정하는 것이 좋다

<br>

**Future 제한**

Future 인터페이스가 비동기 계산 끝났는지 확인할 수 있는 `isDone` 메서드, 계산이 끝나길 기다리는 메서드, 결괴 회수 메서드 등을 제공함

But 이들 메서드만으론 간결한 동시 실행 코드 구현하기에 충분하지 않음. ex) Future의 결과 있을 때 이들의 의존성은 표현하기 어렵다(Future로 구현 어려움)

<br>

<b>`Future` 를 선언형으로 이용하는 `CompletableFuture`클래스(Future 인터페이스를 구현한 클래스) 선언형 기능들</b>

- 두 개의 비동기 계산 결과 하나로 합침. 계산 결과는 서로 독립적일 수 있으며, 두 번째 결과가 첫 번째 결과에 의존하는 상황일 수 있음
- `Future` 집합이 실행한느 모든 테스크의 완료를 기다림
- `Future` 집합에서 가장 빨리 완료되는 테스크를 기다렸다 결과 얻음(ex. 여러 테스크가 다양한 방식으로 같은결과 구함)
- 프로그램적으로 `Future` 를 완료시킨다(비동기 동작에 수동으로 결과 제공)
- `Future` 완료 동작에 반응(결과를 기다리며 블록되지 않고 결과가 준비되었다는 알림 받은 후 `Future` 의 결과로 원하는 추가 동작을 수행할 수 있음

<br>

`Stream`과 `CompletableFuture`는 비슷한 패턴, 즉 람다 표현식과 파이프라이닝을 활용한다.

`Future` 와 `CompletableFuture`의 관계는 `Collections`과 `Stream`의 관계와 같다

<br>

> 컬렉션은 요소 그룹을 저장하고 관리하는 방법을 제공하는 반면 스트림은 게으르고 비파괴적인 방식으로 컬렉션의 요소를 처리하고 변환하는 기능적 프로그래밍 접근 방식을 제공. 스트림은 컬렉션 위에 구축되므로 보다 표현력 있고 효율적인 데이터 조작이 가능
> 
<br>

---

**동기 API와 비동기 API**

**동기 API**

- **동기API** 에서는 메서드를 호출한 후 메서드가 계산을 완료할 때까지 기다렸다, 메서드가 반환되면 호출자는 반환된 값으로 계속 다른 동작을 수행
- 호출자와 피호출자가 각각 다른 스레드에서 실행되는 상황이어도 호출자는 피호출자의 동작 완료를 기다렸을 것
- 동기 API를 사용하는 상황을 **블록 호출**이라함

**비동기 API**

- 메서드가 즉시 반환, 끝내지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당
- 비동기 API를 사용하는 상황을 **비블록 호출**이라함
- 다른 스레드에 할당된 나머지 계산 결과는 콜백 메서드를 호출해서 전달하거나, 호출자가 ‘계산 결과가 끝날 때까지 기다림’ 메서드를 추가로 호출하면서 전달

ex) 주로 I / O 시스템프로그래밍에서 이와 같이 동작 수행. → 즉, 계산 동작 수행 동안 비동기적으로 디스크 접근을 수행, 더 이상 수행할 동작 없으면 디스크 블록이 메모리로 로딩될 때까지 기다림

---
<br>

### **`CompletableFuture`로 비동기 애플리케이션 만들기**

**비동기 API 구현**

```java
public Shop(String name) {
    this.name = name;
    random = new Random(name.charAt(0) * name.charAt(1) * name.charAt(2));
  }

  public String getPrice(String product) {
    double price = calculatePrice(product);
    Discount.Code code = Discount.Code.values()[random.nextInt(Discount.Code.values().length)];
    return name + ":" + price + ":" + code;
  }

  public double calculatePrice(String product) {
    delay();
    return format(random.nextDouble() * product.charAt(0) + product.charAt(1));
		// 임의의 계산 값 반환(제품명에)
  }
```

`getPrice` 메서드는 상점 데이터베이스 이용해 가격 정보 얻는 동시에 다른 외부 서비스에도 접근(ex. 물건 발행사, 제조사 관련 프로모션 할인)

(문제)

사용자가 API호출시 비동기 동작 완료까지 1초동안 블록된다. 네트워크상 모든 인상점 가격 검색해야 해서 블록 동작은 바람직하지 않음

<br>

**동기 메서드 → 비동기 메서드로 변환**

동기메서드 `getPrice`를 비동기 메서드로 변환하기 위해 이름과 반환값 바꾸기

```java
public Future<Double> getPriceAsync(Stirng product) {... }
```

비동기 계산의 결과 표현할 수 있는 Future인터페이스 사용(자바 5부터 제공됨) → 사용함으로서 호출자 스레드가 블록되지 않고 다른 작업을 실행할 수 있음

Future는 결과값의 핸들일 뿐, 계산이 완료되면 get 메서드로 결과 얻을 수 있음. getPriceAsync 메서드는 즉시 반환되어 호출자 스레드는 다른 작업을 수행할 수 있다. 

 `CompletableFuture`클래스는 `getPriceAsync`구현에 도움되는 기능 제공

```java
public Future<Double> getPrice(String product) {
  CompletableFuture<Double> futurePrice = new CompletableFuture<>();
	// 계산 결과를 포함할 CompletableFuture를 생성
  new Thread(() -> {
      double price = calculatePrice(product);
			// 다른 스레드에서 비동기적으로 계산을 수행
      futurePrice.complete(price);
			// 오랜 시간이 걸리는 계산 완료되면 Future에 값 설정
  }).start();
  return futurePrice;
	// 계산 결과 완료되길 기다리지 않고 Future반환
}
```

비동기 계산과 완료결과 포함하는 `CompletableFuture`인스턴스 만듬

실제 가격 계산할 다른 스레드 만든 다음, 오래 걸리는 계산 결과를 기다리지 않고 결과를 포함할 `Future` 인스턴스를 바로 반환. `complete`메서드 이용해 `CompletableFuture`종료

<br>

**에러 처리 방법**

만약, 가격을 계산하는 동안 에러가 발생한다면?

예외가 발생하면 해당 스레드에만 영향을 미침 → 가격 계산은 계속 진행되며 일의 순서 꼬임 → 클라이언트는 get메서드 반환때까지 영원히 기다리게 될 수도 있음

(해결방안)

클라이언트는 타임아웃값 받는 get메서드의 오버로드 버전 만들어 이 문제 해결(블록 문제가 발생할 수 있는 상황에서는 타임아웃 활용하는게 좋다). 문제가 발생했을 때 클라이언트가 영원히 블록되지 않고 타임아웃시간 되면 예외를 받을수 있도록!

But 이때 왜 에러가 발생했는지 알수가 없다. → `completeExceptionally`메서드 이용해 `CompletableFuture`내부에서 발생한 예외를 클라이언트로 전달해야 한다

```java
public Future<Double> getPrice(String product) {
    CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    new Thread(() -> {
      try {
        double price = calculatePrice(product);
        futurePrice.complete(price);
				// 계산이 정상적으로 종료되면 Future에 가격 정보를 저장한 채로 Future를 종료한다
      } catch (Exception ex) {
        futurePrice.completeExceptionally(ex);
				// 도중에 문제가 발생하면 발생한 에러를 포함시켜 Future종료
      }
    }).start();
    return futurePrice;
}
```
<br>

**팩토리 메서드 `supplyAsync`로 `CompletableFuture`만들기**

```java
public Future<Double> getPrice(String product) {
	return CompletableFuture.supplyAsync(() -> calculatePrice(product));
}
```

`supplyAsync`메서드 `Supplier`를 인수로 받아 `CompletableFuture`반환

`CompletableFuture`는 `Supplier`를 실행해 비동기적 결과 생성. `ForkJoinPool`의 `Executor`중 하나가 `Supplier`실행

but 두 번째 인수 받는 오버로드 버전 `supplyAsync`메서드 이용해 다른 `Executor`지정. 

결국 모든 다른 `CompletableFuture`의 팩토리 메서드에 `Executor`선택적 전달

<br>

### 비블록 코드 만들기

```java
public List<String> findPrices(String product) {
    return shops.stream()
        .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product))
        .collect(toList());
  }
```

각각 1초의 대기시간으로 4초 보다 더 걸림

<br>

**병렬 스트림으로 요청 병렬화**

```java
public List<String> findPricesParallel(String product) {
    return shops.parallelStream()
        .map(shop -> shop.getName() + " price is " + shop.getPrice(product))
        .collect(Collectors.toList());
  }
```

1초 남짓 시간으로 완료

<br>

<b>`CompletableFuture`로 비동기 호출 구현하기</b>

팩토리 메서드 `supplyAsync`로 `CompletableFuture`만들기

```java
public List<String> findPricesFuture(String product) {
  List<CompletableFuture<String>> priceFutures =
      shops.stream()
          .map(shop -> CompletableFuture.supplyAsync(() -> shop.getName() + " price is "
					// 각각의 가격 비동기적 계산
              + shop.getPrice(product), executor))
          .collect(Collectors.toList());

	List<String> prices = priceFutures.stream()
        .map(CompletableFuture::join)
				// 모든 비동기 동작이 끝나길 기다림
        .collect(Collectors.toList());
```

`CompletableFuture`포함 리스트 얻을수 있음. 동작 완료후 결과 추출한 후 리스트로 반환해야 한다

두 map연산을 하나의 스트림 처리 파이프라인으로 처리하지 않고 두 개의 스트림 파이프라인처리 한것이 다름

스트림 연산은 게을러서 하나의 파이프 라인으로 했다면 동기적, 순차적으로 이루어질것

`findPrice`로 하면 블록에 비해서는 빠르지면 병렬에 비해서는 느리다.

<br>

**확장성 좋은 해결 방법**

병렬 스트림 - 정확히 네 개의 상점에 하나의 스레드 할당해서 네 개의 작업을 병렬로 수행하며 검색 시간 최소화.

만약, 5번째 상점 추가되면 시간은 1초 더 늘어날 거다. 이때는 `CompletableFuture`를 이용하는게 더 빠르다(다양한 `Executor`를 지정할 수 있다는 장점이 있다)

<br>

**커스텀 Executor 사용하기**

애플리케이션이 실제 필요한 작업량 고려한 풀에서 관리하는 스레드 수에 맞게 Executor를 만들기

<br>

---

**스레드 풀 크기 조절**

스레드 풀이 너무 크면 CPU와 메모리 자원을 서로 경쟁하느라 낭비, 스레드 풀이 너무 작으면 CPU의 일부 코어는 활용되지 않을 수 있음

CPU활용 비율

N(threads) = N(cpu) * U(cpu) * (1 + W / C )

- N(cpu)는 Runtime, getRuntime().availableProcessors()가 반환하는 코어 수
- U(cpu)는 0과 1 사이의 값을 갖는 CPU활용 비율
- W / C는 대기시간과 계산시간의 비율

---
<br>
<br>

```java
private final Executor executor = Executors.newFixedThreadPool(shops.size(), (Runnable r) -> {
// 상점 수만큼의 스레드 갖는 풀 생성
    Thread t = new Thread(r);
    t.setDaemon(true);
		// 프로그램 종료 방해않는 데몬 스레드 사용
    return t;
  });
```

만드는 풀은 **데몬 스레드**를 포함. 자바에서 일반 스레드 실행 중일시 자바 프로그램은 종료되지 않음

→ 어떤 이벤트 한없이 기다리고 있으면 문제 생길 수 있다. 반면 데몬 스레드는 자바 프로그램 종료될 때 강제로 실행이 종료될 수 있다. (두 스레드의 성능은 같다)

<br>

<b>제품가격 얻는 `CompletableFuture`</b>

```java
public List<String> findPricesSequential(String product) {
    return shops.stream()
        .map(shop -> shop.getName() + " price is " + shop.getPrice(product))
        .collect(Collectors.toList());
  }
```

결국 애플리케이션의 특성에 맞는 Executor를 만들어 `CompletableFuture`를 활용하는 것이 바람직

(비동기 동작을 많이 사용하는 상황에서는 지금 살펴본 기법이 가장 효과적일 수 있다)

<br>

---

**스트림 병렬화와 CompletableFuture 병렬화**

컬렉션 계산 병렬화 하는 두 가지 방법

1. 병렬 스트림으로 변환해서 컬렉션 처리하는 방법
2. 컬렉션 반복하며 `CompletableFuture` 내부의 연산으로 만드는 것

`CompletableFuture`를 이용하면 전체적인 계산 블록되지 않게 스레드 풀의 크기 조절할 수 있음

참고해서 어떠한 병렬화 기법 사용할지 선택하기

- I / O 가 포함되지 않은 계산 중심 동작 실행할 때는 → 스트림 인터페이스가 가장 구현하기 간단하며, 효율적일 수 있다(모든 스레드가 계산 작업을 수행하는 상황에서는 프로세서 코어수 이상의 스레드를 가질 필요가 없다)
- I / O 를 기다리는 작업을 병렬로 실행할 땐 `CompletableFuture`가 더 많은 유연성 제공하며 대기/계산(W/C)의 비율에 적합한 스레드 수를 설정
    
    특히, 스트림의 게으른 특성 때문에 스트림에서 I / O를 실제로 언제 처리할지 예측하기 어려운 문제도 있음
    

---

<br>

여기까지가 Future내부 수행하는 일회성 작업 이었음

얖으로 → 스트림 API와 같이 선언형으로 여러 비동기 연산 `CompletableFuture`로 파이프라인화 하기

**할인 서비스 구현**

여러 상점에서 가격 정보 알아옴, 할인 서버에서 할인율 확인해서 최종 가격 계산

```java
public class Quote {

  private final String shopName;
  private final double price;
  private final Discount.Code discountCode;

  public Quote(String shopName, double price, Discount.Code discountCode) {
    this.shopName = shopName;
    this.price = price;
    this.discountCode = discountCode;
  }

  public static Quote parse(String s) {
    String[] split = s.split(":");
    String shopName = split[0];
    double price = Double.parseDouble(split[1]);
    Discount.Code discountCode = Discount.Code.valueOf(split[2]);
    return new Quote(shopName, price, discountCode);
  }

  public String getShopName() {
    return shopName;
  }

  public double getPrice() {
    return price;
  }

  public Discount.Code getDiscountCode() {
    return discountCode;
  }

}
```

상점에서 얻은 문자열 → 정적 팩토리 메서드 parse로 넘김. 

상점 이름, 할인전 가격, 할인된 가격 정보 포함 Quoto 클래스 인스턴스 생성

Quoto객체 인수로 받아 할인된 가격 문자열 반환

<br>

**할인 서비스 사용**

```java
public List<String> findPricesSequential(String product) {
    return shops.stream()
        .map(shop -> shop.getPrice(product))
				// 각 상점에서 할인 전 가격 얻기
        .map(Quote::parse)
				// 상점에서 반환한 문자열을 Quote 객체로 변환
        .map(Discount::applyDiscount)
				// Discount 서비스 이용해 각 Quote에 할인 적용
        .collect(Collectors.toList());
  }
```

이 구현은 성능 최적화와는 거리가 멈. 병렬로 하면 성능은 쉽게 개선되지만 스트림이 사용하는 스레드 풀의 크기가 고정되어 있어서 상점 수가 늘어났을 때처럼 유연하게 대응 불가

`CompletableFuture`에서 수행하는 태스크를 설정할 수 있는 커스텀 `Executor`를 정의해 CPU사용 극대화함

<br>

**동기 작업과 비동기 작업 조합**

`CompletableFuture`에서 제공하는 기능으로 `findPrices`메서드를 비동기적으로 구현

```java
public List<String> findPricesFuture(String product) {
    List<CompletableFuture<String>> priceFutures = findPricesStream(product)
        .collect(Collectors.<CompletableFuture<String>>toList());

    return priceFutures.stream()
        .map(CompletableFuture::join)
        .collect(Collectors.toList());
  }
```
<br>

동기 작업과 비동기 작업 조합
![Untitled](./image/%EB%8F%99%EA%B8%B0%20%EC%9E%91%EC%97%85%EA%B3%BC%20%EB%B9%84%EB%8F%99%EA%B8%B0%20%EC%9E%91%EC%97%85%20%EC%A1%B0%ED%95%A9.png)

<br>

**가격 정보 얻기**

첫 번째 연산. 팩토리 메서드 `supplyAsync`에 람다 표현식 전달해 비동기적으로 상점에서 정보 조회

`CompletableFuture`는 작업 끝났을 때 해당 상점에서 반환하는 문자열 정보 포함

<br>

Quote 파싱하기

첫 번째 결과 문자열 Quote로 변환. 파싱 동작에선 원격 서비스나 I / O 가 없으니 원하는 즉시 지연 없이 동작 수행

첫 번째 과정에서 생성된 `CompletableFuture`에 `thenApply` 메서드 호출후 문자열을 `Quote` 인스턴스로 변환하는 `Function`으로 전달

`thenApply` 메서드는 `CompletableFuture`가 끝날 때까지 블록하지 않는다. `CompletableFuture`가 동작 완전히 완료한 다음 `thenApply` 메서드로 전달된 람다 표현식 적용가능

<br>
<br>

**CompletableFuture를 조합해서 할인된 가격 계산**

람다표현식으로 이 동작 팩토리 메서드 `supplyAsync`에 전달. 그러면 다른 `CompletableFuture`가 반환. 두가지 `CompletableFuture`로 이루어진 두 개의 비동기 동작 만들 수 있음

- 상점에서 가격 정보 얻어와서 Quote로 변환
- 변환도니 Quote를 Discount 서비스로 전달해 할인된 최종가격 획득

자바 8의 `CompletableFuture API`는 이와 같은 두 비동기 연산을 파이브라인 만들 수 있도록 `thenCompose` 메서드 제공. `thenCompose`메서드는 첫 번째 연산의 결과를 두 번째 연산으로 전달

ex) `CompletableFuture`에 `thenCompose` 메서드 호출, `Function`에 넘겨주는 식. 두 `CompletableFuture`조합

<br>

세 개의 map 연산 결과 수집하면 리스트 형식 자료를 얻는다.  join으로 값을 추출한다

Async로 끝나는 버전이 존재하며, 끝나지 않는 메서드는 이전 작업 수행한 스레드와 같은 스레드에서 작업을 실행함을 의미한다. Async로 끝나는 메서드는 다음 작업이 다른 스레드에서 실행되도록 스레드 풀로 작업을 제출

⇒ 최종 결과나 개괄적인 실행시간에는 영향을 미치 않으니, 스레드 전환 오버헤드가 적개 발생하며 효율성이 좋은 thenCompose를 사용했다

<br>
<br>

**독립 `CompletableFuture`와 비독립 `CompletableFuture`합치기**

`thenCombine` 메서드 사용. 여기서도 Async 버전이 존재한다.

`thenCombineAsync` 메서드에서는 `BiFunction`이 정의하는 조합 동작이스레드 풀로 제출되면서 별도의 태스크에서 비동기적으로 수행

ex) 온라인 상점이 유로 가격 정보 제공(고객에게 항상 달러 가격 보여줌)

유로와 달러의 현재 환율 비동기적으로 요청 → 두 데이터 얻고 가격에 환율 곱해서 결과 합침

 

```java
CompletableFuture<Double> futurePriceInUSD =
          CompletableFuture.supplyAsync(() -> shop.getPrice(product))
// 제품가격 정보 요청하는 첫 번째 태스크 생성
          .thenCombine(
              CompletableFuture.supplyAsync(
                  () ->  ExchangeService.getRate(Money.EUR, Money.USD))
		// USD, EUR의 환율 정보 요청하는 독립적인 두 번째 태스크 생
              // 자바 9에 추가된 타임아웃 관리 기능
              .completeOnTimeout(ExchangeService.DEFAULT_RATE, 1, TimeUnit.SECONDS),
              (price, rate) -> price * rate
          )
          // 자바 9에 추가된 타임아웃 관리 기능
          .orTimeout(3, TimeUnit.SECONDS);
      priceFutures.add(futurePriceInUSD);
```

<br>

독립적인 두 개의 비동기 태스크 합치기

![Untitled](./image/%EB%8F%85%EB%A6%BD%EC%A0%81%EC%9D%B8%20%EB%91%90%20%EA%B0%9C%EC%9D%98%20%EB%B9%84%EB%8F%99%EA%B8%B0%20%ED%83%9C%EC%8A%A4%ED%81%AC%20%ED%95%A9%EC%B9%98%EA%B8%B0.png)

<br>

**Future의 리플렉션과 CompletableFuture의 리플렉션**

```java
ExecutorService executor = Executors.newCachedThreadPool();
// 태스크 스레드 풀에 제출할 수 있도록 ExecutorService를 생성
final Future<Double> futureRate = executor.submit(new Callable<Double>() {
        @Override
        public Double call() {
          return ExchangeService.getRate(Money.EUR, Money.USD);
			// EUR, USD 환율 정보 가져올 Future를 생성
        }
      });
      Future<Double> futurePriceInUSD = executor.submit(new Callable<Double>() {
        @Override
        public Double call() {
          try {
            double priceInEUR = shop.getPrice(product);
				// 두 번째 Future로 상점에서 요청 제품의 가격 검색
            return priceInEUR * futureRate.get();
				// 가격 검색한 Future를 이용해서 가격과 환율 곱함
          }
          catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e.getMessage(), e);
          }
        }
      });
```

<br>

**타임아웃 효과적으로 사용하기**

```java
CompletableFuture<Double> futurePriceInUSD =
          CompletableFuture.supplyAsync(() -> shop.getPrice(product))
          .thenCombine(
              CompletableFuture.supplyAsync(
                  () ->  ExchangeService.getRate(Money.EUR, Money.USD))
              // 자바 9에 추가된 타임아웃 관리 기능
              .completeOnTimeout(ExchangeService.DEFAULT_RATE, 1, TimeUnit.SECONDS),
							// 환전 서비스가 일 초 안에 결과 제공하지 않으면 기본 환율값 사용
              (price, rate) -> price * rate
          )
          // 자바 9에 추가된 타임아웃 관리 기능
          .orTimeout(3, TimeUnit.SECONDS);
				// 3초뒤 작업 완료되지 않으면 Future가 예외 발생시키도록 설정, 자바에서는 비동기 타임아웃 관리 기능 추가됨
```

블록은 하지 않는게 좋음

`orTimeout` 메서드는 지정된 시간 후에 `CompletableFuture`를 예외로 완료하며 또 다른 `CompletableFuture`를 반환하도록 한다

<br>

### `CompletableFuture`의 종료에 대응하는 방법

임의 지연 메서드 만들고 바로 정보 보여줄수 있게 하기

```java
private final Random random;

  public AsyncShop(String name) {
    this.name = name;
    random = new Random(name.charAt(0) * name.charAt(1) * name.charAt(2));
  }

private double calculatePrice(String product) {
    delay();
    if (true) {
      throw new RuntimeException("product not available");
    }
    return format(random.nextDouble() * product.charAt(0) + product.charAt(1));
  }
}
```

**최저가격 검색 애플리케이션 리팩터링**

<br>

---
<br>

### 정리

- 한 개 이상의 원격 외부 서비스를 사용하는 긴 동작을 실행할 때는 비동기 방식으로 애플리케이션 성능과 반응성 향상
- 고객에게 비동기 API를 제공하는 것을 고려해야 함. `CompletableFuture`의 기능을 이용하면 쉽게 비동기 API구현가능
- `CompletableFuture`를 이용할 때 비동기 태스크에서 발생한 에러 관리하고 전달
- 동기 API를 `CompletableFuture`로 감싸서 비동기적으로 소비
- 서로 독립적인 비동기 동작이든 아니면 하나의 비동기 동작이 다른 비동기 동작의 결과에 의존하는 상황이든 여러 비동기 동작을 조립하고 조합할 수 있다
- `CompletableFuture`에 콜백을 등록해서 Future가 동작을 끝내고 결과를 생산했을 때 어떤 코드를 실행하도록 지정가능
- `CompletableFuture`리스트의 모든 값이 완료될 때까지 기다릴지, 첫 값만 완료되길 기다릴지 선택가능
- 자바 9에서는 `orTimeout`, `completeOnTimeout`메서드로 `CompletableFuture`에 비동기 타임아웃 기능 추가