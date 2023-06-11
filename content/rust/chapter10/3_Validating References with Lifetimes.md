+++
title = "Validating References with Lifetimes"
weight = 3
template ="book_page_rust.html"

+++

## 요약



## [수명으로 참조 유효성 검사](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes)

수명은 우리가 이미 사용하고 있는 또 다른 종류의 제네릭입니다. 타입이 우리가 원하는 행동을 하도록 보장하는 대신, 라이프타임은 참조가 우리가 필요로 하는 한 유효하도록 보장합니다.

[4장의 `참조 및 차용`](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing) 섹션 에서 논의하지 않은 한 가지 세부 사항은 Rust의 모든 참조에는 해당 참조가 유효한 범위인 *수명이 있다는 것입니다.* 대부분의 경우 수명은 대부분의 경우 유형이 유추되는 것처럼 암시적이고 유추됩니다. 여러 유형이 가능한 경우에만 유형에 주석을 달아야 합니다. 비슷한 방식으로 참조의 수명이 몇 가지 다른 방식으로 관련될 수 있는 경우 수명에 주석을 달아야 합니다. Rust는 런타임에 사용되는 실제 참조가 확실히 유효하도록 일반 수명 매개변수를 사용하여 관계에 주석을 달도록 요구합니다.

수명에 주석을 다는 것은 대부분의 다른 프로그래밍 언어가 가지고 있는 개념도 아니므로 생소하게 느껴질 것입니다. 이 장에서 라이프타임 전체를 다루지는 않겠지만, 개념에 익숙해질 수 있도록 라이프타임 구문을 만날 수 있는 일반적인 방법에 대해 논의할 것입니다.

### [수명이 있는 댕글링 참조 방지](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#preventing-dangling-references-with-lifetimes)

수명의 주요 목표는 프로그램이 참조하려는 데이터가 아닌 다른 데이터를 참조하게 하는 *매달린 참조를 방지하는 것입니다.* 외부 범위와 내부 범위가 있는 Listing 10-16의 프로그램을 고려하십시오.

```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!(`r: {}`, r);
}
```

목록 10-16: 값이 범위를 벗어난 참조를 사용하려는 시도

> 참고: Listings 10-16, 10-17 및 10-23의 예제는 초기 값을 제공하지 않고 변수를 선언하므로 변수 이름은 외부 범위에 존재합니다. 언뜻 보기에 이는 Rust가 null 값을 갖지 않는 것과 충돌하는 것처럼 보일 수 있습니다. 그러나 값을 주기 전에 변수를 사용하려고 하면 Rust가 실제로 null 값을 허용하지 않는다는 것을 보여주는 컴파일 타임 오류가 발생합니다.

외부 범위는 초기값이 없는 명명된 변수를 선언 `r`하고 내부 범위는 초기값 5로 명명된 변수를 선언합니다 `x`. 내부 범위 내에서 의 값을 에 대한 `r`참조로 설정하려고 시도합니다 `x`. 그런 다음 내부 범위가 종료되고 에 값을 인쇄하려고 시도합니다 `r`. `r`이 코드는 사용하려고 시도하기 전에 참조하는 값이 범위를 벗어났기 때문에 컴파일되지 않습니다. 오류 메시지는 다음과 같습니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!(`r: {}`, r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

변수는 `x``충분히 오래 살지` 않습니다. 그 이유는 `x`내부 범위가 7행에서 끝날 때 범위를 벗어나기 때문입니다. 그러나 `r`외부 범위에는 여전히 유효합니다. 그 범위가 더 크기 때문에 우리는 그것이 `더 오래 산다`고 말합니다. Rust가 이 코드가 작동하도록 허용하면 범위를 벗어날 `r`때 할당 해제된 메모리를 참조하게 되고 우리가 시도한 모든 작업이 올바르게 작동하지 않을 것입니다. 그렇다면 Rust는 이 코드가 유효하지 않다는 것을 어떻게 판단할까요? 차입 검사기를 사용합니다.`x``r`

### [차용 체커](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-borrow-checker)

Rust 컴파일러에는 범위를 비교하여 모든 차용이 유효한지 확인하는 *차용 검사기가 있습니다.* 목록 10-17은 목록 10-16과 동일한 코드를 보여주지만 변수의 수명을 보여주는 주석이 있습니다.

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!(`r: {}`, r); //          |
}                         // ---------+
```

`r`Listing 10-17: 및 `x`의 수명에 `'a`대한 `'b`주석

```
r`여기에서 with 의 수명 과 with `'a`의 수명에 주석을 달았습니다. 보시다시피 내부 블록은 외부 수명 블록보다 훨씬 작습니다. 컴파일 시간에 Rust는 두 수명의 크기를 비교하여 수명이 이지만 수명이 . 다음 보다 짧기 때문에 프로그램이 거부됩니다. 참조 대상이 참조만큼 오래 살지 않습니다.`x``'b``'b``'a``r``'a``'b``'b``'a
```

목록 10-18은 코드를 수정하여 매달린 참조가 없고 오류 없이 컴파일합니다.

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!(`r: {}`, r); //   |       |
                          // --+       |
}                         // ----------+
```

