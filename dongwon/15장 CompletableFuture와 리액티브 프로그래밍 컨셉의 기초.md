# CompletableFuture와 리액티브 프로그래밍 컨셉의 기초

### 소프트웨어 개발 방법의 두 가지 추세

- 애플리케이션을 실행하는 하드웨어
    
    멀티코어의 프로세서 발전 → 멀티코어 프로세서를 얼마나 잘 활용하여 소프트웨어를 개발하는가? 에 따른 애플리케이션의 속도 증가
    
    ex )
    
    1. 한 개의 큰 태스크를 병렬로 실행할 수 있는 개별 하위 태스크로 분리할 수 있게 됨
    2. 2. 자바 7부터 포크 / 조인 프레임워크 사용가능
    3. 자바 8에 병렬 스트림 추가되어 병렬 스트림으로 스레드에 비해 단순하고 효과적인 병렬 실행 달성

- 인터넷 서비스에서의 애플리케이션 사용 증가
    
    하나의 거대 애플리케이션이 작은 서비스 애플리케이션으로 세분화 됨(서비스가 작아진 대신, 네트워크 통신이 증가함, 공개 API를 통한 더 많은 인터넷 접함)
    
<br>

**병렬성**

한 태스크르르 여러 하위 태스크로 나눠서  CPU의 다른 코어 또는 다른 머신에서 이들 하위 태스크를 병렬로 실행

<br>

**동시성**

조금씩 연관된 작업을 같은 CPU에서 동작 하는 것, 애플리케이션을 생산성을 극대화할 수 있도록 코어를 바쁘게 유지하는 것(원격 서비스나 데이터베이스 결과를 기다리는 스레드 볼록함으로 연산 자원 낭비하는 일은 피해야 한다)

동시성 대 병렬성
![Screen Shot 2023-07-04 at 2.12.26 AM.png](./image/%EB%8F%99%EC%8B%9C%EC%84%B1%20%EB%8C%80%20%EB%B3%91%EB%A0%AC%EC%84%B1.png)

동시성은 단일 코어 머신에서 발생할 수 있는 프로그래밍 속성으로 실행이 서로 겹칠 수 있는 반면 

병렬성은 병렬 실행을 하드웨어 수준에서 지원한다

> 자바는 이런 환경에서 2가지 주요 도구를 제공한다
Future 인터페이스, 플로 API
> 

<br>

### 동시성을 구현하는 자바 지원의 진화

처음 자바는 Runnable과 Thread를 동기화된 클래스와 메서드를 이용해 잠갔다(한 번에 하나의 스레드만 동기화된 블록이나 메서드에 액세스 하게함)

```java
public class SynchronizedExample {
    private static int counter = 0;
    private static final int THREAD_COUNT = 10;

    public static void main(String[] args) {
        // 동기화된 클래스의 공유 인스턴스 생성
        SynchronizedClass synchronizedObject = new SynchronizedClass();

        // 여러 개의 스레드 생성 및 시작
        Thread[] threads = new Thread[THREAD_COUNT];
        for (int i = 0; i < THREAD_COUNT; i++) {
            threads[i] = new Thread(new SynchronizedRunnable(synchronizedObject));
            threads[i].start();
        }

        // 모든 스레드가 완료될 때까지 대기
        for (int i = 0; i < THREAD_COUNT; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        // 최종 카운터 값 출력
        System.out.println("최종 카운터 값: " + counter);
    }

    // 동기화된 클래스
    private static class SynchronizedClass {
        // 동기화된 메서드
        public synchronized void incrementCounter() {
        // incrementCounter() 메서드 자체가 동기화된 메서드
						counter++;
        }
    }

    // Runnable 구현체
    private static class SynchronizedRunnable implements Runnable {
        private final SynchronizedClass synchronizedObject;

        public SynchronizedRunnable(SynchronizedClass synchronizedObject) {
            this.synchronizedObject = synchronizedObject;
        }

        @Override
        public void run() {
            // 동기화된 블록
            synchronized (synchronizedObject) {
                // 동기화된 객체에서 작업 수행
                synchronizedObject.incrementCounter();
								// synchronizedObject 인스턴스 잠금, 한번에 하나의 스레드만 이 블록에 들어감
            }
        }
    }
}
```

<br>

자바 5에서 추가 된것들

좀 더 표현력있는 동시성 지원하는 특히 스레드 실행과 태스크 제출을 분리하는 `ExecutorService` 인터페이스, 높은 수준의 결과 즉, `Runnable`, `Thread`의 변형을 반환하는 `Callable<T>` and `Future<T>`, 제네릭 코드를 지원

