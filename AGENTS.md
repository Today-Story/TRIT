# TRIT Project AGENTS Instructions

본 문서는 OpenAI Codex/Agents가 TRIT 프로젝트에서 **안전하고 효과적으로 작업**하기 위한 규칙과 프로토콜을 정의합니다. 프로젝트는 백엔드(Java/Spring Boot)와 프론트엔드(Next.js/Turborepo) 서브모듈로 구성되어 있으며, 각 서브모듈은 독립적인 AGENTS.md를 보유하고 있습니다.

---

## 0) 프로젝트 개요 및 범위

### 목적
- 반복적인 작업 자동화, 코드 품질 향상, 안전한 리팩토링, 배포 파이프라인 유지
- 풀스택 기능 개발 시 백엔드와 프론트엔드 간 일관성 유지

### 기술 스택
**Backend (TRIT-BE)**
- Java 17, Spring Boot 3.3.9, Gradle, QueryDSL, MapStruct, Redis, WebFlux WebClient, Log4j2
- PostgreSQL, Liquibase, Docker, GitHub Actions, Nginx, k6, Grafana/Loki/Promtail

**Frontend (TRIT-FE)**
- TypeScript, Next.js 15, Turborepo, PNPM workspace
- Apps: platform (사용자), admin (관리자), backoffice (판매자/크리에이터)
- Packages: ui, types, hooks, utils, next-config, tailwind-config

### 레포지토리 구조
```
trit/
├── TRIT-BE/              # Backend submodule
├── TRIT-FE/              # Frontend submodule
├── docs/                 # 프로젝트 전체 문서
├── ARCHITECTURE.md       # 시스템 아키텍처
├── CONSTITUTION.md       # 개발 원칙 및 헌법
└── AGENTS.md            # 본 파일
```

---

## 1) 기본 원칙 (Guardrails)

### Do (해야 할 것)
- 테스트 추가/확장, 정적 분석 개선, 문서 업데이트
- N+1 최적화, 성능 테스트, 백엔드-프론트엔드 API 계약 준수
- Liquibase 변경 시 문서화, CI 파이프라인 개선
- 도메인별 API 명세(`docs/`) 기준으로 개발
- 백엔드와 프론트엔드 DTO 일치 검증

### Don't (하지 말아야 할 것)
- **절대** ProductResponse 구조 변경 금지 (하위 호환성)
- **절대** Enum을 String으로 변환 금지
- **절대** DTO 검증(`@Valid`, `@NotNull`, `@NotBlank`) 생략 금지
- 로그에 비밀키/PII 노출 금지
- 비용/보안 영향 있는 인프라 변경 금지
- 파괴적 스키마 변경(drop, type 변경)은 명시적 승인 필요

---

## 2) 브랜치 & PR 정책

### 브랜치 전략
- **Backend 작업**: `codex/be-<feature-name>` (영문, 하이픈 구분)
- **Frontend 작업**: `codex/fe-<feature-name>` (영문, 하이픈 구분)
- **풀스택 작업**: `codex/fullstack-<feature-name>` (영문, 하이픈 구분)

### PR 베이스 브랜치
- Backend: `TRIT-BE`의 `codex-bot` 브랜치
- Frontend: `TRIT-FE`의 적절한 베이스 브랜치 (main 또는 develop)

### PR 설명 필수 항목
- 변경 사항 요약 및 이유
- 영향받는 도메인 (API, DB, 인프라, UI 컴포넌트)
- 사용/테스트 방법
- Liquibase 변경 시: 마이그레이션 세부사항, 롤백 전략, 배포 순서
- 성능 영향: k6 스크립트/결과 링크
- 프론트엔드: 스크린샷/녹화, 반응형 테스트 결과

### 머지 요구사항
- CI 통과, 최소 1명 리뷰, 테스트/체인지로그 승인

---

## 3) 레포지토리 맵

### Backend 구조 (`TRIT-BE/backend/src/main/java/today/story/backend/`)
주요 도메인:
- `admin`: 사용자/시스템 관리, 역할 할당
- `auth`: 인증/인가 (JWT 발급/검증, 로그인/로그아웃)
- `businesshour`: 영업 시간 관리
- `campaign`: 캠페인 관리 (생성/등록/신청/콘텐츠 제출/승인)
- `comments`: 댓글 기능
- `common`: 공통 유틸, 상수, 기본 엔티티, 글로벌 enum
- `company`: 회사 정보 및 휴일 관리
- `config`: AWS S3, JPA, Redis, Swagger, Security 등
- `contents`: 콘텐츠 관리
- `coupon`: 쿠폰 발급/사용
- `course`: 코스 관리 (WALK/TRANSPORT)
- `creator`: 크리에이터 정보
- `event`: 이벤트
- `facade`: 통합 API (여러 서비스 조합)
- `hashtags`: 해시태그 관리
- `history`: 좋아요/조회 기록
- `holiday`: 공휴일 스케줄러 및 외부 API 통합
- `location`: 위치 관리, ChatGPT 장소명 보정
- `payment`: 결제 (Eromnet PG)
- `place`: 장소 관리
- `playlist`: 플레이리스트
- `product`: 상품 및 일정 관리 (생성/수정/조회/예약)
- `refundpolicy`: 환불 정책
- `report`: 신고/문의
- `reservation`: 예약 관리
- `review`: 리뷰
- `settlement`: 정산
- `subseller`: 서브셀러
- `theme`: 테마별 콘텐츠 그룹
- `users`: 사용자 관리 (회원가입/로그인/마이페이지)
- `validate`: 검증 로직

