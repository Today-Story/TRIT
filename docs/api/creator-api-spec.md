# 크리에이터 API 명세서 (Creator API Specification)

**Base URL**: `/api/v1/creators`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 개요

Creator API는 크리에이터 프로필 및 활동 관리 기능을 제공합니다.

---

## API 엔드포인트

### 1. 크리에이터 목록 조회

```http
GET /api/v1/creators?page=0&size=12&sortOption=POPULAR
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| sortOption | String | ❌ | POPULAR, LATEST, MOST_FOLLOWERS |
| category | String | ❌ | 카테고리 필터 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "creatorId": 1,
        "userId": 10,
        "creatorName": "여행크리에이터",
        "profileImage": "https://s3.amazonaws.com/trit/creators/1/profile.jpg",
        "introduction": "여행과 사진을 사랑하는 크리에이터입니다",
        "followerCount": 15200,
        "contentCount": 125,
        "totalLikes": 45000,
        "categories": ["ACTIVITY", "FOOD"],
        "socialLinks": {
          "instagram": "https://instagram.com/traveler",
          "youtube": "https://youtube.com/@traveler"
        }
      }
    ],
    "page": 0,
    "size": 12,
    "totalElements": 250,
    "totalPages": 21
  },
  "message": null,
  "errorCode": null
}
```

### 2. 크리에이터 상세 조회

```http
GET /api/v1/creators/{creatorId}
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "creatorId": 1,
    "userId": 10,
    "creatorName": "여행크리에이터",
    "profileImage": "https://s3.amazonaws.com/trit/creators/1/profile.jpg",
    "coverImage": "https://s3.amazonaws.com/trit/creators/1/cover.jpg",
    "introduction": "여행과 사진을 사랑하는 크리에이터입니다...",
    "followerCount": 15200,
    "isFollowing": false,
    "contentCount": 125,
    "totalLikes": 45000,
    "totalViews": 2500000,
    "categories": ["ACTIVITY", "FOOD"],
    "socialLinks": {
      "instagram": "https://instagram.com/traveler",
      "youtube": "https://youtube.com/@traveler",
      "blog": "https://blog.naver.com/traveler"
    },
    "recentContents": [
      {
        "contentsId": 10,
        "title": "제주 숨은 명소",
        "thumbnailImage": "https://s3.amazonaws.com/trit/contents/10/thumb.jpg",
        "viewCount": 12000,
        "likeCount": 850
      }
    ],
    "statistics": {
      "avgViewCount": 20000,
      "avgLikeCount": 360,
      "engagementRate": 4.5
    }
  },
  "message": null,
  "errorCode": null
}
```

### 3. 크리에이터 팔로우/언팔로우

```http
POST /api/v1/creators/{creatorId}/follow
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "isFollowing": true,
    "followerCount": 15201
  },
  "message": "팔로우했습니다.",
  "errorCode": null
}
```

### 4. 내 크리에이터 통계 조회 (크리에이터 본인)

```http
GET /api/v1/creators/me/statistics?period=MONTH
Cookie: accessToken=eyJ...
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| period | String | ❌ | DAY, WEEK, MONTH, YEAR |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "period": "MONTH",
    "startDate": "2025-01-01",
    "endDate": "2025-01-31",
    "metrics": {
      "newFollowers": 520,
      "totalViews": 150000,
      "totalLikes": 8500,
      "newContents": 12,
      "avgViewsPerContent": 12500,
      "engagementRate": 5.7
    },
    "dailyStats": [
      {
        "date": "2025-10-15",
        "views": 5200,
        "likes": 320,
        "newFollowers": 18
      }
    ],
    "topContents": [
      {
        "contentsId": 10,
        "title": "제주 숨은 명소",
        "views": 25000,
        "likes": 1200
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

### 5. 팔로잉 크리에이터 목록 조회

```http
GET /api/v1/creators/following?page=0&size=20
Cookie: accessToken=eyJ...
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| CREATOR_NOT_FOUND | 404 | 크리에이터를 찾을 수 없음 |
| ALREADY_FOLLOWING | 409 | 이미 팔로우 중 |
| CANNOT_FOLLOW_SELF | 400 | 자기 자신을 팔로우할 수 없음 |

---

**문서 작성자**: Ted
