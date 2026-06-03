# Quickstart: 랜덤 수업 생성 (POST /api/lesson/random)

**Date**: 2026-06-03

## 실행 방법

```bash
# 서버 실행
cd Latinhouse.Be
./gradlew bootRun

# 랜덤 수업 생성
curl -X POST http://localhost:8080/api/lesson/random

# 응답 예시
# HTTP 201 Created
# {"id": 3}

# 생성된 수업 확인
curl http://localhost:8080/api/lessons/3
```

## 구현 순서

1. `CreateRandomLessonAppResponse` 생성 (`lesson/application/port/in/`)
2. `CreateRandomLessonUseCase` 인터페이스 생성 (`lesson/application/port/in/`)
3. `CreateRandomLessonAppMapper` 생성 — 랜덤 파라미터 조립 로직 (`lesson/application/port/in/`)
4. `CreateRandomLessonService` 구현 — 강사 조회/생성/할당 + UseCase 호출 (`lesson/application/service/`)
5. `CreateRandomLessonWebResponse` 생성 (`lesson/adapter/in/web/`)
6. `CreateRandomLessonWebMapper` 생성 (`lesson/adapter/in/web/`)
7. `LessonController`에 `POST /random` 엔드포인트 추가

## 주요 의존 관계

```
LessonController
  └── CreateRandomLessonUseCase (interface)
        └── CreateRandomLessonService (impl)
              ├── GetProfilesUseCase       (기존, 강사 목록 조회)
              ├── CreateProfileUseCase     (기존, 신규 프로필 생성)
              ├── SetInstructorUseCase     (기존, 강사 지정)
              └── CreateLessonUseCase      (기존, 수업 생성)
```

## 테스트 검증 포인트

| 케이스 | 검증 항목 |
|--------|-----------|
| 강사 존재 | 201 반환, 기존 강사 ID가 instructorLo/La에 할당됨 |
| 남성 강사만 존재 | 201 반환, 여성 강사 자동 생성 후 할당 |
| 강사 없음 | 201 반환, 남성·여성 강사 자동 생성 후 할당 |
| 프로필 생성 실패 | 500 반환 |
