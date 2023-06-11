+++
title = "18-21 translation"
weight = 1
template ="book_page_rust.html"

+++



# 18-21

# 18

# [패턴과 매칭](https://doc.rust-lang.org/book/ch18-00-patterns.html#patterns-and-matching)

*패턴은* 복잡하고 단순한 유형의 구조와 일치시키기 위한 Rust의 특수 구문입니다. `일치` 식 및 기타 구문과 함께 패턴을 사용하면 프로그램의 제어 흐름을 더 잘 제어할 수 있습니다. 패턴은 다음의 일부 조합으로 구성됩니다.

- 리터럴
- 해체된 배열, 열거형, 구조체 또는 튜플
- 변수
- 와일드카드
- 자리 표시자

일부 예제 패턴에는 `x`, `(a, 3)` 및 `Some(Color::Red)`가 포함됩니다. 패턴이 유효한 컨텍스트에서 이러한 구성 요소는 데이터의 모양을 설명합니다. 그런 다음 우리 프로그램은 패턴과 값을 일치시켜 특정 코드 조각을 계속 실행하는 데 올바른 데이터 모양이 있는지 확인합니다.

패턴을 사용하기 위해 우리는 그것을 어떤 값과 비교합니다. 패턴이 값과 일치하면 코드에서 값 부분을 사용합니다. 동전 분류기 예제와 같이 패턴을 사용했던 6장의 `일치` 표현을 상기하십시오. 값이 패턴의 모양에 맞으면 명명된 조각을 사용할 수 있습니다. 그렇지 않으면 패턴과 연결된 코드가 실행되지 않습니다.

이 장은 패턴과 관련된 모든 것에 대한 참조입니다. 패턴을 사용할 수 있는 유효한 위치, 반박할 수 있는 패턴과 반박할 수 없는 패턴의 차이점, 볼 수 있는 다양한 종류의 패턴 구문을 다룰 것입니다. 이 장을 마치면 패턴을 사용하여 많은 개념을 명확한 방식으로 표현하는 방법을 알게 됩니다.

------

## [모든 장소 패턴을 사용할 수 있습니다.](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#all-the-places-patterns-can-be-used)

패턴은 Rust의 여러 곳에서 나타나며, 여러분은 그것을 깨닫지 못한 채 패턴을 많이 사용하고 있습니다! 이 섹션에서는 패턴이 유효한 모든 위치에 대해 설명합니다.

### [`일치` 팔](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#match-arms)

6장에서 논의한 것처럼 `일치` 표현식의 팔에 패턴을 사용합니다. 공식적으로 `일치` 표현식은 키워드 `일치`, 일치할 값, 패턴으로 구성된 하나 이상의 일치 암과 값이 해당 암의 패턴과 일치하는 경우 실행할 표현식으로 정의됩니다.

```
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

예를 들어, 다음은 `옵션`에서 일치하는 목록 6-5의 `일치` 표현식입니다.` 변수 `x`의 값:

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

이 `일치` 식의 패턴은 각 화살표 왼쪽에 있는 `없음` 및 `Some(i)`입니다.

`일치` 식에 대한 한 가지 요구 사항 은 `일치` 식의 값에 대한 모든 가능성을 고려해야 한다는 점에서 *철저* 해야 한다는 것입니다. 모든 가능성을 다루었는지 확인하는 한 가지 방법은 마지막 팔에 대한 포괄 패턴을 갖는 것입니다. 예를 들어 어떤 값과도 일치하는 변수 이름은 절대 실패할 수 없으므로 나머지 모든 경우를 포함합니다.

특정 패턴 ` *`은 무엇이든 일치하지만 변수에 바인딩되지 않으므로 마지막 일치 부분에서 자주 사용됩니다. `* ` 패턴은 예를 들어 지정되지 않은 값을 무시하려는 경우에 유용할 수 있습니다. [`_` 패턴에 대해서는 이 장 뒷부분의 `패턴의 값 무시`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern) 섹션 에서 자세히 다룰 것입니다.

### [조건부 `if let` 표현식](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#conditional-if-let-expressions)

6장에서 우리는 주로 하나의 경우에만 일치하는 `일치`에 해당하는 것을 작성하는 더 짧은 방법으로 `if let` 표현식을 사용하는 방법에 대해 논의했습니다. 선택적으로 `if let`은 `if let`의 패턴이 일치하지 않는 경우 실행할 코드를 포함하는 해당 `else`를 가질 수 있습니다.

목록 18-1은 `if let`, `else if` 및 `else if let` 표현식을 혼합하고 일치시키는 것도 가능함을 보여줍니다. 이렇게 하면 패턴과 비교할 하나의 값만 표현할 수 있는 `일치` 식보다 더 많은 유연성을 얻을 수 있습니다. 또한 Rust는 일련의 `if let`, `else if`, `else if let` 조건이 서로 관련될 것을 요구하지 않습니다.

목록 18-1의 코드는 여러 조건에 대한 일련의 검사를 기반으로 배경을 만들 색상을 결정합니다. 이 예에서는 실제 프로그램이 사용자 입력에서 수신할 수 있는 하드코딩된 값으로 변수를 만들었습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = `34`.parse();

    if let Some(color) = favorite_color {
        println!(`Using your favorite color, {color}, as the background`);
    } else if is_tuesday {
        println!(`Tuesday is green day!`);
    } else if let Ok(age) = age {
        if age > 30 {
            println!(`Using purple as the background color`);
        } else {
            println!(`Using orange as the background color`);
        }
    } else {
        println!(`Using blue as the background color`);
    }
}
```

목록 18-1: `if let`, `else if`, `else if let` 및 `else` 혼합

사용자가 좋아하는 색상을 지정하면 해당 색상이 배경으로 사용됩니다. 즐겨찾기 색상을 지정하지 않고 오늘이 화요일인 경우 배경색은 녹색입니다. 그렇지 않고 사용자가 나이를 문자열로 지정하고 이를 숫자로 성공적으로 구문 분석할 수 있으면 숫자 값에 따라 색상이 보라색 또는 주황색이 됩니다. 이러한 조건이 적용되지 않는 경우 배경색은 파란색입니다.

이 조건부 구조를 통해 복잡한 요구 사항을 지원할 수 있습니다. 여기에 있는 하드코딩된 값을 사용하여 이 예제는 `배경색으로 보라색 사용`을 인쇄합니다.

`if let`도 `match` 팔이 할 수 있는 것과 같은 방식으로 숨겨진 변수를 도입할 수 있음을 볼 수 있습니다. `if let Ok(age) = age` 줄은 `확인` 변형. 즉, 해당 블록 내에 `if age > 30` 조건을 배치해야 합니다. 이 두 조건을 `if let Ok(age) = age && age > 30`으로 결합할 수 없습니다. 새 범위가 중괄호로 시작될 때까지 30과 비교하려는 숨겨진 `나이`는 유효하지 않습니다.

`if let` 식 사용의 단점은 컴파일러가 철저함을 확인하지 않는 반면 `match` 식에서는 확인한다는 것입니다. 마지막 `else` 블록을 생략하여 일부 사례 처리를 놓친 경우 컴파일러는 가능한 논리 버그에 대해 경고하지 않습니다.

### [`while let` 조건 루프](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#while-let-conditional-loops)

`if let` 구조와 유사하게 `while let` 조건 루프를 사용하면 패턴이 계속 일치하는 동안 `while` 루프가 실행될 수 있습니다. 목록 18-2에서 우리는 벡터를 스택으로 사용하고 푸시된 반대 순서로 벡터의 값을 인쇄하는 `while let` 루프를 코딩합니다.

```rust
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    while let Some(top) = stack.pop() {
        println!(`{}`, top);
    }
```

목록 18-2: `stack.pop()`이 `Some`을 반환하는 한 값을 인쇄하기 위해 `while let` 루프 사용

이 예제는 3, 2, 1을 인쇄합니다. `pop` 메서드는 벡터에서 마지막 요소를 가져와 `Some(value)`를 반환합니다. 벡터가 비어 있으면 `pop`은 `없음`을 반환합니다. `while` 루프는 `pop`이 `Some`을 반환하는 한 해당 블록에서 코드를 계속 실행합니다. `pop`이 `None`을 반환하면 루프가 중지됩니다. `while let`을 사용하여 스택에서 모든 요소를 꺼낼 수 있습니다.

### [`for` 루프](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#for-loops)

`for` 루프에서 키워드 `for` 바로 뒤에 오는 값은 패턴입니다. 예를 들어 `for x in y`에서 `x`는 패턴입니다. Listing 18-3은 `for` 루프에서 패턴을 사용하여 `for` 루프의 일부로 튜플을 분해하거나 분해하는 방법을 보여줍니다.

```rust
    let v = vec![`a`, `b`, `c`];

    for (index, value) in v.iter().enumerate() {
        println!(`{} is at index {}`, value, index);
    }
```

Listing 18-3: `for` 루프의 패턴을 사용하여 튜플을 분해합니다.

목록 18-3의 코드는 다음을 인쇄합니다:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/patterns`
a is at index 0
b is at index 1
c is at index 2
```

우리는 `열거` 방법을 사용하여 반복자를 조정하여 튜플에 배치된 값과 해당 값에 대한 인덱스를 생성합니다. 생성된 첫 번째 값은 튜플 `(0, `a`)`입니다. 이 값이 `(인덱스, 값)` 패턴과 일치하면 `인덱스`는 `0`이 되고 `값`은 ``a``가 되어 출력의 첫 번째 줄을 인쇄합니다.

### [`하자` 문](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#let-statements)

이 장 이전에는 `match` 및 `if let`과 함께 패턴 사용에 대해 명시적으로만 논의했지만 실제로는 `let` 문을 포함하여 다른 위치에서도 패턴을 사용했습니다. 예를 들어 `let`을 사용한 다음과 같은 간단한 변수 할당을 고려하십시오.

```rust
let x = 5;
```

이와 같은 `let` 문을 사용할 때마다 패턴을 사용하고 있는 것입니다. 비록 깨닫지 못했을 수도 있지만요! 보다 공식적으로 `let` 문은 다음과 같습니다.

```
let PATTERN = EXPRESSION;
```

`let x = 5;`와 같은 문장에서 `PATTERN` 슬롯에 변수 이름이 있는 경우 변수 이름은 특히 단순한 패턴 형식입니다. Rust는 표현식을 패턴과 비교하고 찾은 이름을 할당합니다. 그래서 `let x = 5;` 예를 들어, `x`는 `여기에서 일치하는 것을 변수 `x`에 바인딩한다`는 의미의 패턴입니다. `x`라는 이름이 전체 패턴이기 때문에 이 패턴은 사실상 `값이 무엇이든 모든 것을 변수 `x`에 바인딩합니다.`를 의미합니다.

`let`의 패턴 일치 측면을 더 명확하게 보려면 `let`과 함께 패턴을 사용하여 튜플을 분해하는 목록 18-4를 고려하십시오.

```rust
    let (x, y, z) = (1, 2, 3);
```

목록 18-4: 패턴을 사용하여 튜플을 분해하고 세 개의 변수를 한 번에 생성

여기에서 튜플을 패턴과 일치시킵니다. Rust는 값 `(1, 2, 3)`을 패턴 `(x, y, z)`와 비교하고 값이 패턴과 일치하는지 확인하므로 Rust는 `1`을 `x`에, `2`를 `에 바인딩합니다. y` 및 `3`에서 `z`까지. 이 튜플 패턴은 내부에 세 개의 개별 변수 패턴을 중첩하는 것으로 생각할 수 있습니다.

패턴의 요소 수가 튜플의 요소 수와 일치하지 않으면 전체 유형이 일치하지 않고 컴파일러 오류가 발생합니다. 예를 들어, Listing 18-5는 3개의 요소를 가진 튜플을 2개의 변수로 분해하려는 시도를 보여줍니다. 이것은 작동하지 않을 것입니다.

```rust
    let (x, y) = (1, 2, 3);
```

목록 18-5: 변수가 튜플의 요소 수와 일치하지 않는 패턴을 잘못 생성

이 코드를 컴파일하려고 하면 다음 유형 오류가 발생합니다.

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `patterns` due to previous error
```

[오류를 수정하기 위해 `_` 또는 `..`를 사용하여 튜플의 값 중 하나 이상을 무시할 수 있습니다. `패턴의 값 무시`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern) 섹션에서 볼 수 있습니다. 문제가 패턴에 너무 많은 변수가 있다는 것이라면 솔루션은 변수를 제거하여 유형을 일치시켜 변수의 수가 튜플의 요소 수와 같도록 하는 것입니다.

### [함수 매개변수](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#function-parameters)

함수 매개변수는 패턴일 수도 있습니다. `i32` 유형의 `x`라는 이름의 매개변수 하나를 받는 `foo`라는 이름의 함수를 선언하는 Listing 18-6의 코드는 이제 익숙해 보일 것입니다.

```rust
fn foo(x: i32) {
    // code goes here
}
```

목록 18-6: 함수 시그니처는 매개변수의 패턴을 사용합니다.

`x` 부분은 패턴입니다! `let`에서 했던 것처럼 함수의 인수에 있는 튜플을 패턴에 일치시킬 수 있습니다. 목록 18-7은 튜플의 값을 함수에 전달할 때 값을 분할합니다.

파일 이름: src/main.rs

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!(`Current location: ({}, {})`, x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

목록 18-7: 튜플을 분해하는 매개변수가 있는 함수

이 코드는 `현재 위치: (3, 5)`를 인쇄합니다. `&(3, 5)` 값은 `&(x, y)` 패턴과 일치하므로 `x`는 값 `3`이고 `y`는 값 `5`입니다.

13장에서 논의한 바와 같이 클로저는 함수와 유사하기 때문에 함수 매개변수 목록에서와 같은 방식으로 클로저 매개변수 목록에서 패턴을 사용할 수 있습니다.

지금까지 패턴을 사용하는 여러 가지 방법을 보았지만 패턴은 사용할 수 있는 모든 위치에서 동일하게 작동하지 않습니다. 어떤 곳에서는 패턴이 반박할 수 없어야 합니다. 다른 상황에서는 반박할 수 있습니다. 이 두 가지 개념에 대해서는 다음에 설명하겠습니다.

------

## [반박 가능성: 패턴이 일치하지 않을 수 있는지 여부](https://doc.rust-lang.org/book/ch18-02-refutability.html#refutability-whether-a-pattern-might-fail-to-match)

패턴은 반박 가능과 반박 불가의 두 가지 형태로 나타납니다. 전달된 모든 가능한 값과 일치하는 패턴은 *반박할 수 없습니다* . 예를 들어 `let x = 5;` 문에서 `x`가 있습니다. `x`는 어떤 것과도 일치하므로 일치하지 않을 수 없기 때문입니다. 일부 가능한 값과 일치하지 않을 수 있는 패턴은 *반박* 할 수 있습니다. `a_value` 변수의 값이 `Some`이 아니라 `None`인 경우 `Some(x)` 패턴이 일치하지 않습니다.

함수 매개변수, `let` 문 및 `for` 루프는 값이 일치하지 않으면 프로그램이 의미 있는 작업을 수행할 수 없기 때문에 반박할 수 없는 패턴만 허용할 수 있습니다. `if let` 및 `while let` 식은 반박할 수 있는 패턴과 반박할 수 없는 패턴을 허용하지만, 컴파일러는 반박할 수 없는 패턴에 대해 경고합니다. 정의에 따라 이러한 패턴은 가능한 실패를 처리하도록 의도되었기 때문입니다. 성공 또는 실패.

일반적으로 반박 가능한 패턴과 반박 불가능한 패턴의 구분에 대해 걱정할 필요가 없습니다. 그러나 오류 메시지에 표시될 때 대응할 수 있도록 반박 가능성의 개념에 익숙해야 합니다. 이러한 경우 코드의 의도된 동작에 따라 패턴 또는 패턴을 사용하는 구조를 변경해야 합니다.

Rust가 반박할 수 없는 패턴을 요구하고 그 반대의 경우도 있을 때 우리가 반박할 수 있는 패턴을 사용하려고 할 때 어떤 일이 일어나는지 예를 살펴보겠습니다. 목록 18-8은 `let` 문을 보여주지만 패턴에 대해 반박 가능한 패턴인 `Some(x)`를 지정했습니다. 예상하셨겠지만 이 코드는 컴파일되지 않습니다.

```rust
    let Some(x) = some_option_value;
```

Listing 18-8: `let`과 함께 반박 가능한 패턴 사용 시도

`some_option_value`가 `None` 값인 경우 `Some(x)` 패턴과 일치하지 않으므로 패턴이 반박 가능함을 의미합니다. 그러나 코드가 `None` 값으로 수행할 수 있는 유효한 작업이 없기 때문에 `let` 문은 반박할 수 없는 패턴만 허용할 수 있습니다. 컴파일 시간에 Rust는 우리가 반박할 수 없는 패턴이 필요한 곳에 반박할 수 있는 패턴을 사용하려고 시도했다고 불평할 것입니다:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding: `None` not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
  |
  = note: `let` bindings require an `irrefutable pattern`, like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
note: `Option<i32>` defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note: 
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type `Option<i32>`
help: you might want to use `if let` to ignore the variant that isn`t matched
  |
3 |     let x = if let Some(x) = some_option_value { x } else { todo!() };
  |     ++++++++++                                 ++++++++++++++++++++++
help: alternatively, you might want to use let else to handle the variant that isn`t matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` due to previous error
```

`Some(x)` 패턴으로 모든 유효한 값을 다루지 않았기 때문에 (그리고 덮을 수도 없었습니다!) Rust는 당연히 컴파일러 오류를 생성합니다.

반박할 수 없는 패턴이 필요한 경우 패턴을 사용하는 코드를 변경하여 수정할 수 있습니다. `let`을 사용하는 대신 `if let`을 사용할 수 있습니다. 그런 다음 패턴이 일치하지 않으면 코드는 중괄호 안의 코드를 건너뛰어 유효하게 계속할 수 있는 방법을 제공합니다. 목록 18-9는 목록 18-8의 코드를 수정하는 방법을 보여줍니다.

```rust
    if let Some(x) = some_option_value {
        println!(`{}`, x);
    }
```

Listing 18-9: `let` 대신 `if let`과 반박 가능한 패턴이 있는 블록 사용

우리는 코드를 출력했습니다! 이 코드는 오류를 수신하지 않고 반박할 수 없는 패턴을 사용할 수 없음을 의미하지만 완벽하게 유효합니다. Listing 18-10과 같이 `x`와 같이 항상 일치하는 패턴을 `if let`에 지정하면 컴파일러에서 경고를 표시합니다.

```rust
    if let x = 5 {
        println!(`{}`, x);
    };
```

Listing 18-10: `if let`으로 반박할 수 없는 패턴 사용 시도

Rust는 반박할 수 없는 패턴으로 `if let`을 사용하는 것이 이치에 맞지 않는다고 불평합니다:

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `if let` pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^
  |
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`
  = note: `#[warn(irrefutable_let_patterns)]` on by default

warning: `patterns` (bin `patterns`) generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
5
```

이러한 이유로 매치 암은 반박할 수 없는 패턴을 가진 나머지 값과 일치해야 하는 마지막 암을 제외하고 반박할 수 있는 패턴을 사용해야 합니다. Rust를 사용하면 하나의 팔만 있는 `일치`에서 반박할 수 없는 패턴을 사용할 수 있지만 이 구문은 특별히 유용하지 않으며 더 간단한 `let` 문으로 대체될 수 있습니다.

이제 패턴을 사용하는 위치와 반박 가능한 패턴과 반박 불가능한 패턴의 차이점을 알았으므로 패턴을 만드는 데 사용할 수 있는 모든 구문을 살펴보겠습니다.

------

## [패턴 구문](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#pattern-syntax)

이 섹션에서는 패턴에 유효한 모든 구문을 수집하고 각 구문을 사용해야 하는 이유와 시기에 대해 논의합니다.

### [일치하는 리터럴](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-literals)

6장에서 보았듯이 패턴을 리터럴과 직접 일치시킬 수 있습니다. 다음 코드는 몇 가지 예를 제공합니다.

```rust
    let x = 1;

    match x {
        1 => println!(`one`),
        2 => println!(`two`),
        3 => println!(`three`),
        _ => println!(`anything`),
    }
```

이 코드는 `x`의 값이 1이기 때문에 `one`을 인쇄합니다. 이 구문은 특정 구체적인 값을 가져오는 경우 코드에서 작업을 수행하려는 경우에 유용합니다.

### [명명된 변수 일치](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-named-variables)

명명된 변수는 모든 값과 일치하는 반박할 수 없는 패턴이며 책에서 여러 번 사용했습니다. 그러나 `일치` 식에서 명명된 변수를 사용하면 복잡해집니다. `일치`는 새로운 범위를 시작하기 때문에 `일치` 식 내부에서 패턴의 일부로 선언된 변수는 모든 변수의 경우와 마찬가지로 `일치` 구문 외부에서 동일한 이름을 가진 변수를 가리게 됩니다. 목록 18-11에서 `Some(5)` 값을 가진 `x`라는 이름의 변수와 `10` 값을 가진 변수 `y`를 선언합니다. 그런 다음 값 `x`에 대해 `일치` 식을 만듭니다. 성냥 팔의 패턴을 보고 `println!` 마지막에 이 코드를 실행하거나 더 읽기 전에 코드가 무엇을 인쇄할지 알아내십시오.

파일 이름: src/main.rs

```rust
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!(`Got 50`),
        Some(y) => println!(`Matched, y = {y}`),
        _ => println!(`Default case, x = {:?}`, x),
    }
    
    println!(`at the end: x = {:?}, y = {y}`, x);
```

목록 18-11: 그림자 변수 `y`를 도입하는 팔이 있는 `일치` 표현식

`일치` 표현식이 실행될 때 어떤 일이 발생하는지 살펴보겠습니다. 첫 번째 일치 부분의 패턴이 정의된 `x` 값과 일치하지 않으므로 코드가 계속됩니다.

두 번째 일치 부분의 패턴은 `Some` 값 내의 모든 값과 일치하는 `y`라는 새 변수를 도입합니다. 우리는 `일치` 식 내부의 새로운 범위에 있기 때문에 이것은 새로운 `y` 변수이며 처음에 값 10으로 선언한 `y`가 아닙니다. 이 새로운 `y` 바인딩은 내부의 모든 값과 일치합니다. `X`에 있는 `Some`입니다. 따라서 이 새로운 `y`는 `x`의 `Some`의 내부 값에 바인딩됩니다. 해당 값은 `5`이므로 해당 암에 대한 표현식이 실행되고 `Matched, y = 5`가 인쇄됩니다.

`x`가 `Some(5)`가 아니라 `없음` 값이었다면 처음 두 팔의 패턴이 일치하지 않았기 때문에 값이 밑줄과 일치했을 것입니다. 밑줄 암의 패턴에 `x` 변수를 도입하지 않았으므로 표현식의 `x`는 여전히 음영 처리되지 않은 외부 `x`입니다. 이 가상의 경우 `일치`는 `기본 사례, x = 없음`을 인쇄합니다.

`일치` 표현식이 완료되면 해당 범위가 종료되고 내부 `y`의 범위도 종료됩니다. 마지막 `println!` `끝에: x = Some(5), y = 10`을 생성합니다.

숨겨진 변수를 도입하는 대신 외부 `x` 및 `y` 값을 비교하는 `일치` 식을 만들려면 대신 일치 가드 조건을 사용해야 합니다. [매치 가드에 대해서는 나중에 `매치 가드가 있는 추가 조건`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#extra-conditionals-with-match-guards) 섹션 에서 이야기하겠습니다.

### [여러 패턴](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#multiple-patterns)

`일치` 식에서 `|` 기호를 사용하여 여러 패턴을 일치시킬 수 있습니다. 패턴 *또는* 연산자인 구문입니다. 예를 들어, 다음 코드에서 `x` 값을 일치 항목과 일치시킵니다. 첫 번째 항목에는 or *옵션* 이 있습니다. 즉, `x` 값이 해당 항목의 값 중 하나와 일치하면 해당 항목의 코드는 달리다:

```rust
    let x = 1;

    match x {
        1 | 2 => println!(`one or two`),
        3 => println!(`three`),
        _ => println!(`anything`),
    }
```

이 코드는 `one or two`를 출력합니다.

### [`..=`를 사용하여 일치하는 값 범위](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-ranges-of-values-with-)

`..=` 구문을 사용하면 포괄적인 범위의 값과 일치시킬 수 있습니다. 다음 코드에서 패턴이 주어진 범위 내의 값과 일치하면 해당 암이 실행됩니다.

```rust
    let x = 5;

    match x {
        1..=5 => println!(`one through five`),
        _ => println!(`something else`),
    }
```

`x`가 1, 2, 3, 4 또는 5이면 첫 번째 팔이 일치합니다. 이 구문은 `|`를 사용하는 것보다 여러 일치 값에 더 편리합니다. 같은 생각을 표현하는 연산자; `|`를 사용한다면 `1 | 2 | 3 | 4 | 5`를 지정해야 합니다. 범위를 지정하는 것은 훨씬 짧습니다. 특히 1에서 1,000 사이의 숫자를 일치시키려는 경우 더욱 그렇습니다!

컴파일러는 컴파일 시간에 범위가 비어 있지 않은지 확인하고 Rust가 범위가 비어 있는지 여부를 알 수 있는 유일한 유형이 `char` 및 숫자 값이기 때문에 범위는 숫자 또는 `char` 값에만 허용됩니다. .

다음은 `char` 값의 범위를 사용하는 예입니다.

```rust
    let x = `c`;

    match x {
        `a`..=`j` => println!(`early ASCII letter`),
        `k`..=`z` => println!(`late ASCII letter`),
        _ => println!(`something else`),
    }
```

Rust는 ``c``가 첫 번째 패턴의 범위 내에 있고 `초기 ASCII 문자`를 인쇄한다고 말할 수 있습니다.

### [가치를 분리하기 위한 파괴](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-to-break-apart-values)

또한 패턴을 사용하여 구조체, 열거형 및 튜플을 분해하여 이러한 값의 다른 부분을 사용할 수 있습니다. 각 값을 살펴보겠습니다.

#### [구조 파괴](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs)

목록 18-12는 `let` 문이 있는 패턴을 사용하여 분리할 수 있는 `x`와 `y`의 두 필드가 있는 `Point` 구조체를 보여줍니다.

파일 이름: src/main.rs

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

목록 18-12: 구조체의 필드를 별도의 변수로 분해

이 코드는 `p` 구조체의 `x` 및 `y` 필드 값과 일치하는 변수 `a` 및 `b`를 만듭니다. 이 예는 패턴의 변수 이름이 구조체의 필드 이름과 일치할 필요가 없음을 보여줍니다. 그러나 어떤 변수가 어떤 필드에서 왔는지 쉽게 기억할 수 있도록 변수 이름을 필드 이름과 일치시키는 것이 일반적입니다. 이러한 일반적인 사용법과 `let Point { x: x, y: y } = p;`라고 쓰기 때문입니다. 많은 중복이 포함되어 있고 Rust에는 구조체 필드와 일치하는 패턴에 대한 약칭이 있습니다. 구조체 필드의 이름만 나열하면 되며 패턴에서 생성된 변수는 동일한 이름을 갖게 됩니다. Listing 18-13은 Listing 18-12의 코드와 동일한 방식으로 동작하지만 `let` 패턴에서 생성된 변수는 `

파일 이름: src/main.rs

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

Listing 18-13: struct field 속기를 사용하여 구조체 필드 분해

이 코드는 `p` 변수의 `x` 및 `y` 필드와 일치하는 변수 `x` 및 `y`를 생성합니다. 결과는 변수 `x` 및 `y`가 `p` 구조체의 값을 포함한다는 것입니다.

모든 필드에 대한 변수를 생성하는 대신 구조체 패턴의 일부로 리터럴 값을 사용하여 구조를 분해할 수도 있습니다. 이렇게 하면 특정 값에 대한 일부 필드를 테스트하는 동시에 다른 필드를 해체하는 변수를 생성할 수 있습니다.

Listing 18-14에는 `Point` 값을 세 가지 경우로 구분하는 `일치` 표현식이 있습니다. `x` 축(`y = 0`일 때 참)에 바로 있는 점, `y` 축에 있는 점 축(`x = 0`)이거나 둘 다 아닙니다.

파일 이름: src/main.rs

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!(`On the x axis at {x}`),
        Point { x: 0, y } => println!(`On the y axis at {y}`),
        Point { x, y } => {
            println!(`On neither axis: ({x}, {y})`);
        }
    }
}
```

목록 18-14: 하나의 패턴에서 리터럴 값을 분해하고 일치시키기

첫 번째 팔은 값이 리터럴 `0`과 일치하는 경우 `y` 필드가 일치하도록 지정하여 `x`축에 있는 모든 지점과 일치합니다. 패턴은 여전히 이 팔의 코드에서 사용할 수 있는 `x` 변수를 생성합니다.

마찬가지로 두 번째 팔은 값이 `0`인 경우 `x` 필드가 일치하도록 지정하고 `y` 필드 값에 대해 변수 `y`를 생성하여 `y` 축의 모든 지점과 일치시킵니다. 세 번째 팔은 리터럴을 지정하지 않으므로 다른 모든 `포인트`와 일치하고 `x` 및 `y` 필드 모두에 대한 변수를 만듭니다.

이 예제에서 값 `p`는 0을 포함하는 `x` 덕분에 두 번째 팔과 일치하므로 이 코드는 `7에서 y축에`를 인쇄합니다.

`일치` 식은 일치하는 첫 번째 패턴을 찾으면 암 검사를 중지하므로 `점 { x: 0, y: 0}`이 `x`축과 `y`축에 있더라도 이 코드는 `0의 x 축에서`만 인쇄합니다.

#### [열거형 분해](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-enums)

우리는 이 책에서 열거형을 해체했지만(예를 들어 6장의 Listing 6-5) 열거형을 해체하는 패턴이 열거형 내에 저장된 데이터가 정의되는 방식과 일치하는지 아직 명시적으로 논의하지 않았습니다. 예를 들어, 목록 18-15에서 우리는 목록 6-2의 `Message` 열거형을 사용하고 각 내부 값을 분해할 패턴으로 `일치`를 작성합니다.

파일 이름: src/main.rs

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!(`The Quit variant has no data to destructure.`);
        }
        Message::Move { x, y } => {
            println!(`Move in the x direction {x} and in the y direction {y}`);
        }
        Message::Write(text) => {
            println!(`Text message: {text}`);
        }
        Message::ChangeColor(r, g, b) => {
            println!(`Change the color to red {r}, green {g}, and blue {b}`,)
        }
    }
}
```

Listing 18-15: 다른 종류의 값을 보유하는 enum 변형 분해

이 코드는 `색상을 빨간색 0, 녹색 160 및 파란색 255로 변경`을 인쇄합니다. `msg` 값을 변경하여 다른 팔 실행의 코드를 확인하십시오.

`Message::Quit`와 같이 데이터가 없는 열거형 변형의 경우 더 이상 값을 분해할 수 없습니다. 리터럴 `Message::Quit` 값에서만 일치시킬 수 있으며 해당 패턴에는 변수가 없습니다.

`Message::Move`와 같은 구조체와 유사한 열거형 변형의 경우 구조체를 일치시키기 위해 지정하는 패턴과 유사한 패턴을 사용할 수 있습니다. 변형 이름 뒤에 중괄호를 넣은 다음 변수가 있는 필드를 나열하여 이 팔의 코드에서 사용할 조각을 나눕니다. 여기서 우리는 Listing 18-13에서 했던 것처럼 속기 형식을 사용합니다.

하나의 요소가 있는 튜플을 보유하는 `Message::Write` 및 세 개의 요소가 있는 튜플을 보유하는 `Message::ChangeColor`와 같은 튜플과 같은 열거형 변형의 경우 패턴은 튜플을 일치시키기 위해 지정하는 패턴과 유사합니다. 패턴의 변수 수는 일치하는 변형의 요소 수와 일치해야 합니다.

#### [중첩된 구조체 및 열거형 분해](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-nested-structs-and-enums)

지금까지 우리의 예제는 모두 한 수준 깊이의 구조체 또는 열거형을 일치시켰지만 일치는 중첩된 항목에서도 작동할 수 있습니다! 예를 들어, 목록 18-16에 표시된 것처럼 `ChangeColor` 메시지에서 RGB 및 HSV 색상을 지원하도록 목록 18-15의 코드를 리팩터링할 수 있습니다.

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(`Change color to red {r}, green {g}, and blue {b}`);
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(`Change color to hue {h}, saturation {s}, value {v}`)
        }
        _ => (),
    }
}
```

Listing 18-16: 중첩 열거형에 대한 일치

`일치` 식의 첫 번째 암 패턴은 `Color::Rgb` 변형을 포함하는 `Message::ChangeColor` 열거형 변형과 일치합니다. 그런 다음 패턴은 세 개의 내부 `i32` 값에 바인딩됩니다. 두 번째 팔의 패턴도 `Message::ChangeColor` 열거형 변형과 일치하지만 내부 열거형은 대신 `Color::Hsv`와 일치합니다. 두 개의 열거형이 포함되어 있어도 하나의 `일치` 식으로 이러한 복잡한 조건을 지정할 수 있습니다.

#### [구조체와 튜플의 구조 분해](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs-and-tuples)

훨씬 더 복잡한 방식으로 구조 분해 패턴을 혼합, 일치 및 중첩할 수 있습니다. 다음 예제는 튜플 내부에 구조체와 튜플을 중첩하고 모든 기본 값을 분해하는 복잡한 분해를 보여줍니다.

```rust
    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

이 코드를 사용하면 복잡한 유형을 구성 요소 부분으로 나눌 수 있으므로 관심 있는 값을 별도로 사용할 수 있습니다.

패턴을 사용한 구조 분해는 구조체의 각 필드 값과 같은 값 조각을 서로 별도로 사용하는 편리한 방법입니다.

### [패턴의 값 무시](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern)

`일치`의 마지막 부분과 같은 패턴의 값을 무시하여 실제로 아무 작업도 수행하지 않고 나머지 가능한 모든 값을 설명하는 포괄적인 값을 얻는 것이 때때로 유용하다는 것을 확인했습니다. 패턴에서 전체 값 또는 값의 일부를 무시하는 몇 가지 방법이 있습니다. ` *` 패턴 사용,* 다른 패턴 내에서 ` ` 패턴 사용, 밑줄로 시작하는 이름 사용 또는 `..` 값의 나머지 부분을 무시합니다. 이러한 각 패턴을 사용하는 방법과 이유를 살펴보겠습니다.

#### [`_`로 전체 값 무시](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-an-entire-value-with-_)

모든 값과 일치하지만 값에 바인딩되지 않는 와일드카드 패턴으로 밑줄을 사용했습니다. 이것은 `일치` 표현식의 마지막 팔로 특히 유용하지만 목록 18-17에 표시된 것처럼 함수 매개변수를 포함하여 모든 패턴에서 사용할 수도 있습니다.

파일 이름: src/main.rs

```rust
fn foo(_: i32, y: i32) {
    println!(`This code only uses the y parameter: {}`, y);
}

fn main() {
    foo(3, 4);
}
```

Listing 18-17: 함수 서명에서 `_` 사용하기

이 코드는 첫 번째 인수로 전달된 값 `3`을 완전히 무시하고 `이 코드는 y 매개변수만 사용합니다: 4`를 인쇄합니다.

특정 함수 매개변수가 더 이상 필요하지 않은 대부분의 경우 사용하지 않는 매개변수를 포함하지 않도록 서명을 변경합니다. 함수 매개변수를 무시하는 것은 예를 들어 특정 유형 서명이 필요할 때 특성을 구현하는 경우에 특히 유용할 수 있지만 구현의 함수 본문에는 매개변수 중 하나가 필요하지 않습니다. 그러면 컴파일러를 가져오는 것을 피할 수 있습니다. 이름을 대신 사용한 경우처럼 사용하지 않는 함수 매개변수에 대한 경고.

#### [중첩된 `_`가 있는 값의 일부 무시](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-parts-of-a-value-with-a-nested-_)

