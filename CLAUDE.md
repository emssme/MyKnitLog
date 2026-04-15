# MyKnitLog - Claude 개발 지침서

## 프로젝트 개요
뜨개러를 위한 도안 위시리스트, 프로젝트 진행 관리, 작업 캘린더 기능을 제공하는 풀스택 웹 애플리케이션.
포트폴리오 겸 실사용 목적. 추후 React Native 앱으로 확장 예정.

---

## 기술 스택

| 영역 | 기술 |
|---|---|
| Backend | Java 17, Spring Boot 3.x, Spring Data JPA + Hibernate, Spring Security + JWT, Gradle |
| Frontend | React 18, TypeScript, Tailwind CSS, Zustand, Axios, React Router v6 |
| Database | MySQL 8.x (운영) / H2 (로컬 개발) |
| 인프라 | GCP Cloud Run (BE), Firebase Hosting (FE), GCP Cloud Storage (파일 저장) |
| CI/CD | GitHub Actions → Cloud Run 자동 배포 |
| IDE | VSCode |

---

## 패키지 구조

```
com.myknitlog/
├── domain/
│   ├── user/
│   ├── pattern/
│   ├── project/
│   ├── calendar/
│   └── gauge/
└── global/
    ├── config/       # Security, CORS, Swagger 설정
    ├── exception/    # 전역 예외 처리
    ├── jwt/          # JWT 유틸리티
    └── storage/      # GCP Cloud Storage 유틸리티

frontend/src/
├── pages/            # WishList / Projects / Calendar / Gauge / Dashboard
├── components/       # 재사용 공통 컴포넌트
├── api/              # Axios API 모듈 (도메인별 분리)
├── store/            # Zustand 전역 상태
└── types/            # TypeScript 타입 정의
```

---

## 도메인 요약

| 도메인 | 핵심 내용 |
|---|---|
| User | 이메일/비밀번호 로그인. JWT Access Token(30분) + Refresh Token(7일) |
| Pattern | 도안 위시리스트. 종류(CROCHET/KNITTING), 카테고리, 상태(WISH/IN_PROGRESS/DONE) |
| Project | 진행 중인 작업. 바늘(1:N), 실(1:N), 단수 카운터, 줄임 카운터, 작업 일지, 사진 관리 |
| Calendar | 월별 작업 현황. 전체 프로젝트 통합 표시. PROJECT_LOG.work_date 기준 |
| Gauge | 게이지 계산기. stateless (DB 저장 없음) |
| Dashboard | 완성 작품 수, 총 작업 일수, 월별 완성/작업 통계 |

---

## 핵심 테이블

`USER` / `PATTERN` / `PROJECT` / `PROJECT_NEEDLE` / `PROJECT_YARN`
`PROJECT_ROW_COUNTER` / `PROJECT_REPEAT_COUNTER` / `PROJECT_LOG` / `PROJECT_PHOTO`

---

## API 규칙

- Base URL: `/api/v1`
- 인증: 모든 요청에 `Authorization: Bearer {accessToken}` 헤더 필수
- 공통 응답 형식 (ApiResponse 공통 클래스로 감싸서 반환):
```json
{ "success": true, "data": {}, "message": "요청이 처리되었습니다.", "timestamp": "2025-04-01T12:00:00" }
```

---

## 코드 작성 규칙 (반드시 준수)

### Backend
- **레이어 분리**: Controller → Service → Repository, 레이어 간 직접 참조 금지
- **DTO**: 요청/응답은 반드시 별도 DTO 클래스 사용. Entity를 Controller에서 직접 반환 금지
- **예외 처리**: 커스텀 Exception 클래스 정의 후 `@RestControllerAdvice`에서 통일 처리
- **소프트 삭제**: 실제 DELETE 쿼리 사용 금지. `del_yn = 'Y'` 업데이트 방식으로 처리
- **감사 컬럼**: USER, PATTERN, PROJECT 엔티티에는 `createIp`, `updateAt`, `updateIp`, `delYn` 포함
- **API 문서**: 모든 Controller 메서드에 Swagger 어노테이션 작성
- **카운터 저장**: +/- 버튼 클릭마다 즉시 PATCH API 호출하여 DB 저장 (별도 저장 버튼 없음)

### Frontend
- **API 호출**: 반드시 `/api` 디렉토리 내 모듈에서만 호출. 컴포넌트에서 직접 axios 호출 금지
- **상태 관리**: 서버 데이터는 API 모듈에서 관리, 전역 UI 상태만 Zustand 사용
- **카운터 UX**: +/- 클릭 시 Optimistic Update 적용. 네트워크 오류 시 이전 값으로 롤백
- **환경변수**: `.env.local` (개발) / `.env.production` (운영) 분리

### 공통
- **커밋 메시지**: `feat:` / `fix:` / `refactor:` / `docs:` / `chore:` 컨벤션 준수
- **브랜치 전략**: `main` / `develop` / `feature/{기능명}`