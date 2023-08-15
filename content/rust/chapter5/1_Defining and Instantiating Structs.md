+++
title = "Defining and Instantiating Structs"
weight = 1
template = "book_page_rust.html"

+++

## 요약

- 구조체 
  - 파이썬의 딕셔너리와 비슷한 형태
  - 우선 구조체를 정의해서 템플릿을 만든다. {} 내부에 field (데이터 이름과 타입)을 적는 방식
  - 정의할 때 구조체가 가변,불변인지 정한다. 데이터 별로 지정할 수는 없다.
  - 사용은 인스턴스를 만들고 내용을 채운다. 내용을 채우는 방식은 딕셔너리와 동일하게 key : value 형태이다. 
- 구조체 업데이트
  - field 에 `..` 를 사용해서 일부만 다른 구조체를 쉽게 만들수 있다. 
  - 원본 구조체는 사용 못하게 되는 경우가 있다.
- 튜플 구조체(tuple structs)
  - 구조체와 비슷한데 key 정의 없이 value 의 type 만 정의한다.
- 단위 유사 구조체(unit-like structs)
  - field 가 없는 구조체 (데이터가 없다)


## [구조체 정의 및 인스턴스화](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#defining-and-instantiating-structs)

구조체는 [튜플 타입](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션 에서 논의한 튜플과 유사하며, 둘 다 여러 관련 값을 보유합니다. 튜플과 마찬가지로, 구조체의 조각은 다른 유형일 수 있습니다. 튜플과 달리 구조체에서는 값이 의미하는 바가 명확하도록 각 데이터 조각의 이름을 지정합니다. 이러한 이름을 추가한다는 것은 구조체가 튜플보다 더 유연하다는 것을 의미합니다. 인스턴스 값을 지정하거나 액세스하기 위해 데이터 순서에 의존할 필요가 없습니다.

구조체를 정의하려면 키워드 `struct`를 입력하고 전체 구조체의 이름을 지정합니다. 구조체의 이름은 함께 그룹화되는 데이터 조각의 중요성을 설명해야 합니다. 그런 다음 {}중괄호 안에 필드(*fields*)라고 하는 데이터 조각의 이름과 유형을 *정의*합니다 . 예를 들어 Listing 5-1은 사용자 계정에 대한 정보를 저장하는 구조체를 보여줍니다.

파일 이름: src/main.rs

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

목록 5-1: `user` 구조체 정의

구조체를 정의한 후 사용하려면 각 필드에 구체적인 값을 지정하여 해당 구조체의 *인스턴스를 만듭니다.* 구조체의 이름을 명시하여 인스턴스를 생성한 다음 *key: value* 쌍을 포함하는 중괄호를 추가합니다. 여기서 키는 필드의 이름이고 값은 해당 필드에 저장하려는 데이터입니다. 구조체에서 선언한 것과 동일한 순서로 필드를 지정할 필요가 없습니다. 즉, 구조체 정의는 유형에 대한 일반적인 템플릿과 같으며 인스턴스는 유형의 값을 생성하기 위해 특정 데이터로 해당 템플릿을 채웁니다. 예를 들어 Listing 5-2와 같이 특정 사용자를 선언할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from(`someusername123`),
        email: String::from(`someone@example.com`),
        sign_in_count: 1,
    };
}
```

목록 5-2: `User` 구조체의 인스턴스 만들기

구조체에서 특정 값을 얻으려면 점 표기법을 사용합니다. 예를 들어 이 사용자의 이메일 주소에 액세스하려면 `user1.email`을 사용합니다. 인스턴스가 변경 가능한 경우 점 표기법을 사용하고 특정 필드에 할당하여 값을 변경할 수 있습니다. Listing 5-3은 변경 가능한 `User` 인스턴스의 `email` 필드 값을 변경하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from(`someusername123`),
        email: String::from(`someone@example.com`),
        sign_in_count: 1,
    };

    user1.email = String::from(`anotheremail@example.com`);
}
```

Listing 5-3: `User` 인스턴스의 `email` 필드 값 변경

전체 인스턴스는 변경 가능해야 합니다. Rust는 특정 필드만 가변으로 표시하는 것을 허용하지 않습니다. 모든 표현식과 마찬가지로 구조체의 새 인스턴스를 함수 본문의 마지막 표현식으로 구성하여 해당 새 인스턴스를 암시적으로 반환할 수 있습니다.

Listing 5-4는 주어진 이메일과 사용자 이름으로 `User` 인스턴스를 반환하는 `build_user` 함수를 보여줍니다. `활성` 필드는 `true` 값을 가져오고 `sign_in_count`는 `1` 값을 가져옵니다.

파일 이름: src/main.rs

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

