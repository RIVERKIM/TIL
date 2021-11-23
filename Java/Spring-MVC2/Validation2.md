# Validation2

### BindingResult2

- 스프링이 제공하는 검증 오류를 보관하는 객체
- BindingResult가 있으면 @ModelAttribute에 데이터 바인딩 시 오류가 발생해도 컨트롤러가 호출된다.
- @ModelAttribute에 바인딩 시 타입 오류가 발생하면
    - BindingResult가 없으면 → 400 오류가 발생하고 컨트롤러가 호출되지 않고, 오류페이지로 이동
    - BindingResult가 있으면 → 오류 정보(FieldError)를 BindingResult에 담아서 컨트롤러를 정상 호출.
    - 단, 검증할 대상 바로 다음에 BindingResult가 인수로 주어져야 한다.

**BindingResult에 검증 오류 적용 방법**

- @ModelAttribute에 객체에 타입 오류 등으로 바인딩이 실패하는 경우 스프링이 FieldError 생성해서 BindingResult에 넣어준다.
- 개발자가 직접 넣어준다.
- Validator 사용

### **FieldError, ObjectError**

**ValidationItemControllerV2**

```java
@PostMapping("/add")
public String addItemV2(@ModelAttribute Item item, BindingResult bindingResult,
RedirectAttributes redirectAttributes) {
 if (!StringUtils.hasText(item.getItemName())) {
 bindingResult.addError(new FieldError("item", "itemName",
item.getItemName(), false, null, null, "상품 이름은 필수입니다."));
 }
 if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() >
1000000) {
 bindingResult.addError(new FieldError("item", "price", item.getPrice(),
false, null, null, "가격은 1,000 ~ 1,000,000 까지 허용합니다."));
 }
 if (item.getQuantity() == null || item.getQuantity() > 10000) {
 bindingResult.addError(new FieldError("item", "quantity",
item.getQuantity(), false, null, null, "수량은 최대 9,999 까지 허용합니다."));
 }
 //특정 필드 예외가 아닌 전체 예외
 if (item.getPrice() != null && item.getQuantity() != null) {
 int resultPrice = item.getPrice() * item.getQuantity();
 if (resultPrice < 10000) {
 bindingResult.addError(new ObjectError("item", null, null, "가격 *
수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
 }
 }
 if (bindingResult.hasErrors()) {
 log.info("errors={}", bindingResult);
 return "validation/v2/addForm";
 }
 //성공 로직
 Item savedItem = itemRepository.save(item);
 redirectAttributes.addAttribute("itemId", savedItem.getId());
 redirectAttributes.addAttribute("status", true);
 return "redirect:/validation/v2/items/{itemId}";
}
```

**FieldError 생성자**

```java
public FieldError(String objectName, String field, String defaultMessage);
public FieldError(String objectName, String field, @Nullable Object
rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
Object[] arguments, @Nullable String defaultMessage)
```

- objectName: 오류가 발생한 객체 이름
- field: 오류 필드
- rejectedValue: 사용자가 입력한 값 (거절된 값)
- bindingFailure: 타입 오류가 같은 바인딩 실패인지, 검증 실패인지 구분 값.
    - 여기서는 바인딩 실패가 아니기 때문에 false
- codes: 메시지 코드
- arguments: 메시지를 사용하는 인자
- defaultMessage: 기본 오류 메시지

**오류 발생시 사용자 입력 값 유지**

```java
new FieldError("item", "price", item.getPrice(), 
false, null, null, "가격은 1,000 ~1,000,000 까지 허용합니다.")
```

**타임리프의 사용자 입력 값 유지**

```java
th:field="*{price}"
```

- th:field는 정상 상황에는 모델 객체의 값을 사용하지만, 오류가 발생하면, FieldError에서 보관한 값을 사용해서 값을 출력한다.

### 오류 코드와 메시지 처리1

- FieldError, ObjectError의 생성자는 errorCode, arguments를 제공한다. 이것은 오류 발생시 오류 코드로 메시지를 찾기 위함.

**errors 메시지 파일 생성**

- [application.properties](http://application.properties) 다음과 같이 수정
    
    ```java
    spring.messages.basename=messages,errors
    ```
    
- [errors.properties](http://errors.properties) 라는 파일 생성

```java
required.item.itemName=상품 이름은 필수입니다.
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```

- {0}, {1}의 값은 arguments 필드에 들어간 값들이다.

```java
new FieldError("item", "price", item.getPrice(), false, new String[]
{"range.item.price"}, new Object[]{1000, 1000000}
```

- codes: required:item.itemName를 사용해서 메시지 코드를 지정
- arguments: Object[]{1000, 1000000} 를 사용해서 코드의 {0}, {1} 로 치환

### 오류 코드와 메시지 처리2

- 컨트롤러에서 BindingResult는 검증해야 할 객체인 target바로 다음에 온다. 따라서 BindingResult는 이미 본인이 검증해야 할 객체인 target을 알고 있음.

```java
log.info("objectName={}", bindingResult.getObjectName());
log.info("target={}", bindingResult.getTarget());

//objectName=item //@ModelAttribute name
//target=Item(id=null, itemName=상품, price=100, quantity=1234)
```

**rejectValue(), reject()**

- FieldError(), ObjectError를 사용하지 않고 검증 오류를 다룰 수 있음.

```java
void rejectValue(@Nullable String field, String errorCode,
@Nullable Object[] errorArgs, @Nullable String defaultMessage);

bindingResult.rejectValue("itemName", "required");

void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String
defaultMessage);

bindingResult.reject("totalPriceMin", new Object[]{10000,
resultPrice}, null);
```

### 오류 코드와 메시지 처리3

- 오류 코드 만들 때 방식
    - required.item.itemName - 자세하게
    - required: 단순하게

```java
//우선 순위 다르게 조정
#Level1
required.item.itemName: 상품 이름은 필수 입니다.
#Level2
required: 필수 값 입니다.
```

- 스프링은 MessageCodesResolver라는 것으로 이런 기능을 지원한다.