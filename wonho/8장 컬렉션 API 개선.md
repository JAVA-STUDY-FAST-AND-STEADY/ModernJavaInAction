# 컬렉션 API 개선

- [x] 1. 컬렉션 팩토리
- [x] 2. 리스트와 집합 처리
- [x] 3. 맵 처리
- [x] 4. 개선된 ConcurrentHashMap
- [x] 5. 마치며
- [x] 6. 배우고 느낀점

<br><br>

## 1. 컬렉션 팩토리

---

<br>

 거의 모든 자바 애플리케이션에서 컬렉션을 사용한다. 우리는 컬렉션과 스트림 API를 이용하여, 데이터 처리 쿼리를 어떻게 효율적으로 처리할지 고민했다. 하지만, 컬렉션 API에는 이것을 이용하기에는 에러를 유발하는 단점이 있다. 그래서 자바 8, 9에서는 좀더 편리하게 이용할 수 있게 해주는 새로운 컬렉션 API를 제공한다. 새롭게 추가된 작은 단위의 리스트, 집합, 맵을 쉽게 생성하는 **컬렉션 팩토리 메서드**와 요소를 삭제 수정 하는 **관용 패턴**을 적용하는 방법에 대해 소개한다.

 자바 9에서는 작은 단위의 컬렉션 객체를 쉽게 생성하는 방법을 제공한다. 먼저 이 기능이 왜 생겨났는지 확인해보자. 우리는 리스트를 만들고, 3개를 담아 이용하려고 한다. 이 경우 과거에는 다음과 같이 코드를 작성한다.

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
```

 단순하게 작성해도, 벌써 4개의 줄이나 사용되게 되었다. 하지만, 다음처럼 Arrays.asList() 메서드를 사용하게 되면 1줄로 코드를 간단하게 작성할 수 있다.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3);
```

 하지만 이 메서드는 만능이 아니다. 요소를 **갱신**하는 것은 가능하지만, 고정 크기의 리스트를 만들었기에 요소를 **추가**, **삭제**할 수는 없다. 만약에 시도할 경우 **UnsupprotedOperationException**이 발생한다.

```java
List<Integer> list = Arrays.asList(1, 2, 3);
list.set(0, 4); // 이 경우 갱신이므로 가능하다.
list.add(5); // 이경우 요소를 추가하기에 UnsupprotedOperationException 발생한다.
```

 우리는 파이썬, 그루비 등과 같은 언어에서 **컬렉션 리터럴** 즉 [1, 2, 3]과 같은 문법을 사용할 수 는 없을까? 당연하게도 자바에서는 너무 큰 언어 변화와 관련된 비용이 든다는 이유로 다음과 같은 기능을 지원하지 않게되었다. 하지만 새로운 컬렉션API로 동일하게 수행이 가능하다.

<br>


### 1.1 리스트 팩토리

<br>

 새로운 컬렉션 팩토리 메서드인 List.of를 이용하면 간단하게 리스트를 만들 수 있다. 

```java
List<Integer> list = List.of(1, 2, 3);
```

 우리는 기존의 Arrays.asList보다 더 짧고 간단하게 리스트를 생성했다. 그러면 요소를 추가해보자.

```java
List<Integer> list = List.of(1, 2, 3);
list.add(4); // UnsupprotedOperationException 발생
```

 정말 안타깝게도 UnsupprotedOperationException가 발생한다. 왜냐하면 사실 변경할 수 없는 리스트가 만들어 졌기 떄문이다. 하지만 이런 제약은 꼭 나쁜것만은 아니다. **컬렉션이 의도치 않게 변하는 것**을 막을 수 있기 때문이다. 하지만, Arrays.asList와 마찬가지로 요소가 변경되는 것은 막을 수 있는 방법은 없다. List.of메서드는 null값을 허용하지 않으므로, 의도치 않은 버그를 방지할 수 있다는 장점도 있다.

 그렇다면 List.of 메서드는 어떤식으로 되어있는지 확인해보자.

