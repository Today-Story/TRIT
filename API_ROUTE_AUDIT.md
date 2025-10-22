# TRIT Frontend-Backend API Route Audit

**작업일**: 2025-01-22  
**목적**: Frontend settlement.ts의 모든 API 호출 경로를 Backend controller와 대조 검증

---

## 📊 API 라우팅 검증 결과

### 1. 하위셀러 관리 API

| Frontend 함수 | Frontend 경로 | Backend 컨트롤러 | Backend 경로 | 상태 |
|--------------|--------------|----------------|-------------|------|
| `registerSeller()` | `/payments/seller/register` | `EromnetSellerController` | `/api/v1/payments/seller/register` | ✅ 정확 |
| `getSellers()` | `/sub-sellers` | `SubSellerController` | `/api/v1/sub-sellers` | ✅ 정확 |
| `getSellerById()` | 클라이언트 필터링 | - | - | ⚠️ Backend 미구현 |
| `updateSeller()` | `/payments/seller/modify` | `EromnetSellerController` | `/api/v1/payments/seller/modify` | ✅ 정확 |
| `getSellerInfo()` | `/payments/seller/info` | `EromnetSellerController` | `/api/v1/payments/seller/info` | ✅ 정확 |

### 2. 정산 API

| Frontend 함수 | Frontend 경로 | Backend 컨트롤러 | Backend 경로 | 상태 |
|--------------|--------------|----------------|-------------|------|
| `getSettlementList()` | `/settlements` | `SettlementController` | `/api/v1/settlements` | ✅ 정확 |
| `getProductSales()` | `/settlements/sales` | `SettlementController` | `/api/v1/settlements/sales` | ✅ 정확 |
| `getTotalSales()` | `/settlements/sales/total` | `SettlementController` | `/api/v1/settlements/sales/total` | ✅ 정확 |
| `getMonthlySales()` | `/settlements/sales/monthly` | `SettlementController` | `/api/v1/settlements/sales/monthly` | ✅ 정확 |

### 3. Eromnet PG 지급대행 API

| Frontend 함수 | Frontend 경로 | Backend 컨트롤러 | Backend 경로 | 상태 |
|--------------|--------------|----------------|-------------|------|
| `getBalance()` | `/payments/balance` | `EromnetSellerController` | `/api/v1/payments/balance` | ✅ 정확 |
| `requestPayout()` | `/payments/payout` | `EromnetSellerController` | `/api/v1/payments/payout` | ✅ 정확 |
| `requestTransfer()` | `/payments/transfer` | `EromnetSellerController` | `/api/v1/payments/transfer` | ✅ 정확 |
| `cancelPayout()` | `/payments/payout/cancel` | `EromnetSellerController` | `/api/v1/payments/payout/cancel` | ✅ 정확 |
| `cancelTransfer()` | `/payments/transfer/cancel` | `EromnetSellerController` | `/api/v1/payments/transfer/cancel` | ✅ 정확 |
| `getPayoutInfo()` | `/payments/payout/info` | `EromnetSellerController` | `/api/v1/payments/payout/info` | ✅ 정확 |

### 4. Eromnet PG 결제/정산 조회 API (Backend TODO)

| Frontend 함수 | Frontend 경로 | Backend 컨트롤러 | Backend 경로 | 상태 |
|--------------|--------------|----------------|-------------|------|
| `getPaymentStatus()` | `/payments/status` | `PaymentController` | `/api/v1/payments/status` | ⚠️ Backend TODO |
| `getPaymentHistory()` | `/payments/history` | `PaymentController` | `/api/v1/payments/history` | ⚠️ Backend TODO |
| `getSettlementData()` | `/payments/settlement` | `PaymentController` | `/api/v1/payments/settlement` | ⚠️ Backend TODO |
| `getSettlementDataRange()` | `/payments/settlement/range` | `PaymentController` | `/api/v1/payments/settlement/range` | ⚠️ Backend TODO |

---

## 🔍 상세 분석

### ✅ 올바르게 구현된 API

모든 API 경로가 Backend와 정확하게 일치합니다. 

**참고사항:**
- Frontend의 `/sub-sellers` 호출은 **정확합니다**
- `SubSellerController`는 로컬 DB에서 하위셀러 목록을 조회합니다
- `EromnetSellerController`는 Eromnet PG API와 직접 통신합니다

### 🏗️ 아키텍처 설계

