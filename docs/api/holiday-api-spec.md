# 공휴일 API 명세서 (Holiday API Specification)

**Base URL**: `/api/v1/holidays`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15  
**구현 상태**: 🚧 부분 구현 (기본 조회 및 동기화만 가능)

---

## ⚠️ 중요 공지

현재 **공휴일 조회 및 동기화 기능만 구현**되어 있습니다.
영업일 계산, 캘린더 데이터 등의 고급 기능은 향후 구현 예정입니다.

---

## 개요

Holiday API는 공휴일 정보 조회 및 동기화 기능을 제공합니다.

### 현재 구현된 기능
- ✅ 연도/월별 공휴일 조회
- ✅ 공휴일 동기화 (관리자용)

### 🚧 향후 구현 예정
- 특정 날짜 공휴일 여부 확인
- 기간 내 영업일 계산
- N일 후 영업일 계산
- 월별 캘린더 데이터 조회
- 다음 공휴일 조회
- 롱 위켄드 조회

---

## API 엔드포인트

### 1. 연도/월별 공휴일 조회 ✅

```http
GET /api/v1/holidays?year=2025&month=1
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| year | Integer | ✅ | 조회할 연도 |
| month | Integer | ✅ | 조회할 월 (1-12) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "date": "2025-01-01",
      "name": "신정",
      "isRecurring": true
    },
    {
      "date": "2025-01-28",
      "name": "설날 연휴",
      "isRecurring": false
    },
    {
      "date": "2025-01-29",
      "name": "설날",
      "isRecurring": false
    },
    {
      "date": "2025-01-30",
      "name": "설날 연휴",
      "isRecurring": false
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 2. 공휴일 동기화 (관리자용) ✅

외부 API(행정안전부)로부터 공휴일 정보를 가져와 DB에 저장합니다.

```http
POST /api/v1/holidays/sync?year=2025&month=01
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| year | String | ✅ | 4자리 연도 (예: "2025") |
| month | String | ✅ | 2자리 월 (예: "01", "12") |

**Response (200 OK)**

```
공휴일 동기화 완료
```

**설명**:
- 행정안전부 공공데이터 API를 호출하여 공휴일 정보 가져오기
- DB에 저장 (중복 제거)
- 관리자만 호출 가능

---

## 🚧 향후 구현 예정 API

아래 API들은 현재 백엔드에서 구현되지 않았으며, 향후 추가될 예정입니다.

### 3. 특정 날짜 공휴일 여부 확인 🚧

```http
GET /api/v1/holidays/check?date=2025-01-01
```

**예상 Response**

```json
{
  "success": true,
  "data": {
    "date": "2025-01-01",
    "isHoliday": true,
    "isWeekend": false,
    "holiday": {
      "name": "신정",
      "isRecurring": true
    }
  },
  "message": null,
  "errorCode": null
}
```

### 4. 기간 내 영업일 계산 🚧

```http
GET /api/v1/holidays/business-days?startDate=2025-01-01&endDate=2025-01-31
```

### 5. N일 후 영업일 계산 🚧

```http
GET /api/v1/holidays/add-business-days?baseDate=2025-01-02&days=5
```

### 6. 월별 캘린더 데이터 조회 🚧

```http
GET /api/v1/holidays/calendar?year=2025&month=1
```

### 7. 다음 공휴일 조회 🚧

```http
GET /api/v1/holidays/next?baseDate=2025-01-15&count=2
```

### 8. 롱 위켄드 조회 🚧

3일 이상 연휴가 되는 기간을 조회합니다.

```http
GET /api/v1/holidays/long-weekends?year=2025
```

---

## 데이터 모델

### HolidayResponse

```typescript
interface HolidayResponse {
  date: string; // YYYY-MM-DD
  name: string;
  isRecurring: boolean; // 매년 반복 여부
}

// 향후 추가될 필드
interface DetailedHolidayResponse extends HolidayResponse {
  type: HolidayType; // 'NATIONAL' | 'LUNAR' | 'SUBSTITUTE' | 'TEMPORARY'
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INVALID_DATE_FORMAT | 400 | 유효하지 않은 날짜 형식 |
| INVALID_YEAR_RANGE | 400 | 지원하지 않는 연도 범위 |
| SYNC_FAILED | 500 | 공휴일 동기화 실패 |

---

## 비즈니스 로직

### 공휴일 데이터 소스

- **한국 공휴일**: 행정안전부 공공데이터 API 연동
- **음력 공휴일**: 외부 API에서 제공 (설날, 추석)
- **대체공휴일**: 외부 API에서 제공

### 동기화 프로세스

1. 관리자가 `/sync` API 호출
2. 행정안전부 API에 해당 연도/월의 공휴일 요청
3. 응답 데이터 파싱
4. DB에 저장 (중복 제거)
5. 성공 메시지 반환

### 자동 업데이트 (향후 구현 예정)

- 매년 1월 1일 00:00에 다음 연도 공휴일 자동 동기화
- 정부 공휴일 변경 시 관리자가 수동으로 재동기화 가능

---

## 활용 예시

### 예약 가능일 필터링 (향후)

```typescript
// 영업일만 예약 가능하도록 필터링
const checkAvailability = async (date: string) => {
  const response = await fetch(`/api/v1/holidays/check?date=${date}`);
  const data = await response.json();
  
  if (data.data.isHoliday || data.data.isWeekend) {
    return false; // 예약 불가
  }
  return true; // 예약 가능
};
```

### 배송일 계산 (향후)

```typescript
// 주문일로부터 3영업일 후 배송
const calculateDeliveryDate = async (orderDate: string) => {
  const response = await fetch(
    `/api/v1/holidays/add-business-days?baseDate=${orderDate}&days=3`
  );
  const data = await response.json();
  return data.data.resultDate;
};
```

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today

**개발 로드맵**:
- ✅ Phase 1: 공휴일 조회 및 동기화 (완료)
- 🚧 Phase 2: 공휴일 확인 API (개발 예정)
- 🚧 Phase 3: 영업일 계산 기능 (개발 예정)
- 🚧 Phase 4: 캘린더 및 고급 기능 (개발 예정)
