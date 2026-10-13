# Rust 학습형 Text RPG 단계별 개발 계획서

본 계획서는 **Text RPG를 개발하면서 Rust의 핵심 문법과 철학(소유권, 타입 시스템, 패턴 매칭 등)을 자연스럽게 체득**할 수 있도록 설계된 단계별 로드맵입니다.

> 각 단계의 코드 예시, 컴파일 에러 포인트, 완료 체크리스트, 응용 과제는 [STEP_BY_STEP_GUIDE.md](STEP_BY_STEP_GUIDE.md)에 있습니다.
> 게임 규칙은 [GAME_DESIGN.md](GAME_DESIGN.md)를 기준으로 하며, 아직 정해지지 않은 규칙은 단계별로 [STAGE_DECISIONS.md](STAGE_DECISIONS.md)에 정리되어 있습니다.

---

## 🗺️ 단계별 학습 & 개발 로드맵

```
[1단계] 입출력 & 변수 (기본 루프)
   │
[2단계] 구조체(Struct) & 메서드 (플레이어 스탯)
   │
[3단계] 열거형(Enum) & 패턴 매칭 (게임 상태 전환)
   │
[4단계] 소유권 & 대여(Borrowing) + 첫 테스트 (턴제 전투 & 몬스터)
   │
[5단계] 컬렉션(Vec) & 소유권 이동(Move) (인벤토리 & 아이템)
   │
[6단계] 트레이트(Trait) (공통 엔티티 추상화 & 리팩토링)
   │
[7단계] 모듈 분리 & 던전 층(Floor) & 최종 보스전 (게임 완성)
   │
[8단계] (심화) 세이브/로드 & 에러 핸들링 (Serde & File I/O)
```

- **1~6단계**는 `src/main.rs` 한 파일에서 작업합니다. 파일이 충분히 커져서 "나눠야겠다"는 필요를 느낀 뒤에 **7단계에서 모듈로 분리**합니다.
- **4단계부터는 매 단계 단위 테스트**를 1~2개 이상 작성합니다.

---

## 📌 상세 단계별 내용

### 1단계: 프로젝트 기초 & 표준 입출력 (기본 루프)
- **게임 기능**:
  - 플레이어 이름 입력 받기
  - 환영 메시지 출력
  - 메인 메뉴(`[1] 새로운 모험 시작` / `[2] 게임 종료`) 반복 루프
- **핵심 Rust 문법**:
  - `cargo new` / `cargo init` 프로젝트 구조 이해
  - 변수 선언과 가변성: `let` vs `let mut`
  - 표준 입력과 문자열: `std::io::stdin()`, `String`, `&str`
  - 에러 처리 기초: `.expect()`, `Result` 맛보기
  - 반복문: `loop`, `break`

---

### 2단계: 구조체(Struct)와 메서드 (캐릭터 스탯 관리)
- **게임 기능**:
  - `Player` 캐릭터 생성 (이름, Level, HP/MaxHP, MP/MaxMP, ATK, DEF, Gold, EXP)
  - `[상태창 확인]` 메뉴, 휴식(`rest`) 기능 구현
- **핵심 Rust 문법**:
  - `struct` 정의 (필드 및 타입 지정: `u32`, `String`)
  - `impl` 블록: 생성자 함수(`Player::new()`), 메서드 정의
  - 불변 참조(`&self`)와 가변 참조(`&mut self`)
  - 디버그 출력 매크로: `#[derive(Debug)]`, `println!("{:?}", player)`

---

### 3단계: 열거형(Enum)과 패턴 매칭 (게임 상태 머신)
- **게임 기능**:
  - 게임의 장소/상태 전환 (`Town` → `Dungeon` → `GameOver` / `Victory` / `Quit`)
  - 메뉴 선택 번호 입력의 안전한 처리 (숫자가 아니거나 범위 밖이면 재입력)
  - 마을 메뉴의 휴식/상태 확인이 실제로 플레이어에게 반영되도록 `&mut Player` 전달
- **핵심 Rust 문법**:
  - `enum` 정의
  - `match` 표현식, 매치 가드, 완전성 검사(Exhaustive checking)
  - `Option<T>` 타입 (`Some`, `None`)을 활용한 null-safety
  - 문자열 파싱: `.trim().parse::<u32>()`와 `Result` 처리

---

### 4단계: 소유권 & 대여(Borrowing) (몬스터 & 턴제 전투)
- **게임 기능**:
  - 몬스터 조우 (`Monster` 구조체, 일반/보스 생성자)
  - 1:1 턴제 전투 (플레이어 공격/도망 ↔ 몬스터 반격)
  - 데미지 계산 및 전투 결과(`Victory` / `Defeat` / `Fled`) 판정, 보상 지급
  - 도망 성공 확률 (난수)
