# API 개발 고급 - 컬렉션 조회 최적화2

### 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

- 컬렉션을 페치 조인하면 페이징이 불가능하다.
    - 컬렉션을 페치 조인하면 일대다 조인이 발생하므로 데이터가 예측할 수 없이 증가하다.
    - 일대다에서 일을 기준으로 페이징을 하는 것이 목적이다. 그런데 데이터는 다를 기준으로 row가 생성된다.
    - Order를 기준으로 페이징을 하고 싶은데, 다인 OrderItem을 조인하면 OrderItem이 기준이 되어버린다.
- 이경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다.

**한계 돌파 - 페이징 + 컬렉션 엔티티를 함께 조회하는 방법**

- 먼저 ToOne 관계를 모두 페치 조인한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
- 컬렉션은 지연 로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 hibernate.default_batch_fetch_size, @BatchSize를 적용한다.
- @BatchSize: 개별 최적화
- 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size만큼 IN 쿼리로 조회한다.

**OrderRespository에 추가**

```jsx
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                "select distinct o from Order o" +
                "join fetch o.member m" +
                "join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
```

**OrderApiController에 추가**

```jsx

@GetMapping("/api/v3.1/orders")
    public List<OrderDto> ordersV3_page(@RequestParam(value = "offset", defaultValue = "0") int offset,
                                        @RequestParam(value = "limit", defaultValue = "0") int limit) {
        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);

        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        return result;
    }
```

- ToOne 관계만 우선 페치 조인으로 최적화
- 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화

**최적화 옵션**

```jsx
spring:
 jpa:
   properties:
     hibernate:
       default_batch_fetch_size: 1000
```

- 개별로 설정하려면 @BatchSize를 적용하면 된다. (컬렉션은 필드에, 엔티티는 엔티티 클래스에 적용한다.)

**장점**

- 쿼리 호출수가 1 + N → 1 + 1로 최적화 된다.
- 조인보다 DB 데이터 전송량이 최적화 된다.(Order와 OrderItem을 조인하면 Order가 OrderItem만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)
- 페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
- 컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.

**결론**

- ToOne관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne관계는 페치조인으로 쿼리 수를 줄이고, 나머지는 hibernate.default_batch_fetch_size로 최적화 하자.

> 참고: default_batch_fetch_size 의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것을 권장한다. 이 전략을 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를 1000으로 제한하기도 한다. 1000으로 잡으면 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB
에 순간 부하가 증가할 수 있다. 하지만 애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩해야 하므로 메모리 사용량이 같다. 1000으로 설정하는 것이 성능상 가장 좋지만, 결국 DB든 애플리케이션이든 순간 부하를 어디까지 견딜 수 있는지로 결정하면 된다
> 

### 주문 조회V4: JPA에서 DTO 직접 조회

**OrderApiController**

```jsx
@GetMapping("/api/v4/orders")
    public List<OrderQueryDto> ordersV4() {
        return orderQueryRepository.findOrderQueryDtos();
    }
```

**OrderQueryRepository**

```jsx
package jpabook.jpashop.repository.order.query;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import java.util.List;

@RequiredArgsConstructor
@Repository
public class OrderQueryRepository {
   private final EntityManager em;

    public List<OrderQueryDto> findOrderQueryDtos() {
        List<OrderQueryDto> result = findOrders();

        result.forEach(o -> {
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });

        return result;
    }

    private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.ItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count) from OrderItem oi" +
                        "join oi.item i" +
                        "where oi.order.id = :orderId", OrderItemQueryDto.class
                ).setParameter("orderId", orderId)
                .getResultList();
    }

    public List<OrderQueryDto> findOrders() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address) from Order o" +
                        "join o.member o" +
                        "join o.delivery d", OrderQueryDto.class)
                .getResultList();
    }
}
```

- OrderRepository와 분리 이유는 이 Repository 로직이 API 종속적이기에 따로 빼서 DTO들과 관리한다.

**OrderQueryDto**

```jsx
@Data
public class OrderQueryDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.orderItems = orderItems;
    }
}
```

**OrderItemQueryDto**

```jsx
@Data
public class OrderItemQueryDto {
    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

- Query: 루트 1번, 컬렉션 N번 실행
- toOne(N:1, 1:1)관계들을 먼저 조회하고, ToMany(1: N) 관계는 각각 별도로 처리한다.
    - ToOne관계는 조인해도 데이터 row수가 증가하지 않는다.
    - ToMany관계는 조인하면 row수가 증가한다.
- row 수가 증가하지 않는 ToOne관계는 조인으로 최적화 하기 쉬우므로 한번에 조회하고, ToMany관계는 최적화 하기 어려우므로 findOrderItems()같은 별도의 메서드로 조회한다.