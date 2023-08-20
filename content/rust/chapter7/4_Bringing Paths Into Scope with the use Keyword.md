+++
title = "Bringing Paths Into Scope with the use Keyword"
weight = 4
template ="book_page_rust.html"

+++

## 요약

- `use` 키워드 : "경로에 대한 바로 가기" 를 만든다
- 함수의 보통 사용하는 방법
  - `use`를 사용하여 함수의 상위 모듈을 범위로 가져온다.
  - -> `상위 모듈::함수()` 형태로 호출한다
  - 함수 전체경로를 범위로 가져온다음, `함수()` 이름만 써서 사용하면 로컬함수인지, 다른 모듈에서 가져온 함수인지 구분어렵다
- 구조체, 열거형 및 기타 항목의 보통 사용법
  - 전체 경로를 가져온다.
  - 예외 : 부모 모듈이 다른데 타입의 이름이 같다면, `부모모듈::타입` 형태로 구분해줘야. (`as`키워드로 타입의 이름을 바꿀수도 있다)
- 외부패키지 사용
  - Cargo.toml 에 패키지 이름 추가
  - `use 크레이트이::항목 ` 형식으로 사용
- 표준 라이브러리 :  `std` 
  - 외부 패키지인데, Cargo.toml 에 추가 안해도 된다.

## [`use` 키워드를 사용하여 경로를 범위로 가져오기](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#bringing-paths-into-scope-with-the-use-keyword)

함수를 호출하는 경로를 작성해야 하는 것은 불편하고 반복적으로 느껴질 수 있습니다. 목록 7-7에서 `add_to_waitlist` 함수에 대한 절대 경로를 선택했는지 상대 경로를 선택했는지에 관계없이 `add_to_waitlist`를 호출할 때마다 `front_of_house` 및 `hosting`도 지정해야 했습니다. 다행스럽게도 이 프로세스를 단순화하는 방법이 있습니다. `use` 키워드를 사용하여 "경로에 대한 바로 가기"를 한 번 만든 다음 범위의 다른 모든 곳에서 더 짧은 이름을 사용할 수 있습니다.

목록 7-11에서 `crate::front_of_house::hosting` 모듈을 `eat_at_restaurant` 함수의 범위로 가져오므로 `eat_at_restaurant`에서 `add_to_waitlist` 함수를 호출하려면 `hosting::add_to_waitlist`만 지정하면 됩니다. `.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

목록 7-11: `use`를 사용하여 모듈을 범위로 가져오기

범위에 `use`와 경로를 추가하는 것은 파일 시스템에서 심볼릭 링크를 만드는 것과 유사합니다. 크레이트 루트에 `use crate::front_of_house::hosting`을 추가함으로써 `hosting` 모듈이 크레이트 루트에 정의된 것처럼 이제 해당 범위에서 `hosting`이 유효한 이름이 됩니다. `use`으로 범위에 포함된 경로는 다른 경로와 마찬가지로 프라이버시도 확인합니다.

`use`은 `use`이 발생하는 특정 범위에 대한 바로 가기만 생성한다는 점에 유의하십시오. 목록 7-12는 `eat_at_restaurant` 함수를 `customer`라는 이름의 새 하위 모듈로 이동합니다. 이 모듈은 `use` 문과 다른 범위이므로 함수 본문이 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```

목록 7-12: `use` 문은 해당 범위 내에서만 적용됩니다.

컴파일러 오류는 바로 가기가 `customer` 모듈 내에서 더 이상 적용되지 않음을 보여줍니다.

```bash
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
warning: unused import: `crate::front_of_house::hosting`
 --> src/lib.rs:7:5
  |
7 | use crate::front_of_house::hosting;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

error[E0433]: failed to resolve: use of undeclared crate or module `hosting`
  --> src/lib.rs:11:9
   |
11 |         hosting::add_to_waitlist();
   |         ^^^^^^^ use of undeclared crate or module `hosting`

For more information about this error, try `rustc --explain E0433`.
warning: `restaurant` (lib) generated 1 warning
error: could not compile `restaurant` due to previous error; 1 warning emitted
```

