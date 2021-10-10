# 스프링MVC 웹 페이지

**상품 도메인 모델**

- 상품 ID,
- 상품 명
- 상품 가격
- 상품 수량

**상품 관리 기능**

- 상품 목록
- 상품 상세
- 상품 등록
- 상품 수정

**서비스 제공 흐름**

![Untitled](%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BCMVC%20%E1%84%8B%E1%85%B0%E1%86%B8%20%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%8C%E1%85%B5%2039b1d93497984e5cb758f67f5be786f7/Untitled.png)

### 상품 도메인 개발

**Item - 상품 객체**

```java
package hello.itemservice.domain.item;

import lombok.Data;

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
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

**ItemRepository - 상품 저장소**

```java
@Repository
public class ItemRepository {

    private static final Map<Long, Item> store = new ConcurrentHashMap<>(); //static
    private static AtomicLong sequence = new AtomicLong(0); //static

    public Item save(Item item) {
        item.setId(sequence.getAndIncrement());
        store.put(item.getId(), item);
        return item;
    }

    public Item findById(Long id) {
        return store.get(id);
    }

    public List<Item> findAll() {
        return new ArrayList<>(store.values()); // array할당 store 값 수정 없이 진행 가능
    }

    // 정석은 ItemDTO를 만들고 수정할 필요 있는 값만 전달하는게 맞다.
    public void update(Long itemId, Item updateParam) {
        Item findItem = findById(itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    public void clearStore() {
        store.clear();
    }
}
```

**ItemRepositoryTest -상품 저장소 테스트**

```java
class ItemRepositoryTest {

    ItemRepository itemRepository = new ItemRepository();

    @AfterEach
    void afterEach() {
        itemRepository.clearStore();
    }

    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);
        //when
        Item savedItem = itemRepository.save(item);
        //then
        Item findItem = itemRepository.findById(item.getId());
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void findAll() {
        //given
        Item item1 = new Item("itemA", 10000, 10);
        Item item2 = new Item("itemB", 10000, 10);

        itemRepository.save(item1);
        itemRepository.save(item2);
        //when
        List<Item> result = itemRepository.findAll();
        //then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(item1, item2);
    }

    @Test
    void updateItem() {
        //given
        Item item = new Item("itemA", 10000, 10);
        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();

        //when
        Item updateParam = new Item("itemB", 10000, 10);
        itemRepository.update(itemId, updateParam);

        //then
        Item findItem = itemRepository.findById(itemId);

        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());

    }
}
```

### 상품 목록 - 타임 리프

**BasicItem - Controller**

```java
@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }

    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init() {
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));

    }
}
```

- @PostConstruct: 해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다.
- 여기서는 간단히 테스트용 데이터를 넣기 위해서 사용했다.

**/resources/template/basic/items.html**

```java
<!DOCTYPE HTML>
<html xmlns:th = "http://www.thymeleaf.org">
<head>
  <meta charset="utf-8">
  <link th:href="@{/css/bootstrap.min.css}" href="../css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container" style="max-width: 600px">
  <div class="py-5 text-center">
    <h2>상품 목록</h2>
  </div>
  <div class="row">
    <div class="col">
      <button class="btn btn-primary float-end"
              onclick="location.href='addForm.html'"
              th:onclick="|location.href=`@{/basic/items/add}`|" type="button">상품
        등록</button>
    </div>
  </div>
  <hr class="my-4">
  <div>
    <table class="table">
      <thead>
      <tr>
        <th>ID</th>
        <th>상품명</th>
        <th>가격</th>
        <th>수량</th>
      </tr>
      </thead>
                <tbody>

      <tr th:each="item : ${items}">
        <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.id}">회원id</a></td>
        <td><a href="item.html" th:href="@{/basic/items/{itemId}(itemId=${item.id})}" th:text="${item.itemName}">상품명</a></td>
        <td th:text="${item.price}">10000</td>
        <td th:text="${item.quantity}">10</td>
      </tr>

      </tbody>
    </table>
  </div>
</div> <!-- /container -->
</body>
</html>
```

### 상품 상세

**BasicItemController에 추가**

```java
package hello.itemservice.web.basic;

import hello.itemservice.domain.item.Item;
import hello.itemservice.domain.item.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import javax.annotation.PostConstruct;
import java.util.List;