예를 들어 값의 일부만 테스트하고 실행하려는 해당 코드의 다른 부분은 사용하지 않으려는 경우와 같이 다른 패턴 내부에서 `_`를 사용하여 값의 일부만 무시할 수 있습니다. 목록 18-18은 설정 값 관리를 담당하는 코드를 보여줍니다. 비즈니스 요구 사항은 사용자가 설정의 기존 사용자 지정을 덮어쓸 수 없어야 하지만 현재 설정이 해제된 경우 설정을 해제하고 값을 지정할 수 있어야 한다는 것입니다.

```rust
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!(`Can`t overwrite an existing customized value`);
        }
        _ => {
            setting_value = new_setting_value;
        }
    }
    
    println!(`setting is {:?}`, setting_value);
```

목록 18-18: `Some` 내부의 값을 사용할 필요가 없을 때 `Some` 변형과 일치하는 패턴 내에서 밑줄 사용

이 코드는 `기존 사용자 지정 값을 덮어쓸 수 없습니다`를 인쇄한 다음 `설정이 Some(5)입니다`를 인쇄합니다. 첫 번째 일치 항목에서는 `Some` 변형 내부의 값을 일치시키거나 사용할 필요가 없습니다. 하지만 `setting_value`와 `new_setting_value`가 `Some` 변종인 경우에 대한 테스트가 필요합니다. 이 경우 `setting_value`를 변경하지 않은 이유를 인쇄하고 변경되지 않습니다.

두 번째 팔의 `_` 패턴으로 표현되는 다른 모든 경우(`setting_value` 또는 `new_setting_value`가 `None`인 경우)에서는 `new_setting_value`가 `setting_value`가 되도록 허용하려고 합니다.

특정 값을 무시하기 위해 하나의 패턴 내 여러 위치에서 밑줄을 사용할 수도 있습니다. 목록 18-19는 5개 항목의 튜플에서 두 번째 및 네 번째 값을 무시하는 예를 보여줍니다.

```rust
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!(`Some numbers: {first}, {third}, {fifth}`)
        }
    }
```

Listing 18-19: 튜플의 여러 부분 무시하기

이 코드는 `일부 숫자: 2, 8, 32`를 인쇄하고 값 4와 16은 무시합니다.

#### [이름을 `_`로 시작하여 사용하지 않는 변수 무시](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-an-unused-variable-by-starting-its-name-with-_)

변수를 생성했지만 어디에도 사용하지 않으면 Rust는 일반적으로 사용하지 않는 변수가 버그일 수 있으므로 경고를 발행합니다. 그러나 프로토타입을 만들거나 프로젝트를 시작할 때와 같이 아직 사용하지 않을 변수를 만들 수 있는 것이 유용한 경우도 있습니다. 이 상황에서 변수 이름을 밑줄로 시작하여 사용하지 않는 변수에 대해 경고하지 않도록 Rust에 지시할 수 있습니다. 목록 18-20에서 우리는 두 개의 사용하지 않는 변수를 생성하지만 이 코드를 컴파일할 때 그 중 하나에 대한 경고만 받아야 합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

Listing 18-20: 사용하지 않는 변수 경고를 피하기 위해 변수 이름을 밑줄로 시작

여기에서 변수 `y`를 사용하지 않는다는 경고가 표시되지만 `_x`를 사용하지 않는다는 경고는 표시되지 않습니다.

`_`만 사용하는 것과 밑줄로 시작하는 이름을 사용하는 것 사이에는 미묘한 차이가 있습니다. 구문 ` *x`는 여전히 값을 변수에 바인딩하지만 `* `는 전혀 바인딩하지 않습니다. 이 구분이 중요한 경우를 보여주기 위해 Listing 18-21에 오류가 표시됩니다.

```rust
    let s = Some(String::from(`Hello!`));

    if let Some(_s) = s {
        println!(`found a string`);
    }
    
    println!(`{:?}`, s);
```

Listing 18-21: 밑줄로 시작하는 사용되지 않은 변수는 여전히 값을 바인딩하며 값의 소유권을 가질 수 있습니다.

*`s` 값이 여전히 ` s`로 이동되어 `s`를 다시 사용할 수 없기 때문에* 오류가 발생합니다. *그러나 밑줄 자체를 사용하면 값에 바인딩되지 않습니다.* *목록 18-22는 `s`가 ` `로 이동되지 않기 때문에 오류 없이 컴파일됩니다* .

```rust
    let s = Some(String::from(`Hello!`));

    if let Some(_) = s {
        println!(`found a string`);
    }
    
    println!(`{:?}`, s);
```

Listing 18-22: 밑줄을 사용해도 값이 바인딩되지 않습니다.

이 코드는 `s`를 어떤 것에 바인딩하지 않기 때문에 잘 작동합니다. 이동되지 않습니다.

#### [`..`로 값의 나머지 부분 무시](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-remaining-parts-of-a-value-with-)

많은 부분이 있는 값의 경우 `..` 구문을 사용하여 특정 부분을 사용하고 나머지는 무시할 수 있으므로 무시된 각 값에 대해 밑줄을 나열할 필요가 없습니다. `..` 패턴은 패턴의 나머지 부분에서 명시적으로 일치하지 않은 값의 일부를 무시합니다. 목록 18-23에는 3차원 공간에서 좌표를 보유하는 `Point` 구조체가 있습니다. `일치` 표현식에서 `x` 좌표에서만 작동하고 `y` 및 `z` 필드의 값은 무시하려고 합니다.

```rust
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }

    let origin = Point { x: 0, y: 0, z: 0 };
    
    match origin {
        Point { x, .. } => println!(`x is {}`, x),
    }
```

목록 18-23: `..`를 사용하여 `x`를 제외한 `Point`의 모든 필드 무시

`x` 값을 나열한 다음 `..` 패턴만 포함합니다. 이것은 `y: _` 및 `z: _`를 나열하는 것보다 빠릅니다. 특히 하나 또는 두 개의 필드만 관련된 상황에서 많은 필드가 있는 구조체로 작업할 때 그렇습니다.

`..` 구문은 필요한 만큼 많은 값으로 확장됩니다. 목록 18-24는 튜플과 함께 `..`를 사용하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!(`Some numbers: {first}, {last}`);
        }
    }
}
```

목록 18-24: 튜플의 첫 번째 값과 마지막 값만 일치시키고 다른 모든 값은 무시

이 코드에서 첫 번째 값과 마지막 값은 `first` 및 `last`와 일치합니다. `..`는 중간에 있는 모든 항목을 일치시키고 무시합니다.

그러나 `..`를 사용하는 것은 모호하지 않아야 합니다. 일치를 위한 값과 무시해야 하는 값이 명확하지 않은 경우 Rust는 오류를 표시합니다. 목록 18-25는 `..`를 모호하게 사용하는 예를 보여주므로 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!(`Some numbers: {}`, second)
        },
    }
}
```

Listing 18-25: 모호한 방식으로 `..`를 사용하려는 시도

이 예제를 컴파일하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error: `..` can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile `patterns` due to previous error
```

Rust가 값을 `second`와 일치시키기 전에 무시할 튜플의 값 수와 그 이후에 무시할 추가 값 수를 결정하는 것은 불가능합니다. 이 코드는 `2`를 무시하고 `second`를 `4`에 바인딩한 다음 `8`, `16` 및 `32`를 무시하려는 것을 의미할 수 있습니다. 또는 `2`와 `4`를 무시하고 `second`를 `8`에 바인딩한 다음 `16`과 `32`를 무시하려고 합니다. 기타 등등. 변수 이름 `second`는 Rust에 특별한 의미가 없으므로 이렇게 두 위치에 `..`를 사용하는 것이 모호하기 때문에 컴파일러 오류가 발생합니다.

### [매치 가드가 있는 추가 조건부](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#extra-conditionals-with-match-guards)

일치 *가드는* `일치` 암의 패턴 뒤에 지정되는 추가 `if` 조건으로, 해당 암이 선택되도록 일치해야 합니다. 매치 가드는 패턴만으로 허용되는 것보다 더 복잡한 아이디어를 표현하는 데 유용합니다.

조건은 패턴에서 생성된 변수를 사용할 수 있습니다. Listing 18-26은 첫 번째 팔이 `Some(x)` 패턴을 갖고 `if x % 2 == 0`(숫자가 짝수이면 참일 것임)의 일치 가드를 갖는 `일치`를 보여줍니다.

```rust
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!(`The number {} is even`, x),
        Some(x) => println!(`The number {} is odd`, x),
        None => (),
    }
```

Listing 18-26: 패턴에 매치 가드 추가하기

이 예는 `The number 4 is even`을 인쇄합니다. `num`이 첫 번째 팔의 패턴과 비교될 때 `Some(4)`가 `Some(x)`와 일치하기 때문에 일치합니다. 그런 다음 매치 가드는 `x`를 2로 나눈 나머지가 0인지 확인하고 그렇기 때문에 첫 번째 팔을 선택합니다.

대신 `num`이 `Some(5)`인 경우 첫 번째 팔의 매치 가드는 5를 2로 나눈 나머지가 1이고 0이 아니기 때문에 거짓이 됩니다. 그런 다음 Rust는 두 번째 팔로 이동합니다. 두 번째 팔에는 매치 가드가 없으므로 모든 `일부` 변형과 일치하기 때문에 일치합니다.

패턴 내에서 `if x % 2 == 0` 조건을 표현할 방법이 없으므로 매치 가드는 이 논리를 표현할 수 있는 기능을 제공합니다. 이 추가적인 표현력의 단점은 컴파일러가 매치 가드 표현식이 포함될 때 철저함을 확인하려고 시도하지 않는다는 것입니다.

목록 18-11에서 우리는 패턴 그림자 문제를 해결하기 위해 매치 가드를 사용할 수 있다고 언급했습니다. `일치` 외부의 변수를 사용하는 대신 `일치` 표현식의 패턴 내부에 새 변수를 생성했음을 기억하십시오. 새 변수는 외부 변수의 값에 대해 테스트할 수 없음을 의미했습니다. 목록 18-27은 매치 가드를 사용하여 이 문제를 해결하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!(`Got 50`),
        Some(n) if n == y => println!(`Matched, n = {n}`),
        _ => println!(`Default case, x = {:?}`, x),
    }
    
    println!(`at the end: x = {:?}, y = {y}`, x);
}
```

Listing 18-27: 매치 가드를 사용하여 외부 변수와의 동등성 테스트

이 코드는 이제 `Default case, x = Some(5)`를 인쇄합니다. 두 번째 매치 암의 패턴은 외부 `y`를 가리는 새 변수 `y`를 도입하지 않습니다. 즉, 매치 가드에서 외부 `y`를 사용할 수 있습니다. 외부 `y`를 가리는 `Some(y)`로 패턴을 지정하는 대신 `Some(n)`을 지정합니다. 이렇게 하면 `일치` 외부에 `n` 변수가 없기 때문에 아무 것도 숨기지 않는 새 변수 `n`이 생성됩니다.

매치 가드 `if n == y`는 패턴이 아니므로 새 변수를 도입하지 않습니다. 이 `y` *는* 음영 처리된 새로운 `y`가 아닌 외부 `y`이며 `n`과 `y`를 비교하여 외부 `y`와 동일한 값을 갖는 값을 찾을 수 있습니다.

*또는* 연산자 `|`를 사용할 수도 있습니다. 매치 가드에서 여러 패턴을 지정합니다. 매치 가드 조건은 모든 패턴에 적용됩니다. 목록 18-28은 `|`를 사용하는 패턴을 결합할 때의 우선 순위를 보여줍니다. 매치 가드와 함께. 이 예제의 중요한 부분은 `if y`가 `6`에만 적용되는 것처럼 보일 수 있지만 `if y` 일치 가드가 `4`, `5` *및* `6`에 적용된다는 것입니다.

```rust
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!(`yes`),
        _ => println!(`no`),
    }
```

Listing 18-28: 매치 가드로 여러 패턴 결합하기

일치 조건은 `x` 값이 `4`, `5` 또는 `6`과 같고 `y`가 `true`인 경우 *에만 팔이 일치한다고 명시합니다.* 이 코드를 실행하면 `x`가 `4`이므로 첫 번째 팔의 패턴이 일치하지만 일치 가드 `if y`가 false이므로 첫 번째 팔이 선택되지 않습니다. 코드는 일치하는 두 번째 팔로 이동하고 이 프로그램은 `no`를 인쇄합니다. 그 이유는 `if` 조건이 마지막 값 `6`뿐만 아니라 패턴 `4 | 5 | 6` 전체에 적용되기 때문입니다. 즉, 패턴과 관련하여 매치 가드의 우선 순위는 다음과 같이 작동합니다.

```
(4 | 5 | 6) if y => ...
```

이것보다는:

```
4 | 5 | (6 if y) => ...
```

코드를 실행한 후 우선 순위 동작이 분명합니다. 일치 가드가 `|`를 사용하여 지정된 값 목록의 최종 값에만 적용된 경우 연산자를 사용하면 암이 일치하고 프로그램이 `예`를 인쇄했을 것입니다.

### [`@` 바인딩](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#-bindings)

at 연산자 `@`를 사용하면 패턴 일치를 위해 해당 값을 테스트하는 동시에 값을 보유하는 변수를 만들 수 있습니다 *.* 목록 18-29에서 `Message::Hello` `id` 필드가 `3..=7` 범위 내에 있는지 테스트하고 싶습니다. 또한 팔과 관련된 코드에서 사용할 수 있도록 변수 `id_variable`에 값을 바인딩하려고 합니다. 이 변수의 이름을 필드와 동일하게 `id`로 지정할 수 있지만 이 예에서는 다른 이름을 사용합니다.

```rust
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };
    
    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!(`Found an id in range: {}`, id_variable),
        Message::Hello { id: 10..=12 } => {
            println!(`Found an id in another range`)
        }
        Message::Hello { id } => println!(`Found some other id: {}`, id),
    }
```

Listing 18-29: `@`를 사용하여 테스트하는 동안 패턴의 값에 바인딩

이 예는 `Found an id in range: 5`를 인쇄합니다. `3..=7` 범위 앞에 `id_variable @`를 지정하면 값이 범위 패턴과 일치하는지 테스트하면서 범위와 일치하는 모든 값을 캡처합니다.

패턴에 지정된 범위만 있는 두 번째 팔에서 팔과 연결된 코드에는 `id` 필드의 실제 값을 포함하는 변수가 없습니다. `id` 필드의 값은 10, 11 또는 12일 수 있지만 해당 패턴과 관련된 코드는 그것이 무엇인지 모릅니다. `id` 값을 변수에 저장하지 않았기 때문에 패턴 코드는 `id` 필드의 값을 사용할 수 없습니다.

범위 없이 변수를 지정한 마지막 단계에서 `id`라는 변수의 단계 코드에서 사용할 수 있는 값이 있습니다. 그 이유는 우리가 구조체 필드 속기 구문을 사용했기 때문입니다. 그러나 처음 두 개의 팔에서 했던 것처럼 이 팔의 `id` 필드 값에 어떤 테스트도 적용하지 않았습니다. 모든 값이 이 패턴과 일치합니다.

`@`를 사용하면 값을 테스트하고 한 패턴 내의 변수에 저장할 수 있습니다.

## [요약](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#summary)

Rust의 패턴은 서로 다른 종류의 데이터를 구별하는 데 매우 유용합니다. `일치` 표현식에 사용될 때 Rust는 패턴이 가능한 모든 값을 포함하도록 보장합니다. 그렇지 않으면 프로그램이 컴파일되지 않습니다. `let` 문과 함수 매개변수의 패턴은 이러한 구조를 더 유용하게 만들어 변수에 할당하는 동시에 값을 더 작은 부분으로 분해할 수 있도록 합니다. 필요에 따라 간단하거나 복잡한 패턴을 만들 수 있습니다.

다음으로, 이 책의 끝에서 두 번째 장에서는 다양한 Rust 기능의 몇 가지 고급 측면을 살펴보겠습니다.

------

# 19

# [고급 기능](https://doc.rust-lang.org/book/ch19-00-advanced-features.html#advanced-features)

지금까지 Rust 프로그래밍 언어에서 가장 일반적으로 사용되는 부분을 배웠습니다. 20장에서 프로젝트를 하나 더 수행하기 전에 가끔 접할 수 있지만 매일 사용하지는 않는 언어의 몇 가지 측면을 살펴보겠습니다. 미지수를 만났을 때 이 장을 참조로 사용할 수 있습니다. 여기서 다루는 기능은 매우 특정한 상황에서 유용합니다. 자주 접할 수는 없지만 Rust가 제공하는 모든 기능을 이해하고 있는지 확인하고 싶습니다.

이 장에서는 다음을 다룰 것입니다.

- 안전하지 않은 Rust: Rust의 보증 중 일부를 거부하고 이러한 보증을 수동으로 유지하는 책임을 지는 방법
- 고급 특성: 연관 유형, 기본 유형 매개변수, 정규화된 구문, 상위 특성 및 특성과 관련된 새로운 유형 패턴
- 고급 유형: newtype 패턴, 유형 별칭, 절대 유형 및 동적 크기 유형에 대한 추가 정보
- 고급 함수 및 클로저: 함수 포인터 및 반환 클로저
- 매크로: 컴파일 시간에 더 많은 코드를 정의하는 코드를 정의하는 방법

모든 사람을 위한 Rust 기능의 웅장함입니다! 다이빙하자!

------

## [안전하지 않은 녹](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#unsafe-rust)

지금까지 우리가 논의한 모든 코드는 컴파일 타임에 러스트의 메모리 안전 보장을 시행했습니다. 그러나 Rust에는 이러한 메모리 안전 보장을 시행하지 않는 두 번째 언어가 숨겨져 있습니다. *안전하지 않은 Rust* 라고 하며 일반 Rust처럼 작동하지만 추가 초능력을 제공합니다.

Unsafe Rust가 존재하는 이유는 본질적으로 정적 분석이 보수적이기 때문입니다. 컴파일러가 코드가 보증을 유지하는지 여부를 결정하려고 할 때 일부 유효하지 않은 프로그램을 허용하는 것보다 일부 유효한 프로그램을 거부하는 것이 좋습니다. 코드는 괜찮을 *수* 있지만 Rust 컴파일러가 확신할 수 있는 충분한 정보가 없으면 코드를 거부합니다. 이 경우 안전하지 않은 코드를 사용하여 컴파일러에 `날 믿어, 내가 뭘 하는지 알아.`라고 말할 수 있습니다. 그러나 안전하지 않은 Rust를 사용하는 데 따른 위험은 사용자가 감수해야 합니다. 안전하지 않은 코드를 잘못 사용하면 널 포인터 역참조와 같은 메모리 안전하지 않은 문제가 발생할 수 있습니다.

Rust가 안전하지 않은 분신을 갖는 또 다른 이유는 기본 컴퓨터 하드웨어가 본질적으로 안전하지 않기 때문입니다. Rust가 안전하지 않은 작업을 허용하지 않으면 특정 작업을 수행할 수 없습니다. Rust는 운영 체제와 직접 상호 작용하거나 심지어 자신의 운영 체제를 작성하는 것과 같은 저수준 시스템 프로그래밍을 수행할 수 있도록 허용해야 합니다. 저수준 시스템 프로그래밍 작업은 언어의 목표 중 하나입니다. 안전하지 않은 Rust로 무엇을 할 수 있고 어떻게 하는지 탐구해 봅시다.

### [안전하지 않은 초능력](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#unsafe-superpowers)

안전하지 않은 Rust로 전환하려면 `unsafe` 키워드를 사용한 다음 안전하지 않은 코드를 포함하는 새 블록을 시작하세요. 안전하지 않은 Rust에서는 할 수 없는 다섯 가지 조치를 취할 수 *있습니다* . 이러한 초능력에는 다음과 같은 기능이 포함됩니다.

- 원시 포인터 역참조
- 안전하지 않은 함수 또는 메서드 호출
- 변경 가능한 정적 변수 액세스 또는 수정
- 안전하지 않은 특성 구현
- `union`의 액세스 필드

`unsafe`는 빌림 검사기를 끄거나 Rust의 다른 안전 검사를 비활성화하지 않는다는 것을 이해하는 것이 중요합니다. 안전하지 않은 코드에서 참조를 사용하는 경우에도 여전히 검사됩니다. `unsafe` 키워드는 메모리 안전을 위해 컴파일러에서 검사하지 않는 이 다섯 가지 기능에 대한 액세스만 제공합니다. 안전하지 않은 블록 내부에서도 어느 정도의 안전을 확보할 수 있습니다.

또한 `안전하지 않음`은 블록 내부의 코드가 반드시 위험하거나 메모리 안전 문제가 있음을 의미하지 않습니다. 의도는 프로그래머로서 `안전하지 않은` 블록 내부의 코드가 메모리에 액세스하도록 보장하는 것입니다. 유효한 방법으로.

사람들은 오류를 범할 수 있고 실수가 발생할 수 있지만 이러한 다섯 가지 안전하지 않은 작업이 `안전하지 않음`으로 주석이 달린 블록 내부에 있어야 하므로 메모리 안전과 관련된 모든 오류가 `안전하지 않은` 블록 내에 있어야 함을 알 수 있습니다. `안전하지 않은` 블록을 작게 유지하십시오. 나중에 메모리 버그를 조사하면 감사하게 될 것입니다.

안전하지 않은 코드를 최대한 격리하려면 안전한 추상화 내에서 안전하지 않은 코드를 묶고 안전한 API를 제공하는 것이 가장 좋습니다. 이 장의 뒷부분에서 안전하지 않은 함수와 메서드를 검토할 때 이에 대해 논의할 것입니다. 표준 라이브러리의 일부는 감사된 안전하지 않은 코드에 대한 안전한 추상화로 구현됩니다. 안전하지 않은 코드를 안전한 추상화로 래핑하면 안전한 추상화를 사용하는 것이 안전하기 때문에 귀하 또는 귀하의 사용자가 `안전하지 않은` 코드로 구현된 기능을 사용하려는 모든 위치로 `안전하지 않은` 사용이 누출되는 것을 방지할 수 있습니다.

5가지 안전하지 않은 초강대국을 차례로 살펴보겠습니다. 또한 안전하지 않은 코드에 안전한 인터페이스를 제공하는 몇 가지 추상화에 대해서도 살펴보겠습니다.

### [원시 포인터 역참조](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#dereferencing-a-raw-pointer)

4장의 [`댕글링 참조`](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#dangling-references) 섹션에서 우리는 컴파일러가 참조가 항상 유효한지 확인한다고 언급했습니다. Unsafe Rust 에는 참조와 유사한 *원시 포인터* 라는 두 가지 새로운 유형이 있습니다. 참조와 마찬가지로 원시 포인터는 변경할 수 없거나 변경할 수 있으며 각각 `*const T` 및 `*mut T`로 작성됩니다. 별표는 역참조 연산자가 아닙니다. 유형 이름의 일부입니다. 원시 포인터의 맥락에서 *불변이란* 포인터가 역참조된 후에 직접 할당될 수 없음을 의미합니다.

참조 및 스마트 포인터와 달리 원시 포인터는 다음과 같습니다.

- 불변 및 가변 포인터 또는 동일한 위치에 대한 여러 가변 포인터를 가짐으로써 차용 규칙을 무시할 수 있습니다.
- 유효한 메모리를 가리키도록 보장되지 않음
- null이 허용됨
- 자동 정리를 구현하지 마십시오.

Rust가 이러한 보증을 적용하지 않도록 선택하면 더 나은 성능 또는 Rust의 보증이 적용되지 않는 다른 언어 또는 하드웨어와의 인터페이스 기능을 대가로 보장된 안전성을 포기할 수 있습니다.

목록 19-1은 참조에서 불변 및 가변 원시 포인터를 만드는 방법을 보여줍니다.

```rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
```

목록 19-1: 참조에서 원시 포인터 만들기

이 코드에는 `안전하지 않은` 키워드가 포함되어 있지 않습니다. 안전한 코드에서 원시 포인터를 만들 수 있습니다. 곧 알게 되겠지만 안전하지 않은 블록 외부의 원시 포인터를 역참조할 수 없습니다.

`as`를 사용하여 불변 참조와 가변 참조를 해당 원시 포인터 유형으로 변환하여 원시 포인터를 만들었습니다. 유효하다고 보장된 참조에서 직접 생성했기 때문에 이러한 특정 원시 포인터가 유효하다는 것을 알고 있지만 모든 원시 포인터에 대해 그런 가정을 할 수는 없습니다.

이를 증명하기 위해 다음으로 유효성을 확신할 수 없는 원시 포인터를 만듭니다. 목록 19-2는 메모리의 임의 위치에 대한 원시 포인터를 만드는 방법을 보여줍니다. 임의의 메모리를 사용하려는 시도는 정의되지 않습니다. 해당 주소에 데이터가 있을 수도 있고 없을 수도 있고, 컴파일러가 코드를 최적화하여 메모리 액세스가 없거나 프로그램이 분할 오류로 오류가 발생할 수 있습니다. 일반적으로 이와 같은 코드를 작성할 타당한 이유는 없지만 가능합니다.

```rust
    let address = 0x012345usize;
    let r = address as *const i32;
```

목록 19-2: 임의의 메모리 주소에 대한 원시 포인터 생성

안전한 코드에서 원시 포인터를 만들 수 있지만 원시 포인터를 *역참조* 하고 가리키는 데이터를 읽을 수는 없습니다. 목록 19-3에서 우리는 `안전하지 않은` 블록이 필요한 원시 포인터에 대해 역참조 연산자 `*`를 사용합니다.

```rust
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    unsafe {
        println!(`r1 is: {}`, *r1);
        println!(`r2 is: {}`, *r2);
    }
```

목록 19-3: `안전하지 않은` 블록 내에서 원시 포인터 역참조

포인터를 생성해도 아무런 문제가 없습니다. 유효하지 않은 값을 처리하게 될 수도 있는 것은 그것이 가리키는 값에 액세스하려고 할 때만입니다.

또한 Listing 19-1과 19-3에서 `num`이 저장된 동일한 메모리 위치를 가리키는 `*const i32` 및 `*mut i32` 원시 포인터를 만들었습니다. 대신 `num`에 대한 변경 불가능한 참조와 변경 가능한 참조를 만들려고 시도했다면 Rust의 소유권 규칙이 변경 불가능한 참조와 동시에 변경 가능한 참조를 허용하지 않기 때문에 코드가 컴파일되지 않았을 것입니다. 원시 포인터를 사용하면 동일한 위치에 대한 가변 포인터와 불변 포인터를 생성하고 가변 포인터를 통해 데이터를 변경하여 잠재적으로 데이터 경합을 일으킬 수 있습니다. 조심하세요!

이러한 모든 위험이 있는 상황에서 원시 포인터를 사용하는 이유는 무엇입니까? [한 가지 주요 사용 사례는 다음 섹션인 `안전하지 않은 함수 또는 메서드 호출`](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#calling-an-unsafe-function-or-method) 에서 볼 수 있듯이 C 코드와 인터페이스할 때입니다. 또 다른 경우는 차용 검사기가 이해하지 못하는 안전한 추상화를 구축하는 경우입니다. 안전하지 않은 함수를 소개하고 안전하지 않은 코드를 사용하는 안전한 추상화의 예를 살펴보겠습니다.

### [안전하지 않은 함수 또는 메서드 호출](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#calling-an-unsafe-function-or-method)

안전하지 않은 블록에서 수행할 수 있는 두 번째 작업 유형은 안전하지 않은 함수를 호출하는 것입니다. 안전하지 않은 함수 및 메서드는 일반 함수 및 메서드와 똑같이 생겼지만 나머지 정의 앞에 추가 `unsafe`가 있습니다. 이 문맥에서 `안전하지 않은` 키워드는 이 함수를 호출할 때 유지해야 하는 요구 사항이 함수에 있음을 나타냅니다. 왜냐하면 Rust는 우리가 이러한 요구 사항을 충족했다고 보장할 수 없기 때문입니다. `안전하지 않은` 블록 내에서 안전하지 않은 함수를 호출함으로써 우리는 이 함수의 문서를 읽었으며 함수의 계약을 유지할 책임이 있음을 의미합니다.

다음은 본문에서 아무 작업도 수행하지 않는 `위험한`이라는 이름의 안전하지 않은 함수입니다.

```rust
    unsafe fn dangerous() {}

    unsafe {
        dangerous();
    }
```

별도의 `안전하지 않은` 블록 내에서 `위험한` 함수를 호출해야 합니다. `unsafe` 블록 없이 `dangerous`를 호출하려고 하면 오류가 발생합니다.

```bash
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0133]: call to unsafe function is unsafe and requires unsafe function or block
 --> src/main.rs:4:5
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
  |
  = note: consult the function`s documentation for information on how to avoid undefined behavior

For more information about this error, try `rustc --explain E0133`.
error: could not compile `unsafe-example` due to previous error
```

`안전하지 않은` 블록을 사용하여 우리는 함수의 문서를 읽었고 함수를 올바르게 사용하는 방법을 이해했으며 함수의 계약을 이행하고 있음을 확인했다고 Rust에 주장합니다.

안전하지 않은 함수의 본문은 사실상 `안전하지 않은` 블록이므로 안전하지 않은 함수 내에서 다른 안전하지 않은 작업을 수행하기 위해 다른 `안전하지 않은` 블록을 추가할 필요가 없습니다.

#### [안전하지 않은 코드에 대한 안전한 추상화 만들기](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#creating-a-safe-abstraction-over-unsafe-code)

함수에 안전하지 않은 코드가 포함되어 있다고 해서 전체 함수를 안전하지 않은 것으로 표시해야 한다는 의미는 아닙니다. 실제로 안전하지 않은 코드를 안전한 함수로 래핑하는 것은 일반적인 추상화입니다. 예를 들어 안전하지 않은 코드가 필요한 표준 라이브러리의 `split_at_mut` 함수를 살펴보겠습니다. 구현 방법에 대해 알아보겠습니다. 이 안전한 방법은 변경 가능한 슬라이스에 정의되어 있습니다. 하나의 슬라이스를 가져와 인수로 주어진 인덱스에서 슬라이스를 분할하여 두 개로 만듭니다. 목록 19-4는 `split_at_mut`을 사용하는 방법을 보여줍니다.

```rust
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];
    
    let (a, b) = r.split_at_mut(3);
    
    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
```

목록 19-4: 안전한 `split_at_mut` 함수 사용하기

안전한 Rust만으로는 이 기능을 구현할 수 없습니다. 컴파일되지 않는 Listing 19-5와 같은 시도가 있을 수 있습니다. 단순화를 위해 `split_at_mut`을 메서드가 아닌 함수로 구현하고 일반 유형 `T`가 아닌 `i32` 값의 조각에 대해서만 구현합니다.

```rust
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);
    
    (&mut values[..mid], &mut values[mid..])
}
```

목록 19-5: 안전한 Rust만 사용하여 시도한 `split_at_mut` 구현

이 함수는 먼저 슬라이스의 전체 길이를 가져옵니다. 그런 다음 매개변수로 주어진 인덱스가 길이보다 작거나 같은지 확인하여 슬라이스 내에 있다고 주장합니다. 어설션은 슬라이스를 분할할 길이보다 큰 인덱스를 전달하면 해당 인덱스를 사용하려고 시도하기 전에 함수가 패닉 상태가 된다는 것을 의미합니다.

그런 다음 두 개의 변경 가능한 슬라이스를 튜플로 반환합니다. 하나는 원래 슬라이스의 시작부터 `mid` 인덱스까지, 다른 하나는 `mid`부터 슬라이스 끝까지입니다.

목록 19-5의 코드를 컴파일하려고 하면 오류가 발생합니다.

```bash
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let`s call the lifetime of this reference ``1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for ``1`

For more information about this error, try `rustc --explain E0499`.
error: could not compile `unsafe-example` due to previous error
```

Rust의 차용 검사기는 우리가 슬라이스의 다른 부분을 차용하고 있다는 것을 이해할 수 없습니다. 동일한 조각에서 두 번 빌리고 있다는 것만 알고 있습니다. 슬라이스의 다른 부분을 빌리는 것은 두 슬라이스가 겹치지 않기 때문에 근본적으로 괜찮지만 Rust는 이것을 알 만큼 똑똑하지 않습니다. 우리는 코드가 괜찮다는 것을 알지만 Rust는 그렇지 않을 때 안전하지 않은 코드에 도달할 때입니다.

목록 19-6은 `안전하지 않은` 블록, 원시 포인터 및 안전하지 않은 함수에 대한 일부 호출을 사용하여 `split_at_mut` 작업을 구현하는 방법을 보여줍니다.

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);
    
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

Listing 19-6: `split_at_mut` 함수 구현에 안전하지 않은 코드 사용