- **핵심 Rust 문법**:
  - **소유권(Ownership) 규칙**과 **빌림(Borrowing) 규칙**의 구분
  - 함수에 `&mut Player`, `&mut Monster`를 넘겨 소유권 이동 없이 상태 변경
  - `String` vs `&str` 매개변수, `saturating_sub`로 언더플로 방지
  - 결과가 셋 이상인 반환값을 `enum`으로 표현 (`BattleResult`)
  - 외부 라이브러리 사용: `cargo add rand` (현재 0.10, 버전별 API 차이 주의)
  - **단위 테스트**: `#[cfg(test)]`, `#[test]`, `assert_eq!`, `cargo test`

---

### 5단계: 컬렉션(Vec)과 소유권 이동(Move) (인벤토리 & 아이템)
- **게임 기능**:
  - 포션, 무기, 방어구 정의 (`enum Item { Potion(..), Weapon(..), Armor(..) }`)
  - 마을 상점에서 골드로 아이템 구매
  - 포션 사용 시 HP 회복 후 인벤토리에서 제거 (포션 1개 = 인벤토리 1칸)
  - 무기/방어구 장착 슬롯 (`Option<Weapon>`, `Option<Armor>`), 교체 시 기존 장비는 인벤토리로
- **핵심 Rust 문법**:
  - 데이터를 품은 열거형
  - 벡터 컬렉션: `Vec<Item>`, `push`, `remove`, `get`
  - 소유권 이동(Move semantics): 인벤토리에 넣고 꺼낼 때의 소유권 흐름, `Option::replace`
  - 반복자와 클로저: `.iter()`, `.enumerate()`, `.filter(|x| ...)`, `.count()`
  - `Option`을 빌려서 다루기: `.as_ref()`, `.map_or()`

---

### 6단계: 트레이트(Trait)와 다형성 (엔티티 추상화)
- **게임 기능**:
  - `Player`와 `Monster`가 공통으로 가지는 전투 행동 통일 (`name`, `hp`, `atk`, `def`, `take_damage`, `is_alive`)
  - 4단계에서 중복 작성된 메서드를 트레이트로 옮기는 **리팩토링** (테스트로 동작 보존 확인)
  - 상태창 출력을 `Display` 트레이트 구현으로 대체
- **핵심 Rust 문법**:
  - `trait` 선언 및 구현 (`impl Combatant for Player`, `impl Combatant for Monster`)
  - 기본 구현(Default implementation) 제공
  - 트레이트 바운드를 통한 제네릭 함수 (정적 디스패치)
  - `Box<dyn Trait>` (동적 디스패치)와 dyn 호환성
  - 표준 트레이트 구현: `std::fmt::Display`
  - 라이프타임 생략 규칙 기초 (`fn name(&self) -> &str`)

---

### 7단계: 모듈 분리 & 던전 진행 (게임 클리어 루프 완성)
- **게임 기능**:
  - `main.rs`를 역할별 모듈 파일로 분리
  - 던전 층(1층 ~ 5층) 순차 탐험, 층별 몬스터 난이도 차등 적용
  - 5층 최종 보스(고대 화염룡) 조우 및 처치 시 승리 엔딩 화면 출력
  - 전체 흐름을 관리하는 `Game` 구조체
- **핵심 Rust 문법**:
  - 모듈 시스템: `mod`, `use`, `crate`, `super`
  - 모듈 가시성: `pub`, `pub(crate)`, 비공개 필드를 통한 캡슐화
  - `let ... else` 패턴
  - 상태 보존 및 게임 루프 아키텍처 완성

---

### 8단계 (심화 보너스): 파일 입출력 및 직렬화 (세이브 / 로드)
- **게임 기능**:
  - 게임 진행 상황을 `savegame.json` 파일로 저장 및 이어하기
  - 세이브 파일이 없거나 손상되었을 때 패닉 없이 새 게임 시작
- **핵심 Rust 문법**:
  - `serde`(`derive` feature), `serde_json` 크레이트 활용
  - 파일 I/O: `std::fs::write`, `std::fs::read_to_string`
  - 커스텀 에러 타입(`enum SaveError`)과 `From` 트레이트, `Result<T, E>`, `?` 연산자

---

## 🎯 학습 진행 방식
0. 단계를 시작하면 [STAGE_DECISIONS.md](STAGE_DECISIONS.md)에서 그 단계의 **결정 항목과 세부 기획**을 먼저 채웁니다.
1. 각 단계마다 **먼저 이번 단계에서 배울 Rust 문법 핵심 개념**을 간단히 짚고,
2. 예시 코드를 보기 전에 **직접 먼저 작성**해 보고, 컴파일러 에러를 마주하고 해결합니다.
3. 4단계부터는 규칙을 하나 만들 때마다 **단위 테스트**를 추가합니다.
4. 단계가 끝나면 `cargo fmt` → `cargo clippy` → `cargo test` → `cargo run`으로 확인하고, 가이드의 **완료 체크리스트**를 채운 뒤 git 커밋합니다.
5. 여유가 있으면 가이드의 **응용 과제**에 도전합니다.
