# Tasks: 레슨 생성 (POST /api/lesson)

**Input**: Design documents from `specs/003-create-lesson/`

**Prerequisites**: plan.md ✅ spec.md ✅ research.md ✅ data-model.md ✅ contracts/ ✅ quickstart.md ✅

**Organization**: 도메인 → 애플리케이션 → 어댑터 순서로, 4개 User Story(US1–US4) 기준으로 단계적 구현.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 다른 파일 대상, 병렬 실행 가능
- **[Story]**: 해당 User Story 레이블 (US1~US4)
- 모든 경로는 `Latinhouse.Be/src/main/java/com/latinhouse/api/` 기준

---

## Phase 1: Setup

**Purpose**: 신규 패키지 경로 확인 및 기존 코드 파악

- [X] T001 기존 `profile` 도메인 패키지 구조와 `ProfileJpaRepository`, `ProfilePersistenceMapper` 클래스명·경로 확인 (읽기 전용 — 신규 파일 없음)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: 모든 User Story가 의존하는 Domain 객체, JPA Entity, Port 인터페이스, 예외 클래스를 먼저 완성한다.

**⚠️ CRITICAL**: 이 Phase가 완료되기 전까지 어떤 User Story도 시작할 수 없다.

- [X] T002 [P] `Genre.java`, `Region.java`, `DiscountType.java`, `ContactType.java`, `NoticeType.java` Enum 생성 — `lesson/domain/` (각 Enum은 코드값 보유: e.g. `Salsa("S")`, `Gangnam("GN")`, `fromCode(String)` 정적 메서드 포함)
- [X] T003 [P] `LessonOption.java`, `LessonDiscount.java`, `LessonAccount.java`, `LessonContact.java`, `LessonNotice.java` 도메인 클래스 생성 — `lesson/domain/` (Lombok @Getter @Builder, 도메인 타입 사용: Region, DiscountType 등, id 필드 포함)
- [X] T004 `Lesson.java` 루트 도메인 클래스 생성 — `lesson/domain/` (T002·T003 완료 후. Lombok @Getter @Builder. 필드: id:Long, title, genre:Genre, instructorLo, instructorLa, options:List\<LessonOption\>, amount:BigDecimal, discounts:List\<LessonDiscount\>, account:LessonAccount, contacts:List\<LessonContact\>, isActive:Boolean, notices:List\<LessonNotice\>)
- [X] T005 [P] `LessonOptionEntity.java`, `LessonDiscountEntity.java`, `LessonAccountEntity.java`, `LessonContactEntity.java`, `LessonNoticeEntity.java` JPA Entity 생성 — `lesson/adapter/out/persistence/` (@Entity @Table @Id @GeneratedValue, String 타입 코드값 사용, lesson 필드: @ManyToOne/@OneToOne + @JoinColumn(name="lesson_id"), Lombok @Getter @Builder @NoArgsConstructor @AllArgsConstructor)
- [X] T006 `LessonEntity.java` 생성 — `lesson/adapter/out/persistence/` (T005 완료 후. @Entity @Table(name="lesson"). 필드: id:Long(@GeneratedValue AUTO), title, genre(String), instructorLo, instructorLa, amount, isActive. options/discounts/contacts/notices: @OneToMany(mappedBy="lesson", cascade=ALL, orphanRemoval=true). account: @OneToOne(mappedBy="lesson", cascade=ALL). Lombok @Getter @Builder @NoArgsConstructor @AllArgsConstructor)
- [X] T007 `LessonJpaRepository.java` 생성 — `lesson/adapter/out/persistence/` (`extends JpaRepository<LessonEntity, Long>`)
- [X] T008 [P] `SaveLessonPort.java`, `LoadInstructorPort.java` Port 인터페이스 생성 — `lesson/application/port/out/` (SaveLessonPort: `Lesson save(Lesson lesson)`. LoadInstructorPort: `Optional<Profile> findById(String profileId)`, Profile import는 `profile.domain.Profile`)
- [X] T009 [P] `LessonValidationException.java` 생성 — `common/exception/` (`extends RuntimeException`. 생성자: `List<ErrorResponse.FieldError> errors`. getter: `getErrors()`. ErrorResponse import는 `common.exception.ErrorResponse`)

**Checkpoint**: Domain 11개 + JPA Entity 6개 + Port 2개 + Exception 1개 완성. US1 구현 시작 가능.

---

## Phase 3: User Story 1 — 레슨 기본 생성 (Priority: P1) 🎯 MVP

**Goal**: title, genre, instructorLo 또는 instructorLa, options 1개 이상으로 레슨 생성 시 201 + `{"id": N}` 반환.

