+++
title = "Defining an Enum"
weight = 1
template = "book_page_rust.html"

+++

## 요약

- 열거형은 가능한 값들(변형, variant)을 열거해두고 그 중 하나만 선택하는 것
- 구조체 대신 열거형을 사용하면 좋은점
  - 데이터를 각 열거형 변형에 직접 넣으면 더 간결
  - 각 변형에는 연결된 데이터의 타입과 양이 다를 수 있다. (구조체는 필드(key)가 같으면 타입도 같아야 한다.)
  - 열거형 변형 내에 모든 종류의 데이터(예: 문자열, 숫자, 구조체, 다른 열거형 등 )를 넣을 수 있음
- 구조체처럼 `impl`을 사용하여  대한 메서드를 정의할 수 있다.
- Option 열거형
  - None 과 Some 변형을 가지고 있다.
  - 변형을 사용할 때, Option::Some 아니고, Some 으로 사용가능
  - match 식을 함께 사용해서 각 변형에 따른 코드를 실행가능

## [열거형 정의](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#defining-an-enum)

구조체(structs)는 관련 필드와 데이터를 함께 그룹화하는 방법을 제공합니다. `width`와 `height`가 있는 `Rectangle`이 예입니다. 열거형은 값이 가능한 값 집합 중 하나라고 말하는 방법을 제공합니다. 예를 들어, `직사각형`은 `원`과 `삼각형`을 포함하는 가능한 도형 세트 중 하나라고 말하고 싶을 수 있습니다. 이를 위해 Rust는 이러한 가능성을 열거형으로 인코딩할 수 있도록 합니다.

코드로 표현하고 싶은 상황을 살펴보고 이 경우 열거형이 구조체보다 유용하고 더 적합한 이유를 살펴보겠습니다. IP 주소로 작업해야 한다고 가정해 보겠습니다. 현재 IP 주소에는 버전 4와 버전 6의 두 가지 주요 표준이 사용됩니다. 이것이 우리 프로그램이 접하게 될 IP 주소에 대한 유일한 가능성이기 때문에 가능한 모든 변형을 *열거* 할 수 있습니다.

모든 IP 주소는 버전 4 또는 버전 6 주소일 수 있지만 동시에 둘 다일 수는 없습니다. IP 주소의 이러한 속성 때문에 enum 데이터 구조가 적절합니다. enum 값은 변형들 중 하나일 수 밖에 없기 때문입니다.  버전 4 및 버전 6 주소는 여전히 기본적으로 IP 주소이므로, 코드가 모든 종류의 IP 주소에 적용되는 상황을 처리할 때 동일한 유형으로 취급되어야 합니다.

`IpAddrKind` 열거형을 정의하고 IP 주소가 될 수 있는 `V4` 및 `V6` 종류를 나열하여 이 개념을 코드로 표현할 수 있습니다. 열거형의 변형은 다음과 같습니다.

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

`IpAddrKind`는 이제 코드의 다른 곳에서 사용할 수 있는 사용자 정의 데이터 유형입니다.

### [열거형 값(Enum values)](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values)

다음과 같이 `IpAddrKind`의 두 변형 각각의 인스턴스를 만들 수 있습니다.

```rust
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

열거형의 변형은 해당 식별자 아래에 네임스페이스가 지정되며 이중 콜론을 사용하여 둘을 구분합니다. 이제 `IpAddrKind::V4` 및 `IpAddrKind::V6` 값이 모두 `IpAddrKind`타입이기 때문에 유용합니다. 예를 들어, 이후 `IpAddrKind`를 사용하는 함수를 정의할 수 있습니다.

```rust
fn route(ip_kind: IpAddrKind) {}
```

그리고 우리는 이 함수를 두 변형으로 호출할 수 있습니다:

```rust
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
```

열거형을 사용하면 더 많은 이점이 있습니다. *IP 주소 유형에 대해 더 생각해 보면 현재 실제 IP 주소 데이터를* 저장할 방법이 없습니다. 우리는 그것이 어떤 *종류* 인지 만 알고 있습니다. 5장에서 구조체에 대해 방금 배웠다면 목록 6-1에 표시된 대로 구조체를 사용하여 이 문제를 해결하고 싶은 유혹을 느낄 수 있습니다.

```rust
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from(`127.0.0.1`),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from(`::1`),
    };
