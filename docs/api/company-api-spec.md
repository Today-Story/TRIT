# 업체 API 명세서 (Company API Specification)

**Base URL**: `/api/v1/companies`  
**버전**: v1.0

---

## 개요

Company API는 여행 업체 정보 관리 기능을 제공합니다.

---

## API 엔드포인트

### 1. 업체 목록 조회

```http
GET /api/v1/companies?page=0&size=20
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "companyId": 1,
        "name": "제주투어",
        "description": "제주 전문 여행사",
        "logoImage": "https://s3.amazonaws.com/trit/companies/1/logo.jpg",
        "rating": 4.7,
        "reviewCount": 1250,
        "productCount": 45,
        "location": "제주특별자치도"
      }
    ]
  }
}
```

### 2. 업체 상세 조회

```http
GET /api/v1/companies/{companyId}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "companyId": 1,
    "name": "제주투어",
    "description": "제주 전문 여행사입니다...",
    "logoImage": "https://s3.amazonaws.com/trit/companies/1/logo.jpg",
    "coverImage": "https://s3.amazonaws.com/trit/companies/1/cover.jpg",
    "address": "제주특별자치도 제주시...",
    "phone": "064-777-8888",
    "email": "info@jejutour.com",
    "rating": 4.7,
    "reviewCount": 1250,
    "products": [
      {
        "productId": 10,
        "name": "한라산 등반 체험",
        "thumbnailImage": "...",
        "price": 50000
      }
    ],
    "businessHours": [
      {
        "dayOfWeek": "MONDAY",
        "openTime": "09:00",
        "closeTime": "18:00"
      }
    ]
  }
}
```

### 3. 내 업체 정보 조회 (업체 회원)

```http
GET /api/v1/companies/me
Cookie: accessToken=eyJ...
```

---

**문서 작성자**: Backend Team