**Independent Test**: 유효한 남성 강사 ID와 옵션 1개로 POST /api/lesson 요청 → 201 Created + `{"id": 1}` 확인 (비즈니스 검증 없이).

- [X] T010 [P] [US1] `CreateLessonAppRequest.java` 생성 — `lesson/application/port/in/` (Lombok @Getter @Builder. 필드: title, genre:Genre, instructorLo, instructorLa, options:List\<OptionAppReq\>, amount:BigDecimal, discounts:List\<DiscountAppReq\>, account:AccountAppReq, contacts:List\<ContactAppReq\>, isActive:Boolean, notices:List\<NoticeAppReq\>. 정적 내부 클래스: OptionAppReq(startDateTime:LocalDateTime, endDateTime:LocalDateTime, region:Region, place, placeUrl), DiscountAppReq(type:DiscountType, condition, amount), AccountAppReq(bank, account, name), ContactAppReq(type:ContactType, account, name), NoticeAppReq(type:NoticeType, text). 모두 @Getter @Builder)
- [X] T011 [P] [US1] `CreateLessonAppResponse.java` 생성 — `lesson/application/port/in/` (Lombok @Getter @Builder. 필드: id:Long)
- [X] T012 [US1] `CreateLessonAppMapper.java` 생성 — `lesson/application/port/in/` (private 생성자. 정적 메서드: `toDomain(CreateLessonAppRequest)` → Lesson 변환 (sub-entity 리스트 포함, isActive null이면 true로 기본값 처리). `toAppResponse(Lesson)` → CreateLessonAppResponse)
- [X] T013 [US1] `CreateLessonUseCase.java` 인터페이스 생성 — `lesson/application/port/in/` (`CreateLessonAppResponse createLesson(CreateLessonAppRequest request)`)
- [X] T014 [US1] `CreateLessonService.java` 생성 — `lesson/application/service/` (@Service @RequiredArgsConstructor. implements CreateLessonUseCase. 의존: SaveLessonPort. 이 단계에서는 비즈니스 검증 없이 `saveLessonPort.save(CreateLessonAppMapper.toDomain(request))` + `toAppResponse()` 반환만 구현. 트랜잭션: @Transactional)
- [X] T015 [P] [US1] `LessonPersistenceMapper.java` 생성 — `lesson/adapter/out/persistence/` (private 생성자. 정적 메서드: `toEntity(Lesson)` → LessonEntity (sub-entity 리스트 변환 포함, lesson 참조 양방향 설정). `toDomain(LessonEntity)` → Lesson)
- [X] T016 [US1] `LessonPersistenceAdapter.java` 생성 — `lesson/adapter/out/persistence/` (@Repository @RequiredArgsConstructor. implements SaveLessonPort. `save()`: `toEntity()` → `lessonJpaRepository.save()` → `toDomain()` 반환)
- [X] T017 [P] [US1] `CreateLessonWebRequest.java` 생성 — `lesson/adapter/in/web/` (Lombok @Getter. 필드에 Bean Validation 적용: title @NotBlank(message="제목을 입력해 주세요."). genre @NotBlank(message="장르를 입력해 주세요.") + @Pattern(regexp="^[SB]$", message="장르는 S 또는 B만 입력 가능합니다."). options @NotEmpty(message="수업 옵션을 1개 이상 입력해 주세요.") + @Valid List. instructorLo/La/amount/isActive는 @Nullable (검증 없음). 정적 내부 클래스: OptionWebReq(startDate @NotBlank+@Pattern(`^\d{4}-\d{2}-\d{2}$`), startTime @NotBlank+@Pattern(`^\d{2}:\d{2}$`), endDate @NotBlank+@Pattern(`^\d{4}-\d{2}-\d{2}$`), endTime @NotBlank+@Pattern(`^\d{2}:\d{2}$`), region @NotBlank+@Pattern(`^(GN\|HD)$`), place, placeUrl). DiscountWebReq(type @Pattern(`^[ES]$`, message="할인 타입은 E 또는 S만 입력 가능합니다."), condition, amount). ContactWebReq(type @Pattern(`^[YKWILM]$`, message="연락처 타입이 올바르지 않습니다."), account, name). AccountWebReq(bank, account, name). NoticeWebReq(type @Pattern(`^[LTRNU]$`, message="공지 타입이 올바르지 않습니다."), text))
- [X] T018 [P] [US1] `CreateLessonWebResponse.java` 생성 — `lesson/adapter/in/web/` (Lombok @Getter @Builder. 필드: id:Long)
- [X] T019 [US1] `CreateLessonWebMapper.java` 생성 — `lesson/adapter/in/web/` (private 생성자. `toAppRequest(CreateLessonWebRequest)`: options 변환 시 `LocalDateTime.parse(startDate + "T" + startTime)` 형식으로 조합. genre: `Genre.fromCode(web.getGenre())`. region: `Region.fromCode(opt.getRegion())`. discounts: `DiscountType.fromCode(d.getType())` 변환. contacts: `ContactType.fromCode(c.getType())` 변환. notices: `NoticeType.fromCode(n.getType())` 변환. account: null 체크 후 변환. `toWebResponse(CreateLessonAppResponse)`: id 매핑)
- [X] T020 [US1] `LessonController.java` 생성 — `lesson/adapter/in/web/` (@RestController @RequestMapping("/api/lesson") @RequiredArgsConstructor. @Tag(name="Lesson"). POST 메서드: @PostMapping, @Operation(summary="레슨 생성"), @RequestBody @Valid CreateLessonWebRequest, ResponseEntity<CreateLessonWebResponse> 반환, 201 Created. 흐름: `CreateLessonWebMapper.toAppRequest(webReq)` → `createLessonUseCase.createLesson(appReq)` → `CreateLessonWebMapper.toWebResponse(appResp)`)

