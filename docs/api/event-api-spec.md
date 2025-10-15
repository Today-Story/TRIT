# 이벤트 API 명세서 (Event API Specification)

**Base URL**: `/api/v1/events`  
**버전**: v1.0
**최종 업데이트**: 2025-10-15

---

## 개요

Event API는 프로모션 이벤트 관리 기능을 제공합니다.

---

## API 엔드포인트

### 1. 진행 중인 이벤트 목록

```http
GET /api/v1/events?status=ACTIVE&page=0&size=10
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "eventId": 1,
        "title": "신규 가입 웰컴 이벤트",
        "description": "신규 가입 시 10% 할인 쿠폰 증정",
        "thumbnailImage": "https://s3.amazonaws.com/trit/events/1/thumb.jpg",
        "type": "WELCOME",
        "startDate": "2025-01-01",
        "endDate": "2025-12-31",
        "status": "ACTIVE",
        "participantCount": 5200
      }
    ]
  }
}
```

### 2. 이벤트 상세 조회

```http
GET /api/v1/events/{eventId}
```

### 3. 이벤트 참여

```http
POST /api/v1/events/{eventId}/participate
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reward": {
      "type": "COUPON",
      "couponCode": "WELCOME10"
    }
  },
  "message": "이벤트 참여가 완료되었습니다. 쿠폰이 발급되었습니다."
}
```

---

**문서 작성자**: Ted
