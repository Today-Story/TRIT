# 환불 정책 API 명세서 (Refund Policy API Specification)

**Base URL**: `/api/v1/refund-policies`, `/api/v1/refund-policy/group`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15  
**구현 상태**: 🚧 부분 구현 (기본 CRUD만 가능)

---

## ⚠️ 중요 공지

현재 **환불 정책 그룹 및 정책의 기본 CRUD만 구현**되어 있습니다.
환불 금액 계산, 템플릿 제공 등의 고급 기능은 향후 구현 예정입니다.

**Base URL이 2개로 분리**되어 있습니다:
- 환불 정책 그룹: `/api/v1/refund-policy/group`
- 환불 정책: `/api/v1/refund-policies`

---

## 개요

Refund Policy API는 상품별 환불 정책 관리 기능을 제공합니다.

### 현재 구현된 기능
- ✅ 환불 정책 그룹 생성
- ✅ 환불 정책 그룹 목록 조회
- ✅ 환불 정책 그룹 수정
- ✅ 환불 정책 그룹 삭제
- ✅ 환불 정책 등록 (그룹 내)
- ✅ 환불 정책 조회 (그룹별)
- ✅ 환불 정책 삭제

### 🚧 향후 구현 예정
- 상품의 환불 정책 조회
- 환불 가능 금액 자동 계산
- 환불 정책 템플릿 제공
- 상품에 정책 일괄 적용

---

## API 엔드포인트

## 환불 정책 그룹 API

### 1. 환불 정책 그룹 생성 (업체 전용) ✅

```http
POST /api/v1/refund-policy/group
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "일반 투어 환불 정책",
  "description": "당일 투어 상품에 적용되는 기본 환불 정책"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | String | ✅ | 환불 정책 그룹 이름 |
| description | String | ❌ | 설명 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "환불 정책 그룹이 생성되었습니다.",
  "errorCode": null
}
```

---

### 2. 내 환불 정책 그룹 목록 조회 (업체 전용) ✅

