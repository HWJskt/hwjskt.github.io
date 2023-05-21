+++
title = "Variables and Mutability"
weight = 1
template = "book_page_rust.html"

+++

## 요약

- 

- 

<!-- more -->

## [변수와 가변성](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#variables-and-mutability)

[`변수로 값 저장`](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#storing-values-with-variables) 섹션 에서 언급했듯이 기본적으로 변수는 변경할 수 없습니다. 이것은 Rust가 제공하는 안전하고 쉬운 동시성을 활용하는 방식으로 코드를 작성하도록 Rust가 제공하는 많은 넛지 중 하나입니다. 그러나 여전히 변수를 변경 가능하게 만드는 옵션이 있습니다. Rust가 불변성을 선호하도록 권장하는 방법과 이유, 그리고 왜 때때로 선택 해제를 원할 수 있는지 살펴보겠습니다.

변수가 변경 불가능한 경우 값이 이름에 바인딩되면 해당 값을 변경할 수 없습니다. 이를 설명하기 위해 `cargo new variables`를 사용하여 *프로젝트* 디렉토리 에 *variables* 라는 새 프로젝트를 생성합니다 .

그런 다음 새 *변수 디렉토리에서* *src/main.rs를* 열고 해당 코드를 아직 컴파일되지 않는 다음 코드로 바꿉니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 5;
    println!(`The value of x is: {x}`);
    x = 6;
    println!(`The value of x is: {x}`);
}
```

`cargo run`을 사용하여 프로그램을 저장하고 실행합니다. 다음 출력과 같이 불변성 오류에 관한 오류 메시지를 수신해야 합니다.

```console
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!(`The value of x is: {x}`);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error
```

이 예제는 컴파일러가 프로그램에서 오류를 찾는 데 어떻게 도움이 되는지 보여줍니다. 컴파일러 오류는 실망스러울 수 있지만 실제로는 프로그램이 원하는 작업을 아직 안전하게 수행하지 못한다는 의미일 뿐입니다. 그들은 당신이 좋은 프로그래머가 아니라는 것을 의미 *하지 않습니다 !* 숙련된 Rustacean은 여전히 컴파일러 오류가 발생합니다.

변경할 수 없는 `x` 변수에 두 번째 값을 할당하려고 했기 때문에 `변경할 수 없는 변수 `x`에 두 번 할당할 수 없습니다.`라는 오류 메시지가 표시되었습니다.

바로 이 상황이 버그로 이어질 수 있기 때문에 변경할 수 없는 것으로 지정된 값을 변경하려고 시도할 때 컴파일 타임 오류가 발생하는 것이 중요합니다. 코드의 한 부분이 값이 절대 변경되지 않는다는 가정하에 작동하고 코드의 다른 부분이 해당 값을 변경하는 경우 코드의 첫 번째 부분이 설계된 대로 작동하지 않을 수 있습니다. 이러한 종류의 버그의 원인은 사후에 추적하기 어려울 수 있습니다. 특히 두 번째 코드 조각이 가끔 값을 변경하는 경우에는 더욱 *그렇습니다* . Rust 컴파일러는 값이 변경되지 않는다고 선언하면 실제로 변경되지 않으므로 직접 추적할 필요가 없습니다. 따라서 코드를 추론하기가 더 쉽습니다.

그러나 가변성은 매우 유용할 수 있으며 코드를 작성하기 더 편리하게 만들 수 있습니다. [변수는 기본적으로 변경할 수 없지만 2장](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#storing-values-with-variables) 에서 했던 것처럼 변수 이름 앞에 `mut`를 추가하여 변수를 변경할 수 있습니다 . 또한 `mut`를 추가하면 코드의 다른 부분이 이 변수의 값을 변경할 것임을 나타내어 코드의 향후 독자에게 의도를 전달합니다.

*예를 들어 src/main.rs를* 다음과 같이 변경해 보겠습니다 .

파일 이름: src/main.rs

```rust
fn main() {
    let mut x = 5;
    println!(`The value of x is: {x}`);
    x = 6;
    println!(`The value of x is: {x}`);
}
```

지금 프로그램을 실행하면 다음과 같은 결과가 나타납니다.

```console
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

`mut`이 사용될 때 `x`에 바인딩된 값을 `5`에서 `6`으로 변경할 수 있습니다. 궁극적으로 가변성을 사용할지 여부를 결정하는 것은 귀하에게 달려 있으며 특정 상황에서 가장 명확하다고 생각하는 것에 따라 다릅니다.

### [상수](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#constants)

변경할 수 없는 변수와 마찬가지로 *상수는* 이름에 바인딩되어 변경이 허용되지 않는 값이지만 상수와 변수 사이에는 몇 가지 차이점이 있습니다.

