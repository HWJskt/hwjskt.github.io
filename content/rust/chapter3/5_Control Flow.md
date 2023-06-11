+++
title = "Control Flow"
weight = 5
template ="book_page_rust.html"

+++



## 요약

- `if 키워드` 다음에 `조건(조건을 arm이라고 한다)` 다음에 `{중괄호를 열고 실행할 코드}` 를 작성

- 조건은 반드시 `bool` 타입 이어야 함

- let, if 를 이용해서, if 의 결과를 변수에 할당하려면, 모든 arm 의 결과값(반환값) 타입이 같아야

- 세 종류의 반복문(loops)
  - loop: 영원히 반복해서 실행하거나, 명시적으로 중지(break)하라고 지시할 때까지 실행
    - `continue`: 현재 반복 건너뛰고, 다음 반복으로 이동
    - `break` 표현식 뒤에 반환하려는 값을 추가 가능
    - `'레이블: loop {}` 형식으로 loop 에 이름(레이블)을 지정하여, break, continue 와 함께 사용가능
  - while: 조건이 `참`인 동안 반복문가 실행
  - for: while 보다 안전하고 빠르다

<!-- more -->

## [제어문](https://doc.rust-lang.org/book/ch03-05-control-flow.html#control-flow)

조건이 `참`인지 여부에 따라 일부 코드를 실행하고, 조건이 `참`인 동안 일부 코드를 반복적으로 실행하는 기능은 대부분의 프로그래밍 언어의 기본 구성 요소입니다. Rust 코드의 실행 흐름을 제어할 수 있는 가장 일반적인 구조는 `if` 표현식과 `반복문` 입니다.

### [if 표현식](https://doc.rust-lang.org/book/ch03-05-control-flow.html#if-expressions)

`if` 식을 사용하면 조건에 따라 코드를 분기할 수 있습니다. 조건을 제공한 다음 이렇게 쓸 수 있습니다. “이 조건이 충족되면 이 코드 블록을 실행하십시오. 조건이 충족되지 않으면 이 코드 블록을 실행하지 마십시오.”

`if` 식을 더 알아보기 위해 *projects* 디렉터리에 *branches* 라는 새 프로젝트를 만듭니다. *src/main.rs* 파일에 다음을 입력합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!(`condition was true`);
    } else {
        println!(`condition was false`);
    }
}
```

모든 `if` 표현식은 키워드 `if`로 시작하고 그 뒤에 조건이 옵니다. 이 경우 조건은 변수 `number`의 값이 5보다 작은지 여부를 확인합니다. 조건이 `참`이면 실행할 코드 블록을 조건 바로 뒤에 {중괄호로 된 블록} 안에 배치합니다.

<br>

`if` 표현식의 조건과 관련된 코드 블록은 2장의 [`Programming a Guessing Game`](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number) 섹션에서 논의한 `match` 표현식의 arm과 마찬가지로 *arm*이라고도 합니다.

선택적으로, 조건이 `false` 인 경우 실행할 대체 코드 블록을 `else` 표현식으로 포함할 수도 있습니다. `else` 표현식을 제공하지 않았는데 조건이 `false`인 경우, 프로그램은 `if` 블록을 건너뛰고 다음 코드로 이동합니다.

이 코드를 실행해 보십시오. 다음 출력이 표시되어야 합니다.

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
condition was true
```

<br>

`number` 값을 조건을 `거짓`으로 만드는 값으로 변경하면 어떤 일이 발생하는지 보겠습니다.

```rust
    let number = 7;
```

프로그램을 다시 실행하고 출력을 확인합니다.

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
condition was false
```

<br>

이 코드의 조건이 `bool` 이어야 한다는 점도 주목해야 합니다. 조건이 `bool`이 아니면 오류가 발생합니다. 예를 들어 다음 코드를 실행해 보십시오.

파일 이름: src/main.rs

```rust
fn main() {
    let number = 3;

    if number {
        println!(`number was three`);
    }
}
```

`if` 조건은 이번에 `3`의 값으로 평가되고 Rust는 오류를 발생시킵니다:

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if number {
  |        ^^^^^^ expected `bool`, found integer

For more information about this error, try `rustc --explain E0308`.
error: could not compile `branches` due to previous error
```

