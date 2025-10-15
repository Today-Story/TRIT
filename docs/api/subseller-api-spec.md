# 하위 판매자 API 명세서 (SubSeller API Specification)

**Base URL**: `/api/v1/subsellers`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

SubSeller API는 여행 업체가 하위 판매자를 관리하는 기능을 제공합니다.
하위 판매자는 상위 업체의 상품을 판매하고 수수료를 받는 파트너입니다.

### 주요 기능
- 하위 판매자 등록 및 관리
- 하위 판매자별 판매 통계
- 수수료 정산
- 권한 관리

---

## API 엔드포인트

### 1. 하위 판매자 목록 조회 (업체 전용)

```http
GET /api/v1/subsellers?page=0&size=20&status=ACTIVE
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| status | String | ❌ | ACTIVE, SUSPENDED, INACTIVE |
| searchKeyword | String | ❌ | 이름 또는 이메일 검색 |
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 20) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "subsellerId": 1,
        "name": "강남 여행사",
        "email": "gangnam@travel.com",
        "phone": "02-1234-5678",
        "businessNumber": "123-45-67890",
        "status": "ACTIVE",
        "commissionRate": 15.0,
        "totalSales": 25000000,
        "totalCommission": 3750000,
        "productCount": 12,
        "createdAt": "2024-12-01T10:00:00",
        "lastSaleAt": "2025-01-15T14:30:00"
      },
      {
        "subsellerId": 2,
        "name": "부산 투어",
        "email": "busan@tour.com",
        "phone": "051-9876-5432",
        "businessNumber": "987-65-43210",
        "status": "ACTIVE",
        "commissionRate": 12.0,
        "totalSales": 18500000,
        "totalCommission": 2220000,
        "productCount": 8,
        "createdAt": "2024-11-15T09:00:00",
        "lastSaleAt": "2025-01-14T11:20:00"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 15,
    "totalPages": 1
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 하위 판매자 상세 조회 (업체 전용)

```http
GET /api/v1/subsellers/{subsellerId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "subsellerId": 1,
    "name": "강남 여행사",
    "email": "gangnam@travel.com",
    "phone": "02-1234-5678",
    "businessNumber": "123-45-67890",
    "representativeName": "홍길동",
    "address": "서울특별시 강남구 테헤란로 123",
    "status": "ACTIVE",
    "commissionRate": 15.0,
    "contractStartDate": "2024-12-01",
    "contractEndDate": "2025-11-30",
    "bankAccount": {
      "bankName": "신한은행",
      "accountNumber": "110-123-456789",
      "accountHolder": "강남여행사"
    },
    "permissions": ["VIEW_PRODUCTS", "SELL_PRODUCTS", "VIEW_ORDERS"],
    "statistics": {
      "totalSales": 25000000,
      "totalCommission": 3750000,
      "totalOrders": 156,
      "averageOrderValue": 160256,
      "productCount": 12
    },
    "recentOrders": [
      {
        "orderId": 1001,
        "productName": "제주 한라산 투어",
        "amount": 200000,
        "commission": 30000,
        "orderDate": "2025-01-15T14:30:00"
      }
    ],
    "createdAt": "2024-12-01T10:00:00",
    "updatedAt": "2025-01-10T16:00:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 하위 판매자 등록 (업체 전용)

```http
POST /api/v1/subsellers
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "제주 로컬 투어",
  "email": "jeju@localtour.com",
  "phone": "064-123-4567",
  "businessNumber": "456-78-90123",
  "representativeName": "김제주",
  "address": "제주특별자치도 제주시 중앙로 456",
  "commissionRate": 12.0,
  "contractStartDate": "2025-02-01",
  "contractEndDate": "2026-01-31",
  "bankAccount": {
    "bankName": "제주은행",
    "accountNumber": "220-456-789012",
    "accountHolder": "제주로컬투어"
  },
  "permissions": ["VIEW_PRODUCTS", "SELL_PRODUCTS"],
  "memo": "제주 지역 전문 파트너"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | String | ✅ | 하위 판매자 업체명 |
| email | String | ✅ | 이메일 |
| phone | String | ✅ | 전화번호 |
| businessNumber | String | ✅ | 사업자등록번호 |
| representativeName | String | ✅ | 대표자명 |
| address | String | ✅ | 주소 |
| commissionRate | Double | ✅ | 수수료율 (0-100) |
| contractStartDate | String | ✅ | 계약 시작일 |
| contractEndDate | String | ✅ | 계약 종료일 |
| bankAccount | Object | ✅ | 정산 계좌 정보 |
| permissions | Array | ✅ | 권한 목록 |
| memo | String | ❌ | 메모 |

**권한 목록 (permissions)**
- `VIEW_PRODUCTS`: 상품 조회
- `SELL_PRODUCTS`: 상품 판매
- `VIEW_ORDERS`: 주문 조회
- `MANAGE_ORDERS`: 주문 관리
- `VIEW_STATISTICS`: 통계 조회

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "subsellerId": 3,
    "name": "제주 로컬 투어",
    "accessKey": "sk_test_abc123def456",
    "secretKey": "sk_secret_xyz789uvw012"
  },
  "message": "하위 판매자가 등록되었습니다. API 키를 안전하게 보관하세요.",
  "errorCode": null
}
```

---

### 4. 하위 판매자 정보 수정 (업체 전용)

```http
PUT /api/v1/subsellers/{subsellerId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**: 등록과 유사한 형식 (수정 가능한 필드만 포함)

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "subsellerId": 3,
    "name": "제주 로컬 투어"
  },
  "message": "하위 판매자 정보가 수정되었습니다.",
  "errorCode": null
}
```

---

### 5. 하위 판매자 상태 변경 (업체 전용)

```http
PATCH /api/v1/subsellers/{subsellerId}/status
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "status": "SUSPENDED",
  "reason": "계약 위반"
}
```

**상태 옵션**
- `ACTIVE`: 활성
- `SUSPENDED`: 정지
- `INACTIVE`: 비활성

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "subsellerId": 3,
    "status": "SUSPENDED"
  },
  "message": "하위 판매자 상태가 변경되었습니다.",
  "errorCode": null
}
```

---

### 6. 하위 판매자 판매 통계 조회 (업체 전용)

```http
GET /api/v1/subsellers/{subsellerId}/statistics?startDate=2025-01-01&endDate=2025-01-31
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| startDate | String | ❌ | 시작일 (YYYY-MM-DD) |
| endDate | String | ❌ | 종료일 (YYYY-MM-DD) |
| groupBy | String | ❌ | DAY, WEEK, MONTH (기본값: DAY) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "subsellerId": 1,
    "subsellerName": "강남 여행사",
    "period": {
      "startDate": "2025-01-01",
      "endDate": "2025-01-31"
    },
    "summary": {
      "totalSales": 8500000,
      "totalCommission": 1275000,
      "totalOrders": 52,
      "cancelledOrders": 3,
      "averageOrderValue": 163461,
      "conversionRate": 15.2
    },
    "dailyStats": [
      {
        "date": "2025-01-15",
        "sales": 450000,
        "commission": 67500,
        "orders": 3
      },
      {
        "date": "2025-01-14",
        "sales": 320000,
        "commission": 48000,
        "orders": 2
      }
    ],
    "topProducts": [
      {
        "productId": 10,
        "productName": "제주 한라산 투어",
        "sales": 2400000,
        "commission": 360000,
        "orders": 12
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

---

### 7. 하위 판매자 정산 내역 조회 (업체 전용)

```http
GET /api/v1/subsellers/{subsellerId}/settlements?page=0&size=20
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "settlementId": 1,
        "period": "2025-01",
        "totalSales": 8500000,
        "commissionRate": 15.0,
        "totalCommission": 1275000,
        "deductions": 0,
        "finalAmount": 1275000,
        "status": "COMPLETED",
        "paidAt": "2025-02-05T15:00:00"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 3,
    "totalPages": 1
  },
  "message": null,
  "errorCode": null
}
```

---

### 8. 하위 판매자 API 키 재발급 (업체 전용)

```http
POST /api/v1/subsellers/{subsellerId}/regenerate-keys
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "subsellerId": 3,
    "accessKey": "sk_test_new123abc456",
    "secretKey": "sk_secret_new789xyz012"
  },
  "message": "API 키가 재발급되었습니다. 이전 키는 즉시 비활성화됩니다.",
  "errorCode": null
}
```

---

## 하위 판매자용 API

### 9. 내 판매자 정보 조회

```http
GET /api/v1/subsellers/me
Cookie: accessToken=eyJ...
```

### 10. 내 판매 통계 조회

```http
GET /api/v1/subsellers/me/statistics?startDate=2025-01-01&endDate=2025-01-31
Cookie: accessToken=eyJ...
```

### 11. 내 정산 내역 조회

```http
GET /api/v1/subsellers/me/settlements?page=0&size=20
Cookie: accessToken=eyJ...
```

---

## 데이터 모델

### SubSellerResponse

```typescript
interface SubSellerResponse {
  subsellerId: number;
  name: string;
  email: string;
  phone: string;
  businessNumber: string;
  representativeName: string;
  address: string;
  status: SubSellerStatus;
  commissionRate: number;
  contractStartDate: string;
  contractEndDate: string;
  bankAccount: BankAccount;
  permissions: Permission[];
  statistics: SubSellerStatistics;
  createdAt: string;
  updatedAt: string;
}

type SubSellerStatus = 'ACTIVE' | 'SUSPENDED' | 'INACTIVE';
type Permission = 'VIEW_PRODUCTS' | 'SELL_PRODUCTS' | 'VIEW_ORDERS' | 'MANAGE_ORDERS' | 'VIEW_STATISTICS';

interface SubSellerStatistics {
  totalSales: number;
  totalCommission: number;
  totalOrders: number;
  averageOrderValue: number;
  productCount: number;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| SUBSELLER_NOT_FOUND | 404 | 하위 판매자를 찾을 수 없음 |
| DUPLICATE_BUSINESS_NUMBER | 409 | 이미 등록된 사업자등록번호 |
| INVALID_COMMISSION_RATE | 400 | 유효하지 않은 수수료율 (0-100) |
| CONTRACT_DATE_INVALID | 400 | 유효하지 않은 계약 기간 |
| SUBSELLER_SUSPENDED | 403 | 정지된 하위 판매자 |
| INSUFFICIENT_PERMISSION | 403 | 권한 부족 |

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
