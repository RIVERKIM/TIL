# Assertions

생성일: 2021년 11월 19일 오후 2:35

### Assertions

- `org.junit.jupiter.api.Assertions`

```java
//동등 비교
assertEquals(예측값, 실제값); -> assertEquals(2, add(1 + 2));

// 참 거짓 판단.
assertTrue(boolean condition, String message); 
-> assertTrue(isOdd(2), "is even"); 
assertFalse(boolean condition, String message);

// NUll 판단
assertNotNull(Object object, String message)
-> assertNotNull(Member, "is null");
assertNull(Object object, String message)

// 여러개의 assert 동시 수행
assertAll(Excutable...) 
-> assertAll("test",
() -> assertFalse(true, "Exception"),
() -> {
	Member member = null;
	assertNotNull(tObj, "Object is null");
})

//예외 발생
assertThrows(Exception exception, Executable executable);
->
assertThrows(ArithmeticException.class, () -> 3 / 0);

//시간 초과
assertTimeout(Duration timeout, Executable executable);
->
assertTimeout(Duration.ofMillis(100), () -> {
		Thread.sleep(400);
	}
)
```

**Assertion**

- AssertJ

```java
assertThat(T actual, Matcher<? super T> matcher);
-> 
assertThat(3).isEqualTo(3);
// assertThat(예측값) 에 .로 자동완성이 많이 나오니 그것 보고 사용하면 됨
// 많은 함수 지원, 홀 짝, 크다, 작다 등등
```