Listing 10-18: 데이터가 참조보다 수명이 길기 때문에 유효한 참조

여기서 `x`수명은 이며 `'b`, 이 경우에는 수명이 입니다 `'a`. 이것은 `r`참조할 수 있음을 의미합니다 `x`. Rust는 참조가 유효한 `r`동안 참조가 항상 유효하다는 것을 알고 있기 때문입니다.`x`

이제 참조의 수명이 어디에 있고 Rust가 참조가 항상 유효한지 확인하기 위해 수명을 분석하는 방법을 알았으므로 함수 컨텍스트에서 매개 변수의 일반적인 수명과 반환 값을 살펴보겠습니다.

### [함수의 일반 수명](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#generic-lifetimes-in-functions)

우리는 두 문자열 조각 중 더 긴 것을 반환하는 함수를 작성할 것입니다. 이 함수는 두 개의 문자열 슬라이스를 가져와 단일 문자열 슬라이스를 반환합니다. 함수 를 구현한 후 `longest`목록 10-19의 코드는 를 인쇄해야 합니다 `The longest string is abcd`.

파일 이름: src/main.rs

```rust
fn main() {
    let string1 = String::from(`abcd`);
    let string2 = `xyz`;

    let result = longest(string1.as_str(), string2);
    println!(`The longest string is {}`, result);
}
```

목록 10-19: 두 스트링 슬라이스 중 더 긴 것을 찾는 함수를 `main`호출하는 함수`longest`

`longest`우리는 함수가 매개 변수의 소유권을 갖는 것을 원하지 않기 때문에 함수가 문자열이 아닌 참조인 문자열 슬라이스를 취하기를 원한다는 점에 유의하십시오 . 목록 10-19에서 사용하는 매개변수가 우리가 원하는 매개변수인 이유에 대한 자세한 내용은 4장의 [`매개변수로서의 스트링 슬라이스`](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices-as-parameters) 섹션을 참조하십시오 .

Listing 10-20과 같이 함수를 구현하려고 하면 `longest`컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

목록 10-20: `longest`두 문자열 슬라이스 중 더 긴 문자열을 반환하지만 아직 컴파일되지 않은 함수 의 구현

