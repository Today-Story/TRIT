# 정산 관리 API 연동 완료 및 다음 단계 가이드

## 📊 작업 완료 내역

### ✅ Phase 1: 타입 정의 완료
**파일:** `apps/admin/src/types/settlement.ts`

추가된 타입들:
- `SellerModifyRequest/Response` - 하위 셀러 정보 변경
- `BalanceResponse` - 가맹점 송금 잔액 조회
- `PayoutRequest/Response` - 하위 셀러 지급 요청
- `PayoutCancelRequest/Response` - 하위 셀러 지급 취소
- `TransferRequest/Response` - 하위 셀러 송금 요청
- `TransferCancelRequest/Response` - 하위 셀러 송금 취소
- `PayoutInfoRequest/Response` - 하위 셀러 지급/송금 정보 조회
- `TransactionStatus` - 지급 상태 타입 (8가지 상태)

### ✅ Phase 2: API 설정 확장
**파일:** `apps/admin/src/models/settlement.ts`

`API_CONFIG`에 추가된 엔드포인트:
- `balanceUrl`: `/external/seller/balance`
- `payoutRequestUrl`: `/external/seller/payout/request`
- `payoutCancelUrl`: `/external/seller/payout/cancel`
- `transferRequestUrl`: `/external/seller/transfer/request`
- `transferCancelUrl`: `/external/seller/transfer/cancel`
- `payoutInfoUrl`: `/external/seller/payout/info`

### ✅ Phase 3: API Routes 구현
**파일들:**
- `apps/admin/src/app/api/settlement/balance/route.ts`
- `apps/admin/src/app/api/settlement/payout/route.ts`
- `apps/admin/src/app/api/settlement/transfer/route.ts`

**구현된 API:**
1. `GET /api/settlement/balance` - 가맹점 송금 잔액 조회
2. `POST /api/settlement/payout` - 하위 셀러 지급 요청
3. `DELETE /api/settlement/payout` - 하위 셀러 지급 취소
4. `GET /api/settlement/payout` - 하위 셀러 지급/송금 정보 조회
5. `POST /api/settlement/transfer` - 하위 셀러 송금 요청
6. `DELETE /api/settlement/transfer` - 하위 셀러 송금 취소

**모든 API에 적용된 기능:**
- ✅ axios를 사용한 이롬넷 API 호출
- ✅ 에러 핸들링
- ✅ 한국어 에러 메시지 변환
- ✅ HTTP 상태 코드 처리
- ✅ 환경별 URL 분리 (test/production)

### ✅ Phase 4: 서비스 함수 구현
**파일:** `apps/admin/src/services/settlement.ts`

추가된 함수:
- `getBalance()` - 가맹점 송금 잔액 조회
- `requestPayout(data)` - 하위 셀러 지급 요청
- `cancelPayout(data)` - 하위 셀러 지급 취소
- `requestTransfer(data)` - 하위 셀러 송금 요청
- `cancelTransfer(data)` - 하위 셀러 송금 취소
- `getPayoutInfo(data)` - 하위 셀러 지급/송금 정보 조회

---

## 🎯 다음 단계: UI 연동

### Phase 5: React Query 훅 구현

#### Step 5.1: Query 훅 생성
**파일:** `apps/admin/src/lib/tanstack/query/settlement.ts` (새로 생성 또는 추가)

```typescript
import { useQuery } from "@tanstack/react-query";
import { getBalance, getPayoutInfo } from "@/services/settlement";
import { PayoutInfoRequest } from "@/types/settlement";

// 가맹점 송금 잔액 조회
export const useGetBalance = () => {
  return useQuery({
    queryKey: ["settlement", "balance"],
    queryFn: getBalance,
    refetchInterval: 30000, // 30초마다 자동 새로고침
    staleTime: 10000, // 10초
  });
};

// 하위 셀러 지급/송금 정보 조회
export const useGetPayoutInfo = (params: PayoutInfoRequest) => {
  return useQuery({
    queryKey: ["settlement", "payout", params.payoutId],
    queryFn: () => getPayoutInfo(params),
    enabled: !!params.payoutId && !!params.sellerId,
  });
};
```