오류는 Rust가 `bool`을 예상했지만 정수를 얻었음을 나타냅니다.

<br>

Ruby 및 JavaScript와 같은 언어와 달리 Rust는 `bool` 이 아닌 유형을 `bool` 로 자동 변환하지 않습니다. 항상 `if` 의 조건을 `bool` 로 제공해야 합니다.

예를 들어 숫자가 `0`이 아닌 경우에만 `if` 코드 블록을 실행하려면 `if` 식을 다음과 같이 변경할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let number = 3;

    if number != 0 {
        println!(`number was something other than zero`);
    }
}
```

이 코드를 실행하면 `number was something other than zero`가 출력됩니다.

#### [else if 로 여러 조건 처리](https://doc.rust-lang.org/book/ch03-05-control-flow.html#handling-multiple-conditions-with-else-if)

`else if` 식에서 `if`와 `else`를 결합하여 여러 조건을 사용할 수 있습니다. 예를 들어:

파일 이름: src/main.rs

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!(`number is divisible by 4`);
    } else if number % 3 == 0 {
        println!(`number is divisible by 3`);
    } else if number % 2 == 0 {
        println!(`number is divisible by 2`);
    } else {
        println!(`number is not divisible by 4, 3, or 2`);
    }
}
```

이 프로그램에는 네 가지 가능한 경로가 있습니다. 실행하면 다음과 같은 출력이 표시됩니다.

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
number is divisible by 3
```

이 프로그램이 실행될 때 각 `if` 표현식을 차례로 확인하고, 조건이 `true`로 평가되는 첫 번째 본문을 실행합니다. 6은 2로 나눌 수 있지만 `number is divisible by 2`라는 출력이 표시되지 않으며 `else` 블록의 `number is not divisible by 4, 3, or 2` 텍스트도 표시되지 않습니다. 이는 Rust가 첫 번째 `참` 조건에 대해서만 블록을 실행하고, 일단 하나를 찾으면 나머지는 확인조차 하지 않기 때문입니다.

`else if` 식을 너무 많이 사용하면 코드가 복잡해질 수 있으므로 둘 이상의 식을 사용하는 경우 코드를 리팩터링해야 할 수 있습니다. 6장은 이러한 경우에 사용할 수 있는 `match`라고 하는 강력한 Rust 분기 구조를 설명합니다.

#### [let 문에서 if 사용](https://doc.rust-lang.org/book/ch03-05-control-flow.html#using-if-in-a-let-statement)

`if`는 (결과값을 반환하는) 표현식이기 때문에 목록 3-2에서와 같이 결과를 변수에 할당하기 위해 `let` 문의 오른쪽에 이를 사용할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!(`The value of number is: {number}`);
}
```

목록 3-2: `if` 표현식의 결과를 변수에 할당

`number` 변수는 `if` 식의 결과에 따라 값에 바인딩됩니다. 이 코드를 실행하여 무슨 일이 일어나는지 확인하십시오.

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/branches`
The value of number is: 5
```

코드 블록은 코드 블록의 마지막 표현식으로 평가되며, 숫자 자체도 표현식이라는 점을 기억하십시오. 이 경우 전체 `if` 표현식의 값은 실행되는 코드 블록에 따라 다릅니다. 이는 `if`의 각 부분에서 결과가 될 가능성이 있는 값이 동일한 유형이어야 함을 의미합니다. 목록 3-2에서 `if` 팔과 `else` arm 모두의 결과는 `i32` 정수였습니다.

<br>

다음 예와 같이 유형이 일치하지 않으면 오류가 발생합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let condition = true;

    let number = if condition { 5 } else { `six` };

    println!(`The value of number is: {number}`);
}
```

이 코드를 컴파일하려고 하면 오류가 발생합니다. `if` 및 `else` arm에는 호환되지 않는 값 유형이 있으며 Rust는 프로그램에서 문제를 찾을 위치를 정확히 나타냅니다.

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:4:44
  |
4 |     let number = if condition { 5 } else { `six` };
  |                                 -          ^^^^^ expected integer, found `&str`
  |                                 |
  |                                 expected because of this

For more information about this error, try `rustc --explain E0308`.
error: could not compile `branches` due to previous error
```