- `ExecutorService` : 스레드 실행을 작업 제출로부터 분리하여 처리, 스레드 풀을 제공하여 스레드 생성, 실행 및 종료와 같은 저수준의 스레드 관리 세부사항을 추상화
- `Callable<T>` : Runnable과 비슷하지만 T 타입의 결과를 반환하고 체크드 예외를 던질 수 있는 작업을 표현합니다. Callable로 작성된 비동기 작업을 나타냄
- `Future<T>` : Callable에 의해 수행되는 계산의 결과를 나타냅니다. 작업이 완료되었는지 확인하고 결과를 검색하며, 작업 수행 중에 발생한 예외를 처리하는 메서드를 제공합니다. Future<T>는 연산 취소도 지원
- 제네릭 코드 : 제네릭 코드를 사용하여 매개변수화된 타입을 사용할 수 있게 되었으며. 이를 통해 다양한 데이터 유형에 대해 재사용 가능하고 타입 안정성이 보장되는 코드를 작성할 수 있다

<br>

이후 멀티코어 CPU에서 쉽게 병렬 프로그래밍 구현할 수 있게 됨

+ 개선된 동시성 지원

+ 자바 7 `RecursiveTask`추가

+ 자바 8 스트림과 새로 추가된 람다 지원에 기반한 병렬 프로세싱 추가

<br>

Future를 조합하는 기능 추가하며 동시성 강화 함

+ 자바 9 분산 비동기 프로그래밍 명시적 지원 함

매쉬업 애플리케이션, 다양한 웹 서비스를 이용하고 정보를 실시간으로 조합해 사용자에게 제공이나 추가 웹 서비스 통해 제공하는 종류의 애플리케이션 개발에 필수적 기초 모델, 툴킷 제공. → 이 과정을 리액티프 프로그래밍이라 한다(자바 9. 발행-구독 프로토콜로 지원함, **Flow의 궁극적 목표는 동시 실행가능한 독립적 테스크 가능하게 하며 병렬성 쉽게 이용하는 것**) 

<br>

**스레드의 높은 수준 추상화**

단일 CPU도 여러 사용자 지원가능. 운영체제가 각 사용자에 프로세스 하나 할당하기에 가능

두 사용자가 각각 자신만의 공간에 있다고 생각되도록 가상 주소 공간을 각각의 프로세스에 제공

프로세스는 본인이 가진 프로세스와 같은 주소 공간 공유하는 프로세스 요청하며 태스크 동시, 협력적으로 실행

멀티코어 설정에서는(노트북) 스레드의 도움 없이 프로그램이 노트북의 컴퓨팅 파워 모두 활용할 수 없음

실제 네 개 코어로 병렬 실행하면 속도 네 배 향상 가능(이론적으로, 실제론 오버헤드 날거다)

결론적으로 병렬 스트림 반복은 명시적 스레드 사용에 비해 높은 수준 개념이다. 다시 말해 스트림을 이용하면 추**추상화**할 수 있다

<br>

**추상화 기반 개념의 `ExecutorService`개념과 스레드 풀**

자바 5는 Executor 프레임워크와 스레드 풀 통해서 스레드의 힘을 높은 수준 끌어올리는 자바 프로그래머가 태스크 제출과 실행을 분리할 수 있는 기능 제공

<br>

**스레드의 문제**

자바 스레드는 직접 운영체제 스레드에 접근함. 운영체제 스레드는 비싸고 숫자가 제한되어 있음. 운영체제 지원 스레드 수 초과시 자바 애플리케이션이 예상치 못한 방식으로 크래시될 수 있음(기존 스레드 실행되며 새로운 스레드 만드는 상황 일어나선 안될것)

보통 운영체제 + 자바 스레드 개수 > 하드웨어 스레드 개수보다 많아 일부 운영체제 블록되거나 자고있을때 모든 하드웨어 스레드 코드 할당 될수 있음.

주어진 프로그램에서 사용할 최적의 자바 스레드 개수는 사용할 수 있는 하드웨어 코어 갯수에 따라 달라짐

<br>

**스레드 풀이 더 좋은 이유**

자바의 `ExecutorService` 태스크 제출하고 나중에 결과 얻는 인터페이스 제공

```java
**ExecutorService newFixedThreadPool(int nThreads)
// newFixedThreadPool 팩토리 메서드중 하나임
nThreads를 포함하는 ExecutorService만들어 스레드 풀에 저장**
```

스레드 풀에서 사용하지 않은 스레드로 제출된 태스크를, 먼저 온 순서대로 실행. 태스크 실행 종료시 스레드 풀로 반환

장점 : 하드웨어에 맞는 수의 태스크 유지하며 동시에 수 천개 태스크 받아도 오버헤드 없이 제출 가능, 다양한 설정가능(큐의 크기, 거부 정책, 태스크 종류에 따른 우선순위)

> 프로그래머는 태스크(Runnable이나 Callable)를 제공하면 스레드가 이를 실행한다
> 

<br>

**스레드 풀이 나쁜 이유**

