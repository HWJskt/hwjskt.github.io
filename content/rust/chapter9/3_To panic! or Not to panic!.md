+++
title = "To panic! or Not to panic!"
weight = 3
template ="book_page_rust.html"

+++

## 요약

## [에 `panic!`또는 하지`panic!`](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic)

`panic!`그러면 언제 전화를 걸어야 하고 언제 돌아와야 하는지 어떻게 결정합니까 `Result`? 코드 패닉이 발생하면 복구할 방법이 없습니다. `panic!`복구할 수 있는 방법이 있는지 여부에 관계없이 모든 오류 상황을 호출할 수 있지만 호출 코드를 대신하여 상황을 복구할 수 없다는 결정을 내리는 것입니다. 값 을 반환하도록 선택하면 `Result`호출 코드 옵션을 제공합니다. 호출 코드는 해당 상황에 적합한 방식으로 복구를 시도하도록 선택하거나 `Err`이 경우 값을 복구할 수 없다고 결정할 수 있으므로 `panic!`복구 가능한 오류를 호출하여 복구할 수 없는 오류로 전환할 수 있습니다. 따라서 `Result`실패할 수 있는 함수를 정의할 때 반환은 좋은 기본 선택입니다.

예제, 프로토타입 코드, 테스트와 같은 상황에서는 `Result`. 그 이유를 살펴보고 컴파일러는 실패가 불가능하다고 말할 수 없지만 사람은 할 수 있는 상황에 대해 논의해 봅시다. 이 장에서는 라이브러리 코드에서 패닉이 발생하는지 여부를 결정하는 방법에 대한 몇 가지 일반적인 지침으로 결론을 내릴 것입니다.

### [예제, 프로토타입 코드 및 테스트](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#examples-prototype-code-and-tests)

일부 개념을 설명하기 위해 예제를 작성할 때 강력한 오류 처리 코드를 포함하면 예제가 덜 명확해질 수 있습니다. 예제에서 패닉이 발생할 수 있는 것과 같은 메서드에 대한 호출은 `unwrap`애플리케이션에서 오류를 처리하기를 원하는 방식에 대한 자리 표시자로 의미되며 나머지 코드가 수행하는 작업에 따라 다를 수 있습니다.

마찬가지로 `unwrap`및 `expect`메서드는 오류 처리 방법을 결정하기 전에 프로토타이핑할 때 매우 편리합니다. 프로그램을 보다 강력하게 만들 준비가 되었을 때 코드에 명확한 마커를 남깁니다.

테스트에서 메서드 호출이 실패하면 해당 메서드가 테스트 중인 기능이 아니더라도 전체 테스트가 실패하기를 원할 것입니다. `panic!`테스트가 실패로 표시되는 방식이기 때문에 호출 `unwrap`또는 `expect`정확히 발생해야 하는 일입니다.

### [컴파일러보다 더 많은 정보를 가지고 있는 경우](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler)

에 값 이 있는지 확인하는 다른 논리가 있거나 `unwrap`호출 하는 것이 적절할 수도 있지만 그 논리는 컴파일러가 이해하는 것이 아닙니다. 여전히 처리해야 하는 값이 있습니다. 특정 상황에서 논리적으로 불가능하더라도 호출하는 작업은 여전히 일반적으로 실패할 가능성이 있습니다. 코드를 수동으로 검사하여 변형이 없을 것임을 확인할 수 있는 경우 를 호출하는 것이 완벽하게 허용되며 텍스트 에 변형이 없을 것이라고 생각하는 이유를 문서화하는 것이 더 좋습니다. 예를 들면 다음과 같습니다.`expect``Result``Ok``Result``Err``unwrap``Err``expect`

```rust
    use std::net::IpAddr;

    let home: IpAddr = `127.0.0.1`
        .parse()
        .expect(`Hardcoded IP address should be valid`);
```

