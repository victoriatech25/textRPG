# 🦀 단계별 Rust 핵심 개념 가이드 (Rust Concepts Guide)

본 문서는 **Rust Text RPG**를 단계별로 제작하면서 마주하게 되는 **Rust의 핵심 문법, 철학, 메모리 모델 및 관용적 패턴(Idiomatic Patterns)**을 개념별로 깊이 있게 이해하고 학습할 수 있도록 정리한 가이드입니다.

코딩 중 문법적 의문이 생기거나 컴파일러 에러의 원인이 궁금할 때 언제든 이 문서를 참조하세요.

---

## 📑 목차
- [1단계: 기초 문법 & 표준 입출력](#1단계-기초-문법--표준-입출력)
  - 1.1 변수와 가변성 (`let` vs `let mut`, 섀도잉)
  - 1.2 문자열의 두 얼굴: `String`과 `&str`
  - 1.3 표준 입출력과 버퍼 플러시 (`stdin`, `stdout`)
  - 1.4 에러 처리 기초 맛보기 (`Result`, `.expect()`)
  - 1.5 반복문과 제어 (`loop`, `break`)
- [2단계: 데이터 모델링 - 구조체(Struct)와 메서드](#2단계-데이터-모델링---구조체struct와-메서드)
  - 2.1 구조체(Struct)의 정의와 인스턴스화
  - 2.2 `impl` 블록: 연관 함수(생성자)와 메서드
  - 2.3 `self`, `&self`, `&mut self`의 선택 기준
  - 2.4 파생 속성(Derive Attributes)과 디버그 출력
- [3단계: 타입 기반 상태 제어 - 열거형(Enum) & 패턴 매칭](#3단계-타입-기반-상태-제어---열거형enum--패턴-매칭)
  - 3.1 대수적 데이터 타입: 열거형(`enum`)
  - 3.2 패턴 매칭 `match`와 완전성 검사(Exhaustive Checking)
  - 3.3 Null 없는 안전한 세상: `Option<T>`
  - 3.4 문자열 파싱과 에러 핸들링 기초 (`parse`, `match Result`)
- [4단계: Rust의 심장 - 소유권(Ownership) & 대여(Borrowing)](#4단계-rust의-심장---소유권ownership--대여borrowing)
  - 4.1 소유권(Ownership)의 3대 원칙
  - 4.2 이동(Move)과 복사(Copy)
  - 4.3 대여(Borrowing)와 참조 규칙 (Borrow Checker)
  - 4.4 수치 연산과 언더플로우 방지 (`saturating_sub`)
  - 4.5 단위 테스트(Unit Test) 작성 기초
- [5단계: 컬렉션 & 데이터 복합화 - Vec & 소유권 이동](#5단계-컬렉션--데이터-복합화---vec--소유권-이동)
  - 5.1 동적 배열 `Vec<T>`의 동작 원리와 메모리
  - 5.2 안전한 인덱싱 (`v[i]` vs `v.get(i)`)
  - 5.3 데이터를 품은 열거형 (Enum with Data)
  - 5.4 소유권 교체 기법 (`Option::take`, `Option::replace`)
  - 5.5 반복자(Iterator)와 클로저(Closure) 기초
- [6단계: 추상화와 다형성 - 트레이트(Trait)](#6단계-추상화와-다형성---트레이트trait)
  - 6.1 트레이트(Trait)의 정의와 구현
  - 6.2 기본 구현(Default Implementation)
  - 6.3 정적 디스패치(제네릭) vs 동적 디스패치(`dyn Trait`)
  - 6.4 표준 라이브러리 트레이트 (`Display`, `Clone`, `Default`)
- [7단계: 대규모 모듈 구성 - 모듈 시스템(Module System)](#7단계-대규모-모듈-구성---모듈-시스템module-system)
  - 7.1 Rust 모듈 트리 (`crate`, `mod`, `use`, `super`)
  - 7.2 파일/디렉터리 분리 규칙 (Modern Rust 방식)
  - 7.3 가시성(Visibility)과 캡슐화 (`pub`, `pub(crate)`)
  - 7.4 간결한 제어 흐름: `let ... else` 구문
- [8단계: 견고한 영속화 - 에러 전파 & 파일 I/O & 직렬화](#8단계-견고한-영속화---에러-전파--파일-io--직렬화)
  - 8.1 에러 전파 연산자 `?`와 `From` 트레이트
  - 8.2 커스텀 에러 열거형 설계
  - 8.3 파일 읽기/쓰기 (`std::fs`)
  - 8.4 `serde`와 `serde_json`을 통한 직렬화/역직렬화
- [부록: Rust 개발 도구 & 유용한 명령어](#부록-rust-개발-도구--유용한-명령어)

---

## 1단계: 기초 문법 & 표준 입출력

### 1.1 변수와 가변성 (`let` vs `let mut`, 섀도잉)
- **기본 불변성(Immutability)**:
  Rust에서 `let x = 5;`로 선언한 변수는 기본적으로 **불변(immutable)**입니다. 값을 변경하려고 하면 컴파일 타임 에러가 발생합니다.
- **가변성 선언 (`mut`)**:
  변수의 값을 재할당하거나 변경해야 하는 경우 명시적으로 `let mut x = 5;`를 선언해야 합니다.
  ```rust
  let mut count = 0;
  count += 1; // 허용됨
  ```
- **섀도잉(Shadowing)**:
  기존 변수와 동일한 이름으로 새 변수를 `let`으로 선언하는 것을 의미합니다.
  - 기존 변수의 타입이나 불변성 여부를 변경하면서 동일한 이름을 유지하고 싶을 때 유용합니다.
  ```rust
  let input = " 42 \n";
  let input: u32 = input.trim().parse().expect("숫자가 아닙니다");
  // 문자열 input이 숫자 input으로 재정의(섀도잉)됨
  ```

### 1.2 문자열의 두 얼굴: `String`과 `&str`
Rust는 메모리 안전성을 위해 문자열을 명확히 구분합니다.
1. **`&str` (문자열 슬라이스 / 참조)**:
   - 메모리의 특정 지점에 존재하는 UTF-8 바이트 시퀀스를 "가리키는 참조"입니다.
   - 코드에 하드코딩된 리터럴(`"Hello"`)은 컴파일된 실행 파일의 바이너리 영역에 저장되는 `&'static str`입니다.
   - 크기를 동적으로 늘릴 수 없습니다.
2. **`String` (힙 할당 소유 문자열)**:
   - 힙(Heap) 메모리에 저장되며, 크기를 자유롭게 늘리거나 줄일 수 있는 문자열 버퍼입니다.
   - 사용자 입력을 받거나(`stdin().read_line`), 포맷팅된 문자열을 동적으로 생성할 때 사용합니다.
- **변환**:
  - `&str` ➔ `String`: `s.to_string()`, `String::from(s)`
  - `String` ➔ `&str`: `&s` (Deref 역참조 강제를 통해 자동 슬라이스화)

### 1.3 표준 입출력과 버퍼 플러시 (`stdin`, `stdout`)
- 터미널에 프롬프트를 띄우고 입력을 받을 때의 정석 패턴:
  ```rust
  use std::io::{self, Write};

  print!("선택 > ");
  // print! 매크로는 개행(\n)이 없으므로 버퍼에 머물러 있을 수 있음.
  // 즉시 화면에 출력되도록 flush()를 호출해야 함.
  io::stdout().flush().expect("출력 버퍼 플러시 실패");

  let mut input = String::new();
  // read_line은 사용자가 누른 Enter(\n 또는 \r\n)까지 버퍼에 담음.
  io::stdin().read_line(&mut input).expect("입력 읽기 실패");

  let trimmed = input.trim(); // 앞뒤 공백 및 줄바꿈 문자 제거 (&str 반환)
  ```

### 1.4 에러 처리 기초 맛보기 (`Result`, `.expect()`)
- Rust에는 C++/Java 스타일의 `try-catch`나 `null`이 없습니다.
- 실패할 가능성이 있는 작업(I/O, 파싱 등)은 반드시 `Result<T, E>` 타입을 반환합니다.
  - `Ok(T)`: 성공과 그 결과값
  - `Err(E)`: 실패와 그 원인
- `.expect("메시지")`:
  - `Ok`면 안의 값을 꺼내고, `Err`면 제공한 메시지와 함께 프로그램을 즉시 중단(Panic)시킵니다.
  - 초기 프로토타이핑이나 "절대 실패해서는 안 되는 상황"에 주로 사용합니다.

### 1.5 반복문과 제어 (`loop`, `break`)
- `loop`: 조건 없이 무한 반복하는 루프입니다. 컴파일러가 무한 루프임을 인지하므로 `break`로 값을 반환할 수도 있습니다.
- `break`: 루프를 즉시 탈출합니다.
- `continue`: 다음 반복으로 건너뜁니다.
- 루프 라벨: 중첩 루프에서 외부 루프를 탈출할 때 `'label:` 구문을 사용합니다.

---

## 2단계: 데이터 모델링 - 구조체(Struct)와 메서드

### 2.1 구조체(Struct)의 정의와 인스턴스화
관련된 데이터 필드들을 하나의 사용자 정의 타입으로 묶습니다.
```rust
struct Player {
    name: String,
    level: u32,
    hp: u32,
    max_hp: u32,
    atk: u32,
    def: u32,
    gold: u32,
}
```
- **필드 초기화 단축(Field Init Shorthand)**:
  변수명과 구조체 필드명이 같으면 `name: name` 대신 `name`만 쓸 수 있습니다.

### 2.2 `impl` 블록: 연관 함수(생성자)와 메서드
구조체에 동작(함수)을 부여하려면 `impl` 블록을 작성합니다.
1. **연관 함수 (Associated Function)**:
   - 첫 번째 매개변수로 `self`를 받지 않는 함수.
   - 인스턴스 없이 호출하며, 주로 생성자 역할을 합니다: `Player::new(...)`
2. **메서드 (Method)**:
   - 첫 번째 매개변수로 `self`를 받는 함수.
   - 인스턴스에서 점 표기법(`player.rest()`)으로 호출합니다.

### 2.3 `self`, `&self`, `&mut self`의 선택 기준
메서드의 첫 번째 인자는 소유권과 변경 권한을 결정합니다.

| 시그니처 | 명칭 | 권한 | 용도 | 예시 |
| :--- | :--- | :--- | :--- | :--- |
| `&self` | 불변 참조 | 읽기 전용 | 상태 조회, 출력, 계산 | `fn show_status(&self)` |
| `&mut self` | 가변 참조 | 읽기/쓰기 | 인스턴스 내부 필드 수정 | `fn heal(&mut self, amount: u32)` |
| `self` | 소유권 소비 | 완전 소비 | 다른 타입으로 변환하거나 파괴 | `fn into_monster(self) -> Monster` |

### 2.4 파생 속성(Derive Attributes)과 디버그 출력
기본적인 트레이트를 수동 작성 없이 컴파일러가 자동 구현하도록 지시할 수 있습니다.
```rust
#[derive(Debug)]
struct Player { ... }
```
- `println!("{:?}", player)`: 간결한 1줄 디버그 출력
- `println!("{:#?}", player)`: 필드별로 들여쓰기된 보기 편한 다중 줄 디버그 출력

---

## 3단계: 타입 기반 상태 제어 - 열거형(Enum) & 패턴 매칭

### 3.1 대수적 데이터 타입: 열거형(`enum`)
Rust의 `enum`은 여러 가능한 상태 중 하나만을 가질 수 있는 강력한 타입입니다.
```rust
#[derive(Debug, PartialEq, Eq)]
enum GameState {
    Town,
    Dungeon,
    GameOver,
    Victory,
    Quit,
}
```

### 3.2 패턴 매칭 `match`와 완전성 검사(Exhaustive Checking)
- **완전성 검사**: `match` 문은 열거형의 **모든 가능한 경우의 수(variants)**를 반드시 처리해야만 컴파일됩니다. 새 상태를 추가했을 때 누락된 곳을 컴파일러가 정확히 짚어줍니다.
- **와일드카드 `_`**: 처리하지 않은 나머지 모든 경우를 한 번에 처리합니다.
- **매치 가드(Match Guard)**: 패턴 뒤에 `if` 조건을 추가하여 세부 분기를 처리합니다.
```rust
match choice {
    1 => println!("던전으로 출발!"),
    2 => println!("상점 방문"),
    n if n > 10 => println!("너무 큰 번호입니다."),
    _ => println!("잘못된 번호입니다."),
}
```

### 3.3 Null 없는 안전한 세상: `Option<T>`
Rust에는 다른 언어의 `null`이나 `nil`이 없습니다. 값이 있거나 없는 상황은 표준 라이브러리의 `Option<T>` 열거형으로 모델링합니다.
```rust
enum Option<T> {
    Some(T), // 값이 존재하는 경우
    None,    // 값이 없는 경우
}
```
- 사용자가 선택적으로 장비를 착용했는지(`Option<Weapon>`), 리스트에서 요소를 찾지 못했는지 등을 안전하게 다룰 수 있습니다.
- 컴파일러가 `None` 체크를 강제하므로 **NullPointerException이 원천적으로 불가능**합니다.

### 3.4 문자열 파싱과 에러 핸들링 기초 (`parse`, `match Result`)
사용자가 콘솔에 숫자를 입력하지 않고 글자를 쳤을 때 프로그램이 죽지 않도록 방어하는 정석 패턴:
```rust
let input = "abc";
match input.trim().parse::<u32>() {
    Ok(number) => println!("입력한 숫자: {}", number),
    Err(_) => println!("올바른 숫자를 입력해주세요!"),
}
```

---

## 4단계: Rust의 심장 - 소유권(Ownership) & 대여(Borrowing)

### 4.1 소유권(Ownership)의 3대 원칙
Rust가 가비지 컬렉터(GC) 없이도 메모리 안전성과 고성능을 보장하는 핵심 원리입니다.
1. Rust의 각각의 값은 **소유자(owner)**라고 불리는 변수가 있다.
2. 한 번에 딱 **하나의 소유자**만 존재할 수 있다.
3. 소유자가 스코프(Scope, `{}`)를 벗어나면 그 값은 메모리에서 즉시 해제(`drop`)된다.

### 4.2 이동(Move)과 복사(Copy)
- **이동(Move)**:
  힙에 할당된 타입(`String`, `Vec` 등)을 다른 변수에 대입하거나 함수 인자로 넘기면 소유권이 이전(Move)됩니다. 이전 변수는 더 이상 사용할 수 없습니다.
  ```rust
  let s1 = String::from("용사");
  let s2 = s1; // 소유권이 s2로 이동
  // println!("{}", s1); // ❌ 컴파일 에러: value borrowed here after move
  ```
- **복사(Copy)**:
  스택에만 저장되는 고정 크기 스칼라 타입(`u32`, `i32`, `bool`, `char` 등)은 `Copy` 트레이트를 구현하고 있어 대입 시 비트 단위 복사가 일어납니다. 원본 변수도 계속 사용할 수 있습니다.

### 4.3 대여(Borrowing)와 참조 규칙 (Borrow Checker)
값을 매번 소유권 이동시키면 비효율적이므로, 소유권을 넘기지 않고 "빌려 쓰는" **참조자(Reference)**를 사용합니다.
- **불변 참조 (`&T`)**: 값을 읽을 수만 있습니다.
- **가변 참조 (`&mut T`)**: 값을 읽고 수정할 수 있습니다.

> ⚠️ **대여 검사기(Borrow Checker)의 불변 규칙 (매우 중요)**
> 
> 어떤 데이터에 대해 특정 시점에 다음 중 **하나만** 만족할 수 있습니다:
> 1. **여러 개의 불변 참조자(`&T`)**가 존재하거나,
> 2. **단 하나의 가변 참조자(`&mut T`)**만 존재할 수 있다.
> 
> (즉, 쓰기 작업이 일어나는 동안에는 읽기조차 허용하지 않아 데이터 레이스(Data Race)를 원천 차단합니다.)

### 4.4 수치 연산과 언더플로우 방지 (`saturating_sub`)
- Rust에서 부호 없는 정수(`u32`)를 연산할 때 `0` 이하로 내려가면 디버그 모드에서 **Panic(패닉)**이 발생합니다.
- 전투에서 `HP - 데미지`나 `방어력 차감`을 계산할 때는 바운드 연산 메서드를 사용해야 안전합니다:
  - `hp.saturating_sub(damage)`: 결과가 0 미만이 되면 음수가 되는 대신 0을 반환.
  - `max(1, atk.saturating_sub(def))`: 최소 1의 피해를 보장하는 공식.

### 4.5 단위 테스트(Unit Test) 작성 기초
함수가 예상대로 동작하는지 검증하는 테스트 코드는 Rust에 기본 내장되어 있습니다.
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_damage_calculation() {
        let mut player = Player::new("테스터");
        player.take_damage(30);
        assert_eq!(player.hp, 70);
    }
}
```
- 실행 명령어: `cargo test`

---

## 5단계: 컬렉션 & 데이터 복합화 - Vec & 소유권 이동

### 5.1 동적 배열 `Vec<T>`의 동작 원리와 메모리
- 힙 메모리에 연속된 공간을 할당받아 임의 개수의 동일 타입 원소를 보관합니다.
- 내부적으로 **포인터(Pointer)**, **용량(Capacity)**, **길이(Length)**의 3개 머신 워드로 구성됩니다.
- 주요 연산:
  - `vec.push(item)`: 뒤에 원소 추가 (소유권 이동)
  - `vec.pop()`: 마지막 원소 제거 후 반환 (`Option<T>`)
  - `vec.remove(index)`: 해당 인덱스 원소를 꺼내고 뒤 요소들을 앞으로 당김 (`O(N)`)

### 5.2 안전한 인덱싱 (`v[i]` vs `v.get(i)`)
- `v[i]`: 인덱스가 범위를 벗어나면 즉시 프로그램 패닉 발생.
- `v.get(i)`: `Option<&T>`를 반환하여 안전하게 범위 초과 여부를 확인 가능.
  ```rust
  if let Some(item) = inventory.get(choice - 1) {
      println!("선택한 아이템: {:?}", item);
  } else {
      println!("유효하지 않은 번호입니다.");
  }
  ```

### 5.3 데이터를 품은 열거형 (Enum with Data)
Rust 열거형은 각 변형(variant)마다 서로 다른 구조의 데이터를 담을 수 있습니다.
```rust
#[derive(Debug, Clone)]
enum Item {
    Potion { name: String, heal_amount: u32 },
    Weapon { name: String, bonus_atk: u32 },
    Armor  { name: String, bonus_def: u32 },
}
```

### 5.4 소유권 교체 기법 (`Option::take`, `Option::replace`)
장비를 장착할 때: "기존에 끼고 있던 장비를 빼서 인벤토리에 넣고, 새 장비를 슬롯에 넣는" 작업은 가변 참조 상태에서 소유권을 안전하게 맞바꿔야 합니다.
```rust
// player.weapon은 Option<Weapon>
let old_weapon = player.weapon.replace(new_weapon);
// replace는 새 값을 넣고, 기존에 있던 값을 꺼내어 반환함 (Some 또는 None)
if let Some(old) = old_weapon {
    player.inventory.push(old); // 기존 장비는 인벤토리로 복귀
}
```

### 5.5 반복자(Iterator)와 클로저(Closure) 기초
컬렉션의 원소들을 순회하거나 필터링할 때 함수형 스타일을 활용합니다.
```rust
// 인벤토리 목록을 번호와 함께 출력
for (idx, item) in inventory.iter().enumerate() {
    println!("[{}] {:?}", idx + 1, item);
}

// 포션의 개수만 세기
let potion_count = inventory.iter().filter(|item| matches!(item, Item::Potion { .. })).count();
```

---

## 6단계: 추상화와 다형성 - 트레이트(Trait)

### 6.1 트레이트(Trait)의 정의와 구현
다른 언어의 `interface`와 유사하게, 특정 타입들이 공통으로 가져야 할 기능(메서드 규약)을 정의합니다.
```rust
trait Combatant {
    fn name(&self) -> &str;
    fn hp(&self) -> u32;
    fn atk(&self) -> u32;
    fn def(&self) -> u32;
    fn take_damage(&mut self, damage: u32);
    fn is_alive(&self) -> bool;
}

impl Combatant for Player { ... }
impl Combatant for Monster { ... }
```

### 6.2 기본 구현(Default Implementation)
트레이트 내부에서 메서드의 기본 동작을 미리 구현해 둘 수 있습니다. 구현체는 필요할 때만 이를 오버라이딩합니다.
```rust
trait Combatant {
    fn hp(&self) -> u32;
    
    // 기본 구현 제공
    fn is_alive(&self) -> bool {
        self.hp() > 0
    }
}
```

### 6.3 정적 디스패치(제네릭) vs 동적 디스패치(`dyn Trait`)
- **정적 디스패치 (Generics & Trait Bounds)**:
  - `fn attack<T: Combatant>(attacker: &T, target: &mut T)`
  - 컴파일 타임에 타입별로 전용 기계어 코드가 생성됨 (**Monomorphization**).
  - 런타임 오버헤드가 전혀 없는 "제로 코스트 추상화(Zero-Cost Abstraction)".
- **동적 디스패치 (`dyn Trait`)**:
  - `fn battle(a: &mut dyn Combatant, b: &mut dyn Combatant)` 또는 `Box<dyn Combatant>`
  - 컴파일 시점에는 실제 타입을 모르고, 런타임에 vtable(가상 메서드 테이블)을 조회하여 호출.
  - 서로 다른 타입을 하나의 배열에 담거나 런타임에 갈아끼워야 할 때 사용.

### 6.4 표준 라이브러리 트레이트 (`Display`, `Clone`, `Default`)
- `std::fmt::Display`: `{}`로 출력할 사용자 친화적인 문자열 서식을 정의.
- `Clone`: 깊은 복사(Deep copy)를 명시적으로 수행하는 `.clone()` 메서드 제공.
- `Default`: 타입의 기본 초기값을 생성하는 `Type::default()` 제공.

---

## 7단계: 대규모 모듈 구성 - 모듈 시스템(Module System)

### 7.1 Rust 모듈 트리 (`crate`, `mod`, `use`, `super`)
- `crate`: 현재 프로젝트 전체 패키지의 루트(Root).
- `mod foo;`: `foo.rs` 또는 `foo/mod.rs` 파일을 현재 모듈 트리에 자식 모듈로 등록.
- `use crate::player::Player;`: 긴 경로 대신 짧은 이름으로 심볼을 가져오기.
- `super`: 부모 모듈을 가리킴 (주로 `tests` 서브모듈에서 외부 함수 접근 시 사용).

### 7.2 파일/디렉터리 분리 규칙 (Modern Rust 방식)
Rust 2018 이상에서는 더 직관적인 1:1 파일 매핑을 권장합니다.
```
src/
 ├─ main.rs        # mod player; mod monster; mod combat; 선언
 ├─ player.rs      # struct Player 및 impl
 ├─ monster.rs     # struct Monster 및 impl
 ├─ combat.rs      # trait Combatant 및 전투 로직
 └─ io_helper.rs   # 입출력 보조 함수
```

### 7.3 가시성(Visibility)과 캡슐화 (`pub`, `pub(crate)`)
Rust의 모든 항목(구조체, 필드, 함수, 모듈)은 **기본적으로 비공개(private)**입니다.
- 외부 모듈에서 접근하려면 반드시 앞에 `pub`을 명시해야 합니다.
- **구조체 필드의 가시성**: 구조체 자체에 `pub struct Player`를 붙여도, 내부 필드에 `pub`을 붙이지 않으면 외부 모듈에서 직접 접근할 수 없습니다 (캡슐화 보장).
  ```rust
  pub struct Player {
      pub name: String,
      hp: u32, // 외부 모듈에서는 수정 불가, 메서드를 통해서만 제어
  }
  ```

### 7.4 간결한 제어 흐름: `let ... else` 구문
패턴 매칭에 실패했을 때 즉시 탈출(early return/break)하는 최신 관용구입니다.
```rust
let Some(monster) = spawn_monster(floor) else {
    println!("이 층에는 몬스터가 없습니다.");
    return;
};
// 여기서부터 monster를 바로 사용 가능!
```

---

## 8단계: 견고한 영속화 - 에러 전파 & 파일 I/O & 직렬화

### 8.1 에러 전파 연산자 `?`와 `From` 트레이트
함수 내부에서 에러가 발생했을 때 호출자에게 즉시 에러를 반환(Early return)하는 연산자입니다.
```rust
fn load_data() -> Result<String, io::Error> {
    let mut file = std::fs::File::open("save.txt")?; // 실패 시 Err 반환
    let mut content = String::new();
    file.read_to_string(&mut content)?;             // 실패 시 Err 반환
    Ok(content)
}
```
- `?` 연산자는 발생한 에러를 함수의 반환 에러 타입으로 자동 변환(`From::from`)해 줍니다.

### 8.2 커스텀 에러 열거형 설계
여러 종류의 에러(파일 I/O 에러, JSON 파싱 에러, 게임 데이터 불일치 에러 등)를 단일 에러 타입으로 통합합니다.
```rust
#[derive(Debug)]
pub enum SaveError {
    Io(std::io::Error),
    Json(serde_json::Error),
    CorruptedData(String),
}

impl From<std::io::Error> for SaveError {
    fn from(err: std::io::Error) -> Self {
        SaveError::Io(err)
    }
}
```

### 8.3 파일 읽기/쓰기 (`std::fs`)
작은 파일의 단순 저장 및 불러오기는 표준 라이브러리의 원샷 함수가 가장 깔끔합니다.
- 쓰기: `std::fs::write("savegame.json", json_string)`
- 읽기: `std::fs::read_to_string("savegame.json")`

### 8.4 `serde`와 `serde_json`을 통한 직렬화/역직렬화
- **직렬화 (Serialization)**: 메모리 상의 구조체를 문자열(JSON)이나 바이트로 변환 (`to_string`).
- **역직렬화 (Deserialization)**: 문자열(JSON)을 읽어 메모리 상의 구조체로 복원 (`from_str`).
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
pub struct SaveData {
    pub player: Player,
    pub current_floor: u32,
}
```

---

## 부록: Rust 개발 도구 & 유용한 명령어

| 명령어 | 설명 | 권장 사용 타이밍 |
| :--- | :--- | :--- |
| `cargo check` | 바이너리를 빌드하지 않고 문법 및 타입 검사만 수행 (매우 빠름) | 코드 작성 중 수시로 |
| `cargo run` | 프로젝트를 컴파일하고 즉시 실행 | 기능 구현 후 동작 확인할 때 |
| `cargo test` | 프로젝트 내 모든 `#[test]` 단위 테스트 실행 | 로직 변경 및 리팩토링 직후 |
| `cargo fmt` | 공식 코드 스타일 가이드에 맞춰 자동 코드 포맷팅 | 커밋 전 필수 |
| `cargo clippy` | Rust 공식 린터로, 더 나은 관용적 코드 패턴 제안 및 잠재적 버그 감지 | 단계 완료 시 필수 |

---

### 💡 학습 팁
1. **컴파일러 에러 메시지를 꼼꼼히 읽으세요**: Rust 컴파일러는 현존하는 언어 중 가장 친절한 에러 메시지와 함께 해결 힌트(`help: ...`)를 제공합니다.
2. **소유권 에러가 날 때는 복사하기 전에 소유권 흐름을 먼저 추적하세요**: 무작정 `.clone()`으로 해결하기보다 참조자(`&`)로 해결할 수 없는지 고민하는 과정에서 실력이 크게 향상됩니다.