- 주의 사항
    - k 스레드 가진 스레드 풀은 오직 k 만큼만 스레드 동시 실행. 초과 제출 태스크는 큐에 저장, 이전 태스크 중 하나 종료 전까지 스레드에 할당 안함(보통은 문제없음)
        
        ex) 잠을 잠, I / O 를 기다림, 네트워크 연결 기다림 → 기다리면서 아무 작업도 하지 않게 된다
        
        5스레드 중 3개가 잠을 자면 2스레드로 실행시켜서 작업 효율성이 떨어짐
        
        핵심은 블록(자거나 이벤트를 기다리는 상황)할 수 있는 태스크는 스레드 풀에 제출하지 말아야 한다
        
    - 자바 프로그램은 중요한 코드 실행하는 스레드 죽지 않게 main이 반환 전에 모든 스레드 작업 끝나길 기다린다. → 그러니 프로그램 종료전 모든 스레드 풀 종료 습관 가져야 함
        
        이러한 상황 다루는 자바의 Thread.setDaemon 메서드 제공

        자는 태스크 스레드 풀의 성능 저하
        ![Screen Shot 2023-07-04 at 4.08.50 AM.png](./image/%EC%9E%90%EB%8A%94%20%ED%83%9C%EC%8A%A4%ED%81%AC%20%EC%8A%A4%EB%A0%88%EB%93%9C%20%ED%92%80%EC%9D%98%20%EC%84%B1%EB%8A%A5%20%EC%A0%80%ED%95%98.png)
        
<br>

**스레드의 다른 추상화 : 중첩되지 않은 메서드 호출**

7장에서의(병렬 스트림 처리와 포크/조인 프레임워크) 동시성과 현재 동시성 차이

- 7장 (**엄격한 포크 / 조인**)
    
    한 개의 특별한 속성, 즉 태스크나 스레드가 메서드 호출 안에서 시작되면 그 메서드 호출은 반환하지 않고 작업이 끝나길 기다림 - 스레드 생성과 join()이 한 쌍처럼, 중첩된 메서드 호출 내에 추가됨
    
<br>

**엄격한 포크_조인.화살표는 스레드, 원은 포크와 조인, 사각형은 메서드 호출과 반환**
![Screen Shot 2023-07-04 at 4.19.18 AM.png](./image/%EC%97%84%EA%B2%A9%ED%95%9C%20%ED%8F%AC%ED%81%AC_%EC%A1%B0%EC%9D%B8.%ED%99%94%EC%82%B4%ED%91%9C%EB%8A%94%20%EC%8A%A4%EB%A0%88%EB%93%9C%2C%20%EC%9B%90%EC%9D%80%20%ED%8F%AC%ED%81%AC%EC%99%80%20%EC%A1%B0%EC%9D%B8%2C%20%EC%82%AC%EA%B0%81%ED%98%95%EC%9D%80%20%EB%A9%94%EC%84%9C%EB%93%9C%20%ED%98%B8%EC%B6%9C%EA%B3%BC%20%EB%B0%98%ED%99%98.png)

<br>

**여유로운 포크 / 조인**

![Screen Shot 2023-07-04 at 4.22.29 AM.png](./image/%EC%97%AC%EC%9C%A0%EB%A1%9C%EC%9A%B4%20%ED%8F%AC%ED%81%AC%20%20%EC%A1%B0%EC%9D%B8.png)

시작된 태스크를 내부 호출이 아닌 외부 호출로 종료하게 함
제공된 인터페이스를 사용자는 일반 호출로 간주할 수 있음

<br>

**비동기 메서드**

![Screen Shot 2023-07-04 at 4.24.46 AM.png](./image/%EB%B9%84%EB%8F%99%EA%B8%B0%20%EB%A9%94%EC%84%9C%EB%93%9C.png)

메서드 호출자에 기능 제공하도록 메서드 반환후, 만들어진 태스크 실행이 계속됨

<br>

**메서드 사용의 위험성**
    - 스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행됨 - 데이터 경쟁 문제 일으키지 않게 주의해야 함

<br>

(문제상황) 기존 실행 중이던 스레드 종료되지 않은 상황에서 자바의 `main()`메서드 반환한다면?

1. (결과) 애플리케이션 종료하지 못하고 모든 스레드 실행을 끝낼 때까지 기다림

    ⇒ (문제 발생) 종료 못한 스레드에 의해 애플리케이션 크래시 될 수 있음, 디스크에 쓰기 I / O 작업 중단했을 때, 외부 데이터의 일관성이 파괴될 수 있음

    (해결 방법) 애플리케이션에서 만든 모든 스레드를 추적, 애플리케이션 종료전 스레드 풀을 포함한 모든 스레드 종료해야 할 것

<br>