```java
    static <E> List<E> of() {
        return (List<E>) ImmutableCollections.EMPTY_LIST;
    }
    static <E> List<E> of(E e1) {
        return new ImmutableCollections.List12<>(e1);
    }
    static <E> List<E> of(E e1, E e2) {
        return new ImmutableCollections.List12<>(e1, e2);
    }
    static <E> List<E> of(E e1, E e2, E e3) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5,
                                                         e6);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5,
                                                         e6, e7);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5,
                                                         e6, e7, e8);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5,
                                                         e6, e7, e8, e9);
    }
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5,
                                                         e6, e7, e8, e9, e10);
    }
    static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                @SuppressWarnings("unchecked")
                var list = (List<E>) ImmutableCollections.EMPTY_LIST;
                return list;
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return ImmutableCollections.listFromArray(elements);
        }
    }

```

 요소가 몇개이든 오버로딩을 하여 정적 메서드 of를 불러오는 것을 확인할 수 있다. 최대 10개까지는 정적 메서드로 구현하였으며, 10개를 넘는 경우, 가변 인수로 넘어가 static <E> List<E> of(E... elements)를 지원한다. 하지만 10를 넘어가는 경우, 내부적으로 가변 인수 버전을 사용해 추가 배열을 할당해서 리스트로 감싸게 된다. 따라서 배열을 할당하고 초기화하며 나중에 **가비지 컬렉션**을 하는 비용을 지불해야되기 때문에 추천하지않는다. 

<br>

### 1.2 집합 팩토리

<br>

 List.of와 마찬가지로, Set또한 Set.of를 지원한다. 다음의 코드를 봐보자.

```java
Set<Integer> set = Set.of(1, 2, 3);
System.out.println(set); // [1, 2, 3]
```

 우리는 Set을 선언하고 add하여 요소를 추가할 필요없이 쉽게 집합을 생성하였다. 또한 List.of와 마찬가지로, add가 불가능하다. 다른점은 요소를 넣을 때, 중복되는 요소를 넣으면 **IllegalArgumentException**이 발생하게 된다.

```java
Set<Integer> set = Set.of(1, 1, 2); // IllegalArgumentException 발생
```

 List의 of와 마찬가지로 10개까지는 오버로딩, 10를 넘으면 가변 인수로 넘어가게 된다. 다음은 Set인터페이스의 of 메서드이다.

```java
    static <E> Set<E> of() {
        return (Set<E>) ImmutableCollections.EMPTY_SET;
    }
    static <E> Set<E> of(E e1) {
        return new ImmutableCollections.Set12<>(e1);
    }
    static <E> Set<E> of(E e1, E e2) {
        return new ImmutableCollections.Set12<>(e1, e2);
    }
    static <E> Set<E> of(E e1, E e2, E e3) {
        return new ImmutableCollections.SetN<>(e1, e2, e3);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5,
                                               e6);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5,
                                               e6, e7);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5,
                                               e6, e7, e8);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5,
                                               e6, e7, e8, e9);
    }
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5,
                                               e6, e7, e8, e9, e10);
    }
    static <E> Set<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                @SuppressWarnings("unchecked")
                var set = (Set<E>) ImmutableCollections.EMPTY_SET;
                return set;
            case 1:
                return new ImmutableCollections.Set12<>(elements[0]);
            case 2:
                return new ImmutableCollections.Set12<>(elements[0], elements[1]);
            default:
                return new ImmutableCollections.SetN<>(elements);
        }
    }
```

<br>

### 1.3 맵 팩토리

<br>

 맵을 만드는 것 또한 쉽게 만들 수 있게 맵 팩토리 메서드를 제공한다. List, Set과 마찬가지로 of메서드를 사용한다. 다만 맵에 요소를 추가 할때에는 K, V의 형태로 순서대로 추가해주어야 한다. 다음은 맵 팩토리 메서드를 사용하는 예시이다.