[4장의 `슬라이스 유형`](https://doc.rust-lang.org/book/ch04-03-slices.html#the-slice-type) 섹션에서 슬라이스는 일부 데이터 및 슬라이스 길이에 대한 포인터임을 상기하십시오 . `len` 메서드를 사용하여 슬라이스 길이를 가져오고 `as_mut_ptr` 메서드를 사용하여 슬라이스의 원시 포인터에 액세스합니다. 이 경우 `i32` 값에 대한 가변 슬라이스가 있기 때문에 `as_mut_ptr`은 변수 `ptr`에 저장한 `*mut i32` 유형의 원시 포인터를 반환합니다.

`mid` 인덱스가 슬라이스 내에 있다는 주장을 유지합니다. 그런 다음 안전하지 않은 코드에 도달합니다. `slice::from_raw_parts_mut` 함수는 원시 포인터와 길이를 가져와 슬라이스를 생성합니다. 이 함수를 사용하여 `ptr`에서 시작하고 `mid` 항목 길이인 조각을 만듭니다. 그런 다음 `mid`를 인수로 사용하여 `ptr`에서 `add` 메서드를 호출하여 `mid`에서 시작하는 원시 포인터를 가져오고 해당 포인터와 `mid` 이후의 나머지 항목 수를 사용하여 조각을 만듭니다. 길이.

`slice::from_raw_parts_mut` 함수는 원시 포인터를 사용하고 이 포인터가 유효하다고 신뢰해야 하기 때문에 안전하지 않습니다. 원시 포인터의 `add` 메서드도 오프셋 위치가 유효한 포인터임을 신뢰해야 하기 때문에 안전하지 않습니다. 따라서 `slice::from_raw_parts_mut` 및 `add`에 대한 호출 주위에 `안전하지 않은` 블록을 넣어 호출할 수 있도록 해야 했습니다. 코드를 살펴보고 `mid`가 `len`보다 작거나 같아야 한다는 주장을 추가하면 `unsafe` 블록 내에서 사용되는 모든 원시 포인터가 슬라이스 내 데이터에 대한 유효한 포인터임을 알 수 있습니다. 이것은 `안전하지 않은`의 허용 가능하고 적절한 사용입니다.

결과 `split_at_mut` 함수를 `unsafe`로 표시할 필요가 없으며 안전한 Rust에서 이 함수를 호출할 수 있습니다. 안전한 방식으로 `안전하지 않은` 코드를 사용하는 함수 구현으로 안전하지 않은 코드에 대한 안전한 추상화를 만들었습니다. 이 함수가 액세스할 수 있는 데이터에서 유효한 포인터만 생성하기 때문입니다.

대조적으로, Listing 19-7에서 `slice::from_raw_parts_mut`의 사용은 슬라이스가 사용될 때 충돌할 가능성이 있습니다. 이 코드는 임의의 메모리 위치를 사용하여 10,000개 항목 길이의 슬라이스를 만듭니다.

```rust
    use std::slice;

    let address = 0x01234usize;
    let r = address as *mut i32;
    
    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
```

목록 19-7: 임의의 메모리 위치에서 슬라이스 생성

우리는 이 임의의 위치에 있는 메모리를 소유하지 않으며 이 코드가 생성하는 슬라이스에 유효한 `i32` 값이 포함된다는 보장이 없습니다. 유효한 슬라이스인 것처럼 `값`을 사용하려고 하면 정의되지 않은 동작이 발생합니다.

#### [`extern` 함수를 사용하여 외부 코드 호출](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code)

때로는 Rust 코드가 다른 언어로 작성된 코드와 상호 작용해야 할 수도 있습니다. *이를 위해 Rust에는 외부 기능 인터페이스(FFI)* 의 생성 및 사용을 용이하게 하는 키워드 `extern`이 있습니다. FFI는 프로그래밍 언어가 함수를 정의하고 다른(외국) 프로그래밍 언어가 해당 함수를 호출할 수 있도록 하는 방법입니다.

목록 19-8은 C 표준 라이브러리에서 `abs` 함수와 통합을 설정하는 방법을 보여줍니다. `extern` 블록 내에서 선언된 함수는 Rust 코드에서 호출하기에 항상 안전하지 않습니다. 그 이유는 다른 언어는 Rust의 규칙과 보장을 강제하지 않으며 Rust는 이를 확인할 수 없기 때문에 안전을 보장할 책임은 프로그래머에게 있습니다.

파일 이름: src/main.rs

```rust
extern `C` {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!(`Absolute value of -3 according to C: {}`, abs(-3));
    }
}
```

Listing 19-8: 다른 언어로 정의된 `extern` 함수 선언 및 호출

`extern `C`` 블록 내에서 호출하려는 다른 언어의 외부 함수 이름과 서명을 나열합니다. ``C`` 부분은 외부 함수가 사용하는 *애플리케이션 바이너리 인터페이스(ABI)를* 정의합니다. ABI는 어셈블리 수준에서 함수를 호출하는 방법을 정의합니다. ``C`` ABI는 가장 일반적이며 C 프로그래밍 언어의 ABI를 따릅니다.

> #### [다른 언어에서 Rust 함수 호출하기](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#calling-rust-functions-from-other-languages)
>
> 또한 `extern`을 사용하여 다른 언어에서 Rust 함수를 호출할 수 있는 인터페이스를 만들 수 있습니다. 전체 `extern` 블록을 생성하는 대신 `extern` 키워드를 추가하고 관련 기능에 대한 `fn` 키워드 바로 앞에 사용할 ABI를 지정합니다. 또한 `#[no_mangle]` 주석을 추가하여 Rust 컴파일러가 이 함수의 이름을 변경하지 않도록 해야 합니다. *맹글링은* 컴파일러가 우리가 함수에 부여한 이름을 컴파일 프로세스의 다른 부분에서 사용할 더 많은 정보를 포함하지만 사람이 읽을 수 없는 다른 이름으로 변경하는 경우입니다. 모든 프로그래밍 언어 컴파일러는 이름을 조금씩 다르게 변환하므로 다른 언어에서 Rust 함수의 이름을 지정할 수 있으려면 Rust 컴파일러의 이름 변환을 비활성화해야 합니다.
>
> 다음 예제에서는 공유 라이브러리로 컴파일되고 C에서 링크된 후 C 코드에서 `call_from_c` 함수에 액세스할 수 있도록 합니다.
>
> ```rust
> #[no_mangle]
> pub extern `C` fn call_from_c() {
>     println!(`Just called a Rust function from C!`);
> }
> ```
>
> 이 `extern` 사용에는 `unsafe`가 필요하지 않습니다.

### [변경 가능한 정적 변수 액세스 또는 수정](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#accessing-or-modifying-a-mutable-static-variable)

이 책에서 우리는 Rust가 지원하지만 Rust의 소유권 규칙에 문제가 될 수 있는 *전역 변수* 에 대해 아직 이야기하지 않았습니다. 두 스레드가 동일한 변경 가능한 전역 변수에 액세스하는 경우 데이터 경합이 발생할 수 있습니다.

Rust에서는 전역 변수를 *정적* 변수라고 합니다. Listing 19-9는 문자열 슬라이스를 값으로 갖는 정적 변수의 예제 선언 및 사용을 보여줍니다.

파일 이름: src/main.rs

```rust
static HELLO_WORLD: &str = `Hello, world!`;

fn main() {
    println!(`name is: {}`, HELLO_WORLD);
}
```

목록 19-9: 불변 정적 변수 정의 및 사용

[정적 변수는 3장의 `변수와 상수의 차이점`](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#constants) 섹션 에서 논의한 상수와 유사합니다. 정적 변수의 이름은 규칙에 따라 `SCREAMING_SNAKE_CASE`에 있습니다. 정적 변수는 ``정적` 수명이 있는 참조만 저장할 수 있습니다. 이는 Rust 컴파일러가 수명을 파악할 수 있고 명시적으로 주석을 달 필요가 없다는 것을 의미합니다. 불변 정적 변수에 액세스하는 것은 안전합니다.

상수와 불변 정적 변수의 미묘한 차이점은 정적 변수의 값이 메모리에 고정된 주소를 갖는다는 것입니다. 값을 사용하면 항상 동일한 데이터에 액세스합니다. 반면에 상수는 사용될 때마다 데이터를 복제할 수 있습니다. 또 다른 차이점은 정적 변수는 변경 가능하다는 것입니다. 변경 가능한 정적 변수에 액세스하고 수정하는 것은 *안전하지 않습니다* . 목록 19-10은 `COUNTER`라는 가변 정적 변수를 선언, 액세스 및 수정하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!(`COUNTER: {}`, COUNTER);
    }
}
```

목록 19-10: 변경 가능한 정적 변수에서 읽거나 쓰는 것은 안전하지 않습니다.

일반 변수와 마찬가지로 `mut` 키워드를 사용하여 가변성을 지정합니다. `COUNTER`에서 읽거나 쓰는 모든 코드는 `안전하지 않은` 블록 내에 있어야 합니다. 이 코드는 단일 스레드이기 때문에 예상한 대로 `COUNTER: 3`을 컴파일하고 인쇄합니다. 여러 스레드가 `COUNTER`에 액세스하면 데이터 경합이 발생할 수 있습니다.

전역적으로 액세스할 수 있는 변경 가능한 데이터를 사용하면 데이터 경합이 없는지 확인하기 어렵습니다. 이것이 Rust가 변경 가능한 정적 변수를 안전하지 않은 것으로 간주하는 이유입니다. 가능한 경우 16장에서 논의한 동시성 기술과 스레드로부터 안전한 스마트 포인터를 사용하여 컴파일러가 다른 스레드에서 액세스한 데이터가 안전하게 수행되었는지 확인하는 것이 좋습니다.

### [안전하지 않은 특성 구현](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#implementing-an-unsafe-trait)

`unsafe`를 사용하여 안전하지 않은 특성을 구현할 수 있습니다. 해당 메서드 중 적어도 하나에 컴파일러가 확인할 수 없는 불변성이 있는 경우 특성은 안전하지 않습니다. Listing 19-11과 같이 `trait` 앞에 `unsafe` 키워드를 추가하고 트레이트 구현도 `unsafe`로 표시하여 트레이트가 `unsafe`임을 선언합니다.

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
```

목록 19-11: 안전하지 않은 특성 정의 및 구현

`unsafe impl`을 사용함으로써 우리는 컴파일러가 확인할 수 없는 불변성을 유지할 것이라고 약속합니다.

예를 들어, 16장의 `동기화` 및 `전송` 특성을 사용한 확장 가능한 동시성 섹션에서 논의한 ` [동기화` 및 `전송`](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits) 마커 특성을 기억하십시오. 유형이 완전히 구성된 경우 컴파일러는 이러한 특성을 자동으로 구현합니다. `보내기` 및 `동기화` 유형. 원시 포인터와 같이 `Send` 또는 `Sync`가 아닌 유형을 포함하는 유형을 구현하고 해당 유형을 `Send` 또는 `Sync`로 표시하려는 경우 `unsafe`를 사용해야 합니다. Rust는 우리의 유형이 스레드 간에 안전하게 전송되거나 여러 스레드에서 액세스될 수 있다는 보장을 유지하는지 확인할 수 없습니다. 따라서 이러한 검사를 수동으로 수행하고 `안전하지 않음`으로 표시해야 합니다.

### [Union의 필드에 접근하기](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#accessing-fields-of-a-union)

`unsafe`에서만 작동하는 마지막 작업은 *union* 의 필드에 액세스하는 것입니다. `공용체`는 `구조체`와 유사하지만 특정 인스턴스에서 한 번에 하나의 선언된 필드만 사용됩니다. 공용체는 주로 C 코드의 공용체와 인터페이스하는 데 사용됩니다. 러스트가 현재 유니온 인스턴스에 저장되고 있는 데이터의 유형을 보장할 수 없기 때문에 유니온 필드에 접근하는 것은 안전하지 않습니다. [Rust Reference](https://doc.rust-lang.org/reference/items/unions.html) 에서 공용체에 대해 자세히 알아볼 수 있습니다.

### [안전하지 않은 코드를 사용해야 하는 경우](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#when-to-use-unsafe-code)

방금 논의한 5가지 조치(초강대국) 중 하나를 수행하기 위해 `안전하지 않은`을 사용하는 것은 잘못되거나 눈살을 찌푸리게 하지 않습니다. 그러나 컴파일러가 메모리 안전을 유지하는 데 도움을 줄 수 없기 때문에 `안전하지 않은` 코드를 수정하는 것이 더 까다롭습니다. `안전하지 않은` 코드를 사용할 이유가 있을 때 그렇게 할 수 있으며 명시적인 `안전하지 않은` 주석이 있으면 문제가 발생할 때 문제의 원인을 쉽게 추적할 수 있습니다.

------

## [고급 특성](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#advanced-traits)

[10장의 `특성: 공유 행동 정의`](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-defining-shared-behavior) 섹션 에서 특성을 처음 다루었 지만 더 자세한 내용은 다루지 않았습니다. Rust에 대해 더 많이 알게 되었으니 핵심으로 들어갈 수 있습니다.

### [관련 유형을 사용하여 특성 정의에서 자리 표시자 유형 지정](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types)

*연관된 유형은* 특성 메소드 정의가 서명에서 이러한 자리 표시자 유형을 사용할 수 있도록 특성과 유형 자리 표시자를 연결합니다. 특성의 구현자는 특정 구현에 대한 자리 표시자 유형 대신 사용할 구체적인 유형을 지정합니다. 그렇게 하면 특성이 구현될 때까지 해당 유형이 무엇인지 정확히 알 필요 없이 일부 유형을 사용하는 특성을 정의할 수 있습니다.

이 장에서는 대부분의 고급 기능이 거의 필요하지 않은 것으로 설명했습니다. 연관된 유형은 중간 어딘가에 있습니다. 이들은 책의 나머지 부분에서 설명하는 기능보다 드물게 사용되지만 이 장에서 논의하는 다른 많은 기능보다 더 일반적으로 사용됩니다.

관련 유형이 있는 특성의 한 가지 예는 표준 라이브러리에서 제공하는 `반복자` 특성입니다. 연관된 유형의 이름은 `항목`이며 `반복자` 특성을 구현하는 유형이 반복하는 값의 유형을 나타냅니다. `반복자` 특성의 정의는 Listing 19-12와 같습니다.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

Listing 19-12: 연관 유형 `Item`이 있는 `Iterator` 특성의 정의

[`Item` 유형은 자리 표시자이며 `next` 메서드의 정의는 `Option Self::Item](self::Item) ` 유형의 값을 반환할 것임을 보여줍니다. `Iterator` 특성의 구현자는 `Item`에 대한 구체적인 유형을 지정하고 `next` 메서드는 해당 구체적인 유형의 값을 포함하는 `Option`을 반환합니다.

연관 유형은 처리할 수 있는 유형을 지정하지 않고 함수를 정의할 수 있다는 점에서 제네릭과 유사한 개념처럼 보일 수 있습니다. 두 개념의 차이점을 살펴보기 위해 `항목` 유형을 `u32`로 지정하는 `카운터`라는 유형의 `반복자` 특성 구현을 살펴보겠습니다.

파일 이름: src/lib.rs

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

이 구문은 제네릭의 구문과 비슷해 보입니다. 그렇다면 Listing 19-13과 같이 제네릭으로 `Iterator` 트레잇을 정의하지 않는 이유는 무엇입니까?

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

Listing 19-13: 제네릭을 사용한 `반복자` 특성의 가상 정의

차이점은 목록 19-13에서와 같이 제네릭을 사용할 때 각 구현의 유형에 주석을 달아야 한다는 것입니다. `반복자`를 구현할 수도 있기 때문입니다.`카운터` 또는 다른 유형에 대해 `카운터`에 대해 `반복자`를 여러 번 구현할 수 있습니다. 매번 제네릭 유형 매개변수 `카운터`에서 `다음` 메소드를 사용할 때 사용할 `반복자`의 구현을 나타내기 위해 유형 주석을 제공해야 합니다.

연관된 유형을 사용하면 유형에 특성을 여러 번 구현할 수 없기 때문에 유형에 주석을 달 필요가 없습니다. 연관 유형을 사용하는 정의가 있는 Listing 19-12에서 `Impl Iterator for Counter`는 하나만 있을 수 있기 때문에 `항목`의 유형이 한 번만 될지 선택할 수 있습니다. 우리는 `카운터`에서 `다음`이라고 부르는 모든 곳에서 `u32` 값의 반복자를 원한다고 지정할 필요가 없습니다.

관련 유형도 특성 계약의 일부가 됩니다. 특성의 구현자는 관련 유형 자리 표시자를 대신할 유형을 제공해야 합니다. 연관된 유형에는 유형이 사용되는 방법을 설명하는 이름이 있는 경우가 많으며 API 설명서에 연관된 유형을 문서화하는 것이 좋습니다.

### [기본 일반 유형 매개변수 및 연산자 오버로딩](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#default-generic-type-parameters-and-operator-overloading)

제네릭 형식 매개 변수를 사용할 때 제네릭 형식에 대한 기본 구체적인 형식을 지정할 수 있습니다. 이렇게 하면 기본 유형이 작동하는 경우 구체적인 유형을 지정하기 위해 특성 구현자가 필요하지 않습니다. `<PlaceholderType=ConcreteType>` 구문을 사용하여 제네릭 형식을 선언할 때 기본 형식을 지정합니다.

이 기술이 유용한 상황의 좋은 예는 특정 상황에서 연산자(예: `+`)의 동작을 사용자 지정하는 연산자 *오버로딩을 사용하는 것입니다.*

Rust는 자신만의 연산자를 만들거나 임의의 연산자를 오버로드하는 것을 허용하지 않습니다. 그러나 연산자와 관련된 특성을 구현하여 `std::ops`에 나열된 작업 및 해당 특성을 오버로드할 수 있습니다. 예를 들어 Listing 19-14에서 `+` 연산자를 오버로드하여 두 개의 `Point` 인스턴스를 함께 추가합니다. `Point` 구조체에 `Add` 특성을 구현하여 이를 수행합니다.

파일 이름: src/main.rs

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

목록 19-14: `Point` 인스턴스에 대한 `+` 연산자를 오버로드하기 위해 `Add` 특성 구현

`add` 메서드는 두 `Point` 인스턴스의 `x` 값과 두 `Point` 인스턴스의 `y` 값을 추가하여 새 `Point`를 만듭니다. `추가` 특성에는 `추가` 메서드에서 반환된 유형을 결정하는 `출력`이라는 관련 유형이 있습니다.

이 코드의 기본 제네릭 형식은 `추가` 특성 내에 있습니다. 정의는 다음과 같습니다.

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

이 코드는 일반적으로 친숙해 보일 것입니다. 하나의 메서드와 관련 유형이 있는 특성입니다. 새 부분은 `Rhs=Self`입니다. 이 구문을 *기본 유형 매개변수* 라고 합니다. `Rhs` 일반 유형 매개변수(`오른쪽`의 줄임말)는 `add` 메소드에서 `rhs` 매개변수의 유형을 정의합니다. `Add` 트레이트를 구현할 때 `Rhs`에 대한 구체적인 유형을 지정하지 않으면 `Rhs` 유형은 `Add`를 구현하는 유형인 `Self`로 기본 설정됩니다.

`Point`에 대해 `Add`를 구현할 때 두 개의 `Point` 인스턴스를 추가하려고 했기 때문에 `Rhs`에 대한 기본값을 사용했습니다. 기본값을 사용하는 대신 `Rhs` 유형을 사용자 지정하려는 `추가` 특성을 구현하는 예를 살펴보겠습니다.

`밀리미터`와 `미터`라는 두 개의 구조체가 있으며 서로 다른 단위로 값을 보유합니다. *다른 구조체에 있는 기존 유형의 이 얇은 래핑은 newtype 패턴* 으로 알려져 있으며 [`Newtype 패턴을 사용하여 외부 유형에 외부 특성 구현`](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types) 섹션 에서 자세히 설명합니다. 밀리미터 값을 미터 값에 추가하고 `추가` 구현이 변환을 올바르게 수행하도록 하려고 합니다. 목록 19-15에 표시된 것처럼 `Rhs`로 `미터`를 사용하여 `밀리미터`에 대해 `추가`를 구현할 수 있습니다.

파일 이름: src/lib.rs

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

목록 19-15: `미터`에 `밀리미터`를 추가하기 위해 `밀리미터`에 `추가` 특성 구현

`밀리미터`와 `미터`를 추가하려면 `impl 추가`를 지정합니다.`를 사용하여 기본값인 `Self`를 사용하는 대신 `Rhs` 유형 매개변수의 값을 설정합니다.

다음 두 가지 주요 방법으로 기본 유형 매개변수를 사용합니다.

- 기존 코드를 중단하지 않고 유형을 확장하려면
- 특정 경우에 사용자 지정을 허용하려면 대부분의 사용자에게 필요하지 않습니다.

표준 라이브러리의 `추가` 특성은 두 번째 목적의 예입니다. 일반적으로 유사한 두 가지 유형을 추가하지만 `추가` 특성은 그 이상을 사용자 지정할 수 있는 기능을 제공합니다. `추가` 특성 정의에서 기본 유형 매개변수를 사용하면 대부분의 경우 추가 매개변수를 지정할 필요가 없습니다. 즉, 약간의 구현 상용구가 필요하지 않으므로 특성을 더 쉽게 사용할 수 있습니다.

첫 번째 목적은 두 번째 목적과 비슷하지만 그 반대입니다. 기존 트레이트에 유형 매개변수를 추가하려는 경우 기존 구현 코드를 중단하지 않고 트레이트의 기능을 확장할 수 있도록 기본값을 지정할 수 있습니다.

### [명확성을 위한 정규화된 구문: 동일한 이름으로 메서드 호출](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name)

Rust의 어떤 것도 특성이 다른 특성의 메서드와 같은 이름을 가진 메서드를 갖는 것을 막지 않으며, Rust는 한 유형에 두 특성을 구현하는 것을 막지 않습니다. 특성의 메서드와 동일한 이름을 가진 형식에 직접 메서드를 구현하는 것도 가능합니다.

같은 이름을 가진 메서드를 호출할 때 어느 것을 사용하고 싶은지 러스트에게 알려줘야 합니다. Listing 19-16에서 `fly`라는 메서드가 있는 `Pilot`과 `Wizard`의 두 가지 특성을 정의한 코드를 고려하십시오. 그런 다음 이미 구현된 `fly`라는 메서드가 있는 `Human` 유형에 두 특성을 모두 구현합니다. 각각의 `비행` 방법은 다른 작업을 수행합니다.

파일 이름: src/main.rs

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!(`This is your captain speaking.`);
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!(`Up!`);
    }
}

impl Human {
    fn fly(&self) {
        println!(`*waving arms furiously*`);
    }
}
```

Listing 19-16: `fly` 메서드를 갖도록 정의된 두 가지 특성이 `Human` 유형에 구현되고 `fly` 메서드가 `Human`에 직접 구현됩니다.

`Human`의 인스턴스에서 `fly`를 호출하면 컴파일러는 기본적으로 Listing 19-17에 표시된 대로 유형에 직접 구현된 메서드를 호출합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let person = Human;
    person.fly();
}
```

Listing 19-17: `Human` 인스턴스에서 `fly` 호출하기

*이 코드를 실행하면 ` waving arms furiously* `가 인쇄되어 Rust가 `Human`에 직접 구현된 `fly` 메서드를 호출했음을 보여줍니다.

`Pilot` 특성 또는 `Wizard` 특성에서 `fly` 메서드를 호출하려면 의미하는 `fly` 메서드를 지정하기 위해 보다 명시적인 구문을 사용해야 합니다. 목록 19-18은 이 구문을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

Listing 19-18: 호출하려는 특성의 `fly` 메서드 지정하기

메서드 이름 앞에 특성 이름을 지정하면 호출하려는 `fly` 구현이 Rust에 명확해집니다. Listing 19-18에서 사용한 `person.fly()`와 동일한 `Human::fly(&person)`을 작성할 수도 있지만, 필요하지 않은 경우 작성하는 데 조금 더 깁니다. 명확하다.

이 코드를 실행하면 다음이 인쇄됩니다.

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running `target/debug/traits-example`
This is your captain speaking.
Up!
*waving arms furiously*
```

`fly` 메서드는 `self` 매개변수를 취하기 때문에, 둘 다 하나의 *특성을 구현하는 두 가지* *유형이* 있는 경우 러스트는 `self` 유형에 따라 사용할 특성 구현을 알아낼 수 있습니다.

그러나 메소드가 아닌 관련 함수에는 `self` 매개변수가 없습니다. *함수 이름이 같은 비메소드 함수를 정의하는 여러 유형이나 특성이 있을 때 Rust는 완전한 구문을* 사용하지 않는 한 어떤 유형을 의미하는지 항상 알 수 없습니다. 예를 들어 목록 19-19에서 우리는 모든 아기 강아지의 이름을 *Spot 으로* 지정하려는 동물 보호소. 메소드가 아닌 함수 `baby_name`과 연결된 `Animal` 특성을 만듭니다. `Animal` 특성은 구조체 `Dog`에 대해 구현되며 관련 메서드가 아닌 함수 `baby_name` 직접.

파일 이름: src/main.rs

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from(`Spot`)
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from(`puppy`)
    }
}

fn main() {
    println!(`A baby dog is called a {}`, Dog::baby_name());
}
```

Listing 19-19: 연관된 함수가 있는 트레이트 및 해당 트레이트를 구현하는 동일한 이름의 연관된 함수가 있는 유형

`Dog`에 정의된 `baby_name` 관련 함수에서 모든 강아지 Spot의 이름을 지정하는 코드를 구현합니다. `Dog` 유형은 또한 모든 동물이 가지고 있는 특성을 설명하는 특성 `Animal`을 구현합니다. 아기 개는 강아지라고 불리며 `Animal` 특성과 연결된 `baby_name` 함수의 `Dog`에 대한 `Animal` 특성 구현으로 표현됩니다.

`main`에서 우리는 `Dog`에 정의된 관련 함수를 직접 호출하는 `Dog::baby_name` 함수를 호출합니다. 이 코드는 다음을 인쇄합니다.

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.54s
     Running `target/debug/traits-example`
A baby dog is called a Spot
```

이 출력은 우리가 원하는 것이 아닙니다. `Dog`에 구현한 `Animal` 특성의 일부인 `baby_name` 함수를 호출하여 코드가 `A baby dog is called a puppy`를 인쇄하도록 합니다. Listing 19-18에서 사용한 특성 이름을 지정하는 기술은 여기서 도움이 되지 않습니다. `main`을 목록 19-20의 코드로 변경하면 컴파일 오류가 발생합니다.

파일 이름: src/main.rs

```rust
fn main() {
    println!(`A baby dog is called a {}`, Animal::baby_name());
}
```

Listing 19-20: `Animal` 트레이트에서 `baby_name` 함수를 호출하려고 시도하지만 Rust는 어떤 구현을 사용해야 할지 모릅니다.

`Animal::baby_name`에는 `self` 매개변수가 없고 `Animal` 특성을 구현하는 다른 유형이 있을 수 있기 때문에 Rust는 우리가 원하는 `Animal::baby_name` 구현을 파악할 수 없습니다. 다음 컴파일러 오류가 발생합니다.

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0790]: cannot call associated function on trait without specifying the corresponding `impl` type
  --> src/main.rs:20:43
   |
2  |     fn baby_name() -> String;
   |     ------------------------- `Animal::baby_name` defined here
...
20 |     println!(`A baby dog is called a {}`, Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^ cannot call associated function of trait
   |
help: use the fully-qualified path to the only available implementation
   |
20 |     println!(`A baby dog is called a {}`, <Dog as Animal>::baby_name());
   |                                           +++++++       +

For more information about this error, try `rustc --explain E0790`.
error: could not compile `traits-example` due to previous error
```

일부 다른 유형에 대한 `Animal` 구현과 반대로 `Dog`에 대한 `Animal` 구현을 사용하고 싶다고 Rust에게 명확하게 알리려면 정규화된 구문을 사용해야 합니다. 목록 19-21은 정규화된 구문을 사용하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    println!(`A baby dog is called a {}`, <Dog as Animal>::baby_name());
}
```

Listing 19-21: `Dog`에 구현된 `Animal` 특성에서 `baby_name` 함수를 호출하도록 지정하기 위해 정규화된 구문 사용

우리는 Rust에 꺾쇠 괄호 안에 유형 주석을 제공하고 있습니다. 이는 `Dog` 유형을 이 함수 호출에 대한 `동물`. 이 코드는 이제 우리가 원하는 것을 인쇄합니다.

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/traits-example`
A baby dog is called a puppy
```

일반적으로 정규화된 구문은 다음과 같이 정의됩니다.

```rust
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

메서드가 아닌 관련 함수의 경우 `수신자`가 없습니다. 다른 인수 목록만 있습니다. 함수나 메서드를 호출하는 모든 곳에서 정규화된 구문을 사용할 수 있습니다. 그러나 Rust가 프로그램의 다른 정보에서 알아낼 수 있는 이 구문의 일부를 생략할 수 있습니다. 동일한 이름을 사용하는 구현이 여러 개 있고 Rust가 호출하려는 구현을 식별하는 데 도움이 필요한 경우에만 이 더 장황한 구문을 사용하면 됩니다.

### [Supertraits를 사용하여 다른 특성 내에서 한 특성의 기능을 요구](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-supertraits-to-require-one-traits-functionality-within-another-trait)

경우에 따라 다른 특성에 의존하는 특성 정의를 작성할 수 있습니다. 첫 번째 특성을 구현하는 유형의 경우 해당 유형이 두 번째 특성도 구현하도록 요구할 수 있습니다. 이렇게 하면 트레이트 정의가 두 번째 트레이트의 관련 항목을 사용할 수 있습니다. 특성 정의가 의존하는 특성을 특성의 *수퍼 특성 이라고 합니다.*

예를 들어, `outline_print` 메소드로 `OutlinePrint` 특성을 만들고 싶다고 가정해 보겠습니다. 이 특성은 별표로 프레임이 지정되도록 형식이 지정된 주어진 값을 인쇄합니다. 즉, 표준 라이브러리를 구현하는 `Point` 구조체가 제공됩니다. `X`에 `1`, `y`에 `3`이 있는 `Point` 인스턴스에서 `outline_print`를 호출하면 다음과 같이 인쇄되어야 합니다. :

```
**********
*        *
* (1, 3) *
*        *
**********
```

`outline_print` 메서드 구현에서 `Display` 특성의 기능을 사용하려고 합니다. 따라서 `OutlinePrint` 특성이 `Display`도 구현하고 `OutlinePrint`에 필요한 기능을 제공하는 유형에 대해서만 작동하도록 지정해야 합니다. `OutlinePrint: Display`를 지정하여 특성 정의에서 이를 수행할 수 있습니다. 이 기술은 특성에 결합된 특성을 추가하는 것과 유사합니다. 목록 19-22는 `OutlinePrint` 특성의 구현을 보여줍니다.

파일 이름: src/main.rs

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!(`{}`, `*`.repeat(len + 4));
        println!(`*{}*`, ` `.repeat(len + 2));
        println!(`* {} *`, output);
        println!(`*{}*`, ` `.repeat(len + 2));
        println!(`{}`, `*`.repeat(len + 4));
    }
}
```

Listing 19-22: `Display`의 기능을 필요로 하는 `OutlinePrint` 특성 구현

`OutlinePrint`에 `Display` 특성이 필요하다고 지정했기 때문에 `Display`를 구현하는 모든 유형에 대해 자동으로 구현되는 `to_string` 함수를 사용할 수 있습니다. 콜론을 추가하지 않고 특성 이름 뒤에 `Display` 특성을 지정하지 않고 `to_string`을 사용하려고 하면 현재 범위에서 `&Self` 유형에 대해 `to_string`이라는 이름의 메서드를 찾을 수 없다는 오류가 발생합니다. .

`Point` 구조체와 같이 `Display`를 구현하지 않는 유형에서 `OutlinePrint`를 구현하려고 할 때 어떤 일이 발생하는지 살펴보겠습니다.

파일 이름: src/main.rs

```rust
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}
```

`디스플레이`가 필요하지만 구현되지 않았다는 오류가 발생합니다.

```bash
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0277]: `Point` doesn`t implement `std::fmt::Display`
  --> src/main.rs:20:6
   |
20 | impl OutlinePrint for Point {}
   |      ^^^^^^^^^^^^ `Point` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Point`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
note: required by a bound in `OutlinePrint`
  --> src/main.rs:3:21
   |
3  | trait OutlinePrint: fmt::Display {
   |                     ^^^^^^^^^^^^ required by this bound in `OutlinePrint`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `traits-example` due to previous error
```

이 문제를 해결하기 위해 `Point`에 `Display`를 구현하고 `OutlinePrint`에 필요한 제약 조건을 다음과 같이 충족합니다.

파일 이름: src/main.rs

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, `({}, {})`, self.x, self.y)
    }
}
```

그런 다음 `Point`에서 `OutlinePrint` 특성을 구현하면 성공적으로 컴파일되고 `Point` 인스턴스에서 `outline_print`를 호출하여 별표 윤곽선 안에 표시할 수 있습니다.

### [Newtype 패턴을 사용하여 외부 유형에 외부 특성 구현](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types)

10장 [`유형에 대한 특성 구현`](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type) 섹션에서 우리는 특성이나 유형이 크레이트에 로컬인 경우에만 유형에 특성을 구현할 수 있다는 고아 규칙을 언급했습니다. 튜플 구조체에서 새 유형을 생성하는 것과 관련된 *newtype 패턴을* 사용하여 이 제한을 우회하는 것이 가능합니다. [(튜플 구조체는 5장의 `명명된 필드 없이 튜플 구조체를 사용하여 다른 유형 생성`](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types) 섹션 에서 다루었 습니다.) 튜플 구조체는 하나의 필드를 가지며 특성을 구현하려는 유형을 감싸는 얇은 래퍼가 됩니다. 그런 다음 래퍼 유형은 크레이트에 로컬이며 래퍼에서 특성을 구현할 수 있습니다. *뉴타입*Haskell 프로그래밍 언어에서 유래한 용어입니다. 이 패턴을 사용해도 런타임 성능이 저하되지 않으며 래퍼 유형이 컴파일 시간에 제거됩니다.

예를 들어 `Vec`에 `Display`를 구현하고 싶다고 가정해 보겠습니다.`, 고아 규칙은 `Display` 특성과 `Vec` 유형은 크레이트 외부에서 정의됩니다. `Vec` 인스턴스를 보유하는 `Wrapper` 구조체를 만들 수 있습니다.`; 그런 다음 `Wrapper`에 `Display`를 구현하고 `Vec` 값을 Listing 19-23에서 볼 수 있습니다.

파일 이름: src/main.rs

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, `[{}]`, self.0.join(`, `))
    }
}

fn main() {
    let w = Wrapper(vec![String::from(`hello`), String::from(`world`)]);
    println!(`w = {}`, w);
}
```

Listing 19-23: `Vec` 주위에 `Wrapper` 유형 만들기` `디스플레이` 구현

`디스플레이`의 구현은 `self.0`을 사용하여 내부 `Vec`에 액세스합니다.`, `Wrapper`는 튜플 구조체이고 `Vec`는 튜플에서 인덱스 0에 있는 항목입니다. 그러면 `래퍼`에서 `디스플레이` 유형의 기능을 사용할 수 있습니다.

이 기술을 사용할 때의 단점은 `Wrapper`가 새로운 유형이므로 보유한 값의 메소드가 없다는 것입니다. `Vec`의 모든 메서드를 구현해야 합니다.` 메서드가 `self.0`에 위임되도록 `Wrapper`에 직접 `Wrapper`를 `Vec`처럼 취급할 수 있습니다.`. 새 유형이 내부 유형이 가진 모든 메소드를 가지도록 하려면 `Deref` 특성(15장 `` [Deref` 특성이 있는 일반 참조와 같은 스마트 포인터 처리`](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait) 섹션에서 설명)을 `래퍼`에 구현합니다. ` 내부 유형을 반환하는 것이 해결책이 될 것입니다. 예를 들어 `Wrapper` 유형의 동작을 제한하기 위해 `Wrapper` 유형이 내부 유형의 모든 메소드를 가지지 않도록 하려면 다음과 같이 구현해야 합니다. 수동으로 원하는 방법.

이 뉴타입 패턴은 특성이 관련되지 않은 경우에도 유용합니다. 초점을 전환하고 Rust의 유형 시스템과 상호 작용하는 몇 가지 고급 방법을 살펴보겠습니다.

------

## [고급 유형](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#advanced-types)

Rust 유형 시스템에는 지금까지 언급했지만 아직 논의하지 않은 몇 가지 기능이 있습니다. 뉴타입이 유형으로서 유용한 이유를 조사하면서 일반적으로 뉴타입에 대해 논의하는 것으로 시작할 것입니다. 그런 다음 newtypes와 유사하지만 의미 체계가 약간 다른 기능인 유형 별칭으로 이동합니다. 우리는 또한 `!`에 대해 논의할 것입니다. 유형 및 동적 크기 유형.

### [유형 안전 및 추상화를 위한 Newtype 패턴 사용](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#using-the-newtype-pattern-for-type-safety-and-abstraction)

> [참고: 이 섹션에서는 이전 섹션인 `Newtype 패턴을 사용하여 외부 유형에 외부 특성 구현`을](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types) 읽었다고 가정합니다.

newtype 패턴은 값이 절대 혼동되지 않도록 정적으로 적용하고 값의 단위를 표시하는 등 지금까지 논의한 작업 이외의 작업에도 유용합니다. Listing 19-15에서 단위를 나타내기 위해 newtypes를 사용하는 예를 보았습니다. `밀리미터` 유형의 매개변수를 사용하여 함수를 작성하면 실수로 `미터` 또는 일반 `u32` 유형의 값으로 해당 함수를 호출하려고 시도하는 프로그램을 컴파일할 수 없습니다.

또한 newtype 패턴을 사용하여 유형의 일부 구현 세부 정보를 추상화할 수 있습니다. 새 유형은 비공개 내부 유형의 API와 다른 공개 API를 노출할 수 있습니다.

뉴타입은 또한 내부 구현을 숨길 수 있습니다. 예를 들어 이름과 연결된 사람의 ID를 저장하는 `HashMap<i32, String>`을 래핑하기 위해 `People` 유형을 제공할 수 있습니다. `People`을 사용하는 코드는 `People` 컬렉션에 이름 문자열을 추가하는 방법과 같이 우리가 제공하는 공개 API와만 상호 작용합니다. 해당 코드는 내부적으로 이름에 `i32` ID를 할당한다는 사실을 알 필요가 없습니다. 뉴타입 패턴은 17장의 [`구현 세부 사항을 숨기는 캡슐화`](https://doc.rust-lang.org/book/ch17-01-what-is-oo.html#encapsulation-that-hides-implementation-details) 섹션 에서 논의한 구현 세부 사항을 숨기기 위해 캡슐화를 달성하는 가벼운 방법입니다.

### [유형 별칭으로 유형 동의어 만들기](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases)

Rust는 기존 유형에 다른 이름을 부여하기 위해 *유형 별칭을* 선언하는 기능을 제공합니다. 이를 위해 `type` 키워드를 사용합니다. 예를 들어 다음과 같이 `Kilometers`라는 별칭을 `i32`로 만들 수 있습니다.

```rust
    type Kilometers = i32;
```

이제 `Kilometers`라는 별칭은 `i32`와 *동의어 입니다.* 목록 19-15에서 생성한 `밀리미터` 및 `미터` 유형과 달리 `킬로미터`는 별도의 새로운 유형이 아닙니다. `Kilometers` 유형의 값은 `i32` 유형의 값과 동일하게 처리됩니다.

```rust
    type Kilometers = i32;

    let x: i32 = 5;
    let y: Kilometers = 5;
    
    println!(`x + y = {}`, x + y);
