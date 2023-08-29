+++
title = "Storing Lists of Values with Vectors"
weight = 1
template = "book_page_rust.html"

+++

## 요약

- Vec<T> 와 vector 는 같은 말이다.
- 파이썬 리스트처럼 하나에 여러개의 값을 저장
- 각 데이터는 메모리에서 서로 옆에 저장
- 어떤 타입이든 가능하지만, 동일한 타입의 값만 저장가능
- 여러 종류의 타입을 포함하고 싶으면
  - 여러타입이 포함된 enum 을 만들어 사용한다. 
  - (enum은 하나의 타입으로 인식되니까)

- 백터 만들기
  - 빈 백터를 만들 때 `new`사용 /  type 을 적어줘야한다.
  - 값을 가지는 백터를 만들때는 vec! 매크로를 사용하고, type은 자동이다.

- 백터에 내용추가 : 
  - .push 사용
  - 메모리 위치가 바뀌어, 미리 참조해두었던 내용이 없어진다

- 백터의 내용 읽기
  - [index]  : 값을 바로 얻음
  - .get(index) : Option<&T> 형태로 값을 얻음

- 벡터 내부 값 iteration
  - 일반적인 for 문 사용가능
  - 참조한 벡터를 for 문이 사용중이면, for문 내부에서 벡터를 수정할수 없다


## [벡터(Vector)를 사용하여 여러 값을 저장](https://doc.rust-lang.org/book/ch08-01-vectors.html#storing-lists-of-values-with-vectors)

우리가 살펴볼 첫 번째 컬렉션 타입은 `Vec<T>` 이며, *vector* 라고도 합니다. 벡터를 사용하면 단일 데이터 구조에 둘 이상의 값을 저장할 수 있으며, 모든 값은 메모리에서 나란히 배치됩니다. 벡터는 동일한 타입의 값만 저장할 수 있습니다. 벡터는 아이템 리스트의 경우에 유용합니다: 파일의 텍스트 줄, 장바구니 안의 항목의 가격

### [새 벡터 만들기](https://doc.rust-lang.org/book/ch08-01-vectors.html#creating-a-new-vector)

새로운 빈 벡터를 생성하기 위해 목록 8-1에 표시된 것처럼 `Vec::new` 함수를 호출합니다.

```rust
    let v: Vec<i32> = Vec::new();
```

목록 8-1: `i32` 타입의 값을 담을 새 빈 벡터 만들기

여기에 타입 주석을 추가했습니다. 우리가 이 벡터에 어떤 값도 삽입하지 않기 때문에 Rust는 우리가 어떤 종류의 요소를 저장하려고 하는지 알지 못합니다. 이것은 중요한 포인트입니다. 벡터는 제네릭을 사용하여 구현됩니다. 10장에서 고유한 타입으로 제네릭을 사용하는 방법을 다룰 것입니다. 지금은 표준 라이브러리에서 제공하는  `Vec<T>` 타입은 모든 타입을 가질수 있다고만 알아주세요. 특정 타입을 보유하기 위해 벡터를 만들 때 꺾쇠 괄호 안에 타입을 지정할 수 있습니다. Listing 8-1에서 Rust에게 `v`의 `Vec<T>`는 `i32` 타입의 요소를 보유합니다.

흔히, Rust가 저장하려는 값의 타입을 추론할수 있도록  `Vec<T` 가 초기 값을 가지면, 이 타입 주석을 적어줄 필요가 없습니다. Rust는 `vec!` 매크로를 편리하게 제공하며, 이 매크로는 사용자가 지정한 값을 보유하는 새 벡터를 생성합니다. . 목록 8-2는 새로운 `Vec<i32>`는 값 `1`, `2` 및 `3`을 포함합니다. 정수 타입은 `i32`입니다. 3장의 `데이터 타입`](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) 섹션 에서 논의한 것처럼 이것이 기본 정수 타입이기 때문입니다.

```rust
    let v = vec![1, 2, 3];
```

