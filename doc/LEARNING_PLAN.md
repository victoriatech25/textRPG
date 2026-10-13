# Rust 학습형 Text RPG 단계별 개발 계획서

본 계획서는 **Text RPG를 개발하면서 Rust의 핵심 문법과 철학(소유권, 타입 시스템, 패턴 매칭 등)을 자연스럽게 체득**할 수 있도록 설계된 단계별 로드맵입니다.

---

## 🗺️ 단계별 학습 & 개발 로드맵

```
[1단계] 입출력 & 변수 (기본 루프)
   │
[2단계] 구조체(Struct) & 메서드 (플레이어 스탯)
   │
[3단계] 열거형(Enum) & 패턴 매칭 (게임 상태 전환)
   │
[4단계] 모듈화 & 참조자/빌림(Borrowing) (턴제 전투 & 몬스터)
   │
[5단계] 컬렉션(Vec) & 소유권 이동(Move) (인벤토리 & 아이템)
   │
[6단계] 트레이트(Trait) (공통 엔티티 추상화)
   │
[7단계] 던전 층(Floor) & 최종 보스전 (게임 완성)
   │
[8단계] (심화) 세이브/로드 & 에러 핸들링 (Serde & File I/O)
```

---

## 📌 상세 단계별 내용

### 1단계: 프로젝트 기초 & 표준 입출력 (기본 루프)
- **게임 기능**:
  - 플레이어 이름 입력 받기
  - 환영 메시지 출력
  - 메인 메뉴(1. 게임 시작 / 2. 게임 종료) 반복 루프
- **핵심 Rust 문법**:
  - `cargo new` / `cargo init` 프로젝트 구조 이해
  - 변수 선언과 가변성: `let` vs `let mut`
  - 표준 입력과 문자열: `std::io::stdin()`, `String`, `&str`
  - 에러 처리 기초: `.expect()`, `Result` 맛보기
  - 반복문: `loop`, `break`

---

### 2단계: 구조체(Struct)와 메서드 (캐릭터 스탯 관리)
- **게임 기능**:
  - `Player` 캐릭터 생성 (이름, HP, MaxHP, ATK, DEF, Gold, Level, EXP)
  - `[상태창 확인]` 메뉴 구현
- **핵심 Rust 문법**:
  - `struct` 정의 (필드 및 타입 지정: `u32`, `i32`, `String`)
  - `impl` 블록: 생성자 함수(`Player::new()`), 메서드 정의
  - 불변 참조(`&self`)와 가변 참조(`&mut self`)
  - 디버그 출력 매크로: `#[derive(Debug)]`, `println!("{:?}", player)`

---

### 3단계: 열거형(Enum)과 패턴 매칭 (게임 상태 머신)
- **게임 기능**:
  - 게임의 장소/상태 전환 (타이틀 화면 $\rightarrow$ 마을 $\rightarrow$ 던전 $\rightarrow$ 게임 오버)
  - 메뉴 선택 번호 입력의 안전한 처리
- **핵심 Rust 문법**:
  - `enum` 정의 (단순 열거형 및 데이터를 품은 열거형)
  - `match` 표현식과 완전성 검사(Exhaustive checking)
  - `Option<T>` 타입 (`Some`, `None`)을 활용한 null-safety
  - 문자열 파싱: `.trim().parse::<u32>()`

---

### 4단계: 참조/대여(Borrowing)와 외부 크레이트 (몬스터 & 턴제 전투)
- **게임 기능**:
  - 몬스터 조우 (`Monster` 구조체)
  - 1:1 턴제 전투 (플레이어 공격 $\leftrightarrow$ 몬스터 반격)
  - 데미지 계산 및 전투 승리/패배 판정
- **핵심 Rust 문법**:
  - **소유권(Ownership)과 빌림(Borrowing)**: 함수에 인자 넘길 때 소유권 이동 방지 (`&mut Player`, `&mut Monster`)
  - 모듈 분리: `src/player.rs`, `src/monster.rs`, `src/combat.rs`, `mod` 키워드
  - 외부 라이브러리 사용: `Cargo.toml`에 `rand` 크레이트 추가 및 난수 생성

---

### 5단계: 컬렉션(Vec)과 소유권 이동(Move) (인벤토리 & 아이템)
- **게임 기능**:
  - 소비형 포션(HP 포션), 장비 아이템 정의
  - 마을 상점에서 물약 구매
  - 가방(인벤토리)에서 물약 사용 시 수량 차감 및 HP 회복
- **핵심 Rust 문법**:
  - 벡터 컬렉션: `Vec<Item>`
  - 소유권 이동(Move semantics): 아이템을 인벤토리에서 꺼내 사용 후 소멸
  - 슬라이스(`&[T]`)와 반복자(`iterator`, `.iter()`, `.iter_mut()`)

---

### 6단계: 트레이트(Trait)와 다형성 (엔티티 추상화)
- **게임 기능**:
  - `Player`와 `Monster`가 공통으로 가지는 전투 행동 통일 (`take_damage`, `is_alive`, `get_name`)
- **핵심 Rust 문법**:
  - `trait` 선언 및 구현 (`impl Combatant for Player`, `impl Combatant for Monster`)
  - 기본 구현(Default implementation) 제공
  - 트레이트 바운드(Trait bounds)를 통한 일반화(Generics)

---

### 7단계: 던전 진행 & 최종 보스전 (게임 클리어 루프 완성)
- **게임 기능**:
  - 던전 층(1층 ~ 5층) 순차 탐험
  - 층별 몬스터 난이도 차등 적용
  - 5층 최종 보스(Dragon) 조우 및 처치 시 승리 엔딩 화면 출력
- **핵심 Rust 문법**:
  - 모듈 가시성: `pub`, `pub(crate)`
  - 상태 보존 및 게임 루프 아키텍처 완성

---

### 8단계 (심화 보너스): 파일 입출력 및 직렬화 (세이브 / 로드)
- **게임 기능**:
  - 게임 진행 상황을 `save.json` 파일로 저장 및 이어하기
- **핵심 Rust 문법**:
  - `serde`, `serde_json` 크레이트 활용
  - 파일 I/O: `std::fs::File`, `std::io::Write`, `std::io::Read`
  - 커스텀 에러 처리와 `Result<T, E>`, `?` 연산자

---

## 🎯 학습 진행 방식
1. 각 단계마다 **먼저 이번 단계에서 배울 Rust 문법 핵심 개념**을 간단히 짚고,
2. 코드를 점진적으로 작성/리팩토링하며 컴파일러 에러를 마주하고 해결하는 방식으로 진행합니다.
3. 단계가 끝날 때마다 직접 `cargo run`으로 동작을 확인합니다.
