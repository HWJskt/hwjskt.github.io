+++
title = "An Example Program Using Structs"
weight = 2
template ="book_page_rust.html"

+++



## 요약

- 구조체 출력 방법1.  println! 와 `Debug` trait 을 사용
  - 구조체 정의 앞에 #[derive(Debug)] 를 붙이고, 
  - 출력할때  println! 의 {} 대신 다른 걸 사용한다
  - {:?} : 한줄로 출력
  - {:#?} : 여러줄로 출력
  
- 구조체 출력 방법2.  dbg! 사용

  - 구조체 정의 앞에 #[derive(Debug)] 를 붙이고, 

  - 파일이름, 위치까지 같이 출력

  - 값을 반환한다

    


## [구조체를 사용한 예제 프로그램](https://doc.rust-lang.org/book/ch05-02-example-structs.html#an-example-program-using-structs)

언제 구조체를 사용해야 하는지 이해하기 위해 직사각형의 면적을 계산하는 프로그램을 작성해 봅시다. 단일 변수를 사용하여 시작한 다음, 구조체를 대신 사용할 때까지 프로그램을 리팩터링합니다.

픽셀로 지정된 직사각형의 너비와 높이를 가져오고 직사각형의 면적을 계산하는 *rectangles* 이라는 Cargo로 새로운 바이너리 프로젝트를 만들어 봅시다. *Listing 5-8은 우리 프로젝트의 src/main.rs* 에서 정확히 그것을 수행하는 한 가지 방법이 있는 짧은 프로그램을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        `The area of the rectangle is {} square pixels.`,
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

목록 5-8: 별도의 너비 및 높이 변수로 지정된 사각형의 면적 계산

이제 `cargo run`을 사용하여 이 프로그램을 실행합니다.

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/rectangles`
The area of the rectangle is 1500 square pixels.
```

이 코드는 각 치수와 함께 `area` 함수를 호출하여 사각형의 면적을 파악하는 데 성공했지만 이 코드를 명확하고 읽기 쉽게 만들기 위해 더 많은 작업을 수행할 수 있습니다.

이 코드의 문제는 `area`의 시그니쳐에서 잘 보입니다.

```rust
fn area(width: u32, height: u32) -> u32 {
```

`area` 함수는 하나의 직사각형의 면적을 계산하기로 되어 있지만, 우리가 작성한 함수에는 두 개의 매개변수가 있고 매개변수가 관련되어 있다는 것이 우리 프로그램의 어느 곳에서도 명확하지 않습니다. 너비와 높이를 함께 그룹화하는 것이 더 읽기 쉽고 관리하기 쉽습니다. 우리는 이미 3장의 [튜플 타입](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션에서 튜플을 사용하여 이를 수행할 수 있는 한 가지 방법에 대해 논의했습니다.

### [튜플로 리팩토링](https://doc.rust-lang.org/book/ch05-02-example-structs.html#refactoring-with-tuples)

목록 5-9는 튜플을 사용하는 프로그램의 다른 버전을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        `The area of the rectangle is {} square pixels.`,
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

목록 5-9: 튜플로 사각형의 너비와 높이 지정하기

어떤 면에서는 이 프로그램이 더 좋습니다. 튜플을 사용하면 약간의 구조를 추가할 수 있으며 이제 하나의 인수만 전달합니다. 그러나 어떤면에서 이 버전은 덜 명확합니다. 튜플은 요소의 이름을 지정하지 않으므로 튜플의 일부를 인덱싱해야 하므로 계산이 덜 명확해집니다.

너비와 높이를 섞는 것은 면적 계산에 문제가 되지 않지만 화면에 사각형을 그리려면 중요합니다! `width`는 튜플 인덱스 `0`이고 `height`는 튜플 인덱스 `1`이라는 점을 기억해야 하기 때문에 불편합니다. 이것은 다른 사람이 우리 코드를 사용한다면 훨씬 더 어려울 것입니다. 코드에서 데이터의 의미를 전달하지 않았기 때문에 이제 오류가 생기기 더 쉽습니다.

### [구조체로 리팩토링: 더 많은 의미 추가](https://doc.rust-lang.org/book/ch05-02-example-structs.html#refactoring-with-structs-adding-more-meaning)

구조체를 사용하여 데이터에 레이블을 지정하여 의미를 추가합니다. Listing 5-10에 표시된 것처럼 사용 중인 튜플을 전체 이름과 부분 이름이 있는 구조체로 변환할 수 있습니다.

파일 이름: src/main.rs

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        `The area of the rectangle is {} square pixels.`,
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

Listing 5-10: `Rectangle` 구조체 정의하기

여기서 우리는 구조체를 정의하고 이름을 `Rectangle`로 지정했습니다. 중괄호 안에 `width` 및 `height` 필드를 정의했으며 둘 다 `u32` 유형을 가집니다. 그런 다음 `main`에서 너비가 `30`이고 높이가 `50`인 `Rectangle`의 특정 인스턴스를 만들었습니다.

우리의 `area` 함수는 이제 `rectangle`이라는 이름의 매개변수 하나를 사용하여 정의되며, 그 유형은 `Rectangle` 구조체 인스턴스의 불변 차용(immutable borrow)입니다. 4장에서 언급했듯이 소유권을 갖기보다 구조체를 빌리고 싶습니다. 이런 식으로 `main`은 소유권을 유지하고 `rect1`을 계속 사용할 수 있습니다. 이것이 우리가 함수 서명에서 `&`를 사용하고 함수를 호출하는 이유입니다.

`area` 함수는 `Rectangle` 인스턴스의 `width` 및 `height` 필드에 액세스합니다(차용한 구조체 인스턴스의 필드에 액세스해도 필드 값이 이동하지 않으므로 구조체 차용을 자주 볼 수 있습니다). `area`에 대한 함수 시그니쳐는 이제 정확히 `width` 및 `height` 필드를 사용하여 `Rectangle`의 면적을 계산합니다. 이것은 너비와 높이가 서로 관련되어 있음을 전달하고 `0`과 `1`의 튜플 인덱스 값을 사용하는 대신 값에 설명적인 이름을 부여합니다. 이것은 명확성이라는 측면에서 승리입니다.

### [파생 특성으로 유용한 기능 추가](https://doc.rust-lang.org/book/ch05-02-example-structs.html#adding-useful-functionality-with-derived-traits)

프로그램을 디버깅하고 모든 필드의 값을 보는 동안 `Rectangle`의 인스턴스를 인쇄할 수 있다면 유용할 것입니다. Listing 5-11은 [`println!` 매크로는](https://doc.rust-lang.org/std/macro.println.html) 이전 장에서 사용한 것과 같습니다. 그러나 이것은 작동하지 않습니다.

파일 이름: src/main.rs

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(`rect1 is {}`, rect1);
}
```

Listing 5-11: `Rectangle` 인스턴스 인쇄 시도

이 코드를 컴파일하면 다음 핵심 메시지와 함께 오류가 발생합니다.

```
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
```

`println!` 매크로는 많은 종류의 서식을 지정할 수 있으며, 기본적으로 중괄호는 `println!` 에게 `Display`로 알려진 형식 사용한다고 이야기합니다. `Display` 형식은 최종 사용자가 직접 사용하기 위한 결과물입니다. 지금까지 본 기본 유형은 기본적으로 `Display`를 구현합니다. 왜냐하면 사용자에게 `1` 또는 다른 기본 유형을 표시하려는 방법은 한 가지뿐이기 때문입니다. 그러나 구조체를 사용하면 더 많은 표시 가능성이 있기 때문에 `println!`의 출력 형식이 명확하지 않습니다. 쉼표를 원하십니까? 중괄호를 인쇄하시겠습니까? 모든 필드를 표시해야 합니까? 이러한 모호성으로 인해 Rust는 우리가 원하는 것을 추측하려고 시도하지 않으며, 구조체에는 `println!` 및 `{}` 자리 표시자를 함께 사용할 `Display` 구현이 제공되지 않습니다. 

오류를 계속 읽으면 다음과 같은 유용한 메모를 찾을 수 있습니다.

```
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
```

고쳐봅시다! `println!` 매크로 호출은 이제 `println!(rect1 is {:?}, rect1);`처럼 보입니다. 중괄호 안에 지정자 `:?`를 넣으면  `println!`에게 `Debug`라는 출력 형식을 사용하게 합니다.  `Debug` 특성을 사용하면 개발자에게 유용한 방식으로 구조체를 인쇄할 수 있으므로 코드를 디버깅하는 동안 해당 값을 볼 수 있습니다.

이 변경으로 코드를 컴파일합니다. 드랏! 여전히 오류가 발생합니다.

```
error[E0277]: `Rectangle` doesn't implement `Debug`
```

그러나 다시 컴파일러는 다음과 같은 유용한 정보를 제공합니다.

```
   = help: the trait `Debug` is not implemented for `Rectangle`
   = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
```

Rust에는 디버깅 정보를 출력하는 기능이 *포함되어* 있지만 구조체에서 해당 기능을 사용할 수 있도록 명시적으로 선택해야 합니다. 이를 위해 Listing 5-12에 표시된 것처럼 구조체 정의 바로 앞에 외부 속성 `#[derive(Debug)]`를 추가합니다.

파일 이름: src/main.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(`rect1 is {:?}`, rect1);
}
```

Listing 5-12: `Debug` 트레이트를 유도하기 위한 속성 추가 및 디버그 포매팅을 사용하여 `Rectangle` 인스턴스 인쇄

이제 프로그램을 실행하면 오류가 발생하지 않으며 다음과 같은 결과가 표시됩니다.

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle { width: 30, height: 50 }
```

멋진! 가장 예쁜 출력은 아니지만 이 인스턴스에 대한 모든 필드의 값을 표시하므로 디버깅 중에 확실히 도움이 됩니다. 더 큰 구조체가 있는 경우 좀 더 읽기 쉬운 출력을 갖는 것이 유용합니다. 이 경우 `println!`에서 `{:?}` 대신 `{:#?}`를 사용할 수 있습니다. 예시에서 `{:#?}` 스타일을 사용하면 다음이 출력됩니다.

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

`Debug` 형식을 사용하여 값을 출력하는 또 다른 방법은 [dbg! 매크로](https://doc.rust-lang.org/std/macro.dbg.html)를 사용하는 것입니다. 이것은 식의 소유권을 갖는데, 참조를 취하는 `println!`와는 반대입니다. `dbg!` 매크로 호출은 파일이름, 코드의 위치(줄 번호), 해당 식의 결과 값을 출력하고, 값의 소유권을 반환합니다.

> 참고: `dbg!` 매크로는 표준 오류 콘솔 스트림(`stderr`)에 출력합니다. 표준 출력 콘솔 스트림(`stdout`)에 인쇄하는 `println!`과 차이가 있습니다. [12장의 표준 출력 대신 표준 오류에 오류 메시지 쓰기](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html)  섹션에서 `stderr` 및 `stdout`에 대해 자세히 설명합니다.

다음은 `rect1`의 전체 구조체 값뿐만 아니라 `width` 필드에 할당되는 값에 관심이 있는 예입니다.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale), // 값이 (30 * scale) 과 같다
        height: 50,
    };

    dbg!(&rect1);
}
```

 `30 * scale`이라는 식에 `dbg!`를 넣을 수 있습니다. 그리고 `dbg!` 가 표현식 값의 소유권을 반환하기 때문에  `width` 필드는 `dbg!`를 호출하지 않은것과 같은 값을 얻습니다.    우리는 `dbg!` 가 `rect1`의 소유권을 가지는 것을 원하지 않습니다. 그래서 다음 호출에서 `rect1`에 대한 참조를 사용합니다. 이 예제의 출력은 다음과 같습니다.

```bash
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```

첫 번째 출력 비트는 *src/main.rs* 라인 10에서 `30 * scale` 표현식을 디버깅하고 있으며 결과 값은 `60`입니다(정수에 대해 구현된 `디버그` 형식은 값만 출력). *src/main.rs* 의 14번째 줄에 있는 `dbg!` 호출은 `Rectangle` 구조체인 `&rect1`의 값을 출력합니다. 이 출력은 `Rectangle` 타입의 예쁜 `디버그` 포매팅을 사용합니다. `dbg!` 매크로는 코드가 수행하는 작업을 파악하려고 할 때 정말 유용할 수 있습니다!

Rust는 `Debug` 특성 외에도 사용자 지정 유형에 유용한 동작을 추가할 수 있는 `derive` 특성과 함께 사용할 여러 특성을 제공했습니다. 이러한 특성과 행동은 [부록 C](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) 에 나열되어 있습니다. 10장에서 사용자 지정 동작으로 이러한 특성을 구현하는 방법과 고유한 특성을 만드는 방법을 다룰 것입니다. `derive` 이외의 많은 특성도 있습니다. 자세한 정보는 [Rust 참조 문서의 `속성` 섹션을](https://doc.rust-lang.org/reference/attributes.html) 참조하세요 .

우리의 `area` 함수는 매우 구체적입니다. 직사각형의 면적만 계산합니다. 다른 유형에서는 작동하지 않으므로 이 동작을 `Rectangle` 구조체에 더 밀접하게 연결하는 것이 도움이 될 것입니다. `area` 함수를, `Rectangle` 유형에 정의된 `area` *메서드* 로 전환하여 이 코드를 계속 리팩터링하는 방법을 살펴보겠습니다.