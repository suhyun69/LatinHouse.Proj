# Feature Specification: 레슨 생성

**Feature Branch**: `003-create-lesson`

**Created**: 2026-05-29

**Status**: Draft

## User Scenarios & Testing *(mandatory)*

### User Story 1 - 레슨 기본 생성 (Priority: P1)

강사가 제목, 장르, 강사 정보, 수업 옵션(일정·지역)을 입력하여 레슨을 등록한다. 남성 강사 또는 여성 강사 중 하나 이상을 반드시 지정해야 하며, 수업 옵션은 1개 이상 포함되어야 한다.

**Why this priority**: 레슨 생성은 서비스의 핵심 기능이며, 이것 없이는 다른 모든 기능이 무의미하다.

**Independent Test**: 최소 필수 필드(title, genre, instructorLo 또는 instructorLa, options 1개)만 입력하여 레슨 생성 요청 시 201 Created와 생성된 레슨 id가 반환되는지 확인.

**Acceptance Scenarios**:

1. **Given** 유효한 남성 강사 ID(isInstructor=true, sex=M)와 수업 옵션 1개가 준비된 상태에서, **When** title·genre·instructorLo·options를 포함한 생성 요청을 보내면, **Then** 201 Created와 `{"id": <생성된 레슨 id>}`가 반환된다.
2. **Given** 유효한 여성 강사 ID(isInstructor=true, sex=F)와 수업 옵션 1개가 준비된 상태에서, **When** title·genre·instructorLa·options를 포함한 생성 요청을 보내면, **Then** 201 Created와 생성된 레슨 id가 반환된다.
3. **Given** 남성·여성 강사 ID가 모두 준비된 상태에서, **When** instructorLo와 instructorLa를 둘 다 포함하여 요청하면, **Then** 201 Created와 생성된 레슨 id가 반환된다.

---

### User Story 2 - 레슨 부가 정보 등록 (Priority: P2)

레슨 생성 시 할인 정책, 입금 계좌, 연락처, 공지사항 등 부가 정보를 함께 등록한다.

**Why this priority**: 부가 정보는 선택 항목이지만, 실제 서비스에서 수강생에게 필요한 정보를 충분히 제공하기 위해 중요하다.

**Independent Test**: 기본 필수 필드에 추가로 discounts, account, contacts, notices를 포함하여 요청 시 201 Created로 정상 생성되는지 확인.

**Acceptance Scenarios**:

1. **Given** 유효한 레슨 정보와 함께 Earlybird 할인(type=E, condition=`yyyy-MM-dd`)이 포함된 상태에서, **When** 생성 요청을 보내면, **Then** 201 Created가 반환된다.
2. **Given** 유효한 레슨 정보와 함께 성별 할인(type=S, condition=`M` 또는 `F`)이 포함된 상태에서, **When** 생성 요청을 보내면, **Then** 201 Created가 반환된다.
3. **Given** 입금 계좌, 연락처(타입별), 공지사항(타입별)이 모두 포함된 상태에서, **When** 생성 요청을 보내면, **Then** 201 Created가 반환된다.

---

### User Story 3 - 입력 형식 오류 처리 (Priority: P3)

잘못된 형식의 값이 입력되었을 때 명확한 오류 메시지를 반환한다.

**Why this priority**: 잘못된 입력에 대한 명확한 피드백은 클라이언트가 올바른 데이터를 재전송할 수 있도록 돕는다.

**Independent Test**: 각 필드에 유효하지 않은 값을 넣어 400 Bad Request와 `errors` 배열이 포함된 응답이 반환되는지 확인.

**Acceptance Scenarios**:

1. **Given** title이 빈 문자열인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"제목을 입력해 주세요."` 메시지가 반환된다.
2. **Given** genre가 `S`, `B` 외의 값인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"장르는 S 또는 B만 입력 가능합니다."` 메시지가 반환된다.
3. **Given** options가 빈 리스트인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"수업 옵션을 1개 이상 입력해 주세요."` 메시지가 반환된다.
4. **Given** options[].startDate가 `yyyy-MM-dd` 형식이 아닌 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 형식 오류 메시지가 반환된다.
5. **Given** options[].region이 `GN`, `HD` 외의 값인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"지역은 GN 또는 HD만 입력 가능합니다."` 메시지가 반환된다.

---

### User Story 4 - 비즈니스 규칙 오류 처리 (Priority: P4)

도메인 비즈니스 규칙을 위반한 요청에 대해 명확한 오류 메시지를 반환한다.

**Why this priority**: 데이터 일관성과 서비스 정책 준수를 보장하는 핵심 검증 로직이다.