대신 수명에 대해 말하는 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
x`도움말 텍스트는 Rust가 반환되는 참조 가 또는 를 참조하는지 여부를 알 수 없기 때문에 반환 유형에 일반 수명 매개변수가 필요함을 나타냅니다 `y`. `if`실제로 이 함수 본문의 블록이 에 대한 참조를 반환 `x`하고 블록이 ! `else`에 대한 참조를 반환하기 때문에 우리도 모릅니다.`y
```

이 함수를 정의할 때 이 함수에 전달될 구체적인 값을 모르기 때문에 케이스 `if`또는 `else`케이스가 실행될지 알 수 없습니다. 우리는 또한 전달될 참조의 구체적인 수명을 모르기 때문에 목록 10-17 및 10-18에서 우리가 반환하는 참조가 항상 유효한지 여부를 결정하기 위해 범위를 볼 수 없습니다. . 차용 검사기는 반환 값의 수명이 어떻게 관련되어 있는지 `x`모르기 때문에 이것을 결정할 수 없습니다. `y`이 오류를 수정하기 위해 차용 검사기가 분석을 수행할 수 있도록 참조 간의 관계를 정의하는 일반 수명 매개변수를 추가합니다.

### [평생 주석 구문](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotation-syntax)

평생 주석은 참조가 지속되는 기간을 변경하지 않습니다. 오히려 수명에 영향을 주지 않고 서로에 대한 여러 참조의 수명 관계를 설명합니다. 서명이 일반 유형 매개변수를 지정하면 함수가 모든 유형을 허용할 수 있는 것처럼 함수는 일반 수명 매개변수를 지정하여 모든 수명의 참조를 허용할 수 있습니다.

수명 주석에는 약간 특이한 구문이 있습니다. 수명 매개변수의 이름은 아포스트로피( `'`)로 시작해야 하며 일반적으로 모두 소문자이며 일반 유형과 같이 매우 짧습니다. `'a`대부분의 사람들은 첫 번째 평생 주석의 이름을 사용합니다. `&`참조 유형에서 주석을 구분하기 위해 공백을 사용하여 참조 뒤에 수명 매개변수 주석을 배치합니다.

`i32`다음은 몇 가지 예입니다: 수명 매개변수가 없는 an에 대한 참조 , `i32`이라는 수명 매개변수가 있는 an에 대한 참조 `'a`, 그리고 `i32`life가 있는 an에 대한 변경 가능한 참조 `'a`.

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

하나의 라이프타임 주석 자체는 큰 의미가 없습니다. 왜냐하면 주석은 여러 참조의 일반적인 라이프타임 매개변수가 서로 어떻게 관련되어 있는지 Rust에 알리기 위한 것이기 때문입니다. 수명 주석이 함수 컨텍스트에서 서로 어떻게 관련되는지 살펴보겠습니다 `longest`.

### [함수 서명의 수명 주석](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-function-signatures)

*함수 서명에서 라이프타임 주석을 사용하려면 일반 유형 매개* 변수에서 했던 것처럼 함수 이름과 매개변수 목록 사이의 꺾쇠괄호 안에 일반 수명 *매개* 변수 를 선언해야 합니다.

서명이 다음 제약 조건을 표현하기를 원합니다. 반환된 참조는 두 매개 변수가 모두 유효한 한 유효합니다. 이것은 매개변수의 수명과 반환 값 간의 관계입니다. 목록 10-21에 표시된 대로 수명에 이름을 지정한 `'a`다음 각 참조에 추가합니다.

파일 이름: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

목록 10-21: `longest`서명의 모든 참조가 동일한 수명을 가져야 한다고 지정하는 함수 정의`'a`

`main`이 코드는 목록 10-19의 함수 와 함께 사용할 때 원하는 결과를 컴파일하고 생성해야 합니다.

함수 시그니처는 이제 러스트에게 어떤 수명 동안 `'a`함수가 두 개의 매개변수를 취한다고 알려줍니다. 둘 다 최소한 수명만큼 사는 문자열 조각입니다 `'a`. 함수 서명은 또한 Rust에게 함수에서 반환된 문자열 조각이 적어도 평생만큼은 살아있을 것이라고 알려줍니다 `'a`. 실제로 이는 함수가 반환하는 참조의 수명이 `longest`함수 인수가 참조하는 값의 수명 중 더 작은 수명과 동일함을 의미합니다. 이러한 관계는 Rust가 이 코드를 분석할 때 사용하기를 원하는 것입니다.

이 함수 서명에서 수명 매개변수를 지정할 때 전달되거나 반환된 값의 수명을 변경하지 않는다는 점을 기억하십시오. 오히려 우리는 차용 검사기가 이러한 제약 조건을 준수하지 않는 모든 값을 거부하도록 지정하고 있습니다. 함수 가 얼마나 오래 지속 되는지 `longest`정확히 알 필요는 없으며 이 서명을 충족하는 일부 범위만 대체할 수 있습니다.`x``y``'a`

