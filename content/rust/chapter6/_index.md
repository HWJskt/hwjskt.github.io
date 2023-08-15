+++
title = "Enums and Pattern Matching"
weight = 6
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

<!-- more -->

# [열거형 및 패턴 일치](**https://doc.rust-lang.org/book/ch06-00-enums.html#enums-and-pattern-matching**)

이 장에서는 enums이라고도 하는 열거형(enumerations) 에 대해 살펴보겠습니다. 열거형을 사용하면 가능한 변형(*variants*)을 열거하여 유형을 정의할 수 있습니다. 먼저 열거형을 정의하고 사용합니다. 그렇게 열거형이 데이터와 함께 의미를 인코딩하는 방법을 보여줍니다. 다음으로 `Option`이라는 특히 유용한 열거형을 살펴보겠습니다. `Option`은 값이 무언가가 될 수도 있고 아무것도 아닐 수 있음을 나타냅니다. 그런 다음 `match` 식을 이용한 패턴 매칭를 통해 열거형의 값이 달라짐에 따라 다른 코드를 실행하는 쉬운 방법을 살펴보겠습니다. 마지막으로 코드에서 열거형을 처리하는 데 사용할 수 있는 또 다른 편리하고 간결한 관용구인 `if let` 구문을 다룰 것입니다.