### Frontend 구조 (`TRIT-FE/`)
**Apps:**
- `apps/platform`: 사용자 대면 앱 (장소, 상품, 예약, 플레이리스트, 채팅 등)
- `apps/admin`: 관리자 앱
- `apps/backoffice`: 판매자/크리에이터 백오피스

**Packages:**
- `packages/ui`: 공통 디자인 시스템 (Button, Modal, Form 등)
- `packages/types`: TypeScript 타입 정의 (백엔드 DTO와 동기화)
- `packages/hooks`: 공통 React 훅 (useAuth, useFetch 등)
- `packages/utils`: 유틸리티 함수 (날짜, 포맷, 검증 등)
- `packages/next-config`, `packages/tailwind-config`: 공유 설정

---

## 4) Agent 작업 프로토콜

### 작업 흐름
1. **분석** – 범위 파악 (코드, DB, CI, 인프라, UI). Guardrails(§1), 코딩 표준(§5, §6), DB 규칙(§8), CI/CD(§9) 확인
2. **계획** – 작은 PR로 분할, 롤백/마이그레이션 단계 정의, 백엔드-프론트엔드 동기화 계획
3. **구현**
   - Backend: 도메인 규칙 준수 (ProductResponse 변경 금지, Enum 유지, DTO 검증 추가)
   - Frontend: 타입 정의 업데이트 (`packages/types`), API 훅 구현/수정
   - API 명세(`docs/`) 기준으로 개발, 불일치 시 문서 업데이트
4. **테스트**
   - Backend: 단위/통합 테스트 추가/수정, k6 성능 테스트 (필요 시)
   - Frontend: 컴포넌트 테스트, Storybook 업데이트, 반응형 확인
5. **커밋** – `codex/be|fe|fullstack-<feature-name>`. 메시지 스타일: `feat|fix|refactor(scope): summary`
6. **PR** – 체크리스트(§14) 사용. 로그/스크린샷/결과 포함
7. **CI/CD** – GitHub Actions 빌드, 테스트, Slack 알림 확인
8. **배포/롤백** – `deploy-script.sh` 사용. 실패 시 즉시 롤백

### 풀스택 개발 시 주의사항
- Backend API 변경 시 **반드시** Frontend 타입/훅 동기화
- `docs/` 하위 API 명세 업데이트
- 예: Product API 변경 → `docs/product-api-spec.md` 업데이트 → `packages/types` 동기화 → `apps/platform` 반영

---

## 5) Backend 코딩 표준

### 아키텍처
- 엄격한 레이어 구조: `controller` → `service` → `repository`
- Controller는 `ResultResponse`/`PageResponse` 반환
- `validate` 패키지의 검증기 사용
- 예외: `common` exception/handler 사용
- DTO ↔ Entity 매핑: MapStruct (service layer)
- 트랜잭션: 기본 `@Transactional(readOnly=true)`, 쓰기는 `false`

### API 규칙
- REST `/api/v1/...`
- 표준 응답: `ResultResponse`, `PageResponse`
- 페이지네이션: `page`, `size`, 때때로 `SortOption` enum

### 로깅
- Log4j2 사용
- 비밀키/토큰/PII 로그 금지

---

## 6) Frontend 코딩 표준

### 아키텍처
- 각 앱은 독립적인 Next.js 프로젝트
- 공통 로직은 `packages/`로 추상화
- 컴포넌트 코로케이션: 관련 파일(test, styles) 함께 배치

### 스타일
- Prettier (2칸 들여쓰기, 작은따옴표), ESLint `simple-import-sort`
- Import: 앱 내 `@/*` 별칭, workspace 패키지 `@repo/*`
- 컴포넌트: PascalCase, 훅: camelCase
- 공유 상수: `packages/utils/src/constants`

### API 통합
- `packages/types`에 백엔드 DTO 미러링
- `packages/hooks`에 API 호출 훅 (예: `useProducts`, `useReservations`)
- Axios/Fetch 래퍼 사용 (에러 핸들링, 인터셉터)

