# 결제 금액 계산 로직 검증 보고서

**작성일**: 2025-12-12  
**작성자**: AI Assistant  
**검증 범위**: Frontend → Backend 결제 금액 계산 전체 흐름

---

## 📋 요약

결제 금액 계산 로직을 Frontend와 Backend 양측에서 크로스 체크한 결과, **현재 구현은 정확**합니다.  
프론트엔드가 할인된 금액을 전송하더라도, **백엔드가 독립적으로 재계산**하여 이중 할인이 발생하지 않습니다.

### 핵심 결론
- ✅ **33,000원 상품 × 2명 = 66,000원 + 10,000원 할인 쿠폰 → 56,000원 정확 계산**
- ✅ 프론트엔드가 잘못된 금액을 보내도 백엔드가 재계산하여 방어
- ✅ 모든 쿠폰 타입 (정액/정률, 최소금액, 최대할인) 정확히 동작
- ✅ 14개 테스트 케이스 전체 통과

---

## 1️⃣ Frontend 분석

### 위치
- `TRIT-FE/apps/platform/src/page/shopping/book/confirm/index.tsx`

### 로직 (Line 109-123, 152)

```typescript
// 프론트엔드에서 쿠폰 할인 계산
const calculateDiscountAmount = () => {
  if (!selectedCoupon) return 0;

  const basePrice = +searchParams.price; // 예: 66,000원
  const { discountType, discountValue } = selectedCoupon.couponTemplate;

  if (discountType === "PERCENTAGE") {
    return Math.floor((basePrice * discountValue) / 100);
  } else {
    return Math.min(discountValue, basePrice); // 정액 할인
  }
};

const discountAmount = calculateDiscountAmount(); // 10,000
const finalPrice = +searchParams.price - discountAmount; // 66,000 - 10,000 = 56,000

// 서버로 전송
const body = {
  ...
  requestAmount: finalPrice,  // 👈 이미 할인된 금액 (56,000원) 전송
  coupon: coupon,
};
```

### 문제점 가능성
- 프론트엔드가 `requestAmount`에 **이미 할인된 금액 (56,000원)**을 전송
- 만약 백엔드가 이 값을 그대로 사용하면 이중 할인 발생 가능

---

## 2️⃣ Backend 분석

### 위치
- `TRIT-BE/backend/src/main/java/today/story/backend/payment/service/PaymentServiceImpl.java`

### 로직 흐름

#### Step 1: 초기 결제 정보 저장 (`saveInitialPayment`)

```java
// Line 125: 백엔드가 독립적으로 금액 재계산
BigDecimal originalAmount = calculateOriginalAmountBeforeCoupon(product, paymentRequest);
// ↓
// product.price (33,000) × peopleCount (2) = 66,000원

// Line 154-171: Payment 엔티티 생성
Payment payment = Payment.builder()
    .requestAmount(originalAmount)         // 66,000
    .originalRequestAmount(originalAmount) // 66,000
    .amount(originalAmount)                // 66,000 (초기값)
    .build();

// Line 182-183: 쿠폰 적용
if (couponId != null) {
    applyCouponToPayment(payment, user, couponId);
    // ↓ payment.applyCoupon(couponHistory) 호출
}
```

#### Step 2: 쿠폰 할인 적용 (`Payment.applyCoupon`)

```java
// Payment.java Line 205-214
public void applyCoupon(CouponIssuanceHistory couponIssuanceHistory) {
    this.appliedCoupon = couponIssuanceHistory;
    
    if (couponIssuanceHistory != null && this.requestAmount != null) {
      // 👇 requestAmount (66,000원) 기준으로 할인 계산
      Integer discountAmount = couponIssuanceHistory.getCouponTemplate()
          .calculateDiscountAmount(this.requestAmount.intValue());
      
      // 👇 최종 금액 업데이트
      this.amount = this.requestAmount.subtract(BigDecimal.valueOf(discountAmount));
      // 66,000 - 10,000 = 56,000
    }
}
```

#### Step 3: 쿠폰 할인 금액 계산 (`Coupon.calculateDiscountAmount`)