함수의 수명에 주석을 추가할 때 주석은 함수 본문이 아닌 함수 서명에 포함됩니다. 수명 주석은 서명의 유형과 마찬가지로 함수 계약의 일부가 됩니다. 함수 서명이 수명 계약을 포함한다는 것은 Rust 컴파일러가 수행하는 분석이 더 간단할 수 있음을 의미합니다. 함수에 주석을 달거나 호출하는 방식에 문제가 있는 경우 컴파일러 오류는 코드의 일부와 제약 조건을 더 정확하게 가리킬 수 있습니다. 대신에 Rust 컴파일러가 우리가 의도한 수명 관계에 대해 더 많은 추론을 했다면, 컴파일러는 문제의 원인에서 몇 단계 떨어진 우리 코드의 사용만을 지적할 수 있을 것입니다.

에 대한 구체적인 참조를 전달할 때 `longest`대체되는 구체적인 수명은 의 범위 와 겹치는 범위 `'a`의 일부입니다. 즉, 일반 수명은 및 의 수명 중 더 작은 수명과 동일한 구체적인 수명을 갖게 됩니다. 동일한 수명 매개변수로 반환된 참조에 주석을 달았기 때문에 반환된 참조는 및 의 수명 중 더 작은 수명의 길이에 대해서도 유효합니다.`x``y``'a``x``y``'a``x``y`

`longest`구체적인 수명이 다른 참조를 전달하여 수명 주석이 함수를 제한하는 방법을 살펴보겠습니다. 목록 10-22는 간단한 예입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let string1 = String::from(`long string is long`);

    {
        let string2 = String::from(`xyz`);
        let result = longest(string1.as_str(), string2.as_str());
        println!(`The longest string is {}`, result);
    }
}
```

목록 10-22: 구체적인 수명이 다른 값 `longest`에 대한 참조와 함께 함수 사용`String`

이 예에서 `string1`is는 외부 범위 끝까지 유효하고 `string2`내부 범위 끝까지 유효하며 `result`내부 범위 끝까지 유효한 것을 참조합니다. 이 코드를 실행하면 빌림 검사기가 승인하는 것을 볼 수 있습니다. 컴파일하고 인쇄합니다 `The longest string is long string is long`.

다음으로, 참조의 수명이 `result`두 인수의 더 작은 수명이어야 함을 보여주는 예제를 시도해 봅시다. 내부 범위 외부로 변수 선언을 이동 하지만 범위 내부의 변수 `result`에 대한 값 할당은 . 그런 다음 내부 범위가 종료된 후 사용하는 를 내부 범위 외부로 이동합니다. 목록 10-23의 코드는 컴파일되지 않습니다.`result``string2``println!``result`

파일 이름: src/main.rs

```rust
fn main() {
    let string1 = String::from(`long string is long`);
    let result;
    {
        let string2 = String::from(`xyz`);
        result = longest(string1.as_str(), string2.as_str());
    }
    println!(`The longest string is {}`, result);
}
```

목록 10-23: 범위를 벗어난 `result`후 사용 시도`string2`

이 코드를 컴파일하려고 하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!(`The longest string is {}`, result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

이 오류는 for 가 문에 `result`대해 유효하려면 외부 범위가 끝날 때까지 유효해야 함을 보여줍니다. 러스트는 우리가 동일한 수명 매개변수를 사용하여 함수 매개변수와 반환 값의 수명에 주석을 달았기 때문에 이것을 알고 있습니다.`println!``string2``'a`

인간으로서 우리는 이 코드를 볼 수 있고 이것이 `string1`에 대한 참조를 포함할 것 `string2`입니다. 가 아직 범위를 벗어나지 않았기 때문에 에 대한 참조는 여전히 명령문에 유효합니다. 그러나 컴파일러는 이 경우 참조가 유효한지 확인할 수 없습니다. 우리는 러스트에게 함수에 의해 반환된 참조의 수명이 전달 된 참조의 수명 중 더 작은 수명과 같다고 말했습니다. 따라서 차용 검사기는 목록 10-23의 코드가 유효하지 않은 참조를 가질 수 있는 것으로 허용하지 않습니다.`result``string1``string1``string1``println!``longest`

함수에 전달된 참조의 값과 수명 및 `longest`반환된 참조가 사용되는 방법을 변경하는 더 많은 실험을 설계해 보십시오. 컴파일하기 전에 실험이 차용 검사기를 통과할지 여부에 대한 가설을 세우십시오. 그런 다음 당신이 옳은지 확인하십시오!

