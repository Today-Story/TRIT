# 공휴일 API 명세서 (Holiday API Specification)

**Base URL**: `/api/v1/holidays`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Holiday API는 공휴일 정보 조회 및 예약 가능 여부 확인 기능을 제공합니다.

### 주요 기능
- 연도별 공휴일 목록 조회
- 특정 날짜 공휴일 여부 확인
- 영업일 계산
- 공휴일 캘린더 데이터

---

## API 엔드포인트

### 1. 연도별 공휴일 목록 조회

```http
GET /api/v1/holidays?year=2025
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| year | Integer | ✅ | 조회할 연도 (2020-2030) |
| month | Integer | ❌ | 특정 월 필터 (1-12) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "year": 2025,
    "holidays": [
      {
        "date": "2025-01-01",
        "name": "신정",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-01-28",
        "name": "설날 연휴",
        "isRecurring": false,
        "type": "LUNAR"
      },
      {
        "date": "2025-01-29",
        "name": "설날",
        "isRecurring": false,
        "type": "LUNAR"
      },
      {
        "date": "2025-01-30",
        "name": "설날 연휴",
        "isRecurring": false,
        "type": "LUNAR"
      },
      {
        "date": "2025-03-01",
        "name": "삼일절",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-05-05",
        "name": "어린이날",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-05-06",
        "name": "대체공휴일(어린이날)",
        "isRecurring": false,
        "type": "SUBSTITUTE"
      },
      {
        "date": "2025-06-06",
        "name": "현충일",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-08-15",
        "name": "광복절",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-10-03",
        "name": "개천절",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-10-09",
        "name": "한글날",
        "isRecurring": true,
        "type": "NATIONAL"
      },
      {
        "date": "2025-12-25",
        "name": "크리스마스",
        "isRecurring": true,
        "type": "NATIONAL"
      }
    ],
    "totalHolidays": 12
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 특정 날짜 공휴일 여부 확인

```http
GET /api/v1/holidays/check?date=2025-01-01
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| date | String | ✅ | 확인할 날짜 (YYYY-MM-DD) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "date": "2025-01-01",
    "isHoliday": true,
    "isWeekend": false,
    "holiday": {
      "name": "신정",
      "type": "NATIONAL"
    }
  },
  "message": null,
  "errorCode": null
}
```

**주말인 경우 (200 OK)**

```json
{
  "success": true,
  "data": {
    "date": "2025-01-04",
    "isHoliday": false,
    "isWeekend": true,
    "holiday": null
  },
  "message": null,
  "errorCode": null
}
```

---

### 3. 기간 내 영업일 계산

```http
GET /api/v1/holidays/business-days?startDate=2025-01-01&endDate=2025-01-31
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| startDate | String | ✅ | 시작일 (YYYY-MM-DD) |
| endDate | String | ✅ | 종료일 (YYYY-MM-DD) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "startDate": "2025-01-01",
    "endDate": "2025-01-31",
    "totalDays": 31,
    "businessDays": 21,
    "holidays": 4,
    "weekends": 8,
    "details": [
      {
        "date": "2025-01-01",
        "isBusinessDay": false,
        "reason": "신정"
      },
      {
        "date": "2025-01-02",
        "isBusinessDay": true,
        "reason": null
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

---

### 4. N일 후 영업일 계산

```http
GET /api/v1/holidays/add-business-days?baseDate=2025-01-02&days=5
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| baseDate | String | ✅ | 기준일 (YYYY-MM-DD) |
| days | Integer | ✅ | 추가할 영업일 수 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "baseDate": "2025-01-02",
    "businessDaysToAdd": 5,
    "resultDate": "2025-01-09",
    "actualDaysElapsed": 7
  },
  "message": null,
  "errorCode": null
}
```

---

### 5. 월별 캘린더 데이터 조회

```http
GET /api/v1/holidays/calendar?year=2025&month=1
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "year": 2025,
    "month": 1,
    "weeks": [
      {
        "weekNumber": 1,
        "days": [
          {
            "date": "2024-12-29",
            "dayOfWeek": "SUNDAY",
            "isCurrentMonth": false,
            "isHoliday": false,
            "isWeekend": true
          },
          {
            "date": "2024-12-30",
            "dayOfWeek": "MONDAY",
            "isCurrentMonth": false,
            "isHoliday": false,
            "isWeekend": false
          },
          {
            "date": "2025-01-01",
            "dayOfWeek": "WEDNESDAY",
            "isCurrentMonth": true,
            "isHoliday": true,
            "holidayName": "신정",
            "isWeekend": false
          }
        ]
      }
    ]
  },
  "message": null,
  "errorCode": null
}
```

---

### 6. 다음 공휴일 조회

```http
GET /api/v1/holidays/next?baseDate=2025-01-15
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| baseDate | String | ❌ | 기준일 (기본값: 오늘) |
| count | Integer | ❌ | 조회할 공휴일 수 (기본값: 1, 최대: 10) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "date": "2025-01-28",
      "name": "설날 연휴",
      "type": "LUNAR",
      "daysUntil": 13
    },
    {
      "date": "2025-01-29",
      "name": "설날",
      "type": "LUNAR",
      "daysUntil": 14
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

### 7. 롱 위켄드 조회

3일 이상 연휴가 되는 기간을 조회합니다.

```http
GET /api/v1/holidays/long-weekends?year=2025
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": [
    {
      "startDate": "2025-01-28",
      "endDate": "2025-01-30",
      "days": 3,
      "name": "설날 연휴",
      "holidays": ["설날 연휴", "설날", "설날 연휴"]
    },
    {
      "startDate": "2025-05-03",
      "endDate": "2025-05-06",
      "days": 4,
      "name": "어린이날 연휴",
      "holidays": ["토요일", "일요일", "어린이날", "대체공휴일"]
    }
  ],
  "message": null,
  "errorCode": null
}
```

---

## 데이터 모델

### HolidayResponse

```typescript
interface HolidayResponse {
  date: string; // YYYY-MM-DD
  name: string;
  isRecurring: boolean; // 매년 반복 여부
  type: HolidayType;
}

type HolidayType = 'NATIONAL' | 'LUNAR' | 'SUBSTITUTE' | 'TEMPORARY';

interface CalendarDay {
  date: string;
  dayOfWeek: 'MONDAY' | 'TUESDAY' | 'WEDNESDAY' | 'THURSDAY' | 'FRIDAY' | 'SATURDAY' | 'SUNDAY';
  isCurrentMonth: boolean;
  isHoliday: boolean;
  holidayName: string | null;
  isWeekend: boolean;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INVALID_DATE_FORMAT | 400 | 유효하지 않은 날짜 형식 |
| INVALID_YEAR_RANGE | 400 | 지원하지 않는 연도 범위 (2020-2030) |
| DATE_RANGE_TOO_LARGE | 400 | 날짜 범위가 너무 큼 (최대 366일) |

---

## 비즈니스 로직

### 공휴일 데이터 소스

- **한국 공휴일**: 행정안전부 공공데이터 API 연동
- **음력 공휴일**: 자체 계산 로직 (설날, 추석)
- **대체공휴일**: 공휴일이 주말과 겹치는 경우 자동 계산

### 자동 업데이트

- 매년 1월 1일 00:00에 다음 연도 공휴일 자동 업데이트
- 정부 공휴일 변경 시 관리자가 수동으로 업데이트 가능

### 캐싱 전략

- **Redis Cache Key**: `holiday:year:{year}`
- **TTL**: 365일 (1년)
- 공휴일 데이터 변경 시 캐시 무효화

---

## 활용 예시

### 예약 가능일 필터링

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

### 배송일 계산

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
