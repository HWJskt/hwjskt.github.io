+++
title = "Generic Data Types"
weight = 1
template = "book_page_rust.html"

+++

## 요약



## [일반 데이터 유형](https://doc.rust-lang.org/book/ch10-01-syntax.html#generic-data-types)

제네릭을 사용하여 함수 서명 또는 구조체와 같은 항목에 대한 정의를 만든 다음 다양한 구체적인 데이터 유형과 함께 사용할 수 있습니다. 제네릭을 사용하여 함수, 구조체, 열거형 및 메서드를 정의하는 방법을 먼저 살펴보겠습니다. 그런 다음 제네릭이 코드 성능에 미치는 영향에 대해 설명합니다.

### [함수 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-function-definitions)

제네릭을 사용하는 함수를 정의할 때 일반적으로 매개 변수의 데이터 유형과 반환 값을 지정하는 함수 서명에 제네릭을 배치합니다. 이렇게 하면 코드가 더 유연해지고 함수 호출자에게 더 많은 기능을 제공하는 동시에 코드 중복을 방지할 수 있습니다.

우리의 `largest`함수에 대해 계속해서 Listing 10-4는 슬라이스에서 가장 큰 값을 찾는 두 함수를 보여줍니다. 그런 다음 이들을 제네릭을 사용하는 단일 함수로 결합합니다.

