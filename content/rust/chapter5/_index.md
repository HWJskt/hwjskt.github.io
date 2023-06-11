+++
title = "Using Structs to Structure Related Data"
weight = 5
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

<!-- more -->

# [구조체를 사용하여 관련 데이터 구조화](https://doc.rust-lang.org/book/ch05-00-structs.html#using-structs-to-structure-related-data)

struct 또는 *structure 는 함께 패키징하고 의미 있는 그룹을 구성하는 여러 관련 값의* *이름* 을 지정할 수 있는 사용자 정의 데이터 유형입니다. 객체 지향 언어에 익숙하다면 *구조체는* 객체의 데이터 속성과 같습니다. 이 장에서는 튜플과 구조체를 비교 및 대조하여 이미 알고 있는 내용을 기반으로 구축하고 구조체가 데이터를 그룹화하는 더 좋은 방법인 경우를 보여줍니다.

구조체를 정의하고 인스턴스화하는 방법을 보여줍니다. 구조체 유형과 관련된 동작을 지정하기 위해 관련 함수, 특히 *methods* 라는 관련 함수를 정의하는 방법에 대해 설명합니다. 구조체와 열거형(6장에서 논의)은 Rust의 컴파일 시간 유형 검사를 최대한 활용하기 위해 프로그램 도메인에서 새 유형을 생성하기 위한 빌딩 블록입니다.