2. (결과) 애플리케이션 종료 방해하는 스레드 강제종료 시키고 애플리케이션 종료함

    자바 스레드는 setDaemon() 메서드 이용해 **데몬**, 비데몬으로 구분 가능

    데몬 스레드 : 애플리케이션 종료될 때 강제 종료되어, 디시크의 데이터 일관성 파괴하지 않는 동작 수행때 유용

    `main()`메서드는 모든 비데몬 스레드 종료 때까지 프로그램 종료 않고 기다림

<br>

**스레드의 목표**

모든 하드웨어 스레드 활용해 병렬성의 장점 극대화한 프로그램 구조 짜기 → 프로그램 작은 태스크 단위로 구조화(but 태스크 변환 비용 고려해 너무 작은 크기는 아니어야 함)

7장 - 병렬 스트림 처리와 (포크 / 조인) 을 for 루프와 분할, 정복 알고리즘을 처리하는 방법

16, 17장 - 스레드 조작하는 복잡한 코드 구현 않고 메서드 호출 방법

<br>

### 동기 API와 비동기 API

자바 8 스트림을 이용해 명시적으로 병렬 하드웨어를 이용할 수 있다. 두 단계로 병렬성을 이용할 수 있는데

1. 외부 반복(명시적 for루프)을 내부 반복(스트림 메서드 사용)으로 바꿔야 한다. +스트림에 `parallel()`메서드 이용하니 자바 런타임 라이브러리가 복잡한 스레드 작업을 하지 않고 병렬로 요소가 처리되도록 할 수 있다(루프 실행될 때 추측에 의존하는 프로그래머와 달리, 런타임 시스템은 사용할 수 있는 스레드 정확하게 알고 있다는 것도 내부 반복의 장점)

<br>

메서드 호출 합계 반환

```cpp
int y = f(x);
int z = g(x);
System.out.println(y + z);
```

f, g가 서로 상호작용하지 않는다면 f와 g를 별도의 CPU코어로 실행하여 f와 g중 오래 걸리는 작업의 시간으로 합계를 구하는 시간을 단축할 수 있다. 의도는 좋지만 대신 코드가 복잡해진다

<br>

> 복잡한 코드는 스레드에서 결과를 가져오는 부분과 관련 있다. 오직 바깥 최종 결과 변수만 람다나 내부 클래스에서 사용할 수 있다는 제한은 있지만, 실제 문제는 다름아닌 명시적 스레드 조작 부분에 존재한다
> 

```java
class ThreadExample {

  public static void main(String[] args) throws InterruptedException {
    int x = 1337;
    Result result = new Result();

    Thread t1 = new Thread(() -> {
      result.left = f(x);
    });
    Thread t2 = new Thread(() -> {
      result.right = g(x);
    });
    t1.start();
    t2.start();
    t1.join();
    t2.join();
		// 별도 실행, 시간은 단축되나, 코드가 복잡해 진다
    System.out.println(result.left + result.right);
  }

  private static class Result {
    private int left;
    private int right;
  }
}
```
<br>

`Runnable` 대신 `Future API` 인터페이스 이용해 코드 단순화. 이미 `ExecutorService`로 스레드 풀 설정했다 가정하고 코드 구현

```java
public class ExecutorServiceExample {

  public static void main(String[] args) throws ExecutionException, InterruptedException {
    int x = 1337;

    ExecutorService executorService = Executors.newFixedThreadPool(2);
    Future<Integer> y = executorService.submit(() -> fo(x)); // submit 불필요 코드
    Future<Integer> z = executorService.submit(() -> go(x)); // submit 불필요 코드
    System.out.println(y.get() + z.get());

    executorService.shutdown();
  }
}
```

(문제) 명시적 submit 메서드 호출하는 불필요한 코드

(해결방법) 명시적 반복으로 병렬화 수행하던 코드를, 스트림 이용해 내부 반복으로 바꿈

<br>

**방법 1.  비동기 API**

자바의 Future : `CompletableFuture`로 조합할수 있게됨

<br>

**방법 2. Flow 인터페이스 이용**

발행 - 구독 프로토콜 기반

<br>

**Future 형식 API**

```java
int y = f(x);
int g = g(x);

**--->** 

Future<Integer> y = f(x);
Future<Integer> z = g(x);
System.out.println(y.get() + z.get());
```

메서드 f, g 호출 즉시 자신의 원래 바디 평가하는 태스크 포함 Future 반환.

연산에서는 두 Future가 완료되어 결과 합쳐지길 기다림.

<br>

**If. 프로그램이 더 커진다면?**

- 다른 상황에도 Future 형식 필요할 수 있어 API형식 통일해야 함
- 병렬 하드웨어로 프로그램 실행 속도 극대화 하려면 합리적 크기의 태스크로 나누는 것이 좋음

<br>

**리액티브 형식 API**

f, g의 시그니처 바꿔 콜백 형식 프로그래밍 이용

```java
void f(int x, IntConsumer dealWithResult);
```

