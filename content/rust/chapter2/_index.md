+++
title = "Programming a Guessing Game"
weight = 2
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

- 프로젝트 생성 : `cargo new 프로젝트이름`

  <br>

- 표준 라이브러리를 가져오는 명령어 :

  - `use std::라이브러리이름`
  - `use std::메소드이름::타입이름`

- 크레이트를 가져오는 명령어 : `use 크레이트이름::트레잇이름`

  - 크레이트 : Rust 소스 코드 파일의 모음

  - 트레잇 : 크레이트에 포함된 메서드(하나 또는 여러개)

  - 레지스트리 : [Crates.io](https://crates.io/) 의 데이터 사본 

  - Crates.io : 사람들이 오픈 소스 Rust 프로젝트를 게시하는 곳

  <br>

- 변수생성 :

  - `let 변수이름` : 변경불가능 

  - `let mut 변수이름` : 변경가능

  - `let 변수이름 : 타입`, `let mut 변수이름 : 타입` : 타입명시

  - `let mut 변수이름 = String::new()` : 빈 문자열 변수 생성

- 변수이름은 재사용(덮어쓰기, shadowing) 가능

  <br>

- `match, cmp`, `loop, break` 를 사용할 수 있다.

  <br>

- 일부 메서드는 "결과값"과 "Result 타입"을 함께 반환

  - Result 타입 : 여러가지 변형(variant, 종류라고 이해하면 될듯)을 가지는 열거형(enumeration)

  - Result 타입의 variant : Ok(T) 와 Err(E) 

<br>

- 오류해결
  - Result 를 반환하는 메소드 뒤에 `.expect`를 붙인다
  - `메소드().expect("메세지")` : 메소드에서 반환하는 Result 타입의 variant가 Err 이면, 메세지 출력후 프로그램 중단



<!-- more -->

# [추리게임 프로그래밍](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#programming-a-guessing-game)



함께 실습 프로젝트를 통해 Rust에 뛰어들어 봅시다! 이 장에서는 실제 프로그램에서 사용하는 방법을 보여줌으로써 몇 가지 일반적인 Rust 개념을 소개합니다. `let`, `match`, 메서드, 관련 함수, 외부 크레이트 등에 대해 배우게 됩니다! 다음 장에서는 이러한 아이디어에 대해 자세히 살펴보겠습니다. 이 장에서는 기본 사항만 연습합니다.

고전적인 초보자 프로그래밍 문제인 추측 게임을 구현할 것입니다. 작동 방식은 다음과 같습니다. 프로그램은 1에서 100 사이의 임의의 정수를 생성합니다. 그런 다음 플레이어에게 추측을 입력하라는 메시지가 표시됩니다. 추측이 입력되면 프로그램은 추측이 너무 낮은지 또는 너무 높은지 표시합니다. 추측이 맞으면 게임이 축하 메시지를 출력하고 종료됩니다.

## [새 프로젝트 설정](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#setting-up-a-new-project)

새 프로젝트를 설정하려면 1장에서 만든 *프로젝트* 디렉토리로 이동하고 다음과 같이 Cargo를 사용하여 새 프로젝트를 만듭니다.

```
$ cargo new guessing_game
$ cd guessing_game
```

첫 번째 명령어인 `cargo new`는 프로젝트 이름( `guessing_game`)을 첫 번째 인수로 사용합니다. 두 번째 명령은 새 프로젝트의 디렉터리로 변경됩니다.

<br>

생성된 *Cargo.toml* 파일을 살펴보십시오.  
파일 이름: Cargo.toml

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

<br>

1장에서 본 것처럼 `cargo new` 는 "Hello, world!" 프로그램을 생성합니다. *src/main.rs* 파일을 확인해봅시다.   
파일 이름: src/main.rs

```rust
fn main() {
    println!("Hello, world!");
}
```

<br>

이제 "Hello, world!"  프로그램을 `cargo run` 명령을 사용하여 컴파일하고 실행하십시오 .

```
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50s
     Running `target/debug/guessing_game`
Hello, world!
```

이 `run` 명령은 프로젝트를 빠르게 반복해야 할 때 유용합니다. 이 게임에서 우리는 다음 반복으로 이동하기 전에 각 반복을 빠르게 테스트합니다.

*src/main.rs* 파일을 다시 엽니다. 이제부터 이 파일에 모든 코드를 작성하게 됩니다.



## [추측 처리](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#processing-a-guess)

추측 게임 프로그램의 첫 번째 부분은 사용자 입력을 요청하고, 해당 입력을 처리하고, 입력된 값이 예상된 형식인지 확인하는 것입니다. 시작하려면 플레이어가 추측을 입력하도록 허용합니다. 목록 2-1의 코드를 *src/main.rs* 에 입력합니다.

파일 이름: src/main.rs

```rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

목록 2-1: 사용자로부터 추측을 받아 출력하는 코드

이 코드에는 많은 정보가 포함되어 있으므로 한 줄씩 살펴보겠습니다. 사용자 입력을 얻은 다음 결과를 출력으로 인쇄하려면  `io` 라고 하는 입력/출력 라이브러리를 범위(scope)로 가져와야 합니다. `io`라이브러리는 `std`로 알려진 표준 라이브러리에서 가져옵니다.

```rust
use std::io;
```

기본적으로 Rust는 모든 프로그램의 범위로 가져오는 표준 라이브러리에 정의된 일련의 항목을 가지고 있습니다. 이 세트를 *전주곡*(*prelude*)이라고 하며 [표준 라이브러리 문서](https://doc.rust-lang.org/std/prelude/index.html)에서 그 안에 있는 모든 것을 볼 수 있습니다.

사용하려는 유형이 전주곡(prelude)에 없으면 `use`명령문을 사용하여 해당 유형을 명시적으로 범위로 가져와야 합니다. `std::io`라이브러리를 사용하면, 사용자 입력을 수락하는 기능을 포함하여 여러 가지 유용한 기능이 제공됩니다.

<br>

1장에서 본 것처럼 `main`함수는 프로그램의 진입점입니다.

```rust
fn main() {
```

구문 `fn`은 새 함수를 선언합니다. `()`괄호는 매개변수가 없음을 나타냅니다. 중괄호(`{`) 는 함수 본문을 시작합니다.

<br>

1장에서도 배웠듯이 `println!`는 화면에 문자열을 출력하는 매크로입니다.

```rust
    println!("Guess the number!");

    println!("Please input your guess.");
```

<br>

이 코드는 게임이 무엇인지 설명하고 사용자의 입력을 요청하는 프롬프트를 인쇄합니다.



### [변수와 함께 값 저장](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#storing-values-with-variables)

다음으로 다음과 같이 사용자 입력을 저장할 *변수*를 만듭니다.

```rust
let mut guess = String::new();
```

이제 프로그램이 흥미로워지고 있습니다! 이 작은 줄에서 많은 일이 벌어지고 있습니다. `let`문을 사용하여 변수를 만듭니다.  

<br>

다음은 또 다른 예입니다.

```rust
let apples = 5;
```

이 줄은 `apples`이름의 새 변수를 만들고, 변수에 값 5를 바인딩합니다. Rust에서 변수는 기본적으로 변경할 수 없습니다. 즉, 변수에 값을 지정하면 값이 변경되지 않습니다. [3장의 "변수 및 가변성"](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#variables-and-mutability) 섹션에서 이 개념에 대해 자세히 논의할 것입니다. 변수를 변경 가능하게 만들기 위해 변수 이름 앞에 `mut`을 추가합니다.

```rust
let apples = 5; // immutable
let mut bananas = 5; // mutable
```


> 참고: `//`구문은 줄 끝까지 계속되는 주석을 시작합니다. Rust는 주석의 모든 것을 무시합니다. 주석에 대해서는 [3장](https://doc.rust-lang.org/book/ch03-04-comments.html) 에서 더 자세히 논의할 것입니다.

추측 게임 프로그램으로 돌아가서, 이제 `let mut guess` 는 이름이 변경 가능한 변수 `guess`를 소개한다는 것을 알고 있습니다. 등호( `=`)는 Rust에게 우리가 지금 무언가를 변수에 묶고 싶다는 것을 알려줍니다. 등호 오른쪽에는 `guess` 에 바인딩된 값이 있는데, 그 값은 `String`의 새 인스턴스를 반환하는 함수인 `String::new`를 호출한 결과값 입니다. [`String`](https://doc.rust-lang.org/std/string/struct.String.html)은 표준 라이브러리에서 제공하는 문자열 유형인데, 확장 가능하고 UTF-8 로 인코딩된 비트(bit)입니다. 

<br>

`String::new()` 행의 `::`구문은 `new` 가 `String` 타입의 *연관함수*임을 나타냅니다. *연관함수*는 어떤 하나의 타입을 위한 함수이며, 이 경우에는 `String` 타입입니다. `new` 함수는 비어있는 새 문자열을 만듭니다. 어떤 종류의 새로운 값을 만드는 함수의 일반적인 이름이기 때문에, 많은 타입에서 `new` 함수를 찾을 수 있습니다.

<br>

정리하면, `let mut guess = String::new();` 은 변경가능한 `guess` 변수를 만드는데, 이 변수는 현재 비어 있고 새로운 `String` 인스턴스에 바인딩되어 있습니다. 


### [사용자 입력 받기](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#receiving-user-input)

프로그램의 첫 번째 줄에서 `use std::io;` 를 이용해서 표준 라이브러리의 입/출력 기능을 포함했음을 상기하십시오. 이제 `io` 모듈 에서 `stdin` 함수를 호출하여 사용자 입력을 처리할 수 있습니다.

```rust
    io::stdin()
        .read_line(&mut guess)
```

프로그램 시작 부분에서 `use std::io;` 를 이용해서 `io` 라이브러리를 가져오지 않았다면, 이 함수를 호출할때  `std::io::stdin` 이라고 적어야 합니다. `stdin` 함수는 [`std::io::Stdin`](https://doc.rust-lang.org/std/io/struct.Stdin.html) 인스턴스를 반환하는데, 이것은 터미널에서 표준 입력에 대한 핸들을 나타내는 타입입니다.  

<br>

다음줄에 있는 `.read_line(&mut guess)` 는 표준 입력 핸들의 [`read_line`](https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line) 메서드를 호출하여 사용자로부터 입력을 받습니다. 또한 사용자 입력을 저장할 문자열을 알려주기 위해 `&mut guess` 를 `read_line` 함수의 인수(arg)로 전달하고 있습니다. `read_line` 의 전체 작업은 사용자가 표준 입력으로 입력하는 모든 것을 가져와 문자열(string)에 추가하는 것입니다(내용을 덮어쓰지 않고). 따라서 문자열(string)을 인수로 전달하십시오. 메서드가 문자열의 내용을 변경할 수 있도록 문자열 인수는 변경 가능해야 합니다.  

`&` 는 이 인수가 *참조자*(*reference*)임을 나타냅니다. 즉, 해당 데이터를 메모리에 여러 번 복사할 필요 없이, 코드의 여러 부분에서 한 데이터에 액세스할 수 있는 방법을 제공합니다. 참조는 복잡한 기능이며, Rust의 주요 장점 중 하나는 참조를 사용하는 것이 얼마나 안전하고 쉬운가입니다. 이 프로그램을 마치기 위해 이러한 많은 세부 사항을 알 필요는 없습니다. 지금은 변수와 마찬가지로 참조도 기본적으로 변경할 수 없다(immutable)는 점만 알면 됩니다. 따라서 변경 가능하게 만들기 `&guess` 가 아니라 `&mut guess` 로 적어야 합니다. (4장에서 참고 문헌에 대해 더 자세히 설명합니다.)



### [Result 타입으로 잠재적인 실패 처리](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result)

우리는 여전히 이 코드 라인을 작업하고 있습니다. 이제 텍스트의 세 번째 줄에 대해 논의하고 있지만 여전히 하나의 논리적 코드 줄의 일부라는 점에 유의하십시오. 다음 부분은 이 메소드입니다.

```rust
        .expect("Failed to read line");
```

이 코드를 다음과 같이 작성할 수도 있습니다.

```rust
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

하지만, 하나의 긴 줄은 읽기 어려우므로 나누어서 쓰는 것이 좋습니다. `.method_name()` 구문을 사용하여 메서드를 호출할 때, 긴 줄을 구분하는 데 도움이 되도록 줄바꿈 및 기타 공백을 사용하는 것이 좋은 경우가 많습니다. 이제 이 줄이 무엇을 하는지 논의해 봅시다.

앞에서 언급했듯이 `read_line` 은 사용자가 입력하는 문자열은 무엇이든 전달합니다. 동시에  `Result` 값도 반환합니다. [`Result`](https://doc.rust-lang.org/std/result/enum.Result.html)  는 여러 상태(state) 중 하나가 될수 있는 타입인 [*enumeration*](https://doc.rust-lang.org/book/ch06-00-enums.html) 입니다. *enum* 이라고도 합니다. 가능한 각 상태를 *변형*(*variant*) 이라고 합니다.

<br>

[6장에서](https://doc.rust-lang.org/book/ch06-00-enums.html) 열거형에 대해 자세히 다룰 것입니다. 이러한 `Result` 타입의 목적은 오류 처리 정보를 인코딩하는 것입니다.

`Result`의 variant는 `Ok` 와 `Err` 입니다. `Ok` variant는 작업이 성공했음을 나타내며, `Ok` 내부는 성공적으로 생성된 결과값입니다.  `Err` variant는 작업이 실패했음을 의미하며,  `Err` 내부는 작업이 어떻게 또는 왜 실패했는지에 대한 정보를 포함합니다.

다른 타입의 값과 마찬가지로, `Result` 타입의 값에는 메소드가 정의되어 있습니다. `Result`의 인스턴스에는 호출가능한  [`expect`메서드](https://doc.rust-lang.org/std/result/enum.Result.html#method.expect) 가 있습니다.

 `Result`의 인스턴스 값이 `Err` 이면, `expect` 는 프로그램을 중단하고 `expect` 에 인수로 전달한 메시지가 표시됩니다. `read_line`메서드가 `Err`를 반환하는 경우 기본 운영 체제에서 발생한 오류의 결과일 수 있습니다.

 `Result` 의 인스턴스 값이 `Ok` 이면, `expect` 는 `Ok` 가 보유하고 있는 반환값을 가져 와서 사용할 수 있도록 해당 값만 반환합니다. 이 경우 반환값은 사용자가 입력했던 바이트의 갯수입니다.

<br>

만약 `expect`를 호출하지 않으면 프로그램이 컴파일되지만 경고가 표시됩니다.

```
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `Result` that must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: this `Result` may be an `Err` variant, which should be handled
   = note: `#[warn(unused_must_use)]` on by default

warning: `guessing_game` (bin "guessing_game") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Rust는 `read_line`에서 반환된 `Result` 값을 사용하지 않았다고 경고합니다. 이 말은 프로그램이 발생 가능한 오류를 처리하지 않았다는 의미입니다.

경고를 억제하는 올바른 방법은 실제로 오류 처리 코드를 작성하는 것이지만, 우리는 문제가 발생했을 때 이 프로그램을 중단하기를 원하므로 `expect` 를 사용했습니다. 

[9장](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html) 에서 오류 복구에 대해 배웁니다.

### [println! 변경자(placeholer)를 이용한 값 출력](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#printing-values-with-println-placeholders)

이제까지 작성한 코드에서 닫는 중괄호 말고도 살펴봐야 하는 코드가 하나 더 있습니다. 내용은 아래와 같습니다.

```rust
    println!("You guessed: {guess}");
```

이 줄은 현재 사용자가 입력한 값을 저장한 문자열을 인쇄합니다. 중괄호 세트는 변경자입니다.  `{}` 를 값을 제자리에 고정하는 작은 게 집게발이라고생각하세요. 

변수 값을 출력할 때 변수 이름은 중괄호 안에 들어갈 수 있습니다. 

표현식 결과값을 출력할 때는 형식 문자열(format string)에 빈 중괄호를 넣습니다. 그 다음 각 빈 중괄호 변경자와 동일한 순서로, 표현식을 쉼표로 구분된 리스트 형식으로 입력합니다. 그러면 빈 중괄호 안에 순서대로 출력됩니다. 아래 코드는 `println!` 를 한 번 호출해서 변수와 식의 결과를 인쇄하는 방법을 보여줍니다. 

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

이 코드는 `x = 5 and y + 2 = 12` 를 출력합니다.

### [첫 번째 부분을 테스트하기](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#testing-the-first-part)

추측 게임의 첫 번째 부분을 테스트해 봅시다. `cargo run`을 사용하여 실행하십시오 .

```
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

이제 게임의 첫 번째 부분이 완료되었습니다. 키보드에서 입력을 받은 다음, 그 값을 출력합니다.

## [비밀 번호 생성](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-secret-number)

다음으로 사용자가 추측할 비밀 번호를 생성해야 합니다. 시크릿 넘버는 매번 달라야 게임을 여러번 해도 재미있습니다. 게임이 너무 어렵지 않도록 1에서 100 사이의 임의의 숫자를 사용합니다. Rust는 아직 표준 라이브러리에 난수 기능을 포함하지 않습니다. 그러나 Rust 팀은 해당 기능이 포함된 [rand 크레이트](https://crates.io/crates/rand) 를 제공합니다.

### [크레이트를 사용하여 더 많은 기능 얻기](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#using-a-crate-to-get-more-functionality)

크레이트는 Rust 소스 코드 파일의 모음입니다. 우리가 구축한 프로젝트는 실행 파일인 *바이너리 크레이트* 입니다. `rand` 크레이트는 *라이브러리 크레이트*로 , 다른 프로그램에서 사용하기 위한 코드가 들어 있으며 자체적으로는 실행할 수 없습니다.

<br>

Cargo의 외부 크레이트 활용은 Cargo가 정말 빛을 발하는 부분입니다. `rand` 를 사용하는 코드를 작성하려면, 먼저 크레이트를 종속항목(dependency)에 포함하도록 *Cargo.toml*파일을 수정해야 합니다. *Cargo.toml*파일을 열고 맨 아래, Cargo가 생성한 `[dependencies]`섹션 헤더 아래에 다음 행을 추가하십시오. 아래의 버전 번호를 사용하여 특정 `rand` 를 정확하게 지정해야 합니다. 그렇지 않으면 이 자습서의 코드 예제가 작동하지 않을 수 있습니다.

파일 이름: Cargo.toml

```toml
[dependencies]
rand = "0.8.5"
```

*Cargo.toml* 파일에서 헤더 뒤에 오는 모든 것은 해당 섹션의 일부분이 되고, 다른 섹션이 시작될 때까지 계속됩니다.  `[dependencies]` 에서는 프로젝트가 의존하는 외부 크레이트와 그 크레이트의 버전을 Cargo 에 알려줍니다. 이 경우 `rand`크레이트를  `0.8.5` 시맨틱 버전 지정자로  지정합니다. 

<br>

Cargo는 버전 번호 작성의 표준인 [시맨틱 버전 관리](http://semver.org/) ( *SemVer* 라고도 함)를 이용합니다.  `0.8.5`지정자는 실제로는 `^0.8.5` 의 줄임말입니다. 이는 0.8.5 이상 0.9.0 미만의 모든 버전을 의미합니다.

Cargo는 이러한 버전이 버전 0.8.5와 호환되는 공개 API를 갖는 것으로 간주하며, 이 사양은 이 장의 코드와 함께 여전히 컴파일되는 최신 패치 릴리스를 얻을 수 있도록 보장합니다. 버전 0.9.0 이상은 다음 예제에서 사용하는 것과 동일한 API를 갖는다고 보장되지 않습니다.

<br>

`[dependencies]`만 추가하고, 코드를 변경하지 않은채 처음으로 프로젝트를 빌드해 보겠습니다.

```
$ cargo build
    Updating crates.io index
  Downloaded rand_chacha v0.3.1
  Downloaded ppv-lite86 v0.2.17
  Downloaded rand_core v0.6.4
  Downloaded rand v0.8.5
  Downloaded getrandom v0.2.9
  Downloaded cfg-if v1.0.0
  Downloaded libc v0.2.144
  Downloaded 7 crates (871.8 KB) in 1.28s
   Compiling libc v0.2.144
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.17
   Compiling getrandom v0.2.9
   Compiling rand_core v0.6.4
   Compiling rand_chacha v0.3.1
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
```

`cargo build`목록 2-2: rand 크레이트를 종속 항목으로 추가한 후 실행 결과

다른 버전 번호(그러나 SemVer 덕분에 모두 코드와 호환됨!)와 다른 라인(운영 체제에 따라 다름)이 표시될 수 있으며 라인의 순서가 다를 수 있습니다.

외부 종속성(dependency)을 포함하면 Cargo는 *레지스트리*에서 종속성이 필요로 하는 모든 것의 최신 버전을 가져옵니다. 레지스트리는 [Crates.io](https://crates.io/) 의 데이터 사본입니다. Crates.io는 Rust 생태계의 사람들이 다른 사람들이 사용할 수 있도록 오픈 소스 Rust 프로젝트를 게시하는 곳입니다.

레지스트리를 업데이트한 후 Cargo는 `[dependencies]`섹션을 확인하고 목록에 있는 아직 다운로드되지 않은 크레이트를 다운로드합니다. 이 경우, 의존성으로 `rand`만 나열했지만 Cargo는 `rand`가 의존하는 다른 크레이트도 가져왔습니다. 크레이트를 다운로드한 후, Rust는 크레이트를 컴파일한 다음, 사용 가능한 종속성과 함께 프로젝트를 컴파일합니다.

<br>

변경한 내용 없이 `cargo build` 를 다시 실행하면 `Finished`행 외에는 출력이 표시되지 않습니다. Cargo는 의존성을 이미 다운로드하고 컴파일했으며 *Cargo.toml* 파일에서 아무것도 변경하지 않았다는 것을 알고 있습니다. 또한 Cargo는 당신이 코드를 아무 것도 변경하지 않았다는 것을 알고 있으므로 그것을 다시 컴파일하지도 않습니다. 할 일 없으면 그냥 종료됩니다.

<br>

*src/main.rs* 파일을 열고 사소한 변경을 한 다음 저장하고 다시 빌드하면 두 줄의 출력만 표시됩니다.

```
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
```

*이 줄은 Cargo가 src/main.rs* 파일에 대한 작은 변경으로만 빌드를 업데이트한다는 것을 보여줍니다. 여러분의 종속성은 변경되지 않았으므로 Cargo는 이미 다운로드하고 컴파일한 것을 재사용할 수 있음을 알고 있습니다.

#### [*Cargo.lock* 파일로 재현 가능한 빌드 보장](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#ensuring-reproducible-builds-with-the-cargolock-file)

Cargo는 여러분 또는 다른 사람이 여러분의 코드를 빌드할 때마다 동일한 아티팩트를 다시 빌드할 수 있도록 보장하는 메커니즘을 가지고 있습니다. Cargo는 여러분이 수정하기 전까지 여러분이 지정한 종속성 버전만 사용합니다. 

예를 들어 다음 주에 `rand`크레이트의 버전 0.8.6이 나오고 해당 버전에 중요한 버그 수정이 포함되어 있지만 코드를 손상시키는 변경점이 포함되어 있다고 가정해 보겠습니다. 이를 처리하기 위해 Rust는 `cargo build` 를 처음 실행할 때 *Cargo.lock* 파일을 생성하며, *guessing_game* 디렉토리 에 이 파일이 있습니다.

```
guessing_game
  ├── Cargo.lock
  ├── Cargo.toml
  └── src
      └── main.rs
```

처음으로 프로젝트를 빌드할 때 Cargo는 기준에 맞는 종속성의 모든 버전을 파악한 다음 *Cargo.lock* 파일에 기록합니다. 나중에 프로젝트를 빌드할 때 Cargo는 *Cargo.lock* 파일이 존재하는지 확인하고 버전을 다시 파악하는 모든 작업을 수행하는 대신 여기에 지정된 버전을 사용할 것입니다. 이렇게 하면 재현 가능한 빌드가 자동으로 생성됩니다. 즉, *Cargo.lock* 파일 덕분에 명시적으로 업그레이드할 때까지 프로젝트가 0.8.5로 유지됩니다. *Cargo.lock* 파일은 재현 가능한 빌드에 중요하기 때문에 종종 프로젝트의 나머지 코드와 함께 소스 제어에 체크인됩니다.

#### [크레이트를 업데이트하여 새 버전 얻기](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#updating-a-crate-to-get-a-new-version)

*크레이트*를 업데이트하기위해 Cargo는  `update` 명령을 제공합니다. 이 명령어는 *Cargo.lock*파일을 무시 하고 *Cargo.toml* 의 사양에 맞는 모든 최신 버전을 파악합니다. 그다음 Cargo는 해당 버전을 *Cargo.lock* 파일에 기록합니다. 

`rand = "0.8.5"` 로 지정을 했고, 이것은 실제로는 `^0.8.5` 의 줄임말이기 때문에 Cargo는 기본적으로 0.8.5보다 크고 0.9.0보다 작은 버전만 찾습니다. `rand` 크레이트가 두 개의 새 버전 0.8.6 및 0.9.0을 릴리스한 경우 `cargo update` 를 실행하면 다음과 같이 표시됩니다. (실제로 이렇게 실행되지는 않습니다.)

```
$ cargo update
    Updating crates.io index
    Updating rand v0.8.5 -> v0.8.6
```

Cargo는 0.9.0 릴리스를 무시합니다. 이제 *Cargo.lock* 파일에서 `rand` 크레이트의 현재 사용 중인 버전이 0.8.6 으로 변경된 것을 확인할 수 있습니다.

<br>

 `rand` 의 버전 0.9.0 또는 0.9.x 버전을 사용하려면 *Cargo.toml* 파일을 다음과 같이 업데이트해야 합니다. (실제로 이렇게 실행되지는 않습니다.)

```toml
[dependencies]
rand = "0.9.0"
```

다음에 `cargo build` 를 실행하면 Cargo는 사용 가능한 크레이트의 레지스트리를 업데이트하고, 새로 지정된  `rand` 의 버전에 따라 요구 사항을 재평가합니다.

<br>

[Cargo](http://doc.crates.io/) 와 [그 생태계](http://doc.crates.io/crates-io.html) 에 대해 더 많은 이야기가 있습니다. 14장에서 다루겠지만 지금은 이것이 여러분이 알아야 할 전부입니다. Cargo는 라이브러리를 매우 쉽게 재사용할 수 있게 하므로 Rustaceans는 여러 패키지에서 조립되는 더 작은 프로젝트를 작성할 수 있습니다.



### [난수 생성](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number)

추측할 숫자를 생성하는 데 `rand` 크레이트 사용을 시작하겠습니다. 지금 할일은 Listing 2-3에 표시된 것처럼 *src/main.rs 를 업데이트하는 것입니다.*

파일 이름: src/main.rs

```rust
use std::io;
use rand::Rng;  // 추가

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100); //첫번째줄

    println!("The secret number is: {secret_number}");  //두번째줄

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

목록 2-3: 난수를 생성하는 코드 추가

먼저 `use rand::Rng;` 줄을 추가합니다 . `Rng` 트레잇(trait)은 난수 생성기가 구현하는 메서드를 정의합니다. 트레잇은 해당 메서드를 사용할 수 있는 범위 내에 있어야 합니다. 10장에서는 트레잇에 대해 자세히 다룰 것입니다.

다음으로 중간에 두 줄을 추가합니다. 

첫 번째 줄에서 우리가 사용할 특정 난수 생성기를 제공하는 함수 `rand::thread_rng`를 호출합니다. 이 생성기는 운영체제가 시드(seed)를 정하고 현재 스레드에서만 사용되는 특별한 정수생성기를 제공합니다. 그런 다음 난수 생성기에서  `gen_range` 메서드를 호출합니다. `gen_range` 메서드는 우리가 `use rand::Rng;`문으로 범위에 가져온 `Rng` 트레잇에 의해 정의됩니다. 

이 `gen_range` 메서드는 범위 식을 인수로 사용하고, 그 범위 안에서 난수를 생성합니다. 여기서 사용하는 범위 식은 `start..=end` 형식이며 하한과 상한을 포함하므로, 1에서 100 사이의 숫자를 요청하려면 `1..=100` 을 인수로 지정해야 합니다.

> 참고: 어떤 트레이트를 사용할지, 크레이트에서 어떤 메서드와 함수를 호출할지 알 수 없으므로 각 크레이트에는 사용 지침이 포함된 문서가 있습니다. Cargo의 또 다른 멋진 기능은 `cargo doc --open`명령을 실행하면 로컬에서 모든 종속 항목이 제공하는 문서를 빌드하고 브라우저에서 열 수 있습니다.  예를 들어 `rand`크레이트의 다른 기능에 관심이 있는 경우 `cargo doc --open` 을 실행 하고 왼쪽의 사이드바에서 `rand`를 클릭합니다. 
>
> Safari 에서 실행이 잘 안되는 경우, 주소를 복사해서 다른 브라우져를 사용해보세요.

두 번째 줄은 비밀번호를 출력합니다. 이는 테스트할 수 있도록 프로그램을 개발하는 동안만 사용하고 최종 버전에서는 삭제할 것입니다. 프로그램이 시작되자마자 답을 출력한다면 그것은 게임이 아닙니다!

프로그램을 몇 번 실행해 보십시오.

```
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

서로 다른 임의의 숫자를 가져와야 하며 모두 1에서 100 사이의 숫자여야 합니다. 잘하셨습니다!

## [추측과 비밀 번호 비교](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number)

이제 사용자 입력과 임의의 숫자가 있으므로 이를 비교할 수 있습니다. 그 단계는 목록 2-4에 나와 있습니다. 이 코드는 아직 정상적으로 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

목록 2-4: 두 숫자를 비교할 때 가능한 반환 값 처리

<br>

먼저 `use`명령문을 사용하여,  `std::cmp::Ordering` 타입을 표준 라이브러리에서 범위로 가져옵니다. `Ordering` 타입은 또 다른 열거형이며 `Less`, `Greater ` 및 `Equal`  변형(variant)가 있습니다. 이것들은 두 값을 비교할 때 가능한 세 가지 결과입니다.

<br>

그런 다음 `Ordering` 타입을 사용하는 코드 다섯 줄을 맨 아래에 추가합니다.  

이 `cmp` 메서드는 두 값을 비교하고, 비교할 수 있는 모든 것들에 위해 호출할 수 있습니다. 이 메소드는 비교하고 싶은 것들의 참조자를 받습니다.  여기서는 `guess` 와 `secret_number` 를 비교하고 있습니다. 그런 다음 `use `명령문을 사용하여 범위로 가져온 `Ordering` 열거형의 변형(variant)을 반환합니다. 

<br>

우리는 [`match`](https://doc.rust-lang.org/book/ch06-02-match.html) 표현문을 사용하여, `cmp` 가 `guess` 와 `secret_number` 를 비교한 결과인 `Ordering`의 변형(variant)값에 따라 무엇을 할 것인지 결정할 수 있습니다. 

`match` 표현식은 *arm* 으로 이루어져 있습니다.  arm은 일치시킬 *패턴* 과 실행될 코드로 구성되는데, 이 코드는 `match` 에 주어진 값과 arm의 패턴이 일치하는 경우에 실행됩니다. Rust는 `match` 에 주어진 값을 받아서 각 arm의 패턴을 차례로 살펴봅니다. 패턴과 `match` 생성자는 강력한 Rust 기능입니다. 패턴과 구성은 코드에서 발생할 수 있는 다양한 상황을 표현할 수 있게 하고 모든 상황을 처리하도록 합니다. 이러한 기능은 각각 6장과 18장에서 자세히 다룰 것입니다.

<br>

여기서 사용하는 `match` 표현식을 예로 들어 보겠습니다. 사용자가 50을 추측했고 이번에 무작위로 생성된 비밀 번호는 38이라고 가정합니다.

코드가 50과 38을 비교하면 50이 38보다 크기 때문에 `cmp`메서드가 `Ordering::Greater` 를 반환합니다. `match` 표현식은 `Ordering::Greater` 값을 가져오고 각 arm의 패턴을 확인하기 시작합니다. 첫 번째 arm의 패턴은 `Ordering::Less` 이기 때문에, `Ordering::Greater` 값과 일치하지 않습니다. 그러면 해당 arm의 코드를 무시하고 다음 arm으로 이동합니다. 다음 팔의 패턴은 `Ordering::Greater` 이라서 일치합니다! 해당 arm의 관련 코드가 실행되어 화면에 `Too big!` 이 출력됩니다. `match` 표현식은 첫 번째 성공적인 일치  후에 끝나므로, 이 시나리오에서는 마지막 arm은 보지 않습니다.

<br>

그런데 목록 2-4의 코드는 아직 컴파일되지 않습니다. 해봅시다:

```
$ cargo build
   Compiling libc v0.2.86
   Compiling getrandom v0.2.2
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.10
   Compiling rand_core v0.6.2
   Compiling rand_chacha v0.3.0
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
error[E0308]: mismatched types
  --> src/main.rs:22:21
    |
22  |     match guess.cmp(&secret_number) {
    |                 --- ^^^^^^^^^^^^^^ expected `&String`, found `&{integer}`
    |                 |
    |                 arguments to this method are incorrect
    |
    = note: expected reference `&String`
               found reference `&{integer}`

note: associated function defined here
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/cmp.rs:783:8

For more information about this error, try `rustc --explain E0308`.
error: could not compile `guessing_game` due to previous error
```

오류(error)의 핵심은 *일치하지 않는 유형* 이 있다는 것입니다. Rust는 강력한 정적 유형 시스템을 가지고 있습니다. 그러나 타입 유추도 있습니다. 우리가 `let mut guess = String::new()` 를 작성했을 때 Rust는 `guess` 가 `String` 타입이여야 한다고 타입을 유추했기때문에 우리가 타입을 직접 지정하라고 하지 않았습니다. 반면에 `secret_number` 는 숫자(number) 타입입니다. 

<br>

몇몇 숫자 타입들이 1과 100 사이의 값을 가질 수 있습니다. `i32`는 32비트 정수, `u32`는 32비트의 부호없는 정수, `i64`는 64비트의 정수이며 그 외에도 비슷합니다. d우리가 따로 타입을 지정하지 않으면서, 다른 정수형임을 추론할 수 있는 타입 정보를 제공하지 않는다면, 러스트는 기본적으로 숫자들을 `i32` 타입으로 생각합니다. 이 오류의 원인은 러스트가 문자열과 숫자 타입을 비교할 수 없기 때문입니다.

<br>

궁극적으로 우리는 프로그램이 입력으로 받은 `String`을 숫자 타입으로 변환하여, 입력받은 숫자와 비밀번호를 비교하려고 합니다. `main` 함수 본문 에 다음 줄을 추가하면 됩니다. 

파일 이름: src/main.rs

```rust
    // --snip--

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
```

<br>

추가된 줄은 다음과 같습니다.

```rust
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

 `guess` 라는 변수를 만듭니다. 하지만 잠깐, 프로그램에 이미  `guess` 라는 변수가 있지 않습니까? 그렇지만, 유용한 Rust는  `guess` 의 이전 값을 새 값으로 가리게(shadow) 해줍니다. *섀도잉*을 사용하면, 예를 들어 `guess_str` 와 `guess` 같은 두 개의 고유한 변수를 만들지 않고도, `guess` 라는 변수 이름을 재사용할 수 있습니다. [3장](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#shadowing)에서 이에 대해 자세히 다루겠지만, 지금은 이 기능이 값의 유형을 변환하기 위해 자주 사용된다는 점을 알아 두세요.

<br>

우리는 새로만든 변수 `guess`를 `guess.trim().parse()` 표현식과 묶습니다. 표현식 안에 있는  `guess` 는 문자열로 입력 값을 입력받은 원래의  `guess` 변수를 나타냅니다. `String` 인스턴스의 `trim` 메서드는 처음과 끝 부분의 빈칸을 제거하는데, 이 작업은 문자열을 숫자 데이터와 비교하기 위해 반드시 필요합니다. 사용자는 추측하는 숫자를 입력하고 `read_line` 을 만족시키기 위해 Enter 키를 눌러야 합니다. 그러면 문자열에 개행 문자가 추가됩니다. 

예를 들어 사용자가 5를 입력 하고 Enter 키를 누르면 `guess` 는 `5\n` 이 됩니다. 여기서 `\n` 는 "개행"을 나타냅니다. (Windows에서 Enter 키를 누르면 캐리지 리턴과 줄 바꿈이 발생해서 `\r\n` 이 됩니다.) 이 `trim`메서드는 `\n` 또는 `\r\n `를 제거하여 `5` 만 남깁니다.

<br>

[문자열에 대한 parse 메소드](https://doc.rust-lang.org/std/primitive.str.html#method.parse) 는 문자열을 다른 유형으로 변환합니다. 여기서는 문자열을 숫자로 변환하는 데 사용합니다. `let guess: u32` 를 사용하여 원하는 정확한 숫자 유형을 Rust에 알려야 합니다. `guess` 뒤의 콜론( `:`)은 우리가 Rust에게 변수의 타입을 명시한다고 알려줍니다. Rust에는 몇 가지 내장 숫자 유형이 있습니다. 여기서 보이는 `u32` 는 부호 없는 32비트 정수입니다. 이 타입은 작은 양수에 대한 좋은 기본 선택입니다. 다른 숫자 유형에 대해서는  [3장](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types)에서 배우게 됩니다.

또한, 이 예제 프로그램에서 `u32` 타입을 지정하고, `secret_number` 와 비교했다는 것은,  Rust가 추론할 것 `secret_number` 가 `u32` 로 유추해야 함을 의미합니다. 이제 동일한 유형의 두 값을 비교합니다!

이 `parse`메서드는 논리적으로 숫자로 변환할 수 있는 글자에만 작동하므로 쉽게 오류가 발생할 수 있습니다. 예를 들어 문자열에 `A👍%` 가 포함된 경우 이를 숫자로 변환할 방법이 없습니다. 실패할 수 있기 때문에, `parse` 메서드는 `read_line` 메서드 결과와 마찬가지로 `Result` 타입을 반환합니다 (이전의 ["Result 로 잠재적 오류 처리 ](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result)["](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result) 에서 설명 ). 

우리는 이 `Result` 를 `expect` 메소드를 다시 사용하여 동일한 방식으로 처리합니다. 만약 `parse` 가 문자열에서 숫자를 생성할 수 없다는 이유로 `Err` 라는 `Result` 변형자(variant)를 반환한다면,   `expect` 는 게임을 중단하고 우리가 제공하는 메시지를 인쇄합니다. 만약 `parse` 가 문자열을 숫자로 성공적으로 변환할 수 있으면, `Ok` 라는 `Result` 변형자(variant)가 반환되고 `expect` 는 `Ok` 값 에서 원하는 숫자를 반환합니다 .

이제 프로그램을 실행해 봅시다:

```
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Nice! 추측값 입력 전에 공백을 추가해도 프로그램은 여전히 사용자가 76을 추측했음을 알아냈습니다. 프로그램을 몇 번 실행하여 다른 종류의 입력으로 다른 동작을 확인합니다. 너무 낮은 숫자, 정확한 숫자, 높은 숫자를 입력해보세요. 

이제 게임이 잘 작동하지만 우리는 한번엔 한 가지만 추측할 수 있습니다. 루프를 추가하여 변경해 봅시다!

## [루핑(Looping)으로 여러번의 추측 허용](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#allowing-multiple-guesses-with-looping)

`loop` 키워드 는 무한 루프를 생성합니다. 사용자가 숫자를 추측할 수 있는 더 많은 기회를 제공하기 위해 루프를 추가합니다.

파일 이름: src/main.rs

```rust
    // --snip--

    println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
```

보시다시피 추측값 입력 프롬프트의 모든것을 루프로 이동했습니다. 루프 내부의 행을 각각 4칸 더 들여쓰기하고 프로그램을 다시 실행하십시오. 이제 프로그램은 영원히 또 다른 추측값 입력을 요구할 것이며, 이것은 새로운 문제를 만듭니다.  사용자가 종료할 수 없는 것 입니다!

사용자는 언제나 키보드 단축키 ctrl-c를 사용하여 프로그램을 중단할 수 있습니다. 그러나 ["추측과 비밀 번호의 비교"](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number)에서 `parse` 를 언급한 것처럼, 이 만족할 줄 모르는 괴물을 피할 수 있는 또 다른 방법이 있습니다. 사용자가 숫자가 아닌 답을 입력하면 프로그램이 중단(crash)됩니다. 다음과 같이 사용자가 종료할 수 있도록 이점을 활용할 수 있습니다.

```
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread 'main' panicked at 'Please type a number!: ParseIntError { kind: InvalidDigit }', src/main.rs:28:47
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`quit` 를 입력하면 게임이 종료되지만, 알다시피 숫자가 아닌 다른 입력도 마찬가지입니다. 이것은 차선책입니다. 우리는 정확한 숫자가 추측되면 게임도 멈추기를 원합니다.

### [정답 이후 종료하기](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess)

`break` 문장을 추가하여, 사용자가 이기면 게임이 종료되도록 프로그래밍해 보겠습니다 .

파일 이름: src/main.rs

```rust
        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

`You win!` 다음에 `break` 줄을 추가하면 사용자가 비밀 번호를 올바르게 추측할 때 프로그램이 루프를 종료합니다. 루프 종료는 프로그램 종료를 의미하기도 합니다. 루프는 `main` 의 마지막 부분이기 때문입니다 .

### [잘못된 입력 처리](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-invalid-input)

게임의 동작을 더욱 개선하기 위해, 사용자가 숫자가 아닌 것을 입력할 때 프로그램이 충돌(crash)하는 대신, 사용자가 계속 추측할 수 있도록 게임이 숫자가 아닌 것을 무시하도록 합시다. Listing 2-5와 같이`guess` 가  `String `에서 `u32 `로 변환되는 행을 변경하면 됩니다.

파일 이름: src/main.rs

```rust
        // --snip--

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        // --snip--
```

목록 2-5: 프로그램을 충돌시키는 대신, 숫자가 아닌 추측을 무시하고 다른 추측을 요청

`expect` 호출에서 `match` 표현식으로 전환하여, 오류 발생시 충돌하는 대신 오류를  처리하는 방식으로 바꾸었습니다. `parse` 는 `Result` 타입을 반환 하고, `Result` 는 `Ok` 와 `Err` 변형자를 포함하는 열거형임을 기억하세요. 여기서 우리는 `match` 표현식을 사용하고 있는데, `cmp` 메소드의 `Ordering` 결과를 처리했을 때와 같습니다. 


만약 `parse` 가 문자열을 숫자로 성공적으로 변환할 수 있으면 결과 숫자를 포함하는 `Ok ` 값을 반환합니다. `Ok` 값은 첫 번째 arm의 패턴과 일치하며,  `match` 표현식은 `num` 값을 반환합니다. `num` 값은 `parse` 가 생성하고 `Ok` 값 안에 넣어준 값 입니다. 그 숫자는 결국 우리가 만들고 있는 새 `guess` 변수에 위치합니다.

만약 `parse` 가 문자열을 숫자로 변환 할 수 *없으면*, 오류에 대한 자세한 정보가 포함된 `Err` 값을 반환합니다.  `Err` 값은 첫 번째 arm에 있는 `Ok(num)` 패턴과 일치하지 않지만 두 번째 arm의 `Err(_)` 패턴과 일치합니다. 밑줄은 포괄적인 값입니다. 이 예시에서 밑줄을 사용함으로서, 우리는 `Err` 내부에 어떤 정보가 있든 관계없이 일치시키고 싶다고 말합니다. 따라서 프로그램은 두 번째 arm의 코드인 `continue` 를 실행하여 프로그램이 `loop` 의 다음 반복으로 이동하여 다른 추측을 요청하도록 지시합니다. 이렇게 프로그램은 `parse` 가 만날 수 있는 모든 오류를 효과적으로 무시합니다! 

<br>


이제 프로그램의 모든 것이 예상대로 작동해야 합니다. 해봅시다:

```
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 4.45s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

훌륭합니다! 작은 마지막 조정 하나만 하고 추측 게임을 끝낼 것입니다. 프로그램이 여전히 비밀 번호를 미리 출력하고 있음을 기억하십시오. 테스트에는 잘 작동했지만 게임을 망쳤습니다. 비밀 번호를 출력하는 `println!` 를 삭제합시다. 목록 2-6은 최종 코드를 보여줍니다.

<br>

파일 이름: src/main.rs

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

Listing 2-6: 완전한 추측 게임 코드

<br>

이제 추측 게임을 성공적으로 구축했습니다. 축하해요!

## [요약](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#summary)

이 프로젝트는 `let`, `match`, 함수, 외부 크레이트 사용 등 많은 새로운 Rust 개념을 소개하는 실습 방식이었습니다. 다음 장에서는 이러한 개념에 대해 자세히 알아봅니다. 3장은 변수, 데이터 유형, 함수와 같이 대부분의 프로그래밍 언어가 가지고 있는 개념을 다루고 Rust에서 사용하는 방법을 보여줍니다. 4장은 Rust를 다른 언어와 다르게 만드는 기능인 소유권(ownership)을 탐구합니다. 5장에서는 구조체(structs) 및 메서드 구문(method syntax)에 대해 설명하고 6장에서는 열거형(enums)의 작동 방식을 설명합니다.
