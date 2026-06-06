# Contract: GET /api/orders

## Endpoint

```
GET /api/orders
```

## Query Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| buyer | String | N | 구매자 Profile ID |
| lessonNo | Long | N | 레슨 번호 |

## Response — 200 OK

```json
[
  {
    "orderId": "550e8400-e29b-41d4-a716-446655440000",
    "lessonNo": 1,
    "lessonOptionNo": 3,
    "price": 80000,
    "status": "PAYMENT_PENDING",
    "discounts": [
      {
        "discountType": "LESSON",
        "discountId": 10,
        "amount": 5000
      }
    ]
  }
]
```

조건에 맞는 주문 없을 때: `[]`

## Error Cases

없음. 유효하지 않은 파라미터도 빈 배열로 처리.

## Scenarios

| Scenario | Request | Expected Response |
|----------|---------|-------------------|
| buyer로 필터링 | `?buyer=Ab2Cd3Ef` | 해당 buyer 주문 목록 |
| lessonNo로 필터링 | `?lessonNo=1` | 해당 레슨 주문 목록 |
| AND 복합 조건 | `?buyer=Ab2Cd3Ef&lessonNo=1` | 두 조건 모두 만족하는 주문만 |
| 파라미터 없음 | (없음) | 전체 주문 목록 |
| 결과 없음 | `?buyer=unknown` | `[]` |