f의 추가 인수에 콜백(람다)을 전달해 f바디에서 return문으로 결과 반환이 아닌, 결과 준비되면 이를 람다로 호출하는 태스크 만드는 것

> 콜백 용어를 filter, map,과 같은 메서드 인수로 넘겨지는 모든 람다, 메서드 참조 가리킬 때 사용하기도 하고, 메서드 반환된 다음 호출될 수 있는 람다나 메서드 참조 가리키는 용도로도 사용
> 

```java
f바디 실행하며 태스크만들고 즉시 반환하여, 코드형식이 다음과 같음
but 결과가 달라짐. 호출 합계 정확한 출력 아닌, 상황에 따라 먼저 계산된 결과 출력
public static void main(String[] args) {

    int x = 1337;
    Result result = new Result();

    f(x, (int y) -> {
      result.left = y;
      System.out.println((result.left + result.right));
    });

    g(x, (int z) -> {
      result.right = z;
      System.out.println((result.left + result.right));
    });
  }
```

(문제점)

락을 사용하지 않아 값 두번 출력될 수도, 피연산자가 `println` 호출전에 업데이트 될 수 있다

(해결방안)

- if-then-else 이용해 적절한 락 이용, 두 콜백 모두 호출 되었는지 확인
- 리액티브 형식 API는 한 결과가 아닌 일련의 이벤트에 반응하게 설계되었으므로 Future 이용하는 것이 더 적절
    
    (add) 리액티브 형식 프로그래밍은 메서드 f, g의 `dealWithResult` 콜백을 여러번 호출할 수 있다
    
    원래 f, g함수는 한번만 `return` 하게 되어 있다. (`Future`도 한 번만 완료되머 결과는 `get()`으로 얻을 수 있다. 
    

리액티브 형식 비동기 API는 일련의 값(나중에 스트림 연결), `Future`형식 API는 일회성 값 처리에 적합 

> API 사용은 명시적 스레드 처리 코드에 비해, 사용 코드 더 단순하게 높은 수준 구조 유지 할 수 있게 도와준다
> 

ex) 계산이 오래 걸리는 메서드(수 밀리초 이상), 네트워크, 사람의 입력 기다리는 메서드

**잠자기(블로킹 동작)느 해로운 것으로 간주**

상호작용을 위해 어떤 일이 제한되어 일어나느 상황의 애플리케이션 만들때 `sleep()` 메서드를 사용한다. but 스레드는 잠들어 있어도, 여전히 시스템 자원을 점유한다. 만약 스레드가 많아지고 대부분 잔다면 문제가 커진다(잠을 자는 태스크는 스레드 풀에서 다른 태스크 시작하지 못하게 막아서 자원소비한다)

잠자는 스레드분 아닌 블록 동작도 막는데.

블록 동작은 1) 다른 태스크가 어떤 동작을 완료하길 기다리는 동작(Future의 get()) 과
2) 외부 상호작용(네트워크, 데이터베이스에서 읽기 작업 기다림, 키보드 입력과 같은 상호작용 기다림)을 기다리는 동작으로 구분할 수 있다

⇒ 태스크에서 기다리는 일 만들지 않거나, 코드에 예외 일으켜서 처리할 수 있다

```java
**(A)**
work1();
Thread.sleep(10000); // 10초 동안 잠
work2();

----------------------------------------------------------------------------------------
**(B)**
public class ScheduledExecutorServiceExample {

  public static void main(String[] args) {
      ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(1);

      work1();
      scheduledExecutorService.schedule(ScheduledExecutorServiceExample::work2, 10, TimeUnit.SECONDS);
			// work1() 끝난 다음 10초 뒤 work2()를 개별 태스크로 스케줄 함

      scheduledExecutorService.shutdown();
  }

  public static void work1() {
    System.out.println("Hello from Work1!");
  }

  public static void work2() {
    System.out.println("Hello from Work2!");
  }

}
```

A는 코드 스레드 풀 큐에 추가후 차례되면 실행(코드 실행시 워커 스레드 점유`sleep`하고, 아무것도 안함)

B는 A자는 동안 다른 작업 실행될 수 있게 허용함(스레드 사용 필요없이, 메모리만 조금 더 사용)

> 태스크 만들 때, 태스크 블록보다 다음 작업 태스크 제출하고 현재 태스크 종료가 바람직함
> 

I / O 작업에서도 이 원칙 적용이 좋음. 읽기 작업 기다리는 것이 아닌, 블록하지 않는 ‘읽기 시작’ 메서드 호출하고 읽기 작업 끝나면 처리할 다음 태스크 요청하고 종료

**현실성 확인**

새로운 시스템을 설계할 때 시스템을 많은 작은 동시 실행되는 태스크로 설계해서 블록할 수 있는 모든 동작을 비동기 호출로 구현하면 병렬 하드웨어를 최대한 활용할 수 있음

