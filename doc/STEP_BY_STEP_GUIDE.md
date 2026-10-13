# Rust Text RPG 단계별 구현 및 학습 상세 가이드

본 문서는 Rust 문법을 체계적으로 학습하면서 Text RPG를 완성해 나갈 수 있도록, **각 단계별 목표 기능, Rust 핵심 개념, 코딩 가이드라인, 코드 예시, 주의해야 할 컴파일 에러, 완료 체크리스트, 응용 과제**를 정리한 가이드입니다.

> 📌 **예시 코드 사용법**
> - 모든 예시 코드는 Rust 1.98 / edition 2024 기준으로 실제 컴파일 및 `cargo test` 통과를 확인했습니다.
> - 예시는 "정답"이 아니라 **방향 제시**입니다. 먼저 직접 작성해 보고, 막힐 때 참고하세요.
> - 각 코드 블록 첫 줄의 주석(`// 파일: ...`)은 그 코드가 들어갈 파일입니다. **1~6단계는 `src/main.rs` 한 파일**에서 작업하고, **7단계에서 모듈로 분리**합니다.
> - 각 단계는 **`0. 시작 전: 결정 & 기획`** 으로 시작합니다. [STAGE_DECISIONS.md](STAGE_DECISIONS.md)에서 그 단계의 결정 항목과 세부 기획을 먼저 채운 뒤 구현하세요. 예시 코드는 각 결정 항목의 `기본안`을 따릅니다.

---