`use`이 해당 범위에서 더 이상 사용되지 않는다는 경고도 있습니다! 이 문제를 해결하려면 `customer` 모듈 내에서 `use`를 이동하거나 하위 `customer` 모듈 내에서 `super::hosting`을 사용하여 상위 모듈의 바로 가기를 참조하십시오.

### [관용적인 `use` 경로 만들기](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths)

Listing 7-11에서 왜 우리가 `use crate::front_of_house::hosting`을 지정한 다음 `eat_at_restaurant` 함수에서 `hosting::add_to_waitlist`를 호출했는지 궁금할 수 있습니다. Listing 7-13과 같이  `use` 경로를 `add_to_waitlist`의 전체경로(crate::front_of_house::hosting::add_to_waitlist)로 지정하고, 함수 이름만 써서 사용 수도 있기때문입니다. 

파일 이름: src/lib.rs

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

목록 7-13: `use`를 사용하여 `add_to_waitlist` 함수를 범위로 가져오기

Listing 7-11과 7-13 모두 동일한 작업을 수행하지만 Listing 7-11은 `use`를 사용하여 함수를 범위로 가져오는 관용적인 방법입니다. `use`를 사용하여 함수의 상위 모듈을 범위로 가져오는 것은 함수를 호출할 때 상위 모듈을 지정해야 함을 의미합니다. 함수를 호출할 때 부모 모듈을 지정하면, 전체 경로의 반복을 최소화하면서 함수가 로컬로 정의되지 않았음을 분명히 할 수 있습니다. 목록 7-13의 코드는 `add_to_waitlist`가 정의된 위치가 명확하지 않습니다.

반면에 구조체, 열거형 및 기타 항목을 `use`로 가져올 때는 전체 경로를 지정하는 것이 관용적입니다. 목록 7-14는 표준 라이브러리의 `HashMap` 구조체를 바이너리 크레이트의 범위로 가져오는 관용적인 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

Listing 7-14: 관용적인 방식으로 `HashMap`을 범위로 가져오기

이 관용구 뒤에 강력한 이유가 있는 것은 아닙니다. 등장한 관례일 뿐이고 사람들은 이러한 방식으로 Rust 코드를 읽고 쓰는 데 익숙해졌습니다.

이런 관용구 상용법이 안되는 경우는, `use` 문을 사용하여 이름이 같은 두 항목을 범위로 가져 올 때입니다. 이런 경우는 Rust가 허용하지 않습니다. 목록 7-15는 이름은 같지만 상위 모듈이 다른 두 개의 `Result` 타입을 범위로 가져오는 방법과 이를 참조하는 방법을 보여줍니다.

파일 이름: src/lib.rs

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

목록 7-15: 이름이 같은 두 유형을 동일한 범위로 가져오려면 부모 모듈을 사용해야 합니다.

보시다시피 상위 모듈을 사용하면 두 가지 `Result` 타입이 구별됩니다. 대신에 우리가 `use std::fmt::Result`와 `use std::io::Result`를 지정했다면, 우리는 같은 범위에 두 개의 `Result` 유형을 가지게 될 것이고 러스트는 언제 우리가 의미하는 것이 무엇인지 알지 못할 것입니다. 

### [`as` 키워드로 새 이름 제공](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#providing-new-names-with-the-as-keyword)

`use`를 사용하여 동일한 이름의 두 타입을 동일한 범위로 가져오는 문제에 대한 또 다른 솔루션이 있습니다. 경로 뒤에 유형에 대해 `as` 및 새 로컬 이름 또는 *alias 를* 지정할 수 있습니다. Listing 7-16은 `as`를 사용하여 두 개의 `Result` 유형 중 하나의 이름을 변경하여 Listing 7-15의 코드를 작성하는 또 다른 방법을 보여줍니다.

파일 이름: src/lib.rs

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

Listing 7-16: `as` 키워드를 사용하여 범위로 가져올 때 유형 이름 바꾸기

