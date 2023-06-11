+++
title = "Comments"
weight = 4
template ="book_page_rust.html"

+++



## 요약

- Rust 에서 주석은 `//` 뒤에 작성

- `문서 주석` 이라는 것도 있는데, `///` 를 사용해서 HTML형식의 documentation 을 생성하는데 사용됨

<!-- more -->

## [주석](https://doc.rust-lang.org/book/ch03-04-comments.html#comments)

모든 프로그래머는 코드를 이해하기 쉽게 만들기 위해 노력하지만, 때로는 추가 설명이 필요합니다. 이 경우 프로그래머는 소스 코드에 컴파일러가 무시할 *주석*을 남기고, 이 주석들은 소스 코드를 읽는 사람들에게는 유용할 수 있습니다.

다음은 간단한 주석입니다.

```rust
// hello, world
```

Rust에서 주석 스타일은 두 개의 슬래시 `//` 로 주석을 시작하고, 주석은 줄 끝까지 계속됩니다.

<br>

한 줄을 넘는 주석의 경우 다음과 같이 각 줄에 `//`를 포함해야 합니다.

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

<br>

주석은 코드를 포함하는 줄 끝에 배치할 수도 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let lucky_number = 7; // I’m feeling lucky today
}
```

<br>

그러나 주석을 코드 위에 있는 별도의 줄로 작성하는 형식을 더 자주 볼 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    // I’m feeling lucky today
    let lucky_number = 7;
}
```

<br>

Rust에는 또 다른 종류의 주석인 문서 주석(documentation comments)이 있습니다.

이에 대해서는 14장의 [`Crates.io에 크레이트 게시`](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html) 섹션 에서 논의할 것입니다.