but 현실적으론 ‘모든 것은 비동기’ 라는 설계원칙 어겨야 함

**비동기 API에서 예외는 어떻게 처리하는가?**

Future나 리액티브 형식의 비동기 API에서 호출된 메서드의 실제 바디는 별도의 스레드에서 호출되며 이때 발생하는 에러는 호출자의 실행 범위에 관계 없는 상황이 됨

→ 예상치 못한 상황시, 예외 발생시켜 다른 동작 실행하게 해야 함

(해결방안)

`Future`구현한 `CompletableFuture`에서는 런타임 `get()`메서드에 예외 처리할 수 있는 기능 제공, `exceptionally()`제공

> **Netty :** 네트워크 서버의 블록/ 비블록 API 일관적 제공하며. 비동기식 이벤트 기반 네트워크 애플리케이션 프레임워크. 네트워크 서버 및 클라이언트 구축을 위한 확장 가능하고 효율적인 고성능 솔루션을 제공
> 

리액티브 형식의 비동기 API는 return대신 기존 콜백 호출되서 예외 발생시, 추가 콜백 만들어 인터페이스 바꿔야 함

```java
여러 콜백 포함
void f(int x, Consumer<Integer> dealWithResult, Consumer<Throwable> dealWithException);

f 바디 다음 수행
dealWithException(e);
```

콜백이 여러 개 일시 따로 제공보다는 한 객체로 이 메서드 감싸는게 좋음

플로API → 여러 콜백 한 객체(네 개의 콜백을 각각 대표하는 네 메서드를 포함 Subscriber<T>) 로 감쌈

```java
void onComplete() // 값 다 소진, 에러 발생해 더 이상 처리할 데이터 없을 때
void onError(Throwable throwable) // 도중 에러 발생했을 때
void onNext(T item) // 값이 있을 때

-> 이전 f에 적용
void f(int x, Subscriber<Integer> s);
예외 발생시 s.onError(t); 로 알림
```

API의 일부로 볼때 API는 이벤트의 순서(**채널 프로토콜**)에는 전혀 개의치 않는다

### 박스와 채널 모델

**박스화 채널 모델 :** 동시성 모델을 설계하고 개념화하는데 필요한 그림 기법

![Untitled](./image/%EB%B0%95%EC%8A%A4%ED%99%94%20%EC%B1%84%EB%84%90%20%EB%AA%A8%EB%8D%B8.png)

<br>

```java
int t = p(x);
System.out.println( r(q1(t), q2(t)) );
```

q1, q2를 차례로 호출할 시, 하드웨어 병렬성의 활용과 거리가 멈

병렬성을 극대화 하기 위해서는 다섯 함수 p, q1, q2, r, s를 Future로 감싸야 한다

시스템에서 많은 작업 동시 실행이 아니면 잘 동작하지만, 시스템 커지고, 각각 많은 박스와 채널등장하며 내부적으로 자신만의 박스와 채널을 사용하면 문제가 달라짐

(문제 상황)

많은 태스크 `get()` 메서드 호출로 `Future`가 끝나기만 기다리는 상태

(문제 해결)

`CompletableFuture`와 콤비네이터 이용해 문제 해결

```java
Function<Integer, Integer> myfun = add1.andThen(dble);

콤비네이터
p.thenBoth(q1, q2).thenCombine(r)
```

박스와 채널 모델 이용해 생각과 코드를 구조화 가능. 

박스로(또는 프로그램의 콤비네이터) 원하는 연산 표현(계산은 나중에 이루어짐)하면 계산을 손으로 코딩한 결과보다 더 효율적. 

콤비네이터는 수학적 함수분 아니라 Future와 리액티브 스트림 데이터에도 적용가능

박스와 채널 모델은 병렬성을 직접 프로그래밍하는 관점을 → 콤비네이션 이용해 내부적으로 작업 처리하는 관점으로 바꿔줌. 

자바 8스트림은 자료구조를 반복해야 하는 코드를 내부적으로 작업처리하는 스트림 콤비네이터로 바꿔줌

### CompletableFuture와 콤비네이터 이용한 동시성

자바 8에서 `Future` 인터페이스의 구현인 `CompletableFuture`를 이용해 `Future`를 조합할 수 있는 기능 추가

일반적으로 `Future` 실행해 `get()`으로 얻을 수 있는 결과 `Callable`로 만들어짐

But `completable`은 실행할 코드 없이 `Future`를 만들 수 있게 허용하며 `Complete()`메서드 이용해 다른 스레드가 이를 완료할 수 있거나 `get()`으로 값을 얻을 수 있게 허용

