# 🧶 MyKnitLog - 뜨개질 관리 웹 애플리케이션

## 프로젝트 개요

뜨개러를 위한 도안 위시리스트, 프로젝트 진행 관리, 작업 캘린더 기능을 제공하는 풀스택 웹 애플리케이션.
취업준비용 포트폴리오 겸 실사용 목적으로 개발하며, 추후 React Native 기반 모바일 앱으로 확장 예정.

---

## 기술 스택

### Backend
- **Language**: Java 17
- **Framework**: Spring Boot 3.x
- **ORM**: Spring Data JPA + Hibernate
- **Security**: Spring Security + JWT (Access Token / Refresh Token)
- **Build Tool**: Gradle
- **Database**: MySQL 8.x (운영) / H2 (로컬 개발)

### Frontend
- **Framework**: React 18
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **상태 관리**: Zustand
- **HTTP Client**: Axios
- **라우팅**: React Router v6

### 인프라 (GCP)
- **Backend 배포**: GCP Cloud Run (Docker 컨테이너)
- **Frontend 배포**: Firebase Hosting
- **파일 스토리지**: GCP Cloud Storage (도안 사진, 프로젝트 사진)
- **Database**: Supabase (무료 MySQL 호환 PostgreSQL) 또는 PlanetScale
- **CI/CD**: GitHub Actions → Cloud Run 자동 배포

### 개발 환경
- **IDE**: VSCode (Extension Pack for Java + Spring Boot Extension Pack 설치 권장)
- **API 문서**: Swagger (springdoc-openapi)
- **컨테이너**: Docker / Docker Compose (로컬 환경)
- **버전 관리**: Git / GitHub

---

## 프로젝트 구조

```
myknitlog/
├── backend/                          # Spring Boot
│   ├── src/main/java/com/myknitlog/
│   │   ├── domain/
│   │   │   ├── user/                 # 사용자
│   │   │   ├── pattern/              # 도안 위시리스트
│   │   │   ├── project/              # 프로젝트
│   │   │   ├── calendar/             # 캘린더
│   │   │   └── gauge/               # 게이지 계산
│   │   ├── global/
│   │   │   ├── config/              # Security, CORS, Swagger 설정
│   │   │   ├── exception/           # 전역 예외 처리
│   │   │   ├── jwt/                 # JWT 유틸
│   │   │   └── storage/             # GCP Storage 유틸
│   │   └── MyKnitLogApplication.java
│   ├── Dockerfile
│   └── build.gradle
│
└── frontend/                         # React + TypeScript
    ├── src/
    │   ├── pages/
    │   │   ├── WishList/            # 도안 위시리스트
    │   │   ├── Projects/            # 프로젝트 목록/상세
    │   │   ├── Calendar/            # 작업 캘린더
    │   │   ├── Gauge/               # 게이지 계산기
    │   │   └── Dashboard/           # 통계
    │   ├── components/              # 공통 컴포넌트
    │   ├── api/                     # Axios API 모듈
    │   ├── store/                   # Zustand 전역 상태
    │   └── types/                   # TypeScript 타입 정의
    └── package.json
```

---

## 핵심 도메인 및 기능

### 1. 사용자 인증 (User)
- 이메일 / 비밀번호 회원가입 및 로그인
- JWT Access Token (30분) + Refresh Token (7일)
- 모든 기능은 로그인 후 사용 가능 (인증 필수)

---

### 2. 도안 위시리스트 (Pattern Wishlist)
사고 싶거나 관심 있는 도안을 저장·관리하는 기능.

**저장 정보**
| 필드 | 설명 | 필수 여부 |
|---|---|---|
| 도안 종류 | 코바늘 / 대바늘 | ✅ |
| 도안 카테고리 | 니트 / 가디건 / 하의 / 가방 / 스카프 / 모자 / 인형 / 기타 | ✅ |
| 도안명 | 도안 이름 | ✅ |
| 작가명 | 작가 또는 브랜드명 | ❌ |
| 도안 가격 | 숫자 (원 단위) | ❌ |
| 도안 사진 | GCP Storage 업로드 | ❌ |
| 도안 URL | 구매 링크 | ❌ |
| 메모 | 자유 메모 | ❌ |
| 상태 | WISH / IN_PROGRESS / DONE | ✅ |

