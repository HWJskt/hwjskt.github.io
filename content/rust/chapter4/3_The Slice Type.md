+++
title = "The Slice Type"
weight = 3
template ="book_page_rust.html"

+++

## 요약

## [슬라이스 유형](https://doc.rust-lang.org/book/ch04-03-slices.html#the-slice-type)

*슬라이스를 사용하면* 전체 컬렉션이 아닌 컬렉션의 연속적인 요소 시퀀스를 참조할 수 있습니다. 슬라이스는 일종의 참조이므로 소유권이 없습니다.

여기에 작은 프로그래밍 문제가 있습니다. 공백으로 구분된 단어 문자열을 사용하고 해당 문자열에서 찾은 첫 번째 단어를 반환하는 함수를 작성하세요. 함수가 문자열에서 공백을 찾지 못하면 전체 문자열이 한 단어여야 하므로 전체 문자열이 반환되어야 합니다.

슬라이스가 해결하는 문제를 이해하기 위해 슬라이스를 사용하지 않고 이 함수의 시그니처를 작성하는 방법을 살펴보겠습니다.

```rust
fn first_word(s: &String) -> ?
```

`first_word` 함수는 매개변수로 `&String`을 가집니다. 우리는 소유권을 원하지 않으므로 괜찮습니다. 그러나 우리는 무엇을 반환해야 합니까? 문자열의 *일부* 에 대해 말할 방법이 없습니다. 그러나 공백으로 표시된 단어 끝의 인덱스를 반환할 수 있습니다. Listing 4-7과 같이 시도해 봅시다.

파일 이름: src/main.rs

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

목록 4-7: 바이트 인덱스 값을 `String` 매개변수로 반환하는 `first_word` 함수

요소별로 `문자열` 요소를 살펴보고 값이 공백인지 확인해야 하므로 `as_bytes` 메서드를 사용하여 `문자열`을 바이트 배열로 변환합니다.

```rust
    let bytes = s.as_bytes();
```

다음으로 `iter` 메서드를 사용하여 바이트 배열에 대한 반복자를 만듭니다.

```rust
    for (i, &item) in bytes.iter().enumerate() {
```

