# 신고 API 명세서 (Report API Specification)

**Base URL**: `/api/v1/reports`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Report API는 부적절한 콘텐츠, 사용자, 리뷰 등을 신고하는 기능을 제공합니다.

### 주요 기능
- 콘텐츠/댓글/리뷰/사용자 신고
- 내 신고 내역 조회
- 신고 처리 상태 확인
- 신고 취소

---

## API 엔드포인트

### 1. 신고하기

```http
POST /api/v1/reports
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "targetType": "CONTENTS",
  "targetId": 123,
  "reason": "INAPPROPRIATE",
  "detailedReason": "폭력적이고 선정적인 내용이 포함되어 있습니다.",
  "evidenceUrls": [
    "https://s3.amazonaws.com/trit/reports/evidence1.jpg",
    "https://s3.amazonaws.com/trit/reports/evidence2.jpg"
  ]
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| targetType | String | ✅ | CONTENTS, COMMENT, REVIEW, USER, PRODUCT |
| targetId | Long | ✅ | 신고 대상 ID |
| reason | String | ✅ | 신고 사유 (아래 참조) |
| detailedReason | String | ✅ | 상세 사유 (10-500자) |
| evidenceUrls | Array | ❌ | 증거 자료 URL 배열 (최대 5개) |

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
    "status": "PENDING",
    "createdAt": "2025-01-15T10:30:00"
  },
  "message": "신고가 접수되었습니다. 검토 후 조치하겠습니다.",
  "errorCode": null
}
```

**실패 - 이미 신고한 대상 (409 CONFLICT)**

```json
{
  "success": false,
  "data": null,
  "message": "이미 신고한 대상입니다. 중복 신고는 불가능합니다.",
  "errorCode": "ALREADY_REPORTED"
}
```

---

### 2. 내 신고 내역 조회

```http
GET /api/v1/reports/me?page=0&size=20&status=PENDING
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| status | String | ❌ | PENDING, IN_REVIEW, RESOLVED, REJECTED |
| targetType | String | ❌ | 신고 대상 타입 필터 |
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 20) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "reportId": 1,
        "targetType": "CONTENTS",
        "targetId": 123,
        "targetTitle": "부적절한 콘텐츠 제목",
        "reason": "INAPPROPRIATE",
        "detailedReason": "폭력적이고 선정적인 내용이 포함되어 있습니다.",
        "status": "RESOLVED",
        "adminNote": "신고하신 콘텐츠는 삭제 조치되었습니다.",
        "actionTaken": "DELETE_CONTENT",
        "createdAt": "2025-01-15T10:30:00",
        "resolvedAt": "2025-01-16T14:20:00"
      },
      {
        "reportId": 2,
        "targetType": "COMMENT",
        "targetId": 456,
        "targetTitle": "댓글 내용 일부...",
        "reason": "ABUSE",
        "detailedReason": "욕설이 포함된 댓글입니다.",
        "status": "PENDING",
        "adminNote": null,
        "actionTaken": null,
        "createdAt": "2025-01-16T09:15:00",
        "resolvedAt": null
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 8,
    "totalPages": 1
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 신고 상세 조회

```http
GET /api/v1/reports/{reportId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reportId": 1,
    "reporter": {
      "userId": 10,
      "nickname": "여행러버"
    },
    "targetType": "CONTENTS",
    "targetId": 123,
    "targetInfo": {
      "title": "부적절한 콘텐츠 제목",
      "author": "문제작성자",
      "createdAt": "2025-01-10T10:00:00"
    },
    "reason": "INAPPROPRIATE",
    "detailedReason": "폭력적이고 선정적인 내용이 포함되어 있습니다.",
    "evidenceUrls": [
      "https://s3.amazonaws.com/trit/reports/evidence1.jpg"
    ],
    "status": "RESOLVED",
    "adminNote": "신고하신 콘텐츠는 삭제 조치되었습니다.",
    "actionTaken": "DELETE_CONTENT",
    "reviewedBy": "관리자1",
    "createdAt": "2025-01-15T10:30:00",
    "resolvedAt": "2025-01-16T14:20:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 4. 신고 취소

신고 접수 후 24시간 이내, 처리되기 전에만 취소 가능합니다.

```http
DELETE /api/v1/reports/{reportId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "신고가 취소되었습니다.",
  "errorCode": null
}
```

**실패 - 이미 처리된 신고 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "이미 처리된 신고는 취소할 수 없습니다.",
  "errorCode": "REPORT_ALREADY_PROCESSED"
}
```

**실패 - 취소 기한 초과 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "신고 접수 후 24시간이 지나 취소할 수 없습니다.",
  "errorCode": "CANCEL_DEADLINE_PASSED"
}
```

---

### 5. 신고 통계 조회 (본인)

```http
GET /api/v1/reports/me/stats
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "totalReports": 15,
    "pending": 3,
    "inReview": 2,
    "resolved": 8,
    "rejected": 2,
    "byTargetType": {
      "CONTENTS": 6,
      "COMMENT": 5,
      "REVIEW": 3,
      "USER": 1
    },
    "byReason": {
      "ABUSE": 7,
      "INAPPROPRIATE": 4,
      "SPAM": 3,
      "OTHER": 1
    },
    "successRate": 53.3
  },
  "message": null,
  "errorCode": null
}
```

---

## 관리자 전용 API

### 6. 모든 신고 목록 조회 (관리자)

