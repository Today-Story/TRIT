# Admin Memo API 명세

## 개요
어드민이 유저, 콘텐츠, 장소, 상품, 이벤트, 배너, 쿠폰 등에 대한 관리 메모를 작성, 조회, 수정할 수 있는 API입니다.

## 엔드포인트 목록

### 1. 메모 작성
- **URL**: `POST /api/v1/admin/memo`
- **설명**: 특정 엔티티에 대한 관리 메모를 작성합니다.
- **권한**: ADMIN
- **Request**:
  ```json
  {
    "contents": "메모 내용",
    "memoField": "USER",
    "referenceId": 123
  }
  ```
  - `contents` (String, required): 메모 내용
  - `memoField` (Enum, required): 메모 대상 필드 (USER, CONTENT, LOCATION, PRODUCT, EVENT, BANNER, COUPON)
  - `referenceId` (Long, required): 대상 엔티티의 ID
  
- **Response**:
  ```json
  {
    "status": "SUCCESS",
    "message": "어드민 메모 작성 성공",
    "data": null
  }
  ```

- **Error Codes**:
  - `ADMIN_MEMO_ALREADY_EXISTS`: 해당 엔티티에 대한 메모가 이미 존재합니다.
  - `NO_GRANTED_EXCEPTION`: 권한이 없습니다.

---

### 2. 메모 수정
- **URL**: `PUT /api/v1/admin/memo?memoId={memoId}`
- **설명**: 기존 메모의 내용을 수정합니다.
- **권한**: ADMIN
- **Query Parameters**:
  - `memoId` (Long, required): 수정할 메모의 ID
  
- **Request**:
  ```json
  {
    "contents": "수정된 메모 내용",
    "memoField": "USER",
    "referenceId": 123
  }
  ```
  
- **Response**:
  ```json
  {
    "status": "SUCCESS",
    "message": "어드민 메모 수정 성공",
    "data": null
  }
  ```

- **Error Codes**:
  - `ADMIN_MEMO_NOT_FOUND`: 메모를 찾을 수 없습니다.
  - `NO_GRANTED_EXCEPTION`: 권한이 없습니다.

---

### 3. 메모 조회
- **URL**: `GET /api/v1/admin/memo?memoField={memoField}&referenceId={referenceId}`
- **설명**: 특정 엔티티에 대한 메모를 조회합니다.
- **권한**: ADMIN
- **Query Parameters**:
  - `memoField` (String, required): 메모 대상 필드 (USER, CONTENT, LOCATION, PRODUCT, EVENT, BANNER, COUPON)
  - `referenceId` (Long, required): 대상 엔티티의 ID
  
- **Response** (메모가 있는 경우):
  ```json
  {
    "status": "SUCCESS",
    "message": "어드민 메모 조회 성공",
    "data": {
      "id": 1,
      "contents": "메모 내용",
      "memoField": "USER",
      "referenceId": 123
    }
  }
  ```

- **Response** (메모가 없는 경우):
  ```json
  {
    "status": "SUCCESS",
    "message": "어드민 메모 조회 성공",
    "data": null
  }
  ```

- **Error Codes**:
  - `NO_GRANTED_EXCEPTION`: 권한이 없습니다.

---

## MemoField Enum
```
USER - 유저
CONTENT - 콘텐츠
LOCATION - 장소
PRODUCT - 상품
EVENT - 이벤트
BANNER - 배너
COUPON - 쿠폰
```

## 데이터베이스 스키마
```sql
CREATE TABLE admin_memo (
    id BIGSERIAL PRIMARY KEY,
    contents TEXT,
    memo_field VARCHAR(50) NOT NULL,
    reference_id BIGINT NOT NULL,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    UNIQUE(memo_field, reference_id)
);
```

## 사용 예시

### 유저에 대한 메모 작성
```bash
curl -X POST https://api.example.com/api/v1/admin/memo \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token}" \
  -d '{
    "contents": "이 유저는 자주 문의하는 VIP 고객입니다.",
    "memoField": "USER",
    "referenceId": 456
  }'
```

### 상품에 대한 메모 조회
```bash
curl -X GET "https://api.example.com/api/v1/admin/memo?memoField=PRODUCT&referenceId=789" \
  -H "Authorization: Bearer {token}"
```

### 메모 수정
```bash
curl -X PUT "https://api.example.com/api/v1/admin/memo?memoId=1" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {token}" \
  -d '{
    "contents": "수정된 메모 내용",
    "memoField": "PRODUCT",
    "referenceId": 789
  }'
```

## 참고 사항
- 하나의 엔티티(memoField + referenceId 조합)에는 하나의 메모만 존재할 수 있습니다.
- 메모 삭제 기능은 현재 미구현 상태입니다. 필요시 contents를 빈 문자열로 업데이트하거나 별도의 삭제 API를 추가할 수 있습니다.
- 메모는 BaseEntity를 상속하므로 createdAt, updatedAt 필드를 자동으로 가집니다.
