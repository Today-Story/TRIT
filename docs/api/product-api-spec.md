# 상품 API 명세서 (Product API Specification)

**Base URL**: `/api/v1/products`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 목차

- [개요](#개요)
- [API 엔드포인트](#api-엔드포인트)
- [데이터 모델](#데이터-모델)
- [에러 코드](#에러-코드)

---

## 개요

Product API는 여행 상품 관리 기능을 제공합니다.

### 주요 기능
- 상품 목록 조회 (필터링, 정렬, 페이지네이션)
- 상품 상세 조회
- 상품 등록 (업체 전용)
- 상품 수정/삭제 (업체 전용)
- 상품 좋아요
- 스케줄 관리

---

## API 엔드포인트

### 1. 전체 상품 목록 조회

#### Request

```http
GET /api/v1/products?page=0&size=10&category=ACTIVITY&sortOption=LATEST
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 | 기본값 |
|---------|------|------|------|--------|
| page | Integer | ❌ | 페이지 번호 (0부터 시작) | 0 |
| size | Integer | ❌ | 페이지 크기 | 10 |
| category | String | ❌ | 카테고리 필터 (ACTIVITY, FOOD, ACCOMMODATION, TRANSPORTATION, ETC) | - |
| sortOption | String | ❌ | 정렬 옵션 (LATEST, POPULAR, LOW_PRICE, HIGH_PRICE, HIGH_RATING) | LATEST |
| productName | String | ❌ | 상품명 검색 | - |
| productStatus | String | ❌ | 상품 상태 (ACTIVE, INACTIVE, SOLD_OUT) | - |
| minPrice | Double | ❌ | 최소 가격 | - |
| maxPrice | Double | ❌ | 최대 가격 | - |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "productId": 1,
        "name": "제주도 한라산 등반 체험",
        "description": "한라산 백록담까지 오르는 등반 프로그램",
        "price": 50000,
        "discountedPrice": 45000,
        "thumbnailImage": "https://s3.amazonaws.com/trit/products/1/thumb.jpg",
        "category": "ACTIVITY",
        "status": "ACTIVE",
        "rating": 4.8,
        "reviewCount": 256,
        "likeCount": 1024,
        "isLiked": false,
        "companyName": "제주투어",
        "location": "제주특별자치도",
        "duration": 480,
        "tags": ["등산", "자연", "힐링"]
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 150,
    "totalPages": 15,
    "first": true,
    "last": false
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 상품 상세 조회

#### Request

```http
GET /api/v1/products/{productId}
Cookie: accessToken=eyJ... (선택)
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "productId": 1,
    "name": "제주도 한라산 등반 체험",
    "description": "한라산 백록담까지 오르는 등반 프로그램입니다...",
    "price": 50000,
    "discountedPrice": 45000,
    "category": "ACTIVITY",
    "status": "ACTIVE",
    "thumbnailImage": "https://s3.amazonaws.com/trit/products/1/thumb.jpg",
    "images": [
      "https://s3.amazonaws.com/trit/products/1/img1.jpg",
      "https://s3.amazonaws.com/trit/products/1/img2.jpg"
    ],
    "rating": 4.8,
    "reviewCount": 256,
    "likeCount": 1024,
    "isLiked": false,
    "viewCount": 15234,
    "company": {
      "companyId": 10,
      "name": "제주투어",
      "rating": 4.7,
      "reviewCount": 1250
    },
    "location": {
      "address": "제주특별자치도 제주시 한라산로",
      "latitude": 33.3617,
      "longitude": 126.5292
    },
    "duration": 480,
    "minParticipants": 1,
    "maxParticipants": 15,
    "includedItems": ["등산 가이드", "안전 장비", "간식"],
    "excludedItems": ["식사", "개인 등산 장비"],
    "cancellationPolicy": "예약일 기준 7일 전까지 전액 환불",
    "precautions": ["날씨에 따라 일정이 변경될 수 있습니다"],
    "tags": ["등산", "자연", "힐링"],
    "schedules": [
      {
        "scheduleId": 101,
        "date": "2025-01-20",
        "startTime": "08:00",
        "endTime": "16:00",
        "capacity": 15,
        "remainingCapacity": 7,
        "price": 50000,
        "status": "AVAILABLE"
      }
    ],
    "relatedProducts": [
      {
        "productId": 2,
        "name": "제주 올레길 트레킹",
        "thumbnailImage": "https://s3.amazonaws.com/trit/products/2/thumb.jpg",
        "price": 35000
      }
    ],
    "createdAt": "2024-12-01T10:00:00",
    "updatedAt": "2025-01-10T15:30:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 상품 + 스케줄 통합 등록

업체가 상품과 스케줄을 함께 등록합니다. **인증 필요 (COMPANY, ADMIN)**

#### Request

```http
POST /api/v1/products/with-schedule
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "부산 해운대 서핑 체험",
  "description": "초보자도 쉽게 배울 수 있는 서핑 레슨",
  "price": 80000,
  "category": "ACTIVITY",
  "thumbnailImage": "https://s3.amazonaws.com/trit/temp/surf-thumb.jpg",
  "images": [
    "https://s3.amazonaws.com/trit/temp/surf-1.jpg",
    "https://s3.amazonaws.com/trit/temp/surf-2.jpg"
  ],
  "duration": 180,
  "minParticipants": 1,
  "maxParticipants": 8,
  "includedItems": ["서핑 보드", "래쉬가드", "강사", "샤워 시설"],
  "excludedItems": ["식사", "개인 수영복"],
  "cancellationPolicy": "예약일 3일 전까지 전액 환불, 이후 50% 환불",
  "precautions": ["수영 가능자만 참여 가능"],
  "tags": ["서핑", "해양스포츠", "부산"],
  "locationId": 25,
  "schedules": [
    {
      "date": "2025-01-20",
      "startTime": "10:00",
      "endTime": "13:00",
      "capacity": 8,
      "price": 80000
    },
    {
      "date": "2025-01-20",
      "startTime": "14:00",
      "endTime": "17:00",
      "capacity": 8,
      "price": 80000
    }
  ]
}
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "productId": 150,
    "message": "상품이 등록되었습니다. 관리자 승인 후 노출됩니다."
  },
  "message": null,
  "errorCode": null
}
```

---

### 4. 상품 수정

#### Request

```http
PUT /api/v1/products/{productId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**: 등록과 동일한 형식

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "productId": 150,
    "message": "상품이 수정되었습니다."
  },
  "message": null,
  "errorCode": null
}
```

---

### 5. 상품 삭제

#### Request

```http
DELETE /api/v1/products/{productId}
Cookie: accessToken=eyJ...
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "상품이 삭제되었습니다.",
  "errorCode": null
}
```

**비즈니스 로직**:
- 예약이 진행 중인 상품은 삭제 불가
- 소프트 삭제 (status를 DELETED로 변경)

---

### 6. 상품 좋아요 추가/취소

#### Request

```http
POST /api/v1/products/{productId}/like
Cookie: accessToken=eyJ...
```

#### Response

**성공 (200 OK) - 좋아요 추가**

```json
{
  "success": true,
  "data": {
    "isLiked": true,
    "likeCount": 1025
  },
  "message": "좋아요가 추가되었습니다.",
  "errorCode": null
}
```

**성공 (200 OK) - 좋아요 취소**

```json
{
  "success": true,
  "data": {
    "isLiked": false,
    "likeCount": 1023
  },
  "message": "좋아요가 취소되었습니다.",
  "errorCode": null
}
```

---

## 데이터 모델

### ProductResponse

```typescript
interface ProductResponse {
  productId: number;
  name: string;
  description: string;
  price: number;
  discountedPrice: number | null;
  thumbnailImage: string;
  category: Category;
  status: ProductStatus;
  rating: number;
  reviewCount: number;
  likeCount: number;
  isLiked: boolean;
  companyName: string;
  location: string;
  duration: number;
  tags: string[];
}

type Category = 'ACTIVITY' | 'FOOD' | 'ACCOMMODATION' | 'TRANSPORTATION' | 'ETC';
type ProductStatus = 'ACTIVE' | 'INACTIVE' | 'SOLD_OUT' | 'DELETED';
```

### DetailProductResponse

```typescript
interface DetailProductResponse extends ProductResponse {
  images: string[];
  viewCount: number;
  company: CompanyInfo;
  location: LocationInfo;
  minParticipants: number;
  maxParticipants: number;
  includedItems: string[];
  excludedItems: string[];
  cancellationPolicy: string;
  precautions: string[];
  schedules: ProductSchedule[];
  relatedProducts: ProductResponse[];
  createdAt: string;
  updatedAt: string;
}

interface ProductSchedule {
  scheduleId: number;
  date: string;
  startTime: string;
  endTime: string;
  capacity: number;
  remainingCapacity: number;
  price: number;
  status: 'AVAILABLE' | 'FULL' | 'CLOSED';
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| PRODUCT_NOT_FOUND | 404 | 상품을 찾을 수 없음 |
| UNAUTHORIZED_PRODUCT_ACCESS | 403 | 상품 접근 권한 없음 |
| PRODUCT_HAS_ACTIVE_RESERVATIONS | 400 | 진행 중인 예약이 있어 삭제 불가 |
| INVALID_PRICE | 400 | 유효하지 않은 가격 |
| INVALID_CATEGORY | 400 | 유효하지 않은 카테고리 |
| DUPLICATE_SCHEDULE | 409 | 중복된 스케줄 |

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
