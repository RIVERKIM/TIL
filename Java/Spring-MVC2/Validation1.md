# Validation1

- 컨트롤러의 중요한 역할 중 하나는 HTTP 요청이 정상인지 검증하는 것이다.

> 참고: 클라이언트 검증, 서버 검증
> 
- 클라이언트 검증은 조작할 수 있으므로 보안에 취약한다.
- 서버만으로 검증하면, 즉각적인 고객 사용성이 부족해진다.
- 둘을 적절히 섞어서 사용하되, 최종적으로 서버 검증은 필수
- API 방식을 사용하면 API 스펙을 잘 정의해서 검증 오류를 API 응답 결과에 잘 남겨주어야 함.

**검증 직접 처리**

![Untitled](Validation1%2069a57ce864894ce495d68dd1b735956a/Untitled.png)

![Untitled](Validation1%2069a57ce864894ce495d68dd1b735956a/Untitled%201.png)

- 검증이 실패한 다시 폼을 보여주고, 어떤 값이 잘못 되었는지 알려주어야 한다.

### 검증 직접 처리

**ValidationItemV1Controller**

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes
redirectAttributes, Model model) {

 //검증 오류 결과를 보관
 Map<String, String> errors = new HashMap<>();
 //검증 로직
 if (!StringUtils.hasText(item.getItemName())) {
 errors.put("itemName", "상품 이름은 필수입니다.");
 }
 if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() >
1000000) {
 errors.put("price", "가격은 1,000 ~ 1,000,000 까지 허용합니다.");
 }
 if (item.getQuantity() == null || item.getQuantity() >= 9999) {
 errors.put("quantity", "수량은 최대 9,999 까지 허용합니다.");
 }
 //특정 필드가 아닌 복합 룰 검증
 if (item.getPrice() != null && item.getQuantity() != null) {
 int resultPrice = item.getPrice() * item.getQuantity();
 if (resultPrice < 10000) {
 errors.put("globalError", "가격 * 수량의 합은 10,000원 이상이어야 합니다.
현재 값 = " + resultPrice);
 }
 }
 //검증에 실패하면 다시 입력 폼으로
 if (!errors.isEmpty()) {
 model.addAttribute("errors", errors);
 return "validation/v1/addForm";
 }
 //성공 로직
 Item savedItem = itemRepository.save(item);
 redirectAttributes.addAttribute("itemId", savedItem.getId());
 redirectAttributes.addAttribute("status", true);
 return "redirect:/validation/v1/items/{itemId}";
}
```

- 검증 오류 보관
    - Map<String, String> errors = new HashMap<>()
    - 만약 검증시 오류 발생하면 어떤 검증에서 오류가 발생했는지 정보를 담아둔다.

**addForm.html**

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
        .field-error {
            border-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>

<div class="container">

    <div class="py-5 text-center">
        <h2 th:text="#{page.addItem}">상품 등록</h2>
    </div>

    <form action="item.html" th:action th:object="${item}" method="post">

        <div th:if="${errors.containsKey('globalError')}">
            <p class="field-error" th:text="${errors['globalError']}">전체 오류 메시지</p>
        </div>

        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}"
                   th:class="${errors?.containsKey('itemName')} ? 'form-control field-error' : 'form-control'"
                   class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:if="${erro0rs.containsKey('itemName')}" th:text="${errors['itemName']}"></div>
        </div>
        <div>
            <label for="price" th:text="#{label.item.price}">가격</label>
            <input type="text" id="price" th:field="*{price}"
                   th:class="${errors?.containsKey('price')} ? 'form-control field-error' : 'form-control'"
                   class="form-control" placeholder="가격을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('price')}" th:text="${errors['price']}">
                가격 오류
            </div>
        </div>

        <div>
            <label for="quantity" th:text="#{label.item.quantity}">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}"
                   th:class="${errors?.containsKey('quantity')} ? 'form-control field-error' : 'form-control'"
                   class="form-control" placeholder="수량을 입력하세요">
            <div class="field-error" th:if="${errors?.containsKey('quantity')}" th:text="${errors['quantity']}">
                수량 오류
            </div>

        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit" th:text="#{button.save}">상품 등록</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/validation/v1/items}'|"
                        type="button" th:text="#{button.cancel}">취소</button>
            </div>
        </div>

    </form>

</div> <!-- /container -->
</body>
</html>
```