파일 이름: src/main.rs

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!(`The largest number is {}`, result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!(`The largest char is {}`, result);
}
```

목록 10-4: 서명의 이름과 유형만 다른 두 함수

이 함수는 슬라이스에서 `largest_i32`가장 큰 것을 찾는 Listing 10-3에서 추출한 것입니다. `i32`이 함수는 슬라이스에서 `largest_char`가장 큰 것을 찾습니다. `char`함수 본문은 동일한 코드를 가지고 있으므로 단일 함수에 제네릭 형식 매개 변수를 도입하여 중복을 제거하겠습니다.

새로운 단일 함수에서 유형을 매개변수화하려면 함수에 대한 값 매개변수와 마찬가지로 유형 매개변수의 이름을 지정해야 합니다. 모든 식별자를 유형 매개변수 이름으로 사용할 수 있습니다. `T`그러나 관례에 따라 Rust의 유형 매개변수 이름은 짧고 종종 문자일 뿐이며 Rust의 유형 이름 지정 규칙은 UpperCamelCase이기 때문에 사용할 것입니다. `유형`의 줄임말은 `T`대부분의 Rust 프로그래머가 기본적으로 선택하는 것입니다.

함수 본문에서 매개변수를 사용할 때 컴파일러가 해당 이름의 의미를 알 수 있도록 서명에 매개변수 이름을 선언해야 합니다. 유사하게 함수 시그니처에서 타입 매개변수 이름을 사용할 때 사용하기 전에 타입 매개변수 이름을 선언해야 합니다. 제네릭 함수를 정의하려면 다음과 같이 함수 이름과 매개변수 목록 사이의 `largest`꺾쇠 괄호 안에 유형 이름 선언을 배치합니다.`<>`

```rust
fn largest<T>(list: &[T]) -> &T {
```

우리는 이 정의를 다음과 같이 읽습니다. 함수는 `largest`일부 유형에 대해 일반적입니다 `T`. 이 함수에는 `list`유형 값의 슬라이스인 이라는 이름의 매개변수가 하나 있습니다 `T`. 이 `largest`함수는 같은 유형의 값에 대한 참조를 반환합니다 `T`.

`largest`Listing 10-5는 서명에 일반 데이터 유형을 사용하는 결합된 함수 정의를 보여줍니다. `i32`이 목록은 값 조각 또는 `char`값으로 함수를 호출하는 방법도 보여줍니다. 이 코드는 아직 컴파일되지 않지만 이 장의 뒷부분에서 수정할 것입니다.

파일 이름: src/main.rs

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!(`The largest number is {}`, result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!(`The largest char is {}`, result);
}
```

목록 10-5: `largest`일반 유형 매개변수를 사용하는 함수; 이것은 아직 컴파일되지 않습니다

지금 이 코드를 컴파일하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error
```

도움말 텍스트는 특성인 을 언급하며 `std::cmp::PartialOrd`다음 *섹션* 에서 특성에 대해 이야기할 것입니다. 지금은 이 오류가 가능한 모든 유형에 대해 본문이 `largest`작동하지 않음을 나타냅니다 `T`. `T`본문에 있는 유형의 값을 비교하고 싶기 때문에 값을 정렬할 수 있는 유형만 사용할 수 있습니다. 비교를 가능하게 하기 위해 표준 라이브러리에는 `std::cmp::PartialOrd`유형에 대해 구현할 수 있는 특성이 있습니다(이 특성에 대한 자세한 내용은 부록 C 참조). 도움말 텍스트의 제안에 따라 유효한 유형을 `T`구현하는 유형으로만 제한 `PartialOrd`하고 이 예제는 컴파일할 것입니다. 표준 라이브러리는 및 `PartialOrd`모두에서 구현하기 때문입니다.`i32``char`

### [구조체 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-struct-definitions)

구문을 사용하여 하나 이상의 필드에서 일반 유형 매개변수를 사용하도록 구조체를 정의할 수도 있습니다 `<>`. 목록 10-6은 모든 유형의 값을 `Point<T>`보유 `x`하고 `y`조정하는 구조체를 정의합니다.

파일 이름: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

목록 10-6: 유형의 값을 보유 `Point<T>`하는 구조체`x``y``T`

구조체 정의에서 제네릭을 사용하는 구문은 함수 정의에서 사용되는 구문과 유사합니다. 먼저 구조체 이름 바로 뒤에 꺾쇠 괄호 안에 유형 매개변수의 이름을 선언합니다. 그런 다음 구체적인 데이터 유형을 지정하는 구조체 정의에서 일반 유형을 사용합니다.

를 정의하기 위해 하나의 제네릭 유형만 사용했기 때문에 `Point<T>`이 정의에서는 `Point<T>`구조체가 일부 유형에 대해 제네릭 `T`하고 필드 `x`및 필드 `y`가 유형에 관계없이 *동일한* 유형임을 나타냅니다. 목록 10-7에서와 같이 다른 유형의 값을 가진 a 인스턴스를 생성하면 `Point<T>`코드가 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

목록 10-7: 필드 `x`및 필드는 `y`둘 다 동일한 일반 데이터 유형을 갖기 때문에 동일한 유형이어야 합니다 `T`.

이 예에서 정수 값 5를 에 할당하면 제네릭 형식이 의 이 인스턴스에 대한 정수가 될 것임을 `x`컴파일러에 알립니다. 그런 다음 와 동일한 유형을 갖도록 정의한 에 대해 4.0을 지정하면 다음과 같은 유형 불일치 오류가 발생합니다.`T``Point<T>``y``x`

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found floating-point number

For more information about this error, try `rustc --explain E0308`.
error: could not compile `chapter10` due to previous error
```

및 가 모두 제네릭이지만 다른 유형을 가질 수 있는 `Point`구조체를 정의하려면 여러 제네릭 유형 매개변수를 사용할 수 있습니다. 예를 들어 Listing 10-8에서 유형에 대한 정의를 일반으로 변경하고 여기서 is of type 및 is of type 입니다.`x``y``Point``T``U``x``T``y``U`

파일 이름: src/main.rs

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

목록 10-8: 서로 다른 유형의 값이 될 수 `Point<T, U>`있도록 두 가지 유형에 대한 제네릭`x``y`

이제 표시된 모든 인스턴스가 `Point`허용됩니다! 정의에서 제네릭 형식 매개 변수를 원하는 만큼 사용할 수 있지만 몇 개 이상 사용하면 코드를 읽기 어려워집니다. 코드에 많은 제네릭 형식이 필요한 경우 코드를 더 작은 조각으로 재구성해야 함을 나타낼 수 있습니다.

### [열거형 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-enum-definitions)

구조체와 마찬가지로 열거형을 정의하여 변형에 일반 데이터 유형을 담을 수 있습니다. `Option<T>`6장에서 사용한 표준 라이브러리가 제공하는 열거형을 다시 살펴보겠습니다.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

이제 이 정의가 더 이해하기 쉬울 것입니다. 보시 `Option<T>`다시피 열거형은 유형에 대해 일반적 `T`이며 두 가지 변형이 있습니다. `Some`하나의 유형 값을 보유하는 `T`과 `None`값을 보유하지 않는 변형입니다. 열거형 을 사용하면 `Option<T>`옵셔널 값의 추상적인 개념을 표현할 수 있고 `Option<T>`제네릭이기 때문에 옵셔널 값의 타입에 관계없이 이 추상을 사용할 수 있습니다.

열거형은 여러 제네릭 형식도 사용할 수 있습니다. `Result`9장에서 사용한 enum 의 정의는 한 가지 예입니다.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

열거 형은 두 가지 유형 및 `Result`에 대해 일반적 이며 두 가지 변형이 있습니다. 유형의 값을 보유하는 과 유형의 값을 보유하는 . 이 정의는 성공(일부 유형의 값 반환 ) 또는 실패(일부 유형의 오류 반환 ) 작업이 있는 모든 곳에서 enum 을 사용하는 것을 편리하게 만듭니다. 사실 이것은 목록 9-3에서 파일을 여는 데 사용한 것입니다. 파일이 성공적으로 열렸을 때 유형으로 채워지고 파일을 여는 데 문제가 있을 때 유형으로 채워졌습니다.`T``E``Ok``T``Err``E``Result``T``E``T``std::fs::File``E``std::io::Error`

보유한 값의 유형만 다른 여러 구조체 또는 열거형 정의가 있는 코드의 상황을 인식하는 경우 대신 제네릭 유형을 사용하여 중복을 방지할 수 있습니다.

### [메서드 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-method-definitions)

우리는 구조체와 열거형에 메서드를 구현할 수 있고(5장에서 했던 것처럼) 정의에 제네릭 유형을 사용할 수도 있습니다. Listing 10-9는 `Point<T>`Listing 10-6에서 정의한 구조체에 `x`구현된 메서드를 보여줍니다.

파일 이름: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!(`p.x = {}`, p.x());
}
```

목록 10-9: 유형 필드 에 대한 참조를 반환하는 구조체 `x`에 명명된 메서드 구현`Point<T>``x``T`

여기에서 필드의 데이터에 대한 참조를 반환하는 `x`on 이라는 메서드를 정의했습니다.`Point<T>``x`

유형에 메서드를 구현하고 있음을 지정하는 데 사용할 수 있도록 `T`바로 뒤에 선언해야 합니다. 뒤에 일반 유형으로 선언함으로써 Rust는 꺾쇠 괄호 안에 있는 유형이 구체적인 유형이 아닌 일반 유형임을 식별할 수 있습니다. 이 일반 매개변수에 대해 구조체 정의에 선언된 일반 매개변수와 다른 이름을 선택할 수 있지만 동일한 이름을 사용하는 것이 관례입니다. 제네릭 형식을 선언하는 에서 작성된 메서드는 어떤 구체적인 형식이 제네릭 형식을 대체하는지에 관계없이 해당 형식의 모든 인스턴스에서 정의됩니다.`impl``T``Point<T>``T``impl``Point``impl`

유형에 대한 메소드를 정의할 때 제네릭 유형에 대한 제약 조건을 지정할 수도 있습니다. 예를 들어 제네릭 유형이 있는 `Point<f32>`인스턴스가 아닌 인스턴스에서만 메서드를 구현할 수 있습니다. `Point<T>`Listing 10-10에서 구체적인 type 을 사용합니다 `f32`. 즉, . 뒤에 어떤 유형도 선언하지 않습니다 `impl`.

파일 이름: src/main.rs

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

Listing 10-10: `impl`일반 유형 매개변수에 대한 특정 구체적인 유형을 가진 구조체에만 적용되는 블록`T`

이 코드는 유형 에 메소드가 `Point<f32>`있음 을 의미합니다 `distance_from_origin`. 유형이 아닌 `Point<T>`다른 인스턴스에는 이 메소드가 정의되지 않습니다. 이 메서드는 포인트가 좌표(0.0, 0.0)의 포인트에서 얼마나 떨어져 있는지 측정하고 부동 소수점 유형에만 사용할 수 있는 수학 연산을 사용합니다.`T``f32`

구조체 정의의 일반 유형 매개변수는 동일한 구조체의 메서드 서명에서 사용하는 매개변수와 항상 동일하지 않습니다. 목록 10-11은 예제를 더 명확하게 하기 위해 일반 유형 `X1`과 `Y1`구조체 `Point`및 `X2` `Y2`메서드 서명을 사용합니다. `mixup`메서드는 ( type ) 의 값 과 전달된 값 ( type ) 을 `Point`사용하여 새 인스턴스를 만듭니다.`x``self` `Point``X1``y``Point``Y2`

파일 이름: src/main.rs

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: `Hello`, y: 'c' };

    let p3 = p1.mixup(p2);

    println!(`p3.x = {}, p3.y = {}`, p3.x, p3.y);
}
```