### 테스트
- Vitest (유닛 테스트)
- Storybook (컴포넌트 카탈로그)
- Chromatic (시각적 회귀 테스트)

---

## 7) N+1 및 성능 최적화

### Backend
- 두 단계 페이지네이션 + fetch join
- Setter 사용 자제, 헬퍼 메서드 사용
- 멀티 페치: fetch join 또는 bulk query + 매핑
- 낮은 선택도 컬럼(예: ThemeType) → ID 기반 쿼리 선호
- 전/후: 쿼리 수, 응답 필드, 레이턴시(p95) 확인

### Frontend
- React Query/SWR로 캐싱, 중복 요청 제거
- 무한 스크롤 시 페이지네이션 최적화
- Image 최적화 (Next.js Image, lazy loading)
- Code splitting, dynamic import

---

## 8) Database & Liquibase

### 활성화
```bash
./gradlew -PenableLiquibase=true :backend:bootRun
```

### 규칙
- Master: `liquibase/liquibase-master.yaml` (`includeAll`)
- 기능당 하나의 changeSet, 명확한 `author/id`
- 가능한 항상 rollback 추가
- 파괴적 변경: 2단계 마이그레이션 (추가 → 이중 쓰기 → 마이그레이션 → 전환 → 삭제)

### 배포 순서
`update` → 앱 배포 → 모니터링 → changeSet 정리

### 체크리스트
- [ ] Enum/DTO 호환성 유지
- [ ] Rollback 문서화
- [ ] Lock/다운타임 리스크 평가 및 완화

---

## 9) CI/CD (GitHub Actions)

### 트리거
- PR/Push/Tag
- Slack 알림 (별도 job)

### 빌드
- Backend: Gradle → Docker 이미지 → tag → artifact
- Frontend: Turborepo build → 정적 파일 → 배포

### 배포
- Backend: SSH → deploy script (zero-downtime)
- Frontend: Vercel/S3 + CloudFront
- 측정 + 알림

### 실패 시
- 즉시 롤백 스크립트
- 원인 알림 포함

### PR 필수 출력
- CI 실행 링크, 이미지 태그, 빌드 시간/크기
- 인프라 변경 시: compose/env 차이 요약

---

## 10) API 명세 관리 (`docs/`)

### 구조
```
docs/
├── auth-api-spec.md
├── product-api-spec.md
├── reservation-api-spec.md
├── campaign-api-spec.md
├── coupon-api-spec.md
├── course-api-spec.md
├── hashtags-api-spec.md
├── holiday-api-spec.md
├── refundpolicy-api-spec.md
├── report-api-spec.md
└── subseller-api-spec.md
```

### 규칙
- 각 도메인별 API 엔드포인트, 요청/응답 DTO, 에러 코드 명세
- 한글 작성
- Backend 개발 전 명세 작성 → Frontend는 명세 기준으로 개발
- API 변경 시 **반드시** 명세 업데이트
- PR에 명세 변경 내역 포함

### 명세 포맷
```markdown
# {Domain} API 명세

## 개요
도메인 설명

## 엔드포인트 목록

### 1. {기능명}
- **URL**: `POST /api/v1/...`
- **설명**: ...
- **권한**: USER / CREATOR / COMPANY / ADMIN
- **Request**:
  ```json
  {
    "field": "value"
  }
  ```
- **Response**:
  ```json
  {
    "status": "SUCCESS",
    "message": "...",
    "data": {...}
  }
  ```
- **Error Codes**:
  - `ERROR_CODE_001`: 설명

...
```

---

## 11) 보안 & 비밀 관리

### Backend
- `.env.example` 로컬용. 실제 키 커밋 금지
- 비밀: CI secrets 또는 서버 keychain만 사용
- 외부 호출: timeout, retry, 에러 분류 설정 필수

### Frontend
- 환경 변수: `.env.local` (로컬), Vercel 환경 변수 (배포)
- API 키 노출 금지
- CSP, CORS 적절히 설정

---

## 12) 관찰성 & 런북

### Logs
- Backend: Promtail → Loki. 대시보드/쿼리는 내부 위키 참조
- Frontend: Sentry (에러 추적), Vercel Analytics

### 일반 이슈
- Liquibase lock: unlock + 재시도
- 외부 API 에러: timeout/retry/circuit breaker 확인
- Cache 불일치: 무효화/재로드, TTL 조정

---

## 13) 성능 테스트 (k6)

### 스크립트
- `TRIT-BE/scripts/performance-test-scripts.js`

### 실행
```bash
k6 run TRIT-BE/scripts/performance-test-scripts.js
```

### PR 필수 포함 (성능 관련 시)
- 요청 수, 에러율, p95/p99 레이턴시

