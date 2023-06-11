+++
title = "Generic Types, Traits, and Lifetimes"
weight = 10
sort_by = "weight"
template ="book_section_rust.html"

+++

## 요약

<!-- more -->

**# [****일반 유형, 특성 및 수명****](**https://doc.rust-lang.org/book/ch10-00-generics.html#generic-types-traits-and-lifetimes**)**

모든 프로그래밍 언어에는 개념의 중복을 효과적으로 처리하기 위한 도구가 있습니다. Rust에서 그러한 도구 중 하나는 **제네릭** 입니다 : 구체적인 유형 또는 기타 속성에 대한 추상 대체물입니다. 코드를 컴파일하고 실행할 때 제네릭의 위치에 무엇이 있는지 모른 채 제네릭의 동작 또는 제네릭이 다른 제네릭과 어떻게 관련되는지 표현할 수 있습니다.

`i32`함수는 or 와 같은 구체적인 유형 대신 일부 일반 유형의 매개변수를 사용할 수 있습니다 `String`. 이는 함수가 알 수 없는 값을 가진 매개변수를 사용하여 여러 구체적인 값에서 동일한 코드를 실행하는 것과 같은 방식입니다. 사실 우리는 이미 6장에서 `Option<T>`, 8장에서 `Vec<T>`및 `HashMap<K, V>`, 9장에서 제네릭을 사용했습니다 `Result<T, E>`. 이 장에서는 제네릭을 사용하여 고유한 유형, 함수 및 메서드를 정의하는 방법을 살펴봅니다!

먼저 코드 중복을 줄이기 위해 함수를 추출하는 방법을 검토합니다. 그런 다음 동일한 기술을 사용하여 매개 변수 유형만 다른 두 함수에서 일반 함수를 만듭니다. 또한 구조체 및 열거형 정의에서 제네릭 형식을 사용하는 방법도 설명합니다.

**그런 다음 특성을** 사용하여 일반적인 방식으로 동작을 정의하는 방법을 배웁니다. 특성을 일반 유형과 결합하여 모든 유형이 아닌 특정 동작이 있는 유형만 허용하도록 일반 유형을 제한할 수 있습니다.

**마지막으로 수명에** 대해 논의할 것입니다. 참조가 서로 어떻게 관련되어 있는지에 대한 컴파일러 정보를 제공하는 다양한 제네릭입니다. 수명을 통해 빌린 값에 대한 충분한 정보를 컴파일러에 제공하여 우리의 도움 없이 할 수 있는 것보다 더 많은 상황에서 참조가 유효하도록 할 수 있습니다.

**## [****함수를 추출하여 중복 제거****](**https://doc.rust-lang.org/book/ch10-00-generics.html#removing-duplication-by-extracting-a-function**)**

제네릭을 사용하면 특정 유형을 여러 유형을 나타내는 자리 표시자로 대체하여 코드 중복을 제거할 수 있습니다. 제네릭 구문을 살펴보기 전에 먼저 특정 값을 여러 값을 나타내는 자리 표시자로 대체하는 함수를 추출하여 제네릭 형식을 포함하지 않는 방식으로 중복을 제거하는 방법을 살펴보겠습니다. 그런 다음 동일한 기술을 적용하여 일반 함수를 추출합니다! 함수로 추출할 수 있는 중복 코드를 인식하는 방법을 살펴보면 제네릭을 사용할 수 있는 중복 코드를 인식하기 시작할 것입니다.

목록에서 가장 큰 숫자를 찾는 목록 10-1의 짧은 프로그램으로 시작합니다.

파일 이름: src/main.rs

