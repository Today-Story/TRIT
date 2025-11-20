# getProductById 함수 분석 및 비교 보고서

**작성일**: 2025-11-20  
**분석 대상**: ProductSearchService vs TatintaProductService

---

## 📊 현재 상황 분석

### 1. **ProductSearchService.getProductById()** (원본)

```java
@Override
public ResultResponse<DetailProductResponse> getProductById(Long productId, AuthInfo authInfo) {
    Long userId = authInfo != null ? authInfo.getId() : null;

    // ✅ 최적화된 쿼리 사용
    DetailProductWithRelationsResponse productWithRelations =
        productFetcher.fetchProductWithRelations(productId);

    // ✅ 좋아요 여부 확인
    boolean liked = userId != null && 
        productLikeHistoryRepository.existsByProductIdAndUserId(productId, userId);

    // ✅ 회사 정보 조회
    Company company = companyRepository.findById(productWithRelations.companyId())
        .orElseThrow(() -> new NotFoundException(ExceptionCode.COMPANY_NOT_FOUND));

    String companyLoginId = company.getUser() != null ? company.getUser().getUserId() : null;
    String companyThumbnailUrl = company.getProfileUrl();

    // ✅ 운영시간 조회
    List<BusinessHourResponse> businessHours = businessHourRepository
        .findByCompanyId(company.getId()).stream()
        .map(BusinessHourResponse::from)
        .toList();

    // ✅ 컨텐츠 조회
    List<SimpleContentsResponse> simpleContentsResponses =
        productFetcher.fetchSimpleContentsForProduct(productWithRelations);

    // ✅ 쿠폰 존재 여부 확인 후 조건부 조회 (최적화)
    boolean hasRelatedCoupons = couponRepository.existsCouponsByProduct(productId);
    List<CouponResponse> relatedCoupons = hasRelatedCoupons
        ? couponSearchService.searchCouponsByProductId(productId, 0, 100).getData()
        : List.of();

    // ✅ DetailProductResponse 생성
    return ResultResponse.of(
        ResultCode.GET_PRODUCT_SUCCESS,
        DetailProductResponse.fromOptimized(
            productWithRelations,
            liked,
            simpleContentsResponses,
            companyLoginId,
            companyThumbnailUrl,
            businessHours,
            relatedCoupons,
            hasRelatedCoupons
        )
    );
}
```

**특징**:
- ✅ **최적화된 쿼리**: `productFetcher.fetchProductWithRelations()` 사용
- ✅ **N+1 방지**: 한 번에 모든 관계 데이터 조회
- ✅ **조건부 쿠폰 조회**: 쿠폰 존재 여부 확인 후 필요시에만 조회
- ✅ **완전한 데이터**: 모든 필드 포함 (businessHours, contents, coupons 등)
- ✅ **사용자별 좋아요 상태**: AuthInfo 기반 개인화

---

### 2. **TatintaProductService.getProductById()** (현재)

```java
public TatintaResponse<TatintaProductDetailDto> getProductById(Long id) {
    try {
        if (id == null || id <= 0) {
            throw new IllegalArgumentException("Invalid ID format '" + id + "'");
        }

        // ❌ 단순 위임: ProductSearchService 호출만
        DetailProductResponse productDetail = productSearchService
            .getProductById(id, null)  // ❌ authInfo = null
            .getData();

        // ✅ TATINTA 형식으로 변환
        TatintaProductDetailDto tatintaProduct = mapper.toDetailDto(productDetail);

        return TatintaResponse.success(tatintaProduct);

    } catch (IllegalArgumentException e) {
        log.warn("Bad request for product detail: {}", e.getMessage());
        throw e;
    } catch (NoSuchElementException e) {
        log.warn("Product not found: {}", id);
        throw new NoSuchElementException("Product not found with id '" + id + "'");
    } catch (Exception e) {
        log.error("Failed to retrieve product detail for id: {}", id, e);
        throw new RuntimeException("Failed to retrieve product");
    }
}
```

**특징**:
- ✅ **올바른 위임**: ProductSearchService 재사용
- ❌ **authInfo = null**: 좋아요 정보 없음 (Tatinta는 익명이므로 정상)
- ✅ **TATINTA 형식 변환**: mapper 사용
- ⚠️ **예외 처리**: 불필요한 try-catch (이미 ProductSearchService에서 처리됨)

---

## 🔍 **문제점 및 개선 사항**

### ❌ 현재 문제점

1. **불필요한 예외 래핑**
   - `ProductSearchService`가 이미 `NotFoundException`을 던짐
   - `TatintaProductService`에서 `NoSuchElementException`으로 다시 래핑
   - 일관성 없는 예외 처리

