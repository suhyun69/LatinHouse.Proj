# LatinHouse Constitution

## Core Principles

### I. Hexagonal Architecture (Clean Code Pattern)
백엔드는 Hexagonal Architecture(Ports & Adapters 패턴)를 따른다.
- **Why**: 도메인 로직을 외부 기술(JPA, HTTP)로부터 격리하여 독립적으로 테스트·변경 가능하게 한다.
- **원칙**: 의존성 방향은 Adapter → Application → Domain 단방향으로 고정한다. Domain은 어떠한 외부 의존도 갖지 않는다.

패키지 구조:
```
com.latinhouse.api
└── {domain}
    ├── adapter
    │   ├── in.web           # Controller, {Domain}WebRequest (HTTP DTO), {Domain}WebMapper
    │   └── out.persistence  # JPA Entity, Repository 구현체, Persistence Adapter, {Domain}PersistenceMapper
    ├── application
    │   ├── port.in          # UseCase 인터페이스, {Domain}AppRequest, {Domain}AppMapper
    │   ├── port.out         # Port 인터페이스 (e.g. SaveProfilePort)
    │   └── service          # UseCase 구현체
    └── domain               # 순수 도메인 객체, Enum (e.g. Profile, Sex)
```

#### Two-DTO 패턴 (Request/Response 분리 원칙)
HTTP 입력/출력과 애플리케이션 명령/결과는 별도의 DTO 클래스로 분리한다.

| 구분 | 클래스명 | 위치 | 역할 |
|------|----------|------|------|
| Web Request | `{Domain}WebRequest` | `adapter/in/web/` | HTTP 입력 수신. `@NotBlank`, `@Pattern` 등 Bean Validation 적용. 필드 타입은 원시 타입(`String`) 사용. |
| App Request | `{Domain}AppRequest` | `application/port/in/` | 애플리케이션 계층 명령 객체. 도메인 타입(`Sex`, `LocalDate` 등) 사용. Validation 어노테이션 금지. |
| App Response | `{Domain}AppResponse` | `application/port/in/` | 애플리케이션 계층 결과 객체. UseCase 반환 타입. |
| Web Response | `{Domain}WebResponse` | `adapter/in/web/` | HTTP 응답 직렬화 객체. Controller 반환 타입. `ResponseEntity<{Domain}WebResponse>`로 반환한다. |

#### Mapper 패턴
변환 로직은 DTO 내부 메서드가 아닌 **별도 Mapper 클래스**로 분리한다.

| Mapper | 위치 | 변환 방향 | 의존성 |
|--------|------|-----------|--------|
| `{Domain}WebMapper` | `adapter/in/web/` | `WebRequest → AppRequest`, `AppResponse → WebResponse` | Adapter → Application ✅ |
| `{Domain}AppMapper` | `application/port/in/` | `AppRequest → Domain 객체`, `Domain 객체 → AppResponse` | Application → Domain ✅ |
| `{Domain}PersistenceMapper` | `adapter/out/persistence/` | `Domain 객체 ↔ JPA Entity` | Adapter → Domain ✅ |

**규칙**:
- HTTP 원시 타입 → 도메인 타입 변환(e.g. `Sex.valueOf()`)은 **`{Domain}WebMapper`** 에서 수행한다.
- 도메인 객체 생성 시 비즈니스 기본값(e.g. `isInstructor(false)`)은 **`{Domain}AppMapper`** 에서 적용한다.
- Mapper는 인스턴스화 불가(`private` 생성자), 정적 메서드만 제공한다.

```java
// ✅ 올바른 예시
// adapter/in/web/CreateProfileWebMapper.java
public static CreateProfileAppRequest toAppRequest(CreateProfileWebRequest webReq) {
    return CreateProfileAppRequest.builder()
            .nickname(webReq.getNickname())
            .sex(Sex.valueOf(webReq.getSex()))  // ← Adapter에서 변환
            .build();
}

// application/port/in/CreateProfileAppMapper.java
public static Profile toDomain(CreateProfileAppRequest appReq) {
    return Profile.builder()
            .nickname(appReq.getNickname())
            .sex(appReq.getSex())
            .isInstructor(false)               // ← 비즈니스 기본값
            .build();
}

// ProfileController.java
createProfileUseCase.createProfile(CreateProfileWebMapper.toAppRequest(webReq));

// CreateProfileService.java
return saveProfilePort.save(CreateProfileAppMapper.toDomain(appReq));

// ❌ 잘못된 예시 — AppRequest에 String sex 사용 (도메인 타입 미사용)
public class CreateProfileAppRequest {
    private String sex;  // ← 금지: 도메인 타입(Sex)을 써야 함
}
```