```java
Map<String, Integer> map = Map.of("A", 1, "B", 2, "C", 3);
```

 또한 고정된 크기를 생성하기 때문에 put메서드를 사용할 수 없다. 10개 이하의 경우 Map.of로 생성하는 것이 효율적이며 10을 넘어가게 되면 Map.entry를 import해 다음과 같이 요소를 추가하는 것이 올바르다.

```java
Map<String, Integer> map = Map.ofEntries(("A", 1),("B", 2),("C", 3));
``` 

 다음은 Map인터페이스에 대한 코드이다.

```java
    static <K, V> Map<K, V> of() {
        return (Map<K,V>) ImmutableCollections.EMPTY_MAP;
    }
    static <K, V> Map<K, V> of(K k1, V v1) {
        return new ImmutableCollections.Map1<>(k1, v1);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5,
                               K k6, V v6) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5,
                                               k6, v6);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5,
                               K k6, V v6, K k7, V v7) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5,
                                               k6, v6, k7, v7);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5,
                               K k6, V v6, K k7, V v7, K k8, V v8) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5,
                                               k6, v6, k7, v7, k8, v8);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5,
                               K k6, V v6, K k7, V v7, K k8, V v8, K k9, V v9) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5,
                                               k6, v6, k7, v7, k8, v8, k9, v9);
    }
    static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5,
                               K k6, V v6, K k7, V v7, K k8, V v8, K k9, V v9, K k10, V v10) {
        return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5,
                                               k6, v6, k7, v7, k8, v8, k9, v9, k10, v10);
    }
    static <K, V> Map<K, V> ofEntries(Entry<? extends K, ? extends V>... entries) {
        if (entries.length == 0) { // implicit null check of entries array
            @SuppressWarnings("unchecked")
            var map = (Map<K,V>) ImmutableCollections.EMPTY_MAP;
            return map;
        } else if (entries.length == 1) {
            // implicit null check of the array slot
            return new ImmutableCollections.Map1<>(entries[0].getKey(),
                    entries[0].getValue());
        } else {
            Object[] kva = new Object[entries.length << 1];
            int a = 0;
            for (Entry<? extends K, ? extends V> entry : entries) {
                // implicit null checks of each array slot
                kva[a++] = entry.getKey();
                kva[a++] = entry.getValue();
            }
            return new ImmutableCollections.MapN<>(kva);
        }
    }
```

<br><br>


## 2. 리스트와 집합 처리

---

<br>

 자바 8에서는 List, Set 인터페이스에 다음과 같은 메서드가 존재한다.

- **removeIf** : 프레디케이트를 만족하는 요소를 제거한다. List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있다. 

- **replaceAll** : 리스트에서 이용할 수 있는 기능으로 **UnaryOperator** 함수를 이용해 요소를 바꾼다.

- **sort** :  List 인터페이스에서 제공하는 기능으로 리스트를 정렬한다.

 위 메서드들은 호출한 컬렉션 자체를 바꾸게 된다. 새로운 결과를 만드는 스트림 동작과 달리 이들 메서드는 기존 컬렉션만 바꾼다. 왜 이런게 생겨나게 되었을까? 간단하게 코드를 보며 생각해보자.

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(3);
list.add(4);
list.add(2);
list.add(3);
list.add(4);

for (Integer k : list) {
    if(k == 3) {
        list.remove(k);
    }
}
```
 다음과 같은 경우 index : value로 표현할 때
 0 : 1 -> pass
 1 : 3 -> remove 
 List의 고유 특성인 요소가 삭제되면 뒷 요소가 앞 요소로 당겨지게 된다.
 1 : 4 -> Error ConcurrentModificationException
 Iterator가 추적하는 인덱스와 현재 인덱스가 달라 발생하게 된다. (서로 동기화가 이루어 지지 않는다.)

<br>

### 2.1 removeIf 메서드

<br>

 removeIf 메서드를 이용해 요소를 삭제해보자. removeIf의 경우 삭제할 요소를 가르키는 프레디케이트를 사용한다. 다음의 코드를 봐보자.

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(3);
list.add(4);
list.add(2);
list.add(3);
list.add(4);

list.removeIf(element -> element == 3);
```

 [1, 4, 2, 4]가 출력 되며 올바르게 삭제 연산이 잘 되었음을 볼 수 있다.

