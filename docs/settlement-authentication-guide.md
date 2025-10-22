# 정산 관리 API 인증 가이드

## 문제 상황

정산 관리 페이지에서 "0건"으로 표시되거나 데이터가 로드되지 않는 경우, 대부분 **인증 문제**입니다.

백엔드 API가 다음과 같은 응답을 반환합니다:
```json
{
  "code": "USER_001",
  "message": "해당 사용자를 찾을 수 없습니다."
}
```

## 원인

1. **인증 토큰이 없음**: API 요청 시 Authorization 헤더가 포함되지 않음
2. **로그인하지 않음**: 사용자가 로그인하지 않은 상태
3. **세션 만료**: 로그인 세션이 만료됨

## 해결 방법

### 1. 로그인 확인

Admin 페이지에 접속하기 전에 반드시 로그인해야 합니다.

### 2. API 서비스에 인증 토큰 추가

현재 `services/settlement.ts`에서 Authorization 헤더를 추가해야 합니다.

#### 현재 코드
```typescript
const response = await fetch(`${API_BASE_URL}/settlements?${queryParams.toString()}`, {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
  },
});
```

#### 수정 필요
```typescript
const response = await fetch(`${API_BASE_URL}/settlements?${queryParams.toString()}`, {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${getAccessToken()}`, // 토큰 추가
  },
  credentials: "include", // 쿠키 포함
});
```

### 3. 토큰 관리 방법

#### 방법 A: 쿠키 기반 인증 (추천)
```typescript
// API 호출 시 자동으로 쿠키 포함
const response = await fetch(url, {
  credentials: "include", // 쿠키 자동 포함
  headers: {
    "Content-Type": "application/json",
  },
});
```

#### 방법 B: Authorization 헤더 사용
```typescript
// 토큰 가져오기
function getAccessToken(): string | null {
  // 로컬 스토리지에서
  return localStorage.getItem("accessToken");
  
  // 또는 쿠키에서
  // return Cookies.get("accessToken");
}

// API 호출
const token = getAccessToken();
if (!token) {
  throw new Error("인증이 필요합니다");
}

const response = await fetch(url, {
  headers: {
    "Content-Type": "application/json",
    "Authorization": `Bearer ${token}`,
  },
});
```

### 4. 백엔드 인증 확인

백엔드 API가 다음을 확인합니다:

1. **JWT 토큰 검증**
   - 토큰이 유효한지
   - 토큰이 만료되지 않았는지
   - 서명이 올바른지

2. **사용자 정보 확인**
   - `HttpServletRequest`에서 사용자 정보 추출
   - DB에서 사용자 존재 확인
   - 권한 확인 (ADMIN, COMPANY 등)

## 구현 예시

### 1. API 서비스 수정

`TRIT-FE/apps/admin/src/services/settlement.ts`:

```typescript
// 공통 fetch 래퍼 함수
async function authenticatedFetch(url: string, options: RequestInit = {}) {
  const token = getAccessToken(); // 토큰 가져오기 함수
  
  const defaultOptions: RequestInit = {
    ...options,
    credentials: "include", // 쿠키 포함
    headers: {
      "Content-Type": "application/json",
      ...options.headers,
      ...(token && { "Authorization": `Bearer ${token}` }),
    },
  };
  
  const response = await fetch(url, defaultOptions);
  
  if (response.status === 401) {
    // 인증 실패 시 로그인 페이지로 리다이렉트
    window.location.href = "/login";
    throw new Error("인증이 필요합니다");
  }
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return response.json();
}

// 사용 예
export const getSettlementList = async (params: {...}): Promise<...> => {
  const queryParams = new URLSearchParams({...});
  return authenticatedFetch(`${API_BASE_URL}/settlements?${queryParams}`);
};
```

### 2. 토큰 관리 유틸리티

`TRIT-FE/apps/admin/src/utils/auth.ts`:

```typescript
export function getAccessToken(): string | null {
  // 방법 1: localStorage
  return localStorage.getItem("accessToken");
  
  // 방법 2: 쿠키
  // return document.cookie
  //   .split("; ")
  //   .find((row) => row.startsWith("accessToken="))
  //   ?.split("=")[1] || null;
}

export function setAccessToken(token: string): void {
  localStorage.setItem("accessToken", token);
}

export function clearAccessToken(): void {
  localStorage.removeItem("accessToken");
}

export function isAuthenticated(): boolean {
  const token = getAccessToken();
  if (!token) return false;
  
  // TODO: 토큰 만료 확인 (JWT 디코드)
  return true;
}
```

### 3. 인증 체크 HOC 또는 미들웨어

`TRIT-FE/apps/admin/src/components/AuthGuard.tsx`:

```typescript
"use client";

import { useEffect } from "react";
import { useRouter } from "next/navigation";
import { isAuthenticated } from "@/utils/auth";

export function AuthGuard({ children }: { children: React.ReactNode }) {
  const router = useRouter();
  
  useEffect(() => {
    if (!isAuthenticated()) {
      router.push("/login");
    }
  }, [router]);
  
  if (!isAuthenticated()) {
    return <div>인증 확인 중...</div>;
  }
  
  return <>{children}</>;
}
```

사용:
```typescript
// layout.tsx 또는 page.tsx
export default function SettlementPage() {
  return (
    <AuthGuard>
      {/* 정산 관리 컨텐츠 */}
    </AuthGuard>
  );
}
```

## 테스트 방법

### 1. 로그인 상태 확인
```bash
# 브라우저 콘솔에서
localStorage.getItem("accessToken")
# 또는
document.cookie
```

### 2. API 호출 직접 테스트
```bash
# 토큰 있이 (실패)
curl http://localhost:8080/api/v1/settlements

# 토큰 포함 (성공)
curl -H "Authorization: Bearer YOUR_TOKEN" http://localhost:8080/api/v1/settlements
```

### 3. 브라우저 개발자 도구
1. F12 또는 Cmd+Opt+I
2. Network 탭
3. API 요청 확인
4. Request Headers에 Authorization 헤더가 있는지 확인

## 다음 단계

1. **로그인 구현 확인**
   - Admin 로그인 페이지가 있는지
   - 로그인 후 토큰을 제대로 저장하는지

2. **API 서비스 수정**
   - 모든 API 호출에 인증 토큰 추가
   - 에러 핸들링 강화

3. **테스트**
   - 로그인 → 정산 페이지 이동
   - 데이터가 제대로 표시되는지 확인

## 참고사항

- 백엔드 컨트롤러: `SettlementController`
- 인증 요구사항: `HttpServletRequest request` 파라미터
- 필요 권한: ADMIN 또는 COMPANY
- 쿠키 기반 인증이 기본으로 설정되어 있을 가능성이 높음

## 현재 에러 핸들링

프론트엔드에 에러 핸들링이 추가되어 인증 오류 시 다음과 같이 표시됩니다:

```
⚠️ 데이터 로드 중 오류가 발생했습니다
인증이 필요하거나 서버 연결에 문제가 있습니다.
로그인 상태를 확인하거나 백엔드 서버(localhost:8080)가 실행 중인지 확인하세요.
```

이 메시지가 표시되면 로그인이 필요하다는 의미입니다.