목록 10-11: 구조체의 정의와 다른 제네릭 유형을 사용하는 메서드

에서 for (값 포함 ) 및 for (값 포함 ) 가 있는 `main`a를 정의했습니다. 변수 는 for (값 포함 ) 및 for (값 포함 ) 문자열 슬라이스가 있는 구조체 입니다. 인수를 사용하여 호출 하면 for 가 있는 가 제공됩니다. 변수 는 for 를 갖게 됩니다. from 에서 왔기 때문입니다. 매크로 호출이 인쇄됩니다.`Point``i32``x``5``f64``y``10.4``p2``Point``x```Hello```char``y``c``mixup``p1``p2``p3``i32``x``x``p1``p3``char``y``y``p2``println!``p3.x = 5, p3.y = c`

```
impl`이 예제의 목적은 일부 제네릭 매개 변수가 선언되고 일부는 메서드 정의로 선언되는 상황을 보여 주는 것입니다. 여기에서 일반 매개변수 `X1`및 는 구조체 정의와 함께 이동하기 때문에 `Y1`뒤에 선언됩니다. `impl`일반 매개변수 `X2`및 는 메서드에만 관련되기 때문에 `Y2`뒤에 선언됩니다.`fn mixup
```

### [제네릭을 사용한 코드 성능](https://doc.rust-lang.org/book/ch10-01-syntax.html#performance-of-code-using-generics)

