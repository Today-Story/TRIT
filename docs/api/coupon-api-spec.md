# 쿠폰 API 명세서 (Coupon API Specification)

**Base URL**: `/api/v1/coupons`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Coupon API는 할인 쿠폰 관리 및 발급 기능을 제공합니다.

---

## API 엔드포인트

### 1. 사용 가능한 쿠폰 목록 조회

```http
GET /api/v1/coupons/available?productId=10
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| productId | Long | ❌ | 특정 상품에 사용 가능한 쿠폰만 조회 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "couponId": 1,
      "name": "신규 회원 10% 할인",
      "code": "WELCOME10",
      "discountType": "PERCENT",
      "discountValue": 10,
      "minPurchaseAmount": 50000,
      "maxDiscountAmount": 10000,
      "validFrom": "2025-01-01T00:00:00",
      "validUntil": "2025-12-31T23:59:59",
      "applicableCategories": ["ACTIVITY", "FOOD"],
      "isUsed": false
    }
  ],
  "message": null,
  "errorCode": null
}
```

### 2. 내 쿠폰 목록 조회

```http
GET /api/v1/coupons/me?page=0&size=10&status=AVAILABLE
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| status | String | ❌ | AVAILABLE, USED, EXPIRED |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "userCouponId": 1,
        "coupon": {
          "couponId": 1,
          "name": "신규 회원 10% 할인",
          "code": "WELCOME10",
          "discountType": "PERCENT",
          "discountValue": 10
        },
        "status": "AVAILABLE",
        "issuedAt": "2025-01-15T10:00:00",
        "usedAt": null,
        "expiresAt": "2025-12-31T23:59:59"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 5,
    "totalPages": 1
  },
  "message": null,
  "errorCode": null
}
```

### 3. 쿠폰 발급 (코드로)

```http
POST /api/v1/coupons/issue
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "code": "WELCOME10"
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "userCouponId": 1,
    "couponName": "신규 회원 10% 할인",
    "expiresAt": "2025-12-31T23:59:59"
  },
  "message": "쿠폰이 발급되었습니다.",
  "errorCode": null
}
```

### 4. 쿠폰 사용 (결제 시 자동 처리)

결제 API에서 자동으로 처리되며, 별도 엔드포인트는 없습니다.

### 5. 쿠폰 템플릿 목록 조회 (관리자 전용)

```http
GET /api/v1/coupons/templates?page=0&size=20
Cookie: accessToken=eyJ...
```

### 6. 쿠폰 템플릿 생성 (관리자 전용)

```http
POST /api/v1/coupons/templates
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "여름 시즌 20% 할인",
  "code": "SUMMER20",
  "discountType": "PERCENT",
  "discountValue": 20,
  "minPurchaseAmount": 100000,
  "maxDiscountAmount": 50000,
  "validFrom": "2025-06-01T00:00:00",
  "validUntil": "2025-08-31T23:59:59",
  "issuanceLimit": 1000,
  "applicableCategories": ["ACTIVITY"],
  "issueMethod": "CODE"
}
```

---

## 데이터 모델

```typescript
interface CouponResponse {
  couponId: number;
  name: string;
  code: string;
  discountType: 'PERCENT' | 'FIXED';
  discountValue: number;
  minPurchaseAmount: number;
  maxDiscountAmount: number | null;
  validFrom: string;
  validUntil: string;
  applicableCategories: Category[];
  isUsed: boolean;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| COUPON_NOT_FOUND | 404 | 쿠폰을 찾을 수 없음 |
| COUPON_ALREADY_ISSUED | 409 | 이미 발급받은 쿠폰 |
| COUPON_EXPIRED | 400 | 만료된 쿠폰 |
| COUPON_LIMIT_EXCEEDED | 400 | 쿠폰 발급 한도 초과 |
| INVALID_COUPON_CODE | 400 | 유효하지 않은 쿠폰 코드 |
| COUPON_NOT_APPLICABLE | 400 | 해당 상품에 사용 불가한 쿠폰 |

---

**문서 작성자**: Backend Team