### [수명의 관점에서 생각하기](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#thinking-in-terms-of-lifetimes)

수명 매개변수를 지정하는 방법은 함수가 수행하는 작업에 따라 다릅니다. 예를 들어 가장 긴 문자열 조각이 아닌 항상 첫 번째 매개변수를 반환하도록 함수의 구현을 변경한 경우 매개변수 `longest`에 수명을 지정할 필요가 없습니다 `y`. 다음 코드는 컴파일됩니다.

파일 이름: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
'a`매개변수 및 반환 유형에 대해 수명 매개변수를 지정했지만 `x`매개변수에 대해서는 수명 매개변수를 지정하지 않았습니다 `y`. 의 수명은 수명 또는 반환 값 `y`과 관련이 없기 때문입니다.`x
```

함수에서 참조를 반환할 때 반환 유형에 대한 수명 매개변수는 매개변수 중 하나에 대한 수명 매개변수와 일치해야 합니다. 반환된 참조가 매개 변수 중 하나를 참조 *하지 않는* 경우 이 함수 내에서 생성된 값을 참조해야 합니다. 그러나 이것은 값이 함수의 끝에서 범위를 벗어나기 때문에 댕글링 참조가 됩니다. `longest`컴파일되지 않는 함수 의 시도된 구현을 고려하십시오 .

파일 이름: src/main.rs

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from(`really long string`);
    result.as_str()
}
```