**Checkpoint**: POST /api/lesson 호출 시 201 + id 반환 확인. title/genre/options 누락 시 400 에러 반환 확인. (instructorLo/La 비즈니스 검증은 US4에서 추가)

---

## Phase 4: User Story 2 — 레슨 부가 정보 등록 (Priority: P2)

**Goal**: discounts, account, contacts, notices 선택 필드를 포함한 요청이 정상 처리된다.

**Independent Test**: 모든 선택 필드를 포함한 POST /api/lesson 요청 → 201 Created 확인.

> **Note**: CreateLessonWebRequest(T017)와 CreateLessonAppRequest(T010)에 이미 모든 서브 엔티티 내부 클래스를 포함했으므로 Phase 3 완료 시 US2 구현도 완성된다. 아래 태스크는 LessonPersistenceMapper(T015)가 모든 서브 엔티티를 올바르게 변환하는지 검증 후 필요 시 보완한다.

- [X] T021 [US2] `LessonPersistenceMapper.java` 검증 및 보완 — `lesson/adapter/out/persistence/` (T015에서 작성한 `toEntity(Lesson)` 메서드가 discounts, account, contacts, notices 서브 엔티티를 모두 변환하는지 확인. account는 null 가능이므로 null 체크 포함. 각 서브 엔티티의 lesson 참조(`entity.setLesson(lessonEntity)` 등)가 양방향으로 설정되는지 확인. 미흡하면 보완)
- [X] T022 [US2] `CreateLessonAppMapper.java` 검증 및 보완 — `lesson/application/port/in/` (T012에서 작성한 `toDomain()` 메서드가 discounts, account(null 허용), contacts, notices를 올바르게 변환하는지 확인. LessonDiscount.type은 DiscountType, LessonContact.type은 ContactType, LessonNotice.type은 NoticeType 사용 확인. 미흡하면 보완)

**Checkpoint**: 선택 필드(discounts 2개, account, contacts, notices) 포함 요청 → 201 확인.

---

## Phase 5: User Story 3 — 입력 형식 오류 처리 (Priority: P3)

**Goal**: 잘못된 형식·빈 값 입력 시 400 Bad Request + 명확한 에러 메시지 반환.

**Independent Test**: title 누락, genre 오류값, options 빈 리스트, startDate 형식 오류 등 각 케이스에서 400 + `errors` 배열 확인.

> **Note**: Bean Validation 어노테이션은 T017(WebRequest)에서 이미 적용됨. 이 Phase에서는 GlobalExceptionHandler가 `MethodArgumentNotValidException`을 올바른 형식으로 처리하는지 확인하고, 누락된 에러 메시지가 있으면 T017 WebRequest를 보완한다.

- [X] T023 [US3] `CreateLessonWebRequest.java` Bean Validation 검토 — `lesson/adapter/in/web/` (T017 결과물 기준: options[].startDate/startTime/endDate/endTime @NotBlank + @Pattern 확인. options[].region @Pattern(`^(GN\|HD)$`) 확인. discounts[].type, contacts[].type, notices[].type @Pattern 확인. `@Valid` 어노테이션이 중첩 리스트에도 전파되는지(`@Valid @NotEmpty List<@Valid OptionWebReq>`) 확인. 미흡하면 보완)
- [X] T024 [US3] `GlobalExceptionHandler.java` 검토 — `common/exception/` (기존 `MethodArgumentNotValidException` 핸들러가 중첩 필드(`options[0].startDate` 등)의 field 경로를 올바르게 반환하는지 확인. `fe.getField()`가 `options[0].startDate` 형태로 반환되는지 확인. 정상이면 변경 없음)

