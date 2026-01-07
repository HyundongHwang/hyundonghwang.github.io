---
layout: post
title: "React forwardRef와 useImperativeHandle 완벽 가이드"
date: 2024-12-27
categories: [frontend, react]
tags: [react, forwardRef, useImperativeHandle, typescript, ref]
---

# React forwardRef와 useImperativeHandle 완벽 가이드

React에서 부모 컴포넌트가 자식 컴포넌트의 내부 메서드에 직접 접근해야 하는 경우가 있습니다. 이때 `forwardRef`와 `useImperativeHandle`을 함께 사용하면 깔끔하고 타입 안전한 해결책을 제공할 수 있습니다.

## 기본 구현 예제

### 비디오 컴포넌트 예제

```typescript
import { forwardRef, useRef, useImperativeHandle } from 'react';

// 타입 정의
interface MyVideo {
  play: () => void;
  pause: () => void;
}

// forwardRef 필수!
export const MyVideoComp = forwardRef<MyVideo>((props, ref) => {
  const _video = useRef<HTMLVideoElement>(null);

  useImperativeHandle(ref, () => ({
    play() {
      _video.current?.play();
    },
    pause() {
      _video.current?.pause();
    }
  }));

  return <video ref={_video} src="video.mp4" />;
});

// 사용
function MainPageComp() {
  const _myVideo = useRef<MyVideo>(null);

  const handlePlay = () => {
    _myVideo.current?.play();
  };

  const handlePause = () => {
    _myVideo.current?.pause();
  };

  return (
    <div>
      <MyVideoComp ref={_myVideo} />
      <button onClick={handlePlay}>재생</button>
      <button onClick={handlePause}>일시정지</button>
    </div>
  );
}
```

## 핵심 개념 설명

### 1. forwardRef

`forwardRef`는 부모 컴포넌트에서 전달받은 ref를 자식 컴포넌트 내부로 전달하는 HOC(Higher-Order Component)입니다.

```typescript
// forwardRef 사용법
const MyComponent = forwardRef<RefType, PropsType>((props, ref) => {
  // 컴포넌트 로직
  return <div>...</div>;
});
```

### 2. useImperativeHandle

`useImperativeHandle`은 ref로 노출할 메서드들을 정의하는 훅입니다. 이를 통해 부모 컴포넌트가 자식 컴포넌트의 특정 기능에만 접근할 수 있도록 제한할 수 있습니다.

```typescript
useImperativeHandle(ref, () => ({
  // 노출할 메서드들
  method1: () => { /* ... */ },
  method2: () => { /* ... */ },
}));
```

## 실무 활용 사례

### 1. 모달 컴포넌트

```typescript
interface ModalHandle {
  open: () => void;
  close: () => void;
  toggle: () => void;
}

interface ModalProps {
  children: React.ReactNode;
}

export const Modal = forwardRef<ModalHandle, ModalProps>(({ children }, ref) => {
  const [isOpen, setIsOpen] = useState(false);

  useImperativeHandle(ref, () => ({
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
    toggle: () => setIsOpen(prev => !prev),
  }));

  if (!isOpen) return null;

  return (
    <div className="modal-backdrop">
      <div className="modal-content">
        {children}
        <button onClick={() => setIsOpen(false)}>닫기</button>
      </div>
    </div>
  );
});

// 사용
function App() {
  const modalRef = useRef<ModalHandle>(null);

  return (
    <div>
      <button onClick={() => modalRef.current?.open()}>
        모달 열기
      </button>
      
      <Modal ref={modalRef}>
        <h2>모달 내용</h2>
        <p>여기에 내용을 넣으세요.</p>
      </Modal>
    </div>
  );
}
```

### 2. 입력 필드 컴포넌트

```typescript
interface InputHandle {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
  setValue: (value: string) => void;
}

interface InputProps {
  placeholder?: string;
  defaultValue?: string;
}

export const CustomInput = forwardRef<InputHandle, InputProps>(
  ({ placeholder, defaultValue }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);
    const [value, setValue] = useState(defaultValue || '');

    useImperativeHandle(ref, () => ({
      focus: () => inputRef.current?.focus(),
      clear: () => setValue(''),
      getValue: () => value,
      setValue: (newValue: string) => setValue(newValue),
    }));

    return (
      <input
        ref={inputRef}
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder={placeholder}
      />
    );
  }
);

// 사용
function Form() {
  const nameInputRef = useRef<InputHandle>(null);
  const emailInputRef = useRef<InputHandle>(null);

  const handleSubmit = () => {
    const name = nameInputRef.current?.getValue();
    const email = emailInputRef.current?.getValue();
    
    console.log('Name:', name);
    console.log('Email:', email);
  };

  const handleClearAll = () => {
    nameInputRef.current?.clear();
    emailInputRef.current?.clear();
  };

  return (
    <div>
      <CustomInput ref={nameInputRef} placeholder="이름" />
      <CustomInput ref={emailInputRef} placeholder="이메일" />
      
      <button onClick={handleSubmit}>제출</button>
      <button onClick={handleClearAll}>전체 지우기</button>
    </div>
  );
}
```