#### Step 5.2: Mutation 훅 생성
**파일:** `apps/admin/src/lib/tanstack/mutation/settlement.ts` (기존 파일에 추가)

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { 
  requestPayout, 
  cancelPayout, 
  requestTransfer, 
  cancelTransfer 
} from "@/services/settlement";
import { successToast, errorToast } from "@/utils/toast";

// 하위 셀러 지급 요청
export const useRequestPayout = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: requestPayout,
    onSuccess: (data) => {
      if (data.resultStatus === "SUCCESS") {
        successToast("지급 요청이 완료되었습니다.");
        queryClient.invalidateQueries({ queryKey: ["settlement", "balance"] });
      } else {
        errorToast(data.resultMessage);
      }
    },
    onError: (error: Error) => {
      errorToast(error.message);
    },
  });
};

// 하위 셀러 지급 취소
export const useCancelPayout = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: cancelPayout,
    onSuccess: (data) => {
      if (data.resultStatus === "SUCCESS") {
        successToast("지급 취소가 완료되었습니다.");
        queryClient.invalidateQueries({ queryKey: ["settlement", "balance"] });
      } else {
        errorToast(data.resultMessage);
      }
    },
    onError: (error: Error) => {
      errorToast(error.message);
    },
  });
};

// 하위 셀러 송금 요청
export const useRequestTransfer = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: requestTransfer,
    onSuccess: (data) => {
      if (data.resultStatus === "SUCCESS") {
        successToast("송금 요청이 완료되었습니다.");
        queryClient.invalidateQueries({ queryKey: ["settlement", "balance"] });
      } else {
        errorToast(data.resultMessage);
      }
    },
    onError: (error: Error) => {
      errorToast(error.message);
    },
  });
};

// 하위 셀러 송금 취소
export const useCancelTransfer = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: cancelTransfer,
    onSuccess: (data) => {
      if (data.resultStatus === "SUCCESS") {
        successToast("송금 취소가 완료되었습니다.");
        queryClient.invalidateQueries({ queryKey: ["settlement", "balance"] });
      } else {
        errorToast(data.resultMessage);
      }
    },
    onError: (error: Error) => {
      errorToast(error.message);
    },
  });
};
```

### Phase 6: 잔액 조회 페이지 업데이트

**파일:** `apps/admin/src/app/settlement/balance/page.tsx`

**변경사항:**

```typescript
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { cn } from "@/lib/utils";
import { useGetBalance } from "@/lib/tanstack/query/settlement";
import { Calendar, DollarSign, RefreshCw, TrendingDown, TrendingUp } from "lucide-react";