**기능**
- 위시 도안 등록 / 수정 / 삭제
- 상태 필터링 (전체 / 위시 / 진행중 / 완성)
- 도안 종류별 필터링
- 위시 → 프로젝트 전환 시 프로젝트 자동 생성

---

### 3. 프로젝트 관리 (Project)
실제 뜨개질 진행 중인 작업을 관리하는 핵심 기능.

**저장 정보**
| 필드 | 설명 |
|---|---|
| 연결 도안 | 위시리스트의 도안 연결 (optional) |
| 프로젝트명 | |
| 시작일 / 완성일 | |
| 상태 | IN_PROGRESS / PAUSED / DONE |
| 메모 | |

**바늘 정보 (1 프로젝트 : N 바늘)**
| 필드 | 설명 |
|---|---|
| 바늘 종류 | 코바늘 / 대바늘 |
| 바늘명 | 제품 이름 |
| 바늘 사이즈 | mm 단위 |
| 용도 메모 | 예: "몸통용", "소매용" (optional) |

**사용 실 정보 (1 프로젝트 : N 실)**
| 필드 | 설명 |
|---|---|
| 실 이름 | 직접 입력 |
| 브랜드명 | 직접 입력 (optional) |
| 색상명 | 직접 입력 (optional) |
| 색상 코드 | HEX 코드 (optional) |
| 용도 메모 | 예: "메인 색상", "포인트 색상" (optional) |

**진행 상황 추적**
- **단수(Row) 체크**: +/- 버튼으로 실시간 증감, 버튼 클릭 즉시 DB 저장 → 페이지 재진입 시 마지막 값 유지
- **줄임 횟수 체크**: 줄임/늘림 패턴별 +/- 버튼으로 반복 횟수 카운트, 버튼 클릭 즉시 DB 저장 → 페이지 재진입 시 마지막 값 유지
- **작업 일지**: 날짜별 작업 기록 (작업 여부 체크 → 작업 일수 자동 집계)
- **사진 타임라인**: 진행 단계별 사진 업로드 (GCP Storage)
- **완성 예상일 계산**: (목표 단수 - 현재 단수) / 일평균 진행 단수

**카운터 동작 방식**
- +/- 버튼 클릭 → 즉시 `PATCH` API 호출 → DB 업데이트
- 프로젝트 페이지 진입 시 `GET` API로 저장된 현재 카운트 값 로드
- 네트워크 오류 시 UI는 낙관적 업데이트(Optimistic Update) 후 실패 시 롤백
- reset 버튼 클릭 시 현재 카운트 0으로 초기화 후 즉시 DB 저장

**카운터 구조**
```
Project
├── RowCounter (단수 카운터)
│   ├── currentRow: int      ← +/- 클릭마다 즉시 저장
│   ├── targetRow: int
│   └── reset() → currentRow = 0, 즉시 저장
└── RepeatCounter[] (줄임/늘림 카운터 목록)
    ├── name: String (예: "소매 줄임")
    ├── currentCount: int    ← +/- 클릭마다 즉시 저장
    ├── targetCount: int
    └── reset() → currentCount = 0, 즉시 저장
```

---

### 4. 캘린더 (Calendar)
작업 일지 날짜를 기반으로 월별 캘린더에 진행 사진을 표시하는 전체 프로젝트 통합 관리 기능.

**동작 방식**
- 월별 캘린더 뷰에서 작업한 날짜에 **대표 사진 썸네일** 표시 (여러 프로젝트면 첫 번째 사진 위에 프로젝트별 설정한 실 색 동그라미 아이콘)
- 사진이 없는 날짜는 작업 완료 마커(점 또는 아이콘)로 표시
- 하루에 여러 프로젝트 작업 가능 → 날짜 클릭 시 해당 날짜에 작업한 **프로젝트 목록 먼저 표시**
- 목록에서 프로젝트 선택 시 → 해당 프로젝트의 진행 상황 상세 모달로 이동

**UI 흐름**
```
캘린더 날짜 클릭
    ├─→ 작업 프로젝트가 1개  → 바로 상세 모달 표시
    └─→ 작업 프로젝트가 2개 이상 → 프로젝트 목록 표시
            ├─→ [프로젝트 A] 선택 → 프로젝트 A 상세 모달
            └─→ [프로젝트 B] 선택 → 프로젝트 B 상세 모달
```

