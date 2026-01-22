# Notify API 명세

## 개요
FCM 푸시 알림 전송 및 히스토리 조회 API

## 엔드포인트 목록

### 1. 특정 사용자에게 커스텀 알림 전송
- **URL**: `POST /api/v1/notify/push`
- **설명**: 관리자가 특정 사용자에게 커스텀 푸시 알림을 전송합니다
- **권한**: ADMIN
- **Request**:
  ```json
  {
    "type": "CUSTOM",
    "title": "알림 제목",
    "body": "알림 본문",
    "userId": "targetUserId",
    "data": {
      "key": "value"
    },
    "link": "https://example.com",
    "imageUrl": "https://example.com/image.jpg"
  }
  ```
- **Response**:
  ```json
  {
    "status": "SUCCESS",
    "message": "알림이 전송되었습니다",
    "data": null
  }
  ```
- **Error Codes**:
  - `NOTIFY_001`: 알림 전송 실패
  - `NOTIFY_002`: 대상 사용자를 찾을 수 없음

### 2. 전체 사용자에게 커스텀 알림 전송 (브로드캐스트)
- **URL**: `POST /api/v1/notify/push/broadcast`
- **설명**: 관리자가 모든 활성 사용자에게 푸시 알림을 브로드캐스트합니다
- **권한**: ADMIN
- **Request**:
  ```json
  {
    "type": "PROMOTIONAL",
    "title": "프로모션 알림",
    "body": "특별 할인 이벤트를 확인하세요!",
    "data": {
      "eventId": "123"
    },
    "link": "https://example.com/event",
    "imageUrl": "https://example.com/banner.jpg"
  }
  ```
- **Response**:
  ```json
  {
    "status": "SUCCESS",
    "message": "브로드캐스트 알림이 전송되었습니다",
    "data": null
  }
  ```
- **Error Codes**:
  - `NOTIFY_003`: 브로드캐스트 전송 실패

### 3. 알림 히스토리 조회
- **URL**: `POST /api/v1/notify/history`
- **설명**: 관리자가 알림 발송 히스토리를 조회합니다
- **권한**: ADMIN
- **Query Parameters**:
  - `page`: 페이지 번호 (기본값: 0)
  - `size`: 페이지 크기 (기본값: 10)
  - `userId`: 대상 사용자 ID (필수)
  - `startDate`: 시작 날짜 (ISO 8601 형식, 필수)
  - `endDate`: 종료 날짜 (ISO 8601 형식, 필수)
  - `type`: 알림 타입 (기본값: CUSTOM)
- **Response**:
  ```json
  {
    "status": "SUCCESS",
    "message": "알림 히스토리 조회 성공",
    "data": {
      "content": [
        {
          "id": 1,
          "title": "알림 제목",
          "body": "알림 본문",
          "type": "CUSTOM",
          "userId": "targetUserId",
          "isAllUsers": false,
          "isSentSuccessfully": true
        }
      ],
      "page": 0,
      "size": 10,
      "totalElements": 50,
      "totalPages": 5,
      "hasNext": true
    }
  }
  ```

## NotificationType Enum
```typescript
enum NotificationType {
  // 예약 관련
  RESERVATION_REQUEST = "RESERVATION_REQUEST",
  RESERVATION_APPROVED = "RESERVATION_APPROVED",
  RESERVATION_CANCELLED = "RESERVATION_CANCELLED",
  RESERVATION_MODIFIED = "RESERVATION_MODIFIED",
  RESERVATION_REMINDER = "RESERVATION_REMINDER",
  
  // 결제 관련
  PAYMENT_COMPLETED = "PAYMENT_COMPLETED",
  PAYMENT_FAILED = "PAYMENT_FAILED",
  REFUND_COMPLETED = "REFUND_COMPLETED",
  
  // 리뷰 관련
  REVIEW_REQUEST = "REVIEW_REQUEST",
  REVIEW_REPLY = "REVIEW_REPLY",
  
  // 쿠폰 관련
  COUPON_ISSUED = "COUPON_ISSUED",
  COUPON_EXPIRING_SOON = "COUPON_EXPIRING_SOON",
  
  // 캠페인 관련
  CAMPAIGN_APPROVED = "CAMPAIGN_APPROVED",
  CAMPAIGN_REJECTED = "CAMPAIGN_REJECTED",
  
  // 정산 관련
  SETTLEMENT_COMPLETED = "SETTLEMENT_COMPLETED",
  
  // 프로모션/마케팅
  PROMOTIONAL = "PROMOTIONAL",
  
  // 시스템
  SYSTEM_NOTICE = "SYSTEM_NOTICE",
  
  // 커스텀
  CUSTOM = "CUSTOM"
}
```
