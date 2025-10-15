# TRIT API 문서 인덱스

TRIT 프로젝트의 모든 API 명세서 목록입니다.

---

## 📋 목차

### 핵심 API
1. [**인증 (Auth)**](./auth-api-spec.md) - 로그인, 회원가입, 비밀번호 찾기
2. [**사용자 (Users)**](./users-api-spec.md) - 회원 관리, 프로필 수정
3. [**상품 (Product)**](./product-api-spec.md) - 여행 상품 관리
4. [**예약 (Reservation)**](./reservation-api-spec.md) - 예약 생성 및 관리
5. [**결제 (Payment)**](./payment-api-spec.md) - 결제 처리 및 환불

### 콘텐츠 관리
6. [**콘텐츠 (Contents)**](./contents-api-spec.md) - 여행 콘텐츠 작성 및 관리
7. [**댓글 (Comments)**](./comments-api-spec.md) - 댓글 및 대댓글
8. [**리뷰 (Review)**](./review-api-spec.md) - 상품 리뷰 작성
9. [**플레이리스트 (Playlist)**](./playlist-api-spec.md) - 콘텐츠 플레이리스트
10. [**해시태그 (Hashtags)**](./hashtags-api-spec.md) - 해시태그 검색 및 트렌드

### 크리에이터 & 업체
11. [**크리에이터 (Creator)**](./creator-api-spec.md) - 크리에이터 프로필 및 통계
12. [**업체 (Company)**](./company-api-spec.md) - 업체 정보 관리
13. [**캠페인 (Campaign)**](./campaign-api-spec.md) - 마케팅 캠페인
14. [**하위 판매자 (SubSeller)**](./subseller-api-spec.md) - 파트너 판매자 관리

### 프로모션 & 할인
15. [**쿠폰 (Coupon)**](./coupon-api-spec.md) - 쿠폰 발급 및 사용
16. [**이벤트 (Event)**](./event-api-spec.md) - 프로모션 이벤트
17. [**테마 (Theme)**](./theme-api-spec.md) - 큐레이션 테마

### 여행 계획
18. [**코스 (Course)**](./course-api-spec.md) - 여행 코스 추천 및 경로 계산
19. [**장소 (Place)**](./place-api-spec.md) - 여행지 정보
20. [**위치 (Location)**](./location-api-spec.md) - 지역 및 좌표 정보

### 시스템 관리
21. [**관리자 (Admin)**](./admin-api-spec.md) - 시스템 관리
22. [**신고 (Report)**](./report-api-spec.md) - 부적절한 콘텐츠 신고
23. [**공휴일 (Holiday)**](./holiday-api-spec.md) - 공휴일 정보 및 영업일 계산
24. [**환불 정책 (Refund Policy)**](./refundpolicy-api-spec.md) - 상품별 환불 규정

### 부가 기능
25. [**히스토리 (History)**](./history-api-spec.md) - 조회 및 좋아요 기록
26. [**정산 (Settlement)**](./settlement-api-spec.md) - 수익 정산

---

## 🔍 빠른 찾기

### 사용자 유형별 API

#### 일반 사용자 (USER)
- **회원가입/로그인**: [Auth](./auth-api-spec.md), [Users](./users-api-spec.md)
- **여행 계획**: [Course](./course-api-spec.md), [Place](./place-api-spec.md), [Playlist](./playlist-api-spec.md)
- **콘텐츠 탐색**: [Contents](./contents-api-spec.md), [Theme](./theme-api-spec.md), [Hashtags](./hashtags-api-spec.md)
- **상품 예약**: [Product](./product-api-spec.md), [Reservation](./reservation-api-spec.md)
- **결제**: [Payment](./payment-api-spec.md), [Coupon](./coupon-api-spec.md)
- **리뷰 작성**: [Review](./review-api-spec.md), [Comments](./comments-api-spec.md)
- **신고**: [Report](./report-api-spec.md)

#### 크리에이터 (CREATOR)
- **프로필 관리**: [Creator](./creator-api-spec.md), [Users](./users-api-spec.md)
- **콘텐츠 제작**: [Contents](./contents-api-spec.md), [Playlist](./playlist-api-spec.md), [Hashtags](./hashtags-api-spec.md)
- **캠페인 참여**: [Campaign](./campaign-api-spec.md)
- **정산**: [Settlement](./settlement-api-spec.md)

#### 여행 업체 (COMPANY)
- **업체 관리**: [Company](./company-api-spec.md), [Users](./users-api-spec.md)
- **상품 관리**: [Product](./product-api-spec.md), [Refund Policy](./refundpolicy-api-spec.md)
- **예약 관리**: [Reservation](./reservation-api-spec.md), [Holiday](./holiday-api-spec.md)
- **리뷰 답글**: [Review](./review-api-spec.md)
- **캠페인 생성**: [Campaign](./campaign-api-spec.md)
- **파트너 관리**: [SubSeller](./subseller-api-spec.md)
- **정산**: [Settlement](./settlement-api-spec.md)