첫째, 상수와 함께 `mut`를 사용할 수 없습니다. 상수는 기본적으로 변경할 수 없을 뿐만 아니라 항상 변경할 수 없습니다. `let` 키워드 대신 `const` 키워드를 사용하여 상수를 선언하고 값의 유형에 주석을 *달아야 합니다* . [다음 섹션인 `데이터 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) 에서 유형 및 유형 주석을 다룰 것이므로 지금은 세부 사항에 대해 걱정하지 마십시오. 항상 유형에 주석을 달아야 한다는 점만 알아두세요.

상수는 전역 범위를 포함하여 모든 범위에서 선언할 수 있으므로 코드의 많은 부분에서 알아야 하는 값에 유용합니다.

마지막 차이점은 런타임에만 계산할 수 있는 값의 결과가 아니라 상수 식으로만 상수를 설정할 수 있다는 것입니다.

다음은 상수 선언의 예입니다.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

상수의 이름은 `THREE_HOURS_IN_SECONDS`이고 값은 60(1분의 초 수)에 60(1시간의 분 수)에 3(세고 싶은 시간 수)을 곱한 결과로 설정됩니다. 이 프로그램). 상수에 대한 Rust의 명명 규칙은 단어 사이에 밑줄과 함께 모두 대문자를 사용하는 것입니다. 컴파일러는 컴파일 시간에 제한된 작업 집합을 평가할 수 있으므로 이 상수를 값 10,800으로 설정하는 대신 이해하고 확인하기 쉬운 방식으로 이 값을 작성하도록 선택할 수 있습니다. 상수를 선언할 때 사용할 수 있는 작업에 대한 자세한 내용은 [Rust 참조의 상수 평가 섹션을](https://doc.rust-lang.org/reference/const_eval.html) 참조하세요 .

상수는 선언된 범위 내에서 프로그램이 실행되는 전체 시간 동안 유효합니다. 이 속성은 게임의 모든 플레이어가 획득할 수 있는 최대 점수 또는 빛의 속도와 같이 프로그램의 여러 부분에서 알아야 할 수 있는 응용 프로그램 도메인의 값에 대해 상수를 유용하게 만듭니다.

프로그램 전체에서 사용되는 하드코딩된 값의 이름을 상수로 지정하면 향후 코드 관리자에게 해당 값의 의미를 전달하는 데 유용합니다. 또한 나중에 하드코딩된 값을 업데이트해야 하는 경우 변경해야 하는 코드의 한 위치만 지정하는 데 도움이 됩니다.

### [섀도잉](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#shadowing)

[2장의](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number) 추측 게임 튜토리얼에서 본 것처럼 이전 변수와 동일한 이름으로 새 변수를 선언할 수 있습니다. Rustaceans는 첫 번째 변수가 두 번째 변수에 의해 *가려진다고* 말하는데 , 이는 변수 이름을 사용할 때 컴파일러가 두 번째 변수를 보게 된다는 의미입니다. 실제로 두 번째 변수는 첫 번째 변수를 가리고 변수 자체가 가려지거나 범위가 끝날 때까지 변수 이름을 자신에게 사용합니다. 동일한 변수의 이름을 사용하고 다음과 같이 `let` 키워드를 반복 사용하여 변수를 숨길 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!(`The value of x in the inner scope is: {x}`);
    }

    println!(`The value of x is: {x}`);
}
```

이 프로그램은 먼저 `x`를 `5` 값에 바인딩합니다. 그런 다음 `let x =`를 반복하여 원래 값에 `1`을 추가하여 `x`의 값이 `6`이 되도록 새 변수 `x`를 만듭니다. 그런 다음 중괄호로 만든 내부 범위 내에서 세 번째 `let` 문도 `x`를 가리고 새 변수를 만들어 이전 값에 `2`를 곱하여 `x` 값 `12`를 제공합니다. 해당 범위가 끝나면 내부 그림자가 종료되고 `x`는 `6`으로 돌아갑니다. 이 프로그램을 실행하면 다음과 같이 출력됩니다.

```console
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/variables`
The value of x in the inner scope is: 12
The value of x is: 6
```

`let` 키워드를 사용하지 않고 실수로 이 변수에 재할당하려고 하면 컴파일 타임 오류가 발생하기 때문에 섀도잉은 변수를 `mut`로 표시하는 것과 다릅니다. `let`을 사용하여 값에 대해 몇 가지 변환을 수행할 수 있지만 이러한 변환이 완료된 후에는 변수를 변경할 수 없습니다.

`mut`와 섀도잉의 또 다른 차이점은 `let` 키워드를 다시 사용할 때 새 변수를 효과적으로 생성하기 때문에 값의 유형을 변경할 수 있지만 동일한 이름을 재사용할 수 있다는 것입니다. 예를 들어 프로그램에서 사용자에게 공백 문자를 입력하여 일부 텍스트 사이에 원하는 공백 수를 표시하도록 요청한 다음 해당 입력을 숫자로 저장하려고 한다고 가정해 보겠습니다.

```rust
    let spaces = `   `;
    let spaces = spaces.len();
```

첫 번째 `spaces` 변수는 문자열 유형이고 두 번째 `spaces` 변수는 숫자 유형입니다. 따라서 섀도잉은 `spaces_str` 및 `spaces_num`과 같은 다른 이름을 생각해내지 않아도 됩니다. 대신 더 간단한 `공백` 이름을 재사용할 수 있습니다. 그러나 여기에 표시된 것처럼 `mut`를 사용하려고 하면 컴파일 타임 오류가 발생합니다.

```rust
    let mut spaces = `   `;
    spaces = spaces.len();
```

오류는 변수의 유형을 변경할 수 없다고 말합니다.

```console
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
2 |     let mut spaces = `   `;
  |                      ----- expected due to this value
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `variables` due to previous error
```

이제 변수가 작동하는 방식을 살펴보았으므로 변수가 가질 수 있는 더 많은 데이터 유형을 살펴보겠습니다.
