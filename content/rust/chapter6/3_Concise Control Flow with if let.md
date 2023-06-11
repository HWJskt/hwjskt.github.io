+++
title = "Concise Control Flow with if let.md"
weight = 3
template ="book_page_rust.html"

+++

## 요약



## [`if let`을 사용한 간결한 제어 흐름](https://doc.rust-lang.org/book/ch06-03-if-let.html#concise-control-flow-with-if-let)

`if let` 구문을 사용하면 `if`와 `let`을 덜 장황한 방식으로 결합하여 나머지 패턴은 무시하면서 하나의 패턴과 일치하는 값을 처리할 수 있습니다. Listing 6-6에서 `옵션`에 일치하는 프로그램을 고려하십시오.` `config_max` 변수의 값이지만 값이 `Some` 변형인 경우에만 코드를 실행하려고 합니다.

```rust
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!(`The maximum is configured to be {}`, max),
        _ => (),
    }
```

목록 6-6: 값이 `Some`인 경우에만 코드 실행에 관심이 있는 `일치`

값이 `Some`이면 패턴의 변수 `max`에 값을 바인딩하여 `Some` 변형의 값을 출력합니다. 우리는 `None` 값으로 아무 것도 하고 싶지 않습니다. `일치` 표현을 만족시키려면 하나의 변형만 처리한 후 `_ => ()`를 추가해야 하는데, 이는 추가하기 귀찮은 상용구 코드입니다.

대신 `if let`을 사용하여 더 짧게 작성할 수 있습니다. 다음 코드는 목록 6-6의 `일치`와 동일하게 작동합니다.

```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!(`The maximum is configured to be {}`, max);
    }
```

`if let` 구문은 패턴과 등호로 구분된 표현식을 사용합니다. 이것은 `일치`와 같은 방식으로 작동하며, `일치`에 표현이 주어지고 패턴이 첫 번째 암입니다. 이 경우 패턴은 `Some(max)`이고 `max`는 `Some` 내부의 값에 바인딩됩니다. 그런 다음 해당 `match` 팔에서 `max`를 사용한 것과 같은 방식으로 `if let` 블록의 본문에서 `max`를 사용할 수 있습니다. 값이 패턴과 일치하지 않으면 `if let` 블록의 코드가 실행되지 않습니다.

`if let`을 사용하면 타이핑, 들여쓰기, 상용구 코드가 줄어듭니다. 그러나 `일치`가 시행하는 철저한 검사를 잃게 됩니다. `match`와 `if let` 중에서 선택하는 것은 특정 상황에서 수행하는 작업과 간결함을 얻는 것이 철저한 확인을 잃는 것에 대한 적절한 절충안인지 여부에 따라 다릅니다.

즉, 값이 하나의 패턴과 일치할 때 코드를 실행한 다음 다른 모든 값을 무시하는 `일치`에 대한 구문 설탕으로 `if let`을 생각할 수 있습니다.

`if let`과 함께 `else`를 포함할 수 있습니다. `else`와 함께 사용되는 코드 블록은 `if let` 및 `else`와 동일한 `match` 표현식에서 `_` 케이스와 함께 사용되는 코드 블록과 동일합니다. Listing 6-4에서 `Quarter` 변형이 `UsState` 값을 포함하는 `Coin` 열거형 정의를 상기하십시오. 분기 상태를 알리는 동시에 우리가 보는 모든 비 분기 동전을 세고 싶다면 다음과 같이 `일치` 표현식을 사용하여 계산할 수 있습니다.

```rust
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!(`State quarter from {:?}!`, state),
        _ => count += 1,
    }
```

또는 다음과 같이 `if let` 및 `else` 표현식을 사용할 수 있습니다.

```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!(`State quarter from {:?}!`, state);
    } else {
        count += 1;
    }
```

프로그램에 `일치`를 사용하여 표현하기에는 너무 장황한 논리가 있는 상황이 있는 경우 `if let`도 Rust 도구 상자에 있음을 기억하십시오.

## [요약](https://doc.rust-lang.org/book/ch06-03-if-let.html#summary)

이제 열거형을 사용하여 열거된 값 세트 중 하나가 될 수 있는 사용자 정의 유형을 만드는 방법을 다루었습니다. 우리는 표준 라이브러리의 `옵션` type은 유형 시스템을 사용하여 오류를 방지하는 데 도움이 됩니다. enum 값에 데이터가 있는 경우 처리해야 하는 사례 수에 따라 `match` 또는 `if let`을 사용하여 해당 값을 추출하고 사용할 수 있습니다.

Rust 프로그램은 이제 구조체와 열거형을 사용하여 도메인의 개념을 표현할 수 있습니다. API에서 사용할 사용자 정의 유형을 생성하면 유형 안전성이 보장됩니다. 컴파일러는 특정 함수가 각 함수가 기대하는 유형의 값만 얻도록 합니다.

사용자에게 사용하기 쉽고 사용자에게 필요한 것만 정확하게 노출하는 잘 구성된 API를 사용자에게 제공하기 위해 이제 Rust의 모듈을 살펴보겠습니다.
