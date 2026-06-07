# API Contract: Coupon

**Source of Truth**: `docs/api-spec.md` — Coupon API 섹션

---

## POST /api/coupon/template

**Request**
```json
{
  "title": "string (required)",
  "type": "LESSON",
  "target": 1,
  "amount": 5000.00
}
```

**Response 201 Created**
```json
{
  "couponTemplateId": "1"
}
```

**Error 400**
```json
{
  "status": 400,
  "errors": [{ "field": "amount", "message": "must not be null" }]
}
```

---

## POST /api/coupon

**Request**
```json
{
  "templateId": 1,
  "count": 5
}
```

**Response 201 Created** — No Body

**Error 404**
```json
{
  "status": 404,
  "errors": [{ "field": "templateId", "message": "쿠폰 템플릿을 찾을 수 없습니다." }]
}
```

**Error 400** (count < 1)
```json
{
  "status": 400,
  "errors": [{ "field": "count", "message": "must be greater than or equal to 1" }]
}
```
