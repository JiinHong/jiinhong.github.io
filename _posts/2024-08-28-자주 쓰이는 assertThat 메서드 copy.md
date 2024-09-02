---

layout: post
title: 자주 쓰이는 assertThat 메서드
date: 2024-08-28 00:00:00 +0800
categories: [CS Study, testcode]
tags: [Sophomore, Testcode]
pin: true

---


## 자주 쓰이는 assertThat 메서드

<br>

### 1. 단순 값 비교

```java
@Test
public void testSimpleValueComparison() {
    int a = 3 + 5;
    assertThat(a).isEqualTo(8);
}
```

<br>

### 2. 문자열 비교

```java
@Test
public void testStringComparison() {
    String str = "Hello, World!";
    assertThat(str).isEqualTo("Hello, World!");
    assertThat(str).startsWith("Hello");
    assertThat(str).endsWith("World!");
    assertThat(str).contains("Hello");
    assertThat(str).isNotEmpty();
}
```

<br>

### 3. 컬렉션 검증

```java
@Test
public void testCollection() {
    List<String> name = Arrays.asList("Park", "Jin", "Hong");

    assertThat(name)
        .hasSize(3)
        .contains("Jin", "Hong")
        .doesNotContain("Lee")
        .containsExactly("Park", "Jin", "Hong")
        .containsExactlyInAnyOrder("Jin", "Hong", "Park");
}
```

<br>

### 4. 객체 필드 비교

```java
@Getter
@AllArgsConstructor
class Person {
    private String name;
    private int age;
}

@Test
public void testObjectFieldComparison() {
    Person person = new Person("Park", 24);

    assertThat(person.getName()).isEqualTo("Park");
    assertThat(person.getAge()).isEqualTo(24);

	// 람다표현식 사용하면,
    assertThat(person)
        .extracting(Person::getName, Person::getAge)
        .containsExactly("Park", 24);
}
```

<br>

### 5. 예외 검증

```java
@Test
public void testException() {
    Throwable thrown = catchThrowable(() -> {
        throw new IllegalArgumentException("Invalid argument");
    });

    assertThat(thrown)
        .isInstanceOf(IllegalArgumentException.class) // 이 부분!!
        .hasMessage("Invalid argument");

	// <참고> catchThrowable은 람다표현식이나 Callable을 인자로 받고,
	// 람다표현식에서 예외가 발생하면, 그것을 Throwable 타입으로 반환
	// 예외 발생 안하면, null 반환
}
```

<br>

### 6. 배열 검증

```java
@Test
public void testArray() {
    int[] arr = {1, 2, 3, 4, 5};

    assertThat(arr)
        .hasSize(5)
        .contains(3)
        .containsExactly(1, 2, 3, 4, 5)
        .doesNotContain(6);
}
```

<br>

### 8. 참/거짓 검증

```java
@Test
public void testBoolean() {
    boolean b = true;

    assertThat(b).isTrue();
    assertThat(!b).isFalse();
}
```

<br>

### 9. 객체 동등성 검증

```java
@Test
public void testObjectEquality() {
    Person person1 = new Person("Jane", 30);
    Person person2 = new Person("Jane", 30);

    assertThat(person1).isEqualTo(person2);
	// 다른건 isNotEqualTo로 검증!
}
```

<br>

### 10. 날짜와 시간 검증

```java
import java.time.LocalDate;
import java.time.LocalDateTime;

@Test
public void testDateTime() {
    LocalDate today = LocalDate.now();
    LocalDateTime now = LocalDateTime.now();

    assertThat(today).isBeforeOrEqualTo(LocalDate.of(2024, 12, 31));
    assertThat(now).isAfter(LocalDateTime.of(2024, 1, 1, 0, 0));
}
```