`'a`여기 에서 반환 유형에 대해 수명 매개변수를 지정했지만 반환 값 수명이 매개변수의 수명과 전혀 관련이 없기 때문에 이 구현은 컴파일에 실패합니다. 우리가 얻는 오류 메시지는 다음과 같습니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return reference to local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error
```

문제는 `result`범위를 벗어나 함수가 끝날 때 정리된다는 것입니다 `longest`. `result`또한 함수에서 참조를 반환하려고 합니다. 댕글링 참조를 변경하는 수명 매개변수를 지정할 수 있는 방법이 없으며 Rust는 댕글링 참조를 생성하도록 허용하지 않습니다. 이 경우 가장 좋은 해결 방법은 참조가 아닌 소유한 데이터 유형을 반환하여 호출 함수가 값 정리를 담당하도록 하는 것입니다.

궁극적으로 수명 구문은 다양한 매개 변수의 수명과 함수의 반환 값을 연결하는 것입니다. 일단 연결되면 Rust는 메모리 안전 작업을 허용하고 댕글링 포인터를 생성하거나 메모리 안전을 위반하는 작업을 허용하지 않는 충분한 정보를 갖게 됩니다.

### [구조체 정의의 수명 주석](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-struct-definitions)

지금까지 우리가 정의한 모든 구조체는 소유 유형을 보유합니다. 참조를 보유하도록 구조체를 정의할 수 있지만 이 경우 구조체 정의의 모든 참조에 수명 주석을 추가해야 합니다. `ImportantExcerpt`목록 10-24 에는 스트링 슬라이스를 보유하는 명명된 구조체가 있습니다.

파일 이름: src/main.rs

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from(`Call me Ishmael. Some years ago...`);
    let first_sentence = novel.split('.').next().expect(`Could not find a '.'`);
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

목록 10-24: 수명 주석이 필요한 참조를 보유하는 구조체

`part`이 구조체에는 참조인 문자열 슬라이스를 포함하는 단일 필드가 있습니다. 일반 데이터 유형과 마찬가지로 구조체 이름 뒤의 꺾쇠 괄호 안에 일반 수명 매개변수의 이름을 선언하여 구조체 정의 본문에서 수명 매개변수를 사용할 수 있습니다. 이 주석은 의 `ImportantExcerpt`인스턴스가 해당 필드에 보유하고 있는 참조보다 오래 지속될 수 없음을 의미합니다 `part`.

여기서 함수 는 변수가 소유한 첫 번째 문장에 대한 참조를 보유하는 구조체 `main`의 인스턴스를 만듭니다. 인스턴스가 생성되기 전에 데이터가 존재합니다. 또한 범위를 벗어날 때까지 범위를 벗어나지 않으므로 인스턴스의 참조가 유효합니다.`ImportantExcerpt``String``novel``novel``ImportantExcerpt``novel``ImportantExcerpt``ImportantExcerpt`

### [평생 제거](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision)

모든 참조에는 수명이 있으며 참조를 사용하는 함수 또는 구조체에 대한 수명 매개 변수를 지정해야 한다는 것을 배웠습니다. 그러나 4장에서 우리는 Listing 4-9에 수명 주석 없이 컴파일된 Listing 10-25에 다시 표시된 함수가 있었습니다.

파일 이름: src/lib.rs

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

목록 10-25: 매개변수와 반환 유형이 참조인 경우에도 수명 주석 없이 컴파일된 목록 4-9에서 정의한 함수

이 함수가 수명 주석 없이 컴파일되는 이유는 역사적입니다. Rust의 초기 버전(1.0 이전)에서는 모든 참조에 명시적 수명이 필요했기 때문에 이 코드가 컴파일되지 않았을 것입니다. 당시 함수 서명은 다음과 같이 작성되었을 것입니다.

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

많은 Rust 코드를 작성한 후 Rust 팀은 Rust 프로그래머가 특정 상황에서 동일한 수명 주석을 반복해서 입력하고 있음을 발견했습니다. 이러한 상황은 예측 가능했으며 몇 가지 결정론적 패턴을 따랐습니다. 개발자는 차용 검사기가 이러한 상황에서 수명을 추론할 수 있고 명시적인 주석이 필요하지 않도록 컴파일러 코드에 이러한 패턴을 프로그래밍했습니다.

Rust 역사의 이 부분은 더 결정적인 패턴이 나타나고 컴파일러에 추가될 가능성이 있기 때문에 관련이 있습니다. 앞으로는 더 적은 수명 주석이 필요할 수 있습니다.

Rust의 참조 분석에 프로그래밍된 패턴을 *수명 제거 규칙* 이라고 합니다. 이것은 프로그래머가 따라야 할 규칙이 아닙니다. 이들은 컴파일러가 고려할 특정 사례의 집합이며, 코드가 이러한 경우에 적합하면 수명을 명시적으로 작성할 필요가 없습니다.

제거 규칙은 완전한 추론을 제공하지 않습니다. Rust가 결정적으로 규칙을 적용하지만 참조의 수명에 대해 여전히 모호한 경우 컴파일러는 나머지 참조의 수명을 추측하지 않습니다. 추측하는 대신 컴파일러는 수명 주석을 추가하여 해결할 수 있는 오류를 제공합니다.

*함수 또는 메소드 매개변수에 대한 수명을 입력 수명* 이라고 하고 반환 값에 대한 수명을 *출력 수명* 이라고 합니다.

컴파일러는 명시적인 주석이 없을 때 세 가지 규칙을 사용하여 참조의 수명을 파악합니다. 첫 번째 규칙은 입력 수명에 적용되고 두 번째 및 세 번째 규칙은 출력 수명에 적용됩니다. 컴파일러가 세 가지 규칙의 끝에 도달하고 수명을 파악할 수 없는 참조가 여전히 있는 경우 컴파일러는 오류와 함께 중지됩니다. 이러한 규칙은 블록 `fn`뿐만 아니라 정의 에도 적용됩니다 `impl`.

첫 번째 규칙은 컴파일러가 참조인 각 매개변수에 수명 매개변수를 할당한다는 것입니다. 즉, 하나의 매개변수가 있는 함수는 하나의 라이프타임 매개변수를 얻습니다: `fn foo<'a>(x: &'a i32)`; 두 개의 매개변수가 있는 함수는 두 개의 별도 수명 매개변수를 얻습니다: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; 등등.

두 번째 규칙은 정확히 하나의 입력 수명 매개변수가 있는 경우 해당 수명이 모든 출력 수명 매개변수에 할당된다는 것입니다 `fn foo<'a>(x: &'a i32) -> &'a i32`.

