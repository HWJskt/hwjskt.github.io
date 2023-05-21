
+++
title = "11-17 translation"
weight = 1
template ="book_page_rust.html"

+++







# [자동화된 테스트 작성](https://doc.rust-lang.org/book/ch11-00-testing.html#writing-automated-tests)

Edsger W. Dijkstra는 1972년 에세이 `The Humble Programmer`에서 `프로그램 테스트는 버그의 존재를 보여주는 매우 효과적인 방법이 될 수 있지만, 버그의 부재를 보여주기에는 절망적으로 부적절합니다.`라고 말했습니다. 그것은 우리가 할 수 있는 한 많이 테스트하려고 하지 말아야 한다는 것을 의미하지는 않습니다!

프로그램의 정확성은 코드가 의도한 바를 수행하는 정도입니다. Rust는 프로그램의 정확성에 대해 높은 수준의 관심을 가지고 설계되었지만 정확성은 복잡하고 증명하기 쉽지 않습니다. Rust의 유형 시스템은 이러한 부담의 상당 부분을 짊어지고 있지만 유형 시스템이 모든 것을 처리할 수는 없습니다. 따라서 Rust에는 자동화된 소프트웨어 테스트 작성 지원이 포함되어 있습니다.

전달되는 숫자에 2를 더하는 `add_two` 함수를 작성한다고 가정해 보겠습니다. 이 함수의 시그니처는 매개변수로 정수를 받아들이고 결과로 정수를 반환합니다. 우리가 그 함수를 구현하고 컴파일할 때 Rust는 예를 들어 이 함수에 `문자열` 값이나 유효하지 않은 참조를 전달하지 않도록 하기 위해 여러분이 지금까지 배운 모든 유형 검사와 차용 검사를 수행합니다. 그러나 러스트는 이 함수가 정확히 우리가 의도한 바를 수행하는지 확인할 *수 없습니다* . 예를 들어 매개변수에 10을 더하거나 매개변수에서 50을 뺀 것이 아니라 매개변수에 2를 더한 값을 반환하는 것입니다! 그것이 테스트가 들어오는 곳입니다.

예를 들어 `add_two` 함수에 `3`을 전달할 때 반환 값이 `5`라고 주장하는 테스트를 작성할 수 있습니다. 기존의 올바른 동작이 변경되지 않았는지 확인하기 위해 코드를 변경할 때마다 이러한 테스트를 실행할 수 있습니다.

테스트는 복잡한 기술입니다. 한 장에서 좋은 테스트를 작성하는 방법에 대한 모든 세부 사항을 다룰 수는 없지만 Rust의 테스트 기능의 메커니즘에 대해 논의할 것입니다. 테스트를 작성할 때 사용할 수 있는 주석 및 매크로, 테스트 실행을 위해 제공되는 기본 동작 및 옵션, 테스트를 단위 테스트 및 통합 테스트로 구성하는 방법에 대해 설명합니다.

------

## [테스트 작성 방법](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#how-to-write-tests)

테스트는 테스트가 아닌 코드가 예상대로 작동하는지 확인하는 Rust 기능입니다. 테스트 함수 본문은 일반적으로 다음 세 가지 작업을 수행합니다.

1. 필요한 데이터 또는 상태를 설정합니다.
2. 테스트하려는 코드를 실행합니다.
3. 결과가 당신이 기대하는 것이라고 주장하십시오.

`test` 속성, 몇 가지 매크로 및 `should_panic` 속성을 포함하여 Rust가 이러한 작업을 수행하는 테스트 작성을 위해 특별히 제공하는 기능을 살펴보겠습니다.

### [테스트 함수 분석](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#the-anatomy-of-a-test-function)

가장 간단하게 Rust의 테스트는 `test` 속성으로 주석이 달린 함수입니다. 속성은 Rust 코드 조각에 대한 메타데이터입니다. 한 가지 예는 5장에서 구조체와 함께 사용한 `derive` 특성입니다. 함수를 테스트 함수로 변경하려면 `fn` 앞 줄에 `#[test]`를 추가합니다. `cargo test` 명령으로 테스트를 실행할 때 Rust는 주석이 달린 함수를 실행하고 각 테스트 함수의 통과 여부를 보고하는 테스트 실행기 바이너리를 빌드합니다.

Cargo로 새 라이브러리 프로젝트를 만들 때마다 테스트 기능이 포함된 테스트 모듈이 자동으로 생성됩니다. 이 모듈은 테스트 작성을 위한 템플릿을 제공하므로 새 프로젝트를 시작할 때마다 정확한 구조와 구문을 조회할 필요가 없습니다. 추가 테스트 기능과 테스트 모듈을 원하는 만큼 추가할 수 있습니다!

실제로 코드를 테스트하기 전에 템플릿 테스트를 실험하여 테스트 작동 방식의 몇 가지 측면을 살펴보겠습니다. 그런 다음 우리가 작성한 일부 코드를 호출하고 해당 동작이 올바른지 확인하는 실제 테스트를 작성합니다.

두 개의 숫자를 더할 `adder`라는 새 라이브러리 프로젝트를 만들어 보겠습니다.

```bash
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

*`가산기` 라이브러리에 있는 src/lib.rs* 파일 의 내용은 목록 11-1과 같아야 합니다.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

목록 11-1: `cargo new`에 의해 자동으로 생성된 테스트 모듈 및 함수

지금은 맨 위 두 줄을 무시하고 함수에 집중하겠습니다. `#[test]` 주석: 이 속성은 이것이 테스트 함수임을 나타내므로 테스트 러너는 이 함수를 테스트로 취급해야 함을 알고 있습니다. 일반적인 시나리오를 설정하거나 일반적인 작업을 수행하는 데 도움이 되도록 `테스트` 모듈에 테스트가 아닌 기능이 있을 수 있으므로 항상 어떤 기능이 테스트인지 표시해야 합니다.

예제 함수 본문은 `assert_eq!` 매크로는 2와 2를 더한 결과가 포함된 `result`가 4와 같다고 주장합니다. 이 주장은 일반적인 테스트 형식의 예입니다. 이 테스트가 통과하는지 확인하기 위해 실행해 봅시다.

`cargo test` 명령은 목록 11-2와 같이 프로젝트의 모든 테스트를 실행합니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Listing 11-2: 자동으로 생성된 테스트 실행 결과

Cargo는 테스트를 컴파일하고 실행했습니다. `running 1 test`라는 줄이 보입니다. 다음 줄은 `it_works`라는 생성된 테스트 함수의 이름과 해당 테스트를 실행한 결과가 `ok`임을 보여줍니다. 전체 요약 `테스트 결과: ok.` 모든 테스트가 통과되었음을 의미하며 `1 통과, 0 실패`라고 표시된 부분은 통과 또는 실패한 테스트의 총 수입니다.

특정 인스턴스에서 실행되지 않도록 테스트를 무시됨으로 표시할 수 있습니다. [이 장 뒷부분의 `특별히 요청하지 않는 한 일부 테스트 무시`](https://doc.rust-lang.org/book/ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested) 섹션 에서 이에 대해 다룰 것입니다. 여기에서 수행하지 않았기 때문에 요약에는 `0 무시됨`이 표시됩니다. 또한 `cargo test` 명령에 인수를 전달하여 이름이 문자열과 일치하는 테스트만 실행할 수 있습니다. 이것을 *필터링 이라고 하며* [`이름으로 테스트 하위 집합 실행`](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-a-subset-of-tests-by-name) 섹션 에서 다룰 것입니다. 또한 실행 중인 테스트를 필터링하지 않았으므로 요약 끝에 `0 필터링됨`이 표시됩니다.

`0개 측정됨` 통계는 성능을 측정하는 벤치마크 테스트용입니다. 벤치마크 테스트는 이 글을 쓰는 시점에서 nightly Rust에서만 사용할 수 있습니다. 자세한 내용은 [벤치마크 테스트에 대한 설명서를](https://doc.rust-lang.org/unstable-book/library-features/test.html) 참조하십시오 .

`Doc-tests adder`에서 시작하는 테스트 출력의 다음 부분은 모든 문서 테스트의 결과입니다. 아직 문서 테스트가 없지만 Rust는 API 문서에 나타나는 모든 코드 예제를 컴파일할 수 있습니다. 이 기능은 문서와 코드를 동기화하는 데 도움이 됩니다! [14장의 `테스트로서의 문서 주석`](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests) 섹션 에서 문서 테스트를 작성하는 방법에 대해 논의할 것입니다. 지금은 `Doc-tests` 출력을 무시합니다.

우리의 필요에 맞게 테스트를 사용자 지정해 보겠습니다. 먼저 `it_works` 함수의 이름을 `exploration`과 같은 다른 이름으로 다음과 같이 변경합니다.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

그런 다음 `화물 테스트`를 다시 실행하십시오. 이제 출력에 `it_works` 대신 `exploration`이 표시됩니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.59s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

이제 우리는 다른 테스트를 추가할 것이지만 이번에는 실패하는 테스트를 만들 것입니다! 테스트 기능에 패닉이 발생하면 테스트가 실패합니다. 각 테스트는 새 스레드에서 실행되며 기본 스레드에서 테스트 스레드가 종료된 것을 확인하면 테스트가 실패로 표시됩니다. 9장에서 패닉에 빠지는 가장 간단한 방법은 `패닉!` 매크로. 새 테스트를 `another`라는 이름의 함수로 입력하면 *src/lib.rs* 파일이 목록 11-3처럼 보입니다.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!(`Make this test fail`);
    }
}
```

Listing 11-3: `패닉!` 매크로

`화물 테스트`를 사용하여 테스트를 다시 실행하십시오. 출력은 목록 11-4와 같아야 합니다. `탐색` 테스트는 통과했고 `다른` 테스트는 실패했음을 보여줍니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.72s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::another ... FAILED
test tests::exploration ... ok

failures:

---- tests::another stdout ----
thread `tests::another` panicked at `Make this test fail`, src/lib.rs:10:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Listing 11-4: 하나의 테스트는 통과하고 하나는 실패했을 때의 테스트 결과

`ok` 대신 `test tests::another` 줄에 `FAILED`가 표시됩니다. 개별 결과와 요약 사이에 두 개의 새 섹션이 나타납니다. 첫 번째 섹션에는 각 테스트 실패에 대한 자세한 이유가 표시됩니다. 이 경우 `another`가 *src/lib.rs* 파일의 10행에서 ``Make this test fail`에서 당황했기 때문에` 실패했다는 세부 정보를 얻습니다. 다음 섹션에는 실패한 모든 테스트의 이름만 나열되어 있습니다. 이는 많은 테스트와 자세한 실패한 테스트 출력이 많은 경우에 유용합니다. 실패한 테스트의 이름을 사용하여 해당 테스트만 실행하면 더 쉽게 디버그할 수 있습니다. [`테스트 실행 방법 제어`](https://doc.rust-lang.org/book/ch11-02-running-tests.html#controlling-how-tests-are-run) 섹션 에서 테스트를 실행하는 방법에 대해 자세히 설명합니다.

요약 줄은 마지막에 표시됩니다. 전반적으로 테스트 결과는 `FAILED`입니다. 한 번의 테스트 통과와 한 번의 테스트 실패가 있었습니다.

다양한 시나리오에서 테스트 결과가 어떻게 보이는지 확인했으므로 이제 `panic!` 이외의 일부 매크로를 살펴보겠습니다. 테스트에 유용합니다.

### [`assert!`로 결과 확인 매크로](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#checking-results-with-the-assert-macro)

`주장!` 표준 라이브러리에서 제공하는 매크로는 테스트의 일부 조건이 `true`로 평가되도록 하려는 경우에 유용합니다. 우리는 `주장!` 매크로는 부울로 평가되는 인수입니다. 값이 `true`이면 아무 일도 일어나지 않고 테스트가 통과됩니다. 값이 `false`이면 `assert!` 매크로 호출 `패닉!` 테스트가 실패하도록 합니다. `어설션!` 매크로는 코드가 의도한 대로 작동하는지 확인하는 데 도움이 됩니다.

5장, Listing 5-15에서 우리는 `Rectangle` 구조체와 `can_hold` 메서드를 사용했고, 여기 Listing 11-5에서 반복됩니다. *이 코드를 src/lib.rs* 파일 에 넣고 `assert!` 매크로.

파일 이름: src/lib.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Listing 11-5: 5장의 `Rectangle` 구조체와 `can_hold` 메서드 사용

`can_hold` 메서드는 부울을 반환합니다. 즉, `assert!`에 대한 완벽한 사용 사례입니다. 매크로. Listing 11-6에서 너비가 8이고 높이가 7인 `Rectangle` 인스턴스를 생성하고 너비가 7인 다른 `Rectangle` 인스턴스를 보유할 수 있다고 주장하여 `can_hold` 메서드를 실행하는 테스트를 작성합니다. 5의 높이와 1의 높이.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}
```

Listing 11-6: 더 큰 사각형이 실제로 더 작은 사각형을 담을 수 있는지 확인하는 `can_hold` 테스트

`tests` 모듈 내부에 `use super::*;`라는 새 줄을 추가했습니다. [`tests` 모듈은 7장 `모듈 트리에서 항목을 참조하기 위한 경로`](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) 섹션 에서 다룬 일반적인 가시성 규칙을 따르는 일반 모듈입니다. `tests` 모듈은 내부 모듈이기 때문에 외부 모듈의 테스트 중인 코드를 내부 모듈의 범위로 가져와야 합니다. 우리는 외부 모듈에서 정의한 모든 것을 이 `테스트` 모듈에서 사용할 수 있도록 여기에서 glob을 사용합니다.

테스트 이름을 `larger_can_hold_smaller`로 지정하고 필요한 두 개의 `Rectangle` 인스턴스를 만들었습니다. 그런 다음 `assert!`라고 불렀습니다. 매크로를 호출하고 `larger.can_hold(&smaller)`를 호출한 결과를 전달했습니다. 이 식은 `true`를 반환해야 하므로 테스트를 통과해야 합니다. 알아 보자!

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

통과합니다! 이번에는 더 작은 직사각형이 더 큰 직사각형을 수용할 수 없다고 주장하는 또 다른 테스트를 추가해 보겠습니다.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
}
```

이 경우 `can_hold` 함수의 올바른 결과는 `거짓`이므로 `assert!` 매크로. 결과적으로 `can_hold`가 `false`를 반환하면 테스트가 통과됩니다.

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... ok
test tests::smaller_cannot_hold_larger ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

통과하는 두 가지 테스트! 이제 우리 코드에 버그가 생겼을 때 테스트 결과에 어떤 일이 일어나는지 봅시다. 너비를 비교할 때 보다 큼 기호를 보다 작음 기호로 대체하여 `can_hold` 메서드의 구현을 변경합니다.

```rust
// --snip--
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width < other.width && self.height > other.height
    }
}
```

이제 테스트를 실행하면 다음이 생성됩니다.

```bash
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... FAILED
test tests::smaller_cannot_hold_larger ... ok

failures:

---- tests::larger_can_hold_smaller stdout ----
thread `tests::larger_can_hold_smaller` panicked at `assertion failed: larger.can_hold(&smaller)`, src/lib.rs:28:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

테스트에서 버그를 잡았습니다! `larger.width`는 8이고 `smaller.width`는 5이므로 `can_hold`의 너비 비교는 이제 `false`를 반환합니다. 8은 5보다 작지 않습니다.

### [`assert_eq!`로 평등 테스트하기 그리고 `assert_ne!` 매크로](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#testing-equality-with-the-assert_eq-and-assert_ne-macros)

기능을 확인하는 일반적인 방법은 테스트 중인 코드의 결과와 코드가 반환할 것으로 예상하는 값이 같은지 테스트하는 것입니다. `assert!`를 사용하여 이를 수행할 수 있습니다. 매크로를 만들고 `==` 연산자를 사용하여 식을 전달합니다. 그러나 이것은 표준 라이브러리가 한 쌍의 매크로(`assert_eq!`)를 제공하는 일반적인 테스트입니다. 및 `assert_ne!` - 이 테스트를 보다 편리하게 수행합니다. 이러한 매크로는 각각 같음 또는 같지 않음에 대한 두 인수를 비교합니다. 또한 어설션이 실패하면 두 값을 인쇄하므로 테스트가 실패한 *이유를* 쉽게 확인할 수 있습니다. 반대로 `assert!` 매크로는 `false` 값으로 이어진 값을 인쇄하지 않고 `==` 표현식에 대해 `false` 값을 얻었음을 나타냅니다.

목록 11-7에서 매개변수에 `2`를 추가하는 `add_two`라는 함수를 작성한 다음 `assert_eq!` 매크로.

파일 이름: src/lib.rs

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

목록 11-7: `assert_eq!`를 사용하여 `add_two` 함수 테스트 매크로

통과했는지 확인해보자!

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

`add_two(2)`를 호출한 결과와 동일한 `assert_eq!`에 `4`를 인수로 전달합니다. 이 테스트의 줄은 `test tests::it_adds_two ... ok`이고 `ok` 텍스트는 테스트가 통과되었음을 나타냅니다!

코드에 버그를 도입하여 `assert_eq!` 실패했을 때의 모습입니다. 대신 `3`을 추가하도록 `add_two` 함수의 구현을 변경합니다.

```rust
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

테스트를 다시 실행합니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
thread `tests::it_adds_two` panicked at `assertion failed: `(left == right)`
  left: `4`,
 right: `5``, src/lib.rs:11:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

테스트에서 버그를 잡았습니다! `it_adds_two` 테스트가 실패했으며 메시지는 실패한 어설션이 `assertion failed: `(left == right)``이고 `left` 및 `right` 값이 무엇인지 알려줍니다. 이 메시지는 디버깅을 시작하는 데 도움이 됩니다. `왼쪽` 인수는 `4`였지만 `add_two(2)`가 있는 `오른쪽` 인수는 `5`였습니다. 많은 테스트가 진행될 때 이것이 특히 도움이 될 것이라고 상상할 수 있습니다.

일부 언어 및 테스트 프레임워크에서는 동등 주장 함수에 대한 매개변수를 `예상` 및 `실제`라고 하며 인수를 지정하는 순서가 중요합니다. 그러나 Rust에서는 `left`와 `right`라고 부르며 우리가 기대하는 값과 코드가 생성하는 값을 지정하는 순서는 중요하지 않습니다. 이 테스트에서 어설션을 `assert_eq!(add_two(2), 4)`로 작성할 수 있습니다. 그러면 `어설션 실패: `(left == right)``를 표시하는 동일한 실패 메시지가 표시됩니다.

`assert_ne!` 매크로는 우리가 제공한 두 값이 같지 않으면 통과하고 같으면 실패합니다. 이 매크로는 값이 무엇인지 확신할 수 없지만 값이 절대 아니어야 할 값을 알고 있는 경우에 *가장* 유용 *합니다* . 예를 들어, 어떤 식으로든 입력을 변경하는 것이 보장되는 함수를 테스트하고 있지만 입력이 변경되는 방식은 테스트를 실행하는 요일에 따라 달라지는 경우 가장 좋은 주장은 다음과 같습니다. 함수의 출력이 입력과 같지 않다는 것.

표면 아래에서 `assert_eq!` 그리고 `assert_ne!` 매크로는 각각 `==` 및 `!=` 연산자를 사용합니다. 어설션이 실패하면 이러한 매크로는 디버그 형식을 사용하여 인수를 인쇄합니다. 즉, 비교되는 값은 `PartialEq` 및 `Debug` 특성을 구현해야 합니다. 모든 기본 유형과 대부분의 표준 라이브러리 유형은 이러한 특성을 구현합니다. 직접 정의하는 구조체 및 열거형의 경우 이러한 유형의 동등성을 주장하려면 `PartialEq`를 구현해야 합니다. 또한 어설션이 실패할 때 값을 인쇄하려면 `디버그`를 구현해야 합니다. 5장의 Listing 5-12에서 언급한 것처럼 두 특성 모두 파생 가능한 특성이기 때문에 이것은 일반적으로 `#[derive(PartialEq, Debug)]`를 추가하는 것만큼 간단합니다. 구조체 또는 열거형 정의에 대한 주석입니다. [부록 C, `파생 가능한 특성`을 참조하십시오.](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html)이러한 특성 및 기타 파생 특성에 대한 자세한 내용은

### [사용자 지정 실패 메시지 추가](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#adding-custom-failure-messages)

`assert!`, `assert_eq!` 및 `assert_ne!` 매크로. 필수 인수 뒤에 지정된 모든 인수는 `형식!` [매크로( `+` 연산자 또는 `형식!` 매크로를 사용한 연결](https://doc.rust-lang.org/book/ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro) 섹션의 8장에서 설명 )를 사용하여 `{}` 자리 표시자와 해당 자리 표시자에 들어갈 값을 포함하는 형식 문자열을 전달할 수 있습니다. 사용자 지정 메시지는 어설션의 의미를 문서화하는 데 유용합니다. 테스트가 실패하면 코드의 문제가 무엇인지 더 잘 알 수 있습니다.

예를 들어 이름으로 인사하는 함수가 있고 함수에 전달한 이름이 출력에 나타나는지 테스트하고 싶다고 가정해 보겠습니다.

파일 이름: src/lib.rs

```rust
pub fn greeting(name: &str) -> String {
    format!(`Hello {}!`, name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting(`Carol`);
        assert!(result.contains(`Carol`));
    }
}
```

이 프로그램에 대한 요구 사항은 아직 합의되지 않았으며 인사말 시작 부분의 `Hello` 텍스트가 변경될 것이라고 확신합니다. 우리는 요구 사항이 변경될 때 테스트를 업데이트하고 싶지 않기 때문에 `인사말` 함수에서 반환된 값과 정확히 같은지 확인하는 대신 출력에 입력 텍스트가 포함되어 있다고 주장할 것입니다. 매개변수.

이제 `인사말`을 `이름`을 제외하도록 변경하여 이 코드에 버그를 도입하여 기본 테스트 실패가 어떻게 보이는지 확인합니다.

```rust
pub fn greeting(name: &str) -> String {
    String::from(`Hello!`)
}
```

이 테스트를 실행하면 다음이 생성됩니다.

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread `tests::greeting_contains_name` panicked at `assertion failed: result.contains(\`Carol\`)`, src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

이 결과는 어설션이 실패했으며 어설션이 있는 라인을 나타냅니다. 보다 유용한 실패 메시지는 `인사` 기능의 값을 인쇄합니다. `greeting` 함수에서 얻은 실제 값으로 채워진 자리 표시자가 있는 형식 문자열로 구성된 사용자 지정 실패 메시지를 추가해 보겠습니다.

```rust
    #[test]
    fn greeting_contains_name() {
        let result = greeting(`Carol`);
        assert!(
            result.contains(`Carol`),
            `Greeting did not contain name, value was `{}``,
            result
        );
    }
```

이제 테스트를 실행하면 더 많은 정보를 제공하는 오류 메시지가 표시됩니다.

```bash
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.93s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread `tests::greeting_contains_name` panicked at `Greeting did not contain name, value was `Hello!``, src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

우리는 테스트 출력에서 실제로 얻은 값을 볼 수 있으며, 이는 우리가 예상했던 일 대신 발생한 일을 디버깅하는 데 도움이 됩니다.

### [`should_panic`으로 패닉 확인](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#checking-for-panics-with-should_panic)

반환 값을 확인하는 것 외에도 코드가 예상대로 오류 조건을 처리하는지 확인하는 것이 중요합니다. 예를 들어, 9장, 목록 9-13에서 생성한 `Guess` 유형을 고려하십시오. `Guess`를 사용하는 다른 코드는 `Guess` 인스턴스가 1에서 100 사이의 값만 포함한다는 보장에 따라 달라집니다. 해당 범위 밖의 값으로 `Guess` 인스턴스를 만들려고 시도하면 패닉이 발생하는지 확인하는 테스트를 작성할 수 있습니다.

테스트 기능에 `should_panic` 속성을 추가하여 이를 수행합니다. 함수 내부의 코드가 패닉 상태이면 테스트에 통과합니다. 함수 내부의 코드가 패닉하지 않으면 테스트가 실패합니다.

목록 11-8은 `Guess::new`의 오류 조건이 우리가 예상할 때 발생하는지 확인하는 테스트를 보여줍니다.

파일 이름: src/lib.rs

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!(`Guess value must be between 1 and 100, got {}.`, value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

목록 11-8: 조건이 `패닉!`을 유발하는지 테스트

우리는 `#[should_panic]` 속성을 `#[test]` 속성 뒤와 그것이 적용되는 테스트 함수 앞에 배치합니다. 이 테스트가 통과되면 결과를 살펴보겠습니다.

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests guessing_game

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

좋아 보인다! 이제 값이 100보다 크면 `new` 함수가 패닉이 되는 조건을 제거하여 코드에 버그를 도입해 보겠습니다.

```rust
// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(`Guess value must be between 1 and 100, got {}.`, value);
        }

        Guess { value }
    }
}
```

목록 11-8의 테스트를 실행하면 실패합니다.

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
note: test did not panic as expected

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

이 경우 매우 유용한 메시지를 얻지는 못하지만 테스트 기능을 보면 `#[should_panic]` 주석이 달린 것을 볼 수 있습니다. 우리가 얻은 실패는 테스트 기능의 코드가 패닉을 일으키지 않았음을 의미합니다.

`should_panic`을 사용하는 테스트는 부정확할 수 있습니다. `should_panic` 테스트는 우리가 예상했던 것과 다른 이유로 테스트 패닉이 발생하더라도 통과합니다. `should_panic` 테스트를 더 정확하게 만들기 위해 `should_panic` 속성에 선택적 `expected` 매개변수를 추가할 수 있습니다. 테스트 도구는 실패 메시지에 제공된 텍스트가 포함되어 있는지 확인합니다. 예를 들어, 값이 너무 작거나 큰지에 따라 다른 메시지와 함께 `new` 함수 패닉이 발생하는 Listing 11-9의 `Guess`에 대한 수정된 코드를 고려하십시오.

파일 이름: src/lib.rs

```rust
// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                `Guess value must be greater than or equal to 1, got {}.`,
                value
            );
        } else if value > 100 {
            panic!(
                `Guess value must be less than or equal to 100, got {}.`,
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = `less than or equal to 100`)]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

Listing 11-9: `panic!` 테스트 지정된 하위 문자열을 포함하는 패닉 메시지와 함께

`should_panic` 속성의 `expected` 매개변수에 넣은 값이 `Guess::new` 함수가 패닉 상태가 되는 메시지의 하위 문자열이기 때문에 이 테스트는 통과할 것입니다. 예상되는 전체 패닉 메시지를 지정할 수 있습니다. 이 경우 `Guess value must be less or equal to 100, got 200.`이 됩니다. 지정하기 위해 선택하는 항목은 패닉 메시지가 고유하거나 동적인 정도와 원하는 테스트의 정확도에 따라 다릅니다. 이 경우 패닉 메시지의 하위 문자열은 테스트 함수의 코드가 `else if value > 100` 사례를 실행하는 데 충분합니다.

`expected` 메시지가 있는 `should_panic` 테스트가 실패할 때 어떤 일이 발생하는지 확인하기 위해 `if value < 1` 블록과 `else if value > 100` 블록의 본문을 교체하여 코드에 버그를 다시 도입해 보겠습니다.

```rust
        if value < 1 {
            panic!(
                `Guess value must be less than or equal to 100, got {}.`,
                value
            );
        } else if value > 100 {
            panic!(
                `Guess value must be greater than or equal to 1, got {}.`,
                value
            );
        }
```

이번에는 `should_panic` 테스트를 실행하면 실패합니다.

```bash
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
thread `tests::greater_than_100` panicked at `Guess value must be greater than or equal to 1, got 200.`, src/lib.rs:13:13
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
note: panic did not contain expected string
      panic message: ``Guess value must be greater than or equal to 1, got 200.``,
 expected substring: ``less than or equal to 100``

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

실패 메시지는 이 테스트가 실제로 우리가 예상한 대로 패닉이 발생했음을 나타내지만 패닉 메시지에는 예상 문자열 ``Guess value must be less than or equal to 100``이 포함되지 않았습니다. 이 경우에 우리가 받은 패닉 메시지는 `Guess value must be greater or equal to 1, got 200.`이었습니다. 이제 버그가 어디에 있는지 알아낼 수 있습니다!

### [테스트에서 `결과` 사용](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#using-resultt-e-in-tests)

지금까지 우리의 테스트는 실패하면 모두 당황합니다. `Result<T, E>`를 사용하는 테스트를 작성할 수도 있습니다! 다음은 Listing 11-1의 테스트입니다. `Result<T, E>`를 사용하고 당황하지 않고 `Err`를 반환하도록 재작성했습니다.

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from(`two plus two does not equal four`))
        }
    }
}
```

`it_works` 함수에는 이제 `Result<(), String>` 반환 유형이 있습니다. 함수 본문에서 `assert_eq!` 매크로에서 테스트가 통과하면 `Ok(())`를 반환하고 테스트가 실패하면 내부에 `String`이 있는 `Err`를 반환합니다.

`Result<T, E>`를 반환하도록 테스트를 작성하면 테스트 본문에서 물음표 연산자를 사용할 수 있습니다. 이는 테스트 내의 작업이 `Err`를 반환하는 경우 실패해야 하는 테스트를 작성하는 편리한 방법이 될 수 있습니다. 변종.

`Result<T, E>`를 사용하는 테스트에는 `#[should_panic]` 주석을 사용할 수 없습니다. 작업이 `Err` 변형을 반환한다고 주장하려면 `Result<T, E>` 값에 물음표 연산자를 사용 *하지 마세요 .* 대신 `assert!(value.is_err())`를 사용하십시오.

테스트를 작성하는 여러 가지 방법을 알았으니 이제 테스트를 실행할 때 어떤 일이 발생하는지 살펴보고 `화물 테스트`와 함께 사용할 수 있는 다양한 옵션을 살펴보겠습니다.

------

## [테스트 실행 방법 제어](https://doc.rust-lang.org/book/ch11-02-running-tests.html#controlling-how-tests-are-run)

`cargo run`이 코드를 컴파일한 다음 결과 바이너리를 실행하는 것처럼 `cargo test`는 테스트 모드에서 코드를 컴파일하고 결과 테스트 바이너리를 실행합니다. `cargo test`에 의해 생성된 바이너리의 기본 동작은 모든 테스트를 병렬로 실행하고 테스트 실행 중에 생성된 출력을 캡처하여 출력이 표시되지 않도록 하고 테스트 결과와 관련된 출력을 쉽게 읽을 수 있도록 합니다. 그러나 명령줄 옵션을 지정하여 이 기본 동작을 변경할 수 있습니다.

일부 명령줄 옵션은 `cargo test`로 이동하고 일부는 결과 테스트 바이너리로 이동합니다. 이 두 가지 유형의 인수를 구분하려면 `cargo test`로 이동하는 인수를 나열하고 그 뒤에 구분 기호 `--`를 추가한 다음 테스트 바이너리로 이동하는 인수를 나열합니다. `cargo test --help`를 실행하면 `cargo test`와 함께 사용할 수 있는 옵션이 표시되고 `cargo test -- --help`를 실행하면 구분 기호 뒤에 사용할 수 있는 옵션이 표시됩니다.

### [병렬 또는 연속으로 테스트 실행](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-tests-in-parallel-or-consecutively)

여러 테스트를 실행할 때 기본적으로 스레드를 사용하여 병렬로 실행됩니다. 즉, 실행이 더 빨리 완료되고 피드백을 더 빨리 받습니다. 테스트가 동시에 실행되기 때문에 테스트가 서로 또는 현재 작업 디렉터리 또는 환경 변수와 같은 공유 환경을 포함한 공유 상태에 의존하지 않는지 확인해야 합니다.

*예를 들어 각 테스트 에서 test-output.txt* 라는 디스크에 파일을 생성 하고 해당 파일에 일부 데이터를 쓰는 일부 코드를 실행한다고 가정합니다. 그런 다음 각 테스트는 해당 파일의 데이터를 읽고 파일에 각 테스트마다 다른 특정 값이 포함되어 있다고 주장합니다. 테스트가 동시에 실행되기 때문에 다른 테스트가 파일을 쓰고 읽는 사이에 한 테스트가 파일을 덮어쓸 수 있습니다. 두 번째 테스트는 코드가 잘못되어서가 아니라 테스트가 병렬로 실행되는 동안 서로 간섭했기 때문에 실패합니다. 한 가지 해결책은 각 테스트가 다른 파일에 기록되는지 확인하는 것입니다. 또 다른 해결책은 테스트를 한 번에 하나씩 실행하는 것입니다.

테스트를 병렬로 실행하고 싶지 않거나 사용되는 스레드 수를 보다 세밀하게 제어하려면 `--test-threads` 플래그와 사용하려는 스레드 수를 보낼 수 있습니다. 테스트 바이너리. 다음 예를 살펴보십시오.

```bash
$ cargo test -- --test-threads=1
```

테스트 스레드 수를 `1`로 설정하여 프로그램에 병렬 처리를 사용하지 않도록 지시합니다. 하나의 스레드를 사용하여 테스트를 실행하면 병렬로 실행하는 것보다 시간이 더 오래 걸리지만 테스트가 상태를 공유하는 경우 서로 간섭하지 않습니다.

### [함수 출력 표시](https://doc.rust-lang.org/book/ch11-02-running-tests.html#showing-function-output)

기본적으로 테스트가 통과하면 Rust의 테스트 라이브러리는 표준 출력으로 인쇄된 모든 것을 캡처합니다. 예를 들어 `println!` 테스트에서 테스트에 통과하면 `println!` 터미널에서 출력; 테스트 통과를 나타내는 줄만 표시됩니다. 테스트가 실패하면 나머지 실패 메시지와 함께 표준 출력으로 출력된 내용을 볼 수 있습니다.

예를 들어 Listing 11-10에는 매개변수 값을 인쇄하고 10을 반환하는 어리석은 함수와 테스트에 통과한 테스트와 실패한 테스트가 있습니다.

파일 이름: src/lib.rs

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!(`I got the value {}`, a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

Listing 11-10: `println!`을 호출하는 함수를 테스트합니다.

`cargo test`로 이러한 테스트를 실행하면 다음과 같은 결과가 표시됩니다.

```bash
$ cargo test
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread `tests::this_test_will_fail` panicked at `assertion failed: `(left == right)`
  left: `5`,
 right: `10``, src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

이 출력의 어디에도 `I got the value 4`가 표시되지 않는다는 점에 유의하십시오. 통과한 테스트가 실행될 때 출력되는 값입니다. 해당 출력이 캡처되었습니다. 실패한 테스트의 출력 `I got the value 8`이 테스트 요약 출력의 섹션에 나타나며 테스트 실패의 원인도 보여줍니다.

테스트 통과에 대한 인쇄된 값도 보려면 `--show-output`을 사용하여 성공적인 테스트의 출력도 표시하도록 Rust에 지시할 수 있습니다.

```bash
$ cargo test -- --show-output
```

`--show-output` 플래그를 사용하여 Listing 11-10의 테스트를 다시 실행하면 다음과 같은 출력이 표시됩니다.

```bash
$ cargo test -- --show-output
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

successes:

---- tests::this_test_will_pass stdout ----
I got the value 4


successes:
    tests::this_test_will_pass

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread `tests::this_test_will_fail` panicked at `assertion failed: `(left == right)`
  left: `5`,
 right: `10``, src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

### [이름으로 테스트 하위 집합 실행](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-a-subset-of-tests-by-name)

경우에 따라 전체 테스트 스위트를 실행하는 데 시간이 오래 걸릴 수 있습니다. 특정 영역에서 코드 작업을 하는 경우 해당 코드와 관련된 테스트만 실행하려고 할 수 있습니다. `cargo test`에 실행하려는 테스트의 이름을 인수로 전달하여 실행할 테스트를 선택할 수 있습니다.

테스트의 하위 집합을 실행하는 방법을 보여주기 위해 목록 11-11에 표시된 것처럼 먼저 `add_two` 함수에 대한 세 가지 테스트를 만들고 실행할 항목을 선택합니다.

파일 이름: src/lib.rs

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

Listing 11-11: 세 가지 다른 이름을 가진 세 가지 테스트

이전에 본 것처럼 인수를 전달하지 않고 테스트를 실행하면 모든 테스트가 병렬로 실행됩니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 3 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

#### [단일 테스트 실행](https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-single-tests)

테스트 함수의 이름을 `cargo test`에 전달하여 해당 테스트만 실행할 수 있습니다.

```bash
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

`one_hundred`라는 이름의 테스트만 실행되었습니다. 다른 두 테스트는 해당 이름과 일치하지 않았습니다. 테스트 출력은 마지막에 `2 필터링 아웃`을 표시하여 실행되지 않은 테스트가 더 있음을 알려줍니다.

이런 방식으로 여러 테스트의 이름을 지정할 수 없습니다. `cargo test`에 주어진 첫 번째 값만 사용됩니다. 그러나 여러 테스트를 실행하는 방법이 있습니다.

#### [여러 테스트를 실행하기 위한 필터링](https://doc.rust-lang.org/book/ch11-02-running-tests.html#filtering-to-run-multiple-tests)

테스트 이름의 일부를 지정할 수 있으며 이름이 해당 값과 일치하는 모든 테스트가 실행됩니다. 예를 들어 테스트 이름 중 두 개에 `add`가 포함되어 있으므로 `cargo test add`를 실행하여 이 두 개를 실행할 수 있습니다.

```bash
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

이 명령은 이름에 `add`가 포함된 모든 테스트를 실행하고 `one_hundred`라는 테스트를 필터링했습니다. 또한 테스트가 나타나는 모듈은 테스트 이름의 일부가 되므로 모듈 이름을 필터링하여 모듈의 모든 테스트를 실행할 수 있습니다.

### [특별히 요청하지 않는 한 일부 테스트 무시](https://doc.rust-lang.org/book/ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested)

때로는 몇 가지 특정 테스트를 실행하는 데 시간이 많이 걸릴 수 있으므로 대부분의 `화물 테스트` 실행 중에 테스트를 제외하고 싶을 수 있습니다. 실행하려는 모든 테스트를 인수로 나열하는 대신 다음과 같이 제외하기 위해 `ignore` 속성을 사용하여 시간이 많이 걸리는 테스트에 주석을 달 수 있습니다.

파일 이름: src/lib.rs

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

`#[test]` 뒤에 제외하려는 테스트에 `#[ignore]` 줄을 추가합니다. 이제 테스트를 실행할 때 `it_works`는 실행되지만 `expensive_test`는 실행되지 않습니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

`expensive_test` 함수는 `ignored`로 나열됩니다. 무시된 테스트만 실행하려면 `cargo test -- --ignored`를 사용할 수 있습니다.

```bash
$ cargo test -- --ignored
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

실행되는 테스트를 제어하여 `화물 테스트` 결과를 빠르게 얻을 수 있습니다. `무시된` 테스트의 결과를 확인하는 것이 합리적이고 결과를 기다릴 시간이 있는 경우 대신 `cargo test -- --ignored`를 실행할 수 있습니다. 무시 여부에 관계없이 모든 테스트를 실행하려면 `cargo test -- --include-ignored`를 실행할 수 있습니다.

------

## [테스트 조직](https://doc.rust-lang.org/book/ch11-03-test-organization.html#test-organization)

이 장의 시작 부분에서 언급했듯이 테스트는 복잡한 규율이며 서로 다른 사람들이 서로 다른 용어와 조직을 사용합니다. Rust 커뮤니티는 단위 테스트와 통합 테스트라는 두 가지 주요 범주 측면에서 테스트에 대해 생각합니다. *단위 테스트* 는 작고 집중적이며 한 번에 하나의 모듈을 격리하여 테스트하고 개인 인터페이스를 테스트할 수 있습니다. *통합 테스트는* 완전히 라이브러리 외부에 있으며 공개 인터페이스만 사용하고 잠재적으로 테스트당 여러 모듈을 실행하는 다른 외부 코드와 동일한 방식으로 코드를 사용합니다.

두 종류의 테스트를 모두 작성하는 것은 라이브러리의 일부가 개별적으로 그리고 함께 기대하는 대로 작동하는지 확인하는 데 중요합니다.

### [단위 테스트](https://doc.rust-lang.org/book/ch11-03-test-organization.html#unit-tests)

단위 테스트의 목적은 나머지 코드와 격리된 각 코드 단위를 테스트하여 코드가 예상대로 작동하는 위치와 작동하지 않는 위치를 신속하게 찾아내는 것입니다. 테스트 중인 코드가 있는 각 파일의 *src* 디렉터리 에 단위 테스트를 배치합니다. 관례는 테스트 기능을 포함하고 `cfg(test)`로 모듈에 주석을 달기 위해 각 파일에 `tests`라는 모듈을 만드는 것입니다.

#### [테스트 모듈 및 `# [cfg(테스트)\]`](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-module-and-cfgtest)

테스트 모듈의 `#[cfg(test)]` 주석은 러스트에게 `cargo build`를 실행할 때가 아니라 `cargo test`를 실행할 때만 테스트 코드를 컴파일하고 실행하도록 지시합니다. 이렇게 하면 테스트가 포함되지 않기 때문에 라이브러리만 빌드하려는 경우 컴파일 시간이 절약되고 결과 컴파일된 아티팩트의 공간이 절약됩니다. 통합 테스트는 다른 디렉토리로 이동하기 때문에 `#[cfg(test)]` 주석이 필요하지 않습니다. 그러나 단위 테스트는 코드와 동일한 파일에 있기 때문에 `#[cfg(test)]`를 사용하여 컴파일된 결과에 포함되지 않아야 함을 지정합니다.

이 장의 첫 번째 섹션에서 새로운 `가산기` 프로젝트를 생성했을 때 Cargo가 우리를 위해 다음 코드를 생성했음을 기억하십시오.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

이 코드는 자동으로 생성된 테스트 모듈입니다. 속성 `cfg`는 *구성을* 의미하며 다음 항목은 특정 구성 옵션이 주어질 때만 포함되어야 함을 Rust에 알려줍니다. 이 경우 구성 옵션은 테스트를 컴파일하고 실행하기 위해 Rust에서 제공하는 `테스트`입니다. Cargo는 `cfg` 속성을 사용하여 `cargo test`로 테스트를 적극적으로 실행하는 경우에만 테스트 코드를 컴파일합니다. 여기에는 `#[test]` 주석이 달린 함수 외에도 이 모듈 내에 있을 수 있는 모든 도우미 함수가 포함됩니다.

#### [비공개 기능 테스트](https://doc.rust-lang.org/book/ch11-03-test-organization.html#testing-private-functions)

개인 기능을 직접 테스트해야 하는지 여부에 대해 테스트 커뮤니티 내에서 논쟁이 있으며 다른 언어는 개인 기능을 테스트하기 어렵거나 불가능합니다. 당신이 고수하는 테스트 이데올로기에 관계없이 Rust의 개인 정보 보호 규칙은 개인 기능을 테스트할 수 있도록 허용합니다. 비공개 함수 `internal_adder`가 있는 Listing 11-12의 코드를 고려하십시오.

파일 이름: src/lib.rs

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

Listing 11-12: 프라이빗 함수 테스트

`internal_adder` 함수는 `pub`로 표시되지 않습니다. 테스트는 Rust 코드일 뿐이고 `tests` 모듈은 또 다른 모듈일 뿐입니다. [`모듈 트리에서 항목을 참조하기 위한 경로`](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) 섹션 에서 논의한 것처럼 자식 모듈의 항목은 상위 모듈의 항목을 사용할 수 있습니다. 이 테스트에서는 `use super::*`를 사용하여 모든 `테스트` 모듈의 상위 항목을 범위로 가져온 다음 테스트에서 `internal_adder`를 호출할 수 있습니다. 개인 함수를 테스트해야 한다고 생각하지 않는다면 Rust에는 그렇게 하도록 강요하는 것이 없습니다.

### [통합 테스트](https://doc.rust-lang.org/book/ch11-03-test-organization.html#integration-tests)

Rust에서 통합 테스트는 완전히 라이브러리 외부에 있습니다. 그들은 다른 코드와 같은 방식으로 라이브러리를 사용합니다. 즉, 라이브러리의 공개 API의 일부인 함수만 호출할 수 있습니다. 그들의 목적은 라이브러리의 많은 부분이 올바르게 함께 작동하는지 여부를 테스트하는 것입니다. 자체적으로 올바르게 작동하는 코드 단위는 통합 시 문제가 발생할 수 있으므로 통합 코드의 테스트 범위도 중요합니다. 통합 테스트를 만들려면 먼저 *테스트* 디렉터리가 필요합니다.

#### [테스트 디렉토리 *_*](https://doc.rust-lang.org/book/ch11-03-test-organization.html#the-tests-directory)

*src* 옆에 있는 프로젝트 디렉토리의 최상위 레벨에 *테스트* 디렉토리를 만듭니다. Cargo는 이 디렉토리에서 통합 테스트 파일을 찾는 것을 알고 있습니다. 그런 다음 원하는 만큼 많은 테스트 파일을 만들 수 있으며 Cargo는 각 파일을 개별 크레이트로 컴파일합니다.

통합 테스트를 만들어 봅시다. 목록 11-12의 코드가 *src/lib.rs 파일에 있는 상태에서* *테스트* 디렉토리를 만들고 *tests/integration_test.rs* 라는 새 파일을 만듭니다. 디렉토리 구조는 다음과 같아야 합니다.

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

목록 11-13의 코드를 *tests/integration_test.rs* 파일에 입력하십시오:

파일 이름: 테스트/integration_test.rs

```rust
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

Listing 11-13: `adder` 크레이트에 있는 함수의 통합 테스트

`tests` 디렉토리의 각 파일은 별도의 크레이트이므로 라이브러리를 각 테스트 크레이트의 범위로 가져와야 합니다. 그런 이유로 우리는 단위 테스트에서 필요하지 않은 `use adder`를 코드 상단에 추가합니다.

*tests/integration_test.rs* 의 코드에 `#[cfg(test)]`로 주석을 달 필요가 없습니다. Cargo는 `tests` 디렉토리를 특별히 취급하고 `cargo test`를 실행할 때만 이 디렉토리의 파일을 컴파일합니다. 지금 `화물 테스트` 실행:

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests src/lib.rs (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

출력의 세 섹션에는 단위 테스트, 통합 테스트 및 문서 테스트가 포함됩니다. 섹션의 테스트가 실패하면 다음 섹션이 실행되지 않습니다. 예를 들어 단위 테스트가 실패하면 모든 단위 테스트가 통과하는 경우에만 테스트가 실행되기 때문에 통합 및 문서 테스트에 대한 출력이 없습니다.

단위 테스트의 첫 번째 섹션은 우리가 본 것과 동일합니다. 각 단위 테스트에 대한 한 줄(목록 11-12에서 추가한 `internal`이라는 이름의 한 줄)과 단위 테스트에 대한 요약 줄입니다.

통합 테스트 섹션은 `Running tests/integration_test.rs` 줄로 시작합니다. 다음으로, 해당 통합 테스트의 각 테스트 기능에 대한 라인과 `Doc-tests adder` 섹션이 시작되기 직전에 통합 테스트 결과에 대한 요약 라인이 있습니다.

*각 통합 테스트 파일에는 자체 섹션이 있으므로 테스트* 디렉토리 에 더 많은 파일을 추가하면 통합 테스트 섹션이 더 많아집니다.

`cargo test`에 대한 인수로 테스트 함수의 이름을 지정하여 특정 통합 테스트 함수를 계속 실행할 수 있습니다. 특정 통합 테스트 파일에서 모든 테스트를 실행하려면 `cargo test`의 `--test` 인수 뒤에 파일 이름을 사용하십시오.

```bash
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

*이 명령은 tests/integration_test.rs* 파일 의 테스트만 실행합니다.

#### [통합 테스트의 하위 모듈](https://doc.rust-lang.org/book/ch11-03-test-organization.html#submodules-in-integration-tests)

더 많은 통합 테스트를 추가함에 따라 테스트 를 구성하는 데 도움이 되도록 *테스트* 디렉터리 에 더 많은 파일을 만들 수 있습니다. 예를 들어 테스트 중인 기능별로 테스트 기능을 그룹화할 수 있습니다. 앞서 언급했듯이 *테스트* 디렉토리의 각 파일은 별도의 크레이트로 컴파일되며, 이는 최종 사용자가 크레이트를 사용하는 방식을 더 가깝게 모방하기 위해 별도의 범위를 만드는 데 유용합니다. 그러나 이것은 코드를 모듈과 파일로 분리하는 방법에 대해 7장에서 배운 것처럼 *tests 디렉토리의 파일이* *src* 의 파일과 동일한 동작을 공유하지 않는다는 것을 의미합니다.

*테스트* 디렉토리 파일 의 다른 동작은 여러 통합 테스트 파일에서 사용할 헬퍼 함수 집합이 있고 7장의 [`모듈을 다른 파일로 분리`](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html) 섹션의 단계에 따라 압축을 풀려고 할 때 가장 눈에 띕니다. 공통 모듈. *예를 들어, tests/common.rs를* 생성 하고 그 안에 `setup`이라는 함수를 배치하면 여러 테스트 파일의 여러 테스트 함수에서 호출하려는 `setup`에 일부 코드를 추가할 수 있습니다.

파일 이름: tests/common.rs

```rust
pub fn setup() {
    // setup code specific to your library`s tests would go here
}
```

테스트를 다시 실행하면 *common.rs* 파일에 대한 테스트 출력에 새 섹션이 표시됩니다. 이 파일에는 테스트 기능이 포함되어 있지 않으며 어디에서나 `setup` 기능을 호출하지도 않았습니다.

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.89s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/common.rs (target/debug/deps/common-92948b65e88960b4)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-92948b65e88960b4)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

테스트 결과에 `common`이 표시되고 `running 0 tests`가 표시되는 것은 우리가 원하는 것이 아닙니다. 다른 통합 테스트 파일과 일부 코드를 공유하고 싶었습니다.

*테스트 출력에 `common`이 표시되지 않도록 하려면 tests/common.rs 를* 생성하는 대신 *tests/common/mod.rs 를* 생성합니다. 이제 프로젝트 디렉토리는 다음과 같습니다.

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

[이것은 우리가 7장의 `대체 파일 경로`](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#alternate-file-paths) 섹션 에서 언급한 Rust가 이해하는 오래된 명명 규칙이기도 합니다. 이러한 방식으로 파일 이름을 지정하면 Rust는 `공통` 모듈을 통합 테스트 파일로 취급하지 않습니다. *`setup` 기능 코드를 tests/common/mod.rs* 로 이동 하고 *tests/common.rs* 파일을 삭제하면 테스트 출력의 섹션이 더 이상 나타나지 않습니다. *테스트* 디렉토리 의 하위 디렉토리에 있는 파일은 별도의 크레이트로 컴파일되지 않거나 테스트 출력에 섹션이 없습니다.

*tests/common/mod.rs 를* 만든 후에는 모든 통합 테스트 파일에서 모듈로 사용할 수 있습니다. *다음은 tests/integration_test.rs* 의 `it_adds_two` 테스트에서 `setup` 함수를 호출하는 예입니다.

파일 이름: 테스트/integration_test.rs

```rust
use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

`mod common;` 선언은 Listing 7-21에서 보여준 모듈 선언과 동일합니다. 그런 다음 테스트 함수에서 `common::setup()` 함수를 호출할 수 있습니다.

#### [바이너리 크레이트에 대한 통합 테스트](https://doc.rust-lang.org/book/ch11-03-test-organization.html#integration-tests-for-binary-crates)

프로젝트가 *src/main.rs 파일만 포함하고* *src/lib.rs* 파일이 없는 바이너리 크레이트인 경우 *테스트 디렉토리에서 통합 테스트를 생성하고* *src/main* 에 정의된 함수를 가져올 수 없습니다. *.rs* 파일을 `use` 문으로 범위에 넣습니다. 라이브러리 크레이트만이 다른 크레이트가 사용할 수 있는 기능을 노출합니다. 바이너리 크레이트는 자체적으로 실행되도록 되어 있습니다.

*이것은 바이너리를 제공하는 Rust 프로젝트가 src/ lib.rs* 파일 에 있는 논리를 호출하는 간단한 *src/main.rs* 파일을 갖는 이유 중 하나입니다. 이 구조를 사용하여 통합 테스트는 중요한 기능을 사용할 수 있도록 `사용`으로 라이브러리 크레이트를 테스트 *할 수 있습니다.* *중요한 기능이 작동하면 src/main.rs* 파일 에 있는 소량의 코드 도 작동하고 그 소량의 코드는 테스트할 필요가 없습니다.

## [요약](https://doc.rust-lang.org/book/ch11-03-test-organization.html#summary)

Rust의 테스트 기능은 코드를 변경하더라도 예상대로 계속 작동하도록 코드가 작동하는 방법을 지정하는 방법을 제공합니다. 단위 테스트는 라이브러리의 다른 부분을 개별적으로 실행하고 개인 구현 세부 정보를 테스트할 수 있습니다. 통합 테스트는 라이브러리의 많은 부분이 올바르게 함께 작동하는지 확인하고 라이브러리의 공개 API를 사용하여 외부 코드에서 사용하는 것과 동일한 방식으로 코드를 테스트합니다. Rust의 타입 시스템과 소유권 규칙이 일부 종류의 버그를 방지하는 데 도움이 되지만 테스트는 코드의 동작 방식과 관련된 논리 버그를 줄이는 데 여전히 중요합니다.

이 장과 이전 장에서 배운 지식을 결합하여 프로젝트를 진행해 봅시다!

------

# 12

# [I/O 프로젝트: 명령줄 프로그램 구축](https://doc.rust-lang.org/book/ch12-00-an-io-project.html#an-io-project-building-a-command-line-program)

이 장은 지금까지 배운 많은 기술을 요약하고 몇 가지 추가 표준 라이브러리 기능을 탐색합니다. 우리는 파일 및 명령줄 입력/출력과 상호 작용하는 명령줄 도구를 구축하여 현재 가지고 있는 일부 Rust 개념을 연습할 것입니다.

**Rust의 속도, 안전성, 단일 바이너리 출력 및 크로스 플랫폼 지원은 명령줄 도구를 만들기 위한 이상적인 언어로 만들어 주므로 프로젝트를 위해 고전적인 명령줄 검색 도구 `grep`( g** lobally) 의 자체 버전을 만들 것입니다. **정규식** 검색 및 **인쇄** ) **.** _ 가장 간단한 사용 사례에서 `grep`은 지정된 문자열에 대해 지정된 파일을 검색합니다. 이를 위해 `grep`은 파일 경로와 문자열을 인수로 사용합니다. 그런 다음 파일을 읽고 해당 파일에서 문자열 인수를 포함하는 줄을 찾은 다음 해당 줄을 인쇄합니다.

그 과정에서 명령줄 도구가 다른 많은 명령줄 도구에서 사용하는 터미널 기능을 사용하도록 만드는 방법을 보여줍니다. 사용자가 도구의 동작을 구성할 수 있도록 환경 변수의 값을 읽습니다. 또한 표준 출력(`stdout`) 대신 표준 오류 bash 스트림(`stderr`)에 오류 메시지를 인쇄하므로 예를 들어 사용자는 화면에 오류 메시지가 계속 표시되는 동안 성공적인 출력을 파일로 리디렉션할 수 있습니다.

한 Rust 커뮤니티 회원인 Andrew Gallant는 `ripgrep`이라는 완전한 기능을 갖춘 매우 빠른 `grep` 버전을 이미 만들었습니다. 이에 비해 우리 버전은 상당히 단순하지만 이 장에서는 `ripgrep`과 같은 실제 프로젝트를 이해하는 데 필요한 일부 배경 지식을 제공합니다.

`grep` 프로젝트는 지금까지 배운 여러 개념을 결합합니다.

- [코드 구성( 7장](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html) 에서 모듈에 대해 배운 내용 사용 )
- 벡터 및 문자열 사용(컬렉션, [8장](https://doc.rust-lang.org/book/ch08-00-common-collections.html) )
- 오류 처리( [9장](https://doc.rust-lang.org/book/ch09-00-error-handling.html) )
- 적절한 경우 특성 및 수명 사용( [10장](https://doc.rust-lang.org/book/ch10-00-generics.html) )
- 테스트 작성( [11장](https://doc.rust-lang.org/book/ch11-00-testing.html) )

[또한 13](https://doc.rust-lang.org/book/ch13-00-functional-features.html) 장 과 [17](https://doc.rust-lang.org/book/ch17-00-oop.html) 장 에서 자세히 다룰 클로저, 반복자 및 특성 개체를 간략하게 소개합니다.

------

## [명령줄 인수 수락](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#accepting-command-line-arguments)

언제나처럼 `cargo new`로 새 프로젝트를 만들어 봅시다. 시스템에 이미 있을 수 있는 `grep` 도구와 구별하기 위해 프로젝트를 `minigrep`이라고 합니다.

```bash
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

첫 번째 작업은 `minigrep`이 파일 경로와 검색할 문자열이라는 두 개의 명령줄 인수를 받아들이도록 만드는 것입니다. 즉, 우리는 `cargo run`으로 프로그램을 실행할 수 있기를 원합니다. 다음 인수를 나타내는 두 개의 하이픈은 `cargo`가 아닌 프로그램, 검색할 문자열, 검색할 파일의 경로를 나타냅니다. 다음과 같이

```bash
$ cargo run -- searchstring example-filename.txt
```

현재 `cargo new`에 의해 생성된 프로그램은 우리가 제공한 인수를 처리할 수 없습니다. [crates.io](https://crates.io/) 의 일부 기존 라이브러리는 명령줄 인수를 허용하는 프로그램을 작성하는 데 도움이 될 수 있지만 이 개념을 배우는 중이므로 이 기능을 직접 구현해 보겠습니다.

### [인수 값 읽기](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#reading-the-argument-values)

`minigrep`이 전달하는 명령줄 인수의 값을 읽을 수 있도록 하려면 Rust의 표준 라이브러리에서 제공되는 `std::env::args` 함수가 필요합니다. 이 함수는 `minigrep`에 전달된 명령줄 인수의 반복자를 반환합니다. [13장](https://doc.rust-lang.org/book/ch13-00-functional-features.html) 에서 반복자를 완전히 다룰 것입니다. 지금은 반복자에 대한 두 가지 세부 정보만 알면 됩니다. 반복자는 일련의 값을 생성하고 반복자에서 `collect` 메서드를 호출하여 모든 요소를 포함하는 벡터와 같은 컬렉션으로 전환할 수 있습니다. 반복자가 생성합니다.

목록 12-1의 코드를 사용하면 `minigrep` 프로그램이 전달된 명령줄 인수를 읽고 값을 벡터로 수집할 수 있습니다.

파일 이름: src/main.rs

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    dbg!(args);
}
```

Listing 12-1: 명령줄 인수를 벡터로 수집하고 인쇄하기

먼저 `use` 문을 사용하여 `std::env` 모듈을 범위로 가져오면 `args` 기능을 사용할 수 있습니다. `std::env::args` 함수는 두 수준의 모듈에 중첩되어 있습니다. [7장](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths) 에서 논의한 것처럼 원하는 함수가 둘 이상의 모듈에 중첩된 경우 함수가 아닌 상위 모듈을 범위로 가져오도록 선택했습니다. 이렇게 하면 `std::env`의 다른 기능을 쉽게 사용할 수 있습니다. 또한 `use std::env::args`를 추가한 다음 `args`만으로 함수를 호출하는 것보다 모호하지 않습니다. `args`는 현재 모듈에 정의된 함수로 쉽게 오인될 수 있기 때문입니다.

> ### [`args` 함수와 유효하지 않은 유니코드](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#the-args-function-and-invalid-unicode)
>
> 인수에 유효하지 않은 유니코드가 포함된 경우 `std::env::args`가 패닉 상태가 됩니다. 프로그램이 잘못된 유니코드를 포함하는 인수를 수락해야 하는 경우 `std::env::args_os`를 대신 사용하세요. 이 함수는 `String` 값 대신 `OsString` 값을 생성하는 반복자를 반환합니다. 여기에서는 단순화를 위해 `std::env::args`를 사용하기로 했습니다. `OsString` 값은 플랫폼마다 다르고 `String` 값보다 작업하기가 더 복잡하기 때문입니다.

`main`의 첫 번째 줄에서 `env::args`를 호출하고 즉시 `collect`를 사용하여 반복자를 반복자가 생성한 모든 값을 포함하는 벡터로 바꿉니다. `collect` 함수를 사용하여 많은 종류의 컬렉션을 만들 수 있으므로 `args` 유형에 명시적으로 주석을 달아 문자열 벡터를 원한다고 지정합니다. Rust에서는 유형에 주석을 달 필요가 거의 없지만 Rust는 원하는 컬렉션 종류를 추론할 수 없기 때문에 `collect`는 자주 주석을 달아야 하는 함수 중 하나입니다.

마지막으로 디버그 매크로를 사용하여 벡터를 인쇄합니다. 먼저 인수 없이 코드를 실행한 다음 두 개의 인수를 사용하여 코드를 실행해 보겠습니다.

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/minigrep`
[src/main.rs:5] args = [
    `target/debug/minigrep`,
]
$ cargo run -- needle haystack
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 1.57s
     Running `target/debug/minigrep needle haystack`
[src/main.rs:5] args = [
    `target/debug/minigrep`,
    `needle`,
    `haystack`,
]
```

벡터의 첫 번째 값은 바이너리의 이름인 ``target/debug/minigrep``입니다. 이는 프로그램이 실행 시 호출된 이름을 사용하도록 하는 C의 인수 목록 동작과 일치합니다. 메시지에 인쇄하거나 프로그램을 호출하는 데 사용된 명령줄 별칭에 따라 프로그램의 동작을 변경하려는 경우 프로그램 이름에 액세스하는 것이 편리한 경우가 많습니다. 그러나 이 장의 목적을 위해 이를 무시하고 필요한 두 인수만 저장합니다.

### [인수 값을 변수에 저장](https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html#saving-the-argument-values-in-variables)

프로그램은 현재 명령줄 인수로 지정된 값에 액세스할 수 있습니다. 이제 프로그램의 나머지 부분에서 값을 사용할 수 있도록 두 인수의 값을 변수에 저장해야 합니다. 목록 12-2에서 그렇게 합니다.

파일 이름: src/main.rs

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    println!(`Searching for {}`, query);
    println!(`In file {}`, file_path);
}
```

목록 12-2: 쿼리 인수와 파일 경로 인수를 담을 변수 만들기

벡터를 인쇄할 때 보았듯이 프로그램 이름은 `args[0]`에서 벡터의 첫 번째 값을 차지하므로 인덱스 `1`에서 인수를 시작합니다. `minigrep`이 취하는 첫 번째 인수는 우리가 검색하는 문자열이므로 변수 `query`에 첫 번째 인수에 대한 참조를 넣습니다. 두 번째 인수는 파일 경로이므로 `file_path` 변수에 두 번째 인수에 대한 참조를 넣습니다.

코드가 의도한 대로 작동하는지 확인하기 위해 이러한 변수의 값을 일시적으로 인쇄합니다. `test` 및 `sample.txt` 인수를 사용하여 이 프로그램을 다시 실행해 보겠습니다.

```bash
$ cargo run -- test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

좋습니다. 프로그램이 작동 중입니다! 필요한 인수 값이 올바른 변수에 저장되고 있습니다. 나중에 사용자가 인수를 제공하지 않는 경우와 같은 잠재적인 오류 상황을 처리하기 위해 몇 가지 오류 처리를 추가할 것입니다. 지금은 이러한 상황을 무시하고 대신 파일 읽기 기능을 추가하는 작업을 할 것입니다.

------

## [파일 읽기](https://doc.rust-lang.org/book/ch12-02-reading-a-file.html#reading-a-file)

이제 `file_path` 인수에 지정된 파일을 읽는 기능을 추가합니다. 먼저 테스트할 샘플 파일이 필요합니다. 반복되는 단어가 포함된 여러 줄에 걸쳐 적은 양의 텍스트가 있는 파일을 사용합니다. 목록 12-3에는 잘 작동할 Emily Dickinson 시가 있습니다! 프로젝트의 루트 수준에서 *poem.txt* 라는 파일을 만들고 `I'm Nobody!`라는 시를 입력합니다. 누구세요?`

파일명: 시.txt

```text
I`m nobody! Who are you?
Are you nobody, too?
Then there`s a pair of us - don`t tell!
They`d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

Listing 12-3: Emily Dickinson의 시는 좋은 테스트 케이스를 만듭니다.

텍스트가 있는 상태에서 목록 12-4와 같이 *src/main.rs를 편집하고 파일을 읽는 코드를 추가합니다.*

파일 이름: src/main.rs

```rust
use std::env;
use std::fs;

fn main() {
    // --snip--
    println!(`In file {}`, file_path);

    let contents = fs::read_to_string(file_path)
        .expect(`Should have been able to read the file`);

    println!(`With text:\n{contents}`);
}
```

목록 12-4: 두 번째 인수로 지정된 파일의 내용 읽기

먼저 `use` 문을 사용하여 표준 라이브러리의 관련 부분을 가져옵니다. 파일을 처리하려면 `std::fs`가 필요합니다.

`main`에서 새 문 `fs::read_to_string`은 `file_path`를 가져와 해당 파일을 열고 `std::io::Result를 반환합니다.` 파일 내용입니다.

그런 다음 임시 `println!`을 다시 추가합니다. 파일을 읽은 후 `contents` 값을 출력하는 문으로 프로그램이 지금까지 작동하는지 확인할 수 있습니다.

첫 번째 명령줄 인수로 임의의 문자열을 사용하고(아직 검색 부분을 구현하지 않았기 때문에) 두 번째 인수로 *poem.txt 파일을 사용하여 이 코드를 실행해 보겠습니다.*

```bash
$ cargo run -- the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I`m nobody! Who are you?
Are you nobody, too?
Then there`s a pair of us - don`t tell!
They`d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

엄청난! 코드는 파일의 내용을 읽고 인쇄했습니다. 그러나 코드에는 몇 가지 결함이 있습니다. 현재 `main` 기능에는 여러 가지 책임이 있습니다. 일반적으로 각 기능이 하나의 아이디어만 담당하는 경우 기능이 더 명확하고 유지 관리하기 쉽습니다. 다른 문제는 우리가 할 수 있는 만큼 오류를 처리하지 못한다는 것입니다. 프로그램이 아직 작기 때문에 이러한 결함은 큰 문제가 아니지만 프로그램이 커질수록 깔끔하게 수정하기가 더 어려워질 것입니다. 적은 양의 코드를 리팩토링하는 것이 훨씬 쉽기 때문에 프로그램을 개발할 때 초기에 리팩토링을 시작하는 것이 좋습니다. 다음에 그렇게 하겠습니다.

------

## [모듈성 및 오류 처리 개선을 위한 리팩토링](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#refactoring-to-improve-modularity-and-error-handling)

프로그램을 개선하기 위해 프로그램의 구조 및 잠재적 오류를 처리하는 방법과 관련된 네 가지 문제를 수정할 것입니다. 먼저 `main` 함수는 이제 두 가지 작업을 수행합니다. 인수를 구문 분석하고 파일을 읽습니다. 프로그램이 커짐에 따라 `main` 함수가 처리하는 개별 작업의 수가 증가할 것입니다. 함수에 책임이 부여됨에 따라 추론하기, 테스트하기, 부분 중 하나를 손상시키지 않고 변경하기가 더 어려워집니다. 각 기능이 하나의 작업을 담당하도록 기능을 분리하는 것이 가장 좋습니다.

이 문제는 두 번째 문제와도 관련이 있습니다. `query` 및 `file_path`는 우리 프로그램의 구성 변수이지만 `contents`와 같은 변수는 프로그램의 논리를 수행하는 데 사용됩니다. `main`이 길어질수록 더 많은 변수를 범위로 가져와야 합니다. 범위에 변수가 많을수록 각각의 목적을 추적하기가 더 어려워집니다. 목적을 명확히 하기 위해 구성 변수를 하나의 구조로 그룹화하는 것이 가장 좋습니다.

세 번째 문제는 파일 읽기에 실패했을 때 오류 메시지를 인쇄하기 위해 `예상`을 사용했지만 오류 메시지는 `파일을 읽을 수 있어야 했습니다`만 인쇄한다는 것입니다. 파일 읽기는 여러 가지 방법으로 실패할 수 있습니다. 예를 들어 파일이 없거나 파일을 열 수 있는 권한이 없을 수 있습니다. 바로 지금 상황에 관계없이 우리는 모든 것에 대해 동일한 오류 메시지를 인쇄할 것이므로 사용자에게 어떤 정보도 제공하지 않을 것입니다!

넷째, 우리는 `expect`를 반복적으로 사용하여 다른 오류를 처리합니다. 사용자가 충분한 인수를 지정하지 않고 프로그램을 실행하면 문제를 명확하게 설명하지 않는 Rust에서 `인덱스가 범위를 벗어났습니다` 오류가 발생합니다. 모든 오류 처리 코드가 한 곳에 있어서 오류 처리 논리를 변경해야 하는 경우 미래의 관리자가 코드를 참조할 수 있는 곳은 한 곳뿐이면 가장 좋을 것입니다. 모든 오류 처리 코드를 한 곳에 보관하면 최종 사용자에게 의미 있는 메시지를 인쇄할 수 있습니다.

프로젝트를 리팩토링하여 이 네 가지 문제를 해결해 보겠습니다.

### [바이너리 프로젝트에 대한 관심사 분리](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#separation-of-concerns-for-binary-projects)

여러 작업에 대한 책임을 `주요` 기능에 할당하는 조직적 문제는 많은 바이너리 프로젝트에 공통적입니다. 결과적으로 Rust 커뮤니티는 `main`이 커지기 시작할 때 바이너리 프로그램의 개별 관심사를 분할하기 위한 지침을 개발했습니다. 이 프로세스에는 다음 단계가 있습니다.

- 프로그램을 *main.rs* 및 *lib.rs* 로 분할하고 프로그램의 논리를 *lib.rs* 로 이동합니다.
- 명령줄 구문 분석 로직이 작은 한 *main.rs* 에 남아 있을 수 있습니다.
- 명령줄 구문 분석 논리가 복잡해지기 시작하면 *main.rs 에서 추출하여* *lib.rs* 로 이동합니다.

이 프로세스 후에 `main` 기능에 남아 있는 책임은 다음으로 제한되어야 합니다.

- 인수 값을 사용하여 명령줄 구문 분석 논리 호출
- 기타 구성 설정
- *lib.rs* 에서 `실행` 함수 호출
- `실행`이 오류를 반환하는 경우 오류 처리

이 패턴은 관심사 분리에 관한 것입니다. *main.rs* 는 프로그램 실행을 처리하고 *lib.rs는* 당면한 작업의 모든 논리를 처리합니다. *`main` 함수를 직접 테스트할 수 없기 때문에 이 구조를 사용하면 lib.rs* 의 함수로 이동하여 프로그램의 모든 논리를 테스트할 수 있습니다. *main.rs* 에 남아 있는 코드는 읽어서 정확성을 확인할 수 있을 만큼 작을 것입니다. 이 프로세스를 따라 프로그램을 다시 작성해 보겠습니다.

#### [인수 파서 추출](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#extracting-the-argument-parser)

*명령줄 구문 분석 논리를 src/lib.rs* 로 이동하기 위해 준비하기 위해 `main`이 호출하는 함수로 인수를 구문 분석하는 기능을 추출합니다. 목록 12-5는 새 함수 `parse_config`를 호출하는 `main`의 새로운 시작을 보여줍니다. 이 함수는 잠시 동안 *src/main.rs* 에서 정의할 것입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, file_path) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let file_path = &args[2];

    (query, file_path)
}
```

목록 12-5: `main`에서 `parse_config` 함수 추출하기

우리는 여전히 명령줄 인수를 벡터로 수집하고 있지만 인덱스 1의 인수 값을 변수 `query`에 할당하고 인덱스 2의 인수 값을 `main` 함수 내의 변수 `file_path`에 할당하는 대신 전체 벡터를 `parse_config` 함수에 전달합니다. 그런 다음 `parse_config` 함수는 어떤 인수가 어떤 변수에 들어가고 값을 다시 `main`으로 전달하는지 결정하는 논리를 보유합니다. 여전히 `main`에서 `query` 및 `file_path` 변수를 생성하지만 `main`은 더 이상 명령줄 인수와 변수가 일치하는 방법을 결정할 책임이 없습니다.

이 재작업은 우리의 작은 프로그램에 대해 과잉처럼 보일 수 있지만 우리는 작고 점진적인 단계로 리팩토링하고 있습니다. 이렇게 변경한 후 프로그램을 다시 실행하여 인수 구문 분석이 여전히 작동하는지 확인하십시오. 진행 상황을 자주 확인하여 문제가 발생할 때 원인을 파악하는 것이 좋습니다.

#### [구성 값 그룹화](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#grouping-configuration-values)

`parse_config` 기능을 더 개선하기 위해 또 다른 작은 조치를 취할 수 있습니다. 현재 우리는 튜플을 반환하고 있지만 즉시 해당 튜플을 개별 부분으로 다시 나눕니다. 이는 아직 올바른 추상화가 없다는 신호입니다.

개선의 여지가 있음을 보여주는 또 다른 지표는 `parse_config`의 `config` 부분입니다. 이는 우리가 반환하는 두 값이 관련되어 있고 둘 다 하나의 구성 값의 일부임을 의미합니다. 우리는 현재 두 값을 튜플로 그룹화하는 것 외에는 데이터 구조에서 이 의미를 전달하지 않습니다. 대신 두 값을 하나의 구조체에 넣고 각 구조체 필드에 의미 있는 이름을 지정합니다. 이렇게 하면 이 코드의 향후 관리자가 서로 다른 값이 서로 어떻게 관련되어 있고 그 목적이 무엇인지 더 쉽게 이해할 수 있습니다.

목록 12-6은 `parse_config` 기능의 개선 사항을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!(`Searching for {}`, config.query);
    println!(`In file {}`, config.file_path);

    let contents = fs::read_to_string(config.file_path)
        .expect(`Should have been able to read the file`);

    // --snip--
}

struct Config {
    query: String,
    file_path: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let file_path = args[2].clone();

    Config { query, file_path }
}
```

목록 12-6: `Config` 구조체의 인스턴스를 반환하도록 `parse_config` 리팩터링

`query` 및 `file_path`라는 필드를 갖도록 정의된 `Config`라는 구조체를 추가했습니다. 이제 `parse_config`의 서명은 `Config` 값을 반환함을 나타냅니다. `args`의 `문자열` 값을 참조하는 문자열 조각을 반환하는 데 사용했던 `parse_config` 본문에서 이제 소유한 `문자열` 값을 포함하도록 `구성`을 정의합니다. `main`의 `args` 변수는 인수 값의 소유자이며 `parse_config` 함수가 값을 차용하도록 허용하고 있습니다. 즉, `Config`가 `에서 값의 소유권을 가져오려고 하면 Rust의 차용 규칙을 위반하게 됩니다. 인수`.

`문자열` 데이터를 관리할 수 있는 방법에는 여러 가지가 있습니다. 다소 비효율적이지만 가장 쉬운 방법은 값에 대해 `복제` 메서드를 호출하는 것입니다. 이렇게 하면 `구성` 인스턴스가 소유할 데이터의 전체 복사본이 만들어지며 문자열 데이터에 대한 참조를 저장하는 것보다 더 많은 시간과 메모리가 필요합니다. 그러나 데이터를 복제하면 참조의 수명을 관리할 필요가 없기 때문에 코드가 매우 간단해집니다. 이러한 상황에서 단순성을 얻기 위해 약간의 성능을 포기하는 것은 가치 있는 절충안입니다.

> ### [`클론` 사용의 장단점](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#the-trade-offs-of-using-clone)
>
> 런타임 비용 때문에 소유권 문제를 해결하기 위해 `복제`를 사용하는 것을 피하는 경향이 많은 Rustacean 사이에 있습니다. [13장](https://doc.rust-lang.org/book/ch13-00-functional-features.html) 에서는 이러한 유형의 상황에서 보다 효율적인 방법을 사용하는 방법을 배웁니다. 그러나 지금은 이러한 복사본을 한 번만 만들고 파일 경로와 쿼리 문자열이 매우 작기 때문에 진행을 계속하기 위해 몇 개의 문자열을 복사해도 괜찮습니다. 첫 단계에서 코드를 과도하게 최적화하려고 시도하는 것보다 약간 비효율적으로 작동하는 프로그램을 사용하는 것이 좋습니다. Rust에 대한 경험이 많아지면 가장 효율적인 솔루션으로 시작하는 것이 더 쉬울 것이지만 지금은 `복제`라고 부르는 것이 완벽하게 허용됩니다.

`main`을 업데이트하여 `parse_config`에 의해 반환된 `Config` 인스턴스를 `config`라는 변수에 배치하고 이전에 별도의 `query` 및 `file_path` 변수를 사용했던 코드를 업데이트했습니다. 대신 `Config` 구조체의 필드를 사용합니다.

이제 우리의 코드는 `query`와 `file_path`가 서로 관련되어 있고 이들의 목적이 프로그램 작동 방식을 구성하는 것임을 보다 명확하게 전달합니다. 이러한 값을 사용하는 모든 코드는 목적에 맞게 명명된 필드의 `config` 인스턴스에서 값을 찾을 수 있음을 알고 있습니다.

#### [`구성`에 대한 생성자 만들기](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#creating-a-constructor-for-config)

지금까지 `main`에서 명령줄 인수 구문 분석을 담당하는 논리를 추출하여 `parse_config` 함수에 배치했습니다. 이렇게 하면 `query` 및 `file_path` 값이 관련되어 있고 해당 관계가 코드에 전달되어야 한다는 것을 알 수 있습니다. 그런 다음 `query` 및 `file_path`의 관련 목적을 명명하고 `parse_config` 함수에서 값의 이름을 구조체 필드 이름으로 반환할 수 있도록 `Config` 구조체를 추가했습니다.

이제 `parse_config` 함수의 목적은 `Config` 인스턴스를 만드는 것이므로 `parse_config`를 일반 함수에서 `Config` 구조체와 연결된 `new`라는 함수로 변경할 수 있습니다. 이렇게 변경하면 코드가 더 직관적이 됩니다. `String::new`를 호출하여 `String`과 같은 표준 라이브러리 유형의 인스턴스를 만들 수 있습니다. 마찬가지로 `parse_config`를 `Config`와 연결된 `new` 함수로 변경하면 `Config::new`를 호출하여 `Config`의 인스턴스를 만들 수 있습니다. 목록 12-7은 우리가 만들어야 하는 변경 사항을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}
```

목록 12-7: `parse_config`를 `Config::new`로 변경

대신 `Config::new`를 호출하도록 `parse_config`를 호출하던 `main`을 업데이트했습니다. `parse_config`의 이름을 `new`로 변경하고 `new` 기능을 `Config`와 연결하는 `impl` 블록 내로 이동했습니다. 이 코드를 다시 컴파일하여 제대로 작동하는지 확인하십시오.

### [오류 처리 수정](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#fixing-the-error-handling)

이제 우리는 오류 처리를 수정하기 위해 노력할 것입니다. 인덱스 1 또는 인덱스 2에서 `args` 벡터의 값에 액세스하려고 하면 벡터에 세 개 미만의 항목이 포함된 경우 프로그램이 패닉 상태가 됩니다. 인수 없이 프로그램을 실행해 보십시오. 다음과 같이 표시됩니다.

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep`
thread `main` panicked at `index out of bounds: the len is 1 but the index is 1`, src/main.rs:27:21
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`index out of bounds: the len is 1 but the index is 1` 줄은 프로그래머를 위한 오류 메시지입니다. 최종 사용자가 대신 수행해야 하는 작업을 이해하는 데 도움이 되지 않습니다. 지금 수정하겠습니다.

#### [오류 메시지 개선](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#improving-the-error-message)

Listing 12-8에서 인덱스 1과 2에 액세스하기 전에 슬라이스가 충분히 긴지 확인하는 `new` 함수에 검사를 추가합니다. 슬라이스가 충분히 길지 않으면 프로그램이 패닉 상태가 되고 더 나은 오류 메시지를 표시합니다. .

파일 이름: src/main.rs

```rust
    // --snip--
    fn new(args: &[String]) -> Config {
        if args.len() < 3 {
            panic!(`not enough arguments`);
        }
        // --snip--
```

Listing 12-8: 인수 개수 확인 추가

[이 코드는 Listing 9-13에서 작성한 `Guess::new` 함수](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation) 와 유사합니다. `panic!` `value` 인수가 유효한 값의 범위를 벗어난 경우. 여기에서 값의 범위를 확인하는 대신 `args`의 길이가 3 이상이고 함수의 나머지 부분이 이 조건이 충족되었다는 가정하에 작동할 수 있는지 확인하고 있습니다. `args` 항목이 3개 미만이면 이 조건이 참이 되며 `패닉!` 프로그램을 즉시 종료하는 매크로입니다.

`new`에 추가된 몇 줄의 코드를 사용하여 인수 없이 프로그램을 다시 실행하여 이제 오류가 어떻게 보이는지 확인합니다.

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep`
thread `main` panicked at `not enough arguments`, src/main.rs:26:13
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

이 출력이 더 좋습니다. 이제 합리적인 오류 메시지가 표시됩니다. 그러나 사용자에게 제공하고 싶지 않은 외부 정보도 있습니다. Listing 9-13에서 사용한 기법을 여기서 사용하는 것이 최선이 아닐 수도 있습니다. `패닉!` [9장에서 논의한 것처럼](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling) 사용 문제보다 프로그래밍 문제에 더 적합합니다. 대신 9장에서 배운 다른 기술인 성공 또는 오류를 나타내는 [`결과` 반환을 사용합니다.](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html)

#### [`패닉!`을 호출하는 대신 `결과` 반환](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#returning-a-result-instead-of-calling-panic)

대신 성공한 경우 `구성` 인스턴스를 포함하고 오류 경우 문제를 설명하는 `결과` 값을 반환할 수 있습니다. 또한 많은 프로그래머가 `새로운` 함수는 절대 실패하지 않을 것으로 기대하기 때문에 함수 이름을 `new`에서 `build`로 변경할 것입니다. `Config::build`가 `main`과 통신할 때 `Result` 유형을 사용하여 문제가 있음을 알릴 수 있습니다. 그런 다음 `메인`을 변경하여 `패닉!`을 호출하는 `스레드 `메인`` 및 `RUST_BACKTRACE`에 대한 주변 텍스트 없이 `Err` 변형을 사용자를 위한 보다 실용적인 오류로 변환할 수 있습니다. 원인.

목록 12-9는 `Config::build`라고 부르는 함수의 반환 값과 `결과`를 반환하는 데 필요한 함수 본문에 필요한 변경 사항을 보여줍니다. 다음 목록에서 수행할 `main`도 업데이트할 때까지 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &`static str> {
        if args.len() < 3 {
            return Err(`not enough arguments`);
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

목록 12-9: `Config::build`에서 `결과` 반환

`build` 함수는 성공 사례의 `Config` 인스턴스와 오류 사례의 `&`static str`이 있는 `Result`를 반환합니다. 오류 값은 항상 ``정적` 수명을 갖는 문자열 리터럴입니다.

함수 본문에서 `패닉!`을 호출하는 대신 두 가지를 변경했습니다. 사용자가 충분한 인수를 전달하지 않으면 `Err` 값을 반환하고 `Config` 반환 값을 `Ok`로 래핑했습니다. 이러한 변경으로 인해 함수는 새로운 유형 서명을 준수합니다.

`Config::build`에서 `Err` 값을 반환하면 `main` 함수가 `build` 함수에서 반환된 `Result` 값을 처리하고 오류 사례에서 보다 깔끔하게 프로세스를 종료할 수 있습니다.

#### [`Config::build` 호출 및 오류 처리](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#calling-configbuild-and-handling-errors)

오류 사례를 처리하고 사용자에게 친숙한 메시지를 인쇄하려면 Listing 12-10과 같이 `Config::build`에서 반환되는 `결과`를 처리하도록 `main`을 업데이트해야 합니다. 또한 `panic!`에서 벗어나 0이 아닌 오류 코드로 명령줄 도구를 종료하는 책임을 집니다. 대신 손으로 구현하십시오. 0이 아닌 종료 상태는 프로그램이 오류 상태로 종료되었음을 프로그램을 호출한 프로세스에 알리는 규칙입니다.

파일 이름: src/main.rs

```rust
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!(`Problem parsing arguments: {err}`);
        process::exit(1);
    });

    // --snip--
```

Listing 12-10: `Config` 빌드에 실패하면 오류 코드와 함께 종료

이 목록에서는 아직 자세히 다루지 않은 방법인 `unwrap_or_else`를 사용했는데, 이는 표준 라이브러리에 의해 `Result<T, E>`에 정의되어 있습니다. `unwrap_or_else`를 사용하면 `panic!`이 아닌 사용자 정의를 정의할 수 있습니다. 오류 처리. `Result`가 `Ok` 값인 경우 이 메서드의 동작은 `unwrap`과 유사합니다. `Ok`가 래핑되는 내부 값을 반환합니다. 그러나 값이 `Err` 값이면 이 메서드 는 우리가 정의하고 `unwrap_or_else`에 인수로 전달하는 익명 함수인 *클로저* 의 코드를 호출합니다. [클로저에 대해서는 13장](https://doc.rust-lang.org/book/ch13-00-functional-features.html) 에서 자세히 다룰 것입니다.. 지금은 `unwrap_or_else`가 `Err`의 내부 값을 전달한다는 사실만 알면 됩니다. 이 경우 Listing 12-9에서 추가한 정적 문자열 ``인수가 충분하지 않음``이 클로저에 전달됩니다. 수직 파이프 사이에 나타나는 인수 `err`에서. 클로저의 코드는 실행될 때 `err` 값을 사용할 수 있습니다.

표준 라이브러리의 `프로세스`를 범위로 가져오기 위해 새로운 `use` 줄을 추가했습니다. 오류 케이스에서 실행될 클로저의 코드는 단 두 줄입니다. `err` 값을 인쇄한 다음 `process::exit`를 호출합니다. `process::exit` 함수는 프로그램을 즉시 중지하고 종료 상태 코드로 전달된 번호를 반환합니다. 이것은 목록 12-8에서 사용한 `패닉!` 기반 처리와 유사하지만 더 이상 모든 추가 출력을 얻지 못합니다. 해 보자:

```bash
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

엄청난! 이 출력은 사용자에게 훨씬 친숙합니다.

### [`main`에서 논리 추출](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#extracting-logic-from-main)

이제 구성 구문 분석 리팩터링을 마쳤으므로 프로그램의 논리로 전환해 보겠습니다. [`바이너리 프로젝트에 대한 관심사 분리`](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#separation-of-concerns-for-binary-projects) 에서 언급한 것처럼 구성 설정 또는 오류 처리와 관련되지 않은 현재 `main` 함수에 있는 모든 논리를 보유할 `run`이라는 함수를 추출합니다. 완료되면 `main`은 간결하고 검사를 통해 쉽게 확인할 수 있으며 다른 모든 논리에 대한 테스트를 작성할 수 있습니다.

Listing 12-11은 추출된 `run` 함수를 보여줍니다. 지금은 함수 추출을 조금씩 점진적으로 개선하고 있습니다. *우리는 여전히 src/main.rs* 에서 함수를 정의하고 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    // --snip--

    println!(`Searching for {}`, config.query);
    println!(`In file {}`, config.file_path);

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect(`Should have been able to read the file`);

    println!(`With text:\n{contents}`);
}

// --snip--
```

Listing 12-11: 나머지 프로그램 로직을 포함하는 `run` 함수 추출

이제 `run` 함수에는 파일 읽기부터 시작하여 `main`의 나머지 모든 논리가 포함됩니다. `run` 함수는 `Config` 인스턴스를 인수로 사용합니다.

#### [`run` 함수에서 오류 반환](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#returning-errors-from-the-run-function)

`run` 함수로 분리된 나머지 프로그램 로직으로 목록 12-9의 `Config::build`에서 했던 것처럼 오류 처리를 개선할 수 있습니다. `expect`를 호출하여 프로그램이 패닉 상태가 되도록 하는 대신 `run` 함수는 무언가 잘못되었을 때 `Result<T, E>`를 반환합니다. 이렇게 하면 사용자에게 친숙한 방식으로 오류를 처리하는 논리를 `기본`으로 더욱 통합할 수 있습니다. 목록 12-12는 `run`의 서명과 본문에 필요한 변경 사항을 보여줍니다.

파일 이름: src/main.rs

```rust
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    println!(`With text:\n{contents}`);

    Ok(())
}
```

목록 12-12: `결과`를 반환하도록 `실행` 함수 변경

여기에 세 가지 중요한 변경 사항이 있습니다. 먼저 `run` 함수의 리턴 타입을 `Result<(), Box`로 변경했습니다.>`. 이 함수는 이전에 단위 유형 `()`을 반환했으며 `Ok`의 경우 반환된 값으로 유지합니다.

오류 유형의 경우 *특성 개체* `Box`를 사용했습니다.` (그리고 우리는 `std::error::Error`를 `use` 문이 맨 위에 있는 범위로 가져왔습니다.) 우리는 [17장](https://doc.rust-lang.org/book/ch17-00-oop.html) 에서 특성 개체를 다룰 것입니다. 지금은 `Box` 는 함수가 `오류` 특성을 구현하는 유형을 반환하지만 반환 값이 어떤 특정 유형인지 지정할 필요가 없음을 의미합니다. 이는 다른 오류에서 다른 유형일 수 있는 오류 값을 반환할 수 있는 유연성을 제공합니다. `dyn` 키워드는 `dynamic`의 줄임말입니다.

둘째, `?` 대신 `expect` 호출을 제거했습니다. [9장](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator) 에서 언급한 연산자입니다. `패닉!` 오류 시 `?` 호출자가 처리할 현재 함수의 오류 값을 반환합니다.

셋째, `run` 함수는 이제 성공 사례에서 `Ok` 값을 반환합니다. 서명에서 `run` 함수의 성공 유형을 `()`로 선언했습니다. 즉, 단위 유형 값을 `Ok` 값으로 래핑해야 합니다. 이 `Ok(())` 구문은 처음에는 약간 이상하게 보일 수 있지만 이와 같이 `()`를 사용하는 것은 우리가 부작용 때문에 `run`을 호출하고 있음을 나타내는 관용적인 방법입니다. 필요한 값을 반환하지 않습니다.

이 코드를 실행하면 컴파일되지만 경고가 표시됩니다.

```bash
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
warning: unused `Result` that must be used
  --> src/main.rs:19:5
   |
19 |     run(config);
   |     ^^^^^^^^^^^
   |
   = note: this `Result` may be an `Err` variant, which should be handled
   = note: `#[warn(unused_must_use)]` on by default

warning: `minigrep` (bin `minigrep`) generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.71s
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I`m nobody! Who are you?
Are you nobody, too?
Then there`s a pair of us - don`t tell!
They`d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

러스트는 코드가 `결과` 값을 무시하고 `결과` 값이 오류가 발생했음을 나타낼 수 있다고 알려줍니다. 그러나 우리는 오류가 있는지 여부를 확인하지 않으며 컴파일러는 아마도 여기에 오류 처리 코드가 있어야 한다고 상기시켜줍니다! 이제 그 문제를 바로잡자.

#### [`main`의 `run`에서 반환된 오류 처리](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#handling-errors-returned-from-run-in-main)

Listing 12-10에서 `Config::build`에 사용한 것과 비슷한 기술을 사용하여 오류를 확인하고 처리하지만 약간의 차이가 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    // --snip--

    println!(`Searching for {}`, config.query);
    println!(`In file {}`, config.file_path);

    if let Err(e) = run(config) {
        println!(`Application error: {e}`);
        process::exit(1);
    }
}
```

우리는 `unwrap_or_else`가 아닌 `if let`을 사용하여 `run`이 `Err` 값을 반환하는지 여부를 확인하고 반환하는 경우 `process::exit(1)`을 호출합니다. `run` 함수는 `Config::build`가 `Config` 인스턴스를 반환하는 것과 같은 방식으로 `언래핑`하려는 값을 반환하지 않습니다. 성공 사례에서 `run`은 `()`를 반환하기 때문에 우리는 오류 감지에만 관심이 있으므로 래핑되지 않은 값을 반환하기 위해 `unwrap_or_else`가 필요하지 않으며 `()`만 반환됩니다.

`if let` 및 `unwrap_or_else` 함수의 본문은 두 경우 모두 동일합니다. 오류를 인쇄하고 종료합니다.

### [코드를 라이브러리 크레이트로 분할](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#splitting-code-into-a-library-crate)

우리의 `minigrep` 프로젝트는 지금까지 괜찮아 보입니다! *이제 src/main.rs* 파일을 분할 하고 일부 코드를 *src/lib.rs* 파일에 넣습니다. 그렇게 하면 코드를 테스트하고 더 적은 책임으로 *src/main.rs* 파일을 가질 수 있습니다.

`main` 함수가 아닌 모든 코드를 *src/main.rs 에서* *src/lib.rs* 로 이동해 보겠습니다.

- `실행` 함수 정의
- 관련 `사용` 진술
- `구성`의 정의
- `Config::build` 함수 정의

*src/lib.rs* 의 내용은 목록 12-13에 표시된 서명을 가져야 합니다(간결성을 위해 함수 본문을 생략했습니다). Listing 12-14에서 *src/main.rs를* 수정할 때까지는 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
use std::error::Error;
use std::fs;

pub struct Config {
    pub query: String,
    pub file_path: String,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &`static str> {
        // --snip--
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    // --snip--
}
```

*목록 12-13: `Config` 및 `run`을 src/lib.rs* 로 이동

우리는 `pub` 키워드를 `Config`, 해당 필드 및 `build` 메서드, `run` 함수에 자유롭게 사용했습니다. 이제 테스트할 수 있는 공개 API가 있는 라이브러리 크레이트가 생겼습니다!

이제 목록 12-14에 표시된 것처럼 *src/lib.rs 로 이동한 코드를* *src/main.rs* 의 바이너리 크레이트 범위로 가져와야 합니다.

파일 이름: src/main.rs

```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
    // --snip--
    if let Err(e) = minigrep::run(config) {
        // --snip--
    }
}
```

*목록 12-14: src/main.rs* 에서 `minigrep` 라이브러리 크레이트 사용하기

`use minigrep::Config` 행을 추가하여 라이브러리 크레이트의 `Config` 유형을 바이너리 크레이트의 범위로 가져오고 `run` 함수 앞에 크레이트 이름을 붙입니다. 이제 모든 기능이 연결되고 작동해야 합니다. `cargo run`으로 프로그램을 실행하고 모든 것이 올바르게 작동하는지 확인하십시오.

아휴! 그것은 많은 일이었지만 우리는 미래의 성공을 위해 스스로를 준비했습니다. 이제 오류를 처리하기가 훨씬 더 쉬워졌으며 코드를 더 모듈화했습니다. 거의 모든 작업은 여기서부터 *src/lib.rs* 에서 수행됩니다.

이전 코드로는 어려웠지만 새 코드로는 쉬운 작업을 수행하여 이 새로 발견된 모듈성을 활용해 보겠습니다. 몇 가지 테스트를 작성하겠습니다!

------

## [테스트 주도 개발로 라이브러리의 기능 개발](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#developing-the-librarys-functionality-with-test-driven-development)

*이제 논리를 src/lib.rs* 로 추출 하고 인수 수집 및 오류 처리를 *src/main.rs* 에 남겨두었 으므로 코드의 핵심 기능에 대한 테스트를 작성하는 것이 훨씬 쉬워졌습니다. 명령줄에서 바이너리를 호출하지 않고도 다양한 인수로 함수를 직접 호출하고 반환 값을 확인할 수 있습니다.

이 섹션에서는 다음 단계에 따라 TDD(테스트 기반 개발) 프로세스를 사용하여 `minigrep` 프로그램에 검색 논리를 추가합니다.

1. 실패하는 테스트를 작성하고 실행하여 예상한 이유로 실패하는지 확인하십시오.
2. 새 테스트를 통과하기에 충분한 코드만 작성하거나 수정하십시오.
3. 방금 추가하거나 변경한 코드를 리팩터링하고 테스트가 계속 통과하는지 확인하십시오.
4. 1단계부터 반복!

TDD는 소프트웨어를 작성하는 여러 방법 중 하나일 뿐이지만 코드 설계를 추진하는 데 도움이 될 수 있습니다. 테스트를 통과하는 코드를 작성하기 전에 테스트를 작성하면 프로세스 전체에서 높은 테스트 범위를 유지하는 데 도움이 됩니다.

파일 내용에서 쿼리 문자열을 실제로 검색하고 쿼리와 일치하는 줄 목록을 생성하는 기능의 구현을 테스트 드라이브할 것입니다. `검색`이라는 기능에 이 기능을 추가할 것입니다.

### [실패한 테스트 작성](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#writing-a-failing-test)

더 이상 필요하지 않으므로 `println!` 프로그램의 동작을 확인하는 데 사용한 *src/lib.rs* 및 *src/main.rs* 의 명령문 . 그런 다음 *src/lib.rs 에서* [11장](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#the-anatomy-of-a-test-function) 에서 했던 것처럼 테스트 기능이 있는 `tests` 모듈을 추가합니다. 테스트 기능은 `검색` 기능에 원하는 동작을 지정합니다. 검색할 쿼리와 텍스트를 사용하고 쿼리를 포함하는 텍스트의 줄만 반환합니다. Listing 12-15는 아직 컴파일되지 않은 이 테스트를 보여줍니다.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = `duct`;
        let contents = `\
Rust:
safe, fast, productive.
Pick three.`;

        assert_eq!(vec![`safe, fast, productive.`], search(query, contents));
    }
}
```

목록 12-15: `검색` 기능에 대한 실패한 테스트 만들기

이 테스트는 문자열 ``duct``를 검색합니다. 우리가 찾고 있는 텍스트는 세 줄이며, 그 중 하나만 ``duct``를 포함합니다(여는 큰따옴표 뒤의 백슬래시가 Rust에게 이 문자열 리터럴의 내용 시작 부분에 개행 문자를 넣지 말라고 지시한다는 점에 유의하십시오). 우리는 `검색` 함수에서 반환된 값에 우리가 기대하는 줄만 포함되어 있다고 주장합니다.

우리는 아직 이 테스트를 실행할 수 없으며 테스트가 컴파일되지도 않았기 때문에 테스트가 실패하는 것을 볼 수 없습니다. `검색` 기능은 아직 존재하지 않습니다! TDD 원칙에 따라 목록 12-16과 같이 항상 빈 벡터를 반환하는 `검색` 함수의 정의를 추가하여 테스트를 컴파일하고 실행하는 데 충분한 코드를 추가할 것입니다. 그러면 빈 벡터가 `안전하고 빠르며 생산적`이라는 줄을 포함하는 벡터와 일치하지 않기 때문에 테스트가 컴파일되고 실패해야 합니다.

파일 이름: src/lib.rs

```rust
pub fn search<`a>(query: &str, contents: &`a str) -> Vec<&`a str> {
    vec![]
}
```

Listing 12-16: 테스트가 컴파일되도록 `검색` 기능을 충분히 정의하기

`search`의 서명에서 명시적인 수명 ``a`를 정의하고 `contents` 인수 및 반환 값과 함께 해당 수명을 사용해야 한다는 점에 유의하십시오. 10장에서 수명 매개변수가 연결된 인수 수명을 지정한다는 것을 기억 [하십시오](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html) . 반환 값의 수명 이 경우 반환된 벡터에는 `query` 인수가 아니라 `contents` 인수의 슬라이스를 참조하는 문자열 슬라이스가 포함되어야 함을 나타냅니다.

즉, `contents` 인수에서 `search` 함수로 데이터가 전달되는 한 `search` 함수에 의해 반환된 데이터가 계속 유지될 것이라고 Rust에 알립니다. 이건 중요하다! 참조가 유효하려면 슬라이스 *에서* 참조하는 데이터가 유효해야 합니다. 컴파일러가 `콘텐츠`가 아닌 `쿼리`의 문자열 조각을 만들고 있다고 가정하면 안전 검사를 잘못 수행합니다.

수명 주석을 잊어버리고 이 함수를 컴파일하려고 하면 다음 오류가 발생합니다.

```bash
$ cargo build
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
error[E0106]: missing lifetime specifier
  --> src/lib.rs:28:51
   |
28 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
   |                      ----            ----         ^ expected named lifetime parameter
   |
   = help: this function`s return type contains a borrowed value, but the signature does not say whether it is borrowed from `query` or `contents`
help: consider introducing a named lifetime parameter
   |
28 | pub fn search<`a>(query: &`a str, contents: &`a str) -> Vec<&`a str> {
   |              ++++         ++                 ++              ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `minigrep` due to previous error
```

Rust는 우리에게 필요한 두 인수 중 어느 것이 필요한지 알 수 없으므로 명시적으로 알려줄 필요가 있습니다. `contents`는 모든 텍스트를 포함하는 인수이고 일치하는 해당 텍스트의 일부를 반환하기를 원하기 때문에 `contents`가 수명 구문을 사용하여 반환 값에 연결되어야 하는 인수임을 알고 있습니다.

다른 프로그래밍 언어에서는 서명에서 값을 반환하기 위해 인수를 연결할 필요가 없지만 이 방법은 시간이 지나면서 더 쉬워질 것입니다. [이 예제를 10장의 `수명이 있는 참조 유효성 검사`](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes) 섹션 과 비교할 수 있습니다.

이제 테스트를 실행해 보겠습니다.

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 0.97s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 1 test
test tests::one_result ... FAILED

failures:

---- tests::one_result stdout ----
thread `tests::one_result` panicked at `assertion failed: `(left == right)`
  left: `[`safe, fast, productive.`]`,
 right: `[]``, src/lib.rs:44:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::one_result

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

좋습니다. 예상한 대로 테스트가 실패했습니다. 시험을 통과하자!

### [테스트 통과를 위한 코드 작성](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#writing-code-to-pass-the-test)

현재 테스트는 항상 빈 벡터를 반환하기 때문에 실패하고 있습니다. 이를 수정하고 `검색`을 구현하려면 프로그램에서 다음 단계를 따라야 합니다.

- 내용의 각 행을 반복합니다.
- 행에 쿼리 문자열이 포함되어 있는지 확인하십시오.
- 그렇다면 반환하는 값 목록에 추가하십시오.
- 그렇지 않으면 아무것도 하지 마십시오.
- 일치하는 결과 목록을 반환합니다.

줄을 반복하는 것부터 시작하여 각 단계를 살펴보겠습니다.

#### [`lines` 메서드를 사용하여 줄 반복](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#iterating-through-lines-with-the-lines-method)

Rust는 목록 12-17에 표시된 대로 작동하는 편리한 `lines`라는 문자열의 라인별 반복을 처리하는 유용한 방법을 가지고 있습니다. 이것은 아직 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
pub fn search<`a>(query: &str, contents: &`a str) -> Vec<&`a str> {
    for line in contents.lines() {
        // do something with line
    }
}
```

Listing 12-17: `contents`의 각 줄 반복

`lines` 메서드는 반복자를 반환합니다. [13장](https://doc.rust-lang.org/book/ch13-02-iterators.html) 에서 이터레이터에 대해 자세히 이야기하겠지만 [목록 3-5](https://doc.rust-lang.org/book/ch03-05-control-flow.html#looping-through-a-collection-with-for) 에서 이터레이터를 사용하는 방법을 보았던 것을 기억하십시오. 여기서 우리는 컬렉션의 각 항목에 대해 일부 코드를 실행하기 위해 이터레이터와 함께 `for` 루프를 사용했습니다. .

#### [쿼리에 대한 각 줄 검색](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#searching-each-line-for-the-query)

다음으로 현재 줄에 쿼리 문자열이 포함되어 있는지 확인합니다. 다행스럽게도 문자열에는 이 작업을 수행하는 `contains`라는 유용한 메서드가 있습니다! Listing 12-18에 표시된 것처럼 `search` 함수의 `contains` 메서드에 대한 호출을 추가합니다. 이것은 아직 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
pub fn search<`a>(query: &str, contents: &`a str) -> Vec<&`a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

Listing 12-18: 줄에 `query`의 문자열이 포함되어 있는지 확인하는 기능 추가

현재 우리는 기능을 구축하고 있습니다. 컴파일하려면 함수 서명에서 지시한 대로 본문에서 값을 반환해야 합니다.

#### [일치하는 줄 저장](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#storing-matching-lines)

이 기능을 완료하려면 반환하려는 일치하는 줄을 저장할 방법이 필요합니다. 이를 위해 `for` 루프 전에 가변 벡터를 만들고 `push` 메서드를 호출하여 벡터에 `선`을 저장할 수 있습니다. `for` 루프 다음에 목록 12-19에 표시된 것처럼 벡터를 반환합니다.

파일 이름: src/lib.rs

```rust
pub fn search<`a>(query: &str, contents: &`a str) -> Vec<&`a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

Listing 12-19: 반환할 수 있도록 일치하는 줄 저장

이제 `search` 함수는 `query`를 포함하는 줄만 반환해야 하며 테스트는 통과해야 합니다. 테스트를 실행해 보겠습니다.

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 1.22s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 1 test
test tests::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

우리의 테스트는 통과되었으므로 작동한다는 것을 알고 있습니다!

이 시점에서 동일한 기능을 유지하기 위해 테스트 통과를 유지하면서 검색 기능 구현을 리팩토링할 수 있는 기회를 고려할 수 있습니다. 검색 기능의 코드는 그리 나쁘지는 않지만 반복자의 몇 가지 유용한 기능을 활용하지 않습니다. [13장](https://doc.rust-lang.org/book/ch13-02-iterators.html) 에서 이 예제로 돌아가 반복자를 자세히 살펴보고 이를 개선하는 방법을 살펴보겠습니다.

#### [`실행` 기능에서 `검색` 기능 사용](https://doc.rust-lang.org/book/ch12-04-testing-the-librarys-functionality.html#using-the-search-function-in-the-run-function)

이제 `검색` 기능이 작동하고 테스트되었으므로 `실행` 기능에서 `검색`을 호출해야 합니다. `config.query` 값과 `run`이 파일에서 읽는 `contents`를 `search` 기능에 전달해야 합니다. 그런 다음 `run`은 `search`에서 반환된 각 줄을 인쇄합니다.

파일 이름: src/lib.rs

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    for line in search(&config.query, &contents) {
        println!(`{line}`);
    }

    Ok(())
}
```

우리는 여전히 `for` 루프를 사용하여 `검색`에서 각 줄을 반환하고 인쇄합니다.

이제 전체 프로그램이 작동해야 합니다! 먼저 Emily Dickinson의 시 `개구리`에서 정확히 한 줄을 반환해야 하는 단어로 시도해 봅시다.

```bash
$ cargo run -- frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38s
     Running `target/debug/minigrep frog poem.txt`
How public, like a frog
```

시원한! 이제 `body`와 같이 여러 줄과 일치하는 단어를 사용해 보겠습니다.

```bash
$ cargo run -- body poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep body poem.txt`
I`m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

그리고 마지막으로 `monomorphization`과 같이 시 어디에도 없는 단어를 검색할 때 어떤 행도 얻지 않도록 합시다.

```bash
$ cargo run -- monomorphization poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep monomorphization poem.txt`
```

훌륭한! 우리는 고전적인 도구의 자체 미니 버전을 구축했으며 응용 프로그램을 구성하는 방법에 대해 많은 것을 배웠습니다. 또한 파일 입력 및 출력, 수명, 테스트 및 명령줄 구문 분석에 대해 조금 배웠습니다.

이 프로젝트를 마무리하기 위해 명령줄 프로그램을 작성할 때 유용한 환경 변수로 작업하는 방법과 표준 오류로 인쇄하는 방법을 간략하게 설명합니다.

------

## [환경 변수 작업](https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html#working-with-environment-variables)

사용자가 환경 변수를 통해 설정할 수 있는 대소문자를 구분하지 않는 검색 옵션을 추가하여 `minigrep`을 개선할 것입니다. 이 기능을 명령줄 옵션으로 만들고 사용자가 적용하기를 원할 때마다 입력하도록 요구할 수 있지만 대신 환경 변수로 만들어 사용자가 환경 변수를 한 번 설정하고 모든 검색에서 대소문자를 구분하지 않도록 할 수 있습니다. 해당 터미널 세션에서.

### [대소문자를 구분하지 않는 `검색` 기능에 대한 실패한 테스트 작성](https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html#writing-a-failing-test-for-the-case-insensitive-search-function)

먼저 환경 변수에 값이 있을 때 호출되는 새로운 `search_case_insensitive` 함수를 추가합니다. 계속해서 TDD 프로세스를 따를 것이므로 첫 번째 단계는 실패한 테스트를 다시 작성하는 것입니다. 새로운 `search_case_insensitive` 함수에 대한 새 테스트를 추가하고 이전 테스트의 이름을 `one_result`에서 `case_sensitive`로 변경하여 Listing 12-20과 같이 두 테스트 간의 차이점을 명확히 할 것입니다.

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = `duct`;
        let contents = `\
Rust:
safe, fast, productive.
Pick three.
Duct tape.`;

        assert_eq!(vec![`safe, fast, productive.`], search(query, contents));
    }

    #[test]
    fn case_insensitive() {
        let query = `rUsT`;
        let contents = `\
Rust:
safe, fast, productive.
Pick three.
Trust me.`;

        assert_eq!(
            vec![`Rust:`, `Trust me.`],
            search_case_insensitive(query, contents)
        );
    }
}
```

목록 12-20: 추가하려는 대소문자를 구분하지 않는 함수에 대한 새로운 실패 테스트 추가

이전 테스트의 `내용`도 편집했습니다. 대소문자 구분 방식으로 검색할 때 ``duct`` 검색어와 일치하지 않아야 하는 대문자 D를 사용하여 ``Duct tape.``이라는 텍스트가 포함된 새 줄을 추가했습니다. 이러한 방식으로 기존 테스트를 변경하면 이미 구현한 대소문자 구분 검색 기능이 실수로 중단되지 않도록 할 수 있습니다. 이 테스트는 지금 통과해야 하며 대소문자를 구분하지 않는 검색 작업을 진행하는 동안 계속 통과해야 합니다.

*대소문자를 구분하지 않는* 검색 에 대한 새 테스트는 ``rUsT``를 쿼리로 사용합니다. 추가하려는 `search_case_insensitive` 함수에서 쿼리 ``rUsT``는 대문자 R이 있는 ``Rust:``를 포함하는 줄과 일치하고 ``Trust me.`` 줄과 일치해야 합니다. 쿼리와 대소문자가 다릅니다. 이것은 실패한 테스트이며 아직 `search_case_insensitive` 함수를 정의하지 않았기 때문에 컴파일에 실패합니다. 테스트 컴파일 및 실패를 확인하기 위해 Listing 12-16의 `search` 함수에 대해 수행한 방식과 유사하게 항상 빈 벡터를 반환하는 스켈레톤 구현을 자유롭게 추가하십시오.

### [`search_case_insensitive` 기능 구현](https://doc.rust-lang.org/book/ch12-05-working-with-environment-variables.html#implementing-the-search_case_insensitive-function)

목록 12-21에 표시된 `search_case_insensitive` 기능은 `search` 기능과 거의 동일합니다. 유일한 차이점은 `query`와 각 `line`을 소문자로 사용하여 입력 인수의 대소문자가 무엇이든 라인에 쿼리가 포함되어 있는지 여부를 확인할 때 대소문자가 동일하다는 것입니다.

파일 이름: src/lib.rs

```rust
pub fn search_case_insensitive<`a>(
    query: &str,
    contents: &`a str,
) -> Vec<&`a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

목록 12-21: `search_case_insensitive` 함수를 정의하여 쿼리와 줄을 비교하기 전에 소문자로 만들기

먼저 `query` 문자열을 소문자로 만들고 같은 이름의 숨겨진 변수에 저장합니다. 쿼리에서 `to_lowercase`를 호출하는 것이 필요하므로 사용자의 쿼리가 ``rust``, ``RUST``, ``Rust`` 또는 ``rUsT``인지 여부에 관계없이 쿼리를 다음과 같이 처리합니다. 그것은 ``녹``이었고 케이스에 둔감했습니다. `to_lowercase`는 기본 유니코드를 처리하지만 100% 정확하지는 않습니다. 실제 응용 프로그램을 작성하고 있다면 여기에서 좀 더 많은 작업을 수행하고 싶지만 이 섹션은 유니코드가 아닌 환경 변수에 관한 것이므로 여기에서 그대로 두겠습니다.

`to_lowercase`를 호출하면 기존 데이터를 참조하는 대신 새 데이터가 생성되기 때문에 `query`는 이제 문자열 조각이 아니라 `문자열`입니다. 예를 들어 쿼리가 ``rUsT``라고 가정해 보겠습니다. 해당 문자열 조각에는 소문자 `u` 또는 `t`가 포함되어 있지 않으므로 ``rust`를 포함하는 새 `문자열`을 할당해야 합니다. `. 이제 `contains` 메서드에 `query`를 인수로 전달할 때 `contains`의 서명이 문자열 슬라이스를 취하도록 정의되어 있으므로 앰퍼샌드를 추가해야 합니다.

다음으로 각 `줄`에 `to_lowercase`에 대한 호출을 추가하여 모든 문자를 소문자로 만듭니다. 이제 `line`과 `query`를 소문자로 변환했으므로 쿼리의 대소문자에 상관없이 일치하는 항목을 찾을 수 있습니다.

이 구현이 테스트를 통과하는지 봅시다:

```bash
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 1.33s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 2 tests
test tests::case_insensitive ... ok
test tests::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests minigrep

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

엄청난! 통과했습니다. 이제 `run` 함수에서 새로운 `search_case_insensitive` 함수를 호출해 보겠습니다. 먼저 `Config` 구조체에 구성 옵션을 추가하여 대소문자 구분 검색과 대소문자 구분 안 함 검색 간에 전환합니다. 이 필드를 추가하면 아직 이 필드를 초기화하지 않았기 때문에 컴파일러 오류가 발생합니다.

파일 이름: src/lib.rs

```rust
pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}
```

부울을 포함하는 `ignore_case` 필드를 추가했습니다. 다음으로 `ignore_case` 필드의 값을 확인하고 `search` 함수를 호출할지 `search_case_insensitive` 함수를 호출할지 결정하는 데 사용하는 `run` 함수가 필요합니다. Listing 12-22에서 볼 수 있습니다. 아직 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!(`{line}`);
    }

    Ok(())
}
```

목록 12-22: `config.ignore_case`의 값을 기반으로 `search` 또는 `search_case_insensitive` 호출

마지막으로 환경 변수를 확인해야 합니다. *환경 변수 작업을 위한 함수는 표준 라이브러리의 `env` 모듈에 있으므로 해당 모듈을 src/lib.rs* 맨 위에 있는 범위로 가져옵니다. 그런 다음 목록 12-23과 같이 `IGNORE_CASE`라는 환경 변수에 대해 값이 설정되었는지 확인하기 위해 `env` 모듈의 `var` 함수를 사용할 것입니다.

파일 이름: src/lib.rs

```rust
use std::env;
// --snip--

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &`static str> {
        if args.len() < 3 {
            return Err(`not enough arguments`);
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var(`IGNORE_CASE`).is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

목록 12-23: `IGNORE_CASE`라는 이름의 환경 변수에서 값 확인하기

여기에서 새 변수 `ignore_case`를 만듭니다. 값을 설정하기 위해 `env::var` 함수를 호출하고 `IGNORE_CASE` 환경 변수의 이름을 전달합니다. `env::var` 함수는 환경 변수가 임의의 값으로 설정된 경우 환경 변수의 값을 포함하는 성공적인 `Ok` 변형이 될 `결과`를 반환합니다. 환경 변수가 설정되지 않은 경우 `Err` 변형을 반환합니다.

환경 변수가 설정되었는지 확인하기 위해 `결과`에서 `is_ok` 메서드를 사용하고 있습니다. 이는 프로그램이 대소문자를 구분하지 않고 검색해야 함을 의미합니다. `IGNORE_CASE` 환경 변수가 아무 것도 설정되지 않은 경우 `is_ok`는 false를 반환하고 프로그램은 대소문자 구분 검색을 수행합니다. *우리는 환경 변수의 값이* 설정되었는지 여부에 대해 신경 쓰지 않으므로 `unwrap`, `expect` 또는 우리가 본 다른 방법을 사용하는 대신 `is_ok`를 확인합니다. `결과`.

Listing 12-22에서 구현한 것처럼 `ignore_case` 변수의 값을 `Config` 인스턴스로 전달하여 `run` 함수가 해당 값을 읽고 `search_case_insensitive` 또는 `search`를 호출할지 여부를 결정할 수 있습니다.

한번 해보자! 먼저 환경 변수를 설정하지 않고 `to`라는 쿼리를 사용하여 프로그램을 실행합니다. 이 쿼리는 모두 소문자로 `to`라는 단어가 포함된 줄과 일치해야 합니다.

```bash
$ cargo run -- to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
```

여전히 작동하는 것 같습니다! 이제 `IGNORE_CASE`를 `1`로 설정하고 동일한 쿼리 `to`를 사용하여 프로그램을 실행해 보겠습니다.

```bash
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

PowerShell을 사용하는 경우 환경 변수를 설정하고 별도의 명령으로 프로그램을 실행해야 합니다.

```bash
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

이렇게 하면 나머지 셸 세션 동안 `IGNORE_CASE`가 지속됩니다. `Remove-Item` cmdlet을 사용하여 설정을 해제할 수 있습니다.

```bash
PS> Remove-Item Env:IGNORE_CASE
```

대문자를 가질 수 있는 `to`를 포함하는 줄을 가져와야 합니다.

```bash
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

훌륭합니다. `To`가 포함된 대사도 있습니다! `minigrep` 프로그램은 이제 환경 변수에 의해 제어되는 대소문자를 구분하지 않는 검색을 수행할 수 있습니다. 이제 명령줄 인수 또는 환경 변수를 사용하여 옵션 세트를 관리하는 방법을 알게 되었습니다.

일부 프로그램은 동일한 구성에 대해 인수 *와 환경 변수를 허용합니다.* 이러한 경우 프로그램은 둘 중 하나가 우선적으로 적용되도록 결정합니다. 다른 실습을 위해 명령줄 인수 또는 환경 변수를 통해 대소문자 구분을 제어해 보십시오. 하나는 대소문자를 구분하도록 설정하고 다른 하나는 대소문자를 무시하도록 설정된 상태로 프로그램을 실행하는 경우 명령줄 인수 또는 환경 변수가 우선적으로 적용되어야 하는지 여부를 결정합니다.

`std::env` 모듈에는 환경 변수를 처리하기 위한 더 많은 유용한 기능이 포함되어 있습니다. 사용 가능한 기능을 보려면 설명서를 확인하십시오.

------

## [표준 출력 대신 표준 오류에 오류 메시지 쓰기](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#writing-error-messages-to-standard-error-instead-of-standard-output)

지금은 `println!`을 사용하여 모든 출력을 터미널에 쓰고 있습니다. 매크로. *대부분의 터미널에는 일반 정보에 대한 표준 출력* (`stdout`)과 오류 메시지에 대한 *표준 오류* (`stderr`) 의 두 가지 종류의 출력이 있습니다. 이러한 구별을 통해 사용자는 프로그램의 성공적인 출력을 파일로 지정하지만 여전히 오류 메시지를 화면에 인쇄하도록 선택할 수 있습니다.

`println!` 매크로는 표준 출력으로만 인쇄할 수 있으므로 표준 오류로 인쇄하려면 다른 것을 사용해야 합니다.

### [오류가 기록된 위치 확인](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#checking-where-errors-are-written)

먼저 `minigrep`에 의해 인쇄된 내용이 현재 어떻게 표준 출력에 기록되는지 살펴보겠습니다. 여기에는 표준 오류에 기록하려는 오류 메시지도 포함됩니다. 의도적으로 오류를 발생시키면서 표준 출력 스트림을 파일로 리디렉션하여 이를 수행합니다. 표준 오류 스트림을 리디렉션하지 않으므로 표준 오류로 전송된 모든 콘텐츠가 화면에 계속 표시됩니다.

명령줄 프로그램은 오류 메시지를 표준 오류 스트림으로 보낼 것으로 예상되므로 표준 출력 스트림을 파일로 리디렉션하더라도 화면에서 여전히 오류 메시지를 볼 수 있습니다. 우리 프로그램은 현재 제대로 작동하지 않습니다. 대신 오류 메시지 출력을 파일에 저장하는 것을 보게 될 것입니다!

이 동작을 시연하기 위해 `>` 및 표준 출력 스트림을 리디렉션할 파일 경로 *output.txt 를* 사용하여 프로그램을 실행합니다. 오류가 발생하는 인수를 전달하지 않습니다.

```bash
$ cargo run > output.txt
```

`>` 구문은 표준 출력의 내용을 화면 대신 *output.txt* 에 쓰도록 쉘에 지시합니다. 예상했던 오류 메시지가 화면에 표시되지 않았으므로 파일에 오류가 있음을 의미합니다. 이것이 *output.txt* 에 포함된 내용입니다.

```text
Problem parsing arguments: not enough arguments
```

예, 오류 메시지가 표준 출력으로 출력되고 있습니다. 이와 같은 오류 메시지가 표준 오류로 인쇄되어 성공적인 실행의 데이터만 파일에 저장되는 것이 훨씬 더 유용합니다. 우리는 그것을 바꿀 것입니다.

### [오류를 표준 오류로 인쇄](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#printing-errors-to-standard-error)

Listing 12-24의 코드를 사용하여 오류 메시지가 인쇄되는 방식을 변경할 것입니다. 이 장의 앞부분에서 수행한 리팩토링으로 인해 오류 메시지를 인쇄하는 모든 코드는 `main`이라는 하나의 함수에 있습니다. 표준 라이브러리는 `eprintln!` 표준 오류 스트림으로 인쇄하는 매크로이므로 `println!`이라고 부르던 두 위치를 변경하겠습니다. 오류를 인쇄하려면 `eprintln!`을 사용하십시오. 대신에.

파일 이름: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!(`Problem parsing arguments: {err}`);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!(`Application error: {e}`);
        process::exit(1);
    }
}
```

목록 12-24: `eprintln!`을 사용하여 표준 출력 대신 표준 오류에 오류 메시지 쓰기

이제 인수 없이 `>`를 사용하여 표준 출력을 리디렉션하는 동일한 방식으로 프로그램을 다시 실행해 보겠습니다.

```bash
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

이제 화면에 오류가 표시되고 *output.txt* 에 아무것도 포함되지 않습니다. 이는 명령줄 프로그램에서 예상되는 동작입니다.

다음과 같이 오류를 일으키지 않지만 여전히 표준 출력을 파일로 리디렉션하는 인수를 사용하여 프로그램을 다시 실행해 보겠습니다.

```bash
$ cargo run -- to poem.txt > output.txt
```

터미널에 대한 출력이 표시되지 않으며 *output.txt에* 결과가 포함됩니다.

파일 이름: output.txt

```text
Are you nobody, too?
How dreary to be somebody!
```

이것은 성공적인 출력을 위해 표준 출력을 사용하고 오류 출력을 위해 표준 오류를 적절하게 사용하고 있음을 보여줍니다.

## [요약](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html#summary)

이 장에서는 지금까지 배운 몇 가지 주요 개념을 요약하고 Rust에서 일반적인 I/O 작업을 수행하는 방법을 다루었습니다. 명령줄 인수, 파일, 환경 변수 및 `eprintln!` 인쇄 오류에 대한 매크로, 이제 명령줄 응용 프로그램을 작성할 준비가 된 것입니다. 이전 장의 개념과 결합하면 코드가 잘 구성되고 데이터를 적절한 데이터 구조에 효과적으로 저장하며 오류를 잘 처리하고 잘 테스트됩니다.

다음으로 함수형 언어의 영향을 받은 일부 Rust 기능인 클로저와 이터레이터를 살펴보겠습니다.

------

# 13

# [함수형 언어 기능: 반복자와 클로저](https://doc.rust-lang.org/book/ch13-00-functional-features.html#functional-language-features-iterators-and-closures)

Rust의 디자인은 기존의 많은 언어와 기술에서 영감을 얻었으며 한 가지 중요한 영향은 *함수형 프로그래밍* 입니다. 기능적 스타일의 프로그래밍에는 종종 함수를 인수로 전달하고, 다른 함수에서 반환하고, 나중에 실행하기 위해 변수에 할당하는 등 함수를 값으로 사용하는 것이 포함됩니다.

이 장에서 우리는 함수형 프로그래밍이 무엇인지 아닌지에 대한 문제에 대해 토론하지 않을 것입니다. 대신 종종 함수형이라고 하는 많은 언어의 기능과 유사한 Rust의 일부 기능에 대해 논의할 것입니다.

보다 구체적으로 다음 내용을 다룹니다.

- *Closures* , 변수에 저장할 수 있는 함수와 유사한 구조
- 일련의 요소를 처리하는 방법인 *반복자*
- 12장에서 클로저와 반복자를 사용하여 I/O 프로젝트를 개선하는 방법
- 클로저와 이터레이터의 성능(스포일러 경고: 생각보다 빠릅니다!)

우리는 함수형 스타일의 영향을 받는 패턴 일치 및 열거형과 같은 일부 다른 Rust 기능을 이미 다루었습니다. 클로저와 이터레이터를 마스터하는 것은 관용적이고 빠른 Rust 코드를 작성하는 데 중요한 부분이기 때문에 이 장 전체를 여기에 할애할 것입니다.

------

## [클로저: 환경을 캡처하는 익명 함수](https://doc.rust-lang.org/book/ch13-01-closures.html#closures-anonymous-functions-that-capture-their-environment)

Rust의 클로저는 변수에 저장하거나 다른 함수에 인수로 전달할 수 있는 익명 함수입니다. 한 곳에서 클로저를 생성한 다음 다른 곳에서 클로저를 호출하여 다른 컨텍스트에서 평가할 수 있습니다. 함수와 달리 클로저는 정의된 범위에서 값을 캡처할 수 있습니다. 이러한 클로저 기능이 코드 재사용 및 동작 사용자 정의를 허용하는 방법을 보여줍니다.

### [폐쇄로 환경 캡처](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-the-environment-with-closures)

나중에 사용하기 위해 클로저가 정의된 환경에서 값을 캡처하기 위해 클로저를 사용하는 방법을 먼저 살펴보겠습니다. 시나리오는 다음과 같습니다. 우리 티셔츠 회사는 메일링 리스트에 있는 누군가에게 판촉 행사로 독점적인 한정판 셔츠를 무료로 제공합니다. 메일링 리스트에 있는 사람들은 원하는 색상을 프로필에 추가할 수 있습니다. 무료 셔츠로 선택된 사람이 좋아하는 색상 세트를 가지고 있으면 그 색상 셔츠를 받게 됩니다. 선호하는 색상을 지정하지 않은 경우 회사에서 현재 가장 많이 사용하는 색상이 지정됩니다.

이를 구현하는 방법에는 여러 가지가 있습니다. 이 예에서는 `Red` 및 `Blue` 변형이 있는 `ShirtColor`라는 열거형을 사용합니다(단순화를 위해 사용 가능한 색상 수 제한). `Vec`을 포함하는 `shirts`라는 필드가 있는 `Inventory` 구조로 회사의 인벤토리를 나타냅니다.`는 현재 재고가 있는 셔츠 색상을 나타냅니다. `Inventory`에 정의된 `giveaway` 메서드는 무료 셔츠 당첨자의 셔츠 색상 기본 설정(옵션)을 가져오고 그 사람이 받게 될 셔츠 색상을 반환합니다. 이 설정은 목록 13-1에 나와 있습니다. :

파일 이름: src/main.rs

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        `The user with preference {:?} gets {:?}`,
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        `The user with preference {:?} gets {:?}`,
        user_pref2, giveaway2
    );
}
```

목록 13-1: 셔츠 회사 증정품 상황

`메인`에 정의된 `상점`에는 이 한정판 프로모션을 위해 배포할 파란색 셔츠 2개와 빨간색 셔츠 1개가 남아 있습니다. 빨간색 셔츠를 선호하는 사용자와 선호하지 않는 사용자를 `giveaway` 방식이라고 합니다.

다시 말하지만, 이 코드는 여러 가지 방법으로 구현될 수 있으며 여기서는 클로저에 초점을 맞추기 위해 클로저를 사용하는 `giveaway` 메서드의 본문을 제외하고 이미 배운 개념을 고수했습니다. `giveaway` 방식에서는 사용자 선호도를 `Option` 유형의 매개변수로 가져옵니다.` 그리고 `user_preference`에서 `unwrap_or_else` 메서드를 호출합니다. [`Option`에서 `unwrap_or_else` 메서드는](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else) 표준 라이브러리에 의해 정의됩니다. 하나의 인수를 취합니다: 값 `T`를 반환하는 인수 없는 클로저(동일한 유형 `옵션`의 `일부` 변형에 저장됨`, 이 경우 `ShirtColor`). `옵션`는 `Some` 변형이고, `unwrap_or_else`는 `Some` 내에서 값을 반환합니다. `Option`는 `없음` 변형이고 `unwrap_or_else`는 클로저를 호출하고 클로저에서 반환된 값을 반환합니다.

클로저 표현식 `|| self.most_stocked()`를 `unwrap_or_else`의 인수로 지정합니다. 이것은 자체적으로 매개변수를 받지 않는 클로저입니다(클로저에 매개변수가 있는 경우 두 개의 수직 막대 사이에 나타납니다). 클로저의 본문은 `self.most_stocked()`를 호출합니다. 우리는 여기에서 클로저를 정의하고 있으며 `unwrap_or_else` 구현은 나중에 결과가 필요한 경우 클로저를 평가합니다.

이 코드를 실행하면 다음이 인쇄됩니다.

```bash
$ cargo run
   Compiling shirt-company v0.1.0 (file:///projects/shirt-company)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/shirt-company`
The user with preference Some(Red) gets Red
The user with preference None gets Blue
```

여기서 한 가지 흥미로운 측면은 현재 `Inventory` 인스턴스에서 `self.most_stocked()`를 호출하는 클로저를 전달했다는 것입니다. 표준 라이브러리는 우리가 정의한 `Inventory` 또는 `ShirtColor` 유형이나 이 시나리오에서 사용하려는 논리에 대해 알 필요가 없습니다. 클로저는 `self` `Inventory` 인스턴스에 대한 불변 참조를 캡처하고 `unwrap_or_else` 메서드에 지정한 코드와 함께 전달합니다. 반면에 함수는 이러한 방식으로 환경을 캡처할 수 없습니다.

### [클로저 유형 추론 및 주석](https://doc.rust-lang.org/book/ch13-01-closures.html#closure-type-inference-and-annotation)

함수와 클로저 사이에는 더 많은 차이점이 있습니다. 클로저는 일반적으로 `fn` 함수처럼 매개변수의 유형이나 반환 값에 주석을 달 필요가 없습니다. 유형은 사용자에게 노출되는 명시적 인터페이스의 일부이기 때문에 유형 주석은 함수에 필요합니다. 이 인터페이스를 엄격하게 정의하는 것은 모든 사람이 함수가 사용하고 반환하는 값 유형에 동의하도록 하는 데 중요합니다. 반면에 클로저는 다음과 같이 노출된 인터페이스에서 사용되지 않습니다. 클로저는 변수에 저장되고 이름을 지정하지 않고 라이브러리 사용자에게 노출되지 않고 사용됩니다.

폐쇄는 일반적으로 임의의 시나리오가 아닌 좁은 맥락 내에서만 짧고 관련이 있습니다. 이러한 제한된 컨텍스트 내에서 컴파일러는 대부분의 변수 유형을 유추할 수 있는 방법과 유사하게 매개 변수 및 반환 유형의 유형을 유추할 수 있습니다(컴파일러가 클로저 유형 주석도 필요로 하는 드문 경우가 있음).

변수와 마찬가지로 엄격하게 필요한 것보다 더 자세한 비용으로 명확성과 명확성을 높이려는 경우 유형 주석을 추가할 수 있습니다. 클로저에 대한 유형에 주석을 다는 것은 Listing 13-2에 표시된 정의와 같습니다. 이 예제에서 우리는 목록 13-1에서 했던 것처럼 클로저를 인수로 전달하는 지점에서 클로저를 정의하는 대신 클로저를 정의하고 변수에 저장합니다.

파일 이름: src/main.rs

```rust
    let expensive_closure = |num: u32| -> u32 {
        println!(`calculating slowly...`);
        thread::sleep(Duration::from_secs(2));
        num
    };
```

목록 13-2: 클로저에서 매개변수 및 반환 값 유형의 선택적 유형 주석 추가

유형 주석을 추가하면 클로저 구문이 함수 구문과 더 유사해 보입니다. 여기서 우리는 비교를 위해 매개변수에 1을 더하는 함수와 동일한 동작을 하는 클로저를 정의합니다. 관련 부분을 정렬하기 위해 공백을 추가했습니다. 이는 파이프 사용과 선택적 구문의 양을 제외하고 클로저 구문이 함수 구문과 얼마나 유사한지 보여줍니다.

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

첫 번째 줄은 함수 정의를 보여주고 두 번째 줄은 완전히 주석이 달린 클로저 정의를 보여줍니다. 세 번째 줄에서는 클로저 정의에서 유형 주석을 제거합니다. 네 번째 줄에서 대괄호를 제거합니다. 이는 클로저 본문에 표현식이 하나만 있기 때문에 선택적입니다. 이들은 모두 호출될 때 동일한 동작을 생성하는 유효한 정의입니다. `add_one_v3` 및 `add_one_v4` 줄은 유형이 사용에서 유추되기 때문에 컴파일할 수 있도록 클로저를 평가해야 합니다. 이것은 `let v = Vec::new();`와 유사합니다. Rust가 유형을 유추할 수 있도록 `Vec`에 삽입할 유형 주석 또는 일부 유형의 값이 필요합니다.

클로저 정의의 경우 컴파일러는 각 매개 변수와 반환 값에 대해 하나의 구체적인 유형을 유추합니다. 예를 들어, 목록 13-3은 매개변수로 받은 값을 반환하는 짧은 클로저의 정의를 보여줍니다. 이 클로저는 이 예제의 목적을 제외하고는 그다지 유용하지 않습니다. 정의에 유형 주석을 추가하지 않았습니다. 타입 어노테이션이 없기 때문에 어떤 타입으로도 클로저를 호출할 수 있습니다. 여기에서 처음으로 `String`으로 했습니다. 그런 다음 정수로 `example_closure`를 호출하려고 하면 오류가 발생합니다.

파일 이름: src/main.rs

```rust
    let example_closure = |x| x;

    let s = example_closure(String::from(`hello`));
    let n = example_closure(5);
```

목록 13-3: 유형이 두 가지 다른 유형으로 유추되는 클로저 호출 시도

컴파일러는 다음 오류를 제공합니다.

```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
error[E0308]: mismatched types
 --> src/main.rs:5:29
  |
5 |     let n = example_closure(5);
  |             --------------- ^- help: try using a conversion method: `.to_string()`
  |             |               |
  |             |               expected struct `String`, found integer
  |             arguments to this function are incorrect
  |
note: closure parameter defined here
 --> src/main.rs:2:28
  |
2 |     let example_closure = |x| x;
  |                            ^

For more information about this error, try `rustc --explain E0308`.
error: could not compile `closure-example` due to previous error
```

`string` 값으로 `example_closure`를 처음 호출할 때 컴파일러는 `x` 유형과 클로저의 반환 유형을 `String`으로 유추합니다. 그런 다음 이러한 유형은 `example_closure`의 클로저에 잠기며 다음에 동일한 클로저로 다른 유형을 사용하려고 하면 유형 오류가 발생합니다.

### [참조 캡처 또는 소유권 이동](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-references-or-moving-ownership)

클로저는 세 가지 방법으로 환경에서 값을 캡처할 수 있으며, 이는 함수가 매개변수를 취할 수 있는 세 가지 방법(변경불가 차용, 변경가능 차용 및 소유권 가져오기)에 직접 매핑됩니다. 클로저는 함수 본문이 캡처된 값으로 수행하는 작업을 기반으로 이들 중 사용할 것을 결정합니다.

목록 13-4에서 우리는 `list`라는 이름의 벡터에 대한 불변 참조를 캡처하는 클로저를 정의합니다. 값을 출력하기 위해 불변 참조만 필요하기 때문입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!(`Before defining closure: {:?}`, list);

    let only_borrows = || println!(`From closure: {:?}`, list);

    println!(`Before calling closure: {:?}`, list);
    only_borrows();
    println!(`After calling closure: {:?}`, list);
}
```

목록 13-4: 불변 참조를 캡처하는 클로저 정의 및 호출

이 예제는 또한 변수가 클로저 정의에 바인딩될 수 있고 나중에 변수 이름이 함수 이름인 것처럼 변수 이름과 괄호를 사용하여 클로저를 호출할 수 있음을 보여줍니다.

`list`에 대한 여러 불변 참조를 동시에 가질 수 있기 때문에 클로저 정의 이전, 클로저 정의 이후, 클로저 호출 전, 클로저 호출 이후 코드에서 여전히 `list`에 액세스할 수 있습니다. 이 코드는 다음을 컴파일, 실행 및 인쇄합니다.

```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
Before calling closure: [1, 2, 3]
From closure: [1, 2, 3]
After calling closure: [1, 2, 3]
```

다음으로 목록 13-5에서 `목록` 벡터에 요소를 추가하도록 클로저 본문을 변경합니다. 클로저는 이제 변경 가능한 참조를 캡처합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!(`Before defining closure: {:?}`, list);

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!(`After calling closure: {:?}`, list);
}
```

목록 13-5: 변경 가능한 참조를 캡처하는 클로저 정의 및 호출

이 코드는 다음을 컴파일, 실행 및 인쇄합니다.

```bash
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
After calling closure: [1, 2, 3, 7]
```

더 이상 `println!` `borrows_mutably` 클로저의 정의와 호출 사이: `borrows_mutably`가 정의되면 `list`에 대한 가변 참조를 캡처합니다. 클로저가 호출된 후에는 클로저를 다시 사용하지 않으므로 가변 차용이 종료됩니다. 클로저 정의와 클로저 호출 사이에는 변경 가능한 차용이 있을 때 다른 차용이 허용되지 않기 때문에 인쇄할 변경 불가능한 차용이 허용되지 않습니다. `println!`을 추가해 보십시오. 어떤 오류 메시지가 표시되는지 확인하세요!

클로저 본문에 소유권이 엄격하게 필요하지 않더라도 클로저가 환경에서 사용하는 값의 소유권을 갖도록 강제하려면 매개변수 목록 앞에 `move` 키워드를 사용할 수 있습니다.

이 기법은 새 스레드가 데이터를 소유하도록 클로저를 새 스레드에 전달하여 데이터를 이동할 때 주로 유용합니다. 16장에서 동시성에 대해 이야기할 때 스레드와 스레드를 사용하려는 이유에 대해 자세히 설명하겠지만 지금은 `이동` 키워드가 필요한 클로저를 사용하여 새 스레드를 생성하는 방법을 간략하게 살펴보겠습니다. 목록 13-6은 주 스레드가 아닌 새 스레드에서 벡터를 인쇄하도록 수정된 목록 13-4를 보여줍니다.

파일 이름: src/main.rs

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!(`Before defining closure: {:?}`, list);

    thread::spawn(move || println!(`From thread: {:?}`, list))
        .join()
        .unwrap();
}
```

목록 13-6: `이동`을 사용하여 스레드가 `목록`의 소유권을 갖도록 강제 종료

새 스레드를 생성하여 스레드가 인수로 실행할 클로저를 제공합니다. 클로저 본문은 목록을 출력합니다. 목록 13-4에서 클로저는 불변 참조를 사용하여 `목록`만 캡처했습니다. `목록`을 인쇄하는 데 필요한 최소한의 액세스이기 때문입니다. 이 예에서 클로저 본문에는 여전히 불변 참조만 필요하지만 , 클로저 정의의 시작 부분에 `move` 키워드를 넣어 `list`가 클로저로 이동되도록 지정해야 합니다. 새 스레드는 나머지 메인 스레드가 완료되기 전에 완료되거나 메인 스레드가 완료될 수 있습니다. 첫째, 메인 스레드가 `list`의 소유권을 유지했지만 새 스레드가 `list`를 삭제하기 전에 종료된 경우 스레드의 불변 참조는 유효하지 않습니다. 컴파일러는 참조가 유효하도록 `목록`을 새 스레드에 지정된 클로저로 이동해야 합니다. `move` 키워드를 제거하거나 클로저가 정의된 후 메인 스레드에서 `list`를 사용하여 어떤 컴파일러 오류가 발생하는지 확인하십시오!

### [클로저 및 `Fn` 특성에서 캡처된 값 이동](https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-closures-and-the-fn-traits)

클로저가 참조를 캡처하거나 클로저가 정의된 환경에서 값의 소유권을 캡처하면(따라서 클로저 *로* 이동되는 항목에 영향을 미침 ) 클로저 본문의 코드는 참조에 발생하는 작업을 정의합니다. 또는 클로저가 나중에 평가될 때의 값(따라서 클로저 *밖으로* 이동되는 항목에 영향을 미침 ). 클로저 본문은 다음 중 하나를 수행할 수 있습니다. 캡처된 값을 클로저 밖으로 이동, 캡처된 값을 변경, 값을 이동 또는 변경하지 않음, 시작하기 위해 환경에서 아무 것도 캡처하지 않음.

클로저가 환경에서 값을 캡처하고 처리하는 방식은 클로저가 구현하는 특성에 영향을 미치며 특성은 함수와 구조체가 사용할 수 있는 클로저의 종류를 지정하는 방법입니다. 클로저는 클로저의 본문이 값을 처리하는 방식에 따라 추가 방식으로 이러한 `Fn` 특성 중 하나, 둘 또는 셋 모두를 자동으로 구현합니다.

1. `FnOnce`는 한 번 호출할 수 있는 클로저에 적용됩니다. 모든 클로저가 호출될 수 있기 때문에 모든 클로저는 최소한 이 특성을 구현합니다. 캡처된 값을 본문 밖으로 옮기는 클로저는 한 번만 호출할 수 있기 때문에 `FnOnce`만 구현하고 다른 `Fn` 특성은 구현하지 않습니다.
2. `FnMut`는 캡처된 값을 본문 밖으로 이동하지 않지만 캡처된 값을 변경할 수 있는 클로저에 적용됩니다. 이러한 클로저는 한 번 이상 호출될 수 있습니다.
3. `Fn`은 캡처된 값을 본문 밖으로 이동하지 않고 캡처된 값을 변경하지 않는 클로저와 환경에서 아무것도 캡처하지 않는 클로저에 적용됩니다. 이러한 클로저는 환경을 변경하지 않고 두 번 이상 호출할 수 있으며 이는 클로저를 동시에 여러 번 호출하는 것과 같은 경우에 중요합니다.

`옵션`에서 `unwrap_or_else` 메서드의 정의를 살펴보겠습니다.` 우리가 Listing 13-1에서 사용한 것:

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

`T`는 `Option`의 `Some` 변형에 있는 값의 유형을 나타내는 일반 유형임을 상기하십시오. 해당 유형 `T`는 `unwrap_or_else` 함수의 반환 유형이기도 합니다.`, 예를 들어 `문자열`을 가져옵니다.

다음으로 `unwrap_or_else` 함수에는 추가 일반 유형 매개변수 `F`가 있습니다. `F` 유형은 `unwrap_or_else`를 호출할 때 제공하는 클로저인 `f`라는 매개변수의 유형입니다.

제네릭 유형 `F`에 지정된 특성 경계는 `FnOnce() -> T`입니다. 즉, `F`는 한 번 호출할 수 있어야 하고 인수를 취하지 않고 `T`를 반환해야 합니다. 트레이트 바운드에서 `FnOnce`를 사용하면 `unwrap_or_else`가 최대 한 번만 `f`를 호출한다는 제약 조건을 나타냅니다. `unwrap_or_else`의 본문에서 `Option`이 `Some`인 경우 `f`가 호출되지 않는 것을 확인할 수 있습니다. `Option`이 `None`이면 `f`가 한 번 호출됩니다. 모든 클로저가 `FnOnce`를 구현하기 때문에 `unwrap_or_else`는 가장 다양한 종류의 클로저를 허용하고 최대한 유연합니다.

> 참고: 함수는 세 가지 `Fn` 특성도 모두 구현할 수 있습니다. 우리가 하고 싶은 일이 환경에서 값을 캡처할 필요가 없다면 `Fn` 특성 중 하나를 구현하는 무언가가 필요한 클로저 대신 함수 이름을 사용할 수 있습니다. 예를 들어 `Option<Vec>` 값이 `None`이면 `unwrap_or_else(Vec::new)`를 호출하여 새로운 빈 벡터를 얻을 수 있습니다.

이제 슬라이스에 정의된 표준 라이브러리 메서드 `sort_by_key`를 살펴보고 이것이 `unwrap_or_else`와 어떻게 다른지 그리고 `sort_by_key`가 특성 바인딩에 `FnOnce` 대신 `FnMut`을 사용하는 이유를 살펴보겠습니다. 클로저는 고려 중인 슬라이스의 현재 항목에 대한 참조 형식으로 하나의 인수를 가져오고 주문할 수 있는 `K` 유형의 값을 반환합니다. 이 기능은 각 항목의 특정 속성별로 조각을 정렬하려는 경우에 유용합니다. Listing 13-7에는 `Rectangle` 인스턴스 목록이 있고 `sort_by_key`를 사용하여 `width` 속성에 따라 낮은 값에서 높은 값으로 정렬합니다.

파일 이름: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!(`{:#?}`, list);
}
```

목록 13-7: `sort_by_key`를 사용하여 너비별로 사각형 정렬

이 코드는 다음을 인쇄합니다.

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/rectangles`
[
    Rectangle {
        width: 3,
        height: 5,
    },
    Rectangle {
        width: 7,
        height: 12,
    },
    Rectangle {
        width: 10,
        height: 1,
    },
]
```

`sort_by_key`가 `FnMut` 클로저를 취하도록 정의된 이유는 클로저를 슬라이스의 각 항목에 대해 한 번씩 여러 번 호출하기 때문입니다. 클로저 `|r| r.width`는 환경에서 아무것도 캡처, 변경 또는 이동하지 않으므로 특성 제한 요구 사항을 충족합니다.

반대로 Listing 13-8은 `FnOnce` 특성만 구현하는 클로저의 예를 보여줍니다. 환경에서 값을 이동하기 때문입니다. 컴파일러는 `sort_by_key`와 함께 이 클로저를 사용하도록 허용하지 않습니다.

파일 이름: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from(`by key called`);

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!(`{:#?}`, list);
}
```

목록 13-8: `sort_by_key`와 함께 `FnOnce` 클로저 사용 시도

이것은 `목록`을 정렬할 때 `sort_by_key`가 호출되는 횟수를 계산하기 위해 고안되고 복잡한 방법(작동하지 않음)입니다. 이 코드는 클로저 환경의 `문자열`인 `value`를 `sort_operations` 벡터로 푸시하여 이 계산을 시도합니다. 클로저는 `value`를 캡처한 다음 `value`의 소유권을 `sort_operations` 벡터로 전송하여 클로저 밖으로 `value`를 이동합니다. 이 클로저는 한 번만 호출할 수 있습니다. 두 번째로 호출하려고 하면 `value`가 더 이상 `sort_operations`로 다시 푸시되는 환경에 있지 않기 때문에 작동하지 않습니다! 따라서 이 클로저는 `FnOnce`만 구현합니다. 이 코드를 컴파일하려고 하면 `value`라는 오류가 발생합니다.

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
error[E0507]: cannot move out of `value`, a captured variable in an `FnMut` closure
  --> src/main.rs:18:30
   |
15 |     let value = String::from(`by key called`);
   |         ----- captured outer variable
16 |
17 |     list.sort_by_key(|r| {
   |                      --- captured by this `FnMut` closure
18 |         sort_operations.push(value);
   |                              ^^^^^ move occurs because `value` has type `String`, which does not implement the `Copy` trait

For more information about this error, try `rustc --explain E0507`.
error: could not compile `rectangles` due to previous error
```

오류는 `값`을 환경 밖으로 이동시키는 클로저 본문의 라인을 가리킵니다. 이 문제를 해결하려면 값을 환경 밖으로 이동하지 않도록 클로저 본문을 변경해야 합니다. `sort_by_key`가 호출되는 횟수를 계산하려면 환경에 카운터를 유지하고 클로저 본문에서 해당 값을 증가시키는 것이 더 간단하게 계산할 수 있는 방법입니다. 목록 13-9의 클로저는 `num_sort_operations` 카운터에 대한 변경 가능한 참조만 캡처하므로 두 번 이상 호출될 수 있기 때문에 `sort_by_key`와 함께 작동합니다.

파일 이름: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!(`{:#?}, sorted in {num_sort_operations} operations`, list);
}
```

목록 13-9: `sort_by_key`와 함께 `FnMut` 클로저 사용이 허용됨

`Fn` 특성은 클로저를 사용하는 함수 또는 유형을 정의하거나 사용할 때 중요합니다. 다음 섹션에서는 반복자에 대해 설명합니다. 많은 반복자 메서드는 클로저 인수를 사용하므로 계속 진행하면서 이러한 클로저 세부 정보를 염두에 두십시오!

------

## [Iterator로 일련의 항목 처리](https://doc.rust-lang.org/book/ch13-02-iterators.html#processing-a-series-of-items-with-iterators)

반복자 패턴을 사용하면 일련의 항목에 대해 차례로 일부 작업을 수행할 수 있습니다. 반복자는 각 항목을 반복하고 시퀀스가 완료되는 시기를 결정하는 논리를 담당합니다. 반복자를 사용하면 해당 논리를 직접 다시 구현할 필요가 없습니다.

Rust에서 이터레이터는 *게으르다* . 즉, 이터레이터를 소모하는 메서드를 호출하여 소진할 때까지 효과가 없습니다. 예를 들어 목록 13-10의 코드는 `Vec`에 정의된 `iter` 메서드를 호출하여 벡터 `v1`의 항목에 대한 반복자를 만듭니다.`. 이 코드 자체는 유용한 작업을 수행하지 않습니다.

```rust
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();
```

목록 13-10: 이터레이터 만들기

반복자는 `v1_iter` 변수에 저장됩니다. 반복자를 생성하면 다양한 방법으로 사용할 수 있습니다. 3장의 목록 3-5에서 `for` 루프를 사용하여 배열을 반복하여 각 항목에 대해 일부 코드를 실행했습니다. 내부적으로 이것은 암시적으로 반복자를 생성하고 소비했지만 지금까지 그것이 정확히 어떻게 작동하는지에 대해서는 얼버무렸습니다.

목록 13-11의 예에서 `for` 루프에서 반복자의 사용과 반복자의 생성을 분리합니다. `v1_iter`의 반복자를 사용하여 `for` 루프를 호출하면 반복자의 각 요소가 각 값을 출력하는 루프의 한 반복에서 사용됩니다.

```rust
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    for val in v1_iter {
        println!(`Got: {}`, val);
    }
```

Listing 13-11: `for` 루프에서 이터레이터 사용하기

표준 라이브러리에서 제공하는 반복자가 없는 언어에서는 인덱스 0에서 변수를 시작하고 해당 변수를 사용하여 값을 얻기 위해 벡터로 인덱싱하고 루프에서 변수 값을 증가시키는 방식으로 이와 동일한 기능을 작성할 수 있습니다. 벡터의 총 항목 수에 도달할 때까지.

Iterator는 모든 논리를 처리하여 잠재적으로 엉망이 될 수 있는 반복 코드를 줄입니다. 반복자는 벡터와 같이 인덱싱할 수 있는 데이터 구조뿐만 아니라 다양한 종류의 시퀀스와 함께 동일한 논리를 사용할 수 있는 더 많은 유연성을 제공합니다. 이터레이터가 어떻게 하는지 살펴보겠습니다.

### [`반복자` 특성 및 `다음` 방법](https://doc.rust-lang.org/book/ch13-02-iterators.html#the-iterator-trait-and-the-next-method)

모든 반복자는 표준 라이브러리에 정의된 `반복자`라는 특성을 구현합니다. 특성의 정의는 다음과 같습니다.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

이 정의는 `type Item` 및 `Self::Item`과 같은 몇 가지 새로운 구문을 사용하며 이 특성과 *연관된 유형을* 정의합니다. 19장에서 연관 유형에 대해 자세히 이야기하겠습니다. 지금은 이 코드에서 `반복자` 특성을 구현하려면 `항목` 유형도 정의해야 한다고 말하고 있고 이 `항목` 유형은 `next` 메서드의 반환 유형에 사용됩니다. 즉, `항목` 유형은 반복자에서 반환된 유형이 됩니다.

`반복자` 특성은 구현자가 하나의 메서드를 정의하기만 하면 됩니다. `다음` 메서드는 `Some`으로 래핑된 반복자의 한 항목을 반환하고 반복이 끝나면 `없음`을 반환합니다.

반복자에서 `next` 메서드를 직접 호출할 수 있습니다. 목록 13-12는 벡터에서 생성된 반복자에서 `next`에 대한 반복 호출에서 반환되는 값을 보여줍니다.

파일 이름: src/lib.rs

```rust
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
```

Listing 13-12: 이터레이터에서 `next` 메소드 호출하기

우리는 `v1_iter`를 변경 가능하게 만들어야 했습니다. 반복자에서 `next` 메서드를 호출하면 반복자가 시퀀스의 위치를 추적하는 데 사용하는 내부 상태가 변경됩니다. 즉, 이 코드는 iterator를 *소비하거나* 소모합니다. `다음`을 호출할 때마다 반복자에서 항목을 먹습니다. `for` 루프를 사용할 때 `v1_iter`를 변경 가능하게 만들 필요가 없었습니다. 왜냐하면 루프가 `v1_iter`의 소유권을 가지고 뒤에서 변경 가능하게 만들었기 때문입니다.

또한 `next` 호출에서 얻은 값은 벡터의 값에 대한 불변 참조입니다. `iter` 메서드는 불변 참조에 대한 반복자를 생성합니다. `v1`의 소유권을 갖고 소유한 값을 반환하는 반복자를 만들고 싶다면 `iter` 대신 `into_iter`를 호출할 수 있습니다. 마찬가지로 가변 참조를 반복하려면 `iter` 대신 `iter_mut`을 호출할 수 있습니다.

### [Iterator를 사용하는 메서드](https://doc.rust-lang.org/book/ch13-02-iterators.html#methods-that-consume-the-iterator)

`반복자` 특성에는 표준 라이브러리에서 제공하는 기본 구현을 포함하는 다양한 메서드가 있습니다. `반복자` 특성에 대한 표준 라이브러리 API 설명서를 보면 이러한 메서드에 대해 알아볼 수 있습니다. 이러한 메서드 중 일부는 정의에서 `next` 메서드를 호출하므로 `Iterator` 특성을 구현할 때 `next` 메서드를 구현해야 합니다.

`next`를 호출하는 메소드는 이를 호출하는 것이 반복자를 사용하기 때문에 *소비 어댑터 라고 합니다.* 한 가지 예는 반복자의 소유권을 가져오고 반복적으로 `next`를 호출하여 항목을 반복하여 반복자를 소비하는 `sum` 메서드입니다. 반복하면서 각 항목을 누적 합계에 추가하고 반복이 완료되면 합계를 반환합니다. 목록 13-13에는 `sum` 방법의 사용을 보여주는 테스트가 있습니다:

파일 이름: src/lib.rs

```rust
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
```

목록 13-13: 반복자에 있는 모든 항목의 총계를 얻기 위해 `sum` 메서드 호출

`sum`은 호출한 반복자의 소유권을 갖기 때문에 `sum` 호출 후에 `v1_iter`를 사용할 수 없습니다.

### [다른 반복자를 생성하는 메서드](https://doc.rust-lang.org/book/ch13-02-iterators.html#methods-that-produce-other-iterators)

*반복자 어댑터는* 반복자를 소비하지 않는 `반복자` 특성에 정의된 메서드입니다. 대신 원래 반복자의 일부 측면을 변경하여 다른 반복자를 생성합니다.

목록 13-14는 항목이 반복될 때 각 항목을 호출하기 위해 클로저를 취하는 반복자 어댑터 메서드 `map`을 호출하는 예를 보여줍니다. `map` 메서드는 수정된 항목을 생성하는 새 반복자를 반환합니다. 여기서 클로저는 벡터의 각 항목이 1씩 증가하는 새 이터레이터를 만듭니다.

파일 이름: src/main.rs

```rust
    let v1: Vec<i32> = vec![1, 2, 3];

    v1.iter().map(|x| x + 1);
```

Listing 13-14: 새 반복자를 만들기 위해 반복자 어댑터 `map` 호출하기

그러나 이 코드는 경고를 생성합니다.

```bash
$ cargo run
   Compiling iterators v0.1.0 (file:///projects/iterators)
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: iterators are lazy and do nothing unless consumed
  = note: `#[warn(unused_must_use)]` on by default

warning: `iterators` (bin `iterators`) generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.47s
     Running `target/debug/iterators`
```

목록 13-14의 코드는 아무 것도 하지 않습니다. 우리가 지정한 클로저는 절대 호출되지 않습니다. 경고는 반복자 어댑터가 게으르고 여기에서 반복자를 소비해야 하는 이유를 상기시켜 줍니다.

이 경고를 수정하고 반복자를 사용하기 위해 목록 12-1의 `env::args`와 함께 12장에서 사용한 `collect` 메서드를 사용합니다. 이 메서드는 반복자를 사용하고 결과 값을 컬렉션 데이터 형식으로 수집합니다.

Listing 13-15에서 `map` 호출에서 벡터로 반환된 반복자를 반복한 결과를 수집합니다. 이 벡터는 1씩 증가한 원래 벡터의 각 항목을 포함하게 됩니다.

파일 이름: src/main.rs

```rust
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
```

목록 13-15: `map` 메서드를 호출하여 새 반복자를 생성한 다음 `collect` 메서드를 호출하여 새 반복자를 소비하고 벡터를 생성

`map`은 클로저를 사용하기 때문에 각 항목에 대해 수행하려는 작업을 지정할 수 있습니다. 이것은 `Iterator` 특성이 제공하는 반복 동작을 재사용하면서 클로저를 통해 일부 동작을 사용자 정의할 수 있는 방법에 대한 좋은 예입니다.

읽기 쉬운 방식으로 복잡한 작업을 수행하기 위해 반복자 어댑터에 대한 여러 호출을 연결할 수 있습니다. 그러나 모든 반복자는 게으르기 때문에 반복자 어댑터 호출에서 결과를 얻으려면 소비 어댑터 메서드 중 하나를 호출해야 합니다.

### [환경을 캡처하는 클로저 사용](https://doc.rust-lang.org/book/ch13-02-iterators.html#using-closures-that-capture-their-environment)

많은 반복자 어댑터는 클로저를 인수로 사용하며 일반적으로 반복자 어댑터에 대한 인수로 지정하는 클로저는 해당 환경을 캡처하는 클로저입니다.

이 예에서는 클로저를 사용하는 `필터` 메서드를 사용합니다. 클로저는 반복자에서 항목을 가져오고 `부울`을 반환합니다. 클로저가 `true`를 반환하면 값은 `filter`에 의해 생성된 반복에 포함됩니다. 클로저가 `false`를 반환하면 값이 포함되지 않습니다.

목록 13-16에서 `Shoe` 구조체 인스턴스 컬렉션을 반복하기 위해 환경에서 `shoe_size` 변수를 캡처하는 클로저와 함께 `filter`를 사용합니다. 지정된 사이즈의 신발만 반환합니다.

파일 이름: src/lib.rs

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from(`sneaker`),
            },
            Shoe {
                size: 13,
                style: String::from(`sandal`),
            },
            Shoe {
                size: 10,
                style: String::from(`boot`),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from(`sneaker`)
                },
                Shoe {
                    size: 10,
                    style: String::from(`boot`)
                },
            ]
        );
    }
}
```

목록 13-16: `shoe_size`를 캡처하는 클로저와 함께 `filter` 메서드 사용

`shoes_in_size` 함수는 신발 벡터와 신발 사이즈를 매개변수로 사용합니다. 지정된 크기의 신발만 포함하는 벡터를 반환합니다.

`shoes_in_size` 본문에서 `into_iter`를 호출하여 벡터의 소유권을 갖는 이터레이터를 만듭니다. 그런 다음 `filter`를 호출하여 해당 반복자를 클로저가 `true`를 반환하는 요소만 포함하는 새 반복자로 조정합니다.

클로저는 환경에서 `shoe_size` 매개변수를 캡처하고 지정된 크기의 신발만 유지하면서 각 신발의 크기와 값을 비교합니다. 마지막으로 `collect`를 호출하면 적응된 반복자가 반환한 값을 함수가 반환한 벡터로 수집합니다.

테스트는 `shoes_in_size`를 호출할 때 지정한 값과 동일한 크기의 신발만 반환한다는 것을 보여줍니다.

------

## [I/O 프로젝트 개선](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#improving-our-io-project)

반복자에 대한 이 새로운 지식을 통해 반복자를 사용하여 코드의 위치를 보다 명확하고 간결하게 만들어 12장의 I/O 프로젝트를 개선할 수 있습니다. 반복자가 `Config::build` 기능과 `검색` 기능의 구현을 어떻게 개선할 수 있는지 살펴보겠습니다.

### [Iterator를 사용하여 `복제본` 제거](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#removing-a-clone-using-an-iterator)

Listing 12-6에서 우리는 `String` 값의 슬라이스를 가져오고 슬라이스로 인덱싱하고 값을 복제하여 `Config` 구조체가 해당 값을 소유하도록 허용하여 `Config` 구조체의 인스턴스를 생성하는 코드를 추가했습니다. 목록 13-17에서 우리는 목록 12-23에서와 같이 `Config::build` 함수의 구현을 재현했습니다:

파일 이름: src/lib.rs

```rust
impl Config {
    pub fn build(args: &[String]) -> Result<Config, &`static str> {
        if args.len() < 3 {
            return Err(`not enough arguments`);
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var(`IGNORE_CASE`).is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

Listing 13-17: Listing 12-23의 `Config::build` 함수 재현

당시 우리는 비효율적인 `복제` 호출에 대해 걱정하지 말라고 했습니다. 나중에 제거할 것이기 때문입니다. 자, 그 때가 바로 지금입니다!

`args` 매개변수에 `String` 요소가 있는 슬라이스가 있지만 `build` 함수는 `args`를 소유하지 않기 때문에 여기에서 `복제`가 필요했습니다. `Config` 인스턴스의 소유권을 반환하려면 `Config` 인스턴스가 해당 값을 소유할 수 있도록 `Config`의 `query` 및 `file_path` 필드에서 값을 복제해야 했습니다.

반복자에 대한 새로운 지식을 통해 슬라이스를 차용하는 대신 반복자의 소유권을 인수로 사용하도록 `빌드` 함수를 변경할 수 있습니다. 슬라이스의 길이를 확인하고 특정 위치로 인덱싱하는 코드 대신 반복기 기능을 사용합니다. 이터레이터가 값에 액세스하기 때문에 `Config::build` 함수가 수행하는 작업이 명확해집니다.

`Config::build`가 반복자의 소유권을 가져오고 빌린 인덱싱 작업 사용을 중지하면 `복제`를 호출하고 새 할당을 만드는 대신 반복자의 `문자열` 값을 `Config`로 이동할 수 있습니다.

#### [반환된 반복자를 직접 사용하기](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#using-the-returned-iterator-directly)

다음과 같은 I/O 프로젝트의 *src/main.rs 파일을 엽니다.*

파일 이름: src/main.rs

```rust
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!(`Problem parsing arguments: {err}`);
        process::exit(1);
    });

    // --snip--
}
```

우리는 먼저 Listing 12-24에 있는 `main` 함수의 시작을 Listing 13-18의 코드로 변경할 것입니다. 이번에는 이터레이터를 사용합니다. `Config::build`도 업데이트할 때까지 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!(`Problem parsing arguments: {err}`);
        process::exit(1);
    });

    // --snip--
}
```

목록 13-18: `env::args`의 반환 값을 `Config::build`에 전달

`env::args` 함수는 이터레이터를 반환합니다! 반복자 값을 벡터로 수집한 다음 조각을 `Config::build`로 전달하는 대신 이제 `env::args`에서 반환된 반복자의 소유권을 `Config::build`로 직접 전달합니다.

다음으로 `Config::build` 정의를 업데이트해야 합니다. I/O 프로젝트의 *src/lib.rs* 파일에서 `Config::build`의 서명을 목록 13-19처럼 보이도록 변경해 보겠습니다. 함수 본문을 업데이트해야 하므로 여전히 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &`static str> {
        // --snip--
```

Listing 13-19: 반복자를 예상하도록 `Config::build` 서명 업데이트

`env::args` 함수에 대한 표준 라이브러리 문서는 이 함수가 반환하는 반복자의 유형이 `std::env::Args`이고 해당 유형이 `Iterator` 특성을 구현하고 `문자열` 값을 반환함을 보여줍니다.

`Config::build` 함수의 시그니처를 업데이트하여 `args` 매개변수가 `&[String]` 대신 `impl Iterator<Item = String>` 특성 범위를 갖는 일반 유형을 가집니다. 10장의 `매개 변수 [로서의 특성](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters) ` 섹션 에서 논의한 `impl 특성` 구문의 사용은 `인자`가 `반복자` 유형을 구현하고 `문자열` 항목을 반환하는 모든 유형이 될 수 있음을 의미합니다.

우리는 `args`의 소유권을 갖고 그것을 반복함으로써 `args`를 변경하기 때문에 `args` 매개변수의 사양에 `mut` 키워드를 추가하여 변경할 수 있도록 할 수 있습니다.

#### [인덱싱 대신 `반복자` 특성 메서드 사용](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#using-iterator-trait-methods-instead-of-indexing)

다음으로 `Config::build`의 본문을 수정합니다. `args`는 `Iterator` 특성을 구현하기 때문에 `next` 메서드를 호출할 수 있습니다! Listing 13-20은 Listing 12-23의 코드를 업데이트하여 `next` 메소드를 사용합니다:

파일 이름: src/lib.rs

```rust
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &`static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err(`Didn`t get a query string`),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err(`Didn`t get a file path`),
        };

        let ignore_case = env::var(`IGNORE_CASE`).is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
```

목록 13-20: 반복자 메서드를 사용하도록 `Config::build` 본문 변경

`env::args` 반환 값의 첫 번째 값은 프로그램의 이름임을 기억하십시오. 우리는 이를 무시하고 다음 값으로 이동하기를 원하므로 먼저 `next`를 호출하고 반환 값으로 아무 작업도 수행하지 않습니다. 둘째, `next`를 호출하여 `Config`의 `query` 필드에 입력하려는 값을 가져옵니다. `next`가 `Some`을 반환하면 `match`를 사용하여 값을 추출합니다. `None`을 반환하면 충분한 인수가 제공되지 않았음을 의미하며 `Err` 값으로 일찍 반환됩니다. `file_path` 값에 대해 동일한 작업을 수행합니다.

### [반복자 어댑터로 코드를 더 명확하게 만들기](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#making-code-clearer-with-iterator-adaptors)

우리는 또한 I/O 프로젝트의 `검색` 기능에서 반복자를 활용할 수 있습니다. 이는 목록 12-19에서와 같이 여기 목록 13-21에 재현되어 있습니다.

파일 이름: src/lib.rs

```rust
pub fn search<`a>(query: &str, contents: &`a str) -> Vec<&`a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

Listing 13-21: Listing 12-19의 `검색` 기능 구현

반복자 어댑터 메서드를 사용하여 이 코드를 보다 간결하게 작성할 수 있습니다. 이렇게 하면 변경 가능한 중간 `결과` 벡터를 사용하지 않아도 됩니다. 함수형 프로그래밍 스타일은 코드를 더 명확하게 만들기 위해 변경 가능한 상태의 양을 최소화하는 것을 선호합니다. 변경 가능한 상태를 제거하면 `results` 벡터에 대한 동시 액세스를 관리할 필요가 없기 때문에 검색이 병렬로 수행되도록 향후 개선이 가능할 수 있습니다. 목록 13-22는 이 변경 사항을 보여줍니다:

파일 이름: src/lib.rs

```rust
pub fn search<`a>(query: &str, contents: &`a str) -> Vec<&`a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

Listing 13-22: `search` 함수 구현에서 반복자 어댑터 메서드 사용

`검색` 기능의 목적은 `쿼리`를 포함하는 `내용`의 모든 줄을 반환하는 것임을 상기하십시오. 목록 13-16의 `필터` 예제와 유사하게 이 코드는 `line.contains(query)`가 `true`를 반환하는 줄만 유지하기 위해 `필터` 어댑터를 사용합니다. 그런 다음 `collect`를 사용하여 일치하는 라인을 다른 벡터로 수집합니다. 훨씬 간단합니다! `search_case_insensitive` 함수에서도 반복자 메서드를 사용하도록 동일한 변경을 자유롭게 수행하십시오.

### [루프 또는 반복자 중에서 선택](https://doc.rust-lang.org/book/ch13-03-improving-our-io-project.html#choosing-between-loops-or-iterators)

다음 논리적 질문은 자신의 코드에서 어떤 스타일을 선택해야 하는지와 그 이유입니다: 목록 13-21의 원래 구현 또는 목록 13-22의 반복자를 사용하는 버전. 대부분의 Rust 프로그래머는 반복자 스타일을 선호합니다. 처음에는 요령을 터득하기가 조금 더 어렵지만 일단 다양한 반복자 어댑터와 그 역할에 대해 이해하고 나면 반복자를 이해하기가 더 쉬울 수 있습니다. 루프의 다양한 비트를 만지작거리고 새로운 벡터를 구축하는 대신 코드는 루프의 높은 수준의 목표에 초점을 맞춥니다. 이것은 일반적인 코드의 일부를 추상화하므로 반복자의 각 요소가 통과해야 하는 필터링 조건과 같이 이 코드에 고유한 개념을 더 쉽게 볼 수 있습니다.

그러나 두 가지 구현이 정말 동일합니까? 직관적인 가정은 더 낮은 수준의 루프가 더 빠를 것이라는 것입니다. 성능에 대해 이야기합시다.

------

## [성능 비교: 루프와 반복자](https://doc.rust-lang.org/book/ch13-04-performance.html#comparing-performance-loops-vs-iterators)

루프 또는 반복기를 사용할지 여부를 결정하려면 어떤 구현이 더 빠른지 알아야 합니다. 명시적인 `for` 루프가 있는 `검색` 함수 버전 또는 반복자가 있는 버전입니다.

*Arthur Conan Doyle 경의 Sherlock Holmes의 모험* 전체 내용을 `문자열`로 로드 하고 내용에서 *the라는* 단어를 찾아 벤치마크를 실행했습니다. 다음은 `for` 루프를 사용하는 `search` 버전과 반복자를 사용하는 버전에 대한 벤치마크 결과입니다.

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

반복자 버전이 약간 더 빠릅니다! 여기서는 벤치마크 코드에 대해 설명하지 않겠습니다. 요점은 두 버전이 동일하다는 것을 증명하는 것이 아니라 이러한 두 가지 구현이 성능 측면에서 어떻게 비교되는지에 대한 일반적인 이해를 얻는 것이기 때문입니다.

보다 포괄적인 벤치마크를 위해 다양한 크기의 다양한 텍스트를 `내용`으로, 다양한 단어와 다양한 길이의 단어를 `검색어`로 사용하고 모든 종류의 기타 변형을 사용하여 확인해야 합니다. 요점은 이것입니다. 반복자는 상위 수준 추상화이지만 하위 수준 코드를 직접 작성한 것과 거의 동일한 코드로 컴파일됩니다. 반복자는 Rust의 *제로 비용 추상화* 중 하나입니다. 즉, 추상화를 사용하면 추가 런타임 오버헤드가 발생하지 않습니다. 이는 C++의 최초 설계자이자 구현자인 Bjarne Stroustrup이 `Foundations of C++`(2012)에서 *제로 오버헤드를 정의한 방식과 유사합니다.*

> 일반적으로 C++ 구현은 제로 오버헤드 원칙을 따릅니다. 사용하지 않는 것은 비용을 지불하지 않습니다. 그리고 더 나아가, 당신이 사용하는 것을 손으로 더 잘 코딩할 수는 없습니다.

또 다른 예로, 다음 코드는 오디오 디코더에서 가져온 것입니다. 디코딩 알고리즘은 선형 예측 수학 연산을 사용하여 이전 샘플의 선형 함수를 기반으로 미래 값을 추정합니다. 이 코드는 반복자 체인을 사용하여 범위의 세 가지 변수, 데이터의 `버퍼` 조각, 12개의 `계수` 배열 및 `qlp_shift`에서 데이터를 이동할 양에 대해 일부 수학을 수행합니다. 이 예제에서 변수를 선언했지만 값은 지정하지 않았습니다. 이 코드는 문맥 밖에서는 큰 의미가 없지만 Rust가 고수준 아이디어를 저수준 코드로 변환하는 방법에 대한 간결하고 실제적인 예입니다.

```rust
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

`예측` 값을 계산하기 위해 이 코드는 `coefficients`의 12개 값 각각을 반복하고 `zip` 방법을 사용하여 `buffer`의 이전 12개 값과 계수 값을 쌍으로 만듭니다. 그런 다음 각 쌍에 대해 값을 함께 곱하고 모든 결과를 합한 다음 합계 `qlp_shift` 비트의 비트를 오른쪽으로 이동합니다.

오디오 디코더와 같은 응용 프로그램의 계산은 종종 성능을 가장 높게 우선시합니다. 여기에서는 두 개의 어댑터를 사용하여 반복자를 만든 다음 값을 소비합니다. 이 Rust 코드는 어떤 어셈블리 코드로 컴파일됩니까? 글쎄, 이 글을 쓰는 시점에서 그것은 당신이 직접 작성한 것과 같은 어셈블리로 컴파일됩니다. `계수`의 값에 대한 반복에 해당하는 루프가 전혀 없습니다. Rust는 12번의 반복이 있다는 것을 알고 있으므로 루프를 `펼칩니다`. *언롤링은* 루프 제어 코드의 오버헤드를 제거하고 대신 루프의 각 반복에 대해 반복 코드를 생성하는 최적화입니다.

모든 계수는 레지스터에 저장되므로 값에 액세스하는 것이 매우 빠릅니다. 런타임 시 배열 액세스에 대한 경계 검사가 없습니다. Rust가 적용할 수 있는 이러한 모든 최적화는 결과 코드를 매우 효율적으로 만듭니다. 이제 이것을 알았으니 두려움 없이 반복자와 클로저를 사용할 수 있습니다! 그들은 코드를 더 높은 수준으로 보이게 하지만 그렇게 한다고 해서 런타임 성능이 저하되지는 않습니다.

## [요약](https://doc.rust-lang.org/book/ch13-04-performance.html#summary)

클로저와 이터레이터는 함수형 프로그래밍 언어 아이디어에서 영감을 받은 Rust 기능입니다. 그들은 낮은 수준의 성능으로 높은 수준의 아이디어를 명확하게 표현하는 Rust의 기능에 기여합니다. 클로저와 반복자의 구현은 런타임 성능에 영향을 미치지 않습니다. 이것은 비용이 들지 않는 추상화를 제공하기 위해 노력하는 Rust의 목표의 일부입니다.

이제 I/O 프로젝트의 표현력을 개선했으므로 프로젝트를 전 세계와 공유하는 데 도움이 되는 `cargo`의 몇 가지 기능을 더 살펴보겠습니다.

------

# 14

# [Cargo 및 Crates.io에 대한 추가 정보](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html#more-about-cargo-and-cratesio)

지금까지 우리는 코드를 빌드, 실행 및 테스트하기 위해 Cargo의 가장 기본적인 기능만 사용했지만 훨씬 더 많은 일을 할 수 있습니다. 이 장에서는 다음을 수행하는 방법을 보여주는 다른 고급 기능에 대해 설명합니다.

- 릴리스 프로필을 통해 빌드 사용자 지정
- [crates.io](https://crates.io/) 에 라이브러리 게시
- 작업 공간으로 대규모 프로젝트 구성
- [crates.io](https://crates.io/) 에서 바이너리 설치
- 사용자 지정 명령을 사용하여 Cargo 확장

Cargo는 이 장에서 다루는 기능보다 더 많은 기능을 수행할 수 있으므로 Cargo의 모든 기능에 대한 전체 설명은 [설명서를](https://doc.rust-lang.org/cargo/) 참조하십시오 .

------

## [릴리스 프로필로 빌드 사용자 지정](https://doc.rust-lang.org/book/ch14-01-release-profiles.html#customizing-builds-with-release-profiles)

Rust에서 *릴리스 프로필은* 프로그래머가 코드 컴파일을 위한 다양한 옵션을 더 잘 제어할 수 있도록 하는 다양한 구성을 가진 미리 정의되고 사용자 정의 가능한 프로필입니다. 각 프로필은 서로 독립적으로 구성됩니다.

Cargo에는 두 가지 주요 프로필이 있습니다. `cargo build`를 실행할 때 Cargo가 사용하는 `dev` 프로필과 `cargo build --release`를 실행할 때 Cargo가 사용하는 `release` 프로필입니다. `dev` 프로필은 개발에 적합한 기본값으로 정의되고 `release` 프로필은 릴리스 빌드에 적합한 기본값으로 정의됩니다.

다음 프로필 이름은 빌드 출력에서 익숙할 수 있습니다.

```bash
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

`dev`와 `release`는 컴파일러에서 사용하는 서로 다른 프로필입니다.

Cargo는 `[프로필을 명시적으로 추가하지 않았을 때 적용되는 각 프로필에 대한 기본 설정을 가지고 있습니다. *]` 섹션을 프로젝트의 \*Cargo.toml\* 파일에 추가합니다. `[profile.* ]` 섹션에서 기본 설정의 하위 집합을 재정의합니다. 예를 들어 다음은 `dev` 및 `release` 프로필에 대한 `opt-level` 설정의 기본값입니다.

파일 이름: Cargo.toml

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 설정은 Rust가 코드에 적용할 최적화 수를 제어하며 범위는 0에서 3까지입니다. 더 많은 최적화를 적용하면 컴파일 시간이 연장되므로 개발 중이고 코드를 자주 컴파일하는 경우 결과 코드가 더 느리게 실행되더라도 더 적은 최적화를 통해 더 빨리 컴파일하기를 원합니다. 따라서 `dev`의 기본 `opt-level`은 `0`입니다. 코드를 릴리스할 준비가 되면 컴파일에 더 많은 시간을 할애하는 것이 가장 좋습니다. 릴리스 모드에서 한 번만 컴파일하지만 컴파일된 프로그램을 여러 번 실행하게 되므로 릴리스 모드는 더 빠르게 실행되는 코드를 위해 더 긴 컴파일 시간을 맞바꿉니다. 그렇기 때문에 `릴리스` 프로필의 기본 `옵션 수준`은 `3`입니다.

*Cargo.toml* 에 다른 값을 추가하여 기본 설정을 재정의할 수 있습니다. 예를 들어 개발 프로필에서 최적화 수준 1을 사용하려는 경우 프로젝트의 *Cargo.toml* 파일에 다음 두 줄을 추가할 수 있습니다.

파일 이름: Cargo.toml

```toml
[profile.dev]
opt-level = 1
```

이 코드는 기본 설정인 `0`을 재정의합니다. 이제 `cargo build`를 실행하면 Cargo는 `dev` 프로필의 기본값과 `opt-level`에 대한 사용자 정의를 사용합니다. `opt-level`을 `1`로 설정했기 때문에 Cargo는 기본값보다 더 많은 최적화를 적용하지만 릴리스 빌드 만큼은 아닙니다.

구성 옵션의 전체 목록과 각 프로필의 기본값은 [Cargo 문서를](https://doc.rust-lang.org/cargo/reference/profiles.html) 참조하세요 .

------

## [Crates.io에 크레이트 게시하기](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#publishing-a-crate-to-cratesio)

[우리는 crates.io](https://crates.io/) 의 패키지를 프로젝트의 종속성으로 사용했지만 자신의 패키지를 게시하여 다른 사람과 코드를 공유할 수도 있습니다. [crates.io](https://crates.io/) 의 크레이트 레지스트리는 패키지의 소스 코드를 배포하므로 주로 오픈 소스 코드를 호스팅합니다.

Rust 및 Cargo에는 게시된 패키지를 사람들이 더 쉽게 찾고 사용할 수 있도록 하는 기능이 있습니다. 다음에는 이러한 기능 중 일부에 대해 설명하고 패키지를 게시하는 방법에 대해 설명합니다.

### [유용한 문서 주석 만들기](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#making-useful-documentation-comments)

패키지를 정확하게 문서화하면 다른 사용자가 패키지를 사용하는 방법과 시기를 알 수 있으므로 문서 작성에 시간을 투자할 가치가 있습니다. 3장에서 우리는 두 개의 슬래시 `//`를 사용하여 Rust 코드를 주석 처리하는 방법에 대해 논의했습니다. Rust는 또한 HTML 문서를 생성하는 *문서 주석* 으로 편리하게 알려진 문서에 대한 특정 종류의 주석을 가지고 있습니다. HTML은 크레이트 *구현 방법이 아닌 크레이트* *사용* 방법에 관심이 있는 프로그래머를 위한 공개 API 항목에 대한 문서 주석 내용을 표시합니다.

문서 주석은 2개가 아닌 3개의 슬래시 `///`를 사용하며 텍스트 서식 지정을 위한 Markdown 표기법을 지원합니다. 문서화 중인 항목 바로 앞에 문서 주석을 배치합니다. 목록 14-1은 `my_crate`라는 크레이트의 `add_one` 함수에 대한 문서 주석을 보여줍니다.

파일 이름: src/lib.rs

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

목록 14-1: 함수에 대한 문서화 주석

여기에서 `add_one` 함수가 수행하는 작업에 대한 설명을 제공하고 `예제`라는 제목으로 섹션을 시작한 다음 `add_one` 함수를 사용하는 방법을 보여주는 코드를 제공합니다. `cargo doc`를 실행하여 이 문서 주석에서 HTML 문서를 생성할 수 있습니다. 이 명령은 Rust와 함께 배포되는 `rustdoc` 도구를 실행하고 생성된 HTML 문서를 *target/doc* 디렉토리에 넣습니다.

편의를 위해 `cargo doc --open`을 실행하면 현재 크레이트의 문서(크레이트의 모든 종속 항목에 대한 문서도 포함)에 대한 HTML을 빌드하고 웹 브라우저에서 결과를 엽니다. `add_one` 함수로 이동하면 그림 14-1과 같이 문서 주석의 텍스트가 어떻게 렌더링되는지 확인할 수 있습니다.

![`my_crate`의 `add_one` 기능에 대한 렌더링된 HTML 문서](https://doc.rust-lang.org/book/img/trpl14-01.png)

그림 14-1: `add_one` 기능에 대한 HTML 문서

#### [일반적으로 사용되는 섹션](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#commonly-used-sections)

Listing 14-1에서 `# Examples` Markdown 제목을 사용하여 `Examples`라는 제목의 HTML 섹션을 만들었습니다. 크레이트 작성자가 문서에서 일반적으로 사용하는 다른 섹션은 다음과 같습니다.

- **Panics** : 문서화 중인 기능이 패닉할 수 있는 시나리오입니다. 프로그램 패닉을 원하지 않는 함수 호출자는 이러한 상황에서 함수를 호출하지 않도록 해야 합니다.
- **오류** : 함수가 `결과`를 반환하는 경우 발생할 수 있는 오류 종류와 이러한 오류가 반환될 수 있는 조건을 설명하면 호출자가 다른 종류의 오류를 다른 방식으로 처리하는 코드를 작성할 수 있도록 도움이 될 수 있습니다.
- **안전** : 함수가 호출하기에 `안전하지 않은` 경우(19장에서 안전하지 않음에 대해 설명함) 함수가 안전하지 않은 이유를 설명하고 함수가 호출자가 유지하기를 기대하는 불변성을 다루는 섹션이 있어야 합니다.

대부분의 문서 주석에는 이러한 섹션이 모두 필요하지 않지만 이것은 사용자가 관심을 가질 코드 측면을 상기시키는 좋은 체크리스트입니다.

#### [테스트로서의 문서 주석](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests)

문서 주석에 예제 코드 블록을 추가하면 라이브러리 사용 방법을 시연하는 데 도움이 될 수 있으며 추가 보너스가 있습니다. `cargo test`를 실행하면 문서의 코드 예제가 테스트로 실행됩니다! 예제가 있는 문서보다 더 좋은 것은 없습니다. 그러나 문서가 작성된 이후 코드가 변경되었기 때문에 작동하지 않는 예제보다 더 나쁜 것은 없습니다. Listing 14-1의 `add_one` 함수에 대한 문서와 함께 `cargo test`를 실행하면 테스트 결과에 다음과 같은 섹션이 표시됩니다.

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

이제 함수나 예제를 변경하면 `assert_eq!` 예제 패닉에서 `cargo test`를 다시 실행하면 문서 테스트가 예제와 코드가 서로 동기화되지 않은 것을 포착하는 것을 볼 수 있습니다!

#### [포함된 항목에 주석 달기](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#commenting-contained-items)

문서 주석 `//!`의 스타일 주석 뒤에 오는 항목이 아니라 주석이 포함된 항목에 문서를 추가합니다. 우리는 일반적으로 크레이트 루트 파일( 협약에 따라 *src/lib.rs* ) 또는 모듈 내부에서 이러한 문서 주석을 사용하여 크레이트 또는 모듈 전체를 문서화합니다.

예를 들어 `add_one` 기능이 포함된 `my_crate` 크레이트의 목적을 설명하는 문서를 추가하려면 `//!`로 시작하는 문서 주석을 추가합니다. 목록 14-2에 표시된 것처럼 *src/lib.rs* 파일 의 시작 부분에 :

파일 이름: src/lib.rs

```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

목록 14-2: `my_crate` 크레이트 전체에 대한 문서

`//!`로 시작하는 마지막 줄 뒤에는 코드가 없습니다. `//!`로 주석을 시작했기 때문입니다. `///` 대신 이 주석 뒤에 오는 항목이 아니라 이 주석이 포함된 항목을 문서화합니다. 이 경우 해당 항목은 크레이트 루트인 *src/lib.rs 파일입니다.* 이 주석은 전체 크레이트를 설명합니다.

`cargo doc --open`을 실행하면 그림 14-2와 같이 크레이트의 공개 항목 목록 위에 있는 `my_crate` 문서의 첫 페이지에 이러한 주석이 표시됩니다.

![크레이트 전체에 대한 주석이 포함된 렌더링된 HTML 문서](https://doc.rust-lang.org/book/img/trpl14-02.png)

그림 14-2: 상자 전체를 설명하는 주석을 포함하여 `my_crate`에 대한 렌더링된 문서

항목 내의 문서 주석은 특히 크레이트와 모듈을 설명하는 데 유용합니다. 사용자가 크레이트 구성을 이해하는 데 도움이 되도록 컨테이너의 전반적인 목적을 설명하는 데 사용하세요.

### [`pub use`로 편리한 공개 API 내보내기](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use)

공개 API의 구조는 크레이트를 게시할 때 주요 고려 사항입니다. 당신의 상자를 사용하는 사람들은 당신보다 구조에 익숙하지 않으며 상자에 큰 모듈 계층이 있는 경우 사용하려는 조각을 찾는 데 어려움을 겪을 수 있습니다.

7장에서 `pub` 키워드를 사용하여 항목을 공개하는 방법과 `use` 키워드를 사용하여 항목을 범위로 가져오는 방법을 다루었습니다. 그러나 크레이트를 개발하는 동안 이해하기 쉬운 구조가 사용자에게는 그다지 편리하지 않을 수 있습니다. 여러 수준을 포함하는 계층 구조에서 구조를 구성하려고 할 수 있지만 계층 구조 깊숙이 정의한 유형을 사용하려는 사람들은 해당 유형이 존재하는지 찾는 데 어려움을 겪을 수 있습니다. 또한 `use` `my_crate::some_module::another_module::UsefulType;`을 입력해야 하는 것에 짜증이 날 수도 있습니다. `my_crate::UsefulType;`을 `사용`하는 대신.

좋은 소식은 다른 사람이 다른 라이브러리에서 구조를 사용하는 것이 편리 *하지 않은* 경우 내부 조직을 재정렬할 필요가 없다는 것입니다. 대신 항목을 다시 내보내 개인 구조와 다른 공용 구조를 만들 수 있습니다. `펍 사용`을 사용하여. 다시 내보내기는 한 위치에서 공용 항목을 가져와 다른 위치에서 정의된 것처럼 다른 위치에서 공용으로 만듭니다.

예를 들어 예술적 개념을 모델링하기 위해 `art`라는 이름의 라이브러리를 만들었다고 가정합니다. 이 라이브러리에는 두 개의 모듈이 있습니다. `PrimaryColor`와 `SecondaryColor`라는 두 개의 열거형을 포함하는 `kinds` 모듈과 `mix`라는 함수를 포함하는 `utils` 모듈입니다. 목록 14-3에 나와 있습니다.

파일 이름: src/lib.rs

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

목록 14-3: `kinds` 및 `utils` 모듈로 구성된 항목이 있는 `art` 라이브러리

그림 14-3은 `cargo doc`에 의해 생성된 이 상자에 대한 문서의 첫 페이지가 어떻게 생겼는지 보여줍니다.

![`kinds` 및 `utils` 모듈을 나열하는 `art` 크레이트에 대한 렌더링된 문서](https://doc.rust-lang.org/book/img/trpl14-03.png)

그림 14-3: `kinds` 및 `utils` 모듈을 나열하는 `art` 문서의 첫 페이지

`PrimaryColor` 및 `SecondaryColor` 유형은 첫 페이지에 나열되지 않으며 `mix` 기능도 없습니다. `종류`와 `유틸`을 클릭해야 볼 수 있습니다.

이 라이브러리에 의존하는 또 다른 크레이트는 현재 정의된 모듈 구조를 지정하여 `art`의 항목을 범위로 가져오는 `use` 문이 필요합니다. 목록 14-4는 `art` 크레이트의 `PrimaryColor` 및 `mix` 항목을 사용하는 크레이트의 예를 보여줍니다:

파일 이름: src/main.rs

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

목록 14-4: 내부 구조를 내보낸 `art` 크레이트의 항목을 사용하는 크레이트

`art` 크레이트를 사용하는 목록 14-4의 코드 작성자는 `PrimaryColor`가 `kinds` 모듈에 있고 `mix`가 `utils` 모듈에 있음을 알아내야 했습니다. `art` 크레이트의 모듈 구조는 `art` 크레이트를 사용하는 개발자보다 `art` 크레이트에서 작업하는 개발자와 더 관련이 있습니다. 내부 구조는 `아트` 크레이트를 사용하는 방법을 이해하려는 사람에게 유용한 정보를 포함하지 않고 오히려 혼란을 야기합니다. 왜냐하면 이를 사용하는 개발자는 어디를 봐야 하는지 파악해야 하고 모듈 이름을 ` 사용` 문.

공개 API에서 내부 조직을 제거하기 위해 Listing 14-3의 `art` 크레이트 코드를 수정하여 Listing 14-5와 같이 최상위 수준에서 항목을 다시 내보내는 `pub use` 문을 추가할 수 있습니다.

파일 이름: src/lib.rs

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

목록 14-5: 항목을 다시 내보내는 데 `pub use` 문 추가

`cargo doc`이 이 크레이트에 대해 생성하는 API 문서는 이제 그림 14-4와 같이 첫 페이지에 재수출을 나열하고 링크하여 `PrimaryColor` 및 `SecondaryColor` 유형과 `mix` 기능을 더 쉽게 만듭니다. 찾다.

![첫 페이지에 재수출된 `아트` 상자에 대한 렌더링된 문서](https://doc.rust-lang.org/book/img/trpl14-04.png)

그림 14-4: 재수출을 나열한 `art` 문서의 첫 페이지

`art` 크레이트 사용자는 여전히 Listing 14-4에서 보여지는 것처럼 Listing 14-3의 내부 구조를 보고 사용할 수 있거나 Listing 14-6에서 보여지는 것처럼 Listing 14-5에서 더 편리한 구조를 사용할 수 있습니다:

파일 이름: src/main.rs

```rust
use art::mix;
use art::PrimaryColor;

fn main() {
    // --snip--
}
```

목록 14-6: `art` 크레이트에서 다시 내보낸 항목을 사용하는 프로그램

중첩된 모듈이 많은 경우 `pub use`를 사용하여 최상위 수준에서 유형을 다시 내보내면 크레이트를 사용하는 사람들의 경험에 상당한 차이를 만들 수 있습니다. `pub use`의 또 다른 일반적인 용도는 해당 크레이트의 정의를 크레이트의 공개 API의 일부로 만들기 위해 현재 크레이트의 종속성 정의를 다시 내보내는 것입니다.

유용한 공개 API 구조를 만드는 것은 과학이라기보다는 예술에 가깝고 사용자에게 가장 적합한 API를 찾기 위해 반복할 수 있습니다. `pub use`를 선택하면 크레이트를 내부적으로 구성하는 방법에 유연성을 제공하고 내부 구조를 사용자에게 제공하는 것과 분리합니다. 내부 구조가 공개 API와 다른지 확인하려면 설치한 크레이트 코드를 살펴보세요.

### [Crates.io 계정 설정](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#setting-up-a-cratesio-account)

크레이트를 게시하려면 [crates.io](https://crates.io/) 에서 계정을 만들고 API 토큰을 받아야 합니다. [그렇게 하려면 crates.io](https://crates.io/) 홈페이지를 방문하여 GitHub 계정을 통해 로그인하십시오. (GitHub 계정은 현재 요구 사항이지만 사이트는 향후 계정을 만드는 다른 방법을 지원할 수 있습니다.) 로그인한 후 https://crates.io/me/에서 계정 설정을 방문하여 계정을 검색하십시오. API 키. 그런 다음 API 키를 사용하여 다음과 같이 `cargo login` 명령을 실행합니다.

```bash
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

이 명령은 API 토큰을 Cargo에 알리고 *~/.cargo/credentials* 에 로컬로 저장합니다. *이 토큰은 비밀* 입니다. 다른 사람과 공유하지 마십시오. 어떤 이유로든 다른 사람과 공유하는 경우 이를 취소하고 [crates.io](https://crates.io/) 에서 새 토큰을 생성해야 합니다.

### [새 크레이트에 메타데이터 추가](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#adding-metadata-to-a-new-crate)

게시하려는 크레이트가 있다고 가정해 보겠습니다. *게시하기 전에 크레이트 Cargo.toml* 파일 의 `[패키지]` 섹션에 일부 메타데이터를 추가해야 합니다.

상자에는 고유한 이름이 필요합니다. 로컬에서 크레이트 작업을 하는 동안 크레이트 이름을 원하는 대로 지정할 수 있습니다. 그러나 [crates.io](https://crates.io/) 의 크레이트 이름은 선착순으로 할당됩니다. 크레이트 이름이 지정되면 아무도 해당 이름으로 크레이트를 게시할 수 없습니다. 크레이트 게시를 시도하기 전에 사용하려는 이름을 검색하세요. *이름이 사용된 경우 다른 이름을 찾아 `[패키지]` 섹션 아래의 Cargo.toml* 파일에서 `이름` 필드를 편집하여 다음과 같이 게시에 새 이름을 사용해야 합니다.

파일 이름: Cargo.toml

```toml
[package]
name = `guessing_game`
```

고유한 이름을 선택했더라도 이 시점에서 크레이트를 게시하기 위해 `cargo publish`를 실행하면 경고가 표시된 다음 오류가 표시됩니다.

```bash
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata
```

이 오류는 몇 가지 중요한 정보를 누락했기 때문에 발생합니다. 사람들이 상자의 기능과 사용할 수 있는 조건을 알 수 있도록 설명과 라이선스가 필요합니다. *Cargo.toml* 에서 검색 결과에 크레이트와 함께 표시될 것이기 때문에 한 두 문장에 불과한 설명을 추가하십시오. `라이센스` 필드의 경우 *라이센스 식별자 값을* 제공해야 합니다. Linux Foundation의 소프트웨어 패키지 [데이터 Exchange(SPDX)는](http://spdx.org/licenses/) 이 값에 사용할 수 있는 식별자를 나열합니다. 예를 들어 MIT 라이선스를 사용하여 상자에 라이선스를 부여했음을 지정하려면 `MIT` 식별자를 추가합니다.

파일 이름: Cargo.toml

```toml
[package]
name = `guessing_game`
license = `MIT`
```

SPDX에 표시되지 않는 라이선스를 사용하려면 해당 라이선스의 텍스트를 파일에 넣고 프로젝트에 파일을 포함시킨 다음 `license-file`을 사용하여 해당 라이선스의 이름을 지정해야 합니다. `라이센스` 키를 사용하는 대신 파일.

귀하의 프로젝트에 적합한 라이선스에 대한 지침은 이 책의 범위를 벗어납니다. Rust 커뮤니티의 많은 사람들은 `MIT OR Apache-2.0`의 이중 라이선스를 사용하여 Rust와 동일한 방식으로 프로젝트에 라이선스를 부여합니다. 이 방법은 `OR`로 구분된 여러 라이센스 식별자를 지정하여 프로젝트에 대한 여러 라이센스를 가질 수도 있음을 보여줍니다.

고유한 이름, 버전, 설명 및 라이선스가 추가된 경우 게시할 준비가 된 프로젝트의 *Cargo.toml* 파일은 다음과 같습니다.

파일 이름: Cargo.toml

```toml
[package]
name = `guessing_game`
version = `0.1.0`
edition = `2021`
description = `A fun game where you guess what number the computer has chosen.`
license = `MIT OR Apache-2.0`

[dependencies]
```

[Cargo의 문서는](https://doc.rust-lang.org/cargo/) 다른 사람들이 크레이트를 더 쉽게 발견하고 사용할 수 있도록 지정할 수 있는 다른 메타데이터에 대해 설명합니다.

### [Crates.io에 게시](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#publishing-to-cratesio)

이제 계정을 만들고 API 토큰을 저장하고 크레이트 이름을 선택하고 필수 메타데이터를 지정했으므로 게시할 준비가 되었습니다! 크레이트를 게시하면 다른 사람들이 사용할 수 있도록 [crates.io 에 특정 버전이 업로드됩니다.](https://crates.io/)

게시는 *영구적* 이므로 주의하십시오 . 버전을 덮어쓸 수 없으며 코드를 삭제할 수 없습니다. [crates.io](https://crates.io/) 의 주요 목표 중 하나는 영구 코드 아카이브 역할을 하여 [crates.io](https://crates.io/) 의 크레이트에 의존하는 모든 프로젝트의 빌드가 계속 작동하도록 하는 것입니다. 버전 삭제를 허용하면 해당 목표를 달성할 수 없게 됩니다. 그러나 게시할 수 있는 크레이트 버전의 수에는 제한이 없습니다.

`cargo publish` 명령을 다시 실행하십시오. 이제 성공해야 합니다.

```bash
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

축하해요! 이제 당신의 코드를 Rust 커뮤니티와 공유했고 누구나 쉽게 크레이트를 프로젝트의 의존성으로 추가할 수 있습니다.

### [기존 크레이트의 새 버전 게시](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#publishing-a-new-version-of-an-existing-crate)

크레이트를 변경하고 새 버전을 출시할 준비가 되면 *Cargo.toml* 파일에 지정된 `버전` 값을 변경하고 다시 게시합니다. [Semantic Versioning 규칙을](http://semver.org/) 사용하여 수행한 변경 종류에 따라 적절한 다음 버전 번호를 결정합니다. 그런 다음 `cargo publish`를 실행하여 새 버전을 업로드합니다.

### [`cargo yank`가 포함된 Crates.io 버전 사용 중단](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#deprecating-versions-from-cratesio-with-cargo-yank)

크레이트의 이전 버전을 제거할 수는 없지만 향후 프로젝트에서 크레이트를 새 종속성으로 추가하는 것을 방지할 수 있습니다. 이것은 이런저런 이유로 크레이트 버전이 깨졌을 때 유용합니다. 이러한 상황에서 Cargo는 크레이트 버전 *끌어오기를 지원합니다.*

버전을 제거하면 새 프로젝트가 해당 버전에 의존하는 것을 방지하는 동시에 해당 버전에 의존하는 모든 기존 프로젝트를 계속할 수 있습니다. 기본적으로 복사는 *Cargo.lock* 이 있는 모든 프로젝트가 중단되지 않으며 향후 생성되는 모든 *Cargo.lock* 파일이 복사된 버전을 사용하지 않음을 의미합니다.

크레이트 버전을 가져오려면 이전에 게시한 크레이트의 디렉토리에서 `cargo yank`를 실행하고 가져오려는 버전을 지정합니다. 예를 들어 `guessing_game` 버전 1.0.1이라는 이름의 크레이트를 게시하고 `guessing_game`의 프로젝트 디렉토리에서 이를 제거하려는 경우 다음을 실행합니다.

```bash
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

명령에 `--undo`를 추가하면 끌어오기를 실행 취소하고 버전에 따라 프로젝트를 다시 시작할 수 있습니다.

```bash
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

잡아당기기는 어떤 코드도 삭제 *하지 않습니다.* 예를 들어 실수로 업로드된 비밀은 삭제할 수 없습니다. 그런 일이 발생하면 해당 암호를 즉시 재설정해야 합니다.

------

## [화물 작업 공간](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#cargo-workspaces)

12장에서 우리는 바이너리 크레이트와 라이브러리 크레이트를 포함하는 패키지를 만들었습니다. 프로젝트가 진행됨에 따라 라이브러리 크레이트가 계속 커지고 패키지를 여러 라이브러리 크레이트로 더 분할하고 싶을 수 있습니다. Cargo는 함께 개발된 여러 관련 패키지를 관리하는 데 도움이 되는 *작업 공간* 이라는 기능을 제공합니다.

### [작업 공간 만들기](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#creating-a-workspace)

작업 *공간 은 동일한* *Cargo.lock* 및 출력 디렉토리를 공유하는 패키지 세트입니다. 작업 공간을 사용하여 프로젝트를 만들어 봅시다. 작업 공간의 구조에 집중할 수 있도록 간단한 코드를 사용할 것입니다. 작업 공간을 구성하는 방법은 여러 가지가 있으므로 일반적인 방법 하나만 보여드리겠습니다. 바이너리와 두 개의 라이브러리가 포함된 작업 공간이 있습니다. 주요 기능을 제공할 바이너리는 두 라이브러리에 따라 달라집니다. 하나 라이브러리는 `add_one` 기능을 제공하고 두 번째 라이브러리는 `add_two` 기능을 제공합니다. 이 세 크레이트는 동일한 작업 공간의 일부가 됩니다. 작업 공간을 위한 새 디렉토리를 생성하여 시작하겠습니다.

```bash
$ mkdir add
$ cd add
```

다음으로 *add* 디렉토리에서 전체 작업 공간을 구성할 *Cargo.toml* 파일을 생성합니다. 이 파일에는 `[패키지]` 섹션이 없습니다. 대신 바이너리 크레이트가 있는 패키지 경로를 지정하여 작업 공간에 구성원을 추가할 수 있는 `[workspace]` 섹션으로 시작합니다. 이 경우 해당 경로는 *adder* 입니다.

파일 이름: Cargo.toml

```toml
[workspace]

members = [
    `adder`,
]
```

*다음으로 add* 디렉토리 내에서 `cargo new`를 실행하여 `adder` 바이너리 크레이트를 생성합니다.

```bash
$ cargo new adder
     Created binary (application) `adder` package
```

이 시점에서 `cargo build`를 실행하여 작업 공간을 빌드할 수 있습니다. *추가* 디렉토리 의 파일은 다음과 같아야 합니다.

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

작업공간에는 컴파일된 아티팩트가 배치될 최상위 레벨에 하나의 *대상 디렉토리가 있습니다.* *`adder` 패키지에는 자체 대상* 디렉토리 가 없습니다. *adder* 디렉토리 내부에서 `cargo build`를 실행하더라도 컴파일된 아티팩트는 여전히 *add/adder/target* 이 아닌 add */* target 으로 끝납니다. Cargo 는 작업 공간의 상자가 서로 의존하기 때문에 작업 공간의 *대상* 디렉토리를 이와 같이 구성합니다. 각 크레이트에 자체 *대상 디렉토리가 있는 경우 각 크레이트는 자체* *대상* 에 아티팩트를 배치하기 위해 작업 공간의 다른 각 크레이트를 다시 컴파일해야 합니다.예배 규칙서. *하나 의 대상* 디렉토리를 공유함으로써 크레이트는 불필요한 재구축을 피할 수 있습니다.

### [작업 공간에서 두 번째 패키지 만들기](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#creating-the-second-package-in-the-workspace)

다음으로 작업 공간에 다른 구성원 패키지를 만들고 이름을 `add_one`으로 지정하겠습니다. 최상위 *Cargo.toml을* 변경하여 `구성원` 목록에서 *add_one* 경로를 지정합니다.

파일 이름: Cargo.toml

```toml
[workspace]

members = [
    `adder`,
    `add_one`,
]
```

그런 다음 `add_one`이라는 새 라이브러리 크레이트를 생성합니다.

```bash
$ cargo new add_one --lib
     Created library `add_one` package
```

*이제 추가* 디렉토리 에 다음 디렉토리와 파일이 있어야 합니다.

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

*add_one/src/lib.rs* 파일 에서 `add_one` 함수를 추가해 보겠습니다.

파일 이름: add_one/src/lib.rs

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

이제 라이브러리가 있는 `add_one` 패키지에 의존하는 바이너리가 포함된 `adder` 패키지를 가질 수 있습니다. *먼저 adder/Cargo.toml* 에 `add_one`에 대한 경로 종속성을 추가해야 합니다.

파일 이름: adder/Cargo.toml

```toml
[dependencies]
add_one = { path = `../add_one` }
```

Cargo는 작업 공간의 크레이트가 서로 의존할 것이라고 가정하지 않으므로 종속 관계에 대해 명시적으로 설명해야 합니다.

다음으로, `adder` 크레이트에서 `add_one` 함수(`add_one` 크레이트에서)를 사용합시다. *adder/src/main.rs* 파일을 열고 상단에 `use` 줄을 추가하여 새로운 `add_one` 라이브러리 크레이트를 범위로 가져옵니다. 그런 다음 Listing 14-7과 같이 `main` 함수를 `add_one` 함수를 호출하도록 변경합니다.

파일 이름: adder/src/main.rs

```rust
use add_one;

fn main() {
    let num = 10;
    println!(`Hello, world! {num} plus one is {}!`, add_one::add_one(num));
}
```

Listing 14-7: `adder` 크레이트에서 `add_one` 라이브러리 크레이트 사용하기

최상위 *add* 디렉토리에서 `cargo build`를 실행하여 워크스페이스를 빌드하자!

```bash
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

*add* 디렉토리 에서 바이너리 크레이트를 실행하려면 `-p` 인수와 `cargo run`이 포함된 패키지 이름을 사용하여 작업 공간에서 실행할 패키지를 지정할 수 있습니다.

```bash
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

`add_one` 크레이트에 의존하는 *adder/src/main.rs* 의 코드를 실행합니다.

#### [작업 공간의 외부 패키지에 따라](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#depending-on-an-external-package-in-a-workspace)

작업공간에는 각 크레이트의 디렉토리에 *Cargo.lock이* 있는 대신 최상위 수준에 Cargo.lock 파일이 *하나만 있습니다.* 이렇게 하면 모든 크레이트가 동일한 버전의 모든 종속성을 사용하게 됩니다. *`rand` 패키지를 adder/Cargo.toml* 및 *add_one/Cargo.toml* 파일 에 추가하면 Cargo는 이 두 가지를 하나의 `rand` 버전으로 해석하고 하나의 *Cargo.lock* 에 기록합니다. 작업 공간의 모든 크레이트가 동일한 종속성을 사용하도록 하면 크레이트가 항상 서로 호환됩니다. *add_one/Cargo.toml* 파일 의 `[dependencies]` 섹션에 `rand` 크레이트를 추가하여 `add_one` 크레이트에서 `rand` 크레이트를 사용할 수 있도록 합시다:

파일 이름: add_one/Cargo.toml

```toml
[dependencies]
rand = `0.8.5`
```

이제 `use rand;`를 추가할 수 있습니다. *add_one/src/lib.rs* 파일 에 추가 하고 *add* 디렉터리 에서 `cargo build`를 실행하여 전체 작업 공간을 빌드하면 `rand` 크레이트를 가져와 컴파일합니다. 우리는 우리가 범위로 가져온 `rand`를 참조하지 않기 때문에 한 가지 경고를 받게 됩니다.

```bash
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s
```

최상위 *Cargo.lock은* 이제 `rand`에 대한 `add_one`의 종속성에 대한 정보를 포함합니다. *그러나 `rand`가 작업 공간 어딘가에서 사용되더라도 Cargo.toml* 파일에도 `rand`를 추가하지 않는 한 작업 공간의 다른 크레이트에서 사용할 수 없습니다. 예를 들어 `use rand;`를 추가하면 `adder` 패키지의 *adder/src/main.rs* 파일 에 오류가 발생합니다.

```bash
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

이 문제를 해결하려면 `adder` 패키지에 대한 *Cargo.toml* 파일을 편집하고 `rand`도 이에 대한 종속성을 나타내십시오. *`adder` 패키지를 빌드하면 Cargo.lock* 의 `adder` 종속성 목록에 `rand`가 추가되지만 `rand`의 추가 사본은 다운로드되지 않습니다. Cargo는 `rand` 패키지를 사용하는 작업공간의 모든 패키지에 있는 모든 크레이트가 동일한 버전을 사용하도록 보장하여 공간을 절약하고 작업공간의 크레이트가 서로 호환되도록 합니다.

#### [작업 공간에 테스트 추가](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#adding-a-test-to-a-workspace)

또 다른 향상을 위해 `add_one` 크레이트 내에 `add_one::add_one` 함수 테스트를 추가해 보겠습니다.

파일 이름: add_one/src/lib.rs

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

이제 최상위 *추가* 디렉토리에서 `cargo test`를 실행하십시오. 다음과 같이 구성된 작업 공간에서 `화물 테스트`를 실행하면 작업 공간의 모든 상자에 대한 테스트가 실행됩니다.

```bash
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.27s
     Running unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

출력의 첫 번째 섹션은 `add_one` 크레이트의 `it_works` 테스트가 통과되었음을 보여줍니다. 다음 섹션에서는 `adder` 크레이트에서 0개의 테스트가 발견되었음을 보여주고 마지막 섹션에서는 `add_one` 크레이트에서 0개의 문서 테스트가 발견되었음을 보여줍니다.

또한 `-p` 플래그를 사용하고 테스트하려는 크레이트의 이름을 지정하여 최상위 디렉터리의 작업 공간에서 특정 크레이트 하나에 대한 테스트를 실행할 수 있습니다.

```bash
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

이 출력은 `cargo test`가 `add_one` 크레이트에 대한 테스트만 실행했고 `adder` 크레이트 테스트는 실행하지 않았음을 보여줍니다.

작업 공간의 크레이트를 [crates.io](https://crates.io/) 에 게시하는 경우 작업 공간의 각 크레이트를 별도로 게시해야 합니다. `화물 테스트`와 마찬가지로 `-p` 플래그를 사용하고 게시하려는 크레이트의 이름을 지정하여 작업 공간에 특정 크레이트를 게시할 수 있습니다.

추가 연습을 위해 `add_one` 크레이트와 유사한 방식으로 이 작업 공간에 `add_two` 크레이트를 추가합니다!

프로젝트가 커짐에 따라 작업 공간 사용을 고려하십시오. 하나의 큰 코드 덩어리보다 작은 개별 구성 요소를 이해하는 것이 더 쉽습니다. 또한 상자를 작업 공간에 보관하면 동시에 자주 변경되는 경우 상자 간의 조정이 더 쉬워질 수 있습니다.

------

## [`cargo install`로 바이너리 설치하기](https://doc.rust-lang.org/book/ch14-04-installing-binaries.html#installing-binaries-with-cargo-install)

`cargo install` 명령을 사용하면 바이너리 크레이트를 로컬에 설치하고 사용할 수 있습니다. 이는 시스템 패키지를 대체하기 위한 것이 아닙니다. [이는 Rust 개발자가 다른 사람들이 crates.io](https://crates.io/) 에서 공유한 도구를 설치하는 편리한 방법을 의미합니다. 바이너리 대상이 있는 패키지만 설치할 수 있습니다. 바이너리 *타겟은* 크레이트에 *src/main.rs* 파일이나 바이너리로 지정된 다른 파일이 있는 경우 생성되는 실행 가능한 프로그램입니다. 프로그램들. *일반적으로 크레이트는 README* 파일에 크레이트가 라이브러리인지, 바이너리 타겟이 있는지 또는 둘 다에 대한 정보를 가지고 있습니다.

`cargo install`로 설치된 모든 바이너리는 설치 루트의 *bin* 폴더에 저장됩니다. *Rustup.rs를* 사용하여 Rust를 설치했고 사용자 지정 구성이 없는 경우 이 디렉터리는 *$HOME/.cargo/bin* 이 됩니다. `cargo install`로 설치한 프로그램을 실행할 수 있도록 디렉토리가 `$PATH`에 있는지 확인하십시오.

예를 들어, 12장에서 파일 검색을 위한 `ripgrep`이라는 `grep` 도구의 Rust 구현이 있다고 언급했습니다. `ripgrep`을 설치하려면 다음을 실행할 수 있습니다.

```bash
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
```

출력의 마지막에서 두 번째 줄에는 설치된 바이너리의 위치와 이름이 표시되며 `ripgrep`의 경우 `rg`입니다. 이전에 언급한 것처럼 설치 디렉토리가 `$PATH`에 있는 한 `rg --help`를 실행하고 파일 검색을 위한 더 빠르고 더 강력한 도구를 사용할 수 있습니다!

------

## [사용자 지정 명령으로 Cargo 확장](https://doc.rust-lang.org/book/ch14-05-extending-cargo.html#extending-cargo-with-custom-commands)

Cargo는 Cargo를 수정할 필요 없이 새로운 하위 명령으로 확장할 수 있도록 설계되었습니다. `$PATH`의 바이너리 이름이 `cargo-something`인 경우 `cargo something`을 실행하여 Cargo 하위 명령인 것처럼 실행할 수 있습니다. 이와 같은 사용자 지정 명령은 `cargo --list`를 실행할 때도 나열됩니다. `cargo install`을 사용하여 확장 기능을 설치한 다음 내장 Cargo 도구처럼 실행할 수 있다는 것은 Cargo 디자인의 매우 편리한 이점입니다!

## [요약](https://doc.rust-lang.org/book/ch14-05-extending-cargo.html#summary)

Cargo 및 [crates.io](https://crates.io/) 와 코드를 공유하는 것은 Rust 생태계를 다양한 작업에 유용하게 만드는 요소 중 하나입니다. Rust의 표준 라이브러리는 작고 안정적이지만 크레이트는 언어와 다른 타임라인에서 쉽게 공유, 사용 및 개선할 수 있습니다. 당신에게 유용한 코드를 [crates.io](https://crates.io/) 에 공유하는 것을 부끄러워하지 마세요 . 다른 사람에게도 유용할 것 같습니다!

------

# 15

# [스마트 포인터](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html#smart-pointers)

포인터 *는* 메모리의 주소를 포함하는 변수에 대한 일반적인 개념입니다. 이 주소는 다른 데이터를 참조하거나 `가리키는` 것입니다. Rust에서 가장 일반적인 종류의 포인터는 4장에서 배웠던 참조입니다. 참조는 `&` 기호로 표시되며 가리키는 값을 빌립니다. 데이터를 참조하는 것 외에 특별한 기능이 없으며 오버헤드가 없습니다.

반면에 *스마트 포인터는 포인터처럼 작동하지만 추가 메타데이터 및 기능도 포함하는 데이터 구조입니다.* 스마트 포인터의 개념은 Rust에만 있는 것이 아닙니다. 스마트 포인터는 C++에서 시작되었고 다른 언어에도 존재합니다. Rust는 참조에서 제공하는 것 이상의 기능을 제공하는 표준 라이브러리에 정의된 다양한 스마트 포인터를 가지고 있습니다. *일반적인 개념을 살펴보기 위해 참조 카운팅* 스마트 포인터 유형 을 포함하여 스마트 포인터의 몇 가지 다른 예를 살펴보겠습니다. 이 포인터를 사용하면 소유자 수를 추적하고 소유자가 남아 있지 않으면 데이터를 정리하여 데이터가 여러 소유자를 갖도록 허용할 수 있습니다.

소유권과 차용 개념이 있는 Rust는 참조와 스마트 포인터 사이에 또 다른 차이점이 있습니다. 참조는 데이터만 빌릴 수 있지만 대부분의 경우 스마트 포인터는 자신이 가리키는 데이터를 *소유합니다.*

당시에는 그렇게 부르지 않았지만 이 책에서 `String` 및 `Vec`을 포함하여 몇 가지 스마트 포인터를 이미 접했습니다.` 8장에서. 이 두 유형 모두 일부 메모리를 소유하고 조작할 수 있기 때문에 스마트 포인터로 간주됩니다. 또한 메타데이터와 추가 기능 또는 보증이 있습니다. 예를 들어 `문자열`은 용량을 메타데이터로 저장하고 추가 데이터가 항상 유효한 UTF-8인지 확인하는 기능.

스마트 포인터는 일반적으로 구조체를 사용하여 구현됩니다. 일반 구조체와 달리 스마트 포인터는 `Deref` 및 `Drop` 특성을 구현합니다. `Deref` 특성을 사용하면 스마트 포인터 구조체의 인스턴스가 참조처럼 동작할 수 있으므로 참조 또는 스마트 포인터와 함께 작동하도록 코드를 작성할 수 있습니다. `드롭` 특성을 사용하면 스마트 포인터의 인스턴스가 범위를 벗어날 때 실행되는 코드를 사용자 지정할 수 있습니다. 이 장에서는 두 가지 특성에 대해 논의하고 스마트 포인터에 중요한 이유를 설명합니다.

스마트 포인터 패턴이 Rust에서 자주 사용되는 일반적인 디자인 패턴이라는 점을 감안할 때, 이 장에서는 기존의 모든 스마트 포인터를 다루지는 않을 것입니다. 많은 라이브러리에는 자체 스마트 포인터가 있으며 직접 작성할 수도 있습니다. 표준 라이브러리에서 가장 일반적인 스마트 포인터를 다룰 것입니다.

- `상자` 힙에 값을 할당하기 위해
- `RC`, 다중 소유권을 가능하게 하는 참조 카운팅 유형
- `참조` 및 `RefMut`, `RefCell을 통해 액세스`, 컴파일 시간 대신 런타임에 차용 규칙을 적용하는 유형

또한 불변 유형이 내부 값을 변경하기 위해 API를 노출하는 *내부 변경 가능성 패턴을 다룰 것입니다.* 우리는 또한 *참조 순환에* 대해 논의할 것입니다 : 메모리 누수를 방지하는 방법.

다이빙하자!

------

## [`상자`를 사용하여 힙의 데이터 가리키기](https://doc.rust-lang.org/book/ch15-01-box.html#using-boxt-to-point-to-data-on-the-heap)

가장 직관적인 스마트 포인터는 `Box` 형식의 *상자 입니다.*`. 상자를 사용하면 스택이 아닌 힙에 데이터를 저장할 수 있습니다. 스택에 남는 것은 힙 데이터에 대한 포인터입니다. 스택과 힙의 차이점을 검토하려면 4장을 참조하십시오.

박스는 데이터를 스택이 아닌 힙에 저장하는 것 외에는 성능 오버헤드가 없습니다. 그러나 그들은 많은 추가 기능도 가지고 있지 않습니다. 다음과 같은 상황에서 가장 자주 사용하게 됩니다.

- 컴파일 시간에 크기를 알 수 없는 유형이 있고 정확한 크기가 필요한 컨텍스트에서 해당 유형의 값을 사용하려는 경우
- 많은 양의 데이터가 있고 소유권을 이전하고 싶지만 그렇게 할 때 데이터가 복사되지 않도록 할 때
- 값을 소유하고 싶고 특정 유형이 아닌 특정 특성을 구현하는 유형이라는 점에만 관심이 있는 경우

[`박스로 재귀 유형 활성화`](https://doc.rust-lang.org/book/ch15-01-box.html#enabling-recursive-types-with-boxes) 섹션 에서 첫 번째 상황을 보여드리겠습니다. 두 번째 경우에는 데이터가 스택에 복사되기 때문에 많은 양의 데이터 소유권을 이전하는 데 시간이 오래 걸릴 수 있습니다. 이 상황에서 성능을 향상시키기 위해 많은 양의 데이터를 상자의 힙에 저장할 수 있습니다. 그런 다음 적은 양의 포인터 데이터만 스택에 복사되고 참조하는 데이터는 힙의 한 위치에 유지됩니다. *세 번째 경우는 특성 객체* 로 알려져 있으며 17장에서는 [`다른 유형의 값을 허용하는 특성 객체 사용`이라는](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) 전체 섹션을 해당 주제에 할애합니다. 따라서 여기서 배운 내용은 17장에서 다시 적용하게 됩니다!

### [`상자`를 사용하여 힙에 데이터 저장](https://doc.rust-lang.org/book/ch15-01-box.html#using-a-boxt-to-store-data-on-the-heap)

`Box`의 힙 저장소 사용 사례를 논의하기 전에`에서 구문과 `Box` 내에 저장된 값과 상호 작용하는 방법을 다룰 것입니다.`.

목록 15-1은 상자를 사용하여 힙에 `i32` 값을 저장하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let b = Box::new(5);
    println!(`b = {}`, b);
}
```

목록 15-1: 상자를 사용하여 힙에 `i32` 값 저장

힙에 할당된 값 `5`를 가리키는 `Box`의 값을 갖도록 변수 `b`를 정의합니다. 이 프로그램은 `b = 5`를 인쇄합니다. 이 경우 이 데이터가 스택에 있는 경우와 유사하게 상자의 데이터에 액세스할 수 있습니다. 모든 소유된 값과 마찬가지로 상자가 범위를 벗어나면 `b`가 `main`의 끝에서 하는 것처럼 할당이 해제됩니다. 할당 해제는 상자(스택에 저장됨)와 그것이 가리키는 데이터(힙에 저장됨) 모두에 대해 발생합니다.

힙에 단일 값을 넣는 것은 그다지 유용하지 않으므로 이러한 방식으로 박스 자체를 자주 사용하지는 않을 것입니다. 기본적으로 저장되는 스택에 단일 `i32`와 같은 값을 갖는 것이 대부분의 상황에서 더 적절합니다. 상자가 없으면 허용되지 않는 유형을 상자를 통해 정의할 수 있는 경우를 살펴보겠습니다.

### [상자로 재귀 유형 활성화](https://doc.rust-lang.org/book/ch15-01-box.html#enabling-recursive-types-with-boxes)

*재귀 유형* 의 값은 자신의 일부와 동일한 유형의 다른 값을 가질 수 있습니다. 재귀 유형은 컴파일 시간에 러스트가 유형이 차지하는 공간을 알아야 하기 때문에 문제를 제기합니다. 그러나 재귀 유형 값의 중첩은 이론적으로 무한히 계속될 수 있으므로 러스트는 값이 얼마나 많은 공간을 필요로 하는지 알 수 없습니다. 상자의 크기가 알려져 있기 때문에 재귀 유형 정의에 상자를 삽입하여 재귀 유형을 활성화할 수 있습니다.

재귀 유형의 예로 *cons list 를* 살펴보겠습니다. 함수형 프로그래밍 언어에서 흔히 볼 수 있는 데이터 유형입니다. 우리가 정의할 cons 목록 유형은 재귀를 제외하고는 간단합니다. 따라서 우리가 작업할 예제의 개념은 재귀 유형과 관련된 더 복잡한 상황에 처할 때마다 유용할 것입니다.

#### [단점 목록에 대한 추가 정보](https://doc.rust-lang.org/book/ch15-01-box.html#more-information-about-the-cons-list)

cons *list* 는 Lisp 프로그래밍 언어와 그 방언에서 나온 데이터 구조로 중첩된 쌍으로 구성되어 있으며 Lisp 버전의 연결 목록입니다. 그 이름은 두 개의 인수로부터 새로운 쌍을 구성하는 Lisp의 `cons` 함수(`construct function`의 줄임말)에서 유래했습니다. 값과 다른 쌍으로 구성된 쌍에서 `cons`를 호출하여 재귀 쌍으로 구성된 cons 목록을 구성할 수 있습니다.

예를 들어, 다음은 목록 1, 2, 3을 포함하는 cons 목록의 의사 코드 표현이며 각 쌍은 괄호 안에 있습니다.

```text
(1, (2, (3, Nil)))
```

cons 목록의 각 항목에는 현재 항목의 값과 다음 항목의 두 가지 요소가 포함됩니다. 목록의 마지막 항목에는 다음 항목 없이 `Nil`이라는 값만 포함되어 있습니다. cons 목록은 `cons` 함수를 재귀적으로 호출하여 생성됩니다. 재귀의 기본 사례를 나타내는 표준 이름은 `Nil`입니다. 이것은 유효하지 않거나 없는 값인 6장의 `null` 또는 `nil` 개념과 동일하지 않습니다.

cons 목록은 Rust에서 일반적으로 사용되는 데이터 구조가 아닙니다. Rust에 항목 목록이 있을 때 대부분 `Vec`가 사용하기에 더 나은 선택입니다. 다른 더 복잡한 재귀 데이터 유형은 다양한 상황에서 *유용* 하지만 이 장의 단점 목록부터 시작하여 상자를 사용하여 많은 산만함 없이 재귀 데이터 유형을 정의할 수 있는 방법을 탐색할 수 있습니다.

목록 15-2에는 cons 목록에 대한 열거형 정의가 포함되어 있습니다. 이 코드는 `목록` 유형에 알려진 크기가 없기 때문에 아직 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

Listing 15-2: `i32` 값의 cons list 데이터 구조를 나타내기 위해 열거형을 정의하려는 첫 번째 시도

> 참고: 이 예제의 목적을 위해 `i32` 값만 보유하는 cons 목록을 구현하고 있습니다. 10장에서 논의한 것처럼 모든 유형의 값을 저장할 수 있는 cons 목록 유형을 정의하기 위해 제네릭을 사용하여 구현했을 수 있습니다.

목록 `1, 2, 3`을 저장하기 위해 `목록` 유형을 사용하는 것은 목록 15-3의 코드와 같습니다:

파일 이름: src/main.rs

```rust
use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

Listing 15-3: `List` 열거형을 사용하여 `1, 2, 3` 목록 저장

첫 번째 `Cons` 값에는 `1`과 다른 `List` 값이 있습니다. 이 `목록` 값은 `2` 및 다른 `목록` 값을 포함하는 또 다른 `Cons` 값입니다. 이 `목록` 값은 `3`과 `목록` 값을 보유하는 또 하나의 `Cons` 값이며, 최종적으로 목록의 끝을 알리는 비재귀적 변형인 `Nil`입니다.

목록 15-3의 코드를 컴파일하려고 하면 목록 15-4에 표시된 오류가 발생합니다.

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^
2 |     Cons(i32, List),
  |               ---- recursive without indirection
  |
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

For more information about this error, try `rustc --explain E0072`.
error: could not compile `cons-list` due to previous error
```

목록 15-4: 재귀 열거형을 정의하려고 시도할 때 발생하는 오류

오류는 이 유형이 `크기가 무한함`을 나타냅니다. 그 이유는 우리가 재귀적 변형으로 `목록`을 정의했기 때문입니다. 즉, 자체의 또 다른 값을 직접 보유합니다. 결과적으로 Rust는 `목록` 값을 저장하는 데 필요한 공간을 파악할 수 없습니다. 이 오류가 발생하는 이유를 분석해 보겠습니다. 먼저 Rust가 비재귀 유형의 값을 저장하는 데 필요한 공간을 결정하는 방법을 살펴보겠습니다.

#### [비재귀 유형의 크기 계산](https://doc.rust-lang.org/book/ch15-01-box.html#computing-the-size-of-a-non-recursive-type)

6장에서 열거형 정의를 논의할 때 목록 6-2에서 정의한 `Message` 열거형을 상기하십시오.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

`Message` 값에 할당할 공간을 결정하기 위해 Rust는 각 변형을 검토하여 가장 많은 공간이 필요한 변형을 확인합니다. Rust는 `Message::Quit`에 공간이 필요하지 않고 `Message::Move`에 두 개의 `i32` 값을 저장하기에 충분한 공간이 필요하다는 것을 확인합니다. 하나의 변형만 사용되기 때문에 `메시지` 값에 필요한 최대 공간은 가장 큰 변형을 저장하는 데 필요한 공간입니다.

Rust가 목록 15-2의 `List` 열거형과 같은 재귀 유형에 필요한 공간을 결정하려고 할 때 발생하는 일과 대조하십시오. 컴파일러는 `i32` 유형의 값과 `List` 유형의 값을 보유하는 `Cons` 변형을 살펴보는 것으로 시작합니다. 따라서 `Cons`에는 `i32` 크기에 `List` 크기를 더한 것과 같은 공간이 필요합니다. `List` 유형에 필요한 메모리 양을 파악하기 위해 컴파일러는 `Cons` 변형부터 시작하여 변형을 살펴봅니다. `Cons` 변형은 `i32` 유형의 값과 `List` 유형의 값을 보유하며 이 프로세스는 그림 15-1과 같이 무한히 계속됩니다.

![무한한 단점 목록](https://doc.rust-lang.org/book/img/trpl15-01.svg)

그림 15-1: 무한 `Cons` 변형으로 구성된 무한 `목록`

#### [`Box`를 사용하여 알려진 크기의 재귀 유형 가져오기](https://doc.rust-lang.org/book/ch15-01-box.html#using-boxt-to-get-a-recursive-type-with-a-known-size)

Rust는 재귀적으로 정의된 유형에 할당할 공간을 파악할 수 없기 때문에 컴파일러는 다음과 같은 유용한 제안과 함께 오류를 표시합니다.

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

이 제안에서 `간접`이란 값을 직접 저장하는 대신 값에 대한 포인터를 저장하여 간접적으로 값을 저장하도록 데이터 구조를 변경해야 함을 의미합니다.

왜냐하면 `박스`는 포인터이고 Rust는 항상 `Box`가 얼마나 많은 공간을 차지하는지 알고 있습니다.` 필요: 포인터의 크기는 포인터가 가리키는 데이터의 양에 따라 변경되지 않습니다. 즉, `Box`를 다른 `List` 값 대신 `Cons` 변형 내부에 직접 입력합니다. `Box`는 `Cons` 변형 내부가 아닌 힙에 있을 다음 `목록` 값을 가리킵니다. 개념적으로는 여전히 다른 목록을 포함하는 목록으로 생성된 목록이 있지만 이 구현은 이제 항목을 배치하는 것과 비슷합니다. 서로의 내부가 아닌 서로의 옆에.

Listing 15-2의 `List` enum 정의와 Listing 15-3의 `List` 사용법을 Listing 15-5의 코드로 변경할 수 있습니다.

파일 이름: src/main.rs

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

Listing 15-5: `Box`를 사용하는 `List`의 정의` 알려진 크기를 갖기 위해

`Cons` 변형에는 `i32`의 크기와 상자의 포인터 데이터를 저장할 공간이 필요합니다. `Nil` 변형은 값을 저장하지 않으므로 `Cons` 변형보다 적은 공간이 필요합니다. 우리는 이제 모든 `목록` 값이 `i32`의 크기에 상자의 포인터 데이터 크기를 더한 값을 차지한다는 것을 알고 있습니다. 상자를 사용하여 무한한 재귀 체인을 끊었으므로 컴파일러는 `목록` 값을 저장하는 데 필요한 크기를 파악할 수 있습니다. 그림 15-2는 `Cons` 변종의 현재 모습을 보여줍니다.

![한정된 단점 목록](https://doc.rust-lang.org/book/img/trpl15-02.svg)

그림 15-2: `Cons`가 `Box`를 보유하기 때문에 크기가 무한하지 않은 `List`

박스는 간접 및 힙 할당만 제공합니다. 다른 스마트 포인터 유형에서 볼 수 있는 것과 같은 다른 특수 기능이 없습니다. 또한 이러한 특수 기능이 발생시키는 성능 오버헤드가 없으므로 간접 지정이 필요한 유일한 기능인 cons 목록과 같은 경우에 유용할 수 있습니다. 17장에서도 상자에 대한 더 많은 사용 사례를 살펴보겠습니다.

상자` 유형은 `Deref` 특성을 구현하기 때문에 스마트 포인터입니다.` 값은 참조처럼 취급됩니다. `상자` 값이 범위를 벗어나면 `Drop` 특성 구현으로 인해 상자가 가리키는 힙 데이터도 정리됩니다. 이 두 특성은 우리가 제공하는 다른 스마트 포인터 유형에서 제공하는 기능에 훨씬 더 중요합니다. 이 장의 나머지 부분에서 논의할 것이므로 이 두 가지 특성에 대해 자세히 살펴보겠습니다.

------

## [`Deref` 특성을 사용하여 스마트 포인터를 일반 참조처럼 취급](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait)

*`Deref` 특성을 구현하면 역참조 연산자* `*` 의 동작을 사용자 정의할 수 있습니다 (곱하기 또는 glob 연산자와 혼동하지 말 것). 스마트 포인터가 일반 참조처럼 취급될 수 있는 방식으로 `Deref`를 구현하면 참조에서 작동하는 코드를 작성하고 해당 코드를 스마트 포인터와 함께 사용할 수도 있습니다.

먼저 역참조 연산자가 일반 참조와 함께 작동하는 방식을 살펴보겠습니다. 그런 다음 `Box`처럼 동작하는 사용자 지정 유형을 정의하려고 합니다.`, 역참조 연산자가 새로 정의된 유형에 대한 참조처럼 작동하지 않는 이유를 확인합니다. `Deref` 특성을 구현하여 스마트 포인터가 참조와 유사한 방식으로 작동할 수 있는 방법을 살펴보겠습니다. 그런 다음 Rust의 *역참조 강제* 기능과 그것이 참조나 스마트 포인터로 작업하는 방법을 살펴보십시오.

> 참고: `MyBox` 사이에는 한 가지 큰 차이점이 있습니다.` 우리가 만들려고 하는 유형과 실제 `상자`: 우리 버전은 데이터를 힙에 저장하지 않습니다. 이 예제는 `Deref`에 초점을 맞추고 있으므로 데이터가 실제로 저장되는 위치는 포인터와 같은 동작보다 덜 중요합니다.

### [값에 대한 포인터를 따라](https://doc.rust-lang.org/book/ch15-02-deref.html#following-the-pointer-to-the-value)

일반 참조는 일종의 포인터이며 포인터를 다른 곳에 저장된 값에 대한 화살표로 생각하는 한 가지 방법입니다. 목록 15-6에서 `i32` 값에 대한 참조를 생성한 다음 값에 대한 참조를 따르기 위해 역참조 연산자를 사용합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

목록 15-6: 역참조 연산자를 사용하여 `i32` 값에 대한 참조를 따르기

변수 `x`는 `i32` 값 `5`를 보유합니다. `y`를 `x`에 대한 참조와 동일하게 설정합니다. 우리는 `x`가 `5`와 같다고 주장할 수 있습니다. 그러나 `y`의 값에 대한 주장을 하려면 컴파일러가 실제 값을 비교할 수 있도록 `*y`를 사용하여 가리키는 값에 대한 참조를 따라야 합니다(따라서 *역참조 ).* `y`를 역참조하면 `5`와 비교할 수 있는 `y`가 가리키는 정수 값에 액세스할 수 있습니다.

`assert_eq!(5, y);`라고 쓰려고 하면 대신 다음과 같은 컴파일 오류가 발생합니다.

```bash
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0277]: can`t compare `{integer}` with `&{integer}`
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^ no implementation for `{integer} == &{integer}`
  |
  = help: the trait `PartialEq<&{integer}>` is not implemented for `{integer}`
  = help: the following other types implement trait `PartialEq<Rhs>`:
            f32
            f64
            i128
            i16
            i32
            i64
            i8
            isize
          and 6 others
  = note: this error originates in the macro `assert_eq` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
error: could not compile `deref-example` due to previous error
```

숫자와 숫자에 대한 참조는 유형이 다르기 때문에 비교할 수 없습니다. 가리키는 값에 대한 참조를 따르려면 역참조 연산자를 사용해야 합니다.

### [참조처럼 `상자` 사용](https://doc.rust-lang.org/book/ch15-02-deref.html#using-boxt-like-a-reference)

Listing 15-6의 코드를 `Box`를 사용하도록 다시 작성할 수 있습니다.` 참조 대신 `박스에 사용되는 역참조 연산자` 목록 15-7의 기능은 목록 15-6의 참조에 사용된 역참조 연산자와 같은 방식입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Listing 15-7: `Box`에 역참조 연산자 사용하기`

Listing 15-7과 Listing 15-6의 주요 차이점은 여기에서 `y`를 `Box`의 인스턴스로 설정했다는 것입니다.`는 `x`의 값을 가리키는 참조가 아니라 `x`의 복사된 값을 가리킵니다. 마지막 어설션에서 역참조 연산자를 사용하여 `Box`의 포인터를 따라갈 수 있습니다.`y`가 참조일 때 했던 것과 같은 방식입니다. 다음으로 `Box`의 특별한 점을 살펴보겠습니다.` 자체 형식을 정의하여 역참조 연산자를 사용할 수 있습니다.

### [우리만의 스마트 포인터 정의하기](https://doc.rust-lang.org/book/ch15-02-deref.html#defining-our-own-smart-pointer)

`Box`와 유사한 스마트 포인터를 만들어 봅시다.` 스마트 포인터가 기본적으로 참조와 어떻게 다르게 동작하는지 경험하기 위해 표준 라이브러리에서 제공하는 유형. 그런 다음 역참조 연산자를 사용하는 기능을 추가하는 방법을 살펴보겠습니다.

상자` 유형은 궁극적으로 하나의 요소가 있는 튜플 구조체로 정의되므로 목록 15-8은 `MyBox` 같은 방식으로 입력합니다. 또한 `Box`에 정의된 `new` 함수와 일치하도록 `new` 함수를 정의합니다.`.

파일 이름: src/main.rs

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

목록 15-8: `MyBox 정의하기` 유형

`MyBox`라는 구조체를 정의하고 일반 매개변수 `T`를 선언합니다. 유형이 모든 유형의 값을 보유하기를 원하기 때문입니다. `MyBox` 유형은 `T` 유형의 요소가 하나 있는 튜플 구조체입니다. `MyBox::new` 함수는 `T` 유형의 매개변수 하나를 사용하고 전달된 값을 보유하는 `MyBox` 인스턴스를 반환합니다.

Listing 15-7의 `main` 함수를 Listing 15-8에 추가하고 `MyBox`를 사용하도록 변경해 봅시다.` 유형을 `Box` 대신에 정의했습니다.`. Listing 15-9의 코드는 Rust가 `MyBox`를 역참조하는 방법을 모르기 때문에 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

Listing 15-9: `MyBox` 사용 시도` 같은 방식으로 참조를 사용하고 `Box`

결과 컴파일 오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^

For more information about this error, try `rustc --explain E0614`.
error: could not compile `deref-example` due to previous error
```

우리의 `마이박스` 유형에 해당 기능을 구현하지 않았기 때문에 유형을 역참조할 수 없습니다. `*` 연산자로 역참조를 활성화하기 위해 `Deref` 특성을 구현합니다.

### [`Deref` 특성을 구현하여 참조처럼 유형 처리](https://doc.rust-lang.org/book/ch15-02-deref.html#treating-a-type-like-a-reference-by-implementing-the-deref-trait)

[10장의 `유형에 대한 특성 구현`](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type) 섹션 에서 설명한 것처럼 특성을 구현하려면 특성의 필수 메서드에 대한 구현을 제공해야 합니다. 표준 라이브러리에서 제공하는 `Deref` 특성은 `self`를 빌리고 내부 데이터에 대한 참조를 반환하는 `deref`라는 메서드를 구현해야 합니다. 목록 15-10에는 `MyBox` 정의에 추가할 `Deref` 구현이 포함되어 있습니다.

파일 이름: src/main.rs

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

Listing 15-10: `MyBox`에 `Deref` 구현하기`

`유형 대상 = T;` 구문은 사용할 `Deref` 특성에 대한 관련 유형을 정의합니다. 연관 유형은 일반 매개변수를 선언하는 약간 다른 방법이지만 지금은 이에 대해 걱정할 필요가 없습니다. 19장에서 더 자세히 다룰 것입니다.

`deref` 메서드의 본문을 `&self.0`으로 채워서 `deref`가 ` ` 연산자로 액세스하려는 값에 대한 참조를 반환합니다 *. `.0`이 튜플 구조체의 첫 번째 값에 액세스하는 5장의 [`명명된 필드 없이 튜플 구조체를 사용하여 다른 유형 만들기`](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types) 섹션을 기억하십시오 .* `MyBox에서 *` `를 호출하는 Listing 15-9의 `main` 함수*` 이제 값이 컴파일되고 어설션이 통과됩니다!

`Deref` 특성이 없으면 컴파일러는 `&` 참조만 역참조할 수 있습니다. `deref` 메서드는 컴파일러가 `Deref`를 구현하는 모든 유형의 값을 가져오고 `deref` 메서드를 호출하여 역참조하는 방법을 알고 있는 `&` 참조를 가져올 수 있는 기능을 제공합니다.

Listing 15-9에서 `*y`를 입력했을 때 Rust는 실제로 이 코드를 실행했습니다.

```rust
*(y.deref())
```

Rust는 `*` 연산자를 `deref` 메서드에 대한 호출로 대체한 다음 일반 역참조로 대체하므로 `deref` 메서드를 호출해야 하는지 여부를 생각할 필요가 없습니다. 이 Rust 기능을 사용하면 일반 참조가 있든 `Deref`를 구현하는 유형이 있든 동일하게 작동하는 코드를 작성할 수 있습니다.

`deref` 메서드가 값에 대한 참조를 반환하고 `*(y.deref())`에서 괄호 외부의 일반 역참조가 여전히 필요한 이유는 소유권 시스템과 관련이 있습니다. `deref` 메서드가 값에 대한 참조 대신 값을 직접 반환하면 값이 `self` 밖으로 이동됩니다. 우리는 `MyBox` 내부의 내부 가치를 소유하고 싶지 않습니다.` 이 경우 또는 대부분의 경우 역참조 연산자를 사용합니다.

` ` 연산자는 *코드에서* ` `를 사용할 때마다 *` deref` 메서드를 호출한 다음 ` ` 연산자를 한 번만 호출하는 것으로 대체됩니다.* *` ` 연산자의 대체는* 무한 반복되지 않기 때문에 `assert_eq!`의 `5`와 일치하는 `i32` 유형의 데이터로 끝납니다. 목록 15-9에서.

### [함수 및 메서드를 사용한 암시적 역참조 강제 변환](https://doc.rust-lang.org/book/ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods)

*Deref 강제는* `Deref` 특성을 구현하는 유형에 대한 참조를 다른 유형에 대한 참조로 변환합니다. 예를 들어 역참조 강제는 `&String`을 `&str`로 변환할 수 있습니다. `String`은 `&str`을 반환하도록 `Deref` 특성을 구현하기 때문입니다. Deref 강제는 Rust가 함수와 메서드에 대한 인수에 대해 수행하는 편리한 기능이며 `Deref` 특성을 구현하는 유형에서만 작동합니다. 함수 또는 메서드 정의의 매개 변수 유형과 일치하지 않는 함수 또는 메서드에 대한 인수로 특정 유형의 값에 대한 참조를 전달할 때 자동으로 발생합니다. `deref` 메서드에 대한 일련의 호출은 우리가 제공한 유형을 매개변수에 필요한 유형으로 변환합니다.

함수 및 메서드 호출을 작성하는 프로그래머가 `&` 및 `*`를 사용하여 많은 명시적 참조 및 역참조를 추가할 필요가 없도록 역참조 강제가 Rust에 추가되었습니다. 역참조 강제 기능을 사용하면 참조 또는 스마트 포인터에 대해 작동할 수 있는 더 많은 코드를 작성할 수 있습니다.

역참조 강제가 작동하는 것을 보려면 `MyBox` 우리가 Listing 15-8에서 정의한 type과 Listing 15-10에서 추가한 `Deref`의 구현. Listing 15-11은 문자열 슬라이스 매개변수가 있는 함수의 정의를 보여줍니다:

파일 이름: src/main.rs

```rust
fn hello(name: &str) {
    println!(`Hello, {name}!`);
}
```

목록 15-11: `&str` 유형의 매개변수 `name`을 갖는 `hello` 함수

`hello(`Rust`);`와 같이 문자열 슬라이스를 인수로 사용하여 `hello` 함수를 호출할 수 있습니다. 예를 들어. Deref 강제는 `MyBox` 유형의 값에 대한 참조로 `hello`를 호출할 수 있게 합니다.`, Listing 15-12와 같이:

파일 이름: src/main.rs

```rust
fn main() {
    let m = MyBox::new(String::from(`Rust`));
    hello(&m);
}
```

Listing 15-12: `MyBox`에 대한 참조로 `hello` 호출하기역참조 강제로 인해 작동하는 값

여기에서 `MyBox`에 대한 참조인 `&m` 인수를 사용하여 `hello` 함수를 호출합니다.` 값. `MyBox`에 `Deref` 특성을 구현했기 때문입니다.` 목록 15-10에서 Rust는 `&MyBox`를 `deref`를 호출하여 `&String`으로. 표준 라이브러리는 문자열 슬라이스를 반환하는 `Deref` on `Deref` 구현을 제공하며 이는 `Deref`에 대한 API 문서에 있습니다. Rust는 `deref`를 다시 호출하여 다음을 수행합니다. `&String`을 `hello` 함수의 정의와 일치하는 `&str`로 바꿉니다.

러스트가 역참조 강제를 구현하지 않았다면 `&MyBox` 유형의 값으로 `hello`를 호출하기 위해 목록 15-12의 코드 대신 목록 15-13의 코드를 작성해야 합니다.`.

파일 이름: src/main.rs

```rust
fn main() {
    let m = MyBox::new(String::from(`Rust`));
    hello(&(*m)[..]);
}
```

목록 15-13: Rust에 역참조 강제가 없다면 작성해야 할 코드

`(*m)`은 `MyBox`를 `문자열`로 변환합니다. 그런 다음 `&` 및 `[..]`는 전체 문자열과 동일한 `문자열`의 문자열 조각을 가져와 `hello`의 서명과 일치시킵니다. 역참조 강제가 없는 이 코드는 다음과 같습니다. 관련된 모든 기호로 인해 읽고, 쓰고, 이해하기가 더 어렵습니다 역참조 강제는 Rust가 자동으로 이러한 변환을 처리할 수 있도록 합니다.

관련된 유형에 대해 `Deref` 특성이 정의되면 Rust는 유형을 분석하고 `Deref::deref`를 매개변수의 유형과 일치하는 참조를 얻기 위해 필요한 만큼 많이 사용할 것입니다. `Deref::deref`가 삽입되어야 하는 횟수는 컴파일 시간에 해결되므로 역참조 강제를 활용하는 데 따른 런타임 페널티가 없습니다!

### [역참조 강제가 가변성과 상호 작용하는 방법](https://doc.rust-lang.org/book/ch15-02-deref.html#how-deref-coercion-interacts-with-mutability)

*`Deref` 특성을 사용하여 불변 참조에서* ` ` 연산자를 재정의하는 방법과 유사하게 `DerefMut` 특성을 사용하여 가변 참조에서 ` ` 연산자를 재정의할 수 있습니다.

Rust는 세 가지 경우에서 유형과 특성 구현을 찾을 때 역참조 강제를 수행합니다.

- `T: Deref<Target=U>`인 경우 `&T`에서 `&U`로
- `T: DerefMut<Target=U>`일 때 `&mut T`에서 `&mut U`로
- `T: Deref<Target=U>`일 때 `&mut T`에서 `&U`로

처음 두 경우는 두 번째가 가변성을 구현한다는 점을 제외하면 서로 동일합니다. 첫 번째 경우는 `&T`가 있고 `T`가 일부 `U` 유형에 `Deref`를 구현하는 경우 투명하게 `&U`를 얻을 수 있음을 나타냅니다. 두 번째 경우는 변경 가능한 참조에 대해 동일한 역참조 강제가 발생함을 나타냅니다.

세 번째 경우는 더 까다롭습니다. Rust는 변경 가능한 참조를 변경 불가능한 참조로 강제하기도 합니다. 그러나 그 반대는 *불가능* 합니다. 불변 참조는 절대 가변 참조로 강제되지 않습니다. 차용 규칙으로 인해 가변 참조가 있는 경우 해당 가변 참조는 해당 데이터에 대한 유일한 참조여야 합니다(그렇지 않으면 프로그램이 컴파일되지 않음). 하나의 변경 가능한 참조를 하나의 변경 불가능한 참조로 변환해도 차용 규칙이 깨지지 않습니다. 불변 참조를 가변 참조로 변환하려면 초기 불변 참조가 해당 데이터에 대한 유일한 불변 참조여야 하지만 차용 규칙은 이를 보장하지 않습니다. 따라서 Rust는 불변 참조자를 가변 참조자로 변환하는 것이 가능하다는 가정을 할 수 없습니다.

------

## [`드롭` 특성을 사용하여 정리 시 코드 실행](https://doc.rust-lang.org/book/ch15-03-drop.html#running-code-on-cleanup-with-the-drop-trait)

스마트 포인터 패턴에 중요한 두 번째 특성은 값이 범위를 벗어나려고 할 때 발생하는 작업을 사용자 정의할 수 있는 `드롭`입니다. 모든 유형에서 `삭제` 특성에 대한 구현을 제공할 수 있으며 해당 코드를 사용하여 파일이나 네트워크 연결과 같은 리소스를 해제할 수 있습니다.

스마트 포인터를 구현할 때 `Drop` 특성의 기능이 거의 항상 사용되기 때문에 스마트 포인터의 맥락에서 `Drop`을 소개합니다. 예를 들어, `상자`를 드롭하면 상자가 가리키는 힙의 공간이 할당 해제됩니다.

일부 언어에서 일부 형식의 경우 프로그래머는 해당 형식의 인스턴스 사용을 마칠 때마다 메모리나 리소스를 해제하는 코드를 호출해야 합니다. 파일 핸들, 소켓 또는 잠금을 예로 들 수 있습니다. 잊어버리면 시스템이 과부하되어 충돌할 수 있습니다. Rust에서는 값이 범위를 벗어날 때마다 특정 코드 비트가 실행되도록 지정할 수 있으며 컴파일러는 이 코드를 자동으로 삽입합니다. 결과적으로 특정 유형의 인스턴스가 완료된 프로그램의 모든 곳에 정리 코드를 배치하는 데 주의할 필요가 없습니다. 여전히 리소스가 누출되지 않습니다!

`Drop` 특성을 구현하여 값이 범위를 벗어날 때 실행할 코드를 지정합니다. `Drop` 특성을 사용하려면 `self`에 대한 가변 참조를 사용하는 `drop`이라는 메서드를 구현해야 합니다. Rust가 언제 `drop`을 호출하는지 알아보기 위해 `println!`으로 `drop`을 구현해 봅시다. 지금은 진술.

목록 15-14는 `Dropping CustomSmartPointer!`를 인쇄하는 유일한 사용자 정의 기능인 `CustomSmartPointer` 구조체를 보여줍니다. 인스턴스가 범위를 벗어날 때 Rust가 `drop` 기능을 실행할 때 표시합니다.

파일 이름: src/main.rs

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!(`Dropping CustomSmartPointer with data `{}`!`, self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from(`my stuff`),
    };
    let d = CustomSmartPointer {
        data: String::from(`other stuff`),
    };
    println!(`CustomSmartPointers created.`);
}
```

목록 15-14: 정리 코드를 넣을 `Drop` 특성을 구현하는 `CustomSmartPointer` 구조체

`드롭` 특성은 서곡에 포함되어 있으므로 범위로 가져올 필요가 없습니다. `CustomSmartPointer`에 `Drop` 특성을 구현하고 `println!`을 호출하는 `drop` 메서드에 대한 구현을 제공합니다. `drop` 함수의 본문은 유형의 인스턴스가 범위를 벗어날 때 실행하려는 논리를 배치하는 위치입니다. Rust가 언제 `드롭`을 호출하는지 시각적으로 보여주기 위해 여기에 일부 텍스트를 인쇄하고 있습니다.

`main`에서 `CustomSmartPointer`의 두 인스턴스를 만든 다음 `CustomSmartPointers created`를 인쇄합니다. `main`의 끝에서 `CustomSmartPointer`의 인스턴스는 범위를 벗어나고 Rust는 `drop` 메서드에 넣은 코드를 호출하여 최종 메시지를 인쇄합니다. 명시적으로 `drop` 메서드를 호출할 필요가 없다는 점에 유의하십시오.

이 프로그램을 실행하면 다음과 같은 결과가 표시됩니다.

```bash
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/debug/drop-example`
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

Rust는 인스턴스가 범위를 벗어나면 자동으로 `drop`을 호출하여 지정한 코드를 호출합니다. 변수는 생성 순서의 역순으로 삭제되므로 `d`가 `c`보다 먼저 삭제됩니다. 이 예제의 목적은 `drop` 방법이 작동하는 방식에 대한 시각적 가이드를 제공하는 것입니다. 일반적으로 인쇄 메시지가 아닌 유형이 실행해야 하는 정리 코드를 지정합니다.

### [`std::mem::drop`을 사용하여 초기에 값 삭제](https://doc.rust-lang.org/book/ch15-03-drop.html#dropping-a-value-early-with-stdmemdrop)

안타깝게도 자동 `삭제` 기능을 비활성화하는 것은 간단하지 않습니다. 일반적으로 `드롭`을 비활성화할 필요는 없습니다. `Drop` 특성의 요점은 자동으로 처리된다는 것입니다. 그러나 경우에 따라 값을 일찍 정리해야 할 수도 있습니다. 한 가지 예는 잠금을 관리하는 스마트 포인터를 사용하는 경우입니다. 동일한 범위의 다른 코드가 잠금을 획득할 수 있도록 잠금을 해제하는 `드롭` 메서드를 강제로 사용할 수 있습니다. Rust는 `Drop` 특성의 `drop` 메소드를 수동으로 호출하도록 허용하지 않습니다. 대신 범위가 끝나기 전에 값을 강제로 삭제하려면 표준 라이브러리에서 제공하는 `std::mem::drop` 함수를 호출해야 합니다.

Listing 15-15에 표시된 것처럼 Listing 15-14에서 `main` 함수를 수정하여 `Drop` 특성의 `drop` 메서드를 수동으로 호출하려고 하면 컴파일러 오류가 발생합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from(`some data`),
    };
    println!(`CustomSmartPointer created.`);
    c.drop();
    println!(`CustomSmartPointer dropped before the end of main.`);
}
```

목록 15-15: `Drop` 트레이트에서 수동으로 `drop` 메서드를 호출하여 조기에 정리하기 시도

이 코드를 컴파일하려고 하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
error[E0040]: explicit use of destructor method
  --> src/main.rs:16:7
   |
16 |     c.drop();
   |     --^^^^--
   |     | |
   |     | explicit destructor calls not allowed
   |     help: consider using `drop` function: `drop(c)`

For more information about this error, try `rustc --explain E0040`.
error: could not compile `drop-example` due to previous error
```

이 오류 메시지는 `drop`을 명시적으로 호출할 수 없음을 나타냅니다. 오류 메시지는 인스턴스를 정리하는 함수에 대한 일반적인 프로그래밍 용어인 *소멸자 라는 용어를 사용합니다.* 소멸자 *는* 인스턴스를 생성하는 *생성자* 와 유사합니다. Rust의 `드롭` 함수는 특정 소멸자 중 하나입니다.

Rust는 명시적으로 `drop`을 호출하도록 허용하지 않습니다. 왜냐하면 Rust는 여전히 `main` 끝에 있는 값에 대해 `drop`을 자동으로 호출하기 때문입니다. 이것은 Rust가 같은 값을 두 번 정리하려고 시도하기 때문에 *이중 해제* 오류가 발생합니다.

값이 범위를 벗어날 때 `drop`의 자동 삽입을 비활성화할 수 없으며 `drop` 메서드를 명시적으로 호출할 수 없습니다. 따라서 초기에 값을 강제로 정리해야 하는 경우 `std::mem::drop` 함수를 사용합니다.

`std::mem::drop` 함수는 `Drop` 특성의 `drop` 메서드와 다릅니다. 강제로 삭제하려는 값을 인수로 전달하여 호출합니다. 이 함수는 서문에 있으므로 목록 15-16에 표시된 것처럼 `drop` 함수를 호출하도록 목록 15-15의 `main`을 수정할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from(`some data`),
    };
    println!(`CustomSmartPointer created.`);
    drop(c);
    println!(`CustomSmartPointer dropped before the end of main.`);
}
```

목록 15-16: `std::mem::drop`을 호출하여 값이 범위를 벗어나기 전에 명시적으로 삭제

이 코드를 실행하면 다음이 인쇄됩니다.

```bash
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/drop-example`
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```

`일부 데이터` 데이터가 있는 CustomSmartPointer 삭제!` 텍스트 `CustomSmartPointer가 생성되었습니다.` 사이에 인쇄됩니다. 및 `CustomSmartPointer가 메인이 끝나기 전에 떨어졌습니다.` `drop` 메서드 코드가 해당 지점에서 `c`를 드롭하도록 호출되었음을 보여주는 텍스트입니다.

정리를 편리하고 안전하게 만들기 위해 여러 가지 방법으로 `Drop` 특성 구현에 지정된 코드를 사용할 수 있습니다. 예를 들어, 이를 사용하여 고유한 메모리 할당자를 만들 수 있습니다! `드롭` 특성과 Rust의 소유권 시스템을 사용하면 Rust가 자동으로 정리하기 때문에 기억할 필요가 없습니다.

또한 여전히 사용 중인 값을 실수로 정리하여 발생하는 문제에 대해 걱정할 필요가 없습니다. 참조가 항상 유효한지 확인하는 소유권 시스템은 값이 더 이상 사용되지 않을 때 `drop`이 한 번만 호출되도록 합니다.

이제 `상자`를 살펴보았으므로`와 스마트 포인터의 몇 가지 특성, 표준 라이브러리에 정의된 몇 가지 다른 스마트 포인터를 살펴보겠습니다.

------

## [`Rc`, 참조 카운트 스마트 포인터](https://doc.rust-lang.org/book/ch15-04-rc.html#rct-the-reference-counted-smart-pointer)

대부분의 경우 소유권은 명확합니다. 주어진 값을 소유하는 변수를 정확히 알고 있습니다. 그러나 단일 값에 여러 소유자가 있는 경우가 있습니다. 예를 들어 그래프 데이터 구조에서 여러 에지가 동일한 노드를 가리킬 수 있으며 해당 노드는 개념적으로 이를 가리키는 모든 에지가 소유합니다. 노드를 가리키는 에지가 없어서 소유자가 없는 경우가 아니면 노드를 정리하면 안 됩니다.

Rust 유형 `Rc`를 사용하여 명시적으로 다중 소유권을 활성화해야 합니다.*`는 참조 카운팅* 의 약자입니다. `Rc` type은 값이 아직 사용 중인지 여부를 결정하기 위해 값에 대한 참조 수를 추적합니다. 값에 대한 참조가 0인 경우 참조가 무효화되지 않고 값을 정리할 수 있습니다.

`Rc를 상상해보십시오.한 사람이 TV를 보려고 들어오면 틀고, 다른 사람이 방에 들어와서 TV를 볼 수 있다. 다른 사람들이 TV를 보고 있을 때 누군가가 TV를 끄면 나머지 TV 시청자들의 난리가 날 것입니다!

우리는 `RC` 프로그램의 여러 부분을 읽을 수 있도록 힙에 일부 데이터를 할당하고 컴파일 시간에 데이터를 마지막으로 사용할 부분을 결정할 수 없는 경우 입력합니다. 마지막으로 끝나는 부분을 안다면 해당 부분을 데이터 소유자로 만들면 컴파일 시간에 적용되는 일반적인 소유권 규칙이 적용됩니다.

`RC`는 단일 스레드 시나리오에서만 사용됩니다. 16장에서 동시성에 대해 논의할 때 다중 스레드 프로그램에서 참조 카운팅을 수행하는 방법을 다룰 것입니다.

### [`Rc`를 사용하여 데이터 공유](https://doc.rust-lang.org/book/ch15-04-rc.html#using-rct-to-share-data)

목록 15-5의 cons 목록 예제로 돌아가 봅시다. `Box`를 사용하여 정의했음을 기억하십시오.`. 이번에는 세 번째 목록의 소유권을 공유하는 두 개의 목록을 만듭니다. 개념적으로 이것은 그림 15-3과 유사합니다.

![세 번째 목록의 소유권을 공유하는 두 개의 목록](https://doc.rust-lang.org/book/img/trpl15-03.svg)

그림 15-3: 세 번째 목록인 `a`의 소유권을 공유하는 두 개의 목록 `b`와 `c`

5와 10을 포함하는 목록 `a`를 만듭니다. 그런 다음 3으로 시작하는 `b`와 4로 시작하는 `c`라는 두 개의 목록을 더 만듭니다. `b`와 `c` 목록은 모두 그런 다음 5와 10을 포함하는 첫 번째 `a` 목록으로 계속 진행합니다. 즉, 두 목록은 5와 10을 포함하는 첫 번째 목록을 공유합니다.

`목록`과 `상자`의 정의를 사용하여 이 시나리오를 구현하려고 합니다.`는 목록 15-17과 같이 작동하지 않습니다.

파일 이름: src/main.rs

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

목록 15-17: `Box`를 사용하여 두 개의 목록을 가질 수 없음을 보여줍니다.` 세 번째 목록의 소유권을 공유하려는

이 코드를 컴파일하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0382]: use of moved value: `a`
  --> src/main.rs:11:30
   |
9  |     let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
   |         - move occurs because `a` has type `List`, which does not implement the `Copy` trait
10 |     let b = Cons(3, Box::new(a));
   |                              - value moved here
11 |     let c = Cons(4, Box::new(a));
   |                              ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `cons-list` due to previous error
```

`Cons` 변형은 보유한 데이터를 소유하므로 `b` 목록을 만들 때 `a`는 `b`로 이동되고 `b`는 `a`를 소유합니다. 그런 다음 `c`를 만들 때 `a`를 다시 사용하려고 하면 `a`가 이동되었기 때문에 허용되지 않습니다.

대신 참조를 보유하도록 `Cons`의 정의를 변경할 수 있지만, 그러면 수명 매개변수를 지정해야 합니다. 수명 매개변수를 지정하면 목록의 모든 요소가 최소한 전체 목록만큼 오래 지속되도록 지정할 수 있습니다. Listing 15-17에 있는 요소와 목록의 경우이지만 모든 시나리오에 해당되는 것은 아닙니다.

대신 `List`의 정의를 `Rc`를 사용하도록 변경합니다.` 대신 `상자`, 목록 15-18에 표시된 것처럼 각 `Cons` 변형은 이제 값과 `Rc`는 `목록`을 가리킵니다. `a`의 소유권을 가져오는 대신 `b`를 만들 때 `Rc`를 복제합니다.`는 `a`가 보유하고 있으므로 참조 수가 1에서 2로 증가하고 `a`와 `b`가 해당 `Rc의 데이터 소유권을 공유하도록 합니다.`. 또한 `c`를 생성할 때 `a`를 복제하여 참조 수를 2개에서 3개로 늘립니다. `Rc::clone`을 호출할 때마다 `Rc` 내의 데이터에 대한 참조 카운트가`가 증가하고 데이터에 대한 참조가 0이 아닌 한 데이터가 정리되지 않습니다.

파일 이름: src/main.rs

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

Listing 15-18: `Rc`를 사용하는 `List`의 정의`

`Rc`를 가져오려면 `use` 문을 추가해야 합니다.`는 prelude에 없기 때문에 범위에 포함됩니다. `main`에서 5와 10을 포함하는 목록을 만들고 새 `Rc`에 저장합니다.그런 다음 `b`와 `c`를 만들 때 `Rc::clone` 함수를 호출하고 `Rc`에 대한 참조를 전달합니다.`에서 `a`를 인수로 사용합니다.

우리는 `Rc::clone(&a)` 대신 `a.clone()`을 호출할 수 있었지만, Rust의 관례는 이 경우 `Rc::clone`을 사용하는 것입니다. `Rc::clone` 구현은 대부분의 유형의 `clone` 구현처럼 모든 데이터의 전체 복사본을 만들지 않습니다. `Rc::clone`에 대한 호출은 참조 횟수를 증가시킬 뿐이며 시간이 많이 걸리지 않습니다. 데이터의 깊은 복사본은 많은 시간이 걸릴 수 있습니다. 참조 카운팅에 `Rc::clone`을 사용하면 딥 카피 종류의 클론과 참조 카운트를 증가시키는 종류의 클론을 시각적으로 구분할 수 있습니다. 코드에서 성능 문제를 찾을 때 딥 카피 클론만 고려하면 되며 `Rc::clone`에 대한 호출을 무시할 수 있습니다.

### [`Rc`를 복제하면 참조 횟수가 증가합니다.](https://doc.rust-lang.org/book/ch15-04-rc.html#cloning-an-rct-increases-the-reference-count)

목록 15-18의 작업 예제를 변경하여 `Rc`에 대한 참조를 생성하고 삭제할 때 참조 횟수가 변경되는 것을 볼 수 있습니다.` 안에`.

목록 15-19에서 목록 `c` 주위에 내부 범위를 갖도록 `main`을 변경할 것입니다. 그런 다음 `c`가 범위를 벗어날 때 참조 횟수가 어떻게 변경되는지 확인할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!(`count after creating a = {}`, Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!(`count after creating b = {}`, Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!(`count after creating c = {}`, Rc::strong_count(&a));
    }
    println!(`count after c goes out of scope = {}`, Rc::strong_count(&a));
}
```

Listing 15-19: 참조 카운트 출력하기

참조 카운트가 변경되는 프로그램의 각 지점에서 `Rc::strong_count` 함수를 호출하여 얻은 참조 카운트를 인쇄합니다. 이 함수의 이름은 `count`가 아니라 `strong_count`입니다.[` 유형에는 `weak_count`도 있습니다. `참조 순환 방지: `Rc`를 `약한`으로 바꾸기`](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#preventing-reference-cycles-turning-an-rct-into-a-weakt) 섹션 에서 `weak_count`가 사용되는 것을 볼 수 있습니다.

이 코드는 다음을 인쇄합니다.

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.45s
     Running `target/debug/cons-list`
count after creating a = 1
count after creating b = 2
count after creating c = 3
count after c goes out of scope = 2
```

우리는 `Rc`a`의 초기 참조 횟수는 1입니다. 그런 다음 `clone`을 호출할 때마다 횟수가 1씩 증가합니다. `c`가 범위를 벗어나면 횟수가 1씩 감소합니다. 참조 카운트를 늘리기 위해 `Rc::clone`을 호출해야 하는 것처럼 참조 카운트를 줄이는 함수를 호출하려면: `Drop` 특성의 구현은 `Rc` 값이 범위를 벗어납니다.

이 예에서 볼 수 없는 것은 `b`와 `a`가 `main`의 끝에서 범위를 벗어나면 카운트가 0이 되고 `Rc`가 완전히 정리됩니다. `Rc를 사용하여`는 단일 값이 여러 소유자를 가질 수 있도록 허용하며 개수는 소유자가 여전히 존재하는 한 값이 유효한 상태로 유지되도록 합니다.

불변 참조를 통해 `Rc`를 사용하면 읽기 전용으로 프로그램의 여러 부분 간에 데이터를 공유할 수 있습니다. `Rc` 변경 가능한 참조를 여러 개 가질 수 있도록 허용하면 4장에서 논의한 차용 규칙 중 하나를 위반할 수 있습니다. 섹션에서는 내부 가변성 패턴과 `RefCell` `Rc와 함께 사용할 수 있는 유형` 이 불변성 제한에 대해 작업합니다.

------

## [`RefCell` 및 내부 가변성 패턴](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#refcellt-and-the-interior-mutability-pattern)

*내부 가변성은* 해당 데이터에 대한 불변 참조가 있는 경우에도 데이터를 변경할 수 있도록 하는 Rust의 디자인 패턴입니다. 일반적으로 이 작업은 차용 규칙에 의해 허용되지 않습니다. 데이터를 변경하기 위해 패턴은 데이터 구조 내에서 `안전하지 않은` 코드를 사용하여 변경 및 차용을 제어하는 Rust의 일반적인 규칙을 구부립니다. 안전하지 않은 코드는 규칙을 확인하기 위해 컴파일러에 의존하는 대신 수동으로 규칙을 확인하고 있음을 컴파일러에 나타냅니다. 19장에서 안전하지 않은 코드에 대해 더 논의할 것입니다.

컴파일러가 보장할 수 없더라도 런타임 시 차용 규칙을 따를 것임을 보장할 수 있는 경우에만 내부 가변성 패턴을 사용하는 유형을 사용할 수 있습니다. 관련된 `안전하지 않은` 코드는 안전한 API로 래핑되며 외부 유형은 여전히 변경할 수 없습니다.

`RefCell`을 살펴보고 이 개념을 살펴보겠습니다.` 내부 가변성 패턴을 따르는 유형입니다.

### [`RefCell`을 사용하여 런타임에 차용 규칙 적용](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#enforcing-borrowing-rules-at-runtime-with-refcellt)

`Rc와 달리`, `레프셀` 유형은 보유한 데이터에 대한 단일 소유권을 나타냅니다. 따라서 `RefCell`는 `Box와 같은 유형과 다릅니다.`? 4장에서 배운 차용 규칙을 기억해 보십시오.

- 주어진 시간에 하나의 변경 가능한 참조 또는 임의의 수의 변경 불가능한 참조 중 하나(둘 다는 아님)를 가질 수 *있습니다* .
- 참조는 항상 유효해야 합니다.

참조 및 `상자`, 차용 규칙의 불변성은 컴파일 타임에 적용됩니다. `RefCell*`, 이러한 불변성은 런타임 시* 적용됩니다. 참조를 사용하여 이러한 규칙을 위반하면 컴파일러 오류가 발생합니다. `RefCell`, 이 규칙을 어기면 프로그램이 패닉 상태가 되어 종료됩니다.

컴파일 시간에 차용 규칙을 확인하면 개발 프로세스에서 오류가 더 빨리 발견되고 모든 분석이 사전에 완료되기 때문에 런타임 성능에 영향을 미치지 않는다는 장점이 있습니다. 이러한 이유로 컴파일 시간에 차용 규칙을 확인하는 것이 대부분의 경우 최선의 선택이며 이것이 Rust의 기본값인 이유입니다.

대신 런타임에 차용 규칙을 확인하는 것의 이점은 특정 메모리 안전 시나리오가 허용된다는 것입니다. 이 시나리오는 컴파일 타임 검사에서 허용되지 않습니다. Rust 컴파일러와 같은 정적 분석은 본질적으로 보수적입니다. 코드의 일부 속성은 코드를 분석하여 감지하는 것이 불가능합니다. 가장 유명한 예는 정지 문제입니다. 이 문제는 이 책의 범위를 벗어나지만 흥미로운 연구 주제입니다.

일부 분석은 불가능하기 때문에 Rust 컴파일러가 코드가 소유권 규칙을 준수하는지 확신할 수 없으면 올바른 프로그램을 거부할 수 있습니다. 이런 식으로 보수적입니다. Rust가 잘못된 프로그램을 수락하면 사용자는 Rust가 제공하는 보증을 신뢰할 수 없습니다. 그러나 Rust가 올바른 프로그램을 거부하면 프로그래머가 불편을 겪을 뿐 치명적인 일은 발생하지 않습니다. `RefCell` 유형은 코드가 차용 규칙을 따르는 것이 확실하지만 컴파일러가 이를 이해하고 보장할 수 없을 때 유용합니다.

유사 콘텐츠 `RC`, `레프셀`는 단일 스레드 시나리오에서만 사용되며 다중 스레드 컨텍스트에서 사용하려고 하면 컴파일 타임 오류가 발생합니다. `RefCell의 기능을 얻는 방법에 대해 이야기하겠습니다.` 16장의 다중 스레드 프로그램에서.

다음은 `박스`를 선택해야 하는 이유를 요약한 것입니다.`, `RC` 또는 `참조 셀`:

- `RC`는 동일한 데이터의 여러 소유자를 활성화합니다. `상자` 및 `참조 셀`독신 소유자가 있습니다.
- `상자` 컴파일 시간에 확인된 불변 또는 가변 차용을 허용합니다. `Rc` 컴파일 시간에 확인된 불변 차용만 허용합니다. `RefCell` 런타임에 확인된 불변 또는 가변 차용을 허용합니다.
- 왜냐하면 `레프셀` 런타임에 확인된 변경 가능한 차용을 허용합니다. `RefCell 내부의 값을 변경할 수 있습니다.` `RefCell`는 불변입니다.

변경할 수 없는 값 내부의 값을 변경하는 것이 *내부 변경 가능* 패턴입니다. 내부 가변성이 유용한 상황을 살펴보고 이것이 어떻게 가능한지 살펴보겠습니다.

### [내부 가변성: 불변 가치에 대한 가변 차용](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#interior-mutability-a-mutable-borrow-to-an-immutable-value)

차용 규칙의 결과는 불변의 가치를 가지고 있을 때 그것을 가변적으로 차용할 수 없다는 것입니다. 예를 들어 다음 코드는 컴파일되지 않습니다.

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}
```

이 코드를 컴파일하려고 하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling borrowing v0.1.0 (file:///projects/borrowing)
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
 --> src/main.rs:3:13
  |
2 |     let x = 5;
  |         - help: consider changing this to be mutable: `mut x`
3 |     let y = &mut x;
  |             ^^^^^^ cannot borrow as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `borrowing` due to previous error
```

그러나 값이 해당 메서드에서 자체적으로 변경되지만 다른 코드에서는 변경할 수 없는 것처럼 보이는 것이 유용한 상황이 있습니다. 값의 메서드 외부에 있는 코드는 값을 변경할 수 없습니다. `참조 셀 사용`는 내부 가변성을 가질 수 있는 능력을 얻는 한 가지 방법이지만 `RefCell`는 차용 규칙을 완전히 우회하지 않습니다. 컴파일러의 차용 검사기는 이 내부 가변성을 허용하고 차용 규칙은 대신 런타임에 검사됩니다. 규칙을 위반하면 `패닉!` 컴파일러 오류.

`RefCell`을 사용할 수 있는 실용적인 예를 살펴보겠습니다.` 변경 불가능한 값을 변경하고 이것이 유용한 이유를 확인합니다.

#### [내부 가변성에 대한 사용 사례: 모의 객체](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#a-use-case-for-interior-mutability-mock-objects)

때때로 테스트 중에 프로그래머는 특정 동작을 관찰하고 올바르게 구현되었는지 확인하기 위해 다른 유형 대신 유형을 사용합니다. *이 자리 표시자 유형을 테스트 double* 이라고 합니다. 사람이 개입하여 특정 까다로운 장면을 연기하기 위해 배우를 대신하는 영화 제작의 `스턴트 더블`의 의미로 생각하십시오. 테스트 더블은 테스트를 실행할 때 다른 유형을 나타냅니다. *모의 개체는* 테스트 중에 발생하는 일을 기록하는 특정 유형의 테스트 복식이므로 올바른 작업이 수행되었음을 확인할 수 있습니다.

Rust에는 다른 언어에 객체가 있는 것과 같은 의미의 객체가 없으며 Rust에는 일부 다른 언어처럼 표준 라이브러리에 내장된 모의 객체 기능이 없습니다. 그러나 모의 개체와 동일한 목적을 제공하는 구조체를 확실히 만들 수 있습니다.

테스트할 시나리오는 다음과 같습니다. 최대값에 대한 값을 추적하고 현재 값이 최대값에 얼마나 가까운지에 따라 메시지를 보내는 라이브러리를 만듭니다. 이 라이브러리는 예를 들어 허용된 API 호출 수에 대한 사용자 할당량을 추적하는 데 사용할 수 있습니다.

우리 라이브러리는 최대값에 얼마나 근접했는지, 어떤 메시지가 어떤 시간에 있어야 하는지 추적하는 기능만 제공합니다. 우리 라이브러리를 사용하는 응용 프로그램은 메시지를 보내는 메커니즘을 제공할 것으로 예상됩니다. 응용 프로그램은 응용 프로그램에 메시지를 넣거나 전자 메일을 보내거나 문자 메시지를 보내는 등의 작업을 수행할 수 있습니다. 도서관은 그 세부 사항을 알 필요가 없습니다. 필요한 것은 우리가 제공할 `메신저`라는 특성을 구현하는 것입니다. Listing 15-20은 라이브러리 코드를 보여줍니다:

파일 이름: src/lib.rs

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<`a, T: Messenger> {
    messenger: &`a T,
    value: usize,
    max: usize,
}

impl<`a, T> LimitTracker<`a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &`a T, max: usize) -> LimitTracker<`a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send(`Error: You are over your quota!`);
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send(`Urgent warning: You`ve used up over 90% of your quota!`);
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send(`Warning: You`ve used up over 75% of your quota!`);
        }
    }
}
```

목록 15-20: 값이 최대값에 얼마나 가까운지 추적하고 값이 특정 수준에 도달하면 경고하는 라이브러리

이 코드의 한 가지 중요한 부분은 `Messenger` 특성에 `self` 및 메시지 텍스트에 대한 불변 참조를 취하는 `send`라는 메서드가 하나 있다는 것입니다. 이 특성은 모의 객체가 실제 객체와 동일한 방식으로 사용될 수 있도록 구현해야 하는 인터페이스입니다. 다른 중요한 부분은 `LimitTracker`에서 `set_value` 메서드의 동작을 테스트하려는 것입니다. `value` 매개변수에 대해 전달하는 내용을 변경할 수 있지만 `set_value`는 어설션을 만들기 위해 아무 것도 반환하지 않습니다. `Messenger` 특성과 `max`에 대한 특정 값을 구현하는 것으로 `LimitTracker`를 생성하면 `value`에 다른 숫자를 전달할 때

우리는 `send`를 호출할 때 이메일이나 문자 메시지를 보내는 대신 전송하라는 메시지만 추적하는 모의 개체가 필요합니다. 모의 개체의 새 인스턴스를 만들고, 모의 개체를 사용하는 `LimitTracker`를 만들고, `LimitTracker`에서 `set_value` 메서드를 호출한 다음, 모의 개체에 우리가 기대하는 메시지가 있는지 확인할 수 있습니다. Listing 15-21은 이를 수행하기 위해 모의 객체를 구현하려는 시도를 보여주지만 빌리기 검사기는 이를 허용하지 않습니다:

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

Listing 15-21: 빌림 검사기가 허용하지 않는 `MockMessenger` 구현 시도

이 테스트 코드는 전송하라는 메시지를 추적하기 위해 `String` 값의 `Vec`가 있는 `sent_messages` 필드가 있는 `MockMessenger` 구조체를 정의합니다. 또한 빈 메시지 목록으로 시작하는 새로운 `MockMessenger` 값을 생성하는 것이 편리하도록 관련 함수 `new`를 정의합니다. 그런 다음 `MockMessenger`에 대한 `Messenger` 특성을 구현하여 `LimitTracker`에 `MockMessenger`를 제공할 수 있습니다. `send` 메소드의 정의에서 매개변수로 전달된 메시지를 `sent_messages`의 `MockMessenger` 목록에 저장합니다.

테스트에서 `LimitTracker`가 `최대` 값의 75%보다 큰 값으로 `값`을 설정하라는 지시를 받았을 때 어떤 일이 발생하는지 테스트하고 있습니다. 먼저 빈 메시지 목록으로 시작할 새 `MockMessenger`를 만듭니다. 그런 다음 새 `LimitTracker`를 만들고 새 `MockMessenger`에 대한 참조와 100의 `최대` 값을 제공합니다. `LimitTracker`에서 `set_value` 메서드를 80의 값으로 호출합니다. 100의 75%. 그런 다음 `MockMessenger`가 추적하고 있는 메시지 목록에 이제 하나의 메시지가 있어야 한다고 주장합니다.

그러나 이 테스트에는 다음과 같은 한 가지 문제가 있습니다.

```bash
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
error[E0596]: cannot borrow `self.sent_messages` as mutable, as it is behind a `&` reference
  --> src/lib.rs:58:13
   |
2  |     fn send(&self, msg: &str);
   |             ----- help: consider changing that to be a mutable reference: `&mut self`
...
58 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `self` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `limit-tracker` due to previous error
warning: build failed, waiting for other jobs to finish...
```

메시지를 추적하기 위해 `MockMessenger`를 수정할 수 없습니다. `send` 메서드가 `self`에 대한 불변 참조를 사용하기 때문입니다. 또한 `&mut self`를 대신 사용하라는 오류 텍스트의 제안을 받아들일 수 없습니다. 왜냐하면 `send`의 서명이 `Messenger` 특성 정의의 서명과 일치하지 않기 때문입니다. 받는 메시지).

이것은 내부 가변성이 도움이 될 수 있는 상황입니다! `RefCell` 내에 `sent_messages`를 저장합니다.`, 그리고 나서 `send` 메소드는 우리가 본 메시지를 저장하기 위해 `sent_messages`를 수정할 수 있을 것입니다. 목록 15-22는 그 모습을 보여줍니다:

파일 이름: src/lib.rs

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

Listing 15-22: `RefCell 사용하기` 외부 값은 변경할 수 없는 것으로 간주하면서 내부 값을 변경합니다.

`sent_messages` 필드는 이제 `RefCell<Vec 유형입니다.>` 대신 `Vec`. `new` 함수에서 새로운 `RefCell<Vec>` 빈 벡터 주위의 인스턴스.

`send` 메소드 구현을 위해 첫 번째 매개변수는 여전히 특성 정의와 일치하는 `self`의 불변 차용입니다. `RefCell<Vec`에서 `borrow_mut`을 호출합니다.>` `self.sent_messages`에서 `RefCell<Vec 내부의 값에 대한 변경 가능한 참조를 가져옵니다.>`, 벡터입니다. 그런 다음 테스트 중에 전송된 메시지를 추적하기 위해 벡터에 대한 가변 참조에서 `푸시`를 호출할 수 있습니다.

우리가 해야 할 마지막 변경은 어설션에 있습니다. 내부 벡터에 얼마나 많은 항목이 있는지 확인하기 위해 `RefCell<Vec`에서 `borrow`를 호출합니다.>` 벡터에 대한 불변 참조를 가져옵니다.

이제 `RefCell`을 사용하는 방법을 살펴보았습니다.`, 어떻게 작동하는지 파헤쳐 봅시다!

#### [`RefCell`을 사용하여 런타임에 빌림 추적](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#keeping-track-of-borrows-at-runtime-with-refcellt)

불변 및 가변 참조를 생성할 때 각각 `&` 및 `&mut` 구문을 사용합니다. `레프셀`, 우리는 `RefCell에 속한 안전한 API의 일부인 `borrow` 및 `borrow_mut` 메소드를 사용합니다.`. `borrow` 메서드는 스마트 포인터 유형 `Ref` 및 `borrow_mut`은 스마트 포인터 유형 `RefMut`을 반환합니다.`. 두 유형 모두 `Deref`를 구현하므로 일반 참조처럼 취급할 수 있습니다.

`RefCell`는 얼마나 많은 `Ref를 추적합니다.` 및 `RefMut` 스마트 포인터가 현재 활성화되어 있습니다. `빌려`를 호출할 때마다 `RefCell` 활성화된 변경 불가능한 차용 횟수를 증가시킵니다. `Ref` 값이 범위를 벗어나면 변경할 수 없는 차용 횟수가 하나씩 줄어듭니다. 컴파일 타임 차용 규칙과 마찬가지로 `RefCell`언제든지 많은 불변 차용 또는 하나의 가변 차용을 가질 수 있습니다.

이러한 규칙을 위반하려고 하면 참조와 마찬가지로 컴파일러 오류가 발생하지 않고 `RefCell`는 런타임 시 패닉 상태가 됩니다. 목록 15-23은 목록 15-22의 `send` 구현 수정을 보여줍니다. 우리는 `RefCell` 런타임에 이 작업을 수행하지 못하게 합니다.

파일 이름: src/lib.rs

```rust
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            let mut one_borrow = self.sent_messages.borrow_mut();
            let mut two_borrow = self.sent_messages.borrow_mut();

            one_borrow.push(String::from(message));
            two_borrow.push(String::from(message));
        }
    }
```

목록 15-23: `RefCell` 당황 할 것

`RefMut`에 대한 `one_borrow` 변수를 생성합니다.` `borrow_mut`에서 스마트 포인터가 반환되었습니다. 그런 다음 `two_borrow` 변수에서 동일한 방식으로 또 다른 변경 가능한 차용을 생성합니다. 이렇게 하면 동일한 범위에서 두 개의 변경 가능한 참조가 생성되며 이는 허용되지 않습니다. 라이브러리에 대한 테스트를 실행할 때 , 목록 15-23의 코드는 오류 없이 컴파일되지만 테스트는 실패합니다.

```bash
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/limit_tracker-e599811fa246dbde)

running 1 test
test tests::it_sends_an_over_75_percent_warning_message ... FAILED

failures:

---- tests::it_sends_an_over_75_percent_warning_message stdout ----
thread `tests::it_sends_an_over_75_percent_warning_message` panicked at `already borrowed: BorrowMutError`, src/lib.rs:60:53
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_sends_an_over_75_percent_warning_message

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

`이미 빌린 것: BorrowMutError`라는 메시지와 함께 코드가 패닉에 빠진 것을 주목하세요. 이것이 `RefCell` 런타임 시 차용 규칙 위반을 처리합니다.

여기에서 수행한 것처럼 컴파일 시간이 아닌 런타임에 차용 오류를 포착하도록 선택하면 개발 프로세스 후반에 코드에서 잠재적으로 실수를 발견할 수 있음을 의미합니다. 아마도 코드가 프로덕션에 배포될 때까지는 발견되지 않을 것입니다. 또한 코드는 컴파일 시간이 아닌 런타임에 차용을 추적한 결과로 약간의 런타임 성능 저하를 초래합니다. 그러나 `RefCell을 사용하여`를 사용하면 변경 불가능한 값만 허용되는 컨텍스트에서 사용하는 동안 본 메시지를 추적하기 위해 스스로 수정할 수 있는 모의 개체를 작성할 수 있습니다. `RefCell을 사용할 수 있습니다.` 일반 참조가 제공하는 것보다 더 많은 기능을 얻기 위한 절충에도 불구하고.

### [`Rc`와 `RefCell`을 결합하여 가변 데이터의 다중 소유자 보유](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#having-multiple-owners-of-mutable-data-by-combining-rct-and-refcellt)

`RefCell`을 사용하는 일반적인 방법`는 `Rc`. `Rc`를 사용하면 일부 데이터의 소유자가 여러 명일 수 있지만 해당 데이터에 대한 변경 불가능한 액세스만 제공합니다. `Rc가 있는 경우`는 `RefCell*`, 여러 소유자를 가질 수 있고* 변경할 수 있는 값을 얻을 수 있습니다 !

예를 들어, Listing 15-18에서 우리가 `Rc` 여러 목록이 다른 목록의 소유권을 공유하도록 허용합니다. 왜냐하면 `Rc`는 변경할 수 없는 값만 보유하므로 일단 생성한 목록의 값을 변경할 수 없습니다. `RefCell에 추가해 보겠습니다.`를 사용하여 목록의 값을 변경할 수 있습니다. 목록 15-24는 `RefCell` `Cons` 정의에서 모든 목록에 저장된 값을 수정할 수 있습니다.

파일 이름: src/main.rs

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!(`a after = {:?}`, a);
    println!(`b after = {:?}`, b);
    println!(`c after = {:?}`, c);
}
```

Listing 15-24: `Rc<RefCell 사용하기>` 변경 가능한 `목록` 생성

우리는 `Rc<RefCell>` 그리고 나중에 직접 액세스할 수 있도록 `value`라는 변수에 저장합니다. 그런 다음 `value`를 보유하는 `Cons` 변형을 사용하여 `a`에 `List`를 만듭니다. `value`를 복제해야 합니다. 따라서 `a`와 `value`는 `value`에서 `a`로 소유권을 이전하거나 `a`가 `value`에서 차용하는 대신 내부 `5` 값의 소유권을 갖습니다.

목록 `a`를 `Rc`로 래핑합니다.` 따라서 목록 `b`와 `c`를 만들 때 목록 15-18에서 수행한 것과 같이 둘 다 `a`를 참조할 수 있습니다.

`a`, `b` 및 `c`에 목록을 만든 후 `value`의 값에 10을 추가하려고 합니다. `value`에서 `borrow_mut`를 호출하여 이를 수행합니다. 이 기능은 5장에서 논의한 자동 역참조 기능을 사용합니다( [`->` 연산자는 어디에 있습니까?`](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#wheres-the---operator) 절 참조).`를 내부 `RefCell` 값. `borrow_mut` 메서드는 `RefMut` 스마트 포인터에 역참조 연산자를 사용하고 내부 값을 변경합니다.

`a`, `b` 및 `c`를 인쇄하면 모두 5가 아닌 15의 수정된 값을 갖는 것을 볼 수 있습니다.

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.63s
     Running `target/debug/cons-list`
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
```

이 기술은 꽤 깔끔합니다! `RefCell을 사용하여`, 우리는 외부적으로 불변의 `목록` 값을 가지고 있습니다. 그러나 우리는 `RefCell`의 메소드를 사용할 수 있습니다.` 내부 변경 가능성에 대한 액세스를 제공하므로 필요할 때 데이터를 수정할 수 있습니다. 차용 규칙의 런타임 검사는 데이터 경합으로부터 우리를 보호하며 때로는 데이터 구조의 이러한 유연성을 위해 약간의 속도를 거래할 가치가 있습니다. 참고 그 `RefCell` 멀티스레드 코드에서는 작동하지 않습니다! `Mutex`는 `RefCell의 스레드 안전 버전입니다.` 그리고 우리는 `Mutex` 16장에서.

------

## [참조 순환이 메모리를 누수할 수 있음](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#reference-cycles-can-leak-memory)

Rust의 메모리 안전 보장은 결코 정리되지 않는 메모리를 우발적으로 생성하는 것을 어렵게 만들지만 불가능하지는 않습니다(메모리 *누수* 로 알려짐 ). 메모리 누수를 완전히 방지하는 것은 Rust의 보증 중 하나가 아닙니다. 즉, 메모리 누수는 Rust에서 메모리에 안전합니다. Rust가 `Rc`를 사용하여 메모리 누수를 허용한다는 것을 알 수 있습니다.` 및 `참조 셀`: 주기에서 항목이 서로를 참조하는 참조를 생성할 수 있습니다. 주기에서 각 항목의 참조 횟수가 절대 0에 도달하지 않고 값이 삭제되지 않기 때문에 이로 인해 메모리 누수가 발생합니다.

### [참조 순환 만들기](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#creating-a-reference-cycle)

Listing 15-25에 있는 `List` 열거형과 `tail` 메서드의 정의부터 시작하여 참조 순환이 어떻게 발생하고 이를 방지하는 방법을 살펴보겠습니다.

파일 이름: src/main.rs

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
```

목록 15-25: `RefCell`을 보유하는 cons 목록 정의` `Cons` 변형이 참조하는 것을 수정할 수 있습니다.

우리는 목록 15-5에서 `목록` 정의의 또 다른 변형을 사용하고 있습니다. `Cons` 변형의 두 번째 요소는 이제 `RefCell<Rc>`, Listing 15-24에서 했던 것처럼 `i32` 값을 수정하는 기능 대신 `Cons` 변형이 가리키는 `List` 값을 수정하고 싶다는 의미입니다. `Cons` 변형이 있는 경우 두 번째 항목에 쉽게 액세스할 수 있도록 `tail` 메서드를 사용합니다.

Listing 15-26에서는 Listing 15-25의 정의를 사용하는 `main` 함수를 추가하고 있습니다. 이 코드는 `a`에 목록을 만들고 `a`의 목록을 가리키는 `b`에 목록을 만듭니다. 그런 다음 `a`의 목록을 `b`를 가리키도록 수정하여 참조 순환을 만듭니다. `println!`이 있습니다. 이 프로세스의 다양한 지점에서 참조 카운트가 무엇인지 보여주는 방법을 따라 설명합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!(`a initial rc count = {}`, Rc::strong_count(&a));
    println!(`a next item = {:?}`, a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!(`a rc count after b creation = {}`, Rc::strong_count(&a));
    println!(`b initial rc count = {}`, Rc::strong_count(&b));
    println!(`b next item = {:?}`, b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!(`b rc count after changing a = {}`, Rc::strong_count(&b));
    println!(`a rc count after changing a = {}`, Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!(`a next item = {:?}`, a.tail());
}
```

목록 15-26: 서로를 가리키는 두 개의 `목록` 값의 참조 순환 만들기

우리는 `Rc` 초기 목록이 `5, Nil`인 변수 `a`에 `목록` 값을 보유하는 인스턴스. 그런 다음 `Rc`를 생성합니다.값 10을 포함하고 `a`의 목록을 가리키는 변수 `b`에 또 다른 `목록` 값을 보유하는 인스턴스.

`a`를 수정하여 `Nil` 대신 `b`를 가리키도록 하여 순환을 생성합니다. `RefCell<Rc에 대한 참조를 얻기 위해 `tail` 메서드를 사용하여 이를 수행합니다.`a`의 `link` 변수에 넣은 다음 `RefCell<Rc`에서 `borrow_mut` 메서드를 사용합니다.>`는 `Rc에서 내부 값을 변경합니다.`는 `Rc에 `Nil` 값을 보유합니다.`에서 `b`.

이 코드를 실행할 때 마지막 `println!` 잠시 동안 주석 처리하면 다음과 같은 결과가 표시됩니다.

```bash
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/cons-list`
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
```

`Rc의 참조 횟수` `a`의 목록을 `b`를 가리키도록 변경한 후 `a`와 `b`의 인스턴스는 모두 2입니다. `main`의 끝에서 Rust는 참조 횟수를 줄이는 변수 `b`를 삭제합니다. `b` `Rc의` 인스턴스를 2에서 1로. `Rc`는 참조 횟수가 0이 아니라 1이기 때문에 이 시점에서 삭제되지 않습니다. 그런 다음 Rust는 `a`를 삭제하고 `a` `Rc의 참조 횟수를 줄입니다.` 인스턴스도 2에서 1로 변경합니다. 이 인스턴스의 메모리도 삭제할 수 없습니다. 왜냐하면 다른 `Rc` 인스턴스는 여전히 그것을 참조합니다. 목록에 할당된 메모리는 영원히 수집되지 않은 상태로 유지됩니다. 이 참조 순환을 시각화하기 위해 그림 15-4에 다이어그램을 만들었습니다.

![목록의 참조 주기](https://doc.rust-lang.org/book/img/trpl15-04.svg)

그림 15-4: 서로를 가리키는 목록 `a`와 `b`의 참조 순환

마지막 `println!` 그리고 프로그램을 실행하면 Rust는 `a`가 `a`를 가리키는 `a`를 가리키는 `a`로 이 주기를 인쇄하려고 시도할 것입니다. 스택을 넘칠 때까지 계속됩니다.

실제 프로그램과 비교할 때 이 예에서 참조 순환을 생성하는 결과는 그리 심각하지 않습니다. 참조 순환을 생성한 직후 프로그램이 종료됩니다. 그러나 더 복잡한 프로그램이 한 주기에 많은 메모리를 할당하고 오랫동안 유지하면 프로그램은 필요한 것보다 더 많은 메모리를 사용하고 시스템을 압도하여 사용 가능한 메모리가 부족해질 수 있습니다.

참조 순환을 만드는 것은 쉬운 일이 아니지만 불가능한 일도 아닙니다. `RefCell`이 있는 경우` `Rc를 포함하는 값` 값 또는 내부 가변성과 참조 카운팅이 있는 유사한 유형의 중첩된 조합, 순환을 생성하지 않도록 해야 합니다. 이를 포착하기 위해 Rust에 의존할 수 없습니다. 참조 순환을 생성하는 것은 프로그램에서 논리 버그가 될 것입니다. 자동화된 테스트, 코드 검토 및 기타 소프트웨어 개발 방식을 사용하여 최소화해야 합니다.

참조 순환을 피하는 또 다른 솔루션은 데이터 구조를 재구성하여 일부 참조는 소유권을 표현하고 일부 참조는 그렇지 않도록 하는 것입니다. 결과적으로 일부 소유권 관계와 일부 비소유 관계로 구성된 주기를 가질 수 있으며 소유권 관계만 값을 삭제할 수 있는지 여부에 영향을 미칩니다. 목록 15-25에서 우리는 항상 `Cons` 변형이 자신의 목록을 소유하기를 원하므로 데이터 구조를 재구성하는 것은 불가능합니다. 부모 노드와 자식 노드로 구성된 그래프를 사용하여 비소유 관계가 참조 순환을 방지하는 적절한 방법인 경우를 살펴보겠습니다.

### [참조 순환 방지: `Rc`를 `약한`으로 바꾸기](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#preventing-reference-cycles-turning-an-rct-into-a-weakt)

지금까지 `Rc::clone`을 호출하면 `Rc`의 `strong_count`가 증가한다는 것을 보여주었습니다.` 인스턴스 및 `Rc` 인스턴스는 `strong_count`가 0인 경우에만 정리됩니다. `Rc` 내의 값에 대한 *약한 참조를* 만들 수도 있습니다.` 인스턴스를 호출하여 `Rc::downgrade`를 호출하고 `Rc에 대한 참조를 전달합니다.`. 강력한 참조는 `Rc의 소유권을 공유할 수 있는 방법입니다.` 인스턴스. 약한 참조는 소유권 관계를 표현하지 않으며 `Rc` 인스턴스가 정리됩니다. 관련된 값의 강한 참조 수가 0이 되면 일부 약한 참조와 관련된 모든 순환이 중단되기 때문에 참조 순환이 발생하지 않습니다.

`Rc::downgrade`를 호출하면 `Weak` 유형의 스마트 포인터를 얻습니다.`. `Rc에서 `strong_count`를 늘리는 대신` 인스턴스를 1씩 늘리면 `Rc::downgrade`를 호출하면 `weak_count`가 1씩 증가합니다. `Rc` 유형은 `weak_count`를 사용하여 `약한` 수를 추적합니다.`strong_count`와 유사한 참조가 존재합니다. 차이점은 `weak_count`가 `Rc에 대해 0일 필요가 없다는 것입니다.` 정리할 인스턴스입니다.

`약한` 참조가 삭제되어 `약한`가 가리키는 경우 값이 여전히 존재하는지 확인해야 합니다. `약한` 인스턴스는 `Option<Rc>`. `Rc` 값은 아직 삭제되지 않았으며 `Rc` 값이 삭제되었습니다. `upgrade`가 `Option<Rc>`, Rust는 `Some` 사례와 `None` 사례가 처리되도록 하고 유효하지 않은 포인터가 없도록 합니다.

*예를 들어 항목이 다음 항목에 대해서만 알고 있는 목록을 사용하는 대신 자식 항목 과* 부모 항목 에 대해 알고 있는 항목이 있는 트리를 만듭니다.

#### [트리 데이터 구조 만들기: 자식 노드가 있는 `노드`](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#creating-a-tree-data-structure-a-node-with-child-nodes)

시작하려면 자식 노드에 대해 알고 있는 노드로 트리를 만듭니다. 자체 `i32` 값과 하위 `Node` 값에 대한 참조를 보유하는 `Node`라는 구조체를 생성합니다.

파일 이름: src/main.rs

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```

우리는 `노드`가 자식을 소유하기를 원하고 그 소유권을 변수와 공유하여 트리의 각 `노드`에 직접 액세스할 수 있기를 원합니다. 이를 위해 `Vec` 유형 `Rc의 값이 될 항목`. 또한 어떤 노드가 다른 노드의 자식인지 수정하고 싶기 때문에 `RefCell`에서 `Vec<Rc 주위의 `어린이`>`.

다음으로, 구조체 정의를 사용하여 값이 3이고 자식이 없는 `leaf`라는 `Node` 인스턴스 하나와 값이 5이고 자식 중 하나가 `leaf`인 `branch`라는 또 다른 인스턴스를 다음과 같이 만듭니다. 목록 15-27에 나와 있습니다:

파일 이름: src/main.rs

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

목록 15-27: 자식이 없는 `리프` 노드와 자식 중 하나로 `리프`가 있는 `브랜치` 노드 만들기

우리는 `Rc`를 `리프`에 저장하고 `리프`의 `노드`를 의미하는 `브랜치`에 저장하면 이제 `리프`와 `브랜치`라는 두 명의 소유자가 있습니다. `브랜치`에서 `브랜치`를 통해 `리프`로 이동할 수 있습니다. `leaf`에서 `branch`로 이동할 방법이 없습니다. 그 이유는 `leaf`가 `branch`에 대한 참조가 없고 서로 관련되어 있다는 것을 모르기 때문입니다. 우리는 `leaf`가 그것을 알기를 원합니다. `branch`는 그 부모입니다. 다음에 그렇게 하겠습니다.

#### [자식에서 부모로 참조 추가](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#adding-a-reference-from-a-child-to-its-parent)

자식 노드가 부모를 인식하게 하려면 `Node` 구조체 정의에 `parent` 필드를 추가해야 합니다. 문제는 `부모"의 유형을 결정하는 데 있습니다. "Rc"를 포함할 수 없다는 것을 알고 있습니다."는 "branch"를 가리키는 "leaf.parent"와 "leaf"를 가리키는 "branch.children"이 있는 참조 순환을 생성하므로 "strong_count" 값이 0이 되지 않도록 합니다.

관계를 다른 방식으로 생각하면 부모 노드는 자식을 소유해야 합니다. 부모 노드가 삭제되면 자식 노드도 삭제되어야 합니다. 그러나 자식은 부모를 소유해서는 안 됩니다. 자식 노드를 삭제해도 부모는 여전히 존재해야 합니다. 이것은 약한 참조의 경우입니다!

따라서 "Rc 대신", 우리는 "부모"의 유형을 "약함"을 사용하도록 만들 것입니다.", 특히 "RefCell<약함>". 이제 "Node" 구조체 정의는 다음과 같습니다.

파일 이름: src/main.rs

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

노드는 부모 노드를 참조할 수 있지만 부모를 소유하지는 않습니다. 목록 15-28에서 우리는 이 새로운 정의를 사용하기 위해 "main"을 업데이트하여 "leaf" 노드가 부모인 "branch"를 참조할 수 있도록 합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

Listing 15-28: 상위 노드 "branch"에 대한 약한 참조가 있는 "리프" 노드

"리프" 노드를 생성하는 것은 "부모" 필드를 제외하고 목록 15-27과 유사합니다. "리프"는 부모 없이 시작하므로 비어 있는 새 "약한"" 참조 인스턴스.

이 시점에서 "업그레이드" 메서드를 사용하여 "리프"의 부모에 대한 참조를 얻으려고 하면 "없음" 값을 얻습니다. 첫 번째 "println!"의 출력에서 이것을 볼 수 있습니다. 성명:

```text
leaf parent = None
```

"branch" 노드를 생성하면 새로운 "Weak" 노드도 갖게 됩니다." "branch"에는 부모 노드가 없기 때문에 "parent" 필드의 참조. 우리는 여전히 "branch"의 자식 중 하나로 "leaf"를 가지고 있습니다. "branch"에 "Node" 인스턴스가 있으면 "잎"을 수정하여 "약함"을 줄 수 있습니다." 참조. 우리는 "RefCell<Weak>"를 "leaf"의 "parent" 필드에 넣은 다음 "Rc::downgrade" 함수를 사용하여 "Weak" "Rc에서 "분기" 참조"에서 "지점."

"leaf"의 부모를 다시 인쇄하면 이번에는 "branch"를 포함하는 "Some" 변형을 얻게 됩니다. 이제 "leaf"는 부모에 액세스할 수 있습니다! "리프"를 인쇄할 때 목록 15-26에서와 같이 결국 스택 오버플로로 끝나는 주기도 피합니다. 약한" 참조는 "(약함)"으로 인쇄됩니다.

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

무한 출력이 없다는 것은 이 코드가 참조 순환을 생성하지 않았음을 나타냅니다. "Rc::strong_count" 및 "Rc::weak_count"를 호출하여 얻은 값을 보면 이를 알 수 있습니다.

#### ["strong_count" 및 "weak_count" 변경 사항 시각화](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#visualizing-changes-to-strong_count-and-weak_count)

"Rc"의 "strong_count" 및 "weak_count" 값이 어떻게 변하는지 살펴보겠습니다." 인스턴스는 새로운 내부 범위를 생성하고 "branch" 생성을 해당 범위로 이동하여 변경됩니다. 이렇게 하면 "branch"가 생성되고 범위를 벗어날 때 삭제되는 경우 발생하는 상황을 확인할 수 있습니다. 수정 사항이 표시됩니다. 목록 15-29에서:

파일 이름: src/main.rs

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

Listing 15-29: 내부 스코프에 "브랜치" 생성 및 강한 참조 카운트와 약한 참조 카운트 검사

"리프"가 생성된 후 "Rc"는 1의 강한 카운트와 0의 약한 카운트를 가집니다. 내부 범위에서 우리는 "브랜치"를 만들고 "리프"와 연결합니다. 이 지점에서 카운트를 인쇄할 때 "Rc"branch"에서 "branch"를 가리키는 "leaf.parent"의 경우 강한 수는 1이고 약한 수는 1입니다."). "leaf"에 카운트를 인쇄하면 "branch"가 이제 "Rc"branch.children"에 저장된 "leaf"의 "는 여전히 약한 카운트가 0입니다.

내부 범위가 종료되면 "분기"가 범위를 벗어나 "Rc"의 강한 카운트"가 0으로 감소하므로 "Node"가 삭제됩니다. "leaf.parent"의 약한 카운트 1은 "Node"가 삭제되었는지 여부와 관련이 없으므로 메모리 누수가 발생하지 않습니다!

범위가 끝난 후 "leaf"의 부모에 액세스하려고 하면 "None"이 다시 표시됩니다. 프로그램의 끝에서 "Rc"leaf"의 "는 변수 "leaf"가 이제 "Rc"에 대한 유일한 참조이기 때문에 강한 카운트는 1이고 약한 카운트는 0입니다." 다시.

카운트 및 값 삭제를 관리하는 모든 로직은 "Rc"에 내장되어 있습니다."와 "약하다." 및 "Drop" 특성의 구현. 자식에서 부모로의 관계가 "약함"이어야 함을 지정함으로써"노드"의 정의에서 참조를 사용하면 참조 순환 및 메모리 누수를 생성하지 않고 상위 노드가 하위 노드를 가리키도록 할 수 있으며 그 반대도 가능합니다.

## [요약](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html#summary)

이 장에서는 스마트 포인터를 사용하여 러스트가 일반 참조로 기본적으로 만드는 것과 다른 보증 및 장단점을 만드는 방법을 다루었습니다. 상자" 유형은 알려진 크기를 가지며 힙에 할당된 데이터를 가리킵니다. "Rc" 유형은 데이터가 여러 소유자를 가질 수 있도록 힙의 데이터에 대한 참조 수를 추적합니다. "RefCell" 내부 변경 가능성이 있는 유형은 변경할 수 없는 유형이 필요하지만 해당 유형의 내부 값을 변경해야 할 때 사용할 수 있는 유형을 제공합니다. 또한 컴파일 시간이 아닌 런타임에 차용 규칙을 적용합니다.

또한 스마트 포인터의 많은 기능을 가능하게 하는 "Deref" 및 "Drop" 특성에 대해서도 논의했습니다. 메모리 누수를 일으킬 수 있는 참조 순환과 "약함"을 사용하여 이를 방지하는 방법을 살펴보았습니다.".

이 장이 당신의 관심을 불러일으켰고 자신만의 스마트 포인터를 구현하고 싶다면 ["The Rustonomicon"](https://doc.rust-lang.org/nomicon/index.html) 에서 더 유용한 정보를 확인하십시오.

다음으로 Rust의 동시성에 대해 이야기하겠습니다. 몇 가지 새로운 스마트 포인터에 대해서도 배우게 됩니다.

------

# 16

# [두려움 없는 동시성](https://doc.rust-lang.org/book/ch16-00-concurrency.html#fearless-concurrency)

동시 프로그래밍을 안전하고 효율적으로 처리하는 것은 Rust의 주요 목표 중 하나입니다. *프로그램* 의 다른 부분이 독립적으로 실행되는 동시 프로그래밍과 프로그램의 다른 부분이 동시에 실행되는 *병렬 프로그래밍은 더 많은 컴퓨터가 다중 프로세서를 활용함에 따라 점점 더 중요해지고 있습니다.* 역사적으로 이러한 맥락에서 프로그래밍하는 것은 어렵고 오류가 발생하기 쉽습니다. Rust는 이를 변경하기를 희망합니다.

처음에 Rust 팀은 메모리 안전을 보장하는 것과 동시성 문제를 방지하는 것이 서로 다른 방법으로 해결해야 할 두 가지 별도의 과제라고 생각했습니다. 시간이 지남에 따라 팀은 소유권 및 유형 시스템이 메모리 안전 *및*동시성 문제! 소유권 및 유형 검사를 활용함으로써 많은 동시성 오류는 런타임 오류가 아닌 Rust의 컴파일 타임 오류입니다. 따라서 런타임 동시성 버그가 발생하는 정확한 상황을 재현하는 데 많은 시간을 소비하는 대신 잘못된 코드는 컴파일을 거부하고 문제를 설명하는 오류를 표시합니다. 결과적으로 잠재적으로 프로덕션으로 배송된 후가 아니라 작업하는 동안 코드를 수정할 수 있습니다. 우리는 Rust *Fearless* *Concurrency* 의 이러한 측면을 별명으로 지정했습니다. 두려움 없는 동시성을 통해 미묘한 버그가 없는 코드를 작성할 수 있으며 새로운 버그를 도입하지 않고 쉽게 리팩터링할 수 있습니다.

> 참고: 단순화를 위해 많은 문제를 *concurrent 및/또는 parallel* 로 더 정확하게 표현하기보다는 *동시성* 이라고 합니다. 이 책이 동시성 및/또는 병렬성에 관한 것이라면 더 구체적일 것입니다. 이 장에서는 *concurrent 를* 사용할 때마다 마음속으로 *concurrent 및/또는 parallel* 로 대체하세요 .

많은 언어는 동시 발생 문제를 처리하기 위해 제공하는 솔루션에 대해 독단적입니다. 예를 들어 Erlang에는 메시지 전달 동시성을 위한 우아한 기능이 있지만 스레드 간에 상태를 공유하는 모호한 방법만 있습니다. 가능한 솔루션의 하위 집합만 지원하는 것은 고급 언어에 대한 합리적인 전략입니다. 고급 언어는 추상화를 얻기 위해 일부 제어를 포기함으로써 이점을 약속하기 때문입니다. 그러나 낮은 수준의 언어는 주어진 상황에서 최상의 성능을 가진 솔루션을 제공하고 하드웨어에 대한 추상화가 적을 것으로 예상됩니다. 따라서 Rust는 상황과 요구 사항에 적합한 방식으로 문제를 모델링하기 위한 다양한 도구를 제공합니다.

이 장에서 다룰 주제는 다음과 같습니다.

- 동시에 여러 코드 조각을 실행하는 스레드를 만드는 방법
- 채널이 스레드 간에 메시지를 보내는 메시지 *전달 동시성*
- 여러 스레드가 일부 데이터에 액세스할 수 있는 *공유 상태 동시성*
- Rust의 동시성 보장을 사용자 정의 유형 및 표준 라이브러리에서 제공하는 유형으로 확장하는 "동기화" 및 "보내기" 특성

------

## [스레드를 사용하여 코드를 동시에 실행](https://doc.rust-lang.org/book/ch16-01-threads.html#using-threads-to-run-code-simultaneously)

대부분의 현재 운영 체제에서 실행된 프로그램의 코드는 프로세스에서 실행되며 *운영* 체제는 한 번에 여러 프로세스를 관리합니다. 프로그램 내에서 동시에 실행되는 독립적인 부분을 가질 수도 있습니다. 이러한 독립적인 부분을 실행하는 기능을 *스레드* 라고 합니다. 예를 들어, 웹 서버는 동시에 둘 이상의 요청에 응답할 수 있도록 여러 스레드를 가질 수 있습니다.

프로그램의 계산을 여러 스레드로 분할하여 동시에 여러 작업을 실행하면 성능이 향상될 수 있지만 복잡성도 추가됩니다. 스레드가 동시에 실행될 수 있기 때문에 서로 다른 스레드에서 코드 부분이 실행되는 순서에 대한 고유한 보장이 없습니다. 이로 인해 다음과 같은 문제가 발생할 수 있습니다.

- 스레드가 일관성 없는 순서로 데이터 또는 리소스에 액세스하는 경쟁 조건
- 두 스레드가 서로를 기다리고 있어 두 스레드가 계속 진행되지 않는 교착 상태
- 특정 상황에서만 발생하고 안정적으로 재현 및 수정하기 어려운 버그

Rust는 스레드 사용의 부정적인 영향을 완화하려고 시도하지만 다중 스레드 컨텍스트에서 프로그래밍하려면 여전히 신중한 생각이 필요하며 단일 스레드에서 실행되는 프로그램과 다른 코드 구조가 필요합니다.

프로그래밍 언어는 몇 가지 다른 방식으로 스레드를 구현하며 많은 운영 체제는 언어가 새 스레드를 생성하기 위해 호출할 수 있는 API를 제공합니다. Rust 표준 라이브러리는 스레드 구현의 *1:1* 모델을 사용하므로 프로그램은 하나의 언어 스레드당 하나의 운영 체제 스레드를 사용합니다. 1:1 모델과 다른 트레이드오프를 만드는 다른 스레딩 모델을 구현하는 크레이트가 있습니다.

### ["spawn"으로 새 스레드 만들기](https://doc.rust-lang.org/book/ch16-01-threads.html#creating-a-new-thread-with-spawn)

새로운 스레드를 생성하기 위해 "thread::spawn" 함수를 호출하고 새 스레드에서 실행하려는 코드가 포함된 클로저(13장에서 클로저에 대해 이야기함)를 전달합니다. 목록 16-1의 예는 메인 스레드의 일부 텍스트와 새 스레드의 다른 텍스트를 인쇄합니다.

파일 이름: src/main.rs

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

목록 16-1: 메인 스레드가 다른 것을 인쇄하는 동안 한 가지를 인쇄하기 위한 새 스레드 만들기

Rust 프로그램의 메인 스레드가 완료되면 생성된 모든 스레드는 실행 완료 여부에 관계없이 종료됩니다. 이 프로그램의 출력은 매번 조금씩 다를 수 있지만 다음과 유사하게 표시됩니다.

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

"thread::sleep"에 대한 호출은 스레드가 짧은 기간 동안 실행을 중지하도록 강제하여 다른 스레드가 실행될 수 있도록 합니다. 스레드는 순서대로 진행되지만 보장되지는 않습니다. 운영 체제에서 스레드를 예약하는 방법에 따라 다릅니다. 이 실행에서는 생성된 스레드의 인쇄 문이 코드에서 첫 번째로 나타나더라도 기본 스레드가 먼저 인쇄됩니다. 그리고 "i"가 9가 될 때까지 출력하도록 생성된 스레드에 지시했지만 기본 스레드가 종료되기 전에 5에 도달했습니다.

이 코드를 실행하고 기본 스레드의 출력만 표시되거나 겹치는 부분이 표시되지 않는 경우 범위의 숫자를 늘려 운영 체제가 스레드 간에 전환할 수 있는 기회를 더 많이 만드십시오.

### ["조인" 핸들을 사용하여 모든 스레드가 완료될 때까지 대기](https://doc.rust-lang.org/book/ch16-01-threads.html#waiting-for-all-threads-to-finish-using-join-handles)

목록 16-1의 코드는 메인 스레드 종료로 인해 대부분의 경우 생성된 스레드를 조기에 중지할 뿐만 아니라 스레드가 실행되는 순서에 대한 보장이 없기 때문에 생성된 스레드가 실행된다는 보장도 할 수 없습니다. 전혀 실행하십시오!

"thread::spawn"의 반환 값을 변수에 저장하여 생성된 스레드가 실행되지 않거나 조기에 종료되는 문제를 해결할 수 있습니다. "thread::spawn"의 반환 유형은 "JoinHandle"입니다. "JoinHandle"은 "join" 메서드를 호출할 때 스레드가 완료될 때까지 기다리는 소유된 값입니다. Listing 16-2는 Listing 16-1에서 만든 스레드의 "JoinHandle"을 사용하고 "main"이 종료되기 전에 생성된 스레드가 완료되도록 "join"을 호출하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

목록 16-2: "thread::spawn"에서 "JoinHandle"을 저장하여 스레드 실행이 완료되도록 보장

핸들에서 "join"을 호출하면 핸들이 나타내는 스레드가 종료될 때까지 현재 실행 중인 스레드가 차단됩니다. 스레드를 *차단한다는 것은 스레드가 작업을 수행하거나 종료하는 것을 방지함을 의미합니다.* 메인 스레드의 "for" 루프 뒤에 "join"에 대한 호출을 넣었기 때문에 목록 16-2를 실행하면 다음과 유사한 출력이 생성되어야 합니다.

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

두 쓰레드는 계속 번갈아 가며 진행되지만, 메인 쓰레드는 "handle.join()" 호출 때문에 기다리며 생성된 쓰레드가 끝날 때까지 끝나지 않는다.

그러나 다음과 같이 "main"의 "for" 루프 전에 "handle.join()"을 대신 이동하면 어떤 일이 발생하는지 살펴보겠습니다.

파일 이름: src/main.rs

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

메인 스레드는 생성된 스레드가 완료될 때까지 기다린 다음 "for" 루프를 실행하므로 다음과 같이 출력이 더 이상 인터리브되지 않습니다.

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

"join"이 호출되는 위치와 같은 작은 세부 정보는 스레드가 동시에 실행되는지 여부에 영향을 줄 수 있습니다.

### [스레드와 함께 "이동" 클로저 사용](https://doc.rust-lang.org/book/ch16-01-threads.html#using-move-closures-with-threads)

"thread::spawn"으로 전달된 클로저와 함께 "move" 키워드를 자주 사용합니다. 클로저가 환경에서 사용하는 값의 소유권을 가져가서 해당 값의 소유권을 한 스레드에서 다른 스레드로 이전하기 때문입니다. 13장의 ["참조 캡처 또는 소유권 이동"](https://doc.rust-lang.org/book/ch13-01-closures.html#capturing-references-or-moving-ownership) 섹션에서 클로저의 맥락에서 "이동"에 대해 논의했습니다. 이제 "이동"과 "thread::spawn" 사이의 상호 작용에 더 집중하겠습니다.

목록 16-1에서 우리가 "thread::spawn"에 전달하는 클로저는 인자를 취하지 않는다는 것을 주목하세요: 우리는 스폰된 스레드의 코드에서 메인 스레드의 데이터를 사용하지 않습니다. 생성된 스레드에서 메인 스레드의 데이터를 사용하려면 생성된 스레드의 클로저가 필요한 값을 캡처해야 합니다. 목록 16-3은 메인 스레드에서 벡터를 생성하고 생성된 스레드에서 사용하려는 시도를 보여줍니다. 그러나 잠시 후에 보게 되겠지만 이것은 아직 작동하지 않습니다.

파일 이름: src/main.rs

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here"s a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

Listing 16-3: 메인 스레드가 생성한 벡터를 다른 스레드에서 사용하려고 시도

클로저는 "v"를 사용하므로 "v"를 캡처하여 클로저 환경의 일부로 만듭니다. "thread::spawn"은 새 스레드에서 이 클로저를 실행하기 때문에 새 스레드 내부에서 "v"에 액세스할 수 있어야 합니다. 하지만 이 예제를 컴파일하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0373]: closure may outlive the current function, but it borrows "v", which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value "v"
7 |         println!("Here"s a vector: {:?}", v);
  |                                           - "v" is borrowed here
  |
note: function requires argument type to outlive ""static"
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here"s a vector: {:?}", v);
8 | |     });
  | |______^
help: to force the closure to take ownership of "v" (and any other referenced variables), use the "move" keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

For more information about this error, try "rustc --explain E0373".
error: could not compile "threads" due to previous error
```

Rust는 *"* v"를 캡처하는 방법과 "println!" "v"에 대한 참조만 필요하고 클로저는 "v"를 빌리기 위해 시도합니다. 그러나 문제가 있습니다. Rust는 생성된 스레드가 얼마나 오래 실행되는지 알 수 없으므로 "v"에 대한 참조가 항상 유효한지 알 수 없습니다.

목록 16-4는 유효하지 않은 "v"에 대한 참조를 가질 가능성이 더 높은 시나리오를 제공합니다:

파일 이름: src/main.rs

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here"s a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

목록 16-4: "v"를 드롭하는 메인 스레드에서 "v"에 대한 참조를 캡처하려고 시도하는 클로저가 있는 스레드

Rust가 이 코드를 실행하도록 허용하면 생성된 스레드가 전혀 실행되지 않고 즉시 백그라운드에 배치될 가능성이 있습니다. 생성된 스레드는 내부에 "v"에 대한 참조를 가지고 있지만 메인 스레드는 15장에서 논의한 "drop" 함수를 사용하여 즉시 "v"를 삭제합니다. 그러면 생성된 스레드가 실행되기 시작하면 "v"는 더 이상 존재하지 않습니다. 유효하므로 이에 대한 참조도 유효하지 않습니다. 안 돼!

목록 16-3의 컴파일러 오류를 수정하기 위해 오류 메시지의 조언을 사용할 수 있습니다.

```text
help: to force the closure to take ownership of "v" (and any other referenced variables), use the "move" keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

클로저 앞에 "이동" 키워드를 추가함으로써 러스트가 값을 빌려야 한다고 추론하도록 허용하지 않고 클로저가 사용하는 값의 소유권을 갖도록 합니다. Listing 16-5에 표시된 Listing 16-3에 대한 수정 사항은 의도한 대로 컴파일되고 실행됩니다.

파일 이름: src/main.rs

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here"s a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

Listing 16-5: "move" 키워드를 사용하여 클로저가 사용하는 값의 소유권을 가지도록 합니다.

"이동" 클로저를 사용하여 메인 스레드가 "드롭"을 호출한 Listing 16-4의 코드를 수정하기 위해 동일한 작업을 시도하고 싶은 유혹을 느낄 수 있습니다. 그러나 Listing 16-4가 하려는 것이 다른 이유로 허용되지 않기 때문에 이 수정은 작동하지 않습니다. 클로저에 "move"를 추가하면 클로저의 환경으로 "v"를 이동하고 더 이상 메인 스레드에서 "drop"을 호출할 수 없습니다. 대신 다음 컴파일러 오류가 발생합니다.

```bash
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0382]: use of moved value: "v"
  --> src/main.rs:10:10
   |
4  |     let v = vec![1, 2, 3];
   |         - move occurs because "v" has type "Vec<i32>", which does not implement the "Copy" trait
5  |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved into closure here
7  |         println!("Here"s a vector: {:?}", v);
   |                                           - variable moved due to use in closure
...
10 |     drop(v); // oh no!
   |          ^ value used here after move

For more information about this error, try "rustc --explain E0382".
error: could not compile "threads" due to previous error
```

Rust의 소유권 규칙이 우리를 다시 구했습니다! Listing 16-3의 코드에서 오류가 발생했습니다. 왜냐하면 Rust는 보수적이고 스레드에 대해 "v"만 차용했기 때문입니다. 이는 메인 스레드가 이론적으로 생성된 스레드의 참조를 무효화할 수 있음을 의미했습니다. Rust에게 "v"의 소유권을 생성된 스레드로 이동하라고 지시함으로써 우리는 Rust에게 메인 스레드가 더 이상 "v"를 사용하지 않을 것임을 보장합니다. 목록 16-4를 같은 방식으로 변경하면 메인 스레드에서 "v"를 사용하려고 할 때 소유권 규칙을 위반하는 것입니다. "이동" 키워드는 Rust의 보수적인 차용 기본값을 무시합니다. 소유권 규칙을 위반하지 않습니다.

스레드와 스레드 API에 대한 기본적인 이해를 바탕으로 스레드로 무엇 *을 할* 수 있는지 살펴보겠습니다.

------

## [메시지 전달을 사용하여 스레드 간 데이터 전송](https://doc.rust-lang.org/book/ch16-02-message-passing.html#using-message-passing-to-transfer-data-between-threads)

안전한 동시성을 보장하기 위해 점점 인기를 얻고 있는 접근 방식 중 하나는 스레드 또는 액터가 데이터가 포함된 메시지를 서로 전송하여 통신하는 *메시지 전달 입니다.* [다음은 Go 언어 문서](https://golang.org/doc/effective_go.html#concurrency) 의 슬로건에 있는 아이디어입니다. “메모리를 공유하여 통신하지 마십시오. 대신 의사 소통을 통해 메모리를 공유하십시오.”

*메시지 전송 동시성을 달성하기 위해 Rust의 표준 라이브러리는 채널* 구현을 제공합니다. 채널은 한 스레드에서 다른 스레드로 데이터를 보내는 일반적인 프로그래밍 개념입니다.

프로그래밍의 채널은 개울이나 강과 같은 물의 방향성 채널과 같다고 상상할 수 있습니다. 고무 오리 같은 것을 강물에 넣으면 하류로 흘러가 물길 끝까지 갑니다.

채널에는 송신기와 수신기의 두 부분이 있습니다. 송신기 절반은 고무 오리를 강에 넣는 상류 위치이고 수신기 절반은 고무 오리가 하류로 끝나는 곳입니다. 코드의 한 부분은 전송하려는 데이터를 사용하여 전송기의 메서드를 호출하고 다른 부분은 메시지 도착을 위해 수신단을 확인합니다. 송신기 또는 수신기 절반이 떨어지면 채널이 *닫혔다* 고 합니다.

여기에서 우리는 값을 생성하고 채널로 보내는 하나의 스레드와 값을 수신하고 출력하는 다른 스레드가 있는 프로그램을 만들 것입니다. 기능을 설명하기 위해 채널을 사용하여 스레드 간에 간단한 값을 보낼 것입니다. 이 기술에 익숙해지면 채팅 시스템이나 많은 스레드가 계산의 일부를 수행하고 해당 부분을 하나의 스레드로 보내는 시스템과 같이 서로 통신해야 하는 모든 스레드에 대해 채널을 사용할 수 있습니다. 결과.

먼저 Listing 16-6에서 채널을 생성하지만 아무 작업도 수행하지 않습니다. Rust는 채널을 통해 어떤 유형의 값을 보내고 싶은지 알 수 없기 때문에 아직 컴파일되지 않는다는 점에 유의하세요.

파일 이름: src/main.rs

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

목록 16-6: 채널 생성 및 "tx" 및 "rx"에 두 개의 절반 할당

"mpsc::channel" 기능을 사용하여 새 채널을 만듭니다. *"mpsc"는 다중 생산자, 단일 소비자를* 나타냅니다. 요컨대, Rust의 표준 라이브러리가 채널을 구현하는 방식은 채널이 값을 생성하는 여러 송신단을 가질 수 *있지만* 해당 값을 소비하는 수신단은 *하나만* 가질 수 있음을 의미합니다. 하나의 큰 강으로 함께 흐르는 여러 개울을 상상해 보십시오. 개울로 보내진 모든 것은 결국 하나의 강으로 끝납니다. 지금은 단일 생산자로 시작하지만 이 예제가 작동하면 여러 생산자를 추가할 것입니다.

"mpsc::channel" 함수는 튜플을 반환하며, 첫 번째 요소는 송신측(송신기)이고 두 번째 요소는 수신측(수신기)입니다. 약어 "tx" 및 "rx"는 전통적으로 각각 *송신기* 및 *수신기* 에 대해 많은 분야에서 사용되므로 각 끝을 나타내기 위해 변수 이름을 지정합니다. 튜플을 분해하는 패턴이 있는 "let" 문을 사용하고 있습니다. 18장에서 "let" 문에서의 패턴 사용과 구조 분해에 대해 논의할 것입니다. 지금은 "let" 문을 이 방법으로 사용하는 것이 "mpsc::channel에서 반환된 튜플 조각을 추출하는 편리한 접근 방식이라는 것을 알고 계십시오. ".

목록 16-7에 표시된 것처럼 생성된 스레드가 주 스레드와 통신하도록 전송 끝을 생성된 스레드로 이동하고 하나의 문자열을 보내도록 합시다. 이것은 강 상류에 고무 오리를 두거나 한 스레드에서 다른 스레드로 채팅 메시지를 보내는 것과 같습니다.

파일 이름: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

Listing 16-7: 생성된 스레드로 "tx" 이동 및 "hi" 전송

다시, "thread::spawn"을 사용하여 새 스레드를 생성한 다음 "move"를 사용하여 "tx"를 클로저로 이동하여 생성된 스레드가 "tx"를 소유하도록 합니다. 생성된 스레드는 채널을 통해 메시지를 보낼 수 있도록 송신기를 소유해야 합니다. 송신기에는 우리가 보내려는 값을 받는 "보내기" 메서드가 있습니다. "send" 메서드는 "Result<T, E>" 유형을 반환하므로 수신자가 이미 삭제되었고 값을 보낼 곳이 없는 경우 보내기 작업은 오류를 반환합니다. 이 예제에서는 "unwrap"을 호출하여 오류가 발생할 경우 당황하게 합니다. 그러나 실제 응용 프로그램에서는 적절하게 처리할 것입니다. 적절한 오류 처리를 위한 전략을 검토하려면 9장으로 돌아가십시오.

목록 16-8에서 우리는 메인 스레드의 수신자로부터 값을 얻을 것입니다. 이것은 마치 강가에 있는 물에서 고무 오리를 회수하거나 채팅 메시지를 받는 것과 같습니다.

파일 이름: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

Listing 16-8: 메인 스레드에서 "hi" 값을 받고 출력하기

수신기에는 "recv" 및 "try_recv"라는 두 가지 유용한 메서드가 있습니다. 우리는 메인 스레드의 실행을 차단하고 값이 채널 아래로 전송될 때까지 대기하는 *" recv"를 사용하고 있습니다.* 값이 전송되면 "recv"는 "Result<T, E>"로 값을 반환합니다. 송신기가 닫히면 "recv"는 더 이상 값이 오지 않는다는 신호로 오류를 반환합니다.

"try_recv" 메서드는 차단되지 않지만 대신 "Result<T, E>"를 즉시 반환합니다. 메시지가 있는 경우 메시지를 포함하는 "Ok" 값과 메시지가 없으면 "Err" 값을 반환합니다. 이 시간. "try_recv"를 사용하는 것은 이 스레드가 메시지를 기다리는 동안 해야 할 다른 작업이 있는 경우에 유용합니다. "try_recv"를 자주 호출하고 메시지가 사용 가능한 경우 메시지를 처리하고 잠시 동안 다른 작업을 수행하는 루프를 작성할 수 있습니다. 다시 확인할 때까지.

이 예제에서는 단순화를 위해 "recv"를 사용했습니다. 메인 스레드가 메시지를 기다리는 것 외에는 할 일이 없으므로 메인 스레드를 차단하는 것이 적절합니다.

목록 16-8의 코드를 실행하면 메인 스레드에서 출력된 값을 볼 수 있습니다.

```text
Got: hi
```

완벽한!

### [채널 및 소유권 이전](https://doc.rust-lang.org/book/ch16-02-message-passing.html#channels-and-ownership-transference)

소유권 규칙은 안전한 동시 코드를 작성하는 데 도움이 되므로 메시지 전송에서 중요한 역할을 합니다. 동시 프로그래밍에서 오류를 방지하는 것은 Rust 프로그램 전체에서 소유권에 대해 생각하는 이점입니다. 문제를 방지하기 위해 채널과 소유권이 함께 작동하는 방식을 보여주는 실험을 해 봅시다. 스레드를 채널 아래로 보낸 *후* 생성된 스레드에서 "val" 값을 사용하려고 합니다. Listing 16-9의 코드를 컴파일하여 이 코드가 허용되지 않는 이유를 확인하십시오:

파일 이름: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

목록 16-9: "val"을 채널 아래로 보낸 후 사용 시도

여기에서 "tx.send"를 통해 채널 아래로 전송한 후 "val"을 인쇄하려고 합니다. 이를 허용하는 것은 나쁜 생각입니다. 값이 다른 스레드로 전송되면 해당 스레드는 값을 다시 사용하려고 시도하기 전에 값을 수정하거나 삭제할 수 있습니다. 잠재적으로 다른 스레드의 수정으로 인해 일관성이 없거나 존재하지 않는 데이터로 인해 오류 또는 예기치 않은 결과가 발생할 수 있습니다. 그러나 Listing 16-9의 코드를 컴파일하려고 하면 Rust에서 오류가 발생합니다.

```bash
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: "val"
  --> src/main.rs:10:31
   |
8  |         let val = String::from("hi");
   |             --- move occurs because "val" has type "String", which does not implement the "Copy" trait
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value borrowed here after move
   |
   = note: this error originates in the macro "$crate::format_args_nl" which comes from the expansion of the macro "println" (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try "rustc --explain E0382".
error: could not compile "message-passing" due to previous error
```

동시성 실수로 인해 컴파일 시간 오류가 발생했습니다. "보내기" 함수는 해당 매개변수의 소유권을 가지며 값이 이동되면 수신자가 소유권을 갖습니다. 이렇게 하면 값을 보낸 후 실수로 값을 다시 사용하는 것을 방지할 수 있습니다. 소유권 시스템은 모든 것이 정상인지 확인합니다.

### [여러 값을 보내고 기다리는 수신기 보기](https://doc.rust-lang.org/book/ch16-02-message-passing.html#sending-multiple-values-and-seeing-the-receiver-waiting)

목록 16-8의 코드는 컴파일되고 실행되었지만 두 개의 개별 스레드가 채널을 통해 서로 대화하고 있음을 명확하게 보여주지 않았습니다. 목록 16-10에서 우리는 목록 16-8의 코드가 동시에 실행되고 있음을 증명할 약간의 수정을 했습니다. 생성된 스레드는 이제 여러 메시지를 보내고 각 메시지 사이에 1초 동안 일시 중지합니다.

파일 이름: src/main.rs

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

Listing 16-10: 여러 메시지를 보내고 각 메시지 사이에 일시 중지

이번에 생성된 스레드에는 기본 스레드로 보내려는 문자열 벡터가 있습니다. 우리는 그것들을 반복하고, 각각을 개별적으로 보내고, "Duration" 값이 1초인 "thread::sleep" 함수를 호출하여 각각 사이에서 일시 중지합니다.

메인 스레드에서 우리는 더 이상 "recv" 함수를 명시적으로 호출하지 않습니다. 대신 "rx"를 이터레이터로 취급합니다. 받은 각 값에 대해 인쇄합니다. 채널이 닫히면 반복이 종료됩니다.

목록 16-10의 코드를 실행할 때 각 줄 사이에 1초간 일시 중지가 있는 다음 출력이 표시되어야 합니다.

```text
Got: hi
Got: from
Got: the
Got: thread
```

메인 스레드의 "for" 루프에서 일시 중지하거나 지연시키는 코드가 없기 때문에 메인 스레드가 생성된 스레드로부터 값을 받기 위해 대기하고 있음을 알 수 있습니다.

### [송신기를 복제하여 여러 생산자 만들기](https://doc.rust-lang.org/book/ch16-02-message-passing.html#creating-multiple-producers-by-cloning-the-transmitter)

*앞에서 "mpsc"가 다중 생산자, 단일 소비자* 의 약어라고 언급했습니다. "mpsc"를 사용하고 Listing 16-10의 코드를 확장하여 모두 동일한 수신자에게 값을 보내는 여러 스레드를 생성해 보겠습니다. Listing 16-11과 같이 송신기를 복제하여 그렇게 할 수 있습니다.

파일 이름: src/main.rs

```rust
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
```

Listing 16-11: 여러 생산자로부터 여러 메시지 보내기

이번에는 첫 번째 생성된 스레드를 만들기 전에 송신기에서 "clone"을 호출합니다. 이렇게 하면 첫 번째 생성된 스레드에 전달할 수 있는 새 송신기가 제공됩니다. 원래 송신기를 두 번째 생성된 스레드로 전달합니다. 이렇게 하면 각각 다른 메시지를 하나의 수신자에게 보내는 두 개의 스레드가 제공됩니다.

코드를 실행하면 출력은 다음과 같아야 합니다.

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

시스템에 따라 값이 다른 순서로 표시될 수 있습니다. 이것이 동시성을 흥미롭고 어렵게 만드는 것입니다. "thread::sleep"을 실험하고 다른 스레드에서 다양한 값을 제공하면 각 실행은 더 비결정적이며 매번 다른 출력을 생성합니다.

이제 채널이 작동하는 방식을 살펴보았으므로 다른 동시성 방법을 살펴보겠습니다.

------

## [공유 상태 동시성](https://doc.rust-lang.org/book/ch16-03-shared-state.html#shared-state-concurrency)

메시지 전달은 동시성을 처리하는 훌륭한 방법이지만 유일한 방법은 아닙니다. 또 다른 방법은 여러 스레드가 동일한 공유 데이터에 액세스하는 것입니다. Go 언어 문서의 슬로건 중 이 부분을 다시 생각해 보십시오. "메모리를 공유하여 통신하지 마십시오."

메모리 공유를 통한 통신은 어떤 모습일까요? 또한 메시지 전달 애호가들이 메모리 공유를 사용하지 말라고 주의하는 이유는 무엇입니까?

어떤 면에서 모든 프로그래밍 언어의 채널은 단일 소유권과 유사합니다. 값을 채널 아래로 전송하면 더 이상 해당 값을 사용하지 않아야 하기 때문입니다. 공유 메모리 동시성은 다중 소유권과 같습니다. 여러 스레드가 동시에 동일한 메모리 위치에 액세스할 수 있습니다. 스마트 포인터가 다중 소유권을 가능하게 한 15장에서 보았듯이 다중 소유권은 이러한 서로 다른 소유자가 관리해야 하기 때문에 복잡성을 추가할 수 있습니다. Rust의 타입 시스템과 소유권 규칙은 이 관리를 올바르게 하는 데 크게 도움이 됩니다. 예를 들어, 공유 메모리에 대한 보다 일반적인 동시성 프리미티브 중 하나인 뮤텍스를 살펴보겠습니다.

### [뮤텍스를 사용하여 한 번에 하나의 스레드에서 데이터 액세스 허용](https://doc.rust-lang.org/book/ch16-03-shared-state.html#using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time)

*뮤텍스 는* *상호 배제* 의 약자입니다. 뮤텍스는 주어진 시간에 하나의 스레드만 일부 데이터에 액세스할 수 있도록 허용합니다. 뮤텍스의 데이터에 액세스하려면 스레드는 먼저 뮤텍스의 *잠금* 획득을 요청하여 액세스를 원한다는 신호를 보내야 합니다. 잠금은 현재 누가 데이터에 독점적으로 액세스할 수 있는지 추적하는 뮤텍스의 일부인 데이터 구조입니다. 따라서 뮤텍스는 잠금 시스템을 통해 보유하고 있는 데이터를 *보호하는 것으로 설명됩니다.*

뮤텍스는 다음 두 가지 규칙을 기억해야 하기 때문에 사용하기 어렵다는 평판이 있습니다.

- 데이터를 사용하기 전에 잠금 획득을 시도해야 합니다.
- 뮤텍스가 보호하는 데이터 작업을 마치면 다른 스레드가 잠금을 획득할 수 있도록 데이터 잠금을 해제해야 합니다.

뮤텍스에 대한 실제 비유를 들어 회의에서 마이크가 하나만 있는 패널 토론을 상상해 보십시오. 패널리스트가 발언하기 전에 마이크를 사용하고 싶다고 묻거나 신호를 보내야 합니다. 마이크를 받으면 원하는 만큼 길게 이야기한 후 발언을 요청하는 다음 패널리스트에게 마이크를 건네줄 수 있습니다. 패널리스트가 마이크 사용을 마치고 마이크를 빼는 것을 잊어버리면 아무도 발언할 수 없습니다. 공유 마이크 관리가 잘못되면 패널이 계획대로 작동하지 않습니다!

뮤텍스 관리는 제대로 하기가 엄청나게 까다로울 수 있으며, 이것이 바로 많은 사람들이 채널에 열광하는 이유입니다. 그러나 Rust의 유형 시스템 및 소유권 규칙 덕분에 잠금 및 잠금 해제가 잘못될 수 없습니다.

#### ["Mutex"의 API](https://doc.rust-lang.org/book/ch16-03-shared-state.html#the-api-of-mutext)

뮤텍스를 사용하는 방법의 예로, 목록 16-12에 표시된 것처럼 단일 스레드 컨텍스트에서 뮤텍스를 사용하여 시작하겠습니다.

파일 이름: src/main.rs

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

목록 16-12: "Mutex"의 API 탐색" 단순화를 위해 단일 스레드 컨텍스트에서

많은 유형과 마찬가지로 "Mutex"를 생성합니다." 연관된 함수 "new"를 사용합니다. 뮤텍스 내부의 데이터에 액세스하려면 "lock" 메서드를 사용하여 잠금을 획득합니다. 이 호출은 현재 스레드를 차단하므로 우리 차례가 될 때까지 아무 작업도 수행할 수 없습니다. 자물쇠.

잠금을 보유한 다른 스레드가 패닉 상태가 되면 "잠금"에 대한 호출이 실패합니다. 이 경우 아무도 잠금을 얻을 수 없으므로 "언래핑"을 선택하고 해당 상황에 있는 경우 이 스레드를 패닉 상태로 만듭니다.

잠금을 획득한 후 반환 값(이 경우 "num")을 내부 데이터에 대한 변경 가능한 참조로 처리할 수 있습니다. 유형 시스템은 "m"의 값을 사용하기 전에 잠금을 획득하도록 합니다. "m"의 유형은 "Mutex"입니다.", "i32"가 아니라 "i32" 값을 사용하려면 "lock"을 호출 *해야 합니다* . 그렇지 않으면 유형 시스템에서 내부 "i32"에 액세스할 수 없습니다.

짐작할 수 있듯이 "Mutex"는 스마트 포인터입니다. 더 정확하게는 "잠금"에 대한 호출은 " unwrap"에 대한 호출로 처리한 "LockResult"에 래핑된 "MutexGuard"라는 스마트 포인터를 반환합니다. "MutexGuard" 스마트 포인터는 " *Deref* " 내부 데이터를 가리키기 위해 스마트 포인터에는 "MutexGuard"가 내부 범위의 끝에서 발생하는 범위를 벗어날 때 자동으로 잠금을 해제하는 "Drop" 구현도 있습니다. 결과적으로 우리는 ' 잠금 해제가 자동으로 발생하기 때문에 잠금 해제를 잊고 다른 스레드에서 사용하는 뮤텍스를 차단할 위험이 있습니다.

잠금을 해제한 후 뮤텍스 값을 인쇄하고 내부 "i32"를 6으로 변경할 수 있음을 확인할 수 있습니다.

#### [여러 스레드 간에 "뮤텍스" 공유](https://doc.rust-lang.org/book/ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads)

이제 "Mutex"를 사용하여 여러 스레드 간에 값을 공유해 보겠습니다.". 우리는 10개의 스레드를 스핀업하고 각각 카운터 값을 1씩 증가시켜 카운터가 0에서 10으로 가도록 할 것입니다. 목록 16-13의 다음 예제에는 컴파일러 오류가 있으며 이 오류를 사용하겠습니다. "Mutex" 사용에 대해 자세히 알아보려면" 그리고 Rust가 우리가 그것을 올바르게 사용하는 데 어떻게 도움이 되는지.

파일 이름: src/main.rs

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

목록 16-13: 10개의 스레드는 각각 "Mutex"에 의해 보호되는 카운터를 증가시킵니다."

"Mutex" 내부에 "i32"를 보유하기 위해 "카운터" 변수를 생성합니다.", 목록 16-12에서 했던 것처럼. 다음으로, 숫자 범위를 반복하여 10개의 스레드를 생성합니다. "thread::spawn"을 사용하고 모든 스레드에 동일한 클로저를 제공합니다. 하나는 카운터를 스레드로 이동시키는 것입니다. , "Mutex에 대한 잠금을 획득합니다." "lock" 메서드를 호출하여 뮤텍스의 값에 1을 추가합니다. 스레드가 클로저 실행을 완료하면 "num"은 범위를 벗어나 잠금을 해제하여 다른 스레드가 획득할 수 있도록 합니다.

기본 스레드에서 모든 조인 핸들을 수집합니다. 그런 다음 Listing 16-2에서 했던 것처럼 모든 스레드가 완료되었는지 확인하기 위해 각 핸들에서 "join"을 호출합니다. 그 시점에서 주 스레드는 잠금을 획득하고 이 프로그램의 결과를 인쇄합니다.

우리는 이 예제가 컴파일되지 않을 것이라고 암시했습니다. 이제 그 이유를 알아봅시다!

```bash
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: use of moved value: "counter"
  --> src/main.rs:9:36
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because "counter" has type "Mutex<i32>", which does not implement the "Copy" trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^ value moved into closure here, in previous iteration of loop
10 |             let mut num = counter.lock().unwrap();
   |                           ------- use occurs due to use in closure

For more information about this error, try "rustc --explain E0382".
error: could not compile "shared-state" due to previous error
```

오류 메시지는 루프의 이전 반복에서 "카운터" 값이 이동되었음을 나타냅니다. Rust는 잠금 "카운터"의 소유권을 여러 스레드로 옮길 수 없다고 말합니다. 15장에서 논의한 다중 소유권 방식으로 컴파일러 오류를 수정해 보겠습니다.

#### [다중 스레드의 다중 소유권](https://doc.rust-lang.org/book/ch16-03-shared-state.html#multiple-ownership-with-multiple-threads)

15장에서 스마트 포인터 "Rc"를 사용하여 값에 여러 소유자를 부여했습니다." 참조 카운트 값을 생성합니다. 여기에서도 동일한 작업을 수행하고 어떤 일이 발생하는지 확인하겠습니다. "Mutex"에서 "RC"를 목록 16-14에서 "Rc" 소유권을 스레드로 이동하기 전에.

파일 이름: src/main.rs

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

목록 16-14: "Rc 사용 시도" 여러 스레드가 "Mutex"

다시 한 번, 우리는 컴파일하고 ... 다른 오류를 얻습니다! 컴파일러는 우리에게 많은 것을 가르쳐줍니다.

```bash
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0277]: "Rc<Mutex<i32>>" cannot be sent between threads safely
  --> src/main.rs:11:36
   |
11 |           let handle = thread::spawn(move || {
   |                        ------------- ^------
   |                        |             |
   |  ______________________|_____________within this "[closure@src/main.rs:11:36: 11:43]"
   | |                      |
   | |                      required by a bound introduced by this call
12 | |             let mut num = counter.lock().unwrap();
13 | |
14 | |             *num += 1;
15 | |         });
   | |_________^ "Rc<Mutex<i32>>" cannot be sent between threads safely
   |
   = help: within "[closure@src/main.rs:11:36: 11:43]", the trait "Send" is not implemented for "Rc<Mutex<i32>>"
note: required because it"s used within this closure
  --> src/main.rs:11:36
   |
11 |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^
note: required by a bound in "spawn"
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:704:8
   |
   = note: required by this bound in "spawn"

For more information about this error, try "rustc --explain E0277".
error: could not compile "shared-state" due to previous error
```

와우, 그 오류 메시지는 매우 장황합니다! 집중해야 할 중요한 부분은 다음과 같습니다. ""Rc<Mutex>" 스레드 간에 안전하게 보낼 수 없습니다." 컴파일러는 또한 "Rc<Mutex에 대해 "Send" 특성이 구현되지 않은 이유를 알려줍니다.>" ". 다음 섹션에서 "보내기"에 대해 이야기하겠습니다. 스레드와 함께 사용하는 유형이 동시 상황에서 사용되도록 보장하는 특성 중 하나입니다.

아쉽게도 "RC"는 스레드 간에 공유하기에 안전하지 않습니다. "Rc" 참조 카운트를 관리하고, "clone"에 대한 각 호출에 대한 카운트에 추가하고 각 클론이 삭제될 때 카운트에서 뺍니다. 그러나 카운트에 대한 변경이 불가능하도록 동시성 프리미티브를 사용하지 않습니다. 다른 스레드에 의해 중단됩니다. 이로 인해 잘못된 카운트가 발생할 수 있습니다. 메모리 누수 또는 작업이 완료되기 전에 값이 삭제될 수 있는 미묘한 버그입니다. 우리에게 필요한 것은 정확히 "Rc"와 같은 유형입니다." 그러나 스레드로부터 안전한 방식으로 참조 횟수를 변경하는 것입니다.

#### ["Arc"를 사용한 원자 참조 카운팅](https://doc.rust-lang.org/book/ch16-03-shared-state.html#atomic-reference-counting-with-arct)

다행히 "아크" 는 "Rc와 같은 유형 *입니다.*" 는 동시 상황에서 사용하기에 안전합니다. a 는 *atomic* *를* 나타내며 *원자적으로 참조 카운트되는* 유형 임을 의미합니다. Atomics는 여기서 자세히 다루지 않을 동시성 프리미티브의 추가적인 종류입니다. ["std 에 대한 표준 라이브러리 문서를 참조하십시오. ::sync::atomic"을](https://doc.rust-lang.org/std/sync/atomic/index.html) 참조하십시오. 이 시점에서 원자는 기본 유형처럼 작동하지만 스레드 간에 공유하는 것이 안전하다는 것을 알아야 합니다.

그런 다음 모든 기본 유형이 원자적이지 않은 이유와 표준 라이브러리 유형이 "Arc"를 사용하도록 구현되지 않은 이유가 궁금할 수 있습니다." 기본적으로. 그 이유는 스레드 안전에는 실제로 필요할 때만 지불하려는 성능 패널티가 있기 때문입니다. 단일 스레드 내에서 값에 대한 작업을 수행하는 경우 코드가 더 빨리 실행될 수 있습니다. 원자가 제공하는 보증을 시행해야 합니다.

예를 들어 보겠습니다. "아크"와 "RC"는 동일한 API를 가지고 있으므로 "use" 줄, "new" 호출 및 "clone" 호출을 변경하여 프로그램을 수정합니다. Listing 16-15의 코드는 최종적으로 컴파일되고 실행됩니다.

파일 이름: src/main.rs

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Listing 16-15: "호" 사용하기"를 래핑하려면 "뮤텍스" 여러 스레드에서 소유권을 공유할 수 있도록

이 코드는 다음을 인쇄합니다.

```text
Result: 10
```

우리는 해냈다! 우리는 0부터 10까지 세었는데 그다지 인상적이지 않을 수도 있지만 "Mutex"에 대해 많은 것을 가르쳐 주었습니다." 및 스레드 안전성. 또한 이 프로그램의 구조를 사용하여 카운터를 증가시키는 것보다 더 복잡한 작업을 수행할 수 있습니다. 이 전략을 사용하면 계산을 독립적인 부분으로 나누고 해당 부분을 스레드 간에 분할한 다음 "Mutex"를 사용할 수 있습니다." 각 스레드가 해당 부분으로 최종 결과를 업데이트하도록 합니다.

간단한 숫자 연산을 수행하는 경우 "Mutex"보다 간단한 유형이 있습니다.[" 표준 라이브러리의 "std::sync::atomic" 모듈](https://doc.rust-lang.org/std/sync/atomic/index.html) 에서 제공하는 유형 . 이러한 유형은 기본 유형에 대한 안전하고 동시적인 원자 액세스를 제공합니다. 우리는 "Mutex"를 사용하기로 선택했습니다."를 기본 유형으로 사용하여 "Mutex" 방법에 집중할 수 있습니다." 작동합니다.

### ["RefCell"/"Rc"와 "Mutex"/"Arc"의 유사점](https://doc.rust-lang.org/book/ch16-03-shared-state.html#similarities-between-refcelltrct-and-mutextarct)

"카운터"는 변경 불가능하지만 그 안에 있는 값에 대한 변경 가능한 참조를 얻을 수 있음을 알아차렸을 것입니다. 이것은 "뮤텍스"는 "Cell" 계열과 마찬가지로 내부 가변성을 제공합니다. 같은 방식으로 "RefCell" 15장에서 "Rc 내부의 내용을 변경할 수 있도록 합니다.", 우리는 "뮤텍스"는 "Arc 내부의 내용을 변경합니다.".

주의해야 할 또 다른 세부 사항은 Rust가 "Mutex"를 사용할 때 모든 종류의 논리 오류로부터 사용자를 보호할 수 없다는 것입니다.". 15장에서 "Rc" 참조 순환을 생성할 위험이 있습니다. 여기서 두 개의 "Rc" 값이 서로를 참조하여 메모리 누수를 일으킵니다. 유사하게 "Mutex*" 교착 상태* 가 발생할 위험이 있습니다. 작업이 두 개의 리소스를 잠글 필요가 있고 두 스레드가 각각 잠금 중 하나를 획득하여 서로를 영원히 기다리게 할 때 발생합니다. 교착 상태에 관심이 있는 경우 교착 상태를 만들어 보십시오. 교착 상태가 있는 Rust 프로그램; 그런 다음 모든 언어의 뮤텍스에 대한 교착 상태 완화 전략을 연구하고 이를 Rust에서 구현해 봅니다. "Mutex에 대한 표준 라이브러리 API 문서" 및 "MutexGuard"는 유용한 정보를 제공합니다.

"보내기" 및 "동기화" 특성과 이를 사용자 정의 유형과 함께 사용하는 방법에 대해 이야기하면서 이 장을 마무리하겠습니다.

------

## ["동기화" 및 "보내기" 특성을 사용한 확장 가능한 동시성](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits)

흥미롭게도 Rust 언어에는 동시성 기능이 *거의 없습니다.* 이 장에서 지금까지 이야기한 거의 모든 동시성 기능은 언어가 아니라 표준 라이브러리의 일부였습니다. 동시성을 처리하기 위한 옵션은 언어나 표준 라이브러리에 국한되지 않습니다. 자신의 동시성 기능을 작성하거나 다른 사람이 작성한 기능을 사용할 수 있습니다.

그러나 두 가지 동시성 개념이 언어에 포함되어 있습니다. "std::marker" 특성 "Sync" 및 "Send"입니다.

### ["보내기"를 사용하여 스레드 간 소유권 이전 허용](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#allowing-transference-of-ownership-between-threads-with-send)

"보내기" 마커 특성은 "보내기"를 구현하는 유형의 값 소유권이 스레드 간에 전송될 수 있음을 나타냅니다. 거의 모든 Rust 유형은 "Send"이지만 "Rc"를 포함하여 몇 가지 예외가 있습니다.": "Rc를 복제한 경우 "보내기"가 될 수 없습니다." 값을 지정하고 복제본의 소유권을 다른 스레드로 이전하려고 하면 두 스레드가 동시에 참조 횟수를 업데이트할 수 있습니다. 이러한 이유로 "Rc"는 스레드로부터 안전한 성능 패널티를 지불하고 싶지 않은 단일 스레드 상황에서 사용하기 위해 구현됩니다.

따라서 Rust의 유형 시스템과 특성 범위는 실수로 "Rc"를 보내지 않도록 합니다.Listing 16-14에서 이 작업을 시도했을 때 "Rc<Mutex에 대해 특성 Send가 구현되지 않았습니다.>". "Arc"로 전환했을 때", 코드가 컴파일된 "보내기"입니다.

"보내기" 유형으로만 구성된 모든 유형도 자동으로 "보내기"로 표시됩니다. 19장에서 논의할 원시 포인터를 제외하고 거의 모든 기본 유형은 "보내기"입니다.

### ["동기화"를 사용하여 여러 스레드에서 액세스 허용](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#allowing-access-from-multiple-threads-with-sync)

"동기화" 마커 특성은 "동기화"를 구현하는 유형이 여러 스레드에서 참조되는 것이 안전함을 나타냅니다. 즉, "&T"("T"에 대한 불변 참조)가 "보내기"인 경우 모든 유형 "T"는 "동기화"입니다. 이는 참조를 다른 스레드로 안전하게 보낼 수 있음을 의미합니다. "Send"와 유사하게 기본형은 "Sync"이며 "Sync"인 유형으로만 구성된 유형도 "Sync"입니다.

스마트 포인터 "Rc"도 "Send"가 아닌 것과 같은 이유로 "Sync"가 아닙니다. "RefCell" 유형(15장에서 언급) 및 관련된 "세포" 유형은 "동기화"가 아닙니다. "RefCell"을 확인하는 차용 구현"는 런타임에 스레드로부터 안전하지 않습니다. 스마트 포인터 "Mutex["는 "동기화"이며 "여러 스레드 간에 "뮤텍스" 공유"](https://doc.rust-lang.org/book/ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads) 섹션 에서 본 것처럼 여러 스레드와 액세스를 공유하는 데 사용할 수 있습니다.

### ["보내기" 및 "동기화"를 수동으로 구현하는 것은 안전하지 않습니다.](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#implementing-send-and-sync-manually-is-unsafe)

"Send" 및 "Sync" 특성으로 구성된 유형은 자동으로 "Send" 및 "Sync"이기도 하므로 해당 특성을 수동으로 구현할 필요가 없습니다. 마커 특성으로서 구현할 메서드도 없습니다. 동시성과 관련된 불변성을 적용하는 데 유용합니다.

이러한 특성을 수동으로 구현하려면 안전하지 않은 Rust 코드를 구현해야 합니다. 우리는 19장에서 안전하지 않은 Rust 코드를 사용하는 것에 대해 이야기할 것입니다. 지금 중요한 정보는 "Send" 및 "Sync" 부분으로 구성되지 않은 새로운 동시 유형을 빌드하려면 안전 보장을 유지하기 위해 신중한 생각이 필요하다는 것입니다. ["The Rustonomicon"에는](https://doc.rust-lang.org/nomicon/index.html) 이러한 보증과 이를 유지하는 방법에 대한 자세한 정보가 있습니다.

## [요약](https://doc.rust-lang.org/book/ch16-04-extensible-concurrency-sync-and-send.html#summary)

이것이 이 책에서 보게 될 마지막 동시성은 아닙니다. 20장의 프로젝트는 여기서 논의된 작은 예제보다 더 현실적인 상황에서 이 장의 개념을 사용할 것입니다.

앞서 언급했듯이 Rust가 동시성을 처리하는 방식이 언어의 일부이기 때문에 많은 동시성 솔루션이 크레이트로 구현됩니다. 이들은 표준 라이브러리보다 더 빠르게 발전하므로 멀티스레드 상황에서 사용할 최신 최신 크레이트를 온라인에서 검색하십시오.

Rust 표준 라이브러리는 "Mutex"와 같은 메시지 전달 및 스마트 포인터 유형을 위한 채널을 제공합니다." 및 "아크", 동시 컨텍스트에서 사용하기에 안전합니다. 유형 시스템과 차용 검사기는 이러한 솔루션을 사용하는 코드가 데이터 경합 또는 유효하지 않은 참조로 끝나지 않도록 합니다. 일단 코드를 컴파일하면 안심할 수 있습니다. 동시성 프로그래밍은 더 이상 두려워할 개념이 아닙니다.

다음으로 Rust 프로그램이 커짐에 따라 문제를 모델링하고 솔루션을 구조화하는 관용적인 방법에 대해 이야기하겠습니다. 또한 Rust의 관용구가 객체 지향 프로그래밍에서 친숙한 관용구와 어떻게 관련되는지 논의할 것입니다.
