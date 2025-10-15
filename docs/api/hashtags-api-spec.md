# 해시태그 API 명세서 (Hashtags API Specification)

**Base URL**: `/api/v1/hashtags`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15  
**구현 상태**: 🚧 최소 기능만 구현 (인기 해시태그 조회만 가능)

---

## ⚠️ 중요 공지

현재 **인기 해시태그 조회 기능만 구현**되어 있습니다.
검색, 자동완성, 팔로우 등의 고급 기능은 향후 구현 예정입니다.

---

## 개요

Hashtags API는 콘텐츠 및 상품의 해시태그 관리 기능을 제공합니다.

### 현재 구현된 기능
- ✅ 인기 해시태그 조회

### 🚧 향후 구현 예정
- 해시태그 검색
- 해시태그 자동완성
- 해시태그별 콘텐츠/상품 조회
- 트렌딩 해시태그
- 관련 해시태그 추천
- 해시태그 팔로우 기능

---

## API 엔드포인트

### 1. 인기 해시태그 조회 ✅

```http
GET /api/v1/hashtags/popular?limit=5
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| limit | Integer | ❌ | 조회 개수 (기본값: 5) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    "제주여행",
    "부산맛집",
    "서울여행",
    "강릉카페",
    "전주한옥마을"
  ],
  "message": null,
  "errorCode": null
}
```

**설명**:
- 가장 많이 사용된 해시태그를 문자열 배열로 반환
- 사용 빈도 순으로 정렬
- limit 파라미터로 개수 조절 가능

---

## 🚧 향후 구현 예정 API

아래 API들은 현재 백엔드에서 구현되지 않았으며, 향후 추가될 예정입니다.

### 2. 해시태그 검색 🚧

```http
GET /api/v1/hashtags/search?keyword=제주&page=0&size=20
```

### 3. 해시태그 자동 완성 🚧

```http
GET /api/v1/hashtags/autocomplete?query=제
```

**예상 Response**

```json
{
  "success": true,
  "data": [
    {
      "hashtagId": 1,
      "name": "제주여행",
      "count": 15234
    },
    {
      "hashtagId": 5,
      "name": "제주맛집",
      "count": 8567
    },
    {
      "hashtagId": 10,
      "name": "제주도",
      "count": 20123
    }
  ],
  "message": null,
  "errorCode": null
}
```

### 4. 해시태그별 콘텐츠 조회 🚧

```http
GET /api/v1/hashtags/{hashtagId}/contents?page=0&size=20
Cookie: accessToken=eyJ... (선택)
```

### 5. 해시태그별 상품 조회 🚧

```http
GET /api/v1/hashtags/{hashtagId}/products?page=0&size=20
Cookie: accessToken=eyJ... (선택)
```

### 6. 트렌딩 해시태그 조회 🚧

최근 급상승 중인 해시태그를 조회합니다.

```http
GET /api/v1/hashtags/trending?limit=10
```

### 7. 관련 해시태그 조회 🚧

특정 해시태그와 함께 자주 사용되는 해시태그를 조회합니다.

```http
GET /api/v1/hashtags/{hashtagId}/related?limit=10
```

### 8. 해시태그 팔로우 🚧

```http
POST /api/v1/hashtags/{hashtagId}/follow
Cookie: accessToken=eyJ...
```

### 9. 내가 팔로우한 해시태그 목록 🚧

```http
GET /api/v1/hashtags/following?page=0&size=20
Cookie: accessToken=eyJ...
```

---

## 데이터 모델

### HashtagResponse (현재)

```typescript
// 현재 구현: 단순 문자열 배열
type HashtagListResponse = string[];

// 향후 구현 예정: 상세 정보 포함
interface HashtagResponse {
  hashtagId: number;
  name: string;
  count: number; // 사용 횟수
  category: Category | null;
  trendScore: number | null; // 트렌드 점수 (0-100)
  growthRate: number | null; // 증가율 (%)
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| HASHTAG_NOT_FOUND | 404 | 해시태그를 찾을 수 없음 |
| INVALID_HASHTAG_NAME | 400 | 유효하지 않은 해시태그 이름 |
| INVALID_LIMIT | 400 | 유효하지 않은 limit 값 |

---

## 비즈니스 로직

### 인기 해시태그 계산

현재는 단순히 사용 빈도를 기준으로 인기 해시태그를 계산합니다:

1. 콘텐츠와 상품에 사용된 모든 해시태그 집계
2. 사용 횟수 내림차순 정렬
3. 상위 N개 반환

### 향후 구현 예정 기능

#### 해시태그 정규화
입력된 해시태그는 자동으로 정규화될 예정:
- 공백 제거
- 소문자 변환 (영문인 경우)
- 특수문자 제거 (# 제외)
- 최대 길이: 20자

예시:
- `#제주 여행` → `제주여행`
- `#SEOUL` → `seoul`

#### 트렌드 스코어 계산
트렌드 스코어는 다음 요소를 고려하여 계산될 예정:
- 최근 사용 빈도 (50%)
- 증가율 (30%)
- 좋아요/공유 수 (20%)

---

## 사용 예시

### 프론트엔드에서 인기 해시태그 표시

```typescript
const fetchPopularHashtags = async (limit = 5) => {
  const response = await fetch(`/api/v1/hashtags/popular?limit=${limit}`);
  const result = await response.json();
  
  if (result.success) {
    return result.data; // ['제주여행', '부산맛집', ...]
  }
  return [];
};

// 사용
const hashtags = await fetchPopularHashtags(10);
hashtags.forEach(tag => {
  console.log(`#${tag}`);
});
```

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today

**개발 로드맵**:
- ✅ Phase 1: 인기 해시태그 조회 (완료)
- 🚧 Phase 2: 검색 및 자동완성 (개발 예정)
- 🚧 Phase 3: 해시태그별 콘텐츠/상품 조회 (개발 예정)
- 🚧 Phase 4: 트렌딩 및 추천 알고리즘 (개발 예정)
- 🚧 Phase 5: 팔로우 기능 (개발 예정)
