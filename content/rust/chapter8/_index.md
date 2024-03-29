+++
title = "Common Collections"
weight = 8
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

<!-- more -->

- 컬렉션 : 표준 라이브러리에 있는 데이터 구조들
- vector, String, hash map 이 있다.
- 모두 데이터는 힙에 저장

# [공통 컬렉션](**https://doc.rust-lang.org/book/ch08-00-common-collections.html#common-collections**)

**Rust의 표준 라이브러리에는 collections** 라는 매우 유용한 데이터 구조가 많이 포함되어 있습니다. 대부분의 다른 데이터 유형은 하나의 특정 값을 나타내지만, 컬렉션에는 여러 값이 포함될 수 있습니다. 기본 제공되는 배열 및 튜플 타입과 달리, 이러한 컬렉션이 가리키는 데이터는 힙에 저장됩니다. 즉, 데이터 양은 컴파일 타임에 알 필요가 없으며 프로그램 실행에 따라 늘어나거나 줄어들 수 있습니다. 컬렉션 종류마다 기능과 비용이 다르며 현재 상황에 적합한 컬렉션을 선택하는 것은 시간이 지남에 따라 발전하게 될 기술입니다. 이 장에서는 Rust 프로그램에서 매우 자주 사용되는 세 가지 모음에 대해 논의할 것입니다.

- **vector**를 사용하면 *가변 갯수*의 값을 *서로 옆*에 저장할 수 있습니다.

- **String**은 문자 모음입니다. 앞에서 `String` 유형에 대해 언급했지만 이 장에서는 이에 대해 자세히 설명합니다.

- **hash map**을 사용 하면, 값을 특정 키와 연결할 수 있습니다. *map* 이라고 하는, 보다 일반적인 데이터 구조의 특정 구현입니다. 파이썬의 딕셔너리와 비슷합니다.

표준 라이브러리에서 제공하는 다른 종류의 컬렉션에 대해 알아보려면 [설명서](https://doc.rust-lang.org/std/collections/index.html) 를 참조하십시오 .

vectors, strings 및 hash maps을 만들고 업데이트하는 방법과 각 항목을 특별하게 만드는 방법에 대해 설명합니다.
