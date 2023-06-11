+++
title = "Functions"
weight = 3
template ="book_page_rust.html"

+++

## 요약

- 함수,변수 이름은 `소문자`와 ` _ `를 사용하는 스네이크 케이스 형식을 따름

- 사용할 함수를 정의하는 위치는 main 함수 전,후 모두 가능

- 구문과 표현식
  - 구문(Statements)은 어떤 작업을 수행하고, 값은 반환하지 않음.(예: 변수선언, 함수정의)
  - 표현식(Expressions)은 결과값을 반환.(예: 수학연산, 함수호출, 매크로호출, 새 범위를 만드는 {})

<!-- more -->

## [함수](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#functions)

함수는 Rust 코드에서 널리 사용됩니다. 당신은 이미 rust 언어에서 가장 중요한 함수 중 하나인 `main` 함수를 보았습니다. 이것은 많은 프로그램의 시작점입니다. 또한 새 함수를 선언할 수 있는 `fn` 키워드도 보았습니다.

Rust는 함수 및 변수 이름에 대한 일반적인 스타일로 스네이크 케이스를 사용합니다. 여기서 모든 문자는 소문자이고 밑줄은 별도의 단어입니다.

다음 프로그램은 함수를 정의하는 예시입니다.

파일 이름: src/main.rs

```rust
fn main() {
    println!(`Hello, world!`);

    another_function();
}

fn another_function() {
    println!(`Another function.`);
}
```

Rust 에서 `fn` 뒤에 함수 이름과 괄호를 입력하여 함수를 정의합니다. 중괄호는 컴파일러에게 함수 본문이 시작되고 끝나는 위치를 알려줍니다.

<br />

함수를 호출하는 방법은 함수 이름 뒤에 괄호를 입력하는 것 입니다. `another_function`은 프로그램에 정의되어 있기 때문에 `main` 함수 내부에서 호출할 수 있습니다. 소스 코드에서 `main` 함수 다음에 `another_function`을 정의했습니다. `main` 함수 전에도 정의할 수 있었습니다. Rust는 당신이 당신의 함수를 정의하는 곳을 신경쓰지 않습니다. 어디든 호출자가 볼 수 있는 범위에 정의되어 있으면 됩니다.

<br>

함수를 더 자세히 살펴보기 위해 functions 라는 이름의 새 binary 프로젝트를 시작하겠습니다. src/main.rs 에 위 예제를 작성하고 실행합니다. 결과는 다음과 같아야 합니다.

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28s
     Running `target/debug/functions`
Hello, world!
Another function.
```

행은 `main` 함수에 나타나는 순서대로 실행됩니다. 먼저 `Hello, world!` 메시지가 출력된 다음, `another_function`이 호출되고 해당 메시지가 출력됩니다.

### [매개변수(Parameters)](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#parameters)

함수는 매개변수를 갖는 형식으로 정의할 수 있습니다. 함수에 매개변수가 있는 경우 해당 매개변수에 대한 구체적인 값을 함수에 제공할 수 있습니다. 기술적으로는 구체적인 값을 전달인자(arguments)라고 부르지만, 일상적인 대화에서 사람들은 함수의 정의나 호출에서 매개변수와 전달인자라는 단어를 같은 의미로 사용하는 경향이 있습니다.

- 전달인자(arguments): 함수를 호출할 때 전달되는 구체적인 값
- 매개변수(Parameters): 함수를 정의할 때 사용되는 변수이름

이 버전의 `another_function`에서는 매개변수를 추가합니다.

파일 이름: src/main.rs

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!(`The value of x is: {x}`);
}
```

이 프로그램을 실행해 보십시오. 다음 출력을 얻어야 합니다.

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21s
     Running `target/debug/functions`
