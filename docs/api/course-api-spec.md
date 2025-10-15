# 코스 API 명세서 (Course API Specification)

**Base URL**: `/api/v1/courses`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Course API는 여행 코스 추천 및 경로 계산 기능을 제공합니다.

### 주요 기능
- 추천 여행 코스 조회
- 사용자 맞춤 코스 생성
- 코스 경로 계산 (도보/대중교통)
- 코스 저장 및 공유

---

## API 엔드포인트

### 1. 추천 코스 목록 조회

```http
GET /api/v1/courses?location=제주&duration=1&page=0&size=10
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| location | String | ❌ | 지역 필터 |
| duration | Integer | ❌ | 소요 시간 (일 단위) |
| theme | String | ❌ | 테마 (NATURE, FOOD, CULTURE, ACTIVITY) |
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 10) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "courseId": 1,
        "name": "제주 동부 해안 일주 코스",
        "description": "성산일출봉부터 우도까지 동부 해안을 따라가는 코스",
        "thumbnailImage": "https://s3.amazonaws.com/trit/courses/1/thumb.jpg",
        "location": "제주특별자치도",
        "duration": 1,
        "theme": "NATURE",
        "distance": 45.5,
        "estimatedTime": 360,
        "transportType": "CAR",
        "placeCount": 5,
        "likeCount": 856,
        "isLiked": false,
        "difficulty": "EASY",
        "rating": 4.8,
        "reviewCount": 124
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 45,
    "totalPages": 5
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 코스 상세 조회

```http
GET /api/v1/courses/{courseId}
Cookie: accessToken=eyJ... (선택)
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "courseId": 1,
    "name": "제주 동부 해안 일주 코스",
    "description": "성산일출봉부터 우도까지 동부 해안을 따라가는 코스입니다...",
    "thumbnailImage": "https://s3.amazonaws.com/trit/courses/1/thumb.jpg",
    "images": [
      "https://s3.amazonaws.com/trit/courses/1/img1.jpg",
      "https://s3.amazonaws.com/trit/courses/1/img2.jpg"
    ],
    "location": "제주특별자치도",
    "duration": 1,
    "theme": "NATURE",
    "distance": 45.5,
    "estimatedTime": 360,
    "transportType": "CAR",
    "difficulty": "EASY",
    "rating": 4.8,
    "reviewCount": 124,
    "likeCount": 856,
    "isLiked": false,
    "waypoints": [
      {
        "order": 1,
        "place": {
          "placeId": 10,
          "name": "성산일출봉",
          "latitude": 33.4606,
          "longitude": 126.9424,
          "category": "LANDMARK"
        },
        "visitDuration": 90,
        "description": "일출 명소이자 유네스코 세계자연유산",
        "tips": "아침 일찍 방문하면 일출을 볼 수 있습니다"
      },
      {
        "order": 2,
        "place": {
          "placeId": 11,
          "name": "섭지코지",
          "latitude": 33.4274,
          "longitude": 126.9296,
          "category": "SCENIC"
        },
        "visitDuration": 60,
        "description": "해안 절경이 아름다운 곳",
        "tips": "날씨가 좋은 날 방문 추천"
      }
    ],
    "routes": [
      {
        "from": {
          "placeId": 10,
          "name": "성산일출봉"
        },
        "to": {
          "placeId": 11,
          "name": "섭지코지"
        },
        "distance": 5.2,
        "duration": 10,
        "transportType": "CAR",
        "path": [
          {"latitude": 33.4606, "longitude": 126.9424},
          {"latitude": 33.4274, "longitude": 126.9296}
        ]
      }
    ],
    "recommendedProducts": [
      {
        "productId": 15,
        "name": "성산일출봉 가이드 투어",
        "thumbnailImage": "...",
        "price": 30000
      }
    ],
    "tags": ["해안도로", "일출", "자연"],
    "createdAt": "2024-12-01T10:00:00",
    "updatedAt": "2025-01-10T15:00:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 맞춤 코스 생성

사용자가 선택한 장소들로 최적 경로를 계산합니다.

```http
POST /api/v1/courses/custom
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "나만의 제주 코스",
  "placeIds": [10, 11, 15, 20, 25],
  "startPlaceId": 10,
  "endPlaceId": 25,
  "transportType": "CAR",
  "optimizeRoute": true
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | String | ✅ | 코스 이름 |
| placeIds | Array | ✅ | 방문할 장소 ID 배열 |
| startPlaceId | Long | ✅ | 시작 장소 ID |
| endPlaceId | Long | ❌ | 종료 장소 ID (미지정 시 시작 장소로) |
| transportType | String | ✅ | CAR, WALK, PUBLIC_TRANSPORT |
| optimizeRoute | Boolean | ❌ | 경로 최적화 여부 (기본값: true) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "courseId": 100,
    "name": "나만의 제주 코스",
    "totalDistance": 52.3,
    "estimatedTime": 420,
    "waypoints": [
      {
        "order": 1,
        "placeId": 10,
        "estimatedArrival": "09:00"
      },
      {
        "order": 2,
        "placeId": 15,
        "estimatedArrival": "10:30"
      }
    ],
    "routes": [
      {
        "from": {"placeId": 10},
        "to": {"placeId": 15},
        "distance": 12.5,
        "duration": 18,
        "path": [...]
      }
    ]
  },
  "message": "맞춤 코스가 생성되었습니다.",
  "errorCode": null
}
```

