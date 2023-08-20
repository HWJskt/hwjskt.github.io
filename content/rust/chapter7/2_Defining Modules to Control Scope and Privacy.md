+++
title = "Defining Modules to Control Scope and Privacy"
weight = 2
template ="book_page_rust.html"

+++



## 요약

- 모듈
  - 크레이트 루트 파일에서 선언 (`mod`)
  - 모듈 파일에서 하위 모듈을 선언할 수 있다.
  - 모듈 선언 : priviate 는  `mod` , public 은 `pub mod` 이용
  - 모듈 내부에 structs, enums, constants, traits, functions 정의가능

- 경로(paths)
  - :: 를 이용해서 접근한다.
  - `use` 키워드로 short cut 을 만들수도 있다.
- 크레이트 루트
  - *src/main.rs* 와 *src/lib.rs* 파일을 지칭
  - *크레이트의 모듈 구조(=모듈트리)의 루트에서 `크레이트`라는 모듈을 형성

## [범위(scope) 및 프라이버시(Privacy)를 제어하기 위한 모듈 정의](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#defining-modules-to-control-scope-and-privacy)

이 섹션에서는 모듈과 모듈 시스템의 다른 부분, 즉 항목 이름을 지정할 수 있는 *경로(paths)* 에 대해 설명합니다. 경로를 범위로 가져오는 `use` 키워드; 항목을 공개하기 위한 `pub` 키워드. 또한 `as` 키워드, 외부 패키지 및 glob 연산자에 대해서도 설명합니다.

먼저 나중에 코드를 구성할 때 쉽게 참조할 수 있도록 규칙 목록부터 시작하겠습니다. 그러면 각 규칙에 대해 자세히 설명하겠습니다.

### [모듈 치트 시트](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#modules-cheat-sheet)

여기에서는 모듈, 경로, `use` 키워드 및 `pub` 키워드가 컴파일러에서 작동하는 방법과 대부분의 개발자가 코드를 구성하는 방법에 대한 빠른 참조를 제공합니다. 우리는 이 장 전체에서 이러한 각 규칙의 예를 살펴보겠지만, 모듈이 작동하는 방식을 먼저 간략히 살펴보겠습니다.

- **크레이트 루트에서 시작합니다** : 크레이트를 컴파일할 때 컴파일러는 먼저 크레이트 루트 파일(일반적으로 라이브러리 크레이트의 경우 *src/lib.rs* 또는 바이너리 크레이트의 경우 *src/main.rs* )에서 컴파일할 코드를 찾습니다.

- **모듈 선언하는 방법**

  : 크레이트 루트 파일에서 새 모듈을 선언할 수 있습니다. 예를 들어 `garden` 모듈을 다음과 같이 선언합니다.

  ```rust
  mod garden;
  ```

  컴파일러는 다음 위치에서 모듈의 코드를 찾습니다.

  - 인라인에서, `mod garden` 다음의, 세미콜론을 대체하는 중괄호 내
  - *src/garden.rs* 파일에서
  - *src/garden/mod.rs* 파일에서

- **하위 모듈 선언하는 방법**

  : 크레이트 루트 이외의 모든 파일에서 하위 모듈을 선언할 수 있습니다. 예를 들어 *src/garden.rs*에서 *mod vegetables* 를 선언할 수 있습니다.

  ```rust
  mod vegetables;
  ```

  컴파일러는 다음 위치에서 상위 모듈의 이름이 지정된 디렉토리 내에서 하위 모듈의 코드를 찾습니다.

  - 인라인,  `mod vegetables` 바로 뒤에, 세미콜론 대신 중괄호 안에 
- *src/garden/vegetables.rs* 파일에서
  - *src/garden/vegetables/mod.rs* 파일에서

- **모듈 코드로 접근하는 경로** 

  : 모듈이 크레이트의 일부가 되면, privacy rules이 허용하는 한, 동일한 크레이트 내에서 해당 모듈의 코드를 참조할 수 있습니다.  이 때 코드로 접근하는 경로를 사용합니다. 예를 들어 garden vegetables 모듈의 `Asparagus` 타입은 `crate::garden::vegetables::Asparagus`에서 찾을 수 있습니다.

- **비공개(private) 대 공개(public)** 

  : 모듈 내의 코드는 기본적으로 상위 모듈에서 비공개입니다. 모듈을 공개하려면 `mod` 대신 `pub mod`로 모듈을 선언하십시오. 공용 모듈 내의 항목도 공용으로 만들려면 선언 전에 `pub`를 사용하십시오.

- **`use` 키워드** : 같은 범위 내에서 `use` 키워드는 긴 경로의 반복을 줄여주는 바로 가기를 만듭니다. `crate::garden::vegetables::Asparagus`를 참조할 수 있는 모든 범위에서 `use crate::garden::vegetables::Asparagus;`로 바로 가기를 만듭니다. 그런 다음 범위에서 해당 타입을 사용하려면 `Asparagus`만 작성하면 됩니다.

여기에서 이러한 규칙을 설명하는 `backyard`라는 이름의 바이너리 크레이트를 만듭니다. `backyard`라고도 하는 크레이트의 디렉토리에는 다음 파일과 디렉토리가 포함되어 있습니다.