```http
GET /api/v1/refund-policy/group/list
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "groupId": 1,
      "name": "일반 투어 환불 정책",
      "description": "당일 투어 상품에 적용되는 기본 환불 정책",
      "createdAt": "2024-12-01T10:00:00",
      "updatedAt": "2025-01-05T15:00:00"
    },
    {
      "groupId": 2,
      "name": "숙박 환불 정책",
      "description": "숙박 상품에 적용되는 환불 정책",
      "createdAt": "2024-12-05T14:30:00",
      "updatedAt": "2024-12-05T14:30:00"
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 3. 환불 정책 그룹 수정 (업체 전용) ✅

그룹 정보와 하위 환불 정책을 한 번에 수정합니다.

```http
PUT /api/v1/refund-policy/group/{groupId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "일반 투어 환불 정책 (수정)",
  "description": "수정된 설명",
  "refundPolicies": [
    {
      "daysBeforeStart": 7,
      "refundRate": 100,
      "description": "7일 전까지 전액 환불"
    },
    {
      "daysBeforeStart": 3,
      "refundRate": 50,
      "description": "3-6일 전까지 50% 환불"
    },
    {
      "daysBeforeStart": 0,
      "refundRate": 0,
      "description": "2일 전부터 환불 불가"
    }
  ]
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "환불 정책 그룹이 수정되었습니다.",
  "errorCode": null
}
```

---

### 4. 환불 정책 그룹 삭제 (업체 전용) ✅

그룹과 하위 환불 정책을 모두 삭제합니다.

```http
DELETE /api/v1/refund-policy/group/{groupId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "환불 정책 그룹이 삭제되었습니다.",
  "errorCode": null
}
```

---

## 환불 정책 API

### 5. 환불 정책 등록 (그룹 내) ✅

```http
POST /api/v1/refund-policies/{groupId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "daysBeforeStart": 14,
  "refundRate": 100,
  "description": "14일 전까지 전액 환불"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| daysBeforeStart | Integer | ✅ | 예약일 N일 전 |
| refundRate | Integer | ✅ | 환불률 (0-100) |
| description | String | ✅ | 설명 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "환불 정책이 등록되었습니다.",
  "errorCode": null
}
```

---

### 6. 환불 정책 조회 (그룹별) ✅

```http
GET /api/v1/refund-policies/{groupId}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "refundPolicyId": 1,
      "daysBeforeStart": 7,
      "refundRate": 100,
      "description": "7일 전까지 전액 환불",
      "createdAt": "2024-12-01T10:00:00"
    },
    {
      "refundPolicyId": 2,
      "daysBeforeStart": 3,
      "refundRate": 50,
      "description": "3-6일 전까지 50% 환불",
      "createdAt": "2024-12-01T10:00:00"
    },
    {
      "refundPolicyId": 3,
      "daysBeforeStart": 0,
      "refundRate": 0,
      "description": "2일 전부터 환불 불가",
      "createdAt": "2024-12-01T10:00:00"
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 7. 환불 정책 삭제 ✅

```http
DELETE /api/v1/refund-policies/{refundPolicyId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "환불 정책이 삭제되었습니다.",
  "errorCode": null
}
```

---

## 🚧 향후 구현 예정 API

아래 API들은 현재 백엔드에서 구현되지 않았으며, 향후 추가될 예정입니다.

### 8. 상품의 환불 정책 조회 🚧

```http
GET /api/v1/refund-policies/product/{productId}
```

### 9. 환불 가능 금액 계산 🚧

```http
POST /api/v1/refund-policies/calculate
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "reservationId": 123,
  "cancelDate": "2025-01-18"
}
```

**예상 Response**

```json
{
  "success": true,
  "data": {
    "reservationId": 123,
    "originalAmount": 100000,
    "refundRate": 100,
    "refundAmount": 100000,
    "cancellationFee": 0,
    "finalRefundAmount": 100000,
    "canCancel": true
  },
  "message": null,
  "errorCode": null
}
```

### 10. 환불 정책 템플릿 조회 (관리자) 🚧

시스템에서 제공하는 기본 템플릿을 조회합니다.

```http
GET /api/v1/refund-policies/templates
Cookie: accessToken=eyJ...
```

### 11. 상품에 환불 정책 적용 🚧

```http
POST /api/v1/refund-policies/apply
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "productIds": [10, 15, 20],
  "policyGroupId": 15
}
```

---

## 데이터 모델

### RefundPolicyGroupResponse

```typescript
interface RefundPolicyGroupResponse {
  groupId: number;
  name: string;
  description: string;
  createdAt: string;
  updatedAt: string;
}
```

### RefundPolicyResponse

```typescript
interface RefundPolicyResponse {
  refundPolicyId: number;
  daysBeforeStart: number; // 예약일 N일 전
  refundRate: number; // 환불률 (0-100)
  description: string;
  createdAt: string;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| POLICY_NOT_FOUND | 404 | 환불 정책을 찾을 수 없음 |
| POLICY_GROUP_NOT_FOUND | 404 | 환불 정책 그룹을 찾을 수 없음 |
| POLICY_IN_USE | 400 | 사용 중인 정책은 삭제 불가 |
| INVALID_REFUND_RATE | 400 | 유효하지 않은 환불률 (0-100) |
| DUPLICATE_POLICY_NAME | 409 | 중복된 정책 이름 |
| UNAUTHORIZED | 401 | 인증 필요 |
| FORBIDDEN | 403 | 권한 없음 (본인 회사 소속만 가능) |

---

## 비즈니스 로직

### 환불 정책 구조

```
환불 정책 그룹 (RefundPolicyGroup)
  ├── 환불 정책 1 (RefundPolicy): 7일 전 100% 환불
  ├── 환불 정책 2 (RefundPolicy): 3일 전 50% 환불
  └── 환불 정책 3 (RefundPolicy): 당일 환불 불가
```

### 환불 금액 계산 로직 (향후)

```typescript
function calculateRefund(
  originalAmount: number,
  daysBeforeReservation: number,
  policies: RefundPolicy[]
): number {
  // 1. 적용 가능한 규칙 찾기 (daysBeforeStart 내림차순 정렬)
  const sortedPolicies = policies.sort((a, b) => b.daysBeforeStart - a.daysBeforeStart);
  
  // 2. 현재 시점에 적용되는 규칙 선택
  const appliedPolicy = sortedPolicies.find(
    policy => daysBeforeReservation >= policy.daysBeforeStart
  );
  
  // 3. 환불 금액 계산
  const refundRate = appliedPolicy?.refundRate || 0;
  return Math.floor(originalAmount * refundRate / 100);
}
```

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today

**개발 로드맵**:
- ✅ Phase 1: 환불 정책 그룹 및 정책 CRUD (완료)
- 🚧 Phase 2: 환불 금액 자동 계산 (개발 예정)
- 🚧 Phase 3: 정책 템플릿 제공 (개발 예정)
- 🚧 Phase 4: 상품 일괄 적용 기능 (개발 예정)
