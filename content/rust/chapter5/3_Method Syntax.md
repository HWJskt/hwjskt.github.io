+++
title = "Method Syntax"
weight = 3
template ="book_page_rust.html"

+++

## 요약

## [메서드 구문](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#method-syntax)

*메서드는* 함수와 유사합니다. `fn` 키워드와 이름으로 메서드를 선언하고 매개 변수와 반환 값을 가질 수 있으며 메서드가 다른 곳에서 호출될 때 실행되는 일부 코드를 포함합니다. 함수와 달리 메서드는 구조체( 각각 [6장](https://doc.rust-lang.org/book/ch06-00-enums.html) 과 [17장](https://doc.rust-lang.org/book/ch17-02-trait-objects.html) 에서 다루는 열거형 또는 특성 개체 )의 컨텍스트 내에서 정의되며 첫 번째 매개변수는 항상 `self`입니다. struct 메소드가 호출되고 있습니다.

### [방법 정의](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#defining-methods)

목록 5-13과 같이 `Rectangle` 인스턴스를 매개변수로 갖는 `area` 함수를 변경하고 `Rectangle` 구조체에 정의된 `area` 메서드를 대신 만들어 보겠습니다.

파일 이름: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        `The area of the rectangle is {} square pixels.`,
        rect1.area()
    );
}
```

Listing 5-13: `Rectangle` 구조체에 `area` 메서드 정의하기

`Rectangle` 컨텍스트 내에서 함수를 정의하기 위해 `Rectangle`에 대한 `impl`(구현) 블록을 시작합니다. 이 `impl` 블록 내의 모든 것은 `직사각형` 유형과 연결됩니다. 그런 다음 `impl` 중괄호 내에서 `area` 함수를 이동하고 서명 및 본문 내의 모든 위치에서 첫 번째(이 경우에만) 매개 변수를 `self`로 변경합니다. `area` 함수를 호출하고 `rect1`을 인수로 전달한 `main`에서 *메서드 구문을* 대신 사용하여 `Rectangle` 인스턴스에서 `area` 메서드를 호출할 수 있습니다. 메서드 구문은 인스턴스 다음에 옵니다. 점을 추가하고 그 뒤에 메서드 이름, 괄호 및 인수를 추가합니다.

`area`의 시그니처에서 `rectangle: &Rectangle` 대신 `&self`를 사용합니다. `&self`는 실제로 `self: &Self`의 줄임말입니다. `impl` 블록 내에서 `Self` 유형은 `impl` 블록이 있는 유형의 별칭입니다. 메소드는 첫 번째 매개변수에 대해 `Self` 유형의 `self`라는 이름의 매개변수를 가져야 합니다. 그래서 Rust는 첫 번째 매개변수 자리에 `self`라는 이름으로 이것을 축약할 수 있게 합니다. `rectangle: &Rectangle`에서 했던 것처럼 이 메서드가 `Self` 인스턴스를 빌린다는 것을 나타내기 위해 `self` 속기 앞에 `&`를 사용해야 합니다. 메서드는 `자체`의 소유권을 가져갈 수 있고 `자기`를 불변으로 빌릴 수 있습니다.

함수 버전에서 `&Rectangle`을 사용한 것과 같은 이유로 여기에서 `&self`를 선택했습니다. 소유권을 갖고 싶지 않고 구조체의 데이터를 쓰는 것이 아니라 읽기만 원합니다. 메소드가 수행하는 작업의 일부로 메소드를 호출한 인스턴스를 변경하려면 첫 번째 매개변수로 `&mut self`를 사용합니다. 첫 번째 매개 변수로 `self`만 사용하여 인스턴스 소유권을 가져오는 메서드는 거의 없습니다. 이 기술은 일반적으로 메서드가 `self`를 다른 것으로 변환하고 변환 후 호출자가 원래 인스턴스를 사용하지 못하게 하려는 경우에 사용됩니다.

메서드 구문을 제공하고 모든 메서드 서명에서 `self` 유형을 반복하지 않아도 되는 것 외에도 함수 대신 메서드를 사용하는 주된 이유는 구성을 위한 것입니다. 우리는 미래의 코드 사용자가 우리가 제공하는 라이브러리의 다양한 위치에서 `직사각형`의 기능을 검색하도록 하는 대신 유형의 인스턴스로 수행할 수 있는 모든 작업을 하나의 `impl` 블록에 넣었습니다.

구조체의 필드 중 하나와 동일한 이름을 메서드에 지정하도록 선택할 수 있습니다. 예를 들어, `width`라는 이름을 가진 `Rectangle`에 메소드를 정의할 수 있습니다.

파일 이름: src/main.rs

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!(`The rectangle has a nonzero width; it is {}`, rect1.width);
    }
}
```

