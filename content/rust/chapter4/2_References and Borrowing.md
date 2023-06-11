+++
title = "References and Borrowing"
weight = 2
template ="book_page_rust.html"

+++



## 요약

## [참조 및 차용](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing)

Listing 4-5에 있는 튜플 코드의 문제는 호출 함수에 `String`을 반환해야 `calculate_length`를 호출한 후에도 `String`을 계속 사용할 수 있다는 것입니다. `계산_길이`. 대신 `문자열` 값에 대한 참조를 제공할 수 있습니다. 참조 *는* 해당 주소에 저장된 데이터에 액세스하기 위해 따를 수 있는 주소라는 점에서 포인터와 같습니다. 해당 데이터는 다른 변수가 소유합니다. 포인터와 달리 참조는 해당 참조의 수명 동안 특정 유형의 유효한 값을 가리키도록 보장됩니다.

다음은 값의 소유권을 가져오는 대신 개체에 대한 참조를 매개 변수로 포함하는 `calculate_length` 함수를 정의하고 사용하는 방법입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let s1 = String::from(`hello`);

    let len = calculate_length(&s1);

    println!(`The length of '{}' is {}.`, s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

먼저 변수 선언의 모든 튜플 코드와 함수 반환 값이 사라진 것을 확인하십시오. 둘째, `&s1`을 `calculate_length`에 전달하고 정의에서 `String`이 아닌 `&String`을 사용합니다. 이러한 앰퍼샌드는 *참조를* 나타내며 소유권을 갖지 않고 일부 값을 참조할 수 있습니다. 그림 4-5는 이 개념을 보여줍니다.

![세 개의 테이블: s에 대한 테이블에는 s1에 대한 테이블에 대한 포인터만 포함됩니다.  s1에 대한 테이블은 s1에 대한 스택 데이터를 포함하고 힙의 문자열 데이터를 가리킵니다.](https://doc.rust-lang.org/book/img/trpl04-05.svg)

그림 4-5: `String s1`을 가리키는 `&String s` 다이어그램

> 참고: `&`를 사용한 참조의 반대는 역 *참조* 연산자 `*`를 사용하여 수행되는 역참조입니다. 8장에서 역참조 연산자의 일부 사용법을 살펴보고 15장에서 역참조에 대한 자세한 내용을 논의합니다.

여기서 함수 호출을 자세히 살펴보겠습니다.

```rust
    let s1 = String::from(`hello`);

    let len = calculate_length(&s1);
```

`&s1` 구문을 사용하면 `s1` 값을 참조하지만 소유하지는 않는 *참조* 를 만들 수 있습니다. 소유하지 않기 때문에 참조가 사용을 중지해도 가리키는 값은 삭제되지 않습니다.

마찬가지로 함수의 시그니처는 `&`를 사용하여 매개변수 `s`의 유형이 참조임을 나타냅니다. 몇 가지 설명 주석을 추가해 보겠습니다.

```rust
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

변수 `s`가 유효한 범위는 모든 함수 매개 변수의 범위와 동일하지만 `s`는 소유권이 없기 때문에 `s`가 사용 중지될 때 참조가 가리키는 값은 삭제되지 않습니다. 함수에 실제 값 대신 매개 변수로 참조가 있으면 소유권을 가져본 적이 없기 때문에 소유권을 돌려주기 위해 값을 반환할 필요가 없습니다.

*참조 차용* 을 만드는 작업을 호출합니다. 실생활에서와 마찬가지로 사람이 무언가를 소유하고 있으면 그 사람에게서 빌릴 수 있습니다. 끝나면 돌려줘야 합니다. 당신은 그것을 소유하지 않습니다.

그렇다면 빌린 것을 수정하려고 하면 어떻게 될까요? 목록 4-6의 코드를 사용해 보십시오. 스포일러 경고: 작동하지 않습니다!

파일 이름: src/main.rs

```rust
fn main() {
    let s = String::from(`hello`);

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(`, world`);
}
```

목록 4-6: 빌린 값 수정 시도

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
8 |     some_string.push_str(`, world`);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` due to previous error
```

변수가 기본적으로 불변인 것처럼 참조도 마찬가지입니다. 우리는 우리가 참조하는 것을 수정할 수 없습니다.

### [변경 가능한 참조](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#mutable-references)

*목록 4-6의 코드를 수정하여 변경 가능한 참조* 대신 사용하는 몇 가지 작은 조정만으로 빌린 값을 수정할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut s = String::from(`hello`);

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(`, world`);
}
```

먼저 `s`를 `mut`로 변경합니다. 그런 다음 `change` 함수를 호출하는 `&mut s`로 변경 가능한 참조를 만들고 `some_string: &mut String`으로 변경 가능한 참조를 허용하도록 함수 시그니처를 업데이트합니다. 이것은 `변경` 기능이 빌린 값을 변경한다는 것을 매우 분명하게 합니다.

변경 가능한 참조에는 한 가지 큰 제한이 있습니다. 값에 대한 변경 가능한 참조가 있는 경우 해당 값에 대한 다른 참조를 가질 수 없습니다. `s`에 대한 두 개의 변경 가능한 참조를 만들려고 시도하는 이 코드는 실패합니다.

파일 이름: src/main.rs

```rust
    let mut s = String::from(`hello`);

    let r1 = &mut s;
    let r2 = &mut s;

    println!(`{}, {}`, r1, r2);
```

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!(`{}, {}`, r1, r2);
  |                        -- first borrow later used here

For more information about this error, try `rustc --explain E0499`.
error: could not compile `ownership` due to previous error
```

이 오류는 `s`를 한 번에 두 번 이상 가변으로 빌릴 수 없기 때문에 이 코드가 유효하지 않다고 말합니다. 첫 번째 변경 가능한 차용은 `r1`에 있으며 `println!`에서 사용될 때까지 지속되어야 합니다. `r1`로.

동일한 데이터에 대한 여러 가변 참조를 동시에 금지하는 제한은 매우 통제된 방식으로 변형을 허용합니다. 대부분의 언어에서는 원할 때마다 변경할 수 있기 때문에 새로운 Rustacean이 어려움을 겪고 있습니다. 이 제한을 갖는 이점은 Rust가 컴파일 시간에 데이터 경합을 방지할 수 있다는 것입니다. 데이터 *경쟁은* 경쟁 조건과 유사하며 다음 세 가지 동작이 발생할 때 발생합니다.

- 두 개 이상의 포인터가 동시에 동일한 데이터에 액세스합니다.
- 적어도 하나의 포인터가 데이터에 쓰는 데 사용되고 있습니다.
- 데이터에 대한 액세스를 동기화하는 데 사용되는 메커니즘이 없습니다.

데이터 경합은 정의되지 않은 동작을 유발하며 런타임 시 추적하려고 할 때 진단 및 수정이 어려울 수 있습니다. Rust는 데이터 경합으로 코드를 컴파일하는 것을 거부함으로써 이 문제를 방지합니다!

항상 그렇듯이 중괄호를 사용하여 새 스코프를 생성할 수 있으므로 *동시* 참조가 아닌 여러 변경 가능한 참조를 허용할 수 있습니다.

```rust
    let mut s = String::from(`hello`);

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
```

Rust는 변경 가능한 참조와 변경 불가능한 참조를 결합하기 위해 유사한 규칙을 적용합니다. 이 코드는 오류를 발생시킵니다.

```rust
    let mut s = String::from(`hello`);

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!(`{}, {}, and {}`, r1, r2, r3);
```

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!(`{}, {}, and {}`, r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error
```

아휴! *또한* 동일한 값에 대한 불변 참조가 있는 동안에는 가변 참조를 가질 수 없습니다.

불변 참조의 사용자는 값이 갑자기 변경될 것이라고 기대하지 않습니다! 그러나 데이터를 읽기만 하는 사람은 다른 사람의 데이터 읽기에 영향을 줄 수 없기 때문에 여러 불변 참조가 허용됩니다.

참조의 범위는 도입된 위치에서 시작하여 해당 참조가 마지막으로 사용된 시간까지 계속됩니다. 예를 들어, 이 코드는 가변 참조가 도입되기 전에 불변 참조의 마지막 사용인 `println!`이 발생하기 때문에 컴파일됩니다.

```rust
    let mut s = String::from(`hello`);

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!(`{} and {}`, r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!(`{}`, r3);
```

불변 참조 `r1` 및 `r2`의 범위는 `println!` 다음에 끝납니다. 변경 가능한 참조 `r3`이 생성되기 전 마지막으로 사용되는 위치입니다. 이러한 범위는 겹치지 않으므로 이 코드가 허용됩니다. 컴파일러는 범위가 끝나기 전 지점에서 참조가 더 이상 사용되지 않는다는 것을 알 수 있습니다.

차용 오류가 때때로 실망스러울 수 있지만, 잠재적인 버그를 일찍(런타임이 아닌 컴파일 타임에) 지적하고 문제가 있는 곳을 정확히 보여주는 것은 Rust 컴파일러라는 점을 기억하십시오. 그러면 데이터가 생각했던 것과 다른 이유를 추적할 필요가 없습니다.

### [매달린 참조](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#dangling-references)

*포인터가 있는 언어에서는 해당 메모리에 대한 포인터를 유지하면서 일부 메모리를 해제하여 댕글링 포인터* (다른 사람에게 제공되었을 수 있는 메모리의 위치를 참조하는 포인터)를 잘못 생성하기 쉽습니다. 대조적으로 Rust에서 컴파일러는 참조가 댕글링 참조가 되지 않도록 보장합니다. 일부 데이터에 대한 참조가 있는 경우 컴파일러는 데이터에 대한 참조가 범위를 벗어나기 전에 데이터가 범위를 벗어나지 않도록 합니다.

러스트가 어떻게 컴파일 타임 오류를 방지하는지 알아보기 위해 댕글링 참조를 생성해 봅시다:

파일 이름: src/main.rs

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from(`hello`);

    &s
}
```

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` due to previous error
```

이 오류 메시지는 아직 다루지 않은 기능인 수명을 나타냅니다. 10장에서 수명에 대해 자세히 논의할 것입니다. 그러나 수명에 대한 부분을 무시하면 메시지에는 이 코드가 문제인 이유에 대한 핵심이 포함되어 있습니다.

```
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

`dangle` 코드의 각 단계에서 정확히 어떤 일이 발생하는지 자세히 살펴보겠습니다.

파일 이름: src/main.rs

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from(`hello`); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

`dangle` 내부에 `s`가 생성되기 때문에 `dangle`의 코드가 완료되면 `s`가 할당 해제됩니다. 그러나 우리는 그것에 대한 참조를 반환하려고 했습니다. 이는 이 참조가 잘못된 `문자열`을 가리키고 있음을 의미합니다. 좋지 않아! Rust는 우리가 이것을 하도록 허용하지 않을 것입니다.

여기서 해결책은 `문자열`을 직접 반환하는 것입니다.

```rust
fn no_dangle() -> String {
    let s = String::from(`hello`);

    s
}
```

아무 문제 없이 작동합니다. 소유권이 이동되고 할당이 취소되지 않습니다.

### [참조 규칙](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#the-rules-of-references)

참조에 대해 논의한 내용을 요약해 보겠습니다.

- 주어진 시간에 *하나* 의 가변 참조 *또는* 여러 불변 참조를 가질 수 있습니다.
- 참조는 항상 유효해야 합니다.

다음으로 다른 종류의 참조인 슬라이스를 살펴보겠습니다.