```

목록 6-1: `struct`를 사용하여 데이터 및 IP 주소의 `IpAddrKind` 변형 저장

여기에서 `IpAddrKind` 유형의 `kind` 필드(이전에 정의한 enum)와 `String` 유형의 `address` 필드가 있는 구조체 `IpAddr`를 정의했습니다. 이 구조체에는 두 개의 인스턴스가 있습니다. 첫 번째는 `home`이며 `127.0.0.1`의 관련 주소 데이터와 함께 `kind`로 `IpAddrKind::V4` 값을 가집니다. 두 번째 인스턴스는 `loopback`입니다. `kind` 값인 `V6`으로 `IpAddrKind`의 다른 변형이 있으며 이와 연결된 주소 `::1`이 있습니다. 구조체를 사용하여 `kind` 및 `address` 값을 함께 묶었으므로 이제 변형이 값과 연결됩니다.

그러나 열거형만 사용하여 동일한 개념을 나타내는 것이 더 간결합니다. 구조체 내부에 열거형을 사용하는 것보다, 데이터를 각 열거형 변형에 직접 넣으면 더 간결합니다. 아래 코드에서 `IpAddr` 열거형의 이 새로운 정의는 `V4` 및 `V6` 변형 모두 연결된 `문자열` 값을 가질 것이라고 말합니다.

```rust
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from(`127.0.0.1`));

    let loopback = IpAddr::V6(String::from(`::1`));
```

열거형의 각 변형에 직접 데이터를 첨부하므로 추가 구조체가 필요하지 않습니다. 여기에서 열거형의 작동 방식에 대한 또 다른 세부 정보를 더 쉽게 볼 수 있습니다. 우리가 정의하는 각 열거형 변형의 이름도 열거형의 인스턴스를 구성하는 함수가 됩니다. 즉, `IpAddr::V4()`는 `String` 인수를 사용하고 `IpAddr` 유형의 인스턴스를 반환하는 함수 호출입니다. 열거형을 정의한 결과로 정의된 이 생성자 함수를 자동으로 얻습니다.

구조체 대신 열거형을 사용하면 또 다른 이점이 있습니다. 각 변형에는 연결된 데이터의 유형과 양이 다를 수 있습니다. 버전 4 IP 주소에는 항상 0에서 255 사이의 값을 갖는 4개의 숫자 구성 요소가 있습니다. `V4` 주소를 4개의 `u8` 값으로 저장하고 싶지만 `V6` 주소를 하나의 `String` 값으로 표현하려면 구조체로는 할 수 없습니다. 열거형은 이 경우를 쉽게 처리합니다.

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from(`::1`));
```

