# 코스 API 명세서 (Course API Specification)

**Base URL**: `/api/v1/courses`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15  
**구현 상태**: ✅ 대부분 구현됨

---

## 개요

Course API는 여행 코스 추천 및 경로 계산 기능을 제공합니다.

### 주요 기능
- ✅ 코스 생성 및 관리 (CRUD)
- ✅ 코스 조회 (전체, 상세, 내 코스)
- ✅ 코스 스크랩 (찜하기)
- ✅ bbox 영역 내 코스 필터링
- ✅ 코스 검색 및 정렬
- ✅ 코스에 등록 가능한 장소 조회

---

## API 엔드포인트

### 1. 코스 생성 ✅

```http
POST /api/v1/courses
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "제주 동부 해안 일주 코스",
  "description": "성산일출봉부터 우도까지 동부 해안을 따라가는 코스",
  "thumbnailImage": "https://s3.amazonaws.com/trit/courses/thumb.jpg",
  "locationIds": [10, 15, 20, 25],
  "category": "NATURE",
  "tags": ["해안도로", "일출", "자연"]
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": 1,
  "message": "코스가 생성되었습니다.",
  "errorCode": null
}
```

---

### 2. 모든 코스 조회 ✅

```http
GET /api/v1/courses/all?sortOption=LATEST&keyword=제주
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| sortOption | String | ❌ | LATEST, POPULAR, RATING (기본값: LATEST) |
| keyword | String | ❌ | 검색 키워드 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "courseId": 1,
      "name": "제주 동부 해안 일주 코스",
      "description": "성산일출봉부터 우도까지...",
      "thumbnailImage": "https://s3.amazonaws.com/trit/courses/1/thumb.jpg",
      "locationCount": 5,
      "distance": 45.5,
      "estimatedTime": 360,
      "scrapCount": 856,
      "isScrapped": false,
      "category": "NATURE",
      "tags": ["해안도로", "일출", "자연"],
      "createdAt": "2024-12-01T10:00:00"
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 3. bbox 영역 내 코스 조회 ✅

특정 지도 영역(bounding box) 내의 코스만 필터링합니다.

```http
GET /api/v1/courses/all/bbox?bbox=126.0,33.0,127.0,34.0&sortOption=LATEST
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| bbox | Array | ✅ | [west, south, east, north] 4개 좌표 |
| sortOption | String | ❌ | LATEST, POPULAR, RATING |
| keyword | String | ❌ | 검색 키워드 |

**bbox 형식**: `west,south,east,north` (서경, 남위, 동경, 북위)

**Response**: 2번 API와 동일

---

### 4. 코스 상세 조회 ✅

```http
GET /api/v1/courses/detail?courseId=1&latitude=37.413294&longitude=127.0016985
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| courseId | Long | ✅ | 코스 ID |
| latitude | Double | ❌ | 현재 위치 위도 (기본값: 37.413294) |
| longitude | Double | ❌ | 현재 위치 경도 (기본값: 127.0016985) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "courseId": 1,
    "name": "제주 동부 해안 일주 코스",
    "description": "상세 설명...",
    "thumbnailImage": "https://...",
    "images": ["https://...", "https://..."],
    "category": "NATURE",
    "distance": 45.5,
    "estimatedTime": 360,
    "scrapCount": 856,
    "isScrapped": false,
    "tags": ["해안도로", "일출"],
    "locations": [
      {
        "locationId": 10,
        "name": "성산일출봉",
        "latitude": 33.4606,
        "longitude": 126.9424,
        "category": "LANDMARK",
        "address": "제주특별자치도 서귀포시...",
        "description": "일출 명소",
        "distanceFromUser": 150.5
      }
    ],
    "creator": {
      "userId": 5,
      "nickname": "제주여행러버",
      "profileImage": "https://..."
    },
    "createdAt": "2024-12-01T10:00:00",
    "updatedAt": "2025-01-10T15:00:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 5. 코스 수정 ✅

```http
PUT /api/v1/courses/update?courseId=1
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**: 생성과 동일한 형식

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "코스가 수정되었습니다.",
  "errorCode": null
}
```

---

### 6. 코스 삭제 ✅

```http
DELETE /api/v1/courses/delete?courseId=1
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