**상세 모달 표시 내용**
| 항목 | 설명 |
|---|---|
| 프로젝트명 | |
| 해당 날짜 업로드 사진 | 전체 표시 |
| 진행 상황 | 해당 시점 기록 |
| 카운터 현황 | 줄임/늘림 카운터 목록 |
| 메모 | 해당 날짜 작업 메모 |

**데이터 구조**
- `PROJECT_LOG`의 `work_date` 기준으로 해당 날짜에 속한 모든 프로젝트 조회
- `PROJECT_PHOTO`의 `taken_at`과 날짜 매핑하여 사진 연결
- 프로젝트 1개 → 바로 상세 모달, 2개 이상 → 목록 먼저 표시

---

### 5. 게이지 계산기 (Gauge Calculator)
별도 저장 없이 계산 기능만 제공 (stateless).

**입력값**
- 내 게이지 코수 (10cm 기준)
- 내 게이지 단수 (10cm 기준)
- 원하는 완성 너비 (cm)
- 원하는 완성 길이 (cm)

**계산 공식**
```
필요한 코수 = (원하는 너비 × 내 게이지 코수) / 10
필요한 단수 = (원하는 길이 × 내 게이지 단수) / 10
```

**추가 기능**
- 도안 게이지 vs 내 게이지 비교 → 코수 보정값 안내
- 계산 결과 프로젝트 메모로 저장 (선택)

---

### 6. 통계 대시보드 (Dashboard)

**제공 통계**
- 전체 완성 작품 수
- 진행 중인 프로젝트 수
- 총 작업 일수 (프로젝트 전체 합산)
- 월별 완성 현황 (Bar Chart)
- 월별 작업 일수 현황 (Bar Chart)

---

## ERD (Entity Relationship)

```
USER
├── id (PK)
├── email (UNIQUE)
├── password (encrypted)
├── nickname
├── created_at
├── create_ip
├── update_at
├── update_ip
└── del_yn

PATTERN (도안 위시리스트)
├── id (PK)
├── user_id (FK → USER)
├── type (CROCHET / KNITTING)
├── category (KNIT / CARDIGAN / BOTTOM / BAG / HAT / SCARF / DOLL / ETC)
├── name
├── author
├── price
├── image_url
├── source_url
├── status (WISH / IN_PROGRESS / DONE)
├── memo
├── created_at
├── create_ip
├── update_at
├── update_ip
└── del_yn

PROJECT (프로젝트)
├── id (PK)
├── user_id (FK → USER)
├── pattern_id (FK → PATTERN, nullable)
├── name
├── status (IN_PROGRESS / PAUSED / DONE)
├── start_date
├── end_date
├── memo
├── created_at
├── create_ip
├── update_at
├── update_ip
└── del_yn

PROJECT_NEEDLE (프로젝트 바늘 - 1:N)
├── id (PK)
├── project_id (FK → PROJECT)
├── type (CROCHET / KNITTING)
├── name
├── size_mm
└── memo (optional)

PROJECT_YARN (프로젝트 사용 실 - 1:N)
├── id (PK)
├── project_id (FK → PROJECT)
├── yarn_name
├── brand (nullable)
├── color_name (nullable)
├── color_hex (nullable)
└── memo (optional)

PROJECT_ROW_COUNTER (단수 카운터 - 프로젝트당 1개)
├── id (PK)
├── project_id (FK → PROJECT, UNIQUE)
├── current_row
└── target_row

PROJECT_REPEAT_COUNTER (줄임/늘림 카운터 - 1:N)
├── id (PK)
├── project_id (FK → PROJECT)
├── name
├── current_count
└── target_count

PROJECT_LOG (작업 일지)
├── id (PK)
├── project_id (FK → PROJECT)
├── work_date (DATE)  -- 하루 1건
└── memo (nullable)   -- 해당 날짜 작업 메모

PROJECT_PHOTO (프로젝트 사진)
├── id (PK)
├── project_id (FK → PROJECT)
├── image_url
├── memo
└── taken_at          -- 캘린더 날짜 매핑 기준
```

---

