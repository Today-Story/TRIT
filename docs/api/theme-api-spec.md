# 테마 API 명세서 (Theme API Specification)

**Base URL**: `/api/v1/themes`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Theme API는 관리자가 큐레이션한 테마별 콘텐츠 및 상품 모음을 제공합니다.

---

## API 엔드포인트

### 1. 테마 목록 조회

```http
GET /api/v1/themes?page=0&size=10
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "themeId": 1,
        "name": "겨울 여행 추천",
        "description": "겨울에 가기 좋은 따뜻한 남쪽 여행지",
        "thumbnailImage": "https://s3.amazonaws.com/trit/themes/1/thumb.jpg",
        "type": "SEASONAL",
        "contentCount": 25,
        "productCount": 15,
        "startDate": "2024-12-01",
        "endDate": "2025-02-28",
        "isPriority": true
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

### 2. 테마 상세 조회

```http
GET /api/v1/themes/{themeId}
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "themeId": 1,
    "name": "겨울 여행 추천",
    "description": "겨울에 가기 좋은 따뜻한 남쪽 여행지...",
    "thumbnailImage": "https://s3.amazonaws.com/trit/themes/1/thumb.jpg",
    "bannerImage": "https://s3.amazonaws.com/trit/themes/1/banner.jpg",
    "type": "SEASONAL",
    "startDate": "2024-12-01",
    "endDate": "2025-02-28",
    "contents": [
      {
        "contentsId": 10,
        "title": "제주도 겨울 여행 가이드",
        "thumbnailImage": "https://s3.amazonaws.com/trit/contents/10/thumb.jpg",
        "viewCount": 15000,
        "likeCount": 850
      }
    ],
    "products": [
      {
        "productId": 5,
        "name": "부산 겨울 바다 투어",
        "thumbnailImage": "https://s3.amazonaws.com/trit/products/5/thumb.jpg",
        "price": 80000,
        "rating": 4.8
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

### 3. 테마 생성 (관리자 전용)

```http
POST /api/v1/themes
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "봄꽃 여행",
  "description": "봄꽃이 아름다운 여행지 모음",
  "type": "SEASONAL",
  "thumbnailImage": "https://s3.amazonaws.com/trit/temp/spring-thumb.jpg",
  "bannerImage": "https://s3.amazonaws.com/trit/temp/spring-banner.jpg",
  "startDate": "2025-03-01",
  "endDate": "2025-05-31",
  "isPriority": true,
  "contentIds": [10, 15, 20],
  "productIds": [5, 8, 12]
}
```

**테마 타입 (type)**
- `SEASONAL`: 계절별 테마
- `ACTIVITY`: 액티비티별 테마
- `REGION`: 지역별 테마
- `TRENDING`: 트렌딩 테마
- `SPECIAL`: 특별 기획

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| THEME_NOT_FOUND | 404 | 테마를 찾을 수 없음 |
| THEME_EXPIRED | 400 | 만료된 테마 |

---

**문서 작성자**: Backend Team
