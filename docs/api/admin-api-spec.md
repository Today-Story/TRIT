# 관리자 API 명세서 (Admin API Specification)

**Base URL**: `/api/v1/admin`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 개요

Admin API는 시스템 관리자를 위한 관리 기능을 제공합니다. **모든 엔드포인트는 ADMIN 권한 필요**

---

## API 엔드포인트

### 1. 사용자 목록 조회

```http
GET /api/v1/admin/users?page=0&size=20&role=USER&status=ACTIVE
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| role | String | ❌ | USER, CREATOR, COMPANY |
| status | String | ❌ | ACTIVE, SUSPENDED, DELETED |
| searchKeyword | String | ❌ | 이름, 이메일, 아이디 검색 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "userId": 1,
        "loginId": "user123",
        "email": "user@example.com",
        "nickname": "여행러버",
        "role": "USER",
        "status": "ACTIVE",
        "createdAt": "2024-12-01T10:00:00",
        "lastLoginAt": "2025-10-15T09:30:00"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 5230,
    "totalPages": 262
  },
  "message": null,
  "errorCode": null
}
```

### 2. 사용자 상태 변경

```http
PATCH /api/v1/admin/users/{userId}/status
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "status": "SUSPENDED",
  "reason": "서비스 약관 위반",
  "suspensionDays": 7
}
```

**Status 옵션**
- `ACTIVE`: 활성
- `SUSPENDED`: 정지
- `DELETED`: 삭제

### 3. 크리에이터 승인/거절

```http
POST /api/v1/admin/creators/{creatorId}/approve
Cookie: accessToken=eyJ...
```

```http
POST /api/v1/admin/creators/{creatorId}/reject
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body (거절 시)**

```json
{
  "reason": "포트폴리오가 부족합니다."
}
```

### 4. 업체 승인/거절

```http
POST /api/v1/admin/companies/{companyId}/approve
Cookie: accessToken=eyJ...
```

```http
POST /api/v1/admin/companies/{companyId}/reject
Content-Type: application/json
Cookie: accessToken=eyJ...
```

### 5. 상품 승인/거절

```http
POST /api/v1/admin/products/{productId}/approve
Cookie: accessToken=eyJ...
```

```http
POST /api/v1/admin/products/{productId}/reject
Content-Type: application/json
Cookie: accessToken=eyJ...
```

### 6. 신고 목록 조회

```http
GET /api/v1/admin/reports?page=0&size=20&type=COMMENT&status=PENDING
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| type | String | ❌ | COMMENT, REVIEW, CONTENTS, USER |
| status | String | ❌ | PENDING, RESOLVED, REJECTED |

### 7. 신고 처리

```http
POST /api/v1/admin/reports/{reportId}/resolve
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "action": "DELETE_CONTENT",
  "adminNote": "욕설이 포함되어 삭제 조치합니다."
}
```

**Action 옵션**
- `DELETE_CONTENT`: 콘텐츠 삭제
- `SUSPEND_USER`: 사용자 정지
- `WARNING`: 경고
- `NO_ACTION`: 조치 없음

### 8. 대시보드 통계

```http
GET /api/v1/admin/dashboard/stats
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "users": {
      "total": 12500,
      "newToday": 45,
      "newThisWeek": 320,
      "active": 8200
    },
    "products": {
      "total": 1250,
      "pending": 15,
      "active": 1100
    },
    "reservations": {
      "total": 45000,
      "today": 120,
      "thisWeek": 850,
      "completed": 42000
    },
    "revenue": {
      "today": 15000000,
      "thisWeek": 98000000,
      "thisMonth": 350000000
    },
    "reports": {
      "pending": 25,
      "resolved": 180
    }
  },
  "message": null,
  "errorCode": null
}
```

### 9. 공지사항 관리

```http
POST /api/v1/admin/announcements
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "title": "서비스 점검 안내",
  "content": "2025년 1월 20일 02:00~04:00 서비스 점검이 있습니다.",
  "type": "MAINTENANCE",
  "isPinned": true
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| ADMIN_ACCESS_DENIED | 403 | 관리자 권한 없음 |
| USER_NOT_FOUND | 404 | 사용자를 찾을 수 없음 |
| INVALID_STATUS_TRANSITION | 400 | 유효하지 않은 상태 전환 |

---

**문서 작성자**: Ted
