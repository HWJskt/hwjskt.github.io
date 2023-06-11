+++
title = "Unrecoverable Errors with panic!"
weight = 1
template = "book_page_rust.html"

+++

## 요약

## [복구할 수 없는 `패닉!` 오류](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unrecoverable-errors-with-panic)

때로는 코드에서 나쁜 일이 발생하고 이에 대해 할 수 있는 일이 없습니다. 이러한 경우 Rust는 `패닉!` 매크로. 실제로 패닉을 일으키는 방법에는 두 가지가 있습니다. 코드를 패닉 상태로 만드는 작업(예: 끝을 지나 배열에 액세스)을 수행하거나 `패닉!`을 명시적으로 호출하는 것입니다. 매크로. 두 경우 모두 프로그램에 패닉이 발생합니다. 기본적으로 이러한 패닉은 실패 메시지를 인쇄하고, 풀고, 스택을 정리하고, 종료합니다. 환경 변수를 통해 패닉이 발생할 때 러스트가 호출 스택을 표시하도록 하여 패닉의 원인을 쉽게 추적할 수 있습니다.

> ### [패닉에 대한 응답으로 스택 풀기 또는 중단](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic)
>
> 기본적으로 패닉이 발생하면 프로그램은 *풀기* 시작합니다. 즉, Rust가 스택을 백업하고 만나는 각 함수에서 데이터를 정리합니다. 그러나, 이 걷기 및 청소는 많은 작업입니다. 따라서 Rust는 정리하지 않고 프로그램을 종료하는 *즉시 중단* 의 대안을 선택할 수 있습니다.
>
> 프로그램이 사용하고 있던 메모리는 운영 체제에서 정리해야 합니다. *프로젝트에서 결과 바이너리를 가능한 한 작게 만들어야 하는 경우 Cargo.toml* 파일 의 적절한 `[profile]` 섹션에 `panic = 'abort'`를 추가하여 패닉 시 해제에서 중단으로 전환할 수 있습니다. . 예를 들어 릴리스 모드에서 패닉 상태에서 중단하려면 다음을 추가하십시오.
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

`패닉!` 간단한 프로그램에서:

파일 이름: src/main.rs

```rust
fn main() {
    panic!(`crash and burn`);
}
```

프로그램을 실행하면 다음과 같이 표시됩니다.

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`패닉!` 마지막 두 줄에 포함된 오류 메시지가 발생합니다. 첫 번째 줄은 패닉 메시지와 소스 코드에서 패닉이 발생한 위치를 보여줍니다. *src/main.rs:2:5* 는 두 번째 줄, *src/main.rs* 파일의 다섯 번째 문자임을 나타냅니다.

이 경우 표시된 줄은 코드의 일부이며 해당 줄로 이동하면 `패닉!` 매크로 호출. 다른 경우에는 `패닉!` 호출은 우리 코드가 호출하는 코드에 있을 수 있으며 오류 메시지에 의해 보고된 파일 이름과 줄 번호는 `패닉!` 매크로가 호출되지만 결국 `패닉!` 부르다. `panic!` 함수의 역추적을 사용할 수 있습니다. 문제를 일으키는 코드 부분을 파악하기 위해 전화가 왔습니다. 다음에 백트레이스에 대해 더 자세히 설명하겠습니다.

### [`패닉!` 역추적](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#using-a-panic-backtrace)

다른 예를 살펴보고 `패닉!` 호출은 매크로를 직접 호출하는 코드가 아니라 코드의 버그 때문에 라이브러리에서 발생합니다. 목록 9-1에는 유효한 인덱스 범위를 벗어난 벡터의 인덱스에 액세스하려고 시도하는 코드가 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

목록 9-1: 벡터의 끝을 넘어 요소에 액세스하려고 시도하면 `패닉!` 호출이 발생합니다.

여기에서 우리는 벡터의 100번째 요소(인덱싱이 0에서 시작하기 때문에 인덱스 99에 있음)에 액세스하려고 시도하지만 벡터에는 3개의 요소만 있습니다. 이 상황에서 Rust는 당황할 것입니다. `[]`를 사용하면 요소를 반환해야 하지만 유효하지 않은 인덱스를 전달하면 Rust가 반환할 수 있는 올바른 요소가 없습니다.