`IpAddr`하드 코딩된 문자열을 구문 분석하여 인스턴스를 생성하고 있습니다. 이것이 유효한 IP 주소임을 알 수 있으므로 여기에서 `127.0.0.1`사용할 수 있습니다 `expect`. 그러나 하드코딩된 유효한 문자열이 있다고 해서 메서드의 반환 유형이 변경되지는 않습니다 `parse`. 우리는 여전히 값을 얻고 컴파일러는 여전히 충분히 똑똑하지 않기 때문에 변형이 가능한 것처럼 `Result`처리하도록 합니다. 이 문자열이 항상 유효한 IP 주소인지 확인하십시오. IP 주소 문자열이 프로그램에 하드코딩되지 않고 사용자로부터 온 것이므로 오류가 발생할 가능성이 *있는* 경우 대신 보다 강력한 방식으로 처리해야 합니다. 이 IP 주소가 하드코딩되어 있다는 가정을 언급하면 변경하라는 메시지가 표시됩니다.`Result``Err``Result``expect`나중에 더 나은 오류 처리 코드를 위해 다른 소스에서 대신 IP 주소를 가져와야 합니다.

### [오류 처리 지침](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling)

코드가 잘못된 상태가 될 가능성이 있는 경우 코드를 패닉 상태로 만드는 것이 좋습니다. 이 컨텍스트에서 *잘못된 상태* 는 유효하지 않은 값, 모순된 값 또는 누락된 값이 코드에 전달되고 다음 중 하나 이상이 발생하는 경우와 같이 일부 가정, 보증, 계약 또는 불변성이 위반된 경우입니다.

