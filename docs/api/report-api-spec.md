# 신고 API 명세서 (Report API Specification)

**Base URL**: `/api/v1/report` ⚠️  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15  
**구현 상태**: 🚧 부분 구현 (신고 등록만 구현됨)

---

## ⚠️ 중요 공지

현재 **신고 등록 기능만 구현**되어 있습니다.
아래 표시된 API 중 🚧 표시가 있는 기능은 향후 구현 예정입니다.

---

## 개요

Report API는 부적절한 콘텐츠, 사용자, 리뷰 등을 신고하는 기능을 제공합니다.

### 현재 구현된 기능
- ✅ 콘텐츠/댓글/리뷰/사용자/상품 신고

### 🚧 향후 구현 예정
- 내 신고 내역 조회
- 신고 상세 조회
- 신고 취소
- 신고 통계
- 관리자 신고 관리 기능

---

## API 엔드포인트

### 1. 신고하기 ✅

```http
POST /api/v1/report
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "targetType": "CONTENTS",
  "targetId": 123,
  "reason": "INAPPROPRIATE",
  "detailedReason": "폭력적이고 선정적인 내용이 포함되어 있습니다."
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| targetType | String | ✅ | CONTENTS, COMMENT, REVIEW, USER, PRODUCT |
| targetId | Long | ✅ | 신고 대상 ID |
| reason | String | ✅ | 신고 사유 (아래 참조) |
| detailedReason | String | ✅ | 상세 사유 (10-500자) |

**신고 사유 (reason)**
- `ABUSE`: 욕설/비방/혐오 표현
- `SPAM`: 스팸/광고
- `INAPPROPRIATE`: 부적절한 내용 (폭력/선정성)
- `COPYRIGHT`: 저작권 침해
- `FRAUD`: 사기/허위 정보
- `PRIVACY`: 개인정보 유출
- `OTHER`: 기타

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reportId": 1,
    "targetType": "CONTENTS",
    "targetId": 123,
    "reason": "INAPPROPRIATE",
    "detailedReason": "폭력적이고 선정적인 내용이 포함되어 있습니다.",
    "status": "PENDING",
    "createdAt": "2025-01-15T10:30:00"
  },
  "message": "신고가 접수되었습니다.",
  "errorCode": null
}
```

**실패 - 이미 신고한 대상 (409 CONFLICT)**

```json
{
  "success": false,
  "data": null,
  "message": "이미 신고한 대상입니다.",
  "errorCode": "ALREADY_REPORTED"
}
```

---

## 🚧 향후 구현 예정 API

아래 API들은 현재 백엔드에서 구현되지 않았으며, 향후 추가될 예정입니다.

### 2. 내 신고 내역 조회 🚧

```http
GET /api/v1/report/me?page=0&size=20&status=PENDING
Cookie: accessToken=eyJ...
```

### 3. 신고 상세 조회 🚧

```http
GET /api/v1/report/{reportId}
Cookie: accessToken=eyJ...
```

### 4. 신고 취소 🚧

```http
DELETE /api/v1/report/{reportId}
Cookie: accessToken=eyJ...
```

### 5. 신고 통계 조회 (본인) 🚧

```http
GET /api/v1/report/me/stats
Cookie: accessToken=eyJ...
```

---

## 관리자 전용 API 🚧

### 6. 모든 신고 목록 조회 (관리자) 🚧

```http
GET /api/v1/report/admin/all?page=0&size=20&status=PENDING
Cookie: accessToken=eyJ...
```

### 7. 신고 처리 (관리자) 🚧

```http
POST /api/v1/report/{reportId}/resolve
Content-Type: application/json
Cookie: accessToken=eyJ...
```

### 8. 신고 거절 (관리자) 🚧

```http
POST /api/v1/report/{reportId}/reject
Content-Type: application/json
Cookie: accessToken=eyJ...
```

---

## 데이터 모델

### ReportResponse

```typescript
interface ReportResponse {
  reportId: number;
  targetType: TargetType;
  targetId: number;
  reason: ReportReason;
  detailedReason: string;
  status: ReportStatus;
  createdAt: string;
}

type TargetType = 'CONTENTS' | 'COMMENT' | 'REVIEW' | 'USER' | 'PRODUCT';
type ReportReason = 'ABUSE' | 'SPAM' | 'INAPPROPRIATE' | 'COPYRIGHT' | 'FRAUD' | 'PRIVACY' | 'OTHER';
type ReportStatus = 'PENDING' | 'IN_REVIEW' | 'RESOLVED' | 'REJECTED';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| REPORT_NOT_FOUND | 404 | 신고를 찾을 수 없음 |
| TARGET_NOT_FOUND | 404 | 신고 대상을 찾을 수 없음 |
| ALREADY_REPORTED | 409 | 이미 신고한 대상 |
| CANNOT_REPORT_SELF | 400 | 자신의 콘텐츠는 신고 불가 |
| INVALID_REPORT_REASON | 400 | 유효하지 않은 신고 사유 |
| DETAILED_REASON_TOO_SHORT | 400 | 상세 사유가 너무 짧음 (10자 미만) |

---

## 비즈니스 로직

### 신고 처리 흐름

1. **신고 접수**: 사용자가 부적절한 콘텐츠 신고
2. **중복 검사**: 동일 사용자가 동일 대상을 이미 신고했는지 확인
3. **저장**: 신고 정보를 DB에 저장 (상태: PENDING)
4. **(향후) 관리자 검토**: 관리자가 신고 내용 확인
5. **(향후) 조치**: 콘텐츠 삭제, 사용자 경고/정지 등

### 신고 우선순위 (향후 구현 예정)

시스템이 자동으로 신고의 우선순위를 결정할 예정:

- **URGENT**: 
  - 범죄/폭력 관련
  - 개인정보 유출
  - 동일 대상에 대한 신고가 5건 이상

- **HIGH**:
  - 사기/허위 정보
  - 저작권 침해

- **MEDIUM**:
  - 욕설/비방
  - 부적절한 내용

- **LOW**:
  - 스팸/광고
  - 기타

---

**문서 작성자**: Ted  
**문의**: backend-team@trit.today

**개발 로드맵**:
- ✅ Phase 1: 신고 등록 기능 (완료)
- 🚧 Phase 2: 신고 조회 및 관리 기능 (개발 예정)
- 🚧 Phase 3: 관리자 대시보드 (개발 예정)