\```rust

fn main() {

​    let number_list = vec![34, 50, 25, 100, 65];

​    let mut largest = &number_list[0];

​    for number in &number_list {

​        if number > largest {

​            largest = number;

​        }

​    }

​    println!(`The largest number is {}`, largest);

}

\```

Listing 10-1: 숫자 목록에서 가장 큰 숫자 찾기

정수 목록을 변수에 저장 `number_list`하고 목록의 첫 번째 숫자에 대한 참조를 라는 변수에 배치합니다 `largest`. 그런 다음 목록의 모든 숫자를 반복하고 현재 숫자가 에 저장된 숫자보다 크면 `largest`해당 변수의 참조를 바꿉니다. 그러나 현재 숫자가 지금까지 본 가장 큰 숫자보다 작거나 같으면 변수가 변경되지 않고 코드가 목록의 다음 숫자로 이동합니다. 목록의 모든 숫자를 고려한 후 `largest`가장 큰 숫자(이 경우 100)를 참조해야 합니다.

우리는 이제 두 개의 서로 다른 숫자 목록에서 가장 큰 숫자를 찾는 임무를 받았습니다. 이를 위해 Listing 10-1의 코드를 복제하고 Listing 10-2와 같이 프로그램의 서로 다른 두 위치에서 동일한 논리를 사용하도록 선택할 수 있습니다.

파일 이름: src/main.rs

\```rust

fn main() {

​    let number_list = vec![34, 50, 25, 100, 65];

​    let mut largest = &number_list[0];

​    for number in &number_list {

​        if number > largest {

​            largest = number;

​        }

​    }

​    println!(`The largest number is {}`, largest);

​    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

​    let mut largest = &number_list[0];

​    for number in &number_list {

​        if number > largest {

​            largest = number;

​        }

​    }

​    println!(`The largest number is {}`, largest);

}

\```

**목록 10-2: 두 개의** 숫자 목록 에서 가장 큰 숫자를 찾는 코드

이 코드는 작동하지만 코드 복제는 지루하고 오류가 발생하기 쉽습니다. 또한 코드를 변경하고 싶을 때 여러 위치에서 코드를 업데이트해야 한다는 점을 기억해야 합니다.

이 중복을 제거하기 위해 매개변수에 전달된 정수 목록에서 작동하는 함수를 정의하여 추상화를 만듭니다. 이 솔루션은 코드를 더 명확하게 만들고 목록에서 가장 큰 숫자를 찾는 개념을 추상적으로 표현할 수 있게 해줍니다.

Listing 10-3에서는 가장 큰 수를 찾는 코드를 이라는 함수로 추출합니다 `largest`. 그런 다음 목록 10-2의 두 목록에서 가장 큰 숫자를 찾는 함수를 호출합니다. `i32`우리는 미래에 가질 수 있는 다른 값 목록에서도 이 함수를 사용할 수 있습니다.

파일 이름: src/main.rs

\```rust

fn largest(list: &[i32]) -> &i32 {

​    let mut largest = &list[0];

​    for item in list {

​        if item > largest {

​            largest = item;

​        }

​    }

​    largest

}

fn main() {

​    let number_list = vec![34, 50, 25, 100, 65];

​    let result = largest(&number_list);

​    println!(`The largest number is {}`, result);

​    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

​    let result = largest(&number_list);

​    println!(`The largest number is {}`, result);

}

\```

Listing 10-3: 두 목록에서 가장 큰 숫자를 찾는 추상화된 코드

이 함수에는 함수에 전달할 수 있는 구체적인 값 조각을 나타내는 이라는 `largest`매개변수가 있습니다. 결과적으로 함수를 호출하면 코드는 전달한 특정 값에서 실행됩니다.`list``i32`

요약하면 Listing 10-2에서 Listing 10-3으로 코드를 변경하기 위해 수행한 단계는 다음과 같습니다.

1. 중복 코드를 식별합니다.
2. 중복 코드를 함수 본문으로 추출하고 함수 서명에서 해당 코드의 입력 및 반환 값을 지정합니다.
3. 대신 함수를 호출하도록 중복 코드의 두 인스턴스를 업데이트합니다.

다음으로 제네릭과 동일한 단계를 사용하여 코드 중복을 줄입니다. 함수 본문이 `list`특정 값 대신 추상에서 작동할 수 있는 것과 같은 방식으로 제네릭을 사용하면 코드가 추상 유형에서 작동할 수 있습니다.

예를 들어, 값 조각에서 가장 큰 항목을 찾는 함수 `i32`와 값 조각에서 가장 큰 항목을 찾는 함수의 두 가지 함수가 있다고 가정합니다 `char`. 그 중복을 어떻게 제거할까요? 알아 보자!