버전 4 및 버전 6 IP 주소를 저장하기 위해 데이터 구조를 정의하는 여러 가지 방법을 보여주었습니다. 그러나 밝혀진 바와 같이 IP 주소를 저장하고 어떤 종류인지 인코딩하려는 것은 너무 일반적이어서 [표준 라이브러리에 우리가 사용할 수 있는 정의가 있습니다! ](https://doc.rust-lang.org/std/net/enum.IpAddr.html)표준 라이브러리가 `IpAddr`을 정의하는 방법을 살펴보겠습니다. 여기에는 우리가 정의하고 사용한 정확한 열거형 및 변형이 있지만 각각에 대해 다르게 정의되는 두 개의 서로 다른 구조체의 형태로 변형 내부에 주소 데이터를 포함합니다. 변종:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

이 코드는 열거형 변형 내에 모든 종류의 데이터(예를 들어 문자열, 숫자 유형 또는 구조체와 같은 )를 넣을 수 있음을 보여줍니다. 다른 열거형을 포함할 수도 있습니다! 또한 표준 라이브러리 유형은 종종 여러분이 생각하는 것보다 훨씬 더 복잡하지 않습니다.

표준 라이브러리에 `IpAddr`에 대한 정의가 포함되어 있지만 표준 라이브러리의 정의를 범위로 가져오지 않았기 때문에 여전히 충돌 없이 자체 정의를 만들고 사용할 수 있습니다. 유형을 범위로 가져오는 방법에 대해서는 7장에서 자세히 설명합니다.

Listing 6-2에 있는 enum의 또 다른 예를 살펴보겠습니다. 이것은 변형에 다양한 유형이 내장되어 있습니다.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

목록 6-2: 각각 다른 양과 유형의 값을 저장하는 변형이 있는 `메시지` 열거형

이 열거형에는 유형이 다른 네 가지 변형이 있습니다.

- `Quit`에는 이와 관련된 데이터가 전혀 없습니다.
- `Move`에는 구조체와 마찬가지로 명명된 필드가 있습니다.
- `Write`에는 단일 `문자열`이 포함됩니다.
- `ChangeColor`에는 3개의 `i32` 값이 포함됩니다.

목록 6-2에 있는 것과 같은 변형으로 열거형을 정의하는 것은 열거형이 `struct` 키워드를 사용하지 않고 모든 변형이 `Message` 유형 아래에 함께 그룹화된다는 점을 제외하면 다른 종류의 구조체 정의를 정의하는 것과 유사합니다. 다음 구조체는 이전 열거형 변형이 보유하는 것과 동일한 데이터를 보유할 수 있습니다.

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

그러나 각각 고유한 유형이 있는 서로 다른 구조체를 사용하는 경우 Listing 6-2에 정의된 `Message` 열거형으로 할 수 있는 것처럼 이러한 종류의 메시지를 받는 함수를 쉽게 정의할 수 없습니다. 단일 유형입니다.

열거형과 구조체 사이에는 또 다른 유사점이 있습니다. `impl`을 사용하여 구조체에 대한 메서드를 정의할 수 있는 것처럼 열거형에 대한 메서드도 정의할 수 있습니다. 다음은 `Message` 열거형에서 정의할 수 있는 `call`이라는 메서드입니다.

```rust
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from(`hello`));
    m.call();
```

메서드 본문은 `self`를 사용하여 메서드를 호출한 값을 가져옵니다. 이 예에서 우리는 `Message::Write(String::from("hello"))` 값을 갖는 변수 `m`을 생성했으며 이것이 `m.call()`이 실행될 때 호출 메서드의 본문에 있는 `self`입니다.

매우 일반적이고 유용한 표준 라이브러리의 또 다른 열거형인 `Option`을 살펴보겠습니다.

### [Option 열거형과 Null 값에 대한 이점](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#the-option-enum-and-its-advantages-over-null-values)

이 섹션에서는 표준 라이브러리에서 정의한 또 다른 열거형인 `Option`의 사례 연구를 살펴봅니다. `Option` 유형은 값이 무언가가 될 수도 있고 아무것도 아닐 수도 있는 매우 일반적인 시나리오를 인코딩합니다.

예를 들어 비어 있지 않은 목록의 첫 번째 항목을 요청하면 값을 받게 됩니다. 빈 목록의 첫 번째 항목을 요청하면 아무것도 얻지 못합니다. 유형 시스템 측면에서 이 개념을 표현하면 처리해야 하는 모든 사례를 처리했는지 여부를 컴파일러에서 확인할 수 있습니다. 이 기능은 다른 프로그래밍 언어에서 매우 일반적인 버그를 방지할 수 있습니다.

프로그래밍 언어 설계는 종종 어떤 기능을 포함하는지에 따라 생각되지만 제외하는 기능도 중요합니다. Rust에는 다른 많은 언어에 있는 null 기능이 없습니다. *Null* 은 값이 없다는 의미의 값입니다. null이 있는 언어에서 변수는 항상 null 또는 null이 아닌 두 가지 상태 중 하나일 수 있습니다.

2009년 프레젠테이션 `Null References: The Billion Dollar Mistake`에서 null의 발명가인 Tony Hoare는 다음과 같이 말했습니다.

> 나는 그것을 나의 10억 달러짜리 실수라고 부른다. 그 당시 저는 객체 지향 언어의 참조를 위한 최초의 포괄적인 유형 시스템을 설계하고 있었습니다. 내 목표는 컴파일러가 자동으로 검사를 수행하여 모든 참조 사용이 절대적으로 안전하도록 하는 것이었습니다. 하지만 구현하기가 너무 쉽기 때문에 null 참조를 넣고 싶은 유혹을 뿌리칠 수 없었습니다. 이로 인해 수많은 오류, 취약성 및 시스템 충돌이 발생했으며 지난 40년 동안 아마도 10억 달러의 고통과 피해를 입혔을 것입니다.

null 값의 문제는 null 값을 null이 아닌 값으로 사용하려고 하면 일종의 오류가 발생한다는 것입니다. 이 null 또는 null이 아닌 속성은 널리 퍼져 있기 때문에 이러한 종류의 오류를 만들기가 매우 쉽습니다.

그러나 null이 표현하려는 개념은 여전히 유용합니다. null은 현재 유효하지 않거나 어떤 이유로 없는 값입니다.

문제는 실제로 개념이 아니라 특정 구현에 있습니다. 따라서 Rust에는 null이 없지만 존재하거나 존재하지 않는 값의 개념을 인코딩할 수 있는 열거형이 있습니다. 이 열거형은 `Option<T>`이며 [표준 라이브러리](https://doc.rust-lang.org/std/option/enum.Option.html) 에 의해 다음과 같이 정의됩니다.

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` 열거형은 너무 유용해서 서문에 포함되어 있습니다. 명시적으로 범위로 가져올 필요가 없습니다. 해당 변형도 서문에 포함되어 있습니다. 접두어 `Option::` 없이 `Some` 및 `None`을 직접 사용할 수 있습니다. `Option<T>` 열거형은 여전히 일반 열거형이며 `Some(T)` 및 `None`은 여전히 `Option<T>` 유형의 변형입니다.

`<T>` 구문은 우리가 아직 이야기하지 않은 Rust의 기능입니다. 제네릭 유형 매개변수이며, 제네릭에 대해서는 10장에서 자세히 다룰 것입니다. 지금은 `Option` 열거형의 `Some` 변형이 어떤 타입의 데이터 한 조각을 보유할 수 있으며 `<T>` 대신 사용되는 각 구체적인 유형이 전체 `Option<T>`을 만든다는 것을 의미합니다. 다음은 `Option` 값을 사용하여 숫자 유형과 문자열 유형을 보유하는 몇 가지 예입니다.

```rust
    let some_number = Some(5);
    let some_char = Some('e');

    let absent_number: Option<i32> = None;
```

`some_number`의 타입은 `Option<i32>`입니다. 
`some_char`의 타입은 `Option<char>`, 이는 다른 타입입니다. `Some` 변형 내부에 값을 지정했기 때문에 Rust는 이러한 유형을 추론할 수 있습니다. 
`absent_number`의 경우 Rust는 전체 `Option` 유형에 주석을 달도록 요구합니다. 컴파일러는 `None` 값만 보고 해당 `Some` 변형의 타입을 유추할 수 없습니다. 여기에서, 우리는 `absent_number`가 `Option<i32>` 유형임을 의미한다고 Rust에 알립니다.

`Some` 값이 있을 때 값이 존재하고 그 값이 `Some` 내에 있음을 알 수 있습니다. `None` 값이 있는 경우 어떤 의미에서는 null과 같은 의미입니다. 즉, 유효한 값이 없습니다. 그렇다면 왜 `Option<T>` 가 null 보다 낫습니까?

간단히 말하면, `Option<T>`와 `T`(여기서 `T`는 모든 유형일 수 있음)가 서로 다른 유형입니다. 그래서 컴파일러는 확실히 유효한 값인 것처럼 `Option<T>`을 사용하도록 허용하지 않습니다. 예를 들어 이 코드는 `i8`을 `Option`에 추가하려고 하기 때문에 컴파일되지 않습니다.

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

이 코드를 실행하면 다음과 같은 오류 메시지가 나타납니다.

```bash
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  = help: the following other types implement trait `Add<Rhs>`:
            <&'a i8 as Add<i8>>
            <&i8 as Add<&i8>>
            <i8 as Add<&i8>>
            <i8 as Add>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` due to previous error
```

극심한! 사실상 이 오류 메시지는 Rust가 `i8`과 `Option<i8>`을 더하는 방법을 이해하지 못한다는 것을 의미합니다.서로 다른 유형이기 때문입니다. Rust에서 `i8`과 같은 유형의 값이 있을 때, 컴파일러는 항상 유효한 값을 가지고 있는지 확인합니다. 해당 값을 사용하기 전에 null을 확인하지 않고도 자신 있게 진행할 수 있습니다.  `Option<i8>`이 있는 경우에만(또는 우리가 작업 중인 어떤 유형의 값이든) 값이 없을 가능성에 대해 걱정해야 하며 컴파일러는 값을 사용하기 전에 해당 사례를 처리하는지 확인합니다.

즉, 작업을 수행하기 전에 `Option<T>`을 `T` 로 변환해야 합니다.  일반적으로 이것은 null과 관련된 가장 일반적인 문제 중 하나를 파악하는 데 도움이 됩니다. 실제로 null이 아닐 때 null이 아니라고 가정합니다.

null이 아닌 값을 잘못 가정하는 위험을 제거하면 코드에 대한 확신을 가질 수 있습니다. null일 수 있는 값을 가지려면 해당 값의 유형을 `Option<T>`으로 만들어 명시적으로 옵트인해야 합니다. 그런 다음 해당 값을 사용할 때 값이 null인 경우를 명시적으로 처리해야 합니다. 값의 타입이 `Option<T>`가 아니라면, 값이 null이 아니라고 안전하게 가정할 수 있습니다. 이는 Rust가 null의 보급을 제한하고 Rust 코드의 안전성을 높이기 위한 의도적인 설계 결정이었습니다.

그러면 `Option<T>` 타입의 값이 있을 때 `Some` 변형에서 `T` 값을 어떻게 얻어서 사용할까요?   `Option<T>` enum에는 다양한 상황에서 유용한 많은 메서드가 있습니다. [설명서](https://doc.rust-lang.org/std/option/enum.Option.html) 에서 확인할 수 있습니다. `Option<T>`의 메서드에 익숙해지면 Rust와의 여정에 매우 유용할 것입니다.


일반적으로 `Option<T>` 값을 사용하려면 각 변형을 처리할 코드가 있어야 합니다. `Some(T)` 값이 있을 때만 실행되는 코드가 필요하고, 이 코드는 내부` T`를 사용할 수 있습니다. `None` 값이 있는 경우에만 실행되는 코드를 원할 수 있습니다만, 이때는 T 값을 사용할 수 없습니다. `match `식은 열거형과 함께 사용할 때 바로 이 작업을 수행하는 제어 흐름 구성입니다. 포함된 열거형의 변형에 따라 다른 코드를 실행하고, 해당 코드는 일치하는 값 안에 있는 데이터를 사용할 수 있습니다.