세 번째 규칙은 입력 라이프타임 매개변수가 여러 개인 경우 그 중 하나가 메서드이기 `&self`때문에 모든 출력 라이프타임 매개변수에 `&mut self`수명이 `self`할당된다는 것입니다. 이 세 번째 규칙은 더 적은 수의 기호가 필요하기 때문에 메소드를 읽고 쓰기에 훨씬 더 좋습니다.

우리가 컴파일러라고 가정해 봅시다. `first_word`목록 10-25에 있는 함수 서명에서 참조의 수명을 파악하기 위해 이러한 규칙을 적용할 것입니다. 서명은 참조와 연결된 수명 없이 시작됩니다.

```rust
fn first_word(s: &str) -> &str {
```

그런 다음 컴파일러는 각 매개변수가 자체 수명을 갖도록 지정하는 첫 번째 규칙을 적용합니다. 평소와 같이 호출하므로 `'a`이제 서명은 다음과 같습니다.

```rust
fn first_word<'a>(s: &'a str) -> &str {
```

정확히 하나의 입력 수명이 있기 때문에 두 번째 규칙이 적용됩니다. 두 번째 규칙은 하나의 입력 매개변수의 수명이 출력 수명에 할당되도록 지정하므로 서명은 이제 다음과 같습니다.

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

이제 이 함수 서명의 모든 참조에는 수명이 있으며 컴파일러는 프로그래머가 이 함수 서명의 수명에 주석을 달 필요 없이 분석을 계속할 수 있습니다.

또 다른 예를 살펴보겠습니다. 이번에는 `longest`목록 10-20에서 작업을 시작했을 때 수명 매개변수가 없었던 함수를 사용했습니다.

```rust
fn longest(x: &str, y: &str) -> &str {
```

첫 번째 규칙을 적용해 보겠습니다. 각 매개변수는 고유한 수명을 갖습니다. 이번에는 하나가 아닌 두 개의 매개변수가 있으므로 두 개의 수명이 있습니다.

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

두 개 이상의 입력 수명이 있기 때문에 두 번째 규칙이 적용되지 않는 것을 볼 수 있습니다. 세 번째 규칙도 적용되지 않습니다. 는 `longest`메서드가 아니라 함수이므로 매개변수 중 어느 것도 `self`. 세 가지 규칙을 모두 살펴본 후에도 여전히 반환 유형의 수명이 무엇인지 파악하지 못했습니다. 이것이 Listing 10-20의 코드를 컴파일하려고 시도하는 동안 오류가 발생한 이유입니다. 컴파일러는 수명 생략 규칙을 통해 작업했지만 여전히 서명에 있는 참조의 모든 수명을 파악할 수 없었습니다.

세 번째 규칙은 실제로 메서드 서명에만 적용되기 때문에 세 번째 규칙이 메서드 서명에서 수명에 자주 주석을 달 필요가 없음을 의미하는 이유를 알아보기 위해 해당 컨텍스트에서 수명을 살펴보겠습니다.

### [메서드 정의의 수명 주석](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-method-definitions)

수명이 있는 구조체에 메서드를 구현할 때 목록 10-11에 표시된 제네릭 형식 매개 변수와 동일한 구문을 사용합니다. 수명 매개변수를 선언하고 사용하는 위치는 구조 필드 또는 메서드 매개변수 및 반환 값과 관련되는지 여부에 따라 다릅니다.

구조체 필드의 수명 이름은 항상 키워드 다음에 선언한 다음 구조체 이름 다음에 사용해야 합니다 `impl`. 이러한 수명은 구조체 유형의 일부이기 때문입니다.

블록 내부의 메서드 서명에서 `impl`참조는 구조체 필드의 참조 수명에 연결되거나 독립적일 수 있습니다. 또한 수명 생략 규칙은 종종 메서드 서명에 수명 주석이 필요하지 않도록 합니다. `ImportantExcerpt`목록 10-24에서 정의한 구조체 이름을 사용하는 몇 가지 예를 살펴보겠습니다.

```
level`먼저, 유일한 매개변수가 에 대한 참조 이고 반환 값이 어떤 것에도 참조가 아닌 `self`an 이라는 이름의 메서드를 사용합니다.`i32
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

이후의 수명 매개변수 선언 과 유형 이름 이후의 사용이 필요하지만 첫 번째 생략 규칙 때문에 `impl`참조의 수명에 주석을 달 필요가 없습니다.`self`

다음은 세 번째 평생 생략 규칙이 적용되는 예입니다.

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!(`Attention please: {}`, announcement);
        self.part
    }
}
```

