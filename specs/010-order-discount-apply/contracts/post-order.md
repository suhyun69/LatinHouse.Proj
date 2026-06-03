# Contract: POST /api/order (할인 적용 변경사항)

이 피처는 `POST /api/order`의 Request/Response 구조를 변경하지 않는다.
기존 계약(009-create-order)을 그대로 유지하며, 내부 처리 로직(할인 선별)만 변경된다.

## 변경 없음

- **Endpoint**: `POST /api/order`
- **Request Body**: 변경 없음 (`lessonNo`, `lessonOptionNo`, `profileId`)
- **Response 201**: 변경 없음 (`{ "orderId": "uuid" }`)
- **Error 응답**: 변경 없음 (400/404 구조 동일)

## 내부 동작 변경 (외부 계약에 노출되지 않음)

주문 생성 시 `Order.discounts`가 빈 리스트 대신 적용 가능한 할인 항목으로 채워진다.
이 변경은 DB에 저장되는 데이터에만 영향을 미치며 API 응답에는 영향이 없다.