- 잘못된 상태는 사용자가 잘못된 형식으로 데이터를 입력하는 것과 같이 가끔 발생할 수 있는 상황이 아니라 예상치 못한 상태입니다.
- 이 시점 이후의 코드는 모든 단계에서 문제를 확인하기보다는 이 잘못된 상태에 있지 않은지에 의존해야 합니다.
- 사용하는 유형에 이 정보를 인코딩하는 좋은 방법이 없습니다. 17장의 [`유형으로서의 인코딩 상태 및 동작`](https://doc.rust-lang.org/book/ch17-03-oo-design-patterns.html#encoding-states-and-behavior-as-types) 섹션 에서 우리가 의미하는 바의 예를 통해 작업할 것입니다.

누군가가 코드를 호출하고 이치에 맞지 않는 값을 전달하는 경우 라이브러리 사용자가 원하는 작업을 결정할 수 있도록 가능한 경우 오류를 반환하는 것이 가장 좋습니다. 그러나 계속하는 것이 안전하지 않거나 해로울 수 있는 경우 최선의 선택은 `panic!`라이브러리를 사용하는 사람에게 코드의 버그를 호출하여 경고하여 개발 중에 수정할 수 있도록 하는 것입니다. 마찬가지로 `panic!`제어할 수 없는 외부 코드를 호출하고 고칠 방법이 없는 잘못된 상태를 반환하는 경우에 종종 적합합니다.

`Result`그러나 실패가 예상되는 경우 전화를 거는 것보다 a를 반환하는 것이 더 적절합니다 `panic!`. 예를 들어 잘못된 형식의 데이터가 제공되는 파서 또는 속도 제한에 도달했음을 나타내는 상태를 반환하는 HTTP 요청이 있습니다. 이러한 경우 a를 반환하는 것은 `Result`실패가 호출 코드에서 처리 방법을 결정해야 하는 예상 가능성을 나타냅니다.

코드가 유효하지 않은 값을 사용하여 호출되면 사용자를 위험에 빠뜨릴 수 있는 작업을 수행하는 경우 코드는 먼저 값이 유효한지 확인하고 값이 유효하지 않으면 당황해야 합니다. 이는 주로 안전상의 이유 때문입니다. 유효하지 않은 데이터에 대해 작업을 시도하면 코드가 취약성에 노출될 수 있습니다. `panic!`이것이 범위를 벗어난 메모리 액세스를 시도하는 경우 표준 라이브러리가 호출하는 주된 이유입니다. 현재 데이터 구조에 속하지 않는 메모리에 액세스하려는 시도는 일반적인 보안 문제입니다. 함수에는 종종 *계약이 있습니다.*: 입력이 특정 요구 사항을 충족하는 경우에만 해당 동작이 보장됩니다. 계약 위반 시 당황하는 것은 계약 위반이 항상 호출자 측 버그를 나타내며 호출 코드에서 명시적으로 처리해야 하는 일종의 오류가 아니기 때문에 의미가 있습니다. 사실 복구할 코드를 호출하는 합리적인 방법은 없습니다. 호출 *프로그래머는* 코드를 수정해야 합니다. 기능에 대한 계약, 특히 위반으로 인해 패닉이 발생하는 경우 기능에 대한 API 문서에 설명되어야 합니다.

그러나 모든 함수에서 많은 오류 검사를 수행하면 장황하고 성가실 수 있습니다. 다행스럽게도 Rust의 유형 시스템(따라서 컴파일러가 수행하는 유형 검사)을 사용하여 많은 검사를 수행할 수 있습니다. 함수에 매개 변수로 특정 유형이 있는 경우 컴파일러에서 유효한 값이 있음을 이미 확인했음을 알고 코드의 논리를 계속 진행할 수 있습니다. 예를 들어, 가 아닌 유형이 있는 경우 `Option`프로그램은 아무것도 아닌 것보다 있는 *것을* *기대* 합니다. 그러면 코드에서 `Some`및 에 대한 두 가지 사례를 처리할 필요가 없습니다.`None`변종: 확실히 값을 갖는 경우는 한 가지뿐입니다. 함수에 아무것도 전달하지 않으려는 코드는 컴파일조차 되지 않으므로 함수는 런타임에 해당 사례를 확인할 필요가 없습니다. 또 다른 예는 매개변수가 절대 음수가 되지 않도록 하는 와 같은 부호 없는 정수 유형을 사용하는 것입니다 `u32`.

### [유효성 검사를 위한 사용자 정의 유형 만들기](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation)

Rust의 유형 시스템을 사용하여 한 단계 더 나아가 유효한 값이 있는지 확인하고 유효성 검사를 위한 사용자 지정 유형을 만드는 방법을 살펴보겠습니다. 2장에서 우리 코드가 사용자에게 1에서 100 사이의 숫자를 추측하도록 요청한 추측 게임을 기억하십시오. 우리는 비밀 번호와 비교하여 확인하기 전에 사용자의 추측이 해당 숫자 사이에 있는지 확인하지 않았습니다. 추측이 긍정적이라는 것만 확인했습니다. 이 경우 결과는 그다지 심각하지 않았습니다. `너무 높음` 또는 `너무 낮음`의 결과는 여전히 정확합니다. 그러나 사용자가 유효한 추측으로 안내하고 사용자가 범위를 벗어난 숫자를 추측할 때와 예를 들어 대신 문자를 입력할 때 서로 다른 동작을 갖도록 하는 것은 유용한 개선 사항이 될 것입니다.

이를 수행하는 한 가지 방법은 잠재적으로 음수를 허용하기 위해 추측을 a `i32`가 아닌 an으로 구문 분석 `u32`한 다음 다음과 같이 범위 내에 있는 숫자에 대한 검사를 추가하는 것입니다.

```rust
    loop {
        // --snip--

        let guess: i32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        if guess < 1 || guess > 100 {
            println!(`The secret number will be between 1 and 100.`);
            continue;
        }

        match guess.cmp(&secret_number) {
            // --snip--
    }
```

이 `if`표현식은 값이 범위를 벗어나는지 확인하고 사용자에게 문제에 대해 알리고 `continue`루프의 다음 반복을 시작하고 다른 추측을 요청하도록 호출합니다. 표현식 후에 1과 100 사이에 있는 비밀 번호를 `if`비교하여 진행할 수 있습니다.`guess``guess`

그러나 이것은 이상적인 해결책이 아닙니다. 프로그램이 1에서 100 사이의 값에서만 작동하는 것이 절대적으로 중요하고 이 요구 사항을 가진 많은 함수가 있는 경우 모든 함수에서 이와 같은 검사를 수행하는 것은 지루할 것입니다. 성능).