`if` 블록의 식은 정수로 평가되고 `else` 블록의 식은 문자열로 평가됩니다. 이것은 변수가 단일 유형을 가져야 하기 때문에 작동하지 않으며, Rust는 컴파일 타임에 `number` 변수가 어떤 유형인지 확실히 알아야 합니다. `number`의 유형을 알면 컴파일러가 `number`를 사용하는 모든 곳에서 유형이 유효한지 확인할 수 있습니다. Rust는 `number` 의 타입을 실행 할 때 정의되도록 할 수 없습니다. 컴파일러가 모든 변수의 다양한 타입을 추적해서 알아내야 한다면 컴파일러는 보다 복잡해지고 보증할 수 있는 것은 적어지게 됩니다.

### [반복문을 사용한 반복](https://doc.rust-lang.org/book/ch03-05-control-flow.html#repetition-with-loops)

코드 블록을 두 번 이상 실행하는 것이 종종 유용합니다. 이 작업을 위해 Rust는 여러 *반복문*을 제공합니다. 이 반복문은 반복문 본문 내부의 코드를 통해 끝까지 실행된 다음 처음부터 다시 시작됩니다. 반복문를 실험하기 위해 *loops* 라는 새 프로젝트를 만들어 봅시다.

<br>

Rust에는 `loop`, `while` 및 `for` 이라는 세 종류의 반복문(loops)가 있습니다. 각각 시도해 봅시다.

#### [loop 를 사용하여 코드 반복](https://doc.rust-lang.org/book/ch03-05-control-flow.html#repeating-code-with-loop)

`loop` 키워드는 Rust에게 코드 블록을 영원히 반복해서 실행하거나, 명시적으로 중지하라고 지시할 때까지 실행하라고 지시합니다.

예를 들어 *loops* 디렉토리의 *src/main.rs* 파일을 다음과 같이 변경하십시오.

파일 이름: src/main.rs

```rust
fn main() {
    loop {
        println!(`again!`);
    }
}
```

이 프로그램을 실행하면 프로그램을 수동으로 중지할 때까지 `again!` 를 계속 반복해서 출력합니다. 대부분의 터미널은 키보드 단축키 ctrl-c를 지원하여 연속 반복문에 빠진 프로그램을 중단합니다. 시도해 보세요:

```
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

`^C` 기호는 ctrl-c를 누른 위치를 나타냅니다. `again!`라는 단어가 표시될 수도 있고 표시되지 않을 수도 있습니다. 인터럽트 신호를 받았을 때 코드가 반복문에 있던 위치에 따라 `^C` 다음에 인쇄됩니다.

<br>

다행스럽게도 Rust는 코드를 사용하여 반복문에서 벗어날 수 있는 방법도 제공합니다. 반복문 내에 `break` 키워드를 배치하여 반복문 실행을 중지할 시기를 프로그램에 알릴 수 있습니다. 2장의 [올바른 추측 후 종료](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess) 섹션에서 사용자가 올바른 숫자를 추측하여 게임에서 이겼을 때 프로그램을 종료하기 위해 이 작업을 수행한 것을 기억하십시오.

우리는 또한 추측 게임에서 `continue`을 사용했는데, 이는 loop에서 이 반복의 나머지 코드를 건너뛰고 다음 반복으로 이동하도록 프로그램에 지시합니다.

#### [loop 에서 값 반환](https://doc.rust-lang.org/book/ch03-05-control-flow.html#returning-values-from-loops)

`loop`의 용도 중 하나는 스레드가 작업을 완료했는지 여부를 확인하는 것과 같이 `실패할 수 있는 작업을 다시 시도`하는 것입니다. 해당 작업의 결과를 반복문에서 나머지 코드로 전달해야 할 수도 있습니다. 이를 위해 반복문를 중지하는 데 사용하는 `break` 표현식 뒤에 반환하려는 값을 추가할 수 있습니다. 해당 값은 다음과 같이 사용할 수 있도록 반복문 외부로 반환됩니다.

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!(`The result is {result}`);
}
```