```java
// Coupon.java Line 279-307
public Integer calculateDiscountAmount(Integer originalPrice) {
    // 최소 주문 금액 체크
    if (this.minimumOrderPrice != null && originalPrice < this.minimumOrderPrice) {
      return 0;
    }

    Integer discountAmount;

    // 할인 유형에 따른 계산
    if (this.discountType == DiscountType.PERCENTAGE) {
      // 정률 할인
      discountAmount = (originalPrice * this.discountValue) / 100;
    } else {
      // 정액 할인
      discountAmount = this.discountValue;
    }

    // 최대 할인 금액 제한 체크
    if (this.maxDiscountAmount != null && discountAmount > this.maxDiscountAmount) {
      discountAmount = this.maxDiscountAmount;
    }

    // 할인 금액이 원래 가격을 초과하지 않도록
    if (discountAmount > originalPrice) {
      discountAmount = originalPrice;
    }

    return discountAmount;
}
```

### 핵심 방어 메커니즘

1. **백엔드 독립 계산**: `calculateOriginalAmountBeforeCoupon`에서 Product + Options + AdditionalProducts 기준으로 금액 재계산
2. **프론트 값 무시**: `PaymentRequest.requestAmount`는 **무시**되고, 백엔드 계산 값이 우선
3. **단일 할인 적용**: `applyCoupon` 메서드는 **한 번만 호출**되며, `requestAmount` 기준으로 할인

---

## 3️⃣ 테스트 검증 결과

### 테스트 파일
- `TRIT-BE/backend/src/test/java/today/story/backend/payment/domain/PaymentCouponCalculationTest.java`

### 테스트 케이스 (14개 전체 통과 ✅)

#### 정액 할인 쿠폰 (4개 케이스)
1. ✅ **기본 정액 할인**: 66,000원 - 10,000원 = 56,000원
2. ✅ **최소 주문 금액 미달**: 66,000원 (할인 불가, 최소 100,000원)
3. ✅ **할인 금액 초과**: 66,000원 - 100,000원 쿠폰 = 0원 (전액 할인)
4. ✅ **복잡한 금액**: 105,000원 - 10,000원 = 95,000원

#### 정률 할인 쿠폰 (4개 케이스)
5. ✅ **기본 정률 할인**: 66,000원 × 15% = 9,900원 할인 → 56,100원
6. ✅ **최대 할인 제한**: 66,000원 × 30% = 19,800원이지만 최대 10,000원 → 56,000원
7. ✅ **최소 주문 금액 미달**: 66,000원 (할인 불가, 최소 100,000원)
8. ✅ **복잡한 금액**: 105,000원 × 15% = 15,750원 할인 → 89,250원

#### 쿠폰 제거/복원 (2개 케이스)
9. ✅ **쿠폰 제거 시 원래 금액 복원**: 56,000원 → 66,000원
10. ✅ **중복 쿠폰 적용 방지**: 마지막 쿠폰만 적용

#### 엣지 케이스 (4개 케이스)
11. ✅ **0원 주문**: 할인 불가
12. ✅ **100% 할인**: 66,000원 → 0원
13. ✅ **1% 소액 할인**: 66,000원 × 1% = 660원 할인 → 65,340원
14. ✅ **최소 주문 금액 정확히 일치**: 할인 가능

### 테스트 실행 결과

```bash
cd TRIT-BE/backend && ./gradlew test --tests "PaymentCouponCalculationTest"

BUILD SUCCESSFUL in 10s
14 tests completed, 14 passed ✅
```

---

## 4️⃣ 시나리오별 금액 흐름 추적

### 시나리오 1: 기본 정액 할인 (질문 케이스)

| 단계 | 위치 | requestAmount | originalRequestAmount | amount | 설명 |
|------|------|---------------|----------------------|--------|------|
| 1 | Frontend | - | - | 56,000 | 프론트가 할인 적용 (66,000 - 10,000) |
| 2 | Backend 초기 | 66,000 | 66,000 | 66,000 | 백엔드 재계산 (프론트 값 무시) |
| 3 | applyCoupon | 66,000 | 66,000 | 56,000 | 쿠폰 할인 적용 |
| **결과** | **DB 저장** | **66,000** | **66,000** | **56,000** | ✅ **정확** |

### 시나리오 2: 정률 할인 + 최대 제한

| 단계 | 위치 | requestAmount | amount | 할인 금액 | 설명 |
|------|------|---------------|--------|-----------|------|
| 1 | Backend 초기 | 66,000 | 66,000 | 0 | 초기 상태 |
| 2 | 할인 계산 | 66,000 | - | 19,800 | 66,000 × 30% |
| 3 | 최대 제한 적용 | 66,000 | - | 10,000 | maxDiscountAmount |
| 4 | applyCoupon | 66,000 | 56,000 | 10,000 | 최종 금액 |
| **결과** | **DB 저장** | **66,000** | **56,000** | **10,000** | ✅ **정확** |

### 시나리오 3: 모든 옵션 조합 + 쿠폰

