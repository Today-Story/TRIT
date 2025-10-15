# 플레이리스트 API 명세서 (Playlist API Specification)

**Base URL**: `/api/v1/playlists`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Playlist API는 사용자가 여행 콘텐츠를 모아서 플레이리스트를 만드는 기능을 제공합니다.

---

## API 엔드포인트

### 1. 플레이리스트 목록 조회

```http
GET /api/v1/playlists?page=0&size=10&sortOption=LATEST
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "playlistId": 1,
        "name": "제주도 완벽 여행 코스",
        "description": "제주 여행 필수 코스 모음",
        "thumbnailImage": "https://s3.amazonaws.com/trit/playlists/1/thumb.jpg",
        "isPublic": true,
        "contentCount": 15,
        "likeCount": 245,
        "isLiked": false,
        "creator": {
          "userId": 10,
          "nickname": "여행러버"
        },
        "createdAt": "2025-01-10T10:00:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 50,
    "totalPages": 5
  },
  "message": null,
  "errorCode": null
}
```

### 2. 플레이리스트 상세 조회

```http
GET /api/v1/playlists/{playlistId}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "playlistId": 1,
    "name": "제주도 완벽 여행 코스",
    "description": "제주 여행 필수 코스 모음",
    "thumbnailImage": "https://s3.amazonaws.com/trit/playlists/1/thumb.jpg",
    "isPublic": true,
    "contents": [
      {
        "contentsId": 10,
        "title": "제주 동부 해안도로",
        "thumbnailImage": "https://s3.amazonaws.com/trit/contents/10/thumb.jpg",
        "order": 1
      }
    ],
    "likeCount": 245,
    "isLiked": false,
    "creator": {
      "userId": 10,
      "nickname": "여행러버",
      "profileImage": "https://s3.amazonaws.com/trit/profiles/user10.jpg"
    },
    "createdAt": "2025-01-10T10:00:00",
    "updatedAt": "2025-01-12T15:00:00"
  },
  "message": null,
  "errorCode": null
}
```

### 3. 플레이리스트 생성

```http
POST /api/v1/playlists
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "부산 여행 코스",
  "description": "부산 핵심 관광지 모음",
  "isPublic": true,
  "contentIds": [10, 15, 20]
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "playlistId": 2,
    "name": "부산 여행 코스"
  },
  "message": "플레이리스트가 생성되었습니다.",
  "errorCode": null
}
```

### 4. 플레이리스트에 콘텐츠 추가

```http
POST /api/v1/playlists/{playlistId}/contents
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "contentId": 25
}
```

### 5. 플레이리스트에서 콘텐츠 제거

```http
DELETE /api/v1/playlists/{playlistId}/contents/{contentId}
Cookie: accessToken=eyJ...
```

### 6. 플레이리스트 좋아요

```http
POST /api/v1/playlists/{playlistId}/like
Cookie: accessToken=eyJ...
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| PLAYLIST_NOT_FOUND | 404 | 플레이리스트를 찾을 수 없음 |
| UNAUTHORIZED_PLAYLIST_ACCESS | 403 | 권한 없음 |
| DUPLICATE_CONTENT_IN_PLAYLIST | 409 | 이미 추가된 콘텐츠 |

---

**문서 작성자**: Backend Team