C에서 데이터 구조의 끝을 넘어 읽으려는 시도는 정의되지 않은 동작입니다. 메모리가 해당 구조에 속하지 않더라도 데이터 구조의 해당 요소에 해당하는 메모리 위치에 있는 모든 것을 얻을 수 있습니다. 이를 *버퍼 오버 읽기* 라고 하며 공격자가 데이터 구조 뒤에 저장된 허용되지 않아야 하는 데이터를 읽는 방식으로 인덱스를 조작할 수 있는 경우 보안 취약성을 초래할 수 있습니다.

이러한 종류의 취약성으로부터 프로그램을 보호하기 위해 존재하지 않는 인덱스에서 요소를 읽으려고 하면 Rust는 실행을 중지하고 계속 진행을 거부합니다. 그것을 시도하고 보자 :

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

이 오류는 색인 99에 액세스하려고 시도하는 `main.rs`의 4번째 줄을 가리킵니다. 다음 줄은 `RUST_BACKTRACE` 환경 변수를 설정하여 오류의 원인이 된 정확한 역추적을 얻을 수 있음을 알려줍니다. 역 *추적*이 지점에 도달하기 위해 호출된 모든 함수의 목록입니다. Rust의 백트레이스는 다른 언어에서와 마찬가지로 작동합니다. 백트레이스를 읽는 핵심은 맨 위에서 시작하여 작성한 파일이 보일 때까지 읽는 것입니다. 그곳이 문제의 발원지입니다. 해당 지점 위의 줄은 코드가 호출한 코드입니다. 아래 줄은 코드를 호출한 코드입니다. 이러한 전후 줄에는 핵심 Rust 코드, 표준 라이브러리 코드 또는 사용 중인 크레이트가 포함될 수 있습니다. `RUST_BACKTRACE` 환경 변수를 0을 제외한 모든 값으로 설정하여 백트레이스를 가져오도록 합시다. Listing 9-2는 여러분이 보게 될 것과 유사한 출력을 보여줍니다.

```bash
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:142:14
   2: core::panicking::panic_bounds_check
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:84:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:242:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/alloc/src/vec/mod.rs:2591:9
   6: panic::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/ops/function.rs:248:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

Listing 9-2: `panic!` 호출에 의해 생성된 백트레이스 환경 변수 `RUST_BACKTRACE`가 설정되면 표시됩니다.

출력이 엄청나네요! 표시되는 정확한 출력은 운영 체제 및 Rust 버전에 따라 다를 수 있습니다. 이 정보로 백트레이스를 얻으려면 디버그 기호를 활성화해야 합니다. 여기에 있는 것처럼 `--release` 플래그 없이 `cargo build` 또는 `cargo run`을 사용할 때 디버그 기호가 기본적으로 활성화됩니다.

목록 9-2의 출력에서 백트레이스의 6행은 문제를 일으키는 프로젝트의 *src/main.rs* 4행을 가리킵니다. 프로그램이 패닉 상태가 되는 것을 원하지 않으면 우리가 작성한 파일을 언급하는 첫 번째 줄이 가리키는 위치에서 조사를 시작해야 합니다. 목록 9-1에서 일부러 패닉을 일으키는 코드를 작성했는데 패닉을 해결하는 방법은 벡터 인덱스 범위를 벗어나는 요소를 요청하지 않는 것입니다. 나중에 코드 패닉이 발생하면 패닉을 유발하는 값과 대신 코드가 수행해야 하는 작업을 파악해야 합니다.

`패닉!`으로 돌아오겠습니다. `패닉!`을 사용해야 할 때와 사용하지 말아야 할 때 [`패닉!](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic) ` 또는 이 장 뒷부분의 [`패닉!`` 섹션을 참조하십시오 . ](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic)다음으로 `결과`를 사용하여 오류를 복구하는 방법을 살펴보겠습니다.
