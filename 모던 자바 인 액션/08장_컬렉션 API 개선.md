## 8.1 컬렉션 팩토리

자바 9에서는 작은 컬렉션 객체를 쉽게 만들 수 있는 몇 가지 방법을 제공한다.

### 리스트 팩토리

> `List.of` 팩토리 메서드

- 리스트에  요소를 추가하려고 하면 UnsupportedOperationException 예외가 발생
- 요소를 변경할 수 없다.
- null이 들어갈 수 없다.

`불변`이라는 특징을 가진다.

### 오버로딩 vs 가변 인수

```java
// List 인터페이스 안의 of 메서드
static <E> List<E> of()
static <E> List<E> of(E e1)
...
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6)
```

왜 자바 API를 static <E> List<E> of(E… elements)처럼 다중 요소를 받도록 만들지 않았을까?

⇒ 내부적으로 **가변 인수 버전은 추가 배열을 할당해서 리스트로 감싼다.** 따라서 배열을 할당하고 초기화하며 **나중에 가비지 컬렉션을 하는 비용을 지불**해야 한다. **고정된** 최대 10개까지를 API로 정의하므로 **이런 비용을 제거**할 수 있다. (10개 이상 요소를 가진 리스트는 가변 인수를 사용하는 메서드를 사용)

### 집합 팩토리

> `Set.of` 팩토리 메서드

- 중복된 요소를 넣으려고 하면 IllegalArgumentException 예외가 발생

### 맵 팩토리

1. `Map.of` 팩토리 메서드: 키와 값을 번갈아 제공

```java
Map<String, Integer> ageOfFriends = Map.of("A", 10, "B", 20, "C", 30);
```

10개 이상의 키와 값의 쌍을 가진 맵을 생성하려면 아래의 메서드를 사용하자.

2. `Map.ofEntries` 팩토리 메서드: Map.Entry<K, V> 객체를 인수로 받으며 **가변 인수**로 구현되어 있다. 키와 값을 감쌀 추가 객체 할당을 필요로 한다.

```java
import static java.util.Map.entry; // Map.Entry 객체를 만드는 팩터리 메서드

Map<String, Integer> ageOfFriends = Map.ofEntries(entry("A", 10),
                                                  entry("B", 20),
                                                  entry("C", 30));
```

<br>

## 8.2 리스트와 집합 처리

아의 메서드들은 호출한 컬렉션 자체를 바꾼다. 컬렉션을 바꾸는 동작은 에러를 유발하며 복잡함을 더한다. 이런 어려움으로 아래 메서드를 추가했다.

### removeIf 메서드

> `프레디케이트`를 만족하는 요소를 제공한다. (List, Set 구현 / 상속받은 클래스 이용 가능)
> 

```java
for (Transaction transaction : transactions) {
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    transactions.remove(transaction);
  }
}

// for-each 루프는 내부적으로 Iterator 객체를 사용하므로 아래와 같이 해석됨
// 문제점: Iterator 객체(소스 질의)와 Collection 객체(요소 삭제) - 두 개의 개별 객체가 컬렉션을 관리
// 그래서 반복자의 상태는 컬렉션의 상태와 **서로 동기화되지 않음**
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    transactions.remove(transaction);
  }
}

// 해결
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext(); ) {
  Transaction transaction = iterator.next();
  if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
    iterator.remove();
  }
}
```

```java
// 자바 8의 removeIf 메서드
transactions.removeIf(transaction -> 
    Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

### replaceAll 메서드

> `UnaryOperator` 함수를 이용해 요소를 바꾼다. (리스트에서 이용 가능)
> 

```java
// 스트림 API
// 단점: 새 문자열 컬렉션을 만든다.
referenceCodes.stream()
              .map(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1))
              .collect(Collectors.toList());

// replaceAll 메서드 -> 기존의 컬렉션 수정
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
```

### sort 메서드

> List 인터페이스에서 제공, 리스트를 정렬한다.
> 

<br>

## 8.3 맵 처리

자바 8에서는 Map 인터페이스에 몇 가지 **디폴트 메서드를 추가**했다.

### forEach 메서드

> BiConsumer(키와 값을 인수로 받음)를 인수로 받아 맵의 모든 키와 값을 반복한다.
> 

```java
맵변수명.forEach((키, 값) -> ...));
```

### 정렬 메서드

다음과 같이 두 개의 새로운 유틸리티 이용하면 맵을 키 또는 값을 기준으로 정렬 가능

1. `Entry.comparingByValue`
2. `Entry.comparingByKey`

```java
Map<String, String> favouriteMovies = 
	Map.ofEntites(entry("A", "B"), entry("C", "D"), entry("E","G"));

favouriteMovies.entrySet().stream()
		.sorted(Entry.comparingByKey()) //  키 값을 기준으로 알파벳 순서로 정렬한 스트림 반환
		.forEachOrdered(System.out::println); // 정렬된 스트림의 각 엔트리를 순차적으로 처리
