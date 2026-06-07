# API Contract: Coupon Assign Owner

**Source of Truth**: `docs/api-spec.md` — PATCH /api/coupon/{profileId}

---

## PATCH /api/coupon/{profileId}

**Path Parameter**
- `profileId`: String — 쿠폰을 배정받을 프로필 ID

**Request**
```json
{
  "couponId": 1
}
```

**Response 200 OK**
```json
{
  "couponId": 1
}
```

**Error 404 — PROFILE_NOT_FOUND**
```json
{
  "status": 404,
  "errors": [{ "field": "profileId", "message": "프로필을 찾을 수 없습니다." }]
}
```

**Error 404 — COUPON_NOT_FOUND**
```json
{
  "status": 404,
  "errors": [{ "field": "couponId", "message": "쿠폰을 찾을 수 없습니다." }]
}
```

**Error 400 — couponId null**
```json
{
  "status": 400,
  "errors": [{ "field": "couponId", "message": "must not be null" }]
}
```