The value of x is: 5
```

`another_function` 선언에는 `x`라는 매개변수가 하나 있습니다. `x`의 유형은 `i32`로 지정됩니다. `another_function`에 `5`를 전달하면 `println!` 매크로는 `x`를 포함하는 중괄호 쌍이 형식 문자열에 있던 곳에 `5`를 넣습니다.

함수를 선언할 때, 각 매개변수의 타입을 선언해야 합니다. 이는 Rust의 설계에서 신중한 결정입니다. 함수 정의에 타입 주석을 요구한다는 것은, 컴파일러가 타입을 파악하기 위해 코드의 다른 곳에서 우리가 사용하는 것을 통해 타입을 추측할 필요가 없어진다는 것을 의미합니다. 또한 컴파일러는 함수가 기대하는 타입을 알고 있는 경우 더 유용한 오류 메시지를 제공할 수 있습니다.

<br>

여러개의 매개변수를 정의하려면, 다음과 같이 매개변수 사이를 쉼표로 구분합니다.

파일 이름: src/main.rs

```rust
fn main() {
    print_labeled_measurement(5, `h`);
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!(`The measurement is: {value}{unit_label}`);
}
```

이 예제에서는 두 개의 매개변수를 사용하여 `print_labeled_measurement`라는 함수를 생성합니다. 첫 번째 매개변수의 이름은 `value`이고 `i32` 타입입니다. 두 번째는 `unit_label`이라는 이름이고 `char` 타입입니다. 이 후 이 함수는 `value`와 `unit_label`을 모두 포함하는 텍스트를 인쇄합니다.

이 코드를 실행해 봅시다. 현재 함수 프로젝트의 src/main.rs 파일 에 있는 프로그램을 이전 예제로 바꾸고 `cargo run`을 사용하여 실행합니다.

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/functions`
The measurement is: 5h
```

`value`의 값이 `5`이고 `unit_label`의 값이 ``h``인 함수를 호출했기 때문에 프로그램 출력에는 해당 값이 포함됩니다.

### [구문과 표현식(Statements and Expressions)](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#statements-and-expressions)

함수 본문은 표현식으로 끝나는 명령문들로 구성됩니다. 지금까지 우리가 다룬 함수에는 종결 표현식이 포함되지 않았지만, 명령문의 일부로 표현식을 보았습니다. Rust는 표현식 기반 언어이기 때문에 이해해야 할 중요한 차이점입니다. 다른 언어에는 동일한 구분이 없으므로 명령문과 표현식이 무엇이며, 그 차이가 함수 본문에 어떤 영향을 미치는지 살펴보겠습니다.

- 구문(Statements)은 어떤 작업을 수행하고, 값은 반환하지 않는 지침입니다.
- 표현식(Expressions)은 결과값을 반환합니다.

몇 가지 예를 살펴보겠습니다.

우리는 이미 구문과 표현식을 사용했습니다. 변수를 만들고 `let` 키워드를 사용하여 값을 할당하는 것이 구문입니다. 목록 3-1에서 `let y = 6;`은 구문입니다.

파일 이름: src/main.rs

```
fn main() {
    let y = 6;
}
```

목록 3-1: 하나의 명령문을 포함하는 `main` 함수 선언

함수 정의도 구문입니다. 앞의 전체 예는 그 자체로 구문입니다.

<br>

구문은 값을 반환하지 않습니다. 따라서 다음 코드와 같이 `let` 문을 다른 변수에 할당할 수 없습니다. 오류가 발생합니다.

파일 이름: src/main.rs

```
fn main() {
    let x = (let y = 6);
}
```

이 프로그램을 실행하면 다음과 같은 오류가 발생합니다.

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found `let` statement
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^

error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: variable declaration using `let` is a statement

error[E0658]: `let` expressions in this position are unstable
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information

warning: unnecessary parentheses around assigned value
 --> src/main.rs:2:13
  |
2 |     let x = (let y = 6);
  |             ^         ^
  |
  = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
  |
2 -     let x = (let y = 6);
2 +     let x = let y = 6;
  |

For more information about this error, try `rustc --explain E0658`.
warning: `functions` (bin `functions`) generated 1 warning
error: could not compile `functions` due to 3 previous errors; 1 warning emitted
```

`let y = 6` 문은 값을 반환하지 않으므로 `x`가 바인딩할 항목이 없습니다. C 및 Ruby 와 같은 언어와 다른 점입니다. 이러한 언어에서는 `x = y = 6`이라고 쓰고 `x`와 `y` 모두 값이 `6`이 되도록 할 수 있습니다. Rust에서는 그렇지 않습니다.

<Br>

표현식은 값으로 평가(반환)되며, Rust 코드 대부분이 표현식입니다. `5 + 6` 과 같은 간단한 수학 연산을 살펴보면, 이는 `11`이란 값을 산출하는 표현식입니다. 표현식은 구문의 일부가 될 수 있습니다. Listing 3-1에서 `let y = 6;` 구문의 `6`은 값 `6`으로 평가되는 표현식입니다.