<br>

### 2.2 replaceAll 메서드

<br>

 요소 삭제가 아닌 변경의 경우에는 replaceAll메서드를 이용하면 된다. 숫자가 3인 경우 9로 바꾸는 코드를 봐보자.

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(3);
list.add(4);
list.add(2);
list.add(3);
list.add(4);

list.replaceAll(element -> element == 3 ? 9 : element);
```

 [1, 9, 4, 2, 9, 4] 올바르게 작동하는 것을 볼 수 있다. 하지만, 짚고 넘어가야 할 것이 있다. 우리가 replaceAll을 통해 수정한 요소는 기존의 요소를 수정하는 것 이므로, 주소 자체는 변하지 않는다.

<br>

## 3. 맵 처리

---

<br>

 자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다. 디폴트 메서드란 간단하게 말하면, 기본적인 구현을 인터페이스에 제공하는 기능이라 할 수 있다. 자주 사용되는 패턴을 개발자가 직접 구현할 필요가 없도록 이들 메서드를 추가한 것이다.

<br>

### 3.1 forEach 메서드

<br>

 forEach 메서드의 경우 맵에서 키와 값을 반복하면서 확인하는 작업이다. 다음의 코드에서 기존의 코드와 forEach 메서드를 이용한 코드를 봐보자.

```java
Map<String, Integer> map = Map.of("A", 1, "B", 2, "C", 3);

for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " " + entry.getValue());
}

map.forEach((key, value) -> System.out.println(key + " " + value));
```

 서로 동일한 결과를 출력하지만, 한 눈에 봐도 후자가 더 간결하고 보기 쉬운 코드임을 알 수 있다.

<br>

### 3.2 정렬 메서드

<br>

 **Entry.comparingByValue**, **Entry.comparingByKey** 메서드를 이용해 맵의 항목을 키 또는 값 기준으로 정렬을 할 수 있다. 다음의 코드르 봐보자.

```java
Map<String, Integer> map = Map.of("C", 3, "B", 1, "A", 2);

map.entrySet().stream().sorted(Entry.comparingByKey()).forEachOrdered(System.out::println);

map.entrySet().stream().sorted(Entry.comparingByValue()).forEachOrdered(System.out::println);
```

 [A=2, B=1, C=3], [B=1, A=2, C=3]로 손쉽게 맵을 정렬할 수 있다.

<br>

### 3.3 getOrDefault 메서드

<br>

 맵을 사용할 때, 키에 대한 값이 없을 경우 또는 키가 없을 경우 **NullpointerException**이 발생함을 알 수 있다. 이런 경우 요청하는 결과가 null인지 확인해야 하는데 이 때 getOrDefault 메서드를 사용하면 쉽게 null에 대한 두려움 없이 사용할 수 있다.

```java
Map<String, Integer> map = Map.of("C", 3, "B", 1, "A", 2);