두 개의 입력 수명이 있으므로 Rust는 첫 번째 수명 생략 규칙을 적용하고 둘 다 `&self`자신 `announcement`의 수명을 제공합니다. 그런 다음 매개변수 중 하나가 이므로 `&self`반환 유형은 의 수명을 가져오고 `&self`모든 수명이 고려되었습니다.

### [정적 수명](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-static-lifetime)

우리가 논의해야 할 하나의 특별한 수명은 `'static`영향을 받는 참조가 프로그램의 전체 기간 동안 존재할 *수 있음을 나타냅니다.* 모든 문자열 리터럴에는 `'static`다음과 같이 주석을 달 수 있는 수명이 있습니다.

```rust
let s: &'static str = `I have a static lifetime.`;
```

이 문자열의 텍스트는 항상 사용 가능한 프로그램의 바이너리에 직접 저장됩니다. 따라서 모든 문자열 리터럴의 수명은 `'static`.

`'static`오류 메시지에서 수명을 사용하라는 제안을 볼 수 있습니다. 그러나 `'static`참조의 수명으로 지정하기 전에 참조가 실제로 프로그램의 전체 수명 동안 지속되는지 여부와 원하는지 여부를 생각하십시오. 대부분의 경우 수명 을 제안하는 오류 메시지 `'static`는 댕글링 참조를 만들려고 시도하거나 사용 가능한 수명이 일치하지 않아 발생합니다. 이러한 경우 솔루션은 `'static`수명을 지정하는 것이 아니라 해당 문제를 수정하는 것입니다.

## [일반 유형 매개변수, 특성 경계 및 수명 함께](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#generic-type-parameters-trait-bounds-and-lifetimes-together)

제네릭 형식 매개 변수, 특성 범위 및 수명을 모두 하나의 함수로 지정하는 구문을 간단히 살펴보겠습니다!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!(`Announcement! {}`, ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

이것은 `longest`두 문자열 슬라이스 중 더 긴 것을 반환하는 Listing 10-21의 함수입니다. `ann`그러나 이제 제네릭 형식이라는 이름의 추가 매개변수가 있으며 , 이 매개변수는 절 에서 지정한 대로 특성을 `T`구현하는 모든 형식으로 채울 수 있습니다. 이 추가 매개변수는 를 사용하여 인쇄될 것이므로 특성 바인딩이 필요합니다. 수명은 제네릭 유형이므로 수명 매개변수 선언 과 제네릭 유형 매개변수는 함수 이름 뒤의 꺾쇠 괄호 안에 있는 동일한 목록에 들어갑니다.`Display``where``{}``Display``'a``T`

## [요약](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#summary)

이 장에서 많은 내용을 다루었습니다! 이제 일반 유형 매개변수, 특성 및 특성 경계, 일반 수명 매개변수에 대해 알았으므로 다양한 상황에서 작동하는 반복 없이 코드를 작성할 준비가 되었습니다. 일반 유형 매개변수를 사용하면 다른 유형에 코드를 적용할 수 있습니다. 특성 및 특성 범위는 유형이 일반적이더라도 코드에 필요한 동작을 갖도록 합니다. 이 유연한 코드에 매달린 참조가 없도록 수명 주석을 사용하는 방법을 배웠습니다. 그리고 이 모든 분석은 컴파일 시간에 발생하므로 런타임 성능에 영향을 미치지 않습니다!

믿거나 말거나, 이 장에서 논의한 주제에 대해 더 많은 것을 배울 수 있습니다. 17장에서는 특성을 사용하는 또 다른 방법인 특성 개체에 대해 설명합니다. 매우 고급 시나리오에서만 필요한 수명 주석과 관련된 더 복잡한 시나리오도 있습니다. [그런 경우 Rust Reference 를](https://doc.rust-lang.org/reference/index.html) 읽어야 합니다. 그러나 다음에는 Rust로 테스트를 작성하여 코드가 제대로 작동하는지 확인할 수 있습니다.
