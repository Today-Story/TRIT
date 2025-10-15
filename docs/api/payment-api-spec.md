# 결제 API 명세서 (Payment API Specification)

**Base URL**: `/api/v1/payments`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 개요

Payment API는 이롬넷 PG 연동 결제 기능을 제공합니다.

### 결제 프로세스

1. **준비**: 프론트엔드에서 `/prepare` 호출 → 서버에 초기 결제 정보 저장
2. **결제**: 프론트엔드에서 이롬넷 SDK 실행 → 사용자 결제 진행
3. **완료**: 이롬넷 Webhook이 `/confirm` 호출 → 결제 확정 및 예약 생성
4. **확인**: 프론트엔드가 결제 결과 확인

---

## API 엔드포인트

### 1. 초기 결제 정보 저장

결제 SDK를 실행하기 전에 결제 정보를 서버에 저장합니다.

```http
POST /api/v1/payments/prepare
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "orderId": "ORDER_20250115_123456",
  "productId": 10,
  "scheduleId": 101,
  "participants": 2,
  "totalAmount": 90000,
  "options": [
    {
      "optionId": 5,
      "name": "점심 도시락 추가",
      "quantity": 2,
      "price": 10000
    }
  ],
  "specialRequests": "채식 도시락 부탁드립니다",
  "couponId": 20
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "결제 준비가 완료되었습니다.",
  "errorCode": null
}
```

---

### 2. 결제 확인 (Webhook용)

**⚠️ 이 엔드포인트는 이롬넷 PG에서만 호출합니다.**

```http
POST /api/v1/payments/confirm
Content-Type: application/json
```

**Request Body (이롬넷 PG 전송)**

```json
{
  "orderId": "ORDER_20250115_123456",
  "transactionId": "EROMNET_TX_789012",
  "amount": 90000,
  "paymentMethod": "CARD",
  "cardCompany": "신한카드",
  "cardNumber": "1234-****-****-5678",
  "installment": 0,
  "approvalNumber": "12345678",
  "approvedAt": "2025-10-15T10:35:00",
  "status": "SUCCESS"
}
```

**Response (200 OK)**

```json
{
  "receiveResult": "SUCCESS"
}
```

**비즈니스 로직**:
1. 결제 정보 검증
2. 결제 상태 업데이트 (PENDING → SUCCESS)
3. 예약 생성
4. 사용자에게 확인 이메일 발송
5. 스케줄 잔여 좌석 감소

---

### 3. 결제 취소 (환불)

```http
POST /api/v1/payments/{paymentId}/cancel
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "reason": "예약 취소로 인한 환불",
  "refundAmount": 90000
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "paymentId": 1,
    "refundAmount": 90000,
    "refundStatus": "REFUNDED",
    "refundedAt": "2025-01-16T11:00:00",
    "refundExpectedDate": "2025-01-20"
  },
  "message": "환불이 요청되었습니다. 3-5 영업일 소요됩니다.",
  "errorCode": null
}
```

---

### 4. 내 결제 내역 조회

```http
GET /api/v1/payments/me?page=0&size=10&status=SUCCESS
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 10) |
| status | String | ❌ | PENDING, SUCCESS, FAILED, CANCELLED, REFUNDED |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "paymentId": 1,
        "orderId": "ORDER_20250115_123456",
        "productName": "제주도 한라산 등반 체험",
        "amount": 90000,
        "paymentMethod": "CARD",
        "status": "SUCCESS",
        "paidAt": "2025-10-15T10:35:00",
        "canRefund": true,
        "refundDeadline": "2025-01-18T23:59:59"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 25,
    "totalPages": 3
  },
  "message": null,
  "errorCode": null
}
```

---

### 5. 매출 통계 조회 (업체용)

```http
GET /api/v1/payments/sales?yearMonth=2025-01
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "yearMonth": "2025-01",
    "totalRevenue": 15000000,
    "totalOrders": 150,
    "averageOrderValue": 100000,
    "dailySales": [
      {
        "date": "2025-10-15",
        "revenue": 1200000,
        "orders": 12
      }
    ],
    "topProducts": [
      {
        "productId": 10,
        "productName": "제주도 한라산 등반 체험",
        "revenue": 3600000,
        "orders": 40
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

### PaymentResponse

```typescript
interface PaymentResponse {
  paymentId: number;
  orderId: string;
  transactionId: string;
  amount: number;
  paymentMethod: PaymentMethod;
  status: PaymentStatus;
  cardCompany: string | null;
  cardNumber: string | null;
  installment: number;
  approvalNumber: string | null;
  paidAt: string | null;
  refundedAt: string | null;
  refundAmount: number | null;
  canRefund: boolean;
  refundDeadline: string | null;
}

type PaymentMethod = 'CARD' | 'BANK_TRANSFER' | 'KAKAO_PAY' | 'NAVER_PAY';
type PaymentStatus = 'PENDING' | 'SUCCESS' | 'FAILED' | 'CANCELLED' | 'REFUNDED';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| PAYMENT_NOT_FOUND | 404 | 결제를 찾을 수 없음 |
| PAYMENT_ALREADY_PROCESSED | 400 | 이미 처리된 결제 |
| PAYMENT_AMOUNT_MISMATCH | 400 | 결제 금액 불일치 |
| PAYMENT_VERIFICATION_FAILED | 400 | 결제 검증 실패 (PG사 검증) |
| REFUND_NOT_ALLOWED | 400 | 환불 불가 (기한 초과 등) |
| REFUND_AMOUNT_INVALID | 400 | 유효하지 않은 환불 금액 |
| PG_CONNECTION_ERROR | 500 | PG사 연동 오류 |

---

## 보안 고려사항

### 1. 결제 금액 검증
- 프론트엔드에서 전달받은 금액과 서버에서 계산한 금액을 반드시 비교
- PG사에서 전달받은 금액과 서버에 저장된 금액 일치 여부 확인

### 2. 중복 결제 방지
- orderId는 UUID 기반 고유값
- 동일 orderId로 여러 번 결제 시도 시 첫 결제만 처리

### 3. Webhook 보안
- 이롬넷 서명 검증 (HMAC SHA256)
- IP 화이트리스트 (이롬넷 서버 IP만 허용)

---

**문서 작성자**: Ted
