# API 개발 고급 - 컬렉션 조회 최적화3

### 주문 조회V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

**OrderApiController**

```java
@GetMapping("/api/v5/orders")
    public List<OrderQueryDto> orderV5() {
        return orderQueryRepository.findAllByDto_optimization();
    }
```

**OrderQueryRepository**

```java
public List<OrderQueryDto> findAllByDto_optimization() {
//루트 조회(toOne 코드를 모두 한번에 조회)
        List<OrderQueryDto> result = findOrders();
//orderItem 컬렉션을 Map 한방에 조회
        Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(toOrderIds(result));
//루프를 돌면서 컬렉션 추가(추가 쿼리 실행x)
        result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));

        return result;
    }

    private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
        List<OrderItemQueryDto> orderItems = em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.ItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                                "from OrderItem oi" +
                                "join oi.item i" +
                                "where oi.order.id in :orderIds", OrderItemQueryDto.class
                ).setParameter("orderIds", orderIds)
                .getResultList();

        Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream()
                .collect(Collectors.groupingBy(orderItemQueryDto -> orderItemQueryDto.getOrderId()));
        return orderItemMap;
    }

    private List<Long> toOrderIds(List<OrderQueryDto> result) {
        List<Long> orderIds = result.stream()
                .map(o -> o.getOrderId())
                .collect(Collectors.toList());
        return orderIds;
    }
```

- Query: 루트 1번, 컬렉션 1번
- ToOne 관계들을 먼저 조회하고, 여기서 얻은 식별자 orderId로 ToMany관계인 OrderItem을 한꺼번에 조회
- Map을 사용해서 매칭 성능 향상 O(1)

### 주문 조회V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

**OrderApiController** 

```java
@GetMapping("/api/v6/orders")
    public List<OrderQueryDto> ordersV6() {
        List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();
        return flats.stream()
                .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(),
                                o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                        mapping(o -> new OrderItemQueryDto(o.getOrderId(),
                                o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
                )).entrySet().stream()
                .map(e -> new OrderQueryDto(e.getKey().getOrderId(),
                        e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(),
                        e.getKey().getAddress(), e.getValue()))
                .collect(toList());
    }
```

**OrderQuertyRepository**

```java
public List<OrderFlatDto> findAllByDto_flat() {
        List<OrderFlatDto> result = em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate," +
                        "o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                        "from Order o" +
                        "join o.member m" +
                        "join o.delivery d" +
                        "join o.orderItems oi" +
                        "join oi.item i", OrderFlatDto.class
        ).getResultList();

        return result;
    }
```

**OrderFlatDto**

```java
@Data
public class OrderFlatDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

- 쿼리: 1번
- 단점
    - 쿼리는 한번이지만 조인으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복 데이터가 추가되므로 상황에 따라 V5보다 느릴 수 있다.
    - 애플리케이션에서 추가 작업이 크다.
    - 페이징 불가능
    

### API 개발 고급 정리

1. 엔티티 조회 방식으로 우선 접근
    1. 페치 조인으로 쿼리 수를 최적화
    2. 컬렉션 최적화
        1. 페이징 필요 hibernate.default_batch_fetch_size, @BatchSize로 최적화
        2. 페이징 필요x → 페치 조인 사용
2. 엔티티 조회 방식으로 해결이 안되면 DTO 조회 방식 사용
3. DTO 조회 방식으로 해결이 안되면 NativeSql or 스프링 JdbcTemplate

> 참고: 엔티티 조회 방식은 페치 조인이나,hibernate.default_batch_fetch_size , @BatchSize 같이 코드를 거의 수정하지 않고, 옵션만 약간 변경해서, 다양한 성능 최적화를 시도할 수 있다. 반면에 DTO를 직접 조회하는 방식은 성능을 최적화 하거나 성능 최적화 방식을 변경할 때 많은 코드를 변경해야 한다.
> 

> 참고: 개발자는 성능 최적화와 코드 복잡도 사이에서 줄타기를 해야 한다. 항상 그런 것은 아니지만, 보통 성능 최적화는 단순한 코드를 복잡한 코드로 몰고간다. 엔티티 조회 방식은 JPA가 많은 부분을 최적화 해주기 때문에, 단순한 코드를 유지하면서, 성능을 최적화 할 수 있다. 반면에 DTO 조회 방식은 SQL을 직접 다루는 것과 유사하기 때문에, 둘 사이에 줄타기를 해야 한다
> 

**DTO 조회 방식의 선택지**

- DTO로 조회하는 방법도 각각 장단이 있다. V4, V5, V6에서 단순하게 쿼리가 1번 실행된다고 V6이 항상 좋은 방법인 것은 아니다.
- V4는 코드가 단순하다. 특정 주문 한건만 조회하면 이 방식을 사용해도 성능이 잘 나온다. 예를 들어서 조회한 Order 데이터가 1건이면 OrderItem을 찾기 위한 쿼리도 1번만 실행하면 된다.
- V5는 코드가 복잡하다. 여러 주문을 한꺼번에 조회하는 경우에는 V4 대신에 이것을 최적화한 V5 방식을 사용해야 한다. 예를 들어서 조회한 Order 데이터가 1000건인데, V4 방식을 그대로 사용하면, 쿼리가 총 1 + 1000번 실행된다. 여기서 1은 Order 를 조회한 쿼리고, 1000은 조회된 Order의 row 수다. V5방식으로 최적화 하면 쿼리가 총 1 + 1번만 실행된다. 상황에 따라 다르겠지만 운영 환경에서 100배 이상의 성능 차이가 날 수 있다.
- V6는 완전히 다른 접근방식이다. 쿼리 한번으로 최적화 되어서 상당히 좋아보이지만, Order를 기준으로 페이징이 불가능하다. 실무에서는 이정도 데이터면 수백이나, 수천건 단위로 페이징 처리가 꼭 필요하므로, 이 경우 선택하기 어려운 방법이다. 그리고 데이터가 많으면 중복 전송이 증가해서 V5와 비교해서 성능 차이도 미비하다.