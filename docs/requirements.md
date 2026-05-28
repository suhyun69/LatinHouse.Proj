# Requirements

## 1. 기능 요구사항 (Functional Requirements)

### FR-P-001: 프로필 생성

**설명**: 닉네임과 성별을 입력받아 프로필을 생성한다.

**API 매핑**: `POST /api/profile`

**입력 필드**:
| 필드 | 타입 | 필수 | 제약조건 | 기본값 |
|------|------|------|----------|--------|
| nickname | string | O | - | - |
| sex | string | O | M 또는 F | - |

**처리 규칙**:
- id는 8자리 난수 자동 생성 (대/소문자+숫자, i I 1 l 0 o O 제외)
- isInstructor는 false로 자동 설정

**검증 에러 메시지**:
| 조건 | 메시지 |
|------|--------|
| nickname 누락 | "닉네임을 입력해 주세요." |
| sex 누락 | "성별을 입력해 주세요." |
| sex가 M, F 외의 값 | "성별은 M 또는 F만 입력 가능합니다." |

**성공 응답**: 201 Created (Body 없음)

**실패 응답**: 400 Bad Request + 검증 에러 상세
```json
{
  "status": 400,
  "errors": [{ "field": "필드명", "message": "에러 메시지" }]
}
```

---