```java
public class CFComplete {

  public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(10);
      int x = 1337;

      CompletableFuture<Integer> a = new CompletableFuture<>();
			// 실행코드 없이 Future 만들수 있음
      executorService.submit(() -> a.complete(f(x)));
			// complete 이용해 나중에 다른 스레드가 이를 완료할 수 있음
      int b = g(x);
      System.out.println(a.get() + b);

      executorService.shutdown();
  }
}

f(x)나 g(x)가 끝나지 않은 상황에서 get()을 기다려야 해서 프로세싱 자원 낭비할 수 있음
```

(문제) f(x), g(x)는 한 스레드가 다른 스레드 return 통해 종료될 때까지 기다려야함

(해결 방법) 

동작 조합이 필요하다 (ex. 람다)

```java
myStream.map(...).filter(...).sum()
```

<br>

`thenCombine`메서드

```java
CompletableFuture<V> thenCombine(CompletableFuture<U> other, BiFunction<T, U, V> fn)
두 개의 CompletableFuture값 받아 한 개의 새 값 만듬

public class CFCombine {

  public static void main(String[] args) throws ExecutionException, InterruptedException {
      ExecutorService executorService = Executors.newFixedThreadPool(10);
      int x = 1337;

      CompletableFuture<Integer> a = new CompletableFuture<>();
      CompletableFuture<Integer> b = new CompletableFuture<>();
      CompletableFuture<Integer> c = a.thenCombine(b, (y, z)-> y + z);
			// thenCombine 은 두 연산 끝났을 때 스레드 풀에서 실행된 연산 만듬. 
			// c는 다른 두 작업 끝나기 전까지 실행되지 않음!(블록 피함)
			// 두 번째 종료때 스레드 한 개가 아닌 두개가 활성 상태임
      executorService.submit(() -> a.complete(f(x)));
      executorService.submit(() -> b.complete(g(x)));

      System.out.println(c.get());
      executorService.shutdown();
  }
}
```

`CompletableFuture`와 콤비네이터 통해 `get()`에서 블록하지 않으며 병렬 실행 효율성 높이며 데드락 피하는 최상 해결책 구현가능

<br>

**발행 - 구독, 리액티브 프로그래밍**

Future와 `CompletableFuture`는 독립적 실행과 병렬성의 정식적 모델에 기반함. 연산이 끝나면 `get()`으로 `Future`의 결과 얻을 수 있음. 따라서 `Future`는 한 번만 실행해 결과 제공함

리액티브 프로그래밍은 시간 흐르며 여러 `Future`같은 객체 통해 여려 결과를 제공함

ex) 온도계 객체 - 매 초마다 온도 값 반복적으로 제공

웹 서버 컴포넌트 응답 기다리는 리스너 객체 - 네트워크에 HTTP 요청이 발생하길 기다린 후 결과 데이터 생산함 and 다른 코드에서 온도 값 또는 네트워크 결과 처리함

이후 다름 결과 처리할 수 있게 다음 신호 기다림

다른점은 `Future`를 둘다 사용하지만 결과의 횟수가 다르다, 온도계에서 계속 정보를 주지만 가장 최근의 온도만 중요하다. 이러한 종류의 프로그래밍을 **리액티브**라 한다

자바 9에서 Flow의 인터페이스 발행해 리액티브 프로그래밍 제공함

<br>

**플로 API**

- 구독자가 구독할 수 있는 **발행자**
- 이 연결을 **구독**이라함
- 이 연결을 이용해 **메시지**(**이벤트**)를 전송

발행자-구독자 모델

![Untitled](./image/%EB%B0%9C%ED%96%89%EC%9E%90_%EA%B5%AC%EB%8F%85%EC%9E%90%20%EB%AA%A8%EB%8D%B8.png)

여러 컴포넌트, 한 구독자 구입가능

한 컴포넌트, 여러 개별 스트림 발행가능,여러 구독자 가입 가능

---
<aside>
💡 데이터가 발행자(생산자)에서 구독자(소비자)로 흐름에 착안해 개발자는 이를 업스트림, 또는 다운스트림이라 부름. 데이터 `newValue`는 업스트림 `onNext()`메서로 전달되며 `notifyAllSubscribers()` 호출을 통해 다운스트림 `onNext()` 호출로 전달된다

</aside>

---
<br>

**리액티브 프로그래밍**

실생활 플로 사용을 위해서는 `onNext`외에 `onError`, `onComplete`와 같은 메서드 통해 데이터 흐름에서 예외 발생이나 데이터 흐름 종료됨을 알 수 있어야 함(ex. `onNext`로 더 이상 데이터 발생하지 않는다던가 등)

<br>

**플로 인터페이스**

압력, 역압력 기능

<br>

**압력** 

ex) 초마다 보고 하던게 업그레이드 되면서 → 밀리초마다 보고, 약간의 SMS메시지 → 매 초마다 수천개의 메시지

<br>

**역압력**

`Subscriber`객체를(`onNext`, `onError`, `onComplete`) `Publisher`에게 전달해 발행자가 필요한 메서드 호출할 수 있나 살펴봄

