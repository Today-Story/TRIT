# 사용자 API 명세서 (Users API Specification)

**Base URL**: `/api/v1/users`  
**버전**: v1.0  
**최종 업데이트**: 2025-10-15

---

## 목차

- [개요](#개요)
- [인증 방식](#인증-방식)
- [API 엔드포인트](#api-엔드포인트)
  - [회원가입 (일반)](#1-회원가입-일반)
  - [회원가입 (크리에이터)](#2-회원가입-크리에이터)
  - [회원가입 (업체)](#3-회원가입-업체)
  - [로그인](#4-로그인)
  - [로그아웃](#5-로그아웃)
  - [내 정보 조회](#6-내-정보-조회)
  - [회원정보 수정 (일반)](#7-회원정보-수정-일반)
  - [회원정보 수정 (크리에이터)](#8-회원정보-수정-크리에이터)
  - [회원정보 수정 (업체)](#9-회원정보-수정-업체)
  - [회원 탈퇴](#10-회원-탈퇴)
  - [문의하기](#11-문의하기)
- [데이터 모델](#데이터-모델)
- [에러 코드](#에러-코드)

---

## 개요

Users API는 사용자 계정 관리와 관련된 기능을 제공합니다.

### 사용자 타입
- **USER**: 일반 사용자 (여행 상품 예약, 콘텐츠 소비)
- **CREATOR**: 크리에이터 (콘텐츠 제작 및 업로드)
- **COMPANY**: 여행 업체 (상품 등록 및 예약 관리)
- **ADMIN**: 관리자 (시스템 관리)

---

## 인증 방식

### JWT Cookie 방식
로그인 성공 시 HttpOnly Cookie로 JWT 토큰이 설정됩니다.

```http
Set-Cookie: accessToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; HttpOnly; Secure; Path=/; Max-Age=3600
Set-Cookie: refreshToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; HttpOnly; Secure; Path=/; Max-Age=1209600
```

이후 모든 인증 필요 요청에 자동으로 포함됩니다.

---

## API 엔드포인트

### 1. 회원가입 (일반)

일반 사용자로 회원가입합니다.

#### Request

```http
POST /api/v1/users/signup
Content-Type: application/json
```

**Request Body**

```json
{
  "loginId": "user123",
  "password": "password123!",
  "confirmPassword": "password123!",
  "email": "user@example.com",
  "nickname": "여행러버",
  "phoneNumber": "01012345678",
  "birthDate": "1995-05-15",
  "gender": "MALE",
  "agreeTerms": true,
  "agreePrivacy": true,
  "agreeMarketing": false
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 | 유효성 검사 |
|------|------|------|------|------------|
| loginId | String | ✅ | 로그인 아이디 | 5-20자, 영문/숫자/언더스코어 |
| password | String | ✅ | 비밀번호 | 8-20자, 영문/숫자/특수문자 조합 |
| confirmPassword | String | ✅ | 비밀번호 확인 | password와 일치 |
| email | String | ✅ | 이메일 | 유효한 이메일 형식 |
| nickname | String | ✅ | 닉네임 | 2-10자 |
| phoneNumber | String | ✅ | 휴대폰 번호 | 01X-XXXX-XXXX 형식 |
| birthDate | String | ✅ | 생년월일 | YYYY-MM-DD 형식, 만 14세 이상 |
| gender | String | ✅ | 성별 | MALE, FEMALE, OTHER |
| agreeTerms | Boolean | ✅ | 이용약관 동의 | true 필수 |
| agreePrivacy | Boolean | ✅ | 개인정보 처리방침 동의 | true 필수 |
| agreeMarketing | Boolean | ❌ | 마케팅 수신 동의 | 선택 사항 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "회원가입이 완료되었습니다. 환영 이메일을 확인해주세요.",
  "errorCode": null
}
```

**실패 예시**

```json
{
  "success": false,
  "data": null,
  "message": "이미 사용 중인 아이디입니다.",
  "errorCode": "DUPLICATE_LOGIN_ID"
}
```

---

### 2. 회원가입 (크리에이터)

크리에이터로 회원가입합니다.

#### Request

```http
POST /api/v1/users/signup/creator
Content-Type: application/json
```

**Request Body**

```json
{
  "loginId": "creator123",
  "password": "password123!",
  "confirmPassword": "password123!",
  "email": "creator@example.com",
  "nickname": "여행크리에이터",
  "phoneNumber": "01012345678",
  "birthDate": "1990-03-20",
  "gender": "FEMALE",
  "agreeTerms": true,
  "agreePrivacy": true,
  "agreeMarketing": true,
  "creatorName": "트래블포토그래퍼",
  "introduction": "여행과 사진을 사랑하는 크리에이터입니다.",
  "portfolioUrl": "https://portfolio.example.com",
  "instagramUrl": "https://instagram.com/travelphotographer",
  "youtubeUrl": "https://youtube.com/@travelphotographer"
}
```

**추가 필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| creatorName | String | ✅ | 활동명 (2-20자) |
| introduction | String | ✅ | 소개 (10-500자) |
| portfolioUrl | String | ❌ | 포트폴리오 URL |
| instagramUrl | String | ❌ | Instagram URL |
| youtubeUrl | String | ❌ | YouTube URL |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "크리에이터 회원가입 신청이 완료되었습니다. 관리자 승인 후 활동 가능합니다.",
  "errorCode": null
}
```

---

### 3. 회원가입 (업체)

여행 업체로 회원가입합니다.

#### Request

```http
POST /api/v1/users/signup/company
Content-Type: application/json
```

**Request Body**

```json
{
  "loginId": "company123",
  "password": "password123!",
  "confirmPassword": "password123!",
  "email": "company@example.com",
  "nickname": "행복여행사",
  "phoneNumber": "0212345678",
  "agreeTerms": true,
  "agreePrivacy": true,
  "agreeMarketing": false,
  "companyName": "행복여행사",
  "businessNumber": "123-45-67890",
  "representativeName": "홍길동",
  "address": "서울특별시 강남구 테헤란로 123",
  "detailedAddress": "행복빌딩 5층",
  "companyPhone": "0212345678",
  "companyEmail": "info@happytravel.com",
  "businessLicenseImage": "https://s3.amazonaws.com/trit/licenses/business-license.jpg"
}
```

**추가 필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| companyName | String | ✅ | 업체명 |
| businessNumber | String | ✅ | 사업자 등록번호 (XXX-XX-XXXXX) |
| representativeName | String | ✅ | 대표자명 |
| address | String | ✅ | 사업장 주소 |
| detailedAddress | String | ❌ | 상세 주소 |
| companyPhone | String | ✅ | 업체 전화번호 |
| companyEmail | String | ✅ | 업체 이메일 |
| businessLicenseImage | String | ✅ | 사업자등록증 이미지 URL |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "업체 회원가입 신청이 완료되었습니다. 서류 검토 후 승인됩니다.",
  "errorCode": null
}
```

---

### 4. 로그인

사용자 인증 및 JWT 토큰 발급

#### Request

```http
POST /api/v1/users/login
Content-Type: application/json
```

**Request Body**

```json
{
  "loginId": "user123",
  "password": "password123!"
}
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "userId": 1,
    "loginId": "user123",
    "email": "user@example.com",
    "nickname": "여행러버",
    "role": "USER",
    "profileImage": "https://s3.amazonaws.com/trit/profiles/user123.jpg",
    "createdAt": "2024-01-15T10:30:00"
  },
  "message": "로그인에 성공했습니다.",
  "errorCode": null
}
```

**쿠키 설정**
```
Set-Cookie: accessToken=eyJ...; HttpOnly; Secure; Path=/; Max-Age=3600
Set-Cookie: refreshToken=eyJ...; HttpOnly; Secure; Path=/; Max-Age=1209600
```

**실패 (401 UNAUTHORIZED)**

```json
{
  "success": false,
  "data": null,
  "message": "아이디 또는 비밀번호가 올바르지 않습니다.",
  "errorCode": "INVALID_CREDENTIALS"
}
```

---

### 5. 로그아웃

사용자 로그아웃 및 토큰 무효화

#### Request

```http
POST /api/v1/users/logout
Cookie: accessToken=eyJ...
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "로그아웃되었습니다.",
  "errorCode": null
}
```

**쿠키 삭제**
```
Set-Cookie: accessToken=; Max-Age=0
Set-Cookie: refreshToken=; Max-Age=0
```

---

### 6. 내 정보 조회

로그인한 사용자의 상세 정보 조회

#### Request

```http
GET /api/v1/users/me
Cookie: accessToken=eyJ...
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "userId": 1,
    "loginId": "user123",
    "email": "user@example.com",
    "nickname": "여행러버",
    "phoneNumber": "01012345678",
    "birthDate": "1995-05-15",
    "gender": "MALE",
    "role": "USER",
    "profileImage": "https://s3.amazonaws.com/trit/profiles/user123.jpg",
    "agreeMarketing": false,
    "createdAt": "2024-01-15T10:30:00",
    "lastLoginAt": "2025-10-15T09:20:00"
  },
  "message": null,
  "errorCode": null
}
```

---

### 7. 회원정보 수정 (일반)

일반 사용자의 정보 수정

#### Request

```http
PUT /api/v1/users/edit
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "nickname": "새닉네임",
  "phoneNumber": "01098765432",
  "profileImage": "https://s3.amazonaws.com/trit/profiles/new-profile.jpg",
  "agreeMarketing": true
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| nickname | String | ❌ | 닉네임 (2-10자) |
| phoneNumber | String | ❌ | 휴대폰 번호 |
| profileImage | String | ❌ | 프로필 이미지 URL |
| agreeMarketing | Boolean | ❌ | 마케팅 수신 동의 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "userId": 1,
    "nickname": "새닉네임",
    "phoneNumber": "01098765432",
    "profileImage": "https://s3.amazonaws.com/trit/profiles/new-profile.jpg"
  },
  "message": "회원정보가 수정되었습니다.",
  "errorCode": null
}
```

---

### 8. 회원정보 수정 (크리에이터)

크리에이터의 정보 수정

#### Request

```http
PUT /api/v1/users/edit/creator
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "nickname": "새닉네임",
  "phoneNumber": "01098765432",
  "profileImage": "https://s3.amazonaws.com/trit/profiles/new-profile.jpg",
  "creatorName": "새활동명",
  "introduction": "수정된 소개",
  "portfolioUrl": "https://new-portfolio.com",
  "instagramUrl": "https://instagram.com/newhandle",
  "youtubeUrl": "https://youtube.com/@newchannel"
}
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "userId": 2,
    "nickname": "새닉네임",
    "creatorName": "새활동명",
    "introduction": "수정된 소개"
  },
  "message": "크리에이터 정보가 수정되었습니다.",
  "errorCode": null
}
```

---

### 9. 회원정보 수정 (업체)

업체의 정보 수정

#### Request

```http
PUT /api/v1/users/edit/company
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "nickname": "새업체명",
  "phoneNumber": "0298765432",
  "companyPhone": "0298765432",
  "companyEmail": "new-info@happytravel.com",
  "address": "서울특별시 강남구 새주소 456",
  "detailedAddress": "새빌딩 7층"
}
```

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "companyId": 3,
    "companyName": "새업체명",
    "companyPhone": "0298765432",
    "address": "서울특별시 강남구 새주소 456"
  },
  "message": "업체 정보가 수정되었습니다.",
  "errorCode": null
}
```

---

### 10. 회원 탈퇴

사용자 계정 삭제 (소프트 삭제)

#### Request

```http
DELETE /api/v1/users/me
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "password": "password123!",
  "reason": "서비스 불만족",
  "detailedReason": "원하는 여행지 정보가 부족합니다."
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| password | String | ✅ | 현재 비밀번호 (본인 확인) |
| reason | String | ✅ | 탈퇴 사유 |
| detailedReason | String | ❌ | 상세 사유 |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": null,
  "message": "회원 탈퇴가 완료되었습니다. 그동안 이용해주셔서 감사합니다.",
  "errorCode": null
}
```

**실패 - 비밀번호 불일치 (400 BAD REQUEST)**

```json
{
  "success": false,
  "data": null,
  "message": "비밀번호가 일치하지 않습니다.",
  "errorCode": "INVALID_PASSWORD"
}
```

**비즈니스 로직**
- 계정은 즉시 비활성화되며, 30일 후 완전 삭제
- 예약 중인 상품이 있으면 탈퇴 불가
- 탈퇴 후 동일 아이디로 30일간 재가입 불가

---

### 11. 문의하기

1:1 문의 등록

#### Request

```http
POST /api/v1/users/inquire
Content-Type: application/json
Cookie: accessToken=eyJ...
```

**Request Body**

```json
{
  "category": "RESERVATION",
  "title": "예약 취소 문의",
  "content": "예약 취소는 어떻게 하나요?",
  "attachments": [
    "https://s3.amazonaws.com/trit/inquiries/screenshot1.jpg",
    "https://s3.amazonaws.com/trit/inquiries/screenshot2.jpg"
  ]
}
```

**필드 설명**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| category | String | ✅ | 문의 유형 (RESERVATION, PAYMENT, PRODUCT, SERVICE, OTHER) |
| title | String | ✅ | 문의 제목 (최대 100자) |
| content | String | ✅ | 문의 내용 (최대 2000자) |
| attachments | Array | ❌ | 첨부 파일 URL 배열 (최대 5개) |

#### Response

**성공 (200 OK)**

```json
{
  "success": true,
  "data": {
    "inquiryId": 123,
    "status": "PENDING",
    "createdAt": "2025-10-15T10:30:00"
  },
  "message": "문의가 등록되었습니다. 24시간 내에 답변 드리겠습니다.",
  "errorCode": null
}
```

---

## 데이터 모델

### UserResponse

```typescript
interface UserResponse {
  userId: number;
  loginId: string;
  email: string;
  nickname: string;
  phoneNumber: string;
  birthDate: string;
  gender: 'MALE' | 'FEMALE' | 'OTHER';
  role: 'USER' | 'CREATOR' | 'COMPANY' | 'ADMIN';
  profileImage: string | null;
  agreeMarketing: boolean;
  createdAt: string;
  lastLoginAt: string;
}
```

### CreatorResponse

```typescript
interface CreatorResponse extends UserResponse {
  creatorName: string;
  introduction: string;
  portfolioUrl: string | null;
  instagramUrl: string | null;
  youtubeUrl: string | null;
  approvalStatus: 'PENDING' | 'APPROVED' | 'REJECTED';
  approvedAt: string | null;
}
```

### CompanyResponse

```typescript
interface CompanyResponse extends UserResponse {
  companyName: string;
  businessNumber: string;
  representativeName: string;
  address: string;
  detailedAddress: string | null;
  companyPhone: string;
  companyEmail: string;
  approvalStatus: 'PENDING' | 'APPROVED' | 'REJECTED';
  approvedAt: string | null;
}
```

---

## 에러 코드

### 회원가입 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| DUPLICATE_LOGIN_ID | 409 | 이미 사용 중인 아이디 |
| DUPLICATE_EMAIL | 409 | 이미 등록된 이메일 |
| DUPLICATE_NICKNAME | 409 | 이미 사용 중인 닉네임 |
| INVALID_BUSINESS_NUMBER | 400 | 유효하지 않은 사업자 등록번호 |
| UNDERAGE_USER | 400 | 만 14세 미만 가입 불가 |
| TERMS_NOT_AGREED | 400 | 필수 약관 미동의 |

### 로그인 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INVALID_CREDENTIALS | 401 | 아이디 또는 비밀번호 불일치 |
| ACCOUNT_LOCKED | 403 | 계정 잠김 (5회 로그인 실패) |
| ACCOUNT_SUSPENDED | 403 | 계정 정지 (관리자 조치) |
| ACCOUNT_PENDING | 403 | 승인 대기 중 (크리에이터/업체) |

### 회원정보 수정 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| INVALID_PHONE_NUMBER | 400 | 유효하지 않은 전화번호 형식 |
| INVALID_IMAGE_URL | 400 | 유효하지 않은 이미지 URL |
| NICKNAME_TOO_SHORT | 400 | 닉네임이 너무 짧음 (2자 미만) |
| NICKNAME_TOO_LONG | 400 | 닉네임이 너무 김 (10자 초과) |

### 회원 탈퇴 관련

| 에러 코드 | HTTP 상태 | 설명 |
|----------|----------|------|
| ACTIVE_RESERVATION_EXISTS | 400 | 진행 중인 예약 존재 |
| PENDING_PAYMENT_EXISTS | 400 | 미결제 건 존재 |
| WITHDRAWAL_COOLDOWN | 400 | 탈퇴 후 30일 내 재가입 불가 |

---

## 사용 예시 (TypeScript + React Query)

```typescript
import { useMutation, useQuery } from '@tanstack/react-query';
import { apiClient } from '@/lib/api/client';

// 회원가입
interface SignupData {
  loginId: string;
  password: string;
  confirmPassword: string;
  email: string;
  nickname: string;
  // ... other fields
}

export function useSignup() {
  return useMutation({
    mutationFn: (data: SignupData) =>
      apiClient.post('/api/v1/users/signup', data),
    onSuccess: () => {
      alert('회원가입이 완료되었습니다!');
    },
  });
}

// 로그인
export function useLogin() {
  return useMutation({
    mutationFn: (credentials: { loginId: string; password: string }) =>
      apiClient.post('/api/v1/users/login', credentials),
    onSuccess: (data) => {
      console.log('로그인 성공:', data.data);
    },
  });
}

// 내 정보 조회
export function useMyInfo() {
  return useQuery({
    queryKey: ['myInfo'],
    queryFn: () => apiClient.get<UserResponse>('/api/v1/users/me'),
    staleTime: 5 * 60 * 1000, // 5분
  });
}

// 회원정보 수정
interface UpdateProfileData {
  nickname?: string;
  phoneNumber?: string;
  profileImage?: string;
  agreeMarketing?: boolean;
}

export function useUpdateProfile() {
  return useMutation({
    mutationFn: (data: UpdateProfileData) =>
      apiClient.put('/api/v1/users/edit', data),
    onSuccess: () => {
      // 캐시 무효화하여 최신 정보 다시 가져오기
      queryClient.invalidateQueries(['myInfo']);
    },
  });
}

// 로그아웃
export function useLogout() {
  return useMutation({
    mutationFn: () => apiClient.post('/api/v1/users/logout'),
    onSuccess: () => {
      queryClient.clear(); // 모든 캐시 제거
      window.location.href = '/login';
    },
  });
}
```

---

**문서 작성자**: Ted  
**문의**: backend-team@trit.today