여기서 인스턴스의 `width` 필드 값이 `0`보다 크면 `true`를 반환하고 값이 `0`이면 `false`를 반환하도록 `width` 메서드를 선택합니다. 필드를 사용할 수 있습니다. 어떤 목적을 위해 동일한 이름의 메서드 내에서. `main`에서 `rect1.width` 뒤에 괄호가 있으면 Rust는 `width` 메서드를 의미한다는 것을 압니다. 우리가 괄호를 사용하지 않을 때 Rust는 우리가 `너비` 필드를 의미한다는 것을 압니다.

항상 그런 것은 아니지만 종종 메서드에 필드와 동일한 이름을 지정하면 필드의 값만 반환하고 다른 작업은 수행하지 않기를 원합니다. *이와 같은 메서드를 getters* 라고 하며 Rust는 일부 다른 언어처럼 구조체 필드에 자동으로 구현하지 않습니다. Getter는 필드를 비공개로 만들 수 있지만 메서드는 공개할 수 있으므로 유형의 공개 API의 일부로 해당 필드에 대한 읽기 전용 액세스를 활성화할 수 있기 때문에 유용합니다. [7장](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword) 에서 public과 private이 무엇인지, 필드나 메서드를 public 또는 private으로 지정하는 방법에 대해 설명합니다.

> ### [`->` 연산자는 어디에 있습니까?](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#wheres-the---operator)
>
> C 및 C++에서는 메서드를 호출하는 데 두 가지 다른 연산자가 사용됩니다. `.`를 사용합니다. 개체에 대한 메서드를 직접 호출하는 경우 `->` 개체에 대한 포인터에서 메서드를 호출하고 먼저 포인터를 역참조해야 하는 경우. 즉, `object`가 포인터라면 `object->something()`은 `(*object).something()`과 유사하다.
>
> Rust에는 `->` 연산자와 동등한 것이 없습니다. 대신 Rust에는 *자동 참조 및 역참조* 라는 기능이 있습니다. 메소드를 호출하는 것은 러스트에서 이러한 동작을 하는 몇 안 되는 위치 중 하나입니다.
>
> 작동 방식은 다음과 같습니다: `object.something()`으로 메서드를 호출하면 Rust는 자동으로 `&`, `&mut` 또는 `*`를 추가하여 `object`가 메서드의 시그니처와 일치하도록 합니다. 즉, 다음은 동일합니다.
>
> ```rust
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 첫 번째 것이 훨씬 깨끗해 보입니다. 이 자동 참조 동작은 메서드에 명확한 수신자(`self` 유형)가 있기 때문에 작동합니다. 메서드의 리시버와 이름이 주어지면 Rust는 메서드가 읽기(`&self`)인지, 변경(`&mut self`)인지, 소모(`self`)인지 확실히 파악할 수 있습니다. Rust가 메서드 리시버에 대한 차용을 암시적으로 만든다는 사실은 실제로 소유권을 인체 공학적으로 만드는 데 큰 부분을 차지합니다.

### [매개변수가 더 많은 방법](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#methods-with-more-parameters)

`Rectangle` 구조체에 두 번째 메서드를 구현하여 메서드 사용을 연습해 봅시다. 이번에는 `Rectangle`의 인스턴스가 `Rectangle`의 다른 인스턴스를 가져오고 두 번째 `Rectangle`이 `self`(첫 번째 `Rectangle`)에 완전히 맞으면 `true`를 반환하도록 합니다. 그렇지 않으면 `false`를 반환해야 합니다. 즉, 일단 `can_hold` 메소드를 정의하면 Listing 5-14에 표시된 프로그램을 작성할 수 있기를 원합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!(`Can rect1 hold rect2? {}`, rect1.can_hold(&rect2));
    println!(`Can rect1 hold rect3? {}`, rect1.can_hold(&rect3));
}
```

목록 5-14: 아직 작성되지 않은 `can_hold` 메서드 사용

`rect2`의 두 치수가 `rect1`의 치수보다 작지만 `rect3`이 `rect1`보다 넓기 때문에 예상 출력은 다음과 같습니다.