반복문 전에 `counter`라는 변수를 선언하고 `0`으로 초기화합니다. 그런 다음 반복문에서 반환된 값을 보유하기 위해 `result`라는 변수를 선언합니다. 반복문가 반복될 때마다 `카운터` 변수에 `1`을 추가한 다음 `카운터`가 `10`과 같은지 확인합니다. 그럴 때 `counter * 2` 값과 함께 `break` 키워드를 사용합니다. 반복문 다음에 세미콜론을 사용하여 `result`에 값을 할당하는 명령문을 종료합니다. 마지막으로 `result`에 값을 인쇄합니다. 이 경우에는 `20`입니다.

#### [여러 loop 가 있을 때, loop 를 명확히 지정하기 위한 "반복문 레이블"](https://doc.rust-lang.org/book/ch03-05-control-flow.html#loop-labels-to-disambiguate-between-multiple-loops)

반복문 내에 반복문이 있는 경우 `break` 및 `continue`은 해당 지점에서 가장 안쪽 반복문에 적용됩니다.

선택적으로 반복문에 *반복문 레이블*을 지정할 수 있는데, `break` 또는 `continue` 뒤에 `레이블이 지정된 반복문`을 지정하면, 해당 키워드가 가장 안쪽 반복문 대신 `레이블이 지정된 반복문`에 적용되도록 할 수 있습니다. 반복문 레이블은 작은따옴표(')로 시작해야 합니다. 다음은 두 개의 중첩 반복문가 있는 예입니다.

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!(`count = {count}`);
        let mut remaining = 10;

        loop {
            println!(`remaining = {remaining}`);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!(`End count = {count}`);
}
```

외부 반복문에는 `'counting_up` 레이블이 있으며 0에서 2까지 카운트합니다. 레이블이 없는 내부 반복문는 10에서 9까지 카운트다운합니다. 레이블을 지정하지 않은 첫 번째 `break`은 내부 반복문를 종료합니다. `break 'counting_up;` 문은 외부 반복문만 종료합니다. 이 코드는 다음을 인쇄합니다.

```
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.58s
     Running `target/debug/loops`
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

#### [while 이 있는 조건부 반복문](https://doc.rust-lang.org/book/ch03-05-control-flow.html#conditional-loops-with-while)

프로그램은 종종 반복문 내에서 조건을 평가해야 합니다. 조건이 `참`인 동안 반복문가 실행됩니다. 조건이 `참`이 아니게 되면 프로그램은 `break`을 호출하여 반복문를 중지합니다.

`loop`, `if`, `else` 및 `break`의 조합을 사용하여 이와 같은 동작을 구현할 수 있습니다. 원하는 경우 지금 프로그램에서 시도해 볼 수 있습니다. 그러나 이 패턴은 너무 보편적이어서 Rust에는 `while` 반복문라고 하는 내장 언어 구조가 있습니다.

Listing 3-3에서 우리는 `while`을 사용하여 프로그램을 세 번 반복하고 매번 카운트다운한 다음 반복문가 끝난 후 메시지를 인쇄하고 종료합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!(`{number}!`);

        number -= 1;
    }

    println!(`LIFTOFF!!!`);
}
```

목록 3-3: `while` 반복문를 사용하여 조건이 참일 때 코드 실행

이 구성은 `loop`, `if`, `else` 및 `break`를 사용하는 경우 필요한, 많은 중첩을 제거하고 더 명확합니다. 조건이 `true`로 평가되는 동안 코드가 실행됩니다. 그렇지 않으면 반복문를 종료합니다.

#### [for 를 사용하여 컬렉션 반복](https://doc.rust-lang.org/book/ch03-05-control-flow.html#looping-through-a-collection-with-for)

`while` 구성을 사용하여 배열과 같은 컬렉션의 요소를 반복하도록 선택할 수 있습니다. 예를 들어 목록 3-4의 반복문는 배열 `a`의 각 요소를 인쇄합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!(`the value is: {}`, a[index]);

        index += 1;
    }
}
```

