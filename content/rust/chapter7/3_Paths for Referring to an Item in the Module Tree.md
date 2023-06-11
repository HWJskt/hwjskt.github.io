+++
title = "Paths for Referring to an Item in the Module Tree"
weight = 3
template ="book_page_rust.html"

+++

## 요약



## [모듈 트리에서 항목을 참조하기 위한 경로](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#paths-for-referring-to-an-item-in-the-module-tree)

Rust가 모듈 트리에서 항목을 찾을 위치를 보여주기 위해 파일 시스템을 탐색할 때 경로를 사용하는 것과 같은 방식으로 경로를 사용합니다. 함수를 호출하려면 경로를 알아야 합니다.

경로는 두 가지 형식을 취할 수 있습니다.

- 절대 *경로는* 크레이트 루트에서 시작하는 전체 경로입니다. 외부 크레이트의 코드의 경우 절대 경로는 크레이트 이름으로 시작하고 현재 크레이트의 코드의 경우 리터럴 `크레이트`로 시작합니다.
- 상대 *경로는* 현재 모듈에서 시작하여 `self`, `super` 또는 현재 모듈의 식별자를 사용합니다.

절대 경로와 상대 경로 모두 이중 콜론(`::`)으로 구분된 하나 이상의 식별자 뒤에 옵니다.

목록 7-1로 돌아가서 `add_to_waitlist` 함수를 호출하고 싶다고 합시다. 이것은 `add_to_waitlist` 함수의 경로가 무엇인지 묻는 것과 같습니다. 목록 7-3에는 일부 모듈과 함수가 제거된 목록 7-1이 포함되어 있습니다.

크레이트 루트에 정의된 새 함수 `eat_at_restaurant`에서 `add_to_waitlist` 함수를 호출하는 두 가지 방법을 보여줍니다. 이러한 경로는 정확하지만 이 예제를 있는 그대로 컴파일하지 못하게 하는 또 다른 문제가 남아 있습니다. 잠시 후에 그 이유를 설명하겠습니다.

