여러 해가 지난 후 호어는 당시 null 및 예외를 만든 결정을 가리켜 ‘십 달러짜리 실수’라고 표현했다. 모든 자바 프로그래머라면 무심코 어떤 객체의 필드를 사용하려 할 때 NullPointerException이라는 귀찮은 예외가 발생하는 상황을 몸소 겪었을 것이다. (모든 객체가 null일 수 있기 때문)

<br>

## 11.1 값이 없는 상황을 어떻게 처리할까?

### 보수적인 자세로 NullPointerException 줄이기

NullPointerException을 피하려면 null 확인 코드를 추가해서 null 예외 문제를 해결하려 할 것이다. 모든 변수가 null인지 의심된다면 변수를 접근할 때마다 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가할 수 있다. 이와 같은 `반복패턴 코드`는 ‘**깊은 의심**’이라고 부르고 가독성이 떨어진다.

[ 해결 ] 중첩된 if 문은 null 변수 확인 시 즉시 처리를 해 여러 개의 출구를 만들어 들여쓰기 수준을 줄일 수 있지만 여전히 쉽게 에러를 발생시킬 수 있다는 문제는 해결되지 않는다.

### null 때문에 발생하는 문제

- `에러의 근원`: NullPointerException은 자바에서 가장 흔히 발생하는 에러
- `코드를 어지럽힌다`: null 확인 코드를 추가해야 하므로 가독성이 떨어진다.
- `아무런 의미가 없다`: null은 아무 의미도 표현하지 않는다.
- `자바 철학에 위배된다`: 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 null 포인터는 아니다.
- `형식 시스템에 구멍을 만든다`: 모든 참조 형식에 null 할당 가능하다.

<br>

## 11.2 Optional 클래스 소개

자바 8은 java.util.Optional<T>라는 새로운 클래스를 제공한다. Optional은 선택형값을 캡슐화하는 클래스이다. **값이 있으면** Optional 클래스는 **값을 감싸**는 반면 **값이 없으면** `Optional.empty()`라는 **특별한 싱글턴 인스턴스를 반환하는 정적 팩터리 메서드를 반환**한다. Optional.empty()는 Optional 객체이므로 에러가 발생하지 않는다.

> **모든 null 참조를 Optional로 대치하는 것은 바람직하지 않다.** Optional의 역할은 메서드의 시그니처만 보고도 `선택형 값인지 여부`를 구별하도록 하는 것이다.
> 

```java
public class Car {
  private Optional<Insurance> insurance; // 자동차가 보험에 가입되어 있을 수도 있고 아닐 수도 있다.
  public Optional<Insurance> getInsurance() {
    return insurance;
  }
}

public class Insurance {
  private String name; // 보험회사에는 반드시 이름이 있다.
  public String getName() {
    return name;
  }
}
```

<br>

## 11.3 Optional 적용 패턴

### Optional 객체 만들기

[1] 빈 Optional 

```java
Optional<Car> optCar = Optional.empty(); // 정적 팩토리 empty
```

[2] null이 아닌 값으로 Optional 만들기

> 인수 값이 null이라면 즉시 NullPointerException이 발생
> 

```java
Optional<Car> optCar = Optional.of(car); // 정적 팩토리 of
```

[3] null 값으로 Optional 만들기

> 인수 값이 null이면 빈 Optional 객체가 반환
> 

```java
Optional<Car> optCar = Optional.ofNullable(car); // 정적 팩토리 of
```

### 맵으로 Optional의 값을 추출하고 변환하기

```java
// Optional 사용 전
String name = null;
if(insurance != null) {
  name = insurance.getName();
}
```

스트림의 map은 스트림의 각 요소에 제공된 함수를 적용하는 연산이다. **여기서 Optional 객체를 최대 요소의 개수가 한 개 이하인 데이터 컬렉션으로 생각**할 수 있다. **Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾸지만 값이 비어 있으면 아무 일도 일어나지 않는다.**

```java
Optional<Insurance> optInsurance = Optioanl.ofNullable(insuracne);
Optional<String> name = optInsurance.map(Insurance::getName);
```

### flatMap으로 Optional 객체 연결

아래 코드를 어떻게 안전하게 호출할까?

```java
public String getCarInsuranceName(Person person) {
  return person.getCar().getInsuracne.getName();
}
```

아래 코드의 첫 번째 flatMap() 대신 map()을 사용하면 Optional<Car>이 아닌 `Optional<Optional<Car>> 형식`의 객체가 반환된다. 반환된 객체는 getInsurance 메서드를 가지지 않으므로 **map을 사용할 수 없고 flatMap을 사용해야 한다.**

> flatMap은 함수를 적용해서 생성된 모든 스트림이 하나의 스트림으로 병합되어 평준화된다. 빈 Optional이라면 아무 일도 일어나지 않고 그대로 반환된다.
> 
> 
> (**2차원 스트림이 3개** ⇒ **6개의 원소를 가진 일차원 스트림**)
> 

```java
public String getCarInsuranceName(Optional<Person> person) {
  return person.flatMap(Person::getCar) // getCar한 반환 값이 Optional<Optional<Car>>으로 평준화 필요
          .flatMap(Car::getInsurance)
          .map(Insurance::getName)
          .orElse("Unknown"); // Optional이 비어있으면 기본값 사용
}
```

### Optional 스트림 조작

자바 9에서 Optional에 stream() 메서드를 추가했다. Optional 스트림 값을 가진 스트름으로 변환할 때 이 기능을 유용하게 활용할 수 있다.

