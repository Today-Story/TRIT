# 정산 API 명세서 (Settlement API Specification)

**Base URL**: `/api/v1/settlements`  
**버전**: v1.0

---

## 개요

Settlement API는 크리에이터 및 업체의 정산 관리 기능을 제공합니다.

---

## API 엔드포인트

### 1. 정산 내역 조회

```http
GET /api/v1/settlements/me?page=0&size=20&status=COMPLETED
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| status | String | ❌ | PENDING, IN_PROGRESS, COMPLETED |
| startDate | String | ❌ | 시작일 (YYYY-MM-DD) |
| endDate | String | ❌ | 종료일 (YYYY-MM-DD) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "settlementId": 1,
        "period": "2025-01",
        "totalAmount": 2500000,
        "commission": 250000,
        "settlementAmount": 2250000,
        "status": "COMPLETED",
        "requestedAt": "2025-02-01T10:00:00",
        "completedAt": "2025-02-05T15:30:00",
        "details": [
          {
            "type": "RESERVATION",
            "orderId": "ORDER_20250115_001",
            "amount": 100000,
            "commission": 10000,
            "date": "2025-10-15"
          }
        ]
      }
    ]
  }
}
```

### 2. 정산 요청

```http
POST /api/v1/settlements/request
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "period": "2025-01",
  "bankName": "신한은행",
  "accountNumber": "110-123-456789",
  "accountHolder": "홍길동"
}
```

### 3. 정산 통계 조회

```http
GET /api/v1/settlements/stats?year=2025
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "year": 2025,
    "totalRevenue": 35000000,
    "totalCommission": 3500000,
    "totalSettlement": 31500000,
    "monthlyStats": [
      {
        "month": 1,
        "revenue": 2500000,
        "commission": 250000,
        "settlement": 2250000
      }
    ]
  }
}
```

---

**문서 작성자**: Ted
