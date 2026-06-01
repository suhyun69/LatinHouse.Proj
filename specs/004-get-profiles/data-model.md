# Data Model: GET /api/profiles

## 조회 대상 Entity

### Profile (기존 — 변경 없음)

| 필드 | 타입 | 제약 | 역할 |
|------|------|------|------|
| id | String | PK, 8자리 | 프로필 고유 식별자 |
| nickname | String | NOT NULL | 닉네임 |
| sex | String | `"M"` 또는 `"F"` | 성별 |
| isInstructor | Boolean | default false | 강사 여부 |

이 API는 데이터를 변경하지 않는다 (순수 조회).

---

## 신규 DTO 목록

### Application Layer (port/in)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `GetProfilesAppResponse` | `id: String`, `nickname: String`, `sex: Sex`, `isInstructor: boolean` | Service 결과 항목 DTO. 도메인 타입(`Sex` enum) 사용. |

### Web Adapter Layer (adapter/in/web)

| 클래스 | 필드 | 설명 |
|--------|------|------|
| `GetProfilesWebResponse` | `id: String`, `nickname: String`, `sex: String`, `isInstructor: boolean` | HTTP 응답 직렬화 객체. `sex`는 문자열(`"M"`, `"F"`)로 직렬화. |

> **GetProfilesWebRequest 미생성**: 쿼리 파라미터 `isInstructor`는 Controller 메서드 파라미터로 직접 수신하므로 별도 DTO 불필요.

> **GetProfilesAppRequest 미생성**: 단일 Boolean 파라미터이므로 DTO 래핑 불필요. UseCase 메서드에 직접 전달.

---

## Port 인터페이스 변경

### FindProfilePort (port/out) — 메서드 추가

```
FindProfilePort
├── findById(profileId: String): Optional<Profile>   ← 기존
└── findAll(isInstructor: Boolean): List<Profile>    ← 신규 추가
```

- `isInstructor = null`: 전체 프로필 반환
- `isInstructor = true`: 강사 프로필만 반환
- `isInstructor = false`: 비강사 프로필만 반환

---

## UseCase 인터페이스

### GetProfilesUseCase (port/in)

```
GetProfilesUseCase
└── getProfiles(isInstructor: Boolean): List<GetProfilesAppResponse>
```

---

## JPA Repository 변경

### ProfileJpaRepository — 메서드 추가

```
ProfileJpaRepository extends JpaRepository<ProfileEntity, String>
└── findAllByIsInstructor(isInstructor: boolean): List<ProfileEntity>  ← 신규
```

기존 `findAll()`은 전체 조회에 그대로 사용.

---

## 응답 구조

```json
[
  {
    "id": "Ab2Cd3Ef",
    "nickname": "홍길동",
    "sex": "M",
    "isInstructor": true
  },
  {
    "id": "Zx9Yy8Ww",
    "nickname": "김영희",
    "sex": "F",
    "isInstructor": false
  }
]
```

결과 없을 때: `[]` (빈 배열)
