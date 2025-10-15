# 해시태그 API 명세서 (Hashtags API Specification)

**Base URL**: `/api/v1/hashtags`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Hashtags API는 콘텐츠 및 상품의 해시태그 관리 기능을 제공합니다.

### 주요 기능
- 인기 해시태그 조회
- 해시태그 검색
- 해시태그별 콘텐츠/상품 조회
- 해시태그 자동 완성

---

## API 엔드포인트

### 1. 인기 해시태그 목록 조회

```http
GET /api/v1/hashtags/popular?limit=20&category=ACTIVITY
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| limit | Integer | ❌ | 조회 개수 (기본값: 20, 최대: 50) |
| category | String | ❌ | 카테고리 필터 |
| period | String | ❌ | DAY, WEEK, MONTH (기본값: WEEK) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "hashtagId": 1,
      "name": "제주여행",
      "count": 15234,
      "category": "ACTIVITY",
      "trendScore": 95.5
    },
    {
      "hashtagId": 2,
      "name": "부산맛집",
      "count": 12456,
      "category": "FOOD",
      "trendScore": 88.3
    },
    {
      "hashtagId": 3,
      "name": "서울여행",
      "count": 10890,
      "category": "ACTIVITY",
      "trendScore": 82.1
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 2. 해시태그 검색

```http
GET /api/v1/hashtags/search?keyword=제주&page=0&size=20
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| keyword | String | ✅ | 검색 키워드 |
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 20) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "hashtagId": 1,
        "name": "제주여행",
        "count": 15234,
        "category": "ACTIVITY"
      },
      {
        "hashtagId": 5,
        "name": "제주맛집",
        "count": 8567,
        "category": "FOOD"
      },
      {
        "hashtagId": 10,
        "name": "제주도",
        "count": 20123,
        "category": "ACTIVITY"
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

### 3. 해시태그 자동 완성

```http
GET /api/v1/hashtags/autocomplete?query=제
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| query | String | ✅ | 검색어 (최소 1자) |
| limit | Integer | ❌ | 결과 개수 (기본값: 10, 최대: 20) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "hashtagId": 1,
      "name": "제주여행",
      "count": 15234
    },
    {
      "hashtagId": 5,
      "name": "제주맛집",
      "count": 8567
    },
    {
      "hashtagId": 10,
      "name": "제주도",
      "count": 20123
    },
    {
      "hashtagId": 15,
      "name": "제주카페",
      "count": 6234
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 4. 해시태그별 콘텐츠 조회

```http
GET /api/v1/hashtags/{hashtagId}/contents?page=0&size=20
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "hashtag": {
      "hashtagId": 1,
      "name": "제주여행",
      "count": 15234
    },
    "contents": {
      "content": [
        {
          "contentsId": 10,
          "title": "제주 3박 4일 완벽 코스",
          "thumbnailImage": "...",
          "viewCount": 12000,
          "likeCount": 850,
          "createdAt": "2025-01-10T10:00:00"
        }
      ],
      "page": 0,
      "size": 20,
      "totalElements": 1523,
      "totalPages": 77
    }
  },
  "message": null,
  "errorCode": null
}
```

---

### 5. 해시태그별 상품 조회

```http
GET /api/v1/hashtags/{hashtagId}/products?page=0&size=20
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "hashtag": {
      "hashtagId": 1,
      "name": "제주여행",
      "count": 15234
    },
    "products": {
      "content": [
        {
          "productId": 10,
          "name": "제주 한라산 등반 투어",
          "thumbnailImage": "...",
          "price": 50000,
          "rating": 4.8,
          "reviewCount": 256
        }
      ],
      "page": 0,
      "size": 20,
      "totalElements": 234,
      "totalPages": 12
    }
  },
  "message": null,
  "errorCode": null
}
```

---

### 6. 트렌딩 해시태그 조회

최근 급상승 중인 해시태그를 조회합니다.

```http
GET /api/v1/hashtags/trending?limit=10
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "hashtagId": 25,
      "name": "벚꽃명소",
      "count": 3456,
      "growthRate": 450.5,
      "rank": 1,
      "previousRank": 15
    },
    {
      "hashtagId": 30,
      "name": "봄여행",
      "count": 2890,
      "growthRate": 320.2,
      "rank": 2,
      "previousRank": 25
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 7. 관련 해시태그 조회

특정 해시태그와 함께 자주 사용되는 해시태그를 조회합니다.

```http
GET /api/v1/hashtags/{hashtagId}/related?limit=10
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "hashtag": {
      "hashtagId": 1,
      "name": "제주여행"
    },
    "related": [
      {
        "hashtagId": 5,
        "name": "제주맛집",
        "count": 8567,
        "correlation": 0.85
      },
      {
        "hashtagId": 10,
        "name": "제주카페",
        "count": 6234,
        "correlation": 0.78
      },
      {
        "hashtagId": 15,
        "name": "제주숙소",
        "count": 5123,
        "correlation": 0.72
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

---

### 8. 해시태그 팔로우 (알림 설정)

```http
POST /api/v1/hashtags/{hashtagId}/follow
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "hashtagId": 1,
    "isFollowing": true
  },
  "message": "해시태그를 팔로우했습니다. 새 콘텐츠 알림을 받습니다.",
  "errorCode": null
}
```

---

### 9. 내가 팔로우한 해시태그 목록

```http
GET /api/v1/hashtags/following?page=0&size=20
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "hashtagId": 1,
        "name": "제주여행",
        "count": 15234,
        "newContentCount": 45,
        "followedAt": "2025-01-01T10:00:00"
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

## 데이터 모델

### HashtagResponse

```typescript
interface HashtagResponse {
  hashtagId: number;
  name: string;
  count: number; // 사용 횟수
  category: Category | null;
  trendScore: number; // 트렌드 점수 (0-100)
  growthRate: number | null; // 증가율 (%)
  rank: number | null; // 순위
  previousRank: number | null; // 이전 순위
  correlation: number | null; // 연관도 (0-1)
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| HASHTAG_NOT_FOUND | 404 | 해시태그를 찾을 수 없음 |
| INVALID_HASHTAG_NAME | 400 | 유효하지 않은 해시태그 이름 |
| ALREADY_FOLLOWING_HASHTAG | 409 | 이미 팔로우 중인 해시태그 |

---

## 특수 기능

### 해시태그 정규화

입력된 해시태그는 자동으로 정규화됩니다:
- 공백 제거
- 소문자 변환 (영문인 경우)
- 특수문자 제거 (# 제외)
- 최대 길이: 20자

예시:
- `#제주 여행` → `제주여행`
- `#SEOUL` → `seoul`

### 트렌드 스코어 계산

트렌드 스코어는 다음 요소를 고려하여 계산:
- 최근 사용 빈도 (50%)
- 증가율 (30%)
- 좋아요/공유 수 (20%)

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