하위셀러 관리는 **하이브리드 아키텍처**를 사용합니다:

1. **등록/수정** → Eromnet PG API (`EromnetSellerController`)
   - Eromnet PG에 하위셀러 등록/수정 요청
   - 결과를 로컬 DB에도 저장 (별도 로직 필요)

2. **목록 조회** → 로컬 DB (`SubSellerController`)
   - 우리 DB에 저장된 하위셀러 목록 조회
   - 페이지네이션 지원

3. **상세 조회** → Eromnet PG API (`EromnetSellerController.getSellerInfo()`)
   - Eromnet PG에서 최신 정보 조회
   - 실시간 상태 확인

### ⚠️ 주의사항

#### 1. getSellerById() - 클라이언트 필터링 방식

현재 Frontend는 전체 목록을 가져온 후 클라이언트에서 필터링:

```typescript
const sellers = await getSellers();
const seller = sellers.data.content.find((s) => s.id === id);
```

**권장사항:**
- Backend에 `GET /api/v1/sub-sellers/{id}` 엔드포인트 추가
- 또는 Eromnet API의 `getSellerInfo(sellerId)` 사용

#### 2. Backend TODO 항목

다음 엔드포인트들은 Backend에서 아직 구현되지 않았습니다 (PaymentController.java):

```java
@GetMapping("/status")              // TODO: Eromnet PG 결제 상태 조회
@GetMapping("/history")             // TODO: Eromnet PG 결제 내역 조회
@GetMapping("/settlement")          // TODO: Eromnet PG 정산 데이터 조회
@GetMapping("/settlement/range")    // TODO: Eromnet PG 정산 데이터 범위 조회
```

Frontend에서는 이미 구현되어 있으므로, Backend 구현 완료 시 즉시 사용 가능합니다.

---

## 📝 Backend Controller 매핑

### EromnetSellerController (`/api/v1/payments`)

```java
POST   /seller/register          ✅ registerSeller()
POST   /seller/modify            ✅ updateSeller()
GET    /seller/info              ✅ getSellerInfo()
GET    /balance                  ✅ getBalance()
POST   /payout                   ✅ requestPayout()
POST   /payout/modify            ❌ Frontend 미사용
POST   /payout/cancel            ✅ cancelPayout()
POST   /transfer                 ✅ requestTransfer()
POST   /transfer/cancel          ✅ cancelTransfer()
GET    /payout/info              ✅ getPayoutInfo()
```

### SubSellerController (`/api/v1/sub-sellers`)

```java
POST   /                         ❌ Frontend 미사용 (대신 Eromnet API 사용)
GET    /                         ✅ getSellers()
```

### SettlementController (`/api/v1/settlements`)

```java
GET    /                         ✅ getSettlementList()
GET    /sales                    ✅ getProductSales()
GET    /sales/total              ✅ getTotalSales()
GET    /sales/monthly            ✅ getMonthlySales()
GET    /export                   ❌ Frontend 미사용
GET    /export/statistics        ❌ Frontend 미사용
```

### PaymentController (`/api/v1/payments`)

```java
GET    /status                   ⚠️ TODO (Frontend 준비됨)
GET    /history                  ⚠️ TODO (Frontend 준비됨)
GET    /settlement               ⚠️ TODO (Frontend 준비됨)
GET    /settlement/range         ⚠️ TODO (Frontend 준비됨)
```

---

## 🎯 결론

### Frontend API 라우팅 상태: ✅ 정확

모든 Frontend API 호출 경로가 Backend 컨트롤러와 **정확하게 일치**합니다.

- `/sub-sellers` 사용은 **올바른 구현**입니다
- `SubSellerController`가 로컬 DB 목록 조회를 담당합니다
- Eromnet PG API 경로도 모두 정확합니다

### 개선 권장사항

1. **Backend 추가 필요**:
   - `GET /api/v1/sub-sellers/{id}` - 단일 셀러 조회
   - PaymentController의 TODO 항목 구현

2. **데이터 동기화 전략**:
   - Eromnet PG에 등록 시 로컬 DB에도 저장
   - 또는 주기적 동기화 배치 작업

3. **Frontend 최적화**:
   - `getSellerById()` - Backend API 구현 후 직접 호출로 변경

---

**작성일**: 2025-01-22  
**검증 완료**: 모든 API 경로 Backend와 일치 확인