| 구성 요소 | 금액 | 설명 |
|-----------|------|------|
| 인원 옵션 (성인 × 2) | 60,000 | 30,000 × 2 |
| 인원 옵션 (아동 × 1) | 20,000 | 20,000 × 1 |
| 추가 옵션 (사진 × 2) | 10,000 | 5,000 × 2 |
| 추가 상품 (기념품) | 15,000 | 15,000 × 1 |
| **합계** | **105,000** | |
| 쿠폰 할인 (15%) | -15,750 | 105,000 × 0.15 |
| **최종 금액** | **89,250** | ✅ **정확** |

---

## 5️⃣ 잠재적 문제 시나리오 (현재는 안전)

### ❌ 만약 백엔드가 프론트 값을 사용했다면?

```java
// 잘못된 구현 (현재는 이렇게 안 됨)
BigDecimal originalAmount = paymentRequest.getRequestAmount(); // 56,000 (프론트 값)

Payment payment = Payment.builder()
    .requestAmount(originalAmount)  // 56,000 ❌
    .amount(originalAmount)         // 56,000 ❌
    .build();

payment.applyCoupon(history); // 56,000 - 10,000 = 46,000 ❌❌ 이중 할인!
```

### ✅ 현재 안전한 구현

```java
// 올바른 구현 (현재 코드)
BigDecimal originalAmount = calculateOriginalAmountBeforeCoupon(...); // 66,000

Payment payment = Payment.builder()
    .requestAmount(originalAmount)  // 66,000 ✅
    .amount(originalAmount)         // 66,000 ✅
    .build();

payment.applyCoupon(history); // 66,000 - 10,000 = 56,000 ✅
```

---

## 6️⃣ 권장 사항

### Frontend 개선
```typescript
// 현재: 할인된 금액 전송
requestAmount: finalPrice,  // 56,000

// 권장: 원래 금액 전송 (백엔드와 검증 목적)
requestAmount: +searchParams.price,  // 66,000
```

**이유**: 
- 백엔드와 프론트엔드 계산 결과를 비교하여 불일치 감지 가능
- 현재는 백엔드가 프론트 값을 무시하므로 큰 문제는 없지만, 검증 로직 추가 가능

### Backend 검증 로직 추가 (선택사항)

```java
// PaymentServiceImpl.java
BigDecimal calculatedAmount = calculateOriginalAmountBeforeCoupon(...);
BigDecimal frontendAmount = paymentRequest.getRequestAmount();

// 프론트엔드와 백엔드 계산 불일치 로깅
if (frontendAmount != null && 
    !calculatedAmount.equals(frontendAmount) && 
    paymentRequest.getCoupon() == null) { // 쿠폰 없는 경우만
    
    log.warn("금액 불일치 감지 - Frontend: {}, Backend: {}, OrderId: {}",
        frontendAmount, calculatedAmount, paymentRequest.getOrderId());
}
```

---

## 7️⃣ 결론

### ✅ 현재 시스템 안전성
1. **이중 할인 불가**: 백엔드가 독립적으로 금액 재계산
2. **정확한 할인 계산**: 모든 쿠폰 타입 및 제한 조건 정확히 동작
3. **프론트 값 방어**: 프론트엔드가 잘못된 값을 보내도 백엔드가 재계산

### 📊 테스트 커버리지
- ✅ 정액 할인 쿠폰: 4개 케이스
- ✅ 정률 할인 쿠폰: 4개 케이스
- ✅ 쿠폰 제거/복원: 2개 케이스
- ✅ 엣지 케이스: 4개 케이스
- **총 14개 케이스 전체 통과**

### 🎯 최종 답변
**질문**: "66,000원에 10,000원 할인 쿠폰 적용 시 46,000원으로 저장되는가?"  
**답변**: **아니오, 56,000원으로 정확히 저장됩니다.** ✅

만약 실제로 46,000원이 저장되었다면, 다음을 확인해야 합니다:
1. DB에 저장된 실제 데이터 (payment 테이블 직접 조회)
2. 쿠폰 타입이 정액이 아닌 정률일 가능성
3. 다른 프로모션이나 이벤트 할인이 추가로 적용되었을 가능성
4. 결제 로그 확인 (applyCoupon 호출 횟수 및 파라미터)

---

**작성자 노트**: 본 검증은 코드 레벨 분석 및 단위 테스트 기반이며, 실제 운영 환경에서 발생한 46,000원 케이스가 있다면 해당 트랜잭션의 상세 로그를 확인해야 합니다.