```

`Kilometers`와 `i32`는 동일한 유형이므로 두 유형의 값을 모두 추가할 수 있으며 `Kilometers` 값을 `i32` 매개변수를 사용하는 함수에 전달할 수 있습니다. 그러나 이 방법을 사용하면 이전에 논의한 newtype 패턴에서 얻은 유형 검사 이점을 얻지 못합니다. 즉, 어딘가에서 `Kilometers`와 `i32` 값을 섞어도 컴파일러에서 오류가 발생하지 않습니다.

유형 동의어의 주요 사용 사례는 반복을 줄이는 것입니다. 예를 들어 다음과 같은 긴 유형이 있을 수 있습니다.

```rust
Box<dyn Fn() + Send + `static>
```

이 긴 유형을 함수 서명에 작성하고 코드 전체에 유형 주석을 작성하는 것은 번거롭고 오류가 발생하기 쉽습니다. 목록 19-24와 같은 코드로 가득 찬 프로젝트가 있다고 상상해 보십시오.

```rust
    let f: Box<dyn Fn() + Send + `static> = Box::new(|| println!(`hi`));

    fn takes_long_type(f: Box<dyn Fn() + Send + `static>) {
        // --snip--
    }
    
    fn returns_long_type() -> Box<dyn Fn() + Send + `static> {
        // --snip--
    }
```

Listing 19-24: 여러 곳에서 long 유형 사용하기

유형 별칭은 반복을 줄여서 이 코드를 더 관리하기 쉽게 만듭니다. Listing 19-25에서 우리는 verbose 타입에 대해 `Thunk`라는 별칭을 도입했으며 이 유형의 모든 사용을 더 짧은 별칭인 `Thunk`로 대체할 수 있습니다.

```rust
    type Thunk = Box<dyn Fn() + Send + `static>;

    let f: Thunk = Box::new(|| println!(`hi`));
    
    fn takes_long_type(f: Thunk) {
        // --snip--
    }
    
    fn returns_long_type() -> Thunk {
        // --snip--
    }
```

목록 19-25: 반복을 줄이기 위해 유형 별칭 `Thunk` 도입

이 코드는 읽고 쓰기가 훨씬 쉽습니다! 유형 별칭에 의미 있는 이름을 선택하면 의도를 전달하는 데 도움이 될 수 있습니다( *thunk* 는 나중에 평가할 코드의 단어이므로 저장되는 클로저에 적합한 이름입니다).

유형 별칭은 반복을 줄이기 위해 일반적으로 `Result<T, E>` 유형과 함께 사용됩니다. 표준 라이브러리의 `std::io` 모듈을 고려하십시오. I/O 작업은 종종 `Result<T, E>`를 반환하여 작업이 작동하지 않는 상황을 처리합니다. 이 라이브러리에는 가능한 모든 I/O 오류를 나타내는 `std::io::Error` 구조체가 있습니다. `std::io`의 많은 함수는 `E`가 `std::io::Error`인 `Result<T, E>`를 반환합니다. 예를 들어 `Write` 특성의 다음 함수와 같습니다.

```rust
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

`결과<..., 오류>`가 많이 반복됩니다. 따라서 `std::io`에는 다음 유형 별칭 선언이 있습니다.

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

이 선언은 `std::io` 모듈에 있으므로 정규화된 별칭 `std::io::Result`를 사용할 수 있습니다.`; 즉, `E`가 `std::io::Error`로 채워진 `Result<T, E>`입니다. `Write` 특성 함수 시그니처는 다음과 같이 표시됩니다.

```rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```

*유형 별칭은 두 가지 방식으로 도움 이* 됩니다. 코드 작성을 더 쉽게 만들고 모든 `std::io`에서 일관된 인터페이스를 제공합니다. 이는 별칭이기 때문에 또 다른 `Result<T, E>`일 뿐이며 `Result<T, E>`에서 작동하는 모든 메서드와 `?` 운영자.

### [돌아오지 않는 Never Type](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#the-never-type-that-never-returns)

Rust에는 `!`라는 특별한 유형이 있습니다. 값이 없기 때문에 유형 이론 용어로는 *빈 유형* 으로 알려져 있습니다. *우리는 함수가 절대* 반환하지 않을 때 반환 유형을 대신하기 때문에 never 유형 이라고 부르는 것을 선호합니다. 다음은 예입니다.

```rust
fn bar() -> ! {
    // --snip--
}
```

이 코드는 ``bar` 함수가 절대 반환하지 않음`으로 읽힙니다. *절대 반환하지 않는 함수를 발산 함수* 라고 합니다. `!` 유형의 값을 만들 수 없습니다. 따라서 `바`는 절대 반환할 수 없습니다.

그러나 절대 값을 생성할 수 없는 유형이 무슨 소용이 있습니까? 숫자 추측 게임의 일부인 목록 2-5의 코드를 상기하십시오. 우리는 Listing 19-26에서 그것의 일부를 재생산했습니다.

```rust
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
```

Listing 19-26: `continue`로 끝나는 arm이 있는 `match`

당시 우리는 이 코드의 일부 세부 사항을 건너뛰었습니다. 6장 [``일치` 제어 흐름 연산자`](https://doc.rust-lang.org/book/ch06-02-match.html#the-match-control-flow-operator) 섹션에서 `일치` 항목은 모두 동일한 유형을 반환해야 한다고 설명했습니다. 예를 들어 다음 코드는 작동하지 않습니다.

```rust
    let guess = match guess.trim().parse() {
        Ok(_) => 5,
        Err(_) => `hello`,
    };
```

*이 코드에서 `guess`의 유형은 정수 와* 문자열 이어야 하고 Rust는 `guess`가 한 가지 유형만 가질 것을 요구합니다. 그렇다면 `계속`은 무엇을 반환합니까? 목록 19-26에서 어떻게 한 팔에서 `u32`를 반환하고 `계속`으로 끝나는 다른 팔을 가질 수 있었습니까?

짐작하셨겠지만 `계속`에는 `!`가 있습니다. 값. 즉, Rust가 `guess`의 유형을 계산할 때 두 매치 암을 살펴봅니다. 전자는 `u32` 값을, 후자는 `!` 값. 왜냐하면 `!` 값을 가질 수 없는 경우 Rust는 `guess` 유형을 `u32`로 결정합니다.

이 동작을 설명하는 공식적인 방법은 `!` 유형의 표현식입니다. 다른 유형으로 강제 변환될 수 있습니다. `계속`은 값을 반환하지 않기 때문에 `계속`으로 이 `일치` 암을 종료할 수 있습니다. 대신 루프의 맨 위로 제어를 이동하므로 `Err`의 경우 `guess`에 값을 할당하지 않습니다.

never 유형은 `panic!`과 함께 유용합니다. 매크로도. `옵션`에서 호출하는 `unwrap` 함수를 기억하십시오.` 이 정의를 사용하여 값 또는 패닉을 생성하는 값:

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!(`called `Option::unwrap()` on a `None` value`),
        }
    }
}
```

이 코드에서 Listing 19-26의 `match`에서와 같은 일이 발생합니다. Rust는 `val`에 `T` 유형이 있고 `panic!` 유형이 `!`이므로 전체 `일치` 표현식의 결과는 `T`입니다. 이 코드는 `패닉!` 때문에 작동합니다. 값을 생성하지 않습니다. 프로그램을 종료합니다. `None`의 경우 `unwrap`에서 값을 반환하지 않으므로 이 코드는 유효합니다.

유형이 `!`인 마지막 표현식 `루프`입니다.

```rust
    print!(`forever `);

    loop {
        print!(`and ever `);
    }
```

여기서는 루프가 끝나지 않으므로 `!` 표현의 값입니다. 그러나 `break`를 포함하면 루프가 `break`에 도달하면 종료되기 때문에 이것은 사실이 아닙니다.

### [동적으로 크기가 조정된 유형 및 `크기 조정` 특성](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait)

Rust는 특정 유형의 값에 할당할 공간과 같은 유형에 대한 특정 세부 정보를 알아야 합니다. 이로 인해 유형 시스템의 한 구석이 처음에는 약간 혼란스러워집니다. 즉, *동적으로 크기가 조정되는 유형* 의 개념입니다. *DST* 또는 *크기가 지정되지 않은 유형* 이라고도 하는 이러한 유형을 사용하면 런타임에만 크기를 알 수 있는 값을 사용하여 코드를 작성할 수 있습니다.

책 전체에서 사용한 `str`이라는 동적으로 크기가 조정되는 유형에 대해 자세히 살펴보겠습니다. 맞습니다. `&str`이 아니라 `str` 자체가 DST입니다. 문자열이 런타임까지 얼마나 걸리는지 알 수 없습니다. 즉, `str` 유형의 변수를 생성할 수 없으며 `str` 유형의 인수를 사용할 수도 없습니다. 작동하지 않는 다음 코드를 고려하십시오.

```rust
    let s1: str = `Hello there!`;
    let s2: str = `How`s it going?`;
```

Rust는 특정 유형의 값에 할당할 메모리 양을 알아야 하며 유형의 모든 값은 동일한 양의 메모리를 사용해야 합니다. Rust가 이 코드를 작성할 수 있도록 허용했다면 이 두 `str` 값은 같은 양의 공간을 차지해야 합니다. 그러나 길이가 다릅니다. `s1`은 12바이트의 저장 공간이 필요하고 `s2`는 15바이트가 필요합니다. 이것이 동적으로 크기가 조정된 유형을 보유하는 변수를 생성할 수 없는 이유입니다.

그래서 우리는 무엇을 해야 합니까? 이 경우, 당신은 이미 답을 알고 있습니다: 우리는 `s1`과 `s2`의 타입을 `str`이 아닌 `&str`로 만듭니다. [4장의 `스트링 슬라이스`](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices) 섹션 에서 슬라이스 데이터 구조는 슬라이스의 시작 위치와 길이만 저장한다는 점을 상기하십시오. 따라서 `&T`는 `T`가 있는 메모리 주소를 저장하는 단일 값이지만 `&str`은 *2개 입니다.*값: `str`의 주소와 길이. 따라서 컴파일 시간에 `&str` 값의 크기를 알 수 있습니다. `usize` 길이의 두 배입니다. 즉, 참조하는 문자열의 길이에 관계없이 `&str`의 크기를 항상 알고 있습니다. 일반적으로 이것은 Rust에서 동적으로 크기가 조정된 유형이 사용되는 방식입니다. 유형에는 동적 정보의 크기를 저장하는 추가 메타데이터가 있습니다. 동적으로 크기가 조정되는 유형의 황금률은 항상 동적으로 크기가 조정되는 유형의 값을 어떤 종류의 포인터 뒤에 두어야 한다는 것입니다.

`str`을 모든 종류의 포인터와 결합할 수 있습니다. 예를 들어 `Box` 또는 `Rc[`. 사실, 이전에 본 적이 있지만 동적으로 크기가 조정되는 다른 유형인 특성이 있습니다. 모든 특성은 특성](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) 의 이름을 사용하여 참조할 수 있는 동적으로 크기가 조정되는 유형입니다. [다른 유형의 값 허용”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) 섹션에서 특성을 특성 개체로 사용하려면 `&dyn Trait` 또는 `Box`와 같은 포인터 뒤에 배치해야 한다고 언급했습니다.` (`RC`도 작동합니다).

DST와 함께 작업하기 위해 Rust는 컴파일 시간에 유형의 크기를 알 수 있는지 여부를 결정하는 `Sized` 특성을 제공합니다. 이 특성은 컴파일 시간에 크기가 알려진 모든 것에 대해 자동으로 구현됩니다. 또한 Rust는 모든 일반 함수에 `Sized`에 대한 경계를 암묵적으로 추가합니다. 즉, 다음과 같은 일반적인 함수 정의입니다.

```rust
fn generic<T>(t: T) {
    // --snip--
}
```

실제로 다음과 같이 작성한 것처럼 처리됩니다.

```rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

기본적으로 제네릭 함수는 컴파일 타임에 크기가 알려진 유형에서만 작동합니다. 그러나 다음 특수 구문을 사용하여 이 제한을 완화할 수 있습니다.

```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

`?Sized`에 바인딩된 특성은 ``T`가 `Sized`일 수도 있고 아닐 수도 있음을 의미합니다. 이 표기법은 일반 유형이 컴파일 시간에 알려진 크기를 가져야 한다는 기본값을 재정의합니다. 이 의미를 가진 `?Trait` 구문은 `Sized`에만 사용할 수 있으며 다른 특성에는 사용할 수 없습니다.

또한 `t` 매개변수의 유형을 `T`에서 `&T`로 전환했습니다. 유형이 `크기 조정`되지 않을 수 있기 때문에 일종의 포인터 뒤에 유형을 사용해야 합니다. 이 경우 참조를 선택했습니다.

다음으로 함수와 클로저에 대해 이야기하겠습니다!

------

## [고급 기능 및 클로저](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html#advanced-functions-and-closures)

이 섹션에서는 함수 포인터 및 반환 클로저를 포함하여 함수 및 클로저와 관련된 일부 고급 기능을 살펴봅니다.

### [함수 포인터](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html#function-pointers)

클로저를 함수에 전달하는 방법에 대해 이야기했습니다. 일반 함수를 함수에 전달할 수도 있습니다! 이 기술은 새 클로저를 정의하는 대신 이미 정의한 함수를 전달하려는 경우에 유용합니다. 함수는 `Fn` 클로저 특성과 혼동하지 않도록 `fn`(소문자 f 포함) 유형으로 강제 변환합니다. `fn` 유형을 *함수 포인터* 라고 합니다. 함수 포인터와 함께 함수를 전달하면 함수를 다른 함수의 인수로 사용할 수 있습니다.

매개변수가 함수 포인터임을 지정하는 구문은 목록 19-27에 표시된 것처럼 클로저의 구문과 유사합니다. 여기에서 매개변수에 1을 더하는 `add_one` 함수를 정의했습니다. 함수 `do_twice`는 `i32` 매개변수를 취하고 `i32`를 반환하는 함수에 대한 함수 포인터와 `i32 값`이라는 두 개의 매개변수를 사용합니다. `do_twice` 함수는 `f` 함수를 두 번 호출하여 `arg` 값을 전달한 다음 두 함수 호출 결과를 함께 더합니다. `main` 함수는 `add_one` 및 `5` 인수를 사용하여 `do_twice`를 호출합니다.

파일 이름: src/main.rs

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!(`The answer is: {}`, answer);
}
```

목록 19-27: `fn` 유형을 사용하여 함수 포인터를 인수로 수락

이 코드는 `답은 12`를 출력합니다. 우리는 `do_twice`의 매개변수 `f`가 `i32` 유형의 매개변수 하나를 취하고 `i32`를 반환하는 `fn`이라고 지정합니다. 그런 다음 `do_twice`의 본문에서 `f`를 호출할 수 있습니다. `main`에서 함수 이름 `add_one`을 `do_twice`의 첫 번째 인수로 전달할 수 있습니다.

클로저와 달리 `fn`은 특성이 아닌 유형이므로 `Fn` 특성 중 하나를 특성 바인딩으로 사용하여 제네릭 유형 매개변수를 선언하는 대신 `fn`을 매개변수 유형으로 직접 지정합니다.

함수 포인터는 세 가지 클로저 특성(`Fn`, `FnMut` 및 `FnOnce`)을 모두 구현합니다. 즉, 클로저를 예상하는 함수의 인수로 함수 포인터를 항상 전달할 수 있습니다. 함수가 함수 또는 클로저를 허용할 수 있도록 제네릭 유형과 클로저 특성 중 하나를 사용하여 함수를 작성하는 것이 가장 좋습니다.

즉, 클로저가 아닌 `fn`만 허용하려는 경우의 한 가지 예는 클로저가 없는 외부 코드와 인터페이스할 때입니다. C 함수는 함수를 인수로 허용할 수 있지만 C에는 클로저가 없습니다.

인라인으로 정의된 클로저 또는 명명된 함수를 사용할 수 있는 경우의 예로 표준 라이브러리의 `Iterator` 특성에서 제공하는 `map` 메서드의 사용을 살펴보겠습니다. `map` 함수를 사용하여 숫자 벡터를 문자열 벡터로 바꾸려면 다음과 같이 클로저를 사용할 수 있습니다.

```rust
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
```

또는 다음과 같이 클로저 대신 `map`에 대한 인수로 함수의 이름을 지정할 수 있습니다.

```rust
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
```

`to_string`이라는 이름으로 사용할 수 있는 여러 함수가 있으므로 [`고급 특성`](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#advanced-traits) 섹션 에서 앞서 언급한 정규화된 구문을 사용해야 합니다. 여기서 우리는 표준 라이브러리가 `Display`를 구현하는 모든 유형에 대해 구현한 `ToString` 특성에 정의된 `to_string` 함수를 사용하고 있습니다.

[6장의 `열거형 값`](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values) 섹션에서 우리가 정의하는 각 열거형 변형의 이름도 초기화 함수가 된다는 점을 상기하십시오 . 이 초기화 함수를 클로저 특성을 구현하는 함수 포인터로 사용할 수 있습니다. 즉, 다음과 같이 클로저를 취하는 메서드의 인수로 초기화 함수를 지정할 수 있습니다.

```rust
    enum Status {
        Value(u32),
        Stop,
    }

    let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```

여기에서 `Status::Value`의 초기화 함수를 사용하여 `map`이 호출되는 범위의 각 `u32` 값을 사용하여 `Status::Value` 인스턴스를 생성합니다. 이 스타일을 선호하는 사람도 있고 클로저를 선호하는 사람도 있습니다. 동일한 코드로 컴파일되므로 더 명확한 스타일을 사용하십시오.

### [반환 폐쇄](https://doc.rust-lang.org/book/ch19-05-advanced-functions-and-closures.html#returning-closures)

클로저는 특성으로 표현되며 이는 클로저를 직접 반환할 수 없음을 의미합니다. 트레이트를 반환하려는 대부분의 경우 트레이트를 구현하는 구체적인 유형을 함수의 반환 값으로 대신 사용할 수 있습니다. 그러나 반환 가능한 구체적인 유형이 없기 때문에 클로저로는 그렇게 할 수 없습니다. 예를 들어 함수 포인터 `fn`을 반환 유형으로 사용할 수 없습니다.

다음 코드는 클로저를 직접 반환하려고 시도하지만 컴파일되지는 않습니다.

```rust
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}
```

컴파일러 오류는 다음과 같습니다.

```bash
$ cargo build
   Compiling functions-example v0.1.0 (file:///projects/functions-example)
error[E0746]: return type cannot have an unboxed trait object
 --> src/lib.rs:1:25
  |
1 | fn returns_closure() -> dyn Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^^^^^ doesn`t have a size known at compile-time
  |
  = note: for information on `impl Trait`, see <https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits>
help: use `impl Fn(i32) -> i32` as the return type, as all return paths are of type `[closure@src/lib.rs:2:5: 2:8]`, which implements `Fn(i32) -> i32`
  |
1 | fn returns_closure() -> impl Fn(i32) -> i32 {
  |                         ~~~~~~~~~~~~~~~~~~~

For more information about this error, try `rustc --explain E0746`.
error: could not compile `functions-example` due to previous error
```

오류는 `Sized` 특성을 다시 참조합니다! Rust는 클로저를 저장하는 데 얼마나 많은 공간이 필요한지 모릅니다. 우리는 앞서 이 문제에 대한 해결책을 보았습니다. 특성 개체를 사용할 수 있습니다.

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

이 코드는 잘 컴파일됩니다. 특성 개체에 대한 자세한 내용은 17장의 [`다른 유형의 값을 허용하는 특성 개체 사용`](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) 섹션을 참조하세요 .

다음으로 매크로를 살펴보겠습니다!

------

## [매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#macros)

우리는 `println!`과 같은 매크로를 사용했습니다. 이 책 전체에서 매크로가 무엇이고 어떻게 작동하는지 완전히 탐구하지는 않았습니다. *매크로* 라는 용어는 `macro_rules!`가 있는 *선언적* 매크로 와 같은 Rust의 기능 계열을 나타냅니다. 세 가지 종류의 *절차* 적 매크로:

- 구조체 및 열거형에 사용되는 `derive` 특성으로 추가된 코드를 지정하는 사용자 지정 `#[derive]` 매크로
- 모든 항목에서 사용할 수 있는 사용자 정의 속성을 정의하는 속성 유사 매크로
- 함수 호출처럼 보이지만 인수로 지정된 토큰에서 작동하는 함수형 매크로

이들 각각에 대해 차례로 이야기하겠지만 먼저 이미 함수가 있는데 왜 매크로가 필요한지 살펴보겠습니다.

### [매크로와 함수의 차이점](https://doc.rust-lang.org/book/ch19-06-macros.html#the-difference-between-macros-and-functions)

기본적으로 매크로는 다른 코드를 작성하는 코드를 작성하는 방법으로, 이를 *메타프로그래밍* 이라고 합니다. 부록 C에서는 다양한 트레이트 구현을 생성하는 `파생` 특성에 대해 설명합니다. 우리는 또한 `println!` 그리고 `벡!` 책 전체의 매크로. 이러한 모든 매크로는 *확장되어* 수동으로 작성한 코드보다 더 많은 코드를 생성합니다.

메타 프로그래밍은 작성하고 유지해야 하는 코드의 양을 줄이는 데 유용하며, 이는 함수의 역할 중 하나이기도 합니다. 그러나 매크로에는 기능에는 없는 몇 가지 추가 기능이 있습니다.

함수 시그니처는 함수가 가지고 있는 매개변수의 수와 유형을 선언해야 합니다. 반면에 매크로는 가변 개수의 매개변수를 사용할 수 있습니다. 하나의 인수로 `println!(`hello`)`를 호출하거나 두 개의 인수로 `println!(`hello {}`, name)`을 호출할 수 있습니다. 또한 매크로는 컴파일러가 코드의 의미를 해석하기 전에 확장되므로 매크로는 예를 들어 주어진 유형에 특성을 구현할 수 있습니다. 함수는 실행 시간에 호출되고 특성은 컴파일 시간에 구현되어야 하기 때문에 불가능합니다.

함수 대신 매크로를 구현하는 단점은 Rust 코드를 작성하는 Rust 코드를 작성하기 때문에 매크로 정의가 함수 정의보다 더 복잡하다는 것입니다. 이러한 간접 참조로 인해 매크로 정의는 일반적으로 함수 정의보다 읽고 이해하고 유지하기가 더 어렵습니다.

매크로와 함수의 또 다른 중요한 차이점 은 어디에서나 정의하고 호출할 수 있는 함수와 달리 파일에서 매크로를 호출하기 *전에* 매크로를 정의하거나 범위로 가져와야 한다는 것입니다.

### [`macro_rules!`가 포함된 선언적 매크로 일반 메타프로그래밍용](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming)

Rust에서 가장 널리 사용되는 매크로 형식은 *선언적 매크로 입니다.*. 이들은 또한 `예제에 의한 매크로`, ``macro_rules!` 매크로` 또는 그냥 일반 `매크로`입니다. 본질적으로 선언적 매크로는 Rust `일치` 식과 유사한 것을 작성할 수 있도록 합니다. 6장에서 설명한 것처럼 `일치` 식은 식을 가져와서 식의 결과 값을 패턴과 비교한 다음 일치하는 패턴과 관련된 코드를 실행하는 제어 구조입니다. 매크로는 또한 값을 특정 코드와 관련된 패턴과 비교합니다. 이 상황에서 값은 매크로에 전달된 리터럴 Rust 소스 코드입니다. 패턴은 해당 소스 코드의 구조와 비교됩니다. 각 패턴과 관련된 코드는 일치할 때 매크로에 전달된 코드를 대체합니다. 이 모든 것은 컴파일 중에 발생합니다.

매크로를 정의하려면 `macro_rules!` 건설하다. `macro_rules!`를 사용하는 방법을 살펴보겠습니다. `vec!` 매크로가 정의됩니다. 8장에서는 `vec!` 매크로를 사용하여 특정 값으로 새 벡터를 만듭니다. 예를 들어 다음 매크로는 세 개의 정수를 포함하는 새 벡터를 만듭니다.

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

`vec!` 두 개의 정수로 구성된 벡터 또는 다섯 개의 스트링 슬라이스로 구성된 벡터를 만드는 매크로입니다. 값의 수나 유형을 미리 알 수 없기 때문에 함수를 사용하여 동일한 작업을 수행할 수 없습니다.

목록 19-28은 `vec!`의 약간 단순화된 정의를 보여줍니다. 매크로.

파일 이름: src/lib.rs

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

목록 19-28: `vec!` 매크로 정의

> 참고: `vec!` 표준 라이브러리의 매크로에는 올바른 양의 메모리를 미리 할당하는 코드가 포함되어 있습니다. 해당 코드는 예제를 더 간단하게 만들기 위해 여기에 포함하지 않는 최적화입니다.

`#[macro_export]` 주석은 매크로가 정의된 상자가 범위에 들어올 때마다 이 매크로를 사용할 수 있어야 함을 나타냅니다. 이 주석이 없으면 매크로를 범위로 가져올 수 없습니다.

그런 다음 `macro_rules!`로 매크로 정의를 시작합니다. 느낌표 *없이* 정의할 매크로의 이름입니다. 이름(이 경우 `vec`) 뒤에는 매크로 정의의 본문을 나타내는 중괄호가 옵니다.

`vec!` 본문은 `일치` 식의 구조와 비슷합니다. 여기에 `( $( $x:expr ),* )` 패턴과 `=>` 및 이 패턴과 관련된 코드 블록이 있는 하나의 팔이 있습니다. 패턴이 일치하면 연결된 코드 블록이 방출됩니다. 이것이 이 매크로의 유일한 패턴이라는 점을 감안할 때 유효한 일치 방법은 하나뿐입니다. 다른 패턴은 오류가 발생합니다. 더 복잡한 매크로에는 둘 이상의 팔이 있습니다.

매크로 정의의 유효한 패턴 구문은 18장에서 다룬 패턴 구문과 다릅니다. 매크로 패턴은 값이 아닌 Rust 코드 구조와 일치하기 때문입니다. 목록 19-28의 패턴 조각이 무엇을 의미하는지 살펴보겠습니다. 전체 매크로 패턴 구문은 [Rust 참조 문서를](https://doc.rust-lang.org/reference/macros-by-example.html) 참조하세요 .

먼저 전체 패턴을 포함하기 위해 일련의 괄호를 사용합니다. 패턴과 일치하는 Rust 코드를 포함할 변수를 매크로 시스템에서 선언하기 위해 달러 기호(`$`)를 사용합니다. 달러 기호는 이것이 일반 Rust 변수가 아닌 매크로 변수임을 분명히 합니다. 다음으로 대체 코드에서 사용하기 위해 괄호 안의 패턴과 일치하는 값을 캡처하는 일련의 괄호가 나옵니다. `$()` 안에는 `$x:expr`이 있는데, 이는 Rust 표현식과 일치하고 표현식에 `$x`라는 이름을 부여합니다.

`$()` 다음의 쉼표는 리터럴 쉼표 구분 문자가 `$()`의 코드와 일치하는 코드 뒤에 선택적으로 나타날 수 있음을 나타냅니다. ` *`는 패턴이 `* ` 앞에 있는 것과 0개 이상 일치함을 지정합니다.

`vec![1, 2, 3];`로 이 매크로를 호출하면 `$x` 패턴이 `1`, `2` 및 `3` 세 개의 표현식과 세 번 일치합니다.

이제 이 팔과 관련된 코드 본문의 패턴을 살펴보겠습니다. `$()*` 내의 `temp_vec.push()`는 패턴의 `$()`와 0번 이상 일치하는 각 부분에 대해 생성됩니다. 패턴이 일치하는 횟수에 따라 다릅니다. `$x`는 일치하는 각 표현식으로 대체됩니다. 이 매크로를 `vec![1, 2, 3];`으로 호출하면 이 매크로 호출을 대체하는 생성된 코드는 다음과 같습니다.

```rust
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

우리는 모든 유형의 인수를 얼마든지 취할 수 있고 지정된 요소를 포함하는 벡터를 생성하는 코드를 생성할 수 있는 매크로를 정의했습니다.

매크로를 작성하는 방법에 대해 자세히 알아보려면 온라인 문서나 Daniel Keep이 시작하고 Lukas Wirth가 이어가는 [`The Little Book of Rust Macros`](https://veykril.github.io/tlborm/) 와 같은 기타 리소스를 참조하십시오 .

### [속성에서 코드를 생성하기 위한 절차적 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#procedural-macros-for-generating-code-from-attributes)

매크로의 두 번째 형태는 *프로시저 매크로* 로 , 함수처럼 작동하며 프로시저의 한 유형입니다. 절차적 매크로는 일부 코드를 입력으로 받아들이고, 해당 코드에서 작동하며, 선언적 매크로처럼 패턴과 일치시키고 코드를 다른 코드로 대체하는 대신 일부 코드를 출력으로 생성합니다. 세 가지 종류의 절차적 매크로는 사용자 지정 파생, 속성 유사 및 함수 유사이며 모두 비슷한 방식으로 작동합니다.

절차적 매크로를 만들 때 정의는 특수 크레이트 유형이 있는 자체 크레이트에 있어야 합니다. 이것은 우리가 미래에 제거하기를 희망하는 복잡한 기술적인 이유 때문입니다. 목록 19-29에서 절차적 매크로를 정의하는 방법을 보여줍니다. 여기서 `some_attribute`는 특정 매크로 종류를 사용하기 위한 자리 표시자입니다.

파일 이름: src/lib.rs

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

Listing 19-29: 절차적 매크로를 정의하는 예

절차적 매크로를 정의하는 함수는 `TokenStream`을 입력으로 사용하고 `TokenStream`을 출력으로 생성합니다. `TokenStream` 유형은 Rust에 포함된 `proc_macro` 크레이트에 의해 정의되며 일련의 토큰을 나타냅니다. 이것이 매크로의 핵심입니다. 매크로가 작동하는 소스 코드는 입력 `TokenStream`을 구성하고 매크로가 생성하는 코드는 출력 `TokenStream`입니다. 이 함수에는 우리가 만들고 있는 절차적 매크로의 종류를 지정하는 속성이 첨부되어 있습니다. 동일한 크레이트에 여러 종류의 절차적 매크로가 있을 수 있습니다.

다른 종류의 절차적 매크로를 살펴보겠습니다. 사용자 정의 파생 매크로로 시작한 다음 다른 양식을 다르게 만드는 작은 차이점을 설명합니다.

### [사용자 지정 `파생` 매크로를 작성하는 방법](https://doc.rust-lang.org/book/ch19-06-macros.html#how-to-write-a-custom-derive-macro)

`hello_macro`라는 하나의 관련 함수와 함께 `HelloMacro`라는 특성을 정의하는 `hello_macro`라는 크레이트를 만들어 봅시다. 사용자가 각 유형에 대해 `HelloMacro` 특성을 구현하도록 하는 대신 절차적 매크로를 제공하여 사용자가 `hello_macro`의 기본 구현을 얻기 위해 `#[derive(HelloMacro)]`로 유형에 주석을 달 수 있도록 합니다. 기능. 기본 구현은 `Hello, Macro! My name is TypeName!`을 인쇄합니다. 여기서 `TypeName`은 이 특성이 정의된 유형의 이름입니다. 다른 말로, 우리는 다른 프로그래머가 우리 크레이트를 사용하여 목록 19-30과 같은 코드를 작성할 수 있도록 하는 크레이트를 작성할 것입니다.

파일 이름: src/main.rs

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

Listing 19-30: 프로시저 매크로를 사용할 때 크레이트 사용자가 작성할 수 있는 코드

이 코드는 `Hello, Macro! My name is Pancakes!`를 인쇄합니다. 우리가 끝나면. 첫 번째 단계는 다음과 같이 새 라이브러리 크레이트를 만드는 것입니다.

```bash
$ cargo new hello_macro --lib
```

다음으로 `HelloMacro` 특성 및 관련 함수를 정의합니다.

파일 이름: src/lib.rs

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

우리에게는 특성과 기능이 있습니다. 이 시점에서 크레이트 사용자는 다음과 같이 원하는 기능을 달성하기 위해 특성을 구현할 수 있습니다.

```rust
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!(`Hello, Macro! My name is Pancakes!`);
    }
}

fn main() {
    Pancakes::hello_macro();
}
```

그러나 `hello_macro`와 함께 사용하려는 각 유형에 대한 구현 블록을 작성해야 합니다. 우리는 그들이 이 일을 하지 않아도 되기를 원합니다.

추가로, 우리는 트레이트가 구현된 타입의 이름을 인쇄할 기본 구현으로 `hello_macro` 함수를 아직 제공할 수 없습니다: Rust에는 리플렉션 기능이 없으므로 런타임에 타입의 이름을 조회할 수 없습니다. . 컴파일 타임에 코드를 생성하려면 매크로가 필요합니다.

다음 단계는 절차적 매크로를 정의하는 것입니다. 이 글을 쓰는 시점에서 절차적 매크로는 자체 크레이트에 있어야 합니다. 결국 이 제한이 해제될 수 있습니다. 크레이트와 매크로 크레이트를 구성하는 규칙은 다음과 같습니다: `foo`라는 이름의 크레이트의 경우 사용자 정의 파생 절차 매크로 크레이트를 `foo_derive`라고 합니다. `hello_macro` 프로젝트 내에서 `hello_macro_derive`라는 새 크레이트를 시작하겠습니다.

```bash
$ cargo new hello_macro_derive --lib
```

우리의 두 크레이트는 밀접하게 관련되어 있으므로 `hello_macro` 크레이트의 디렉토리 내에 절차적 매크로 크레이트를 만듭니다. `hello_macro`에서 특성 정의를 변경하면 `hello_macro_derive`에서 절차적 매크로의 구현도 변경해야 합니다. 두 크레이트는 별도로 게시해야 하며, 이러한 크레이트를 사용하는 프로그래머는 두 크레이트를 종속성으로 추가하고 둘 다 범위로 가져와야 합니다. 대신 `hello_macro` 크레이트가 `hello_macro_derive`를 종속성으로 사용하고 절차적 매크로 코드를 다시 내보낼 수 있습니다. 그러나 우리가 프로젝트를 구성한 방식은 프로그래머가 `파생` 기능을 원하지 않는 경우에도 `hello_macro`를 사용할 수 있도록 합니다.

`hello_macro_derive` 크레이트를 절차적 매크로 크레이트로 선언해야 합니다. 잠시 후에 보게 되겠지만 `syn` 및 `quote` 크레이트의 기능도 필요하므로 종속 항목으로 추가해야 합니다. `hello_macro_derive`에 대한 *Cargo.toml* 파일 에 다음을 추가합니다.

파일 이름: hello_macro_derive/Cargo.toml

```toml
[lib]
proc-macro = true

[dependencies]
syn = `1.0`
quote = `1.0`
```

절차적 매크로 정의를 시작하려면 Listing 19-31의 코드를 `hello_macro_derive` 크레이트에 대한 *src/lib.rs 파일에 넣습니다.* 이 코드는 `impl_hello_macro` 함수에 대한 정의를 추가할 때까지 컴파일되지 않습니다.

파일 이름: hello_macro_derive/src/lib.rs

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
```

Listing 19-31: Rust 코드를 처리하기 위해 대부분의 절차적 매크로 크레이트에 필요한 코드

코드를 `TokenStream` 구문 분석을 담당하는 `hello_macro_derive` 함수와 구문 트리 변환을 담당하는 `impl_hello_macro` 함수로 분할했습니다. 이렇게 하면 절차 매크로를 더 편리하게 작성할 수 있습니다. 외부 함수의 코드(이 경우 `hello_macro_derive`)는 보거나 생성하는 거의 모든 절차적 매크로 크레이트에 대해 동일합니다. 내부 함수의 본문에 지정하는 코드(이 경우 `impl_hello_macro`)는 절차 매크로의 목적에 따라 달라집니다.

[`proc_macro`, `syn`](https://crates.io/crates/syn) 및 [`quote` 의](https://crates.io/crates/quote) 세 가지 새로운 크레이트를 도입했습니다. *`proc_macro` 크레이트는 Rust와 함께 제공되므로 Cargo.toml* 의 종속성에 추가할 필요가 없습니다. `proc_macro` 크레이트는 우리 코드에서 Rust 코드를 읽고 조작할 수 있게 해주는 컴파일러의 API입니다.

`syn` 크레이트는 문자열의 Rust 코드를 작업을 수행할 수 있는 데이터 구조로 구문 분석합니다. `quote` 크레이트는 `syn` 데이터 구조를 Rust 코드로 되돌립니다. 이러한 크레이트는 우리가 처리하고자 하는 모든 종류의 Rust 코드를 파싱하는 것을 훨씬 더 간단하게 만듭니다. Rust 코드에 대한 전체 파서를 작성하는 것은 간단한 작업이 아닙니다.

`hello_macro_derive` 함수는 라이브러리 사용자가 유형에 `#[derive(HelloMacro)]`를 지정하면 호출됩니다. 이는 여기에서 `proc_macro_derive`로 `hello_macro_derive` 함수에 주석을 달고 특성 이름과 일치하는 `HelloMacro`라는 이름을 지정했기 때문에 가능합니다. 이것은 대부분의 절차적 매크로가 따르는 규칙입니다.

`hello_macro_derive` 함수는 먼저 `TokenStream`의 `입력`을 우리가 해석하고 작업을 수행할 수 있는 데이터 구조로 변환합니다. 여기에서 `syn`이 작동합니다. `syn`의 `parse` 함수는 `TokenStream`을 취하고 구문 분석된 Rust 코드를 나타내는 `DeriveInput` 구조체를 반환합니다. 목록 19-32는 `struct Pancakes`를 구문 분석하여 얻은 `DeriveInput` 구조체의 관련 부분을 보여줍니다. 끈:

```rust
DeriveInput {
    // --snip--

    ident: Ident {
        ident: `Pancakes`,
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

Listing 19-32: Listing 19-30에서 매크로 속성을 가진 코드를 구문 분석할 때 얻는 `DeriveInput` 인스턴스

이 구조체의 필드는 우리가 파싱한 Rust 코드가 `Pancakes`의 `ident`(식별자, 이름을 의미)가 있는 단위 구조체임을 보여줍니다. 이 구조체에는 모든 종류의 Rust 코드를 설명하기 위한 더 많은 필드가 있습니다. 자세한 내용은 [`DeriveInput`에 대한 `syn` 문서를](https://docs.rs/syn/1.0/syn/struct.DeriveInput.html) 확인하십시오 .

곧 우리는 `impl_hello_macro` 함수를 정의할 것입니다. 여기서 우리는 포함하고 싶은 새로운 Rust 코드를 만들 것입니다. 그러나 그 전에 파생 매크로의 출력도 `TokenStream`이라는 점에 유의하십시오. 반환된 `TokenStream`은 크레이트 사용자가 작성하는 코드에 추가되므로 크레이트를 컴파일할 때 수정된 `TokenStream`에서 제공하는 추가 기능을 얻게 됩니다.

여기에서 `syn::parse` 함수에 대한 호출이 실패하면 `hello_macro_derive` 함수가 패닉 상태가 되도록 `unwrap`을 호출하고 있음을 알아차렸을 것입니다. 절차적 매크로 API를 준수하려면 `proc_macro_derive` 함수가 `결과`가 아닌 `TokenStream`을 반환해야 하므로 오류 시 절차적 매크로가 패닉 상태가 되어야 합니다. `unwrap`을 사용하여 이 예제를 단순화했습니다. 프로덕션 코드에서는 `패닉!`을 사용하여 무엇이 잘못되었는지에 대한 보다 구체적인 오류 메시지를 제공해야 합니다. 또는 `기대`.

주석이 달린 Rust 코드를 `TokenStream`에서 `DeriveInput` 인스턴스로 전환하는 코드가 있으므로 Listing 19-33과 같이 주석이 달린 유형에 `HelloMacro` 특성을 구현하는 코드를 생성해 보겠습니다.

파일 이름: hello_macro_derive/src/lib.rs

```rust
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!(`Hello, Macro! My name is {}!`, stringify!(#name));
            }
        }
    };
    gen.into()
}
```

목록 19-33: 파싱된 Rust 코드를 사용하여 `HelloMacro` 특성 구현하기

`ast.ident`를 사용하여 주석이 달린 유형의 이름(식별자)을 포함하는 `Ident` 구조체 인스턴스를 얻습니다. 목록 19-32의 구조체는 우리가 목록 19-30의 코드에서 `impl_hello_macro` 함수를 실행할 때 우리가 얻는 `ident`가 ``Pancakes`` 값을 가진 `ident` 필드를 가질 것임을 보여줍니다. 따라서 Listing 19-33의 `name` 변수는 인쇄될 때 Listing 19-30의 구조체 이름인 ``Pancakes`` 문자열이 될 `Ident` 구조체 인스턴스를 포함합니다.

인용구!` 매크로를 사용하면 반환하려는 Rust 코드를 정의할 수 있습니다. 컴파일러는 `quote!`의 직접적인 결과와 다른 결과를 기대합니다. 매크로의 실행이므로 `TokenStream`으로 변환해야 합니다. 이 중간 표현을 사용하고 필요한 `TokenStream` 유형의 값을 반환하는 `into` 메서드를 호출하여 이를 수행합니다.

인용구!` 매크로는 또한 몇 가지 매우 멋진 템플릿 메커니즘을 제공합니다. `#name` 및 `quote!`를 입력할 수 있습니다. 변수 `name`의 값으로 대체합니다. 일반 매크로가 작동하는 방식과 유사한 반복 작업을 수행할 수도 있습니다. 자세한 소개는 [`quote` 크레이트의 문서를](https://docs.rs/quote) 확인하세요 .

우리는 절차적 매크로가 사용자가 주석을 추가한 유형에 대한 `HelloMacro` 특성의 구현을 생성하기를 원합니다. `#name`을 사용하여 얻을 수 있습니다. 특성 구현에는 하나의 함수 `hello_macro`가 있으며, 그 본문에는 우리가 제공하려는 기능이 포함되어 있습니다. `Hello, Macro! My name is`를 인쇄한 다음 주석이 달린 유형의 이름을 인쇄합니다.

`문자열화!` 여기에 사용된 매크로는 Rust에 내장되어 있습니다. 이것은 `1 + 2`와 같은 Rust 표현식을 취하며 컴파일 타임에 표현식을 ``1 + 2``와 같은 문자열 리터럴로 바꿉니다. 이것은 `포맷!`과는 다릅니다. 또는 `println!`, 식을 평가한 다음 결과를 `문자열`로 변환하는 매크로. `#name` 입력은 문자 그대로 출력하는 표현식일 가능성이 있으므로 `stringify!`를 사용합니다. `문자열화!` 사용 또한 컴파일 시간에 `#name`을 문자열 리터럴로 변환하여 할당을 저장합니다.

이 시점에서 `cargo build`는 `hello_macro`와 `hello_macro_derive` 모두에서 성공적으로 완료되어야 합니다. 이 크레이트를 Listing 19-30의 코드에 연결하여 절차적 매크로가 작동하는 것을 봅시다! *`cargo new 팬케이크`를 사용하여 프로젝트* 디렉토리 에 새 바이너리 프로젝트를 만듭니다. *`hello_macro` 및 `hello_macro_derive`를 `pancakes` 크레이트의 Cargo.toml* 에 종속 항목으로 추가해야 합니다. `hello_macro` 및 `hello_macro_derive` 버전을 [crates.io](https://crates.io/) 에 게시하는 경우 일반 종속성이 됩니다. 그렇지 않은 경우 다음과 같이 `경로` 종속성으로 지정할 수 있습니다.

```toml
hello_macro = { path = `../hello_macro` }
hello_macro_derive = { path = `../hello_macro/hello_macro_derive` }
```

Listing 19-30의 코드를 *src/main.rs* 에 넣고 `cargo run`을 실행합니다: `Hello, Macro! My name is Pancakes!`를 출력해야 합니다. 절차적 매크로의 `HelloMacro` 특성 구현은 이를 구현할 필요가 있는 `팬케이크` 상자 없이 포함되었습니다. `#[derive(HelloMacro)]`는 특성 구현을 추가했습니다.

다음으로 다른 종류의 절차적 매크로가 사용자 지정 파생 매크로와 어떻게 다른지 살펴보겠습니다.

### [속성과 같은 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#attribute-like-macros)

특성 유사 매크로는 사용자 지정 파생 매크로와 유사하지만 `파생` 특성에 대한 코드를 생성하는 대신 새 특성을 만들 수 있습니다. 또한 더 유연합니다. `파생`은 구조체 및 열거형에 대해서만 작동합니다. 속성은 기능과 같은 다른 항목에도 적용될 수 있습니다. 다음은 속성과 유사한 매크로를 사용하는 예입니다. 웹 애플리케이션 프레임워크를 사용할 때 함수에 주석을 추가하는 `route`라는 속성이 있다고 가정해 보겠습니다.

```rust
#[route(GET, `/`)]
fn index() {
```

이 `#[route]` 속성은 프레임워크에서 절차적 매크로로 정의됩니다. 매크로 정의 함수의 서명은 다음과 같습니다.

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

여기에 `TokenStream` 유형의 두 매개변수가 있습니다. 첫 번째는 속성의 내용인 `GET, `/`` 부분입니다. 두 번째는 속성이 첨부된 항목의 본문입니다. 이 경우 `fn index() {}` 및 나머지 함수 본문입니다.

그 외에 속성 유사 매크로는 사용자 정의 파생 매크로와 동일한 방식으로 작동합니다. `proc-macro` 크레이트 유형으로 크레이트를 생성하고 원하는 코드를 생성하는 함수를 구현합니다!

### [함수형 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#function-like-macros)

함수형 매크로는 함수 호출처럼 보이는 매크로를 정의합니다. `macro_rules!`와 유사합니다. 매크로는 함수보다 더 유연합니다. 예를 들어 알 수 없는 수의 인수를 사용할 수 있습니다. 그러나 `macro_rules!` [매크로는 `macro_rules를 사용한 선언적 매크로`](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming) 섹션에서 논의한 일치 유사 구문을 사용해서만 정의할 수 있습니다. [일반 메타프로그래밍용”을 참조하십시오](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming) . 함수와 같은 매크로는 `TokenStream` 매개변수를 사용하며 그 정의는 다른 두 가지 유형의 절차적 매크로가 수행하는 것처럼 Rust 코드를 사용하여 해당 `TokenStream`을 조작합니다. 함수와 유사한 매크로의 예는 `sql!` 다음과 같이 호출될 수 있는 매크로:

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

이 매크로는 내부의 SQL 문을 구문 분석하고 구문이 올바른지 확인합니다. 이는 `macro_rules!`보다 훨씬 복잡한 처리입니다. 매크로는 할 수 있습니다. `SQL!` 매크로는 다음과 같이 정의됩니다.

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

이 정의는 사용자 지정 파생 매크로의 서명과 유사합니다. 괄호 안에 있는 토큰을 받고 생성하려는 코드를 반환합니다.

## [요약](https://doc.rust-lang.org/book/ch19-06-macros.html#summary)

아휴! 이제 도구 상자에 자주 사용하지 않을 것 같은 일부 Rust 기능이 있지만 매우 특정한 상황에서 사용할 수 있다는 것을 알게 될 것입니다. 오류 메시지 제안 또는 다른 사람의 코드에서 이러한 개념과 구문을 인식할 수 있도록 몇 가지 복잡한 항목을 도입했습니다. 솔루션을 안내하는 참조로 이 장을 사용하십시오.

다음으로 책 전체에서 논의한 모든 내용을 실행에 옮기고 프로젝트를 하나 더 수행합니다!

------

# 20

# [최종 프로젝트: 다중 스레드 웹 서버 구축](https://doc.rust-lang.org/book/ch20-00-final-project-a-web-server.html#final-project-building-a-multithreaded-web-server)

긴 여정이었지만 책의 끝에 도달했습니다. 이 장에서는 마지막 장에서 다룬 개념 중 일부를 시연하고 이전 학습 내용을 요약하기 위해 함께 프로젝트를 하나 더 빌드합니다.

최종 프로젝트를 위해 웹 브라우저에서 `hello`라고 말하고 그림 20-1처럼 보이는 웹 서버를 만들 것입니다.

![안녕하세요 녹에서](https://doc.rust-lang.org/book/img/trpl20-01.png)

그림 20-1: 최종 공유 프로젝트

다음은 웹 서버 구축 계획입니다.

1. TCP와 HTTP에 대해 조금 알아보세요.
2. 소켓에서 TCP 연결을 수신합니다.
3. 소수의 HTTP 요청을 구문 분석합니다.
4. 적절한 HTTP 응답을 만듭니다.
5. 스레드 풀로 서버의 처리량을 향상시킵니다.

시작하기 전에 한 가지 세부 사항을 언급해야 합니다. 우리가 사용할 방법은 Rust로 웹 서버를 구축하는 최선의 방법이 아닙니다. 커뮤니티 회원들은 우리가 구축할 것보다 더 완벽한 웹 서버 및 스레드 풀 구현을 제공하는 [crates.io](https://crates.io/) 에서 사용할 수 있는 생산 준비가 된 여러 크레이트를 게시했습니다. 그러나 이 장에서 우리의 의도는 쉬운 길을 택하는 것이 아니라 학습을 돕는 것입니다. Rust는 시스템 프로그래밍 언어이기 때문에 작업하려는 추상화 수준을 선택할 수 있고 다른 언어에서 가능하거나 실용적인 수준보다 낮은 수준으로 이동할 수 있습니다. 따라서 기본 HTTP 서버와 스레드 풀을 수동으로 작성하여 향후 사용할 크레이트의 일반적인 아이디어와 기술을 배울 수 있습니다.

------

## [단일 스레드 웹 서버 구축](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#building-a-single-threaded-web-server)

단일 스레드 웹 서버를 작동시키는 것으로 시작하겠습니다. 시작하기 전에 웹 서버 구축과 관련된 프로토콜에 대한 간략한 개요를 살펴보겠습니다. 이러한 프로토콜에 대한 자세한 내용은 이 책의 범위를 벗어나지만 간략한 개요를 통해 필요한 정보를 얻을 수 있습니다.

웹 서버와 관련된 두 가지 주요 프로토콜은 *HTTP(* *Hypertext Transfer Protocol* ) 와 *TCP(* *Transmission Control Protocol* )입니다. 두 프로토콜 모두 *요청-응답* 프로토콜입니다. 즉, *클라이언트가* 요청을 시작하고 *서버가* 요청을 듣고 클라이언트에 응답을 제공합니다. 이러한 요청 및 응답의 내용은 프로토콜에 의해 정의됩니다.

TCP는 한 서버에서 다른 서버로 정보를 가져오는 방법에 대한 세부 정보를 설명하지만 해당 정보가 무엇인지는 지정하지 않는 하위 수준 프로토콜입니다. HTTP는 요청 및 응답의 내용을 정의하여 TCP 위에 구축됩니다. HTTP를 다른 프로토콜과 함께 사용하는 것은 기술적으로 가능하지만 대부분의 경우 HTTP는 TCP를 통해 데이터를 보냅니다. 우리는 TCP 및 HTTP 요청과 응답의 원시 바이트로 작업할 것입니다.

### [TCP 연결 듣기](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#listening-to-the-tcp-connection)

웹 서버는 TCP 연결을 수신해야 하므로 이것이 우리가 작업할 첫 번째 부분입니다. 표준 라이브러리는 이를 가능하게 하는 `std::net` 모듈을 제공합니다. 일반적인 방식으로 새 프로젝트를 만들어 봅시다.

```bash
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

이제 시작하려면 *src/main.rs* 에 목록 20-1의 코드를 입력하십시오 . 이 코드는 들어오는 TCP 스트림에 대해 로컬 주소 `127.0.0.1:7878`에서 수신 대기합니다. 들어오는 스트림을 받으면 `연결 설정!`을 인쇄합니다.

파일 이름: src/main.rs

```rust
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind(`127.0.0.1:7878`).unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        println!(`Connection established!`);
    }
}
```

Listing 20-1: 들어오는 스트림을 듣고 스트림을 받으면 메시지 출력하기

`TcpListener`를 사용하여 주소 `127.0.0.1:7878`에서 TCP 연결을 수신할 수 있습니다. 주소에서 콜론 앞의 섹션은 컴퓨터를 나타내는 IP 주소(이는 모든 컴퓨터에서 동일하며 작성자의 컴퓨터를 구체적으로 나타내지 않음)이고 `7878`은 포트입니다. 우리는 두 가지 이유로 이 포트를 선택했습니다. HTTP는 일반적으로 이 포트에서 허용되지 않으므로 우리 서버는 컴퓨터에서 실행 중인 다른 웹 서버와 충돌할 가능성이 없으며 7878은 전화기에 녹슬게 입력 됩니다 *.*

이 시나리오의 `bind` 함수는 새 `TcpListener` 인스턴스를 반환한다는 점에서 `new` 함수처럼 작동합니다. 이 기능을 `바인딩`이라고 부르는 이유는 네트워킹에서 수신할 포트에 연결하는 것을 `포트에 바인딩`하는 것으로 알려져 있기 때문입니다.

`bind` 함수는 바인딩이 실패할 수 있음을 나타내는 `Result<T, E>`를 반환합니다. 예를 들어 포트 80에 연결하려면 관리자 권한이 필요하므로(관리자가 아닌 사용자는 1023보다 높은 포트에서만 수신할 수 있음) 관리자가 아닌 상태에서 포트 80에 연결하려고 하면 바인딩이 작동하지 않습니다. 예를 들어 프로그램의 두 인스턴스를 실행하여 동일한 포트를 수신하는 두 개의 프로그램이 있는 경우 바인딩도 작동하지 않습니다. 학습 목적으로 기본 서버를 작성하고 있기 때문에 이러한 종류의 오류 처리에 대해 걱정하지 않습니다. 대신 오류가 발생하면 `unwrap`을 사용하여 프로그램을 중지합니다.

`TcpListener`의 `incoming` 메서드는 일련의 스트림(더 구체적으로는 `TcpStream` 유형의 스트림)을 제공하는 반복자를 반환합니다. 단일 *스트림은* 클라이언트와 서버 간의 열린 연결을 나타냅니다. 연결 *은* 클라이언트가 서버에 연결하고 서버가 응답을 생성하며 서버가 연결을 닫는 전체 요청 및 응답 프로세스의 이름입니다. 따라서 `TcpStream`에서 읽어 클라이언트가 보낸 내용을 확인한 다음 스트림에 대한 응답을 작성하여 데이터를 다시 클라이언트로 보냅니다. 전반적으로 이 `for` 루프는 각 연결을 차례로 처리하고 처리할 일련의 스트림을 생성합니다.

현재 스트림 처리는 스트림에 오류가 있는 경우 프로그램을 종료하기 위해 `unwrap`을 호출하는 것으로 구성됩니다. 오류가 없으면 프로그램은 메시지를 인쇄합니다. 다음 목록에서 성공 사례에 대한 기능을 더 추가할 것입니다. 클라이언트가 서버에 연결할 때 `수신` 방법에서 오류가 발생할 수 있는 이유는 실제로 연결을 반복하지 않기 때문입니다. *대신 연결 시도를* 반복합니다. 여러 가지 이유로 연결에 실패할 수 있으며 그 중 대부분은 운영 체제에 따라 다릅니다. 예를 들어, 많은 운영 체제는 지원할 수 있는 동시 개방 연결 수에 제한이 있습니다. 해당 숫자를 초과하는 새 연결 시도는 열려 있는 연결 중 일부가 닫힐 때까지 오류를 생성합니다.

이 코드를 실행해 봅시다! 터미널에서 `cargo run`을 호출한 다음 웹 브라우저에서 *127.0.0.1:7878을 로드합니다.* 서버가 현재 데이터를 다시 전송하지 않기 때문에 브라우저에 `연결 재설정`과 같은 오류 메시지가 표시되어야 합니다. 그러나 터미널을 보면 브라우저가 서버에 연결될 때 인쇄된 여러 메시지가 표시되어야 합니다!

```
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

때로는 하나의 브라우저 요청에 대해 여러 메시지가 인쇄되는 것을 볼 수 있습니다. 그 이유는 브라우저가 브라우저 탭에 표시되는 *favicon.ico* 아이콘 과 같은 다른 리소스에 대한 요청뿐만 아니라 페이지에 대한 요청을 하기 때문일 수 있습니다.

서버가 어떤 데이터로도 응답하지 않기 때문에 브라우저가 서버에 여러 번 연결을 시도하는 것일 수도 있습니다. `stream`이 범위를 벗어나 루프 끝에서 삭제되면 `drop` 구현의 일부로 연결이 닫힙니다. 문제가 일시적일 수 있기 때문에 브라우저는 때때로 재시도를 통해 닫힌 연결을 처리합니다. 중요한 요소는 TCP 연결에 대한 핸들을 성공적으로 얻었다는 것입니다!

특정 버전의 코드 실행이 완료되면 ctrl-c를 눌러 프로그램을 중지해야 합니다. 그런 다음 최신 코드를 실행하고 있는지 확인하기 위해 각 코드 세트를 변경한 후 `cargo run` 명령을 호출하여 프로그램을 다시 시작하십시오.

### [요청 읽기](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#reading-the-request)

브라우저에서 요청을 읽는 기능을 구현해 봅시다! 먼저 연결을 얻은 다음 연결에 대해 몇 가지 작업을 수행하는 문제를 분리하기 위해 연결을 처리하는 새 기능을 시작합니다. 이 새로운 `handle_connection` 함수에서는 TCP 스트림에서 데이터를 읽고 인쇄하여 브라우저에서 전송되는 데이터를 볼 수 있습니다. 목록 20-2와 같이 코드를 변경합니다.

파일 이름: src/main.rs

```rust
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind(`127.0.0.1:7878`).unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!(`Request: {:#?}`, http_request);
}
```

목록 20-2: `TcpStream`에서 읽고 데이터 인쇄

`std::io::prelude` 및 `std::io::BufReader`를 범위로 가져와 스트림에서 읽고 쓸 수 있는 특성 및 유형에 액세스합니다. `main` 함수의 `for` 루프에서 우리가 연결했다는 메시지를 인쇄하는 대신 이제 새로운 `handle_connection` 함수를 호출하고 `stream`을 전달합니다.

`handle_connection` 함수에서 `스트림`에 대한 변경 가능한 참조를 래핑하는 새 `BufReader` 인스턴스를 만듭니다. `BufReader`는 `std::io::Read` 특성 메서드에 대한 호출을 관리하여 버퍼링을 추가합니다.

브라우저가 서버로 보내는 요청 라인을 수집하기 위해 `http_request`라는 변수를 생성합니다. `Vec<_>` 유형 주석을 추가하여 벡터에서 이러한 라인을 수집하고 싶다는 것을 나타냅니다.

`BufReader`는 `lines` 메소드를 제공하는 `std::io::BufRead` 특성을 구현합니다. `lines` 메서드는 개행 바이트를 볼 때마다 데이터 스트림을 분할하여 `Result<String, std::io::Error>`의 반복자를 반환합니다. 각 `문자열`을 얻기 위해 각 `결과`를 매핑하고 `언래핑`합니다. 데이터가 유효한 UTF-8이 아니거나 스트림에서 읽는 데 문제가 있는 경우 `결과`는 오류일 수 있습니다. 다시 말하지만 프로덕션 프로그램은 이러한 오류를 보다 우아하게 처리해야 하지만 단순성을 위해 오류 사례에서 프로그램을 중지하도록 선택했습니다.

브라우저는 줄 바꿈 문자 두 개를 연속으로 전송하여 HTTP 요청의 끝을 알립니다. 따라서 스트림에서 하나의 요청을 얻으려면 빈 문자열인 줄을 얻을 때까지 줄을 섭니다. 라인을 벡터로 수집하면 웹 브라우저가 서버로 보내는 명령을 볼 수 있도록 예쁜 디버그 형식을 사용하여 라인을 인쇄합니다.

이 코드를 사용해 봅시다! 프로그램을 시작하고 웹 브라우저에서 다시 요청하십시오. 여전히 브라우저에 오류 페이지가 표시되지만 터미널의 프로그램 출력은 이제 다음과 유사하게 표시됩니다.

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    `GET / HTTP/1.1`,
    `Host: 127.0.0.1:7878`,
    `User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0`,
    `Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8`,
    `Accept-Language: en-US,en;q=0.5`,
    `Accept-Encoding: gzip, deflate, br`,
    `DNT: 1`,
    `Connection: keep-alive`,
    `Upgrade-Insecure-Requests: 1`,
    `Sec-Fetch-Dest: document`,
    `Sec-Fetch-Mode: navigate`,
    `Sec-Fetch-Site: none`,
    `Sec-Fetch-User: ?1`,
    `Cache-Control: max-age=0`,
]
```

브라우저에 따라 출력이 약간 다를 수 있습니다. 이제 요청 데이터를 인쇄하고 있으므로 요청의 첫 번째 줄에서 `GET` 뒤에 있는 경로를 보면 하나의 브라우저 요청에서 여러 연결을 얻는 이유를 알 수 있습니다. 반복되는 연결이 모두 */ 를* 요청하는 경우 프로그램에서 응답을 받지 못하기 때문에 브라우저가 */ 반복적으로 가져오려고 시도하고 있음을 알 수 있습니다.*

브라우저가 우리 프로그램에 무엇을 요구하는지 이해하기 위해 이 요청 데이터를 분석해 봅시다.

### [HTTP 요청 자세히 살펴보기](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#a-closer-look-at-an-http-request)

HTTP는 텍스트 기반 프로토콜이며 요청은 다음 형식을 사용합니다.

```
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

첫 번째 줄은 클라이언트가 요청한 내용에 대한 정보를 담고 있는 *요청 줄 입니다.* 요청 줄의 첫 번째 부분은 클라이언트가 이 요청을 만드는 방법을 설명하는 `GET` 또는 `POST`와 같이 사용 중인 *메서드를 나타냅니다.* 우리 클라이언트는 정보를 요청하는 `GET` 요청을 사용했습니다.

요청 줄의 다음 부분은 */ 로, 클라이언트가 요청하는 URI(* *Uniform Resource Identifier* *)를* 나타냅니다. URI는 URL( *Uniform Resource Locator* *)* 과 거의 동일하지만 완전하지는 않습니다. URI와 URL의 차이는 이 장의 목적에 중요하지 않지만 HTTP 사양은 URI라는 용어를 사용하므로 여기서는 정신적으로 URL을 URI로 대체할 수 있습니다.

마지막 부분은 클라이언트가 사용하는 HTTP 버전이며 요청 라인은 *CRLF 시퀀스* 로 끝납니다. *(CRLF는 캐리지 리턴* 과 *줄 바꿈을* 의미하며 타자기 시대의 용어입니다!) CRLF 시퀀스는 `\r\n`으로도 쓸 수 있습니다. 여기서 `\r`은 캐리지 리턴이고 `\n`은 줄 바꿈. CRLF 시퀀스는 나머지 요청 데이터에서 요청 라인을 분리합니다. CRLF가 인쇄될 때 `\r\n`이 아닌 새 줄 시작이 표시됩니다.

지금까지 프로그램을 실행하여 받은 요청 라인 데이터를 보면 `GET`이 메서드이고 */* 가 요청 URI이고 `HTTP/1.1`이 버전임을 알 수 있습니다.

요청 줄 다음에 `Host:`부터 시작하여 나머지 줄은 헤더입니다. `GET` 요청에는 본문이 없습니다.

*다른 브라우저에서 요청하거나 127.0.0.1:7878/test* 와 같은 다른 주소를 요청하여 요청 데이터가 어떻게 변경되는지 확인하십시오.

이제 브라우저가 무엇을 요청하는지 알았으므로 일부 데이터를 다시 보내겠습니다!

### [응답 작성](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#writing-a-response)

클라이언트 요청에 대한 응답으로 데이터 전송을 구현할 것입니다. 응답의 형식은 다음과 같습니다.

```
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

첫 번째 줄은 응답에 사용된 HTTP 버전, 요청 결과를 요약하는 숫자 상태 코드 및 상태 코드에 대한 텍스트 설명을 제공하는 이유 문구를 포함하는 상태 줄입니다 *.* CRLF 시퀀스 뒤에는 헤더, 다른 CRLF 시퀀스 및 응답 본문이 있습니다.

다음은 HTTP 버전 1.1을 사용하고 상태 코드 200, OK 이유 문구, 헤더 및 본문이 없는 예제 응답입니다.

```
HTTP/1.1 200 OK\r\n\r\n
```

상태 코드 200은 표준 성공 응답입니다. 텍스트는 아주 작은 성공적인 HTTP 응답입니다. 성공적인 요청에 대한 응답으로 이것을 스트림에 작성해 봅시다! `handle_connection` 함수에서 `println!` 요청 데이터를 인쇄하고 Listing 20-3의 코드로 대체합니다.

파일 이름: src/main.rs

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let response = `HTTP/1.1 200 OK\r\n\r\n`;
    
    stream.write_all(response.as_bytes()).unwrap();
}
```

Listing 20-3: 스트림에 작은 성공 HTTP 응답 쓰기

첫 번째 줄은 성공 메시지의 데이터를 보유하는 `response` 변수를 정의합니다. 그런 다음 `응답`에서 `as_bytes`를 호출하여 문자열 데이터를 바이트로 변환합니다. `stream`의 `write_all` 메서드는 `&[u8]`을 사용하여 해당 바이트를 연결 아래로 직접 보냅니다. `write_all` 작업이 실패할 수 있으므로 이전과 같이 모든 오류 결과에 대해 `unwrap`을 사용합니다. 다시 말하지만 실제 애플리케이션에서는 여기에 오류 처리를 추가합니다.

이러한 변경 사항을 사용하여 코드를 실행하고 요청해 보겠습니다. 더 이상 터미널에 데이터를 인쇄하지 않으므로 Cargo의 출력 외에는 출력이 표시되지 않습니다. *웹 브라우저에서 127.0.0.1:7878을* 로드하면 오류 대신 빈 페이지가 표시됩니다. HTTP 요청 수신 및 응답 전송을 직접 코딩했습니다!

### [실제 HTML 반환](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#returning-real-html)

빈 페이지 이상을 반환하는 기능을 구현해 봅시다. *src* 디렉터리 가 아니라 프로젝트 디렉터리의 루트에 새 파일 *hello.html을* 만듭니다. 원하는 HTML을 입력할 수 있습니다. 목록 20-4는 한 가지 가능성을 보여줍니다.

파일명: hello.html

```html
<!DOCTYPE html>
<html lang=`en`>
  <head>
    <meta charset=`utf-8`>
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

목록 20-4: 응답으로 반환할 샘플 HTML 파일

이것은 제목과 일부 텍스트가 있는 최소한의 HTML5 문서입니다. 요청을 받았을 때 서버에서 이를 반환하기 위해 Listing 20-5와 같이 `handle_connection`을 수정하여 HTML 파일을 읽고 본문으로 응답에 추가하여 보냅니다.

파일 이름: src/main.rs

```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = `HTTP/1.1 200 OK`;
    let contents = fs::read_to_string(`hello.html`).unwrap();
    let length = contents.len();
    
    let response =
        format!(`{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}`);
    
    stream.write_all(response.as_bytes()).unwrap();
}
```

목록 20-5: 응답 본문으로 *hello.html* 의 내용 보내기

표준 라이브러리의 파일 시스템 모듈을 범위로 가져오기 위해 `use` 문에 `fs`를 추가했습니다. 파일의 내용을 문자열로 읽는 코드는 익숙할 것입니다. Listing 12-4의 I/O 프로젝트에 대한 파일 내용을 읽을 때 12장에서 사용했습니다.

다음으로 `포맷!`을 사용합니다. 파일의 내용을 성공 응답의 본문으로 추가합니다. 유효한 HTTP 응답을 보장하기 위해 응답 본문의 크기(이 경우 `hello.html` 크기)로 설정된 `Content-Length` 헤더를 추가합니다.

`cargo run`으로 이 코드를 실행하고 브라우저에서 *127.0.0.1:7878을 로드합니다.* HTML이 렌더링되는 것을 볼 수 있습니다!

현재는 `http_request`의 요청 데이터를 무시하고 무조건 HTML 파일의 내용만 되돌려 보내고 있습니다. *즉, 브라우저에서 127.0.0.1:7878/something-else를* 요청하면 여전히 동일한 HTML 응답을 받게 됩니다. 현재 저희 서버는 매우 제한적이며 대부분의 웹 서버가 하는 일을 하지 않습니다. *우리는 요청에 따라 응답을 사용자 지정하고 /* 에 대한 올바른 형식의 요청에 대한 HTML 파일만 다시 보내길 원합니다.

### [요청 확인 및 선택적으로 응답](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#validating-the-request-and-selectively-responding)

지금 우리 웹 서버는 클라이언트가 무엇을 요청하든 파일에 HTML을 반환합니다. HTML 파일을 반환하기 전에 브라우저가 요청하고 있는지 확인하고 브라우저가 다른 것을 요청하면 오류를 반환하는 기능을 추가해 *봅시다* . 이를 위해 Listing 20-6에 표시된 것처럼 `handle_connection`을 수정해야 합니다. *이 새로운 코드는 우리가 알고 있는 요청의 모양 과* 수신된 요청의 내용을 확인 하고 `if` 및 `else` 블록을 추가하여 요청을 다르게 처리합니다.

파일 이름: src/main.rs

```rust
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == `GET / HTTP/1.1` {
        let status_line = `HTTP/1.1 200 OK`;
        let contents = fs::read_to_string(`hello.html`).unwrap();
        let length = contents.len();
    
        let response = format!(
            `{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}`
        );
    
        stream.write_all(response.as_bytes()).unwrap();
    } else {
        // some other request
    }
}
```

Listing 20-6: 다른 요청에 대한 */* 다른 요청 처리

HTTP 요청의 첫 번째 줄만 볼 것이므로 전체 요청을 벡터로 읽는 대신 반복자에서 첫 번째 항목을 가져오기 위해 `next`를 호출합니다. 첫 번째 `unwrap`은 `Option`을 처리하고 반복자에 항목이 없으면 프로그램을 중지합니다. 두 번째 `unwrap`은 `Result`를 처리하며 Listing 20-2에 추가된 `map`에 있던 `unwrap`과 동일한 효과를 가집니다.

다음으로 `request_line`을 확인하여 */* 경로에 대한 GET 요청의 요청 라인과 동일한지 확인합니다. 그렇다면 `if` 블록은 HTML 파일의 내용을 반환합니다.

`request_line`이 */* 경로 에 대한 GET 요청과 같지 *않으면* 다른 요청을 수신했음을 의미합니다. 잠시 후 `else` 블록에 코드를 추가하여 다른 모든 요청에 응답합니다.

지금 이 코드를 실행하고 *127.0.0.1:7878을* 요청하십시오 . *hello.html* 에서 HTML을 가져와야 합니다. *127.0.0.1:7878/something-else* 와 같은 다른 요청을 하면 Listing 20-1 및 Listing 20-2에서 코드를 실행할 때 본 것과 같은 연결 오류가 발생합니다.

이제 Listing 20-7의 코드를 `else` 블록에 추가하여 상태 코드 404와 함께 응답을 반환하도록 합시다. 이 코드는 요청 내용을 찾을 수 없다는 신호입니다. 또한 최종 사용자에게 응답을 나타내는 브라우저에서 렌더링할 페이지에 대한 일부 HTML을 반환합니다.

파일 이름: src/main.rs

```rust
    // --snip--
    } else {
        let status_line = `HTTP/1.1 404 NOT FOUND`;
        let contents = fs::read_to_string(`404.html`).unwrap();
        let length = contents.len();

        let response = format!(
            `{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}`
        );
    
        stream.write_all(response.as_bytes()).unwrap();
    }
```

*목록 20-7: /* 이외의 것이 요청된 경우 상태 코드 404 및 오류 페이지로 응답

여기에서 응답에는 상태 코드 404와 이유 문구 `NOT FOUND`가 있는 상태 표시줄이 있습니다. 응답 본문은 *404.html* 파일의 HTML입니다. 오류 페이지에 대한 *hello.html* 옆에 *404.html* 파일을 만들어야 합니다. 원하는 HTML을 자유롭게 사용하거나 Listing 20-8에 있는 예제 HTML을 다시 사용하십시오.

파일 이름: 404.html

```html
<!DOCTYPE html>
<html lang=`en`>
  <head>
    <meta charset=`utf-8`>
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don`t know what you`re asking for.</p>
  </body>
</html>
```

목록 20-8: 404 응답과 함께 다시 보낼 페이지의 샘플 콘텐츠

이러한 변경 사항으로 서버를 다시 실행하십시오. *127.0.0.1:7878을* 요청하면 *hello.html* 의 내용이 반환되어야 하며 *127.0.0.1:7878/foo* 와 같은 다른 요청은 *404.html* 에서 오류 HTML을 반환해야 합니다.

### [리팩토링의 손길](https://doc.rust-lang.org/book/ch20-01-single-threaded.html#a-touch-of-refactoring)

현재 `if` 및 `else` 블록은 반복되는 부분이 많습니다. 둘 다 파일을 읽고 파일 내용을 스트림에 씁니다. 유일한 차이점은 상태 표시줄과 파일 이름입니다. 상태 표시줄의 값과 파일 이름을 변수에 할당하는 별도의 `if` 및 `else` 줄로 이러한 차이점을 제거하여 코드를 더 간결하게 만들어 보겠습니다. 그런 다음 코드에서 해당 변수를 무조건 사용하여 파일을 읽고 응답을 작성할 수 있습니다. 목록 20-9는 큰 `if` 및 `else` 블록을 교체한 후의 결과 코드를 보여줍니다.

파일 이름: src/main.rs

```rust
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = if request_line == `GET / HTTP/1.1` {
        (`HTTP/1.1 200 OK`, `hello.html`)
    } else {
        (`HTTP/1.1 404 NOT FOUND`, `404.html`)
    };
    
    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();
    
    let response =
        format!(`{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}`);
    
    stream.write_all(response.as_bytes()).unwrap();
}
```

목록 20-9: `if` 및 `else` 블록을 리팩터링하여 두 경우 사이에 다른 코드만 포함

이제 `if` 및 `else` 블록은 튜플의 상태 표시줄 및 파일 이름에 대한 적절한 값만 반환합니다. 그런 다음 18장에서 설명한 대로 `let` 문의 패턴을 사용하여 이 두 값을 `status_line` 및 `filename`에 할당하기 위해 구조 분해를 사용합니다.

이전에 복제된 코드는 이제 `if` 및 `else` 블록 외부에 있으며 `status_line` 및 `filename` 변수를 사용합니다. 이렇게 하면 두 경우의 차이를 더 쉽게 볼 수 있으며 파일 읽기 및 응답 쓰기 작동 방식을 변경하려는 경우 코드를 업데이트할 위치가 한 곳뿐이라는 의미입니다. Listing 20-9의 코드 동작은 Listing 20-8과 동일합니다.

엄청난! 이제 우리는 콘텐츠 페이지로 한 요청에 응답하고 404 응답으로 다른 모든 요청에 응답하는 약 40줄의 Rust 코드로 된 간단한 웹 서버를 갖게 되었습니다.

현재 우리 서버는 단일 스레드에서 실행되므로 한 번에 하나의 요청만 처리할 수 있습니다. 일부 느린 요청을 시뮬레이트하여 이것이 어떻게 문제가 될 수 있는지 살펴보겠습니다. 그런 다음 서버가 한 번에 여러 요청을 처리할 수 있도록 수정합니다.

------

## [단일 스레드 서버를 다중 스레드 서버로 전환](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#turning-our-single-threaded-server-into-a-multithreaded-server)

현재 서버는 각 요청을 차례로 처리합니다. 즉, 첫 번째 연결이 완료될 때까지 두 번째 연결을 처리하지 않습니다. 서버가 점점 더 많은 요청을 받으면 이 직렬 실행은 점점 더 최적이 되지 않습니다. 서버가 처리하는 데 시간이 오래 걸리는 요청을 받으면 후속 요청은 새 요청을 빠르게 처리할 수 있더라도 긴 요청이 완료될 때까지 기다려야 합니다. 이 문제를 해결해야 하지만 먼저 작동 중인 문제를 살펴보겠습니다.

### [현재 서버 구현에서 느린 요청 시뮬레이션](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#simulating-a-slow-request-in-the-current-server-implementation)

처리 속도가 느린 요청이 현재 서버 구현에 대한 다른 요청에 어떤 영향을 미칠 수 있는지 살펴보겠습니다. 목록 20-10 은 서버가 응답하기 전에 5초 동안 휴면 상태가 되게 하는 시뮬레이션된 느린 응답으로 */sleep* 에 대한 요청 처리를 구현합니다.

파일 이름: src/main.rs

```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = match &request_line[..] {
        `GET / HTTP/1.1` => (`HTTP/1.1 200 OK`, `hello.html`),
        `GET /sleep HTTP/1.1` => {
            thread::sleep(Duration::from_secs(5));
            (`HTTP/1.1 200 OK`, `hello.html`)
        }
        _ => (`HTTP/1.1 404 NOT FOUND`, `404.html`),
    };
    
    // --snip--
}
```

목록 20-10: 5초 동안 휴면하여 느린 요청 시뮬레이션

이제 세 가지 경우가 있으므로 `if`에서 `match`로 전환했습니다. 문자열 리터럴 값에 대한 패턴 일치를 위해 `request_line` 조각에서 명시적으로 일치해야 합니다. `match`는 같음 메서드처럼 자동 참조 및 역참조를 수행하지 않습니다.

첫 번째 팔은 목록 20-9의 `if` 블록과 동일합니다. *두 번째 팔은 /sleep* 에 대한 요청과 일치합니다. 해당 요청이 수신되면 서버는 성공적인 HTML 페이지를 렌더링하기 전에 5초 동안 휴면 상태가 됩니다. 세 번째 팔은 목록 20-9의 `else` 블록과 동일합니다.

우리 서버가 얼마나 원시적인지 알 수 있습니다. 실제 라이브러리는 훨씬 덜 장황한 방식으로 여러 요청의 인식을 처리합니다!

`cargo run`을 사용하여 서버를 시작하십시오. 그런 다음 두 개의 브라우저 창을 엽니다. 하나는 *http://127.0.0.1:7878/* 용 이고 다른 하나는 *http://127.0.0.1:7878/sleep* 용입니다. *이전처럼 /* URI를 몇 번 입력하면 빠르게 응답하는 것을 볼 수 있습니다. *그러나 /sleep을* 입력한 다음 */를 로드하면 로드하기 전에 `sleep`이 전체 5초 동안 절전 모드로 전환될 때까지* */* 가 대기하는 것을 볼 수 있습니다.

느린 요청 뒤에 요청이 백업되는 것을 방지하기 위해 사용할 수 있는 여러 기술이 있습니다. 우리가 구현할 것은 스레드 풀입니다.

### [스레드 풀로 처리량 향상](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#improving-throughput-with-a-thread-pool)

스레드 *풀은* 작업을 처리할 준비가 되어 대기 중인 생성된 스레드 그룹입니다. 프로그램이 새 작업을 수신하면 풀의 스레드 중 하나를 작업에 할당하고 해당 스레드가 작업을 처리합니다. 풀의 나머지 스레드는 첫 번째 스레드가 처리되는 동안 들어오는 다른 작업을 처리하는 데 사용할 수 있습니다. 첫 번째 스레드가 작업 처리를 완료하면 새 작업을 처리할 준비가 된 유휴 스레드 풀로 반환됩니다. 스레드 풀을 사용하면 연결을 동시에 처리하여 서버의 처리량을 높일 수 있습니다.

서비스 거부(DoS) 공격으로부터 보호하기 위해 풀의 스레드 수를 적은 수로 제한합니다. 프로그램이 요청이 들어올 때마다 새 스레드를 생성하도록 하면 서버에 천만 건의 요청을 하는 누군가가 서버의 리소스를 모두 사용하고 요청 처리를 중단시켜 대혼란을 일으킬 수 있습니다.

그러면 무제한 스레드를 생성하는 대신 풀에서 대기 중인 고정된 수의 스레드를 갖게 됩니다. 들어오는 요청은 처리를 위해 풀로 전송됩니다. 풀은 들어오는 요청의 대기열을 유지합니다. 풀의 각 스레드는 이 큐에서 요청을 팝 오프하고 요청을 처리한 다음 큐에 다른 요청을 요청합니다. 이 디자인을 사용하면 최대 `N`개의 요청을 동시에 처리할 수 있습니다. 여기서 `N`은 스레드 수입니다. 각 스레드가 장기 실행 요청에 응답하는 경우 후속 요청은 여전히 대기열에 백업될 수 있지만 해당 지점에 도달하기 전에 처리할 수 있는 장기 실행 요청의 수를 늘렸습니다.

이 기술은 웹 서버의 처리량을 향상시키는 여러 방법 중 하나일 뿐입니다. 탐색할 수 있는 다른 옵션은 *fork/join 모델* , *단일 스레드 비동기 I/O 모델* 또는 *다중 스레드 비동기 I/O 모델 입니다* . 이 항목에 관심이 있는 경우 다른 솔루션에 대해 자세히 읽고 구현해 볼 수 있습니다. Rust와 같은 저수준 언어를 사용하면 이러한 모든 옵션이 가능합니다.

스레드 풀 구현을 시작하기 전에 풀을 사용하는 모습에 대해 이야기해 봅시다. 코드를 디자인하려고 할 때 클라이언트 인터페이스를 먼저 작성하면 디자인을 안내하는 데 도움이 될 수 있습니다. 호출하려는 방식으로 구조화되도록 코드의 API를 작성합니다. 그런 다음 기능을 구현한 다음 공용 API를 설계하는 대신 해당 구조 내에서 기능을 구현합니다.

12장에서 프로젝트에서 테스트 기반 개발을 사용한 방법과 유사하게 여기서는 컴파일러 기반 개발을 사용합니다. 원하는 함수를 호출하는 코드를 작성한 다음 컴파일러의 오류를 살펴보고 코드가 작동하도록 하기 위해 다음에 무엇을 변경해야 하는지 결정합니다. 그러나 그 전에 시작점으로 사용하지 않을 기술을 살펴보겠습니다.

#### [각 요청에 대한 스레드 생성](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#spawning-a-thread-for-each-request)

먼저 모든 연결에 대해 새 스레드를 생성한 경우 코드가 어떻게 표시되는지 살펴보겠습니다. 앞에서 언급한 바와 같이 잠재적으로 무제한의 스레드를 생성할 수 있는 문제로 인해 이것이 우리의 최종 계획은 아니지만 작동하는 다중 스레드 서버를 먼저 얻는 출발점입니다. 그런 다음 스레드 풀을 개선 사항으로 추가하고 두 솔루션을 대조하는 것이 더 쉬울 것입니다. 목록 20-11은 `for` 루프 내에서 각 스트림을 처리하기 위해 새 스레드를 생성하기 위해 `main`에 대한 변경 사항을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let listener = TcpListener::bind(`127.0.0.1:7878`).unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
```

Listing 20-11: 각 스트림에 대한 새 스레드 생성

16장에서 배운 것처럼 `thread::spawn`은 새 스레드를 생성한 다음 클로저의 코드를 새 스레드에서 실행합니다. *이* 코드를 실행하고 브라우저에서 */sleep을 로드하면 두 개의 추가 브라우저 탭에서* */ 에 대한 요청이* */sleep이* 완료될 때까지 기다릴 필요가 없음 을 실제로 확인할 수 있습니다. 그러나 우리가 언급한 것처럼 제한 없이 새 스레드를 만들게 되므로 결국 시스템을 압도할 것입니다.

#### [한정된 수의 스레드 생성](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#creating-a-finite-number-of-threads)

우리는 스레드 풀이 유사하고 친숙한 방식으로 작동하기를 원하므로 스레드에서 스레드 풀로 전환할 때 API를 사용하는 코드를 크게 변경할 필요가 없습니다. 목록 20-12는 `thread::spawn` 대신 사용하려는 `ThreadPool` 구조체에 대한 가상 인터페이스를 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let listener = TcpListener::bind(`127.0.0.1:7878`).unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();
    
        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

목록 20-12: 이상적인 `ThreadPool` 인터페이스

`ThreadPool::new`를 사용하여 구성 가능한 수의 스레드(이 경우에는 4개)로 새 스레드 풀을 만듭니다. 그런 다음 `for` 루프에서 `pool.execute`는 풀이 각 스트림에 대해 실행되어야 하는 클로저를 취한다는 점에서 `thread::spawn`과 유사한 인터페이스를 가집니다. `pool.execute`를 구현해야 클로저를 가져와 풀의 스레드에 제공하여 실행할 수 있습니다. 이 코드는 아직 컴파일되지 않지만 컴파일러가 수정 방법을 안내할 수 있도록 노력하겠습니다.

#### [컴파일러 기반 개발을 사용하여 `ThreadPool` 구축](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#building-threadpool-using-compiler-driven-development)

*Listing 20-12에서 src/main.rs* 로 변경한 다음 `cargo check`의 컴파일러 오류를 사용하여 개발을 추진해 봅시다. 다음은 첫 번째 오류입니다.

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:11:16
   |
11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`

For more information about this error, try `rustc --explain E0433`.
error: could not compile `hello` due to previous error
```

엄청난! 이 오류는 `ThreadPool` 유형 또는 모듈이 필요하다는 것을 알려주므로 지금 빌드하겠습니다. 우리의 `ThreadPool` 구현은 웹 서버가 수행하는 작업 종류와 독립적입니다. 따라서 `hello` 크레이트를 바이너리 크레이트에서 라이브러리 크레이트로 전환하여 `ThreadPool` 구현을 유지해 보겠습니다. 라이브러리 크레이트로 변경한 후에는 웹 요청을 제공하는 것뿐만 아니라 스레드 풀을 사용하여 수행하려는 모든 작업에 대해 별도의 스레드 풀 라이브러리를 사용할 수도 있습니다.

*다음을 포함하는 src/lib.rs를* 생성합니다. 이것은 현재 우리가 가질 수 있는 `ThreadPool` 구조체의 가장 간단한 정의입니다.

파일 이름: src/lib.rs

```rust
pub struct ThreadPool;
```

그런 다음 *src/main.rs* 맨 위에 다음 코드를 추가하여 라이브러리 크레이트에서 `ThreadPool`을 범위로 가져오도록 *main.rs* 파일을 편집합니다.

파일 이름: src/main.rs

```rust
use hello::ThreadPool;
```

이 코드는 여전히 작동하지 않지만 다시 확인하여 해결해야 할 다음 오류를 가져옵니다.

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` due to previous error
```

이 오류는 다음에 `ThreadPool`에 대해 `new`라는 관련 함수를 만들어야 함을 나타냅니다. 또한 `new`에는 `4`를 인수로 허용하고 `ThreadPool` 인스턴스를 반환할 수 있는 매개 변수가 하나 있어야 한다는 것도 알고 있습니다. 이러한 특성을 갖는 가장 간단한 `새` 함수를 구현해 보겠습니다.

파일 이름: src/lib.rs

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

`크기` 매개변수의 유형으로 `usize`를 선택했습니다. 스레드 수가 음수인 것은 의미가 없다는 것을 알고 있기 때문입니다. 우리는 또한 3장의 [`정수 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types) 섹션에서 논의한 것처럼 `usize` 유형이 필요한 스레드 모음의 요소 수로 이 4를 사용할 것임을 알고 있습니다.

코드를 다시 확인해 보겠습니다.

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |              ^^^^^^^ method not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` due to previous error
```

이제 `ThreadPool`에 `execute` 메서드가 없기 때문에 오류가 발생합니다. 스레드 풀이 `thread::spawn`과 유사한 인터페이스를 가져야 한다고 결정한 [`유한한 수의 스레드 만들기`](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#creating-a-finite-number-of-threads) 섹션을 기억하십시오 . 또한 `실행` 기능을 구현하여 주어진 클로저를 가져와 풀의 유휴 스레드에 제공하여 실행할 수 있도록 합니다.

`ThreadPool`에 `execute` 메소드를 정의하여 클로저를 매개변수로 사용합니다. `Fn`, `FnMut` 및 `FnOnce`의 세 가지 다른 특성을 사용하여 클로저를 매개변수로 사용할 수 있는 13장의 [`캡처된 값을 클로저 외부로 이동 및 `Fn` 특성](https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits) ` 섹션 에서 기억하십시오 . 여기서 어떤 종류의 클로저를 사용할지 결정해야 합니다. 우리는 표준 라이브러리 `thread::spawn` 구현과 유사한 작업을 수행하게 될 것임을 알고 있으므로 `thread::spawn`의 서명이 해당 매개 변수에 어떤 범위를 갖는지 확인할 수 있습니다. 문서는 다음을 보여줍니다.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + `static,
        T: Send + `static,
```

`F` 유형 매개변수는 여기서 우리가 관심을 갖는 매개변수입니다. `T` 유형 매개변수는 반환 값과 관련이 있으며 우리는 그것에 관심이 없습니다. `spawn`이 `F`에 바인딩된 특성으로 `FnOnce`를 사용하는 것을 볼 수 있습니다. `execute`에서 얻은 인수를 `spawn`으로 전달할 것이기 때문에 이것은 아마도 우리가 원하는 것일 수도 있습니다. `FnOnce`는 요청을 실행하는 스레드가 해당 요청의 클로저를 한 번만 실행하므로 `FnOnce`의 `Once`와 일치하기 때문에 `FnOnce`가 우리가 사용하려는 특성임을 확신할 수 있습니다.

`F` 유형 매개변수에는 특성 바인딩 `Send`와 수명 바인딩 ``정적`이 있으며 이는 우리 상황에서 유용합니다. 클로저를 한 스레드에서 다른 스레드로 전송하려면 `Send`가 필요하고 `정적`은 `정적`이기 때문입니다. 우리는 스레드가 실행되는 데 얼마나 오래 걸릴지 모릅니다. `F` 유형의 일반 매개변수를 사용하는 `ThreadPool`에서 `execute` 메서드를 만들어 보겠습니다.

파일 이름: src/lib.rs

```rust
impl ThreadPool {
    // --snip--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + `static,
    {
    }
}
```

이 `FnOnce`는 매개변수를 받지 않고 단위 유형 `()`을 반환하는 클로저를 나타내기 때문에 여전히 `FnOnce` 뒤에 `()`를 사용합니다. 함수 정의와 마찬가지로 서명에서 반환 유형을 생략할 수 있지만 매개 변수가 없더라도 여전히 괄호가 필요합니다.

다시 말하지만 이것은 `execute` 메서드의 가장 간단한 구현입니다. 아무 작업도 수행하지 않지만 우리는 코드를 컴파일하려고만 합니다. 다시 확인해 봅시다:

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
```

컴파일됩니다! 그러나 `cargo run`을 시도하고 브라우저에서 요청을 하면 이 장의 시작 부분에서 보았던 오류가 브라우저에서 표시됩니다. 우리 라이브러리는 아직 `실행`으로 전달된 클로저를 실제로 호출하지 않습니다!

> 참고: Haskell 및 Rust와 같이 엄격한 컴파일러를 사용하는 언어에 대해 들을 수 있는 말은 `코드가 컴파일되면 작동합니다.`입니다. 그러나 이 말은 보편적인 사실이 아니다. 우리 프로젝트는 컴파일되지만 아무것도 하지 않습니다! 실제 완전한 프로젝트를 구축하고 있다면 코드가 컴파일되고 원하는 동작이 있는지 확인하기 위해 단위 테스트 작성을 시작하기에 좋은 시기 *입니다* .

#### [`new`의 스레드 수 유효성 검사](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#validating-the-number-of-threads-in-new)

우리는 `new`와 `execute`에 대한 매개변수로 아무것도 하지 않습니다. 원하는 동작으로 이러한 함수의 본문을 구현해 보겠습니다. 시작하려면 `새로운`에 대해 생각해 봅시다. 이전에는 `크기` 매개변수에 대해 부호 없는 유형을 선택했습니다. 스레드 수가 음수인 풀은 의미가 없기 때문입니다. 그러나 스레드가 0인 풀도 의미가 없지만 0은 완벽하게 유효한 `사용`입니다. `ThreadPool` 인스턴스를 반환하기 전에 `크기`가 0보다 큰지 확인하는 코드를 추가하고 `assert!` 목록 20-13에 표시된 대로 매크로.

파일 이름: src/lib.rs

```rust
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }
    
    // --snip--
}
```

목록 20-13: `크기`가 0인 경우 `ThreadPool::new` 패닉 구현

문서 주석과 함께 `ThreadPool`에 대한 일부 문서도 추가했습니다. 14장에서 설명한 것처럼 함수가 패닉할 수 있는 상황을 호출하는 섹션을 추가하여 좋은 문서화 방법을 따랐습니다. `cargo doc --open`을 실행하고 `ThreadPool` 구조체를 클릭하여 `신규`에 대한 문서는 다음과 같습니다!

`assert!`를 추가하는 대신 여기에서 수행한 것처럼 `new`를 `build`로 변경하고 Listing 12-9의 I/O 프로젝트에서 `Config::build`로 수행한 것처럼 `Result`를 반환할 수 있습니다. 그러나 이 경우 스레드 없이 스레드 풀을 만들려고 하면 복구할 수 없는 오류가 발생한다고 결정했습니다. 야망이 있다면 `new` 함수와 비교하기 위해 다음 서명을 사용하여 `build`라는 함수를 작성해 보십시오.

```rust
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### [스레드를 저장할 공간 만들기](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#creating-space-to-store-the-threads)

이제 풀에 저장할 유효한 수의 스레드가 있는지 알 수 있는 방법이 있으므로 해당 스레드를 만들고 구조체를 반환하기 전에 `ThreadPool` 구조체에 저장할 수 있습니다. 하지만 스레드를 어떻게 `저장`합니까? `thread::spawn` 서명을 다시 살펴보겠습니다.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + `static,
        T: Send + `static,
```

`spawn` 함수는 `JoinHandle`을 반환합니다.`, 여기서 `T`는 클로저가 반환하는 유형입니다. `JoinHandle`도 사용해보고 무슨 일이 일어나는지 봅시다. 우리의 경우 스레드 풀에 전달하는 클로저는 연결을 처리하고 아무 것도 반환하지 않으므로 `T`는 단위 유형 `()`이 됩니다.

목록 20-14의 코드는 컴파일되지만 아직 스레드를 생성하지는 않습니다. 우리는 `thread::JoinHandle<()>` 인스턴스의 벡터를 보유하도록 `ThreadPool`의 정의를 변경했고, `크기` 용량으로 벡터를 초기화했으며, 일부 코드를 실행할 `for` 루프를 설정했습니다. 스레드를 생성하고 이를 포함하는 `ThreadPool` 인스턴스를 반환했습니다.

파일 이름: src/lib.rs

```rust
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);
    
        for _ in 0..size {
            // create some threads and store them in the vector
        }
    
        ThreadPool { threads }
    }
    // --snip--
}
```

목록 20-14: 스레드를 보관할 `ThreadPool`용 벡터 생성

`ThreadPool`의 벡터 항목 유형으로 `thread::JoinHandle`을 사용하고 있기 때문에 `std::thread`를 라이브러리 크레이트의 범위로 가져왔습니다.

유효한 크기가 수신되면 `ThreadPool`은 `크기` 항목을 저장할 수 있는 새 벡터를 생성합니다. `with_capacity` 함수는 `Vec::new`와 같은 작업을 수행하지만 중요한 차이점이 있습니다. 벡터에 공간을 미리 할당합니다. 벡터에 `크기` 요소를 저장해야 한다는 것을 알고 있기 때문에 이 할당을 미리 수행하는 것이 요소가 삽입될 때 자체 크기를 조정하는 `Vec::new`를 사용하는 것보다 약간 더 효율적입니다.

`cargo check`를 다시 실행하면 성공해야 합니다.

#### [`ThreadPool`에서 스레드로 코드 전송을 담당하는 `작업자` 구조체](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread)

스레드 생성과 관련하여 Listing 20-14의 `for` 루프에 주석을 남겼습니다. 여기서는 실제로 스레드를 만드는 방법을 살펴보겠습니다. 표준 라이브러리는 스레드를 생성하는 방법으로 `thread::spawn`을 제공하고 `thread::spawn`은 스레드가 생성되자마자 스레드가 실행해야 하는 일부 코드를 얻을 것으로 예상합니다. 그러나 우리의 경우에는 스레드를 만들고 나중에 보낼 코드를 *기다리게* 하려고 합니다. 표준 라이브러리의 스레드 구현에는 이를 수행하는 방법이 포함되어 있지 않습니다. 수동으로 구현해야 합니다.

`ThreadPool`과 이 새로운 동작을 관리할 스레드 사이에 새로운 데이터 구조를 도입하여 이 동작을 구현할 것입니다. 우리는 이 데이터 구조를 *Worker 라고* 부를 것입니다. 이는 풀링 구현에서 일반적인 용어입니다. Worker는 실행해야 하는 코드를 선택하고 Worker의 스레드에서 코드를 실행합니다. 식당의 주방에서 일하는 사람들을 생각해 보십시오. 직원들은 고객의 주문이 들어올 때까지 기다렸다가 주문을 받고 처리하는 일을 담당합니다.

스레드 풀에 `JoinHandle<()>` 인스턴스의 벡터를 저장하는 대신 `Worker` 구조체의 인스턴스를 저장합니다. 각 `작업자`는 단일 `JoinHandle<()>` 인스턴스를 저장합니다. 그런 다음 실행할 코드의 클로저를 가져와 실행을 위해 이미 실행 중인 스레드로 보내는 메서드를 `Worker`에 구현합니다. 또한 각 작업자에게 `id`를 부여하여 로깅 또는 디버깅할 때 풀의 서로 다른 작업자를 구별할 수 있습니다.

다음은 `ThreadPool`을 만들 때 발생하는 새로운 프로세스입니다. 다음과 같은 방식으로 `Worker`를 설정한 후 클로저를 스레드로 보내는 코드를 구현합니다.

1. `id`와 `JoinHandle<()>`을 보유하는 `Worker` 구조체를 정의합니다.
2. `작업자` 인스턴스의 벡터를 보유하도록 `ThreadPool`을 변경합니다.
3. `id` 번호를 사용하고 `id`를 보유하는 `Worker` 인스턴스와 빈 클로저로 생성된 스레드를 반환하는 `Worker::new` 함수를 정의합니다.
4. `ThreadPool::new`에서 `for` 루프 카운터를 사용하여 `id`를 생성하고 해당 `id`로 새 `Worker`를 만들고 작업자를 벡터에 저장합니다.

도전하고 싶다면 목록 20-15의 코드를 보기 전에 이러한 변경 사항을 직접 구현해 보십시오.

준비가 된? 다음은 이전 수정을 수행하는 한 가지 방법이 있는 목록 20-15입니다.

파일 이름: src/lib.rs

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id));
        }
    
        ThreadPool { workers }
    }
    // --snip--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
```

목록 20-15: 스레드를 직접 보유하는 대신 `Worker` 인스턴스를 보유하도록 `ThreadPool` 수정

이제 `JoinHandle<()>` 인스턴스 대신 `Worker` 인스턴스를 보유하고 있기 때문에 `ThreadPool`의 필드 이름을 `threads`에서 `workers`로 변경했습니다. 우리는 `Worker::new`에 대한 인수로 `for` 루프의 카운터를 사용하고 각각의 새로운 `Worker`를 `workers`라는 벡터에 저장합니다.

*외부 코드( src/main.rs* 의 서버와 같은 )는 `ThreadPool` 내에서 `Worker` 구조체 사용에 관한 구현 세부 정보를 알 필요가 없으므로 `Worker` 구조체와 해당 `new` 함수를 비공개로 만듭니다. `Worker::new` 함수는 우리가 제공한 `id`를 사용하고 빈 클로저를 사용하여 새 스레드를 생성하여 생성된 `JoinHandle<()>` 인스턴스를 저장합니다.

> 참고: 시스템 리소스가 부족하여 운영 체제에서 스레드를 생성할 수 없는 경우 `thread::spawn`이 패닉 상태가 됩니다. 그러면 일부 스레드 생성이 성공하더라도 전체 서버가 패닉 상태가 됩니다. [단순함을 위해 이 동작은 괜찮지만 프로덕션 스레드 풀 구현에서는 `std:🧵:Builder`](https://doc.rust-lang.org/std/thread/struct.Builder.html) 및 `Result`를 반환하는 [`spawn`](https://doc.rust-lang.org/std/thread/struct.Builder.html#method.spawn) 메서드를 대신 사용할 수 있습니다.

이 코드는 `ThreadPool::new`에 대한 인수로 지정한 `Worker` 인스턴스의 수를 컴파일하고 저장합니다. 그러나 우리는 *여전히* `실행`에서 얻은 클로저를 처리하지 않습니다. 다음에 그 방법을 살펴보겠습니다.

#### [채널을 통해 스레드에 요청 보내기](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#sending-requests-to-threads-via-channels)

우리가 다룰 다음 문제는 `thread::spawn`에 주어진 클로저가 아무것도 하지 않는다는 것입니다. 현재 `execute` 메서드에서 실행하려는 클로저를 얻습니다. 그러나 `ThreadPool`을 생성하는 동안 각 `Worker`를 생성할 때 실행할 클로저를 `thread::spawn`에 제공해야 합니다.

방금 생성한 `Worker` 구조체가 `ThreadPool`에 보관된 대기열에서 실행할 코드를 가져오고 해당 코드를 스레드로 전송하여 실행하기를 원합니다.

16장에서 배운 채널(두 스레드 간에 통신하는 간단한 방법)은 이 사용 사례에 적합합니다. 채널을 사용하여 작업 큐로 작동하고 `execute`는 작업을 `ThreadPool`에서 `Worker` 인스턴스로 보내고 작업을 해당 스레드로 보냅니다. 계획은 다음과 같습니다.

1. `ThreadPool`은 채널을 만들고 발신자를 유지합니다.
2. 각 `작업자`는 수신자를 붙잡습니다.
3. 우리는 채널 아래로 보내려는 클로저를 보유할 새로운 `Job` 구조체를 생성할 것입니다.
4. `execute` 메소드는 송신자를 통해 실행하려는 작업을 송신합니다.
5. 스레드에서 `작업자`는 수신자를 반복하고 수신한 모든 작업의 클로저를 실행합니다.

`ThreadPool::new`에 채널을 생성하고 Listing 20-16과 같이 `ThreadPool` 인스턴스에 발신자를 유지하는 것으로 시작하겠습니다. `Job` 구조체는 지금은 아무 것도 담지 않지만 채널 아래로 보내는 항목 유형이 됩니다.

파일 이름: src/lib.rs

```rust
use std::{sync::mpsc, thread};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id));
        }
    
        ThreadPool { workers, sender }
    }
    // --snip--
}
```

Listing 20-16: `Job` 인스턴스를 전송하는 채널의 발신자를 저장하도록 `ThreadPool` 수정

`ThreadPool::new`에서 새 채널을 만들고 풀이 발신자를 보유하도록 합니다. 이것은 성공적으로 컴파일됩니다.

스레드 풀이 채널을 생성할 때 채널의 수신자를 각 작업자에게 전달해 봅시다. 작업자가 생성하는 스레드에서 리시버를 사용하기를 원하므로 클로저에서 `receiver` 매개변수를 참조할 것입니다. 목록 20-17의 코드는 아직 제대로 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }
    
        ThreadPool { workers, sender }
    }
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
```

Listing 20-17: 리시버를 워커에게 넘기기

우리는 몇 가지 작고 간단한 변경을 했습니다. 수신기를 `Worker::new`로 전달한 다음 클로저 내부에서 사용합니다.

이 코드를 확인하려고 하면 다음 오류가 발생합니다.

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:26:42
   |
21 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because `receiver` has type `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
...
26 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello` due to previous error
```