**Independent Test**: 비즈니스 규칙을 위반하는 각 케이스에서 400 Bad Request와 적절한 에러 코드·메시지가 반환되는지 확인.

**Acceptance Scenarios**:

1. **Given** instructorLo와 instructorLa가 모두 null인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"남성 강사 또는 여성 강사 중 하나는 반드시 입력해야 합니다."` 메시지가 반환된다.
2. **Given** instructorLo에 존재하지 않는 Profile ID가 입력된 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `INSTRUCTOR_NOT_FOUND` 에러 코드가 반환된다.
3. **Given** instructorLo에 isInstructor=false인 Profile ID가 입력된 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `INSTRUCTOR_NOT_VALID` 에러 코드가 반환된다.
4. **Given** instructorLo에 sex=F인 Profile ID가 입력된 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다."` 메시지가 반환된다.
5. **Given** options[].startDate+startTime이 endDate+endTime보다 이후인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"시작 일시는 종료 일시보다 이전이어야 합니다."` 메시지가 반환된다.
6. **Given** discounts[].type=E이고 condition이 날짜 형식이 아닌 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"얼리버드 할인 조건은 yyyy-MM-dd 형식의 날짜여야 합니다."` 메시지가 반환된다.
7. **Given** discounts[].type=S이고 condition이 `M`, `F` 외의 값인 상태에서, **When** 생성 요청을 보내면, **Then** 400 Bad Request와 `"성별 할인 조건은 M 또는 F만 입력 가능합니다."` 메시지가 반환된다.

---

### Edge Cases

- instructorLo와 instructorLa가 동일한 Profile ID일 경우는 별도로 다루지 않는다 (남성·여성 분리로 자연히 불가).
- options를 여러 개 입력했을 때 일부만 시간 순서가 잘못된 경우: 해당 옵션에 대한 오류 메시지만 반환한다.
- discounts를 여러 개 입력했을 때 일부만 condition이 잘못된 경우: 해당 항목에 대한 오류 메시지만 반환한다.
- 여러 필드가 동시에 유효하지 않은 경우: `errors` 배열에 복수의 오류를 담아 반환한다.
- isActive를 입력하지 않은 경우: 기본값 `true`로 생성된다.

## Requirements *(mandatory)*

### Functional Requirements

**webRequest 레이어 (형식·타입·필수값 검증)**

- **FR-001**: 시스템은 title이 null 또는 빈 문자열이면 `"제목을 입력해 주세요."` 오류를 반환해야 한다.
- **FR-002**: 시스템은 genre가 null 또는 빈 문자열이면 `"장르를 입력해 주세요."` 오류를 반환해야 한다.
- **FR-003**: 시스템은 genre가 `S`, `B` 외의 값이면 `"장르는 S 또는 B만 입력 가능합니다."` 오류를 반환해야 한다.
- **FR-004**: 시스템은 options가 null 또는 빈 리스트이면 `"수업 옵션을 1개 이상 입력해 주세요."` 오류를 반환해야 한다.
- **FR-005**: 시스템은 options[].startDate가 null/빈 문자열이거나 `yyyy-MM-dd` 형식이 아니면 오류를 반환해야 한다.
- **FR-006**: 시스템은 options[].startTime이 null/빈 문자열이거나 `HH:mm` 형식이 아니면 오류를 반환해야 한다.
- **FR-007**: 시스템은 options[].endDate가 null/빈 문자열이거나 `yyyy-MM-dd` 형식이 아니면 오류를 반환해야 한다.
- **FR-008**: 시스템은 options[].endTime이 null/빈 문자열이거나 `HH:mm` 형식이 아니면 오류를 반환해야 한다.
- **FR-009**: 시스템은 options[].region이 null/빈 문자열이면 `"지역을 입력해 주세요."` 오류를 반환해야 한다.
- **FR-010**: 시스템은 options[].region이 `GN`, `HD` 외의 값이면 `"지역은 GN 또는 HD만 입력 가능합니다."` 오류를 반환해야 한다.
- **FR-011**: 시스템은 discounts[].type이 `E`, `S` 외의 값이면 오류를 반환해야 한다.
- **FR-012**: 시스템은 contacts[].type이 `Y`, `K`, `W`, `I`, `L`, `M` 외의 값이면 오류를 반환해야 한다.
- **FR-013**: 시스템은 notices[].type이 `L`, `T`, `R`, `N`, `U` 외의 값이면 오류를 반환해야 한다.

**appRequest 레이어 (비즈니스 규칙 검증)**

