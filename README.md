# TRIT (Travel Recommendation & Integrated Tourism)

> 여행 추천 및 예약 통합 플랫폼

TRIT은 사용자에게 맞춤형 여행 콘텐츠를 제공하고, 여행 상품 예약 및 결제를 통합 관리하는 풀스택 웹 애플리케이션입니다.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Frontend](https://img.shields.io/badge/frontend-Next.js-black)](TRIT-FE)
[![Backend](https://img.shields.io/badge/backend-Spring%20Boot-green)](TRIT-BE)

---

## 📋 목차

- [프로젝트 소개](#-프로젝트-소개)
- [주요 기능](#-주요-기능)
- [기술 스택](#-기술-스택)
- [프로젝트 구조](#-프로젝트-구조)
- [시작하기](#-시작하기)
- [개발 가이드](#-개발-가이드)
- [문서](#-문서)
- [기여하기](#-기여하기)
- [팀](#-팀)

---

## 🎯 프로젝트 소개

TRIT은 여행자, 크리에이터, 여행 업체를 연결하는 통합 플랫폼입니다.

### 핵심 가치

- **맞춤형 추천**: AI 기반 사용자 맞춤 여행 콘텐츠 추천
- **통합 예약**: 다양한 여행 상품의 원스톱 예약 서비스
- **크리에이터 생태계**: 여행 크리에이터가 콘텐츠를 공유하고 수익을 창출
- **비즈니스 지원**: 여행 업체를 위한 상품 관리 및 예약 관리 시스템

---

## ✨ 주요 기능

### 사용자 (User)
- 🔐 회원가입 및 로그인 (일반 / 소셜 로그인)
- 🗺️ 카테고리별 여행 콘텐츠 탐색
- ❤️ 콘텐츠 및 상품 좋아요, 북마크
- 📝 여행 플레이리스트 생성 및 공유
- 🎫 여행 상품 예약 및 결제
- ⭐ 리뷰 작성 및 평점 등록
- 💬 1:1 문의 및 고객 지원

### 크리에이터 (Creator)
- 📸 여행 콘텐츠 제작 및 업로드
- 🏷️ 해시태그 및 장소 태깅
- 📊 콘텐츠 조회수 및 좋아요 통계
- 💰 수익 정산 관리
- 🎯 캠페인 참여 및 협업

### 여행 업체 (Company)
- 🏢 업체 정보 등록 및 관리
- 📦 여행 상품 등록 및 관리
- 📅 스케줄 및 재고 관리
- 📋 예약 현황 실시간 확인
- 💳 결제 및 정산 관리
- 🎟️ 쿠폰 및 프로모션 관리
- 📈 판매 통계 및 리포트

### 관리자 (Admin)
- 👥 사용자 및 권한 관리
- ✅ 크리에이터 및 업체 승인
- 🚨 콘텐츠 모니터링 및 신고 처리
- 📊 시스템 통계 및 대시보드
- 🎨 테마 및 캠페인 관리

---

## 🛠 기술 스택

### Frontend
| 분류 | 기술 스택 |
|------|----------|
| **Framework** | Next.js 14 (App Router), React 18 |
| **Language** | TypeScript |
| **Styling** | TailwindCSS, CSS Modules |
| **State Management** | React Query, Zustand |
| **Monorepo** | Turborepo, PNPM Workspaces |
| **UI Components** | Custom Design System (@repo/ui) |
| **Testing** | Vitest, React Testing Library, Storybook |
| **Code Quality** | ESLint, Prettier, TypeScript |
| **Deployment** | Vercel / AWS CloudFront |

### Backend
| 분류 | 기술 스택 |
|------|----------|
| **Framework** | Spring Boot 3.3.9 |
| **Language** | Java 17 |
| **Build Tool** | Gradle 8.x |
| **Database** | PostgreSQL 15 (AWS RDS) |
| **Cache** | Redis 7.x |
| **ORM** | JPA, QueryDSL, MapStruct |
| **Migration** | Liquibase |
| **Authentication** | JWT, Spring Security |
| **API Docs** | Swagger / OpenAPI 3.0 |
| **Testing** | JUnit 5, Mockito |
| **Monitoring** | Prometheus, Grafana, Loki, Promtail |
| **Deployment** | Docker, AWS EC2, Nginx |
| **CI/CD** | GitHub Actions |
| **Performance** | k6 Load Testing |

### Infrastructure
- **Cloud Provider**: AWS (EC2, RDS, S3, CloudFront)
- **Container**: Docker, Docker Compose
- **Web Server**: Nginx (Load Balancer / Reverse Proxy)
- **Logging**: Loki + Promtail + Grafana
- **Metrics**: Prometheus + cAdvisor + node-exporter
- **External APIs**: 이롬넷 PG, ChatGPT API, 공휴일 API

---

## 📁 프로젝트 구조

```
trit/
├── TRIT-FE/                 # Frontend Monorepo (Submodule)
│   ├── apps/
│   │   ├── platform/       # 사용자용 메인 웹 애플리케이션
│   │   ├── admin/          # 관리자 대시보드
│   │   └── backoffice/     # 내부 운영 도구
│   ├── packages/
│   │   ├── ui/             # 공유 컴포넌트 라이브러리
│   │   ├── types/          # 공유 타입 정의
│   │   ├── hooks/          # 공유 React Hooks
│   │   ├── utils/          # 공유 유틸리티 함수
│   │   ├── next-config/    # Next.js 공통 설정
│   │   └── tailwind-config/# Tailwind 공통 설정
│   └── turbo.json          # Turborepo 설정
│
├── TRIT-BE/                 # Backend Monorepo (Submodule)
│   ├── backend/
│   │   └── src/main/java/today/story/backend/
│   │       ├── admin/      # 관리자 기능
│   │       ├── auth/       # 인증/인가
│   │       ├── campaign/   # 캠페인 관리
│   │       ├── comments/   # 댓글 기능
│   │       ├── contents/   # 콘텐츠 관리
│   │       ├── coupon/     # 쿠폰 관리
│   │       ├── course/     # 코스 관리
│   │       ├── creator/    # 크리에이터 관리
│   │       ├── payment/    # 결제 처리
│   │       ├── playlist/   # 플레이리스트
│   │       ├── product/    # 상품 관리
│   │       ├── reservation/# 예약 관리
│   │       ├── review/     # 리뷰 관리
│   │       ├── theme/      # 테마 관리
│   │       └── users/      # 사용자 관리
│   ├── nginx/              # Nginx 설정
│   └── docker-compose*.yml # Docker Compose 파일들
│
├── docs/                    # 프로젝트 문서
│   ├── api/                # API 명세서
│   └── architecture/       # 아키텍처 문서
│
├── CONSTITUTION.md         # AI Agent 개발 헌장
├── ARCHITECTURE.md         # 시스템 아키텍처 문서
└── README.md              # 이 파일
```

---

## 🚀 시작하기

### 필수 요구사항

#### Frontend
- Node.js 18.x 이상
- PNPM 8.x 이상

#### Backend
- JDK 17
- Gradle 8.x
- Docker & Docker Compose
- PostgreSQL 15 (Docker로 실행 가능)
- Redis 7.x (Docker로 실행 가능)

### 설치 및 실행

#### 1. 저장소 클론

```bash
# 메인 저장소와 서브모듈 함께 클론
git clone --recursive https://github.com/Today-Story/TRIT.git
cd TRIT

# 서브모듈이 클론되지 않은 경우
git submodule update --init --recursive
```

#### 2. Frontend 개발 환경 설정

```bash
cd TRIT-FE

# 의존성 설치
pnpm install

# 환경변수 설정
cp .env.example .env.local
# .env.local 파일을 열어 필요한 환경변수 설정

# 개발 서버 실행
pnpm dev              # 모든 앱 실행
pnpm dev:platform     # Platform 앱만 실행
pnpm dev:admin        # Admin 앱만 실행
```

접속: 
- Platform: http://localhost:3000
- Admin: http://localhost:3001
- Backoffice: http://localhost:3002

#### 3. Backend 개발 환경 설정

##### 방법 1: Docker Compose 사용 (권장)

```bash
cd TRIT-BE

# 개발 환경 전체 실행 (Backend + DB + Redis + Nginx)
docker compose -f docker-compose.develop.yml up -d

# 로그 확인
docker compose -f docker-compose.develop.yml logs -f

# 중지
docker compose -f docker-compose.develop.yml down
```

접속: http://localhost (Nginx를 통한 접속)

##### 방법 2: Gradle 직접 실행

```bash
cd TRIT-BE/backend

# 빌드
./gradlew clean build

# 로컬 프로필로 실행
./gradlew bootRun

# 특정 프로필로 실행
./gradlew bootRun --args='--spring.profiles.active=dev'
```

접속: http://localhost:8080

#### 4. API 문서 확인

백엔드가 실행 중일 때:
- Swagger UI: http://localhost:8080/swagger-ui.html
- OpenAPI JSON: http://localhost:8080/v3/api-docs

---

## 💻 개발 가이드

### Frontend 개발

#### 코드 품질 검사

```bash
cd TRIT-FE

# 린트 검사
pnpm lint

# 코드 포맷팅
pnpm format

# 타입 체크
pnpm check-types

# 전체 빌드
pnpm build
```

#### 컴포넌트 개발

```bash
# Storybook 실행
pnpm --filter @repo/ui storybook

# 컴포넌트 테스트
pnpm --filter @repo/ui test

# 커버리지와 함께 테스트
pnpm --filter @repo/ui vitest run --coverage
```

#### 새 컴포넌트 생성

```bash
pnpm turbo:gen "Generate Component"
```

### Backend 개발

#### 테스트 실행

```bash
cd TRIT-BE/backend

# 전체 테스트
./gradlew test

# 특정 패키지 테스트
./gradlew test --tests 'today.story.backend.product.*'

# 특정 테스트 클래스
./gradlew test --tests 'ProductServiceTest'
```

#### 데이터베이스 마이그레이션

```bash
# Liquibase 활성화하여 실행
./gradlew -PenableLiquibase=true :backend:bootRun

# 마이그레이션 업데이트
./gradlew liquibaseUpdate

# 롤백
./gradlew liquibaseRollback
```

#### 성능 테스트

```bash
# k6 스크립트 실행
k6 run scripts/performance-test-scripts.js

# Docker Compose로 실행
docker compose -f docker-compose.k6.yml run k6 run /scripts/load-test.js
```

### 커밋 컨벤션

#### Frontend
```
[TEAM](type) - 간단한 설명 (#이슈번호)

예시:
[TRIPLE](feat) - 상품 필터링 컴포넌트 추가 (#123)
[TACOS](fix) - 예약 페이지 날짜 선택 버그 수정 (#456)
[TRIPLE](refactor) - 검색 컴포넌트 성능 개선 (#789)
```

#### Backend
```
type(scope): 간단한 설명

예시:
feat(product): 상품 필터링 API 추가
fix(reservation): 예약 날짜 검증 로직 수정
refactor(payment): 결제 서비스 코드 개선
```

### 브랜치 전략

- `main` - 프로덕션 배포 브랜치
- `develop` - 개발 통합 브랜치
- `feat/<feature-name>` - 기능 개발 브랜치 (Frontend)
- `codex/<feature-name>` - AI Agent 작업 브랜치 (Backend)
- `fix/<bug-name>` - 버그 수정 브랜치
- `hotfix/<critical-bug>` - 긴급 수정 브랜치

---

## 📚 문서

- [**CONSTITUTION.md**](./CONSTITUTION.md) - AI Agent 개발 가이드 및 프로젝트 헌장
- [**ARCHITECTURE.md**](./ARCHITECTURE.md) - 시스템 아키텍처 상세 설명
- [**API 문서**](./docs/api/) - 도메인별 API 명세서
  - [인증 API](./docs/api/auth-api-spec.md)
  - [사용자 API](./docs/api/users-api-spec.md)
  - [상품 API](./docs/api/product-api-spec.md)
  - [예약 API](./docs/api/reservation-api-spec.md)
  - [결제 API](./docs/api/payment-api-spec.md)
  - [콘텐츠 API](./docs/api/contents-api-spec.md)
  - [그 외 도메인별 API](./docs/api/)

### 서브모듈 문서

- [Frontend README](./TRIT-FE/README.md)
- [Frontend AGENTS Guide](./TRIT-FE/AGENTS.md)
- [Backend README](./TRIT-BE/README.md)
- [Backend AGENTS Guide](./TRIT-BE/AGENTS.md)

---

## 🤝 기여하기

TRIT 프로젝트에 기여해주셔서 감사합니다!

### 기여 프로세스

1. **이슈 확인**: 작업하기 전에 관련 이슈를 확인하거나 새로 생성
2. **브랜치 생성**: 적절한 브랜치 명명 규칙에 따라 생성
3. **개발**: 코딩 스타일 가이드와 CONSTITUTION.md 준수
4. **테스트**: 모든 테스트 통과 확인
5. **커밋**: 커밋 컨벤션에 맞춰 작성
6. **Pull Request**: 상세한 설명과 함께 PR 생성
7. **코드 리뷰**: 팀원의 리뷰 및 피드백 반영
8. **병합**: 승인 후 메인 브랜치에 병합

### 코드 리뷰 기준

- 코드 품질 (가독성, 유지보수성)
- 테스트 커버리지
- 성능 영향
- 보안 고려사항
- 문서화 업데이트

---

## 👥 팀

### Development Team

- **Frontend Team**: @TRIPLE, @TACOS
- **Backend Team**: Development Team
- **DevOps**: Infrastructure Team

### Communication

- **Slack**: 실시간 커뮤니케이션
- **Notion**: 설계 문서, 아키텍처 결정
- **GitHub**: 코드 리뷰, 기술 논의
- **Jira**: 작업 추적, 스프린트 관리

---

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 있습니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

---

## 📞 문의

프로젝트에 대한 문의사항이 있으시면:
- GitHub Issues를 통해 문의
- 팀 Slack 채널에서 논의

---

**Built with ❤️ by Today-Story Team**
