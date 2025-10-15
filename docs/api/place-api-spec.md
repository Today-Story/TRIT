# 장소 API 명세서 (Place API Specification)

**Base URL**: `/api/v1/places`  
**버전**: v1.0
**최종 업데이트**: 2025-10-15

---

## 개요

Place API는 여행 장소 정보 관리 기능을 제공합니다.

---

## API 엔드포인트

### 1. 장소 검색

```http
GET /api/v1/places/search?keyword=제주&page=0&size=20
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "placeId": 1,
        "name": "제주 한라산",
        "address": "제주특별자치도 제주시",
        "latitude": 33.3617,
        "longitude": 126.5292,
        "category": "MOUNTAIN",
        "thumbnailImage": "https://s3.amazonaws.com/trit/places/1/thumb.jpg",
        "rating": 4.8,
        "reviewCount": 1250
      }
    ]
  }
}
```

### 2. 장소 상세 조회

```http
GET /api/v1/places/{placeId}
```

### 3. 주변 장소 조회

```http
GET /api/v1/places/nearby?latitude=33.3617&longitude=126.5292&radius=5000
```

---

**문서 작성자**: Ted