대신 새 유형을 만들고 모든 곳에서 유효성 검사를 반복하는 대신 유형의 인스턴스를 생성하는 함수에 유효성 검사를 넣을 수 있습니다. 이렇게 하면 함수가 서명에 새 유형을 사용하고 수신한 값을 자신 있게 사용할 수 있습니다. 목록 9-13은 함수가 1에서 100 사이의 값을 받는 경우 `Guess`에만 인스턴스를 생성하는 유형을 정의하는 한 가지 방법을 보여줍니다.`Guess``new`

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!(`Guess value must be between 1 and 100, got {}.`, value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

목록 9-13: `Guess`1에서 100 사이의 값으로만 계속되는 유형

먼저, 를 보유하는 `Guess`named 필드가 있는 named 구조체를 정의합니다. 여기에 번호가 저장됩니다.`value``i32`

그런 다음 값 의 인스턴스를 생성하는 `new`on 이라는 관련 함수를 구현합니다. 이 함수는 유형이라는 이름의 매개변수 하나를 갖고 a를 반환하도록 정의됩니다. 함수 본문의 코드는 1과 100 사이인지 확인하기 위해 테스트합니다. 이 테스트를 통과하지 못하면 호출 코드를 작성하는 프로그래머에게 필요한 버그가 있음을 알리는 호출을 합니다. 이 범위 를 벗어나는 를 생성하면 의존하는 계약을 위반하게 되므로 수정합니다. 조건`Guess``Guess``new``value``i32``Guess``new``value``value``panic!``Guess``value``Guess::new``Guess::new`공개 API 문서에서 패닉에 대해 논의해야 할 수도 있습니다. `panic!`14장에서 만든 API 문서에서 a의 가능성을 나타내는 문서 규칙을 다룰 것입니다 `value`. 테스트를 통과하면 필드가 매개변수로 설정된 새 `Guess`항목 을 `value`만들고 .`value``Guess`

```
value`다음으로 를 빌리고 `self`다른 매개변수가 없으며 를 반환하는 메서드를 구현합니다 `i32`. *이러한 종류의 메서드는 getter* 라고도 하는데 , 그 목적이 해당 필드에서 일부 데이터를 가져와서 반환하는 것이기 때문입니다. `value`구조체 의 필드 가 비공개 이기 때문에 이 공개 메서드가 필요합니다 `Guess`. `value`필드가 비공개여서 `Guess`구조체를 사용하는 코드가 직접 설정할 수 없도록 하는 것이 중요합니다 `value`. 모듈 외부의 코드는 함수를 사용하여 의 인스턴스를 생성 *해야* 하므로 a가 확인하지 않은 a를 가질 수 있는 방법이 없습니다. 함수 의 조건 .`Guess::new``Guess``Guess``value``Guess::new
```

`Guess`매개 변수가 있거나 1에서 100 사이의 숫자만 반환하는 함수는 서명에서 an이 아닌 a 를 받거나 반환하며 `i32`본문에서 추가 검사를 수행할 필요가 없다고 선언할 수 있습니다.

## [요약](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#summary)

Rust의 오류 처리 기능은 보다 강력한 코드를 작성하는 데 도움이 되도록 설계되었습니다. 매크로 `panic!`는 프로그램이 처리할 수 없는 상태에 있음을 알리고 유효하지 않거나 잘못된 값으로 진행하려고 시도하는 대신 프로세스를 중지하도록 지시할 수 있습니다. 열거 `Result`형은 Rust의 유형 시스템을 사용하여 코드가 복구할 수 있는 방식으로 작업이 실패할 수 있음을 나타냅니다. `Result`코드를 호출하는 코드에 잠재적인 성공 또는 실패도 처리해야 함을 알리는 데 사용할 수 있습니다. 적절한 상황에서 `panic!`및 를 사용하면 피할 수 없는 문제에 직면했을 때 코드를 보다 안정적으로 만들 수 있습니다.`Result`

`Option`이제 표준 라이브러리가 및 enum 과 함께 제네릭을 사용하는 유용한 방법을 보았으므로 `Result`제네릭이 작동하는 방식과 코드에서 이를 사용할 수 있는 방법에 대해 설명하겠습니다.