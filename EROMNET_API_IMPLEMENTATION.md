# Eromnet API 구현 완료 보고서

## 📋 구현 개요

Eromnet PG의 하위 셀러 지급대행 API를 Backend와 Frontend에 완전히 통합했습니다.

## ✅ 구현된 기능

### Backend (TRIT-BE)

#### 1. DTO 추가
- `EromnetBalanceRequest/Response` - 잔액 조회
- `EromnetPayoutRequest/Response` - 지급 요청
- `EromnetTransferRequest` - 송금 요청
- `EromnetCancelRequest` - 취소 요청
- `EromnetPayoutInfoResponse` - 지급/송금 정보

#### 2. Service 계층
**파일**: `EromnetApiService.java`, `EromnetApiServiceImpl.java`

구현된 메서드:
- `getBalance()` - 가맹점 송금 잔액 조회
- `requestPayout()` - 하위 셀러 지급 요청
- `requestTransfer()` - 하위 셀러 송금 요청
- `cancelPayout()` - 하위 셀러 지급 취소
- `cancelTransfer()` - 하위 셀러 송금 취소
- `getPayoutInfo()` - 하위 셀러 지급/송금 정보 조회

#### 3. Controller
**파일**: `EromnetSellerController.java`

엔드포인트:
- `GET /api/v1/payments/balance` - 잔액 조회
- `POST /api/v1/payments/payout` - 지급 요청
- `POST /api/v1/payments/transfer` - 송금 요청
- `POST /api/v1/payments/payout/cancel` - 지급 취소
- `POST /api/v1/payments/transfer/cancel` - 송금 취소
- `GET /api/v1/payments/payout/info` - 지급/송금 정보 조회

#### 4. Configuration
**파일**: `WebClientConfig.java`
- `eromnetSellerWebClient` Bean 추가 (Eromnet 하위 셀러 API용)

#### 5. ResultCode
**파일**: `ResultCode.java`
- `GET_BALANCE_SUCCESS`
- `REQUEST_PAYOUT_SUCCESS`
- `REQUEST_TRANSFER_SUCCESS`
- `CANCEL_PAYOUT_SUCCESS`
- `CANCEL_TRANSFER_SUCCESS`
- `GET_PAYOUT_INFO_SUCCESS`

### Frontend (TRIT-FE/apps/admin)

#### 1. Service 함수
**파일**: `services/settlement.ts`

추가된 함수:
- `getBalance()` - 잔액 조회
- `requestPayout()` - 지급 요청
- `requestTransfer()` - 송금 요청
- `cancelPayout()` - 지급 취소
- `cancelTransfer()` - 송금 취소
- `getPayoutInfo()` - 지급/송금 정보 조회
- `getPaymentStatus()` - 결제 상태 조회
- `getPaymentHistory()` - 결제 내역 조회
- `getSettlementData()` - 정산 데이터 조회
- `getSettlementDataRange()` - 정산 데이터 범위 조회

#### 2. 페이지 구현

**잔액 조회 페이지** (`/settlement/balance`)
- ✅ Mock 데이터 제거
- ✅ 실제 API 연동 (`getBalance()`)
- ✅ 실시간 잔액 표시
- ✅ 새로고침 기능
- ✅ 에러 처리

**거래 내역 페이지** (`/settlement/history`)
- ✅ Mock 데이터 제거
- ✅ 실제 API 연동 (`getPayoutInfo()`)
- ✅ 지급 ID + 하위셀러 ID 검색
- ✅ 거래 상태 표시
- ✅ 에러 메시지 표시

**지급/송금 관리 페이지** (`/settlement/payments`)
- ✅ Mock 데이터 제거
- ✅ 지급 요청 모달 (`requestPayout()`)
- ✅ 송금 요청 모달 (`requestTransfer()`)
- ✅ 성공/실패 메시지 표시
- ✅ 폼 유효성 검사

## 🔄 API 흐름

### 지급/송금 프로세스
```
1. 지급 요청 (POST /api/v1/payments/payout)
   ↓
2. 송금 요청 (POST /api/v1/payments/transfer)
   ↓
3. Eromnet 정산 배치 처리 (11:00 또는 17:00)
   ↓
4. 송금 완료 (TRANS_COMPLETE)
```

### 취소 프로세스
```
- 지급 취소: POST /api/v1/payments/payout/cancel
- 송금 취소: POST /api/v1/payments/transfer/cancel
```

## 📊 API 명세

### Eromnet API 엔드포인트