목록 8-2: 값을 포함하는 새 벡터 만들기

초기 `i32` 값을 제공했기 때문에 Rust는 `v`의 타입이 `Vec<i32>`임을 추론할 수 있습니다. 타입 어노테이션은 필요하지 않습니다. 다음으로 벡터를 수정하는 방법을 살펴보겠습니다.

### [벡터 업데이트](https://doc.rust-lang.org/book/ch08-01-vectors.html#updating-a-vector)

벡터를 생성한 다음 여기에 요소를 추가하려면 Listing 8-3과 같이 `push` 방법을 사용할 수 있습니다.

```rust
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
```

Listing 8-3: `push` 방법을 사용하여 벡터에 값 추가

모든 변수와 마찬가지로, 값을 변경하려면 3장에서 설명한 것처럼 `mut` 키워드를 사용하여 변경 가능하게 만들어야 합니다. 내부에 배치하는 숫자는 모두 `i32` 타입이고 Rust 데이터에서 이것을 추론하므로 `Vec<i32`> 라고 주석을 적지 않아도 됩니다.

### [벡터의 요소 읽기](https://doc.rust-lang.org/book/ch08-01-vectors.html#reading-elements-of-vectors)

벡터에 저장된 값을 참조하는 방법에는 인덱싱을 통하거나 `get` 메서드를 사용하는 두 가지 방법이 있습니다. 다음 예제에서는 명확성을 높이기 위해 이러한 함수에서 반환되는 값의 타입에 주석을 달았습니다.

목록 8-4는 인덱싱 구문과 `get` 방법을 사용하여 벡터의 값에 액세스하는 두 가지 방법을 보여줍니다.

```rust
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!(`The third element is {third}`);

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!(`The third element is {third}`),
        None => println!(`There is no third element.`),
    }
```

목록 8-4: 인덱싱 구문 또는 `get` 메서드를 사용하여 벡터의 항목에 액세스

여기에 몇 가지 세부 사항이 있습니다. 벡터는 0부터 시작하여 숫자로 인덱싱되기 때문에 인덱스 값 `2`를 사용하여 세 번째 요소를 얻습니다. `&` 및 `[]`를 사용하면 인덱스 값에 있는 요소에 대한 참조를 제공합니다. 
인수로 전달된 인덱스와 함께 `get` 메서드를 사용하면 `match`와 함께 사용할 수 있는 `Option<&T>`를 얻습니다.

Rust가 요소를 참조하는 이 두 가지 방법을 제공하는 이유는 기존 요소 범위 밖의 인덱스 값을 사용하려고 할 때 프로그램이 동작하는 방식을 선택할 수 있도록 하기 위함입니다. 예를 들어, 목록 8-5에 표시된 것처럼 5개 요소의 벡터가 있고 각 기술을 사용하여 인덱스 100의 요소에 액세스하려고 하면 어떤 일이 발생하는지 봅시다.

```rust
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
```

목록 8-5: 5개의 요소를 포함하는 벡터에서 인덱스 100의 요소에 액세스 시도

이 코드를 실행할 때 첫 번째 `[]` 메서드는 존재하지 않는 요소를 참조하기 때문에 프로그램을 패닉 상태로 만듭니다. 이 방법은 벡터의 끝을 지나 요소에 액세스하려는 시도가 있는 경우 프로그램을 중단시키려는 경우에 가장 적합합니다.

`get` 메서드에 벡터 외부에 있는 인덱스가 전달되면 당황하지 않고 `None`을 반환합니다. 벡터 범위를 벗어난 요소에 액세스하는 것이 정상적인 상황에서 가끔 발생할 수 있는 경우 이 방법을 사용합니다. 그러면 코드는 6장에서 설명한 대로 `Some(&element)` 또는 `None`을 처리하는 논리를 갖게 됩니다. 예를 들어 인덱스는 사람이 숫자를 입력하는 것에서 나올 수 있습니다. 사용자가 실수로 너무 큰 숫자를 입력하여 프로그램이 `None` 값을 얻는 경우 사용자에게 현재 벡터에 몇 개의 항목이 있는지 알려주고 유효한 값을 입력할 수 있는 또 다른 기회를 제공할 수 있습니다. 오타로 인해 프로그램이 충돌하는 것보다 더 사용자 친화적일 것입니다!