#### 관리자 (ADMIN)
- **시스템 관리**: [Admin](./admin-api-spec.md)
- **사용자 관리**: [Users](./users-api-spec.md)
- **승인 관리**: [Creator](./creator-api-spec.md), [Company](./company-api-spec.md)
- **신고 처리**: [Report](./report-api-spec.md)
- **테마 큐레이션**: [Theme](./theme-api-spec.md)
- **쿠폰 관리**: [Coupon](./coupon-api-spec.md)
- **공휴일 관리**: [Holiday](./holiday-api-spec.md)

---

## 📊 API 통계

| 분류 | 도메인 수 | 주요 엔드포인트 | 문서 |
|------|----------|---------------|------|
| **핵심 API** | 5개 | 35+ | Auth, Users, Product, Reservation, Payment |
| **콘텐츠** | 5개 | 40+ | Contents, Comments, Review, Playlist, Hashtags |
| **크리에이터 & 업체** | 4개 | 30+ | Creator, Company, Campaign, SubSeller |
| **프로모션** | 3개 | 20+ | Coupon, Event, Theme |
| **여행 계획** | 3개 | 25+ | Course, Place, Location |
| **시스템** | 4개 | 35+ | Admin, Report, Holiday, Refund Policy |
| **기타** | 2개 | 15+ | History, Settlement |

**총 도메인**: 26개  
**총 예상 엔드포인트**: 약 200개

---

## 🔗 공통 정보

### Base URL
```
개발: https://dev-api.trit.today
스테이징: https://staging-api.trit.today
프로덕션: https://api.trit.today
```

### 인증 방식
모든 인증이 필요한 API는 다음 방식으로 JWT 토큰을 전달합니다:

```http
Cookie: accessToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 공통 응답 형식
```typescript
interface ResultResponse<T> {
  success: boolean;
  data: T | null;
  message: string | null;
  errorCode: string | null;
}

interface PageResponse<T> {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
  first: boolean;
  last: boolean;
}
```

### HTTP 상태 코드
- `200 OK`: 성공
- `201 Created`: 생성 성공
- `400 Bad Request`: 잘못된 요청
- `401 Unauthorized`: 인증 필요
- `403 Forbidden`: 권한 없음
- `404 Not Found`: 리소스를 찾을 수 없음
- `409 Conflict`: 중복 또는 충돌
- `429 Too Many Requests`: 요청 횟수 초과
- `500 Internal Server Error`: 서버 오류

### Rate Limiting
- 일반 API: 100 req/min
- 로그인: 5 req/min
- 이메일 발송: 3 req/min

---

## 📝 문서 사용 가이드

### 1. API 테스트
각 API 문서에는 Request/Response 예시가 포함되어 있습니다. 
다음 도구들을 사용하여 테스트할 수 있습니다:

- **Postman**: API 컬렉션 가져오기
- **cURL**: 커맨드라인 테스트
- **Swagger UI**: http://localhost:8080/swagger-ui.html

### 2. 프론트엔드 통합
TypeScript 타입 정의가 각 문서에 포함되어 있습니다.
`@repo/types` 패키지에 해당 타입들을 정의하여 사용하세요.

### 3. 에러 처리
모든 API 문서는 에러 코드 섹션을 포함합니다.
프론트엔드에서는 `errorCode`를 기반으로 사용자 친화적인 메시지를 표시하세요.

---

## 🆕 최근 추가된 API

### 신규 추가 (2025-10-15)
- ✅ **Course API**: 여행 코스 추천 및 경로 계산 (8개 엔드포인트)
- ✅ **Hashtags API**: 해시태그 검색, 트렌드, 자동완성 (9개 엔드포인트)
- ✅ **Holiday API**: 공휴일 정보 및 영업일 계산 (7개 엔드포인트)
- ✅ **Refund Policy API**: 환불 정책 관리 (8개 엔드포인트)
- ✅ **Report API**: 콘텐츠 신고 및 관리 (9개 엔드포인트)
- ✅ **SubSeller API**: 하위 판매자 관리 (11개 엔드포인트)

---

## 🔄 문서 업데이트

이 문서들은 지속적으로 업데이트됩니다.

### 버전 이력
- **v1.0 (2025-10-15)**: 초기 버전 작성 (26개 도메인, 200+ 엔드포인트)

### 업데이트 규칙
- API 변경 시 반드시 문서 업데이트
- Breaking Change는 별도 마이그레이션 가이드 작성
- 버전 정보 명시 (v1.0, v2.0 등)

---

## 💬 문의 및 피드백

API 문서에 대한 문의사항이나 개선 제안이 있으시면:

- **GitHub Issues**: [TRIT Repository](https://github.com/Today-Story/TRIT)
- **이메일**: backend-team@trit.today
- **Slack**: #backend-api 채널

---

## 📖 관련 문서

- [프로젝트 README](../../README.md)
- [시스템 아키텍처](../../ARCHITECTURE.md)
- [AI Agent 헌장](../../CONSTITUTION.md)
- [Frontend AGENTS Guide](../../TRIT-FE/AGENTS.md)
- [Backend AGENTS Guide](../../TRIT-BE/AGENTS.md)

---

**문서 관리자**: Backend Team  
**최종 업데이트**: 2025-10-15  
**문서 개수**: 26개 API 명세서
