# 예약 API 명세서 (Reservation API Specification)

**Base URL**: `/api/v1/reservation`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Reservation API는 여행 상품 예약 관리 기능을 제공합니다.

### 주요 기능
- 예약 생성 (결제와 연동)
- 예약 목록 조회 (사용자/업체)
- 예약 상세 조회
- 예약 취소
- 예약 승인/거절 (업체)

---

## API 엔드포인트

### 1. 예약 상세 조회

```http
GET /api/v1/reservation/{reservationId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reservationId": 1,
    "product": {
      "productId": 10,
      "name": "제주도 한라산 등반 체험",
      "thumbnailImage": "https://s3.amazonaws.com/trit/products/10/thumb.jpg"
    },
    "schedule": {
      "scheduleId": 101,
      "date": "2025-01-25",
      "startTime": "08:00",
      "endTime": "16:00"
    },
    "user": {
      "userId": 5,
      "nickname": "여행러버",
      "phoneNumber": "01012345678"
    },
    "company": {
      "companyId": 3,
      "name": "제주투어",
      "phone": "0647778888"
    },
    "status": "CONFIRMED",
    "participants": 2,
    "totalPrice": 90000,
    "paymentStatus": "SUCCESS",
    "options": [
      {
        "name": "점심 도시락 추가",
        "quantity": 2,
        "price": 10000
      }
    ],
    "specialRequests": "채식 도시락 부탁드립니다",
    "createdAt": "2025-01-15T10:30:00",
    "confirmedAt": "2025-01-15T10:35:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 내 예약 목록 조회

```http
GET /api/v1/reservation/me?page=0&size=10&reservationStatus=CONFIRMED
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 10) |
| reservationStatus | String | ❌ | PENDING, CONFIRMED, CANCELLED, COMPLETED |
| reservationTimeFilter | String | ❌ | UPCOMING, PAST, ALL |
| reviewed | Boolean | ❌ | 리뷰 작성 여부 필터 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "reservationId": 1,
        "productName": "제주도 한라산 등반 체험",
        "productThumbnail": "https://s3.amazonaws.com/trit/products/10/thumb.jpg",
        "reservationDate": "2025-01-25",
        "startTime": "08:00",
        "status": "CONFIRMED",
        "participants": 2,
        "totalPrice": 90000,
        "isReviewed": false,
        "createdAt": "2025-01-15T10:30:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 15,
    "totalPages": 2,
    "first": true,
    "last": false
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 예약 취소

```http
POST /api/v1/reservation/{reservationId}/cancel
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "reason": "일정 변경",
  "detailedReason": "개인 사정으로 여행 일정을 변경하게 되었습니다."
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reservationId": 1,
    "status": "CANCELLED",
    "refundAmount": 90000,
    "refundMethod": "CARD",
    "refundExpectedDate": "2025-01-20"
  },
  "message": "예약이 취소되었습니다. 환불은 3-5 영업일 소요됩니다.",
  "errorCode": null
}
```

**취소 정책**:
- 예약일 7일 전: 100% 환불
- 예약일 3-6일 전: 50% 환불
- 예약일 1-2일 전: 30% 환불
- 예약일 당일: 환불 불가

---

### 4. 예약 승인 (업체용)

```http
POST /api/v1/reservation/{reservationId}/approve
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reservationId": 1,
    "status": "CONFIRMED"
  },
  "message": "예약이 승인되었습니다. 고객에게 알림이 전송됩니다.",
  "errorCode": null
}
```

---

### 5. 특정 기간별 예약 통계 조회 (업체용)

```http
GET /api/v1/reservation/by-period-products?startDate=2025-01-01&endDate=2025-01-31
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "2025-01-25": {
      "date": "2025-01-25",
      "reservationCount": 15,
      "totalRevenue": 1350000,
      "products": [
        {
          "productId": 10,
          "productName": "제주도 한라산 등반 체험",
          "reservationCount": 8,
          "revenue": 720000
        }
      ]
    }
  },
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

### ReservationResponse

```typescript
interface ReservationResponse {
  reservationId: number;
  product: {
    productId: number;
    name: string;
    thumbnailImage: string;
  };
  schedule: {
    scheduleId: number;
    date: string;
    startTime: string;
    endTime: string;
  };
  user: {
    userId: number;
    nickname: string;
    phoneNumber: string;
  };
  status: ReservationStatus;
  participants: number;
  totalPrice: number;
  paymentStatus: PaymentStatus;
  options: ReservationOption[];
  specialRequests: string | null;
  createdAt: string;
  confirmedAt: string | null;
  cancelledAt: string | null;
}

type ReservationStatus = 'PENDING' | 'CONFIRMED' | 'CANCELLED' | 'COMPLETED';
type PaymentStatus = 'PENDING' | 'SUCCESS' | 'FAILED' | 'REFUNDED';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| RESERVATION_NOT_FOUND | 404 | 예약을 찾을 수 없음 |
| UNAUTHORIZED_RESERVATION_ACCESS | 403 | 예약 접근 권한 없음 |
| RESERVATION_ALREADY_CANCELLED | 400 | 이미 취소된 예약 |
| RESERVATION_CANNOT_BE_CANCELLED | 400 | 취소 불가 (당일 예약 등) |
| SCHEDULE_FULLY_BOOKED | 400 | 스케줄이 만석 |
| INVALID_PARTICIPANT_COUNT | 400 | 유효하지 않은 참가 인원 |

---

**문서 작성자**: Backend Team