System.out.println(map.getOrDefault("D", -1));
System.out.println(map.getOrDefault("C", -1));
```

 "D"에 해당하는 값이 존재하지 않으므로 Null이 아닌 -1이 출력되었다. "C"에 해당하는 값이 존재하므로 3이 출력되었다. 즉 값이 존재하냐 안하냐에 따라 두번째 인수가 주어질지 주어지지 않을지 결정하게 된다.

<br>

### 3.4 계산 패턴

<br>

 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 필요할 때가 있다. 예를 들어 키를 이용해 값비싼 동작을 실행해서 얻은 결과를 캐시하려고 할 때, 다음 메서드가 도움이 된다.

- **computeIfAbsent** : 제공된 키에 해당하는 값이 없으면(값이 없거나 Null), 키를 이용해 새 값을 계산하고 맵에 추가한다.

- **computeIfPresent** : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다.

- **compute** : 제공된 키로 새 값을 계산하고 맵에 저장한다.

 정보를 **캐시**할 때 computeIfAbsent를 활용할 수 있다. 다음의 코드를 보며 이해해 보자.

```java
Map<String, Integer> map = new HashMap<>();
map.computeIfAbsent("C", key -> 3);
map.computeIfAbsent("B", key -> 1);
map.computeIfAbsent("A", key -> 2);
```

 ["C" : 3, "B" : 1, "A" : 2]가 출력된다. computeIfAbsent를 통해 우리는 값이 존재하지 않을 경우 계산을 해서 map에 요소를 추가할 수 있음을 알 수 있다.

 computeIfPresent 메서드의 경우 현재 키와 관련된 값이 맵에 존재하며 널이 아닐 때만 새 값을 계산한다. 값을 만드는 함수가 널을 반환하면 현재 매핑을 맵에서 제거한다. (하지만 매핑을 제거할 때는 remove 메서드를 오버라이드 하는 것이 더 바람직하다.) 코드를 통해 이해해보자.

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", null);
map.computeIfPresent("B", (key, value) -> value * 2);
map.computeIfPresent("C", (key, value) -> value + 1);
```

 ["C" : null, "B" : 4, "A" : 1] 이 출력됨을 알 수 있다.

<br>

### 3.5 삭제 패턴

<br>

 제공된 키에 해당하는 맵 항목을 제거하는 remove 메서드는 이미 알고 있을 것이다. 자바 8에서는 키가 특정한 값과 연관되어 있을 때만 삭제 연산을 하는 오버로드 버전 메서드를 제공한다. (아주 손쉽게 제거할 수 있다.)

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", null);

map.remove("C", null);
``` 

 ["B" : 4, "A" : 1] 이 출력됨을 알 수 있다.

<br>

### 3.6 교체 패턴

<br>

 맵의 항목을 바꾸는 데 사용할 수 있는 두 개의 메서드가 맵에 추가되었다.

- **replaceAll** : BiFunction을 적용한 결과로 각 항목의 값을 교체한다. 이 메서드는 이전에 살펴본 List의 replaceAll과 비슷한 동작을 수행한다.

- **replace** : 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만, 값을 교체하는 오버로드 버전도 있다.

 다음의 코드를 보며 이해해보자.

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

map.replaceAll((key, value) -> value % 2 == 0 ? value * 2 : value);
```

 map의 요소중 값이 짝수인 경우 2배를 해주었고, 출력 결과 ["A" : 1, "B" : 4, "C", 3]으로 올바른 출력이 나왔다.

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

map.replace("A", 3);
```

 맵에 키가 존재할 경우 값을 바꾼다. 출력 결과 ["A" : 3, "B" : 2, "C" : 3]으로 올바른 출력이 나왔다.

 사실 우리는 map.put으로 동일한 결과를 낼 수 있음을 알 수 있다. 이 둘의 차이는 무엇일까? 당연한 말이지만, 용도에 따라 다르게 사용한다. 모든 요소에서 특정한 값을 바꾸고 싶을 때는 replaceAll을 사용하며, 맵에 키가 만약 존재한다면 값을 바꿀 때 replace를 맵에 요소를 추가하고 싶거나 덮어 씌우고 싶을 때는 put을 사용한다.

<br>

### 3.7 합침

<br>

 두 개의 그룹이 존재할 때 우리는 두 개의 맵을 합쳐 하나로 만들어야 할 때가 있다. 그런 경우 다음 처럼 putAll을 이용해 합칠 수도있지만, 서로 중복되는 경우 문제가 발생할 수 있다. 이 경우 merge 메서드를 이용해 BiFunction으로 중복 요소를 어떻게 합칠지 결정할 수 있다. 다음의 코드를 보며 이해해 보자

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
Map<String, Integer> map2 = new HashMap<>();
map2.put("A", 4);
map2.put("B", 2);
map2.put("C", 3);
map2.put("D", 5);

map.putAll(map2);

// 기존의 요소들이 새롭게 추가된 중복 키의 값으로 덮어씌워지게 된다.
// ["A" : 4, "B" : 2, "C" : 3, "D" : 5]

Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
Map<String, Integer> map2 = new HashMap<>();
map2.put("A", 4);
map2.put("B", 2);
map2.put("C", 3);
map2.put("D", 5);

map2.forEach((key, value) -> map.merge(key, value, (value1, value2) -> value1));

// 중복되는 키에 대한 값을 결정할 수 있다. 위 코드는 기존의 값을 유지하는 코드이다.
// ["A" : 1, "B" : 2, "C" : 3, "D" : 5]
```