코드는 `수신자`를 여러 `작업자` 인스턴스에 전달하려고 합니다. 이것은 작동하지 않을 것입니다. 16장에서 기억할 것입니다: Rust가 제공하는 채널 구현은 다중 *생산자* , 단일 *소비자* 입니다. 즉, 이 코드를 수정하기 위해 채널의 소비측 끝을 복제할 수 없습니다. 또한 여러 소비자에게 메시지를 여러 번 보내고 싶지 않습니다. 우리는 각 메시지가 한 번 처리되도록 여러 작업자가 있는 하나의 메시지 목록을 원합니다.

또한 채널 대기열에서 작업을 가져오려면 `수신자`를 변경해야 하므로 스레드는 `수신자`를 공유하고 수정할 수 있는 안전한 방법이 필요합니다. 그렇지 않으면 경쟁 조건이 발생할 수 있습니다(16장에서 설명).

16장에서 논의한 스레드 안전 스마트 포인터를 상기하십시오. 여러 스레드 간에 소유권을 공유하고 스레드가 값을 변경할 수 있도록 하려면 `Arc<Mutex`를 사용해야 합니다.>`. `Arc` 유형은 여러 작업자가 수신기를 소유하도록 하고 `Mutex`는 한 번에 한 작업자만 수신기에서 작업을 가져오도록 합니다. 목록 20-18은 우리가 만들어야 하는 변경 사항을 보여줍니다.

파일 이름: src/lib.rs

```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};
// --snip--

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let receiver = Arc::new(Mutex::new(receiver));
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
    
        ThreadPool { workers, sender }
    }
    
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
}
```

Listing 20-18: `Arc` 및 `Mutex`를 사용하여 작업자 간에 수신기 공유

`ThreadPool::new`에서 수신기를 `Arc` 및 `Mutex`에 넣습니다. 각각의 새 작업자에 대해 `Arc`를 복제하여 참조 횟수를 늘리면 작업자가 수신기의 소유권을 공유할 수 있습니다.

이러한 변경으로 코드가 컴파일됩니다! 우리는 거기에 도달하고 있습니다!

#### [`실행` 방법 구현](https://doc.rust-lang.org/book/ch20-02-multithreaded.html#implementing-the-execute-method)

마지막으로 `ThreadPool`에 `execute` 메서드를 구현해 봅시다. 또한 구조체에서 `실행`이 받는 클로저 유형을 보유하는 특성 개체의 유형 별칭으로 `Job`을 변경할 것입니다. [19장의 `유형 별칭으로 유형 동의어 만들기`](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases) 섹션 에서 설명한 것처럼 유형 별칭을 사용하면 긴 유형을 사용하기 쉽도록 더 짧게 만들 수 있습니다. 목록 20-19를 보십시오.

파일 이름: src/lib.rs

```rust
// --snip--

