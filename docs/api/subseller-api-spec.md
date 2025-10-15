# 하위 판매자 API 명세서 (SubSeller API Specification)

**Base URL**: `/api/v1/sub-sellers` ⚠️  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15  
**구현 상태**: 🚧 부분 구현 (기본 CRUD만 구현됨)

---

## ⚠️ 중요 공지

현재 **기본 등록 및 조회 기능만 구현**되어 있습니다.
통계, 정산, API 키 관리 등의 고급 기능은 향후 구현 예정입니다.

---

## 개요

SubSeller API는 여행 업체가 하위 판매자를 관리하는 기능을 제공합니다.
하위 판매자는 상위 업체의 상품을 판매하고 수수료를 받는 파트너입니다.

### 현재 구현된 기능
- ✅ 하위 판매자 등록 (ADMIN 전용)
- ✅ 하위 판매자 목록 조회 (페이지네이션)

### 🚧 향후 구현 예정
- 하위 판매자 상세 조회
- 하위 판매자 정보 수정
- 하위 판매자 상태 변경
- 판매 통계 및 정산 기능
- API 키 관리

---

## API 엔드포인트

### 1. 하위 판매자 등록 (ADMIN 전용) ✅

```http
POST /api/v1/sub-sellers
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "name": "제주 로컬 투어",
  "email": "jeju@localtour.com",
  "phone": "064-123-4567",
  "businessNumber": "456-78-90123",
  "representativeName": "김제주",
  "address": "제주특별자치도 제주시 중앙로 456"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | String | ✅ | 하위 판매자 업체명 |
| email | String | ✅ | 이메일 |
| phone | String | ✅ | 전화번호 |
| businessNumber | String | ✅ | 사업자등록번호 |
| representativeName | String | ✅ | 대표자명 |
| address | String | ✅ | 주소 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "하위 판매자가 등록되었습니다.",
  "errorCode": null
}
```

---

### 2. 하위 판매자 목록 조회 ✅

```http
GET /api/v1/sub-sellers?page=0&size=10
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 10) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "subsellerId": 1,
        "name": "강남 여행사",
        "email": "gangnam@travel.com",
        "phone": "02-1234-5678",
        "businessNumber": "123-45-67890",
        "representativeName": "홍길동",
        "address": "서울특별시 강남구 테헤란로 123",
        "createdAt": "2024-12-01T10:00:00"
      },
      {
        "subsellerId": 2,
        "name": "부산 투어",
        "email": "busan@tour.com",
        "phone": "051-9876-5432",
        "businessNumber": "987-65-43210",
        "representativeName": "김부산",
        "address": "부산광역시 해운대구 해운대로 456",
        "createdAt": "2024-11-15T09:00:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 15,
    "totalPages": 2,
    "first": true,
    "last": false
  },
  "message": null,
  "errorCode": null
}
```

---

## 🚧 향후 구현 예정 API

아래 API들은 현재 백엔드에서 구현되지 않았으며, 향후 추가될 예정입니다.

### 3. 하위 판매자 상세 조회 🚧

```http
GET /api/v1/sub-sellers/{subsellerId}
Cookie: accessToken=eyJ...
```

### 4. 하위 판매자 정보 수정 🚧

```http
PUT /api/v1/sub-sellers/{subsellerId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

### 5. 하위 판매자 상태 변경 🚧

```http
PATCH /api/v1/sub-sellers/{subsellerId}/status
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "status": "SUSPENDED",
  "reason": "계약 위반"
}
```

**상태 옵션**
- `ACTIVE`: 활성
- `SUSPENDED`: 정지
- `INACTIVE`: 비활성

### 6. 하위 판매자 판매 통계 조회 🚧

```http
GET /api/v1/sub-sellers/{subsellerId}/statistics?startDate=2025-01-01&endDate=2025-01-31
Cookie: accessToken=eyJ...
```

### 7. 하위 판매자 정산 내역 조회 🚧

```http
GET /api/v1/sub-sellers/{subsellerId}/settlements?page=0&size=20
Cookie: accessToken=eyJ...
```

### 8. 하위 판매자 API 키 재발급 🚧

```http
POST /api/v1/sub-sellers/{subsellerId}/regenerate-keys
Cookie: accessToken=eyJ...
```

---

## 하위 판매자용 API 🚧

### 9. 내 판매자 정보 조회 🚧

```http
GET /api/v1/sub-sellers/me
Cookie: accessToken=eyJ...
```

### 10. 내 판매 통계 조회 🚧

```http
GET /api/v1/sub-sellers/me/statistics?startDate=2025-01-01&endDate=2025-01-31
Cookie: accessToken=eyJ...
```

### 11. 내 정산 내역 조회 🚧

```http
GET /api/v1/sub-sellers/me/settlements?page=0&size=20
Cookie: accessToken=eyJ...
```

---

## 데이터 모델

### SubSellerResponse

```typescript
interface SubSellerResponse {
  subsellerId: number;
  name: string;
  email: string;
  phone: string;
  businessNumber: string;
  representativeName: string;
  address: string;
  createdAt: string;
}

// 향후 추가될 필드들
interface DetailedSubSellerResponse extends SubSellerResponse {
  status: SubSellerStatus;
  commissionRate: number;
  contractStartDate: string;
  contractEndDate: string;
  bankAccount: BankAccount;
  permissions: Permission[];
  statistics: SubSellerStatistics;
  updatedAt: string;
}

type SubSellerStatus = 'ACTIVE' | 'SUSPENDED' | 'INACTIVE';
type Permission = 'VIEW_PRODUCTS' | 'SELL_PRODUCTS' | 'VIEW_ORDERS' | 'MANAGE_ORDERS' | 'VIEW_STATISTICS';

interface SubSellerStatistics {
  totalSales: number;
  totalCommission: number;
  totalOrders: number;
  averageOrderValue: number;
  productCount: number;
}
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| SUBSELLER_NOT_FOUND | 404 | 하위 판매자를 찾을 수 없음 |
| DUPLICATE_BUSINESS_NUMBER | 409 | 이미 등록된 사업자등록번호 |
| UNAUTHORIZED | 401 | 인증 필요 |
| FORBIDDEN | 403 | 권한 없음 (ADMIN만 가능) |

---

## 비즈니스 로직

### 하위 판매자 등록 프로세스

1. **관리자 확인**: ADMIN 권한 확인
2. **중복 검사**: 사업자등록번호 중복 확인
3. **정보 저장**: 하위 판매자 정보 DB에 저장
4. **초기 상태**: ACTIVE 상태로 생성

### 향후 구현 예정 기능

#### 수수료 정산 시스템
- 매월 1일 자동 정산 처리
- 정산 금액 = 판매 금액 × 수수료율
- 은행 계좌로 자동 송금

#### 권한 관리
- 상품 조회 권한
- 상품 판매 권한
- 주문 조회 권한
- 주문 관리 권한
- 통계 조회 권한

#### API 키 관리
- Access Key / Secret Key 발급
- 외부 시스템 연동용
- 보안을 위한 주기적 재발급

---

**문서 작성자**: Ted  
**문의**: backend-team@trit.today

**개발 로드맵**:
- ✅ Phase 1: 기본 CRUD (완료)
- 🚧 Phase 2: 상세 조회 및 수정 기능 (개발 예정)
- 🚧 Phase 3: 통계 및 정산 기능 (개발 예정)
- 🚧 Phase 4: API 키 관리 및 외부 연동 (개발 예정)
