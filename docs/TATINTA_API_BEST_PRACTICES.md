# Tatinta API Integration - Best Practices Guide

**작성일**: 2025-01-20  
**버전**: 1.0  
**대상**: Tatinta 개발팀

---

## 📋 목차

1. [개요](#개요)
2. [AvailableTimes 처리 베스트 프랙티스](#availabletimes-처리-베스트-프랙티스)
3. [타임슬롯 처리 베스트 프랙티스](#타임슬롯-처리-베스트-프랙티스)
4. [옵션 선택 베스트 프랙티스](#옵션-선택-베스트-프랙티스)
5. [캘린더 구현 권장사항](#캘린더-구현-권장사항)
6. [성능 최적화](#성능-최적화)
7. [에러 핸들링](#에러-핸들링)
8. [사용자 경험 개선](#사용자-경험-개선)

---

## 개요

본 문서는 TRIT API를 활용한 예약 시스템 구현 시 권장되는 베스트 프랙티스를 정리합니다.

### 핵심 개념

```
상품 (Product)
  ├── 스케줄 (ProductSchedule)
  │   ├── startDate / endDate (판매 기간)
  │   └── availableTimes[] (예약 가능 시간대)
  ├── 옵션 (ProductOption[])
  │   ├── optionType (SINGLE_SELECT / MULTI_SELECT)
  │   └── optionValues[] (선택 가능한 값들)
  ├── 업체 정보 (Company)
  │   ├── businessHours[] (운영 시간)
  │   └── companyHolidays[] (휴무일)
  └── 환불 정책 (RefundPolicy)
```

---

## AvailableTimes 처리 베스트 프랙티스

### 1. 데이터 구조 이해

```json
{
  "availableTimes": [
    {
      "id": 1,
      "groupId": "morning-slot",
      "dayOfWeek": "MONDAY",
      "startTime": "09:00",
      "endTime": "12:00",
      "minimumPeople": 2.0,
      "maximumPeople": 10.0
    },
    {
      "id": 2,
      "groupId": "afternoon-slot",
      "dayOfWeek": "MONDAY",
      "startTime": "14:00",
      "endTime": "17:00",
      "minimumPeople": 2.0,
      "maximumPeople": 10.0
    }
  ]
}
```

### 2. 요일별 그룹핑 (권장)

```typescript
// ✅ BEST PRACTICE: 요일별로 미리 그룹핑
interface GroupedAvailableTimes {
  [key in DayOfWeek]?: ProductAvailableTimeResponse[];
}

function groupByDayOfWeek(
  availableTimes: ProductAvailableTimeResponse[]
): GroupedAvailableTimes {
  return availableTimes.reduce((acc, time) => {
    const day = time.dayOfWeek;
    if (!acc[day]) {
      acc[day] = [];
    }
    acc[day].push(time);
    return acc;
  }, {} as GroupedAvailableTimes);
}

// 사용 예시
const grouped = groupByDayOfWeek(productData.availableTimes);
const mondaySlots = grouped['MONDAY'] || [];
```

### 3. GroupId 활용

`groupId`는 동일한 시간대를 그룹핑하는 데 사용됩니다. 예를 들어:

```typescript
// ✅ BEST PRACTICE: groupId로 반복 패턴 인식
const morningSlots = availableTimes.filter(
  time => time.groupId === 'morning-slot'
);

// 월~금 오전 9시 슬롯이 모두 같은 groupId를 가질 수 있음
// → UI에서 "매일 오전 9시" 같은 표현 가능
```

### 4. 캐싱 전략

```typescript
// ✅ BEST PRACTICE: 상품 상세 데이터 캐싱
import { useQuery } from '@tanstack/react-query';

function useProductDetail(productId: number) {
  return useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProductDetail(productId),
    staleTime: 5 * 60 * 1000, // 5분간 신선
    cacheTime: 30 * 60 * 1000, // 30분간 캐시 유지
  });
}
```

---

## 타임슬롯 처리 베스트 프랙티스

### 1. 캘린더 날짜 선택 로직

```typescript
// ✅ BEST PRACTICE: 날짜 비활성화 종합 체크
function isDateDisabled(
  date: Date,
  productData: DetailProductResponse
): boolean {
  const dayOfWeek = getDayOfWeek(date); // 'MONDAY', 'TUESDAY', ...
  
  // 1. 판매 기간 체크
  const startDate = new Date(productData.schedule.startDate);
  const endDate = new Date(productData.schedule.endDate);
  if (date < startDate || date > endDate) {
    return true;
  }
  
  // 2. 해당 요일에 availableTimes 있는지 체크
  const hasAvailableTime = productData.availableTimes.some(
    time => time.dayOfWeek === dayOfWeek
  );
  if (!hasAvailableTime) {
    return true;
  }
  
  // 3. 업체 휴무일 체크
  const isCompanyHoliday = productData.companyHolidays.some(
    holiday => isSameDay(new Date(holiday.holidayDate), date)
  );
  if (isCompanyHoliday) {
    return true;
  }
  
  // 4. 운영 시간 체크 (선택 사항)
  const hasBusinessHours = productData.businessHours.some(
    hour => hour.dayOfWeek === dayOfWeek
  );
  if (!hasBusinessHours) {
    return true;
  }
  
  // 5. 예약 마감 시간 체크
  if (productData.reservationDeadlineHours) {
    const now = new Date();
    const deadlineDate = new Date(date);
    deadlineDate.setHours(
      deadlineDate.getHours() - productData.reservationDeadlineHours
    );
    if (now > deadlineDate) {
      return true;
    }
  }
  
  return false;
}
```

### 2. 타임슬롯 필터링

```typescript
// ✅ BEST PRACTICE: 선택된 날짜의 타임슬롯만 표시
function getAvailableTimeSlotsForDate(
  date: Date,
  productData: DetailProductResponse
): ProductAvailableTimeResponse[] {
  const dayOfWeek = getDayOfWeek(date);
  
  // 해당 요일의 타임슬롯 필터링
  const slots = productData.availableTimes.filter(
    time => time.dayOfWeek === dayOfWeek
  );
  
  // 시간순 정렬
  return slots.sort((a, b) => {
    const timeA = a.startTime.split(':').map(Number);
    const timeB = b.startTime.split(':').map(Number);
    return timeA[0] * 60 + timeA[1] - (timeB[0] * 60 + timeB[1]);
  });
}
```

### 3. 지난 시간 비활성화

```typescript
// ✅ BEST PRACTICE: 오늘 날짜는 현재 시간 이후만 선택 가능
function isTimeSlotDisabled(
  slot: ProductAvailableTimeResponse,
  selectedDate: Date
): boolean {
  const now = new Date();
  const isToday = isSameDay(selectedDate, now);
  
  if (!isToday) {
    return false; // 미래 날짜는 모든 슬롯 활성화
  }
  
  // 오늘 날짜는 현재 시간 이후 슬롯만 활성화
  const [hours, minutes] = slot.startTime.split(':').map(Number);
  const slotTime = new Date(selectedDate);
  slotTime.setHours(hours, minutes, 0, 0);
  
  return slotTime < now;
}
```

### 4. 실시간 재고 확인 (향후 구현 시)

```typescript
// 🚧 FUTURE: 실시간 재고 확인 API
async function checkSlotAvailability(
  productId: number,
  date: string,
  slotId: number
): Promise<{
  available: boolean;
  remainingCapacity: number;
  totalCapacity: number;
}> {
  // GET /api/v1/products/{productId}/availability?date=2025-01-20&slotId=1
  const response = await fetch(
    `/api/v1/products/${productId}/availability?date=${date}&slotId=${slotId}`
  );
  return response.json();
}
```

---

## 옵션 선택 베스트 프랙티스

### 1. 옵션 타입 처리

TRIT API는 두 가지 옵션 타입을 지원합니다:

```typescript
type OptionType = 'SINGLE_SELECT' | 'MULTI_SELECT';

interface ProductOption {
  id: number;
  optionName: string;
  optionType: OptionType;
  essential: boolean; // 필수 선택 여부
  optionValues: OptionValue[];
}

interface OptionValue {
  id: number;
  valueName: string;
  additionalPrice: number;
  stock?: number; // 재고 (선택 사항)
}
```

### 2. 단일 선택 옵션 (SINGLE_SELECT)

```typescript
// ✅ BEST PRACTICE: 라디오 버튼으로 구현
function SingleSelectOption({ option }: { option: ProductOption }) {
  const [selectedValue, setSelectedValue] = useState<number | null>(null);
  
  return (
    <div className="option-group">
      <h4>
        {option.optionName}
        {option.essential && <span className="required">*</span>}
      </h4>
      {option.optionValues.map(value => (
        <label key={value.id}>
          <input
            type="radio"
            name={`option-${option.id}`}
            value={value.id}
            checked={selectedValue === value.id}
            onChange={() => setSelectedValue(value.id)}
          />
          <span>{value.valueName}</span>
          {value.additionalPrice > 0 && (
            <span className="price">+{value.additionalPrice}원</span>
          )}
        </label>
      ))}
    </div>
  );
}
```

### 3. 다중 선택 옵션 (MULTI_SELECT)

```typescript
// ✅ BEST PRACTICE: 체크박스로 구현
function MultiSelectOption({ option }: { option: ProductOption }) {
  const [selectedValues, setSelectedValues] = useState<Set<number>>(
    new Set()
  );
  
  const toggleValue = (valueId: number) => {
    setSelectedValues(prev => {
      const next = new Set(prev);
      if (next.has(valueId)) {
        next.delete(valueId);
      } else {
        next.add(valueId);
      }
      return next;
    });
  };
  
  return (
    <div className="option-group">
      <h4>{option.optionName}</h4>
      {option.optionValues.map(value => (
        <label key={value.id}>
          <input
            type="checkbox"
            checked={selectedValues.has(value.id)}
            onChange={() => toggleValue(value.id)}
          />
          <span>{value.valueName}</span>
          {value.additionalPrice > 0 && (
            <span className="price">+{value.additionalPrice}원</span>
          )}
        </label>
      ))}
    </div>
  );
}
```

### 4. 옵션 검증

```typescript
// ✅ BEST PRACTICE: 필수 옵션 검증
function validateOptions(
  productOptions: ProductOption[],
  selectedOptions: Map<number, number[]> // optionId -> valueIds[]
): { valid: boolean; errors: string[] } {
  const errors: string[] = [];
  
  productOptions.forEach(option => {
    if (option.essential) {
      const selected = selectedOptions.get(option.id);
      if (!selected || selected.length === 0) {
        errors.push(`${option.optionName}은(는) 필수 선택 항목입니다.`);
      }
    }
  });
  
  return {
    valid: errors.length === 0,
    errors,
  };
}
```

### 5. 가격 계산

```typescript
// ✅ BEST PRACTICE: 옵션 추가 가격 계산
function calculateTotalPrice(
  basePrice: number,
  participants: number,
  productOptions: ProductOption[],
  selectedOptions: Map<number, number[]>
): number {
  let optionPrice = 0;
  
  selectedOptions.forEach((valueIds, optionId) => {
    const option = productOptions.find(o => o.id === optionId);
    if (!option) return;
    
    valueIds.forEach(valueId => {
      const value = option.optionValues.find(v => v.id === valueId);
      if (value) {
        optionPrice += value.additionalPrice * participants;
      }
    });
  });
  
  return basePrice * participants + optionPrice;
}
```

---

## 캘린더 구현 권장사항

### 1. UI/UX 권장사항

```typescript
// ✅ BEST PRACTICE: 캘린더 상태 관리
interface CalendarState {
  selectedDate: Date | null;
  selectedSlot: ProductAvailableTimeResponse | null;
  viewMonth: Date; // 현재 보고 있는 월
}

function BookingCalendar({ productData }: { productData: DetailProductResponse }) {
  const [state, setState] = useState<CalendarState>({
    selectedDate: null,
    selectedSlot: null,
    viewMonth: new Date(),
  });
  
  return (
    <div className="booking-calendar">
      {/* 1단계: 날짜 선택 */}
      <Calendar
        month={state.viewMonth}
        onMonthChange={month => setState(s => ({ ...s, viewMonth: month }))}
        isDateDisabled={date => isDateDisabled(date, productData)}
        selectedDate={state.selectedDate}
        onDateSelect={date => setState(s => ({ 
          ...s, 
          selectedDate: date,
          selectedSlot: null // 날짜 변경 시 슬롯 초기화
        }))}
      />
      
      {/* 2단계: 타임슬롯 선택 (날짜 선택 후 표시) */}
      {state.selectedDate && (
        <TimeSlotPicker
          date={state.selectedDate}
          slots={getAvailableTimeSlotsForDate(state.selectedDate, productData)}
          selectedSlot={state.selectedSlot}
          onSlotSelect={slot => setState(s => ({ ...s, selectedSlot: slot }))}
        />
      )}
    </div>
  );
}
```

### 2. 접근성 (Accessibility)

```typescript
// ✅ BEST PRACTICE: 키보드 네비게이션 지원
function Calendar({ ... }) {
  const handleKeyDown = (e: KeyboardEvent, date: Date) => {
    switch (e.key) {
      case 'ArrowRight':
        // 다음 날짜로 이동
        break;
      case 'ArrowLeft':
        // 이전 날짜로 이동
        break;
      case 'Enter':
      case ' ':
        // 날짜 선택
        onDateSelect(date);
        break;
    }
  };
  
  return (
    <button
      role="button"
      aria-label={`${date.toLocaleDateString('ko-KR')} 선택`}
      aria-disabled={isDisabled}
      tabIndex={isDisabled ? -1 : 0}
      onKeyDown={e => handleKeyDown(e, date)}
    >
      {date.getDate()}
    </button>
  );
}
```

### 3. 로딩 상태 처리

```typescript
// ✅ BEST PRACTICE: 로딩/에러 상태 표시
function BookingPage({ productId }: { productId: number }) {
  const { data, isLoading, error } = useProductDetail(productId);
  
  if (isLoading) {
    return <CalendarSkeleton />;
  }
  
  if (error) {
    return (
      <ErrorMessage>
        상품 정보를 불러올 수 없습니다. 다시 시도해주세요.
      </ErrorMessage>
    );
  }
  
  return <BookingCalendar productData={data} />;
}
```

---

## 성능 최적화

### 1. 메모이제이션

```typescript
// ✅ BEST PRACTICE: 비용이 큰 계산 메모이제이션
import { useMemo } from 'react';

function useGroupedAvailableTimes(availableTimes: ProductAvailableTimeResponse[]) {
  return useMemo(
    () => groupByDayOfWeek(availableTimes),
    [availableTimes]
  );
}

function useDisabledDates(productData: DetailProductResponse) {
  return useMemo(() => {
    const disabled = new Set<string>();
    const startDate = new Date(productData.schedule.startDate);
    const endDate = new Date(productData.schedule.endDate);
    
    // 판매 기간 밖의 날짜들을 미리 계산
    for (let d = new Date(startDate); d <= endDate; d.setDate(d.getDate() + 1)) {
      if (isDateDisabled(d, productData)) {
        disabled.add(d.toISOString().split('T')[0]);
      }
    }
    
    return disabled;
  }, [productData]);
}
```

### 2. Virtual Scrolling (긴 옵션 리스트)

```typescript
// ✅ BEST PRACTICE: 옵션이 많을 경우 가상 스크롤
import { FixedSizeList } from 'react-window';

function LongOptionList({ optionValues }: { optionValues: OptionValue[] }) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      <OptionValueItem value={optionValues[index]} />
    </div>
  );
  
  return (
    <FixedSizeList
      height={400}
      itemCount={optionValues.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### 3. Debounce (실시간 검색)

```typescript
// ✅ BEST PRACTICE: 옵션 검색 시 debounce
import { useDebouncedValue } from '@/hooks/useDebouncedValue';

function SearchableOptions({ options }: { options: ProductOption[] }) {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearch = useDebouncedValue(searchTerm, 300);
  
  const filteredOptions = useMemo(() => {
    if (!debouncedSearch) return options;
    
    return options.filter(option =>
      option.optionName.toLowerCase().includes(debouncedSearch.toLowerCase())
    );
  }, [options, debouncedSearch]);
  
  return (
    <>
      <input
        type="text"
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
        placeholder="옵션 검색..."
      />
      <OptionList options={filteredOptions} />
    </>
  );
}
```

---

## 에러 핸들링

### 1. API 에러 처리

```typescript
// ✅ BEST PRACTICE: 일관된 에러 처리
interface ApiError {
  status: number;
  errorCode: string;
  message: string;
}

async function fetchProductDetail(productId: number): Promise<DetailProductResponse> {
  try {
    const response = await fetch(`/api/v1/products/${productId}`);
    
    if (!response.ok) {
      const error: ApiError = await response.json();
      
      switch (error.errorCode) {
        case 'PRODUCT_NOT_FOUND':
          throw new Error('상품을 찾을 수 없습니다.');
        case 'PRODUCT_INACTIVE':
          throw new Error('현재 판매 중이지 않은 상품입니다.');
        default:
          throw new Error(error.message || '알 수 없는 오류가 발생했습니다.');
      }
    }
    
    const result = await response.json();
    return result.data;
  } catch (error) {
    console.error('Failed to fetch product:', error);
    throw error;
  }
}
```

### 2. 유효성 검증 에러

```typescript
// ✅ BEST PRACTICE: 사용자 친화적인 에러 메시지
interface ValidationError {
  field: string;
  message: string;
}

function validateBookingForm(formData: BookingFormData): ValidationError[] {
  const errors: ValidationError[] = [];
  
  if (!formData.selectedDate) {
    errors.push({
      field: 'date',
      message: '날짜를 선택해주세요.',
    });
  }
  
  if (!formData.selectedSlot) {
    errors.push({
      field: 'slot',
      message: '시간대를 선택해주세요.',
    });
  }
  
  if (formData.participants < 1) {
    errors.push({
      field: 'participants',
      message: '참여 인원을 입력해주세요.',
    });
  }
  
  // 필수 옵션 검증
  const optionValidation = validateOptions(
    formData.productOptions,
    formData.selectedOptions
  );
  
  if (!optionValidation.valid) {
    optionValidation.errors.forEach(msg => {
      errors.push({ field: 'options', message: msg });
    });
  }
  
  return errors;
}
```

### 3. 재시도 로직

```typescript
// ✅ BEST PRACTICE: 네트워크 에러 시 자동 재시도
async function fetchWithRetry<T>(
  fetchFn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fetchFn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // 네트워크 에러인 경우만 재시도
      if (error instanceof TypeError && error.message.includes('fetch')) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
        continue;
      }
      
      throw error;
    }
  }
  
  throw new Error('Max retries reached');
}

// 사용 예시
const productData = await fetchWithRetry(() => fetchProductDetail(productId));
```

---

## 사용자 경험 개선

### 1. 스켈레톤 UI

```typescript
// ✅ BEST PRACTICE: 로딩 시 스켈레톤 표시
function CalendarSkeleton() {
  return (
    <div className="calendar-skeleton">
      {/* 캘린더 헤더 */}
      <div className="skeleton-header" />
      
      {/* 요일 */}
      <div className="skeleton-weekdays">
        {[...Array(7)].map((_, i) => (
          <div key={i} className="skeleton-box" />
        ))}
      </div>
      
      {/* 날짜 */}
      <div className="skeleton-dates">
        {[...Array(35)].map((_, i) => (
          <div key={i} className="skeleton-box" />
        ))}
      </div>
    </div>
  );
}
```

### 2. 낙관적 업데이트

```typescript
// ✅ BEST PRACTICE: 즉각적인 피드백 제공
function useOptimisticBooking() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createBooking,
    onMutate: async (newBooking) => {
      // 이전 데이터 백업
      await queryClient.cancelQueries({ queryKey: ['bookings'] });
      const previousBookings = queryClient.getQueryData(['bookings']);
      
      // 낙관적 업데이트
      queryClient.setQueryData(['bookings'], (old: any) => [
        ...old,
        { ...newBooking, status: 'PENDING' },
      ]);
      
      return { previousBookings };
    },
    onError: (err, newBooking, context) => {
      // 에러 시 롤백
      queryClient.setQueryData(['bookings'], context?.previousBookings);
    },
    onSettled: () => {
      // 완료 후 실제 데이터 다시 가져오기
      queryClient.invalidateQueries({ queryKey: ['bookings'] });
    },
  });
}
```

### 3. 툴팁 및 도움말

```typescript
// ✅ BEST PRACTICE: 컨텍스트 도움말 제공
function TimeSlotItem({ slot }: { slot: ProductAvailableTimeResponse }) {
  return (
    <div className="time-slot">
      <span>{slot.startTime} - {slot.endTime}</span>
      
      {/* 인원 제한 정보 */}
      <Tooltip content={`최소 ${slot.minimumPeople}명 ~ 최대 ${slot.maximumPeople}명`}>
        <InfoIcon />
      </Tooltip>
      
      {/* 남은 좌석 표시 (향후) */}
      {slot.remainingCapacity && (
        <span className="capacity-badge">
          {slot.remainingCapacity}석 남음
        </span>
      )}
    </div>
  );
}
```

### 4. 진행 단계 표시

```typescript
// ✅ BEST PRACTICE: 예약 진행 단계 시각화
function BookingWizard() {
  const [step, setStep] = useState(1);
  
  const steps = [
    { id: 1, label: '날짜 선택', component: DatePicker },
    { id: 2, label: '시간 선택', component: TimeSlotPicker },
    { id: 3, label: '옵션 선택', component: OptionSelector },
    { id: 4, label: '정보 입력', component: BookingForm },
    { id: 5, label: '결제', component: PaymentPage },
  ];
  
  return (
    <div className="booking-wizard">
      {/* 진행 표시줄 */}
      <ProgressBar current={step} total={steps.length} />
      
      {/* 현재 단계 */}
      {steps.map(s => (
        <div key={s.id} style={{ display: step === s.id ? 'block' : 'none' }}>
          <s.component onNext={() => setStep(step + 1)} />
        </div>
      ))}
      
      {/* 네비게이션 */}
      <div className="wizard-nav">
        {step > 1 && (
          <button onClick={() => setStep(step - 1)}>이전</button>
        )}
        {step < steps.length && (
          <button onClick={() => setStep(step + 1)}>다음</button>
        )}
      </div>
    </div>
  );
}
```

---

## 추가 권장사항

### 1. 타임존 처리

```typescript
// ✅ BEST PRACTICE: 한국 시간대 명시적 처리
import { format, toZonedTime } from 'date-fns-tz';

const KST = 'Asia/Seoul';

function formatKoreanDateTime(date: Date | string): string {
  const d = typeof date === 'string' ? new Date(date) : date;
  const zonedDate = toZonedTime(d, KST);
  return format(zonedDate, 'yyyy-MM-dd HH:mm', { timeZone: KST });
}
```

### 2. 모바일 최적화

```typescript
// ✅ BEST PRACTICE: 모바일 친화적 달력
import { isMobile } from 'react-device-detect';

function ResponsiveCalendar() {
  if (isMobile) {
    return <MobileDatePicker />; // Native date picker 사용
  }
  
  return <DesktopCalendar />; // 커스텀 캘린더
}
```

### 3. 분석 및 추적

```typescript
// ✅ BEST PRACTICE: 사용자 행동 추적
import { trackEvent } from '@/analytics';

function BookingCalendar() {
  const handleDateSelect = (date: Date) => {
    trackEvent('booking_date_selected', {
      productId,
      selectedDate: date.toISOString(),
    });
    
    // ... 날짜 선택 로직
  };
  
  const handleSlotSelect = (slot: ProductAvailableTimeResponse) => {
    trackEvent('booking_slot_selected', {
      productId,
      slotId: slot.id,
      startTime: slot.startTime,
    });
    
    // ... 슬롯 선택 로직
  };
}
```

---

## 요약

### 핵심 포인트

1. **AvailableTimes**: 요일별로 그룹핑하여 캐싱, groupId 활용
2. **타임슬롯**: 날짜 선택 → 해당 요일의 슬롯 필터링 → 시간순 정렬
3. **옵션 선택**: SINGLE_SELECT는 라디오, MULTI_SELECT는 체크박스
4. **캘린더**: 다중 조건 체크 (판매 기간, availableTimes, 휴무일, 운영시간)
5. **성능**: 메모이제이션, 가상 스크롤, debounce 활용
6. **UX**: 스켈레톤 UI, 진행 단계 표시, 에러 메시지 명확히

### 체크리스트

- [ ] availableTimes를 요일별로 그룹핑
- [ ] 캘린더 비활성화 로직 구현 (5가지 조건)
- [ ] 타임슬롯 시간순 정렬
- [ ] 필수 옵션 검증
- [ ] 총 가격 계산 (옵션 포함)
- [ ] 로딩/에러 상태 처리
- [ ] 모바일 최적화
- [ ] 접근성 지원 (키보드 네비게이션)

---

**문서 작성자**: TRIT API Team  
**문의**: backend-team@trit.today

**다음 단계**: 실제 구현 후 피드백 주시면 문서를 업데이트하겠습니다.
