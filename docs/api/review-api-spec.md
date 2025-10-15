# 리뷰 API 명세서 (Review API Specification)

**Base URL**: `/api/v1/reviews`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 개요

Review API는 여행 상품에 대한 리뷰 관리 기능을 제공합니다.

### 주요 기능
- 리뷰 작성 (예약 완료 후)
- 리뷰 목록 조회
- 리뷰 수정/삭제
- 리뷰 좋아요
- 리뷰 이미지 업로드

---

## API 엔드포인트

### 1. 리뷰 목록 조회

특정 상품의 리뷰를 조회합니다.

```http
GET /api/v1/reviews?productId=10&page=0&size=10&sortOption=LATEST
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| productId | Long | ✅ | 상품 ID |
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 10) |
| sortOption | String | ❌ | LATEST, HIGHEST_RATING, LOWEST_RATING, MOST_HELPFUL |
| rating | Integer | ❌ | 평점 필터 (1-5) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "reviewId": 1,
        "user": {
          "userId": 10,
          "nickname": "여행러버",
          "profileImage": "https://s3.amazonaws.com/trit/profiles/user10.jpg"
        },
        "rating": 5,
        "content": "정말 멋진 경험이었습니다! 가이드님도 친절하시고 일정도 알차서 만족스러웠어요.",
        "images": [
          "https://s3.amazonaws.com/trit/reviews/1/img1.jpg",
          "https://s3.amazonaws.com/trit/reviews/1/img2.jpg"
        ],
        "visitDate": "2025-01-10",
        "likeCount": 24,
        "isLiked": false,
        "helpfulCount": 15,
        "isHelpful": false,
        "createdAt": "2025-01-12T14:30:00",
        "isEdited": false,
        "companyReply": {
          "content": "소중한 리뷰 감사합니다! 다음에도 방문해주세요.",
          "createdAt": "2025-01-13T09:00:00"
        }
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 256,
    "totalPages": 26,
    "averageRating": 4.7,
    "ratingDistribution": {
      "5": 180,
      "4": 50,
      "3": 15,
      "2": 8,
      "1": 3
    }
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 리뷰 작성

예약 완료 후 리뷰를 작성합니다. **인증 필요**

```http
POST /api/v1/reviews
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "reservationId": 123,
  "productId": 10,
  "rating": 5,
  "content": "정말 멋진 경험이었습니다! 가이드님도 친절하시고...",
  "images": [
    "https://s3.amazonaws.com/trit/temp/review-img1.jpg",
    "https://s3.amazonaws.com/trit/temp/review-img2.jpg"
  ]
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| reservationId | Long | ✅ | 예약 ID |
| productId | Long | ✅ | 상품 ID |
| rating | Integer | ✅ | 평점 (1-5) |
| content | String | ✅ | 리뷰 내용 (최소 10자, 최대 1000자) |
| images | Array | ❌ | 리뷰 이미지 URL 배열 (최대 5개) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reviewId": 1,
    "rating": 5,
    "createdAt": "2025-01-12T14:30:00"
  },
  "message": "리뷰가 등록되었습니다.",
  "errorCode": null
}
```

---

### 3. 리뷰 수정

```http
PUT /api/v1/reviews/{reviewId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "rating": 4,
  "content": "수정된 리뷰 내용입니다.",
  "images": [
    "https://s3.amazonaws.com/trit/reviews/1/new-img1.jpg"
  ]
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "reviewId": 1,
    "rating": 4,
    "updatedAt": "2025-01-13T10:00:00"
  },
  "message": "리뷰가 수정되었습니다.",
  "errorCode": null
}
```

---

### 4. 리뷰 삭제

```http
DELETE /api/v1/reviews/{reviewId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "리뷰가 삭제되었습니다.",
  "errorCode": null
}
```

---

### 5. 리뷰 좋아요 추가/취소

```http
POST /api/v1/reviews/{reviewId}/like
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "isLiked": true,
    "likeCount": 25
  },
  "message": "좋아요가 추가되었습니다.",
  "errorCode": null
}
```

---

### 6. 리뷰 도움됨 표시 추가/취소

```http
POST /api/v1/reviews/{reviewId}/helpful
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "isHelpful": true,
    "helpfulCount": 16
  },
  "message": "도움됨이 추가되었습니다.",
  "errorCode": null
}
```

---

### 7. 업체 답글 작성 (업체 전용)

```http
POST /api/v1/reviews/{reviewId}/reply
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "content": "소중한 리뷰 감사합니다! 다음에도 방문해주세요."
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "replyId": 10,
    "content": "소중한 리뷰 감사합니다! 다음에도 방문해주세요.",
    "createdAt": "2025-01-13T09:00:00"
  },
  "message": "답글이 등록되었습니다.",
  "errorCode": null
}
```

---

### 8. 내 리뷰 목록 조회

```http
GET /api/v1/reviews/me?page=0&size=10
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "reviewId": 1,
        "product": {
          "productId": 10,
          "name": "제주도 한라산 등반 체험",
          "thumbnailImage": "https://s3.amazonaws.com/trit/products/10/thumb.jpg"
        },
        "rating": 5,
        "content": "정말 멋진 경험이었습니다!",
        "createdAt": "2025-01-12T14:30:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 15,
    "totalPages": 2
  },
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

```typescript
interface ReviewResponse {
  reviewId: number;
  user: {
    userId: number;
    nickname: string;
    profileImage: string | null;
  };
  rating: number;
  content: string;
  images: string[];
  visitDate: string;
  likeCount: number;
  isLiked: boolean;
  helpfulCount: number;
  isHelpful: boolean;
  createdAt: string;
  updatedAt: string | null;
  isEdited: boolean;
  companyReply: {
    content: string;
    createdAt: string;
  } | null;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| REVIEW_NOT_FOUND | 404 | 리뷰를 찾을 수 없음 |
| UNAUTHORIZED_REVIEW_ACCESS | 403 | 리뷰 수정/삭제 권한 없음 |
| RESERVATION_NOT_COMPLETED | 400 | 아직 완료되지 않은 예약 |
| REVIEW_ALREADY_EXISTS | 409 | 이미 작성한 리뷰 |
| INVALID_RATING | 400 | 유효하지 않은 평점 (1-5 범위) |
| REVIEW_TOO_SHORT | 400 | 리뷰 내용이 너무 짧음 (10자 미만) |
| TOO_MANY_IMAGES | 400 | 이미지가 너무 많음 (최대 5개) |

---

**문서 작성자**: Ted
