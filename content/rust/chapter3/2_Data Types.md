+++
title = "Data Types"
weight = 2
template ="book_page_rust.html"

+++



## 요약

- 

- 



<!-- more -->

## [데이터 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types)

Rust의 모든 값은 특정 *데이터 유형을* 가지고 있으며 , 이는 Rust에게 어떤 종류의 데이터가 지정되는지 알려주므로 해당 데이터로 작업하는 방법을 알 수 있습니다. 스칼라와 복합이라는 두 가지 데이터 유형 하위 집합을 살펴보겠습니다.

*Rust는 정적으로 타입이 지정된* 언어 라는 점을 명심하세요 . 즉, 컴파일 시간에 모든 변수의 타입을 알아야 합니다. 컴파일러는 일반적으로 값과 사용 방법을 기반으로 사용하려는 유형을 추론할 수 있습니다. [2장 ` 추측과 비밀 번호 비교`](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number) 절 에서 `구문 분석`을 사용하여 `문자열`을 숫자 유형으로 변환한 경우와 같이 많은 유형이 가능한 경우에는 다음과 같이 유형 주석을 추가해야 합니다. :

```rust
let guess: u32 = `42`.parse().expect(`Not a number!`);
```

앞의 코드에 표시된 `: u32` 유형 주석을 추가하지 않으면 Rust는 다음과 같은 오류를 표시할 것입니다. 이는 컴파일러가 어떤 유형을 사용하려는지 알기 위해 더 많은 정보가 필요함을 의미합니다.

```console
$ cargo build
   Compiling no_type_annotations v0.1.0 (file:///projects/no_type_annotations)
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = `42`.parse().expect(`Not a number!`);
  |         ^^^^^
  |
help: consider giving `guess` an explicit type
  |
2 |     let guess: _ = `42`.parse().expect(`Not a number!`);
  |              +++

For more information about this error, try `rustc --explain E0282`.
error: could not compile `no_type_annotations` due to previous error
```

다른 데이터 유형에 대한 다른 유형 주석이 표시됩니다.

### [스칼라 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#scalar-types)

스칼라 유형 *은* 단일 값을 나타냅니다. Rust에는 네 가지 기본 스칼라 유형이 있습니다: 정수, 부동 소수점 숫자, 부울 및 문자입니다. 다른 프로그래밍 언어에서 이를 인식할 수 있습니다. Rust에서 어떻게 작동하는지 살펴보겠습니다.

#### [정수 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types)

정수 *는* 분수 구성요소가 없는 숫자입니다. 2장에서 하나의 정수 유형인 `u32` 유형을 사용했습니다. 이 유형 선언은 연관된 값이 32비트 공간을 차지하는 부호 없는 정수(`u` 대신 `i`로 시작하는 부호 있는 정수 유형)여야 함을 나타냅니다. 표 3-1은 Rust에 내장된 정수 유형을 보여줍니다. 이러한 변형을 사용하여 정수 값의 유형을 선언할 수 있습니다.

표 3-1: Rust의 정수형

| 길이    | 서명됨   | 서명되지 않은 |
| ------- | -------- | ------------- |
| 8비트   | `i8`     | `u8`          |
| 16비트  | `i16`    | `u16`         |
| 32비트  | `i32`    | `u32`         |
| 64비트  | `i64`    | `u64`         |
| 128비트 | `i128`   | `u128`        |
| 아치    | `사이즈` | `사용`        |