2. **중복 검증**
   - `id == null || id <= 0` 검증은 ProductSearchService에서 이미 수행
   - 불필요한 중복 검증

3. **Generic Exception Catch**
   - `catch (Exception e)` → 너무 광범위한 예외 처리
   - 예상치 못한 에러를 숨길 수 있음

---

## ✅ **개선안**

### 수정된 TatintaProductService.getProductById()

```java
/**
 * 상품 상세 조회
 *
 * @param id 상품 ID
 * @return 상품 상세 정보 응답
 */
public TatintaResponse<TatintaProductDetailDto> getProductById(Long id) {
    try {
        // ProductSearchService를 통한 상세 조회
        // - authInfo는 null (Tatinta는 익명 사용자이므로 liked = false)
        // - ProductSearchService에서 모든 검증 및 조회 처리
        DetailProductResponse productDetail = productSearchService
            .getProductById(id, null)
            .getData();

        // TATINTA 형식으로 변환
        TatintaProductDetailDto tatintaProduct = mapper.toDetailDto(productDetail);

        return TatintaResponse.success(tatintaProduct);

    } catch (NotFoundException e) {
        // ProductSearchService에서 던진 NotFoundException 처리
        log.warn("Product not found: {}", id);
        throw new NoSuchElementException("Product not found with id '" + id + "'");
    }
}
```

**개선 사항**:
- ✅ 불필요한 `id` 검증 제거 (ProductSearchService에서 처리)
- ✅ 명확한 예외 처리 (`NotFoundException`만 catch)
- ✅ Generic Exception catch 제거
- ✅ 주석 추가로 authInfo=null 이유 명시

---

## 📝 **결론**

### ✅ **올바른 아키텍처**

```
TatintaProductController
    ↓ (REST API 요청)
TatintaProductService
    ↓ (위임)
ProductSearchService ← ✅ 여기가 핵심 비즈니스 로직
    ↓ (조회)
ProductRepository, ProductFetcher, 기타 Repository들
```

**현재 구조는 올바름**:
- `ProductSearchService`가 핵심 비즈니스 로직 담당
- `TatintaProductService`는 단순 어댑터 역할
- 코드 중복 없이 재사용성 확보

### ⚠️ **수정 필요 사항**

**TatintaProductService**:
1. 불필요한 검증 제거
2. 예외 처리 간소화
3. 주석 개선

**ProductSearchService**:
- 수정 불필요 (현재 구조가 최적)

---

## 🔧 **권장 사항**

### 1. TatintaProductService 간소화

```java
public TatintaResponse<TatintaProductDetailDto> getProductById(Long id) {
    try {
        DetailProductResponse productDetail = productSearchService
            .getProductById(id, null)
            .getData();

        TatintaProductDetailDto tatintaProduct = mapper.toDetailDto(productDetail);
        return TatintaResponse.success(tatintaProduct);

    } catch (NotFoundException e) {
        log.warn("Product not found for Tatinta request: {}", id);
        throw new NoSuchElementException("Product not found with id '" + id + "'");
    }
}
```

### 2. 공통 예외 처리기 활용

Tatinta API 전용 `@ControllerAdvice` 생성:

```java
@ControllerAdvice(basePackages = "today.story.backend.external.tatinta")
public class TatintaExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<TatintaErrorResponse> handleNotFound(NotFoundException e) {
        return ResponseEntity.status(404)
            .body(TatintaErrorResponse.of("NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(NoSuchElementException.class)
    public ResponseEntity<TatintaErrorResponse> handleNoSuchElement(NoSuchElementException e) {
        return ResponseEntity.status(404)
            .body(TatintaErrorResponse.of("NOT_FOUND", e.getMessage()));
    }
}
```

---

## 📊 **최종 비교**

| 항목 | ProductSearchService | TatintaProductService |
|------|---------------------|---------------------|
| **역할** | 핵심 비즈니스 로직 | 어댑터 (형식 변환) |
| **데이터 조회** | ✅ 최적화된 쿼리 | ❌ 직접 조회 없음 (위임) |
| **N+1 방지** | ✅ Fetch Join 사용 | N/A (위임) |
| **사용자별 좋아요** | ✅ AuthInfo 기반 | ❌ 항상 null (익명) |
| **예외 처리** | ✅ 명확한 도메인 예외 | ⚠️ 개선 필요 |
| **응답 형식** | ResultResponse | TatintaResponse |
| **수정 필요** | ❌ 현재 최적 | ✅ 간소화 필요 |

---

**결론**: ProductSearchService가 더 올바른 구현이며, TatintaProductService는 이를 활용하되 예외 처리만 개선하면 됩니다.