```
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

우리는 메서드를 정의하고 싶다는 것을 알고 있으므로 `impl Rectangle` 블록 내에 있을 것입니다. 메서드 이름은 `can_hold`이고 다른 `Rectangle`의 불변 차용을 매개변수로 사용합니다. 메서드를 호출하는 코드를 보면 매개변수의 유형이 무엇인지 알 수 있습니다. `rect1.can_hold(&rect2)`는 `&rect2`를 전달합니다. `. 우리는 `rect2`를 읽기만 하면 되고(쓰기가 아닌 변경 가능한 빌림이 필요함을 의미함) `main`이 `rect2`의 소유권을 유지하여 호출 후 다시 사용할 수 있기를 원하기 때문에 이는 의미가 있습니다. `can_hold` 방법. `can_hold`의 반환 값은 부울입니다. 구현 시 `self`의 너비와 높이가 각각 다른 `Rectangle`의 너비와 높이보다 큰지 여부를 확인합니다. Listing 5-15에 표시된 Listing 5-13의 `impl` 블록에 새로운 `can_hold` 메소드를 추가해 보겠습니다.

파일 이름: src/main.rs

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Listing 5-15: 다른 `Rectangle` 인스턴스를 매개변수로 사용하는 `Rectangle`의 `can_hold` 메서드 구현

목록 5-14의 `main` 함수로 이 코드를 실행하면 원하는 출력을 얻을 수 있습니다. 메서드는 서명에 `self` 매개변수 뒤에 추가하는 여러 매개변수를 사용할 수 있으며 이러한 매개변수는 함수의 매개변수처럼 작동합니다.

### [관련 기능](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#associated-functions)

`impl` 블록 내에서 정의된 모든 함수는 `impl` 다음에 이름이 지정된 유형과 연관되기 때문에 *연관된 함수* 라고 합니다. 작업할 유형의 인스턴스가 필요하지 않기 때문에 첫 번째 매개변수로 `self`가 없는(따라서 메소드가 아닌) 연관된 함수를 정의할 수 있습니다. 우리는 이미 `String` 유형에 정의된 `String::from` 함수와 같은 하나의 함수를 사용했습니다.

메서드가 아닌 관련 함수는 구조체의 새 인스턴스를 반환하는 생성자에 자주 사용됩니다. 이들은 종종 `new`라고 불리지만 `new`는 특별한 이름이 아니며 언어에 내장되어 있지 않습니다. 예를 들어, 하나의 치수 매개변수를 갖고 너비와 높이 모두로 사용하는 `square`라는 관련 함수를 제공하도록 선택할 수 있으므로 동일한 값을 두 번 지정하지 않고 정사각형 `Rectangle`을 더 쉽게 만들 수 있습니다. :

파일 이름: src/main.rs

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

반환 유형과 함수 본문의 `Self` 키워드는 `impl` 키워드 뒤에 나타나는 유형의 별칭이며 이 경우에는 `Rectangle`입니다.

이 관련 함수를 호출하려면 구조체 이름과 함께 `::` 구문을 사용합니다. `let sq = 직사각형::square(3);` 예입니다. 이 함수는 구조체에 의해 네임스페이스가 지정됩니다. `::` 구문은 관련 함수와 모듈에서 생성된 네임스페이스 모두에 사용됩니다. [7장](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html) 에서 모듈에 대해 논의할 것입니다.

### [다중 `impl` 블록](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#multiple-impl-blocks)

각 구조체는 여러 `impl` 블록을 가질 수 있습니다. 예를 들어, Listing 5-15는 Listing 5-16에 표시된 코드와 동일하며 각 메소드는 자체 `impl` 블록에 있습니다.

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Listing 5-16: 여러 `impl` 블록을 사용하여 Listing 5-15 재작성

여기서는 이러한 메서드를 여러 `impl` 블록으로 분리할 이유가 없지만 유효한 구문입니다. 제네릭 타입과 특성에 대해 논의하는 10장에서 여러 개의 `impl` 블록이 유용한 경우를 보게 될 것입니다.

## [요약](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#summary)

구조체를 사용하면 도메인에 의미 있는 사용자 지정 유형을 만들 수 있습니다. 구조체를 사용하면 연결된 데이터 조각을 서로 연결하고 각 조각의 이름을 지정하여 코드를 명확하게 만들 수 있습니다. `impl` 블록에서 유형과 연결된 함수를 정의할 수 있으며 메서드는 구조체 인스턴스의 동작을 지정할 수 있는 일종의 연결된 함수입니다.

그러나 구조체가 사용자 지정 유형을 생성할 수 있는 유일한 방법은 아닙니다. Rust의 열거형 기능을 사용하여 도구 상자에 다른 도구를 추가해 보겠습니다.