각 변형은 서명되거나 서명되지 않을 수 있으며 명시적인 크기를 가집니다. *부호 있음* 및 *부호 없음은* 숫자가 음수가 될 수 있는지 여부를 나타냅니다. 즉, 숫자에 부호가 있어야 하는지(signed) 또는 양수만 가능하므로 부호 없이 나타낼 수 있는지(unsigned)를 나타냅니다. ). 그것은 종이에 숫자를 쓰는 것과 같습니다. 부호가 중요한 경우 숫자는 더하기 기호 또는 빼기 기호로 표시됩니다. 그러나 숫자가 양수라고 가정하는 것이 안전할 때는 부호 없이 표시됩니다. [부호 있는 숫자는 2의 보수](https://en.wikipedia.org/wiki/Two's_complement) 표현을 사용하여 저장됩니다 .

각 부호 있는 변형은 -(2n - 1)에서 2n - 1 - 1까지의 숫자를 저장할 수 있습니다. 여기서 *n* 은 변형이 사용하는 비트 수입니다. 따라서 `i8`은 -(27)에서 27 - 1까지의 숫자를 저장할 수 있으며, 이는 -128에서 127까지입니다. 부호 없는 변형은 0에서 2n - 1까지의 숫자를 저장할 수 있으므로 `u8`은 0에서 28 - 1, 0에서 255 사이입니다.

또한 `isize` 및 `usize` 유형은 프로그램이 실행되는 컴퓨터의 아키텍처에 따라 달라지며 표에서 `arch`로 표시됩니다. 64비트 아키텍처를 사용하는 경우 64비트, 32비트 32비트 아키텍처를 사용하는 경우.

표 3-2에 표시된 형식으로 정수 리터럴을 작성할 수 있습니다. 여러 숫자 유형이 될 수 있는 숫자 리터럴은 유형을 지정하기 위해 `57u8`과 같은 유형 접미사를 허용합니다. 숫자 리터럴은 시각적 구분 기호로 `_`를 사용하여 숫자를 더 쉽게 읽을 수 있습니다(예: `1000`을 지정한 것과 동일한 값을 갖는 `1_000`).

표 3-2: Rust의 정수 리터럴

| 숫자 리터럴         | 예            |
| ------------------- | ------------- |
| 소수                | `98_222`      |
| 마녀                | `0xff`        |
| 8진법               | `0o77`        |
| 바이너리            | `0b1111_0000` |
| 바이트(`u8`만 해당) | `바`          |

그렇다면 사용할 정수 유형을 어떻게 알 수 있습니까? 확실하지 않은 경우 Rust의 기본값은 일반적으로 시작하기에 좋은 위치입니다. 정수 유형의 기본값은 `i32`입니다. `isize` 또는 `usize`를 사용하는 기본 상황은 일종의 컬렉션을 인덱싱할 때입니다.

> ##### [정수 오버플로](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-overflow)
>
> 0에서 255 사이의 값을 보유할 수 있는 `u8` 유형의 변수가 있다고 가정해 보겠습니다. 변수를 해당 범위 밖의 값(예: 256)으로 변경하려고 하면 정수 오버플로가 발생하여 두 가지 중 하나가 발생할 수 *있습니다* . 행동. 디버그 모드에서 컴파일할 때 Rust는 이러한 동작이 발생하면 런타임에 프로그램 패닉을 유발하는 정수 오버플로에 대한 검사를 포함합니다 *.* Rust는 프로그램이 오류와 함께 종료될 때 *패닉이라는* 용어를 사용합니다 . 9장의 [`패닉!`이 있는 복구할 수 없는 오류`](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html) 섹션 에서 패닉에 대해 더 깊이 논의할 것입니다 .
>
> `--release` 플래그를 사용하여 릴리스 모드에서 컴파일할 때 Rust는 패닉을 유발하는 정수 오버플로 검사를 포함 *하지 않습니다 .* 대신, 오버플로가 발생하면 Rust는 *2의 보수 래핑을* 수행합니다 . 요컨대, 유형이 보유할 수 있는 최대값보다 큰 값은 유형이 보유할 수 있는 최소값으로 `랩 어라운드`됩니다. `u8`의 경우 값 256은 0이 되고 값 257은 1이 됩니다. 프로그램은 당황하지 않지만 변수는 예상했던 것과 다른 값을 갖게 됩니다. 정수 오버플로의 래핑 동작에 의존하는 것은 오류로 간주됩니다.
>
> 오버플로 가능성을 명시적으로 처리하기 위해 기본 숫자 유형에 대해 표준 라이브러리에서 제공하는 다음 메서드 계열을 사용할 수 있습니다.
>
> - `wrapping_add`와 같은 `wrapping_*` 메서드를 사용하여 모든 모드에서 래핑합니다.
> - `checked_*` 메서드에 오버플로가 있는 경우 `없음` 값을 반환합니다.
> - `overflowing_*` 메서드로 오버플로가 발생했는지 여부를 나타내는 부울 값과 값을 반환합니다.
> - `saturating_*` 방법을 사용하여 값의 최소값 또는 최대값에서 포화시킵니다.

#### [부동 소수점 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#floating-point-types)

Rust에는 소수점을 가진 숫자인 *부동* 소수점 숫자에 대한 두 가지 기본 유형도 있습니다. Rust의 부동 소수점 유형은 각각 크기가 32비트와 64비트인 `f32`와 `f64`입니다. 기본 유형은 `f64`입니다. 최신 CPU에서는 `f32`와 거의 같은 속도이지만 더 정밀할 수 있기 때문입니다. 모든 부동 소수점 유형은 부호가 있습니다.

다음은 실제 부동 소수점 숫자를 보여주는 예입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

부동 소수점 숫자는 IEEE-754 표준에 따라 표현됩니다. `f32` 유형은 단정밀도 float이고 `f64`는 배정밀도입니다.

#### [수치 연산](https://doc.rust-lang.org/book/ch03-02-data-types.html#numeric-operations)

Rust는 더하기, 빼기, 곱하기, 나누기 및 나머지와 같은 모든 숫자 유형에 대해 예상할 수 있는 기본 수학 연산을 지원합니다. 정수 나눗셈은 0을 향해 가장 가까운 정수로 자릅니다. 다음 코드는 `let` 문에서 각 숫자 연산을 사용하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;
    
    // multiplication
    let product = 4 * 30;
    
    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // Results in -1
    
    // remainder
    let remainder = 43 % 5;
}
```

이러한 문의 각 식은 수학 연산자를 사용하고 단일 값으로 평가된 다음 변수에 바인딩됩니다. [부록 B에는](https://doc.rust-lang.org/book/appendix-02-operators.html) Rust가 제공하는 모든 연산자 목록이 포함되어 있습니다.

#### [부울 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-boolean-type)

대부분의 다른 프로그래밍 언어와 마찬가지로 Rust의 부울 유형에는 `true`와 `false`의 두 가지 가능한 값이 있습니다. 부울은 크기가 1바이트입니다. Rust의 부울 유형은 `bool`을 사용하여 지정됩니다. 예를 들어:

파일 이름: src/main.rs

```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

부울 값을 사용하는 주요 방법은 `if` 식과 같은 조건을 사용하는 것입니다. [`제어 흐름`](https://doc.rust-lang.org/book/ch03-05-control-flow.html#control-flow) 섹션 에서 Rust에서 `if` 표현식이 작동하는 방식을 다룰 것입니다 .

#### [문자 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-character-type)

Rust의 `char` 유형은 언어의 가장 원시적인 알파벳 유형입니다. 다음은 `char` 값을 선언하는 몇 가지 예입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

큰따옴표를 사용하는 문자열 리터럴과 달리 작은따옴표로 `char` 리터럴을 지정합니다. Rust의 `char` 유형은 크기가 4바이트이고 유니코드 스칼라 값을 나타냅니다. 즉, ASCII보다 훨씬 더 많은 것을 나타낼 수 있습니다. 악센트 문자; 중국어, 일본어 및 한국어 문자; 이모티콘; 너비가 0인 공백은 모두 Rust에서 유효한 `char` 값입니다. 유니코드 스칼라 값의 범위는 `U+0000`에서 `U+D7FF`까지, `U+E000`에서 `U+10FFFF`까지입니다. 그러나 `문자`는 실제로 유니코드의 개념이 아니므로 `문자`가 무엇인지에 대한 인간의 직감은 Rust의 `문자`가 무엇인지와 일치하지 않을 수 있습니다. 8장의 [`UTF-8 인코딩된 텍스트를 문자열과 함께 저장`](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings) 에서 이 주제에 대해 자세히 설명합니다 .

### [복합 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#compound-types)

*복합 유형은* 여러 값을 하나의 유형으로 그룹화할 수 있습니다. Rust에는 튜플과 배열이라는 두 가지 기본 복합 유형이 있습니다.

#### [튜플 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type)

튜플 *은* 다양한 유형의 여러 값을 하나의 복합 유형으로 그룹화하는 일반적인 방법입니다. 튜플의 길이는 고정되어 있습니다. 일단 선언되면 크기가 늘어나거나 줄어들 수 없습니다.

괄호 안에 쉼표로 구분된 값 목록을 작성하여 튜플을 만듭니다. 튜플의 각 위치에는 유형이 있으며 튜플의 서로 다른 값의 유형이 동일할 필요는 없습니다. 이 예에서는 선택적 유형 주석을 추가했습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

변수 `tup`은 튜플이 단일 복합 요소로 간주되기 때문에 전체 튜플에 바인딩됩니다. 튜플에서 개별 값을 가져오려면 다음과 같이 패턴 일치를 사용하여 튜플 값을 분해할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;
    
    println!(`The value of y is: {y}`);
}
```

이 프로그램은 먼저 튜플을 만들고 변수 `tup`에 바인딩합니다. 그런 다음 `let`이 포함된 패턴을 사용하여 `tup`을 가져오고 `x`, `y` 및 `z`의 세 가지 개별 변수로 변환합니다. 단일 튜플을 세 부분으로 나누기 때문에 이를 *구조 분해* 라고 합니다 . 마지막으로 프로그램은 `y`의 값인 `6.4`를 인쇄합니다.

마침표(`.`) 다음에 액세스하려는 값의 인덱스를 사용하여 튜플 요소에 직접 액세스할 수도 있습니다. 예를 들어:

파일 이름: src/main.rs

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;
    
    let six_point_four = x.1;
    
    let one = x.2;
}
```

이 프로그램은 튜플 `x`를 만든 다음 해당 인덱스를 사용하여 튜플의 각 요소에 액세스합니다. 대부분의 프로그래밍 언어와 마찬가지로 튜플의 첫 번째 인덱스는 0입니다.

값이 없는 튜플은 특별한 이름인 *unit 을* 가집니다 . 이 값과 해당 유형은 모두 `()`로 작성되며 빈 값 또는 빈 반환 유형을 나타냅니다. 식은 다른 값을 반환하지 않는 경우 암시적으로 단위 값을 반환합니다.

#### [배열 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-array-type)

여러 값의 모음을 갖는 또 다른 방법은 *배열을* 사용하는 것입니다 . 튜플과 달리 배열의 모든 요소는 동일한 유형을 가져야 합니다. 다른 언어의 배열과 달리 Rust의 배열은 길이가 고정되어 있습니다.

대괄호 안에 쉼표로 구분된 목록으로 배열의 값을 씁니다.

파일 이름: src/main.rs

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

배열은 데이터를 힙이 아닌 스택에 할당하려는 경우(스택과 힙에 대해서는 [4장](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap) 에서 자세히 설명함 ) 또는 항상 고정된 수의 요소를 갖도록 하려는 경우에 유용합니다. 그러나 배열은 벡터 유형만큼 유연하지 않습니다. 벡터 *는* 크기를 늘리거나 줄일 수 있는 *표준* 라이브러리에서 제공하는 유사한 컬렉션 유형입니다 . 배열을 사용할지 벡터를 사용할지 확실하지 않은 경우 벡터를 사용해야 할 가능성이 있습니다. [8장](https://doc.rust-lang.org/book/ch08-01-vectors.html) 에서는 벡터에 대해 자세히 설명합니다.

그러나 배열은 요소의 수를 변경할 필요가 없다는 것을 알고 있을 때 더 유용합니다. 예를 들어 프로그램에서 월 이름을 사용하는 경우 벡터가 항상 12개의 요소를 포함한다는 것을 알기 때문에 벡터 대신 배열을 사용할 것입니다.

```rust
let months = [`January`, `February`, `March`, `April`, `May`, `June`, `July`,
              `August`, `September`, `October`, `November`, `December`];
```

다음과 같이 각 요소의 유형, 세미콜론, 배열의 요소 수와 함께 대괄호를 사용하여 배열의 유형을 작성합니다.

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

여기서 `i32`는 각 요소의 유형입니다. 세미콜론 뒤의 숫자 `5`는 배열에 5개의 요소가 포함되어 있음을 나타냅니다.

다음과 같이 초기 값, 세미콜론, 대괄호 안에 배열 길이를 지정하여 각 요소에 대해 동일한 값을 포함하도록 배열을 초기화할 수도 있습니다.

```rust
let a = [3; 5];
```

`a`라는 이름의 배열에는 처음에 값 `3`으로 모두 설정되는 `5` 요소가 포함됩니다. 이것은 `let a = [3, 3, 3, 3, 3];`이라고 쓰는 것과 같습니다. 그러나 더 간결한 방법으로.

##### [배열 요소에 액세스](https://doc.rust-lang.org/book/ch03-02-data-types.html#accessing-array-elements)

배열은 스택에 할당할 수 있는 알려진 고정 크기의 메모리 단일 청크입니다. 다음과 같이 인덱싱을 사용하여 배열의 요소에 액세스할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```

이 예에서 `first`라는 변수는 배열의 인덱스 `[0]`에 있는 값이기 때문에 값 `1`을 가져옵니다. `second`라는 변수는 배열의 인덱스 `[1]`에서 값 `2`를 가져옵니다.

##### [잘못된 배열 요소 액세스](https://doc.rust-lang.org/book/ch03-02-data-types.html#invalid-array-element-access)

배열의 끝을 지난 배열의 요소에 액세스하려고 하면 어떤 일이 발생하는지 살펴보겠습니다. 사용자로부터 배열 인덱스를 얻기 위해 2장의 추측 게임과 유사한 이 코드를 실행한다고 가정해 보겠습니다.

파일 이름: src/main.rs

```rust
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!(`Please enter an array index.`);
    
    let mut index = String::new();
    
    io::stdin()
        .read_line(&mut index)
        .expect(`Failed to read line`);
    
    let index: usize = index
        .trim()
        .parse()
        .expect(`Index entered was not a number`);
    
    let element = a[index];
    
    println!(`The value of the element at index {index} is: {element}`);
}
```

이 코드는 성공적으로 컴파일됩니다. `cargo run`을 사용하여 이 코드를 실행하고 `0`, `1`, `2`, `3` 또는 `4`를 입력하면 프로그램이 배열의 해당 인덱스에서 해당 값을 인쇄합니다. 대신 `10`과 같이 배열의 끝을 지나 숫자를 입력하면 다음과 같은 출력이 표시됩니다.

```console
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 10', src/main.rs:19:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

프로그램은 인덱싱 작업에서 유효하지 않은 값을 사용하는 시점에서 *런타임 오류가 발생했습니다.* 프로그램이 오류 메시지와 함께 종료되었고 마지막 `println!` 성명. 인덱싱을 사용하여 요소에 액세스하려고 하면 Rust는 지정한 인덱스가 배열 길이보다 작은지 확인합니다. 인덱스가 길이보다 크거나 같으면 Rust는 패닉 상태가 됩니다. 이 검사는 특히 이 경우 런타임 시 수행되어야 합니다. 컴파일러는 사용자가 나중에 코드를 실행할 때 어떤 값을 입력할지 알 수 없기 때문입니다.

이것은 실행 중인 Rust의 메모리 안전 원칙의 예입니다. 많은 저수준 언어에서는 이러한 종류의 검사가 수행되지 않으며 잘못된 인덱스를 제공하면 잘못된 메모리에 액세스할 수 있습니다. Rust는 메모리 액세스를 허용하고 계속하는 대신 즉시 종료하여 이러한 종류의 오류로부터 사용자를 보호합니다. 9장에서는 Rust의 오류 처리에 대해 자세히 설명하고 패닉이 발생하지 않고 유효하지 않은 메모리 액세스를 허용하지 않는 읽기 쉽고 안전한 코드를 작성하는 방법에 대해 설명합니다.
