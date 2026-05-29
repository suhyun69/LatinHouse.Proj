# Quickstart: POST /api/lesson 레슨 생성

## 전제 조건

- 기존 `001-create-profile`, `002-patch-profile-instructor` 구현 완료
- `ProfileJpaRepository`, `ProfilePersistenceMapper`, `Profile` 도메인 클래스 사용 가능

## 구현 순서 요약

```
1. Domain 레이어
   Genre, Region, DiscountType, ContactType, NoticeType (Enum 5개)
   LessonOption, LessonDiscount, LessonAccount, LessonContact, LessonNotice (도메인 클래스 5개)
   Lesson (루트 도메인 클래스)

2. Application Port 레이어
   SaveLessonPort, LoadInstructorPort (port/out 인터페이스)
   CreateLessonAppRequest (내부 클래스 포함), CreateLessonAppResponse (port/in DTO)
   CreateLessonAppMapper (port/in Mapper)
   CreateLessonUseCase (port/in 인터페이스)

3. Application Service 레이어
   CreateLessonService (비즈니스 검증 + 저장 흐름)

4. Common Exception
   LessonValidationException (신규)
   GlobalExceptionHandler 수정 (핸들러 추가)

5. Web Adapter 레이어 (adapter/in/web)
   CreateLessonWebRequest (Bean Validation + 내부 클래스 포함)
   CreateLessonWebResponse
   CreateLessonWebMapper (startDate+startTime → LocalDateTime 변환)
   LessonController (POST /api/lesson)

6. Persistence Adapter 레이어 (adapter/out/persistence)
   LessonOptionEntity, LessonDiscountEntity, LessonAccountEntity,
   LessonContactEntity, LessonNoticeEntity, LessonEntity (JPA Entity 6개)
   LessonJpaRepository
   LessonPersistenceMapper
   LessonPersistenceAdapter (SaveLessonPort 구현)
   InstructorPersistenceAdapter (LoadInstructorPort 구현)

7. 테스트
   LessonControllerTest (@WebMvcTest)
   CreateLessonServiceTest (Mockito)
```

## 핵심 코드 패턴

### CreateLessonService — 비즈니스 검증 흐름

```java
@Override
public CreateLessonAppResponse createLesson(CreateLessonAppRequest request) {
    List<ErrorResponse.FieldError> errors = new ArrayList<>();

    // 1. 강사 최소 1명 체크
    if (request.getInstructorLo() == null && request.getInstructorLa() == null) {
        errors.add(FieldError.of("instructorLo",
            "남성 강사 또는 여성 강사 중 하나는 반드시 입력해야 합니다."));
    }

    // 2. instructorLo 검증
    if (request.getInstructorLo() != null) {
        loadInstructorPort.findById(request.getInstructorLo()).ifPresentOrElse(
            profile -> {
                if (!profile.isInstructor())
                    errors.add(FieldError.of("instructorLo", "강사로 등록되지 않은 프로필입니다."));
                else if (profile.getSex() != Sex.M)
                    errors.add(FieldError.of("instructorLo",
                        "남성 강사(instructorLo)에는 남성(M) 프로필만 등록 가능합니다."));
            },
            () -> errors.add(FieldError.of("instructorLo", "존재하지 않는 강사 ID입니다."))
        );
    }

    // 3. instructorLa 검증 (대칭 구조)

    // 4. option 시간 순서 검증
    List<CreateLessonAppRequest.OptionAppReq> options = request.getOptions();
    for (int i = 0; i < options.size(); i++) {
        OptionAppReq opt = options.get(i);
        if (!opt.getStartDateTime().isBefore(opt.getEndDateTime())) {
            errors.add(FieldError.of("options[" + i + "].startDateTime",
                "시작 일시는 종료 일시보다 이전이어야 합니다."));
        }
    }

    // 5. discount condition 검증
    // ...

    if (!errors.isEmpty()) throw new LessonValidationException(errors);

    Lesson lesson = CreateLessonAppMapper.toDomain(request);
    Lesson saved = saveLessonPort.save(lesson);
    return CreateLessonAppMapper.toAppResponse(saved);
}
```

### CreateLessonWebMapper — 날짜+시간 결합

```java
public static CreateLessonAppRequest toAppRequest(CreateLessonWebRequest web) {
    List<OptionAppReq> options = web.getOptions().stream()
        .map(opt -> OptionAppReq.builder()
            .startDateTime(LocalDateTime.parse(opt.getStartDate() + "T" + opt.getStartTime()))
            .endDateTime(LocalDateTime.parse(opt.getEndDate() + "T" + opt.getEndTime()))
            .region(Region.fromCode(opt.getRegion()))
            .place(opt.getPlace())
            .placeUrl(opt.getPlaceUrl())
            .build())
        .toList();
    // ... 나머지 필드 매핑
}
```

### InstructorPersistenceAdapter

```java
// lesson/adapter/out/persistence/InstructorPersistenceAdapter.java
@Repository
@RequiredArgsConstructor
class InstructorPersistenceAdapter implements LoadInstructorPort {

    private final ProfileJpaRepository profileJpaRepository;

    @Override
    public Optional<Profile> findById(String profileId) {
        return profileJpaRepository.findById(profileId)
                .map(ProfilePersistenceMapper::toDomain);
    }
}
```

## 테스트 포인트

### LessonControllerTest (@WebMvcTest)
- 정상 생성 → 201 + `{"id": 1}`
- title 누락 → 400 + 에러 메시지
- options 빈 리스트 → 400 + 에러 메시지
- startDate 형식 오류 → 400 + 에러 메시지
- genre 허용 외 값 → 400 + 에러 메시지

### CreateLessonServiceTest (Mockito)
- instructorLo/La 모두 null → LessonValidationException
- instructorLo 프로필 없음 → errors에 INSTRUCTOR_NOT_FOUND 포함
- instructorLo isInstructor=false → errors에 INSTRUCTOR_NOT_VALID 포함
- instructorLo sex=F → errors에 INSTRUCTOR_NOT_VALID 포함
- startDateTime >= endDateTime → errors에 datetime 오류 포함
- discount type=E, condition 형식 오류 → errors에 포함
- discount type=S, condition 오류 → errors에 포함
- 복수 오류 동시 발생 → errors에 모두 포함
- 정상 케이스 → saveLessonPort.save() 호출 확인