- Safe Navigation Operator "?"
    - errors? → errors 가 null일 떄 NullPointerException이 발생하는 대신 null을 반환하는 문법.
- 위와 같은 방법으로 처리하면, 오류가 발생해서 폼을 다시 보여줄 때에도 기존에 입력한 값이 유지될 수 있다.

**문제점**

- 타입 처리 미흡 - 숫자 타입에 문자가 들어오면 오류가 발생하나 이러한 오류는 스프링 MVC 진입전에 예외가 발생하기에 Controller가 호출되지 못하고 400 예외가 발생한다.
- Item처럼 Integer값에 문자를 넣은 경우, 값을 보존할 수 없다.
- 결국 입력한 값이 어딘가에 별도로 관리가 되어야 한다.

### BindingResult1

- 스프링이 검증 오류 처리 방법

**ValidationItemControllerV2**

```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult,
RedirectAttributes redirectAttributes) {
 if (!StringUtils.hasText(item.getItemName())) {
 bindingResult.addError(new FieldError("item", "itemName", "상품 이름은
필수입니다."));
 }
 if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() >
1000000) {
 bindingResult.addError(new FieldError("item", "price", "가격은 1,000 ~
1,000,000 까지 허용합니다."));
 }
 if (item.getQuantity() == null || item.getQuantity() > 10000) {
 bindingResult.addError(new FieldError("item", "quantity", "수량은 최대
9,999 까지 허용합니다."));
 }
 //특정 필드 예외가 아닌 전체 예외
 if (item.getPrice() != null && item.getQuantity() != null) {
 int resultPrice = item.getPrice() * item.getQuantity();
 if (resultPrice < 10000) {
 bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은
10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
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

- BindingResult에 들어간 값은 자동으로 모델에 들어간다.
- FieldError
    - 필드에 오류가 있으면 FieldError객체를 생성해서 bindingResult에 담아두면 된다.
    - 특정 필드를 넘어서는 오류는 ObjectError객체를 생성해서 담아두면 된다.

```java
public FieldError(String objectName, String field, String defaultMessage) {}
```

**addForm.html**

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
        .field-error {
            border-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>

<div class="container">

    <div class="py-5 text-center">
        <h2 th:text="#{page.addItem}">상품 등록</h2>
    </div>

    <form action="item.html" th:action th:object="${item}" method="post">
        <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p>
        </div>

        <div>
            <label for="itemName" th:text="#{label.item.itemName}">상품명</label>
            <input type="text" id="itemName" th:field="*{itemName}"
                   th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
            <div class="field-error" th:errors="*{itemName}">
                상품명 오류
            </div>
        </div>
        <div>
            <label for="price" th:text="#{label.item.price}">가격</label>
            <input type="text" id="price" th:field="*{price}"
                   th:errorclass="field-error" class="form-control" placeholder="가격을 입력하세요">
            <div class="field-error" th:errors="*{price}">
                가격 오류
            </div>
        </div>

        <div>
            <label for="quantity" th:text="#{label.item.quantity}">수량</label>
            <input type="text" id="quantity" th:field="*{quantity}"
                   th:errorclass="field-error" class="form-control" placeholder="수량을 입력하세요">
            <div class="field-error" th:errors="*{quantity}">
                수량 오류
            </div>

        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit" th:text="#{button.save}">상품 등록</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/validation/v2/items}'|"
                        type="button" th:text="#{button.cancel}">취소</button>
            </div>
        </div>

    </form>

</div> <!-- /container -->
</body>
</html>
```

- 타임리프는 스프링의 bindingResult를 활용해서 편리하게 검증 오류를 표현하는 기능 제공
- #fields: #fields로 BindingResult가 제공하는 검증 오류 접근 가능
- th:erros: 해당 필드에 오류가 있는 경우에 태그를 출력.
- th:errorclass: th:field 에서 지정한 필드에 오류가 있으면 class 정보를 추가.