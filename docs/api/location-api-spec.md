# 위치 API 명세서 (Location API Specification)

**Base URL**: `/api/v1/locations`  
**버전**: v1.0

---

## 개요

Location API는 지역 정보 및 좌표 관련 기능을 제공합니다.

---

## API 엔드포인트

### 1. 지역 목록 조회

```http
GET /api/v1/locations
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "locationId": 1,
      "name": "제주특별자치도",
      "code": "JEJU",
      "latitude": 33.4890,
      "longitude": 126.4983
    }
  ]
}
```

### 2. 좌표로 주소 찾기 (Reverse Geocoding)

```http
GET /api/v1/locations/reverse?latitude=33.4890&longitude=126.4983
```

### 3. 주소로 좌표 찾기 (Geocoding)

```http
GET /api/v1/locations/geocode?address=제주특별자치도 제주시
```

---

**문서 작성자**: Backend Team