[13장](https://doc.rust-lang.org/book/ch13-02-iterators.html) 에서 이터레이터에 대해 더 자세히 논의할 것입니다. 지금은 `iter`가 컬렉션의 각 요소를 반환하는 메서드이고 `enumerate`가 `iter`의 결과를 래핑하고 대신 튜플의 일부로 각 요소를 반환한다는 것을 알아두세요. `enumerate`에서 반환된 튜플의 첫 번째 요소는 인덱스이고 두 번째 요소는 요소에 대한 참조입니다. 지수를 직접 계산하는 것보다 조금 더 편리합니다.

`enumerate` 메서드는 튜플을 반환하기 때문에 패턴을 사용하여 해당 튜플을 분해할 수 있습니다. [6장](https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values) 에서 패턴에 대해 더 논의할 것입니다. `for` 루프에서 튜플의 인덱스에 `i`가 있고 튜플의 단일 바이트에 `&item`이 있는 패턴을 지정합니다. `.iter().enumerate()`에서 요소에 대한 참조를 가져오므로 패턴에서 `&`를 사용합니다.

`for` 루프 내에서 바이트 리터럴 구문을 사용하여 공백을 나타내는 바이트를 검색합니다. 공백을 찾으면 위치를 반환합니다. 그렇지 않으면 `s.len()`을 사용하여 문자열의 길이를 반환합니다.

```rust
        if item == b' ' {
            return i;
        }
    }

    s.len()
```

이제 문자열에서 첫 번째 단어의 끝 인덱스를 찾을 수 있는 방법이 있지만 문제가 있습니다. 자체적으로 `usize`를 반환하지만 `&String` 컨텍스트에서 의미 있는 숫자일 뿐입니다. 즉, `문자열`과는 별개의 값이기 때문에 앞으로도 유효하다는 보장이 없습니다. 목록 4-7의 `first_word` 함수를 사용하는 목록 4-8의 프로그램을 고려하십시오.

파일 이름: src/main.rs

```rust
fn main() {
    let mut s = String::from(`hello world`);

    let word = first_word(&s); // word will get the value 5

    s.clear(); // this empties the String, making it equal to ``

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with. word is now totally invalid!
}
```

목록 4-8: `first_word` 함수 호출 결과 저장 및 `문자열` 내용 변경

이 프로그램은 오류 없이 컴파일되며 `s.clear()`를 호출한 후 `word`를 사용한 경우에도 오류가 발생합니다. `word`는 `s`의 상태와 전혀 연결되어 있지 않기 때문에 `word`는 여전히 값 `5`를 포함합니다. 변수 `s`와 함께 값 `5`를 사용하여 첫 번째 단어를 추출하려고 시도할 수 있지만 `word`에 `5`를 저장한 이후로 `s`의 내용이 변경되었기 때문에 이것은 버그가 됩니다.

`word`의 인덱스가 `s`의 데이터와 동기화되지 않는 것에 대해 걱정하는 것은 지루하고 오류가 발생하기 쉽습니다! `second_word` 함수를 작성하면 이러한 인덱스를 관리하기가 훨씬 더 어려워집니다. 서명은 다음과 같아야 합니다.

```rust
fn second_word(s: &String) -> (usize, usize) {
```

이제 우리는 시작 *및* 종료 인덱스를 추적하고 있으며 특정 상태의 데이터에서 계산되었지만 해당 상태에 전혀 연결되지 않은 더 많은 값을 가지고 있습니다. 동기화를 유지해야 하는 세 개의 관련 없는 변수가 떠다니고 있습니다.

운 좋게도 Rust는 이 문제에 대한 해결책을 가지고 있습니다: 스트링 슬라이스입니다.

### [스트링 슬라이스](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices)

문자열 *조각은* `문자열`의 일부에 대한 참조이며 다음과 같습니다.

```rust
    let s = String::from(`hello world`);

    let hello = &s[0..5];
    let world = &s[6..11];
```

전체 `문자열`에 대한 참조가 아니라 `hello`는 추가 `[0..5]` 비트에 지정된 `문자열` 부분에 대한 참조입니다. `[starting_index..ending_index]`를 지정하여 괄호 안의 범위를 사용하여 슬라이스를 만듭니다. 여기서 `starting_index`는 슬라이스의 첫 번째 위치이고 `ending_index`는 슬라이스의 마지막 위치보다 하나 더 많습니다. 내부적으로 슬라이스 데이터 구조는 `ending_index`에서 `starting_index`를 뺀 값에 해당하는 슬라이스의 시작 위치와 길이를 저장합니다. 따라서 `let world = &s[6..11];`의 경우 `world`는 길이 값이 `5`인 `s`의 인덱스 6에 있는 바이트에 대한 포인터를 포함하는 슬라이스입니다.

그림 4-6은 이를 다이어그램으로 보여줍니다.

![3개의 테이블: 힙의 문자열 데이터 `hello world` 테이블에서 인덱스 0의 바이트를 가리키는 s의 스택 데이터를 나타내는 테이블.  세 번째 테이블은 길이 값이 5이고 힙 데이터 테이블의 바이트 6을 가리키는 슬라이스 세계의 스택 데이터를 나타냅니다.](https://doc.rust-lang.org/book/img/trpl04-06.svg)

그림 4-6: `문자열`의 일부를 참조하는 문자열 슬라이스

Rust의 `..` 범위 구문을 사용하여 인덱스 0에서 시작하려면 두 마침표 앞에 값을 놓을 수 있습니다. 즉, 다음과 같습니다.

```rust
let s = String::from(`hello`);

let slice = &s[0..2];
let slice = &s[..2];
```

마찬가지로 슬라이스에 `문자열`의 마지막 바이트가 포함되어 있으면 후행 숫자를 삭제할 수 있습니다. 즉, 다음과 같이 동일합니다.

```rust
let s = String::from(`hello`);

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

두 값을 모두 삭제하여 전체 문자열의 조각을 가져올 수도 있습니다. 따라서 이들은 동일합니다.

```rust
let s = String::from(`hello`);

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 참고: 문자열 슬라이스 범위 인덱스는 유효한 UTF-8 문자 경계에서 발생해야 합니다. 멀티바이트 문자 중간에 문자열 조각을 만들려고 하면 프로그램이 오류와 함께 종료됩니다. 문자열 조각을 소개하기 위해 이 섹션에서는 ASCII만 가정합니다. UTF-8 처리에 대한 자세한 내용은 8장의 [`UTF-8 인코딩된 텍스트를 문자열로 저장` 섹션에 있습니다.](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings)

이 모든 정보를 염두에 두고 슬라이스를 반환하도록 `first_word`를 다시 작성해 보겠습니다. `문자열 조각`을 나타내는 유형은 `&str`로 작성됩니다.

파일 이름: src/main.rs

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

우리는 Listing 4-7에서 했던 것과 같은 방법으로 단어의 끝에 대한 색인을 얻습니다. 공백의 첫 번째 항목을 찾는 것입니다. 공백을 찾으면 문자열의 시작과 공백의 인덱스를 시작 및 끝 인덱스로 사용하여 문자열 슬라이스를 반환합니다.

이제 `first_word`를 호출하면 기본 데이터에 연결된 단일 값을 반환합니다. 값은 슬라이스의 시작점에 대한 참조와 슬라이스의 요소 수로 구성됩니다.

슬라이스를 반환하는 것은 `second_word` 함수에서도 작동합니다.

```rust
fn second_word(s: &String) -> &str {
```

이제 컴파일러가 `문자열`에 대한 참조가 유효한지 확인하기 때문에 엉망으로 만들기 훨씬 더 어려운 간단한 API가 있습니다. Listing 4-8에 있는 프로그램의 버그를 기억하십니까? 첫 번째 단어의 끝에 인덱스를 얻었지만 문자열을 지워서 인덱스가 유효하지 않게 되었을 때의 버그를 기억하십니까? 해당 코드는 논리적으로 잘못되었지만 즉각적인 오류는 표시되지 않았습니다. 빈 문자열과 함께 첫 번째 단어 색인을 계속 사용하려고 하면 나중에 문제가 나타납니다. 슬라이스는 이 버그를 불가능하게 만들고 코드에 문제가 있음을 훨씬 빨리 알려줍니다. `first_word`의 슬라이스 버전을 사용하면 컴파일 타임 오류가 발생합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut s = String::from(`hello world`);

    let word = first_word(&s);

    s.clear(); // error!

    println!(`the first word is: {}`, word);
}
```

다음은 컴파일러 오류입니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!(`the first word is: {}`, word);
   |                                       ---- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error
```

무언가에 대한 불변 참조가 있으면 가변 참조도 사용할 수 없다는 차용 규칙을 상기하십시오. `clear`는 `String`을 잘라야 하기 때문에 변경 가능한 참조를 가져와야 합니다. `println!` `clear`에 대한 호출이 `word`의 참조를 사용한 후에는 변경 불가능한 참조가 해당 시점에서 여전히 활성 상태여야 합니다. Rust는 `clear`의 변경 가능한 참조와 `word`의 변경 불가능한 참조가 동시에 존재하는 것을 허용하지 않으며 컴파일이 실패합니다. Rust는 API를 사용하기 쉽게 만들었을 뿐만 아니라 컴파일 시간에 전체 오류 클래스를 제거했습니다!

#### [슬라이스로서의 문자열 리터럴](https://doc.rust-lang.org/book/ch04-03-slices.html#string-literals-as-slices)

바이너리 내부에 저장되는 문자열 리터럴에 대해 이야기했던 것을 기억하십시오. 이제 슬라이스에 대해 알았으므로 문자열 리터럴을 제대로 이해할 수 있습니다.

```rust
let s = `Hello, world!`;
```

여기서 `s`의 유형은 `&str`입니다. 바이너리의 특정 지점을 가리키는 슬라이스입니다. 이것이 문자열 리터럴이 불변인 이유이기도 합니다. `&str`은 변경할 수 없는 참조입니다.

#### [문자열 조각을 매개변수로](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices-as-parameters)

리터럴과 `문자열` 값의 조각을 사용할 수 있다는 사실을 알면 `first_word`에서 한 가지 더 개선할 수 있으며 이것이 시그니처입니다.

```rust
fn first_word(s: &String) -> &str {
```

더 경험이 많은 Rustacean은 `&String` 값과 `&str` 값 모두에 대해 동일한 함수를 사용할 수 있기 때문에 목록 4-9에 표시된 서명을 대신 작성할 것입니다.

```rust
fn first_word(s: &str) -> &str {
```

Listing 4-9: `s` 매개변수 유형에 문자열 슬라이스를 사용하여 `first_word` 함수 개선

문자열 슬라이스가 있으면 직접 전달할 수 있습니다. `문자열`이 있는 경우 `문자열` 조각이나 `문자열`에 대한 참조를 전달할 수 있습니다. [이러한 유연성은 15장의 `암시적 역참조 강제 변환`](https://doc.rust-lang.org/book/ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods) 섹션 에서 다룰 기능인 *역참조* 강제를 활용합니다.

`문자열`에 대한 참조 대신 문자열 슬라이스를 사용하도록 함수를 정의하면 기능 손실 없이 API가 더 일반적이고 유용해집니다.

파일 이름: src/main.rs

```rust
fn main() {
    let my_string = String::from(`hello world`);

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which are equivalent
    // to whole slices of `String`s
    let word = first_word(&my_string);

    let my_string_literal = `hello world`;

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### [다른 조각](https://doc.rust-lang.org/book/ch04-03-slices.html#other-slices)

상상할 수 있듯이 스트링 슬라이스는 스트링에 따라 다릅니다. 그러나 보다 일반적인 슬라이스 유형도 있습니다. 다음 배열을 고려하십시오.

```rust
let a = [1, 2, 3, 4, 5];
```

문자열의 일부를 참조하려는 것처럼 배열의 일부를 참조하고 싶을 수도 있습니다. 우리는 이렇게 할 것입니다:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

이 슬라이스의 유형은 `&[i32]`입니다. 첫 번째 요소에 대한 참조와 길이를 저장하여 스트링 슬라이스와 동일한 방식으로 작동합니다. 모든 종류의 다른 컬렉션에 대해 이러한 종류의 슬라이스를 사용하게 됩니다. 8장에서 벡터에 대해 이야기할 때 이러한 모음에 대해 자세히 논의할 것입니다.

## [요약](https://doc.rust-lang.org/book/ch04-03-slices.html#summary)

소유권, 차용 및 슬라이스의 개념은 컴파일 타임에 Rust 프로그램의 메모리 안전을 보장합니다. Rust 언어는 다른 시스템 프로그래밍 언어와 같은 방식으로 메모리 사용을 제어할 수 있지만, 데이터 소유자가 범위를 벗어날 때 데이터 소유자가 해당 데이터를 자동으로 정리하면 추가 코드를 작성하고 디버깅할 필요가 없습니다. 이 컨트롤을 얻으려면.

소유권은 Rust의 다른 많은 부분이 작동하는 방식에 영향을 미치므로 책의 나머지 부분에서 이러한 개념에 대해 더 자세히 이야기할 것입니다. 5장으로 이동하여 `구조체`에서 데이터 조각을 함께 그룹화하는 방법을 살펴보겠습니다.