두 번째 `use` 문에서 우리는 `std::io::Result` 유형에 대해 `IoResult`라는 새 이름을 선택했습니다. 이는 `std::fmt`의 `Result`와 충돌하지 않습니다. 또한 범위에 포함되었습니다. Listing 7-15와 Listing 7-16은 관용적인 것으로 간주되므로 선택은 여러분에게 달려 있습니다!

### [`pub use`로 이름을 다시 내보내기](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#re-exporting-names-with-pub-use)

`use` 키워드를 사용하여 이름을 범위로 가져오면, 새 범위에서 사용할 수 있는 이름은 비공개입니다. 코드 범위에 정의된 것처럼 해당 이름을 참조하도록 코드를 호출하는 코드를 활성화하려면 `pub`와 `use`를 결합할 수 있습니다. 이 기술을 *다시 내보내기*라고 부르는 이유는 항목을 범위로 가져오는 동시에 다른 사람이 해당 항목을 범위로 가져올 수 있도록 만들기 때문입니다.

Listing 7-17은 루트 모듈의 `use`가 `pub use`로 변경된 Listing 7-11의 코드를 보여줍니다.

파일 이름: src/lib.rs

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

목록 7-17: `pub use`를 사용하여 새 범위에서 사용할 모든 코드에 사용할 수 있는 이름 만들기

이 변경 전에는 외부 코드에서 `restaurant::front_of_house::hosting::add_to_waitlist()` 경로를 사용하여 `add_to_waitlist` 함수를 호출해야 했습니다. 이제 이 `pub use`가 루트 모듈에서 `hosting` 모듈을 다시 내보냈으므로, 이제 외부 코드에서 `restaurant::hosting::add_to_waitlist()` 경로를 대신 사용할 수 있습니다.

다시 내보내기는 코드의 내부 구조가, 코드를 호출하는 프로그래머가 도메인에 대해 생각하는 방식과 다를 때 유용합니다. 예를 들어, 이 식당 비유에서 식당을 운영하는 사람들은 `front of house`과 `back of house`를 생각합니다. 그러나 레스토랑을 방문하는 고객은 아마도 그러한 용어로 레스토랑의 부분에 대해 생각하지 않을 것입니다. `pub use`를 사용하면 하나의 구조로 코드를 작성할 수 있지만, 다른 구조를 노출할 수 있습니다. 이렇게 하면 라이브러리에서 작업하는 프로그래머와 라이브러리를 호출하는 프로그래머를 위해 라이브러리가 잘 정리됩니다. 14장의[ `pub use` 로 편리한 공용 API 내보내기](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use) 섹션 에서 `pub use`의 또 다른 예와 이것이 크레이트 문서에 미치는 영향을 살펴보겠습니다.

### [외부 패키지 사용](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#using-external-packages)

2장에서 추측 게임 프로젝트를 프로그래밍하면서, 우리는 임의의 숫자를 얻기 위해 `rand`라는 외부 패키지를 사용했습니다. 프로젝트에서 `rand`를 사용하기 위해 *Cargo.toml* 에 다음 행을 추가했습니다.

파일 이름: Cargo.toml

```toml
rand = `0.8.5`
```

