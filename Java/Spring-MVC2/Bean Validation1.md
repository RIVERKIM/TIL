# Bean Validation1

### Bean Validation이란

- 특정 구현체가 아니라 Bean Validation 2.0이라는 기술 표준
- 일반적으로 구현체로 하이버네이트 Validator 사용. [링크](http://hibernate.org/validator/)

**의존관계 추가**

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

**예시**

```java
@Data
public class Item {
	 private Long id;
	 @NotBlank
	 private String itemName;
	 @NotNull
	 @Range(min = 1000, max = 1000000)
	 private Integer price;
	 @NotNull
	 @Max(9999)
	 private Integer quantity;
	 public Item() {
	 }
	 public Item(String itemName, Integer price, Integer quantity) {
	 this.itemName = itemName;
	 this.price = price;
	 this.quantity = quantity;
	 }
}
```

- @NotBlank: 빈값  + 공백만 있는 경우 허용 x
- @NotNull: null을 허용 x
- @Range(min = 1000, max =10000000): 범위 안의 값
- @Max(999): 최대 999만 허용

> javax.validation으로 시작하는 라이브러리는 구현에 관계없이 제공되는 표준 인터페이스이고, org.hibernate.validator로 시작하면 하이버네이트 validator 구현체를 사용할 때만 제공되는 기능
> 

**BeanValidationTest**

```java
@Test
    void beanValidation() {
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        Validator validator = validatorFactory.getValidator();

        Item item = new Item();
        item.setItemName(" ");
        item.setPrice(0);
        item.setQuantity(100000);

        Set<ConstraintViolation<Item>> violations = validator.validate(item);
        for (ConstraintViolation<Item> violation : violations) {
            System.out.println("violations = " + violation);
            System.out.println("violation.getMessage() = " + violation.getMessage());

        }
    }
```

- 코드는 참고만,
- 검증 결과는 검증 오류가 담긴다.

**Spring BeanValidation 적용**

```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult,
RedirectAttributes redirectAttributes) {
	if(bindingResult.hasError()) {
		log.info("errors = {}", bindingResult);
		return "validation/v3/addForm";
	}

	Item savedItem = itemRepository.save(item);
	redirectAttributes.addAttributes("itemId", savedItem.getId());
	redirectAttributes.addAttributes("status", true);
	return "redirect:/validation/v3/items/{itemId}";
}
```

- 검증 시 @Validated (스프링 전용), @Valid(자바 표준) 둘을 사용해서 검증
- 검증 순서
    - @ModelAttribute 각각의 필드 타입 변환 시도
        - 성공하면 다음으로
        - 실패하면 typeMismatch로 FieldError 추가
    - Validator 적용
- 바인딩에 성공한 필드만 Bean Validation 적용
    - BeanValidator는 바인딩에 실패한 필드는 BeanValidation을 적용하지 않음.

### Bean Validator

- 스프링 부트가 spring-boot-starter-validation 라이브러리를 넣으면 자동으로 Bean Validator를 인지하고 스프링에 통합
- 자동으로 LocalValidatorFactoryBean을 글로벌 Validator등록
- 검증 오류가 발생하면, FieldError, ObjectError를 생성해서 BindingResult에 담아준다.
- 글로벌 Validtor를 직접 등록하면 스프링 부트는 Bean Validator를 글로번 Validator로 등록하지 않는다.

```java
@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {
 // 글로벌 검증기 추가
	@Override
	public Validator getValidator() {
		return new ItemValidator();
	}
 // ...
}
```

### Bean Validation  에러 코드

- Bean Validation이 제공하는 오류 메시지를 변경할 수 있다
- 오류 코드를 기반으로 MessageCodesResolver를 통해 다양한 메시지가 순서대로 생성된다.

**@NotBlank**

- NotBlank.item.itemName
- NotBlank.itemName
- NotBlnak.java.lang.String
- NotBlank

**메시지 등록**

- errors.properties

```java
#Bean Validation 추가
NotBlank={0} 공백X
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}

#{0}필드명, {1} ~은 애노테이션 마다 다름.
```

**BeanValidation 메시지 찾는 순서**

- 생성된 메시지 코드 순서대로 messageSource에서 메시지 찾기
- 애노테이션의 message 속성 사용 → @NotBlank(message = “공백! {0}”)
- 라이브러리가 제공하는 기본 값 사용 → 공백일 수 없습니다.

### Bean Validation 오브젝트 오류

- @ScriptAssert를 사용하는 것보다 오브젝트 관련 부분만 직접 자바 코드로 작성

```java
if (item.getPrice() != null && item.getQuantity() != null) {
	 int resultPrice = item.getPrice() * item.getQuantity();
	 if (resultPrice < 10000) {
	 bindingResult.reject("totalPriceMin", new Object[]{10000,
	resultPrice}, null);
 }
 }

if (bindingResult.hasErrors()) {
	 log.info("errors={}", bindingResult);
	 return "validation/v3/addForm";
 }
```