프로그램 안에 유효한 참조가 있는 경우, 차용 검사기(borrow checker)는 소유권 및 차용 규칙(4장에서 다룸)을 시행하여 이 참조 및 벡터 내용에 대한 다른 참조가 유효한지 확인합니다. 동일한 범위에서 가변 및 불변 참조를 가질 수 없다는 규칙을 기억하세요. 이 규칙은 벡터의 첫 번째 요소에 대한 불변 참조를 보유하고 끝에 요소를 추가하려고 시도하는 목록 8-6에 적용됩니다. 이 프로그램은 나중에 함수에서 해당 요소를 참조하려고 하면 작동하지 않습니다.

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];  //첫번재 요소를 참조

    v.push(6);  // 마지막에 요소를 추가 -> 메모리 새 공간으로 이동

    println!(`The first element is: {first}`);
```

목록 8-6: 항목에 대한 참조를 유지하면서 벡터에 요소 추가 시도

이 코드를 컴파일하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!(`The first element is: {first}`);
  |                                      ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error
```

목록 8-6의 코드는 제대로 작동하는 것처럼 보일 수 있습니다. 벡터의 첫 번째 요소를 참조했는데, 벡터의 마지막에서 일어난 변경 사항에 대해 왜 관심을 가져야 할까요? 이 오류는 벡터가 작동하는 방식 때문입니다. 벡터는 메모리에서 값을 서로 옆에 위치시키기 때문에, 벡터 끝에 새 요소를 추가하려면 새 메모리를 할당하고 이전 요소를 새 공간에 복사해야 할 수 있습니다. 벡터가 현재 저장된 위치에 모든 요소를 나란히 놓을 공간이 충분하지 않습니다. 이 경우 첫 번째 요소에 대한 참조는 할당 해제된 메모리를 가리킵니다. 차용 규칙은 프로그램이 그러한 상황에서 종료되는 것을 방지합니다.

> 참고: `Vec<T>`타입의 구현 세부 사항에 대한 자세한 내용은 [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/vec/vec.html) 참조 .

### [벡터의 값에 대한 반복(iterating)](https://doc.rust-lang.org/book/ch08-01-vectors.html#iterating-over-the-values-in-a-vector)

벡터의 각 요소에 차례로 액세스하려면 인덱스를 사용하여 한 번에 하나씩 액세스하는 대신 모든 요소를 반복합니다. 목록 8-7은 벡터에 있는 `i32` 값의  각 요소에 대한 불변 참조를 가져오고, 인쇄하기 위해 `for` 루프를 사용하는 방법을 보여줍니다.

```rust
    let v = vec![100, 32, 57];
    for i in &v {
        println!(`{i}`);
    }
```

목록 8-7: `for` 루프를 사용하여 요소를 반복하여 벡터의 각 요소 인쇄

모든 요소를 변경하기 위해 가변 벡터의 각 요소에 대한 가변 참조를 반복할 수도 있습니다. 목록 8-8의 `for` 루프는 각 요소에 `50`을 추가합니다.

```rust
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
```

목록 8-8: 벡터의 요소에 대한 변경 가능한 참조에 대한 반복