`eat_at_restaurant` 함수는 라이브러리 크레이트의 공개 API의 일부이므로 `pub` 키워드로 표시합니다. [`pub` 키워드를 사용하여 경로 노출`](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword) 섹션 에서 `pub`에 대해 자세히 설명합니다.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

목록 7-3: 절대 경로와 상대 경로를 사용하여 `add_to_waitlist` 함수 호출

`eat_at_restaurant`에서 `add_to_waitlist` 함수를 처음 호출할 때 절대 경로를 사용합니다. `add_to_waitlist` 기능은 `eat_at_restaurant`와 같은 크레이트에 정의되어 있습니다. 즉, `crate` 키워드를 사용하여 절대 경로를 시작할 수 있습니다. 그런 다음 `add_to_waitlist`로 이동할 때까지 각 연속 모듈을 포함합니다. 동일한 구조를 가진 파일 시스템을 상상할 수 있습니다. `add_to_waitlist` 프로그램을 실행하기 위해 `/front_of_house/hosting/add_to_waitlist` 경로를 지정합니다. 크레이트 루트에서 시작하기 위해 `크레이트` 이름을 사용하는 것은 쉘의 파일 시스템 루트에서 시작하기 위해 `/`를 사용하는 것과 같습니다.

두 번째로 `eat_at_restaurant`에서 `add_to_waitlist`를 호출할 때는 상대 경로를 사용합니다. 경로는 `eat_at_restaurant`와 동일한 수준의 모듈 트리에 정의된 모듈 이름인 `front_of_house`로 시작합니다. 여기서 동등한 파일 시스템은 `front_of_house/hosting/add_to_waitlist` 경로를 사용합니다. 모듈 이름으로 시작하는 것은 경로가 상대적임을 의미합니다.

상대 경로를 사용할지 절대 경로를 사용할지 선택하는 것은 프로젝트를 기반으로 결정하며 항목 정의 코드를 항목을 사용하는 코드와 별도로 또는 함께 이동할 가능성이 더 높은지에 따라 달라집니다. 예를 들어 `front_of_house` 모듈과 `eat_at_restaurant` 함수를 `customer_experience`라는 모듈로 이동하면 절대 경로를 `add_to_waitlist`로 업데이트해야 하지만 상대 경로는 여전히 유효합니다. 그러나 `eat_at_restaurant` 함수를 별도로 `dining`이라는 모듈로 이동하면 `add_to_waitlist` 호출에 대한 절대 경로는 동일하게 유지되지만 상대 경로는 업데이트해야 합니다.

목록 7-3을 컴파일하고 왜 아직 컴파일되지 않는지 알아봅시다! 우리가 얻은 오류는 Listing 7-4에 나와 있습니다.

```bash
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
  |
note: the module `hosting` is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

Listing 7-4: Listing 7-3의 코드를 빌드할 때 발생하는 컴파일러 오류

오류 메시지는 모듈 `호스팅`이 비공개라고 말합니다. 다시 말해, 우리는 `hosting` 모듈과 `add_to_waitlist` 함수에 대한 올바른 경로를 가지고 있지만 Rust는 개인 섹션에 대한 액세스 권한이 없기 때문에 우리가 사용하도록 허용하지 않습니다. Rust에서 모든 항목(함수, 메서드, 구조체, 열거형, 모듈 및 상수)은 기본적으로 부모 모듈에 대해 비공개입니다. 함수나 구조체와 같은 항목을 비공개로 만들고 싶다면 모듈에 넣습니다.

상위 모듈의 항목은 하위 모듈 내의 개인 항목을 사용할 수 없지만 하위 모듈의 항목은 상위 모듈의 항목을 사용할 수 있습니다. 하위 모듈은 구현 세부 정보를 래핑하고 숨기지만 하위 모듈은 정의된 컨텍스트를 볼 수 있기 때문입니다. 우리의 은유를 계속하기 위해 프라이버시 규칙을 레스토랑의 백오피스와 같다고 생각하십시오. 그곳에서 일어나는 일은 레스토랑 고객에게만 공개되지만 사무실 관리자는 자신이 운영하는 레스토랑에서 모든 것을 보고 할 수 있습니다.

Rust는 내부 구현 세부 사항을 숨기는 것이 기본값이 되도록 모듈 시스템 기능을 이런 방식으로 선택했습니다. 이렇게 하면 외부 코드를 손상시키지 않고 변경할 수 있는 내부 코드 부분을 알 수 있습니다. 그러나 Rust는 항목을 공개하기 위해 `pub` 키워드를 사용하여 자식 모듈 코드의 내부 부분을 외부 조상 모듈에 노출하는 옵션을 제공합니다.

### [`pub` 키워드로 경로 노출](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword)

Listing 7-4에서 `호스팅` 모듈이 비공개라는 오류로 돌아가 봅시다. 부모 모듈의 `eat_at_restaurant` 함수가 자식 모듈의 `add_to_waitlist` 함수에 액세스하기를 원하므로 목록 7-5에 표시된 것처럼 `pub` 키워드로 `hosting` 모듈을 표시합니다.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Listing 7-5: `eat_at_restaurant`에서 사용하기 위해 `hosting` 모듈을 `pub`으로 선언

안타깝게도 Listing 7-5의 코드는 Listing 7-6과 같이 여전히 오류를 발생시킵니다.

```bash
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: function `add_to_waitlist` is private
 --> src/lib.rs:9:37
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                                     ^^^^^^^^^^^^^^^ private function
  |
note: the function `add_to_waitlist` is defined here
 --> src/lib.rs:3:9
  |
3 |         fn add_to_waitlist() {}
  |         ^^^^^^^^^^^^^^^^^^^^

error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
   |
note: the function `add_to_waitlist` is defined here
  --> src/lib.rs:3:9
   |
3  |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

Listing 7-6: Listing 7-5의 코드를 빌드할 때 발생하는 컴파일러 오류

무슨 일이에요? `mod hosting` 앞에 `pub` 키워드를 추가하면 모듈이 공개됩니다. 이 변경으로 `front_of_house`에 액세스할 수 있으면 `hosting`에 액세스할 수 있습니다. 그러나 `호스팅`의 *내용은 여전히 비공개입니다.* 모듈을 공개한다고 해서 내용이 공개되는 것은 아닙니다. 모듈의 `pub` 키워드는 상위 모듈의 코드가 내부 코드에 액세스하지 않고 참조만 허용합니다. 모듈은 컨테이너이기 때문에 모듈을 공개하는 것만으로는 할 수 있는 일이 많지 않습니다. 더 나아가 모듈 내의 항목 중 하나 이상을 공개하도록 선택해야 합니다.

목록 7-6의 오류는 `add_to_waitlist` 함수가 비공개라고 말합니다. 개인 정보 보호 규칙은 구조체, 열거형, 함수 및 메서드와 모듈에 적용됩니다.

목록 7-7에서와 같이 정의 앞에 `pub` 키워드를 추가하여 `add_to_waitlist` 함수를 공용으로 만들어 보겠습니다.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

목록 7-7: `mod hosting` 및 `fn add_to_waitlist`에 `pub` 키워드를 추가하면 `eat_at_restaurant`에서 함수를 호출할 수 있습니다.

이제 코드가 컴파일됩니다! `pub` 키워드를 추가하면 개인 정보 보호 규칙과 관련하여 `add_to_waitlist`에서 이러한 경로를 사용할 수 있는 이유를 알아보기 위해 절대 경로와 상대 경로를 살펴보겠습니다.

절대 경로에서 크레이트 모듈 트리의 루트인 `크레이트`로 시작합니다. `front_of_house` 모듈은 크레이트 루트에 정의되어 있습니다. `front_of_house`는 공개되지 않지만 `eat_at_restaurant` 함수는 `front_of_house`와 동일한 모듈에서 정의되기 때문에(즉, `eat_at_restaurant`와 `front_of_house`는 형제임) `eat_at_restaurant에서 `front_of_house`를 참조할 수 있습니다. `. 다음은 `pub`으로 표시된 `hosting` 모듈입니다. `hosting`의 상위 모듈에 액세스할 수 있으므로 `hosting`에 액세스할 수 있습니다. 마지막으로, `add_to_waitlist` 함수는 `pub`로 표시되고 부모 모듈에 액세스할 수 있으므로 이 함수 호출이 작동합니다!

상대 경로에서 논리는 첫 번째 단계를 제외하고는 절대 경로와 동일합니다. 경로는 크레이트 루트에서 시작하지 않고 `front_of_house`에서 시작합니다. `front_of_house` 모듈은 `eat_at_restaurant`와 동일한 모듈 내에서 정의되므로 `eat_at_restaurant`가 정의된 모듈에서 시작하는 상대 경로가 작동합니다. 그런 다음 `hosting` 및 `add_to_waitlist`가 `pub`로 표시되기 때문에 나머지 경로가 작동하고 이 함수 호출이 유효합니다!

다른 프로젝트에서 코드를 사용할 수 있도록 라이브러리 크레이트를 공유할 계획이라면 공개 API는 크레이트 사용자가 코드와 상호 작용할 수 있는 방법을 결정하는 계약입니다. 사람들이 당신의 크레이트에 더 쉽게 의존할 수 있도록 공개 API에 대한 변경 사항을 관리하는 것과 관련하여 많은 고려 사항이 있습니다. 이러한 고려 사항은 이 책의 범위를 벗어납니다. 이 주제에 관심이 있다면 [Rust API 가이드라인 을](https://rust-lang.github.io/api-guidelines/) 참조하세요 .

> #### [바이너리 및 라이브러리가 포함된 패키지에 대한 모범 사례](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#best-practices-for-packages-with-a-binary-and-a-library)
>
> 우리는 패키지가 *src/main.rs* 바이너리 크레이트 루트와 *src/lib.rs* 라이브러리 크레이트 루트를 모두 포함할 수 있으며 두 크레이트 모두 기본적으로 패키지 이름을 가질 것이라고 언급했습니다. 일반적으로 라이브러리와 바이너리 크레이트를 모두 포함하는 이 패턴을 가진 패키지는 바이너리 크레이트에 라이브러리 크레이트로 코드를 호출하는 실행 파일을 시작하기에 충분한 코드만 있습니다. 이렇게 하면 라이브러리 크레이트의 코드를 공유할 수 있으므로 다른 프로젝트에서 패키지가 제공하는 대부분의 기능을 활용할 수 있습니다.
>
> 모듈 트리는 *src/lib.rs* 에 정의되어야 합니다. 그런 다음 패키지 이름으로 경로를 시작하여 바이너리 크레이트에서 공용 항목을 사용할 수 있습니다. 바이너리 크레이트는 완전히 외부에 있는 크레이트가 라이브러리 크레이트를 사용하는 것처럼 라이브러리 크레이트의 사용자가 됩니다. 공용 API만 사용할 수 있습니다. 이는 좋은 API를 설계하는 데 도움이 됩니다. 당신은 저자일 뿐만 아니라 고객이기도 합니다!
>
> [12장](https://doc.rust-lang.org/book/ch12-00-an-io-project.html) 에서는 바이너리 크레이트와 라이브러리 크레이트를 모두 포함하는 명령줄 프로그램을 사용하여 이 조직적 관행을 시연할 것입니다.

### [`super`로 상대 경로 시작](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#starting-relative-paths-with-super)

경로 시작 부분에 `super`를 사용하여 현재 모듈이나 크레이트 루트가 아닌 상위 모듈에서 시작하는 상대 경로를 구성할 수 있습니다. 이것은 `..` 구문으로 파일 시스템 경로를 시작하는 것과 같습니다. `super`를 사용하면 부모 모듈에 있는 것으로 알고 있는 항목을 참조할 수 있으므로 모듈이 부모와 밀접하게 관련되어 있을 때 모듈 트리를 쉽게 재정렬할 수 있지만 부모는 언젠가 모듈 트리의 다른 곳으로 이동할 수 있습니다.

셰프가 잘못된 주문을 수정하고 직접 고객에게 가져오는 상황을 모델링하는 Listing 7-8의 코드를 고려하십시오. `back_of_house` 모듈에 정의된 `fix_incorrect_order` 함수는 `super`로 시작하는 `deliver_order`에 대한 경로를 지정하여 상위 모듈에 정의된 `deliver_order` 함수를 호출합니다.

파일 이름: src/lib.rs

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

목록 7-8: `super`로 시작하는 상대 경로를 사용하여 함수 호출

`fix_incorrect_order` 함수는 `back_of_house` 모듈에 있으므로 `super`를 사용하여 `back_of_house`의 상위 모듈로 이동할 수 있습니다. 이 경우 루트인 `crate`입니다. 거기에서 `deliver_order`를 찾아 찾습니다. 성공! 우리는 `back_of_house` 모듈과 `deliver_order` 기능이 서로 같은 관계를 유지하고 상자의 모듈 트리를 재구성하기로 결정하면 함께 이동할 가능성이 있다고 생각합니다. 따라서 우리는 `super`를 사용하여 이 코드가 다른 모듈로 이동될 경우 향후에 코드를 업데이트할 위치가 줄어들게 됩니다.

### [구조체와 열거형을 공개하기](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#making-structs-and-enums-public)

또한 `pub`을 사용하여 구조체 및 열거형을 공용으로 지정할 수 있지만 구조체 및 열거형과 함께 `pub`를 사용하는 데 추가로 몇 가지 세부 정보가 있습니다. 구조체 정의 전에 `pub`를 사용하면 구조체를 공개하지만 구조체의 필드는 여전히 비공개입니다. 사례별로 각 필드를 공개하거나 공개하지 않을 수 있습니다. 목록 7-9에서 공개 `toast` 필드와 비공개 `seasonal_fruit` 필드가 있는 공개 `back_of_house::Breakfast` 구조체를 정의했습니다. 이것은 고객이 식사와 함께 제공되는 빵의 종류를 선택할 수 있는 레스토랑의 경우를 모델로 하지만 요리사는 계절과 재고에 따라 식사와 함께 제공되는 과일을 결정합니다. 사용 가능한 과일은 빠르게 변경되므로 고객은 과일을 선택할 수 없으며 어떤 과일을 얻을지 알 수도 없습니다.

파일 이름: src/lib.rs

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from(`peaches`),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer(`Rye`);
    // Change our mind about what bread we'd like
    meal.toast = String::from(`Wheat`);
    println!(`I'd like {} toast please`, meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from(`blueberries`);
}
```

Listing 7-9: 일부 공개 필드와 일부 비공개 필드가 있는 구조체

`back_of_house::Breakfast` 구조체의 `toast` 필드가 공개되어 있기 때문에 `eat_at_restaurant`에서 점 표기법을 사용하여 `toast` 필드에 쓰고 읽을 수 있습니다. `seasonal_fruit`가 비공개이기 때문에 `eat_at_restaurant`에서 `seasonal_fruit` 필드를 사용할 수 없습니다. `seasonal_fruit` 필드 값을 수정하는 줄의 주석을 제거하여 어떤 오류가 발생하는지 확인하십시오!

또한 `back_of_house::Breakfast`에는 전용 필드가 있기 때문에 구조체는 `Breakfast`(여기서는 `summer`라고 함)의 인스턴스를 구성하는 공용 관련 함수를 제공해야 합니다. `Breakfast`에 이러한 기능이 없으면 `eat_at_restaurant`에서 비공개 `seasonal_fruit` 필드의 값을 설정할 수 없기 때문에 `eat_at_restaurant`에서 `Breakfast` 인스턴스를 생성할 수 없습니다.

반대로 열거형을 공개하면 모든 변형이 공개됩니다. Listing 7-10과 같이 `enum` 키워드 앞에 `pub`만 있으면 됩니다.

파일 이름: src/lib.rs

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

목록 7-10: 열거형을 공개로 지정하면 모든 변형이 공개됩니다.

`Appetizer` 열거형을 공개했기 때문에 `eat_at_restaurant`에서 `Soup` 및 `Salad` 변형을 사용할 수 있습니다.

열거형은 해당 변형이 공개되지 않는 한 그다지 유용하지 않습니다. 모든 경우에 `pub`로 모든 열거형 변형에 주석을 달아야 하는 것은 성가신 일이므로 열거형 변형의 기본값은 공개입니다. 구조체는 필드가 공개되지 않아도 유용한 경우가 많으므로 구조체 필드는 `pub`로 주석이 지정되지 않는 한 기본적으로 모든 것이 비공개라는 일반적인 규칙을 따릅니다.

우리가 다루지 않은 `pub`와 관련된 또 다른 상황이 있습니다. 그것은 우리의 마지막 모듈 시스템 기능인 `use` 키워드입니다. 먼저 `use` 자체를 다룬 다음 `pub`와 `use`를 결합하는 방법을 보여줍니다.
