+++
title = "Separating Modules into Different Files"
weight = 5
template ="book_page_rust.html"

+++

## 요약

## [모듈을 각각의 파일들로 분리](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#separating-modules-into-different-files)

지금까지 이 장의 모든 예제는 하나의 파일 안에 여러 모듈을 정의했습니다. 모듈이 커지면 해당 정의를 별도의 파일로 이동하여 코드를 더 쉽게 탐색할 수 있습니다.

Filename: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Listing 7-17: `pub use`을 통해, 새 범위의 모든 코드에서 사용가능한 이름 만들기

예를 들어, 여러 식당 모듈이 있는 Listing 7-17의 코드부터 시작해 봅시다. 크레이트 루트 파일에 모든 모듈을 정의하는 대신, 모듈을 파일들로 추출겠습니다. 이 경우 크레이트 루트 파일은 *src/lib.rs* 이지만 이 절차는 크레이트 루트 파일이 *src/main.rs* 인 바이너리 크레이트에서도 작동합니다.

먼저 `front_of_house` 모듈을 자체 파일로 추출합니다. `front_of_house` 모듈의 중괄호 안에 있는 코드를 제거하고 `mod front_of_house;`만 남깁니다. 그래서 *src/lib.rs는* 목록 7-21에 표시된 코드를 포함합니다. *목록 7-22 과 같은  src/front_of_house.rs* 파일을 생성하기 전까지 컴파일은 되지 않습니다.

파일 이름: src/lib.rs

```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

*Listing 7-21: 본문이 src/front_of_house.rs* 으로 옮겨진 `front_of_house` 모듈 선언

다음으로 목록 7-22에 표시된 것처럼 중괄호 안에 있던 코드를 *src/front_of_house.rs 라는 새 파일에 배치합니다.* 컴파일러는 `front_of_house`라는 이름의 크레이트 루트에서 모듈 선언을 발견했기 때문에 이 파일을 살펴봐야 한다는 것을 압니다.

파일 이름: src/front_of_house.rs

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

*목록 7-22: src/front_of_house.rs* 의 `front_of_house` 모듈 내부 정의

모듈 트리에서 *한 번만* `mod` 선언을 사용하여 파일을 로드하면 됩니다. 컴파일러가 파일이 프로젝트의 일부임을 알게 되면(그리고 모듈 트리에서 `mod` 문을 넣은 위치로 인해 코드가 상주하는 위치를 알게 되면) 프로젝트의 다른 파일은 다음을 사용하여 로드된 파일의 코드를 참조해야 합니다. [`모듈 트리에서 항목을 참조하기 위한 경로`](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) 섹션 에서 설명한 대로 선언된 위치의 경로입니다. 즉, `mod`는 다른 프로그래밍 언어에서 볼 수 있는 `include` 작업이 *아닙니다.*

다음으로 `hosting` 모듈을 자체 파일로 추출해보겠습니다. `hosting`은 루트 모듈이 아닌 `front_of_house`의 하위 모듈이기 때문에 프로세스가 약간 다릅니다. `hosting`을 위한 파일을 모듈 트리의 조상 이름을 따서 명명될 새 디렉토리(이 경우 *src/front_of_house/ )* 에 배치합니다.

`hosting` 이동을 시작하려면 `hosting` 모듈의 선언만 포함하도록 *src/front_of_house.rs를* 변경합니다.

파일 이름: src/front_of_house.rs

```rust
pub mod hosting;
```

그런 다음 `hosting` 모듈에서 만든 정의를 포함하기 위해 *src/front_of_house* 디렉터리와 *hosting.rs* 파일을 만듭니다.

파일 이름: src/front_of_house/hosting.rs

```rust
pub fn add_to_waitlist() {}
```

이렇게 하지 않고, *hosting.rs* 를 *src* 디렉토리 에 넣으면 컴파일러는 *hosting.rs* 코드가 `front_of_house` 모듈의 자식으로 선언되지 않고 크레이트 루트에 선언된 `hosting` 모듈에 있을 것으로 예상합니다. 어떤 모듈의 코드를 검사할 파일에 대한 컴파일러의 규칙은 디렉토리와 파일이 모듈 트리와 더 밀접하게 일치함을 의미합니다.

> ### [파일 경로를 지정하는 다른 방법](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#alternate-file-paths)
>
> 지금까지 Rust 컴파일러가 사용하는 가장 관용적인 파일 경로를 다루었지만 Rust는 이전 스타일의 파일 경로도 지원합니다. 크레이트 루트에 선언된 `front_of_house`라는 모듈의 경우 컴파일러는 다음에서 모듈의 코드를 찾습니다.
>
> - *src/front_of_house.rs* (우리가 다룬 것)
> - *src/front_of_house/mod.rs* (이전 스타일, 여전히 지원되는 경로)
>
> `front_of_house`의 하위 모듈인 `hosting`이라는 모듈의 경우 컴파일러는 다음에서 모듈의 코드를 찾습니다.
>
> - *src/front_of_house/hosting.rs* (우리가 다룬 것)
> - *src/front_of_house/hosting/mod.rs* (이전 스타일, 계속 지원되는 경로)
>
> 동일한 모듈에 대해 두 스타일을 모두 사용하면 컴파일러 오류가 발생합니다. 동일한 프로젝트의 서로 다른 모듈에 대해 두 가지 스타일을 혼합하여 사용할 수 있지만 프로젝트를 탐색하는 사람들에게 혼동을 줄 수 있습니다.
>
> *mod.rs* 라는 이름의 파일을 사용하는 스타일의 주요 단점은 프로젝트가 *mod.rs* 라는 이름의 많은 파일로 끝날 수 있다는 것입니다. 이는 편집기에서 파일을 동시에 열 때 혼동될 수 있습니다.

각 모듈의 코드를 별도의 파일로 옮겼으며 모듈 트리는 그대로 유지됩니다. `eat_at_restaurant`의 함수 호출은 정의가 다른 파일에 있더라도 수정하지 않고 작동합니다. 이 기술을 사용하면 모듈의 크기가 커짐에 따라 모듈을 새 파일로 이동할 수 있습니다.

*src/lib.rs* 의 `pub use crate::front_of_house::hosting` 문도 변경되지 않았으며 `use`는 크레이트의 일부로 컴파일되는 파일에 영향을 미치지 않습니다. `mod` 키워드는 모듈을 선언하고,  Rust는 해당 모듈에 들어가는 코드의 모듈과 동일한 이름을 가진 파일을 찾습니다.

## [요약](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#summary)

Rust를 사용하면 패키지를 여러 크레이트로 분할하고, 크레이트를 모듈로 분할하여 한 모듈에 정의된 항목을 다른 모듈에서 참조할 수 있습니다. 절대 또는 상대 경로를 지정하여 이를 수행할 수 있습니다. 이러한 경로는 `use` 문을 사용하여 범위로 가져올 수 있으므로, 해당 범위에서 항목을 여러 번 사용하는 경우 더 짧은 경로를 사용할 수 있습니다. 모듈 코드는 기본적으로 비공개이지만 `pub` 키워드를 추가하여 정의를 공개할 수 있습니다.

다음 장에서는 깔끔하게 정리된 코드에서 사용할 수 있는 표준 라이브러리의 일부 컬렉션 데이터 구조를 살펴보겠습니다.