```

### getOrDefault 메서드

> 존재하지 않는 키를 찾으려고하면 NullPointerException이 반환된다. 에러 대신 기본 값을 반환하도록 한다.
> 

```java
맵변수명.getOrDefault("키 값", "기본 값");
```

### 계산 패턴

맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 필요할 때가 있다. **값 비싼 동작을 실행해서 얻은 결과를 캐시하려고 할 때 키가 존재하면 결과를 다시 계산할 필요가 없다.** 다음 세 가지 연산이 이런 상황에서 도움을 준다.

- `computeIfAbsent`: 제공된 키에 해당하는 값이 없으면(값이 없거나 널), 키를 이용해 새 값을 계산하고 맵에 추가한다.
- `computIfPresent`: 제공된 키가 존재(null이 아니여야함)하면 새 값을 계산하고 맵에 추가한다.
- `compute`: 제공된 키로 새 값을 계산하고 맵에 저장한다.

```java
// 키가 존재하지 않으면 값을 계산해 맵에 추가하고 키가 존재하면 기존 값을 반환
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>()).add("Star Wars");
```

### 삭제 패턴

1. `remove`

```java
맵변수명.remove(키, 값);
```

1. `removeIf`: 맵의 항목 집합에 프리디케이티를 인수로 받는다.

```java
맵변수명.entrySet().removeIf(entry -> entry.getValue() < 10);
```

### 교체 패턴

1. `replaceAll`: BiFunction을 적용한 결과로 각 항목의 값을 교체한다. (리스트의 replaceAll와 비슷한 동작 수행)
2. `replace`: 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다.

```java
// Map<String, String> 타입인 경우
맵변수명.replaceAll((키, 값) -> 값.toUpperCase());
```

### 합침

`putAll`: 두 개의 맵을 합칠 때 사용

> 중복된 키가 없다면 아래의 코드는 잘 동작한다.
> 

```java
Map<String, String> x = Map.ofEntries(...);
Map<String, String> y = Map.ofEntries(...);
Map<String, String> z = new HashMap<>(x);
z.putAll(y); // y의 모든 항목을 z로 복사
```

`merge` : **중복된 키를 어떻게 합칠지 결정하는 BiFunction을 인수로 받는다.**

- 중복된 키가 있을 시 값들을 합치는 예제

```java
y.forEach((키, 값) -> z.merge(키, 값, (confilctVal1, confilctVal2) -> confilctVal1 + " & " + confilctVal2));
```

- 영화 시청 횟수를 기록하기 전에 영화가 맵에 존재하는지 확인해야 한다.

영화가 존재하지 않는다면 두 번째 인자가 해당 키와 연결

```java
// [ 자바독에 있는 두 번째 인자의 설명 ] "키와 연관된 기존 값에 합쳐질 널이 아닌 값 
// 또는 값이 없거나 키에 널 값이 연관되어 있다면 이 값을 키와 연결"
맵변수명.merge(키, 1L, (key, count) -> count + 1L);
```

<br>

## 8.4 개선된 ConcurrentHashMap

동시성 친화적이며 최신 기술을 반영한 HashMap 버전이다. 내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다. 그래서 Hashtable 버전에 비해 읽기 / 쓰기 연산 성능이 월등하다.

> 표준 HashMap은 비동기로 동작함
> 

### 리듀스와 검색

세 가지 새로운 연산을 지원한다.

- `forEach`: 각 (키, 값) 쌍에 주어진 액션을 실행
- `reduce` : 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
    - 기본 값에는 전용 reduceValuesToInt와 같은 연산이 제공되므로 박싱 작업이 필요 없어 효율적으로 작업 처리 가능
- `search`: null이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용

네 가지 연산 형태를 지원한다.

- 키, 값으로 연산(forEach, reduce, search)
- 키로 연산(forEachKey, reduceKeys, searchKeys)
- 값으로 연산(forEachVlaue, reduceVlaues, searchVlaues)
- Map.Entry 객체로 연산(forEachEntry, reduceEntries, searchEntries)

> 이들 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행한다는 점을 주목하자.
> → 이들 연산에 제공한 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.
> 

### 계수

맵의 매핑 계수를 반환하는 `mappingCount` 메서드를 제공한다. 기존의 size 메서드 대신 해당 메서드를 사용하는 것이 좋다.

### 집합

집합 뷰로 변환하는 `keySet`이라는 메서드를 제공한다. 맵을 바꾸면 집합도 바뀌고 반대로 집합을 바꾸면 맵도 영향을 받는다. `newKeySet`이라는 새 매세드를 이용해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있다.

<br>

## 8.5 마치며

- 자바 9는 작은 원소를 포함한 바꿀 수 없는 리스트, 집합, 맵을 쉽게 만들 수 있도록 List.of, Set.of, Map.of, Map.ofEntites 등의 컬렉션 팩토리 지원
- List 인터페이스는 removeIf, replaceAll, sort 세 가지 디폴트 메서드 지원
- Set 인터페이스는 removeIf 디폴트 메서드 지원
- Map 인터페이스는 자주 사용되는 패턴과 버그를 방지할 수 있도록 다양한 디폴트 메서드 지원
- ConcurrentHashMap는 Map에서 상속받은 새 디폴트 메서드를 지원함과 동시에 스레드 안전성도 제공