*Cargo.toml* 에 `rand`를 종속 항목으로 추가하면 Cargo가 [crates.io](https://crates.io/) 에서 `rand` 패키지와 모든 종속 항목을 다운로드 하고 프로젝트에서 `rand`를 사용할 수 있도록 합니다.

그런 다음 `rand` 정의를 패키지 범위로 가져오기 위해, 크레이트 이름 `rand`로 시작하는 `use` 행을 추가하고, 범위로 가져오고자 하는 항목을 나열했습니다. [2장의 `난수 생성`](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number) 섹션 에서 `Rng` 특성을 범위로 가져오고 `rand::thread_rng` 함수를 호출했습니다.

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

Rust 커뮤니티의 구성원은 [crates.io](https://crates.io/) 에서 사용할 수 있는 많은 패키지를 만들었습니다. 우리 패키지에 외부 패키지를 가져오려면 *Cargo.toml* 파일에 외부 패키지를 나열하고, `use`를 사용하여 외부 패키지의 크레이트에서 항목을 범위로 가져옵니다. .

표준 `std` 라이브러리는 패키지 외부에 있는 크레이트이기도 합니다. 표준 라이브러리는 Rust 언어와 함께 제공되기 때문에 `std`를 포함하도록 *Cargo.toml을* 변경할 필요가 없습니다. 그러나 항목을 패키지 범위로 가져오려면 `use`으로 참조해야 합니다. 예를 들어 `HashMap`의 경우 다음 행을 사용합니다.

```rust
use std::collections::HashMap;
```

이것은 표준 라이브러리 크레이트의 이름인 `std`로 시작하는 절대 경로입니다.

### [중첩된 경로를 사용하여 큰 `use` 목록 정리](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#using-nested-paths-to-clean-up-large-use-lists)

동일한 크레이트 또는 동일한 모듈에 정의된 여러 항목을 사용하는 경우 각 항목을 한 줄에 나열하면 파일에서 세로 공간을 많이 차지할 수 있습니다. 예를 들어 목록 2-4의 추측 게임에 있는 두 개의 `use` 문은 `std`의 항목을 범위로 가져옵니다.

파일 이름: src/main.rs

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--
```

대신 중첩된 경로를 사용하여 동일한 항목을 한 줄의 범위로 가져올 수 있습니다. Listing 7-18과 같이 경로의 공통 부분을 지정하고 두 개의 콜론을 지정한 다음 서로 다른 경로 부분의 목록 주위에 중괄호를 지정하여 이를 수행합니다.

파일 이름: src/main.rs

```rust
// --snip--
use std::{cmp::Ordering, io};
// --snip--
```

Listing 7-18: 동일한 접두사를 가진 여러 항목을 범위로 가져오는 중첩 경로 지정

더 큰 프로그램에서 중첩된 경로를 사용하여 동일한 크레이트 또는 모듈에서 많은 항목을 범위로 가져오면, 필요한 별도의 `use` 문 수를 크게 줄일 수 있습니다!

경로의 모든 수준에서 중첩 경로를 사용할 수 있습니다. 이는 하위 경로를 공유하는 두 개의 `use` 문을 결합할 때 유용합니다. 예를 들어 Listing 7-19는 두 개의 `use` 문을 보여줍니다. 하나는 `std::io`를 범위로 가져오고 다른 하나는 `std::io::Write`를 범위로 가져옵니다.

파일 이름: src/lib.rs

```rust
use std::io;
use std::io::Write;
```

목록 7-19: 하나가 다른 하나의 하위 경로인 두 개의 `use` 문

이 두 경로의 공통 부분은 `std::io`이며 이것이 완전한 첫 번째 경로입니다. 이 두 경로를 하나의 `use` 문으로 병합하려면 Listing 7-20에 표시된 것처럼 중첩된 경로에서 `self`를 사용할 수 있습니다.

파일 이름: src/lib.rs

```rust
use std::io::{self, Write};
```

목록 7-20: 목록 7-19의 경로를 하나의 `use` 문으로 결합

이 줄은 `std::io` 및 `std::io::Write`를 범위로 가져옵니다.

### [Glob(*) 연산자](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#the-glob-operator)

경로에 정의된 *모든* 공용 항목을 범위로 가져오려면 해당 경로 뒤에 `*` glob 연산자를 지정할 수 있습니다.

```rust
use std::collections::*;
```

이 `use` 문은 `std::collections`에 정의된 모든 공용 항목을 현재 범위로 가져옵니다. glob 연산자를 사용할 때 주의하십시오! Glob은 범위 내에 어떤 이름이 있고 프로그램에서 사용된 이름이 정의된 위치를 구분하기 어렵게 만들 수 있습니다.

glob 연산자는 테스트 중인 모든 항목을 `tests` 모듈로 가져오기 위해 테스트할 때 자주 사용됩니다. [11장의 `테스트 작성 방법`](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#how-to-write-tests) 섹션 에서 이에 대해 이야기하겠습니다. glob 연산자는 때때로 prelude 패턴의 일부로 사용되기도 합니다. 해당 패턴에 대한 자세한 내용은 [표준 라이브러리 문서](https://doc.rust-lang.org/std/prelude/index.html#other-preludes) 를 참조하십시오.
