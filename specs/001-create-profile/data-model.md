# Data Model: Create Profile

**Branch**: `001-create-profile` | **Date**: 2026-05-28
**Source**: [docs/data-model.md](../../docs/data-model.md)

## Profile — JPA Entity

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | PK, NOT NULL | 8자리 난수 ID (허용 문자 55종) |
| nickname | String | NOT NULL | 닉네임 |
| sex | String | NOT NULL | `"M"` 또는 `"F"` |
| isInstructor | Boolean | NOT NULL, default false | 강사 여부 |

## Profile — Domain Object

| 필드 | 타입 | 제약 | 설명 |
|------|------|------|------|
| id | String | non-null | 8자리 난수 ID |
| nickname | String | non-null | 닉네임 |
| sex | Sex (Enum) | non-null | `Sex.M` / `Sex.F` |
| isInstructor | Boolean | non-null, default false | 강사 여부 |

## Sex Enum

값: `M`, `F`

## ID 생성 규칙

- 길이: 8자
- 허용 문자 (총 55자):
  - 대문자 24자: `ABCDEFGHJKLMNPQRSTUVWXYZ` (I 제외)
  - 소문자 23자: `abcdefghjkmnpqrstuvwxyz` (i, l 제외)
  - 숫자 8자: `23456789` (0, 1 제외)
- 생성: `SecureRandom` + 커스텀 허용 문자셋 (`ProfileIdGenerator.generate()`)

## Validation Rules (WebRequest 레이어에만 적용)

| 필드 | 어노테이션 | 에러 메시지 |
|------|------------|-------------|
| nickname | `@NotBlank` | "닉네임을 입력해 주세요." |
| sex | `@NotBlank` | "성별을 입력해 주세요." |
| sex | `@Pattern(regexp = "^[MF]$")` | "성별은 M 또는 F만 입력 가능합니다." |

**주의**: `@NotBlank` + `@Pattern` 조합 시, blank이면 `@NotBlank` 메시지, 유효하지 않은 값이면 `@Pattern` 메시지가 반환된다.

## DTO 흐름 (Two-DTO 패턴)

```
HTTP Request JSON
    ↓
CreateProfileWebRequest   (adapter/in/web — String nickname, String sex)
    ↓ CreateProfileWebMapper.toAppRequest()
CreateProfileAppRequest   (application/port/in — String nickname, Sex sex)
    ↓ CreateProfileAppMapper.toDomain()
Profile (Domain)          (domain — id=null, nickname, sex, isInstructor=false)
    ↓ SaveProfilePort.save()
Profile (Domain, id 생성됨)
    ↓ CreateProfileAppMapper.toAppResponse()
CreateProfileAppResponse  (application/port/in — String id)
    ↓ CreateProfileWebMapper.toWebResponse()
CreateProfileWebResponse  (adapter/in/web — String id)
    ↓
HTTP 201 ResponseEntity<CreateProfileWebResponse>
```
