# 환불 정책 API 명세서 (Refund Policy API Specification)

**Base URL**: `/api/v1/refund-policies`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Refund Policy API는 상품별 환불 정책 관리 기능을 제공합니다.

### 주요 기능
- 환불 정책 조회
- 환불 정책 그룹 관리 (업체 전용)
- 환불 가능 여부 및 금액 계산
- 정책별 환불 규칙 설정

---

## API 엔드포인트

### 1. 상품의 환불 정책 조회

```http
GET /api/v1/refund-policies/product/{productId}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "policyId": 1,
    "policyGroupId": 10,
    "policyGroupName": "일반 투어 환불 정책",
    "description": "예약일 기준 환불 규정",
    "rules": [
      {
        "daysBeforeStart": 7,
        "refundRate": 100,
        "description": "예약일 7일 전까지 전액 환불"
      },
      {
        "daysBeforeStart": 3,
        "refundRate": 50,
        "description": "예약일 3-6일 전까지 50% 환불"
      },
      {
        "daysBeforeStart": 1,
        "refundRate": 30,
        "description": "예약일 1-2일 전까지 30% 환불"
      },
      {
        "daysBeforeStart": 0,
        "refundRate": 0,
        "description": "예약일 당일 환불 불가"
      }
    ],
    "specialConditions": [
      "천재지변으로 인한 취소는 100% 환불",
      "업체 사정으로 인한 취소는 100% 환불 및 보상"
    ],
    "cancellationFee": 0,
    "createdAt": "2024-12-01T10:00:00",
    "updatedAt": "2025-01-05T15:00:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 환불 가능 금액 계산

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

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| reservationId | Long | ✅ | 예약 ID |
| cancelDate | String | ❌ | 취소 예정일 (미지정 시 오늘) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reservationId": 123,
    "productName": "제주 한라산 등반 투어",
    "reservationDate": "2025-01-25",
    "cancelDate": "2025-01-18",
    "daysBeforeReservation": 7,
    "originalAmount": 100000,
    "refundRate": 100,
    "refundAmount": 100000,
    "cancellationFee": 0,
    "finalRefundAmount": 100000,
    "appliedRule": {
      "daysBeforeStart": 7,
      "refundRate": 100,
      "description": "예약일 7일 전까지 전액 환불"
    },
    "canCancel": true,
    "refundMethod": "CARD",
    "estimatedRefundDate": "2025-01-22"
  },
  "message": null,
  "errorCode": null
}
```

**환불 불가인 경우 (200 OK)**

```json
{
  "success": true,
  "data": {
    "reservationId": 123,
    "reservationDate": "2025-01-25",
    "cancelDate": "2025-01-25",
    "daysBeforeReservation": 0,
    "originalAmount": 100000,
    "refundRate": 0,
    "refundAmount": 0,
    "cancellationFee": 0,
    "finalRefundAmount": 0,
    "appliedRule": {
      "daysBeforeStart": 0,
      "refundRate": 0,
      "description": "예약일 당일 환불 불가"
    },
    "canCancel": false,
    "reason": "예약일 당일에는 환불이 불가능합니다."
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 환불 정책 그룹 목록 조회 (업체 전용)

```http
GET /api/v1/refund-policies/groups?page=0&size=20
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "policyGroupId": 10,
        "name": "일반 투어 환불 정책",
        "description": "당일 투어 상품에 적용되는 기본 환불 정책",
        "isDefault": true,
        "productCount": 25,
        "createdAt": "2024-12-01T10:00:00"
      },
      {
        "policyGroupId": 11,
        "name": "숙박 환불 정책",
        "description": "숙박 상품에 적용되는 환불 정책",
        "isDefault": false,
        "productCount": 10,
        "createdAt": "2024-12-05T14:30:00"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 5,
    "totalPages": 1
  },
  "message": null,
  "errorCode": null
}
```

---

### 4. 환불 정책 그룹 생성 (업체 전용)

```http
POST /api/v1/refund-policies/groups
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "프리미엄 투어 환불 정책",
  "description": "고가 상품에 적용되는 유연한 환불 정책",
  "isDefault": false,
  "rules": [
    {
      "daysBeforeStart": 14,
      "refundRate": 100,
      "description": "예약일 14일 전까지 전액 환불"
    },
    {
      "daysBeforeStart": 7,
      "refundRate": 80,
      "description": "예약일 7-13일 전까지 80% 환불"
    },
    {
      "daysBeforeStart": 3,
      "refundRate": 50,
      "description": "예약일 3-6일 전까지 50% 환불"
    },
    {
      "daysBeforeStart": 0,
      "refundRate": 0,
      "description": "예약일 2일 전부터 환불 불가"
    }
  ],
  "specialConditions": [
    "천재지변으로 인한 취소는 100% 환불",
    "의료 사유로 인한 취소 시 증빙서류 제출 후 100% 환불"
  ],
  "cancellationFee": 0
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "policyGroupId": 15,
    "name": "프리미엄 투어 환불 정책"
  },
  "message": "환불 정책 그룹이 생성되었습니다.",
  "errorCode": null
}
```

---

### 5. 환불 정책 그룹 수정 (업체 전용)

```http
PUT /api/v1/refund-policies/groups/{policyGroupId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**: 생성과 동일한 형식

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "policyGroupId": 15,
    "name": "프리미엄 투어 환불 정책"
  },
  "message": "환불 정책 그룹이 수정되었습니다.",
  "errorCode": null
}
```

---

### 6. 환불 정책 그룹 삭제 (업체 전용)

```http
DELETE /api/v1/refund-policies/groups/{policyGroupId}
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

