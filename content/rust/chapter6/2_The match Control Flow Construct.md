+++
title = "The match Control Flow Construct"
weight = 2
template ="book_page_rust.html"

+++



## 요약



**## [****`일치` 제어 흐름 구성****](**https://doc.rust-lang.org/book/ch06-02-match.html#the-match-control-flow-construct**)**

Rust에는 일련의 패턴과 값을 비교한 다음 어떤 패턴이 일치하는지에 따라 코드를 실행할 수 있는 `일치`라는 매우 강력한 제어 흐름 구조가 있습니다. 패턴은 리터럴 값, 변수 이름, 와일드카드 및 기타 여러 항목으로 구성될 수 있습니다. [18장에서는](https://doc.rust-lang.org/book/ch18-00-patterns.html) 모든 종류의 패턴과 그 기능을 다룹니다. `일치`의 힘은 패턴의 표현력과 컴파일러가 가능한 모든 사례가 처리되었음을 확인한다는 사실에서 비롯됩니다.

`일치` 표현을 동전 분류 기계와 같다고 생각하십시오. 동전은 다양한 크기의 구멍이 있는 트랙을 따라 미끄러져 내려오고 각 동전은 맞는 첫 번째 구멍을 통해 떨어집니다. 같은 방식으로 값은 `일치`의 각 패턴을 통과하고 첫 번째 패턴에서 값이 `적합`하고 실행 중에 사용되는 관련 코드 블록에 값이 떨어집니다.

동전이라고 하면 `일치`를 사용하여 예를 들어 보겠습니다! Listing 6-3과 같이 미지의 미국 동전을 가져와 계수기와 유사한 방식으로 어떤 동전인지 결정하고 그 값을 센트 단위로 반환하는 함수를 작성할 수 있습니다.

\```rust

enum Coin {

​    Penny,

​    Nickel,

​    Dime,

​    Quarter,

}

fn value_in_cents(coin: Coin) -> u8 {

​    match coin {

​        Coin::Penny => 1,

​        Coin::Nickel => 5,

​        Coin::Dime => 10,

​        Coin::Quarter => 25,

​    }

}

\```

목록 6-3: 열거형 및 열거형의 변형을 패턴으로 갖는 `일치` 식

`value_in_cents` 함수에서 `일치`를 분석해 보겠습니다. 먼저 `match` 키워드와 표현식을 나열합니다. 이 경우 값은 `coin`입니다. 이것은 `if`와 함께 사용되는 조건식과 매우 유사해 보이지만 큰 차이점이 있습니다. `if`를 사용하면 조건이 부울 값으로 평가되어야 하지만 여기서는 모든 유형이 될 수 있습니다. 이 예제에서 `coin`의 유형은 첫 번째 줄에서 정의한 `Coin` 열거형입니다.

다음은 `일치` 암입니다. 팔은 패턴과 일부 코드의 두 부분으로 구성됩니다. 여기에서 첫 번째 팔에는 `Coin::Penny` 값인 패턴이 있고 패턴과 실행할 코드를 구분하는 `=>` 연산자가 있습니다. 이 경우 코드는 값 `1`일 뿐입니다. 각 팔은 쉼표로 다음 팔과 구분됩니다.

`일치` 표현식이 실행되면 결과 값을 각 팔의 패턴과 순서대로 비교합니다. 패턴이 값과 일치하면 해당 패턴과 연결된 코드가 실행됩니다. 해당 패턴이 값과 일치하지 않으면 동전 분류기에서와 같이 실행이 다음 팔로 계속됩니다. 우리는 필요한 만큼 많은 팔을 가질 수 있습니다: Listing 6-3에서 `일치`에는 4개의 팔이 있습니다.

각 암과 연관된 코드는 식이며 일치하는 암에서 식의 결과 값은 전체 `일치` 식에 대해 반환되는 값입니다.

매치 암 코드가 짧으면 일반적으로 중괄호를 사용하지 않습니다. Listing 6-3에서 각 암은 값을 반환합니다. 매치 암에서 여러 줄의 코드를 실행하려면 중괄호를 사용해야 하며 암 다음에 오는 쉼표는 선택 사항입니다. 예를 들어 다음 코드는 `Lucky penny!`를 인쇄합니다. 메서드가 `Coin::Penny`로 호출될 때마다 블록의 마지막 값인 `1`을 반환합니다.

\```rust

fn value_in_cents(coin: Coin) -> u8 {

​    match coin {

​        Coin::Penny => {

​            println!(`Lucky penny!`);

​            1

​        }

​        Coin::Nickel => 5,

​        Coin::Dime => 10,

​        Coin::Quarter => 25,

​    }

}

\```

**### [****값에 바인딩되는 패턴****](**https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values**)**

일치 암의 또 다른 유용한 기능은 패턴과 일치하는 값 부분에 바인딩할 수 있다는 것입니다. 열거형 변형에서 값을 추출할 수 있는 방법입니다.

예를 들어 열거형 변형 중 하나를 변경하여 내부에 데이터를 보관하도록 하겠습니다. 1999년부터 2008년까지 미국은 한 면에 50개 주마다 다른 디자인의 쿼터를 주조했습니다. 다른 주화에는 주 디자인이 없으므로 분기에만 이 추가 가치가 있습니다. Listing 6-4에서 수행한 것처럼 내부에 저장된 `UsState` 값을 포함하도록 `Quarter` 변형을 변경하여 이 정보를 `enum`에 추가할 수 있습니다.

\```rust

\#[derive(Debug)] // so we can inspect the state in a minute

enum UsState {

​    Alabama,

​    Alaska,

​    // --snip--

}

enum Coin {

​    Penny,

​    Nickel,

​    Dime,

​    Quarter(UsState),

}

\```

목록 6-4: `Quarter` 변형이 `UsState` 값도 포함하는 `Coin` 열거형

친구가 50개의 주 쿼터를 모두 수집하려고 한다고 상상해 봅시다. 동전 유형별로 느슨한 거스름돈을 정렬하는 동안 각 분기와 관련된 주 이름을 불러서 친구가 가지고 있지 않은 것이 있으면 컬렉션에 추가할 수 있습니다.

이 코드의 일치 표현식에서 변형 `Coin::Quarter`의 값과 일치하는 패턴에 `state`라는 변수를 추가합니다. `Coin::Quarter`가 일치하면 `state` 변수는 해당 분기의 상태 값에 바인딩됩니다. 그런 다음 해당 팔의 코드에서 다음과 같이 `상태`를 사용할 수 있습니다.

\```rust

fn value_in_cents(coin: Coin) -> u8 {

​    match coin {

​        Coin::Penny => 1,

​        Coin::Nickel => 5,

​        Coin::Dime => 10,

​        Coin::Quarter(state) => {

​            println!(`State quarter from {:?}!`, state);

​            25

​        }

​    }

}

\```

`value_in_cents(Coin::Quarter(UsState::Alaska))`를 호출하면 `coin`은 `Coin::Quarter(UsState::Alaska)`가 됩니다. 해당 값을 각 일치 부문과 비교할 때 `Coin::Quarter(state)`에 도달할 때까지 일치하는 항목이 없습니다. 이 시점에서 `state`에 대한 바인딩은 `UsState::Alaska` 값이 됩니다. 그런 다음 `println!`에서 해당 바인딩을 사용할 수 있습니다. `Quarter`에 대한 `Coin` 열거형 변형에서 내부 상태 값을 가져옵니다.

**### [****`옵션`과 일치****](**https://doc.rust-lang.org/book/ch06-02-match.html#matching-with-optiont**)**

이전 섹션에서 `Option`을 사용할 때 `Some` 케이스에서 내부 `T` 값을 가져오고 싶었습니다.`; `옵션을 처리할 수도 있습니다.` `Coin` 열거형과 마찬가지로 `match`를 사용합니다! 동전을 비교하는 대신 `Option의 변형을 비교하겠습니다.`이지만 `일치` 표현식이 작동하는 방식은 동일하게 유지됩니다.

`옵션`을 받는 함수를 작성하고 싶다고 가정해 보겠습니다.` 내부에 값이 있으면 해당 값에 1을 더합니다. 내부에 값이 없으면 함수는 `없음` 값을 반환하고 어떤 작업도 수행하지 않아야 합니다.

이 함수는 `match` 덕분에 작성하기가 매우 쉽고 Listing 6-5와 같습니다.

\```rust

​    fn plus_one(x: Option<i32>) -> Option<i32> {

​        match x {

​            None => None,

​            Some(i) => Some(i + 1),

​        }

​    }

​    let five = Some(5);

​    let six = plus_one(five);

​    let none = plus_one(None);

\```

목록 6-5: `옵션`에 `일치` 표현식을 사용하는 함수`

`plus_one`의 첫 번째 실행을 자세히 살펴보겠습니다. `plus_one(five)`를 호출하면 `plus_one` 본문의 변수 `x`는 `Some(5)` 값을 갖게 됩니다. 그런 다음 이를 각 매치 부문과 비교합니다.

\```rust

​            None => None,

\```

`Some(5)` 값은 패턴 `None`과 일치하지 않으므로 다음 단계로 계속 진행합니다.

\```rust

​            Some(i) => Some(i + 1),

\```

`Some(5)`는 `Some(i)`와 일치합니까? 그렇습니다! 동일한 변형이 있습니다. `i`는 `Some`에 포함된 값에 바인딩되므로 `i`는 값 `5`를 사용합니다. 그런 다음 매치 암의 코드가 실행되므로 `i` 값에 1을 더하고 총 `6`이 포함된 새로운 `Some` 값을 생성합니다.

이제 Listing 6-5에서 `x`가 `None`인 `plus_one`의 두 번째 호출을 살펴보겠습니다. `일치`를 입력하고 첫 번째 팔과 비교합니다.

\```rust

​            None => None,

\```

일치합니다! 추가할 값이 없으므로 프로그램이 중지되고 `=>` 오른쪽에 `None` 값이 반환됩니다. 첫 번째 팔이 일치했기 때문에 다른 팔은 비교되지 않습니다.

`일치`와 열거형을 결합하면 많은 상황에서 유용합니다. Rust 코드에서 이 패턴을 많이 볼 수 있습니다. 열거형과 `일치`하고 변수를 내부 데이터에 바인딩한 다음 이를 기반으로 코드를 실행합니다. 처음에는 조금 까다롭지만 익숙해지면 모든 언어로 제공되기를 바랄 것입니다. 지속적으로 사용자가 선호하는 제품입니다.

**### [****일치는 철저합니다****](**https://doc.rust-lang.org/book/ch06-02-match.html#matches-are-exhaustive**)**

우리가 논의해야 할 `일치`의 또 다른 측면이 있습니다. 팔의 패턴은 모든 가능성을 커버해야 합니다. 버그가 있고 컴파일되지 않는 `plus_one` 함수의 이 버전을 고려하십시오.

\```rust

​    fn plus_one(x: Option<i32>) -> Option<i32> {

​        match x {

​            Some(i) => Some(i + 1),

​        }

​    }

\```

우리는 `없음` 사례를 처리하지 않았으므로 이 코드는 버그를 일으킬 것입니다. 운 좋게도 Rust가 잡는 방법을 알고 있는 버그입니다. 이 코드를 컴파일하려고 하면 다음 오류가 발생합니다.

\```bash

$ cargo run

   Compiling enums v0.1.0 (file:///projects/enums)

error[E0004]: non-exhaustive patterns: `None` not covered

 --> src/main.rs:3:15

  |

3 |         match x {

  |               ^ pattern `None` not covered

  |

note: `Option<i32>` defined here

 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1

  |

  = note: 

/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered

  = note: the matched value is of type `Option<i32>`

help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown

  |

4 ~             Some(i) => Some(i + 1),

5 ~             None => todo!(),

  |

For more information about this error, try `rustc --explain E0004`.

error: could not compile `enums` due to previous error

\```

Rust는 우리가 모든 가능한 경우를 다루지 않았다는 것과 우리가 어떤 패턴을 잊어버렸는지 알고 있습니다! Rust의 일치는 **철저** 합니다. 코드가 유효하려면 마지막 가능성을 모두 소진해야 합니다. 특히 `옵션`의 경우`, Rust는 우리가 `없음` 사례를 명시적으로 처리하는 것을 잊는 것을 방지할 때 null이 있을 수 있는 값을 가지고 있다고 가정하는 것을 방지하여 앞에서 논의한 수십억 달러의 실수를 불가능하게 만듭니다.

**### [****범용 패턴 및 `_` 자리 표시자****](**https://doc.rust-lang.org/book/ch06-02-match.html#catch-all-patterns-and-the-_-placeholder**)**

열거형을 사용하면 몇 가지 특정 값에 대해 특별한 작업을 수행할 수도 있지만 다른 모든 값에 대해서는 하나의 기본 작업을 수행합니다. 주사위 굴림에서 3을 굴리면 플레이어가 움직이지 않고 대신 새 멋진 모자를 받는 게임을 구현한다고 상상해 보십시오. 7이 나오면 플레이어는 멋진 모자를 잃습니다. 다른 모든 값의 경우 플레이어는 게임 보드에서 해당 수의 공간을 이동합니다. 다음은 임의의 값이 아닌 하드코딩된 주사위 굴림의 결과와 본문이 없는 함수로 표현되는 다른 모든 논리를 사용하여 논리를 구현하는 `일치`입니다. 왜냐하면 실제로 구현하는 것은 이 예제의 범위를 벗어나기 때문입니다.

\```rust

​    let dice_roll = 9;

​    match dice_roll {

​        3 => add_fancy_hat(),

​        7 => remove_fancy_hat(),

​        other => move_player(other),

​    }

​    fn add_fancy_hat() {}

​    fn remove_fancy_hat() {}

​    fn move_player(num_spaces: u8) {}

\```

처음 두 팔의 경우 패턴은 리터럴 값 `3` 및 `7`입니다. 다른 모든 가능한 값을 포함하는 마지막 팔의 경우 패턴은 `기타`라는 이름을 지정하기 위해 선택한 변수입니다. `기타` 암에 대해 실행되는 코드는 변수를 `move_player` 함수에 전달하여 사용합니다.

이 코드는 `u8`이 가질 수 있는 모든 가능한 값을 나열하지 않았지만 마지막 패턴이 구체적으로 나열되지 않은 모든 값과 일치하기 때문에 컴파일됩니다. 이 범용 패턴은 `일치`가 철저해야 한다는 요구 사항을 충족합니다. 패턴이 순서대로 평가되기 때문에 포괄적인 팔을 마지막에 두어야 합니다. catch-all arm을 더 일찍 넣으면 다른 arm은 실행되지 않으므로, catch-all 후에 arm을 추가하면 Rust가 경고합니다!

Rust는 또한 포괄적인 것을 원하지만 포괄적인 패턴의 값을 **사용하고** 싶지 않을 때 사용할 수 있는 패턴이 있습니다. `_`는 모든 값과 일치하고 해당 값에 바인딩되지 않는 특수 패턴입니다. 이는 Rust에게 우리가 그 값을 사용하지 않을 것임을 알려주므로 Rust는 사용하지 않는 변수에 대해 경고하지 않습니다.

게임의 규칙을 변경해 보겠습니다. 이제 3이나 7이 아닌 다른 것을 굴리면 다시 굴려야 합니다. 더 이상 범용 값을 사용할 필요가 없으므로 `other`라는 변수 대신 `_`를 사용하도록 코드를 변경할 수 있습니다.

\```rust

​    let dice_roll = 9;

​    match dice_roll {

​        3 => add_fancy_hat(),

​        7 => remove_fancy_hat(),

​        _ => reroll(),

​    }

​    fn add_fancy_hat() {}

​    fn remove_fancy_hat() {}

​    fn reroll() {}

\```

이 예제는 또한 마지막 팔의 다른 모든 값을 명시적으로 무시하기 때문에 완전성 요구 사항을 충족합니다. 우리는 아무것도 잊지 않았습니다.

마지막으로 게임의 규칙을 한 번 더 변경하여 3이나 7 이외의 것을 굴려도 자신의 차례에 아무 일도 일어나지 않도록 하겠습니다. [`튜플 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션 에서 ) `_` 팔과 함께 가는 코드:

\```rust

​    let dice_roll = 9;

​    match dice_roll {

​        3 => add_fancy_hat(),

​        7 => remove_fancy_hat(),

​        _ => (),

​    }

​    fn add_fancy_hat() {}

​    fn remove_fancy_hat() {}

\```

여기에서 우리는 명시적으로 이전 버전의 패턴과 일치하지 않는 다른 값을 사용하지 않을 것이며 이 경우 어떤 코드도 실행하고 싶지 않다고 명시적으로 말하고 있습니다.

[패턴과 일치에 대한 자세한 내용은 18장](https://doc.rust-lang.org/book/ch18-00-patterns.html) 에서 다룰 것입니다. 지금은 `일치` 표현이 다소 장황한 상황에서 유용할 수 있는 `if let` 구문으로 넘어갈 것입니다.
