# Quickstart: GET /api/lessons 레슨 목록 조회

## 로컬 실행

```bash
cd Latinhouse.Be
./gradlew bootRun
```

## API 호출 예시

```bash
# 전체 목록
curl http://localhost:8080/api/lessons

# 지역 필터
curl "http://localhost:8080/api/lessons?region=GN"

# 장르 + 지역 필터
curl "http://localhost:8080/api/lessons?genre=S&region=GN"

# 강사 필터
curl "http://localhost:8080/api/lessons?instructor=Ab2Cd3Ef"
```

## 응답 예시

```json
[
  {
    "optionId": 1,
    "lessonNo": 10,
    "instructorLo": "Ab2Cd3Ef",
    "instructorLa": null,
    "title": "살사 초급반",
    "genre": "S",
    "startDate": "2026-07-01",
    "startTime": "10:00",
    "endDate": "2026-07-01",
    "endTime": "12:00",
    "region": "GN",
    "price": 80000,
    "discountCondition": "2026-06-20",
    "discountAmount": 10000,
    "status": "PENDING"
  }
]
```

## 테스트

```bash
./gradlew test
```