- **FR-014**: 시스템은 instructorLo와 instructorLa가 모두 null이면 `"남성 강사 또는 여성 강사 중 하나는 반드시 입력해야 합니다."` 오류를 반환해야 한다.
- **FR-015**: 시스템은 instructorLo에 해당하는 Profile이 존재하지 않으면 `INSTRUCTOR_NOT_FOUND` 오류를 반환해야 한다.
- **FR-016**: 시스템은 instructorLo의 isInstructor=false이면 `INSTRUCTOR_NOT_VALID` 오류를 반환해야 한다.
- **FR-017**: 시스템은 instructorLo의 sex=F이면 `"남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다."` 오류를 반환해야 한다.
- **FR-018**: 시스템은 instructorLa에 해당하는 Profile이 존재하지 않으면 `INSTRUCTOR_NOT_FOUND` 오류를 반환해야 한다.
- **FR-019**: 시스템은 instructorLa의 isInstructor=false이면 `INSTRUCTOR_NOT_VALID` 오류를 반환해야 한다.
- **FR-020**: 시스템은 instructorLa의 sex=M이면 `"여성 강사(instructorLa)에는 여성(F) 프로필만 등록 가능합니다."` 오류를 반환해야 한다.
- **FR-021**: 시스템은 options[] 각 항목에서 startDate+startTime이 endDate+endTime보다 이후이거나 같으면 `"시작 일시는 종료 일시보다 이전이어야 합니다."` 오류를 반환해야 한다.
- **FR-022**: 시스템은 discounts[].type=E이고 condition이 `yyyy-MM-dd` 형식이 아니면 `"얼리버드 할인 조건은 yyyy-MM-dd 형식의 날짜여야 합니다."` 오류를 반환해야 한다.
- **FR-023**: 시스템은 discounts[].type=S이고 condition이 `M`, `F` 외의 값이면 `"성별 할인 조건은 M 또는 F만 입력 가능합니다."` 오류를 반환해야 한다.

**정상 처리**

- **FR-024**: 모든 검증을 통과한 요청에 대해 레슨을 저장하고 201 Created와 `{"id": <생성된 레슨 id>}`를 반환해야 한다.
- **FR-025**: isActive 값을 입력하지 않으면 기본값 `true`로 저장해야 한다.
- **FR-026**: 오류가 복수인 경우 `errors` 배열에 모든 오류를 담아 반환해야 한다.

### Key Entities

- **Lesson**: 레슨 기본 정보 (제목, 장르, 강사, 수강료, 활성 여부)
- **LessonOption**: 수업 일정 옵션 (시작/종료 일시, 지역, 장소). 레슨당 1개 이상 필수
- **LessonDiscount**: 할인 정책 (타입별 조건 및 금액). 선택 항목
- **LessonAccount**: 입금 계좌 정보. 레슨당 1개, 선택 항목
- **LessonContact**: 연락처 정보 (유튜브, 카카오톡, 웹, 인스타그램, 라인, 모바일). 선택 항목
- **LessonNotice**: 공지사항 (레슨/시간/지역/일반/긴급 타입). 선택 항목
- **Profile**: 강사로 등록된 사용자 프로필. instructorLo(sex=M)/instructorLa(sex=F)에 참조됨

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 유효한 필수 필드만 포함한 최소 요청으로 레슨 생성이 완료되고 레슨 ID가 반환된다.
- **SC-002**: 모든 선택 필드(discounts, account, contacts, notices)를 포함한 요청도 정상 처리된다.
- **SC-003**: 형식 오류(webRequest) 및 비즈니스 규칙 오류(appRequest) 각각에 대해 100% 명확한 오류 메시지가 반환된다.
- **SC-004**: 복수의 검증 오류가 발생하는 경우 하나의 응답에 모든 오류가 포함되어 반환된다.
- **SC-005**: 존재하지 않거나 조건을 충족하지 않는 강사 ID에 대해 적절한 에러 코드(INSTRUCTOR_NOT_FOUND, INSTRUCTOR_NOT_VALID)가 반환된다.

## Assumptions

- 요청자(API 클라이언트)에 대한 인증/인가 처리는 이 기능의 범위 밖이다.
- Profile 조회는 이미 구현된 Profile 도메인을 재사용한다.
- 레슨 ID는 자동 증가(auto increment) Long 타입으로 생성된다.
- options의 startDate/startTime 및 endDate/endTime은 서버에서 LocalDateTime으로 조합하여 저장된다.
- 동일한 강사가 instructorLo와 instructorLa에 동시에 등록되는 케이스는 성별 제약으로 인해 구조적으로 불가하다.
- discounts, account, contacts, notices는 레슨 생성 시 함께 저장되며, 별도 등록 API는 이 기능의 범위 밖이다.