Listing 5-4: 이메일과 사용자 이름을 받아 `User` 인스턴스를 반환하는 `build_user` 함수

username: username 처럼 구조체 필드와 같은 이름으로 함수 매개변수의 이름을 지정하는 것은 의미가 있지만 `email` 및 `username` 필드 이름과 변수를 반복해야 하는 것은 약간 지루합니다. 구조체에 더 많은 필드가 있는 경우 각 이름을 반복하면 훨씬 더 짜증이 날 것입니다. 다행히도 편리한 속기가 있습니다!

### [필드 초기화 약어 사용](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-the-field-init-shorthand)

목록 5-4에서 매개변수 이름과 구조체 필드 이름이 정확히 동일하기 때문에 *필드 초기화 약식* 구문을 사용하여 `build_user`를 다시 작성할 수 있습니다. Listing 5-5와 같이 하면 정확히 동일하게 작동하지만 `username` 및 `email`을 반복하진 않습니다.

파일 이름: src/main.rs

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

목록 5-5: `username` 및 `email` 매개변수가 구조체 필드와 이름이 같기 때문에 필드 초기화 약어를 사용하는 `build_user` 함수

여기서는 `email`이라는 필드가 있는 `User` 구조체의 새 인스턴스를 만듭니다. `email` 필드의 값을 `build_user` 함수의 `email` 매개변수 값으로 설정하려고 합니다. `email` 필드와 `email` 매개변수의 이름이 같기 때문에 `email: email`이 아닌 `email`만 작성하면 됩니다.

### [구조체 업데이트 구문을 사용하여 다른 인스턴스에서 인스턴스 생성](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax)

구조체에서 다른 인스턴스의 값 대부분을 포함하지만 일부만 변경된 새 인스턴스를 만드는 것이 종종 유용합니다. *구조체 업데이트 구문을* 사용하여 이 작업을 수행할 수 있습니다.

먼저 Listing 5-6에서 업데이트 구문 없이 정기적으로 `user2`에 새 `User` 인스턴스를 생성하는 방법을 보여줍니다. `email`에 새 값을 설정하지만 그 외에는 Listing 5-2에서 생성한 `user1`의 동일한 값을 사용합니다.

파일 이름: src/main.rs

```rust
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from(`another@example.com`),
        sign_in_count: user1.sign_in_count,
    };
}
```

목록 5-6: `user1`의 값 중 하나를 사용하여 새 `User` 인스턴스 만들기

구조체 업데이트 구문을 사용하면 Listing 5-7과 같이 더 적은 코드로 동일한 효과를 얻을 수 있습니다. 구문 `..`은 명시되지 않은 나머지 필드가 `..` 뒤에 지정된 인스턴스의 필드와 동일한 값을 갖도록 지정합니다.