**Checkpoint**: webRequest 계층 검증 에러 전체 케이스 400 반환 확인.

---

## Phase 6: User Story 4 — 비즈니스 규칙 오류 처리 (Priority: P4)

**Goal**: instructorLo/La 강사 조건 위반, 시간 순서 오류, discount condition 오류 시 400 + 구체적 에러 메시지 반환.

**Independent Test**: instructorLo에 sex=F인 프로필 ID 입력 → 400 + `"남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다."` 확인.

- [X] T025 [US4] `InstructorPersistenceAdapter.java` 생성 — `lesson/adapter/out/persistence/` (@Repository @RequiredArgsConstructor. implements LoadInstructorPort. 의존: `ProfileJpaRepository`(profile.adapter.out.persistence), `ProfilePersistenceMapper`(profile.adapter.out.persistence). `findById(String profileId)`: `profileJpaRepository.findById(profileId).map(ProfilePersistenceMapper::toDomain)`)
- [X] T026 [US4] `CreateLessonService.java` 비즈니스 검증 로직 추가 — `lesson/application/service/` (T014 수정. 의존에 LoadInstructorPort 추가. `createLesson()` 내 비즈니스 검증 구현: ① instructorLo+La 모두 null → errors 추가. ② instructorLo 존재 시: findById → empty이면 INSTRUCTOR_NOT_FOUND 추가, isInstructor=false이면 INSTRUCTOR_NOT_VALID 추가, sex≠M이면 INSTRUCTOR_NOT_VALID 추가. ③ instructorLa 존재 시: 대칭 검증(sex=F). ④ options 순회: startDateTime.isBefore(endDateTime) 실패 시 `"options[i].startDateTime"` 필드로 에러 추가. ⑤ discounts 순회: type=E이면 condition이 `\d{4}-\d{2}-\d{2}` 패턴 아닐 시 에러 추가; type=S이면 condition이 "M","F" 외일 시 에러 추가. errors 비어있지 않으면 throw new LessonValidationException(errors))
- [X] T027 [US4] `GlobalExceptionHandler.java` 수정 — `common/exception/` (LessonValidationException 핸들러 추가: @ExceptionHandler(LessonValidationException.class) @ResponseStatus(HttpStatus.BAD_REQUEST). `ex.getErrors()`를 그대로 사용해 ErrorResponse 반환)

**Checkpoint**: 모든 appRequest 비즈니스 규칙 케이스에서 400 + 구체적 에러 메시지 반환. 복수 오류 동시 반환 확인.

---

## Phase 7: Tests

**Purpose**: Controller 슬라이스 테스트 + Service 유닛 테스트

- [X] T028 [P] `LessonControllerTest.java` 생성 — `lesson/adapter/in/web/` (@WebMvcTest(LessonController.class). @MockitoBean: CreateLessonUseCase. 테스트 케이스: ① 정상 생성 → 201 + `{"id":1}`. ② title 누락 → 400 + "제목을 입력해 주세요.". ③ genre 오류값("X") → 400. ④ options 빈 리스트 → 400. ⑤ startDate 형식 오류("20260601") → 400. ⑥ region 오류값("SE") → 400. JSON 요청/응답은 MockMvc + ObjectMapper 사용)
- [X] T029 [P] `CreateLessonServiceTest.java` 생성 — `lesson/application/service/` (Mockito @ExtendWith(MockitoExtension.class). @Mock: SaveLessonPort, LoadInstructorPort. @InjectMocks: CreateLessonService. 테스트 케이스: ① instructorLo+La 모두 null → LessonValidationException. ② instructorLo 프로필 없음 → errors에 "존재하지 않는 강사 ID입니다." 포함. ③ instructorLo isInstructor=false → errors에 "강사로 등록되지 않은 프로필입니다." 포함. ④ instructorLo sex=F → errors에 sex 오류 포함. ⑤ startDateTime >= endDateTime → errors에 datetime 오류 포함. ⑥ discount type=E + condition 형식 오류 → errors에 earlybird 오류 포함. ⑦ discount type=S + condition 오류 → errors에 sex 오류 포함. ⑧ 복수 오류 동시 → errors에 모두 포함. ⑨ 정상 케이스 → saveLessonPort.save() 1회 호출 검증)

