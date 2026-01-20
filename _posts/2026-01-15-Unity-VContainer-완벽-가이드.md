---
layout: post
title: 260115 Unity VContainer 완벽 가이드
comments: true
tags:
- Unity
- VContainer
- DependencyInjection
- Zenject
- LifetimeScope
- Factory
- Architecture
- EntryPoint
- MonoBehaviour
---

{"title":"260115 Unity VContainer 완벽 가이드"}

# 🎯 Unity VContainer 완벽 가이드

> **VContainer**는 Unity를 위한 초고속, 최소 코드 크기, GC-Free 의존성 주입(DI) 라이브러리입니다. Zenject 대비 5-10배 빠른 성능과 더 작은 IL2CPP 바이너리 크기를 자랑하며, 현대적인 Unity 게임 아키텍처를 위한 필수 도구입니다.
---

# 📚 목차

1. [의존성 주입(DI) 개념](#1-의존성-주입di-개념)
2. [VContainer란 무엇인가](#2-vcontainer란-무엇인가)
3. [설치 및 설정](#3-설치-및-설정)
4. [기본 사용법](#4-기본-사용법)
5. [등록(Registration) 완전 정복](#5-등록registration-완전-정복)
6. [해결(Resolution) 및 주입(Injection)](#6-해결resolution-및-주입injection)
7. [LifetimeScope와 스코프 관리](#7-lifetimescope와-스코프-관리)
8. [EntryPoint와 Unity 라이프사이클](#8-entrypoint와-unity-라이프사이클)
9. [Factory 패턴과 동적 생성](#9-factory-패턴과-동적-생성)
10. [MonoBehaviour 통합](#10-monobehaviour-통합)
11. [고급 기능](#11-고급-기능)
12. [아키텍처 패턴](#12-아키텍처-패턴)
13. [성능 최적화](#13-성능-최적화)
14. [VContainer vs Zenject 비교](#14-vcontainer-vs-zenject-비교)
15. [실전 프로젝트 예제](#15-실전-프로젝트-예제)
16. [트러블슈팅](#16-트러블슈팅)
17. [FAQ](#17-faq)
18. [참고 자료](#18-참고-자료)
---

# 1. 의존성 주입(DI) 개념 💡

## 1.1 의존성 주입이란?

**의존성 주입(Dependency Injection, DI)**은 객체 지향 프로그래밍에서 **객체 간의 의존 관계를 외부에서 설정**하는 디자인 패턴입니다.

### ❌ DI 없는 코드 (강한 결합)

```
    public class PlayerHealth
    {
        private AudioManager _audioManager;
        public PlayerHealth()
        {
            // 직접 생성 - 강한 결합
            _audioManager = GameObject.Find("AudioManager").GetComponent<AudioManager>();
        }
        public void TakeDamage(int damage)
        {
            _audioManager.PlaySound("hurt");
        }
    }
```

**문제점:**
- PlayerHealth가 AudioManager의 구체적인 생성 방식에 의존
- 테스트가 어려움 (AudioManager를 Mock으로 교체 불가)
- GameObject.Find는 성능 문제 발생
- AudioManager 변경 시 PlayerHealth도 수정 필요

### ✅ DI 적용 코드 (느슨한 결합)

```
    public class PlayerHealth
    {
        private readonly IAudioService _audioService;
        // 생성자 주입
        public PlayerHealth(IAudioService audioService)
        {
            _audioService = audioService;
        }
        public void TakeDamage(int damage)
        {
            _audioService.PlaySound("hurt");
        }
    }
```

**장점:**
- PlayerHealth는 IAudioService 인터페이스에만 의존
- 테스트 시 Mock 객체로 쉽게 교체 가능
- 객체 생성 책임이 외부(DI 컨테이너)로 이동
- 코드 재사용성 향상

## 1.2 의존성 주입의 장점

### 🎯 테스트 용이성

```
    // 테스트 코드
    [Test]
    public void TakeDamage_ShouldPlayHurtSound()
    {
        // Arrange
        var mockAudio = new MockAudioService();
        var playerHealth = new PlayerHealth(mockAudio);
        // Act
        playerHealth.TakeDamage(10);
        // Assert
        Assert.IsTrue(mockAudio.WasPlaySoundCalled);
    }
```

### 🔄 유연성

```
    // 런타임에 다른 구현체 주입 가능
    builder.Register<IAudioService, SpatialAudioService>(Lifetime.Singleton);
    // 또는
    builder.Register<IAudioService, Simple2DAudioService>(Lifetime.Singleton);
```

### 📦 모듈화

```
    // 각 시스템이 독립적으로 동작
    public class GameInstaller : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // 오디오 시스템
            builder.Register<IAudioService, AudioManager>(Lifetime.Singleton);
            // 세이브 시스템
            builder.Register<ISaveService, CloudSaveManager>(Lifetime.Singleton);
            // 플레이어 시스템
            builder.Register<PlayerHealth>(Lifetime.Scoped);
        }
    }
```

## 1.3 DI의 핵심 원칙

### 제어의 역전(Inversion of Control, IoC)

- 객체의 생성과 생명주기 관리를 프레임워크(DI 컨테이너)가 담당
- 개발자는 "무엇을" 주입할지만 선언

### 의존성 역전 원칙(Dependency Inversion Principle, DIP)

- 상위 모듈이 하위 모듈에 의존하지 않음
- 둘 다 추상화(인터페이스)에 의존

```
    graph TB
        A[PlayerHealth] -->|의존| B[IAudioService]
        C[AudioManager] -.구현.-> B
        D[MockAudioService] -.구현.-> B
        style A fill:#e1f5ff
        style B fill:#fff4e1
        style C fill:#e8f5e9
        style D fill:#e8f5e9
```

---

# 2. VContainer란 무엇인가 🚀

## 2.1 VContainer 소개

**VContainer**는 hadashiA가 개발한 Unity 전용 DI 프레임워크로, 게임 개발에 최적화된 설계를 가지고 있습니다.

### 🎖️ 주요 특징

#### ⚡ 압도적인 성능

- **Zenject 대비 5-10배 빠른 Resolve 속도**
- **Zero GC Allocation** (인스턴스 생성 제외)
- **작은 IL2CPP 바이너리 크기**

#### 🧵 Thread-Safe

- **Immutable Container** 구조로 멀티스레딩 환경에서 안전

#### 🎮 Unity 통합

- MonoBehaviour 주입 지원
- Unity PlayerLoop 통합 (IStartable, ITickable 등)
- LifetimeScope 기반 씬 관리

#### 🛠️ 최적화 기능

- ILPostProcessor를 통한 코드 생성 (reflection 제거)
- Async Container Build (메인 스레드 블로킹 최소화)

## 2.2 VContainer의 철학

### 신중하게 선택된 기능

VContainer는 Zenject의 모든 기능을 포함하지 않습니다. 대신 **게임 개발에 정말 필요한 기능만 선별**하여 복잡도를 낮췄습니다.

```
    // ❌ VContainer는 조건부 바인딩을 지원하지 않음
    // Zenject: container.Bind<IFoo>().To<Foo>().WhenInjectedInto<Bar>();
    // ✅ VContainer는 명시적 등록을 권장
    builder.Register<IFoo, Foo>(Lifetime.Singleton);
```

### 데이터와 로직의 분리

- **DI 컨테이너는 로직(서비스)만 등록**
- 데이터 지향 객체(DTO, ValueObject)는 컨테이너에 등록하지 않음

```
    // ✅ 컨테이너에 등록
    builder.Register<PlayerController>(Lifetime.Scoped); // 로직
    // ❌ 컨테이너에 등록하지 않음
    var playerData = new PlayerData { hp = 100 }; // 데이터
```

## 2.3 언제 VContainer를 사용해야 하나?

### ✅ 적합한 경우

- 중대형 Unity 프로젝트 (여러 씬, 복잡한 시스템)
- 테스트 가능한 코드가 필요한 경우
- 팀 협업 프로젝트
- 성능이 중요한 게임 (모바일 등)
- 깔끔한 아키텍처를 원하는 경우

### ⚠️ 오버엔지니어링인 경우

- 매우 작은 프로토타입 (1-2씬)
- 혼자 만드는 간단한 게임
- DI 학습 비용이 부담스러운 팀
---

# 3. 설치 및 설정 🔧

## 3.1 Unity Package Manager를 통한 설치

### 방법 1: Git URL로 설치 (권장)

1. Unity 에디터에서 **Window \> Package Manager** 열기
2. 좌측 상단 **+** 버튼 클릭
3. **Add package from git URL** 선택
4. 다음 URL 입력:

```
```

### 방법 2: manifest.json 직접 수정

`Packages/manifest.json` 파일에 다음 추가:

```
    {
      "dependencies": {
        "jp.hadashikick.vcontainer": "
```

### 방법 3: OpenUPM 사용

```
    oppm add jp.hadashikick.vcontainer
```

## 3.2 프로젝트 구조 설정

VContainer를 사용하는 권장 프로젝트 구조:

```
    Assets/
    ├── Scripts/
    │   ├── Installers/          # LifetimeScope 스크립트
    │   │   ├── RootLifetimeScope.cs
    │   │   ├── GameLifetimeScope.cs
    │   │   └── BattleLifetimeScope.cs
    │   ├── Services/            # 싱글톤 서비스
    │   │   ├── IAudioService.cs
    │   │   ├── AudioManager.cs
    │   │   ├── ISaveService.cs
    │   │   └── SaveManager.cs
    │   ├── Entities/            # 게임 엔티티
    │   │   ├── Player/
    │   │   └── Enemy/
    │   ├── Presenters/          # EntryPoint 클래스
    │   │   ├── GamePresenter.cs
    │   │   └── UIPresenter.cs
    │   └── Views/               # MonoBehaviour UI/GameObject
    │       └── PlayerView.cs
    ├── Scenes/
    │   ├── Root.unity           # RootLifetimeScope
    │   ├── Game.unity           # GameLifetimeScope
    │   └── Battle.unity         # BattleLifetimeScope
```

## 3.3 첫 번째 LifetimeScope 생성

### Step 1: RootLifetimeScope 생성

```
    using VContainer;
    using VContainer.Unity;
    using UnityEngine;
    public class RootLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // 전역 싱글톤 서비스 등록
            builder.Register<AudioManager>(Lifetime.Singleton)
                .AsImplementedInterfaces()
                .AsSelf();
            Debug.Log("RootLifetimeScope configured");
        }
    }
```

### Step 2: Root 씬에 추가

1. 새로운 GameObject 생성 (이름: `RootLifetimeScope`)
2. `RootLifetimeScope` 컴포넌트 추가
3. Inspector에서 **Root Lifetime Scope** 체크박스 활성화

### Step 3: VContainerSettings 설정 (선택사항)

전역 설정을 위해 `VContainerSettings` ScriptableObject 생성:
1. **Assets \> Create \> VContainer \> VContainer Settings**
2. `Root Lifetime Scope` 필드에 위에서 만든 Prefab 할당
---

# 4. 기본 사용법 🎓

## 4.1 Hello World 예제

가장 간단한 VContainer 사용 예제입니다.

### Step 1: 서비스 클래스 작성

```
    using UnityEngine;
    // 인터페이스 정의
    public interface IGreetingService
    {
        void Greet(string name);
    }
    // 구현 클래스
    public class GreetingService : IGreetingService
    {
        public void Greet(string name)
        {
            Debug.Log($"Hello, {name}!");
        }
    }
```

### Step 2: EntryPoint 작성

```
    using VContainer.Unity;
    // EntryPoint는 게임 시작 시 자동 실행됨
    public class GameEntryPoint : IStartable
    {
        private readonly IGreetingService _greetingService;
        // 생성자 주입
        public GameEntryPoint(IGreetingService greetingService)
        {
            _greetingService = greetingService;
        }
        // Unity의 Start와 동일한 타이밍
        public void Start()
        {
            _greetingService.Greet("VContainer");
        }
    }
```

### Step 3: LifetimeScope에서 등록

```
    using VContainer;
    using VContainer.Unity;
    public class GameLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // 서비스 등록
            builder.Register<IGreetingService, GreetingService>(Lifetime.Singleton);
            // EntryPoint 등록
            builder.RegisterEntryPoint<GameEntryPoint>();
        }
    }
```

### Step 4: 실행

1. 씬에 GameObject 생성 후 `GameLifetimeScope` 추가
2. Play 버튼 클릭
3. Console에 "Hello, VContainer!" 출력 확인

## 4.2 기본 개념 정리

### 🔹 IContainerBuilder

- 의존성을 **등록(Register)**하는 인터페이스
- `Configure` 메서드에서 사용

### 🔹 IObjectResolver

- 등록된 의존성을 **해결(Resolve)**하는 인터페이스
- 일반적으로 직접 사용하지 않음 (자동 주입)

### 🔹 Lifetime

- 객체의 **생명주기** 정의
- `Singleton`, `Transient`, `Scoped`

### 🔹 EntryPoint

- Unity 라이프사이클에 통합되는 **Pure C# 클래스**
- MonoBehaviour 없이 Start, Update 등 사용 가능
---

# 5. 등록(Registration) 완전 정복 📝

## 5.1 기본 등록 메서드

### Register\<TConcrete\>

가장 기본적인 등록 방법:

```
    builder.Register<PlayerController>(Lifetime.Scoped);
```

- `PlayerController` 타입으로 등록 및 해결
- 생성자의 매개변수는 자동으로 주입됨

### Register\<TInterface, TImplementation\>

인터페이스와 구현 분리:

```
    builder.Register<IAudioService, AudioManager>(Lifetime.Singleton);
```

- `IAudioService` 타입으로 요청 시 `AudioManager` 인스턴스 반환
- 느슨한 결합 구현

### RegisterInstance

이미 생성된 인스턴스 등록:

```
    var config = new GameConfig { maxPlayers = 10 };
    builder.RegisterInstance(config);
```

- 주로 ScriptableObject나 설정 객체 등록 시 사용
- 항상 Singleton처럼 동작

## 5.2 Lifetime 완전 이해

### Singleton

```
    builder.Register<GameManager>(Lifetime.Singleton);
```

- **컨테이너당 하나의 인스턴스**만 생성
- 첫 Resolve 시 생성, 이후 재사용
- 컨테이너가 Dispose될 때 함께 Dispose
**사용 사례:**
- 전역 매니저 (AudioManager, InputManager)
- 게임 설정 (GameConfig)
- 네트워크 클라이언트

### Transient

```
    builder.Register<Bullet>(Lifetime.Transient);
```

- **Resolve할 때마다 새 인스턴스 생성**
- 가장 짧은 생명주기
- Dispose 책임은 사용자에게 있음
**사용 사례:**
- 일시적인 객체 (Bullet, Particle)
- 상태를 공유하지 않는 객체

### Scoped

```
    builder.Register<StageController>(Lifetime.Scoped);
```

- **LifetimeScope와 생명주기가 동일**
- 씬 로드 시 생성, 씬 언로드 시 Dispose
- Unity 게임에서 가장 많이 사용
**사용 사례:**
- 씬별 컨트롤러
- 현재 스테이지/레벨 관련 객체

### Lifetime 비교표

<table header-row="true">
<tr>
<td>Lifetime</td>
<td>생성 시점</td>
<td>파괴 시점</td>
<td>재사용</td>
<td>주요 용도</td>
</tr>
<tr>
<td>**Singleton**</td>
<td>첫 Resolve</td>
<td>Container Dispose</td>
<td>✅</td>
<td>전역 매니저</td>
</tr>
<tr>
<td>**Transient**</td>
<td>매 Resolve</td>
<td>수동 관리</td>
<td>❌</td>
<td>일시 객체</td>
</tr>
<tr>
<td>**Scoped**</td>
<td>첫 Resolve</td>
<td>Scope Dispose</td>
<td>✅ (Scope 내)</td>
<td>씬 컨트롤러</td>
</tr>
</table>

## 5.3 인터페이스 다중 등록

### AsImplementedInterfaces

```
    public class AudioManager : IAudioService, IDisposable
    {
        // ...
    }
    builder.Register<AudioManager>(Lifetime.Singleton)
        .AsImplementedInterfaces(); // IAudioService와 IDisposable로 등록
```

### As\<TInterface\>

```
    builder.Register<AudioManager>(Lifetime.Singleton)
        .As<IAudioService>()
        .As<IDisposable>();
```

### AsSelf (명시적 타입 등록)

```
    builder.Register<AudioManager>(Lifetime.Singleton)
        .AsImplementedInterfaces() // 인터페이스로도 해결 가능
        .AsSelf();                 // AudioManager 타입으로도 해결 가능
```

## 5.4 고급 등록 기법

### RegisterBuildCallback

컨테이너 빌드 완료 후 초기화:

```
    builder.RegisterBuildCallback(container =>
    {
        var audioManager = container.Resolve<IAudioService>();
        audioManager.Initialize();
    });
```

### WithParameter

생성자 매개변수 직접 지정:

```
    builder.Register<PlayerController>(Lifetime.Scoped)
        .WithParameter("playerName", "Hero");
```

### Register 조건부 등록

```
    #if UNITY_EDITOR
    builder.Register<IAnalyticsService, DebugAnalytics>(Lifetime.Singleton);
    #else
    builder.Register<IAnalyticsService, ProductionAnalytics>(Lifetime.Singleton);
    #endif
```

## 5.5 Collection 등록

여러 구현체를 컬렉션으로 주입:

```
    // 등록
    builder.Register<IEffect, FireEffect>(Lifetime.Transient);
    builder.Register<IEffect, IceEffect>(Lifetime.Transient);
    builder.Register<IEffect, ThunderEffect>(Lifetime.Transient);
    // 주입
    public class EffectManager
    {
        private readonly IEnumerable<IEffect> _effects;
        public EffectManager(IEnumerable<IEffect> effects)
        {
            _effects = effects; // 모든 IEffect 구현체가 주입됨
        }
    }
```

---

# 6. 해결(Resolution) 및 주입(Injection) 💉

## 6.1 생성자 주입 (Constructor Injection) - 권장

가장 권장되는 주입 방식입니다.

```
    public class PlayerController
    {
        private readonly IInputService _inputService;
        private readonly IAudioService _audioService;
        // 생성자 주입 - [Inject] 속성 불필요
        public PlayerController(
            IInputService inputService,
            IAudioService audioService)
        {
            _inputService = inputService;
            _audioService = audioService;
        }
    }
```

**장점:**
- ✅ 불변성 보장 (readonly)
- ✅ 명시적 의존성 표현
- ✅ 테스트 용이
- ✅ NullReferenceException 방지

## 6.2 메서드 주입 (Method Injection)

특정 메서드에 의존성 주입:

```
    public class DataProcessor
    {
        [Inject]
        public void Construct(ILogger logger, IConfig config)
        {
            // 초기화 로직
        }
    }
```

**사용 시기:**
- 생성자 주입이 불가능한 경우 (MonoBehaviour)
- 선택적 의존성

## 6.3 프로퍼티/필드 주입

```
    public class LegacySystem
    {
        [Inject]
        public IAudioService AudioService { get; set; }
        [Inject]
        private ILogger _logger;
    }
```

**주의:**
- ⚠️ nullable 가능성
- ⚠️ 의존성이 명확하지 않음
- ⚠️ 권장하지 않음 (최후의 수단)

## 6.4 IObjectResolver를 사용한 수동 Resolve

일반적으로 자동 주입을 사용하지만, 특수한 경우 수동 Resolve:

```
    public class EnemyFactory
    {
        private readonly IObjectResolver _resolver;
        public EnemyFactory(IObjectResolver resolver)
        {
            _resolver = resolver;
        }
        public Enemy CreateEnemy(EnemyType type)
        {
            return type switch
            {
                EnemyType.Goblin => _resolver.Resolve<Goblin>(),
                EnemyType.Orc => _resolver.Resolve<Orc>(),
                _ => throw new ArgumentException()
            };
        }
    }
```

**주의:**
- ⚠️ Service Locator 안티패턴에 가까움
- ✅ Factory 패턴에서는 예외적으로 허용
---

# 7. LifetimeScope와 스코프 관리 🌳

## 7.1 LifetimeScope 계층 구조

VContainer는 **부모-자식 관계**의 스코프 계층을 지원합니다.

```
    RootLifetimeScope (DontDestroyOnLoad)
    ├── TitleLifetimeScope (Title Scene)
    ├── GameLifetimeScope (Game Scene)
    │   ├── StageLifetimeScope (Stage Prefab)
    │   └── BossLifetimeScope (Boss Prefab)
    └── SettingsLifetimeScope (Settings Scene)
```

### 해결 규칙

1. **현재 스코프**에서 먼저 검색
2. 없으면 **부모 스코프**에서 검색
3. 재귀적으로 Root까지 탐색

```
    // RootLifetimeScope
    builder.Register<IAudioService, AudioManager>(Lifetime.Singleton);
    // GameLifetimeScope (자식)
    builder.Register<GameController>(Lifetime.Scoped);
    // GameController는 부모의 IAudioService 주입 가능
    public class GameController
    {
        public GameController(IAudioService audioService) // ✅ Root에서 해결
        {
        }
    }
```

## 7.2 RootLifetimeScope 설정

전역 싱글톤 서비스를 등록하는 최상위 스코프:

```
    using VContainer;
    using VContainer.Unity;
    public class RootLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // DontDestroyOnLoad 싱글톤
            builder.Register<AudioManager>(Lifetime.Singleton)
                .AsImplementedInterfaces();
            builder.Register<SaveManager>(Lifetime.Singleton)
                .AsImplementedInterfaces();
            builder.Register<NetworkClient>(Lifetime.Singleton)
                .AsImplementedInterfaces();
        }
        protected override void Awake()
        {
            base.Awake();
            IsRoot = true; // Root로 설정
            DontDestroyOnLoad(gameObject);
        }
    }
```

## 7.3 자식 스코프 생성

### 방법 1: 씬에 LifetimeScope 배치

```
    // GameScene에 GameLifetimeScope GameObject 배치
    public class GameLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            builder.Register<GameController>(Lifetime.Scoped);
            builder.Register<UIManager>(Lifetime.Scoped);
            builder.RegisterEntryPoint<GameEntryPoint>();
        }
    }
```

- Inspector에서 **Parent** 필드에 Root 할당 (자동 감지됨)

### 방법 2: 코드로 동적 생성

```
    public class StageLoader
    {
        private readonly LifetimeScope _parentScope;
        public StageLoader(LifetimeScope parentScope)
        {
            _parentScope = parentScope;
        }
        public void LoadStage(int stageId)
        {
            // 자식 스코프 생성
            var childScope = _parentScope.CreateChild(builder =>
            {
                builder.RegisterInstance(new StageData { id = stageId });
                builder.Register<StageController>(Lifetime.Scoped);
            });
        }
    }
```

### 방법 3: Prefab으로 생성

```
    public class BossSpawner
    {
        private readonly LifetimeScope _parentScope;
        [SerializeField]
        private LifetimeScope _bossLifetimeScopePrefab;
        public void SpawnBoss()
        {
            var bossScope = _parentScope.CreateChildFromPrefab(_bossLifetimeScopePrefab);
        }
    }
```

## 7.4 씬 로딩과 LifetimeScope

### EnqueueParent를 사용한 씬 로딩

```
    using UnityEngine.SceneManagement;
    using VContainer.Unity;
    public class SceneLoader
    {
        private readonly LifetimeScope _currentScope;
        public SceneLoader(LifetimeScope currentScope)
        {
            _currentScope = currentScope;
        }
        public async void LoadGameScene()
        {
            // 씬 로드 전에 부모 스코프 큐에 추가
            LifetimeScope.EnqueueParent(_currentScope);
            await SceneManager.LoadSceneAsync("Game");
            // 로드된 씬의 LifetimeScope가 자동으로 _currentScope를 부모로 설정
        }
    }
```

## 7.5 Scope의 생명주기 관리

### IDisposable 구현

```
    public class ResourceManager : IDisposable
    {
        private List<Texture> _loadedTextures = new();
        public void LoadTexture(string path)
        {
            var texture = Resources.Load<Texture>(path);
            _loadedTextures.Add(texture);
        }
        // Scope Dispose 시 자동 호출
        public void Dispose()
        {
            foreach (var texture in _loadedTextures)
            {
                Resources.UnloadAsset(texture);
            }
            _loadedTextures.Clear();
        }
    }
```

### 수동 Scope Dispose

```
    LifetimeScope childScope = parentScope.CreateChild(builder => { });
    // 사용 후 수동 정리
    childScope.Dispose();
```

---

# 8. EntryPoint와 Unity 라이프사이클 🎮

## 8.1 EntryPoint란?

**EntryPoint**는 MonoBehaviour 없이 Unity의 라이프사이클 이벤트를 받을 수 있는 Pure C# 클래스입니다.

### 장점

- ✅ 도메인 로직과 프레젠테이션 분리
- ✅ 테스트 용이
- ✅ GameObject에 종속되지 않음

## 8.2 라이프사이클 인터페이스

### IStartable

```
    using VContainer.Unity;
    public class GameInitializer : IStartable
    {
        private readonly IAudioService _audioService;
        public GameInitializer(IAudioService audioService)
        {
            _audioService = audioService;
        }
        // Unity의 Start()와 동일한 타이밍
        public void Start()
        {
            _audioService.PlayBGM("title");
        }
    }
```

### ITickable

```
    public class PlayerInputHandler : ITickable
    {
        private readonly IInputService _inputService;
        public PlayerInputHandler(IInputService inputService)
        {
            _inputService = inputService;
        }
        // Unity의 Update()와 동일한 타이밍
        public void Tick()
        {
            if (_inputService.GetKeyDown(
```

### IFixedTickable

```
    public class PhysicsController : IFixedTickable
    {
        // Unity의 FixedUpdate()와 동일한 타이밍
        public void FixedTick()
        {
            // 물리 연산
        }
    }
```

### ILateTickable

```
    public class CameraController : ILateTickable
    {
        // Unity의 LateUpdate()와 동일한 타이밍
        public void LateTick()
        {
            // 카메라 추적
        }
    }
```

### IPostStartable

```
    public class UIInitializer : IPostStartable
    {
        // Start() 이후 실행
        public void PostStart()
        {
            // UI 초기화
        }
    }
```

### IPostLateTickable

```
    public class EffectController : IPostLateTickable
    {
        // LateUpdate() 이후 실행
        public void PostLateTick()
        {
            // 후처리 효과
        }
    }
```

### IAsyncStartable

```
    using Cysharp.Threading.Tasks;
    public class ResourceLoader : IAsyncStartable
    {
        public async UniTask StartAsync(CancellationToken cancellation)
        {
            // 비동기 초기화
            await LoadResourcesAsync();
        }
    }
```

### IDisposable

```
    using System;
    public class ConnectionManager : IDisposable
    {
        // Scope Dispose 시 호출
        public void Dispose()
        {
            // 정리 작업
        }
    }
```

## 8.3 EntryPoint 등록

```
    public class GameLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // 방법 1: RegisterEntryPoint
            builder.RegisterEntryPoint<GameInitializer>();
            // 방법 2: Register + As<Interface>
            builder.Register<PlayerInputHandler>(Lifetime.Singleton)
                .As<ITickable>();
            // 방법 3: 여러 인터페이스 구현
            builder.RegisterEntryPoint<GamePresenter>(); // IStartable, ITickable 모두 구현
        }
    }
```

## 8.4 EntryPoint 실행 순서

```
    Container Build
    ↓
    IStartable.Start()
    ↓
    IPostStartable.PostStart()
    ↓
    [매 프레임]
    ├── ITickable.Tick()
    ├── IFixedTickable.FixedTick()
    ├── ILateTickable.LateTick()
    └── IPostLateTickable.PostLateTick()
    ↓
    Container Dispose
    ↓
    IDisposable.Dispose()
```

## 8.5 EntryPoint 실전 예제

### 게임 진행 관리

```
    using VContainer.Unity;
    using Cysharp.Threading.Tasks;
    public class GameFlowController : IAsyncStartable, ITickable, IDisposable
    {
        private readonly IUIService _uiService;
        private readonly IStageService _stageService;
        private readonly IAudioService _audioService;
        private GameState _currentState;
        public GameFlowController(
            IUIService uiService,
            IStageService stageService,
            IAudioService audioService)
        {
            _uiService = uiService;
            _stageService = stageService;
            _audioService = audioService;
        }
        // 게임 초기화
        public async UniTask StartAsync(CancellationToken cancellation)
        {
            await _uiService.ShowLoadingScreen();
            await _stageService.LoadStage(1);
            await _uiService.HideLoadingScreen();
            _audioService.PlayBGM("stage1");
            _currentState = GameState.Playing;
        }
        // 매 프레임 게임 상태 확인
        public void Tick()
        {
            switch (_currentState)
            {
                case GameState.Playing:
                    if (_stageService.IsCleared())
                    {
                        OnStageCleared();
                    }
                    break;
                case GameState.Paused:
                    // ...
                    break;
            }
        }
        private void OnStageCleared()
        {
            _uiService.ShowVictoryScreen();
            _currentState = GameState.Cleared;
        }
        // 정리
        public void Dispose()
        {
            _audioService.StopBGM();
        }
    }
```

---

# 9. Factory 패턴과 동적 생성 🏭

## 9.1 Factory가 필요한 이유

DI 컨테이너는 빌드 시점에 의존성 그래프를 구성합니다. 하지만 게임에서는 **런타임에 동적으로 객체를 생성**해야 하는 경우가 많습니다.

```
    // ❌ 런타임에 Resolve 불가
    public class EnemySpawner
    {
        private readonly IObjectResolver _resolver;
        public void SpawnEnemy()
        {
            // Anti-pattern: Service Locator
            var enemy = _resolver.Resolve<Enemy>();
        }
    }
    // ✅ Factory 패턴 사용
    public class EnemySpawner
    {
        private readonly Func<Enemy> _enemyFactory;
        public EnemySpawner(Func<Enemy> enemyFactory)
        {
            _enemyFactory = enemyFactory;
        }
        public void SpawnEnemy()
        {
            var enemy = _enemyFactory(); // 런타임에 생성
        }
    }
```

## 9.2 RegisterFactory 기본

### 의존성 없는 Factory

```
    // 등록
    builder.RegisterFactory<Enemy>(container =>
    {
        return () => new Enemy();
    });
    // 또는 간단하게
    builder.RegisterFactory<Enemy>(container => () => new Enemy());
    // 주입
    public class EnemySpawner
    {
        private readonly Func<Enemy> _enemyFactory;
        public EnemySpawner(Func<Enemy> enemyFactory)
        {
            _enemyFactory = enemyFactory;
        }
        public Enemy Spawn()
        {
            return _enemyFactory();
        }
    }
```

### 매개변수가 있는 Factory

```
    // 등록: int 매개변수를 받는 Factory
    builder.RegisterFactory<int, Enemy>(level =>
    {
        return new Enemy { level = level };
    });
    // 주입
    public class EnemySpawner
    {
        private readonly Func<int, Enemy> _enemyFactory;
        public EnemySpawner(Func<int, Enemy> enemyFactory)
        {
            _enemyFactory = enemyFactory;
        }
        public Enemy SpawnEnemy(int level)
        {
            return _enemyFactory(level); // 레벨을 전달하여 생성
        }
    }
```

### 여러 매개변수

```
    // 등록: 2개 매개변수
    builder.RegisterFactory<int, string, Enemy>((level, name) =>
    {
        return new Enemy { level = level, name = name };
    });
    // 주입
    private readonly Func<int, string, Enemy> _enemyFactory;
```

## 9.3 의존성이 있는 Factory

Factory 내부에서도 DI 사용:

```
    // Enemy가 IAudioService에 의존
    public class Enemy
    {
        private readonly IAudioService _audioService;
        public Enemy(IAudioService audioService)
        {
            _audioService = audioService;
        }
    }
    // 등록
    builder.RegisterFactory<int, Enemy>(container =>
    {
        var audioService = container.Resolve<IAudioService>();
        return level => new Enemy(audioService) { level = level };
    }, Lifetime.Scoped);
```

## 9.4 커스텀 Factory 클래스

복잡한 생성 로직은 전용 Factory 클래스로 분리:

```
    // Factory 인터페이스
    public interface IEnemyFactory
    {
        Enemy Create(EnemyType type, int level);
    }
    // Factory 구현
    public class EnemyFactory : IEnemyFactory
    {
        private readonly IObjectResolver _resolver;
        private readonly EnemyDatabase _database;
        public EnemyFactory(IObjectResolver resolver, EnemyDatabase database)
        {
            _resolver = resolver;
            _database = database;
        }
        public Enemy Create(EnemyType type, int level)
        {
            var data = _database.Get(type);
            // IObjectResolver를 사용하여 DI 지원
            return type switch
            {
                EnemyType.Goblin => _resolver.Instantiate<Goblin>(new object[] { data, level }),
                EnemyType.Orc => _resolver.Instantiate<Orc>(new object[] { data, level }),
                _ => throw new ArgumentException()
            };
        }
    }
    // 등록
    builder.Register<IEnemyFactory, EnemyFactory>(Lifetime.Singleton);
    // 사용
    public class EnemySpawner
    {
        private readonly IEnemyFactory _enemyFactory;
        public EnemySpawner(IEnemyFactory enemyFactory)
        {
            _enemyFactory = enemyFactory;
        }
        public void SpawnGoblin()
        {
            var goblin = _enemyFactory.Create(EnemyType.Goblin, 5);
        }
    }
```

## 9.5 GameObject Factory

GameObject/MonoBehaviour를 동적으로 생성하는 Factory:

```
    public interface IPrefabFactory<T> where T : MonoBehaviour
    {
        T Create(Transform parent = null);
    }
    public class PrefabFactory<T> : IPrefabFactory<T> where T : MonoBehaviour
    {
        private readonly IObjectResolver _resolver;
        private readonly T _prefab;
        public PrefabFactory(IObjectResolver resolver, T prefab)
        {
            _resolver = resolver;
            _prefab = prefab;
        }
        public T Create(Transform parent = null)
        {
            // IObjectResolver.Instantiate를 사용하여 DI 지원
            return _resolver.Instantiate(_prefab, parent);
        }
    }
    // 등록
    [SerializeField] private EnemyView _enemyPrefab;
    builder.Register<IPrefabFactory<EnemyView>>(container =>
    {
        return new PrefabFactory<EnemyView>(container, _enemyPrefab);
    }, Lifetime.Scoped);
    // 사용
    public class EnemySpawner
    {
        private readonly IPrefabFactory<EnemyView> _enemyViewFactory;
        private readonly Transform _spawnPoint;
        public EnemySpawner(IPrefabFactory<EnemyView> enemyViewFactory)
        {
            _enemyViewFactory = enemyViewFactory;
        }
        public void Spawn()
        {
            var enemyView = _enemyViewFactory.Create(_spawnPoint);
            // enemyView의 의존성도 자동 주입됨
        }
    }
```

## 9.6 Object Pool과 Factory 통합

```
    using UnityEngine.Pool;
    public class PooledFactory<T> where T : MonoBehaviour
    {
        private readonly IObjectResolver _resolver;
        private readonly T _prefab;
        private readonly ObjectPool<T> _pool;
        public PooledFactory(IObjectResolver resolver, T prefab, int capacity = 10)
        {
            _resolver = resolver;
            _prefab = prefab;
            _pool = new ObjectPool<T>(
                createFunc: () => _resolver.Instantiate(_prefab),
                actionOnGet: obj => obj.gameObject.SetActive(true),
                actionOnRelease: obj => obj.gameObject.SetActive(false),
                actionOnDestroy: obj => Object.Destroy(obj.gameObject),
                defaultCapacity: capacity
            );
        }
        public T Get() => _pool.Get();
        public void Release(T obj) => _pool.Release(obj);
    }
```

---

# 10. MonoBehaviour 통합 🎨

## 10.1 MonoBehaviour에 의존성 주입

MonoBehaviour는 생성자를 사용할 수 없으므로 **Method Injection** 사용:

```
    using UnityEngine;
    using VContainer;
    public class PlayerView : MonoBehaviour
    {
        private IInputService _inputService;
        private IAudioService _audioService;
        // [Inject] 속성으로 메서드 주입
        [Inject]
        public void Construct(IInputService inputService, IAudioService audioService)
        {
            _inputService = inputService;
            _audioService = audioService;
        }
        private void Update()
        {
            if (_inputService.GetKeyDown(
```

## 10.2 MonoBehaviour 등록 방법

### RegisterComponentInHierarchy

씬에 이미 존재하는 MonoBehaviour 등록:

```
    public class GameLifetimeScope : LifetimeScope
    {
        [SerializeField] private PlayerView _playerView;
        protected override void Configure(IContainerBuilder builder)
        {
            // Hierarchy의 특정 인스턴스 등록
            builder.RegisterComponent(_playerView);
            // 또는 타입으로 검색
            builder.RegisterComponentInHierarchy<UIManager>();
        }
    }
```

### RegisterComponentOnNewGameObject

새 GameObject를 생성하여 컴포넌트 추가:

```
    builder.RegisterComponentOnNewGameObject<GameManager>(
        Lifetime.Scoped,
        "GameManagerObject"
    );
```

### RegisterComponentInNewPrefab

Prefab을 인스턴스화하여 등록:

```
    public class GameLifetimeScope : LifetimeScope
    {
        [SerializeField] private PlayerView _playerPrefab;
        protected override void Configure(IContainerBuilder builder)
        {
            builder.RegisterComponentInNewPrefab(_playerPrefab, Lifetime.Scoped);
        }
    }
```

### RegisterComponentFromInstance

이미 생성된 인스턴스 등록:

```
    var playerView = Instantiate(_playerPrefab);
    builder.RegisterComponentFromInstance(playerView);
```

## 10.3 IObjectResolver.Instantiate

런타임에 Prefab을 인스턴스화하며 DI 적용:

```
    public class UIFactory
    {
        private readonly IObjectResolver _resolver;
        [SerializeField] private UIPanel _panelPrefab;
        public UIFactory(IObjectResolver resolver)
        {
            _resolver = resolver;
        }
        public UIPanel CreatePanel()
        {
            // Instantiate + 의존성 주입
            var panel = _resolver.Instantiate(_panelPrefab);
            return panel;
        }
    }
```

### InjectGameObject로 GameObject 전체 주입

```
    var instance = Instantiate(_prefab);
    // GameObject의 모든 컴포넌트에 주입
    _resolver.InjectGameObject(instance);
```

## 10.4 MonoBehaviour와 Pure C# 분리 패턴 (권장)

View(MonoBehaviour)와 Presenter(Pure C#) 분리:

### View (MonoBehaviour)

```
    using UnityEngine;
    using UnityEngine.UI;
    public class PlayerHealthView : MonoBehaviour
    {
        [SerializeField] private Slider _healthBar;
        public void UpdateHealth(float ratio)
        {
            _healthBar.value = ratio;
        }
    }
```

### Model (Pure C#)

```
    using System;
    public class PlayerHealthModel
    {
        public event Action<int> OnHealthChanged;
        private int _currentHealth;
        private int _maxHealth;
        public PlayerHealthModel(int maxHealth)
        {
            _maxHealth = maxHealth;
            _currentHealth = maxHealth;
        }
        public void TakeDamage(int damage)
        {
            _currentHealth = Mathf.Max(0, _currentHealth - damage);
            OnHealthChanged?.Invoke(_currentHealth);
        }
        public float HealthRatio => (float)_currentHealth / _maxHealth;
    }
```

### Presenter (Pure C#)

```
    using VContainer.Unity;
    public class PlayerHealthPresenter : IStartable, IDisposable
    {
        private readonly PlayerHealthModel _model;
        private readonly PlayerHealthView _view;
        public PlayerHealthPresenter(PlayerHealthModel model, PlayerHealthView view)
        {
            _model = model;
            _view = view;
        }
        public void Start()
        {
            _model.OnHealthChanged += OnHealthChanged;
            _view.UpdateHealth(_model.HealthRatio);
        }
        private void OnHealthChanged(int health)
        {
            _view.UpdateHealth(_model.HealthRatio);
        }
        public void Dispose()
        {
            _model.OnHealthChanged -= OnHealthChanged;
        }
    }
```

### 등록

```
    public class GameLifetimeScope : LifetimeScope
    {
        [SerializeField] private PlayerHealthView _playerHealthView;
        protected override void Configure(IContainerBuilder builder)
        {
            // Model
            builder.Register<PlayerHealthModel>(Lifetime.Scoped)
                .WithParameter(100); // maxHealth
            // View (MonoBehaviour)
            builder.RegisterComponent(_playerHealthView);
            // Presenter (EntryPoint)
            builder.RegisterEntryPoint<PlayerHealthPresenter>();
        }
    }
```

---

# 11. 고급 기능 🚀

## 11.1 Async Container Build

컨테이너 빌드는 Reflection 등으로 인해 시간이 걸릴 수 있습니다. **비동기 빌드**로 메인 스레드 블로킹을 최소화:

```
    using Cysharp.Threading.Tasks;
    using UnityEngine.SceneManagement;
    using VContainer.Unity;
    public class SceneLoader
    {
        public async UniTask LoadSceneAsync(string sceneName)
        {
            // 씬 로드
            await SceneManager.LoadSceneAsync(sceneName);
            // LifetimeScope 수동 빌드
            var lifetimeScope = FindObjectOfType<LifetimeScope>();
            if (lifetimeScope != null && !lifetimeScope.IsBuilt)
            {
                // 비동기 빌드 (백그라운드 스레드 활용)
                await lifetimeScope.BuildAsync();
            }
        }
    }
```

### AutoRun 비활성화

```
    public class ManualLifetimeScope : LifetimeScope
    {
        protected override void Awake()
        {
            base.Awake();
            autoRun = false; // 자동 빌드 비활성화
        }
        public async UniTask InitializeAsync()
        {
            await BuildAsync(); // 수동 비동기 빌드
        }
    }
```

## 11.2 Code Generation (ILPostProcessor)

Reflection 오버헤드를 제거하는 코드 생성:

### 설정 방법

1. **VContainerSettings** ScriptableObject 생성
2. **Code Gen** 탭에서 **Enable Code Generation** 활성화
3. 빌드 시 자동으로 IL 코드 생성

### 생성 대상

```
    // [Inject] 속성이 있는 클래스
    public class MyService
    {
        [Inject]
        public void Construct(IDependency dependency)
        {
            // 이 메서드는 코드 생성 대상
        }
    }
```

### 효과

- ✅ Reflection 제거로 **성능 향상**
- ✅ **IL2CPP 바이너리 크기 감소**
- ✅ AOT 환경(iOS 등)에서 안정성 향상

## 11.3 Immutable Container

VContainer의 컨테이너는 **Immutable(불변)**입니다.

```
    // ❌ 빌드 후 등록 불가
    container.Register<NewService>(Lifetime.Singleton); // 컴파일 에러
```

**장점:**
- ✅ Thread-Safe
- ✅ 예측 가능한 동작
- ✅ 성능 최적화 가능
**동적 등록이 필요한 경우:**
- 자식 스코프 생성 사용

```
    var childScope = parentScope.CreateChild(builder =>
    {
        builder.Register<NewService>(Lifetime.Scoped);
    });
```

## 11.4 Diagnostics

등록된 의존성을 시각적으로 확인:

```
    public class DiagnosticsLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            builder.Register<ServiceA>(Lifetime.Singleton);
            builder.Register<ServiceB>(Lifetime.Scoped);
            // 빌드 후 진단 정보 출력
            builder.RegisterBuildCallback(container =>
            {
                var diagnostics = container.Diagnostics;
                Debug.Log(diagnostics.GetRegistrationInfo());
            });
        }
    }
```

## 11.5 EntryPoint Exception Handling

EntryPoint에서 발생한 예외는 기본적으로 `Debug.LogException`으로 출력됩니다.

### 커스텀 예외 처리

```
    using VContainer;
    using VContainer.Unity;
    public class CustomLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // 커스텀 예외 핸들러 등록
            builder.RegisterEntryPointExceptionHandler(ex =>
            {
                // 크래시 리포팅 서비스에 전송
                CrashReporter.LogException(ex);
                // 사용자에게 에러 다이얼로그 표시
                ShowErrorDialog(ex.Message);
            });
            builder.RegisterEntryPoint<GameEntryPoint>();
        }
    }
```

---

# 12. 아키텍처 패턴 🏗️

## 12.1 Clean Architecture with VContainer

### 계층 구조

```
    ┌─────────────────────────────────────┐
    │   Presentation (View, Presenter)    │
    ├─────────────────────────────────────┤
    │   Application (Use Cases)           │
    ├─────────────────────────────────────┤
    │   Domain (Entities, Interfaces)     │
    ├─────────────────────────────────────┤
    │   Infrastructure (Services, Repo)   │
    └─────────────────────────────────────┘
```

### Domain Layer

```
    // Entities
    public class Player
    {
        public string Id { get; }
        public string Name { get; private set; }
        public int Level { get; private set; }
        public void LevelUp()
        {
            Level++;
        }
    }
    // Repository Interface
    public interface IPlayerRepository
    {
        Player Get(string id);
        void Save(Player player);
    }
```

### Application Layer

```
    // Use Case
    public class LevelUpPlayerUseCase
    {
        private readonly IPlayerRepository _playerRepository;
        public LevelUpPlayerUseCase(IPlayerRepository playerRepository)
        {
            _playerRepository = playerRepository;
        }
        public void Execute(string playerId)
        {
            var player = _playerRepository.Get(playerId);
            player.LevelUp();
            _
```

### Infrastructure Layer

```
    // Repository Implementation
    public class PlayerRepository : IPlayerRepository
    {
        private readonly ISaveService _saveService;
        public PlayerRepository(ISaveService saveService)
        {
            _saveService = saveService;
        }
        public Player Get(string id)
        {
            var data = _saveService.Load<PlayerData>(id);
            return new Player(data);
        }
        public void Save(Player player)
        {
            _
```

### Presentation Layer

```
    // Presenter
    public class PlayerPresenter : IStartable
    {
        private readonly LevelUpPlayerUseCase _levelUpUseCase;
        private readonly PlayerView _view;
        public PlayerPresenter(
            LevelUpPlayerUseCase levelUpUseCase,
            PlayerView view)
        {
            _levelUpUseCase = levelUpUseCase;
            _view = view;
        }
        public void Start()
        {
            _view.OnLevelUpButtonClicked += HandleLevelUp;
        }
        private void HandleLevelUp()
        {
            _levelUpUseCase.Execute("player1");
            _view.UpdateUI();
        }
    }
```

### DI 구성

```
    public class RootLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // Infrastructure
            builder.Register<ISaveService, SaveService>(Lifetime.Singleton);
            builder.Register<IPlayerRepository, PlayerRepository>(Lifetime.Singleton);
            // Application
            builder.Register<LevelUpPlayerUseCase>(Lifetime.Singleton);
            // Presentation
            builder.RegisterComponentInHierarchy<PlayerView>();
            builder.RegisterEntryPoint<PlayerPresenter>();
        }
    }
```

## 12.2 MVC with VContainer

### Model

```
    using System;
    public class GameModel
    {
        public event Action<int> OnScoreChanged;
        private int _score;
        public int Score
        {
            get => _score;
            set
            {
                _score = value;
                OnScoreChanged?.Invoke(_score);
            }
        }
    }
```

### View

```
    using UnityEngine;
    using UnityEngine.UI;
    public class GameView : MonoBehaviour
    {
        [SerializeField] private Text _scoreText;
        public void UpdateScore(int score)
        {
            _scoreText.text = $"Score: {score}";
        }
    }
```

### Controller

```
    using VContainer.Unity;
    public class GameController : IStartable, ITickable, IDisposable
    {
        private readonly GameModel _model;
        private readonly GameView _view;
        private readonly IInputService _inputService;
        public GameController(
            GameModel model,
            GameView view,
            IInputService inputService)
        {
            _model = model;
            _view = view;
            _inputService = inputService;
        }
        public void Start()
        {
            _model.OnScoreChanged += _view.UpdateScore;
            _view.UpdateScore(_model.Score);
        }
        public void Tick()
        {
            if (_inputService.GetKeyDown(
```

## 12.3 MessagePipe 통합 (Pub/Sub)

**MessagePipe**는 VContainer와 궁합이 좋은 메시징 라이브러리입니다.

### 설치

```
    "dependencies": {
        "com.cysharp.messagepipe": "
```

### 설정

```
    using MessagePipe;
    using VContainer;
    using VContainer.Unity;
    public class RootLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // MessagePipe 등록
            builder.RegisterMessagePipe();
            // Publisher/Subscriber 등록
            builder.RegisterEntryPoint<ScorePublisher>();
            builder.RegisterEntryPoint<ScoreSubscriber>();
        }
    }
```

### Publisher

```
    using MessagePipe;
    using VContainer.Unity;
    public class ScorePublisher : ITickable
    {
        private readonly IPublisher<ScoreChangedMessage> _publisher;
        private readonly IInputService _inputService;
        public ScorePublisher(
            IPublisher<ScoreChangedMessage> publisher,
            IInputService inputService)
        {
            _publisher = publisher;
            _inputService = inputService;
        }
        public void Tick()
        {
            if (_inputService.GetKeyDown(
```

### Subscriber

```
    using MessagePipe;
    using VContainer.Unity;
    public class ScoreSubscriber : IStartable, IDisposable
    {
        private readonly ISubscriber<ScoreChangedMessage> _subscriber;
        private readonly IDisposable _subscription;
        public ScoreSubscriber(ISubscriber<ScoreChangedMessage> subscriber)
        {
            _subscriber = subscriber;
        }
        public void Start()
        {
            _subscription = _subscriber.Subscribe(msg =>
            {
                Debug.Log($"Score changed: {msg.Score}");
            });
        }
        public void Dispose()
        {
            _subscription?.Dispose();
        }
    }
    public struct ScoreChangedMessage
    {
        public int Score;
    }
```

## 12.4 ECS (Entity Component System) 통합

VContainer는 Unity ECS나 Morpeh ECS와 통합 가능:

```
    using Scellecs.Morpeh;
    using VContainer;
    using VContainer.Unity;
    public class EcsLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // World 등록
            builder.RegisterInstance(World.Create());
            // System 등록
            builder.Register<PlayerSystem>(Lifetime.Singleton)
                .AsImplementedInterfaces();
            builder.RegisterEntryPoint<EcsInitializer>();
        }
    }
    public class EcsInitializer : IStartable
    {
        private readonly World _world;
        private readonly IEnumerable<ISystem> _systems;
        public EcsInitializer(World world, IEnumerable<ISystem> systems)
        {
            _world = world;
            _systems = systems;
        }
        public void Start()
        {
            foreach (var system in _systems)
            {
                _world.AddSystem(system);
            }
        }
    }
```

---

# 13. 성능 최적화 ⚡

## 13.1 성능 특징

VContainer는 이미 매우 빠르지만, 추가 최적화 가능:

### 벤치마크 (Zenject 대비)

<table header-row="true">
<tr>
<td>작업</td>
<td>VContainer</td>
<td>Zenject</td>
<td>비율</td>
</tr>
<tr>
<td>Resolve (Singleton)</td>
<td>**0.5μs**</td>
<td>4.2μs</td>
<td>**8.4x 빠름**</td>
</tr>
<tr>
<td>Resolve (Transient)</td>
<td>**1.2μs**</td>
<td>6.8μs</td>
<td>**5.7x 빠름**</td>
</tr>
<tr>
<td>Container Build</td>
<td>**12ms**</td>
<td>85ms</td>
<td>**7x 빠름**</td>
</tr>
<tr>
<td>GC Allocation</td>
<td>**0 byte**</td>
<td>240 byte</td>
<td>**Zero GC**</td>
</tr>
</table>

## 13.2 Code Generation 활성화

### 설정

1. **Assets \> Create \> VContainer \> VContainer Settings**
2. Inspector에서 **Enable Code Generation** 활성화
3. **Tools \> VContainer \> Generate Code** 실행

### 효과

- ✅ Reflection 제거
- ✅ IL2CPP 빌드 크기 감소 (\~10%)
- ✅ iOS/Android AOT 환경 안정성

## 13.3 Lifetime 선택 최적화

### Singleton 사용

```
    // ✅ 권장: 전역 서비스는 Singleton
    builder.Register<AudioManager>(Lifetime.Singleton);
```

- 한 번만 생성
- 캐싱으로 Resolve 속도 빠름

### Transient 최소화

```
    // ⚠️ 주의: 매번 생성하므로 성능 저하
    builder.Register<TemporaryObject>(Lifetime.Transient);
```

- GC Pressure 증가
- Pool 패턴 고려

### Scoped 활용

```
    // ✅ 권장: 씬별 객체는 Scoped
    builder.Register<StageController>(Lifetime.Scoped);
```

- 씬 단위 재사용
- 메모리 효율적

## 13.4 Async Container Build

```
    using Cysharp.Threading.Tasks;
    public class OptimizedLifetimeScope : LifetimeScope
    {
        protected override void Awake()
        {
            base.Awake();
            autoRun = false;
        }
        private async void Start()
        {
            // 비동기 빌드로 프레임 드롭 방지
            await BuildAsync();
            // 빌드 완료 후 게임 시작
            StartGame();
        }
    }
```

## 13.5 Container 재사용

```
    // ❌ 비효율: 씬마다 새 Container
    // 모든 씬에 LifetimeScope 배치
    // ✅ 효율: Root Container 재사용
    RootLifetimeScope (DontDestroyOnLoad)
    └── Child Scopes (씬별)
```

## 13.6 등록 최소화

```
    // ❌ 과도한 등록
    builder.Register<TinyHelper1>(Lifetime.Singleton);
    builder.Register<TinyHelper2>(Lifetime.Singleton);
    // ... 수백 개
    // ✅ 필요한 것만 등록
    // 작은 Helper는 static 메서드로 대체
    public static class MathHelper
    {
        public static float Lerp(float a, float b, float t) => a + (b - a) * t;
    }
```

## 13.7 IObjectResolver 사용 최소화

```
    // ❌ Service Locator 패턴 (느림)
    public class BadExample
    {
        private readonly IObjectResolver _resolver;
        public void DoSomething()
        {
            var service = _resolver.Resolve<IService>(); // 매번 Resolve
        }
    }
    // ✅ 생성자 주입 (빠름)
    public class GoodExample
    {
        private readonly IService _service;
        public GoodExample(IService service)
        {
            _service = service; // 한 번만 주입
        }
    }
```

---

# 14. VContainer vs Zenject 비교 ⚖️

## 14.1 성능 비교

<table header-row="true">
<tr>
<td>항목</td>
<td>VContainer</td>
<td>Zenject</td>
</tr>
<tr>
<td>Resolve 속도</td>
<td>⚡ **5-10배 빠름**</td>
<td>느림</td>
</tr>
<tr>
<td>GC Allocation</td>
<td>✅ **Zero**</td>
<td>매번 발생</td>
</tr>
<tr>
<td>IL2CPP 크기</td>
<td>✅ **작음**</td>
<td>큼</td>
</tr>
<tr>
<td>Thread Safety</td>
<td>✅ **Immutable**</td>
<td>제한적</td>
</tr>
<tr>
<td>학습 곡선</td>
<td>✅ **낮음**</td>
<td>높음</td>
</tr>
</table>

## 14.2 기능 비교

### VContainer에 있는 기능

✅ Unity PlayerLoop 통합 (IStartable, ITickable)  
✅ Async Container Build  
✅ Code Generation (ILPostProcessor)  
✅ Immutable Container  
✅ Simpler API  

### Zenject에만 있는 기능

❌ 조건부 바인딩 (`.WhenInjectedInto()`)  
❌ Decorator 패턴  
❌ Auto-Mocking  
❌ Signal (VContainer는 MessagePipe 권장)  
❌ Memory Pool (Unity ObjectPool 사용 권장)  

## 14.3 코드 비교

### 등록

```
    // Zenject
    Container.Bind<IFoo>().To<Foo>().AsSingle();
    Container.Bind<IBar>().FromInstance(bar).AsSingle();
    // VContainer
    builder.Register<IFoo, Foo>(Lifetime.Singleton);
    builder.RegisterInstance(bar);
```

### EntryPoint

```
    // Zenject
    public class GameController : IInitializable, ITickable
    {
        public void Initialize() { }
        public void Tick() { }
    }
    // VContainer
    public class GameController : IStartable, ITickable
    {
        public void Start() { } // IStartable
        public void Tick() { }
    }
```

### Factory

```
    // Zenject
    Container.BindFactory<Enemy, Enemy.Factory>().FromComponentInNewPrefab(enemyPrefab);
    // VContainer
    builder.RegisterFactory<Enemy>(container =>
        () => container.Instantiate(enemyPrefab)
    );
```

## 14.4 마이그레이션 가이드

### Zenject → VContainer 전환

1. **Bind → Register**

```
    // Zenject
    Container.Bind<IFoo>().To<Foo>().AsSingle();
    // VContainer
    builder.Register<IFoo, Foo>(Lifetime.Singleton);
```

1. **AsSingle/AsCached/AsTransient → Lifetime**

```
    // Zenject
    Container.Bind<IFoo>().To<Foo>().AsSingle();      // Singleton
    Container.Bind<IBar>().To<Bar>().AsCached();      // Scoped
    Container.Bind<IBaz>().To<Baz>().AsTransient();   // Transient
    // VContainer
    builder.Register<IFoo, Foo>(Lifetime.Singleton);
    builder.Register<IBar, Bar>(Lifetime.Scoped);
    builder.Register<IBaz, Baz>(Lifetime.Transient);
```

1. **IInitializable → IStartable**

```
    // Zenject
    public class GameInit : IInitializable
    {
        public void Initialize() { }
    }
    // VContainer
    public class GameInit : IStartable
    {
        public void Start() { }
    }
```

1. **SignalBus → MessagePipe**

```
    // Zenject
    Container.DeclareSignal<ScoreChangedSignal>();
    Container.BindSignal<ScoreChangedSignal>().ToMethod<ScoreUI>(x => x.OnScoreChanged);
    // VContainer + MessagePipe
    builder.RegisterMessagePipe();
    // Publisher: IPublisher<ScoreChangedMessage>
    // Subscriber: ISubscriber<ScoreChangedMessage>
```

## 14.5 어떤 것을 선택해야 하나?

### VContainer 선택

✅ 새 프로젝트  
✅ 성능이 중요한 게임 (모바일)  
✅ 간단하고 명확한 API 선호  
✅ IL2CPP 빌드 크기 최소화  
✅ 현대적인 Unity 개발  

### Zenject 선택

⚠️ 레거시 프로젝트 (이미 Zenject 사용 중)  
⚠️ 고급 기능 필요 (Decorator, Auto-Mock 등)  
⚠️ 마이그레이션 비용이 부담스러운 경우  
---

# 15. 실전 프로젝트 예제 🎮

## 15.1 간단한 슈팅 게임 아키텍처

### 프로젝트 구조

```
    Assets/Scripts/
    ├── Installers/
    │   ├── RootLifetimeScope.cs
    │   └── GameLifetimeScope.cs
    ├── Services/
    │   ├── IAudioService.cs
    │   ├── AudioService.cs
    │   ├── IInputService.cs
    │   └── InputService.cs
    ├── Entities/
    │   ├── Player.cs
    │   ├── Enemy.cs
    │   └── Bullet.cs
    ├── Factories/
    │   ├── IEnemyFactory.cs
    │   ├── EnemyFactory.cs
    │   ├── IBulletFactory.cs
    │   └── BulletFactory.cs
    ├── Presenters/
    │   ├── GamePresenter.cs
    │   └── UIPresenter.cs
    └── Views/
        ├── PlayerView.cs
        ├── EnemyView.cs
        └── UIView.cs
```

### RootLifetimeScope

```
    using VContainer;
    using VContainer.Unity;
    using UnityEngine;
    public class RootLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            // 전역 서비스
            builder.Register<IAudioService, AudioService>(Lifetime.Singleton);
            builder.Register<IInputService, InputService>(Lifetime.Singleton);
            Debug.Log("[VContainer] RootLifetimeScope configured");
        }
        protected override void Awake()
        {
            base.Awake();
            IsRoot = true;
            DontDestroyOnLoad(gameObject);
        }
    }
```

### GameLifetimeScope

```
    using VContainer;
    using VContainer.Unity;
    using UnityEngine;
    public class GameLifetimeScope : LifetimeScope
    {
        [SerializeField] private PlayerView _playerView;
        [SerializeField] private UIView _uiView;
        [SerializeField] private EnemyView _enemyPrefab;
        [SerializeField] private BulletView _bulletPrefab;
        protected override void Configure(IContainerBuilder builder)
        {
            // Entities
            builder.Register<Player>(Lifetime.Scoped);
            // Factories
            builder.Register<IEnemyFactory, EnemyFactory>(Lifetime.Scoped);
            builder.Register<IBulletFactory, BulletFactory>(Lifetime.Scoped);
            // Prefabs for Factories
            builder.RegisterInstance(_enemyPrefab);
            builder.RegisterInstance(_bulletPrefab);
            // Views
            builder.RegisterComponent(_playerView);
            builder.RegisterComponent(_uiView);
            // Presenters
            builder.RegisterEntryPoint<GamePresenter>();
            builder.RegisterEntryPoint<UIPresenter>();
            Debug.Log("[VContainer] GameLifetimeScope configured");
        }
    }
```

### Player Entity

```
    using System;
    public class Player
    {
        public event Action<int> OnHealthChanged;
        public event Action OnDied;
        private int _health;
        private int _maxHealth;
        public int Health => _health;
        public bool IsAlive => _health > 0;
        public Player(int maxHealth = 100)
        {
            _maxHealth = maxHealth;
            _health = maxHealth;
        }
        public void TakeDamage(int damage)
        {
            if (!IsAlive) return;
            _health = Mathf.Max(0, _health - damage);
            OnHealthChanged?.Invoke(_health);
            if (_health <= 0)
            {
                OnDied?.Invoke();
            }
        }
        public void Heal(int amount)
        {
            _health = Mathf.Min(_maxHealth, _health + amount);
            OnHealthChanged?.Invoke(_health);
        }
    }
```

### PlayerView (MonoBehaviour)

```
    using UnityEngine;
    using VContainer;
    public class PlayerView : MonoBehaviour
    {
        [SerializeField] private float _moveSpeed = 5f;
        private IInputService _inputService;
        private IAudioService _audioService;
        [Inject]
        public void Construct(IInputService inputService, IAudioService audioService)
        {
            _inputService = inputService;
            _audioService = audioService;
        }
        private void Update()
        {
            // 이동
            var moveDir = _inputService.GetMoveDirection();
            transform.position += moveDir * _moveSpeed * Time.deltaTime;
            // 공격
            if (_inputService.GetFireButton())
            {
                _audioService.PlaySound("shoot");
            }
        }
        public void PlayHitEffect()
        {
            _audioService.PlaySound("hit");
            // 피격 애니메이션
        }
    }
```

### GamePresenter (EntryPoint)

```
    using VContainer.Unity;
    using UnityEngine;
    public class GamePresenter : IStartable, ITickable, IDisposable
    {
        private readonly Player _player;
        private readonly PlayerView _playerView;
        private readonly IInputService _inputService;
        private readonly IBulletFactory _bulletFactory;
        private readonly IEnemyFactory _enemyFactory;
        private float _enemySpawnTimer;
        private const float EnemySpawnInterval = 2f;
        public GamePresenter(
            Player player,
            PlayerView playerView,
            IInputService inputService,
            IBulletFactory bulletFactory,
            IEnemyFactory enemyFactory)
        {
            _player = player;
            _playerView = playerView;
            _inputService = inputService;
            _bulletFactory = bulletFactory;
            _enemyFactory = enemyFactory;
        }
        public void Start()
        {
            _player.OnHealthChanged += OnPlayerHealthChanged;
            _player.OnDied += OnPlayerDied;
            Debug.Log("[Game] Started");
        }
        public void Tick()
        {
            // 총알 발사
            if (_inputService.GetFireButton())
            {
                FireBullet();
            }
            // 적 생성
            _enemySpawnTimer += Time.deltaTime;
            if (_enemySpawnTimer >= EnemySpawnInterval)
            {
                _enemySpawnTimer = 0f;
                SpawnEnemy();
            }
        }
        private void FireBullet()
        {
            var bulletPos = _playerView.transform.position + Vector3.up;
            _bulletFactory.Create(bulletPos, Vector3.up);
        }
        private void SpawnEnemy()
        {
            var spawnPos = new Vector3(Random.Range(-8f, 8f), 6f, 0f);
            _enemyFactory.Create(spawnPos);
        }
        private void OnPlayerHealthChanged(int health)
        {
            _playerView.PlayHitEffect();
            Debug.Log($"[Player] Health: {health}");
        }
        private void OnPlayerDied()
        {
            Debug.Log("[Player] Died - Game Over");
        }
        public void Dispose()
        {
            _player.OnHealthChanged -= OnPlayerHealthChanged;
            _player.OnDied -= OnPlayerDied;
        }
    }
```

### BulletFactory

```
    using UnityEngine;
    public interface IBulletFactory
    {
        BulletView Create(Vector3 position, Vector3 direction);
    }
    public class BulletFactory : IBulletFactory
    {
        private readonly IObjectResolver _resolver;
        private readonly BulletView _bulletPrefab;
        public BulletFactory(IObjectResolver resolver, BulletView bulletPrefab)
        {
            _resolver = resolver;
            _bulletPrefab = bulletPrefab;
        }
        public BulletView Create(Vector3 position, Vector3 direction)
        {
            var bullet = _resolver.Instantiate(_bulletPrefab);
            bullet.transform.position = position;
            bullet.Initialize(direction);
            return bullet;
        }
    }
```

## 15.2 멀티씬 게임 구조

### 씬 구성

```
    Scenes/
    ├── Root.unity          (RootLifetimeScope)
    ├── Title.unity         (TitleLifetimeScope)
    ├── Game.unity          (GameLifetimeScope)
    └── Result.unity        (ResultLifetimeScope)
```

### SceneLoader Service

```
    using Cysharp.Threading.Tasks;
    using UnityEngine.SceneManagement;
    using VContainer.Unity;
    public interface ISceneLoader
    {
        UniTask LoadSceneAsync(string sceneName);
    }
    public class SceneLoader : ISceneLoader
    {
        private readonly LifetimeScope _currentScope;
        public SceneLoader(LifetimeScope currentScope)
        {
            _currentScope = currentScope;
        }
        public async UniTask LoadSceneAsync(string sceneName)
        {
            // 다음 씬의 LifetimeScope가 현재 스코프를 부모로 설정
            LifetimeScope.EnqueueParent(_currentScope);
            await SceneManager.LoadSceneAsync(sceneName);
        }
    }
```

### TitleLifetimeScope

```
    using VContainer;
    using VContainer.Unity;
    public class TitleLifetimeScope : LifetimeScope
    {
        protected override void Configure(IContainerBuilder builder)
        {
            builder.Register<ISceneLoader, SceneLoader>(Lifetime.Scoped);
            builder.RegisterEntryPoint<TitlePresenter>();
        }
    }
```

### TitlePresenter

```
    using Cysharp.Threading.Tasks;
    using VContainer.Unity;
    public class TitlePresenter : IStartable
    {
        private readonly ISceneLoader _sceneLoader;
        private readonly IInputService _inputService;
        public TitlePresenter(ISceneLoader sceneLoader, IInputService inputService)
        {
            _sceneLoader = sceneLoader;
            _inputService = inputService;
        }
        public void Start()
        {
            WaitForStart().Forget();
        }
        private async UniTaskVoid WaitForStart()
        {
            await UniTask.WaitUntil(() => _inputService.GetKeyDown(
```

---

# 16. 트러블슈팅 🔧

## 16.1 NullReferenceException: 의존성이 주입되지 않음

### 증상

```
    public class MyService
    {
        private IDependency _dependency; // null!
        public void DoSomething()
        {
            _dependency.Execute(); // NullReferenceException!
        }
    }
```

### 원인

1. LifetimeScope에 등록하지 않음
2. MonoBehaviour에서 생성자 주입 시도
3. \[Inject\] 속성 누락

### 해결

```
    // 1. LifetimeScope에 등록
    builder.Register<IDependency, Dependency>(Lifetime.Singleton);
    // 2. MonoBehaviour는 메서드 주입 사용
    [Inject]
    public void Construct(IDependency dependency)
    {
        _dependency = dependency;
    }
    // 3. Pure C#은 생성자 주입 (속성 불필요)
    public MyService(IDependency dependency)
    {
        _dependency = dependency;
    }
```

## 16.2 Multiple Entry Points 호출

### 증상

MonoBehaviour의 EntryPoint 메서드가 여러 번 호출됨

### 원인

부모 스코프에 등록된 MonoBehaviour가 자식 스코프마다 호출됨

### 해결

```
    // ❌ 부모 스코프에 MonoBehaviour EntryPoint 등록
    // RootLifetimeScope
    builder.RegisterEntryPoint<MyMonoBehaviour>(); // 여러 번 호출됨
    // ✅ 해당 스코프에만 등록
    // GameLifetimeScope
    builder.RegisterEntryPoint<MyMonoBehaviour>(); // 한 번만 호출됨
```

## 16.3 Cannot Resolve Type

### 증상

```
    VContainerException: Cannot resolve type 'MyService'
```

### 원인

1. 등록되지 않은 타입
2. 잘못된 Lifetime
3. 자식 스코프에서 부모 스코프 타입 오버라이드

### 해결

```
    // 1. 등록 확인
    builder.Register<MyService>(Lifetime.Singleton);
    // 2. Lifetime 확인
    builder.Register<MyService>(Lifetime.Scoped); // Transient가 아닌 Scoped
    // 3. 부모 스코프 확인
    // 자식 스코프에서 같은 타입 재등록 시 부모의 것이 숨겨짐
```

## 16.4 Circular Dependency

### 증상

```
    VContainerException: Circular dependency detected: A -> B -> A
```

### 원인

```
    public class ServiceA
    {
        public ServiceA(ServiceB b) { } // A가 B에 의존
    }
    public class ServiceB
    {
        public ServiceB(ServiceA a) { } // B가 A에 의존 → 순환!
    }
```

### 해결

```
    // 방법 1: 인터페이스로 분리
    public interface IServiceA { }
    public interface IServiceB { }
    public class ServiceA : IServiceA
    {
        public ServiceA(IServiceB b) { }
    }
    public class ServiceB : IServiceB
    {
        public ServiceB(IServiceA a) { }
    }
    // 방법 2: 이벤트/메시지 시스템 사용
    // MessagePipe로 간접 통신
    // 방법 3: 구조 개선 (권장)
    // 공통 의존성을 별도 클래스로 추출
```

## 16.5 Container Already Built

### 증상

```
    InvalidOperationException: Container is already built
```

### 원인

빌드 후 등록 시도

### 해결

```
    // ❌ 빌드 후 등록 불가
    var container = 
```

## 16.6 IL2CPP Stripping 문제

### 증상

빌드 후 특정 타입이 제거되어 런타임 에러

### 해결

```
    // 방법 1: [Preserve] 속성 추가
    [Preserve]
    public class MyService
    {
        [Preserve]
        public MyService(IDependency dependency) { }
    }
    // 방법 2: Code Generation 활성화
    // VContainerSettings에서 Enable Code Generation 체크
    // 방법 3: link.xml 추가
    // <linker>
    //   <assembly fullname="YourAssembly" preserve="all"/>
    // </linker>
```

---

# 17. FAQ 💬

## Q1. VContainer vs Extenject?

**A:** Extenject는 Zenject의 커뮤니티 포크입니다. VContainer는 다음과 같은 이유로 더 나은 선택:
- ⚡ 5-10배 빠른 성능
- ✅ Zero GC Allocation
- ✅ 더 작은 IL2CPP 바이너리
- ✅ 현대적이고 간결한 API
- ✅ 활발한 유지보수

## Q2. 작은 프로젝트에도 필요한가?

**A:** 프로젝트 크기에 따라 다름:
- ✅ 권장: 3개 이상 씬, 여러 시스템, 팀 협업
- ⚠️ 선택: 1-2씬, 간단한 로직
- ❌ 불필요: 프로토타입, 게임잼

## Q3. MonoBehaviour를 완전히 대체할 수 있나?

**A:** 아니요. MonoBehaviour는 여전히 필요:
- ✅ View/Presentation 레이어에 사용
- ✅ Unity 에디터 연동 (Inspector, Prefab 등)
- ❌ 로직은 Pure C# EntryPoint로 분리

## Q4. UniTask/UniRx와 함께 사용 가능?

**A:** 완벽하게 호환됩니다.

```
    using Cysharp.Threading.Tasks;
    using VContainer.Unity;
    public class AsyncService : IAsyncStartable
    {
        public async UniTask StartAsync(CancellationToken cancellation)
        {
            await UniTask.Delay(1000, cancellationToken: cancellation);
        }
    }
```

## Q5. 여러 LifetimeScope를 동시에 사용 가능?

**A:** 가능하지만 권장하지 않음. 계층 구조 사용:

```
    Root
    ├── Title
    └── Game
        ├── Stage1
        └── Stage2
```

## Q6. 씬 로딩 시 부모 스코프 유지 방법?

**A:** `LifetimeScope.EnqueueParent` 사용:

```
    LifetimeScope.EnqueueParent(currentScope);
    await SceneManager.LoadSceneAsync("NextScene");
```

## Q7. 테스트는 어떻게?

**A:** Pure C# 클래스는 일반 단위 테스트 가능:

```
    [Test]
    public void PlayerTakeDamage_ReducesHealth()
    {
        // Arrange
        var player = new Player(maxHealth: 100);
        // Act
        player.TakeDamage(30);
        // Assert
        Assert.AreEqual(70, 
```

MonoBehaviour 테스트는 Unity Test Framework 사용.

## Q8. 성능 오버헤드는?

**A:** 거의 없음:
- Resolve: **0.5μs** (마이크로초)
- GC: **Zero**
- Container Build: 씬 로드 시 한 번만

## Q9. 기존 프로젝트에 적용 가능?

**A:** 가능하지만 점진적으로:
1. 새 기능부터 VContainer 적용
2. 레거시 코드는 천천히 리팩토링
3. 먼저 서비스 레이어부터 시작

## Q10. 학습 리소스는?

**A:**
- 📖 공식 문서: [https://vcontainer.hadashikick.jp/](https://vcontainer.hadashikick.jp/)
- 💻 GitHub: [https://github.com/hadashiA/VContainer](https://github.com/hadashiA/VContainer)
- 📦 예제: [https://github.com/mackysoft/VContainer-Examples](https://github.com/mackysoft/VContainer-Examples)
---

# 18. 참고 자료 📚

## 18.1 공식 리소스

### 문서

- **공식 문서**: [VContainer Documentation](https://vcontainer.hadashikick.jp/)
- **GitHub 저장소**: [hadashiA/VContainer](https://github.com/hadashiA/VContainer)
- **Releases**: [VContainer Releases](https://github.com/hadashiA/VContainer/releases)

### 튜토리얼

- **Hello World**: [Getting Started](https://vcontainer.hadashikick.jp/getting-started/hello-world)
- **Plain C# Entry Point**: [EntryPoint Guide](https://vcontainer.hadashikick.jp/integrations/entrypoint)
- **Container API**: [API Reference](https://vcontainer.hadashikick.jp/resolving/container-api)

## 18.2 예제 프로젝트

- **VContainer-Examples** by mackysoft: [GitHub](https://github.com/mackysoft/VContainer-Examples)
	- VContainer, UniRx, MessagePipe 통합 예제
- **VContainerSample** by KonH: [GitHub](https://github.com/KonH/VContainerSample)
	- 기본 사용법 샘플

## 18.3 블로그 및 아티클

- **Dependency Injection with VContainer**: [DEV Community](https://dev.to/longchau/dependency-injection-with-vcontainer-n9i)
- **Unity game architecture Part 1**: [DEV Community](https://dev.to/clandais/unity-game-architecture-part-1-4a9j)
- **Learning Architecture with VContainer**: [Medium](https://medium.com/kadinche-engineering/learning-architecture-messagepipe-and-vcontainer-by-creating-a-game-requested-by-my-own-daughter-8ee8303a718)

## 18.4 비교 및 토론

- **Comparing to Zenject**: [VContainer Docs](https://vcontainer.hadashikick.jp/comparing/comparing-to-zenject)
- **VContainer vs Zenject Discussion**: [Unity Discussions](https://discussions.unity.com/t/is-vcontainer-officially-supported-by-unity-im-looking-for-zenject-replacement/926328)
- **DI Frameworks in 2022**: [GitHub Gist](https://gist.github.com/dogfuntom/6aa1e7cb6e9bf3e482b6a6e790e28776)

## 18.5 통합 라이브러리

### MessagePipe

- **GitHub**: [Cysharp/MessagePipe](https://github.com/Cysharp/MessagePipe)
- VContainer와 완벽한 Pub/Sub 통합

### UniTask

- **GitHub**: [Cysharp/UniTask](https://github.com/Cysharp/UniTask)
- `IAsyncStartable` 등 비동기 EntryPoint 지원

### UniRx

- **GitHub**: [neuecc/UniRx](https://github.com/neuecc/UniRx)
- Reactive Programming 통합

## 18.6 성능 및 최적화

- **Async Container Build**: [Optimization Guide](https://vcontainer.hadashikick.jp/optimization/async-container-build)
- **Code Generation**: VContainerSettings에서 활성화

## 18.7 커뮤니티

- **Unity Discussions**: [DI in Unity](https://discussions.unity.com/t/dependency-injection-in-unity-inversion-of-control-container/914827)
- **GitHub Discussions**: [VContainer Discussions](https://github.com/hadashiA/VContainer/discussions)
- **GitHub Issues**: [Troubleshooting](https://github.com/hadashiA/VContainer/issues)

## 18.8 관련 개념

### 의존성 주입 (DI)

- **Martin Fowler - Inversion of Control**: [Article](https://martinfowler.com/articles/injection.html)
- **SOLID 원칙**: Dependency Inversion Principle

### 게임 아키텍처

- **Unity Architecture Patterns**: [Unity Manual](https://unity.com/how-to/how-architect-code-your-project-scales)
- **ECS with DI**: [GameDev Center](https://gamedev.center/how-to-use-dependency-injection-with-ecs-for-scalable-game-development/)
---

# 🎉 마치며

VContainer는 Unity 게임 개발에서 **깔끔하고 테스트 가능한 아키텍처**를 구축하는 강력한 도구입니다. Zenject 대비 압도적인 성능과 간결한 API로, 현대적인 Unity 프로젝트의 표준이 되고 있습니다.

## 핵심 요약

### ✅ VContainer를 사용해야 하는 이유

1. **성능**: Zenject 대비 5-10배 빠른 Resolve, Zero GC
2. **생산성**: 깔끔한 코드, 테스트 용이성, 모듈화
3. **유지보수**: 명확한 의존성, 변경 영향 최소화
4. **최적화**: Code Generation, Async Build, IL2CPP 최적화

### 🚀 시작하기

1. Unity Package Manager로 설치
2. RootLifetimeScope 생성
3. 서비스 등록 (Register)
4. EntryPoint 작성
5. 실행!

### 📖 학습 경로

1. **기초**: Hello World, 기본 등록/주입
2. **중급**: LifetimeScope 계층, Factory 패턴
3. **고급**: Clean Architecture, MessagePipe 통합

### 🔗 다음 단계

- 공식 문서 정독: [https://vcontainer.hadashikick.jp/](https://vcontainer.hadashikick.jp/)
- 예제 프로젝트 분석
- 작은 프로젝트부터 적용
**Happy Coding with VContainer!** 🎮✨
---
**작성일**: 2026-01-15  
**버전**: VContainer 1.17.0 기준  
**작성자**: Claude (Anthropic)

## 참고 링크 모음

- [VContainer 공식 문서](https://vcontainer.hadashikick.jp/)
- [VContainer GitHub](https://github.com/hadashiA/VContainer)
- [VContainer vs Zenject 비교](https://vcontainer.hadashikick.jp/comparing/comparing-to-zenject)
- [VContainer 예제 (mackysoft)](https://github.com/mackysoft/VContainer-Examples)
- [VContainer 예제 (KonH)](https://github.com/KonH/VContainerSample)
- [Unity 게임 아키텍처 Part 1](https://dev.to/clandais/unity-game-architecture-part-1-4a9j)
- [ECS with DI Guide](https://gamedev.center/how-to-use-dependency-injection-with-ecs-for-scalable-game-development/)
- [Async Container Build 최적화](https://vcontainer.hadashikick.jp/optimization/async-container-build)
- [Factory 등록 가이드](https://vcontainer.hadashikick.jp/registering/register-factory)
- [MonoBehaviour 주입 가이드](https://vcontainer.hadashikick.jp/resolving/gameobject-injection)