@Controller
@RequestMapping("/basic/items")
@RequiredArgsConstructor
public class BasicController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "basic/items";
    }

    @GetMapping("/{itemId}")
    public String Item(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById((itemId));
        model.addAttribute("item", item);
        return "/basic/item";
    }

    @GetMapping("/add")
    public String addForm() {
        return "basic/addForm";
    }

//    @PostMapping("/add")
//    public String addItemV1(@RequestParam String itemName,
//                       @RequestParam int price,
//                       @RequestParam Integer quantity,
//                       Model model) {
//
//        Item item = new Item();
//        item.setItemName(itemName);
//        item.setPrice(price);
//        item.setQuantity(quantity);
//
//        itemRepository.save(item);
//
//        model.addAttribute("item", item);
//
//        return "basic/item";
//    }

//    @PostMapping("/add")
//    public String addItemV2(@ModelAttribute("item") Item item,
//                       Model model) {
//        itemRepository.save(item);
//
//        //ModelAttribute에서 지정한 이름으로 이미 model에 값이 담겨져 있다.
//        //model.addAttribute("item", item);
//
//        return "basic/item";
//    }

    //객체 타입의 첫 글자만 소문자로 바꾼 이름이 default 이름
    // 이것을 model에 넣어준다.
//    @PostMapping("/add")
//    public String addItemV3(@ModelAttribute Item item, Model model) {
//        itemRepository.save(item);
//
//        return "basic/item";
//    }
//
//    @PostMapping("/add")
//    public String addItemV3(Item item) {
//        itemRepository.save(item);
//
//        return "basic/item";
//    }

//    @PostMapping("/add")
//    public String adItemV5(Item item) {
//        itemRepository.save(item);
    //url은 인코딩해서 붙여줘야 하는 문제가 있다.
//        return "redirect:/basic/items/" + item.getId();
//    }

    @PostMapping("/add")
    public String adItemV6(Item item, RedirectAttributes redirectAttributes) {
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
    }

    @GetMapping("/{itemId}/edit")
    public String getItem(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "basic/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
        itemRepository.update(itemId, item);
        return "redirect:/basic/items/{itemId}";
    }

    /**
     * 테스트용 데이터 추가
     */
    @PostConstruct
    public void init() {

        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));

    }
}
```

- @ModelAttribute는 Item 객체를 생성하고, 모델에 지정한 객체를 자동으로 넣어준다.
- @ModelAttribute 이름 생략하면 모델에 저장될 때 클래스 명에서 첫글자만 소문자로 변경한 후 저장한다.
- redirect: /basic/items/{itemId}
    - 컨트롤러에 매핑된 @PathVariable의 값은 redirect에도 사용할 수 있다.
    - redirect:/basic/items/{itemId}의 {itemId}는 @PathVariable Long itemId의 값을 그대로 사용한다.

### PRG Post/Redirect/Get

**POST 등록 후 새로고침**

- 상품 등록 폼에서 데이터를 입력하고 저장을 선택하면 POST /add + 상품 데이터를 서버에 전송
- 이 상태에서 새로 고침을 하면 마지막에 전송한 데이터가 서버로 다시 전송된다.
- 새로 고침 문제를 해결하려면 상품 저장 후에 뷰 템플릿으로 이동하는 것이 아니라, 상품 상세 화면으로 리다이렉트를 호출해주면 된다.

![Untitled](%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BCMVC%20%E1%84%8B%E1%85%B0%E1%86%B8%20%E1%84%91%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%8C%E1%85%B5%2039b1d93497984e5cb758f67f5be786f7/Untitled%201.png)

**RedirectAttributes**

- URL 인코딩을 해주고, pathVariable, 쿼리 파라미터까지 처리해준다.
- redirect: /basic/items/{itemId}
    - pathVariable 바인딩: {itemId}
    - 나머지는 쿼리 파라미터로 처리: ?status=true

```java
@PostMapping("/add")
    public String adItemV6(Item item, RedirectAttributes redirectAttributes) {
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/basic/items/{itemId}";
    }
```

> HTML Form 전송은 PUT, PATCH를 지원하지 않는다. GET, POST만 사용할 수 있다.
>