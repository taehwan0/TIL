# JavaEnum

>열거형(enum)은 서로 관련된 상수를 편리하게 선언하기 위한 것으로 여러 상수를 정의할 때 사용하면 유용하다.
> 자바의 열거형은 C언어의 열거형보다 더 향상된 것으로 열거형이 갖는 값 뿐만 아니라 타입도 관리하기 때문에 보다
> 논리적인 오류를 줄일 수 있다.
> <br><br> ***자바의정석3판 - ch12***

## enum 으로 상수 정의하기 

enum 은 하나의 클래스다. 고로, 클래스의 선언 방법과 유사하다.  
가장 기본적인 상수 정의 방법은 아래와 같다.

```java
enum Role {
    ROLE_ADMIN,
    ROLE_MANAGER,
    ROLE_GUEST
}
```

## enum 멤버 추가하기

enum 은 멤버를 추가할 수 있다. 아래와 같이 열거형 상수의 이름 옆의 `()` 에 멤버를 나열한다.
하나의 열거형 상수를 정의하고 나서 `,` 이후 또 다른 열거형 상수를 정의하며 마지막 상수를 추가한 뒤에는 
세미콜론(`;`)을 추가한다. 마찬가지로 열거형도 클래스이기 때문에 열거형에 나열된 멤버들에 맞게 변수들을
선언해야 하며, 메서드를 가질 수도 있다.

enum 의 생성자는 기본적으로 `private` 이다. 이는 상수들의 집합인 enum 은 런타임이 아닌 컴파일타임에
모든 값을 알고 있어야 하기 때문에, enum 인스턴스 사용을 제어해 싱글톤으로 관리하기 위함이다.  
(https://www.nextree.co.kr/p11686/)

```java
enum Role {
    ROLE_ADMIN("ADMIN", 1),
    ROLE_MANAGER("MANAGER", 2),
    ROLE_GUEST("GUEST", 3),
    ;
    
    private final String value;
    private final int accessLevel;
    
    Role(String value, int accessLevel) {
        this.value = value;
        this.accessLevel = accessLevel;
    }
}
```

## enum 적용 예시

기존 코드에는 아래와 같이 `User` 클래스에서 권한인 `ROLE`을 저장하기 위해
`static final` 로 상수를 선언하여 사용하였다. 권한의 종류가 더 늘어난다면,
`User` 클래스의 생성자 또는 메서드의 수정이 필요하게 될 것이다.
```java
class User {

    private static final String ROLE_ADMIN = "ADMIN";
    private static final String ROLE_MANAGER = "MANAGER";
    private static final String ROLE_GUEST = "GUEST";
    
    private String role;
    
    public User(String role) {
        if (role.equals(ROLE_ADMIN)) {
            this.role = role;
        } 
        ...
    }
}
```

권한을 enum 으로 추출하여 파일을 분리했다. `User` 클래스의 코드도 짧아지고 깔끔해졌다.
권한의 추가에 따라 `User` 클래스의 생성자 또는 메서드가 수정될 일도 없을 것이다.
그저 enum `Role` 에 타입을 추가하기만 하면 된다.

```java
// User.java
class User {
    private Role role;
    
    public User(Role role) {
        this.role = role;
    }
    
    public User(String role) {
        this.role = Role.valueOf(role);
    }
}

// Role.java
enum Role {
    ROLE_ADMIN("ADMIN"),
    ROLE_MANAGER("MANAGER"),
    ROLE_GUEST("GUEST"),
    ;
    
    private final String value;
    ...
}
```

