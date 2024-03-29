# 목차

- [객체 생성과 파괴](#객체-생성과-파괴)
  - 아이템 1. 생성자 대신 정적 패터리 메서드를 고려하라
  - 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라
  - 아이템 3. private 생성자나 열거타입으로 싱글턴임을 보증하라
  - 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
  - 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
  - 아이템 6. 불필요한 객체 생성을 피하라
  - 아이템 7. 다 쓴 객체 참조를 해제하라
  - 아이템 9. try-finally 보다는 try-with-resource 를 사용하라
- [모든 객체의 공통 메서드](#모든-객체의-공통-메서드)
  - 아이템 10. equlas는 일반 규약을 지켜 재정의하라
  - 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라
  - 아이템 12. toString을 항상 재정의하라
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

## 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

클래스가 다른 자원에 의존하는 경우가 있다. 이 때, 상황에 따라 다른 자원을 주입해 클래스를 사용하게 된다면 의존 객체 주입 방식을 사용하는 것이 좋다. 아래의 예시 코드는 맞춤법 검사 클래스에 사전을 주입하는 형태이다.

```java
class SpellChecker {
		
		private final Lexion dictionary;

		public SpellChecker(Lexion dictionary) {
			this.dictionary = dictionary;
		}
		
		...
}
```

고정적으로 하나의 사전만 사용한다면 `private static final Lexion dictionary = new KoreanDectionary()` 와 같이 정적 멤버를 사용하거나 싱글턴으로 만드는 등의 방법도 괜찮다. 하지만 위의 케이스에서는 사용하는 자원에 따라 동작이 달라지기를 원하기 때문에 의존 객체 주입을 사용한다.

의존 객체 주입을 사용하면,
1. 아이템의 유연성이 증가한다.
2. 아이템의 불변을 보장할 수 있다.
3. 여러 클라이언트가 의존 객체를 안심하고 공유할 수 있다.

`대거(Dagger)`, `주스(Guice)`, `스프링(Spring)` 과 같은 프레임워크들은 의존 객체 주입 형태로 설계되어 있다.

> 클래스 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 주는 경우 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 대신 생성자 또는 빌더에 필요한 자원을 넘기는 의존 객체 주입 방식을 사용하자. 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.

---

## 아이템 6. 불필요한 객체 생성을 피하라

똑같은 기능을 하는 객체를 여러 번 생성하기 보다 하나의 객체로 재사용하는 것이 좋다. 특히 불변 객체는 언제든 재사용이 가능하다. 

### String, Boolean에서의 재활용

1. `String s = new String("hello"); // 실행시마다 새로운 인스턴스가 생성된다.`
2. `String s = "hello"; // 여러번 실행하더라도 하나의 String 인스턴스를 사용한다.`

2번 방법은 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보
장된다. 문자열이 아닌 `Boolean`의 경우에도 `new Boolean();` 으로 생성한다면 매번 새로운 객체가 생성된다. (이 방법은 자바9에서 deprecated 됐다.) 대신 `Boolean.valueOf();` 방식으로 생성한다면, 

```java
public final class Boolean implements java.io.Serializable,  
                                      Comparable<Boolean>  
{
	public static final Boolean TRUE = new Boolean(true);  
	public static final Boolean FALSE = new Boolean(false);

	public static Boolean valueOf(String s) {
		return parseBoolean(s) ? TRUE : FALSE;
	}
}
```

매번 새로운 객체를 반환하는 것이 아니라 이미 생성된 불변 객체를 반환하는 것으로 재사용한다.

### 정규식에서의 재활용

```java
static boolean isPasswordRegex(String s) {
	return s.mathces(
	"^(?=.*[A-Za-z])(?=.*\\d)(?=.*[@$!%*#?&])[A-Za-z\\d@$!%*#?&]{8,}$");
}
```

위의 방식에는 문제점이 있는데, `String.matches()` 메서드를 사용하는 것이다. 이 메서드는 내부적으로 `Pattern` 인스턴스를 생성하고 사용한다. 한 번 사용된 `Pattern` 인스턴스는 재사용 되지 않고 버려진다. 이는 인스턴스 생성 비용이 높기 때문에 효율적이지 못하다. 때문에 이 인스턴스를 재사용 할 수 있도록 하는 것이 좋다.

```java
class Matcher {
	private static final Pattern PASSWORD = 
	"^(?=.*[A-Za-z])(?=.*\\d)(?=.*[@$!%*#?&])[A-Za-z\\d@$!%*#?&]{8,}$"

	static boolean isPasswordRegex(String s) {
		return PASSWORD.matchers(s).matches();
	}
}
```

이 방법은 성능의 개선도 눈에 띄게 좋아지지만 내부적으로만 생성되고 사용되던 `Pattern`을 밖으로 끄집어내 코드도 더 명확해지는 효과를 가지고 있다.

> 이번 아이템을 "객체 생성은 비싸니 피해야 한다."로 받아들이면 안된다. 일반적으로 객체의 생성을 피하는 것이 좋긴 하지만, 사실 요즘의 JVM에서는 단순하게 객체를 생성하고 회수하는 것으로 성능에서 큰 이슈가 되지 않는다. 객체를 재사용 하지 않더라도 성능에 조그만 차이만 있을 뿐 기능상으로는 문제가 되지 않는다. 재사용을 위해서 기능에 문제가 생긴다면, 문제가 훨씬 크다.

---

## 아이템 7. 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터의 존재로 메모리 관리에 신경을 쓰지 않아도 된다고 느껴질 수 있지만 그렇지 않다. 아래는 책에서 제시하는 `Stack` 코드이다. 

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}

	/**
	* 배열의 크기를 늘릴 필요가 있을 때 2배씩 늘린다.
	*/
	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}
}
```

사용에 문제는 없겠지만 '메모리 누수'의 문제를 가지고 있다. 이 코드의 스택에서 꺼내진 객체들은 가비지컬렉터가 회수하지 않는다. 이 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 가지고 있기 때문이다. 위의 케이스에서는 해당 참조를 다 썼을 때 null 처리를 해주는 것으로 해결이 가능하다.

```java
public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null; // 참조 해제
	return result;
}
```

객체의 참조를 해제 시키는 것 말고도 null 처리를 함으로써 추후 해당 참조를 실수로라도 사용하게 되면 `NullPointerException`이 던져지는 이점을 가지고 있다. 

>자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다. 캐시 역시 메모리 누수를 일으키는 주범이다. 

---

## 아이템 9. try-finally 보다는 try-with-resource 를 사용하라

`InputStream`, `OutputStream`, `java.sql.Connection` 등의 자원처럼 `close` 메서드를 호출하여 직접 닫아줘야 하는 경우가 많다. 이는  성능 문제가 발생할 수 있지만 놓치기 쉬운 부분이다. 따라서 `finalizer` 로 대비하기는 하나 `finalizer` 는 실행이 완전히 보장되지 않아 위험하다.

### try-finally

가장 전통적인 방법은 `try-finally` 방법이다. 

```java
static String readLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

