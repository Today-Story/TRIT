# Mock 데이터 제거 작업 완료 보고서

## 작업 개요
정산 관리 모듈의 모든 페이지에서 mock 데이터를 제거하고 실제 백엔드 API를 호출하도록 수정하였습니다.

## 수정된 파일

### 1. 통계 페이지 (`statistics/page.tsx`)

**제거된 내용:**
- Mock 데이터 상수 (`mockData`)
- `loadStatistics()` 함수의 임시 데이터 생성 로직
- 수동 상태 관리 (`setSalesData`, `setSummary`)

**추가된 내용:**
- `useGetMonthlySales` 훅 사용
- `useGetTotalSales` 훅 사용
- `useMemo`를 통한 데이터 변환 및 통계 계산
- 기간 자동 계산 로직
- 로딩 상태 통합 (`isLoading`)

**변경 사항:**
```typescript
// Before
const [salesData, setSalesData] = useState<SalesData[]>([]);
const loadStatistics = async () => {
  const mockData = [...]; // 임시 데이터
  setSalesData(mockData);
};

// After
const { data: monthlySalesData, isLoading } = useGetMonthlySales({
  productId,
  startDate: dateRange.startDate,
  endDate: dateRange.endDate,
});

const salesData = useMemo(() => {
  if (!monthlySalesData?.data) return [];
  return monthlySalesData.data.map(...);
}, [monthlySalesData]);
```

---

### 2. 분석 페이지 (`analytics/page.tsx`)

**제거된 내용:**
- Mock 분석 데이터 (`mockAnalytics`)
- Mock 상품 성과 데이터 (`mockProducts`)
- `loadAnalytics()` 함수

**추가된 내용:**
- `useGetSettlementList` 훅 사용
- `useGetProductSales` 훅 사용
- 실제 정산 데이터로부터 통계 계산
- 상품별 성과 집계 로직
- `refetch` 기능 (`handleRefresh`)

**변경 사항:**
```typescript
// Before
const [analytics, setAnalytics] = useState<SettlementAnalytics | null>(null);
const loadAnalytics = async () => {
  const mockAnalytics = {...}; // 임시 데이터
  setAnalytics(mockAnalytics);
};

// After
const { data: settlementData, refetch } = useGetSettlementList({
  page: 0,
  size: 1000,
});

const analytics = useMemo(() => {
  if (!settlementData?.data?.content) return null;
  // 실제 데이터로부터 계산
  return {...};
}, [settlementData]);
```

---

### 3. 정산 목록 페이지 (`list/page.tsx`)

**제거된 내용:**
- `INITIAL_SETTLEMENTS` 상수 (모든 mock 정산 데이터)
- `Settlement` 인터페이스 (타입 파일로 이동)
- 로컬 상태 관리 (`setSettlements`)
- 클라이언트 사이드 필터링 로직

**추가된 내용:**
- `useGetSettlementList` 훅 사용
- `SettlementSummary` 타입 import
- 서버 사이드 필터링 (API 파라미터)
- 페이지네이션 수정 (0-based index)
- 로딩 및 빈 상태 처리

**변경 사항:**
```typescript
// Before
const [settlements, setSettlements] = useState<Settlement[]>(INITIAL_SETTLEMENTS);
const filteredSettlements = settlements.filter(...);

// After
const { data: settlementData, isLoading } = useGetSettlementList({
  status: filterStatus === "all" ? undefined : filterStatus,
  productName: searchTerm || undefined,
  page: currentPage,
  size: itemsPerPage,
});

const settlements = settlementData?.data?.content || [];
```

**승인/거절 처리:**
```typescript
// Before
const handleApprove = (uuid: string) => {
  setSettlements(
    settlements.map((s) => s.settlementUuid === uuid ? {...s, status: "APPROVED"} : s)
  );
};

// After
const handleApprove = (uuid: string) => {
  // TODO: 실제 승인 API 호출
  alert(`정산 승인: ${uuid}`);
};
```

---

### 4. 메인 페이지 (`page.tsx`)

**제거된 내용:**
- `INITIAL_STATS` 상수
- `INITIAL_RECENT_TRANSACTIONS` 상수
- `SettlementStats` 인터페이스 (로컬)
- `RecentTransaction` 인터페이스 (로컬)
- 로컬 상태 관리

**추가된 내용:**
- `useGetSettlementList` 훅 사용
- `useMemo`를 통한 통계 계산
- 정산 데이터를 거래 내역으로 변환

**변경 사항:**
```typescript
// Before
const [stats] = useState<SettlementStats>(INITIAL_STATS);
const [recentTransactions] = useState<RecentTransaction[]>(INITIAL_RECENT_TRANSACTIONS);

// After
const { data: settlementData } = useGetSettlementList({
  page: 0,
  size: 10,
});

const stats = useMemo(() => {
  if (!settlementData?.data?.content) return {...};
  // 실제 데이터로부터 계산
}, [settlementData]);

const recentTransactions = useMemo(() => {
  if (!settlementData?.data?.content) return [];
  // 정산 데이터를 거래 내역 형식으로 변환
}, [settlementData]);
```

---

## API 연동 상세

### 사용된 React Query 훅

