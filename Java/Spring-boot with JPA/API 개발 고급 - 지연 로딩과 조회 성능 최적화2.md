# API 개발 고급 - 지연 로딩과 조회 성능 최적화2

### 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 패치 조인 최적화

**OrderSimpleApiController**

```java
@GetMapping("/api/v3/simple-orders")
    public List<SimpleOrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        List<SimpleOrderDto> result = orders.stream()
                .map(SimpleOrderDto::new)
                .collect(Collectors.toList());

        return result;
    }
```

**OrderRepository**

```java
public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o" +
                        "join fetch o.member m" +
                        "join fetch o.delivery d", Order.class
        ).getResultList();
    }
```

- 엔티티를 패치 조인을 사용해서 쿼리 1번에 조회
- 패치 조인으로 order→ member, order → delivery는 이미 조회 된 상태이므로 지연 로딩 안됨.

### 간단한 주문 조회: V4: JPA에서 DTO로 바로 조회

**OrderSimpleApiController**

```java
private final OrderSimpleQueryRepository orderSimpleQueryRepository;

@GetMapping("/api/v4/simple-orders")
    public List<SimpleOrderQueryDto> ordersV4() {
        return orderSimpleQueryRepository.findOrderDtos();
    }
```

**SimpleOrderQueryDto**

```java
@Data
public class SimpleOrderQueryDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderQueryDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();//Lazy 초기화
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();//Lazy 초기화
        }
}
```

**SimpleOrderQueryDtoRepository - 조회 전용 리포지토리**

```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<SimpleOrderQueryDto> findOrderDtos() {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.simplequery.SimpleOrderQueryDto(o.id, m.name, o.orderDate, o.status o.address)" +
                                "join o.member m" +
                                "join o.delivery d", SimpleOrderQueryDto.class)
                .getResultList();
    }
}
```

- 일반적인 SQL을 사용할 때 처럼 원하는 값을 선택해서 조회
- new 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
- Select 절에서 원하는 데이터를 직접 선택하므로 → 애플리케이션 네트웍 용량 최적화
- 리포지토리 재사용성 떨어짐, API 스펙에 맞춘 코드가 리포지토리에 들어가는 단점.

**쿼리 방식 선택 권장 순서**

1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 패치 조인으로 성능을 최적화 한다. → 대부분 성능 이슈 해결됨.
3. 그래도 안되면 DTO로 직접 조회하는 방법을 선택한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용.