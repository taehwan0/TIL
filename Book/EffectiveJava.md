# 목차

- [객체 생성과 파괴](#객체-생성과-파괴)
  - 아이템 1. 생성자 대신 정적 패터리 메서드를 고려하라
  - 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
  - 아이템 3. private 생성자나 열거타입으로 싱글턴임을 보증하라
  - 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
- [모든 객체의 공통 메서드](#모든-객체의-공통-메서드)
  - 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라
- [일반적인 프로그래밍 원칙](#일반적인-프로그래밍-원칙)
  - 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

---

# 객체 생성과 파괴

## 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클래스의 인스턴스를 얻기 위해서는 `public` 생성자를 사용하는 것이 일반적이나, 생성자와 별도로
해당하는 클래스의 인스턴스를 반환하는 정적 팩터리 메서드를 제공할 수 있다. `boolean` 기본 타입의 
박싱 클래스인 `Boolean` 에서는 아래와 같은 방식을 사용하고 있다. 새로운 인스턴스를 생성하지 않고,
미리 만들어놓은 인스턴스를 재활용한다.

```java
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FLASE;
    }
```

> Boolean 클래스에서는 클래스 멤버로 TRUE, FALSE 의 인스턴스를 생성해 가지고있고,
> valueOf() 메서드를 호출하면 이들을 반환한다. public 생성자를 사용하는 방법 또한 존재하지만,
> java9 버전 부터 Deprecated 되었다.

### 정적 팩터리 메서드의 장점

1. 이름을 가질 수 있다.
    - 반환될 객체의 특징을 더 상세히 설명할 수 있다.
2. 호출될 때 마다 인스턴스를 새로 생성하지 않아도 된다.
    - 불필요한 객체 생성을 피할 수 있다.
3. 반환 타입의 하위 타입 객체를 반환할 수 있다.
   - 반환 타입에 유연성을 제공할 수 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

### 정적 팩터리 메서드의 단점

1. 상속을 하려면 `public` 또는 `protected` 생성자가 필요하기 때문에 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

### 정적 팩터리 메서드 명명 방식

1. `from`: 매개변수 하나를 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드
2. `of`: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
3. `valueOf`: from, of 의 더 자세한 버전의 메서드
4. `instance`, `getInstance`: (매개 변수를 받을 경우)매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는 메서드
5. `create`, `newInstance`: instance, getInstance 와 같지만 새로운 인스턴스를 생성해 반환함을 보장
6. `getType`: getInstance 와 같지만 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
7. `newType`: getType 과 같으나 새로운 인스턴스를 생성해 반환함을 보장한다.
8. `type`: getType, newType 의 간결한 방법

> 생성자와 정적 팩터리 메서드의 장단점을 이해하고 사용하자. 정적 팩터리 메서드를 사용하는 것이 유리한 케이스가 많으니 
> 무작정 public 생성자를 사용하던 습관이 있다면 고치는 것이 좋겠다.

---

## 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

### 점층적 생성자 패턴

정적 팩터리와 생성자에는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다. 클래스의 선택적 매개변수가 존재할때
가장 간단한 방법은 `점층적 생성자 패턴`이다. 선택적 매개변수를 1개만 받는 생성자, 2개만 받는 생성자, 3개만 받는 생성자...
의 형태로 생성자를 늘려나가는 방식이다. 

```java
    // 점층적 생성자 패턴
    public User(Long id, String name, String email, String address, Long phoneNumber) {
    ...
    }
    public User(Long id, String name, String email, Long phoneNumber) {
    ...
    }

    public User(Long id, String name, String email, String address) {
    ...
    }

    public User(Long id, String name, String email) {
    ...
    }
```

`점층적 생성자 패턴`은 구현 방법은 간단하지만 
1. 매개변수와 선택적 매개변수의 수가 늘어날 수록 생성자의 개수가 늘어날 수 있고
2. 이에 따라 생성자를 호출하는 코드를 작성하기 복잡해진다. 

### 자바빈즈 패턴

두 번째 방법은 자바빈즈 패턴이다. 매개변수가 없는 생성자로 객체를 생성하고 `setter` 메서드를 정의해 
매개변수의 값들을 설정한다.

```java
    public User() {}

    public void setId(Long id) {
        ...
    }
    
    public void setName(String name) {
        ...
    }
    
    public void setEmail(String email) {
        ... 
    }
    
    public void setAddress(String address) {
        ...
    }
    
    public void setPhoneNumber(Long phoneNumber) {
       ...
    }
    
    ...
```

`점층적 생성자 패턴`에서의 단점인 생성자의 개수가 늘어나거나, 호출하는 코드가 복잡해지는 등의 단점은 `자바빈즈 패턴`에서는
해소가 되었다. 하지만, 
1. 객체 하나를 생성하기 위해서는 메서드의 호출이 여러번 일어나야 하고 
2. 객체가 완성되기 전까지 일관성이 무너진 상태에 놓이게 된다.
3. 상태 변화를 허용하기 때문에 불변 객체로 만들 수 없다.
4. 스레드 안정성을 얻으려면 추가 작업이 필요하다.

### 빌더 패턴

`빌더패턴`은 `점층적 생성자 패턴`의 안정성과 `자바빈즈 패턴`의 가독성을 겸비한 패턴이다. 클라이언트가 직접 객체를
생성하는 대신, 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻고, 빌더 객체의 메서드들을 호출해 선택적 매개변수들을 
설정한다. 마지막으로 `build` 메서드를 호출해 최종적으로 객체를 생성한다.

```java
// User.java
public class User {

    public static class Builder {

        private final Long id;
        private final String name;
        private final String email;

        private String address = null;
        private Long phoneNumber = null;

        public Builder(Long id, String name, String email) {
            this.id = id;
            this.name = name;
            this.email = email;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public Builder phoneNumber(Long phoneNumber) {
            this.phoneNumber = phoneNumber;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }

    private final Long id;
    private final String name;
    private final String email;
    private final String address;
    private final Long phoneNumber;

    private User(Builder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.email = builder.email;
        this.address = builder.address;
        this.phoneNumber = builder.phoneNumber;
    }
}
```

```java
    // main 에서의 호출
    public static void main(String[] args) {
        User user = new Builder(1L, "KIM", "KIM@email.com")
            .address("ADDRESS")
            .phoneNumber(821012345678L)
            .build();
    }
```

`빌더패턴` 은 빌더 자신을 반환함으로써 연쇄적으로 메서드를 호출하는 방식으로 쓸 수 있다. 이런 메서드 호출 방식을
`플루언트API(fluent API)` 혹은 `메서드 연쇄(method chaining)` 라고 한다.

`빌더패턴` 은  

1. 쓰기 쉽고, 읽기 쉽다.
2. 불변 객체를 생성할 수 있다.
3. **계층적으로 설계된 클래스와 함께 쓰기에 좋다.**

[계층적으로 설계된 클래스에 적용 예시](https://github.com/jbloch/effective-java-3e-source-code/tree/master/src/effectivejava/chapter2/item2/hierarchicalbuilder)

의 장점을 갖지만 각 클래스의 빌더를 정의 및 생성이 필요하기 때문에 코드를 추가적으로 작성하고,
자원을 추가로 사용한다는 단점도 존재한다. 하지만, 생성자나 정적 팩터리 방식을 택했다가,
추후에 매개변수가 늘어난다면 수정해야 하는 부분이 많을 수 있다. 때문에 애초에 `빌더패턴` 으로 
시작하는 것이 좋을 수 있다.

> 생성자나 정적 팩터리가 처리할 매개변수가 많다면, 빌더 패턴을 선택하는게 낫다. 특히, 선택적
> 매개변수의 개수가 많을 때 효율적이며 객체를 생성하기에 안전하고도 읽고 쓰기 간결한 좋은 방법이다.

---

## 아이템 3. private 생성자나 열거타입으로 싱글턴임을 보증하라

> 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
> 싱글턴의 전형적인 예로는 함수와 같은 무상태 객체나 설계상 유일해야 하는
> 시스템 컴포넌트를 들 수 있다. **그런데 클래스를 싱글턴으로 만들면 클라이언트를 테스트하기가 어려워질 수 있다.**

### 싱글턴을 만드는 방법 1

싱글턴을 만드는 방법은 보통 2가지가 있는데 우선 생성자를 `private`으로 선언하여 감추는 것으로 시작한다.

첫 번째 방법은 `public static final`로 선언된 INSTANCE 필드를 사용하는 방법이다.

```java
public class User {

    public static final User INSTANCE = new User();
    
    private User() {
    }
}
```

`private`으로 선언된 생성자 하나만을 가지고 있고, 클래스가 초기화 될 때 하나의 인스턴스가 생성됨으로써 싱글턴이 보장된다.
예외적으로 권한이 있는 클라이언트가 리플렉션 API를 사용하여 `private` 생성자를 호출하는 것이 가능하다.

### 싱글턴을 만드는 방법 2
두 번째 방법은 필드에 직접 접근하도록 하는 것이 아닌 정적 팩터리 메서드를 제공하는 방법이다.

```java
public class User {

    public static final User INSTANCE = new User();
    
    private User() {
    }
   
    public static User getIntsance() {
        return INSTANCE;
    }
}
```

첫 번째 방법과 마찬가지로 클래스 초기화 시 하나의 인스턴스만 생성되고, 항상 같은 객체가 반환된다.
정적 팩터리 방식은 싱글턴으로 사용하지 않고자 할 때 코드를 수정하기가 용이하다.
호출하여 사용하는 부분에서는 변화가 없기 때문에 메서드 내부에서 새로운 인스턴스를 생성해 반환하도록 하면 된다.

### 싱글턴을 만드는 방법 3
세 번째 방법은 원소가 하나인 열거타입을 선언하는 방법이다.

```java
public enum User {
    INSTANCE;
 
}
```

`public` 필드 방식과 비슷하나 더 간결하고 직렬화에 대한 문제점이 없다. (1, 2번째 방법에는 직렬화, 역직렬화를 위해서 추가적인 구현이 필요하다.)
또한 리플렉션 API를 사용하는 경우에도 싱글턴이 파훼되지 않고 보장된다. 대부분의 상황에서 이 방법이 가장 좋다.
단, 만드려는 싱글턴이 Enum 이외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

---

## 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

`java.lang.Math`, `java.util.Arrays`, `java.util.Collections` 와 같이 정적 메서드와 정적 필드만을 담은 클래스를 생성해야 할 때가 있다. 이 때는 정적인 멤버들만 가지고 있기 때문에 생성자를 호출 할 필요가 없다. 하지만 생성자를 선언하지 않으면 컴파일러가 자동으로 기본 생성자를 만든다.

```java
class ExClass {

		public static void method1() {
			...
		}
}
// -> 생성자가 자동적으로 생성된다.
class ExClass {

		public DefaultConstructorClass() {
		}

		public static void method1() {
			...
		}
}
```

이렇게 기본 생성자가 존재한다면, 의도와 달리 사용자가 해당 클래스의 인스턴스를 생성하게 될 수 있다. `추상 클래스로 만드는 경우에도 상속을 통해 하위 클래스로 인스턴스화가 가능하기 때문에 인스턴스화를 막을 수 없다.` 오히려 추상클래스는 상속해 사용하라는 의미로 받아들이게 될 수 있다. 때문에 명시적으로 private 생성자의 추가가 필요하다.

```java
class ExClass {
		// 인스턴스화 방지용
		private ExClass() {
		}

		public static void method1() {
			...
		}
}
```

private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다. 다른 클래스에 해당 클래스를 상속해 사용하게 되는 경우에도 상위 클래스의 생성자에 접근하는 길을 막을 수 있다. 인스턴스화를 방지하기 위한 생성자 선언임을 주석으로 명시하면 도움이 될 수 있다.

---

# 모든 객체의 공통 메서드

## 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

> equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을
> 어기게 되어 해당 클래스의 인스턴스를 HashMap 이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

### Object 명세의 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을
호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행하면 값이 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

`equals`를 재정의 하여 물리적으로 서로 다른 객체가 같은 객체를 논리적으로 같은 객체로 취급할 수 있다.
이 때 같은 객체로 판단된 두 객체의 `hashCode` 가 다르다면, 위의 Object 명세 규약 2번째 조항을 어기게 된다.

### equals만 재정의가 될 경우 일어나는 문제

```java
    Map<User, String> usersNickname = new HashMap<>();
    usersNickname.put(new User(1L, "J", 26), "BUDDY");
    String nickname = usersNickname.get(new User(1L, "J", 26));
    System.out.println(nickname); // null
```

`equals`가 재정의 되었다면, 위의 `new User(1L, "J", 26)` 와 아래의 `new User(1L, "J", 26)` 는 같은 객체다.
하지만 `hashCode` 를 재정의 하지 않아 다른 객체로 취급된다면, 위처럼 HashMap 에서 다른 객체로 취급되게 되는 것이다.

IDE가 지원하는 기능으로 `hashCode`를 쉽게 재정의 할 수 있으나, 성능이 좋지 않다. 때문에 성능에 민감하다면 직접 재정의하는 것이 좋다.
이 때, 성능을 위해서 핵심 필드를 생략하고 계산하는 경우는 없어야 한다. 속도는 빨라질 수 있으나, 해시의 품질이 나빠져 해시 테이블의 성능이 떨어질 수 있다.  
[해시코드 재정의](https://www.baeldung.com/java-hashcode)  

> equals를 재정의 할 때 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 수 있다.
> Object 명세의 규약을 지키도록 해야한다. 직접 재정의하기 어렵다면, IDE의 기능을 적극 활용하자.

---

# 일반적인 프로그래밍 원칙

## 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

자바의 데이터 타입을 크게 두 가지로 나눌 수 있다. `int`, `double`, `boolean` 등과 같은 기본 타입과 
`String`, `List` 와 같은 참조 타입이다. 각각의 기본 타입은 대응하는 참조 타입을 가지고 있다. 이를 `박싱된 기본 타입` 이라고 부른다.
오토박싱, 오토언박싱 덕분에 박싱타입과 기본 타입을 사용하는데 크게 구분이 필요하지 않지만, 분명한 차이가 존재하기 때문에 어떤 타입을 사용하는지 상당히 중요하다.

### 박싱된 기본 타입과 기본 타입의 차이

1. 기본 타입은 값만 가지고 있지만, 박싱된 기본 타입은 값과 식별성이라는 속성을 갖는다.
   - `Integer i = new Integer(42)` 와 `Integer j = new Integer(42)` 에서 i, j 는 같은 값이지만, 다른 개체라는 이야기다.
2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 `null` 을 가질 수 있다.
3. 기본 타입이 메모리 사용면에서 더 효율적이다.

### 박싱된 기본 타입의 값 비교

```java
    Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
    naturalOrder.compare(new Integer(42), new Integer(42)); // 1

    Comparator<Integer> unboxingNaturalOrder = (iBoxed, jBoxed) -> {
        int i = iBoxed;
        int j = jBoxed;
        return (i < j) ? -1 : (i == j ? 0 : 1);
    };
    unboxingNaturalOrder.compare(new Integer(42), new Integer(42)); // 0
```

`naturalOrder` 는 결과로 1을 반환한다. i < j 의 결과는 거짓이 되어 i == j 를 실행하고, `new Integer(42)` 로 생성된 
두 개의 박싱된 기본 타입은 서로 다른 객체이기 때문에 조건은 거짓으로 1이 반환된다. 이를 해결하기 위해서는 `unboxingNaturalOrder` 와 같이
언박싱 처리를 해줄 필요가 있다.

```java
public class Type {
    static Integer i;

    public static void main(String[] args) {
        if (i == 42) {
            System.out.println("같아요.");
        }
    }
}
```

여기서 결과는 i == 42 가 거짓이기 때문에 "같아요." 가 출력되지 않고 종료될 것 같지만,
실제로는 `NullPointerException` 을 던지게 된다. i의 값이 초기화 되지 않아 현재 `null` 값을 가지고 있기 때문이다.
i의 타입을 Integer 대신 int 로 선언했다면, 기본적으로 0의 값을 갖기 때문에 문제가 생기지 않는다.
또는 선언 후 값을 초기화 했다면 `i(Integer) == 42(int)` 에서 i가 오토 언박싱되어 문제가 생기지 않는다.

> 기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 기본 타입을 사용하자. 기본 타입이 더 간단하고 더 빠르다.
> 두 박싱 타입을 == 로 비교하면 식별성 비교가 이루어지기 때문에 원하지 않는 결과를 가져올 가능성이 있다.
> 오토박싱, 오토언박싱이 사용할 때의 번거로움을 줄여줄 수는 있지만 성능에 문제가 있을 수 있다.
> 기본 타입을 용도에 맞지 않게 박싱할 경우 객체를 생성하는 부작용을 초래한다.

---