Listing 3-4: `while` 반복문를 사용하여 컬렉션의 각 요소를 순환

여기서 코드는 배열의 요소를 통해 계산됩니다. 인덱스 `0`에서 시작한 다음 배열의 마지막 인덱스에 도달할 때까지(즉, `index < 5`가 더 이상 `true`가 아닌 경우) 반복합니다. 이 코드를 실행하면 배열의 모든 요소가 인쇄됩니다.

```
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32s
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

5개의 배열 값이 모두 예상대로 터미널에 나타납니다. `index`가 어느 시점에서 `5` 값에 도달하더라도 반복문는 배열에서 6번째 값을 가져오려고 시도하기 전에 실행을 중지합니다.

<br>

그러나 이 방법은 오류가 발생하기 쉽습니다. 인덱스 값이나 테스트 조건이 올바르지 않으면 프로그램이 패닉 상태가 될 수 있습니다. 예를 들어 `a` 배열의 정의를 4개의 요소를 갖도록 변경했지만 조건을 `while index < 4`로 업데이트하는 것을 잊은 경우 코드가 패닉 상태가 됩니다.

또한 컴파일러가 반복문를 통한 모든 반복에서 인덱스가 배열 범위 내에 있는지 여부에 대한 조건부 검사를 수행하기 위해 런타임 코드를 추가하기 때문에 속도가 느립니다.

<br>

보다 간결한 대안으로 `for` 반복문를 사용하고 컬렉션의 각 항목에 대해 일부 코드를 실행할 수 있습니다. `for` 반복문는 Listing 3-5의 코드처럼 보입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!(`the value is: {element}`);
    }
}
```

Listing 3-5: `for` 반복문를 사용하여 컬렉션의 각 요소를 순환

이 코드를 실행하면 목록 3-4와 동일한 출력이 표시됩니다. 더 중요한 것은 이제 코드의 안전성을 높이고 배열의 끝을 벗어나거나 충분히 멀리 가지 않고 일부 항목이 누락되어 발생할 수 있는 버그의 가능성을 제거했습니다.

`for` 반복문를 사용하면 Listing 3-4에서 사용된 방법과 같이 배열의 값 수를 변경한 경우 다른 코드를 변경할 필요가 없습니다.

`for` 반복문의 안전성과 간결함은 Rust에서 가장 일반적으로 사용되는 반복문 구조입니다. Listing 3-3에서 `while` 반복문를 사용한 카운트다운 예제와 같이 일부 코드를 특정 횟수만큼 실행하려는 상황에서도 대부분의 Rustacean은 `for` 반복문를 사용합니다. 이를 수행하는 방법은 표준 라이브러리에서 제공하는 `Range`를 사용하는 것입니다. 이 범위는 한 숫자에서 시작하여 다른 숫자 앞에서 끝나는 모든 숫자를 순서대로 생성합니다.

카운트다운은 `for` 반복문와 아직 언급하지 않은 `rev` 방법을 사용하여 범위를 역전시키는 것과 같습니다.

파일 이름: src/main.rs

```rust
fn main() {
    for number in (1..4).rev() {
        println!(`{number}!`);
    }
    println!(`LIFTOFF!!!`);
}
```

이 코드가 좀 더 멋지죠?

## [요약](https://doc.rust-lang.org/book/ch03-05-control-flow.html#summary)

해냈어요! 변수, 스칼라 및 복합 데이터 유형, 함수, 주석, if 표현식 및 반복문에 대해 배웠습니다! 이 장에서 설명한 개념을 연습하려면 다음을 수행하는 프로그램을 작성해 보십시오.

- 화씨와 섭씨 사이의 온도를 변환합니다.
- *n* 번째 피보나치 수를 생성합니다 .
- 노래의 반복을 활용하여 크리스마스 캐롤 `The Twelve Days of Christmas`의 가사를 인쇄하세요.

계속 진행할 준비가 되면 다른 프로그래밍 언어에는 일반적으로 존재하지 않는 Rust의 개념인 소유권에 대해 이야기하겠습니다.