`Publisher`에서 → `Subscriber`로 정보 전달. 정보 흐름 속도를 **역압력**(흐름 제어)으로 반대로 정보 요청해야 할 필요 있을 수 있음. `Publisher`는 여러 `Subscriber`가지고 있어 역압력 요청이 한 연결에만 영향 미쳐야 한다는 것이 문제가 될 수 있음

```java
자바 9의 플로 API의 Subscriber 인터페이스는 네 번째 메서드를 포함함
void onSubscribe(Subscription subscription);
```

> 콜백을 통한 ‘역방향’ 소통 효과에 주목. `Publisher`는 `Subscriber`객체를 만들어 `Subscriber`로 전달하면 `Subscriber`는 이를 이용해 `Publisher`로 정보 보낼 수 있음
> 

<br>

**실제 역압력의 간단한 형태**

한 개의 이벤트 처리에 발행 - 구독 연결 구성

- `Subscriber`가 `OnSubscribe`로 전달된 `Subscriber`객체를 `subscription`같은 필드에 로컬로 저장
- `Subscriber`가 수많은 이벤트 받지 않게 마지막 동작에 `channel.request(1)` 추가해 한 이벤트만 요청함
- 요청 보낸 채널에만 `onNext`, `onError` 이벤트 보내도로 `Publisher`의 `notifyAllSubscribers`코드를 바꿈(보통 `Subscriber`가 자신만의 속도 유지를 위해 `Publisher`는 새 `Subscription`을 만들어 각 `Subscriber`와 연결함

<br>

역압력 구현할때 생각해야할 장단점

- 여러 `Subscriber`가있을 때 이벤트를 가장 느린 속도로 보낼 것인가?
`Subscriber`에게 보내지 않은 데이터를 저장할 별도의 큐를 가질 것인가?
- 큐가 너무 커지면 어떻게 할 것인가?
- `Subscriber`가 준비 안 되었으면 큐의 데이터를 폐기할 건가?

<br>

**리액티브 당김 기반**이란?

`Subscriber`가 `Publisher`로부터 요청을 당긴다는 의미에서 쓰이며. 이런 방식으로 역압력 구현할 수 있음

<br>

### 리액티브 시스템 vs 리액티브 프로그래밍

- 리액티브 시스템
    
    런타임 환경이 변환에 대응하도록 전체 아키텍처가 설계된 프로그램. 가져야할 공식적인 크게 3가지로 볼수 있다
    
    1. 반응성 : 큰 작업 처리하느라 간단한 질의 응답을 지연하지 않고 실시간으로 입력에 반응하는 것을 의미
    2. 회복성 : 한 컴포넌트의 실패로 전체 시스템이 실패하지 않음을 의미.
    
    ex) 네트워크가 고장나도 이와 관계 없는 질의에는 아무 영향이 없어야 하며, 반응 없는 컴포넌트를 향한 질의가 있을시 다른 대안 컴포넌트를 찾아야 한다.
    
    1. 탄력성 : 시스템이 자신의 작업 부하에 맞게 적응하며 작업을 효율적으로 처리함을 의미한다
    
    ex) 바에서 음식과 음료 서빙하는 직원을 동적으로 재배치하여, 두 가지 주문의 대기줄이 일정하게 유지되도록 하듯, 각 큐가 원할하게 처리 되도록 스레드를 적절히 재배치 한다
    
- 리액티브 프로그래밍
    
    Flow와 관련된 자바 인터페이스 제공하는 형식. Reactive Manifesto의 네번째 속성인 메시지 주도 속성을 반영한다. 이는 박스와 채널 모델에 기반한 내부 API를 가지고 있으며, 여기서 컴포넌트는 처리할 입력을 기다리며. 결과를 다른 컴포넌트로 보내며 시스템이 반응한다
    
<br>

---

<br>

### 정리

- 자바는 계속해서 동시성을 지원한다. 스레드 풀은 유용하지만 블록되는 태스크가 많아지면 문제가 발생한다
- 메서드를 비동기(결과를 처리전에 반환)로 만들면 병렬성 추가할 수 있으며 부수적으로 루프 최적화함
- 박스와 채널 모델 이용해 비동기 시스템 시각화
- 자바 8 `CompletableFuture` 클래스와 자바 9 플로 API 모두 박스와 채널 다이어그램으로 표현가능
- `CompletableFuture`클래스는 한번의 비동기 연산을 표현. 콤비네이터로 비동기 연산 조합해서 `Future`이용할 때 발생한 기존 블로킹 문제 해결함
- 플로 API는 발행 - 구독 프로토콜, 역압력 이요해 자바의 리액티브 프로그래밍의 기초 제공
- 리액티브 프로그래밍 이용해 → 리액티브 시스템 구현 가능