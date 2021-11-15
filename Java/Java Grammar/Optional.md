# Optional

생성일: 2021년 9월 6일 오후 6:45

### NPE(NullPointerException)

- NPE를 피하기 위해서는 null을 검사하는 로직을 추가해야 하는데, null 검사를 해야하는 변수가 많은 경우 코드가 복잡해지고 로직이 상당히 번거롭다.

### Optional

- Optional<T> 클래스를 사용해서 NPE를 방지할 수 있도록 도와준다.
- Optional<T>는 null 이 올 수 있는 값을 감싸는 Wrapper 클래스로, 참조하더라도 NPE가 발생하지 않도록 도와준다.

### Optional 생성

```java
Optional<String> optional = Optional.empty();
optional // Optional.empty
optional.isPresent()// false

//orElse 또는 orElseGet 메서드를 이용해서 값이 없는 경우라도 안전하게 값을 가져옴
Optional<String> optional = Optional.ofNullable(getName())
String name = optional.orElse("a") // 값이 업다면 a를 리턴

```

### Optional 활용

예시1)

```java
//java8 이전
List<String> names = getNames();
List<STring> tempNames = list != null ? list : new ArrayList<>();

//Optional 사용
List<String> nameList = Optional.ofNullable(getList()).orElseGet(
() -> new ArrayList<>());

```

예시2)

```java
//java8 이전
UserVO userVO = getUser();
if (userVO != null) {
  Address address = user.getAddress();
  if (address != null) {
    String postCode = address.getPostCode();
    if (postCode != null) {
      return postCode;
    }
  }
}
return "우편번호 없음";
//아래와 같음.
Optional<UserVO> userVO = Optional.ofNullable(getUser());
Optional<Address> address = userVO.map(UserVO::getAddress());
Optional <String> postCode = address.map(Address::getPostCode);
String result = postCode.orElse("우편번호 없음");

//축약
String result = user.map(UserVO::getAddress).map(Address::getPostCode).orElse("우편 번호 없음")

```

예시3)

```java
String name = getName();
String result = "";

try {
	result = name.toUpperCase();
} catch (NullPointerException e) {
	throw new CustomUpperCaseException();
}

//아래와 같다.
Optional<String> nameOpt = Optional.ofNullable(getName());
String result = nameOpt.orElseThrow(CustomUppercaseException::new).toUpperCase();

```

Optional 정리

Optional은 값을 Wrapping하고 다시 풀고, null일 경우에는 대체하는 함수를 호출하는 등의 오버헤드가 있으므로 성능이 저하될 수 있다. 그렇기 때문에 메소드의 반환 값이 절대 null이 아니라면 Optional을 사용하지 않는 것이 성능저하가 적다. 즉, Optional은 메소드의 결과가 null이 될 수 있으며, 클라이언트가 이 상황을 처리해야 할 때 사용하는 것이 좋다.