export default function BalancePage() {
  // Mock 데이터 제거하고 실제 API 호출
  const { data: balanceData, isLoading, refetch, isRefetching } = useGetBalance();
  
  const handleRefresh = () => {
    refetch();
  };

  if (isLoading) {
    return (
      <div className="flex h-screen items-center justify-center">
        <div className="text-center">
          <RefreshCw className="mx-auto h-8 w-8 animate-spin text-gray-400" />
          <p className="mt-2 text-gray-600">로딩 중...</p>
        </div>
      </div>
    );
  }

  if (!balanceData || balanceData.resultStatus === "FAILED") {
    return (
      <div className="flex h-screen items-center justify-center">
        <div className="text-center">
          <p className="text-red-600">
            {balanceData?.resultMessage || "잔액 정보를 불러올 수 없습니다."}
          </p>
          <Button onClick={handleRefresh} className="mt-4">
            다시 시도
          </Button>
        </div>
      </div>
    );
  }

  // 기존 UI 유지하되 데이터만 balanceData로 교체
  const currentBalance = balanceData.totalBalance || 0;
  const availableBalance = balanceData.availableBalance || 0;
  const frozenBalance = balanceData.lockedBalance || 0;

  return (
    <div className="space-y-6 p-6">
      {/* 페이지 헤더 */}
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-3xl font-bold text-gray-900">잔액 조회</h1>
          <p className="mt-2 text-gray-600">가맹점 송금 잔액을 실시간으로 확인하세요</p>
        </div>
        <Button 
          onClick={handleRefresh} 
          disabled={isRefetching} 
          className="flex items-center"
        >
          <RefreshCw className={cn("mr-2 h-4 w-4", isRefetching && "animate-spin")} />
          {isRefetching ? "새로고침 중..." : "새로고침"}
        </Button>
      </div>

      {/* 잔액 카드 */}
      <div className="grid grid-cols-1 gap-6 md:grid-cols-3">
        <div className="rounded-lg bg-white p-6 shadow">
          <div className="flex items-center">
            <div className="rounded-lg bg-blue-100 p-2">
              <DollarSign className="h-6 w-6 text-blue-600" />
            </div>
            <div className="ml-4">
              <p className="text-sm font-medium text-gray-600">총 잔액</p>
              <p className="text-2xl font-bold text-gray-900">
                ₩{(currentBalance / 1000000).toFixed(1)}M
              </p>
              <p className="text-xs text-gray-500">
                ₩{currentBalance.toLocaleString()}
              </p>
            </div>
          </div>
        </div>

        <div className="rounded-lg bg-white p-6 shadow">
          <div className="flex items-center">
            <div className="rounded-lg bg-green-100 p-2">
              <TrendingUp className="h-6 w-6 text-green-600" />
            </div>
            <div className="ml-4">
              <p className="text-sm font-medium text-gray-600">사용 가능 잔액</p>
              <p className="text-2xl font-bold text-gray-900">
                ₩{(availableBalance / 1000000).toFixed(1)}M
              </p>
              <p className="text-xs text-gray-500">
                ₩{availableBalance.toLocaleString()}
              </p>
            </div>
          </div>
        </div>

        <div className="rounded-lg bg-white p-6 shadow">
          <div className="flex items-center">
            <div className="rounded-lg bg-yellow-100 p-2">
              <TrendingDown className="h-6 w-6 text-yellow-600" />
            </div>
            <div className="ml-4">
              <p className="text-sm font-medium text-gray-600">잠금 잔액</p>
              <p className="text-2xl font-bold text-gray-900">
                ₩{(frozenBalance / 1000000).toFixed(1)}M
              </p>
              <p className="text-xs text-gray-500">
                ₩{frozenBalance.toLocaleString()}
              </p>
            </div>
          </div>
        </div>
      </div>

      {/* 나머지 UI는 기존과 동일 */}
    </div>
  );
}
```

### Phase 7: 지급/송금 관리 페이지 업데이트

**파일:** `apps/admin/src/app/settlement/payments/page.tsx`

핵심 변경사항만 적용:

```typescript
import { useRequestPayout, useRequestTransfer, useCancelPayout } from "@/lib/tanstack/mutation/settlement";

