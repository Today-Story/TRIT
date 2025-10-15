# 인증 API 명세서 (Auth API Specification)

**Base URL**: `/api/v1/auth`  
**버전**: v1.0  
**최종 업데이트**: 2025-01-15

---

## 목차

- [개요](#개요)
- [인증 방식](#인증-방식)
- [API 엔드포인트](#api-엔드포인트)
  - [아이디 중복 확인](#1-아이디-중복-확인)
  - [이메일 중복 확인](#2-이메일-중복-확인)
  - [아이디 찾기](#3-아이디-찾기)
  - [비밀번호 찾기 - 인증 코드 전송](#4-비밀번호-찾기---인증-코드-전송)
  - [비밀번호 찾기 - 인증 코드 검증](#5-비밀번호-찾기---인증-코드-검증)
  - [비밀번호 찾기 - 인증 코드 재전송](#6-비밀번호-찾기---인증-코드-재전송)
  - [비밀번호 변경](#7-비밀번호-변경)
- [공통 응답 형식](#공통-응답-형식)
- [에러 코드](#에러-코드)

---

## 개요

Auth API는 사용자 인증 및 계정 복구와 관련된 기능을 제공합니다. 로그인은 별도의 Users API에서 처리됩니다.

### 주요 기능
- 회원가입 시 아이디/이메일 중복 확인
- 아이디 찾기 (이메일 + 닉네임으로 조회)
- 비밀번호 찾기 (이메일 인증 코드 방식)
- 비밀번호 변경

---

## 인증 방식

대부분의 Auth API는 **인증 불필요**하지만, 비밀번호 변경은 **JWT 인증 필요**합니다.

### JWT 토큰 전달 방법
```http
Cookie: accessToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## API 엔드포인트

### 1. 아이디 중복 확인

회원가입 시 사용자가 입력한 아이디가 이미 사용 중인지 확인합니다.

#### Request

```http
GET /api/v1/auth/check/id?loginId={loginId}
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| loginId | String | ✅ | 확인할 로그인 아이디 |

**예시**
```http
GET /api/v1/auth/check/id?loginId=user123
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": true,
  "message": null,
  "errorCode": null
}
```

**응답 필드**

| 필드 | 타입 | 설명 |
|------|------|------|
| success | Boolean | 요청 성공 여부 |
| data | Boolean | `true`: 중복됨 (사용 불가), `false`: 사용 가능 |
| message | String | 에러 메시지 (성공 시 null) |
| errorCode | String | 에러 코드 (성공 시 null) |

---

### 2. 이메일 중복 확인

회원가입 시 사용자가 입력한 이메일이 이미 등록되어 있는지 확인합니다.

#### Request

```http
GET /api/v1/auth/check/email?email={email}
```

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| email | String | ✅ | 확인할 이메일 주소 |

**예시**
```http
GET /api/v1/auth/check/email?email=user@example.com
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": false,
  "message": null,
  "errorCode": null
}
```

**응답 필드**

| 필드 | 타입 | 설명 |
|------|------|------|
| success | Boolean | 요청 성공 여부 |
| data | Boolean | `true`: 중복됨 (사용 불가), `false`: 사용 가능 |
| message | String | 에러 메시지 (성공 시 null) |
| errorCode | String | 에러 코드 (성공 시 null) |

---

### 3. 아이디 찾기

사용자가 이메일과 닉네임을 제공하면 해당하는 로그인 아이디를 반환합니다.

#### Request

```http
POST /api/v1/auth/find/id
Content-Type: application/json
```

**Request Body**

```json
{
  "email": "user@example.com",
  "nickname": "여행러버"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| email | String | ✅ | 가입 시 사용한 이메일 |
| nickname | String | ✅ | 가입 시 설정한 닉네임 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": "user123",
  "message": null,
  "errorCode": null
}
```

**실패 - 사용자를 찾을 수 없음 (404 NOT FOUND)**

```json
{
  "success": false,
  "data": null,
  "message": "해당 정보와 일치하는 사용자를 찾을 수 없습니다.",
  "errorCode": "USER_NOT_FOUND"
}
```

---

### 4. 비밀번호 찾기 - 인증 코드 전송

비밀번호 재설정을 위해 사용자의 이메일로 6자리 인증 코드를 전송합니다.

#### Request

```http
POST /api/v1/auth/find/password
Content-Type: application/json
```

**Request Body**

```json
{
  "loginId": "user123",
  "email": "user@example.com"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| loginId | String | ✅ | 로그인 아이디 |
| email | String | ✅ | 가입 시 사용한 이메일 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "인증 코드가 이메일로 전송되었습니다.",
  "errorCode": null
}
```

**실패 - 사용자를 찾을 수 없음 (404 NOT FOUND)**

```json
{
  "success": false,
  "data": null,
  "message": "아이디와 이메일이 일치하는 사용자를 찾을 수 없습니다.",
  "errorCode": "USER_NOT_FOUND"
}
```

**비즈니스 로직**
- 인증 코드는 Redis에 5분간 저장됩니다.
- 이메일은 비동기로 전송됩니다.
- 동일 사용자는 1분에 최대 3회까지만 요청 가능합니다 (Rate Limiting).

---

### 5. 비밀번호 찾기 - 인증 코드 검증

사용자가 입력한 인증 코드가 유효한지 검증합니다.

#### Request

```http
POST /api/v1/auth/verify
Content-Type: application/json
```

**Request Body**

```json
{
  "email": "user@example.com",
  "code": "123456"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| email | String | ✅ | 인증 코드를 받은 이메일 |
| code | String | ✅ | 6자리 인증 코드 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "인증에 성공했습니다.",
  "errorCode": null
}
```

**실패 - 인증 코드 불일치 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "인증 코드가 일치하지 않습니다.",
  "errorCode": "INVALID_VERIFICATION_CODE"
}
```

**실패 - 인증 코드 만료 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "인증 코드가 만료되었습니다.",
  "errorCode": "VERIFICATION_CODE_EXPIRED"
}
```

---

### 6. 비밀번호 찾기 - 인증 코드 재전송

인증 코드를 다시 전송합니다.

#### Request

```http
POST /api/v1/auth/resend
Content-Type: application/json
```

**Request Body**

```json
{
  "loginId": "user123",
  "email": "user@example.com"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| loginId | String | ✅ | 로그인 아이디 |
| email | String | ✅ | 가입 시 사용한 이메일 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "인증 코드가 재전송되었습니다.",
  "errorCode": null
}
```

**실패 - 너무 많은 요청 (429 TOO MANY REQUESTS)**

```json
{
  "success": false,
  "data": null,
  "message": "인증 코드 재전송은 1분 후에 다시 시도해주세요.",
  "errorCode": "TOO_MANY_REQUESTS"
}
```

---

### 7. 비밀번호 변경

인증된 사용자의 비밀번호를 변경합니다. **JWT 인증 필요**.

#### Request

```http
PUT /api/v1/auth/password
Content-Type: application/json
Cookie: accessToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Request Body**

```json
{
  "currentPassword": "oldPassword123!",
  "newPassword": "newPassword456!",
  "confirmPassword": "newPassword456!"
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 | 유효성 검사 |
|------|------|------|------|------------|
| currentPassword | String | ✅ | 현재 비밀번호 | - |
| newPassword | String | ✅ | 새 비밀번호 | 8-20자, 영문/숫자/특수문자 조합 |
| confirmPassword | String | ✅ | 새 비밀번호 확인 | newPassword와 일치 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "비밀번호가 변경되었습니다.",
  "errorCode": null
}
```

**실패 - 현재 비밀번호 불일치 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "현재 비밀번호가 일치하지 않습니다.",
  "errorCode": "INVALID_PASSWORD"
}
```

**실패 - 새 비밀번호 형식 오류 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "비밀번호는 8-20자의 영문, 숫자, 특수문자 조합이어야 합니다.",
  "errorCode": "INVALID_PASSWORD_FORMAT"
}
```

**실패 - 비밀번호 확인 불일치 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "새 비밀번호와 확인 비밀번호가 일치하지 않습니다.",
  "errorCode": "PASSWORD_MISMATCH"
}
```

**실패 - 인증 필요 (401 UNAUTHORIZED)**

```json
{
  "success": false,
  "data": null,
  "message": "인증이 필요합니다.",
  "errorCode": "UNAUTHORIZED"
}
```

---

## 공통 응답 형식

모든 API는 다음 형식의 응답을 반환합니다:

```typescript
interface ResultResponse<T> {
  success: boolean;      // 요청 성공 여부
  data: T | null;        // 응답 데이터 (실패 시 null)
  message: string | null;// 메시지 (성공 시 null, 실패 시 에러 메시지)
  errorCode: string | null; // 에러 코드 (성공 시 null)
}
```

---

## 에러 코드

### 인증 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| UNAUTHORIZED | 401 | 인증 토큰이 없거나 유효하지 않음 |
| TOKEN_EXPIRED | 401 | 액세스 토큰 만료 |
| INVALID_TOKEN | 401 | 토큰 형식 오류 또는 변조됨 |

### 사용자 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| USER_NOT_FOUND | 404 | 사용자를 찾을 수 없음 |
| DUPLICATE_LOGIN_ID | 409 | 이미 사용 중인 아이디 |
| DUPLICATE_EMAIL | 409 | 이미 등록된 이메일 |

### 비밀번호 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INVALID_PASSWORD | 400 | 현재 비밀번호가 일치하지 않음 |
| INVALID_PASSWORD_FORMAT | 400 | 비밀번호 형식 오류 |
| PASSWORD_MISMATCH | 400 | 새 비밀번호와 확인 비밀번호 불일치 |
| WEAK_PASSWORD | 400 | 비밀번호 강도가 약함 |

### 인증 코드 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INVALID_VERIFICATION_CODE | 400 | 인증 코드 불일치 |
| VERIFICATION_CODE_EXPIRED | 400 | 인증 코드 만료 (5분 초과) |
| VERIFICATION_CODE_NOT_FOUND | 400 | 인증 코드 발송 이력 없음 |

### Rate Limiting

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| TOO_MANY_REQUESTS | 429 | 요청 횟수 초과 (1분에 3회 제한) |

### 서버 오류

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INTERNAL_SERVER_ERROR | 500 | 서버 내부 오류 |
| EMAIL_SEND_FAILED | 500 | 이메일 전송 실패 |

---

## 사용 예시

### JavaScript (Fetch API)

```javascript
// 아이디 중복 확인
async function checkLoginId(loginId) {
  const response = await fetch(
    `/api/v1/auth/check/id?loginId=${encodeURIComponent(loginId)}`
  );
  const result = await response.json();
  
  if (result.success) {
    if (result.data) {
      console.log('이미 사용 중인 아이디입니다.');
    } else {
      console.log('사용 가능한 아이디입니다.');
    }
  }
}

// 비밀번호 찾기
async function findPassword(loginId, email) {
  const response = await fetch('/api/v1/auth/find/password', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ loginId, email }),
  });
  
  const result = await response.json();
  
  if (result.success) {
    alert('인증 코드가 이메일로 전송되었습니다.');
  } else {
    alert(result.message);
  }
}

// 인증 코드 검증
async function verifyCode(email, code) {
  const response = await fetch('/api/v1/auth/verify', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ email, code }),
  });
  
  const result = await response.json();
  
  if (result.success) {
    console.log('인증 성공! 비밀번호 재설정 페이지로 이동합니다.');
  } else {
    console.error(result.message);
  }
}

// 비밀번호 변경 (인증 필요)
async function changePassword(currentPassword, newPassword, confirmPassword) {
  const response = await fetch('/api/v1/auth/password', {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    credentials: 'include', // 쿠키 포함
    body: JSON.stringify({
      currentPassword,
      newPassword,
      confirmPassword,
    }),
  });
  
  const result = await response.json();
  
  if (result.success) {
    alert('비밀번호가 성공적으로 변경되었습니다.');
  } else {
    alert(result.message);
  }
}
```

### TypeScript (React Query)

```typescript
import { useMutation, useQuery } from '@tanstack/react-query';
import { apiClient } from '@/lib/api/client';

// 아이디 중복 확인
export function useCheckLoginId(loginId: string) {
  return useQuery({
    queryKey: ['checkLoginId', loginId],
    queryFn: () => apiClient.get<boolean>(`/api/v1/auth/check/id`, { 
      params: { loginId } 
    }),
    enabled: loginId.length > 0,
  });
}

// 비밀번호 찾기 Mutation
export function useFindPassword() {
  return useMutation({
    mutationFn: (data: { loginId: string; email: string }) =>
      apiClient.post('/api/v1/auth/find/password', data),
    onSuccess: () => {
      alert('인증 코드가 이메일로 전송되었습니다.');
    },
    onError: (error: any) => {
      alert(error.response?.data?.message || '오류가 발생했습니다.');
    },
  });
}

// 인증 코드 검증 Mutation
export function useVerifyCode() {
  return useMutation({
    mutationFn: (data: { email: string; code: string }) =>
      apiClient.post('/api/v1/auth/verify', data),
    onSuccess: () => {
      console.log('인증 성공!');
    },
  });
}

// 비밀번호 변경 Mutation
interface ChangePasswordData {
  currentPassword: string;
  newPassword: string;
  confirmPassword: string;
}

export function useChangePassword() {
  return useMutation({
    mutationFn: (data: ChangePasswordData) =>
      apiClient.put('/api/v1/auth/password', data),
    onSuccess: () => {
      alert('비밀번호가 변경되었습니다.');
    },
  });
}
```

---

## 보안 고려사항

### 1. Rate Limiting
- 아이디/이메일 찾기: 1분에 최대 5회
- 인증 코드 전송: 1분에 최대 3회
- 비밀번호 변경: 1시간에 최대 5회

### 2. 인증 코드 보안
- 6자리 숫자 랜덤 생성
- Redis에 5분간 저장 후 자동 삭제
- 3회 틀릴 경우 해당 코드 무효화

### 3. 비밀번호 정책
- 최소 8자, 최대 20자
- 영문 대소문자, 숫자, 특수문자 중 3가지 이상 조합
- 이전 비밀번호 재사용 방지 (최근 3개)
- BCrypt로 해싱 (강도 12)

### 4. HTTPS 필수
- 모든 Auth API는 HTTPS를 통해서만 접근 가능
- HTTP 요청은 자동으로 HTTPS로 리다이렉트

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|----------|
| 1.0 | 2025-01-15 | 초기 버전 작성 |

---

**문서 작성자**: Backend Team  
**문의**: backend-team@trit.today