가변 참조가 참조하는 값을 변경하려면 `+=` 연산자를 사용하기 전에 `*` 역참조 연산자를 사용하여 `i`의 값을 가져와야 합니다. 역참조 연산자에 대한 자세한 내용은 15장의 [`역참조 연산자를 사용하여 값에 대한 포인터 추적`](https://doc.rust-lang.org/book/ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator) 섹션 에서 설명합니다.

불변이든 가변이든 벡터를 반복하는 것은 빌림 검사기의 규칙 때문에 안전합니다. Listing 8-7 및 Listing 8-8의 `for` 루프 본문에서 항목을 삽입하거나 제거하려고 시도하면 Listing 8-6의 코드에서 얻은 것과 유사한 컴파일러 오류가 발생합니다. `for` 루프가 "참조하고 있는 벡터" 를 사용하고 있으면, 전체 벡터는 수정될 수 없습니다. 

### [열거형을 사용하여 여러 타입 저장](https://doc.rust-lang.org/book/ch08-01-vectors.html#using-an-enum-to-store-multiple-types)

벡터는 동일한 타입의 값만 저장할 수 있습니다. 이는 불편할 수 있습니다. 다양한 타입의 항목 리스트를 저장해야 하는 경우가 분명히 있습니다. 다행스럽게도 열거형(enum)의 변형들(variants)은 동일한 열거형 타입으로 정의되므로, 다른 타입의 요소를 나타내기 위해 하나의 타입이 필요할 때 열거형을 정의하고 사용할 수 있습니다!

예를 들어 행의 일부 열에 정수, 부동 소수점 숫자 및 문자열이 포함된 스프레드시트의 행에서 값을 가져오고 싶다고 가정해 보겠습니다. 변형이 다른 값 타입을 보유할 열거형을 정의할 수 있으며 모든 열거형 변형은 동일한 타입, 즉 열거형으로 간주됩니다. 그런 다음 해당 열거형을 보유할 벡터를 생성할 수 있으므로 궁극적으로 다른 타입을 보유합니다. Listing 8-9에서 이를 증명했습니다.

```rust
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from(`blue`)),
        SpreadsheetCell::Float(10.12),
    ];
```

Listing 8-9: 하나의 벡터에 다른 타입의 값을 저장하는 `enum` 정의

Rust는 각 요소를 저장하는 데 필요한 힙의 메모리 양을 정확히 알기 위해 컴파일 시간에 벡터에 어떤 타입이 있는지 알아야 합니다. 또한 이 벡터에서 어떤 타입이 허용되는지 명시해야 합니다. Rust가 벡터가 어떤 타입이든 가질 수 있도록 허용했다면, 하나 이상의 타입이 벡터의 요소에 대해 수행되는 작업에서 오류를 일으킬 가능성이 있습니다. 열거형과 `match` 표현식을 사용한다는 것은 6장에서 논의한 것처럼 Rust가 컴파일 타임에 가능한 모든 경우를 처리한다는 것을 의미합니다.

프로그램이 벡터에 저장하기 위해 런타임에 얻을 수 있는 전체 타입 세트를 모른다면 enum 기술이 작동하지 않습니다. 대신 17장에서 다룰 특성 개체를 사용할 수 있습니다.

벡터를 사용하는 가장 일반적인 방법에 대해 논의했으므로, 표준라이브러리에서 `Vec<T>`에 정의된 모든 유용한 방법에 대한 [API 문서](https://doc.rust-lang.org/std/vec/struct.Vec.html)를 검토하십시오. 예를 들어 `push` 외에도 `pop` 메서드는 마지막 요소를 제거하고 반환합니다.

### [벡터를 삭제하면 요소가 삭제됨](https://doc.rust-lang.org/book/ch08-01-vectors.html#dropping-a-vector-drops-its-elements)

다른 `구조체(struct)`와 마찬가지로 벡터는 목록 8-10에 설명된 대로 범위를 벗어나면 해제됩니다.

```rust
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
```

Listing 8-10: 벡터와 해당 요소가 놓이는 위치 표시

벡터가 삭제되면 모든 내용도 삭제됩니다. 즉, 보유하고 있는 정수가 정리됩니다. 차용 검사기는 벡터 자체가 유효한 동안에만 벡터 내용에 대한 참조가 사용되는지 확인합니다.

다음 컬렉션 타입인 `String`로 이동하겠습니다!
