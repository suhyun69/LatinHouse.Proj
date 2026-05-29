# Data Model

## Profile

### Entity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | 8자리 난수. 대/소문자+숫자 혼용, `i I 1 l 0 o O` 제외 |
| nickname | String | NOT NULL | 중복 허용 |
| sex | String | | `"M"` 또는 `"F"` |
| isInstructor | Boolean | default false | 강사 여부. `PATCH /api/profile/{profileId}/instructor` 호출 시 `true`로 변경됨 |

### Domain

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK | 8자리 난수. 대/소문자+숫자 혼용, `i I 1 l 0 o O` 제외 |
| nickname | String | Not null | 중복 허용 |
| sex | Sex | Enum | `M(Male)` / `F(Female)` |
| isInstructor | Boolean | default false | 강사 여부. `PATCH /api/profile/{profileId}/instructor` 호출 시 `true`로 변경됨 |

### ID 규칙

- 길이: 8자
- 허용 문자 (총 55자):
  - 대문자 24자: `ABCDEFGHJKLMNPQRSTUVWXYZ`
  - 소문자 23자: `abcdefghjkmnpqrstuvwxyz`
  - 숫자 8자: `23456789`