export default function PaymentsPage() {
  // Mutations
  const requestPayoutMutation = useRequestPayout();
  const requestTransferMutation = useRequestTransfer();
  const cancelPayoutMutation = useCancelPayout();

  // 지급 요청 핸들러
  const handleCreatePayment = async (paymentData: any) => {
    const result = await requestPayoutMutation.mutateAsync({
      sellerId: paymentData.sellerId,
      payoutCurrency: "KRW",
      paymentAmt: paymentData.amount,
      description: paymentData.description,
    });

    if (result.resultStatus === "SUCCESS" && result.payoutId) {
      // 지급 요청 성공 후 자동으로 송금 요청
      await requestTransferMutation.mutateAsync({
        payoutId: result.payoutId,
        sellerId: paymentData.sellerId,
      });
    }
  };

  // 취소 핸들러
  const handleCancelPayment = async (payoutId: string, sellerId: string) => {
    await cancelPayoutMutation.mutateAsync({ payoutId, sellerId });
  };

  // 기존 UI 유지
  // ...
}
```

---

## 📋 체크리스트

### 완료된 작업
- [x] TypeScript 타입 정의
- [x] API 설정 확장
- [x] API Routes 구현 (6개)
- [x] 서비스 함수 구현 (6개)
- [x] 에러 처리 및 한국어 메시지
- [x] Git 커밋 (3개 커밋)

### 다음 단계
- [ ] React Query 훅 구현 (Query 2개, Mutation 4개)
- [ ] 잔액 조회 페이지 실제 API 연동
- [ ] 지급/송금 관리 페이지 실제 API 연동
- [ ] 거래 내역 페이지 업데이트
- [ ] 로딩 상태 및 에러 UI 개선
- [ ] 테스트 (Sandbox 환경)

---

## 🔍 API 엔드포인트 정리

### 이미 구현된 API ✅
1. **하위 셀러 등록** - `POST /external/seller/merchant/regist`
2. **하위 셀러 정보 조회** - `GET /external/seller/merchant/search`

### 새로 구현된 API ✅
3. **가맹점 송금 잔액 조회** - `GET /external/seller/balance`
4. **하위 셀러 지급 요청** - `POST /external/seller/payout/request`
5. **하위 셀러 지급 취소** - `POST /external/seller/payout/cancel`
6. **하위 셀러 송금 요청** - `POST /external/seller/transfer/request`
7. **하위 셀러 송금 취소** - `POST /external/seller/transfer/cancel`
8. **하위 셀러 지급/송금 정보 조회** - `GET /external/seller/payout/info`

### 아직 구현 안 된 API (선택적)
- **하위 셀러 정보 변경** - `POST /external/seller/modify`

---

## 🚀 빠른 시작 가이드

### 1. 환경 변수 확인
`.env.local` 파일에 다음 변수들이 설정되어 있는지 확인:
```
NEXT_PUBLIC_PAYMENT_MID=your_mid
NEXT_PUBLIC_PAYMENT_CLIENT_KEY=your_client_key
NEXT_PUBLIC_PAYMENT_SECRET_KEY=your_secret_key
```

### 2. React Query 훅 생성
위의 Phase 5.1, 5.2 코드를 참고하여 훅 파일 생성

### 3. UI 페이지 업데이트
- `/settlement/balance` 페이지부터 시작 (가장 간단)
- 그 다음 `/settlement/payments` 페이지
- 마지막으로 `/settlement/history` 페이지

### 4. 테스트
- Sandbox 환경에서 각 기능 테스트
- 성공/실패 케이스 확인
- 에러 메시지 확인

---

## 💡 참고사항

### 지급 → 송금 플로우
이롬넷 API의 정산 플로우:
1. **지급 요청** (`POST /payout/request`) → `payoutId` 획득
2. **송금 요청** (`POST /transfer/request`) → `payoutId` 사용
3. 이롬넷에서 영업일 오전 11시, 오후 5시에 실제 송금 처리
4. **지급/송금 정보 조회** (`GET /payout/info`)로 상태 확인

### 지급 상태 (TransactionStatus)
- `PAYMENT_REQUEST` - 지급요청
- `TRANS_REQUEST` - 송금요청
- `TRANS_PROCESS` - 송금 처리 중
- `TRANS_COMPLETE` - 송금완료
- `TRANS_FAIL` - 송금실패
- `PAYMENT_CANCEL` - 지급취소
- `TRANS_CANCEL` - 송금취소
- `TRANS_INQUIRE` - 송금결과조회

### 주의사항
1. 송금 취소는 `TRANS_REQUEST` 상태에서만 가능
2. 지급 금액은 0보다 커야 함
3. description은 최대 20자
4. 송금은 영업일에만 처리됨
5. 계좌번호와 예금주명은 암호화 필요 (이미 구현됨)

---

## 📞 문의

구현 중 문제가 발생하면:
1. 이롬넷 API 문서 (`eromnet_api_docs.pdf`) 재확인
2. 기존 구현된 `/api/settlement` 코드 참고
3. 에러 메시지 확인 (한국어로 변환됨)
4. Sandbox 환경에서 테스트

---

**작성일:** 2025-10-21
**작성자:** GitHub Copilot CLI
**관련 브랜치:** feat/TRUMIN-923