## 📑 목차
1. [1단계: 프로젝트 기초 & 표준 입출력 (기본 루프)](#1단계-프로젝트-기초--표준-입출력-기본-루프)
2. [2단계: 구조체(Struct)와 메서드 (캐릭터 스탯 관리)](#2단계-구조체struct와-메서드-캐릭터-스탯-관리)
3. [3단계: 열거형(Enum)과 패턴 매칭 (게임 상태 머신 & 입력 검증)](#3단계-열거형enum과-패턴-매칭-게임-상태-머신--입력-검증)
4. [4단계: 소유권(Ownership) & 대여(Borrowing) (몬스터 & 턴제 전투)](#4단계-소유권ownership--대여borrowing-몬스터--턴제-전투)
5. [5단계: 컬렉션(Vec) & 소유권 이동 (인벤토리 & 아이템)](#5단계-컬렉션vec--소유권-이동-인벤토리--아이템)
6. [6단계: 트레이트(Trait)와 다형성 (전투 엔티티 추상화)](#6단계-트레이트trait와-다형성-전투-엔티티-추상화)
7. [7단계: 모듈 분리 & 던전 진행 (최종 보스 및 게임 클리어)](#7단계-모듈-분리--던전-진행-최종-보스-및-게임-클리어)
8. [8단계: 에러 핸들링 & 파일 I/O (세이브 / 로드 시스템)](#8단계-에러-핸들링--파일-io-세이브--로드-시스템)
9. [💡 개발 사이클 팁](#-개발-사이클-팁)

---

## 1단계: 프로젝트 기초 & 표준 입출력 (기본 루프)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 1단계](STAGE_DECISIONS.md#1단계-입출력--기본-루프)를 채우세요: 게임 제목, 메뉴 번호 표기(`[1]` / `1.`), 문체, 이름 입력 규칙
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- 터미널에 게임 시작 타이틀 화면 출력
- 플레이어의 이름 입력받기 (`stdin`)
- 메인 루프에서 메뉴 출력: `[1] 모험 시작`, `[2] 게임 종료`
- 번호 입력에 따라 분기 처리 (`2` 입력 시 반복문 종료 및 프로그램 정상 종료)

### 2. Rust 핵심 개념
- **`cargo init` / `Cargo.toml`**: Cargo 패키지 매니저의 역할과 프로젝트 구조 (`src/main.rs`).
- **변수 불변성(Immutability)**: Rust의 변수는 기본적으로 불변(`let`). 값을 변경하려면 명시적으로 `let mut` 사용.
- **문자열 타입의 구분**:
  - `String`: 힙(Heap) 메모리에 할당되는 크기 변경 가능한 문자열.
  - `&str`: 문자열 슬라이스(참조), 주로 코드 내 하드코딩된 리터럴(`"Hello"`)에 사용.
- **에러 처리 맛보기 (`Result`, `.expect()`)**: `std::io::stdin().read_line()`은 실패할 수 있으므로 `Result`를 반환. `.expect("에러 메시지")`로 패닉 유도 및 언래핑.
- **무한 루프와 탈출**: `loop { ... }` 및 `break`.

### 3. 코딩 방향 및 가이드
- 사용자 입력 함수를 별도로 만들어두면 재사용하기 좋습니다. (7단계에서 `io_helper.rs`로 분리)
- 입력을 받을 때 줄바꿈(`\n`, `\r\n`)이 포함되므로 반드시 `.trim()`을 호출해야 합니다.

```rust
// 파일: src/main.rs
use std::io::{self, Write};

fn read_line(prompt: &str) -> String {
    print!("{}", prompt);
    io::stdout().flush().expect("출력 버퍼 비우기 실패");

    let mut input = String::new();
    io::stdin().read_line(&mut input).expect("입력 읽기 실패");
    input.trim().to_string()
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- `io::stdout().flush()`를 하지 않으면 `print!` 뒤의 프롬프트가 즉시 화면에 나타나지 않을 수 있습니다 (`println!`은 자동 개행 시 flush).
- `read_line`에 `&mut input` 대신 `&input`을 넘기면 "cannot borrow as mutable" 에러 발생.

### 5. 완료 체크리스트
- [ ] `cargo run`으로 실행하면 타이틀과 이름 입력 프롬프트가 나온다.
- [ ] `2`를 입력하면 프로그램이 정상 종료된다.
- [ ] `1`, `2` 이외의 값(빈 입력, 문자 등)을 넣어도 프로그램이 죽지 않는다.

### 6. 응용 과제
- 이름에 빈 문자열이 들어오면 다시 입력받도록 만들어 보세요. (`String::is_empty()`)

---

## 2단계: 구조체(Struct)와 메서드 (캐릭터 스탯 관리)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 2단계](STAGE_DECISIONS.md#2단계-플레이어-스탯)를 채우세요: 스탯 구성, 초기 스탯 수치, 직업 유무, 상태창 레이아웃
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- `Player` 구조체 정의: 이름, 레벨, HP/최대 HP, MP/최대 MP, 공격력(ATK), 방어력(DEF), 소지 골드, 경험치.
- 플레이어 생성 연관 함수 (`Player::new(name)`).
- 플레이어 상태를 보여주는 `print_status()` 메서드, HP/MP를 회복하는 `rest()` 메서드 구현.
- 메인 메뉴에 `[상태 확인]` 추가하여 현재 스탯 출력.

### 2. Rust 핵심 개념
- **구조체(`struct`)**: 데이터 필드를 묶는 사용자 정의 타입.
- **`impl` 블록과 연관 함수(Associated Function)**:
  - `Player::new(...)`: 인스턴스 없이 호출하는 생성자 패턴 (C++/Java의 static 메서드와 유사).
- **메서드와 `self` 파라미터**:
  - `&self`: 인스턴스를 읽기 전용(불변 참조)으로 빌려옴 (예: `print_status`).
  - `&mut self`: 인스턴스의 내부 상태를 수정하기 위해 가변으로 빌려옴 (예: `rest`, `take_damage`).
  - `self`: 인스턴스의 소유권을 가져와 소비함.
- **파생 어트리뷰트(`#[derive(Debug)]`)**: `{:?}`로 구조체 내부 필드를 한 번에 디버그 출력할 수 있게 해줌.

### 3. 코딩 방향 및 가이드
> 인벤토리·장비 필드(`inventory`, `weapon`, `armor`)는 아직 `Item` 타입이 없으므로 **5단계에서 추가**합니다.

```rust
// 파일: src/main.rs
#[derive(Debug)]
pub struct Player {
    pub name: String,
    pub level: u32,
    pub hp: u32,
    pub max_hp: u32,
    pub mp: u32,
    pub max_mp: u32,
    pub atk: u32,
    pub def: u32,
    pub gold: u32,
    pub exp: u32,
}

impl Player {
    pub fn new(name: String) -> Self {
        Self {
            name,
            level: 1,
            hp: 50,
            max_hp: 50,
            mp: 20,
            max_mp: 20,
            atk: 10,
            def: 2,
            gold: 100,
            exp: 0,
        }
    }

    pub fn print_status(&self) {
        println!("====== [{}] 상태창 ======", self.name);
        println!(
            "Lv.{} | HP: {}/{} | MP: {}/{} | ATK: {} | DEF: {} | EXP: {} | Gold: {}G",
            self.level, self.hp, self.max_hp, self.mp, self.max_mp,
            self.atk, self.def, self.exp, self.gold
        );
    }

    pub fn rest(&mut self) {
        self.hp = self.max_hp;
        self.mp = self.max_mp;
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 구조체 필드 이름과 매개변수 이름이 같으면 `name: name` 대신 `name`으로 축약(Field Init Shorthand) 가능.
- 인스턴스를 변경하는 메서드를 호출할 때 변수가 `let player = ...`로 선언되어 있으면 호출 불가 → `let mut player = ...`로 선언해야 함.

### 5. 완료 체크리스트
- [ ] 입력한 이름으로 `Player`가 생성되고, `[상태 확인]` 메뉴에서 스탯이 출력된다.
- [ ] `println!("{:?}", player)`로 디버그 출력을 한 번 확인해 봤다.

### 6. 응용 과제
- `rest()`를 `&self`로 바꿔 보고 어떤 컴파일 에러가 나는지 읽어 보세요.

---

## 3단계: 열거형(Enum)과 패턴 매칭 (게임 상태 머신 & 입력 검증)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 3단계](STAGE_DECISIONS.md#3단계-게임-상태--마을-메뉴)를 채우세요: `GameState` 목록, **마을 메뉴 최종 번호**, 여관 이용료, 상태 전이도
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- 게임의 현재 화면/모드를 `GameState` 열거형으로 관리:
  - `Town`(마을), `Dungeon`(던전), `Inventory`(가방), `GameOver`(패배), `Victory`(클리어), `Quit`(종료)
  - 전투(Battle)는 던전 안에서 일어나는 일이므로 별도 상태로 두지 않고 4단계의 전투 함수가 담당합니다.
- 플레이어의 메뉴 번호 입력을 숫자로 파싱하고, 범위를 벗어나면 `None`으로 처리.
- 마을 메뉴에서 `[여관 휴식]`, `[상태 확인]`이 실제로 동작하도록 `Player`를 넘겨받기.

### 2. Rust 핵심 개념
- **열거형(`enum`)**: 가능한 여러 상태 중 단 하나만을 표현하는 타입. Rust의 enum은 각 변형(variant)에 데이터를 포함할 수도 있음 (5단계 `Item`에서 사용).
- **`match` 표현식과 완전성(Exhaustiveness)**: 모든 경우의 수를 처리하지 않으면 컴파일 거부 → 버그 원천 차단.
- **`Option<T>`과 `Result<T, E>`**:
  - `Some(value)` / `None`: null 대신 사용하는 안전한 컨테이너.
  - `Ok(value)` / `Err(e)`: 연산 성공/실패 표현.
- **문자열 정수 파싱**: `input.parse::<u32>()`의 반환값 `Result`를 `match`로 처리.
- **매치 가드(match guard)**: `Ok(n) if 조건 => ...`
- **가변 참조 맛보기**: 함수가 `&mut Player`를 받아 플레이어 상태를 수정 (4단계에서 본격적으로 다룸).

### 3. 코딩 방향 및 가이드
```rust
// 파일: src/main.rs
fn read_choice(prompt: &str, max: u32) -> Option<u32> {
    let input = read_line(prompt);
    match input.parse::<u32>() {
        Ok(n) if (1..=max).contains(&n) => Some(n),
        _ => None, // 숫자가 아니거나 범위를 벗어남
    }
}

pub enum GameState {
    Town,
    Dungeon,
    Inventory,
    GameOver,
    Victory,
    Quit,
}

fn handle_town_menu(player: &mut Player) -> GameState {
    println!("\n[마을 거점]");
    println!("1. 던전 입장");
    println!("2. 여관에서 휴식 (HP/MP 회복)");
    println!("3. 상태 확인");
    println!("4. 게임 종료");

    match read_choice("선택 > ", 4) {
        Some(1) => GameState::Dungeon,
        Some(2) => {
            player.rest();
            println!("여관에서 푹 쉬었습니다. HP/MP가 모두 회복되었습니다!");
            GameState::Town
        }
        Some(3) => {
            player.print_status();
            GameState::Town
        }
        Some(4) => GameState::Quit,
        _ => {
            println!("잘못된 번호입니다. 다시 선택해주세요.");
            GameState::Town
        }
    }
}

fn main() {
    let name = read_line("모험가의 이름을 입력하세요 > ");
    let mut player = Player::new(name);
    let mut state = GameState::Town;

    loop {
        state = match state {
            GameState::Town => handle_town_menu(&mut player),
            GameState::Dungeon => GameState::Town,   // 4단계에서 구현
            GameState::Inventory => GameState::Town, // 5단계에서 구현
            GameState::GameOver => {
                println!("GAME OVER");
                break;
            }
            GameState::Victory => {
                println!("축하합니다! 게임을 클리어했습니다!");
                break;
            }
            GameState::Quit => break,
        };
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- `read_choice`의 결과는 `Option<u32>`이므로 `Some(1)`~`Some(4)`만 나열하면 `Some(5..)`와 `None`이 빠져 **"non-exhaustive patterns"** 컴파일 에러 발생 → 마지막에 `_`가 필요합니다. 직접 `_` 줄을 지워 에러 메시지를 확인해 보세요.
- 아직 만들지 않은 `Inventory`, `GameOver`, `Victory` 변형은 "variant is never constructed" **경고**가 날 수 있습니다. 이후 단계에서 사용하면 사라집니다.

### 5. 완료 체크리스트
- [ ] `abc`, `0`, `9`, 빈 입력을 넣어도 "잘못된 번호" 안내 후 메뉴가 다시 나온다.
- [ ] `[여관 휴식]`과 `[상태 확인]`이 실제로 동작한다.

### 6. 응용 과제
- `handle_town_menu`의 매개변수를 `player: &Player`(불변 참조)로 바꾸면 어디서 에러가 나는지 확인하고 이유를 설명해 보세요.

---

## 4단계: 소유권(Ownership) & 대여(Borrowing) (몬스터 & 턴제 전투)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 4단계](STAGE_DECISIONS.md#4단계-턴제-전투--성장)를 채우세요: 데미지 공식, 턴 순서, 도망 규칙, **사망 처리**, **레벨업 테이블**, 전투 메시지
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- `Monster` 구조체 정의 (이름, HP, MaxHP, ATK, DEF, 보상 EXP, 보상 Gold, 보스 여부).
- 턴제 전투 루프:
  1. 플레이어 턴: `[1] 공격` / `[2] 도망` (스킬·포션은 이후 단계에서 추가)
  2. 몬스터 턴: 플레이어에게 반격 (도망 실패 시에도 반격)
- 데미지 계산(`max(1, ATK - DEF)`) 및 사망/승리 판정, 보상 지급.
- 전투 결과를 `BattleResult` 열거형(`Victory` / `Defeat` / `Fled`)으로 반환.
- `rand` 크레이트로 도망 성공 확률(기본안 50%) 구현.
- 데미지 공식 단위 테스트 작성 (`cargo test`).

### 2. Rust 핵심 개념
- **소유권(Ownership) 규칙** (공식 3규칙):
  1. Rust의 각 값은 소유자(Owner)를 가진다.
  2. 한 번에 소유자는 단 하나뿐이다.
  3. 소유자가 스코프를 벗어나면 값은 버려진다(`drop`).
- **빌림(Borrowing) 규칙**: 어느 시점이든 **하나의 가변 참조(`&mut T`)** 또는 **여러 개의 불변 참조(`&T`)** 중 하나만 존재할 수 있다 (데이터 레이스 방지).
- **빌림 실습**: 전투 함수 `execute_combat(player: &mut Player, monster: &mut Monster)`처럼 소유권을 넘기지 않고 대여하여 상태 변경.
- **`String` vs `&str` 매개변수**: `Monster::new(name: &str, ...)`로 받고 내부에서 `name.to_string()`으로 소유권 있는 `String` 생성.
- **안전한 산술**: `saturating_sub` 로 `u32` 언더플로(음수) 방지.
- **외부 크레이트 의존성 추가**: `cargo add rand` 실행 → `Cargo.toml`에 `rand = "0.10"` 추가.
  - ⚠️ `rand`는 버전마다 API 이름이 자주 바뀝니다 (0.8의 `thread_rng()`/`gen_range` → 0.9 이후 `rng()`/`random_range`). 인터넷 예제가 컴파일되지 않으면 버전부터 확인하세요. 가장 간단한 방법은 `rand::random_bool(0.5)`, `rand::random_range(1..=6)` 같은 함수를 쓰는 것입니다.
- **단위 테스트**: `#[cfg(test)] mod tests`, `#[test]`, `assert_eq!`, `cargo test`.

### 3. 코딩 방향 및 가이드
`Player`에도 `is_alive`, `take_damage`, `attack(&self, target: &mut Monster)`, `gain_rewards`를 `Monster`와 같은 방식으로 추가합니다. (6단계에서 트레이트로 합칩니다.)

```rust
// 파일: src/main.rs
pub struct Monster {
    pub name: String,
    pub hp: u32,
    pub max_hp: u32,
    pub atk: u32,
    pub def: u32,
    pub reward_exp: u32,
    pub reward_gold: u32,
    pub is_boss: bool,
}

impl Monster {
    pub fn new(name: &str, hp: u32, atk: u32, def: u32, reward_exp: u32, reward_gold: u32) -> Self {
        Self {
            name: name.to_string(),
            hp,
            max_hp: hp,
            atk,
            def,
            reward_exp,
            reward_gold,
            is_boss: false,
        }
    }

    pub fn boss(name: &str, hp: u32, atk: u32, def: u32, reward_exp: u32, reward_gold: u32) -> Self {
        Self {
            is_boss: true,
            ..Self::new(name, hp, atk, def, reward_exp, reward_gold) // 구조체 갱신 문법
        }
    }

    pub fn is_alive(&self) -> bool {
        self.hp > 0
    }

    pub fn take_damage(&mut self, damage: u32) {
        self.hp = self.hp.saturating_sub(damage);
    }

    pub fn attack(&self, target: &mut Player) -> u32 {
        let damage = calc_damage(self.atk, target.def);
        target.take_damage(damage);
        damage
    }
}

/// 데미지 공식: max(1, ATK - DEF)
pub fn calc_damage(atk: u32, def: u32) -> u32 {
    atk.saturating_sub(def).max(1)
}

pub enum BattleResult {
    Victory,
    Defeat,
    Fled,
}

pub fn execute_combat(player: &mut Player, monster: &mut Monster) -> BattleResult {
    println!("\n야생의 [{}]이(가) 나타났다!", monster.name);

    loop {
        println!("\n[{} HP: {}/{}]  VS  [{} HP: {}/{}]",
            player.name, player.hp, player.max_hp, monster.name, monster.hp, monster.max_hp);
        println!("1. 공격  |  2. 도망");

        // --- 플레이어 턴 ---
        match read_choice("행동 선택 > ", 2) {
            Some(1) => {
                let damage = player.attack(monster);
                println!("{}에게 {}의 피해를 입혔습니다!", monster.name, damage);
                if !monster.is_alive() {
                    println!("\n승리했습니다! {} EXP와 {} Gold를 획득합니다.",
                        monster.reward_exp, monster.reward_gold);
                    player.gain_rewards(monster.reward_exp, monster.reward_gold);
                    return BattleResult::Victory;
                }
            }
            Some(2) => {
                if rand::random_bool(0.5) {
                    println!("전투에서 도망쳤습니다!");
                    return BattleResult::Fled;
                }
                println!("도망에 실패했습니다!");
            }
            _ => {
                println!("잘못된 입력입니다.");
                continue; // 잘못된 입력은 턴을 소모하지 않음
            }
        }

        // --- 몬스터 턴 ---
        let damage = monster.attack(player);
        println!("{}의 반격! {}의 피해를 받았습니다!", monster.name, damage);
        if !player.is_alive() {
            println!("\n패배했습니다... 눈앞이 캄캄해집니다.");
            return BattleResult::Defeat;
        }
    }
}
```

메인 루프의 `GameState::Dungeon`에서 전투 결과로 다음 상태를 정합니다.

```rust
// 파일: src/main.rs (main 함수의 loop 안)
GameState::Dungeon => {
    let mut monster = Monster::new("슬라임", 20, 5, 1, 15, 20);
    match execute_combat(&mut player, &mut monster) {
        BattleResult::Victory | BattleResult::Fled => GameState::Town,
        BattleResult::Defeat => GameState::GameOver,
    }
}
```

> 💡 **왜 `bool`이 아니라 `BattleResult`인가?** `true/false`로 반환하면 "도망"과 "패배"가 둘 다 `false`가 되어 호출하는 쪽이 게임 오버 여부를 구분할 수 없습니다. 의미가 3가지 이상이면 열거형을 쓰세요.

**첫 단위 테스트** (파일 맨 아래에 추가):
```rust
// 파일: src/main.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn damage_is_atk_minus_def() {
        assert_eq!(calc_damage(10, 2), 8);
    }

    #[test]
    fn damage_is_at_least_one() {
        assert_eq!(calc_damage(3, 10), 1);
    }

    #[test]
    fn hp_does_not_underflow() {
        let mut player = Player::new(String::from("테스트"));
        player.take_damage(999);
        assert_eq!(player.hp, 0);
        assert!(!player.is_alive());
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 불변 참조가 살아있는 동안 가변 참조를 만들면 빌림 검사기(Borrow Checker) 에러 **E0502** 발생:
  ```rust
  let name = &player.name;   // 불변 대여 시작
  player.take_damage(5);     // ❌ 가변 대여 시도
  println!("{}", name);      // 불변 대여가 여기까지 살아 있음
  ```
  `error[E0502]: cannot borrow `player` as mutable because it is also borrowed as immutable`
  → `println!`을 `take_damage` 앞으로 옮기거나, `player.name.clone()`으로 복사하면 해결됩니다.
- `u32`끼리 `self.hp - damage`를 하면 결과가 음수가 될 때 **디버그 빌드에서 패닉**(attempt to subtract with overflow)이 납니다 → `saturating_sub` 사용.

### 5. 완료 체크리스트
- [ ] 던전에 들어가면 슬라임과 전투가 시작되고, 승리/도망/패배 각각이 올바른 상태로 이어진다.
- [ ] `cargo test`가 통과한다.
- [ ] E0502 에러를 직접 한 번 만들어 보고 메시지를 읽어 봤다.

### 6. 응용 과제
- `rand::random_range(0..=2)`로 데미지에 0~2의 편차를 더해 보세요. 이때 `calc_damage`는 그대로 두고(테스트 유지) 편차는 별도 함수로 분리하는 것이 핵심입니다.

---

## 5단계: 컬렉션(Vec) & 소유권 이동 (인벤토리 & 아이템)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 5단계](STAGE_DECISIONS.md#5단계-아이템-상점-인벤토리)를 채우세요: **아이템 목록과 가격**, 방어구 효과, 인벤토리 표시 방식, 상점 흐름
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- 아이템 정의: 포션(`Potion`), 무기(`Weapon`), 방어구(`Armor`) 구조체와 이를 묶는 `enum Item`.
- `Player`에 인벤토리(`Vec<Item>`)와 장비 슬롯(`weapon: Option<Weapon>`, `armor: Option<Armor>`) 추가.
- 마을 상점에서 골드로 아이템 구매 (`buy`).
- 인벤토리에서 아이템 사용:
  - 포션 → HP 회복 후 인벤토리에서 **제거(소비)**. 포션 1개 = `Vec`의 요소 1개.
  - 무기/방어구 → 장비 슬롯에 장착. 기존 장비는 인벤토리로 되돌아감.
- 공격력/방어력은 `기본 스탯 + 장비 보너스`로 계산 (`total_atk()`, `total_def()`).

### 2. Rust 핵심 개념
- **데이터를 품은 열거형**: `Item::Potion(Potion)`처럼 변형마다 다른 데이터를 가짐. 무기 슬롯 타입을 `Option<Weapon>`으로 하면 "무기 슬롯에 포션이 들어가는" 잘못된 상태를 **타입 차원에서 불가능**하게 만들 수 있습니다.
- **벡터(`Vec<T>`)**: 힙 메모리에 동적으로 크기가 늘어나는 배열.
- **소유권 이동(Move Semantics)**:
  - `self.inventory.push(item)`: `item`의 소유권이 벡터로 이동 → 이후 `item` 사용 불가.
  - `self.inventory.remove(index)`: 아이템의 소유권이 벡터에서 호출자에게 이동.
  - `Option::replace(new)`: 슬롯에 새 값을 넣고 **기존 값을 꺼내서 반환**.
- **인덱싱과 경계 검사**:
  - `vec[i]`는 인덱스 초과 시 panic 발생.
  - 안전한 접근: `vec.get(i)`는 `Option<&T>`를 반환.
- **이터레이터와 클로저**:
  - `for (idx, item) in player.inventory.iter().enumerate()`로 목록 출력.
  - `.iter().filter(|item| matches!(item, Item::Potion(_))).count()`로 포션 개수 세기.
  - `self.weapon.as_ref().map_or(0, |w| w.atk)`: `Option`을 소유권 이동 없이 빌려서 값 꺼내기.

### 3. 코딩 방향 및 가이드
```rust
// 파일: src/main.rs
#[derive(Debug, Clone)]
pub struct Potion { pub name: String, pub heal: u32, pub price: u32 }

#[derive(Debug, Clone)]
pub struct Weapon { pub name: String, pub atk: u32, pub price: u32 }

#[derive(Debug, Clone)]
pub struct Armor { pub name: String, pub def: u32, pub price: u32 }

#[derive(Debug, Clone)]
pub enum Item {
    Potion(Potion),
    Weapon(Weapon),
    Armor(Armor),
}

impl Item {
    pub fn name(&self) -> &str {
        match self {
            Item::Potion(p) => &p.name,
            Item::Weapon(w) => &w.name,
            Item::Armor(a) => &a.name,
        }
    }

    pub fn price(&self) -> u32 {
        match self {
            Item::Potion(p) => p.price,
            Item::Weapon(w) => w.price,
            Item::Armor(a) => a.price,
        }
    }
}
```

`Player` 구조체에 필드를 추가하고(`Player::new`에서는 `Vec::new()`, `None`으로 초기화), 메서드를 구현합니다.

```rust
// 파일: src/main.rs
pub struct Player {
    // ... 기존 필드 ...
    pub inventory: Vec<Item>,
    pub weapon: Option<Weapon>,
    pub armor: Option<Armor>,
}

impl Player {
    pub fn total_atk(&self) -> u32 {
        self.atk + self.weapon.as_ref().map_or(0, |w| w.atk)
    }

    pub fn total_def(&self) -> u32 {
        self.def + self.armor.as_ref().map_or(0, |a| a.def)
    }

    pub fn buy(&mut self, item: Item) -> bool {
        if self.gold < item.price() {
            println!("골드가 부족합니다. (보유: {}G / 가격: {}G)", self.gold, item.price());
            return false;
        }
        self.gold -= item.price();
        println!("{}을(를) 구매했습니다!", item.name());
        self.inventory.push(item); // item의 소유권이 인벤토리로 이동
        true
    }

    pub fn use_item(&mut self, index: usize) {
        if index >= self.inventory.len() {
            println!("유효하지 않은 아이템 번호입니다.");
            return;
        }

        // 소유권을 인벤토리에서 꺼내옴 (Move)
        let item = self.inventory.remove(index);
        match item {
            Item::Potion(potion) => {
                self.hp = (self.hp + potion.heal).min(self.max_hp);
                println!("{}을(를) 사용했습니다! (현재 HP: {}/{})",
                    potion.name, self.hp, self.max_hp);
            } // potion은 여기서 drop되어 사라짐
            Item::Weapon(weapon) => {
                println!("{}을(를) 장착했습니다!", weapon.name);
                // 기존 무기가 있으면 꺼내서 인벤토리로 되돌림
                if let Some(old) = self.weapon.replace(weapon) {
                    println!("{}을(를) 가방에 넣었습니다.", old.name);
                    self.inventory.push(Item::Weapon(old));
                }
            }
            Item::Armor(armor) => {
                println!("{}을(를) 장착했습니다!", armor.name);
                if let Some(old) = self.armor.replace(armor) {
                    println!("{}을(를) 가방에 넣었습니다.", old.name);
                    self.inventory.push(Item::Armor(old));
                }
            }
        }
    }
}
```

> ⚠️ 장비 보너스를 반영하려면 4단계의 `calc_damage(self.atk, target.def)`를 `calc_damage(self.total_atk(), target.total_def())` 형태로 바꿔야 합니다. (6단계 트레이트에서 깔끔하게 정리됩니다.)

**테스트 예시**:
```rust
#[test]
fn equipping_weapon_returns_old_one_to_inventory() {
    let mut player = Player::new(String::from("테스트"));
    player.inventory.push(Item::Weapon(Weapon { name: "나무 검".into(), atk: 2, price: 30 }));
    player.inventory.push(Item::Weapon(Weapon { name: "철 검".into(), atk: 5, price: 80 }));
    player.use_item(0); // 나무 검 장착
    player.use_item(0); // 철 검 장착 → 나무 검은 가방으로
    assert_eq!(player.total_atk(), 15);
    assert_eq!(player.inventory.len(), 1);
    assert_eq!(player.inventory[0].name(), "나무 검");
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 벡터를 `for`로 순회하는 도중 `.remove()`를 호출하면 **E0502** 발생:
  `cannot borrow `v` as mutable because it is also borrowed as immutable`
  → 루프에서는 인덱스만 고르고, 루프가 끝난 뒤 `.remove()`를 수행합니다.
- `buy`에서 `self.inventory.push(item)` 뒤에 `item.name()`을 호출하면 **"borrow of moved value"(E0382)** 에러 → 이동이 일어나기 전에 필요한 값을 먼저 사용하세요.
- `self.weapon.map_or(...)`처럼 `as_ref()` 없이 쓰면 `self.weapon`의 소유권을 꺼내려 해서 **E0507** (cannot move out of ... which is behind a shared reference) 에러가 납니다.

### 5. 완료 체크리스트
- [ ] 상점에서 포션/무기/방어구를 사고, 골드가 부족하면 구매가 거절된다.
- [ ] 포션을 쓰면 HP가 최대 HP를 넘지 않게 회복되고 인벤토리에서 사라진다.
- [ ] 무기를 두 번 장착해도 공격력이 중복으로 쌓이지 않는다.
- [ ] 위 동작을 확인하는 테스트가 있다.

### 6. 응용 과제
- 전투 메뉴에 `[3] 포션 사용`을 추가해 보세요. (포션만 골라 보여주려면 `.iter().enumerate().filter(...)`)
- `vec.get(i)`를 사용해 인덱스 검사(`if index >= len`) 없이 `use_item`을 다시 작성해 보세요.

---

## 6단계: 트레이트(Trait)와 다형성 (전투 엔티티 추상화)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 6단계](STAGE_DECISIONS.md#6단계-트레이트--스킬)를 채우세요: **스킬 도입 여부와 목록**, 트레이트에 포함할 범위, 전투 메뉴 최종안
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- `Player`와 `Monster`의 공통된 전투 로직을 `Combatant` 트레이트로 추상화.
- 4단계에서 두 구조체에 **중복으로 작성한** `is_alive`, `take_damage`, `attack` 메서드를 트레이트와 공용 함수로 옮기는 리팩토링.
- `print_status()`를 표준 트레이트 `Display` 구현으로 대체 (`println!("{}", player)`).

### 2. Rust 핵심 개념
- **트레이트(`trait`)**: 특정 타입이 구현해야 하는 기능(메서드 시그니처)을 정의하는 인터페이스.
- **기본 구현(Default Implementation)**: 트레이트 메서드에 기본 구현을 두어 중복 코드 제거.
- **정적 디스패치 & 트레이트 바운드**: `fn attack<A: Combatant, D: Combatant>(attacker: &A, defender: &mut D)` — 컴파일 시점에 타입별로 코드가 생성됨.
- **동적 디스패치(`dyn Trait`)**: `Vec<Box<dyn Combatant>>`처럼 서로 다른 타입을 한 컬렉션에 담을 때 사용. 실행 시점에 메서드를 찾음.
- **표준 트레이트 구현**: `impl fmt::Display for Player`.
- **라이프타임 생략(Lifetime Elision)**: `fn name(&self) -> &str`는 사실 `fn name<'a>(&'a self) -> &'a str`의 축약입니다. "반환된 `&str`은 `self`가 살아있는 동안만 유효하다"는 뜻이며, 입력 참조가 `&self` 하나라 컴파일러가 자동으로 추론합니다.

### 3. 코딩 방향 및 가이드
```rust
// 파일: src/main.rs
pub trait Combatant {
    fn name(&self) -> &str;
    fn hp(&self) -> u32;
    fn atk(&self) -> u32;
    fn def(&self) -> u32;
    fn take_damage(&mut self, damage: u32);

    // 기본 구현 메서드
    fn is_alive(&self) -> bool {
        self.hp() > 0
    }
}

/// 트레이트 바운드: Combatant를 구현한 타입이라면 무엇이든 서로 공격할 수 있음
pub fn attack<A: Combatant, D: Combatant>(attacker: &A, defender: &mut D) -> u32 {
    let damage = calc_damage(attacker.atk(), defender.def());
    defender.take_damage(damage);
    damage
}

impl Combatant for Player {
    fn name(&self) -> &str { &self.name }
    fn hp(&self) -> u32 { self.hp }
    fn atk(&self) -> u32 { self.total_atk() } // 장비 보너스 포함
    fn def(&self) -> u32 { self.total_def() }
    fn take_damage(&mut self, damage: u32) {
        self.hp = self.hp.saturating_sub(damage);
    }
}

// impl Combatant for Monster { ... } 도 같은 방식으로 작성

impl std::fmt::Display for Player {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        writeln!(f, "====== [{}] 상태창 ======", self.name)?;
        write!(
            f,
            "Lv.{} | HP: {}/{} | MP: {}/{} | ATK: {} | DEF: {} | EXP: {} | Gold: {}G",
            self.level, self.hp, self.max_hp, self.mp, self.max_mp,
            self.total_atk(), self.total_def(), self.exp, self.gold
        )
    }
}
```

**리팩토링 순서**:
1. `Combatant` 트레이트와 `attack` 함수를 추가하고, `Player`/`Monster`에 `impl Combatant`를 작성한다.
2. 4단계에서 만든 `Player`/`Monster`의 고유 메서드 `is_alive`, `take_damage`, `attack`을 **삭제**한다. (남겨두면 같은 이름의 고유 메서드가 트레이트 메서드보다 우선 호출되어 혼란스럽습니다.)
3. `execute_combat`의 `player.attack(monster)`를 `attack(player, monster)`로, `monster.attack(player)`를 `attack(monster, player)`로 바꾼다.
4. `print_status()` 호출을 `println!("{}", player)`로 바꾸고 `print_status`를 삭제한다.
5. `cargo test`로 기존 테스트가 그대로 통과하는지 확인한다. (리팩토링에서 테스트가 가장 빛나는 순간입니다.)

### 4. 주의할 점 & 컴파일 에러 포인트
- 트레이트 메서드를 호출하려면 그 트레이트가 **스코프 안에 있어야** 합니다. 지금은 한 파일이라 문제없지만, 7단계에서 모듈을 나누면 `use crate::combat::Combatant;`가 없는 파일에서 `player.is_alive()` 호출 시 "no method named `is_alive` found" 에러가 납니다.
- 트레이트에 **제네릭 메서드**(`fn attack<T: Combatant>(&self, target: &mut T)`)를 넣으면 그 트레이트는 `dyn Combatant`로 쓸 수 없게 됩니다(dyn 호환성). 그래서 예시에서는 `attack`을 트레이트 밖의 자유 함수로 두었습니다.

### 5. 완료 체크리스트
- [ ] `Player`/`Monster`에 중복된 전투 메서드가 남아 있지 않다.
- [ ] `println!("{}", player)`로 상태창이 출력된다.
- [ ] 리팩토링 전후로 `cargo test`가 모두 통과한다.

### 6. 응용 과제
- `Vec<Box<dyn Combatant>>`에 플레이어와 몬스터를 함께 담고, 반복문으로 이름과 HP를 출력해 보세요.
- `impl Display for Monster`도 구현해 보세요.

---

## 7단계: 모듈 분리 & 던전 진행 (최종 보스 및 게임 클리어)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 7단계](STAGE_DECISIONS.md#7단계-던전--최종-보스)를 채우세요: **던전 진행 규칙**, 층별 몬스터 표, 보스 패턴, 중간 보스·이벤트, 밸런스 검증표
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- 커진 `main.rs`를 역할별 파일(모듈)로 분리.
- 1층부터 5층까지 진행되는 `Dungeon` 시스템:
  - 1층: 슬라임 (초급)
  - 2층: 고블린 (초급)
  - 3층: 오크 (중급)
  - 4층: 해골 기사 (중급)
  - 5층: **고대 화염룡 (최종 보스)**
- 층의 몬스터를 처치하면 다음 층이 열림 (층 진행 세부 규칙은 `D7-1` ~ `D7-4`에서 결정).
- 5층 보스 처치 시 엔딩 및 게임 클리어 출력.
- 전체 흐름을 주관하는 `Game` 구조체로 상태 관리 정리.

### 2. Rust 핵심 개념
- **모듈 시스템(`mod`, `pub`, `use`, `crate`, `super`)**:
  - `main.rs`에 `mod player;`를 선언하면 컴파일러가 `src/player.rs`를 찾아 모듈로 포함.
  - 다른 모듈의 항목은 `pub`이어야 접근 가능. 크레이트 내부에만 공개하려면 `pub(crate)`.
  - `use crate::player::Player;`로 경로를 가져옴.
- **캡슐화**: `Game`의 필드를 `pub` 없이 두어 외부에서는 `run()`으로만 게임을 진행하게 만듦.
- **`Option` 반환으로 불가능한 입력 처리**: 존재하지 않는 층이면 아무 몬스터나 만들지 말고 `None` 반환.
- **`let ... else`**: `Option`이 `None`이면 즉시 빠져나가는 패턴.

### 3. 코딩 방향 및 가이드
**모듈 구조** ([GAME_DESIGN.md §5](GAME_DESIGN.md#5-rust-아키텍처-및-모듈-구성)와 동일):
```rust
// 파일: src/main.rs
mod combat;    // Combatant, attack, calc_damage, BattleResult, execute_combat
mod dungeon;   // Dungeon
mod game;      // GameState, Game
mod io_helper; // read_line, read_choice
mod item;      // Item, Potion, Weapon, Armor
mod monster;   // Monster
mod player;    // Player

fn main() {
    let name = io_helper::read_line("모험가의 이름을 입력하세요 > ");
    let mut game = game::Game::new(player::Player::new(name), dungeon::Dungeon::new());
    game.run();
}
```

```rust
// 파일: src/dungeon.rs
use crate::monster::Monster;

pub struct Dungeon {
    pub current_floor: u32,
    pub max_floor: u32,
}

impl Dungeon {
    pub fn new() -> Self {
        Self { current_floor: 1, max_floor: 5 }
    }

    pub fn spawn_monster(&self) -> Option<Monster> {
        match self.current_floor {
            1 => Some(Monster::new("슬라임", 20, 5, 1, 15, 20)),
            2 => Some(Monster::new("고블린", 35, 8, 2, 30, 40)),
            3 => Some(Monster::new("오크", 55, 12, 4, 60, 70)),
            4 => Some(Monster::new("해골 기사", 80, 16, 6, 100, 120)),
            5 => Some(Monster::boss("고대 화염룡", 150, 25, 10, 500, 1000)),
            _ => None,
        }
    }

    pub fn is_last_floor(&self) -> bool {
        self.current_floor == self.max_floor
    }

    pub fn advance(&mut self) {
        if self.current_floor < self.max_floor {
            self.current_floor += 1;
        }
    }
}
```

```rust
// 파일: src/game.rs
use crate::combat::{execute_combat, BattleResult};
use crate::dungeon::Dungeon;
use crate::player::Player;

pub struct Game {
    player: Player,   // pub이 아님 → 외부에서 직접 접근 불가
    dungeon: Dungeon,
    state: GameState,
}

impl Game {
    // new(), run(), town() 은 3단계의 main 루프 / handle_town_menu를 옮겨 온 것

    fn explore(&mut self) -> GameState {
        println!("\n--- 던전 {}층 ---", self.dungeon.current_floor);
        let Some(mut monster) = self.dungeon.spawn_monster() else {
            return GameState::Town;
        };

        match execute_combat(&mut self.player, &mut monster) {
            BattleResult::Victory if monster.is_boss => GameState::Victory,
            BattleResult::Victory => {
                if !self.dungeon.is_last_floor() {
                    self.dungeon.advance();
                }
                GameState::Town
            }
            BattleResult::Fled => GameState::Town,
            BattleResult::Defeat => GameState::GameOver,
        }
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 모듈을 분리하면 **비공개(private) 항목 접근 에러(E0603, E0616)** 가 쏟아집니다. 하나씩 읽으며 "이건 정말 외부에 공개해야 하나?"를 판단하는 것이 이 단계의 핵심 학습입니다.
- `mod` 선언은 **`main.rs`(크레이트 루트)에 한 번만** 합니다. 다른 파일에서는 `mod`가 아니라 `use crate::...`로 가져옵니다.
- 트레이트 메서드를 쓰는 파일마다 `use crate::combat::Combatant;`가 필요합니다.

### 5. 완료 체크리스트
- [ ] `main.rs`에는 `mod` 선언과 짧은 `main` 함수만 남았다.
- [ ] 1층부터 5층까지 진행하고, 보스를 잡으면 엔딩이 출력된다.
- [ ] `cargo test`가 통과한다. (테스트도 각 모듈 파일 안으로 옮겼다.)
- [ ] `cargo clippy` 경고가 없다.

### 6. 응용 과제
- 같은 층에서 여러 종류 중 무작위 몬스터가 나오도록 `rand::random_range`를 사용해 보세요.
- 보스전에서만 특수 행동(예: 3턴마다 강한 공격)을 하도록 만들어 보세요.

---

## 8단계: 에러 핸들링 & 파일 I/O (세이브 / 로드 시스템)

### 0. 시작 전: 결정 & 기획
> 🔀 코딩 전에 [STAGE_DECISIONS.md › 8단계](STAGE_DECISIONS.md#8단계-세이브--로드)를 채우세요: 저장 시점, 슬롯, 사망 시 세이브 처리, 저장 데이터 명세, 타이틀 메뉴
> 아래 예시 코드는 각 항목의 `기본안`을 따릅니다. 다르게 결정했다면 결정에 맞게 바꿔서 구현합니다.

### 1. 목표 기능
- 플레이어 진행 상황(스탯, 골드, 인벤토리, 장비, 던전 진행도)을 `savegame.json`으로 저장.
- 게임 시작 시 `[이어하기]`로 직전 저장 시점부터 플레이.
- 세이브 파일이 없거나 깨졌을 때 프로그램이 죽지 않고 안내 메시지 출력 후 새 게임 시작.

### 2. Rust 핵심 개념
- **직렬화/역직렬화 (`serde`, `serde_json`)**: 구조체를 JSON 문자열로 변환하고 복원.
  - 저장할 **모든 타입**(`Player`, `Item`, `Potion`, `Weapon`, `Armor`, `Dungeon`)에 `#[derive(Serialize, Deserialize)]`가 필요합니다.
- **파일 I/O**: `std::fs::write`, `std::fs::read_to_string`.
- **커스텀 에러 타입**: 여러 종류의 에러(`io::Error`, `serde_json::Error`)를 하나의 `enum SaveError`로 묶기.
- **`?` 연산자와 `From` 트레이트**: `From<io::Error> for SaveError`를 구현하면 `?`가 에러를 자동 변환하여 전파.
- **에러 종류별 분기**: `e.kind() == io::ErrorKind::NotFound`로 "파일 없음"과 "파일 깨짐"을 구분.

### 3. 코딩 방향 및 가이드
**의존성 추가** (터미널):
```bash
cargo add serde --features derive
```
```bash
cargo add serde_json
```
→ `Cargo.toml`에 `serde = { version = "1", features = ["derive"] }`, `serde_json = "1"`이 추가됩니다. `features = ["derive"]`가 없으면 `#[derive(Serialize)]`를 찾을 수 없다는 에러가 납니다.

**저장 대상 타입에 derive 추가**:
```rust
// 파일: src/player.rs (item.rs, dungeon.rs도 동일하게)
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct Player { /* ... */ }
```

```rust
// 파일: src/save.rs
use std::fmt;
use std::fs;

use serde::{Deserialize, Serialize};

use crate::dungeon::Dungeon;
use crate::player::Player;

pub const SAVE_PATH: &str = "savegame.json";

#[derive(Serialize, Deserialize)]
pub struct SaveData {
    pub player: Player,
    pub dungeon: Dungeon,
}

#[derive(Debug)]
pub enum SaveError {
    Io(std::io::Error),
    Json(serde_json::Error),
}

impl fmt::Display for SaveError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            SaveError::Io(e) => write!(f, "파일 입출력 오류: {}", e),
            SaveError::Json(e) => write!(f, "세이브 데이터 형식 오류: {}", e),
        }
    }
}

impl std::error::Error for SaveError {}

// From 구현 덕분에 `?`가 io::Error / serde_json::Error를 SaveError로 자동 변환해 줌
impl From<std::io::Error> for SaveError {
    fn from(e: std::io::Error) -> Self {
        SaveError::Io(e)
    }
}

impl From<serde_json::Error> for SaveError {
    fn from(e: serde_json::Error) -> Self {
        SaveError::Json(e)
    }
}

impl SaveData {
    pub fn save_to_file(&self, path: &str) -> Result<(), SaveError> {
        let json = serde_json::to_string_pretty(self)?;
        fs::write(path, json)?;
        Ok(())
    }

    pub fn load_from_file(path: &str) -> Result<Self, SaveError> {
        let content = fs::read_to_string(path)?;
        let data = serde_json::from_str(&content)?;
        Ok(data)
    }
}

/// 이어하기: 실패 원인에 따라 다른 안내를 출력
pub fn try_load() -> Option<SaveData> {
    match SaveData::load_from_file(SAVE_PATH) {
        Ok(data) => Some(data),
        Err(SaveError::Io(e)) if e.kind() == std::io::ErrorKind::NotFound => {
            println!("저장된 게임이 없습니다. 새 게임을 시작합니다.");
            None
        }
        Err(e) => {
            println!("세이브 파일을 불러오지 못했습니다: {}", e);
            None
        }
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 구조체 안의 필드 타입 하나라도 `Serialize`를 구현하지 않으면 "the trait bound `Item: Serialize` is not satisfied" 에러가 납니다. 에러가 가리키는 타입에 derive를 추가하세요.
- `Game`이 `player`를 소유하고 있으므로, 저장할 때 `SaveData`로 옮기려면 소유권 이동(`fn into_save_data(self) -> SaveData`) 또는 참조를 담는 구조체가 필요합니다. 어떤 방식이 좋을지 직접 고민해 보세요.
- `savegame.json`은 플레이 데이터이므로 `.gitignore`에 추가해 커밋되지 않게 합니다.

### 5. 완료 체크리스트
- [ ] 저장 후 종료하고 다시 실행하면 이어하기가 된다.
- [ ] `savegame.json`을 지우거나 내용을 망가뜨려도 프로그램이 패닉 없이 새 게임을 시작한다.
- [ ] 저장 → 불러오기 왕복 테스트가 있다. (테스트에서는 `std::env::temp_dir()` 경로 사용)

### 6. 응용 과제
- `SaveError`와 `From` 구현을 직접 작성하는 대신 `thiserror` 크레이트로 바꿔 보고 코드량을 비교해 보세요.

---

## 💡 개발 사이클 팁
1. **작게 시작하고 바로 컴파일**: 기능을 한 줄, 한 함수 추가할 때마다 `cargo check`를 실행하여 컴파일러의 조언을 듣습니다.
2. **컴파일러 에러 메시지는 친절한 선생님**: Rust 컴파일러(`rustc`)는 문제의 원인뿐만 아니라 해결 코드 제안(`help: consider ...`)까지 정확하게 안내합니다. 에러 코드(`E0502` 등)는 `rustc --explain E0502`로 자세한 설명을 볼 수 있습니다.
3. **경고(Warnings) 무시하지 않기**: 사용되지 않는 변수, 불필요한 `mut` 등 컴파일러 경고를 깔끔하게 정리하며 진행합니다.
4. **4단계부터는 테스트와 함께**: 규칙(데미지, 회복, 장착 등)을 만들면 `#[test]`를 하나씩 추가하고 `cargo test`로 확인합니다. 6~7단계 리팩토링 때 큰 도움이 됩니다.
5. **단계를 마칠 때 3종 세트**: `cargo fmt`(코드 정렬) → `cargo clippy`(관용적 코드 제안) → `cargo test`. 그리고 단계별로 git 커밋을 남깁니다.