---

## Phase 8: Polish & Cross-Cutting Concerns

- [ ] T030 [P] `docs/api-spec.md` 최종 검토 — 구현된 Controller, WebRequest, WebResponse가 명세와 일치하는지 확인. 불일치 시 명세 먼저 수정 후 구현 변경 (Constitution II)
- [ ] T031 서버 기동 후 Swagger UI(`/swagger-ui.html`) 접속 → `/api/lesson` 엔드포인트 `@Tag`, `@Operation` 노출 확인

---

## Dependencies & Execution Order

### Phase Dependencies

```
Phase 1 (Setup)
    └── Phase 2 (Foundational)     ← T002~T009 모두 완료 필요
            └── Phase 3 (US1)      ← T010~T020
                    └── Phase 4 (US2)  ← T021~T022 (Phase 3 완료 후)
                            └── Phase 5 (US3) ← T023~T024
                                    └── Phase 6 (US4) ← T025~T027
                                            └── Phase 7 (Tests) ← T028~T029
                                                    └── Phase 8 (Polish) ← T030~T031
```

### User Story Dependencies

- **US1 (P1)**: Phase 2 완료 후 시작 가능
- **US2 (P2)**: US1 완료 후 (동일 클래스 확장)
- **US3 (P3)**: US2 완료 후 (WebRequest Bean Validation 검토)
- **US4 (P4)**: US3 완료 후 (Service 비즈니스 검증 추가)

### Within Each Phase

- [P] 표시된 태스크는 서로 다른 파일 대상 → 동시 진행 가능
- T004는 T002·T003 완료 후 (Lesson이 Genre, LessonOption 참조)
- T006은 T005 완료 후 (LessonEntity가 서브 Entity 참조)
- T012는 T010·T011 완료 후 (Mapper가 AppRequest/Response 참조)
- T016은 T015 완료 후 (Adapter가 Mapper 사용)
- T019는 T017·T010 완료 후 (WebMapper가 WebRequest→AppRequest 변환)
- T020은 T019 완료 후 (Controller가 WebMapper 사용)

---

## Parallel Example: Phase 2

```
T002 (Enum 5개)  ─────┐
T003 (Domain 5개) ────┤
T005 (SubEntity 5개) ─┤→ T004 (Lesson) → T006 (LessonEntity) → T007 (JpaRepo)
T008 (Port 2개) ──────┤
T009 (Exception) ─────┘
```

## Parallel Example: Phase 3 (US1)

```
T010 (AppRequest)  ─┐
T011 (AppResponse) ─┤→ T012 (AppMapper) → T013 (UseCase) → T014 (Service)
T015 (PersistMapper)─┤→ T016 (Adapter)
T017 (WebRequest)  ─┐
T018 (WebResponse) ─┤→ T019 (WebMapper) → T020 (Controller)
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Phase 1: Setup
2. Phase 2: Foundational (**CRITICAL** — 모든 하위 단계 블로킹)
3. Phase 3: US1 — 기본 레슨 생성 (최소 필수 필드)
4. **STOP & VALIDATE**: POST /api/lesson 요청 → 201 + id 반환 확인
5. 선택: 이후 Phase 순차 추가

### Incremental Delivery

| 완료 Phase | 달성 상태 |
|------------|----------|
| Phase 2 완료 | 도메인·인프라 준비 |
| Phase 3 완료 | MVP: 레슨 생성 동작 |
| Phase 4 완료 | 선택 필드 지원 |
| Phase 5 완료 | webRequest 검증 완성 |
| Phase 6 완료 | 비즈니스 규칙 검증 완성 |
| Phase 7 완료 | 테스트 커버리지 확보 |
| Phase 8 완료 | Swagger 문서화 완성 |

---

## Notes

- [P] 태스크 = 다른 파일 대상, 의존성 없음
- [Story] 레이블 = spec.md의 User Story와 1:1 대응
- Phase 2 완료 전까지 US1 시작 불가
- `InstructorPersistenceAdapter`(T025)는 `profile.adapter.out.persistence` 패키지의 `ProfileJpaRepository`·`ProfilePersistenceMapper`를 import함 (Adapter 간 참조 허용)
- Bean Validation에서 `@Valid`를 리스트 요소에 전파하려면 `@Valid @NotEmpty List<@Valid OptionWebReq> options` 형태 사용
- `Region.fromCode()` 등 `fromCode()` 메서드는 유효하지 않은 코드 입력 시 IllegalArgumentException을 throw할 수 있으나, Bean Validation의 @Pattern이 먼저 차단하므로 WebMapper에서는 발생하지 않음
