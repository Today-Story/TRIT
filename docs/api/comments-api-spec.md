# 댓글 API 명세서 (Comments API Specification)

**Base URL**: `/api/v1/comments`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 개요

Comments API는 콘텐츠 및 상품에 대한 댓글 관리 기능을 제공합니다.

### 주요 기능
- 댓글 작성
- 댓글 조회 (계층형 구조)
- 댓글 수정/삭제
- 대댓글 작성
- 댓글 신고

---

## API 엔드포인트

### 1. 댓글 목록 조회

특정 콘텐츠 또는 상품의 댓글을 조회합니다.

```http
GET /api/v1/comments?targetType=CONTENTS&targetId=123&page=0&size=20
Cookie: accessToken=eyJ... (선택)
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| targetType | String | ✅ | CONTENTS, PRODUCT |
| targetId | Long | ✅ | 대상 ID |
| page | Integer | ❌ | 페이지 번호 (기본값: 0) |
| size | Integer | ❌ | 페이지 크기 (기본값: 20) |
| sortOption | String | ❌ | LATEST, OLDEST (기본값: LATEST) |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "commentId": 1,
        "content": "정말 유익한 정보네요! 감사합니다.",
        "user": {
          "userId": 10,
          "nickname": "여행러버",
          "profileImage": "https://s3.amazonaws.com/trit/profiles/user10.jpg"
        },
        "likeCount": 15,
        "isLiked": false,
        "replies": [
          {
            "commentId": 2,
            "content": "저도 도움 많이 받았어요!",
            "user": {
              "userId": 11,
              "nickname": "트래블메이트",
              "profileImage": "https://s3.amazonaws.com/trit/profiles/user11.jpg"
            },
            "likeCount": 3,
            "isLiked": false,
            "createdAt": "2025-01-15T11:30:00",
            "isEdited": false,
            "isDeleted": false
          }
        ],
        "replyCount": 1,
        "createdAt": "2025-01-15T10:20:00",
        "updatedAt": "2025-01-15T10:25:00",
        "isEdited": true,
        "isDeleted": false
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 256,
    "totalPages": 13
  },
  "message": null,
  "errorCode": null
}
```

---

### 2. 댓글 작성

```http
POST /api/v1/comments
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "targetType": "CONTENTS",
  "targetId": 123,
  "content": "정말 유익한 정보네요! 감사합니다.",
  "parentCommentId": null
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| targetType | String | ✅ | CONTENTS, PRODUCT |
| targetId | Long | ✅ | 대상 ID |
| content | String | ✅ | 댓글 내용 (최소 2자, 최대 500자) |
| parentCommentId | Long | ❌ | 대댓글인 경우 부모 댓글 ID |

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "commentId": 1,
    "content": "정말 유익한 정보네요! 감사합니다.",
    "createdAt": "2025-01-15T10:20:00"
  },
  "message": "댓글이 등록되었습니다.",
  "errorCode": null
}
```

---

### 3. 댓글 수정

```http
PUT /api/v1/comments/{commentId}
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "content": "수정된 댓글 내용입니다."
}
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "commentId": 1,
    "content": "수정된 댓글 내용입니다.",
    "updatedAt": "2025-01-15T10:25:00"
  },
  "message": "댓글이 수정되었습니다.",
  "errorCode": null
}
```

---

### 4. 댓글 삭제

```http
DELETE /api/v1/comments/{commentId}
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "댓글이 삭제되었습니다.",
  "errorCode": null
}
```

**비즈니스 로직**:
- 대댓글이 있는 경우: "삭제된 댓글입니다" 표시 (소프트 삭제)
- 대댓글이 없는 경우: 완전 삭제

---

### 5. 댓글 좋아요 추가/취소

```http
POST /api/v1/comments/{commentId}/like
Cookie: accessToken=eyJ...
```

**Response (200 OK)**

```json
{
  "success": true,
  "data": {
    "isLiked": true,
    "likeCount": 16
  },
  "message": "좋아요가 추가되었습니다.",
  "errorCode": null
}
```

---

### 6. 댓글 신고

```http
POST /api/v1/comments/{commentId}/report
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "reason": "ABUSE",
  "detailedReason": "욕설이 포함된 댓글입니다."
}
```

**신고 사유 (reason)**
- `ABUSE`: 욕설/비방
- `SPAM`: 스팸/광고
- `INAPPROPRIATE`: 부적절한 내용
- `COPYRIGHT`: 저작권 침해
- `OTHER`: 기타

**Response (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "신고가 접수되었습니다. 관리자가 검토 후 조치합니다.",
  "errorCode": null
}
```

---

## 데이터 모델

```typescript
interface CommentResponse {
  commentId: number;
  content: string;
  user: {
    userId: number;
    nickname: string;
    profileImage: string | null;
  };
  likeCount: number;
  isLiked: boolean;
  replies: CommentResponse[];
  replyCount: number;
  createdAt: string;
  updatedAt: string | null;
  isEdited: boolean;
  isDeleted: boolean;
}

type TargetType = 'CONTENTS' | 'PRODUCT';
type ReportReason = 'ABUSE' | 'SPAM' | 'INAPPROPRIATE' | 'COPYRIGHT' | 'OTHER';
```

---

## 에러 코드

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| COMMENT_NOT_FOUND | 404 | 댓글을 찾을 수 없음 |
| UNAUTHORIZED_COMMENT_ACCESS | 403 | 댓글 수정/삭제 권한 없음 |
| COMMENT_TOO_SHORT | 400 | 댓글이 너무 짧음 (2자 미만) |
| COMMENT_TOO_LONG | 400 | 댓글이 너무 김 (500자 초과) |
| PARENT_COMMENT_NOT_FOUND | 404 | 부모 댓글을 찾을 수 없음 |
| INVALID_TARGET_TYPE | 400 | 유효하지 않은 대상 타입 |
| ALREADY_REPORTED | 409 | 이미 신고한 댓글 |

---

**문서 작성자**: Backend Team
