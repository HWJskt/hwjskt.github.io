+++
title = "Using Structs to Structure Related Data"
weight = 5
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

<!-- more -->

# [구조체를 사용하여 관련 데이터 구조화](https://doc.rust-lang.org/book/ch05-00-structs.html#using-structs-to-structure-related-data)

struct 또는 *structure 는 사용자 정의 데이터 유형입니다. 이것으로 값들을 함께 모으고(패키징하고)  이 값들의 이름을 지정해서 의미 있는 그룹을 구성할 수 있습니다. 객체 지향 언어에 익숙하다면 *구조체는* 객체의 데이터 속성과 같습니다. 이 장에서는 튜플과 구조체를 비교해보고, 어떨때 구조체가 데이터를 그룹화하는데 더  좋은지 보여줍니다.

구조체를 정의하고 인스턴스화하는 방법을 보여줍니다. 구조체 유형과 관련된 동작을 지정하기 위해 관련 함수를 정의하는 방법에 대해 설명합니다. 이 함수를 특별히 *methods* 라고 합니다. 구조체와 열거형(enums)(6장에서 논의)은 Rust가 컴파일 할때 하는 유형 검사를 최대한 활용하기 위해 사용하는 것으로, 프로그램 도메인에서 새 유형을 생성하기 위한 빌딩 블록입니다.