이 방법은 나쁘지 않지만, 자원이 둘 이상으로 늘어난다면

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0) {
				out.write(buf, 0, n);
			}
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

난잡한 코드가 되어버린다. 또한 `try`, `finally` 블록 모두에서 예외가 발생할 가능성이 있는데 물리적인 문제로  두 블록에서 모두 예외가 발생한다면 두 번째 예외가 첫 번째 예외를 집어삼켜버려 스택을 추척해도 첫 번째 예외에 관한 정보가 남지 않는다.

### try-with-resource

자바7 에서는 `try-with-resource` 로 위와 같은 문제점을 해결한다. 이 구조는 `AutoCloseable` 인터페이스를 구현하는 것을 조건으로 한다. `InputStream` 과 같은 클래스에서 `Closeable` 을 구현하고 있다. (`Closeable` 은 `AutoCloseable` 을 상속한다.)

```java
public abstract class InputStream implements Closeable {
	...
}
```

`Closeable`(`AutoCloseable`) 인터페이스는 반환 값이 없는 `close()` 메서드를 정의하고 있다. 여기에 자원의 사용을 닫는 구현을 필요로 한다. 위의 코드들에 `try-with-resource` 를 적용하면,

```java
static String readLineOfFile(String path) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader(path))) {
		return br.readLine();
	}
}
```

```java
static void copy(String src, String dst) throws IOException {
	
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)){
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0) {
			out.write(buf, 0, n);
		}
	}
}
```

훨씬 간결하고 가독성이 좋은 코드가 된다. 심지어 `catch` 절을 사용할 수 있기 때문에 `try-finally` 에 비해 예외 처리에 유용하다. 또한 `try()` 안의 자원들은 메서드 종료 후 `close()` 가 자동 호출되기 때문에 자원 관리에도 효율적이다.

