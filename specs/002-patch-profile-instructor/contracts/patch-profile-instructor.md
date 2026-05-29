# Contract: PATCH /api/profile/{profileId}/instructor

**Source**: `docs/api-spec.md` (Contract-First — Constitution II)

---

## Endpoint

```
PATCH /api/profile/{profileId}/instructor
```

## Path Parameters

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| profileId | String | Y | 강사로 지정할 프로필의 ID (8자리) |

## Request Body

없음

## Response

### 200 OK — 강사 지정 성공

```json
{
  "id": "Ab2Cd3Ef"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | String | 강사로 지정된 프로필 ID |

### 404 Not Found — 프로필 없음

```json
{
  "status": 404,
  "errors": [
    {
      "field": "profileId",
      "message": "프로필을 찾을 수 없습니다."
    }
  ]
}
```

---

## 계층별 클래스 매핑

```
HTTP Request
  PATCH /api/profile/Ab2Cd3Ef/instructor
          │
          ▼ @PathVariable profileId
  ProfileController.setInstructor(String profileId)
          │
          │ SetInstructorWebMapper.toAppRequest(profileId)
          ▼
  SetInstructorAppRequest { profileId: "Ab2Cd3Ef" }
          │
          ▼ SetInstructorUseCase.setInstructor(request)
  SetInstructorService
    ├── FindProfilePort.findById("Ab2Cd3Ef")
    │     └─ ProfileNotFoundException if not found
    ├── profile.asInstructor()          ← 도메인 메서드
    └── UpdateProfilePort.update(profile)
          │
          ▼
  SetInstructorAppResponse { id: "Ab2Cd3Ef" }
          │
          │ SetInstructorWebMapper.toWebResponse(appResponse)
          ▼
  SetInstructorWebResponse { id: "Ab2Cd3Ef" }
          │
          ▼ ResponseEntity.ok(response)
HTTP Response: 200 OK
  { "id": "Ab2Cd3Ef" }
```

---

## 멱등성

동일 profileId에 대한 반복 호출은 항상 200 OK를 반환한다.  
`isInstructor`가 이미 `true`이면 도메인 메서드 `asInstructor()`가 동일한 상태를 반환하고 UpdateProfilePort를 통해 재저장한다.

---

## Swagger 문서화 (Constitution Quality Gate)

```java
@Tag(name = "Profile", description = "프로필 관리 API")           // 컨트롤러 클래스
@Operation(summary = "강사 지정", description = "프로필을 강사로 지정합니다.")  // 메서드
```