### 3. 애니메이션 컴포넌트

```typescript
interface AnimationHandle {
  play: () => void;
  pause: () => void;
  reset: () => void;
  reverse: () => void;
}

interface AnimationProps {
  duration?: number;
  children: React.ReactNode;
}

export const AnimatedBox = forwardRef<AnimationHandle, AnimationProps>(
  ({ duration = 1000, children }, ref) => {
    const [isPlaying, setIsPlaying] = useState(false);
    const [isReversed, setIsReversed] = useState(false);
    const elementRef = useRef<HTMLDivElement>(null);

    useImperativeHandle(ref, () => ({
      play: () => setIsPlaying(true),
      pause: () => setIsPlaying(false),
      reset: () => {
        setIsPlaying(false);
        setIsReversed(false);
      },
      reverse: () => {
        setIsReversed(true);
        setIsPlaying(true);
      },
    }));

    return (
      <div
        ref={elementRef}
        style={{
          transform: isPlaying 
            ? `translateX(${isReversed ? 0 : 100}px)` 
            : 'translateX(0)',
          transition: `transform ${duration}ms ease-in-out`,
        }}
      >
        {children}
      </div>
    );
  }
);

// 사용
function AnimationDemo() {
  const animationRef = useRef<AnimationHandle>(null);

  return (
    <div>
      <AnimatedBox ref={animationRef}>
        <div className="animated-content">애니메이션 박스</div>
      </AnimatedBox>
      
      <div className="controls">
        <button onClick={() => animationRef.current?.play()}>재생</button>
        <button onClick={() => animationRef.current?.pause()}>일시정지</button>
        <button onClick={() => animationRef.current?.reset()}>리셋</button>
        <button onClick={() => animationRef.current?.reverse()}>역재생</button>
      </div>
    </div>
  );
}
```

## 베스트 프랙티스

### 1. 타입 안전성 확보

```typescript
// 명확한 인터페이스 정의
interface ComponentHandle {
  method1: () => void;
  method2: (param: string) => string;
}

// forwardRef에 타입 명시
const Component = forwardRef<ComponentHandle, ComponentProps>(
  (props, ref) => {
    // 구현
  }
);
```

### 2. 메서드 제한

DOM 요소의 모든 메서드를 노출하지 말고, 필요한 메서드만 선별적으로 노출하세요.

```typescript
// ❌ 나쁜 예: 모든 DOM 메서드 노출
useImperativeHandle(ref, () => inputRef.current);

// ✅ 좋은 예: 필요한 메서드만 노출
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current?.focus(),
  clear: () => setValue(''),
}));
```

### 3. null 체크

ref를 사용할 때 항상 null 체크를 수행하세요.

```typescript
const handleClick = () => {
  // ref가 null일 수 있으므로 optional chaining 사용
  myComponentRef.current?.method();
};
```

### 4. cleanup 함수 고려

복잡한 로직이 있는 경우 cleanup 함수를 제공하는 것을 고려하세요.

```typescript
useImperativeHandle(ref, () => ({
  start: () => { /* 시작 로직 */ },
  stop: () => { /* 정지 로직 */ },
  cleanup: () => { /* 정리 로직 */ },
}));
```

## 주의사항

1. **남용 금지**: `useImperativeHandle`은 꼭 필요한 경우에만 사용하세요. 대부분의 경우 props와 콜백으로 충분합니다.

2. **선언적 패턴 우선**: React의 선언적 패턴을 먼저 고려하고, 명령형 접근이 정말 필요한 경우에만 사용하세요.

3. **컴포넌트 격리**: ref를 통해 노출하는 메서드는 해당 컴포넌트의 책임 범위 내에서만 정의하세요.

## 언제 사용해야 할까?

- 포커스 관리 (input, modal 등)
- 애니메이션 제어
- 스크롤 위치 제어
- 미디어 재생 제어 (audio, video)
- 써드파티 라이브러리 래핑

`forwardRef`와 `useImperativeHandle`을 올바르게 사용하면 React의 선언적 특성을 해치지 않으면서도 필요한 명령형 API를 제공할 수 있습니다.