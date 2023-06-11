+++
title = "Understanding Ownership"
weight = 4
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

- 소유권(ownership)
- 빌림(borrowing)
- 슬라이스(slices)
- Rust가 메모리에 데이터를 배치하는 방법

<!-- more -->

# [소유권 이해](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html#understanding-ownership)

소유권은 Rust의 가장 고유한 기능이며, 나머지 언어에 깊은 영향을 미칩니다. 이를 통해 Rust는 가비지 콜렉터 없이 메모리 안전을 보장할 수 있으므로, 소유권이 작동하는 방식을 이해하는 것이 중요합니다. 이 장에서 우리는 소유권 뿐만아니라, 빌림(borrowing), 슬라이스(slices), Rust가 메모리에 데이터를 배치하는 방법과 같은 몇 가지 관련 기능에 대해 이야기할 것입니다.