type Job = Box<dyn FnOnce() + Send + `static>;

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + `static,
    {
        let job = Box::new(f);
    
        self.sender.send(job).unwrap();
    }
}

// --snip--
```

목록 20-19: 각 클로저를 보유하고 있는 `Box`에 대한 `Job` 유형 별칭을 생성한 다음 작업을 채널 아래로 전송

클로저를 사용하여 새 `Job` 인스턴스를 생성한 후 `실행`에서 해당 작업을 채널의 전송 끝으로 보냅니다. 전송에 실패한 경우를 위해 `send`에서 `unwrap`을 호출합니다. 예를 들어 모든 스레드의 실행을 중지하면 수신 측에서 새 메시지 수신을 중지한 경우에 이러한 상황이 발생할 수 있습니다. 현재로서는 스레드 실행을 중지할 수 없습니다. 풀이 존재하는 한 스레드는 계속 실행됩니다. `unwrap`을 사용하는 이유는 실패 사례가 발생하지 않는다는 것을 알고 있지만 컴파일러는 이를 알지 못하기 때문입니다.

하지만 아직 끝나지 않았습니다! 작업자에서 `thread::spawn`으로 전달되는 클로저는 여전히 채널의 수신측만 *참조합니다.* 대신에 우리는 채널의 수신 측에 작업을 요청하고 작업을 받으면 작업을 실행하면서 영원히 반복되는 클로저가 필요합니다. 목록 20-20에 표시된 `Worker::new`로 변경해 보겠습니다.

파일 이름: src/lib.rs

```rust
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!(`Worker {id} got a job; executing.`);
    
            job();
        });
    
        Worker { id, thread }
    }
}
```

목록 20-20: 작업자의 스레드에서 작업 수신 및 실행

여기에서 먼저 `수신기`에서 `잠금`을 호출하여 뮤텍스를 획득한 다음 `unwrap`을 호출하여 오류가 있을 때 당황하게 합니다. *뮤텍스가 오염된* 상태 인 경우 잠금 획득이 실패할 수 있습니다. 이는 잠금을 해제하지 않고 잠금을 유지하는 동안 다른 스레드가 패닉 상태가 된 경우 발생할 수 있습니다. 이 상황에서 `unwrap`을 호출하여 이 스레드를 패닉 상태로 만드는 것이 올바른 조치입니다. 이 `unwrap`을 의미 있는 오류 메시지와 함께 `expect`로 자유롭게 변경하십시오.

뮤텍스를 잠그면 채널에서 `작업`을 수신하기 위해 `recv`를 호출합니다. 최종 `unwrap`은 여기서도 모든 오류를 지나서 이동합니다. 이는 보낸 사람을 보유하고 있는 스레드가 종료된 경우 발생할 수 있습니다. 이는 받는 사람이 종료된 경우 `send` 메서드가 `Err`를 반환하는 방식과 유사합니다.

`recv`에 대한 호출이 차단되므로 아직 작업이 없으면 현재 스레드는 작업을 사용할 수 있을 때까지 대기합니다. `뮤텍스`는 한 번에 하나의 `작업자` 스레드만 작업 요청을 시도하도록 합니다.

이제 스레드 풀이 작동 상태에 있습니다! `화물 실행`을 제공하고 몇 가지 요청을 합니다.

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

성공! 이제 연결을 비동기적으로 실행하는 스레드 풀이 있습니다. 4개 이상의 스레드가 생성되지 않으므로 서버가 많은 요청을 수신하더라도 시스템이 과부하되지 않습니다. */sleep* 에 요청을 하면 서버는 다른 스레드가 요청을 실행하도록 하여 다른 요청을 처리할 수 있습니다.

> 참고: 여러 브라우저 창에서 동시에 */sleep을* 열면 5초 간격으로 한 번에 하나씩 로드될 수 있습니다. 일부 웹 브라우저는 캐싱을 이유로 동일한 요청의 여러 인스턴스를 순차적으로 실행합니다. 이 제한은 웹 서버에 의해 발생하지 않습니다.

18장에서 `while let` 루프에 대해 배운 후 목록 20-21에 표시된 것처럼 작업자 스레드 코드를 작성하지 않은 이유가 궁금할 수 있습니다.

파일 이름: src/lib.rs

```rust
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            while let Ok(job) = receiver.lock().unwrap().recv() {
                println!(`Worker {id} got a job; executing.`);

                job();
            }
        });
    
        Worker { id, thread }
    }
}
```

목록 20-21: `while let`을 사용한 `Worker::new`의 대체 구현

이 코드는 컴파일되고 실행되지만 원하는 스레딩 동작이 발생하지 않습니다. 느린 요청으로 인해 다른 요청이 처리되기를 기다리게 됩니다. 그 이유는 다소 미묘합니다. 잠금 소유권이 `MutexGuard`의 수명을 기반으로 하기 때문에 `Mutex` 구조체에는 공용 `잠금 해제` 메서드가 없습니다.` 내의 `LockResult<MutexGuard`잠금` 메서드가 반환하는 `>`입니다. 컴파일 시간에 차용 검사기는 잠금을 보유하지 않는 한 `Mutex`로 보호되는 리소스에 액세스할 수 없다는 규칙을 적용할 수 있습니다. 그러나 이 구현으로 인해 잠금이 발생할 수도 있습니다. `MutexGuard의 수명을 염두에 두지 않으면 의도한 것보다 오래 유지됩니다.`.

`let job = receiver.lock().unwrap().recv().unwrap();`을 사용하는 Listing 20-20의 코드 `let`을 사용하면 등호 오른쪽에 있는 표현식에 사용된 모든 임시 값이 `let` 문이 끝날 때 즉시 삭제되기 때문에 작동합니다. 그러나 `while let`(및 `if let` 및 `match`)은 연결된 블록이 끝날 때까지 임시 값을 삭제하지 않습니다. 목록 20-21에서 잠금은 `job()` 호출 기간 동안 유지되며 이는 다른 작업자가 작업을 받을 수 없음을 의미합니다.

------

## [정상적인 종료 및 정리](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#graceful-shutdown-and-cleanup)

목록 20-20의 코드는 우리가 의도한 대로 스레드 풀을 사용하여 비동기적으로 요청에 응답합니다. 사용하지 않는 `workers`, `id` 및 `thread` 필드에 대한 경고가 표시되어 아무것도 정리하지 않는다는 것을 상기시켜 줍니다. 덜 우아한 ctrl-c 방법을 사용하여 기본 스레드를 중지하면 요청을 처리하는 중이더라도 다른 모든 스레드도 즉시 중지됩니다.

그런 다음 풀의 각 스레드에서 `연결`을 호출하는 `삭제` 특성을 구현하여 닫히기 전에 작업 중인 요청을 완료할 수 있도록 합니다. 그런 다음 새 요청 수락을 중지하고 종료해야 한다고 스레드에 알리는 방법을 구현할 것입니다. 이 코드가 실제로 작동하는지 확인하기 위해 스레드 풀을 정상적으로 종료하기 전에 두 개의 요청만 수락하도록 서버를 수정합니다.

### [`ThreadPool`에서 `Drop` 특성 구현](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#implementing-the-drop-trait-on-threadpool)

스레드 풀에서 `Drop`을 구현하는 것부터 시작하겠습니다. 풀이 삭제되면 스레드가 모두 합류하여 작업을 완료했는지 확인해야 합니다. 목록 20-22는 `Drop` 구현의 첫 번째 시도를 보여줍니다. 이 코드는 아직 작동하지 않습니다.

파일 이름: src/lib.rs

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!(`Shutting down worker {}`, worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

목록 20-22: 스레드 풀이 범위를 벗어날 때 각 스레드 조인

먼저 각 스레드 풀 `작업자`를 반복합니다. `self`는 변경 가능한 참조이고 `worker`도 변경할 수 있어야 하기 때문에 `&mut`를 사용합니다. 각 작업자에 대해 이 특정 작업자가 종료된다는 메시지를 인쇄한 다음 해당 작업자의 스레드에서 `join`을 호출합니다. `join` 호출이 실패하면 `unwrap`을 사용하여 Rust를 패닉 상태로 만들고 부적절하게 종료합니다.

다음은 이 코드를 컴파일할 때 발생하는 오류입니다.

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:52:13
   |
52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ ------ `worker.thread` moved due to this method call
   |             |
   |             move occurs because `worker.thread` has type `JoinHandle<()>`, which does not implement the `Copy` trait
   |
note: this function takes ownership of the receiver `self`, which moves `worker.thread`
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:1581:17

For more information about this error, try `rustc --explain E0507`.
error: could not compile `hello` due to previous error
```