함수 호출, 매크로 호출, 새로운 범위를 만드는 {} 은 모두 표현식입니다. 예를 들면 다음과 같습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!(`The value of y is: {y}`);
}
```

위 코드에서 아래 부분은 표현식입니다:

```rust
{
    let x = 3;
    x + 1
}
```

이 경우 `4`로 평가되는 블록입니다. 해당 값은 `let` 문의 일부로 `y`에 바인딩됩니다. `x + 1` 행에는 지금까지 본 대부분의 행과 달리 끝에 세미콜론이 없습니다. 표현식에는 마지막에 세미콜론이 붙지 않습니다. 표현식 끝에 세미콜론을 추가하면 문장으로 바뀌고 값을 반환하지 않습니다. 이후에 함수의 반환 값과 표현식을 살펴볼 때 이 점을 유의하세요.

### [반환 값이 있는 함수](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html#functions-with-return-values)

함수는 이를 호출하는 코드에 값을 반환할 수 있습니다. 반환 값의 이름은 지정하지 않지만 화살표(`->`) 다음에 반환 값의 유형을 선언해야 합니다. Rust에서 함수의 반환 값은 함수 본문 블록의 최종 표현식 값과 동의어입니다. 함수에서 `return` 키워드와 값을 사용하여 일찍 반환할 수 있지만, 대부분의 함수는 암시적으로 마지막 표현식을 반환합니다.

다음은 값을 반환하는 함수의 예입니다.

파일 이름: src/main.rs

```
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!(`The value of x is: {x}`);
}
```

`five` 함수에는 함수 호출, 매크로 또는 `let` 문이 없으며 숫자 `5`만 있습니다. 이것은 Rust에서 완벽하게 유효한 함수입니다. 함수의 반환 유형도 `-> i32`로 지정됩니다. 이 코드를 실행해 보십시오. 출력은 다음과 같습니다.

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/functions`
The value of x is: 5
```

`five`의 `5`는 함수의 반환 값이며 반환 타입이 `i32`입니다. 이에 대해 자세히 살펴보겠습니다. 두 가지 중요한 부분이 있습니다.

첫째, `let x = five();` 줄입니다. 변수를 초기화하기 위해 함수의 반환 값을 사용하고 있음을 보여줍니다. 함수 `five`는 `5`를 반환하므로 해당 행은 다음과 동일합니다.

```rust
let x = 5;
```

둘째, `five` 함수는 매개변수가 없고 반환 값의 유형을 정의하지만 함수의 본문은 우리가 반환하려는 값을 가진 표현식이기 때문에 세미콜론이 없는 외로운 `5`입니다.

<br>

다른 예를 살펴보겠습니다.

파일 이름: src/main.rs

```
fn main() {
    let x = plus_one(5);

    println!(`The value of x is: {x}`);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

이 코드를 실행하면 `The value of x is: 6`이 인쇄됩니다.

<br>

그러나 `x + 1`을 포함하는 줄 끝에 세미콜론을 배치하여 표현식에서 명령문으로 변경하면 오류가 발생합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = plus_one(5);

    println!(`The value of x is: {x}`);
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

이 코드를 컴파일하면 다음과 같은 오류가 발생합니다.

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error[E0308]: mismatched types
 --> src/main.rs:7:24
  |
7 | fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
8 |     x + 1;
  |          - help: remove this semicolon to return this value

For more information about this error, try `rustc --explain E0308`.
error: could not compile `functions` due to previous error
```

기본 오류 메시지인 `mismatched types`은 이 코드의 핵심 문제를 나타냅니다. `plus_one` 함수의 정의는 `i32`를 반환한다고 하지만 `x + 1;` 구문은 값을 산출(반환)하지 않기 때문에, `()` 처럼 비어있는 튜플로 표현됩니다. 따라서 이 함수에서는 아무 것도 반환되지 않으며, 이는 함수 정의와 모순되고 오류가 발생합니다. 이 출력에서 ​​Rust는 이 문제를 수정하는 데 도움이 될 수 있는 메시지를 제공합니다. 오류를 수정하는 세미콜론을 제거할 것을 제안합니다.