[ 아래 코드에서 주의 깊게 볼 것 ]
1. Stream<Optional<String>>으로 매핑하는 부분: **보험 여부가 비어있을 수 있지만 Optional 덕분에 널 걱정없이 안전하게 처리가능**
2. **마지막에 결과**를 얻으려면 **Optional을 제거하고 값을 `언랩`해야 하는 문제점**을 가진다.

```java
public Set<String> getCarInsuranceNames(List<Person> persons) {
  return persons.stream()
                .map(Person::getCar) // Stream<Optional<Car>> 반환
                .map(optCar -> optCar.flatMap(Car::getInsurance) // Stream<Optional<Insurance>>로 변환
                .map(optIns -> optIns.map(Insurance::getName) // Stream<Optional<String>>으로 매핑
                .flatMap(Optional::stream) // Stream<String>으로 변환
                .collect(toSet());
}
```

아래 코드 처럼 결과를 얻을 수 있다. (위에는 한 줄로 가능)

```java
Stream<Optional<String>> stream = ...
Set<String> result = stream.filter(Optional::isPresent)
                           .map(Optional::get)
                           .collect(toSet());
```

### 디폴트 액션과 Optional 언랩

Optrional 인스턴스에 포함된 값을 읽는 다양한 방법을 제공한다.

- `get()`: 반드시 값이 있다고 가정할 수 있는 상황이 아니라면 사용하지 말자.
- `orElse(T other)`: 기본 값 제공
- `orElseGet(Supplier<? extends T> other)`: orElse에 대응하는 게으른 버전의 메서드이다. Optional에 값이 없을 때만 Supplier가 실행되기 때문이다.
- `orElseThrow(Supplier<? extends X> exceptionSupplier)`: 값이 비어있을 때 에러 발생시킨다. get 메서드와 다르게 원하는 에러를 발생시킬 수 있다.
- `ifPresent(Consumer<? extends T> consumer)`: 값이 존재할 때 인수로 넘겨준 동작 실행 가능
- `ifPresentOrElse(Consumer(Supplier<? extends T> action, Runnable emptyAction)`: Optional이 비어있을 때 실행할 수 있는 Runnable을 인수로 받는다는 점만 ifPresent 메서드와 다르다.

### 두 Optional 합치기

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    if (person.isPresent() && car.isPersent()) {
        return Optional.of(findCheapestInsurance(person.get(), car.get());
    } else {
        return Optional.empty();
    }
}
```

안타깝게도 구현 코드는 null 확인 코드와 크게 다른 점이 없다. Optional 클래스에서 제공하는 기능을 사용해 더 자연스럽게 코드를 개선해보자.

- **첫 번째 Optional에서 flatMap을 호출**했으므로 첫 번째 Optional이 비어있다면 인수로 전달한 람다 표현식이 실행되지 않고 그대로 빈 Optional을 반환하다.
- **두 번째 Optional에 map을 호출**하므로 값이 없다면 Function은 빈 Optional을 반환한다.

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
        return person.flatMap(p -> car.map(c ->findCheapestInsurance(p, c));
}
```

### 필터로 특정값 거르기

Optional에 값이 비어있으면 filter 연산은 아무 동작을 하지 않고, 값이 있다면 해당 값에 프레디케이트를 적용한다.

> **Optional은 최대 한 개의 요소를 포함할 수 있는 스트림과 동일**하다.
> 

```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
            .ifPresent(x -> System.out.println("ok"));
```

<br>

## 11.4 Optional을 사용한 실용 예제

호환성을 유지하다보니 기존 자바 API는 Optional을 적절하게 활용하지 못하고 있다. Optional 기능을 활용할 수 있도록 우리 코드에 작은 유틸리티 메서드를 추가하는 방식으로 이 문제를 해결할 수 있다.

### 잠재적으로 null이 될 수 있는 대상 Optional로 감싸기

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));
```

### 예외와 Optional 클래스

어떤 값을 제공할 수 없을 때 null을 반환하는 대신 예외를 발생 시킬 때가 있다.

문자열을 정수로 바꾸지 못할 때 NumberFormatException을 발생시킨다. 에러 대신 빈 Optional을 반환하도록 해 문제를 해결할 수 있다. 다음과 같은 작은 유틸리티 메서드를 구현해서 간단하게 사용이 가능하다.

```java
public static Optional<Integer> stringToInt(String s) {
  try {
    return Optional.of(Integer.parseInt(s));
  } catch (NumberFormatException e) {
    return Optional.empty();
  }
}

// 사용 - 유틸리티클래스.stringToInt(s)
```

### 기본형 Optional을 사용하지 말아야 하는 이유

> Optional도 기본형으로 특화된 OptionalInt, OptionalLong, OptionalDouble 등의 클래스를 제공한다.
> 
1. **Optional의 최대 요소 수는 한 개**이므로 기본형 클래스로 성능을 개선할 수 없다.
2. 이 클래스는 **map, flatMap, filter 등을 지원하지 않으므로** 사용을 권장하지 않는다. 
3. **일반 Optional과 혼용해서 사용할 수 없다.**

<br>

## 11.5 마치며

- 역사적으로 null 참조로 값이 없는 상황을 표현해왔다.
- 값이 있거나 없음을 표현할 수 있는 클래스 java.util.Optional<T>를 제공
- 팩토리 메서드 Optional.empty, Optional.of, Optional.ofNullable 등으로 Optional 객체 생성 가능
- Optional 클래스는 스트림과 비슷한 연산을 수행하는 map, flatMap, filter 등의 메서드를 제공
- Optional로 값이 없는 상황을 적절하게 처리하도록 강제 가능
    - 즉, Optional로 예상치 못한 null 예외를 방지할 수 있다.
- 사용자는 메서드의 시그니처만 보고도 Optional 값이 사용되거나 반환되는지 예측할 수 있어 더 좋은 API 설계 가능
