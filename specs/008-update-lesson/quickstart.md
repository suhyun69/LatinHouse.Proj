# Quickstart: 레슨 수정 (PUT /api/lesson/{lessonNo})

**Date**: 2026-06-03

## 구현 요약

`PUT /api/lesson/{lessonNo}` 엔드포인트. 기존 레슨 데이터를 요청 바디로 전체 교체(replace)한다.

## 생성할 파일 목록

### 신규 생성

| 파일 | 위치 |
|------|------|
| `UpdateLessonWebRequest.java` | `adapter/in/web/` |
| `UpdateLessonWebResponse.java` | `adapter/in/web/` |
| `UpdateLessonWebMapper.java` | `adapter/in/web/` |
| `UpdateLessonAppRequest.java` | `application/port/in/` |
| `UpdateLessonAppResponse.java` | `application/port/in/` |
| `UpdateLessonAppMapper.java` | `application/port/in/` |
| `UpdateLessonUseCase.java` | `application/port/in/` |
| `UpdateLessonService.java` | `application/service/` |
| `UpdateLessonControllerTest.java` | `test/.../adapter/in/web/` |
| `UpdateLessonServiceTest.java` | `test/.../application/service/` |

### 수정할 파일

| 파일 | 변경 내용 |
|------|-----------|
| `LessonController.java` | `@PutMapping("/lesson/{lessonNo}")` 메서드 추가 |

## 핵심 패턴 참조

- **WebRequest**: `CreateLessonWebRequest` — 동일 필드·어노테이션 구조
- **Service 검증**: `CreateLessonService` — validateInstructors/OptionDateTimes/DiscountConditions 동일 로직
- **AppMapper toDomain**: `id = appReq.getLessonNo()` 주입 필수 (JPA UPDATE 조건)
- **컬렉션 교체**: `LessonEntity` orphanRemoval=true로 자동 처리

## 검증 체크리스트

- [ ] `PUT /api/lesson/{lessonNo}` 성공 → 200 OK + `{"id": N}`
- [ ] `GET /api/lessons/{lessonNo}` 후 수정된 데이터 확인
- [ ] 존재하지 않는 lessonNo → 404 + `LESSON_NOT_FOUND`
- [ ] title 누락 → 400 + `"제목을 입력해 주세요."`
- [ ] options 빈 배열 → 400 + `"수업 옵션을 1개 이상 입력해 주세요."`
- [ ] instructorLo/La 모두 null → 400 + 강사 필수 에러
- [ ] 강사 성별 불일치 → 400 + `INSTRUCTOR_NOT_VALID`
- [ ] 옵션 endTime < startTime → 400 + `VALIDATION_ERROR`
