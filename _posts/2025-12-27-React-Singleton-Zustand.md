---
layout: post
title: "251227 React에서 Singleton 패턴과 Zustand로 상태 관리하기"
comments: true
tags:
  - react
  - zustand
  - singleton
  - state-management
  - typescript
---

# React에서 Singleton 패턴과 Zustand로 상태 관리하기

React 애플리케이션에서 전역 상태를 효율적으로 관리하기 위해 Zustand와 Singleton 패턴을 결합한 실용적인 접근법을 소개합니다.

## MySingle - User + Product 통합 관리

### 타입 정의

```typescript
// src/types/User.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

// src/types/Product.ts
export interface Product {
  id: number;
  name: string;
  imgUrl: string;
}
```

### Zustand Store와 Singleton 클래스

```typescript
// src/utils/MySingle.ts
import * as Zustand from 'zustand';
import * as ZustandMiddleware from 'zustand/middleware';
import type { User } from '../types/User';
import type { Product } from '../types/Product';

const store = Zustand.create<{
  user: User | null;
  currentLookingProduct: Product | null;
  setUser: (user: User | null) => void;
  setProduct: (product: Product | null) => void;
}>()( 
  ZustandMiddleware.persist(
    (set) => ({
      user: null,
      currentLookingProduct: null,
      setUser: (user) => set({ user }),
      setProduct: (product) => set({ currentLookingProduct: product })
    }),
    {
      name: 'app-storage'
    }
  )
);

export class MySingle {
  // ========== User 관련 ==========
  
  public static getUser(): User | null {
    return store.getState().user;
  }

  public static setUser(user: User | null): void {
    store.getState().setUser(user);
  }

  public static useUser() {
    return store((state) => state.user);
  }

  // ========== Product 관련 ==========
  
  public static getProduct(): Product | null {
    return store.getState().currentLookingProduct;
  }

  public static setProduct(product: Product | null): void {
    store.getState().setProduct(product);
  }

  public static useProduct() {
    return store((state) => state.currentLookingProduct);
  }

  // ========== 공통 ==========
  
  public static clear(): void {
    store.getState().setUser(null);
    store.getState().setProduct(null);
  }
}
```

## 사용 예제

```typescript
// src/App.tsx
import { MySingle } from './utils/MySingle';
import type { User } from './types/User';
import type { Product } from './types/Product';

function App() {
  const user = MySingle.useUser();
  const product = MySingle.useProduct();

  const handleLogin = () => {
    const newUser: User = {
      id: 1,
      name: '홍길동',
      email: 'hong@example.com'
    };
    MySingle.setUser(newUser);
  };

  const handleViewProduct = () => {
    const newProduct: Product = {
      id: 101,
      name: '아이폰 15 Pro',
      imgUrl: 'https://example.com/iphone15.jpg'
    };
    MySingle.setProduct(newProduct);
  };

  const handleLogout = () => {
    MySingle.clear();
  };

  return (
    <div className="p-8">
      {/* User 정보 */}
      <div className="mb-8 border-b pb-4">
        <h2 className="text-2xl font-bold mb-4">사용자 정보</h2>
        {user ? (
          <div>
            <p>ID: {user.id}</p>
            <p>이름: {user.name}</p>
            <p>이메일: {user.email}</p>
          </div>
        ) : (
          <p>로그인 필요</p>
        )}
        <button 
          onClick={handleLogin}
          className="mt-2 bg-blue-500 text-white px-4 py-2 rounded"
        >
          로그인
        </button>
      </div>

      {/* Product 정보 */}
      <div className="mb-8 border-b pb-4">
        <h2 className="text-2xl font-bold mb-4">현재 보고 있는 상품</h2>
        {product ? (
          <div className="flex items-center gap-4">
            <img 
              src={product.imgUrl} 
              alt={product.name}
              className="w-32 h-32 object-cover rounded"
            />
            <div>
              <p>ID: {product.id}</p>
              <p>상품명: {product.name}</p>
            </div>
          </div>
        ) : (
          <p>상품을 선택해주세요</p>
        )}
        <button 
          onClick={handleViewProduct}
          className="mt-2 bg-green-500 text-white px-4 py-2 rounded"
        >
          상품 보기
        </button>
      </div>

      {/* 초기화 */}
      <button 
        onClick={handleLogout}
        className="bg-red-500 text-white px-4 py-2 rounded"
      >
        전체 초기화
      </button>
    </div>
  );
}

export default App;
```

## 핵심 특징

### 1. **Singleton 패턴의 장점**
- 전역에서 하나의 인스턴스만 유지
- 일관된 인터페이스 제공
- 메모리 효율성

### 2. **Zustand의 장점**
- 간단한 API와 최소한의 보일러플레이트
- TypeScript 완벽 지원
- React DevTools 연동
- 미들웨어 지원 (persist, devtools 등)

### 3. **통합된 접근법**
- `useUser()`, `useProduct()`: React 컴포넌트에서 반응형 상태 사용
- `getUser()`, `getProduct()`: 비동기 함수나 이벤트 핸들러에서 현재 값 접근
- `setUser()`, `setProduct()`: 어디서나 상태 업데이트
- `clear()`: 전체 상태 초기화

### 4. **영속성 지원**
- `zustand/middleware`의 `persist`를 사용하여 localStorage에 자동 저장
- 페이지 새로고침 후에도 상태 유지

## 사용 시나리오

이 패턴은 다음과 같은 상황에서 특히 유용합니다:

- 사용자 로그인 정보 관리
- 현재 선택된 항목 추적
- 전역 설정값 관리
- 임시 데이터 캐싱

## 확장 가능성

필요에 따라 다음과 같이 확장할 수 있습니다:

```typescript
// 새로운 상태 추가
export class MySingle {
  // ... 기존 코드 ...

  // ========== Cart 관련 ==========
  
  public static getCart(): CartItem[] {
    return store.getState().cart;
  }

  public static addToCart(item: CartItem): void {
    const cart = store.getState().cart;
    store.getState().setCart([...cart, item]);
  }

  public static useCart() {
    return store((state) => state.cart);
  }
}
```

이 접근법을 통해 복잡한 상태 관리 라이브러리 없이도 효율적이고 타입 안전한 전역 상태 관리를 구현할 수 있습니다.