```
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

이 경우 크레이트 루트 파일은 *src/main.rs* 이며 다음을 포함합니다:

파일 이름: src/main.rs

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!(`I'm growing {:?}!`, plant);
}
```

`pub mod garden;` 줄은 *src/garden.rs* 에서 찾은 코드를 포함하도록 컴파일러에 지시합니다.

파일명: src/garden.rs

```rust
pub mod vegetables;
```

여기 `pub mod vegetables;` *src/garden/vegetables.rs* 의 코드 도 포함되어 있음을 의미합니다. 해당 코드는 다음과 같습니다.

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

이제 이러한 규칙에 대해 자세히 알아보고 실제로 시연해 보겠습니다.

### [모듈로 관련 코드를 그룹화](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#grouping-related-code-in-modules)

*모듈*을 사용하면 크레이트 내에 있는 코드의 가독성을 좋게하고, 손쉽게 재사용하도록 구성할 수 있습니다. 모듈은 또한 모듈 내의 코드가 기본적으로 비공개이기 때문에 항목의 *프라이버시(privacy)*를 제어할 수 있습니다. 개인 항목(private item)은 외부에서 사용할 수 없는 내부 구현 세부 정보입니다. 우리는 모듈과 그 안에 있는 항목을 공개하도록 선택할 수 있습니다. 이렇게 하면 외부 코드가 모듈을 사용하고 의존할 수 있도록 노출됩니다.

예를 들어 레스토랑의 기능을 제공하는 라이브러리 크레이트를 작성해 보겠습니다. 우리는 함수의 시그니쳐를 정의하지만, 식당 구현보다는 코드 구성에 집중하기 위해 본문을 비워 둡니다.

레스토랑 업계에서 레스토랑의 일부는 *프론트 오브 하우스(front of house)* , 다른 일부는 *백 오브 하우스(back of house)* 라고 합니다. 프론트 오브 하우스는 고객이 있는 곳입니다. 여기에는 호스트가 고객을 앉히고 서버가 주문 및 지불을 받고 바텐더가 음료를 만드는 곳이 포함됩니다. 백오브하우스는 셰프와 요리사가 주방에서 일하고, 식기 세척기가 청소하고, 관리자가 관리 업무를 수행하는 곳입니다.

이러한 방식으로 크레이트를 구성하기 위해 함수를 중첩된 모듈로 구성할 수 있습니다. `cargo new restaurant --lib`를 실행하여 `restaurant`라는 새 라이브러리를 만듭니다. 그런 다음 목록 7-1의 코드를 *src/lib.rs* 에 입력하여 일부 모듈 및 함수 서명을 정의합니다. 집 앞 부분은 다음과 같습니다.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

목록 7-1: 함수를 포함하는 다른 모듈을 포함하는 `front_of_house` 모듈

`mod` 키워드 뒤에 모듈 이름(이 경우 `front_of_house`)을 사용하여 모듈을 정의합니다. 그러면 모듈 본문이 중괄호 안에 들어갑니다. `hosting` 및 `serving` 모듈처럼 모듈 내부에  다른 모듈을 배치할 수 있습니다. 모듈은 또한 구조체(structs), 열거형(enums), 상수(constants), 특성(traits) 및 목록 7-1에서와 같이 함수(function)와 같은 다른 항목에 대한 정의를 보유할 수 있습니다.

모듈을 사용하여, 관련 정의를 함께 그룹화하고 관련된 이유를 명명할 수 있습니다. 이 코드를 사용하는 프로그래머는 모든 정의를 읽을 필요 없이 그룹을 기반으로 코드를 탐색할 수 있으므로 관련된 정의를 더 쉽게 찾을 수 있습니다. 이 코드에 새로운 기능을 추가하는 프로그래머는 프로그램을 체계적으로 유지하기 위해 코드를 어디에 배치해야 하는지 알 것입니다.

앞에서 우리는 *src/main.rs* 와 *src/lib.rs를* 크레이트 루트라고 부른다고 언급했습니다. 크레이트 루트라는 이름이 붙은 이유는, 이 두 파일 중 하나의 내용이, *모듈 트리*로 알려진 크레이트 모듈 구조의 루트에서 `크레이트`라는 모듈을 형성하기 때문입니다.

목록 7-2는 목록 7-1의 구조에 대한 모듈 트리를 보여줍니다.

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

목록 7-2: 목록 7-1의 코드에 대한 모듈 트리

이 트리는 일부 모듈이 서로 중첩되는 방식을 보여줍니다. 예를 들어 `hosting`은 `front_of_house` 내부에 중첩됩니다. 트리는 또한 일부 모듈이 서로 *형제* 임을 보여줍니다. 즉, 동일한 모듈에서 정의된다는 의미입니다. `hosting` 및 `serving`은 `front_of_house` 내에 정의된 형제입니다. 모듈 A가 모듈 B 안에 포함되어 있으면 모듈 A는 모듈 B의 *자식 이고 모듈 B는 모듈 A의* *부모* 라고 말합니다. 전체 모듈 트리는 `crate`라는 암시적 모듈 아래에 뿌리를 두고 있습니다.

모듈 트리는 컴퓨터에 있는 파일 시스템의 디렉토리 트리를 상기시킬 수 있습니다. 이것은 매우 적절한 비교입니다! 파일 시스템의 디렉토리와 마찬가지로 모듈을 사용하여 코드를 구성합니다. 그리고 디렉토리의 파일과 마찬가지로 모듈을 찾을 방법이 필요합니다.
