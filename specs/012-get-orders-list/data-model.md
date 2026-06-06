# Data Model: 주문 목록 조회 (GET /api/orders)

신규 Entity/Domain 없음. 기존 `Order`, `OrderDiscount`, `OrderEntity`, `OrderDiscountEntity` 재사용.

## 변경 대상

### OrderJpaRepository (수정)
`JpaSpecificationExecutor<OrderEntity>` 추가:
```java
interface OrderJpaRepository extends JpaRepository<OrderEntity, String>,
        JpaSpecificationExecutor<OrderEntity> { }
```

### OrderPersistenceAdapter (수정)
`LoadOrderPort` 구현 추가:
```java
@Override
public List<Order> loadOrders(String buyer, Long lessonNo) {
    Specification<OrderEntity> spec = buildSpec(buyer, lessonNo);
    return orderJpaRepository.findAll(spec).stream()
            .map(OrderPersistenceMapper::toDomain)
            .toList();
}

private Specification<OrderEntity> buildSpec(String buyer, Long lessonNo) {
    Specification<OrderEntity> spec = (root, query, cb) -> cb.conjunction();
    if (buyer != null) {
        spec = spec.and((root, query, cb) -> cb.equal(root.get("buyer"), buyer));
    }
    if (lessonNo != null) {
        spec = spec.and((root, query, cb) -> cb.equal(root.get("lessonNo"), lessonNo));
    }
    return spec;
}
```

## 신규 클래스

### GetOrdersUseCase (신규)
```
application/port/in/GetOrdersUseCase.java
```

### GetOrdersAppRequest (신규)
```
application/port/in/GetOrdersAppRequest.java
```
| 필드 | 타입 | 설명 |
|------|------|------|
| buyer | String | nullable, 구매자 Profile ID |
| lessonNo | Long | nullable, 레슨 번호 |

### GetOrdersAppResponse (신규)
```
application/port/in/GetOrdersAppResponse.java
```
| 필드 | 타입 | 설명 |
|------|------|------|
| orderId | String | 주문 ID |
| lessonNo | Long | 레슨 번호 |
| lessonOptionNo | Long | 레슨 옵션 번호 |
| price | BigDecimal | 주문 금액 |
| status | String | 주문 상태 |
| discounts | List\<DiscountInfo\> | 적용 할인 목록 |

**DiscountInfo (static inner class)**
| 필드 | 타입 |
|------|------|
| discountType | String |
| discountId | Long |
| amount | BigDecimal |

### LoadOrderPort (신규)
```
application/port/out/LoadOrderPort.java
```
```java
List<Order> loadOrders(String buyer, Long lessonNo);
```

### GetOrdersService (신규)
```
application/service/GetOrdersService.java
```

### GetOrdersWebResponse (신규)
```
adapter/in/web/GetOrdersWebResponse.java
```
동일 필드 구조. `OrderDiscountInfo` static inner class 포함.

### GetOrdersWebMapper (신규)
```
adapter/in/web/GetOrdersWebMapper.java
```
- `toAppRequest(String buyer, String lessonNo)` → `GetOrdersAppRequest`
- `toWebResponse(GetOrdersAppResponse)` → `GetOrdersWebResponse`
- `toWebResponseList(List<GetOrdersAppResponse>)` → `List<GetOrdersWebResponse>`

## 재사용 클래스

| 클래스 | 패키지 | 재사용 방식 |
|--------|--------|------------|
| `Order` | `order.domain` | 조회 결과 도메인 객체 |
| `OrderDiscount` | `order.domain` | discounts 필드 매핑 |
| `OrderEntity` | `order.adapter.out.persistence` | JPA Specification 필터링 대상 |
| `OrderPersistenceMapper` | `order.adapter.out.persistence` | toDomain() 재사용 |
| `SecurityConfig` | `common.config` | GET /api/orders permitAll 추가 |

## 테스트 변경

### GetOrdersServiceTest (신규)
| 테스트 메서드 | 검증 내용 |
|---------------|-----------|
| `getOrders_byBuyer_returnsOnlyBuyerOrders` | buyer 필터 정확성 |
| `getOrders_byLessonNo_returnsOnlyLessonOrders` | lessonNo 필터 정확성 |
| `getOrders_byBuyerAndLessonNo_andCondition` | AND 복합 조건 |
| `getOrders_noMatch_returnsEmptyList` | 빈 배열 반환 |
| `getOrders_noParams_returnsAll` | 전체 조회 |

### OrderControllerTest (수정)
기존 POST 테스트에 GET /api/orders 테스트 추가:
| 테스트 메서드 | 검증 내용 |
|---------------|-----------|
| `getOrders_withBuyer_returns200` | 응답 구조 및 buyer 필터 |
| `getOrders_noParams_returns200` | 파라미터 없는 전체 조회 |
