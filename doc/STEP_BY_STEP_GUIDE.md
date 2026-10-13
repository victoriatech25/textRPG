# Rust Text RPG 단계별 구현 및 학습 상세 가이드

본 문서는 Rust 문법을 체계적으로 학습하면서 Text RPG를 완성해 나갈 수 있도록, **각 단계별 목표 기능, Rust 핵심 개념, 코딩 가이드라인, 코드 예시, 주의해야 할 컴파일 에러**를 상세히 정리한 가이드입니다.

---

## 📑 목차
1. [1단계: 프로젝트 기초 & 표준 입출력 (기본 루프)](#1단계-프로젝트-기초--표준-입출력-기본-루프)
2. [2단계: 구조체(Struct)와 메서드 (캐릭터 스탯 관리)](#2단계-구조체struct와-메서드-캐릭터-스탯-관리)
3. [3단계: 열거형(Enum)과 패턴 매칭 (게임 상태 머신 & 입력 검증)](#3단계-열거형enum과-패턴-매칭-게임-상태-머신--입력-검증)
4. [4단계: 소유권(Ownership) & 대여(Borrowing) (몬스터 & 턴제 전투)](#4단계-소유권ownership--대여borrowing-몬스터--턴제-전투)
5. [5단계: 컬렉션(Vec) & 소유권 이동 (인벤토리 & 아이템 소비)](#5단계-컬렉션vec--소유권-이동-인벤토리--아이템-소비)
6. [6단계: 트레이트(Trait)와 다형성 (전투 엔티티 추상화)](#6단계-트레이트trait와-다형성-전투-엔티티-추상화)
7. [7단계: 모듈 분리 & 던전 진행 (최종 보스 및 게임 클리어)](#7단계-모듈-분리--던전-진행-최종-보스-및-게임-클리어)
8. [8단계: 에러 핸들링 & 파일 I/O (세이브 / 로드 시스템)](#8단계-에러-핸들링--파일-io-세이브--로드-시스템)

---

## 1단계: 프로젝트 기초 & 표준 입출력 (기본 루프)

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
- `io_helper` 성격의 사용자 입력 함수를 별도로 만들어두면 재사용하기 좋습니다.
- 입력을 받을 때 줄바꿈(`\n`, `\r\n`)이 포함되므로 반드시 `.trim()`을 호출해야 합니다.

```rust
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

---

## 2단계: 구조체(Struct)와 메서드 (캐릭터 스탯 관리)

### 1. 목표 기능
- `Player` 구조체 정의: 이름, 레벨, 현재 HP, 최대 HP, 공격력(ATK), 방어력(DEF), 소지 골드, 경험치.
- 플레이어 생성 연관 함수 (`Player::new(name)`).
- 플레이어 상태를 보여주는 `print_status()` 메서드 구현.
- 메인 메뉴에 `[상태 확인]` 추가하여 현재 스탯 출력.

### 2. Rust 핵심 개념
- **구조체(`struct`)**: 데이터 필드를 묶는 사용자 정의 타입.
- **`impl` 블록과 연관 함수(Associated Function)**:
  - `Player::new(...)`: 인스턴스 없이 호출하는 생성자 패턴 (C++/Java의 static 메서드와 유사).
- **메서드와 `self` 파라미터**:
  - `&self`: 인스턴스를 읽기 전용(불변 참조)으로 빌려옴 (예: `print_status`).
  - `&mut self`: 인스턴스의 내부 상태를 수정하기 위해 가변으로 빌려옴 (예: `take_damage`, `heal`).
  - `self`: 인스턴스의 소유권을 가져와 소비함.
- **파생 어트리뷰트(`#[derive(Debug)]`)**: `{:?}`로 구조체 내부 필드를 한 번에 디버그 출력할 수 있게 해줌.

### 3. 코딩 방향 및 가이드
```rust
#[derive(Debug)]
pub struct Player {
    pub name: String,
    pub level: u32,
    pub hp: u32,
    pub max_hp: u32,
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
            atk: 10,
            def: 2,
            gold: 100,
            exp: 0,
        }
    }

    pub fn print_status(&self) {
        println!("====== [{}] 상태창 ======", self.name);
        println!("Lv.{} | HP: {}/{} | ATK: {} | DEF: {} | Gold: {}G",
            self.level, self.hp, self.max_hp, self.atk, self.def, self.gold);
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 구조체 필드 이름과 매개변수 이름이 같으면 `name: name` 대신 `name`으로 축약(Field Init Shorthand) 가능.
- 인스턴스를 변경하는 메서드를 호출할 때 변수가 `let player = ...`로 선언되어 있으면 호출 불가 $\rightarrow$ `let mut player = ...`로 선언해야 함.

---

## 3단계: 열거형(Enum)과 패턴 매칭 (게임 상태 머신 & 입력 검증)

### 1. 목표 기능
- 게임의 현재 화면/모드를 `GameState` 열거형으로 관리:
  - `Town`(마을), `Dungeon`(던전), `Battle`(전투), `Inventory`(가방), `GameOver`, `Victory`
- 플레이어의 메뉴 번호 입력을 안전하게 파싱하여 `match`로 상태 전환.
- 유효하지 않은 번호 입력 시 에러 안내 후 다시 입력받기.

### 2. Rust 핵심 개념
- **열거형(`enum`)**: 가능한 여러 상태 중 단 하나만을 표현하는 타입. Rust의 enum은 각 변형(variant)에 데이터를 포함할 수도 있음.
- **`match` 표현식과 완전성(Exhaustiveness)**: 모든 경우의 수를 처리하지 않으면 컴파일 거부 $\rightarrow$ 버그 원천 차단.
- **`Option<T>`과 `Result<T, E>`**:
  - `Some(value)` / `None`: null 대신 사용하는 안전한 컨테이너.
  - `Ok(value)` / `Err(e)`: 연산 성공/실패 표현.
- **문자열 정수 파싱**: `input.parse::<u32>()`의 반환값 `Result` 처리 (`match` 또는 `if let`).

### 3. 코딩 방향 및 가이드
```rust
pub enum GameState {
    Town,
    Dungeon,
    Battle,
    Quit,
}

fn handle_town_menu() -> GameState {
    println!("\n[마을 거점]");
    println!("1. 던전 입장");
    println!("2. 여관에서 휴식 (HP 회복)");
    println!("3. 상태 확인");
    println!("4. 게임 종료");

    let input = read_line("선택 > ");
    match input.as_str() {
        "1" => GameState::Dungeon,
        "2" => {
            println!("여관에서 푹 쉬었습니다. HP가 모두 회복되었습니다!");
            GameState::Town
        }
        "3" => GameState::Town,
        "4" => GameState::Quit,
        _ => {
            println!("잘못된 번호입니다. 다시 선택해주세요.");
            GameState::Town
        }
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- `match`에서 모든 케이스를 나열하지 않거나 기본 케이스 와일드카드(`_`)를 빠뜨리면 "non-exhaustive patterns" 컴파일 에러 발생.

---

## 4단계: 소유권(Ownership) & 대여(Borrowing) (몬스터 & 턴제 전투)

### 1. 목표 기능
- `Monster` 구조체 정의 (이름, HP, MaxHP, ATK, DEF, 보상 EXP, 보상 Gold).
- 턴제 전투 루프:
  1. 플레이어 턴: 일반 공격 / 방어 / 도망
  2. 몬스터 턴: 플레이어에게 반격
- 데미지 계산 및 사망/승리 판정, 보상 지급.
- `rand` 크레이트를 사용한 데미지 편차(난수) 도입.

### 2. Rust 핵심 개념
- **Rust의 3대 소유권 규칙**:
  1. 각 값은 단 하나의 소유자(Owner) 변수를 가진다.
  2. 소유자가 스코프를 벗어나면 값은 버려진다(`drop`).
  3. 한 번에 하나의 가변 참조(`&mut T`) 또는 여러 개의 불변 참조(`&T`)만 존재할 수 있다 (데이터 레이스 방지).
- **빌림(Borrowing)**:
  - 전투 함수 `battle(player: &mut Player, monster: &mut Monster)` 처럼 소유권을 넘기지 않고 대여하여 상태 변경.
- **외부 크레이트 의존성 추가**: `Cargo.toml`에 `rand = "0.8"` 추가 및 사용.
- **모듈 분리 기초**: `mod player;`, `mod monster;`, `mod combat;` 분리.

### 3. 코딩 방향 및 가이드
```rust
// combat.rs
use crate::player::Player;
use crate::monster::Monster;

pub fn execute_combat(player: &mut Player, monster: &mut Monster) -> bool {
    println!("\n야생의 [{}]이(가) 나타났다!", monster.name);

    while player.is_alive() && monster.is_alive() {
        println!("\n[플레이어 HP: {}/{}]  VS  [{} HP: {}/{}]",
            player.hp, player.max_hp, monster.name, monster.hp, monster.max_hp);
        println!("1. 공격  |  2. 도망");

        let choice = read_line("행동 선택 > ");
        match choice.as_str() {
            "1" => {
                let dmg = player.attack(monster);
                println!("{}에게 {}의 피해를 입혔습니다!", monster.name, dmg);
                if !monster.is_alive() { break; }

                let monster_dmg = monster.attack(player);
                println!("{}의 반격! {}의 피해를 받았습니다!", monster.name, monster_dmg);
            }
            "2" => {
                println!("전투에서 도망쳤습니다!");
                return false;
            }
            _ => println!("잘못된 입력입니다."),
        }
    }

    if player.is_alive() {
        println!("\n승리했습니다! {} EXP와 {} Gold를 획득합니다.", monster.reward_exp, monster.reward_gold);
        player.gain_rewards(monster.reward_exp, monster.reward_gold);
        true
    } else {
        println!("\n패배했습니다... 눈앞이 캄캄해집니다.");
        false
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 한 스코프 내에서 같은 변수를 `&player`와 `&mut player`로 동시에 빌리려고 하면 발생하는 빌림 검사기(Borrow Checker) 에러:
  `cannot borrow *player as mutable because it is also borrowed as immutable`

---

## 5단계: 컬렉션(Vec) & 소유권 이동 (인벤토리 & 아이템 소비)

### 1. 목표 기능
- 아이템 타입 정의: `ItemType::Potion`, `ItemType::Weapon`, `ItemType::Armor`.
- `Item` 구조체 및 `Inventory` (`Vec<Item>`) 구현.
- 마을 상점에서 골드로 포션 구매하기.
- 인벤토리 열기 $\rightarrow$ 포션 선택 $\rightarrow$ 사용 시 HP 회복 후 인벤토리에서 제거(소비).

### 2. Rust 핵심 개념
- **벡터(`Vec<T>`)**: 힙 메모리에 동적으로 크기가 늘어나는 배열.
- **소유권 이동(Move Semantics)**:
  - `inventory.items.remove(index)`를 호출하면 해당 아이템의 소유권이 호출자에게 이동.
- **인덱싱과 경계 검사**:
  - `vec[i]`는 인덱스 초과 시 panic 발생.
  - 안전한 접근: `vec.get(i)`는 `Option<&T>`를 반환하여 안전하게 처리 가능.
- **이터레이터(`Iterator`)**:
  - `for (idx, item) in player.inventory.iter().enumerate()` 패턴으로 목록 출력.

### 3. 코딩 방향 및 가이드
```rust
#[derive(Clone, Debug)]
pub enum Item {
    HealthPotion { name: String, heal_amount: u32, price: u32 },
    Sword { name: String, bonus_atk: u32, price: u32 },
}

impl Player {
    pub fn use_item(&mut self, index: usize) {
        if index < self.inventory.len() {
            // 소유권을 인벤토리에서 꺼내옴 (Move)
            let item = self.inventory.remove(index);
            match item {
                Item::HealthPotion { name, heal_amount, .. } => {
                    self.hp = (self.hp + heal_amount).min(self.max_hp);
                    println!("{}을(를) 사용하여 HP가 {} 회복되었습니다! (현재 HP: {}/{})",
                        name, heal_amount, self.hp, self.max_hp);
                }
                Item::Sword { name, bonus_atk, .. } => {
                    self.atk += bonus_atk;
                    println!("{}을(를) 장착했습니다! 공격력이 +{} 상승했습니다.", name, bonus_atk);
                }
            }
        } else {
            println!("유효하지 않은 아이템 번호입니다.");
        }
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 벡터 순회 도중 요소를 수정하거나 삭제하려고 하면 "cannot borrow as mutable more than once" 에러 발생 $\rightarrow$ 인덱스를 먼저 선택받고 루프 밖에서 `.remove()` 수행.

---

## 6단계: 트레이트(Trait)와 다형성 (전투 엔티티 추상화)

### 1. 목표 기능
- `Player`와 `Monster`의 공통된 전투 로직을 `Combatant` 트레이트로 추상화.
- 일반화된 데미지 계산 함수 및 사망 판정 공통화.

### 2. Rust 핵심 개념
- **트레이트(`trait`)**: 특정 타입이 구현해야 하는 기능(메서드 시그니처)을 정의하는 인터페이스.
- **기본 구현(Default Implementation)**: 트레이트 메서드에 기본 구현을 두어 중복 코드 제거.
- **정적 디스패치 & 트레이트 바운드**:
  - `fn attack_target<T: Combatant>(attacker: &impl Combatant, target: &mut T)`

### 3. 코딩 방향 및 가이드
```rust
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

    fn calculate_damage_against(&self, defender_def: u32) -> u32 {
        if self.atk() > defender_def {
            self.atk() - defender_def
        } else {
            1 // 최소 데미지 1
        }
    }
}

impl Combatant for Player {
    fn name(&self) -> &str { &self.name }
    fn hp(&self) -> u32 { self.hp }
    fn atk(&self) -> u32 { self.atk }
    fn def(&self) -> u32 { self.def }
    fn take_damage(&mut self, damage: u32) {
        self.hp = self.hp.saturating_sub(damage);
    }
}
```

### 4. 주의할 점 & 컴파일 에러 포인트
- 트레이트의 메서드를 사용하려면 해당 트레이트가 스코프 안에 임포트(`use crate::combat::Combatant;`)되어 있어야 호출 가능.

---

## 7단계: 모듈 분리 & 던전 진행 (최종 보스 및 게임 클리어)

### 1. 목표 기능
- 1층부터 5층까지 진행되는 `Dungeon` 시스템:
  - 1층: 슬라임 (초급)
  - 2층: 고블린
  - 3층: 오크
  - 4층: 스켈레톤 나이트 (엘리트)
  - 5층: **드래곤 (최종 보스)**
- 층 클리어 시 다음 층 전진 또는 마을 귀환 선택.
- 5층 보스 처치 시 대망의 엔딩 크레딧 및 게임 클리어 출력!

### 2. Rust 핵심 개념
- **모듈 시스템(`mod`, `pub`, `use`, `crate`, `super`)**:
  - 캡슐화와 접근 제어자(`pub(crate)`, `pub`).
- **상태 관리의 완성도**: 전체 흐름을 주관하는 `Game` 구조체 생성 및 오케스트레이션.

### 3. 코딩 방향 및 가이드
```rust
pub struct Dungeon {
    pub current_floor: u32,
    pub max_floor: u32,
}

impl Dungeon {
    pub fn new() -> Self {
        Self { current_floor: 1, max_floor: 5 }
    }

    pub fn spawn_monster_for_current_floor(&self) -> Monster {
        match self.current_floor {
            1 => Monster::new("초록 슬라임", 20, 5, 1, 15, 20),
            2 => Monster::new("동굴 고블린", 35, 8, 2, 30, 40),
            3 => Monster::new("돌도끼 오크", 55, 12, 4, 60, 70),
            4 => Monster::new("해골 기사", 80, 16, 6, 100, 120),
            5 => Monster::boss("고대 화염룡", 150, 25, 10, 500, 1000),
            _ => Monster::new("길 잃은 방랑자", 10, 1, 0, 5, 5),
        }
    }
}
```

---

## 8단계: 에러 핸들링 & 파일 I/O (세이브 / 로드 시스템)

### 1. 목표 기능
- 플레이어 진행 상황(스탯, 소지 골드, 장비, 던전 진행도)을 `savegame.json`으로 저장.
- 타이틀 화면에서 `[이어하기]`를 선택하여 직전 저장 시점부터 플레이.

### 2. Rust 핵심 개념
- **직렬화/역직렬화 (`serde`, `serde_json`)**: 구조체를 JSON 문자열로 변환하고 복원.
- **파일 I/O**: `std::fs::File`, `std::fs::read_to_string`.
- **`Result<T, E>`와 `?` 연산자**: 에러 전파(Error propagation)의 우아한 패턴.

### 3. 코딩 방향 및 가이드
```rust
use serde::{Serialize, Deserialize};
use std::fs;

#[derive(Serialize, Deserialize, Debug)]
pub struct SaveData {
    pub player: Player,
    pub dungeon_floor: u32,
}

impl SaveData {
    pub fn save_to_file(&self, path: &str) -> Result<(), std::io::Error> {
        let json = serde_json::to_string_pretty(self)?;
        fs::write(path, json)?;
        Ok(())
    }

    pub fn load_from_file(path: &str) -> Result<Self, Box<dyn std::error::Error>> {
        let content = fs::read_to_string(path)?;
        let data: SaveData = serde_json::from_str(&content)?;
        Ok(data)
    }
}
```

---

## 💡 개발 사이클 팁
1. **작게 시작하고 바로 컴파일**: 기능을 한 줄, 한 함수 추가할 때마다 `cargo check`를 실행하여 컴파일러의 조언을 듣습니다.
2. **컴파일러 에러 메시지는 친절한 선생님**: Rust 컴파일러(`rustc`)는 문제의 원인뿐만 아니라 해결 코드 제안(`help: consider ...`)까지 정확하게 안내합니다.
3. **경고(Warnings) 무시하지 않기**: 사용되지 않는 변수, 불필요한 `mut` 등 컴파일러 경고를 깔끔하게 정리하며 진행합니다.
