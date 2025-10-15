# 콘텐츠 API 명세서 (Contents API Specification)

**Base URL**: `/api/v1/contents`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 개요

Contents API는 크리에이터가 작성한 여행 콘텐츠 관리 기능을 제공합니다.

### 주요 기능
- 콘텐츠 목록 조회 (필터링, 정렬)
- 콘텐츠 상세 조회
- 콘텐츠 등록 (크리에이터 전용)
- 콘텐츠 수정/삭제
- 콘텐츠 좋아요
- 댓글 기능

---

## API 엔드포인트

### 1. 콘텐츠 전체 조회

```http
GET /api/v1/contents/all?page=0&size=10&sortOption=LATEST
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 10) |
| sortOption | String | ❌ | LATEST, POPULAR, MOST_VIEWED |
| category | String | ❌ | 카테고리 필터 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "contentsId": 1,
        "title": "제주도 숨은 명소 BEST 10",
        "summary": "현지인만 아는 제주의 숨은 명소를 소개합니다",
        "thumbnailImage": "https://s3.amazonaws.com/trit/contents/1/thumb.jpg",
        "category": "ACTIVITY",
        "creator": {
          "creatorId": 5,
          "name": "여행크리에이터",
          "profileImage": "https://s3.amazonaws.com/trit/profiles/creator5.jpg"
        },
        "viewCount": 12450,
        "likeCount": 856,
        "commentCount": 42,
        "isLiked": false,
        "createdAt": "2025-01-10T14:30:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 500,
    "totalPages": 50
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 콘텐츠 상세 조회

```http
GET /api/v1/contents/{contentsId}
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "contentsId": 1,
    "title": "제주도 숨은 명소 BEST 10",
    "content": "제주도에는 많은 사람들이 모르는 숨은 명소들이...",
    "thumbnailImage": "https://s3.amazonaws.com/trit/contents/1/thumb.jpg",
    "images": [
      "https://s3.amazonaws.com/trit/contents/1/img1.jpg",
      "https://s3.amazonaws.com/trit/contents/1/img2.jpg"
    ],
    "category": "ACTIVITY",
    "tags": ["제주도", "숨은명소", "여행"],
    "places": [
      {
        "placeId": 10,
        "name": "비밀스러운 해변",
        "latitude": 33.2567,
        "longitude": 126.5678
      }
    ],
    "creator": {
      "creatorId": 5,
      "name": "여행크리에이터",
      "profileImage": "https://s3.amazonaws.com/trit/profiles/creator5.jpg",
      "followerCount": 5200
    },
    "viewCount": 12450,
    "likeCount": 856,
    "commentCount": 42,
    "isLiked": false,
    "createdAt": "2025-01-10T14:30:00",
    "updatedAt": "2025-01-11T09:00:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 콘텐츠 등록

크리에이터가 새로운 콘텐츠를 등록합니다. **인증 필요 (CREATOR, ADMIN)**

```http
POST /api/v1/contents/register
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "title": "부산 감천문화마을 완벽 가이드",
  "content": "부산의 대표 관광지 감천문화마을을 소개합니다...",
  "category": "ACTIVITY",
  "thumbnailImage": "https://s3.amazonaws.com/trit/temp/gamcheon-thumb.jpg",
  "images": [
    "https://s3.amazonaws.com/trit/temp/gamcheon-1.jpg",
    "https://s3.amazonaws.com/trit/temp/gamcheon-2.jpg"
  ],
  "tags": ["부산", "감천문화마을", "포토스팟"],
  "placeIds": [15, 16, 17]
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": 150,
  "message": "콘텐츠가 등록되었습니다.",
  "errorCode": null
}
```

---

### 4. 콘텐츠 수정

```http
PUT /api/v1/contents/{contentsId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**: 등록과 동일한 형식

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "콘텐츠가 수정되었습니다.",
  "errorCode": null
}
```

---

### 5. 콘텐츠 삭제

```http
DELETE /api/v1/contents/{contentsId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "콘텐츠가 삭제되었습니다.",
  "errorCode": null
}
```

---

### 6. 콘텐츠 좋아요 추가/취소

```http
POST /api/v1/contents/{contentsId}/like
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "isLiked": true,
    "likeCount": 857
  },
  "message": "좋아요가 추가되었습니다.",
  "errorCode": null
}
```

---

## 데이터 모델

```typescript
interface ContentsResponse {
  contentsId: number;
  title: string;
  summary: string;
  thumbnailImage: string;
  category: Category;
  creator: CreatorInfo;
  viewCount: number;
  likeCount: number;
  commentCount: number;
  isLiked: boolean;
  createdAt: string;
}

interface DetailContentsResponse extends ContentsResponse {
  content: string;
  images: string[];
  tags: string[];
  places: PlaceInfo[];
  updatedAt: string;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| CONTENTS_NOT_FOUND | 404 | 콘텐츠를 찾을 수 없음 |
| UNAUTHORIZED_CONTENTS_ACCESS | 403 | 콘텐츠 접근 권한 없음 |
| INVALID_CATEGORY | 400 | 유효하지 않은 카테고리 |
| TITLE_TOO_SHORT | 400 | 제목이 너무 짧음 (5자 미만) |
| CONTENT_TOO_SHORT | 400 | 본문이 너무 짧음 (50자 미만) |

---

**문서 작성자**: Ted