이 오류는 각 `작업자`의 변경 가능한 빌림만 있고 `조인`이 해당 인수의 소유권을 가지기 때문에 `조인`을 호출할 수 없음을 알려줍니다. 이 문제를 해결하려면 `조인`이 스레드를 사용할 수 있도록 `스레드`를 소유한 `작업자` 인스턴스 밖으로 스레드를 이동해야 합니다. Listing 17-15에서 이를 수행했습니다. `Worker`가 `Option<thread::JoinHandle<()>>`를 보유하는 경우 `Option`에서 `take` 메소드를 호출하여 값을 외부로 이동할 수 있습니다. `일부` 변형을 선택하고 그 자리에 `없음` 변형을 남겨둡니다. 즉, 실행 중인 `Worker`는 `thread`에 `Some` 변형을 가지며 `Worker`를 정리하려는 경우 `Some`을 `None`으로 대체하므로 `

따라서 우리는 `Worker`의 정의를 다음과 같이 업데이트하고 싶다는 것을 알고 있습니다.

파일 이름: src/lib.rs

```rust
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

이제 변경해야 할 다른 위치를 찾기 위해 컴파일러에 의존해 봅시다. 이 코드를 확인하면 두 가지 오류가 발생합니다.

```bash
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `join` found for enum `Option` in the current scope
  --> src/lib.rs:52:27
   |
52 |             worker.thread.join().unwrap();
   |                           ^^^^ method not found in `Option<JoinHandle<()>>`
   |
note: the method `join` exists on the type `JoinHandle<()>`
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:1581:5
help: consider using `Option::expect` to unwrap the `JoinHandle<()>` value, panicking if the value is an `Option::None`
   |
52 |             worker.thread.expect(`REASON`).join().unwrap();
   |                          +++++++++++++++++

error[E0308]: mismatched types
  --> src/lib.rs:72:22
   |
72 |         Worker { id, thread }
   |                      ^^^^^^ expected enum `Option`, found struct `JoinHandle`
   |
   = note: expected enum `Option<JoinHandle<()>>`
            found struct `JoinHandle<_>`
help: try wrapping the expression in `Some`
   |
72 |         Worker { id, thread: Some(thread) }
   |                      +++++++++++++      +

Some errors have detailed explanations: E0308, E0599.
For more information about an error, try `rustc --explain E0308`.
error: could not compile `hello` due to 2 previous errors
```

`Worker::new` 끝에 있는 코드를 가리키는 두 번째 오류를 해결해 보겠습니다. 새 `Worker`를 만들 때 `Some`에 `thread` 값을 래핑해야 합니다. 이 오류를 수정하려면 다음과 같이 변경하십시오.

파일 이름: src/lib.rs

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

첫 번째 오류는 `Drop` 구현에 있습니다. 앞에서 `작업자`에서 `스레드`를 이동하기 위해 `옵션` 값에서 `테이크`를 호출하려고 한다고 언급했습니다. 다음과 같은 변경 사항이 적용됩니다.

파일 이름: src/lib.rs

```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!(`Shutting down worker {}`, worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

17장에서 설명한 것처럼 `Option`의 `take` 메소드는 `Some` 변형을 제거하고 그 자리에 `None`을 남겨둡니다. 우리는 `Some`을 분해하고 스레드를 얻기 위해 `if let`을 사용하고 있습니다. 그런 다음 스레드에서 `join`을 호출합니다. 작업자의 스레드가 이미 `None`인 경우 작업자가 이미 스레드를 정리한 것을 알고 있으므로 이 경우 아무 일도 일어나지 않습니다.

### [작업 수신을 중지하도록 스레드에 신호 보내기](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#signaling-to-the-threads-to-stop-listening-for-jobs)

모든 변경 사항을 적용하면 코드가 경고 없이 컴파일됩니다. 그러나 나쁜 소식은 이 코드가 아직 우리가 원하는 방식으로 작동하지 않는다는 것입니다. 핵심은 `Worker` 인스턴스의 스레드에 의해 실행되는 클로저의 논리입니다. 현재 우리는 `join`을 호출하지만 작업을 영원히 `루프`하기 때문에 스레드를 종료하지 않습니다. 현재 구현된 `drop`으로 `ThreadPool`을 삭제하려고 하면 메인 스레드는 첫 번째 스레드가 완료될 때까지 영원히 차단됩니다.

이 문제를 해결하려면 `ThreadPool` `drop` 구현을 변경한 다음 `Worker` 루프를 변경해야 합니다.

먼저 `ThreadPool` `drop` 구현을 변경하여 스레드가 완료되기를 기다리기 전에 `sender`를 명시적으로 삭제합니다. 목록 20-23은 `sender`를 명시적으로 삭제하기 위해 `ThreadPool`에 대한 변경 사항을 보여줍니다. `ThreadPool`에서 `sender`를 이동할 수 있도록 스레드에서 사용한 것과 동일한 `Option` 및 `take` 기술을 사용합니다.

파일 이름: src/lib.rs

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// --snip--
impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        // --snip--

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }
    
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + `static,
    {
        let job = Box::new(f);
    
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!(`Shutting down worker {}`, worker.id);
    
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

목록 20-23: 작업자 스레드를 결합하기 전에 `sender`를 명시적으로 삭제합니다.

`발신자`를 삭제하면 채널이 닫히고 메시지가 더 이상 전송되지 않음을 나타냅니다. 이 경우 작업자가 무한 루프에서 수행하는 `recv`에 대한 모든 호출은 오류를 반환합니다. 목록 20-24에서 `Worker` 루프를 변경하여 이 경우 루프를 정상적으로 종료합니다. 즉, `ThreadPool` `drop` 구현이 `join`을 호출할 때 스레드가 완료됩니다.

파일 이름: src/lib.rs

```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!(`Worker {id} got a job; executing.`);
    
                    job();
                }
                Err(_) => {
                    println!(`Worker {id} disconnected; shutting down.`);
                    break;
                }
            }
        });
    
        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

목록 20-24: `recv`가 오류를 반환할 때 루프에서 명시적으로 중단

이 코드가 동작하는 것을 보기 위해 Listing 20-25와 같이 서버를 정상적으로 종료하기 전에 두 개의 요청만 수락하도록 `main`을 수정해 보겠습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let listener = TcpListener::bind(`127.0.0.1:7878`).unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();
    
        pool.execute(|| {
            handle_connection(stream);
        });
    }
    
    println!(`Shutting down.`);
}
```

목록 20-25: 루프를 종료하여 두 개의 요청을 처리한 후 서버 종료

두 개의 요청만 처리한 후 실제 웹 서버가 종료되는 것을 원하지 않을 것입니다. 이 코드는 정상적인 종료 및 정리가 제대로 작동하고 있음을 보여줍니다.

`take` 메서드는 `Iterator` 특성에 정의되어 있으며 반복을 최대 처음 두 항목으로 제한합니다. `ThreadPool`은 `main`의 끝에서 범위를 벗어나고 `drop` 구현이 실행됩니다.

`cargo run`으로 서버를 시작하고 세 가지 요청을 합니다. 세 번째 요청은 오류가 발생하고 터미널에 다음과 유사한 출력이 표시되어야 합니다.

```bash
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

작업자 및 메시지의 다른 순서가 인쇄되는 것을 볼 수 있습니다. 메시지에서 이 코드가 어떻게 작동하는지 확인할 수 있습니다. 작업자 0과 3은 처음 두 요청을 받았습니다. 서버는 두 번째 연결 후 연결 수락을 중지했으며 `ThreadPool`의 `Drop` 구현은 작업자 3이 작업을 시작하기도 전에 실행을 시작합니다. `발신자`를 삭제하면 모든 작업자의 연결이 끊어지고 종료하라는 메시지가 표시됩니다. 작업자는 연결을 끊을 때 각각 메시지를 인쇄하고 스레드 풀은 `join`을 호출하여 각 작업자 스레드가 완료될 때까지 기다립니다.

이 특정 실행의 한 가지 흥미로운 측면에 주목하십시오. `ThreadPool`은 `sender`를 삭제하고 작업자가 오류를 수신하기 전에 작업자 0에 조인하려고 시도했습니다. 작업자 0은 아직 `recv`에서 오류를 수신하지 않았으므로 기본 작업자 0이 완료되기를 기다리는 스레드가 차단되었습니다. 그동안 작업자 3이 작업을 수신한 후 모든 스레드에서 오류를 수신했습니다. 작업자 0이 완료되면 기본 스레드는 나머지 작업자가 완료될 때까지 기다렸습니다. 그 시점에서 그들은 모두 루프를 종료하고 멈췄습니다.

축하해요! 이제 프로젝트를 완료했습니다. 스레드 풀을 사용하여 비동기적으로 응답하는 기본 웹 서버가 있습니다. 풀의 모든 스레드를 정리하는 서버의 정상적인 종료를 수행할 수 있습니다.

참조를 위한 전체 코드는 다음과 같습니다.

파일 이름: src/main.rs

```rust
use hello::ThreadPool;
use std::fs;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind(`127.0.0.1:7878`).unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();
    
        pool.execute(|| {
            handle_connection(stream);
        });
    }
    
    println!(`Shutting down.`);
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b`GET / HTTP/1.1\r\n`;
    let sleep = b`GET /sleep HTTP/1.1\r\n`;
    
    let (status_line, filename) = if buffer.starts_with(get) {
        (`HTTP/1.1 200 OK`, `hello.html`)
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        (`HTTP/1.1 200 OK`, `hello.html`)
    } else {
        (`HTTP/1.1 404 NOT FOUND`, `404.html`)
    };
    
    let contents = fs::read_to_string(filename).unwrap();
    
    let response = format!(
        `{}\r\nContent-Length: {}\r\n\r\n{}`,
        status_line,
        contents.len(),
        contents
    );
    
    stream.write_all(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

파일 이름: src/lib.rs

```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}

type Job = Box<dyn FnOnce() + Send + `static>;

impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
    
        let receiver = Arc::new(Mutex::new(receiver));
    
        let mut workers = Vec::with_capacity(size);
    
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
    
        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }
    
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + `static,
    {
        let job = Box::new(f);
    
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!(`Shutting down worker {}`, worker.id);
    
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!(`Worker {id} got a job; executing.`);
    
                    job();
                }
                Err(_) => {
                    println!(`Worker {id} disconnected; shutting down.`);
                    break;
                }
            }
        });
    
        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

우리는 여기서 더 많은 것을 할 수 있습니다! 이 프로젝트를 계속 개선하려면 다음과 같은 몇 가지 아이디어가 있습니다.

- `ThreadPool` 및 해당 공용 메서드에 더 많은 문서를 추가합니다.
- 라이브러리 기능 테스트를 추가합니다.
- 보다 강력한 오류 처리를 위해 호출을 `unwrap`으로 변경합니다.
- `ThreadPool`을 사용하여 웹 요청 제공 이외의 일부 작업을 수행합니다.
- [crates.io](https://crates.io/) 에서 스레드 풀 크레이트를 찾고 대신 크레이트를 사용하여 유사한 웹 서버를 구현합니다. 그런 다음 API 및 견고성을 우리가 구현한 스레드 풀과 비교합니다.

## [요약](https://doc.rust-lang.org/book/ch20-03-graceful-shutdown-and-cleanup.html#summary)

잘하셨어요! 책을 끝까지 읽으셨습니다! 이 Rust 투어에 참여해 주셔서 감사합니다. 이제 자신만의 Rust 프로젝트를 구현하고 다른 사람들의 프로젝트를 도울 준비가 되었습니다. Rust 여정에서 직면하는 모든 문제를 기꺼이 도와줄 다른 Rustacean 커뮤니티가 있음을 명심하세요.

------

# [부록](https://doc.rust-lang.org/book/appendix-00.html#appendix)

다음 섹션에는 Rust 여행에 유용할 수 있는 참조 자료가 포함되어 있습니다.

------

## [부록 A: 키워드](https://doc.rust-lang.org/book/appendix-01-keywords.html#appendix-a-keywords)

다음 목록에는 Rust 언어에서 현재 또는 미래에 사용하도록 예약된 키워드가 포함되어 있습니다. 따라서 식별자로 사용할 수 없습니다(`원시 [식별자](https://doc.rust-lang.org/book/appendix-01-keywords.html#raw-identifiers) ` 섹션에서 논의할 원시 식별자 제외). 식별자는 함수, 변수, 매개변수, 구조체 필드, 모듈, 크레이트, 상수, 매크로, 정적 값, 속성, 유형, 특성 또는 수명의 이름입니다.

### [현재 사용중인 키워드](https://doc.rust-lang.org/book/appendix-01-keywords.html#keywords-currently-in-use)

다음은 기능이 설명된 현재 사용 중인 키워드 목록입니다.

- `as` - 기본 캐스팅을 수행하거나, 항목을 포함하는 특정 특성을 명확하게 하거나, `use` 문에서 항목의 이름을 바꿉니다.
- `async` - 현재 스레드를 차단하는 대신 `Future` 반환
- `await` - `Future`의 결과가 준비될 때까지 실행을 일시 중지합니다.
- `break` - 루프를 즉시 종료합니다.
- `const` - 상수 항목 또는 상수 원시 포인터를 정의합니다.
- `계속` - 다음 루프 반복을 계속합니다.
- `크레이트` - 모듈 경로에서 크레이트 루트를 나타냅니다.
- `dyn` - 특성 개체에 대한 동적 디스패치
- `else` - `if` 및 `if let` 제어 흐름 구성에 대한 폴백
- `enum` - 열거형 정의
- `extern` - 외부 함수 또는 변수 연결
- `false` - 부울 거짓 리터럴
- `fn` - 함수 또는 함수 포인터 유형 정의
- `for` - 반복자의 항목을 반복하거나 특성을 구현하거나 더 높은 순위의 수명을 지정합니다.
- `if` - 조건식의 결과에 따라 분기
- `impl` - 고유 또는 특성 기능 구현
- `in` - `for` 루프 구문의 일부
- `let` - 변수 바인딩
- `loop` - 무조건 반복
- `match` - 값을 패턴에 일치
- `mod` - 모듈 정의
- `이동` - 클로저가 모든 캡처의 소유권을 갖도록 합니다.
- `mut` - 참조, 원시 포인터 또는 패턴 바인딩의 가변성을 나타냅니다.
- `pub` - 구조체 필드, `impl` 블록 또는 모듈의 공개 가시성을 나타냅니다.
- `ref` - 참조로 바인드
- `return` - 함수에서 반환
- `Self` - 정의하거나 구현하는 유형의 유형 별칭
- `self` - 메소드 주제 또는 현재 모듈
- `정적` - 전체 프로그램 실행을 지속하는 전역 변수 또는 수명
- `struct` - 구조 정의
- `super` - 현재 모듈의 상위 모듈
- `trait` - 특성 정의
- `true` - 부울 true 리터럴
- `type` - 유형 별칭 또는 관련 유형을 정의합니다.
- `공동체` - [공용체를](https://doc.rust-lang.org/reference/items/unions.html) 정의합니다. 유니온 선언에서 사용되는 경우에만 키워드입니다.
- `unsafe` - 안전하지 않은 코드, 기능, 특성 또는 구현을 나타냅니다.
- `use` - 기호를 범위로 가져오기
- `where` - 유형을 제한하는 절을 나타냅니다.
- `while` - 식의 결과에 따라 조건부 루프

### [향후 사용을 위해 예약된 키워드](https://doc.rust-lang.org/book/appendix-01-keywords.html#keywords-reserved-for-future-use)

다음 키워드는 아직 기능이 없지만 잠재적인 향후 사용을 위해 Rust에 예약되어 있습니다.

- `추상적인`
- `이 되다`
- `상자`
- `하다`
- `결정적인`
- `매크로`
- `우세하다`
- `개인`
- `노력하다`
- `유형`
- `크기 미정`
- `가상`
- `생산하다`

### [원시 식별자](https://doc.rust-lang.org/book/appendix-01-keywords.html#raw-identifiers)

*원시 식별자는* 일반적으로 허용되지 않는 키워드를 사용할 수 있게 해주는 구문입니다. 키워드 앞에 `r#`을 붙여 원시 식별자를 사용합니다.

예를 들어 `일치`는 키워드입니다. 이름으로 `match`를 사용하는 다음 함수를 컴파일하려고 하면:

파일 이름: src/main.rs

```rust
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

이 오류가 발생합니다.

```
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

오류는 함수 식별자로 키워드 `일치`를 사용할 수 없음을 보여줍니다. 함수 이름으로 `match`를 사용하려면 다음과 같이 원시 식별자 구문을 사용해야 합니다.

파일 이름: src/main.rs

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match(`foo`, `foobar`));
}
```

이 코드는 오류 없이 컴파일됩니다. 정의에서 함수 이름의 `r#` 접두어와 `main`에서 함수가 호출되는 위치에 유의하십시오.

원시 식별자를 사용하면 해당 단어가 예약 키워드인 경우에도 식별자로 선택한 단어를 사용할 수 있습니다. 이렇게 하면 식별자 이름을 더 자유롭게 선택할 수 있을 뿐만 아니라 이러한 단어가 키워드가 아닌 언어로 작성된 프로그램과 통합할 수 있습니다. 또한 원시 식별자를 사용하면 크레이트에서 사용하는 것과 다른 Rust 에디션으로 작성된 라이브러리를 사용할 수 있습니다. 예를 들어 `try`는 2015년 버전에서는 키워드가 아니지만 2018년 버전에서는 키워드입니다. 2015 에디션을 사용하여 작성되고 `try` 함수가 있는 라이브러리에 의존하는 경우 원시 식별자 구문(이 경우 `r#try`)을 사용하여 2018 에디션 코드에서 해당 함수를 호출해야 합니다. [에디션에 대한 자세한 내용은 부록 E를](https://doc.rust-lang.org/book/appendix-05-editions.html) 참조하십시오 .

------

## [부록 B: 연산자 및 기호](https://doc.rust-lang.org/book/appendix-02-operators.html#appendix-b-operators-and-symbols)

이 부록에는 자체적으로 또는 경로, 제네릭, 특성 범위, 매크로, 속성, 주석, 튜플 및 괄호의 맥락에서 나타나는 연산자 및 기타 기호를 포함하여 Rust의 구문에 대한 용어집이 포함되어 있습니다.

### [연산자](https://doc.rust-lang.org/book/appendix-02-operators.html#operators)

표 B-1에는 Rust의 연산자, 컨텍스트에서 연산자가 어떻게 표시되는지에 대한 예, 간단한 설명, 해당 연산자가 오버로드 가능한지 여부가 포함되어 있습니다. 연산자가 오버로드 가능한 경우 해당 연산자를 오버로드하는 데 사용할 관련 특성이 나열됩니다.

표 B-1: 연산자

| 운영자 | 예                                             | 설명                                                         | 과부하 가능?     |
| ------ | ---------------------------------------------- | ------------------------------------------------------------ | ---------------- |
| `!`    | `식별!(...)`, `식별!{...}`, `식별![...]`       | 매크로 확장                                                  |                  |
| `!`    | `!expr`                                        | 비트 또는 논리 보수                                          | `아니다`         |
| `!=`   | `expr != expr`                                 | 비동일성 비교                                                | `부분 방정식`    |
| `%`    | `expr % expr`                                  | 산술 나머지                                                  | `렘`             |
| `%=`   | `변수 %= 경험치`                               | 산술 나머지 및 할당                                          | `재할당`         |
| `&`    | `&expr`, `&mut expr`                           | 빌리다                                                       |                  |
| `&`    | `&type`, `&mut 유형`, `&`a 유형`, `&`mut 유형` | 빌린 포인터 유형                                             |                  |
| `&`    | `expr & expr`                                  | 비트 AND                                                     | `비트앤드`       |
| `&=`   | `var &= expr`                                  | 비트 AND 및 대입                                             | `BitAndAssign`   |
| `&&`   | `expr && expr`                                 | 단락 논리 AND                                                |                  |
| `*`    | `expr * expr`                                  | 산술 곱셈                                                    | `물`             |
| `*=`   | `var *= expr`                                  | 산술 곱셈 및 대입                                            | `물할당`         |
| `*`    | `*expr`                                        | 역참조                                                       | `데레프`         |
| `*`    | `*const 유형`, `*mut 유형`                     | 원시 포인터                                                  |                  |
| `+`    | `특성 + 특성`, ``a + 특성`                     | 복합 유형 제약                                               |                  |
| `+`    | `expr + expr`                                  | 산술 덧셈                                                    | `추가하다`       |
| `+=`   | `var += expr`                                  | 산술 덧셈과 대입                                             | `할당 추가`      |
| `,`    | `익스프레스`                                   | 인수 및 요소 구분 기호                                       |                  |
| `-`    | `- 특급`                                       | 산술 부정                                                    | `네그`           |
| `-`    | `expr-expr`                                    | 산술 뺄셈                                                    | `보결`           |
| `-=`   | `var -= expr`                                  | 산술 뺄셈 및 대입                                            | `하위 할당`      |
| `->`   | `fn(...) -> 유형`, `                           | ...                                                          | -> 입력`         |
| `.`    | `expr.ident`                                   | 회원 액세스                                                  |                  |
| `..`   | `..`, `expr..`, `..expr`, `expr..expr`         | 오른쪽 제외 범위 리터럴                                      | `부분 순서`      |
| `..=`  | `..=expr`, `expr..=expr`                       | 오른쪽 포함 범위 리터럴                                      | `부분 순서`      |
| `..`   | `..expr`                                       | 구조체 리터럴 업데이트 구문                                  |                  |
| `..`   | `variant(x, ..)`, `struct_type { x, .. }`      | `그리고 나머지` 패턴 제본                                    |                  |
| `...`  | `expr...expr`                                  | (더 이상 사용되지 않음, 대신 `..=` 사용) 패턴에서: 포함 범위 패턴 |                  |
| `/`    | `익스프레스/익스프레스`                        | 산술 나눗셈                                                  | `사업부`         |
| `/=`   | `var /= expr`                                  | 산술 나눗셈과 대입                                           | `사업부 할당`    |
| `:`    | `pat: 유형`, `ident: 유형`                     | 제약                                                         |                  |
| `:`    | `식별: expr`                                   | 구조체 필드 이니셜라이저                                     |                  |
| `:`    | ``a: 루프 {...}`                               | 루프 라벨                                                    |                  |
| `;`    | `익스프레스;`                                  | 문 및 항목 종결자                                            |                  |
| `;`    | `[...; 렌]`                                    | 고정 크기 배열 구문의 일부                                   |                  |
| `<<`   | `expr << 익스퍼`                               | 왼쪽 시프트                                                  | `쉬`             |
| `<<=`  | `var <<= expr`                                 | 왼쪽 시프트 및 할당                                          | `Shl할당`        |
| `<`    | `익스프레스 < 엑스프레스`                      | 비교보다 작음                                                | `부분 순서`      |
| `<=`   | `expr <= expr`                                 | 작거나 같음 비교                                             | `부분 순서`      |
| `=`    | `var = expr`, `ident = 유형`                   | 할당/등가                                                    |                  |
| `==`   | `expr == expr`                                 | 평등 비교                                                    | `부분 방정식`    |
| `=>`   | `팻 => expr`                                   | 매치 암 구문의 일부                                          |                  |
| `>`    | `익스프레스 > 엑스프레스`                      | 비교보다 큼                                                  | `부분 순서`      |
| `>=`   | `expr >= expr`                                 | 비교보다 크거나 같음                                         | `부분 순서`      |
| `>>`   | `expr >> expr`                                 | 오른쪽 시프트                                                | `슈`             |
| `>>=`  | `변수 >>= 경험치`                              | 오른쪽 시프트 및 할당                                        | `ShrAssign`      |
| `@`    | `식별 @ 팻`                                    | 패턴 바인딩                                                  |                  |
| `^`    | `expr ^ expr`                                  | 비트 배타적 OR                                               | `BitXor`         |
| `^=`   | `var ^= expr`                                  | 비트 배타적 OR 및 대입                                       | `BitXorAssign`   |
| `      | `                                              | `가볍게 두드리기                                             | 가볍게 두드리기` |
| `      | `                                              | `expr                                                        | 특급`            |
| `      | =`                                             | `var                                                         | = 경험치`        |
| `      |                                                | `                                                            | `expr            |
| `?`    | `익스프레스?`                                  | 오류 전파                                                    |                  |

### [비연산자 기호](https://doc.rust-lang.org/book/appendix-02-operators.html#non-operator-symbols)

다음 목록에는 연산자로 작동하지 않는 모든 기호가 포함되어 있습니다. 즉, 함수 또는 메서드 호출처럼 동작하지 않습니다.

표 B-2는 자체적으로 나타나고 다양한 위치에서 유효한 기호를 보여줍니다.

표 B-2: 독립형 구문

| 상징                                       | 설명                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| ``정체성`                                  | 명명된 수명 또는 루프 레이블                                 |
| `...u8`, `...i32`, `...f64`, `...usize` 등 | 특정 유형의 숫자 리터럴                                      |
| ``...``                                    | 문자열 리터럴                                                |
| `r`...``, `r#`...`#`, `r##`...`##`, 등.    | 원시 문자열 리터럴, 이스케이프 문자가 처리되지 않음          |
| `비`...``                                  | 바이트 문자열 리터럴 문자열 대신 바이트 배열을 구성합니다.   |
| `br`...``, `br#`...`#`, `br##`...`##` 등   | 원시 바이트 문자열 리터럴, 원시 및 바이트 문자열 리터럴의 조합 |
| ``...``                                    | 문자 리터럴                                                  |
| `비`...``                                  | ASCII 바이트 리터럴                                          |
| `                                          | ...                                                          |
| `!`                                        | 발산 기능을 위한 항상 빈 바닥 유형                           |
| `_`                                        | `무시된` 패턴 바인딩; 또한 정수 리터럴을 읽을 수 있도록 만드는 데 사용됩니다. |

표 B-3은 항목에 대한 모듈 계층 구조를 통한 경로 컨텍스트에 나타나는 기호를 보여줍니다.

표 B-3: 경로 관련 구문

| 상징                           | 설명                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `식별::식별`                   | 네임스페이스 경로                                            |
| `::길`                         | 크레이트 루트에 상대적인 경로(즉, 명시적으로 절대 경로)      |
| `자신::경로`                   | 현재 모듈에 상대적인 경로(즉, 명시적 상대 경로).             |
| `슈퍼::경로`                   | 현재 모듈의 부모에 상대적인 경로                             |
| `유형::식별자`, `::아이덴티티` | 관련 상수, 함수 및 유형                                      |
| `::...`                        | 직접 이름을 지정할 수 없는 유형의 관련 항목(예: `<&T>::...`, `<[T]>::...` 등) |
| `특성::방법(...)`              | 메서드 호출을 정의하는 트레이트의 이름을 지정하여 메서드 호출 명확화 |
| `유형::방법(...)`              | 정의된 유형의 이름을 지정하여 메서드 호출 명확화             |
| `::방법(...)`                  | 특성과 유형의 이름을 지정하여 메서드 호출 명확화             |

표 B-4는 제네릭 형식 매개변수를 사용할 때 나타나는 기호를 보여줍니다.

표 B-4: 제네릭

| 상징                         | 설명                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `경로<...>`                  | 유형의 제네릭 유형에 대한 매개변수를 지정합니다(예: `Vec`)   |
| `경로::<...>`, `방법::<...>` | 식의 제네릭 형식, 함수 또는 메서드에 대한 매개 변수를 지정합니다. 종종 터보피시라고 합니다(예: ``42`.parse::()`) |
| `fn ident<...> ...`          | 제네릭 함수 정의                                             |
| `구조체 식별<...> ...`       | 일반 구조 정의                                               |
| `열거 ID<...> ...`           | 일반 열거 정의                                               |
| `impl<...> ...`              | 일반 구현 정의                                               |
| `for<...> 유형`              | 더 높은 순위의 수명 범위                                     |
| `유형<식별=유형>`            | 하나 이상의 관련 유형에 특정 할당이 있는 일반 유형(예: `Iterator<Item=T>`) |

표 B-5는 특성 범위를 사용하여 일반 유형 매개변수를 제한하는 컨텍스트에서 나타나는 기호를 보여줍니다.

표 B-5: 특성 바운드 제약

| 상징                       | 설명                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `티: 유`                   | `U`를 구현하는 유형으로 제한되는 일반 매개변수 `T`           |
| `티: `아`                  | 일반 유형 `T`는 수명 ``a`보다 오래 지속되어야 합니다(유형은 수명이 ``a`보다 짧은 참조를 전이적으로 포함할 수 없음을 의미). |
| `T: `정적`                 | 제네릭 유형 `T`에는 ``정적` 참조 이외의 빌린 참조가 없습니다. |
| ``b: `아`                  | 일반 수명 ``b`는 수명 ``a`보다 오래 지속되어야 합니다.       |
| `T: ?크기`                 | 일반 유형 매개변수가 동적으로 크기가 조정되는 유형이 되도록 허용 |
| ``a + 특성`, `특성 + 특성` | 복합 유형 제약                                               |

표 B-6은 매크로를 호출하거나 정의하고 항목에 대한 속성을 지정하는 상황에서 나타나는 기호를 보여줍니다.

표 B-6: 매크로 및 속성

| 상징                                     | 설명        |
| ---------------------------------------- | ----------- |
| `#[메타]`                                | 외부 속성   |
| `#![메타]`                               | 내부 속성   |
| `$ident`                                 | 매크로 대체 |
| `$ident:종류`                            | 매크로 캡처 |
| `$(…)…`                                  | 매크로 반복 |
| `식별!(...)`, `식별!{...}`, `식별![...]` | 매크로 호출 |