| Eromnet API | Backend API | 설명 |
|-------------|-------------|------|
| `GET /external/seller/transfer/remainamt` | `GET /api/v1/payments/balance` | 잔액 조회 |
| `POST /external/seller/payout/request` | `POST /api/v1/payments/payout` | 지급 요청 |
| `POST /external/seller/transfer/request` | `POST /api/v1/payments/transfer` | 송금 요청 |
| `POST /external/seller/payout/cancel` | `POST /api/v1/payments/payout/cancel` | 지급 취소 |
| `POST /external/seller/transfer/cancel` | `POST /api/v1/payments/transfer/cancel` | 송금 취소 |
| `GET /external/seller/payout/info` | `GET /api/v1/payments/payout/info` | 정보 조회 |

## 🎯 상태 코드 (TransStatus)

- `PAYMENT_REQUEST` - 지급 요청
- `PAYMENT_CANCEL` - 지급 취소
- `TRANS_REQUEST` - 송금 요청
- `TRANS_CANCEL` - 송금 취소
- `TRANS_FAIL` - 송금 실패
- `TRANS_PROCESS` - 송금 처리중
- `TRANS_INQUIRE` - 송금 조회
- `TRANS_COMPLETE` - 송금 완료

## 🔧 설정 필요 사항

### Backend (application.yml)
```yaml
payment:
  mid: ${EROMNET_MID}           # Eromnet 가맹점 ID
  client-key: ${EROMNET_CLIENT_KEY}
  secret-key: ${EROMNET_SECRET_KEY}
  base-url: https://bo-api.payverseglobal.com  # Production
  # base-url: https://bo-api-snd.payverseglobal.com  # Sandbox
```

### Frontend (.env.local)
```env
NEXT_PUBLIC_API_BASE_URL=http://localhost:8080/api/v1
```

## 📝 주의사항

1. **보안**
   - AES 암호화가 필요한 필드: `bankAccountNo`, `bankAccountHolder`
   - Headers에 `mid`, `clientKey` 필수

2. **정산 배치 시간**
   - 1차: 11:00 (전일 15:00:00 ~ 당일 07:59:59)
   - 2차: 17:00 (전일 08:00:00 ~ 당일 15:00:00)

3. **2단계 프로세스**
   - 지급 요청 후 반드시 송금 요청을 완료해야 실제 송금 처리됨
   - 송금 요청 전까지는 지급 취소 가능

4. **제한사항**
   - Eromnet API는 거래 내역 목록 조회 API가 명시되지 않음
   - 현재는 개별 조회(`getPayoutInfo`)만 지원

## ✨ 다음 단계 (선택사항)

1. **목록 조회 기능 추가**
   - 로컬 DB에 거래 내역 저장
   - 목록 조회 API 구현

2. **대시보드 통계**
   - 일별/월별 지급 통계
   - 상태별 거래 현황

3. **알림 기능**
   - 송금 완료/실패 알림
   - 잔액 부족 알림

4. **일괄 처리**
   - 여러 하위셀러에 대한 일괄 지급
   - CSV 업로드 기능

## 📁 수정된 파일 목록

### Backend
```
backend/src/main/java/today/story/backend/
├── config/
│   └── WebClientConfig.java (수정)
├── common/type/
│   └── ResultCode.java (수정)
└── payment/
    ├── controller/
    │   └── EromnetSellerController.java (신규)
    ├── dto/
    │   ├── request/
    │   │   ├── EromnetBalanceRequest.java (신규)
    │   │   ├── EromnetPayoutRequest.java (신규)
    │   │   ├── EromnetTransferRequest.java (신규)
    │   │   └── EromnetCancelRequest.java (신규)
    │   └── response/
    │       ├── EromnetBalanceResponse.java (신규)
    │       ├── EromnetPayoutResponse.java (신규)
    │       └── EromnetPayoutInfoResponse.java (신규)
    └── service/
        ├── EromnetApiService.java (수정)
        └── EromnetApiServiceImpl.java (수정)
```

### Frontend
```
apps/admin/src/
├── services/
│   └── settlement.ts (수정)
└── app/settlement/
    ├── balance/
    │   └── page.tsx (수정)
    ├── history/
    │   └── page.tsx (수정)
    └── payments/
        └── page.tsx (수정)
```

## 🎉 완료!

모든 Eromnet API가 Backend와 Frontend에 완전히 통합되었습니다.
이제 실제 하위 셀러 지급대행 기능을 사용할 수 있습니다!