```http
GET /api/v1/reports/admin/all?page=0&size=20&status=PENDING
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "reportId": 1,
        "reporter": {
          "userId": 10,
          "nickname": "여행러버"
        },
        "targetType": "CONTENTS",
        "targetId": 123,
        "targetTitle": "부적절한 콘텐츠",
        "reason": "INAPPROPRIATE",
        "status": "PENDING",
        "createdAt": "2025-01-15T10:30:00",
        "priority": "HIGH"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 125,
    "totalPages": 7
  },
  "message": null,
  "errorCode": null
}
```

---

### 7. 신고 처리 (관리자)

```http
POST /api/v1/reports/{reportId}/resolve
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "action": "DELETE_CONTENT",
  "adminNote": "신고하신 콘텐츠는 커뮤니티 가이드라인 위반으로 삭제 조치되었습니다.",
  "notifyReporter": true,
  "additionalActions": [
    {
      "type": "SUSPEND_USER",
      "targetUserId": 50,
      "duration": 7,
      "reason": "반복적인 부적절한 콘텐츠 게시"
    }
  ]
}
```

**처리 조치 (action)**
- `DELETE_CONTENT`: 콘텐츠 삭제
- `SUSPEND_USER`: 사용자 정지
- `WARNING`: 경고 부여
- `NO_ACTION`: 조치 없음 (신고 기각)
- `CONTENT_EDIT`: 콘텐츠 수정 요청

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reportId": 1,
    "status": "RESOLVED",
    "actionTaken": "DELETE_CONTENT",
    "resolvedAt": "2025-01-16T14:20:00"
  },
  "message": "신고가 처리되었습니다.",
  "errorCode": null
}
```

---

### 8. 신고 거절 (관리자)

```http
POST /api/v1/reports/{reportId}/reject
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "reason": "신고하신 내용은 커뮤니티 가이드라인에 위반되지 않습니다.",
  "notifyReporter": true
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reportId": 1,
    "status": "REJECTED",
    "resolvedAt": "2025-01-16T14:20:00"
  },
  "message": "신고가 거절되었습니다.",
  "errorCode": null
}
```

---

## 데이터 모델

### ReportResponse

```typescript
interface ReportResponse {
  reportId: number;
  reporter: {
    userId: number;
    nickname: string;
  };
  targetType: TargetType;
  targetId: number;
  targetTitle: string;
  targetInfo: {
    title: string;
    author: string;
    createdAt: string;
  };
  reason: ReportReason;
  detailedReason: string;
  evidenceUrls: string[];
  status: ReportStatus;
  adminNote: string | null;
  actionTaken: ActionType | null;
  reviewedBy: string | null;
  createdAt: string;
  resolvedAt: string | null;
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'URGENT';
}

type TargetType = 'CONTENTS' | 'COMMENT' | 'REVIEW' | 'USER' | 'PRODUCT';
type ReportReason = 'ABUSE' | 'SPAM' | 'INAPPROPRIATE' | 'COPYRIGHT' | 'FRAUD' | 'PRIVACY' | 'OTHER';
type ReportStatus = 'PENDING' | 'IN_REVIEW' | 'RESOLVED' | 'REJECTED';
type ActionType = 'DELETE_CONTENT' | 'SUSPEND_USER' | 'WARNING' | 'NO_ACTION' | 'CONTENT_EDIT';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| REPORT_NOT_FOUND | 404 | 신고를 찾을 수 없음 |
| TARGET_NOT_FOUND | 404 | 신고 대상을 찾을 수 없음 |
| ALREADY_REPORTED | 409 | 이미 신고한 대상 |
| CANNOT_REPORT_SELF | 400 | 자신의 콘텐츠는 신고 불가 |
| REPORT_ALREADY_PROCESSED | 400 | 이미 처리된 신고 |
| CANCEL_DEADLINE_PASSED | 400 | 취소 기한 초과 (24시간) |
| INVALID_REPORT_REASON | 400 | 유효하지 않은 신고 사유 |
| DETAILED_REASON_TOO_SHORT | 400 | 상세 사유가 너무 짧음 (10자 미만) |
| TOO_MANY_EVIDENCE_FILES | 400 | 증거 파일이 너무 많음 (최대 5개) |

---

## 비즈니스 로직

### 신고 우선순위 자동 결정

시스템이 자동으로 신고의 우선순위를 결정합니다:

- **URGENT**: 
  - 범죄/폭력 관련 (`INAPPROPRIATE` + 특정 키워드)
  - 개인정보 유출 (`PRIVACY`)
  - 동일 대상에 대한 신고가 5건 이상

- **HIGH**:
  - 사기/허위 정보 (`FRAUD`)
  - 저작권 침해 (`COPYRIGHT`)

- **MEDIUM**:
  - 욕설/비방 (`ABUSE`)
  - 부적절한 내용 (`INAPPROPRIATE`)

- **LOW**:
  - 스팸/광고 (`SPAM`)
  - 기타 (`OTHER`)

### 자동 처리 규칙

특정 조건 충족 시 자동으로 처리됩니다:

1. **동일 콘텐츠 10건 이상 신고**: 자동 숨김 처리
2. **욕설 키워드 5개 이상 포함**: 자동 삭제 후 사용자 경고
3. **개인정보 패턴 감지**: 즉시 삭제 및 관리자 알림

### 신고자 신뢰도 점수

악의적인 허위 신고를 방지하기 위해 신고자의 신뢰도를 추적:
- 초기 신뢰도: 100점
- 신고 승인 시: +5점
- 신고 거절 시: -10점
- 신뢰도 50점 미만: 신고 기능 제한

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