표 B-7은 주석을 만드는 기호를 보여줍니다.

표 B-7: 설명

| 상징         | 설명                |
| ------------ | ------------------- |
| `//`         | 라인 코멘트         |
| `//!`        | 내부 라인 문서 주석 |
| `///`        | 외곽선 문서 주석    |
| `/ *...* /`  | 댓글 차단           |
| `/ *!...* /` | 내부 블록 문서 주석 |
| `/**...*/`   | 외부 블록 문서 주석 |

표 B-8은 튜플 사용과 관련하여 나타나는 기호를 보여줍니다.

표 B-8: 튜플

| 상징                  | 설명                                                         |
| --------------------- | ------------------------------------------------------------ |
| `()`                  | 빈 튜플(일명 단위), 리터럴 및 유형 모두                      |
| `(익스프레스)`        | 괄호로 묶은 표현                                             |
| `(익스프레스,)`       | 단일 요소 튜플 표현식                                        |
| `(유형,)`             | 단일 요소 튜플 유형                                          |
| `(익스프레스, ...)`   | 튜플 표현식                                                  |
| `(유형, ...)`         | 튜플 유형                                                    |
| `expr(expr, ...)`     | 함수 호출 표현; 또한 튜플 `struct` 및 튜플 `enum` 변형을 초기화하는 데 사용됩니다. |
| `expr.0`, `expr.1` 등 | 튜플 인덱싱                                                  |

표 B-9는 중괄호가 사용되는 문맥을 보여줍니다.

표 B-9: 중괄호

| 문맥         | 설명            |
| ------------ | --------------- |
| `{...}`      | 블록 표현       |
| `유형 {...}` | `구조체` 리터럴 |

표 B-10은 대괄호가 사용되는 문맥을 보여줍니다.

표 B-10: 대괄호

| 문맥                                               | 설명                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `[...]`                                            | 배열 리터럴                                                  |
| `[expr; 렌]`                                       | `expr`의 `len` 복사본을 포함하는 배열 리터럴                 |
| `[유형; 렌]`                                       | `type`의 `len` 인스턴스를 포함하는 배열 유형                 |
| `expr[익스프레]`                                   | 컬렉션 인덱싱. 오버로드 가능(`Index`, `IndexMut`)            |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | `Range`, `RangeFrom`, `RangeTo` 또는 `RangeFull`을 `인덱스`로 사용하여 컬렉션 슬라이싱인 척하는 컬렉션 인덱싱 |

------

## [부록 C: 파생 특성](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#appendix-c-derivable-traits)

이 책의 여러 곳에서 구조체 또는 열거형 정의에 적용할 수 있는 `derive` 속성에 대해 논의했습니다. `derive` 특성은 `derive` 구문으로 주석을 추가한 유형에 자체 기본 구현을 사용하여 특성을 구현하는 코드를 생성합니다.

이 부록에서는 `파생`과 함께 사용할 수 있는 표준 라이브러리의 모든 특성에 대한 참조를 제공합니다. 각 섹션에서는 다음을 다룹니다.

- 이 트레이트를 유도하는 연산자와 메서드는 무엇을 가능하게 합니까?
- `파생`에서 제공하는 특성의 구현이 수행하는 작업
- 특성을 구현하는 것이 유형에 대해 의미하는 것
- 특성을 구현하도록 허용되거나 허용되지 않는 조건
- 특성이 필요한 작업의 예

`derive` 속성에서 제공하는 것과 다른 동작을 원하는 경우 수동으로 구현하는 방법에 대한 자세한 내용은 각 특성에 대한 [표준 라이브러리 문서를 참조하세요.](https://doc.rust-lang.org/std/index.html)

여기에 나열된 이러한 특성은 `derive`를 사용하여 유형에 구현할 수 있는 표준 라이브러리에 의해 정의된 유일한 특성입니다. 표준 라이브러리에 정의된 다른 특성에는 합리적인 기본 동작이 없으므로 달성하려는 작업에 적합한 방식으로 구현하는 것은 사용자에게 달려 있습니다.

파생할 수 없는 트레이트의 예로는 최종 사용자의 서식을 처리하는 `디스플레이`가 있습니다. 최종 사용자에게 유형을 표시하는 적절한 방법을 항상 고려해야 합니다. 최종 사용자가 볼 수 있도록 유형의 어떤 부분을 허용해야 합니까? 어떤 부분이 관련성이 있다고 생각합니까? 그들에게 가장 관련 있는 데이터 형식은 무엇입니까? Rust 컴파일러에는 이러한 통찰력이 없으므로 적절한 기본 동작을 제공할 수 없습니다.

이 부록에서 제공되는 파생 가능한 특성 목록은 포괄적이지 않습니다. 라이브러리는 고유한 특성에 대해 `파생`을 구현할 수 있으므로 진정한 개방형으로 `파생`할 수 있는 특성 목록을 만들 수 있습니다. [`파생` 구현에는 19장의 `매크로`](https://doc.rust-lang.org/book/ch19-06-macros.html#macros) 섹션 에서 다루는 절차적 매크로 사용이 포함됩니다.

### [프로그래머 출력을 위한 `디버그`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#debug-for-programmer-output)

`디버그` 특성은 `:?`를 추가하여 표시하는 형식 문자열에서 디버그 형식화를 활성화합니다. `{}` 자리 표시자 내.

`디버그` 특성을 사용하면 디버깅 목적으로 유형의 인스턴스를 인쇄할 수 있으므로 귀하와 귀하의 유형을 사용하는 다른 프로그래머가 프로그램 실행의 특정 지점에서 인스턴스를 검사할 수 있습니다.

예를 들어 `assert_eq!`를 사용하려면 `디버그` 특성이 필요합니다. 매크로. 이 매크로는 같음 주장이 실패하면 인수로 제공된 인스턴스 값을 인쇄하여 프로그래머가 두 인스턴스가 같지 않은 이유를 확인할 수 있습니다.

### [등식 비교를 위한 `PartialEq` 및 `Eq`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#partialeq-and-eq-for-equality-comparisons)

`PartialEq` 특성을 사용하면 유형의 인스턴스를 비교하여 동등성을 확인하고 `==` 및 `!=` 연산자를 사용할 수 있습니다.

`PartialEq` 파생은 `eq` 방법을 구현합니다. *구조체에서 `PartialEq`가 파생된 경우 모든* 필드가 동일한 경우에만 두 인스턴스가 같고 필드가 같지 않은 경우 인스턴스가 같지 않습니다. 열거형에서 파생된 경우 각 변형은 자체와 동일하며 다른 변형과 동일하지 않습니다.

예를 들어 `assert_eq!`를 사용하려면 `PartialEq` 특성이 필요합니다. 같은 유형의 두 인스턴스를 비교할 수 있어야 하는 매크로.

`Eq` 특성에는 방법이 없습니다. 그 목적은 주석이 달린 유형의 모든 값에 대해 값이 자신과 같다는 신호를 보내는 것입니다. `Eq` 특성은 `PartialEq`도 구현하는 유형에만 적용할 수 있지만 `PartialEq`를 구현하는 모든 유형이 `Eq`를 구현할 수 있는 것은 아닙니다. 이에 대한 한 가지 예는 부동 소수점 숫자 유형입니다. 부동 소수점 숫자의 구현은 숫자가 아닌(`NaN`) 값의 두 인스턴스가 서로 같지 않음을 나타냅니다.

`Eq`가 필요한 경우의 예는 `HashMap<K, V>`의 키에 대한 것이므로 `HashMap<K, V>`는 두 키가 동일한지 여부를 알 수 있습니다.

### [순서 비교를 위한 `PartialOrd` 및 `Ord`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#partialord-and-ord-for-ordering-comparisons)

`PartialOrd` 특성을 사용하면 정렬 목적으로 유형의 인스턴스를 비교할 수 있습니다. `PartialOrd`를 구현하는 형식은 `<`, `>`, `<=` 및 `>=` 연산자와 함께 사용할 수 있습니다. `PartialEq`도 구현하는 유형에만 `PartialOrd` 특성을 적용할 수 있습니다.

`PartialOrd` 파생은 `partial_cmp` 메서드를 구현하여 `옵션`을 반환합니다.` 해당 유형의 대부분의 값을 비교할 수 있지만 순서를 생성하지 않는 값의 예는 숫자가 아닌( `NaN`) 부동 소수점 값 임의의 부동 소수점 숫자 및 `NaN` 부동 소수점 값으로 `partial_cmp`를 호출하면 `None`이 반환됩니다.

구조체에서 파생된 경우 `PartialOrd`는 필드가 구조체 정의에 나타나는 순서대로 각 필드의 값을 비교하여 두 인스턴스를 비교합니다. 열거형에서 파생된 경우 열거형 정의에서 앞서 선언된 열거형의 변형은 나중에 나열된 변형보다 작은 것으로 간주됩니다.

`PartialOrd` 특성은 예를 들어 범위 표현식으로 지정된 범위에서 무작위 값을 생성하는 `rand` 크레이트의 `gen_range` 메서드에 필요합니다.

`Ord` 특성을 사용하면 주석이 달린 유형의 두 값에 대해 유효한 순서가 존재함을 알 수 있습니다. `Ord` 특성은 `Option`이 아닌 `Ordering`을 반환하는 `cmp` 메서드를 구현합니다.` 유효한 순서 지정이 항상 가능하기 때문입니다. `Ord` 특성은 `PartialOrd` 및 `Eq`(및 `Eq`에는 `PartialEq`가 필요함)도 구현하는 유형에만 적용할 수 있습니다. 구조체 및 열거형에서 파생될 때 ` cmp`는 `partial_cmp`에 대한 파생 구현이 `PartialOrd`와 동일한 방식으로 동작합니다.

`Ord`가 필요한 경우의 예는 `BTreeSet`에 값을 저장할 때입니다.`, 값의 정렬 순서에 따라 데이터를 저장하는 데이터 구조입니다.

### [값 복제를 위한 `복제` 및 `복사`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#clone-and-copy-for-duplicating-values)

`복제` 특성을 사용하면 값의 깊은 복사본을 명시적으로 만들 수 있으며 복제 프로세스에는 임의 코드 실행 및 힙 데이터 복사가 포함될 수 있습니다. `복제 `에 대한 자세한 내용은 4장의 [`변수와 데이터가 상호 작용하는 방식: 복제`](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ways-variables-and-data-interact-clone) 섹션을 참조하십시오 .

`Clone` 파생은 전체 유형에 대해 구현될 때 유형의 각 부분에서 `clone`을 호출하는 `clone` 메소드를 구현합니다. 이는 유형의 모든 필드 또는 값이 `복제`를 파생시키기 위해 `복제`도 구현해야 함을 의미합니다.

`복제`가 필요한 경우의 예는 슬라이스에서 `to_vec` 메서드를 호출할 때입니다. 슬라이스는 포함된 유형 인스턴스를 소유하지 않지만 `to_vec`에서 반환된 벡터는 해당 인스턴스를 소유해야 하므로 `to_vec`는 각 항목에서 `clone`을 호출합니다. 따라서 슬라이스에 저장된 유형은 `복제`를 구현해야 합니다.

`복사` 특성을 사용하면 스택에 저장된 비트만 복사하여 값을 복제할 수 있습니다. 임의의 코드가 필요하지 않습니다. ` 복사`에 대한 자세한 내용은 4장의 [`스택 전용 데이터: 복사`](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy) 섹션을 참조하십시오 .

`복사` 특성은 프로그래머가 해당 메서드를 오버로드하고 임의의 코드가 실행되지 않는다는 가정을 위반하는 것을 방지하는 메서드를 정의하지 않습니다. 그런 식으로 모든 프로그래머는 값 복사가 매우 빠를 것이라고 가정할 수 있습니다.

부분이 모두 `복사`를 구현하는 모든 유형에서 `복사`를 파생시킬 수 있습니다. `복사`를 구현하는 유형은 `복사`도 구현해야 합니다. `복사`를 구현하는 유형에는 `복사`와 동일한 작업을 수행하는 간단한 `복제` 구현이 있기 때문입니다.

`복사` 특성은 거의 필요하지 않습니다. `Copy`를 구현하는 유형에는 최적화가 가능합니다. 즉, `clone`을 호출할 필요가 없으므로 코드가 더 간결해집니다.

`복사`로 가능한 모든 작업은 `복제`로도 수행할 수 있지만 코드가 느려지거나 위치에서 `복제`를 사용해야 할 수 있습니다.

### [값을 고정 크기 값에 매핑하기 위한 `해시`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#hash-for-mapping-a-value-to-a-value-of-fixed-size)

`해시` 특성을 사용하면 임의 크기 유형의 인스턴스를 가져오고 해시 함수를 사용하여 해당 인스턴스를 고정 크기 값에 매핑할 수 있습니다. `해시` 파생은 `해시` 메서드를 구현합니다. `해시` 메서드의 파생된 구현은 유형의 각 부분에서 `해시`를 호출한 결과를 결합합니다. 즉, 모든 필드 또는 값도 `해시`를 파생하려면 `해시`를 구현해야 합니다.

`Hash`가 필요한 경우의 예는 데이터를 효율적으로 저장하기 위해 `HashMap<K, V>`에 키를 저장하는 것입니다.

### [기본값에 대한 `기본값`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#default-for-default-values)

`Default` 특성을 사용하면 유형의 기본값을 만들 수 있습니다. `Default` 파생은 `default` 기능을 구현합니다. `default` 함수의 파생된 구현은 유형의 각 부분에서 `default` 함수를 호출합니다. 즉, 유형의 모든 필드 또는 값은 `Default`를 파생시키기 위해 `Default`도 구현해야 합니다.

[`Default::default` 함수는 일반적으로 5장의 `구조체 업데이트 구문을 사용하여 다른 인스턴스에서 인스턴스 만들기`](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax) 섹션 에서 설명한 구조체 업데이트 구문과 함께 사용됩니다. 구조체의 몇 가지 필드를 사용자 정의한 다음 설정할 수 있습니다. `..Default::default()`를 사용하여 나머지 필드에 기본값을 사용하십시오.

`기본` 특성은 `옵션`에서 `unwrap_or_default` 메서드를 사용할 때 필요합니다.` 인스턴스를 예로 들 수 있습니다. `옵션이`가 `없음`인 경우 `unwrap_or_default` 메서드는 `옵션`에 저장된 `T` 유형에 대해 `Default::default` 결과를 반환합니다.`.

------

## [부록 D - 유용한 개발 도구](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#appendix-d---useful-development-tools)

이 부록에서는 Rust 프로젝트가 제공하는 몇 가지 유용한 개발 도구에 대해 이야기합니다. 자동 서식 지정, 경고 수정을 적용하는 빠른 방법, linter 및 IDE와의 통합에 대해 살펴보겠습니다.

### [`rustfmt`를 사용한 자동 서식 지정](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#automatic-formatting-with-rustfmt)

`rustfmt` 도구는 커뮤니티 코드 스타일에 따라 코드 형식을 다시 지정합니다. 많은 협업 프로젝트는 Rust를 작성할 때 사용할 스타일에 대한 논쟁을 방지하기 위해 `rustfmt`를 사용합니다. 모든 사람이 도구를 사용하여 코드 형식을 지정합니다.

`rustfmt`를 설치하려면 다음을 입력하십시오.

```bash
$ rustup component add rustfmt
```

이 명령은 Rust가 `rustc`와 `cargo`를 모두 제공하는 것과 유사하게 `rustfmt`와 `cargo-fmt`를 제공합니다. Cargo 프로젝트를 포맷하려면 다음을 입력하십시오:

```bash
$ cargo fmt
```

이 명령을 실행하면 현재 크레이트의 모든 Rust 코드가 다시 포맷됩니다. 이것은 코드 의미 체계가 아니라 코드 스타일만 변경해야 합니다. `rustfmt`에 대한 자세한 내용은 [해당 설명서를](https://github.com/rust-lang/rustfmt) 참조하십시오 .

### [`rustfix`로 코드 수정](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#fix-your-code-with-rustfix)

Rustfix 도구는 Rust 설치에 포함되어 있으며 원하는 문제일 가능성이 있는 문제를 수정하는 명확한 방법이 있는 컴파일러 경고를 자동으로 수정할 수 있습니다. 이전에 컴파일러 경고를 본 적이 있을 것입니다. 예를 들어 다음 코드를 고려하십시오.

파일 이름: src/main.rs

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

여기에서 우리는 `do_something` 함수를 100번 호출하지만 `for` 루프 본문에 변수 `i`를 사용하지 않습니다. Rust는 이에 대해 다음과 같이 경고합니다.

```bash
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: `i`
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using `_i` instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

경고는 이름으로 `_i`를 대신 사용할 것을 제안합니다. 밑줄은 이 변수를 사용하지 않을 것임을 나타냅니다. `cargo fix` 명령을 실행하여 `rustfix` 도구를 사용하여 해당 제안을 자동으로 적용할 수 있습니다.

```bash
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

*src/main.rs를* 다시 보면 `cargo fix`가 코드를 변경했음을 알 수 있습니다.

파일 이름: src/main.rs

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

`for` 루프 변수의 이름은 이제 `_i`로 지정되고 더 이상 경고가 표시되지 않습니다.

`cargo fix` 명령을 사용하여 다른 Rust 버전 간에 코드를 전환할 수도 있습니다. 에디션은 부록 E에서 다룹니다.

### [Clippy로 더 많은 린트](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#more-lints-with-clippy)

Clippy 도구는 일반적인 실수를 포착하고 Rust 코드를 개선할 수 있도록 코드를 분석하기 위한 린트 모음입니다.

Clippy를 설치하려면 다음을 입력하십시오.

```bash
$ rustup component add clippy
```

Cargo 프로젝트에서 Clippy의 린트를 실행하려면 다음을 입력하십시오.

```bash
$ cargo clippy
```

예를 들어, 다음 프로그램처럼 파이와 같은 수학 상수의 근사치를 사용하는 프로그램을 작성한다고 가정해 보겠습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!(`the area of the circle is {}`, x * r * r);
}
```

이 프로젝트에서 `cargo clippy`를 실행하면 다음 오류가 발생합니다.

```
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

이 오류는 Rust에 이미 더 정확한 `PI` 상수가 정의되어 있고 대신 상수를 사용하면 프로그램이 더 정확하다는 것을 알려줍니다. 그런 다음 `PI` 상수를 사용하도록 코드를 변경합니다. 다음 코드는 Clippy에서 오류나 경고를 발생시키지 않습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!(`the area of the circle is {}`, x * r * r);
}
```

Clippy에 대한 자세한 내용은 [설명서를](https://github.com/rust-lang/rust-clippy) 참조하십시오 .

### [`rust-analyzer`를 사용한 IDE 통합](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html#ide-integration-using-rust-analyzer)

IDE 통합을 돕기 위해 Rust 커뮤니티는 [`rust-analyzer`](https://rust-analyzer.github.io/) 사용을 권장합니다. 이 도구 는 IDE와 프로그래밍 언어가 서로 통신하기 위한 사양인 [언어 서버 프로토콜을](http://langserver.org/) 말하는 컴파일러 중심 유틸리티 세트입니다. [다른 클라이언트는 Visual Studio Code용 Rust 분석기 플러그인](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer) 과 같은 `rust-analyzer`를 사용할 수 있습니다.

설치 지침을 보려면 `rust-analyzer` 프로젝트의 [홈 페이지를](https://rust-analyzer.github.io/) 방문 하고 특정 IDE에 언어 서버 지원을 설치하십시오. IDE는 자동 완성, 정의로 이동, 인라인 오류와 같은 기능을 얻게 됩니다.

------

## [부록 E - 에디션](https://doc.rust-lang.org/book/appendix-05-editions.html#appendix-e---editions)

1장에서 `cargo new`가 에디션에 대한 *Cargo.toml* 파일 에 약간의 메타데이터를 추가하는 것을 보았습니다. 이 부록은 그 의미에 대해 설명합니다!

Rust 언어와 컴파일러는 6주 릴리스 주기를 가지며, 이는 사용자가 새로운 기능을 지속적으로 얻을 수 있음을 의미합니다. 다른 프로그래밍 언어는 더 큰 변경 사항을 덜 자주 릴리스합니다. Rust는 더 작은 업데이트를 더 자주 릴리스합니다. 잠시 후 이러한 모든 작은 변화가 합산됩니다. 그러나 출시마다 `와, Rust 1.10과 Rust 1.31 사이에 Rust가 많이 바뀌었어요!`라고 되돌아보며 말하기는 어려울 수 있습니다.

Rust 팀은 2년 또는 3년마다 새로운 Rust *에디션을* 제작합니다. 각 에디션은 완전히 업데이트된 문서 및 도구와 함께 명확한 패키지에 포함된 기능을 함께 제공합니다. 새 에디션은 일반적인 6주 출시 프로세스의 일부로 배송됩니다.

에디션은 사람마다 다른 용도로 사용됩니다.

- 활성 Rust 사용자를 위해 새 에디션은 점진적인 변경 사항을 이해하기 쉬운 패키지로 통합합니다.
- 비사용자의 경우 새 에디션은 몇 가지 주요 발전이 이루어졌음을 알리므로 Rust를 다시 살펴볼 가치가 있습니다.
- Rust를 개발하는 사람들을 위해 새 에디션은 전체적으로 프로젝트의 집결 지점을 제공합니다.

이 글을 쓰는 시점에 Rust 2015, Rust 2018 및 Rust 2021의 세 가지 Rust 에디션을 사용할 수 있습니다. 이 책은 Rust 2021 에디션 관용구를 사용하여 작성되었습니다.

*Cargo.toml* 의 `edition` 키는 컴파일러가 코드에 사용해야 하는 에디션을 나타냅니다. 키가 존재하지 않으면 Rust는 이전 버전과의 호환성을 위해 에디션 값으로 `2015`를 사용합니다.

각 프로젝트는 기본 2015 에디션 이외의 에디션을 선택할 수 있습니다. 버전에는 코드의 식별자와 충돌하는 새 키워드를 포함하는 것과 같이 호환되지 않는 변경 사항이 포함될 수 있습니다. 그러나 이러한 변경 사항을 선택하지 않으면 사용하는 Rust 컴파일러 버전을 업그레이드하더라도 코드는 계속 컴파일됩니다.

모든 Rust 컴파일러 버전은 해당 컴파일러 릴리스 이전에 존재했던 모든 버전을 지원하며 지원되는 모든 버전의 크레이트를 함께 연결할 수 있습니다. 버전 변경은 컴파일러가 처음에 코드를 구문 분석하는 방식에만 영향을 미칩니다. 따라서 Rust 2015를 사용 중이고 종속성 중 하나가 Rust 2018을 사용하는 경우 프로젝트가 컴파일되고 해당 종속성을 사용할 수 있습니다. 프로젝트가 Rust 2018을 사용하고 종속성이 Rust 2015를 사용하는 반대 상황도 작동합니다.

명확하게 말하면 대부분의 기능은 모든 버전에서 사용할 수 있습니다. Rust 에디션을 사용하는 개발자는 새로운 안정적인 릴리스가 만들어짐에 따라 계속해서 개선 사항을 확인할 수 있습니다. 그러나 경우에 따라 주로 새 키워드가 추가될 때 일부 새 기능은 이후 버전에서만 사용할 수 있습니다. 이러한 기능을 이용하려면 버전을 전환해야 합니다.

자세한 내용은 [*에디션 가이드에서*](https://doc.rust-lang.org/stable/edition-guide/) 에디션 간의 차이점을 열거하고 `cargo fix`를 통해 코드를 새 에디션으로 자동 업그레이드하는 방법을 설명하는 에디션에 대한 완전한 책입니다.

------

## [부록 F: 책의 번역](https://doc.rust-lang.org/book/appendix-06-translation.html#appendix-f-translations-of-the-book)

영어 이외의 언어로 된 리소스의 경우. 대부분은 아직 진행 중입니다. [번역 레이블을](https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations) 참조하여 도움을 받거나 새 번역에 대해 알려주십시오!

- [포르투갈어](https://github.com/rust-br/rust-book-pt-br) (BR)
- [포르투갈어](https://github.com/nunojesus/rust-book-pt-pt) (PT)
- [简體中文](https://github.com/KaiserY/trpl-zh-cn)
- [正體中文](https://github.com/rust-tw/book-tw)
- [Українська](https://github.com/pavloslav/rust-book-uk-ua)
- [스페인어](https://github.com/thecodix/book) , [대체](https://github.com/ManRR/rust-book-es)
- [이탈리아노](https://github.com/EmanueleGurini/book_it)
- [러시아인](https://github.com/rust-lang-ru/book)
- [한국어](https://github.com/rinthel/rust-lang-book-ko)
- [일본어](https://github.com/rust-lang-ja/book-ja)
- [프랑세즈](https://github.com/Jimskapt/rust-book-fr)
- [폴스키](https://github.com/paytchoo/book-pl)
- [세부아노어](https://github.com/agentzero1/book)
- [타갈로그어](https://github.com/josephace135/book)
- [에스페란토 말](https://github.com/psychoslave/Rust-libro)
- [ελληνική](https://github.com/TChatzigiannakis/rust-book-greek)
- [스벤스카](https://github.com/sebras/book)
- [페르시아어](https://github.com/pomokhtari/rust-book-fa)
- [독일어](https://github.com/rust-lang-de/rustbook-de)
- [힌디어](https://github.com/venkatarun95/rust-book-hindi)
- [ไทย](https://github.com/rust-lang-th/book-th)
- [단스케](https://github.com/DanKHansen/book-dk)

------

## [부록 G - Rust가 만들어지는 방식과 “Nightly Rust”](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#appendix-g---how-rust-is-made-and-nightly-rust)

이 부록은 Rust가 어떻게 만들어지고 그것이 Rust 개발자로서 당신에게 어떤 영향을 미치는지에 관한 것입니다.

### [정체 없는 안정성](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#stability-without-stagnation)

언어로서 Rust는 코드의 안정성에 *많은 관심을 기울입니다.* 우리는 Rust가 여러분이 구축할 수 있는 견고한 기반이 되기를 원합니다. 상황이 끊임없이 변화한다면 그것은 불가능할 것입니다. 동시에 새로운 기능을 실험할 수 없다면 더 이상 변경할 수 없는 출시 이후까지 중요한 결함을 발견하지 못할 수 있습니다.

이 문제에 대한 우리의 해결책은 우리가 `정체 없는 안정성`이라고 부르는 것이며, 우리의 기본 원칙은 다음과 같습니다: 안정적인 Rust의 새 버전으로 업그레이드하는 것을 두려워할 필요가 없습니다. 각 업그레이드는 수월해야 하지만 새로운 기능, 더 적은 버그, 더 빠른 컴파일 시간을 제공해야 합니다.

### [추추! 릴리스 채널 및 기차 타기](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#choo-choo-release-channels-and-riding-the-trains)

Rust 개발은 *기차 일정* 에 따라 운영됩니다. 즉, 모든 개발은 Rust 저장소의 `마스터` 분기에서 수행됩니다. 릴리스는 Cisco IOS 및 기타 소프트웨어 프로젝트에서 사용된 소프트웨어 릴리스 트레인 모델을 따릅니다. Rust에는 세 가지 *릴리스 채널이* 있습니다.

- 나이틀리
- 베타
- 안정적인

대부분의 Rust 개발자는 주로 안정적인 채널을 사용하지만 실험적인 새 기능을 시도하려는 사용자는 nightly 또는 베타를 사용할 수 있습니다.

다음은 개발 및 출시 프로세스가 어떻게 작동하는지에 대한 예입니다. Rust 팀이 Rust 1.5 출시 작업을 하고 있다고 가정해 봅시다. 해당 릴리스는 2015년 12월에 이루어졌지만 현실적인 버전 번호를 제공할 것입니다. Rust에 새로운 기능이 추가되었습니다: 새로운 커밋이 `마스터` 브랜치에 도달합니다. 매일 밤 Rust의 새로운 야간 버전이 생성됩니다. 매일이 출시일이며 이러한 출시는 출시 인프라에서 자동으로 생성됩니다. 그래서 시간이 지남에 따라 우리의 릴리스는 밤에 한 번 다음과 같이 보입니다.

```
nightly: * - - * - - *
```

6주마다 새로운 릴리스를 준비할 시간입니다! Rust 저장소의 `베타` 분기는 nightly에서 사용하는 `마스터` 분기에서 분기됩니다. 이제 두 가지 릴리스가 있습니다.

```
nightly: * - - * - - *
                     |
beta:                *
```

대부분의 Rust 사용자는 베타 릴리스를 적극적으로 사용하지 않지만 Rust가 가능한 회귀를 발견할 수 있도록 CI 시스템에서 베타에 대해 테스트합니다. 그 동안 매일 밤마다 야간 릴리스가 있습니다.

```
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

회귀가 발견되었다고 가정 해 봅시다. 회귀가 안정적인 릴리스에 들어가기 전에 베타 릴리스를 테스트할 시간이 있어서 다행입니다! 수정 사항이 `마스터`에 적용되어 nightly가 수정된 다음 수정 사항이 `베타` 분기로 백포트되고 새로운 베타 릴리스가 생성됩니다.

```
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

첫 번째 베타가 생성된 지 6주가 지나면 안정적인 릴리스를 시작할 때입니다! `안정적인` 분기는 `베타` 분기에서 생성됩니다.

```
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

만세! 러스트 1.5가 완료되었습니다! 그러나 우리는 한 가지를 잊었습니다. 6주가 지났기 때문에 Rust의 *다음 버전인 1.6의 새로운 베타 버전도 필요합니다.* 따라서 `stable`이 `beta`에서 분기된 후 `beta`의 다음 버전이 `nightly`에서 다시 분기됩니다.

```
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

6주마다 릴리스가 `정류장을 떠나지만` 안정적인 릴리스로 도착하기 전에 베타 채널을 통과해야 하기 때문에 이를 `기차 모델`이라고 합니다.

Rust는 시계 장치처럼 6주마다 릴리스됩니다. 한 Rust 릴리스의 날짜를 알면 다음 날짜를 알 수 있습니다. 6주 후입니다. 6주마다 출시 일정을 잡는 것의 좋은 점은 다음 기차가 곧 온다는 것입니다. 특정 기능이 특정 릴리스를 놓치더라도 걱정할 필요가 없습니다. 곧 다른 기능이 출시될 것입니다! 이렇게 하면 릴리스 마감일에 가까워질 가능성이 있는 다듬지 않은 기능을 몰래 숨겨야 한다는 압력을 줄이는 데 도움이 됩니다.

이 프로세스 덕분에 언제든지 Rust의 다음 빌드를 확인하고 업그레이드하기 쉬운지 직접 확인할 수 있습니다. 베타 릴리스가 예상대로 작동하지 않으면 팀에 보고하고 다음 안정 릴리스가 발생합니다! 베타 릴리스에서 파손은 상대적으로 드물지만 `rustc`는 여전히 소프트웨어의 일부이며 버그가 존재합니다.

### [불안정한 기능](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#unstable-features)

이 릴리스 모델에는 불안정한 기능이 하나 더 있습니다. Rust는 `기능 플래그`라는 기술을 사용하여 주어진 릴리스에서 어떤 기능을 사용할 수 있는지 결정합니다. 새 기능이 활발히 개발 중인 경우 `마스터`에 도달하므로 야간이지만 *기능 플래그* 뒤에 있습니다. 사용자로서 진행 중인 기능을 시험해보고 싶다면 가능하지만 Rust의 나이틀리 릴리스를 사용하고 소스 코드에 옵트인할 적절한 플래그로 주석을 달아야 합니다.

Rust의 베타 또는 안정적인 릴리스를 사용하는 경우 기능 플래그를 사용할 수 없습니다. 이것은 우리가 새로운 기능을 영원히 안정적이라고 선언하기 전에 실제로 사용할 수 있게 해주는 열쇠입니다. 최신 기술을 선택하려는 사용자는 그렇게 할 수 있으며 견고한 경험을 원하는 사용자는 안정을 고수할 수 있으며 코드가 손상되지 않는다는 것을 알 수 있습니다. 정체가 없는 안정성.

이 책에는 진행 중인 기능이 계속 변경되고 있기 때문에 안정적인 기능에 대한 정보만 포함되어 있으며 이 책이 쓰여진 시점과 안정적인 빌드에서 활성화되는 시점 사이에는 분명히 다를 것입니다. 야간 전용 기능에 대한 설명서는 온라인에서 찾을 수 있습니다.

### [Rustup과 Rust Nightly의 역할](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#rustup-and-the-role-of-rust-nightly)

Rustup을 사용하면 글로벌 또는 프로젝트별로 Rust의 서로 다른 릴리스 채널 간에 쉽게 변경할 수 있습니다. 기본적으로 안정적인 Rust가 설치되어 있습니다. 예를 들어 야간에 설치하려면 다음과 같이 하십시오.

```bash
$ rustup toolchain install nightly
```

`rustup`으로 설치한 모든 *툴체인 (Rust 릴리스 및 관련 구성 요소)도 볼 수 있습니다.* 다음은 작성자의 Windows 컴퓨터 중 하나에 대한 예입니다.

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

보시다시피 안정적인 툴체인이 기본값입니다. 대부분의 Rust 사용자는 대부분 안정을 사용합니다. 대부분의 경우 안정적으로 사용하고 싶을 수 있지만 최신 기능에 관심이 있기 때문에 특정 프로젝트에서는 야간에 사용하십시오. 이렇게 하려면 해당 프로젝트의 디렉토리에서 `rustup override`를 사용하여 해당 디렉토리에 있을 때 `rustup`이 사용해야 하는 야간 툴체인을 설정할 수 있습니다.

```bash
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

*이제 ~/projects/needs-nightly* 내부에서 `rustc` 또는 `cargo`를 호출할 때마다 `rustup`은 기본 안정적인 Rust가 아닌 야간 Rust를 사용하고 있는지 확인합니다. 이는 Rust 프로젝트가 많을 때 유용합니다!

### [RFC 프로세스 및 팀](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#the-rfc-process-and-teams)

그렇다면 이러한 새로운 기능에 대해 어떻게 알 수 있습니까? Rust의 개발 모델은 *RFC(Request For Comments) 프로세스를* 따릅니다. Rust를 개선하고 싶다면 RFC라는 제안을 작성할 수 있습니다.

누구든지 Rust를 개선하기 위해 RFC를 작성할 수 있으며 제안은 많은 주제 하위 팀으로 구성된 Rust 팀에서 검토하고 논의합니다. [Rust의 웹사이트에](https://www.rust-lang.org/governance) 팀의 전체 목록이 있습니다. 여기에는 언어 설계, 컴파일러 구현, 인프라, 문서 등 프로젝트의 각 영역에 대한 팀이 포함되어 있습니다. 적절한 팀이 제안서와 의견을 읽고, 의견을 직접 작성하고, 결국 기능을 수락하거나 거부하기 위한 합의가 이루어집니다.

기능이 승인되면 Rust 리포지토리에 이슈가 열리고 누군가 이를 구현할 수 있습니다. 그것을 아주 잘 구현하는 사람은 애초에 그 기능을 제안한 사람이 아닐 수도 있습니다! [구현이 준비되면 `불안정한 기능`](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#unstable-features) 섹션 에서 논의한 것처럼 기능 게이트 뒤의 `마스터` 브랜치에 도달합니다.

시간이 지나면 나이틀리 릴리스를 사용하는 Rust 개발자가 새로운 기능을 시험해 볼 수 있게 되면 팀원들은 이 기능에 대해 논의하고 나이틀리에서 어떻게 작동하는지, 안정적인 Rust로 만들어야 하는지 여부를 결정할 것입니다. 앞으로 나아가기로 결정했다면 기능 게이트가 제거되고 이제 기능이 안정적인 것으로 간주됩니다! Rust의 안정적인 새 릴리스로 열차를 타고 이동합니다.