## API 설계 원칙

- **REST API** 기반, JSON 통신
- **Base URL**: `/api/v1`
- **인증**: 모든 요청에 `Authorization: Bearer {accessToken}` 헤더 필요
- **공통 응답 형식**:
```json
{
  "success": true,
  "data": { },
  "message": "요청이 처리되었습니다.",
  "timestamp": "2025-04-01T12:00:00"
}
```

**주요 엔드포인트**
```
POST   /api/v1/auth/signup
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh

GET    /api/v1/patterns
POST   /api/v1/patterns
PUT    /api/v1/patterns/{id}
DELETE /api/v1/patterns/{id}
PATCH  /api/v1/patterns/{id}/status

GET    /api/v1/projects
POST   /api/v1/projects
GET    /api/v1/projects/{id}
PUT    /api/v1/projects/{id}
DELETE /api/v1/projects/{id}
PATCH  /api/v1/projects/{id}/row                         # 단수 업데이트
PATCH  /api/v1/projects/{id}/row/reset                   # 단수 초기화
POST   /api/v1/projects/{id}/needles                     # 바늘 추가
DELETE /api/v1/projects/{id}/needles/{needleId}
POST   /api/v1/projects/{id}/yarns                       # 사용 실 추가
DELETE /api/v1/projects/{id}/yarns/{yarnId}
POST   /api/v1/projects/{id}/counters                    # 카운터 추가
PATCH  /api/v1/projects/{id}/counters/{counterId}        # 카운터 값 업데이트
PATCH  /api/v1/projects/{id}/counters/{counterId}/reset  # 카운터 초기화
DELETE /api/v1/projects/{id}/counters/{counterId}
POST   /api/v1/projects/{id}/logs                        # 작업 일지 등록
GET    /api/v1/projects/{id}/logs                        # 작업 일지 조회
POST   /api/v1/projects/{id}/photos                      # 사진 업로드
DELETE /api/v1/projects/{id}/photos/{photoId}

GET    /api/v1/calendar?year={year}&month={month}        # 월별 캘린더 데이터 조회
GET    /api/v1/calendar/{date}                           # 특정 날짜 프로젝트 목록 조회
GET    /api/v1/calendar/{date}/projects/{projectId}      # 특정 날짜 프로젝트 상세 조회

GET    /api/v1/gauge/calculate                           # 쿼리 파라미터로 계산 요청

GET    /api/v1/dashboard/stats
```

---

## 개발 규칙

### 공통
- 커밋 메시지: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:` 컨벤션 사용
- 브랜치 전략: `main` / `develop` / `feature/{기능명}`

### Backend
- Controller → Service → Repository 레이어 분리 철저
- DTO 는 record 또는 별도 클래스 사용 (Entity 직접 노출 금지)
- 예외는 커스텀 Exception + `@RestControllerAdvice` 로 통일 처리
- 모든 API는 Swagger 문서화

### Frontend
- 컴포넌트 단위 개발, 재사용 가능한 공통 컴포넌트 분리
- API 호출은 `/api` 디렉토리에서 모듈화
- 환경변수는 `.env` 파일로 관리 (`.env.local` / `.env.production`)

---

## 환경 변수

### Backend (`application.yml`)
```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

jwt:
  secret: ${JWT_SECRET}
  access-expiration: 1800000    # 30분
  refresh-expiration: 604800000 # 7일

gcp:
  storage:
    bucket-name: ${GCP_BUCKET_NAME}
    project-id: ${GCP_PROJECT_ID}
```

### Frontend (`.env`)
```
VITE_API_BASE_URL=https://your-cloud-run-url/api/v1
```

---

## 확장 계획 (Phase 2)

- [ ] React Native 모바일 앱 (기존 REST API 재사용)

---

## 참고 사항

- 이 프로젝트는 **포트폴리오 및 실사용 목적**으로 개발되며, 코드 품질과 설계 원칙을 중요시함
- **모바일 확장**을 항상 고려하여 API는 플랫폼 독립적으로 설계
- GCP 무료 티어 범위 내에서 운영: Cloud Run (월 200만 요청), Firebase Hosting (10GB/월)
- Cloud SQL 대신 **Supabase 또는 PlanetScale 무료 플랜** 사용 권장