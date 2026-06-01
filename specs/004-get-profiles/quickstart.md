# Quickstart: GET /api/profiles

## 전체 프로필 조회

```bash
curl -X GET http://localhost:8080/api/profiles
```

응답:
```json
[
  {"id": "Ab2Cd3Ef", "nickname": "홍길동", "sex": "M", "isInstructor": true},
  {"id": "Zx9Yy8Ww", "nickname": "김영희", "sex": "F", "isInstructor": false}
]
```

## 강사 프로필만 조회

```bash
curl -X GET "http://localhost:8080/api/profiles?isInstructor=true"
```

응답:
```json
[
  {"id": "Ab2Cd3Ef", "nickname": "홍길동", "sex": "M", "isInstructor": true}
]
```

## 비강사 프로필만 조회

```bash
curl -X GET "http://localhost:8080/api/profiles?isInstructor=false"
```

## 잘못된 파라미터

```bash
curl -X GET "http://localhost:8080/api/profiles?isInstructor=yes"
```

응답 (400):
```json
{
  "status": 400,
  "errors": [{"field": "isInstructor", "message": "isInstructor는 true 또는 false만 입력 가능합니다."}]
}
```