<br><br>

## 4. 개선된 ConcurrentHashMap

---

<br>

 ConcurrentHashMap은 **동시성 친화적**이며 최신 기술을 반영한 HashMap버전이다. HashMap은 비동기이다.(HashTable이 동기) ConcurrentHashMap은 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다. 따라서 기존 HashTable 버전에 비해 읽기 쓰기 연산 성능이 월등히 뛰어나다.

<br>

### 4.1 리듀스와 검색

<br>

 ConcurrentHashMap은 스트림에서 봤던 것과 비슷한 종류의 세 가지 새로운 연산을 지원한다.

- **forEach** : 각 (키, 값) 쌍에 대한 주어진 액션을 수행한다.

- **reduce** : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합친다.

- **search** : 널이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용한다.

 다음처럼 키에 함수 받기, 값, Map.Entrym (키, 값) 인수를 이용한 네 가지 연산 형태를 지원한다.

- 키, 값으로 연산(forEach, reduce, search)

- 키로 연산(forEachKey, reduceKeys, searchKeys)

- 값으로 연산(forEachValue, reduceValues, searchValues)

- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

 여기서 우리는 이들 연산이 ConcurrenthashMap의 상태를 잠그지 않고, 연산을 수행한다는 점을 주목하자. 즉 이들 연산에서 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다. 또한 이들 연산에 병렬성 기준값을 지정해야한다. 만약 맵의 크기가 주어진 기준값보다 작으면 순차적으로 연산을 실행한다. 

<br>

### 4.2 계수

<br>

 ConcurrentHashMap은 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다. 기존의 size 메서드 대신 새 코드에서는 int를 반환하는 mappingCount 메서드를 사용하는 것이 좋다. 그래야 매핑의 개수가 int의 범위를 넘어서는 상황에 대처가 가능하기 때문이다.

<br>

### 4.3 집합뷰

<br>

 ConcurrentHashMap은 집합 뷰로 반환하는 KeySet이라는 새로운 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌며, 반대로 집합을 바꾸면 맵도 바뀌게 된다. newKeySet이라는 새 메서드를 이용해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.

<br><br>

## 5. 마치며

---

<br>

- 자바 9는 적의 원소를 포함하며, 바꿀 수 없는 리스트, 집합, 맵을 쉽게 만들 수 있도록 List.of, Map.of, Map.ofEntries 등의 컬렉션 팩토리를 지원한다.

- 이들 컬렉션 팩토리가 반환한 객체는 만들어진 다음 바꿀 수 없다.

- List 인터페이스는 removeIf, replaceAll, sort 세 가지 디폴트 메서드를 지원한다.

- Set 인터페이스는 removeIf 디폴트 메서드를 지원한다.

- Map 인터페이스는 자주 사용하는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드를 지원한다.

- ConcurrentHashMap은 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공한다.

<br><br>


## 6. 배우고 느낀점

---

<br>

 이번 챕터는 읽고 이해하고 응용하는데 쉽게 느껴졌다. 아마 다른사람들도 마찬가지라고 생각한다. 7장에서 궁금한점이 많아 같이 스터디를 진행하는 팀원들과 GPT를 이용하여 모르는 문제들을 해결했고, 이번 챕터에서 ConcurrentHashMap에 대해 이론적인 부분만 나왔기에 부족한 부분을 스터디원들과 해결할 것이다. 

<br><br>