파일 이름: src/main.rs

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from(`another@example.com`),
        ..user1
    };
}
```

목록 5-7: 구조체 업데이트 구문을 사용하여 `User` 인스턴스에 대한 새 `email` 값을 설정하지만 `user1`의 나머지 값을 사용

목록 5-7의 코드는 또한 `이메일`에 대해 다른 값을 갖지만 `user1`의 `username`, `active` 및 `sign_in_count` 필드에 대해 동일한 값을 갖는 `user2`에 인스턴스를 생성합니다. `..user1`은 나머지 필드가 `user1`의 해당 필드에서 값을 가져와야 함을 지정하기 위해 마지막에 와야 하지만,구조체 정의에 있는 필드의 순서에 상관없이 원하는 만큼 많은 필드에 대한 값을 지정하도록 선택할 수 있습니다. 

구조체 업데이트 구문은 할당처럼 `=`를 사용합니다. [이동과 상호 작용하는 변수 및 데이터](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move) 섹션 에서 본 것처럼 데이터를 이동하기 때문입니다. 이 예제에서는 `user1`의 `username` 필드에 있는 `String`이 `user2`로 이동되었기 때문에 `user2`를 생성한 후 더 이상 `user1`을 전체적으로 사용할 수 없습니다. 만약 `user2`에 `email`과 `username` 에 대해 새 `String` 값을 지정하고,  `user1`에서 `active` 및 `sign_in_count` 값만 가져와 사용한 경우, `user1`은  `user2`생성 후에도 여전히 유효합니다. `active` 및 `sign_in_count`는 모두 `Copy` 특성을 구현하는 유형입이기 때문입니다. 

### [명명된 필드 없이 튜플 구조체를 사용하여 다른 유형 만들기](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types)

Rust는 *튜플 구조체(tuple structs)* 라고 하는 튜플과 비슷하게 보이는 구조체를 지원합니다. 튜플 구조체에는 구조체 이름이 제공하는 추가 의미가 있지만 해당 필드와 연결된 이름은 없습니다. 오히려 그들은 단지 필드의 유형을 가지고 있습니다. 튜플 구조체는 전체 튜플에 이름을 지정하고 튜플을 다른 튜플과 다른 유형으로 만들고 싶을 때 그리고 일반 구조체에서와 같이 각 필드의 이름을 지정하는 것이 장황하거나 중복될 때 유용합니다.

튜플 구조체를 정의하려면 `struct` 키워드로 시작하고 구조체 이름 뒤에 튜플의 유형이 옵니다. 예를 들어 여기에서는 `Color`와 `Point`라는 두 개의 튜플 구조체를 정의하고 사용합니다.

파일 이름: src/main.rs

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

`black` 및 `origin` 값은 서로 다른 튜플 구조체의 인스턴스이기 때문에 서로 다른 유형입니다. 정의하는 각 구조체는 구조체 내의 필드가 동일한 유형을 가질 수 있더라도 고유한 유형입니다. 예를 들어 `Color` 유형의 매개변수를 사용하는 함수는 두 유형 모두 세 개의 `i32` 값으로 구성되어 있어도 `Point`을 인수로 사용할 수 없습니다. 그렇지 않으면 튜플 구조체 인스턴스는 개별 조각으로 분해할 수 있고 `.`를 사용할 수 있다는 점에서 튜플과 유사합니다. 개별 값에 액세스하기 위한 인덱스가 뒤따릅니다.

### [필드가 없는 단위 유사 구조체](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#unit-like-structs-without-any-fields)

필드가 없는 구조체를 정의할 수도 있습니다! 이들은 [튜플 타입](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션 에서 언급한 단위 유형인 `()`와 유사하게 동작하기 때문에 *단위 유사 구조체(unit-like structs)* 라고 합니다. 단위와 같은 구조체는 일부 유형에 특성을 구현해야 하지만 유형 자체에 저장하려는 데이터가 없을 때 유용할 수 있습니다. 특성에 대해서는 10장에서 논의할 것입니다. 다음은 `AlwaysEqual`이라는 단위 구조체를 선언하고 인스턴스화하는 예입니다.

파일 이름: src/main.rs

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

`AlwaysEqual`을 정의하려면 `struct` 키워드, 원하는 이름, 세미콜론을 사용합니다. 중괄호나 괄호가 필요하지 않습니다! 그런 다음 유사한 방식으로 `subject` 변수에서 `AlwaysEqual` 인스턴스를 얻을 수 있습니다. 중괄호나 괄호 없이 정의한 이름을 사용합니다. 나중에 `AlwaysEqual`의 모든 인스턴스가 항상 다른 유형의 모든 인스턴스와 동일하도록 이 유형에 대한 동작을 구현하여 테스트 목적으로 알려진 결과를 가질 것이라고 상상해 보십시오. 해당 동작을 구현하는 데 데이터가 필요하지 않습니다! 10장에서 특성을 정의하고 유닛과 같은 구조체를 포함하여 모든 유형에서 구현하는 방법을 볼 수 있습니다.

> ### [구조체 데이터의 소유권](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#ownership-of-struct-data)
>
> Listing 5-1의 `User` 구조체 정의에서 우리는 `&str` 문자열 슬라이스 유형이 아닌 소유된 `String` 유형을 사용했습니다. 이것은 이 구조체의 각 인스턴스가 모든 데이터를 소유하고 전체 구조체가 유효한 한 해당 데이터가 유효하기를 원하기 때문에 의도적인 선택입니다.
>
> 구조체가 다른 것이 소유한 데이터에 대한 참조를 저장하는 것도 가능하지만 그렇게 하려면 10장에서 논의할 Rust 기능인 lifetimes 를 *사용해야* 합니다. 수명은 구조체가 있는 동안 구조체가 참조하는 데이터가 유효하도록 보장합니다. 다음과 같이 수명을 지정하지 않고 구조체에 참조를 저장하려고 한다고 가정해 보겠습니다. 이것은 작동하지 않습니다:
>
> 파일 이름: src/main.rs
>
> ```rust
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
> 
> fn main() {
>     let user1 = User {
>         active: true,
>         username: `someusername123`,
>         email: `someone@example.com`,
>         sign_in_count: 1,
>     };
> }
> ```
>
> 컴파일러는 수명 지정자가 필요하다고 불평합니다.
>
> ```bash
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
> 
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
> 
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` due to 2 previous errors
> ```
>
> 10장에서 구조체에 참조를 저장할 수 있도록 이러한 오류를 수정하는 방법에 대해 논의하지만 지금은 `&str`과 같은 참조 대신 `String`과 같은 소유 유형을 사용하여 이러한 오류를 수정합니다.
