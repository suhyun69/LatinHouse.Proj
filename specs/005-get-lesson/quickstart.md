# Quickstart: 레슨 단건 조회 구현

## 전제 조건

- 기존 `POST /api/lesson`(003-create-lesson) 구현 완료
- `LessonEntity`, `LessonJpaRepository`, `LessonPersistenceMapper` 존재

## 구현 순서

### 1. LessonNotFoundException 추가

```
common/exception/LessonNotFoundException.java
```

```java
public class LessonNotFoundException extends RuntimeException {
    public LessonNotFoundException(Long lessonNo) {
        super("레슨을 찾을 수 없습니다. id=" + lessonNo);
    }
}
```

### 2. LoadLessonPort 정의

```
lesson/application/port/out/LoadLessonPort.java
```

```java
public interface LoadLessonPort {
    Lesson loadLesson(Long lessonNo);
}
```

### 3. GetLessonUseCase + AppResponse + AppMapper

```
lesson/application/port/in/
├── GetLessonUseCase.java
├── GetLessonAppResponse.java   (정적 내부 클래스: OptionResponse, DiscountResponse, AccountResponse, ContactResponse, NoticeResponse)
└── GetLessonAppMapper.java
```

- `GetLessonAppMapper.toAppResponse(Lesson lesson)`: Lesson 도메인 → GetLessonAppResponse

### 4. GetLessonService

```
lesson/application/service/GetLessonService.java
```

```java
@Service
@RequiredArgsConstructor
public class GetLessonService implements GetLessonUseCase {
    private final LoadLessonPort loadLessonPort;

    @Override
    public GetLessonAppResponse getLesson(Long lessonNo) {
        Lesson lesson = loadLessonPort.loadLesson(lessonNo);
        return GetLessonAppMapper.toAppResponse(lesson);
    }
}
```

### 5. LessonPersistenceAdapter — LoadLessonPort 구현 추가

기존 `LessonPersistenceAdapter`에 `LoadLessonPort` 구현 추가:

```java
@Override
public Lesson loadLesson(Long lessonNo) {
    LessonEntity entity = lessonJpaRepository.findById(lessonNo)
            .orElseThrow(() -> new LessonNotFoundException(lessonNo));
    return LessonPersistenceMapper.toDomain(entity);
}
```

### 6. GetLessonWebResponse + GetLessonWebMapper

```
lesson/adapter/in/web/
├── GetLessonWebResponse.java   (정적 내부 클래스: OptionResponse, DiscountResponse, AccountResponse, ContactResponse, NoticeResponse)
└── GetLessonWebMapper.java
```

- `GetLessonWebMapper.toWebResponse(GetLessonAppResponse)`: AppResponse → WebResponse
- LocalDateTime 분리: `startDateTime.toLocalDate().toString()` → startDate, `startDateTime.toLocalTime().format(HH:mm)` → startTime

### 7. LessonController — GET 엔드포인트 추가

```java
@GetMapping("/lessons/{lessonNo}")
public ResponseEntity<GetLessonWebResponse> getLesson(@PathVariable Long lessonNo) {
    GetLessonWebResponse response = GetLessonWebMapper.toWebResponse(
            getLessonUseCase.getLesson(lessonNo)
    );
    return ResponseEntity.ok(response);
}
```

- 클래스 레벨 `@RequestMapping`을 `/api`로 변경하고 기존 POST에 `@PostMapping("/lesson")` 적용

### 8. GlobalExceptionHandler — LessonNotFoundException 핸들러 추가

```java
@ExceptionHandler(LessonNotFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
public ErrorResponse handleLessonNotFoundException(LessonNotFoundException ex) {
    return ErrorResponse.builder()
            .status(HttpStatus.NOT_FOUND.value())
            .errors(List.of(ErrorResponse.FieldError.builder()
                    .field("lessonNo")
                    .message("레슨을 찾을 수 없습니다.")
                    .build()))
            .build();
}
```

### 9. 테스트 작성

**LessonControllerTest** — 기존 파일에 추가:
- `getLesson_existingId_returns200WithFullBody()`
- `getLesson_notExistingId_returns404()`

**GetLessonServiceTest** (신규):
- `getLesson_found_returnsAppResponse()`
- `getLesson_notFound_throwsLessonNotFoundException()`

## 검증 체크리스트

- [ ] `GET /api/lessons/1` → 200 + 전체 하위 엔티티 포함 응답
- [ ] `GET /api/lessons/9999` → 404 + `{ "status": 404, "errors": [{ "field": "lessonNo", "message": "..." }] }`
- [ ] discounts/contacts/notices 없는 레슨 → 빈 배열 반환
- [ ] account 없는 레슨 → null 반환
- [ ] Swagger UI에서 GET /api/lessons/{lessonNo} 엔드포인트 확인
- [ ] 모든 테스트 통과
