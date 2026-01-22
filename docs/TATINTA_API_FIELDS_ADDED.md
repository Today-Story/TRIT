# Tatinta API 필드 추가 완료 보고서

**작성일**: 2025-01-20  
**작업자**: Backend Team  
**관련 이슈**: Tatinta API `/external/api/tatinta/product/{id}` 응답 필드 추가

---

## 📋 작업 개요

`DetailProductResponse`에 있는 모든 필드를 `TatintaProductDetailDto`에 추가하여, Tatinta 파트너가 TRIT의 모든 상품 정보를 받을 수 있도록 개선했습니다.

---

## ✅ 추가된 필드 목록

### 1. **기본 정보 필드**

| 필드명 | 타입 | 설명 | JSON 키 |
|--------|------|------|----------|
| includeService | String | 포함 서비스 | `include_service` |
| excludeService | String | 제외 서비스 | `exclude_service` |
| detailDescription | String | 한 줄 설명 | `detail_description` |
| notices | String | 유의사항 | `notices` |
| reservationDeadlineHours | Integer | 예약 마감 시간 (시간 단위) | `reservation_deadline_hours` |

### 2. **예약 가능 시간대 (Available Times)**

```json
{
  "available_times": [
    {
      "id": 1,
      "group_id": "morning-slot",
      "day_of_week": "MONDAY",
      "start_time": "09:00",
      "end_time": "18:00",
      "minimum_people": 2.0,
      "maximum_people": 10.0
    }
  ]
}
```

**중요**: 요일별 예약 가능 시간대를 제공하여 캘린더 구현에 활용 가능합니다.

### 3. **판매 기간 (Schedule)**

| 필드명 | 타입 | 설명 | JSON 키 |
|--------|------|------|----------|
| startDate | String | 판매 시작일 | `start_date` |
| endDate | String | 판매 종료일 | `end_date` |

**형식**: `yyyy-MM-dd` (예: `"2025-01-01"`)

### 4. **운영 시간 (Business Hours)**

```json
{
  "business_hours": [
    {
      "day_of_week": "MONDAY",
      "start_time": "09:00",
      "end_time": "18:00"
    }
  ]
}
```

### 5. **업체 휴무일 (Company Holidays)**

```json
{
  "company_holidays": [
    {
      "holiday_date": "2025-01-01",
      "holiday_name": "신정"
    }
  ]
}
```

### 6. **상품 옵션 (Product Options)**

```json
{
  "product_options": [
    {
      "id": 1,
      "option_name": "사이즈 선택",
      "option_type": "SINGLE_SELECT",
      "essential": true,
      "option_values": [
        {
          "id": 1,
          "value_name": "S",
          "additional_price": 0
        },
        {
          "id": 2,
          "value_name": "M",
          "additional_price": 5000
        }
      ]
    }
  ]
}
```

**옵션 타입**:
- `SINGLE_SELECT`: 라디오 버튼 (단일 선택)
- `MULTI_SELECT`: 체크박스 (다중 선택)

### 7. **인원별 옵션 (People Options)**

```json
{
  "people_options": [
    {
      "id": 1,
      "people_count": 10000.0,
      "price": 10000.0
    }
  ]
}
```

**참고**: 현재 `PeopleOptionResponse`에는 `optionName`과 `optionPrice`만 있어서, `peopleCount`와 `price`에 `optionPrice` 값을 사용합니다.

### 8. **추가 상품 (Additional Products)**

```json
{
  "additional_products": [
    {
      "id": 1,
      "name": "점심 도시락",
      "price": 10000.0,
      "description": "일반: 10000원, 채식: 12000원"
    }
  ]
}
```

---

## 🔧 구현 상세

### 수정된 파일

#### 1. `TatintaProductDetailDto.java`
- 추가 필드 선언
- 내부 DTO 클래스 추가:
  - `AvailableTimeDto`
  - `BusinessHourDto`
  - `HolidayDto`
  - `ProductOptionDto`
  - `OptionValueDto`
  - `PeopleOptionDto`
  - `AdditionalProductDto`

#### 2. `TatintaProductMapper.java`
- `toDetailDto()` 메서드에 필드 매핑 추가
- 변환 헬퍼 메서드 추가:
  - `convertAvailableTimes()`
  - `convertBusinessHours()`
  - `convertCompanyHolidays()`
  - `convertProductOptions()`
  - `convertOptionValues()`
  - `convertPeopleOptions()`
  - `convertAdditionalProducts()`

---

## 📊 전체 응답 구조

```json
{
  "status": "SUCCESS",
  "data": {
    // 기존 필드
    "id": 123,
    "name": "제주도 한라산 등반 체험",
    "category": "PLAY",
    "price": 50000.0,
    "currency": "KRW",
    "discount_percent": 10,
    "price_type": "person",
    "duration": 1,
    "duration_type": "day",
    "images": [...],
    "city": {...},
    "description": "상세 설명",
    "status": "SALE",
    "min_people": 1,
    "max_people": 10,
    "company_id": "company123",
    "company_name": "제주투어",
    "company_phone": null,
    "location": {
      "address": "제주특별자치도 제주시",
      "latitude": 33.5101,
      "longitude": 126.5219,
      "google_map_id": "ChIJ..."
    },
    
    // ✅ 추가된 필드
    "include_service": "가이드, 입장권, 간식 포함",
    "exclude_service": "개인 식사, 교통비 불포함",
    "detail_description": "제주도 한라산 등반 특별 체험",
    "notices": "우천 시 일정이 변경될 수 있습니다.",
    "reservation_deadline_hours": 24,
    "start_date": "2025-01-01",
    "end_date": "2025-12-31",
    "available_times": [...],
    "business_hours": [...],
    "company_holidays": [...],
    "product_options": [...],
    "people_options": [...],
    "additional_products": [...]
  },
  "message": null
}
```