---

### 4. 코스 저장 (내 코스로 저장)

```http
POST /api/v1/courses/{courseId}/save
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "savedCourseId": 1
  },
  "message": "코스가 저장되었습니다.",
  "errorCode": null
}
```

---

### 5. 내 코스 목록 조회

```http
GET /api/v1/courses/me?page=0&size=10
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "courseId": 100,
        "name": "나만의 제주 코스",
        "thumbnailImage": "...",
        "placeCount": 5,
        "totalDistance": 52.3,
        "estimatedTime": 420,
        "savedAt": "2025-01-15T10:00:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 3,
    "totalPages": 1
  },
  "message": null,
  "errorCode": null
}
```

---

### 6. 코스 삭제

```http
DELETE /api/v1/courses/{courseId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "코스가 삭제되었습니다.",
  "errorCode": null
}
```

---

### 7. 코스 좋아요 추가/취소

```http
POST /api/v1/courses/{courseId}/like
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

### 8. 경로 계산 (두 장소 간)

```http
POST /api/v1/courses/calculate-route
Content-Type: application/json
```

**Request Body**

```json
{
  "fromPlaceId": 10,
  "toPlaceId": 15,
  "transportType": "CAR"
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "distance": 12.5,
    "duration": 18,
    "transportType": "CAR",
    "path": [
      {"latitude": 33.4606, "longitude": 126.9424},
      {"latitude": 33.4500, "longitude": 126.9350},
      {"latitude": 33.4400, "longitude": 126.9300}
    ],
    "instructions": [
      {
        "step": 1,
        "instruction": "성산일출봉에서 출발",
        "distance": 0,
        "duration": 0
      },
      {
        "step": 2,
        "instruction": "동쪽 방향으로 5.2km 직진",
        "distance": 5.2,
        "duration": 8
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

### CourseResponse

```typescript
interface CourseResponse {
  courseId: number;
  name: string;
  description: string;
  thumbnailImage: string;
  images: string[];
  location: string;
  duration: number; // 일 단위
  theme: CourseTheme;
  distance: number; // km
  estimatedTime: number; // 분
  transportType: TransportType;
  difficulty: Difficulty;
  rating: number;
  reviewCount: number;
  likeCount: number;
  isLiked: boolean;
  waypoints: Waypoint[];
  routes: Route[];
  recommendedProducts: ProductResponse[];
  tags: string[];
  createdAt: string;
  updatedAt: string;
}

interface Waypoint {
  order: number;
  place: PlaceInfo;
  visitDuration: number; // 분
  description: string;
  tips: string;
}

interface Route {
  from: PlaceInfo;
  to: PlaceInfo;
  distance: number; // km
  duration: number; // 분
  transportType: TransportType;
  path: Coordinate[];
}

interface Coordinate {
  latitude: number;
  longitude: number;
}

type CourseTheme = 'NATURE' | 'FOOD' | 'CULTURE' | 'ACTIVITY' | 'SHOPPING' | 'MIXED';
type TransportType = 'CAR' | 'WALK' | 'PUBLIC_TRANSPORT' | 'BICYCLE';
type Difficulty = 'EASY' | 'MODERATE' | 'HARD';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| COURSE_NOT_FOUND | 404 | 코스를 찾을 수 없음 |
| INVALID_PLACE_SEQUENCE | 400 | 유효하지 않은 장소 순서 |
| TOO_MANY_WAYPOINTS | 400 | 경유지가 너무 많음 (최대 20개) |
| ROUTE_CALCULATION_FAILED | 500 | 경로 계산 실패 |
| UNREACHABLE_DESTINATION | 400 | 도달 불가능한 목적지 |

---

## 특수 기능

### 경로 최적화 알고리즘

코스 생성 시 `optimizeRoute: true`로 설정하면:
- **TSP (Traveling Salesman Problem)** 알고리즘 적용
- 시작점과 종료점을 제외한 중간 경유지의 방문 순서 최적화
- 총 이동 거리 최소화

### 캐싱 전략

자주 조회되는 코스 경로는 Redis에 캐싱:
- **Cache Key**: `course:route:{fromPlaceId}:{toPlaceId}:{transportType}`
- **TTL**: 24시간
- 도로 상황 변경 시 자동 갱신

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