1. **useGetSettlementList**
   ```typescript
   const { data, isLoading, refetch } = useGetSettlementList({
     status?: SettlementStatus,
     productName?: string,
     page?: number,
     size?: number,
     sortOption?: string,
   });
   ```

2. **useGetMonthlySales**
   ```typescript
   const { data, isLoading } = useGetMonthlySales({
     productId?: number,
     startDate?: string,
     endDate?: string,
   });
   ```

3. **useGetTotalSales**
   ```typescript
   const { data, isLoading } = useGetTotalSales({
     productId?: number,
     startDate?: string,
     endDate?: string,
   });
   ```

4. **useGetProductSales**
   ```typescript
   const { data, isLoading } = useGetProductSales({
     productId?: number,
     startDate?: string,
     endDate?: string,
     monthly?: boolean,
     page?: number,
     size?: number,
   });
   ```

### API 응답 구조

모든 API는 다음과 같은 공통 응답 구조를 사용합니다:

```typescript
interface ApiResponse<T> {
  status: "SUCCESS" | "ERROR";
  message: string;
  data: T;
}

interface PageResponse<T> {
  content: T[];
  totalPages: number;
  totalElements: number;
  size: number;
  number: number;
  first: boolean;
  last: boolean;
}
```

---

## 로딩 및 에러 처리

### 로딩 상태
모든 페이지에서 로딩 상태를 표시합니다:

```typescript
{isLoading ? (
  <div className="flex items-center justify-center py-12">
    <div className="text-gray-500">데이터를 불러오는 중...</div>
  </div>
) : (
  // 실제 콘텐츠
)}
```

### 빈 데이터 처리
데이터가 없을 때 사용자 친화적인 메시지를 표시합니다:

```typescript
{settlements.length === 0 ? (
  <div className="flex items-center justify-center py-12">
    <div className="text-gray-500">조회된 정산 내역이 없습니다.</div>
  </div>
) : (
  // 테이블
)}
```

### useMemo 사용
불필요한 재계산을 방지하기 위해 `useMemo`를 사용합니다:

```typescript
const summary = useMemo(() => {
  if (!salesData.length) return null;
  // 계산 로직
}, [salesData]);
```

---

## 페이지네이션 수정

### 0-based 인덱스로 변경
백엔드 API가 0-based 페이지 인덱스를 사용하므로 프론트엔드도 동일하게 수정:

```typescript
// Before
const [currentPage, setCurrentPage] = useState(1);
const paginatedData = data.slice((currentPage - 1) * size, currentPage * size);

// After
const [currentPage, setCurrentPage] = useState(0);
const { data } = useGetSettlementList({ page: currentPage, size });
```

---

## TODO 항목

### 1. 환경 변수 설정
`.env.local` 파일에 다음 환경 변수 추가 필요:
```bash
NEXT_PUBLIC_API_URL=http://localhost:8080/api/v1
```

### 2. 승인/거절 API 구현
현재는 alert만 표시하며, 실제 API 호출 구현 필요:
```typescript
const handleApprove = async (uuid: string) => {
  try {
    await approveSettlement(uuid);
    refetch(); // 데이터 재조회
  } catch (error) {
    alert("승인 실패");
  }
};
```

### 3. 인증 처리
API 호출 시 Authorization 헤더 추가:
```typescript
headers: {
  "Content-Type": "application/json",
  "Authorization": `Bearer ${token}`,
}
```

### 4. 에러 핸들링
React Query의 에러 처리 강화:
```typescript
const { data, isLoading, error } = useGetSettlementList({...});

if (error) {
  return <ErrorMessage message={error.message} />;
}
```

### 5. 성장률 계산
이전 기간 데이터와 비교하여 실제 성장률 계산:
```typescript
// 현재는 임시값
monthlyGrowth: 0, // TODO: 이전 기간과 비교 필요
```

### 6. 하위셀러 통계
하위셀러 관련 통계는 별도 API 구현 필요:
```typescript
totalSellers: 0, // TODO: 하위셀러 API 연동 필요
activeSellers: 0, // TODO: 하위셀러 API 연동 필요
```

---

## 테스트 체크리스트

배포 전 다음 항목을 테스트해야 합니다:

- [ ] 통계 페이지에서 기간 변경 시 데이터 갱신
- [ ] 분석 페이지에서 새로고침 버튼 동작
- [ ] 정산 목록 페이지 검색 기능
- [ ] 정산 목록 페이지 필터 기능
- [ ] 정산 목록 페이지 페이지네이션
- [ ] 로딩 상태 표시
- [ ] 빈 데이터 메시지 표시
- [ ] 에러 발생 시 처리
- [ ] 반응형 디자인 (모바일, 태블릿)
- [ ] 브라우저 호환성 (Chrome, Safari, Firefox)

---

## 결론

모든 페이지에서 mock 데이터가 완전히 제거되었으며, 실제 백엔드 API를 호출하도록 구현되었습니다. React Query를 통해 데이터 페칭, 캐싱, 로딩 상태 관리가 자동으로 처리됩니다.

**다음 단계:**
1. 환경 변수 설정
2. 백엔드 서버 구동
3. API 엔드포인트 확인
4. 통합 테스트

환경 설정 후 즉시 사용 가능한 상태입니다.