### II. Contract-First API Design
API는 `docs/api-spec.md`에 정의된 계약을 기반으로 구현된다.
- **Why**: 명세 없는 구현은 불확실성을 야기하고 클라이언트와의 통합에서 큰 비용을 발생시킨다.
- **원칙**: 모든 요청/응답 형식은 `api-spec.md`를 정확히 따른다. 명세와 구현이 불일치하면 명세를 먼저 수정한 후 구현을 변경한다.

### III. Unified Error Response
모든 에러 응답은 단일 구조를 사용한다.
- **Why**: 일관된 에러 구조는 클라이언트가 에러를 예측 가능하게 처리할 수 있게 한다.
- **원칙**:
  ```json
  {
    "status": 400,
    "errors": [{ "field": "필드명", "message": "에러 메시지" }]
  }
  ```

### IV. Validated Inputs
모든 외부 입력은 반드시 검증된다.
- **Why**: 신뢰할 수 없는 입력은 보안 취약점과 런타임 에러의 주요 원인이다.
- **원칙**:
  - Bean Validation(`@Valid`)을 통한 입력 검증 필수. 검증 실패 시 즉시 400 반환.
  - Validation 어노테이션(`@NotBlank`, `@Pattern` 등)은 `{Domain}WebRequest`에만 적용한다.
  - `{Domain}AppRequest`(application 계층)에는 Validation 어노테이션을 사용하지 않는다.

## 🚨 Guardrails (절대 준수 사항)

AI 코딩 에이전트가 실수로 위험한 작업을 수행하지 않도록 명시적으로 금지하는 규칙들이다.
**이 규칙들은 어떤 상황에서도 위반할 수 없다.**

### 데이터베이스 금지 명령어
- `DROP TABLE`, `DROP DATABASE` — 절대 금지
- `TRUNCATE` — 절대 금지
- `DELETE FROM` (WHERE 절 없이) — 절대 금지
- DDL 스키마 변경 — 사용자 명시적 허가 필요

### 데이터베이스 안전 규칙
- 삭제/리셋 작업 시 반드시 사용자 승인 요청
- 운영 DB 자동 변경 절대 금지

### Git 금지 명령어
- `git push --force` — 절대 금지
- `git reset --hard` — 절대 금지
- `git branch -D` (main/master) — 절대 금지

### 파일 시스템 금지 명령어
- 프로젝트 외부 파일 수정 — 절대 금지
- `.env`, `application-prod.yml` 삭제 — 사용자 확인 필요
- `src/` 디렉토리 전체 삭제 — 절대 금지

### 안전 작업 원칙
- 파괴적 작업(삭제, 초기화) 전 반드시 사용자 확인
- 복구 불가능한 작업은 백업 방법 먼저 안내

## Architecture Constraints

### Immutable Boundaries
- Domain 레이어는 JPA, Spring, HTTP 등 외부 의존 금지
- Application 레이어는 영속성 기술(JPA Entity)에 직접 접근 금지
- Application 레이어는 HTTP 원시 타입(`String` sex 등)을 직접 수신하지 않는다 — 변환은 Adapter 책임
- 계층 간 데이터 전달은 Domain 객체 또는 DTO를 통해서만

### Single Source of Truth
- 도메인 규칙: `domain/` 레이어
- 영속성 매핑: `adapter/out.persistence/`
- API 계약: `docs/api-spec.md`
- 데이터 모델: `docs/data-model.md`

## Quality Standards

### Non-Negotiable Quality Gates
다음 검증을 통과하지 못하면 커밋할 수 없다.
1. 컴파일 오류 없음
2. 모든 테스트 통과
3. `docs/api-spec.md` 명세와 일치
4. 모든 API 엔드포인트는 `@Tag`, `@Operation`으로 Swagger 문서화한다.

### Security Requirements
- 비밀번호 등 민감 정보 코드 하드코딩 금지
- SQL Injection 방지: JPA Parameterized Query만 사용
- 에러 메시지에 내부 스택트레이스 노출 금지

## Governance

### Constitution Authority
이 Constitution은 모든 다른 개발 관행, 가이드, 제안보다 우선한다.

### Amendment Process
Constitution 수정은 `docs/` 문서와의 정합성을 함께 검토 후 진행한다.

### Living Document
실무 개발 가이드는 `CLAUDE.md` 참조

---

**Version**: 1.6.0
**Ratified**: 2026-05-26
**Last Amended**: 2026-05-26
