+++
title = "Common Programming Concepts"
weight = 3
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

- 변수, 기본 타입, 함수, 주석 및 흐름 제어를 알아보자.

- 키워드로 선점된 이름은 변수나 함수의 이름으로 사용할 수 없다.




<!-- more -->

# [일반적인 프로그래밍 개념](https://doc.rust-lang.org/book/ch03-00-common-programming-concepts.html#common-programming-concepts)

이 장에서는 거의 모든 프로그래밍 언어에 나타나는 개념을 다루고, Rust에서 어떻게 작동하는지 알려드립니다. 프로그래밍 언어들은 그 핵심에 많은 공통점이 있고, 이 장에 제시된 개념들은 Rust에만 있는 것이 아닙니다. 이 개념들을 Rust의 맥락에서 논의하고, 사용하는 방법을 설명할 것입니다.

특히 변수, 기본 타입, 함수, 주석 및 흐름 제어에 대해 배웁니다. 이러한 기초는 모든 Rust 프로그램에서 사용되며, 이를 초반에 배우면 매우 도움이 됩니다. 



> #### [키워드(keywords)](https://doc.rust-lang.org/book/ch03-00-common-programming-concepts.html#keywords)
>
> Rust 언어에는 다른 언어와 마찬가지로 해당 언어에서만 사용하도록 예약된 *키워드*(*keywords*)들이 있습니다. 이 키워드 이름을 변수나 함수의 이름으로 사용할 수 없음을 명심하십시오. 대부분의 키워드는 특별한 의미를 가지고 있으며 Rust 프로그램에서 다양한 작업을 수행하기 위해 키워드를 사용할 것입니다. 몇몇은 관련된 현재 기능이 없지만 미래에 Rust에 추가될 수 있는 기능을 위해 선점되었습니다. [부록 A](https://doc.rust-lang.org/book/appendix-01-keywords.html) 에서 키워드 목록을 찾을 수 있습니다 .