제네릭 형식 매개 변수를 사용할 때 런타임 비용이 있는지 궁금할 수 있습니다. 좋은 소식은 제네릭 유형을 사용해도 구체적인 유형을 사용할 때보다 프로그램 실행 속도가 느려지지 않는다는 것입니다.

Rust는 컴파일 타임에 제네릭을 사용하여 코드의 단일형화를 수행함으로써 이를 달성합니다. *단일형화는* 컴파일할 때 사용되는 구체적인 유형을 채워 일반 코드를 특정 코드로 바꾸는 프로세스입니다. 이 프로세스에서 컴파일러는 Listing 10-5에서 제네릭 함수를 생성하는 데 사용한 단계와 반대로 수행합니다. 컴파일러는 제네릭 코드가 호출되는 모든 위치를 살펴보고 제네릭 코드가 호출되는 구체적인 유형에 대한 코드를 생성합니다. .

표준 라이브러리의 일반 열거형을 사용하여 이것이 어떻게 작동하는지 살펴보겠습니다 `Option<T>`.

```rust
let integer = Some(5);
let float = Some(5.0);
```

Rust는 이 코드를 컴파일할 때 단일형화를 수행합니다. 이 과정에서 컴파일러는 인스턴스에서 사용된 값을 읽고 `Option<T>`두 가지 종류를 식별합니다 `Option<T>`. 하나는 이고 `i32`다른 하나는 입니다 `f64`. 따라서 의 일반 정의를 및 `Option<T>`에 특화된 두 가지 정의로 확장하여 일반 정의를 특정 정의로 바꿉니다.`i32``f64`

코드의 단일형 버전은 다음과 유사합니다(컴파일러는 설명을 위해 여기에서 사용하는 이름과 다른 이름을 사용함).

파일 이름: src/main.rs

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

제네릭은 `Option<T>`컴파일러에서 만든 특정 정의로 대체됩니다. Rust는 제네릭 코드를 각 인스턴스의 유형을 지정하는 코드로 컴파일하기 때문에 제네릭 사용에 대한 런타임 비용을 지불하지 않습니다. 코드가 실행되면 각 정의를 직접 복사한 것처럼 수행됩니다. 단일형화 프로세스는 Rust의 제네릭을 실행 시간에 매우 효율적으로 만듭니다.