---

## 14) 로컬 개발

### Backend
```bash
cd TRIT-BE/backend
../gradlew test
../gradlew test --tests 'today.story.backend.payment.*'
docker compose -f docker-compose.develop.yml up
```

### Frontend
```bash
cd TRIT-FE
pnpm install
pnpm dev                    # 모든 앱 실행
pnpm dev:platform           # platform 앱만
pnpm build
pnpm lint
pnpm format
pnpm check-types
```

---

## 15) 템플릿

### PR 체크리스트
- [ ] 변경 사항 요약 명확
- [ ] 영향받는 도메인 명시
- [ ] 테스트 통과 (단위/통합/E2E)
- [ ] API 명세 업데이트 (변경 시)
- [ ] 백엔드-프론트엔드 타입 동기화 (풀스택 시)
- [ ] 성능 테스트 결과 (필요 시)
- [ ] 스크린샷/녹화 (UI 변경 시)
- [ ] Liquibase changeSet + rollback (DB 변경 시)
- [ ] CI 통과

### Liquibase changeSet 예시
```yaml
- changeSet:
    id: 2025-09-26-add-new-col
    author: codex
    changes:
      - addColumn:
          tableName: sample
          columns:
            - column:
                name: new_col
                type: varchar(64)
    rollback:
      - dropColumn:
          columnName: new_col
          tableName: sample
```

---

## 16) 의사결정 기록 (Light ADR)

주요 결정 기록: YYYY-MM-DD | 결정 | 이유 | 영향

---

## 17) 주요 아키텍처 리마인더

### Backend
- **Auth**: `@Login`이 `AuthInfo` 주입, 일부 엔드포인트는 쿠키에서 JWT 파싱
- **Payments**: Eromnet PG 통합 (서명된 페이로드). `PaymentAutoScheduler`가 대기 중 예약 취소 및 방문 자동 완료 (시간당)
- **Courses**: `CourseCaching`이 WALK/TRANSPORT 경로 캐싱 (haversine 임계값)
- **Coupons**: `CouponIssuanceValidator` 및 enum 제어 (CouponTemplateStatus, CouponIssueMethod, CouponIssuedByType, CouponTargetType)

### Frontend
- **Platform**: 사용자 대면, 장소/상품/예약/플레이리스트/채팅
- **Admin**: 시스템 관리, 사용자/권한 관리
- **Backoffice**: 판매자/크리에이터 대시보드, 상품/예약 관리

> 이 규칙과 충돌하는 변경은 PR에서 명시적 승인 필요.

---

## 18) 서브모듈 특화 가이드

### Backend 작업 시
- `TRIT-BE/AGENTS.md` 참조
- API 변경 시 `docs/` 업데이트 필수
- 배포 전 `deploy-script.sh` 테스트

### Frontend 작업 시
- `TRIT-FE/AGENTS.md` 참조
- `packages/types` 백엔드 DTO와 동기화
- Storybook 업데이트 (UI 변경 시)

### 풀스택 작업 시
1. Backend API 개발 + 테스트
2. `docs/` API 명세 작성/업데이트
3. Frontend 타입 동기화 (`packages/types`)
4. Frontend 훅/컴포넌트 구현
5. 통합 테스트 (E2E)

---

## 19) 커밋 메시지 컨벤션

### 형식
```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Types
- `feat`: 새 기능
- `fix`: 버그 수정
- `refactor`: 리팩토링
- `chore`: 빌드/설정 변경
- `docs`: 문서 업데이트
- `test`: 테스트 추가/수정
- `style`: 코드 스타일 (포맷, 세미콜론 등)
- `perf`: 성능 개선

### Scope 예시
- Backend: `auth`, `product`, `reservation`, `payment` 등
- Frontend: `platform`, `admin`, `backoffice`, `ui`, `types` 등
- 풀스택: `fullstack`, `api`, `e2e` 등

### 예시
```
feat(product): 상품 일정 다중 선택 기능 추가

- ProductScheduleRequest에 scheduleIds 배열 필드 추가
- Frontend ProductBooking 컴포넌트 다중 선택 UI 구현
- API 명세 업데이트 (docs/product-api-spec.md)

Closes #123
```

---

## 20) 문서 우선순위

개발 시 참조 순서:
1. 본 문서 (`AGENTS.md`) - 전체 가이드
2. 서브모듈 `AGENTS.md` - 세부 규칙
3. `docs/` API 명세 - 계약 기준
4. `ARCHITECTURE.md` - 시스템 설계
5. `CONSTITUTION.md` - 개발 철학

---

> 이 문서의 규칙을 위반하는 PR은 자동 거부될 수 있습니다.
> 의문사항은 팀 리드에게 문의하세요.
