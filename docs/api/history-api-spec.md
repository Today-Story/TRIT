# 히스토리 API 명세서 (History API Specification)

**Base URL**: `/api/v1/history`  
**버전**: v1.0

---

## 개요

History API는 사용자의 조회 기록 및 좋아요 기록을 관리합니다.

---

## API 엔드포인트

### 1. 최근 본 콘텐츠

```http
GET /api/v1/history/viewed/contents?page=0&size=20
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "contentsId": 10,
        "title": "제주 숨은 명소",
        "thumbnailImage": "...",
        "viewedAt": "2025-10-15T10:30:00"
      }
    ]
  }
}
```

### 2. 최근 본 상품

```http
GET /api/v1/history/viewed/products?page=0&size=20
Cookie: accessToken=eyJ...
```

### 3. 좋아요한 콘텐츠

```http
GET /api/v1/history/liked/contents?page=0&size=20
Cookie: accessToken=eyJ...
```

### 4. 좋아요한 상품

```http
GET /api/v1/history/liked/products?page=0&size=20
Cookie: accessToken=eyJ...
```

### 5. 조회 기록 삭제

```http
DELETE /api/v1/history/viewed/contents/{contentsId}
Cookie: accessToken=eyJ...
```

---

**문서 작성자**: Ted