**실패 - 사용 중인 정책 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "현재 상품에 적용 중인 환불 정책은 삭제할 수 없습니다.",
  "errorCode": "POLICY_IN_USE"
}
```

---

### 7. 상품에 환불 정책 적용 (업체 전용)

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

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "appliedCount": 3,
    "policyGroupName": "프리미엄 투어 환불 정책"
  },
  "message": "3개 상품에 환불 정책이 적용되었습니다.",
  "errorCode": null
}
```

---

### 8. 환불 정책 템플릿 조회 (관리자 전용)

시스템에서 제공하는 기본 템플릿을 조회합니다.

```http
GET /api/v1/refund-policies/templates
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "templateId": 1,
      "name": "일반 투어 표준 정책",
      "description": "가장 많이 사용되는 기본 환불 정책",
      "category": "TOUR",
      "rules": [
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
    },
    {
      "templateId": 2,
      "name": "숙박 표준 정책",
      "description": "숙박 시설에 적합한 환불 정책",
      "category": "ACCOMMODATION",
      "rules": [
        {
          "daysBeforeStart": 14,
          "refundRate": 100,
          "description": "14일 전까지 전액 환불"
        },
        {
          "daysBeforeStart": 7,
          "refundRate": 70,
          "description": "7-13일 전까지 70% 환불"
        },
        {
          "daysBeforeStart": 3,
          "refundRate": 30,
          "description": "3-6일 전까지 30% 환불"
        },
        {
          "daysBeforeStart": 0,
          "refundRate": 0,
          "description": "2일 전부터 환불 불가"
        }
      ]
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

### RefundPolicyResponse

```typescript
interface RefundPolicyResponse {
  policyId: number;
  policyGroupId: number;
  policyGroupName: string;
  description: string;
  rules: RefundRule[];
  specialConditions: string[];
  cancellationFee: number;
  createdAt: string;
  updatedAt: string;
}

interface RefundRule {
  daysBeforeStart: number; // 예약일 N일 전
  refundRate: number; // 환불률 (0-100)
  description: string;
}

interface RefundCalculation {
  reservationId: number;
  productName: string;
  reservationDate: string;
  cancelDate: string;
  daysBeforeReservation: number;
  originalAmount: number;
  refundRate: number;
  refundAmount: number;
  cancellationFee: number;
  finalRefundAmount: number;
  appliedRule: RefundRule;
  canCancel: boolean;
  refundMethod: string;
  estimatedRefundDate: string;
  reason?: string;
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
| RESERVATION_NOT_FOUND | 404 | 예약을 찾을 수 없음 |
| CANNOT_CALCULATE_PAST_DATE | 400 | 과거 날짜로 계산 불가 |

---

## 비즈니스 로직

### 환불 금액 계산 알고리즘

```typescript
function calculateRefund(
  originalAmount: number,
  daysBeforeReservation: number,
  policy: RefundPolicyResponse
): RefundCalculation {
  // 1. 적용 가능한 규칙 찾기 (daysBeforeStart 내림차순 정렬)
  const sortedRules = policy.rules.sort((a, b) => b.daysBeforeStart - a.daysBeforeStart);
  
  // 2. 현재 시점에 적용되는 규칙 선택
  const appliedRule = sortedRules.find(
    rule => daysBeforeReservation >= rule.daysBeforeStart
  );
  
  // 3. 환불 금액 계산
  const refundRate = appliedRule?.refundRate || 0;
  const refundAmount = Math.floor(originalAmount * refundRate / 100);
  const finalRefundAmount = refundAmount - policy.cancellationFee;
  
  return {
    originalAmount,
    refundRate,
    refundAmount,
    cancellationFee: policy.cancellationFee,
    finalRefundAmount: Math.max(0, finalRefundAmount),
    appliedRule,
    canCancel: refundRate > 0
  };
}
```

### 특수 조건 처리

1. **천재지변**: 관리자 승인 후 100% 환불
2. **의료 사유**: 증빙서류 제출 후 100% 환불
3. **업체 귀책**: 자동으로 100% 환불 + 보상 쿠폰 지급

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