### 7. 내가 만든 코스 조회 ✅

```http
GET /api/v1/courses/my?sortOption=LATEST&keyword=제주
Cookie: accessToken=eyJ...
```

**Query Parameters**: 2번 API와 동일

**Response**: 2번 API와 동일

---

### 8. 내가 스크랩한 코스 조회 ✅

```http
GET /api/v1/courses/my/scrap?sortOption=LATEST&keyword=제주
Cookie: accessToken=eyJ...
```

**Query Parameters**: 2번 API와 동일

**Response**: 2번 API와 동일

---

### 9. 코스 스크랩/스크랩 취소 ✅

코스를 찜하거나 찜 취소합니다. 토글 방식으로 동작합니다.

```http
PATCH /api/v1/courses/scrap?courseId=1
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "코스가 스크랩되었습니다.",
  "errorCode": null
}
```

또는

```json
{
  "success": true,
  "data": null,
  "message": "코스 스크랩이 취소되었습니다.",
  "errorCode": null
}
```

---

### 10. 코스에 등록 가능한 장소 조회 ✅

코스를 만들 때 추가할 수 있는 장소(Location) 목록을 조회합니다.

```http
GET /api/v1/courses/locations?keyword=성산&category=LANDMARK
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| keyword | String | ❌ | 장소 이름 검색 |
| category | String | ❌ | 카테고리 필터 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "locationId": 10,
        "name": "성산일출봉",
        "latitude": 33.4606,
        "longitude": 126.9424,
        "category": "LANDMARK",
        "address": "제주특별자치도 서귀포시 성산읍",
        "thumbnailImage": "https://..."
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 45,
    "totalPages": 3
  },
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

### SimpleCourseResponse

```typescript
interface SimpleCourseResponse {
  courseId: number;
  name: string;
  description: string;
  thumbnailImage: string;
  locationCount: number;
  distance: number; // km
  estimatedTime: number; // 분
  scrapCount: number;
  isScrapped: boolean;
  category: Category;
  tags: string[];
  createdAt: string;
}
```

### DetailCourseResponse

```typescript
interface DetailCourseResponse extends SimpleCourseResponse {
  images: string[];
  locations: CourseLocation[];
  creator: {
    userId: number;
    nickname: string;
    profileImage: string;
  };
  updatedAt: string;
}

interface CourseLocation {
  locationId: number;
  name: string;
  latitude: number;
  longitude: number;
  category: Category;
  address: string;
  description: string;
  distanceFromUser: number; // km (사용자 현재 위치로부터)
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| COURSE_NOT_FOUND | 404 | 코스를 찾을 수 없음 |
| INVALID_BBOX | 400 | bbox는 4개 좌표 필요 (west,south,east,north) |
| UNAUTHORIZED | 401 | 인증 필요 |
| FORBIDDEN | 403 | 권한 없음 (본인 코스만 수정/삭제 가능) |

---

## 비즈니스 로직

### 정렬 옵션 (SortOption)

- **LATEST**: 최신 생성일 순
- **POPULAR**: 스크랩 수 많은 순
- **RATING**: 평점 높은 순 (향후 구현)

### 코스 거리 및 소요 시간 계산

- 거리: 각 장소 간 직선 거리의 합
- 소요 시간: 거리 × 평균 속도 + 각 장소 방문 시간

### Bbox 필터링

지도에서 보이는 영역 내의 코스만 표시하기 위해 사용:
1. 코스의 모든 장소 위치 확인
2. 하나라도 bbox 내에 있으면 해당 코스 포함

---

## 용어 정리

**⚠️ 중요**: 이 API에서는 "스크랩(scrap)"이라는 용어를 사용합니다.
- **스크랩(Scrap)**: 코스를 찜하는 기능 (좋아요와 유사)
- **스크랩 수(scrapCount)**: 해당 코스를 스크랩한 사용자 수
- **내가 스크랩한 코스**: 내가 찜한 코스 목록

다른 도메인(Product, Contents)에서는 "좋아요(like)"를 사용하지만, Course에서는 "스크랩"을 사용합니다.

---

**문서 작성자**: Ted  
**문의**: backend-team@trit.today