>회수해야 하는 자원을 다룰 때는 try-with-resource 를 사용하는 것이 좋다. 이 케이스에 예외는 없다. 코드가 더 간결하고 분명해지며, 예외 정보도 훨씬 유용하며, 자원을 회수하기도 용이하다.

---

# 모든 객체의 공통 메서드

## 아이템 10. equlas는 일반 규약을 지켜 재정의하라

>equals 메서드는 재정의 하기 쉬워보이지만, 자칫하면 실수를 하기가 쉽다. 실수를 하지 않기 위한 가장 좋은 방법은 재정의 하지 않는 것이다. 재정의 하지 않는다면 그 클래스의 인스턴스는 오직 자기 자신과만 같게된다. 그러니 다음 열거한 상황 중 하나에 해당한다면 재정의 하지 않는 것이 최선이다.

- 각 인스턴스가 본질적으로 고유하다.
- 인스턴스의 '논리적 동치성'을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 private 이거나 package-private이고 equals 메서드를 호출할 일이 없다.

또한 equals 메서드를 재정의 할 경우 Object 명세의 규약에 주의해야 한다.

### Object 명세의 규약
- 반사성: null이 아닌 모든 참조값 x에 대해 x.equals(x) 는 true다.
- 대칭성: null이 아닌 모든 참조값 x, y에 대해 x.equals(y)가 true라면, y.equals(x)도 true다.
- 추이성: null이 아닌 모든 참조값 x, y, z에 대해 x.equals(y), y.equals(z)가 true라면 z.equals(x)도 true다.
- 일관성: null이 아닌 참조값 x, y에 대해 x.equals(y)를 반복해서 호출해도 항상 true 또는 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조값 x에 대해 x.equals(null)은 false다.

### 양질의 equals 메서드 구현 방법을 위한 단계와 주의점

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
   이 때 입력 타입은 Object여야 한다! 다른 타입을 받는다면 재정의(오버라이딩)가 아닌 다중 정의(오버로딩)가 된다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 개체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.

---

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

## 아이템 12. toString을 항상 재정의하라

Object의 기본 toString 메서드는 `클래스이름@16진수_해시코드`의 형태를 가지고 있기에 원하고자 하는 정보를 반환하지 않을 확률이 높다. 

toString의 일반 규약에는,
1. '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야 한다.
2. 모든 하위 클래스에서 이 메서드를 재정의하라.
가 포함된다.

만약 `User@abcd` 라는 정보가 주어진다면, 간결하긴 하나 사람이 이해할 수 있는 유익한 정보는 아닐 것이다.
`name=kim, age=20`의 형태가 된다면 훨씬 유익한 정보를 담고 있게 될 것이다.

toString 메서드는 직접 호출하는 것 외에도 aseert 구문이나 문자열 연결 연산자(+)에 의해 자동으로 호출 될 확률이 있다. 따라서 미리 재정의하는 편이 좋으며 해당 클래스의 중요 정보들을 모두 반환하는 형태로 정의하는 것이 좋다.

### 반환값 포맷의 필요성

toString 메서드를 구현할 때 반환값의 포맷을 문서화할지 고려해야 한다.
전화번호나 행렬과 같은 값 클래스라면 정해진 형태로 문서화 하는 것이 좋다.
(toString 메서드를 통해 해당 값을 파싱해야 하는 등 데이터가 2차 가공 될 것을 고려해야 한다.)

다만, 자주 사용되는 클래스에서 포맷을 한번 명시하면 그 포맷에 얽매이게 될 수 있다. 해당 클래스를 포맷에 따라 파싱하고 새로운 데이터로 가공하여 쓰이는데, 포맷이 변경된다면 여기에 의존된 클래스들을 변경해야 할 수 있다.

포맷을 명시하지 않았다면, 하나의 포맷에 얽매이지 않고 자유롭게 수정이 가능하다.

때문에, 포맷을 명시하든 아니든 의도를 명확히 밝혀야 한다.

>상위 클래스에서 알맞은 toString 메서드를 재정의한 경우가 아니라면, 모든 구체 클래스에서 toString 메서드를 재정의 하도록 하자. 시스템을 디버깅 하기 쉬워지며, 직접 호출하지 않는 경우에도 쓰이게 될 가능성이 높다. toString 메서드는 해당 객체에 관한 핵심 정보를 이해하기 쉽고 가공하기 좋은 형태로 반환해야 한다.

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
