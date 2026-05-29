# Data Model: PATCH /api/profile/{profileId}/instructor

## 변경 대상 Entity

### Profile (기존 — 변경 없음)

| 필드 | 타입 | 제약 | 변경 여부 |
|------|------|------|-----------|
| id | String | PK, 8자리 | 변경 없음 |
| nickname | String | NOT NULL | 변경 없음 |
| sex | String | `"M"` 또는 `"F"` | 변경 없음 |
| isInstructor | Boolean | default false | **이 API가 true로 변경** |

변경 사항: `isInstructor`를 `true`로 설정. 이 API는 단방향 변경만 지원한다.

---

## 도메인 객체 변경

### Profile (Domain)

기존 `Profile` 도메인 클래스에 `asInstructor()` 메서드 추가.

```
Profile
├── id: String
├── nickname: String
├── sex: Sex
├── isInstructor: boolean
└── + asInstructor(): Profile   ← 신규 도메인 메서드
```

**asInstructor() 동작**: 현재 Profile의 모든 필드를 그대로 유지하고 `isInstructor`를 `true`로 설정한 새 Profile 인스턴스를 반환한다 (불변 객체 패턴).

---

## 신규 DTO 목록

### Application Layer (port/in)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `SetInstructorAppRequest` | `profileId: String` | Service 입력 명령 객체 |
| `SetInstructorAppResponse` | `id: String` | Service 결과 반환 객체 |

### Web Adapter Layer (adapter/in/web)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `SetInstructorWebResponse` | `id: String` | HTTP 응답 직렬화 객체 |

> **SetInstructorWebRequest 미생성**: 요청 바디가 없고 Path variable만 사용하므로 불필요.

---

## 신규 Port 인터페이스

### FindProfilePort (port/out)

```
FindProfilePort
└── findById(profileId: String): Optional<Profile>
```

- 존재하지 않으면 `Optional.empty()` 반환
- Service에서 empty일 경우 `ProfileNotFoundException` throw

### UpdateProfilePort (port/out)

```
UpdateProfilePort
└── update(profile: Profile): Profile
```

- 변경된 Profile 도메인 객체를 받아 영속화 후 반환

---

## 신규 예외

### ProfileNotFoundException (common/exception)

| 항목 | 내용 |
|------|------|
| 슈퍼클래스 | `RuntimeException` |
| 생성자 파라미터 | `String profileId` |
| HTTP 상태 | 404 Not Found |
| 에러 코드 | `PROFILE_NOT_FOUND` |
| 에러 응답 field | `"profileId"` |
| 에러 응답 message | `"프로필을 찾을 수 없습니다."` |

에러 응답 예시:
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

## 상태 전이

```
Profile.isInstructor = false
        │
        │  PATCH /api/profile/{profileId}/instructor
        ▼
Profile.isInstructor = true  (이 상태에서 동일 요청 시 멱등으로 200 OK)
```
