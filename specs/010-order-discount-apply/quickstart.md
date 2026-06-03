# Quickstart: 주문 생성 시 레슨 할인 자동 적용

## 변경 파일 목록

| 파일 | 변경 유형 | 설명 |
|------|-----------|------|
| `order/application/service/CreateOrderService.java` | **수정** | 할인 선별 로직 추가 |
| `order/application/service/CreateOrderServiceTest.java` | **수정** | 할인 관련 테스트 케이스 추가 |

## 통합 시나리오

### 시나리오 1: SEX 할인 매칭
1. 레슨에 `DiscountType.SEX`, condition="M", amount=5000 할인 존재
2. Profile.sex=M으로 `POST /api/order` 요청
3. DB `order_discount` 테이블에 1건 저장: discountType=LESSON, discountId={할인 ID}, amount=5000

### 시나리오 2: SEX 할인 불일치
1. 레슨에 `DiscountType.SEX`, condition="M" 할인 존재
2. Profile.sex=F로 `POST /api/order` 요청
3. DB `order_discount` 테이블에 할인 미저장

### 시나리오 3: EARLYBIRD 다중 할인
1. 레슨에 EARLYBIRD condition="2026-07-01", condition="2026-06-01" 2건 존재
2. `POST /api/order` 요청
3. DB `order_discount`에 condition="2026-06-01" 1건만 저장

## 검증 체크리스트

- [ ] `CreateOrderService` 수정 후 컴파일 성공
- [ ] `CreateOrderServiceTest` 신규 테스트 전체 통과
- [ ] 기존 `OrderControllerTest` 테스트 전체 통과 (회귀 없음)
- [ ] `./gradlew test` 전체 통과
