# Quickstart: PATCH /api/profile/{profileId}/instructor

## 로컬 실행

```bash
cd Latinhouse.Be
./gradlew bootRun
```

서버 기동 후 `http://localhost:8080` 에서 응답 확인.

---

## 수동 테스트 시나리오

### 1. 프로필 생성 (테스트용 profileId 확보)

```bash
curl -s -X POST http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d '{"nickname":"테스트유저","sex":"M"}'
```

**예상 응답 (201 Created)**:
```json
{ "id": "Ab2Cd3Ef" }
```

### 2. 강사 지정 — 정상 케이스

```bash
curl -s -X PATCH http://localhost:8080/api/profile/Ab2Cd3Ef/instructor
```

**예상 응답 (200 OK)**:
```json
{ "id": "Ab2Cd3Ef" }
```

### 3. 멱등성 확인 — 동일 요청 재호출

```bash
curl -s -X PATCH http://localhost:8080/api/profile/Ab2Cd3Ef/instructor
```

**예상 응답 (200 OK)**: 동일 응답 반환.

### 4. 존재하지 않는 profileId

```bash
curl -s -X PATCH http://localhost:8080/api/profile/NOTEXIST/instructor
```

**예상 응답 (404 Not Found)**:
```json
{
  "status": 404,
  "errors": [
    { "field": "profileId", "message": "프로필을 찾을 수 없습니다." }
  ]
}
```

---

## Swagger UI

`http://localhost:8080/swagger-ui/index.html` → **Profile** 섹션 → `PATCH /api/profile/{profileId}/instructor`

---

## 테스트 실행

```bash
cd Latinhouse.Be
./gradlew test
```

주요 테스트 클래스:
- `ProfileControllerTest` — `@WebMvcTest`, PATCH 엔드포인트 슬라이스 테스트
- `SetInstructorServiceTest` — 서비스 유닛 테스트 (FindProfilePort, UpdateProfilePort Mock)
