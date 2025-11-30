---
title: 와플스튜디오 프로젝트 중 발생한 이슈 및 해결
author: mathrabbit
date: 2025-11-30 12:00:00 +0900
categories: []
tags: []
---

<!--more-->

와플스튜디오 5-7주차 프로젝트 중 발생했던 세 가지 이슈와 해결 방법을 정리해보고자 합니다.

## 1. Context API를 활용한 전역 상태 관리

처음에는 props drilling으로 사용자 정보를 전달했었는데, 컴포넌트 depth가 깊어지면서 코드가 지저분해졌습니다. 

**문제 상황:**
```tsx
// 이런 식으로 3~4 depth를 거쳐 user를 전달해야 했음
<App user={user}>
  <NavBar user={user}>
    <UserMenu user={user} />
  </NavBar>
</App>
```

**해결 방법: AuthContext 도입**

```tsx
// src/auth/AuthContext.tsx
const AuthContext = createContext<AuthContextValue | undefined>(undefined);

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [user, setUser] = useState<User | null>(() => {
    // localStorage에서 캐시된 유저 정보 복원
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      if (!raw) return null;
      return JSON.parse(raw) as User;
    } catch {
      return null;
    }
  });

  const login = async (input: { email: string; password: string }) => {
    await apiLogin(input);
    await refresh(); // 유저 정보 갱신
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, ... }}>
      {children}
    </AuthContext.Provider>
  );
};
```

이제 어떤 컴포넌트에서든 `useAuth()` 훅으로 간단하게 사용자 정보에 접근할 수 있습니다:

```tsx
// src/components/PostCard.tsx
const { user } = useAuth();

const toggle = async () => {
  if (!user) {
    onRequireLogin?.(); // 로그인 안 되어있으면 모달 표시
    return;
  }
  // 북마크 처리 로직...
};
```

- Context API는 props drilling을 해결하는 강력한 도구
- 초기 렌더링 시 localStorage에서 상태를 복원하면 새로고침해도 로그인 상태 유지 가능
- `useCallback`으로 함수를 메모이제이션하면 불필요한 리렌더링 방지

---

## 2. URL 상태 관리로 필터링 구현하기

필터 상태를 컴포넌트 state로만 관리하면 **새로고침하면 필터가 초기화**되는 문제가 있었습니다.

**해결: URLSearchParams 활용**

```tsx
// src/pages/Posts.tsx
const [sp] = useSearchParams();
const page = parseInt(sp.get('page') ?? '1', 10);

const queryParams = useMemo(() => {
  const roles = sp.getAll('roles');      // 역할 (프론트엔드, 백엔드 등)
  const domains = sp.getAll('domains');  // 도메인 (핀테크, 헬스테크 등)
  const isActive = sp.get('isActive') ?? 'false';
  const order = sp.get('order') ?? '0';
  
  return { page: page - 1, roles, domains, isActive, order };
}, [sp, page, user]);
```

URL 예시: `/?page=2&roles=FRONT&roles=BACKEND&domains=FINTECH&isActive=true`

**장점:**
- **공유 가능**: 필터링된 결과를 URL로 공유 가능
- **새로고침 안전**: 페이지 새로고침해도 필터 유지
- **뒤로가기 지원**: 브라우저 뒤로가기/앞으로가기 자동 지원

---

## 3. 비동기 처리와 Race Condition 해결

**마주친 버그:**

필터를 빠르게 여러 번 변경하면, 이전 요청의 응답이 나중에 도착해서 화면에 잘못된 데이터가 표시되는 현상이 발생했습니다.

```
사용자: "프론트엔드" 선택 → API 요청 A 시작
사용자: "백엔드"로 변경 → API 요청 B 시작
응답 B 도착 (백엔드 공고 표시)
응답 A 도착 (프론트엔드 공고로 덮어씀) --> 문제!
```

**해결: Cleanup 함수 활용**

```tsx
useEffect(() => {
  let alive = true;  // 플래그 변수
  
  setLoading(true);
  getPosts(queryParams)
    .then((data) => {
      if (!alive) return;  // 이미 unmount되었거나 새 요청이 시작되면 무시
      setPosts(data.posts ?? []);
      setLastPage(data.paginator?.lastPage ?? 1);
    })
    .catch((_e) => {
      if (!alive) return;
      setPosts([]);
      setLastPage(1);
    })
    .finally(() => alive && setLoading(false));
  
  return () => {
    alive = false;  // cleanup 시 플래그를 false로
  };
}, [queryParams]);
```

**핵심 원리:**
1. `alive` 플래그로 현재 effect가 유효한지 추적
2. 새로운 effect가 실행되면 이전 effect의 cleanup 함수가 호출되어 `alive = false`
3. 이후 이전 요청의 응답이 와도 `if (!alive) return`으로 무시

---

끝!