---

## 🎯 Tatinta 활용 가이드

### 1. **캘린더 비활성화 로직**

```typescript
function isDateDisabled(date: Date, product: TatintaProductDetailDto): boolean {
  // 1. 판매 기간 체크
  if (date < new Date(product.start_date) || date > new Date(product.end_date)) {
    return true;
  }
  
  // 2. 요일별 예약 가능 시간 체크
  const dayOfWeek = getDayOfWeek(date); // 'MONDAY', 'TUESDAY', ...
  const hasAvailableTime = product.available_times.some(
    time => time.day_of_week === dayOfWeek
  );
  if (!hasAvailableTime) {
    return true;
  }
  
  // 3. 휴무일 체크
  const isHoliday = product.company_holidays.some(
    holiday => holiday.holiday_date === formatDate(date)
  );
  if (isHoliday) {
    return true;
  }
  
  // 4. 운영 시간 체크
  const hasBusinessHour = product.business_hours.some(
    hour => hour.day_of_week === dayOfWeek
  );
  if (!hasBusinessHour) {
    return true;
  }
  
  return false;
}
```

### 2. **타임슬롯 필터링**

```typescript
function getTimeSlotsForDate(
  date: Date,
  product: TatintaProductDetailDto
): AvailableTimeDto[] {
  const dayOfWeek = getDayOfWeek(date);
  
  return product.available_times
    .filter(time => time.day_of_week === dayOfWeek)
    .sort((a, b) => a.start_time.localeCompare(b.start_time));
}
```

### 3. **옵션 UI 렌더링**

```typescript
function renderOption(option: ProductOptionDto) {
  if (option.option_type === 'SINGLE_SELECT') {
    return <RadioGroup options={option.option_values} />;
  } else {
    return <CheckboxGroup options={option.option_values} />;
  }
}
```

### 4. **가격 계산**

```typescript
function calculateTotalPrice(
  basePrice: number,
  participants: number,
  selectedOptions: Map<number, number[]>
): number {
  let optionPrice = 0;
  
  selectedOptions.forEach((valueIds, optionId) => {
    const option = product.product_options.find(o => o.id === optionId);
    if (!option) return;
    
    valueIds.forEach(valueId => {
      const value = option.option_values.find(v => v.id === valueId);
      if (value) {
        optionPrice += value.additional_price * participants;
      }
    });
  });
  
  return basePrice * participants + optionPrice;
}
```

---

## ✅ 테스트 방법

### 1. Swagger UI 접속

```
http://localhost:8080/swagger-ui/index.html
```

### 2. 상품 상세 조회 API 테스트

```
GET /external/api/tatinta/product/{id}
```

**예시**: `GET /external/api/tatinta/product/172`

### 3. 응답 확인

모든 추가 필드가 포함되어 있는지 확인:
- `available_times` 배열
- `business_hours` 배열
- `company_holidays` 배열
- `product_options` 배열
- `start_date`, `end_date`
- `include_service`, `exclude_service`
- `notices`, `detail_description`

---

## 🔄 호환성

### ✅ 하위 호환성 유지
- 기존 필드는 변경 없음
- 새 필드는 null 가능 (`@JsonInclude(JsonInclude.Include.NON_NULL)`)
- 기존 Tatinta 클라이언트는 새 필드를 무시하고 정상 동작

### ⚠️ 주의사항
- `people_options`의 `people_count`와 `price`는 현재 같은 값 (`optionPrice`)을 사용
  - 향후 `PeopleOptionResponse`에 실제 `peopleCount` 필드 추가 필요

---

## 📝 다음 단계

### Tatinta 측 작업 필요
1. **API 통합 테스트**
   - 실제 상품 ID로 API 호출
   - 모든 필드 파싱 확인

2. **캘린더 구현**
   - `available_times` 기반 날짜 비활성화
   - `business_hours` 기반 운영시간 표시
   - `company_holidays` 기반 휴무일 표시

3. **옵션 UI 구현**
   - `SINGLE_SELECT` → 라디오 버튼
   - `MULTI_SELECT` → 체크박스
   - 필수 옵션(`essential: true`) 검증

4. **가격 계산 로직 구현**
   - 기본 가격 × 인원 수
   - 옵션별 추가 가격 × 인원 수
   - 총 가격 계산

---

## 📞 문의

추가 필드가 필요하거나 기존 필드 수정이 필요한 경우 연락 주세요.

**TRIT Backend Team**  
backend-team@trit.today

---

**변경 이력**:
- 2025-01-20: 초기 작성 및 모든 필드 추가 완료
