# 캠페인 API 명세서 (Campaign API Specification)

**Base URL**: `/api/v1/campaigns`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 개요

Campaign API는 업체와 크리에이터 간의 마케팅 캠페인 관리 기능을 제공합니다.

---

## API 엔드포인트

### 1. 캠페인 목록 조회

```http
GET /api/v1/campaigns?page=0&size=10&status=ACTIVE
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| status | String | ❌ | ACTIVE, CLOSED, COMPLETED |
| category | String | ❌ | 카테고리 필터 |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "campaignId": 1,
        "title": "제주도 여행 상품 홍보 캠페인",
        "description": "제주도 여행 상품을 SNS에 홍보해주실 크리에이터를 찾습니다",
        "thumbnailImage": "https://s3.amazonaws.com/trit/campaigns/1/thumb.jpg",
        "company": {
          "companyId": 5,
          "name": "제주투어"
        },
        "category": "ACTIVITY",
        "budget": 1000000,
        "requiredFollowers": 5000,
        "startDate": "2025-02-01",
        "endDate": "2025-02-28",
        "applicationDeadline": "2025-01-25",
        "status": "ACTIVE",
        "applicantCount": 15,
        "selectedCount": 5,
        "createdAt": "2025-01-10T10:00:00"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 25,
    "totalPages": 3
  },
  "message": null,
  "errorCode": null
}
```

### 2. 캠페인 상세 조회

```http
GET /api/v1/campaigns/{campaignId}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "campaignId": 1,
    "title": "제주도 여행 상품 홍보 캠페인",
    "description": "제주도 여행 상품을 SNS에 홍보해주실 크리에이터를 찾습니다...",
    "thumbnailImage": "https://s3.amazonaws.com/trit/campaigns/1/thumb.jpg",
    "company": {
      "companyId": 5,
      "name": "제주투어",
      "logoImage": "https://s3.amazonaws.com/trit/companies/5/logo.jpg"
    },
    "category": "ACTIVITY",
    "budget": 1000000,
    "requiredFollowers": 5000,
    "requiredPlatforms": ["INSTAGRAM", "YOUTUBE"],
    "deliverables": [
      "인스타그램 피드 2개",
      "유튜브 영상 1개",
      "스토리 5개"
    ],
    "startDate": "2025-02-01",
    "endDate": "2025-02-28",
    "applicationDeadline": "2025-01-25",
    "status": "ACTIVE",
    "applicantCount": 15,
    "selectedCreators": 5,
    "maxCreators": 10,
    "isApplied": false,
    "createdAt": "2025-01-10T10:00:00"
  },
  "message": null,
  "errorCode": null
}
```

### 3. 캠페인 신청 (크리에이터 전용)

```http
POST /api/v1/campaigns/{campaignId}/apply
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "proposalMessage": "제주 여행 콘텐츠를 전문적으로 다루고 있습니다. 참여하고 싶습니다!",
  "portfolioUrls": [
    "https://instagram.com/p/example1",
    "https://youtube.com/watch?v=example2"
  ],
  "expectedReach": 50000
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "applicationId": 10,
    "status": "PENDING"
  },
  "message": "캠페인 신청이 완료되었습니다.",
  "errorCode": null
}
```

### 4. 캠페인 생성 (업체 전용)

```http
POST /api/v1/campaigns
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "title": "부산 해양 스포츠 캠페인",
  "description": "부산 서핑 체험 상품 홍보 캠페인입니다",
  "category": "ACTIVITY",
  "budget": 2000000,
  "requiredFollowers": 10000,
  "requiredPlatforms": ["INSTAGRAM", "TIKTOK"],
  "deliverables": ["인스타그램 피드 3개", "릴스 5개"],
  "startDate": "2025-03-01",
  "endDate": "2025-03-31",
  "applicationDeadline": "2025-02-20",
  "maxCreators": 8
}
```

### 5. 캠페인 신청 승인/거절 (업체 전용)

```http
POST /api/v1/campaigns/{campaignId}/applications/{applicationId}/approve
Cookie: accessToken=eyJ...
```

```http
POST /api/v1/campaigns/{campaignId}/applications/{applicationId}/reject
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body (거절 시)**

```json
{
  "reason": "팔로워 수가 요구사항에 미달합니다."
}
```

### 6. 내 캠페인 신청 목록 조회 (크리에이터)

```http
GET /api/v1/campaigns/my-applications?page=0&size=10
Cookie: accessToken=eyJ...
```

---

## 데이터 모델

```typescript
interface CampaignResponse {
  campaignId: number;
  title: string;
  description: string;
  thumbnailImage: string;
  company: {
    companyId: number;
    name: string;
  };
  category: Category;
  budget: number;
  requiredFollowers: number;
  requiredPlatforms: Platform[];
  deliverables: string[];
  startDate: string;
  endDate: string;
  applicationDeadline: string;
  status: CampaignStatus;
  applicantCount: number;
  selectedCreators: number;
  maxCreators: number;
  isApplied: boolean;
}

type CampaignStatus = 'DRAFT' | 'ACTIVE' | 'CLOSED' | 'IN_PROGRESS' | 'COMPLETED';
type Platform = 'INSTAGRAM' | 'YOUTUBE' | 'TIKTOK' | 'BLOG';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| CAMPAIGN_NOT_FOUND | 404 | 캠페인을 찾을 수 없음 |
| CAMPAIGN_CLOSED | 400 | 마감된 캠페인 |
| ALREADY_APPLIED | 409 | 이미 신청한 캠페인 |
| APPLICATION_DEADLINE_PASSED | 400 | 신청 마감일 지남 |
| INSUFFICIENT_FOLLOWERS | 400 | 팔로워 수 부족 |
| NOT_CREATOR | 403 | 크리에이터가 아님 |

---

**문서 작성자**: Ted
