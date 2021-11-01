# API 개발 고급 - 지연 로딩과 조회 성능 최적화1

### 조회용 샘플 데이터 입력

```java
package jpabook.jpashop;

import jpabook.jpashop.domain.*;
import jpabook.jpashop.domain.item.Book;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.PostConstruct;
import javax.persistence.EntityManager;

@Component
@RequiredArgsConstructor
public class InitDb {
    private final InitService initService;
    @PostConstruct
    public void init() {
        initService.dbInit1();
        initService.dbInit2();
    }
    @Component
    @Transactional
    @RequiredArgsConstructor
    static class InitService {
        private final EntityManager em;
        public void dbInit1() {
            Member member = createMember("userA", "서울", "1", "1111");
            em.persist(member);
            Book book1 = createBook("JPA1 BOOK", 10000, 100);
            em.persist(book1);
            Book book2 = createBook("JPA2 BOOK", 20000, 100);
            em.persist(book2);
            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 10000, 1);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 20000, 2);
            Order order = Order.createOrder(member, createDelivery(member),
                    orderItem1, orderItem2);
            em.persist(order);
        }
        public void dbInit2() {
            Member member = createMember("userB", "진주", "2", "2222");
            em.persist(member);
            Book book1 = createBook("SPRING1 BOOK", 20000, 200);
            em.persist(book1);
            Book book2 = createBook("SPRING2 BOOK", 40000, 300);
            em.persist(book2);
            Delivery delivery = createDelivery(member);
            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 20000, 3);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 40000, 4);
            Order order = Order.createOrder(member, delivery, orderItem1,
                    orderItem2);
            em.persist(order);
        }
        private Member createMember(String name, String city, String street,
                                    String zipcode) {
            Member member = new Member();
            member.setName(name);
            member.setAddress(new Address(city, street, zipcode));
            return member;
        }
        private Book createBook(String name, int price, int stockQuantity) {
            Book book = new Book();
            book.setName(name);
            book.setPrice(price);
            book.setStockQuantity(stockQuantity);
            return book;
        }
        private Delivery createDelivery(Member member) {
            Delivery delivery = new Delivery();
            delivery.setAddress(member.getAddress());
            return delivery;
        }
    }
}
```

### 간단한 주문 조회V1: 엔티티를 직접 노출

**OrderSimpleApiController**

```java
/**
 * xToOne 관계 최적화
 * Order
 * Order -> Member
 * Order -> Delivery
 */

@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;

    /**
     * V1 엔티티 직접 노출
     * - Hibernate5Module 모듈 등록, LAZY = null 처리
     * - 양방향 관계 문제 발생 -> @JsonIgnore
     */
    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        //Gson에서 순환참조 문제 발생.
        List<Order> all = orderRepository.findAllByString(new OrderSearch());

        for (Order order : all) {
            order.getMember().getName(); // lazy 강제 초기화
            order.getDelivery().getAddress();
        }
        return all;
    }

}
```

- 엔티티를 직접 노출하는 것은 좋지않다.
- order → member, order → address는 지연 로딩이다. 따라서 실제 엔티티 대신에 프록시가 존재하고 실제 값에 접근할 때 쿼리가 실행된다.
- jackson라이브러리는 기본적으로 이 프록시 객체를 json으로 어떻게 생성해야 하는지 모름 → 예외 발생
- Hibernate5Module을 스프링 빈으로 등록하면 해경

**Hidernate5Module 등록**

```java
@Bean
Hibernate5Module hibernate5Module() {
 return new Hibernate5Module();
}

//implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'

/*
Hibernate5Module hibernate5Module() {
 Hibernate5Module hibernate5Module = new Hibernate5Module();
 //강제 지연 로딩 설정
 hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING,
true);
 return hibernate5Module;
}*/
```

- 기본적으로 초기화 된 프록시 객체만 노출, 초기화 되지 않는 프록시 객체는 노출 안함
- 강제 지연로딩을 하면 양방향 연관관계를 계속 로딩하게 된다. 따라서 @JsonIgnore 옵션을 한곳에 주어야 한다.

> 참고:
엔티티를 API 응답으로 외부로 노출하는 것은 좋지 않다. 따라서 Hibernate5Module를 사용하기 보다는 DTO로 변환해서 반환하는 것이 더 좋은 방법이다.
> 

> 주의:
지연 로딩을 피하기 위해 즉시 로딩(EAGET)를 설정하면 안된다. 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다. 
항상 지연 로딩을 기본으로 하고 , 성능 최적화가 필요한 경우에는 fetch join을 사용하자.
> 

### 간단한 주문 조회V2: 엔티티를 DTO로 반환

**OrderSimpleApiController**

```java
@GetMapping("/api/v2/simple-orders")
    //원래 result로 한번 감싸야 한다.
    public List<SimpleOrderDto> ordersV2() {
        //Order 2개
        // N + 1 -> 1 + N(회원) + N(배송)
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        List<SimpleOrderDto> result = orders.stream()
                .map(SimpleOrderDto::new)
                .collect(Collectors.toList());

        return result;

    }

    @Data
    static class SimpleOrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();//Lazy 초기화
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();//Lazy 초기화
        }
    }
```

- 엔티티를 DTO로 변환하는 일반적인 방법이다.
- 쿼리가 총 1 + N + N 번 실행된다. (V1과 같은 쿼리수)
    - order 조회 1번 (order 조회 결과 수가 N이 된다.)
    - order → member 지연 로딩 조회 N번
    - order → delivery 지연 로딩 조회 N번