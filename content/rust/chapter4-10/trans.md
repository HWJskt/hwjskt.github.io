

+++
title = "4-10 translation"
weight = 1
template ="book_page_rust.html"

+++



 


# [소유권 이해](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html#understanding-ownership)

소유권은 Rust의 가장 고유한 기능이며 나머지 언어에 깊은 영향을 미칩니다. 이를 통해 Rust는 가비지 수집기가 필요 없이 메모리 안전을 보장할 수 있으므로 소유권이 작동하는 방식을 이해하는 것이 중요합니다. 이 장에서 우리는 차용, 슬라이스, Rust가 메모리에 데이터를 배치하는 방법과 같은 몇 가지 관련 기능뿐만 아니라 소유권에 대해 이야기할 것입니다.

------

## [소유권이란 무엇입니까?](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#what-is-ownership)

*소유권은* Rust 프로그램이 메모리를 관리하는 방식을 지배하는 일련의 규칙입니다. 모든 프로그램은 실행하는 동안 컴퓨터의 메모리를 사용하는 방식을 관리해야 합니다. 일부 언어에는 프로그램이 실행될 때 더 이상 사용되지 않는 메모리를 정기적으로 찾는 가비지 수집 기능이 있습니다. 다른 언어에서는 프로그래머가 명시적으로 메모리를 할당하고 해제해야 합니다. Rust는 세 번째 접근 방식을 사용합니다. 메모리는 컴파일러가 확인하는 일련의 규칙을 사용하여 소유권 시스템을 통해 관리됩니다. 규칙 중 하나라도 위반되면 프로그램이 컴파일되지 않습니다. 소유권의 어떤 기능도 프로그램이 실행되는 동안 느려지지 않습니다.

소유권은 많은 프로그래머에게 새로운 개념이기 때문에 익숙해지는 데 시간이 걸립니다. 좋은 소식은 Rust와 소유권 시스템의 규칙에 대한 경험이 많을수록 안전하고 효율적인 코드를 자연스럽게 개발하는 것이 더 쉬워진다는 것입니다. 견디어 내다!

소유권을 이해하면 Rust를 고유하게 만드는 기능을 이해하기 위한 견고한 기반을 갖게 됩니다. 이 장에서는 매우 일반적인 데이터 구조인 문자열에 초점을 맞춘 몇 가지 예제를 통해 소유권을 배웁니다.

> ### [스택과 힙](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap)
>
> 많은 프로그래밍 언어에서는 스택과 힙에 대해 자주 생각할 필요가 없습니다. 그러나 Rust와 같은 시스템 프로그래밍 언어에서는 값이 스택에 있는지 힙에 있는지에 따라 언어의 작동 방식과 특정 결정을 내려야 하는 이유가 달라집니다. 소유권의 일부는 이 장의 뒷부분에서 스택과 힙과 관련하여 설명할 것이므로 준비를 위해 간단히 설명합니다.
>
> 스택과 힙은 모두 코드에서 런타임에 사용할 수 있는 메모리의 일부이지만 서로 다른 방식으로 구성됩니다. 스택은 값을 가져온 순서대로 저장하고 반대 순서로 값을 제거합니다. *이것을 last in, first out* 이라고 합니다. 접시 더미를 생각해 보십시오. 접시를 더 추가하면 더미 위에 놓고 접시가 필요하면 위에서 하나를 치웁니다. 중간이나 바닥에서 접시를 추가하거나 제거해도 작동하지 않습니다! 데이터를 추가하는 것을 *스택에 밀어 넣는 것을* , 데이터를 제거하는 것을 *스택에서 팝하는 것을* 말합니다. 스택에 저장된 모든 데이터는 알려지고 고정된 크기를 가져야 합니다. 컴파일 시 크기를 알 수 없거나 크기가 변경될 수 있는 데이터는 대신 힙에 저장해야 합니다.
>
> 힙은 덜 조직적입니다. 힙에 데이터를 넣을 때 일정량의 공간을 요청합니다. 메모리 할당자는 힙에서 충분히 큰 빈 지점을 찾아 사용 중인 것으로 표시하고 해당 위치의 주소인 *포인터를 반환합니다.* 이 프로세스를 *힙에 할당 이라고 하며 때로는 그냥* *할당* 으로 축약됩니다.(스택에 값을 푸시하는 것은 할당으로 간주되지 않습니다). 힙에 대한 포인터는 알려진 고정 크기이므로 포인터를 스택에 저장할 수 있지만 실제 데이터를 원할 때는 포인터를 따라야 합니다. 식당에 앉아 있다고 생각해보세요. 들어갈 때 그룹의 인원수를 말하면 호스트가 모든 사람에게 맞는 빈 테이블을 찾아 그곳으로 안내합니다. 그룹의 누군가가 늦게 오면 그들은 당신을 찾기 위해 당신이 어디에 앉았는지 물어볼 수 있습니다.
>
> 할당자가 새 데이터를 저장할 위치를 검색할 필요가 없기 때문에 스택에 푸시하는 것이 힙에 할당하는 것보다 빠릅니다. 해당 위치는 항상 스택의 맨 위에 있습니다. 상대적으로 힙에 공간을 할당하려면 할당자가 먼저 데이터를 보유할 수 있을 만큼 충분히 큰 공간을 찾은 다음 다음 할당을 준비하기 위해 기록을 수행해야 하기 때문에 더 많은 작업이 필요합니다.
>
> 스택에 있는 데이터에 액세스하려면 포인터를 따라가야 하므로 힙에 있는 데이터에 액세스하는 것이 스택에 있는 데이터에 액세스하는 것보다 느립니다. 최신 프로세서는 메모리에서 덜 점프하면 더 빠릅니다. 비유를 계속해서 많은 테이블에서 주문을 받는 식당의 서버를 생각해 보십시오. 다음 테이블로 이동하기 전에 한 테이블에서 모든 주문을 받는 것이 가장 효율적입니다. 테이블 A에서 주문을 받은 다음 테이블 B에서 주문을 받고, 다시 A에서 주문을 받고, 다시 B에서 주문을 받는 것은 훨씬 더 느린 프로세스입니다. 마찬가지로, 프로세서는 다른 데이터와 멀리 떨어져 있는 데이터(힙에 있을 수 있으므로)보다 가까운 데이터(스택에 있는 데이터)에 대해 작업하는 경우 작업을 더 잘 수행할 수 있습니다.
>
> 코드가 함수를 호출하면 함수에 전달된 값(힙의 데이터에 대한 포인터 포함)과 함수의 로컬 변수가 스택으로 푸시됩니다. 함수가 끝나면 해당 값이 스택에서 제거됩니다.
>
> 코드의 어떤 부분이 힙에서 어떤 데이터를 사용하는지 추적하고, 힙에서 중복 데이터의 양을 최소화하고, 공간이 부족하지 않도록 힙에서 사용하지 않는 데이터를 정리하는 것은 소유권이 해결하는 모든 문제입니다. 소유권을 이해하면 스택과 힙에 대해 자주 생각할 필요가 없지만 소유권의 주요 목적이 힙 데이터를 관리하는 것임을 알면 소유권이 작동하는 방식을 설명하는 데 도움이 될 수 있습니다.

### [소유권 규칙](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ownership-rules)

먼저 소유권 규정을 살펴보겠습니다. 이러한 규칙을 설명하는 예제를 통해 작업할 때 이러한 규칙을 염두에 두십시오.

- Rust의 각 값에는 *소유자가* 있습니다.
- 한 번에 한 명의 소유자만 있을 수 있습니다.
- 소유자가 범위를 벗어나면 값이 삭제됩니다.

### [변수 범위](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variable-scope)

이제 우리는 기본 Rust 구문을 지나쳤으므로 예제에 모든 `fn main() {` 코드를 포함하지 않을 것입니다. 따라서 다음 예제를 수동으로 `main` 함수 안에 넣어야 합니다. . 결과적으로 우리의 예제는 좀 더 간결해지고 상용구 코드가 아닌 실제 세부 사항에 집중할 수 있습니다.

*소유권의 첫 번째 예로서 일부 변수의 범위를* 살펴보겠습니다. 범위는 항목이 유효한 프로그램 내의 범위입니다. 다음 변수를 사용하십시오.

```rust
let s = `hello`;
```

변수 `s`는 문자열 리터럴을 참조하며 문자열 값은 프로그램의 텍스트에 하드코딩됩니다. 변수는 선언된 시점부터 현재 *범위가* 끝날 때까지 유효합니다. 목록 4-1은 변수 `s`가 유효한 위치에 주석이 달린 프로그램을 보여줍니다.

```rust
    {                      // s is not valid here, it’s not yet declared
        let s = `hello`;   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

목록 4-1: 변수와 변수가 유효한 범위

즉, 여기에는 두 가지 중요한 시점이 있습니다.

- `s`가 범위 *에 포함* 되면 유효합니다.
- *범위를 벗어날* 때까지 유효합니다.

이 시점에서 범위 간의 관계와 변수가 유효한 시기는 다른 프로그래밍 언어와 유사합니다. 이제 `문자열` 유형을 도입하여 이러한 이해를 바탕으로 빌드하겠습니다.

### [`문자열` 유형](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-string-type)

소유권 규칙을 설명하려면 3장의 [`데이터 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) 섹션에서 다룬 것보다 더 복잡한 데이터 유형이 필요합니다. 이전에 다룬 유형은 알려진 크기이며 스택에 저장하고 꺼낼 수 있습니다. 코드의 다른 부분이 다른 범위에서 동일한 값을 사용해야 하는 경우 새롭고 독립적인 인스턴스를 만들기 위해 신속하고 간단하게 복사할 수 있습니다. 그러나 우리는 힙에 저장된 데이터를 살펴보고 Rust가 해당 데이터를 정리할 시기를 어떻게 아는지 살펴보고자 합니다. `문자열` 유형이 좋은 예입니다.

소유권과 관련된 `문자열` 부분에 집중하겠습니다. 이러한 측면은 표준 라이브러리에서 제공하든 사용자가 생성하든 관계없이 다른 복합 데이터 유형에도 적용됩니다. [8장](https://doc.rust-lang.org/book/ch08-02-strings.html) 에서 `문자열`에 대해 더 깊이 논의할 것입니다.

우리는 문자열 값이 우리 프로그램에 하드코딩되는 문자열 리터럴을 이미 보았습니다. 문자열 리터럴은 편리하지만 텍스트를 사용하려는 모든 상황에 적합하지는 않습니다. 한 가지 이유는 그것들이 불변이라는 것입니다. 다른 하나는 코드를 작성할 때 모든 문자열 값을 알 수 있는 것은 아니라는 것입니다. 예를 들어 사용자 입력을 받아 저장하려면 어떻게 해야 할까요? 이러한 상황을 위해 Rust에는 두 번째 문자열 유형인 `String`이 있습니다. 이 유형은 힙에 할당된 데이터를 관리하므로 컴파일 시 알 수 없는 양의 텍스트를 저장할 수 있습니다. 다음과 같이 `from` 함수를 사용하여 문자열 리터럴에서 `문자열`을 만들 수 있습니다.

```rust
let s = String::from(`hello`);
```

이중 콜론 `::` 연산자를 사용하면 `string_from`과 같은 일종의 이름을 사용하는 대신 `String` 유형 아래에서 이 특정 `from` 함수의 이름을 지정할 수 있습니다. [이 구문은 5장의 `메서드 구문`](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#method-syntax) 섹션과 7장의 [`모듈 트리에서 항목을 참조하기 위한 경로`](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) 에서 모듈의 네임스페이스에 대해 설명할 때 더 자세히 설명합니다.

이러한 종류의 문자열은 변경될 *수 있습니다* .

```rust
    let mut s = String::from(`hello`);

    s.push_str(`, world!`); // push_str() appends a literal to a String

    println!(`{}`, s); // This will print `hello, world!`
```

차이점은 무엇입니까? `문자열`은 변경할 수 있지만 리터럴은 변경할 수 없는 이유는 무엇입니까? 차이점은 이 두 가지 유형이 메모리를 처리하는 방식입니다.

### [메모리 및 할당](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#memory-and-allocation)

문자열 리터럴의 경우 컴파일 시간에 내용을 알기 때문에 텍스트가 최종 실행 파일에 직접 하드코딩됩니다. 이것이 문자열 리터럴이 빠르고 효율적인 이유입니다. 그러나 이러한 속성은 문자열 리터럴의 불변성에서만 나옵니다. 불행하게도 우리는 컴파일 타임에 크기를 알 수 없고 프로그램을 실행하는 동안 크기가 변경될 수 있는 각 텍스트 조각에 대한 메모리 덩어리를 바이너리에 넣을 수 없습니다.

`문자열` 유형을 사용하여 변경 가능하고 확장 가능한 텍스트 조각을 지원하려면 컴파일 시간에 알 수 없는 내용을 보관할 메모리 양을 힙에 할당해야 합니다. 이는 다음을 의미합니다.

- 런타임 시 메모리 할당자에서 메모리를 요청해야 합니다.
- `문자열` 작업이 완료되면 이 메모리를 할당자에게 반환하는 방법이 필요합니다.

첫 번째 부분은 우리가 수행합니다. `String::from`을 호출하면 해당 구현이 필요한 메모리를 요청합니다. 이것은 프로그래밍 언어에서 거의 보편적입니다.

그러나 두 번째 부분은 다릅니다. *가비지 컬렉터(GC)* 가 있는 언어에서 GC는 더 이상 사용되지 않는 메모리를 추적하고 정리하므로 우리는 그것에 대해 생각할 필요가 없습니다. GC가 없는 대부분의 언어에서 메모리를 더 이상 사용하지 않는 시기를 식별하고 메모리를 요청한 것처럼 명시적으로 해제하는 코드를 호출하는 것은 우리의 책임입니다. 이를 올바르게 수행하는 것은 역사적으로 어려운 프로그래밍 문제였습니다. 잊어버리면 메모리를 낭비하게 됩니다. 너무 일찍 수행하면 유효하지 않은 변수가 생깁니다. 두 번 하면 그것도 버그입니다. 정확히 하나의 `할당`과 정확히 하나의 `자유`를 쌍으로 연결해야 합니다.

Rust는 다른 경로를 취합니다. 메모리를 소유한 변수가 범위를 벗어나면 메모리가 자동으로 반환됩니다. 다음은 문자열 리터럴 대신 `문자열`을 사용하는 Listing 4-1의 범위 예제 버전입니다.

```rust
    {
        let s = String::from(`hello`); // s is valid from this point forward

        // do stuff with s
    }                                  // this scope is now over, and s is no
                                       // longer valid
```

`String`이 할당자에게 필요한 메모리를 반환할 수 있는 자연스러운 지점이 있습니다. `s`가 범위를 벗어날 때입니다. 변수가 범위를 벗어나면 Rust는 우리를 위해 특별한 함수를 호출합니다. 이 함수를 [`drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop) 이라고 하며 `String`의 작성자가 메모리를 반환하는 코드를 넣을 수 있는 곳입니다. Rust는 닫는 중괄호에서 자동으로 `drop`을 호출합니다.

> 참고: C++에서 항목의 수명이 끝날 때 리소스 할당을 취소하는 이 패턴을 *리소스 획득이 초기화(RAII)* 라고도 합니다. Rust의 `드롭` 기능은 RAII 패턴을 사용해 본 적이 있다면 익숙할 것입니다.

이 패턴은 Rust 코드 작성 방식에 지대한 영향을 미칩니다. 지금 당장은 간단해 보일 수 있지만 여러 변수가 힙에 할당한 데이터를 사용하도록 하려는 더 복잡한 상황에서는 코드의 동작이 예상치 못한 것일 수 있습니다. 이제 그러한 상황 중 일부를 살펴보겠습니다.

#### [Move와 상호 작용하는 변수 및 데이터](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move)

여러 변수는 Rust에서 다른 방식으로 동일한 데이터와 상호 작용할 수 있습니다. 목록 4-2에서 정수를 사용하는 예를 살펴보겠습니다.

```rust
    let x = 5;
    let y = x;
```

목록 4-2: 변수 `x`의 정수 값을 `y`에 할당

이것이 무엇을 하는지 짐작할 수 있을 것입니다. `값 `5`를 `x`에 바인딩합니다. 그런 다음 `x`의 값을 복사하여 `y`에 바인딩합니다.” 이제 두 개의 변수 `x`와 `y`가 있고 둘 다 `5`입니다. 정수는 알려진 고정 크기를 가진 단순한 값이고 이 두 `5` 값이 스택에 푸시되기 때문에 이것이 실제로 일어나고 있는 일입니다.

이제 `문자열` 버전을 살펴보겠습니다.

```rust
    let s1 = String::from(`hello`);
    let s2 = s1;
```

이것은 매우 유사해 보이기 때문에 작동 방식이 동일할 것이라고 가정할 수 있습니다. 즉, 두 번째 줄은 `s1`의 값을 복사하여 `s2`에 바인딩합니다. 그러나 이것은 실제로 일어나는 일이 아닙니다.

표지 아래 `문자열`에 무슨 일이 일어나고 있는지 보려면 그림 4-1을 살펴보십시오. `문자열`은 왼쪽에 표시된 것처럼 세 부분으로 구성됩니다. 문자열의 내용을 보유하는 메모리에 대한 포인터, 길이 및 용량입니다. 이 데이터 그룹은 스택에 저장됩니다. 오른쪽에는 내용을 보유하는 힙의 메모리가 있습니다.

![두 개의 테이블: 첫 번째 테이블에는 길이(5), 용량(5) 및 두 번째 테이블의 첫 번째 값에 대한 포인터로 구성된 스택의 s1 표현이 포함됩니다.  두 번째 테이블에는 바이트 단위로 힙에 있는 문자열 데이터의 표현이 포함됩니다.](https://doc.rust-lang.org/book/img/trpl04-01.svg)

그림 4-1: `s1`에 바인딩된 ``hello`` 값을 보유하는 `문자열`의 메모리 표현

길이는 `문자열`의 내용이 현재 사용 중인 메모리 양(바이트)입니다. 용량은 `문자열`이 할당자로부터 받은 총 메모리 양(바이트)입니다. 길이와 용량의 차이는 중요하지만 이 맥락에서는 중요하지 않으므로 지금은 용량을 무시해도 됩니다.

`s1`을 `s2`에 할당하면 `문자열` 데이터가 복사됩니다. 즉, 스택에 있는 포인터, 길이 및 용량을 복사합니다. 포인터가 참조하는 힙에 데이터를 복사하지 않습니다. 즉, 메모리의 데이터 표현은 그림 4-2와 같습니다.

![3개의 테이블: 테이블 s1 및 s2는 각각 스택의 해당 문자열을 나타내며 둘 다 힙의 동일한 문자열 데이터를 가리킵니다.](https://doc.rust-lang.org/book/img/trpl04-02.svg)

그림 4-2: `s1`의 포인터, 길이 및 용량의 복사본이 있는 변수 `s2`의 메모리 표현

표현은 그림 4-3처럼 보이지 *않습니다* . Rust가 대신 힙 데이터도 복사했다면 메모리는 어떻게 생겼을 것입니다. Rust가 이렇게 하면 힙의 데이터가 큰 경우 `s2 = s1` 작업은 런타임 성능 측면에서 매우 비쌀 수 있습니다.

![테이블 4개: s1 및 s2에 대한 스택 데이터를 나타내는 테이블 2개와 각각 힙에 있는 자체 문자열 데이터 복사본을 가리킵니다.](https://doc.rust-lang.org/book/img/trpl04-03.svg)

그림 4-3: Rust가 힙 데이터도 복사한 경우 `s2 = s1`이 무엇을 할 수 있는지에 대한 또 다른 가능성

이전에 우리는 변수가 범위를 벗어나면 Rust가 자동으로 `drop` 함수를 호출하고 해당 변수에 대한 힙 메모리를 정리한다고 말했습니다. 그러나 그림 4-2는 동일한 위치를 가리키는 두 데이터 포인터를 보여줍니다. 이것은 문제입니다. `s2`와 `s1`이 범위를 벗어나면 둘 다 동일한 메모리를 해제하려고 합니다. 이것은 *이중 자유* 오류로 알려져 있으며 이전에 언급한 메모리 안전 버그 중 하나입니다. 메모리를 두 번 해제하면 메모리 손상이 발생하여 잠재적으로 보안 취약성이 발생할 수 있습니다.

메모리 안전을 보장하기 위해 `let s2 = s1;` 줄 다음에 Rust는 `s1`을 더 이상 유효하지 않은 것으로 간주합니다. 따라서 Rust는 `s1`이 범위를 벗어날 때 아무것도 해제할 필요가 없습니다. `s2`가 생성된 후 `s1`을 사용하려고 하면 어떤 일이 발생하는지 확인하십시오. 작동하지 않습니다:

```rust
    let s1 = String::from(`hello`);
    let s2 = s1;

    println!(`{}, world!`, s1);
```

Rust가 무효화된 참조를 사용하지 못하게 하기 때문에 다음과 같은 오류가 발생합니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from(`hello`);
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!(`{}, world!`, s1);
  |                            ^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let s2 = s1.clone();
  |                ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership` due to previous error
```

*다른 언어로 작업하면서 얕은 복사* 와 *깊은 복사라는* 용어를 들어본 적이 있다면 데이터를 복사하지 않고 포인터, 길이, 용량을 복사한다는 개념은 아마도 얕은 복사를 하는 것처럼 들릴 것입니다. 그러나 Rust는 첫 번째 변수도 무효화하기 때문에 얕은 복사본이라고 하는 대신 이동 이라고 *합니다* . 이 예에서는 `s1`이 `s2`로 *이동되었다고* 말합니다. 따라서 실제로 일어나는 일은 그림 4-4에 나와 있습니다.

![3개의 테이블: 테이블 s1 및 s2는 각각 스택의 해당 문자열을 나타내며 둘 다 힙의 동일한 문자열 데이터를 가리킵니다.  s1이 더 이상 유효하지 않기 때문에 테이블 s1이 회색으로 표시됩니다.  s2만 힙 데이터에 액세스하는 데 사용할 수 있습니다.](https://doc.rust-lang.org/book/img/trpl04-04.svg)

그림 4-4: `s1`이 무효화된 후 메모리의 표현

그것은 우리의 문제를 해결합니다! `s2`만 유효하면 범위를 벗어나면 단독으로 메모리를 해제하고 완료됩니다.

추가로 이것이 암시하는 디자인 선택이 있습니다: Rust는 데이터의 `깊은` 복사본을 자동으로 생성하지 않습니다. 따라서 *자동* 복사는 런타임 성능 측면에서 저렴하다고 가정할 수 있습니다.

#### [클론과 상호 작용하는 변수 및 데이터](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone)

스택 데이터뿐만 아니라 `String`의 힙 데이터를 깊이 복사하려는 경우 `clone`이라는 일반적인 방법을 사용할 수 *있습니다* . 5장에서 메서드 구문에 대해 논의하겠지만 메서드는 많은 프로그래밍 언어에서 공통적인 기능이기 때문에 이전에 본 적이 있을 것입니다.

다음은 작동 중인 `복제` 방법의 예입니다.

```rust
    let s1 = String::from(`hello`);
    let s2 = s1.clone();

    println!(`s1 = {}, s2 = {}`, s1, s2);
```

*이것은 잘 작동하며 힙 데이터가 복사* 되는 그림 4-3에 표시된 동작을 명시적으로 생성합니다.

`복제`에 대한 호출을 보면 임의의 코드가 실행되고 있고 해당 코드가 비쌀 수 있음을 알 수 있습니다. 뭔가 다른 일이 벌어지고 있다는 시각적 지표입니다.

#### [스택 전용 데이터: 복사](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy)

우리가 아직 이야기하지 않은 또 다른 주름이 있습니다. 정수를 사용하는 이 코드(목록 4-2에 그 일부가 표시됨)는 작동하고 유효합니다.

```rust
    let x = 5;
    let y = x;

    println!(`x = {}, y = {}`, x, y);
```

그러나 이 코드는 우리가 방금 배운 것과 모순되는 것 같습니다. `복제`에 대한 호출이 없지만 `x`는 여전히 유효하고 `y`로 이동되지 않았습니다.

그 이유는 컴파일 타임에 알려진 크기를 갖는 정수와 같은 유형이 스택에 완전히 저장되기 때문에 실제 값의 복사본을 빠르게 만들 수 있기 때문입니다. 즉, 변수 `y`를 만든 후에 `x`가 유효하지 않도록 할 이유가 없습니다. 즉, 여기에서는 깊은 복사와 얕은 복사 사이에 차이가 없으므로 `복제`라고 부르는 것은 일반적인 얕은 복사와 다른 작업을 수행하지 않으며 생략할 수 있습니다.

[Rust에는 `복사` 특성이라는 특수 주석이 있습니다. 정수와 마찬가지로 스택에 저장된 유형에 배치할 수 있습니다(특성에 대한 자세한 내용은 10장](https://doc.rust-lang.org/book/ch10-02-traits.html) 에서 설명 ). 유형이 `복사` 특성을 구현하는 경우 이를 사용하는 변수는 이동하지 않고 사소하게 복사되어 다른 변수에 할당된 후에도 여전히 유효합니다.

러스트는 유형 또는 그 부분이 `Drop` 특성을 구현한 경우 `Copy`로 유형에 주석을 달지 못하게 합니다. 값이 범위를 벗어나고 해당 유형에 `복사` 주석을 추가할 때 유형에 특별한 일이 발생해야 하는 경우 컴파일 타임 오류가 발생합니다. 특성을 구현하기 위해 유형에 `복사` 주석을 추가하는 방법에 대해 알아보려면 부록 C의 [`파생 가능한 특성`을 참조하세요.](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html)

그렇다면 `복사` 특성을 구현하는 유형은 무엇입니까? 지정된 유형에 대한 설명서를 확인하여 확인할 수 있지만 일반적으로 간단한 스칼라 값 그룹은 `복사`를 구현할 수 있으며 할당이 필요하거나 리소스의 일부 형식은 `복사`를 구현할 수 없습니다. 다음은 `복사`를 구현하는 몇 가지 유형입니다.

- `u32`와 같은 모든 정수 유형.
- 값이 `true` 및 `false`인 부울 유형 `bool`.
- `f64`와 같은 모든 부동 소수점 유형.
- 문자 유형 `char`.
- 튜플(`복사`도 구현하는 유형만 포함하는 경우). 예를 들어 `(i32, i32)`는 `복사`를 구현하지만 `(i32, String)`은 구현하지 않습니다.

### [소유권 및 기능](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ownership-and-functions)

함수에 값을 전달하는 메커니즘은 변수에 값을 할당할 때와 비슷합니다. 변수를 함수에 전달하면 할당과 마찬가지로 이동하거나 복사합니다. 목록 4-3에는 변수가 범위에 들어가고 나가는 위치를 보여주는 몇 가지 주석이 있는 예제가 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let s = String::from(`hello`);  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!(`{}`, some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!(`{}`, some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

목록 4-3: 소유권과 범위가 주석으로 표시된 함수

`takes_ownership`을 호출한 후에 `s`를 사용하려고 하면 Rust에서 컴파일 타임 오류가 발생합니다. 이러한 정적 검사는 실수로부터 우리를 보호합니다. `s` 및 `x`를 사용하는 코드를 `main`에 추가하여 사용할 수 있는 위치와 소유권 규칙에 따라 사용할 수 없는 위치를 확인하십시오.

### [반환 값 및 범위](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#return-values-and-scope)

반환 값은 소유권을 이전할 수도 있습니다. Listing 4-4는 Listing 4-3과 유사한 주석을 사용하여 일부 값을 반환하는 함수의 예를 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from(`hello`);     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from(`yours`); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

Listing 4-4: 반환 값의 소유권 이전

변수의 소유권은 매번 동일한 패턴을 따릅니다. 다른 변수에 값을 할당하면 변수가 이동합니다. 힙에 데이터를 포함하는 변수가 범위를 벗어나면 데이터의 소유권이 다른 변수로 이동되지 않는 한 `삭제`로 값이 정리됩니다.

이것이 작동하는 동안 소유권을 가져간 다음 모든 기능에 대한 소유권을 반환하는 것은 약간 지루합니다. 함수가 값을 사용하지만 소유권은 가지지 않으려면 어떻게 해야 할까요? 반환하려는 함수 본문의 결과 데이터 외에도 다시 사용하려는 경우 전달하는 모든 항목도 다시 전달해야 한다는 점은 매우 성가신 일입니다.

Rust는 Listing 4-5에 표시된 것처럼 튜플을 사용하여 여러 값을 반환할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let s1 = String::from(`hello`);

    let (s2, len) = calculate_length(s1);

    println!(`The length of '{}' is {}.`, s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

목록 4-5: 매개변수 소유권 반환

그러나 이것은 일반적이어야 하는 개념에 대해 너무 많은 의식과 많은 작업입니다. 운 좋게도 Rust에는 소유권을 이전하지 않고 값을 사용하는 참조라는 기능이 *있습니다* .

------

## [참조 및 차용](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing)

Listing 4-5에 있는 튜플 코드의 문제는 호출 함수에 `String`을 반환해야 `calculate_length`를 호출한 후에도 `String`을 계속 사용할 수 있다는 것입니다. `계산_길이`. 대신 `문자열` 값에 대한 참조를 제공할 수 있습니다. 참조 *는* 해당 주소에 저장된 데이터에 액세스하기 위해 따를 수 있는 주소라는 점에서 포인터와 같습니다. 해당 데이터는 다른 변수가 소유합니다. 포인터와 달리 참조는 해당 참조의 수명 동안 특정 유형의 유효한 값을 가리키도록 보장됩니다.

다음은 값의 소유권을 가져오는 대신 개체에 대한 참조를 매개 변수로 포함하는 `calculate_length` 함수를 정의하고 사용하는 방법입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let s1 = String::from(`hello`);

    let len = calculate_length(&s1);

    println!(`The length of '{}' is {}.`, s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

먼저 변수 선언의 모든 튜플 코드와 함수 반환 값이 사라진 것을 확인하십시오. 둘째, `&s1`을 `calculate_length`에 전달하고 정의에서 `String`이 아닌 `&String`을 사용합니다. 이러한 앰퍼샌드는 *참조를* 나타내며 소유권을 갖지 않고 일부 값을 참조할 수 있습니다. 그림 4-5는 이 개념을 보여줍니다.

![세 개의 테이블: s에 대한 테이블에는 s1에 대한 테이블에 대한 포인터만 포함됩니다.  s1에 대한 테이블은 s1에 대한 스택 데이터를 포함하고 힙의 문자열 데이터를 가리킵니다.](https://doc.rust-lang.org/book/img/trpl04-05.svg)

그림 4-5: `String s1`을 가리키는 `&String s` 다이어그램

> 참고: `&`를 사용한 참조의 반대는 역 *참조* 연산자 `*`를 사용하여 수행되는 역참조입니다. 8장에서 역참조 연산자의 일부 사용법을 살펴보고 15장에서 역참조에 대한 자세한 내용을 논의합니다.

여기서 함수 호출을 자세히 살펴보겠습니다.

```rust
    let s1 = String::from(`hello`);

    let len = calculate_length(&s1);
```

`&s1` 구문을 사용하면 `s1` 값을 참조하지만 소유하지는 않는 *참조* 를 만들 수 있습니다. 소유하지 않기 때문에 참조가 사용을 중지해도 가리키는 값은 삭제되지 않습니다.

마찬가지로 함수의 시그니처는 `&`를 사용하여 매개변수 `s`의 유형이 참조임을 나타냅니다. 몇 가지 설명 주석을 추가해 보겠습니다.

```rust
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

변수 `s`가 유효한 범위는 모든 함수 매개 변수의 범위와 동일하지만 `s`는 소유권이 없기 때문에 `s`가 사용 중지될 때 참조가 가리키는 값은 삭제되지 않습니다. 함수에 실제 값 대신 매개 변수로 참조가 있으면 소유권을 가져본 적이 없기 때문에 소유권을 돌려주기 위해 값을 반환할 필요가 없습니다.

*참조 차용* 을 만드는 작업을 호출합니다. 실생활에서와 마찬가지로 사람이 무언가를 소유하고 있으면 그 사람에게서 빌릴 수 있습니다. 끝나면 돌려줘야 합니다. 당신은 그것을 소유하지 않습니다.

그렇다면 빌린 것을 수정하려고 하면 어떻게 될까요? 목록 4-6의 코드를 사용해 보십시오. 스포일러 경고: 작동하지 않습니다!

파일 이름: src/main.rs

```rust
fn main() {
    let s = String::from(`hello`);

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(`, world`);
}
```

목록 4-6: 빌린 값 수정 시도

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
8 |     some_string.push_str(`, world`);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` due to previous error
```

변수가 기본적으로 불변인 것처럼 참조도 마찬가지입니다. 우리는 우리가 참조하는 것을 수정할 수 없습니다.

### [변경 가능한 참조](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#mutable-references)

*목록 4-6의 코드를 수정하여 변경 가능한 참조* 대신 사용하는 몇 가지 작은 조정만으로 빌린 값을 수정할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut s = String::from(`hello`);

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(`, world`);
}
```

먼저 `s`를 `mut`로 변경합니다. 그런 다음 `change` 함수를 호출하는 `&mut s`로 변경 가능한 참조를 만들고 `some_string: &mut String`으로 변경 가능한 참조를 허용하도록 함수 시그니처를 업데이트합니다. 이것은 `변경` 기능이 빌린 값을 변경한다는 것을 매우 분명하게 합니다.

변경 가능한 참조에는 한 가지 큰 제한이 있습니다. 값에 대한 변경 가능한 참조가 있는 경우 해당 값에 대한 다른 참조를 가질 수 없습니다. `s`에 대한 두 개의 변경 가능한 참조를 만들려고 시도하는 이 코드는 실패합니다.

파일 이름: src/main.rs

```rust
    let mut s = String::from(`hello`);

    let r1 = &mut s;
    let r2 = &mut s;

    println!(`{}, {}`, r1, r2);
```

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!(`{}, {}`, r1, r2);
  |                        -- first borrow later used here

For more information about this error, try `rustc --explain E0499`.
error: could not compile `ownership` due to previous error
```

이 오류는 `s`를 한 번에 두 번 이상 가변으로 빌릴 수 없기 때문에 이 코드가 유효하지 않다고 말합니다. 첫 번째 변경 가능한 차용은 `r1`에 있으며 `println!`에서 사용될 때까지 지속되어야 합니다. `r1`로.

동일한 데이터에 대한 여러 가변 참조를 동시에 금지하는 제한은 매우 통제된 방식으로 변형을 허용합니다. 대부분의 언어에서는 원할 때마다 변경할 수 있기 때문에 새로운 Rustacean이 어려움을 겪고 있습니다. 이 제한을 갖는 이점은 Rust가 컴파일 시간에 데이터 경합을 방지할 수 있다는 것입니다. 데이터 *경쟁은* 경쟁 조건과 유사하며 다음 세 가지 동작이 발생할 때 발생합니다.

- 두 개 이상의 포인터가 동시에 동일한 데이터에 액세스합니다.
- 적어도 하나의 포인터가 데이터에 쓰는 데 사용되고 있습니다.
- 데이터에 대한 액세스를 동기화하는 데 사용되는 메커니즘이 없습니다.

데이터 경합은 정의되지 않은 동작을 유발하며 런타임 시 추적하려고 할 때 진단 및 수정이 어려울 수 있습니다. Rust는 데이터 경합으로 코드를 컴파일하는 것을 거부함으로써 이 문제를 방지합니다!

항상 그렇듯이 중괄호를 사용하여 새 스코프를 생성할 수 있으므로 *동시* 참조가 아닌 여러 변경 가능한 참조를 허용할 수 있습니다.

```rust
    let mut s = String::from(`hello`);

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
```

Rust는 변경 가능한 참조와 변경 불가능한 참조를 결합하기 위해 유사한 규칙을 적용합니다. 이 코드는 오류를 발생시킵니다.

```rust
    let mut s = String::from(`hello`);

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!(`{}, {}, and {}`, r1, r2, r3);
```

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!(`{}, {}, and {}`, r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error
```

아휴! *또한* 동일한 값에 대한 불변 참조가 있는 동안에는 가변 참조를 가질 수 없습니다.

불변 참조의 사용자는 값이 갑자기 변경될 것이라고 기대하지 않습니다! 그러나 데이터를 읽기만 하는 사람은 다른 사람의 데이터 읽기에 영향을 줄 수 없기 때문에 여러 불변 참조가 허용됩니다.

참조의 범위는 도입된 위치에서 시작하여 해당 참조가 마지막으로 사용된 시간까지 계속됩니다. 예를 들어, 이 코드는 가변 참조가 도입되기 전에 불변 참조의 마지막 사용인 `println!`이 발생하기 때문에 컴파일됩니다.

```rust
    let mut s = String::from(`hello`);

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!(`{} and {}`, r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!(`{}`, r3);
```

불변 참조 `r1` 및 `r2`의 범위는 `println!` 다음에 끝납니다. 변경 가능한 참조 `r3`이 생성되기 전 마지막으로 사용되는 위치입니다. 이러한 범위는 겹치지 않으므로 이 코드가 허용됩니다. 컴파일러는 범위가 끝나기 전 지점에서 참조가 더 이상 사용되지 않는다는 것을 알 수 있습니다.

차용 오류가 때때로 실망스러울 수 있지만, 잠재적인 버그를 일찍(런타임이 아닌 컴파일 타임에) 지적하고 문제가 있는 곳을 정확히 보여주는 것은 Rust 컴파일러라는 점을 기억하십시오. 그러면 데이터가 생각했던 것과 다른 이유를 추적할 필요가 없습니다.

### [매달린 참조](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#dangling-references)

*포인터가 있는 언어에서는 해당 메모리에 대한 포인터를 유지하면서 일부 메모리를 해제하여 댕글링 포인터* (다른 사람에게 제공되었을 수 있는 메모리의 위치를 참조하는 포인터)를 잘못 생성하기 쉽습니다. 대조적으로 Rust에서 컴파일러는 참조가 댕글링 참조가 되지 않도록 보장합니다. 일부 데이터에 대한 참조가 있는 경우 컴파일러는 데이터에 대한 참조가 범위를 벗어나기 전에 데이터가 범위를 벗어나지 않도록 합니다.

러스트가 어떻게 컴파일 타임 오류를 방지하는지 알아보기 위해 댕글링 참조를 생성해 봅시다:

파일 이름: src/main.rs

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from(`hello`);

    &s
}
```

오류는 다음과 같습니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` due to previous error
```

이 오류 메시지는 아직 다루지 않은 기능인 수명을 나타냅니다. 10장에서 수명에 대해 자세히 논의할 것입니다. 그러나 수명에 대한 부분을 무시하면 메시지에는 이 코드가 문제인 이유에 대한 핵심이 포함되어 있습니다.

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```

`dangle` 코드의 각 단계에서 정확히 어떤 일이 발생하는지 자세히 살펴보겠습니다.

파일 이름: src/main.rs

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from(`hello`); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

`dangle` 내부에 `s`가 생성되기 때문에 `dangle`의 코드가 완료되면 `s`가 할당 해제됩니다. 그러나 우리는 그것에 대한 참조를 반환하려고 했습니다. 이는 이 참조가 잘못된 `문자열`을 가리키고 있음을 의미합니다. 좋지 않아! Rust는 우리가 이것을 하도록 허용하지 않을 것입니다.

여기서 해결책은 `문자열`을 직접 반환하는 것입니다.

```rust
fn no_dangle() -> String {
    let s = String::from(`hello`);

    s
}
```

아무 문제 없이 작동합니다. 소유권이 이동되고 할당이 취소되지 않습니다.

### [참조 규칙](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#the-rules-of-references)

참조에 대해 논의한 내용을 요약해 보겠습니다.

- 주어진 시간에 *하나* 의 가변 참조 *또는* 여러 불변 참조를 가질 수 있습니다.
- 참조는 항상 유효해야 합니다.

다음으로 다른 종류의 참조인 슬라이스를 살펴보겠습니다.

------

## [슬라이스 유형](https://doc.rust-lang.org/book/ch04-03-slices.html#the-slice-type)

*슬라이스를 사용하면* 전체 컬렉션이 아닌 컬렉션의 연속적인 요소 시퀀스를 참조할 수 있습니다. 슬라이스는 일종의 참조이므로 소유권이 없습니다.

여기에 작은 프로그래밍 문제가 있습니다. 공백으로 구분된 단어 문자열을 사용하고 해당 문자열에서 찾은 첫 번째 단어를 반환하는 함수를 작성하세요. 함수가 문자열에서 공백을 찾지 못하면 전체 문자열이 한 단어여야 하므로 전체 문자열이 반환되어야 합니다.

슬라이스가 해결하는 문제를 이해하기 위해 슬라이스를 사용하지 않고 이 함수의 시그니처를 작성하는 방법을 살펴보겠습니다.

```rust
fn first_word(s: &String) -> ?
```

`first_word` 함수는 매개변수로 `&String`을 가집니다. 우리는 소유권을 원하지 않으므로 괜찮습니다. 그러나 우리는 무엇을 반환해야 합니까? 문자열의 *일부* 에 대해 말할 방법이 없습니다. 그러나 공백으로 표시된 단어 끝의 인덱스를 반환할 수 있습니다. Listing 4-7과 같이 시도해 봅시다.

파일 이름: src/main.rs

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

목록 4-7: 바이트 인덱스 값을 `String` 매개변수로 반환하는 `first_word` 함수

요소별로 `문자열` 요소를 살펴보고 값이 공백인지 확인해야 하므로 `as_bytes` 메서드를 사용하여 `문자열`을 바이트 배열로 변환합니다.

```rust
    let bytes = s.as_bytes();
```

다음으로 `iter` 메서드를 사용하여 바이트 배열에 대한 반복자를 만듭니다.

```rust
    for (i, &item) in bytes.iter().enumerate() {
```

[13장](https://doc.rust-lang.org/book/ch13-02-iterators.html) 에서 이터레이터에 대해 더 자세히 논의할 것입니다. 지금은 `iter`가 컬렉션의 각 요소를 반환하는 메서드이고 `enumerate`가 `iter`의 결과를 래핑하고 대신 튜플의 일부로 각 요소를 반환한다는 것을 알아두세요. `enumerate`에서 반환된 튜플의 첫 번째 요소는 인덱스이고 두 번째 요소는 요소에 대한 참조입니다. 지수를 직접 계산하는 것보다 조금 더 편리합니다.

`enumerate` 메서드는 튜플을 반환하기 때문에 패턴을 사용하여 해당 튜플을 분해할 수 있습니다. [6장](https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values) 에서 패턴에 대해 더 논의할 것입니다. `for` 루프에서 튜플의 인덱스에 `i`가 있고 튜플의 단일 바이트에 `&item`이 있는 패턴을 지정합니다. `.iter().enumerate()`에서 요소에 대한 참조를 가져오므로 패턴에서 `&`를 사용합니다.

`for` 루프 내에서 바이트 리터럴 구문을 사용하여 공백을 나타내는 바이트를 검색합니다. 공백을 찾으면 위치를 반환합니다. 그렇지 않으면 `s.len()`을 사용하여 문자열의 길이를 반환합니다.

```rust
        if item == b' ' {
            return i;
        }
    }

    s.len()
```

이제 문자열에서 첫 번째 단어의 끝 인덱스를 찾을 수 있는 방법이 있지만 문제가 있습니다. 자체적으로 `usize`를 반환하지만 `&String` 컨텍스트에서 의미 있는 숫자일 뿐입니다. 즉, `문자열`과는 별개의 값이기 때문에 앞으로도 유효하다는 보장이 없습니다. 목록 4-7의 `first_word` 함수를 사용하는 목록 4-8의 프로그램을 고려하십시오.

파일 이름: src/main.rs

```rust
fn main() {
    let mut s = String::from(`hello world`);

    let word = first_word(&s); // word will get the value 5

    s.clear(); // this empties the String, making it equal to ``

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with. word is now totally invalid!
}
```

목록 4-8: `first_word` 함수 호출 결과 저장 및 `문자열` 내용 변경

이 프로그램은 오류 없이 컴파일되며 `s.clear()`를 호출한 후 `word`를 사용한 경우에도 오류가 발생합니다. `word`는 `s`의 상태와 전혀 연결되어 있지 않기 때문에 `word`는 여전히 값 `5`를 포함합니다. 변수 `s`와 함께 값 `5`를 사용하여 첫 번째 단어를 추출하려고 시도할 수 있지만 `word`에 `5`를 저장한 이후로 `s`의 내용이 변경되었기 때문에 이것은 버그가 됩니다.

`word`의 인덱스가 `s`의 데이터와 동기화되지 않는 것에 대해 걱정하는 것은 지루하고 오류가 발생하기 쉽습니다! `second_word` 함수를 작성하면 이러한 인덱스를 관리하기가 훨씬 더 어려워집니다. 서명은 다음과 같아야 합니다.

```rust
fn second_word(s: &String) -> (usize, usize) {
```

이제 우리는 시작 *및* 종료 인덱스를 추적하고 있으며 특정 상태의 데이터에서 계산되었지만 해당 상태에 전혀 연결되지 않은 더 많은 값을 가지고 있습니다. 동기화를 유지해야 하는 세 개의 관련 없는 변수가 떠다니고 있습니다.

운 좋게도 Rust는 이 문제에 대한 해결책을 가지고 있습니다: 스트링 슬라이스입니다.

### [스트링 슬라이스](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices)

문자열 *조각은* `문자열`의 일부에 대한 참조이며 다음과 같습니다.

```rust
    let s = String::from(`hello world`);

    let hello = &s[0..5];
    let world = &s[6..11];
```

전체 `문자열`에 대한 참조가 아니라 `hello`는 추가 `[0..5]` 비트에 지정된 `문자열` 부분에 대한 참조입니다. `[starting_index..ending_index]`를 지정하여 괄호 안의 범위를 사용하여 슬라이스를 만듭니다. 여기서 `starting_index`는 슬라이스의 첫 번째 위치이고 `ending_index`는 슬라이스의 마지막 위치보다 하나 더 많습니다. 내부적으로 슬라이스 데이터 구조는 `ending_index`에서 `starting_index`를 뺀 값에 해당하는 슬라이스의 시작 위치와 길이를 저장합니다. 따라서 `let world = &s[6..11];`의 경우 `world`는 길이 값이 `5`인 `s`의 인덱스 6에 있는 바이트에 대한 포인터를 포함하는 슬라이스입니다.

그림 4-6은 이를 다이어그램으로 보여줍니다.

![3개의 테이블: 힙의 문자열 데이터 `hello world` 테이블에서 인덱스 0의 바이트를 가리키는 s의 스택 데이터를 나타내는 테이블.  세 번째 테이블은 길이 값이 5이고 힙 데이터 테이블의 바이트 6을 가리키는 슬라이스 세계의 스택 데이터를 나타냅니다.](https://doc.rust-lang.org/book/img/trpl04-06.svg)

그림 4-6: `문자열`의 일부를 참조하는 문자열 슬라이스

Rust의 `..` 범위 구문을 사용하여 인덱스 0에서 시작하려면 두 마침표 앞에 값을 놓을 수 있습니다. 즉, 다음과 같습니다.

```rust
let s = String::from(`hello`);

let slice = &s[0..2];
let slice = &s[..2];
```

마찬가지로 슬라이스에 `문자열`의 마지막 바이트가 포함되어 있으면 후행 숫자를 삭제할 수 있습니다. 즉, 다음과 같이 동일합니다.

```rust
let s = String::from(`hello`);

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

두 값을 모두 삭제하여 전체 문자열의 조각을 가져올 수도 있습니다. 따라서 이들은 동일합니다.

```rust
let s = String::from(`hello`);

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 참고: 문자열 슬라이스 범위 인덱스는 유효한 UTF-8 문자 경계에서 발생해야 합니다. 멀티바이트 문자 중간에 문자열 조각을 만들려고 하면 프로그램이 오류와 함께 종료됩니다. 문자열 조각을 소개하기 위해 이 섹션에서는 ASCII만 가정합니다. UTF-8 처리에 대한 자세한 내용은 8장의 [`UTF-8 인코딩된 텍스트를 문자열로 저장` 섹션에 있습니다.](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings)

이 모든 정보를 염두에 두고 슬라이스를 반환하도록 `first_word`를 다시 작성해 보겠습니다. `문자열 조각`을 나타내는 유형은 `&str`로 작성됩니다.

파일 이름: src/main.rs

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

우리는 Listing 4-7에서 했던 것과 같은 방법으로 단어의 끝에 대한 색인을 얻습니다. 공백의 첫 번째 항목을 찾는 것입니다. 공백을 찾으면 문자열의 시작과 공백의 인덱스를 시작 및 끝 인덱스로 사용하여 문자열 슬라이스를 반환합니다.

이제 `first_word`를 호출하면 기본 데이터에 연결된 단일 값을 반환합니다. 값은 슬라이스의 시작점에 대한 참조와 슬라이스의 요소 수로 구성됩니다.

슬라이스를 반환하는 것은 `second_word` 함수에서도 작동합니다.

```rust
fn second_word(s: &String) -> &str {
```

이제 컴파일러가 `문자열`에 대한 참조가 유효한지 확인하기 때문에 엉망으로 만들기 훨씬 더 어려운 간단한 API가 있습니다. Listing 4-8에 있는 프로그램의 버그를 기억하십니까? 첫 번째 단어의 끝에 인덱스를 얻었지만 문자열을 지워서 인덱스가 유효하지 않게 되었을 때의 버그를 기억하십니까? 해당 코드는 논리적으로 잘못되었지만 즉각적인 오류는 표시되지 않았습니다. 빈 문자열과 함께 첫 번째 단어 색인을 계속 사용하려고 하면 나중에 문제가 나타납니다. 슬라이스는 이 버그를 불가능하게 만들고 코드에 문제가 있음을 훨씬 빨리 알려줍니다. `first_word`의 슬라이스 버전을 사용하면 컴파일 타임 오류가 발생합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut s = String::from(`hello world`);

    let word = first_word(&s);

    s.clear(); // error!

    println!(`the first word is: {}`, word);
}
```

다음은 컴파일러 오류입니다.

```bash
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!(`the first word is: {}`, word);
   |                                       ---- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error
```

무언가에 대한 불변 참조가 있으면 가변 참조도 사용할 수 없다는 차용 규칙을 상기하십시오. `clear`는 `String`을 잘라야 하기 때문에 변경 가능한 참조를 가져와야 합니다. `println!` `clear`에 대한 호출이 `word`의 참조를 사용한 후에는 변경 불가능한 참조가 해당 시점에서 여전히 활성 상태여야 합니다. Rust는 `clear`의 변경 가능한 참조와 `word`의 변경 불가능한 참조가 동시에 존재하는 것을 허용하지 않으며 컴파일이 실패합니다. Rust는 API를 사용하기 쉽게 만들었을 뿐만 아니라 컴파일 시간에 전체 오류 클래스를 제거했습니다!

#### [슬라이스로서의 문자열 리터럴](https://doc.rust-lang.org/book/ch04-03-slices.html#string-literals-as-slices)

바이너리 내부에 저장되는 문자열 리터럴에 대해 이야기했던 것을 기억하십시오. 이제 슬라이스에 대해 알았으므로 문자열 리터럴을 제대로 이해할 수 있습니다.

```rust
let s = `Hello, world!`;
```

여기서 `s`의 유형은 `&str`입니다. 바이너리의 특정 지점을 가리키는 슬라이스입니다. 이것이 문자열 리터럴이 불변인 이유이기도 합니다. `&str`은 변경할 수 없는 참조입니다.

#### [문자열 조각을 매개변수로](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices-as-parameters)

리터럴과 `문자열` 값의 조각을 사용할 수 있다는 사실을 알면 `first_word`에서 한 가지 더 개선할 수 있으며 이것이 시그니처입니다.

```rust
fn first_word(s: &String) -> &str {
```

더 경험이 많은 Rustacean은 `&String` 값과 `&str` 값 모두에 대해 동일한 함수를 사용할 수 있기 때문에 목록 4-9에 표시된 서명을 대신 작성할 것입니다.

```rust
fn first_word(s: &str) -> &str {
```

Listing 4-9: `s` 매개변수 유형에 문자열 슬라이스를 사용하여 `first_word` 함수 개선

문자열 슬라이스가 있으면 직접 전달할 수 있습니다. `문자열`이 있는 경우 `문자열` 조각이나 `문자열`에 대한 참조를 전달할 수 있습니다. [이러한 유연성은 15장의 `암시적 역참조 강제 변환`](https://doc.rust-lang.org/book/ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods) 섹션 에서 다룰 기능인 *역참조* 강제를 활용합니다.

`문자열`에 대한 참조 대신 문자열 슬라이스를 사용하도록 함수를 정의하면 기능 손실 없이 API가 더 일반적이고 유용해집니다.

파일 이름: src/main.rs

```rust
fn main() {
    let my_string = String::from(`hello world`);

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which are equivalent
    // to whole slices of `String`s
    let word = first_word(&my_string);

    let my_string_literal = `hello world`;

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### [다른 조각](https://doc.rust-lang.org/book/ch04-03-slices.html#other-slices)

상상할 수 있듯이 스트링 슬라이스는 스트링에 따라 다릅니다. 그러나 보다 일반적인 슬라이스 유형도 있습니다. 다음 배열을 고려하십시오.

```rust
let a = [1, 2, 3, 4, 5];
```

문자열의 일부를 참조하려는 것처럼 배열의 일부를 참조하고 싶을 수도 있습니다. 우리는 이렇게 할 것입니다:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

이 슬라이스의 유형은 `&[i32]`입니다. 첫 번째 요소에 대한 참조와 길이를 저장하여 스트링 슬라이스와 동일한 방식으로 작동합니다. 모든 종류의 다른 컬렉션에 대해 이러한 종류의 슬라이스를 사용하게 됩니다. 8장에서 벡터에 대해 이야기할 때 이러한 모음에 대해 자세히 논의할 것입니다.

## [요약](https://doc.rust-lang.org/book/ch04-03-slices.html#summary)

소유권, 차용 및 슬라이스의 개념은 컴파일 타임에 Rust 프로그램의 메모리 안전을 보장합니다. Rust 언어는 다른 시스템 프로그래밍 언어와 같은 방식으로 메모리 사용을 제어할 수 있지만, 데이터 소유자가 범위를 벗어날 때 데이터 소유자가 해당 데이터를 자동으로 정리하면 추가 코드를 작성하고 디버깅할 필요가 없습니다. 이 컨트롤을 얻으려면.

소유권은 Rust의 다른 많은 부분이 작동하는 방식에 영향을 미치므로 책의 나머지 부분에서 이러한 개념에 대해 더 자세히 이야기할 것입니다. 5장으로 이동하여 `구조체`에서 데이터 조각을 함께 그룹화하는 방법을 살펴보겠습니다.

------

# [구조체를 사용하여 관련 데이터 구조화](https://doc.rust-lang.org/book/ch05-00-structs.html#using-structs-to-structure-related-data)

struct 또는 *structure 는 함께 패키징하고 의미 있는 그룹을 구성하는 여러 관련 값의* *이름* 을 지정할 수 있는 사용자 정의 데이터 유형입니다. 객체 지향 언어에 익숙하다면 *구조체는* 객체의 데이터 속성과 같습니다. 이 장에서는 튜플과 구조체를 비교 및 대조하여 이미 알고 있는 내용을 기반으로 구축하고 구조체가 데이터를 그룹화하는 더 좋은 방법인 경우를 보여줍니다.

구조체를 정의하고 인스턴스화하는 방법을 보여줍니다. 구조체 유형과 관련된 동작을 지정하기 위해 관련 함수, 특히 *methods* 라는 관련 함수를 정의하는 방법에 대해 설명합니다. 구조체와 열거형(6장에서 논의)은 Rust의 컴파일 시간 유형 검사를 최대한 활용하기 위해 프로그램 도메인에서 새 유형을 생성하기 위한 빌딩 블록입니다.

------

## [구조체 정의 및 인스턴스화](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#defining-and-instantiating-structs)

[구조체는 `튜플 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션 에서 논의한 튜플과 유사하며 둘 다 여러 관련 값을 보유합니다. 튜플과 마찬가지로 구조체의 조각은 다른 유형일 수 있습니다. 튜플과 달리 구조체에서는 값이 의미하는 바가 명확하도록 각 데이터 조각의 이름을 지정합니다. 이러한 이름을 추가한다는 것은 구조체가 튜플보다 더 유연하다는 것을 의미합니다. 인스턴스 값을 지정하거나 액세스하기 위해 데이터 순서에 의존할 필요가 없습니다.

구조체를 정의하려면 키워드 `struct`를 입력하고 전체 구조체의 이름을 지정합니다. 구조체의 이름은 함께 그룹화되는 데이터 조각의 중요성을 설명해야 합니다. 그런 다음 중괄호 안에 필드라고 하는 데이터 조각의 이름과 유형을 정의 *합니다* . 예를 들어 Listing 5-1은 사용자 계정에 대한 정보를 저장하는 구조체를 보여줍니다.

파일 이름: src/main.rs

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

목록 5-1: `사용자` 구조체 정의

구조체를 정의한 후 사용하려면 각 필드에 구체적인 값을 지정하여 해당 구조체의 *인스턴스를 만듭니다.* 구조체의 이름을 명시하여 인스턴스를 생성한 다음 *key: value* 쌍을 포함하는 중괄호를 추가합니다. 여기서 키는 필드의 이름이고 값은 해당 필드에 저장하려는 데이터입니다. 구조체에서 선언한 것과 동일한 순서로 필드를 지정할 필요가 없습니다. 즉, 구조체 정의는 유형에 대한 일반적인 템플릿과 같으며 인스턴스는 유형의 값을 생성하기 위해 특정 데이터로 해당 템플릿을 채웁니다. 예를 들어 Listing 5-2와 같이 특정 사용자를 선언할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from(`someusername123`),
        email: String::from(`someone@example.com`),
        sign_in_count: 1,
    };
}
```

목록 5-2: `User` 구조체의 인스턴스 만들기

구조체에서 특정 값을 얻으려면 점 표기법을 사용합니다. 예를 들어 이 사용자의 이메일 주소에 액세스하려면 `user1.email`을 사용합니다. 인스턴스가 변경 가능한 경우 점 표기법을 사용하고 특정 필드에 할당하여 값을 변경할 수 있습니다. Listing 5-3은 변경 가능한 `User` 인스턴스의 `email` 필드 값을 변경하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from(`someusername123`),
        email: String::from(`someone@example.com`),
        sign_in_count: 1,
    };

    user1.email = String::from(`anotheremail@example.com`);
}
```

Listing 5-3: `User` 인스턴스의 `email` 필드 값 변경

전체 인스턴스는 변경 가능해야 합니다. Rust는 특정 필드만 가변으로 표시하는 것을 허용하지 않습니다. 모든 표현식과 마찬가지로 구조체의 새 인스턴스를 함수 본문의 마지막 표현식으로 구성하여 해당 새 인스턴스를 암시적으로 반환할 수 있습니다.

Listing 5-4는 주어진 이메일과 사용자 이름으로 `User` 인스턴스를 반환하는 `build_user` 함수를 보여줍니다. `활성` 필드는 `true` 값을 가져오고 `sign_in_count`는 `1` 값을 가져옵니다.

파일 이름: src/main.rs

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

Listing 5-4: 이메일과 사용자 이름을 받아 `User` 인스턴스를 반환하는 `build_user` 함수

구조체 필드와 같은 이름으로 함수 매개변수의 이름을 지정하는 것은 의미가 있지만 `email` 및 `username` 필드 이름과 변수를 반복해야 하는 것은 약간 지루합니다. 구조체에 더 많은 필드가 있는 경우 각 이름을 반복하면 훨씬 더 짜증이 날 것입니다. 다행히도 편리한 속기가 있습니다!

### [필드 초기화 약어 사용](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-the-field-init-shorthand)

목록 5-4에서 매개변수 이름과 구조체 필드 이름이 정확히 동일하기 때문에 *필드 초기화 약식* 구문을 사용하여 `build_user`를 다시 작성할 수 있습니다. 이렇게 하면 정확히 동일하게 작동하지만 `username` 및 Listing 5-5와 같이 `email`.

파일 이름: src/main.rs

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

목록 5-5: `username` 및 `email` 매개변수가 구조체 필드와 이름이 같기 때문에 필드 초기화 약어를 사용하는 `build_user` 함수

여기서는 `email`이라는 필드가 있는 `User` 구조체의 새 인스턴스를 만듭니다. `이메일` 필드의 값을 `build_user` 함수의 `이메일` 매개변수 값으로 설정하려고 합니다. `email` 필드와 `email` 매개변수의 이름이 같기 때문에 `email: email`이 아닌 `email`만 작성하면 됩니다.

### [Struct Update 구문을 사용하여 다른 인스턴스에서 인스턴스 생성](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax)

다른 인스턴스의 값 대부분을 포함하지만 일부는 변경하는 구조체의 새 인스턴스를 만드는 것이 종종 유용합니다. *구조체 업데이트 구문을* 사용하여 이 작업을 수행할 수 있습니다.

먼저 Listing 5-6에서 업데이트 구문 없이 정기적으로 `user2`에 새 `User` 인스턴스를 생성하는 방법을 보여줍니다. `email`에 새 값을 설정하지만 그 외에는 Listing 5-2에서 생성한 `user1`의 동일한 값을 사용합니다.

파일 이름: src/main.rs

```rust
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from(`another@example.com`),
        sign_in_count: user1.sign_in_count,
    };
}
```

목록 5-6: `user1`의 값 중 하나를 사용하여 새 `User` 인스턴스 만들기

구조체 업데이트 구문을 사용하면 Listing 5-7과 같이 더 적은 코드로 동일한 효과를 얻을 수 있습니다. 구문 `..`은 명시적으로 설정되지 않은 나머지 필드가 지정된 인스턴스의 필드와 동일한 값을 갖도록 지정합니다.

파일 이름: src/main.rs

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from(`another@example.com`),
        ..user1
    };
}
```

목록 5-7: 구조체 업데이트 구문을 사용하여 `User` 인스턴스에 대한 새 `email` 값을 설정하지만 `user1`의 나머지 값을 사용

목록 5-7의 코드는 또한 `이메일`에 대해 다른 값을 갖지만 `user1`의 `username`, `active` 및 `sign_in_count` 필드에 대해 동일한 값을 갖는 `user2`에 인스턴스를 생성합니다. `..user1`은 나머지 필드가 `user1`의 해당 필드에서 값을 가져와야 함을 지정하기 위해 마지막에 와야 하지만 순서에 상관없이 원하는 만큼 많은 필드에 대한 값을 지정하도록 선택할 수 있습니다. 구조체 정의에 있는 필드의

구조체 업데이트 구문은 할당처럼 `=`를 사용합니다. [`이동과 상호 작용하는 변수 및 데이터`](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move) 섹션 에서 본 것처럼 데이터를 이동하기 때문입니다. 이 예제에서는 `user1`의 `username` 필드에 있는 `String`이 `user2`로 이동되었기 때문에 `user2`를 생성한 후 더 이상 `user1`을 전체적으로 사용할 수 없습니다. `user2`에 `email`과 `username` 모두에 대해 새 `String` 값을 지정하여 `user1`에서 `active` 및 `sign_in_count` 값만 사용한 경우 `user1`은 생성 후에도 여전히 유효합니다. `사용자2`. `활성` 및 `sign_in_count`는 모두 `복사` 특성을 구현하는 유형입니다.

### [명명된 필드 없이 튜플 구조체를 사용하여 다른 유형 만들기](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types)

*Rust는 또한 튜플 구조체* 라고 하는 튜플과 비슷하게 보이는 구조체를 지원합니다. 튜플 구조체에는 구조체 이름이 제공하는 추가 의미가 있지만 해당 필드와 연결된 이름은 없습니다. 오히려 그들은 단지 필드의 유형을 가지고 있습니다. 튜플 구조체는 전체 튜플에 이름을 지정하고 튜플을 다른 튜플과 다른 유형으로 만들고 싶을 때 그리고 일반 구조체에서와 같이 각 필드의 이름을 지정하는 것이 장황하거나 중복될 때 유용합니다.

튜플 구조체를 정의하려면 `struct` 키워드로 시작하고 구조체 이름 뒤에 튜플의 유형이 옵니다. 예를 들어 여기에서는 `Color`와 `Point`라는 두 개의 튜플 구조체를 정의하고 사용합니다.

파일 이름: src/main.rs

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

`검은색` 및 `원점` 값은 서로 다른 튜플 구조체의 인스턴스이기 때문에 서로 다른 유형입니다. 정의하는 각 구조체는 구조체 내의 필드가 동일한 유형을 가질 수 있더라도 고유한 유형입니다. 예를 들어 `색상` 유형의 매개변수를 사용하는 함수는 두 유형 모두 세 개의 `i32` 값으로 구성되어 있어도 `점`을 인수로 사용할 수 없습니다. 그렇지 않으면 튜플 구조체 인스턴스는 개별 조각으로 분해할 수 있고 `.`를 사용할 수 있다는 점에서 튜플과 유사합니다. 개별 값에 액세스하기 위한 인덱스가 뒤따릅니다.

### [필드가 없는 단위 유사 구조체](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#unit-like-structs-without-any-fields)

필드가 없는 구조체를 정의할 수도 있습니다! [이들은 `튜플 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션 에서 언급한 단위 유형인 `()`와 유사하게 동작하기 때문에 *단위 유사 구조체* 라고 합니다. 단위와 같은 구조체는 일부 유형에 특성을 구현해야 하지만 유형 자체에 저장하려는 데이터가 없을 때 유용할 수 있습니다. 특성에 대해서는 10장에서 논의할 것입니다. 다음은 `AlwaysEqual`이라는 단위 구조체를 선언하고 인스턴스화하는 예입니다.

파일 이름: src/main.rs

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

`AlwaysEqual`을 정의하려면 `struct` 키워드, 원하는 이름, 세미콜론을 사용합니다. 중괄호나 괄호가 필요하지 않습니다! 그런 다음 유사한 방식으로 `subject` 변수에서 `AlwaysEqual` 인스턴스를 얻을 수 있습니다. 중괄호나 괄호 없이 정의한 이름을 사용합니다. 나중에 `AlwaysEqual`의 모든 인스턴스가 항상 다른 유형의 모든 인스턴스와 동일하도록 이 유형에 대한 동작을 구현하여 테스트 목적으로 알려진 결과를 가질 것이라고 상상해 보십시오. 해당 동작을 구현하는 데 데이터가 필요하지 않습니다! 10장에서 특성을 정의하고 유닛과 같은 구조체를 포함하여 모든 유형에서 구현하는 방법을 볼 수 있습니다.

> ### [구조체 데이터의 소유권](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#ownership-of-struct-data)
>
> Listing 5-1의 `User` 구조체 정의에서 우리는 `&str` 문자열 슬라이스 유형이 아닌 소유된 `String` 유형을 사용했습니다. 이것은 이 구조체의 각 인스턴스가 모든 데이터를 소유하고 전체 구조체가 유효한 한 해당 데이터가 유효하기를 원하기 때문에 의도적인 선택입니다.
>
> 구조체가 다른 것이 소유한 데이터에 대한 참조를 저장하는 것도 가능하지만 그렇게 하려면 10장에서 논의할 Rust 기능인 lifetimes 를 *사용해야* 합니다. 수명은 구조체가 참조하는 데이터가 다음과 같이 유효한지 확인합니다. 구조체가 있는 한. 다음과 같이 수명을 지정하지 않고 구조체에 참조를 저장하려고 한다고 가정해 보겠습니다. 이것은 작동하지 않습니다:
>
> 파일 이름: src/main.rs
>
> ```rust
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
> 
> fn main() {
>     let user1 = User {
>         active: true,
>         username: `someusername123`,
>         email: `someone@example.com`,
>         sign_in_count: 1,
>     };
> }
> ```
>
> 컴파일러는 수명 지정자가 필요하다고 불평합니다.
>
> ```bash
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
> 
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
> 
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` due to 2 previous errors
> ```
>
> 10장에서 구조체에 참조를 저장할 수 있도록 이러한 오류를 수정하는 방법에 대해 논의하지만 지금은 `&str`과 같은 참조 대신 `String`과 같은 소유 유형을 사용하여 이러한 오류를 수정합니다.

------

## [구조체를 사용한 예제 프로그램](https://doc.rust-lang.org/book/ch05-02-example-structs.html#an-example-program-using-structs)

언제 구조체를 사용해야 하는지 이해하기 위해 직사각형의 면적을 계산하는 프로그램을 작성해 봅시다. 단일 변수를 사용하여 시작한 다음 구조체를 대신 사용할 때까지 프로그램을 리팩터링합니다.

픽셀로 지정된 직사각형의 너비와 높이를 가져오고 직사각형의 면적을 계산하는 *직사각형* 이라는 Cargo로 새로운 바이너리 프로젝트를 만들어 봅시다. *Listing 5-8은 우리 프로젝트의 src/main.rs* 에서 정확히 그것을 수행하는 한 가지 방법이 있는 짧은 프로그램을 보여줍니다.

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

이 코드의 문제는 `area`의 서명에서 분명합니다.

```rust
fn area(width: u32, height: u32) -> u32 {
```

`area` 함수는 하나의 직사각형의 면적을 계산하기로 되어 있지만, 우리가 작성한 함수에는 두 개의 매개변수가 있고 매개변수가 관련되어 있다는 것이 우리 프로그램의 어느 곳에서도 명확하지 않습니다. 너비와 높이를 함께 그룹화하는 것이 더 읽기 쉽고 관리하기 쉽습니다. [우리는 이미 3장의 `튜플 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션에서 튜플을 사용하여 이를 수행할 수 있는 한 가지 방법에 대해 논의했습니다.

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

어떤 면에서는 이 프로그램이 더 좋습니다. 튜플을 사용하면 약간의 구조를 추가할 수 있으며 이제 하나의 인수만 전달합니다. 그러나 또 다른 방식으로 이 버전은 덜 명확합니다. 튜플은 요소의 이름을 지정하지 않으므로 튜플의 일부를 인덱싱해야 하므로 계산이 덜 명확해집니다.

너비와 높이를 섞는 것은 면적 계산에 문제가 되지 않지만 화면에 사각형을 그리려면 중요합니다! `너비`는 튜플 인덱스 `0`이고 `높이`는 튜플 인덱스 `1`이라는 점을 명심해야 합니다. 이것은 다른 사람이 우리 코드를 사용한다면 알아내고 기억하기가 훨씬 더 어려울 것입니다. 코드에서 데이터의 의미를 전달하지 않았기 때문에 이제 오류를 도입하기가 더 쉬워졌습니다.

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

여기서 우리는 구조체를 정의하고 이름을 `Rectangle`로 지정했습니다. 중괄호 안에 `너비` 및 `높이` 필드를 정의했으며 둘 다 `u32` 유형을 가집니다. 그런 다음 `main`에서 너비가 `30`이고 높이가 `50`인 `Rectangle`의 특정 인스턴스를 만들었습니다.

우리의 `area` 함수는 이제 `rectangle`이라는 이름의 매개변수 하나를 사용하여 정의되며, 그 유형은 `Rectangle` 구조체 인스턴스의 불변 차용입니다. 4장에서 언급했듯이 소유권을 갖기보다 구조체를 빌리고 싶습니다. 이런 식으로 `main`은 소유권을 유지하고 `rect1`을 계속 사용할 수 있습니다. 이것이 우리가 함수 서명에서 `&`를 사용하고 함수를 호출하는 이유입니다.

`area` 함수는 `Rectangle` 인스턴스의 `width` 및 `height` 필드에 액세스합니다(차용한 구조체 인스턴스의 필드에 액세스해도 필드 값이 이동하지 않으므로 구조체 차용을 자주 볼 수 있습니다). `면적`에 대한 함수 서명은 이제 정확히 `너비` 및 `높이` 필드를 사용하여 `직사각형`의 면적을 계산합니다. 이것은 너비와 높이가 서로 관련되어 있음을 전달하고 `0`과 `1`의 튜플 인덱스 값을 사용하는 대신 값에 설명적인 이름을 부여합니다. 이것은 명확성을 위한 승리입니다.

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

```text
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
```

`println!` 매크로는 많은 종류의 서식을 지정할 수 있으며 기본적으로 중괄호는 `println!` `디스플레이`로 알려진 형식 사용: 최종 사용자가 직접 사용하기 위한 출력. 지금까지 본 기본 유형은 기본적으로 `디스플레이`를 구현합니다. 사용자에게 `1` 또는 다른 기본 유형을 표시하려는 방법은 한 가지뿐이기 때문입니다. 그러나 구조체를 사용하면 `println!` 더 많은 표시 가능성이 있기 때문에 출력 형식이 명확하지 않습니다. 쉼표를 원하십니까? 중괄호를 인쇄하시겠습니까? 모든 필드를 표시해야 합니까? 이러한 모호성으로 인해 Rust는 우리가 원하는 것을 추측하려고 시도하지 않으며 구조체에는 `println!`과 함께 사용할 `Display` 구현이 제공되지 않습니다. 및 `{}` 자리 표시자.

오류를 계속 읽으면 다음과 같은 유용한 메모를 찾을 수 있습니다.

```text
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
```

해 보자! `println!` 매크로 호출은 이제 `println!(`rect1 is {:?}`, rect1);`처럼 보입니다. 지정자 `:?` 넣기 중괄호 안에 `println!` `디버그`라는 출력 형식을 사용하려고 합니다. `디버그` 특성을 사용하면 개발자에게 유용한 방식으로 구조체를 인쇄할 수 있으므로 코드를 디버깅하는 동안 해당 값을 볼 수 있습니다.

이 변경으로 코드를 컴파일합니다. 드랏! 여전히 오류가 발생합니다.

```text
error[E0277]: `Rectangle` doesn't implement `Debug`
```

그러나 다시 컴파일러는 다음과 같은 유용한 정보를 제공합니다.

```text
   = help: the trait `Debug` is not implemented for `Rectangle`
   = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
```

Rust에는 디버깅 정보를 출력하는 기능이 포함 *되어* 있지만 구조체에서 해당 기능을 사용할 수 있도록 명시적으로 선택해야 합니다. 이를 위해 Listing 5-12에 표시된 것처럼 구조체 정의 바로 앞에 외부 속성 `#[derive(Debug)]`를 추가합니다.

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

멋진! 가장 예쁜 출력은 아니지만 이 인스턴스에 대한 모든 필드의 값을 표시하므로 디버깅 중에 확실히 도움이 됩니다. 더 큰 구조체가 있는 경우 좀 더 읽기 쉬운 출력을 갖는 것이 유용합니다. 이 경우 `println!`에서 `{:?}` 대신 `{:#?}`를 사용할 수 있습니다. 끈. 이 예에서 `{:#?}` 스타일을 사용하면 다음이 출력됩니다.

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

`디버그` 형식을 사용하여 값을 출력하는 또 다른 방법은 [`dbg!` 식의 소유권을 갖는 매크로](https://doc.rust-lang.org/std/macro.dbg.html) (참조를 취하는 `println!`와 반대)는 `dbg!` 매크로 호출은 해당 식의 결과 값과 함께 코드에서 발생하고 값의 소유권을 반환합니다.

> 참고: `dbg!` 매크로는 표준 출력 콘솔 스트림(`stdout`)에 인쇄하는 `println!`과 달리 표준 오류 콘솔 스트림(`stderr`)에 인쇄합니다. [12장의 `표준 출력 대신 표준 오류에 오류 메시지 쓰기` 섹션에서](https://doc.rust-lang.org/book/ch12-06-writing-to-stderr-instead-of-stdout.html) `stderr` 및 `stdout`에 대해 자세히 설명합니다.

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
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

`dbg!`를 넣을 수 있습니다. `30 * scale`이라는 표현과 `dbg!` 표현식 값의 소유권을 반환하면 `width` 필드는 `dbg!` 거기에 전화하십시오. 우리는 `dbg!`를 원하지 않습니다. `rect1`의 소유권을 가져오므로 다음 호출에서 `rect1`에 대한 참조를 사용합니다. 이 예제의 출력은 다음과 같습니다.

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

첫 번째 출력 비트는 *src/main.rs* 라인 10에서 `30 * scale` 표현식을 디버깅하고 있으며 결과 값은 `60`입니다(정수에 대해 구현된 `디버그` 형식은 값만). `dbg!` *src/main.rs* 의 14번째 줄에 있는 호출은 `Rectangle` 구조체인 `&rect1`의 값을 출력합니다. 이 출력은 `직사각형` 유형의 예쁜 `디버그` 형식을 사용합니다. `dbg!` 매크로는 코드가 수행하는 작업을 파악하려고 할 때 정말 유용할 수 있습니다!

Rust는 `Debug` 특성 외에도 사용자 지정 유형에 유용한 동작을 추가할 수 있는 `derive` 특성과 함께 사용할 여러 특성을 제공했습니다. 이러한 특성과 행동은 [부록 C](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html) 에 나열되어 있습니다. 10장에서 사용자 지정 동작으로 이러한 특성을 구현하는 방법과 고유한 특성을 만드는 방법을 다룰 것입니다. `파생` 이외의 많은 특성도 있습니다. 자세한 정보는 [Rust 참조 문서의 `속성` 섹션을](https://doc.rust-lang.org/reference/attributes.html) 참조하세요 .

우리의 `면적` 함수는 매우 구체적입니다. 직사각형의 면적만 계산합니다. 다른 유형에서는 작동하지 않으므로 이 동작을 `직사각형` 구조체에 더 밀접하게 연결하는 것이 도움이 될 것입니다. `area` 함수를 `Rectangle` 유형에 정의된 `area` *메서드* 로 전환하여 이 코드를 계속 리팩터링하는 방법을 살펴보겠습니다.

------

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

```text
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

------

# [열거형 및 패턴 일치](https://doc.rust-lang.org/book/ch06-00-enums.html#enums-and-pattern-matching)

*이 장에서는 열거* 형 이라고도 하는 열거 *형* 에 대해 살펴보겠습니다. *열거형을 사용하면 가능한 변형을* 열거하여 유형을 정의할 수 있습니다. 먼저 열거형을 정의하고 사용하여 열거형이 데이터와 함께 의미를 인코딩하는 방법을 보여줍니다. 다음으로 `옵션`이라는 특히 유용한 열거형을 살펴보겠습니다. 이 열거형은 값이 무언가가 될 수도 있고 없을 수도 있음을 나타냅니다. 그런 다음 `일치` 식의 패턴 일치를 통해 열거형의 서로 다른 값에 대해 서로 다른 코드를 쉽게 실행할 수 있는 방법을 살펴보겠습니다. 마지막으로 `if let` 구문이 코드에서 열거형을 처리하는 데 사용할 수 있는 또 다른 편리하고 간결한 관용구인 방법을 다룰 것입니다.

------

## [열거형 정의](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#defining-an-enum)

구조체가 `너비`와 `높이`가 있는 `직사각형`과 같이 관련 필드와 데이터를 함께 그룹화하는 방법을 제공하는 경우 열거형은 값이 가능한 값 집합 중 하나라고 말하는 방법을 제공합니다. 예를 들어, `직사각형`은 `원`과 `삼각형`을 포함하는 가능한 도형 세트 중 하나라고 말하고 싶을 수 있습니다. 이를 위해 Rust는 이러한 가능성을 열거형으로 인코딩할 수 있도록 합니다.

코드로 표현하고 싶은 상황을 살펴보고 이 경우 열거형이 구조체보다 유용하고 더 적합한 이유를 살펴보겠습니다. IP 주소로 작업해야 한다고 가정해 보겠습니다. 현재 IP 주소에는 버전 4와 버전 6의 두 가지 주요 표준이 사용됩니다. 이것이 우리 프로그램이 접하게 될 IP 주소에 대한 유일한 가능성이기 때문에 가능한 모든 변형을 *열거* 할 수 있으며 여기에서 열거가 이름을 얻습니다.

모든 IP 주소는 버전 4 또는 버전 6 주소일 수 있지만 동시에 둘 다일 수는 없습니다. IP 주소의 이러한 속성은 enum 값이 변형 중 하나일 수 있기 때문에 enum 데이터 구조를 적절하게 만듭니다. 버전 4 및 버전 6 주소는 여전히 기본적으로 IP 주소이므로 코드가 모든 종류의 IP 주소에 적용되는 상황을 처리할 때 동일한 유형으로 취급되어야 합니다.

`IpAddrKind` 열거형을 정의하고 IP 주소가 될 수 있는 `V4` 및 `V6` 종류를 나열하여 이 개념을 코드로 표현할 수 있습니다. 열거형의 변형은 다음과 같습니다.

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

`IpAddrKind`는 이제 코드의 다른 곳에서 사용할 수 있는 사용자 정의 데이터 유형입니다.

### [열거형 값](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values)

다음과 같이 `IpAddrKind`의 두 변형 각각의 인스턴스를 만들 수 있습니다.

```rust
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

열거형의 변형은 해당 식별자 아래에 네임스페이스가 지정되며 이중 콜론을 사용하여 둘을 구분합니다. 이것은 이제 `IpAddrKind::V4` 및 `IpAddrKind::V6` 값이 모두 `IpAddrKind`와 같이 동일한 유형이기 때문에 유용합니다. 그런 다음 예를 들어 `IpAddrKind`를 사용하는 함수를 정의할 수 있습니다.

```rust
fn route(ip_kind: IpAddrKind) {}
```

그리고 우리는 이 함수를 두 변형으로 호출할 수 있습니다:

```rust
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
```

열거형을 사용하면 더 많은 이점이 있습니다. *IP 주소 유형에 대해 더 생각해 보면 현재 실제 IP 주소 데이터를* 저장할 방법이 없습니다. 우리는 그것이 어떤 *종류* 인지 만 알고 있습니다. 5장에서 구조체에 대해 방금 배웠다면 목록 6-1에 표시된 대로 구조체를 사용하여 이 문제를 해결하고 싶은 유혹을 느낄 수 있습니다.

```rust
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from(`127.0.0.1`),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from(`::1`),
    };
```

목록 6-1: `struct`를 사용하여 데이터 및 IP 주소의 `IpAddrKind` 변형 저장

여기에서 `IpAddrKind` 유형의 `kind` 필드(이전에 정의한 enum)와 `String` 유형의 `address` 필드가 있는 구조체 `IpAddr`를 정의했습니다. 이 구조체에는 두 개의 인스턴스가 있습니다. 첫 번째는 `home`이며 `127.0.0.1`의 관련 주소 데이터와 함께 `종류`로 `IpAddrKind::V4` 값을 가집니다. 두 번째 인스턴스는 `루프백`입니다. `종류` 값인 `V6`으로 `IpAddrKind`의 다른 변형이 있으며 이와 연결된 주소 `::1`이 있습니다. 구조체를 사용하여 `종류` 및 `주소` 값을 함께 묶었으므로 이제 변형이 값과 연결됩니다.

그러나 열거형만 사용하여 동일한 개념을 나타내는 것이 더 간결합니다. 구조체 내부의 열거형보다 데이터를 각 열거형 변형에 직접 넣을 수 있습니다. `IpAddr` 열거형의 이 새로운 정의는 `V4` 및 `V6` 변형 모두 연결된 `문자열` 값을 가질 것이라고 말합니다.

```rust
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from(`127.0.0.1`));

    let loopback = IpAddr::V6(String::from(`::1`));
```

열거형의 각 변형에 직접 데이터를 첨부하므로 추가 구조체가 필요하지 않습니다. 여기에서 열거형의 작동 방식에 대한 또 다른 세부 정보를 더 쉽게 볼 수 있습니다. 우리가 정의하는 각 열거형 변형의 이름도 열거형의 인스턴스를 구성하는 함수가 됩니다. 즉, `IpAddr::V4()`는 `String` 인수를 사용하고 `IpAddr` 유형의 인스턴스를 반환하는 함수 호출입니다. 열거형을 정의한 결과로 정의된 이 생성자 함수를 자동으로 얻습니다.

구조체 대신 열거형을 사용하면 또 다른 이점이 있습니다. 각 변형에는 연결된 데이터의 유형과 양이 다를 수 있습니다. 버전 4 IP 주소에는 항상 0에서 255 사이의 값을 갖는 4개의 숫자 구성 요소가 있습니다. `V4` 주소를 4개의 `u8` 값으로 저장하고 싶지만 `V6` 주소를 하나의 `문자열` 값으로 표현하려면 구조체로는 할 수 없습니다. 열거형은 이 경우를 쉽게 처리합니다.

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from(`::1`));
```

버전 4 및 버전 6 IP 주소를 저장하기 위해 데이터 구조를 정의하는 여러 가지 방법을 보여주었습니다. 그러나 밝혀진 바와 같이 IP 주소를 저장하고 어떤 종류인지 인코딩하려는 것은 너무 일반적이어서 [표준 라이브러리에 우리가 사용할 수 있는 정의가 있습니다! ](https://doc.rust-lang.org/std/net/enum.IpAddr.html)표준 라이브러리가 `IpAddr`을 정의하는 방법을 살펴보겠습니다. 여기에는 우리가 정의하고 사용한 정확한 열거형 및 변형이 있지만 각각에 대해 다르게 정의되는 두 개의 서로 다른 구조체의 형태로 변형 내부에 주소 데이터를 포함합니다. 변종:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

이 코드는 예를 들어 문자열, 숫자 유형 또는 구조체와 같은 열거형 변형 내에 모든 종류의 데이터를 넣을 수 있음을 보여줍니다. 다른 열거형을 포함할 수도 있습니다! 또한 표준 라이브러리 유형은 종종 여러분이 생각하는 것보다 훨씬 더 복잡하지 않습니다.

표준 라이브러리에 `IpAddr`에 대한 정의가 포함되어 있지만 표준 라이브러리의 정의를 범위로 가져오지 않았기 때문에 여전히 충돌 없이 자체 정의를 만들고 사용할 수 있습니다. 유형을 범위로 가져오는 방법에 대해서는 7장에서 자세히 설명합니다.

Listing 6-2에 있는 enum의 또 다른 예를 살펴보겠습니다. 이것은 변형에 다양한 유형이 내장되어 있습니다.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

목록 6-2: 각각 다른 양과 유형의 값을 저장하는 변형이 있는 `메시지` 열거형

이 열거형에는 유형이 다른 네 가지 변형이 있습니다.

- `종료`에는 이와 관련된 데이터가 전혀 없습니다.
- `이동`에는 구조체와 마찬가지로 명명된 필드가 있습니다.
- `쓰기`에는 단일 `문자열`이 포함됩니다.
- `ChangeColor`에는 3개의 `i32` 값이 포함됩니다.

목록 6-2에 있는 것과 같은 변형으로 열거형을 정의하는 것은 열거형이 `struct` 키워드를 사용하지 않고 모든 변형이 `Message` 유형 아래에 함께 그룹화된다는 점을 제외하면 다른 종류의 구조체 정의를 정의하는 것과 유사합니다. 다음 구조체는 이전 열거형 변형이 보유하는 것과 동일한 데이터를 보유할 수 있습니다.

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

그러나 각각 고유한 유형이 있는 서로 다른 구조체를 사용하는 경우 Listing 6-2에 정의된 `Message` 열거형으로 할 수 있는 것처럼 이러한 종류의 메시지를 받는 함수를 쉽게 정의할 수 없습니다. 단일 유형입니다.

열거형과 구조체 사이에는 또 다른 유사점이 있습니다. `impl`을 사용하여 구조체에 대한 메서드를 정의할 수 있는 것처럼 열거형에 대한 메서드도 정의할 수 있습니다. 다음은 `Message` 열거형에서 정의할 수 있는 `call`이라는 메서드입니다.

```rust
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from(`hello`));
    m.call();
```

메서드 본문은 `self`를 사용하여 메서드를 호출한 값을 가져옵니다. 이 예에서 우리는 `Message::Write(String::from(`hello`))` 값을 갖는 변수 `m`을 생성했으며 이것이 `self`가 ` `m.call()`이 실행될 때 `메소드를 호출합니다.

매우 일반적이고 유용한 표준 라이브러리의 또 다른 열거형인 `Option`을 살펴보겠습니다.

### [`옵션` 열거형과 Null 값에 대한 이점](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#the-option-enum-and-its-advantages-over-null-values)

이 섹션에서는 표준 라이브러리에서 정의한 또 다른 열거형인 `Option`의 사례 연구를 살펴봅니다. `옵션` 유형은 값이 무언가가 될 수도 있고 아무것도 아닐 수도 있는 매우 일반적인 시나리오를 인코딩합니다.

예를 들어 비어 있지 않은 목록의 첫 번째 항목을 요청하면 값을 받게 됩니다. 빈 목록의 첫 번째 항목을 요청하면 아무것도 얻지 못합니다. 유형 시스템 측면에서 이 개념을 표현하면 처리해야 하는 모든 사례를 처리했는지 여부를 컴파일러에서 확인할 수 있습니다. 이 기능은 다른 프로그래밍 언어에서 매우 일반적인 버그를 방지할 수 있습니다.

프로그래밍 언어 설계는 종종 어떤 기능을 포함하는지에 따라 생각되지만 제외하는 기능도 중요합니다. Rust에는 다른 많은 언어에 있는 null 기능이 없습니다. *Null* 은 값이 없다는 의미의 값입니다. null이 있는 언어에서 변수는 항상 null 또는 null이 아닌 두 가지 상태 중 하나일 수 있습니다.

2009년 프레젠테이션 `Null References: The Billion Dollar Mistake`에서 null의 발명가인 Tony Hoare는 다음과 같이 말했습니다.

> 나는 그것을 나의 10억 달러짜리 실수라고 부른다. 그 당시 저는 객체 지향 언어의 참조를 위한 최초의 포괄적인 유형 시스템을 설계하고 있었습니다. 내 목표는 컴파일러가 자동으로 검사를 수행하여 모든 참조 사용이 절대적으로 안전하도록 하는 것이었습니다. 하지만 구현하기가 너무 쉽기 때문에 null 참조를 넣고 싶은 유혹을 뿌리칠 수 없었습니다. 이로 인해 수많은 오류, 취약성 및 시스템 충돌이 발생했으며 지난 40년 동안 아마도 10억 달러의 고통과 피해를 입혔을 것입니다.

null 값의 문제는 null 값을 null이 아닌 값으로 사용하려고 하면 일종의 오류가 발생한다는 것입니다. 이 null 또는 null이 아닌 속성은 널리 퍼져 있기 때문에 이러한 종류의 오류를 만들기가 매우 쉽습니다.

그러나 null이 표현하려는 개념은 여전히 유용합니다. null은 현재 유효하지 않거나 어떤 이유로 없는 값입니다.

문제는 실제로 개념이 아니라 특정 구현에 있습니다. 따라서 Rust에는 null이 없지만 존재하거나 존재하지 않는 값의 개념을 인코딩할 수 있는 열거형이 있습니다. 이 열거형은 `옵션`이며 [표준 라이브러리에 의해](https://doc.rust-lang.org/std/option/enum.Option.html) 다음과 같이 정의됩니다.

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`옵션` 열거형은 너무 유용해서 서문에 포함되어 있습니다. 명시적으로 범위로 가져올 필요가 없습니다. 해당 변형도 서문에 포함되어 있습니다. `옵션:` 없이 `Some` 및 `None`을 직접 사용할 수 있습니다. :` 접두어입니다. `옵션` 열거형은 여전히 일반 열거형이며 `Some(T)` 및 `None`은 여전히 `Option` 유형의 변형입니다.`.

`` 구문은 우리가 아직 이야기하지 않은 Rust의 기능입니다. 제네릭 유형 매개변수이며, 제네릭에 대해서는 10장에서 자세히 다룰 것입니다. 지금은 ``는 `Option` 열거형의 `Some` 변형이 모든 유형의 데이터 한 조각을 보유할 수 있으며 `T` 대신 사용되는 각 구체적인 유형이 전체 `Option`을 만든다는 것을 의미합니다.` 다른 유형을 입력하십시오. 다음은 `옵션` 값을 사용하여 숫자 유형과 문자열 유형을 보유하는 몇 가지 예입니다.

```rust
    let some_number = Some(5);
    let some_char = Some('e');

    let absent_number: Option<i32> = None;
```

`some_number`의 타입은 `Option`입니다.`. `some_char`의 유형은 `Option`, 이는 다른 유형입니다. `Some` 변형 내부에 값을 지정했기 때문에 Rust는 이러한 유형을 추론할 수 있습니다. `absent_number`의 경우 Rust는 전체 `Option` 유형에 주석을 달도록 요구합니다. 컴파일러는 추론할 수 없습니다. 해당 `Some` 변종이 `None` 값만 보고 보유할 유형 여기서 우리는 `absent_number`가 `Option` 유형임을 의미한다고 Rust에 알립니다.`.

`일부` 값이 있을 때 값이 존재하고 그 값이 `일부` 내에 있음을 알 수 있습니다. `None` 값이 있는 경우 어떤 의미에서는 null과 같은 의미입니다. 즉, 유효한 값이 없습니다. 그렇다면 왜 `옵션` null을 갖는 것보다 낫습니까?

요컨대, `옵션`와 `T`(여기서 `T`는 모든 유형일 수 있음)가 서로 다른 유형이므로 컴파일러는 `옵션`을 사용하도록 허용하지 않습니다.` 값이 확실히 유효한 값인 것처럼. 예를 들어 이 코드는 `i8`을 `Option`에 추가하려고 하기 때문에 컴파일되지 않습니다.`:

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

이 코드를 실행하면 다음과 같은 오류 메시지가 나타납니다.

```bash
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  = help: the following other types implement trait `Add<Rhs>`:
            <&'a i8 as Add<i8>>
            <&i8 as Add<&i8>>
            <i8 as Add<&i8>>
            <i8 as Add>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` due to previous error
```

극심한! 사실상 이 오류 메시지는 Rust가 `i8`과 `옵션`을 추가하는 방법을 이해하지 못한다는 것을 의미합니다.서로 다른 유형이기 때문입니다. Rust에서 `i8`과 같은 유형의 값이 있을 때 컴파일러는 항상 유효한 값을 가지고 있는지 확인합니다. 해당 값을 사용하기 전에 null을 확인하지 않고도 자신 있게 진행할 수 있습니다. . `옵션이 있는 경우에만`(또는 우리가 작업 중인 어떤 유형의 값이든) 값이 없을 가능성에 대해 걱정해야 하며 컴파일러는 값을 사용하기 전에 해당 사례를 처리하는지 확인합니다.

즉, `옵션`을 변환해야 합니다.`T` 작업을 수행하기 전에 `T`로. 일반적으로 이것은 null과 관련된 가장 일반적인 문제 중 하나를 파악하는 데 도움이 됩니다. 실제로는 null이 아니지만 null이 아니라고 가정합니다.

null이 아닌 값을 잘못 가정하는 위험을 제거하면 코드에 대한 확신을 가질 수 있습니다. null일 수 있는 값을 가지려면 해당 값의 유형을 `옵션`으로 만들어 명시적으로 옵트인해야 합니다.`. 그런 다음 해당 값을 사용할 때 값이 null인 경우를 명시적으로 처리해야 합니다. 값이 `옵션이 아닌 유형을 갖는 모든 위치`, 값이 null이 아니라고 안전하게 가정할 *수 있습니다. 이는 Rust가 null의 보급을 제한하고 Rust 코드의 안전성을 높이기 위한 의도적인 설계 결정이었습니다.*

따라서 `Option` 유형의 값이 있을 때 `Some` 변형에서 `T` 값을 어떻게 얻습니까?` 그 값을 사용할 수 있도록? `옵션[` enum에는 다양한 상황에서 유용한 많은 메서드가 있습니다. 설명서](https://doc.rust-lang.org/std/option/enum.Option.html) 에서 확인할 수 있습니다. `옵션의 메서드에 익숙해지기`는 Rust와의 여정에 매우 유용할 것입니다.

일반적으로 `옵션`을 사용하기 위해서는` 값, 각 변형을 처리할 코드가 필요합니다. `Some(T)` 값이 있을 때만 실행되는 일부 코드가 필요하며 이 코드는 내부 `T`를 사용할 수 있습니다. 일부를 원합니다. `없음` 값이 있고 해당 코드에 사용 가능한 `T` 값이 없는 경우에만 실행할 다른 코드 `일치` 식은 열거형과 함께 사용할 때 바로 이 작업을 수행하는 제어 흐름 구조입니다. 가지고 있는 열거형의 변형에 따라 다른 코드이며 해당 코드는 일치하는 값 내부의 데이터를 사용할 수 있습니다.

------

## [`일치` 제어 흐름 구성](https://doc.rust-lang.org/book/ch06-02-match.html#the-match-control-flow-construct)

Rust에는 일련의 패턴과 값을 비교한 다음 어떤 패턴이 일치하는지에 따라 코드를 실행할 수 있는 `일치`라는 매우 강력한 제어 흐름 구조가 있습니다. 패턴은 리터럴 값, 변수 이름, 와일드카드 및 기타 여러 항목으로 구성될 수 있습니다. [18장에서는](https://doc.rust-lang.org/book/ch18-00-patterns.html) 모든 종류의 패턴과 그 기능을 다룹니다. `일치`의 힘은 패턴의 표현력과 컴파일러가 가능한 모든 사례가 처리되었음을 확인한다는 사실에서 비롯됩니다.

`일치` 표현을 동전 분류 기계와 같다고 생각하십시오. 동전은 다양한 크기의 구멍이 있는 트랙을 따라 미끄러져 내려오고 각 동전은 맞는 첫 번째 구멍을 통해 떨어집니다. 같은 방식으로 값은 `일치`의 각 패턴을 통과하고 첫 번째 패턴에서 값이 `적합`하고 실행 중에 사용되는 관련 코드 블록에 값이 떨어집니다.

동전이라고 하면 `일치`를 사용하여 예를 들어 보겠습니다! Listing 6-3과 같이 미지의 미국 동전을 가져와 계수기와 유사한 방식으로 어떤 동전인지 결정하고 그 값을 센트 단위로 반환하는 함수를 작성할 수 있습니다.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

목록 6-3: 열거형 및 열거형의 변형을 패턴으로 갖는 `일치` 식

`value_in_cents` 함수에서 `일치`를 분석해 보겠습니다. 먼저 `match` 키워드와 표현식을 나열합니다. 이 경우 값은 `coin`입니다. 이것은 `if`와 함께 사용되는 조건식과 매우 유사해 보이지만 큰 차이점이 있습니다. `if`를 사용하면 조건이 부울 값으로 평가되어야 하지만 여기서는 모든 유형이 될 수 있습니다. 이 예제에서 `coin`의 유형은 첫 번째 줄에서 정의한 `Coin` 열거형입니다.

다음은 `일치` 암입니다. 팔은 패턴과 일부 코드의 두 부분으로 구성됩니다. 여기에서 첫 번째 팔에는 `Coin::Penny` 값인 패턴이 있고 패턴과 실행할 코드를 구분하는 `=>` 연산자가 있습니다. 이 경우 코드는 값 `1`일 뿐입니다. 각 팔은 쉼표로 다음 팔과 구분됩니다.

`일치` 표현식이 실행되면 결과 값을 각 팔의 패턴과 순서대로 비교합니다. 패턴이 값과 일치하면 해당 패턴과 연결된 코드가 실행됩니다. 해당 패턴이 값과 일치하지 않으면 동전 분류기에서와 같이 실행이 다음 팔로 계속됩니다. 우리는 필요한 만큼 많은 팔을 가질 수 있습니다: Listing 6-3에서 `일치`에는 4개의 팔이 있습니다.

각 암과 연관된 코드는 식이며 일치하는 암에서 식의 결과 값은 전체 `일치` 식에 대해 반환되는 값입니다.

매치 암 코드가 짧으면 일반적으로 중괄호를 사용하지 않습니다. Listing 6-3에서 각 암은 값을 반환합니다. 매치 암에서 여러 줄의 코드를 실행하려면 중괄호를 사용해야 하며 암 다음에 오는 쉼표는 선택 사항입니다. 예를 들어 다음 코드는 `Lucky penny!`를 인쇄합니다. 메서드가 `Coin::Penny`로 호출될 때마다 블록의 마지막 값인 `1`을 반환합니다.

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!(`Lucky penny!`);
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### [값에 바인딩되는 패턴](https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values)

일치 암의 또 다른 유용한 기능은 패턴과 일치하는 값 부분에 바인딩할 수 있다는 것입니다. 열거형 변형에서 값을 추출할 수 있는 방법입니다.

예를 들어 열거형 변형 중 하나를 변경하여 내부에 데이터를 보관하도록 하겠습니다. 1999년부터 2008년까지 미국은 한 면에 50개 주마다 다른 디자인의 쿼터를 주조했습니다. 다른 주화에는 주 디자인이 없으므로 분기에만 이 추가 가치가 있습니다. Listing 6-4에서 수행한 것처럼 내부에 저장된 `UsState` 값을 포함하도록 `Quarter` 변형을 변경하여 이 정보를 `enum`에 추가할 수 있습니다.

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

목록 6-4: `Quarter` 변형이 `UsState` 값도 포함하는 `Coin` 열거형

친구가 50개의 주 쿼터를 모두 수집하려고 한다고 상상해 봅시다. 동전 유형별로 느슨한 거스름돈을 정렬하는 동안 각 분기와 관련된 주 이름을 불러서 친구가 가지고 있지 않은 것이 있으면 컬렉션에 추가할 수 있습니다.

이 코드의 일치 표현식에서 변형 `Coin::Quarter`의 값과 일치하는 패턴에 `state`라는 변수를 추가합니다. `Coin::Quarter`가 일치하면 `state` 변수는 해당 분기의 상태 값에 바인딩됩니다. 그런 다음 해당 팔의 코드에서 다음과 같이 `상태`를 사용할 수 있습니다.

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!(`State quarter from {:?}!`, state);
            25
        }
    }
}
```

`value_in_cents(Coin::Quarter(UsState::Alaska))`를 호출하면 `coin`은 `Coin::Quarter(UsState::Alaska)`가 됩니다. 해당 값을 각 일치 부문과 비교할 때 `Coin::Quarter(state)`에 도달할 때까지 일치하는 항목이 없습니다. 이 시점에서 `state`에 대한 바인딩은 `UsState::Alaska` 값이 됩니다. 그런 다음 `println!`에서 해당 바인딩을 사용할 수 있습니다. `Quarter`에 대한 `Coin` 열거형 변형에서 내부 상태 값을 가져옵니다.

### [`옵션`과 일치](https://doc.rust-lang.org/book/ch06-02-match.html#matching-with-optiont)

이전 섹션에서 `Option`을 사용할 때 `Some` 케이스에서 내부 `T` 값을 가져오고 싶었습니다.`; `옵션을 처리할 수도 있습니다.` `Coin` 열거형과 마찬가지로 `match`를 사용합니다! 동전을 비교하는 대신 `Option의 변형을 비교하겠습니다.`이지만 `일치` 표현식이 작동하는 방식은 동일하게 유지됩니다.

`옵션`을 받는 함수를 작성하고 싶다고 가정해 보겠습니다.` 내부에 값이 있으면 해당 값에 1을 더합니다. 내부에 값이 없으면 함수는 `없음` 값을 반환하고 어떤 작업도 수행하지 않아야 합니다.

이 함수는 `match` 덕분에 작성하기가 매우 쉽고 Listing 6-5와 같습니다.

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```

목록 6-5: `옵션`에 `일치` 표현식을 사용하는 함수`

`plus_one`의 첫 번째 실행을 자세히 살펴보겠습니다. `plus_one(five)`를 호출하면 `plus_one` 본문의 변수 `x`는 `Some(5)` 값을 갖게 됩니다. 그런 다음 이를 각 매치 부문과 비교합니다.

```rust
            None => None,
```

`Some(5)` 값은 패턴 `None`과 일치하지 않으므로 다음 단계로 계속 진행합니다.

```rust
            Some(i) => Some(i + 1),
```

`Some(5)`는 `Some(i)`와 일치합니까? 그렇습니다! 동일한 변형이 있습니다. `i`는 `Some`에 포함된 값에 바인딩되므로 `i`는 값 `5`를 사용합니다. 그런 다음 매치 암의 코드가 실행되므로 `i` 값에 1을 더하고 총 `6`이 포함된 새로운 `Some` 값을 생성합니다.

이제 Listing 6-5에서 `x`가 `None`인 `plus_one`의 두 번째 호출을 살펴보겠습니다. `일치`를 입력하고 첫 번째 팔과 비교합니다.

```rust
            None => None,
```

일치합니다! 추가할 값이 없으므로 프로그램이 중지되고 `=>` 오른쪽에 `None` 값이 반환됩니다. 첫 번째 팔이 일치했기 때문에 다른 팔은 비교되지 않습니다.

`일치`와 열거형을 결합하면 많은 상황에서 유용합니다. Rust 코드에서 이 패턴을 많이 볼 수 있습니다. 열거형과 `일치`하고 변수를 내부 데이터에 바인딩한 다음 이를 기반으로 코드를 실행합니다. 처음에는 조금 까다롭지만 익숙해지면 모든 언어로 제공되기를 바랄 것입니다. 지속적으로 사용자가 선호하는 제품입니다.

### [일치는 철저합니다](https://doc.rust-lang.org/book/ch06-02-match.html#matches-are-exhaustive)

우리가 논의해야 할 `일치`의 또 다른 측면이 있습니다. 팔의 패턴은 모든 가능성을 커버해야 합니다. 버그가 있고 컴파일되지 않는 `plus_one` 함수의 이 버전을 고려하십시오.

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
```

우리는 `없음` 사례를 처리하지 않았으므로 이 코드는 버그를 일으킬 것입니다. 운 좋게도 Rust가 잡는 방법을 알고 있는 버그입니다. 이 코드를 컴파일하려고 하면 다음 오류가 발생합니다.

```bash
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
```

Rust는 우리가 모든 가능한 경우를 다루지 않았다는 것과 우리가 어떤 패턴을 잊어버렸는지 알고 있습니다! Rust의 일치는 *철저* 합니다. 코드가 유효하려면 마지막 가능성을 모두 소진해야 합니다. 특히 `옵션`의 경우`, Rust는 우리가 `없음` 사례를 명시적으로 처리하는 것을 잊는 것을 방지할 때 null이 있을 수 있는 값을 가지고 있다고 가정하는 것을 방지하여 앞에서 논의한 수십억 달러의 실수를 불가능하게 만듭니다.

### [범용 패턴 및 `_` 자리 표시자](https://doc.rust-lang.org/book/ch06-02-match.html#catch-all-patterns-and-the-_-placeholder)

열거형을 사용하면 몇 가지 특정 값에 대해 특별한 작업을 수행할 수도 있지만 다른 모든 값에 대해서는 하나의 기본 작업을 수행합니다. 주사위 굴림에서 3을 굴리면 플레이어가 움직이지 않고 대신 새 멋진 모자를 받는 게임을 구현한다고 상상해 보십시오. 7이 나오면 플레이어는 멋진 모자를 잃습니다. 다른 모든 값의 경우 플레이어는 게임 보드에서 해당 수의 공간을 이동합니다. 다음은 임의의 값이 아닌 하드코딩된 주사위 굴림의 결과와 본문이 없는 함수로 표현되는 다른 모든 논리를 사용하여 논리를 구현하는 `일치`입니다. 왜냐하면 실제로 구현하는 것은 이 예제의 범위를 벗어나기 때문입니다.

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

처음 두 팔의 경우 패턴은 리터럴 값 `3` 및 `7`입니다. 다른 모든 가능한 값을 포함하는 마지막 팔의 경우 패턴은 `기타`라는 이름을 지정하기 위해 선택한 변수입니다. `기타` 암에 대해 실행되는 코드는 변수를 `move_player` 함수에 전달하여 사용합니다.

이 코드는 `u8`이 가질 수 있는 모든 가능한 값을 나열하지 않았지만 마지막 패턴이 구체적으로 나열되지 않은 모든 값과 일치하기 때문에 컴파일됩니다. 이 범용 패턴은 `일치`가 철저해야 한다는 요구 사항을 충족합니다. 패턴이 순서대로 평가되기 때문에 포괄적인 팔을 마지막에 두어야 합니다. catch-all arm을 더 일찍 넣으면 다른 arm은 실행되지 않으므로, catch-all 후에 arm을 추가하면 Rust가 경고합니다!

Rust는 또한 포괄적인 것을 원하지만 포괄적인 패턴의 값을 *사용하고* 싶지 않을 때 사용할 수 있는 패턴이 있습니다. `_`는 모든 값과 일치하고 해당 값에 바인딩되지 않는 특수 패턴입니다. 이는 Rust에게 우리가 그 값을 사용하지 않을 것임을 알려주므로 Rust는 사용하지 않는 변수에 대해 경고하지 않습니다.

게임의 규칙을 변경해 보겠습니다. 이제 3이나 7이 아닌 다른 것을 굴리면 다시 굴려야 합니다. 더 이상 범용 값을 사용할 필요가 없으므로 `other`라는 변수 대신 `_`를 사용하도록 코드를 변경할 수 있습니다.

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

이 예제는 또한 마지막 팔의 다른 모든 값을 명시적으로 무시하기 때문에 완전성 요구 사항을 충족합니다. 우리는 아무것도 잊지 않았습니다.

마지막으로 게임의 규칙을 한 번 더 변경하여 3이나 7 이외의 것을 굴려도 자신의 차례에 아무 일도 일어나지 않도록 하겠습니다. [`튜플 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-tuple-type) 섹션 에서 ) `_` 팔과 함께 가는 코드:

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```

여기에서 우리는 명시적으로 이전 버전의 패턴과 일치하지 않는 다른 값을 사용하지 않을 것이며 이 경우 어떤 코드도 실행하고 싶지 않다고 명시적으로 말하고 있습니다.

[패턴과 일치에 대한 자세한 내용은 18장](https://doc.rust-lang.org/book/ch18-00-patterns.html) 에서 다룰 것입니다. 지금은 `일치` 표현이 다소 장황한 상황에서 유용할 수 있는 `if let` 구문으로 넘어갈 것입니다.

------

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

------

# [패키지, 크레이트 및 모듈로 성장하는 프로젝트 관리](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html#managing-growing-projects-with-packages-crates-and-modules)

대규모 프로그램을 작성할 때 코드 구성이 점점 더 중요해집니다. 관련 기능을 그룹화하고 고유한 기능으로 코드를 분리하면 특정 기능을 구현하는 코드를 찾을 위치와 기능 작동 방식을 변경하기 위해 이동해야 하는 위치를 명확히 할 수 있습니다.

지금까지 우리가 작성한 프로그램은 하나의 파일에 하나의 모듈에 있었습니다. 프로젝트가 커짐에 따라 코드를 여러 모듈로 분할한 다음 여러 파일로 분할하여 구성해야 합니다. 패키지는 여러 바이너리 크레이트와 선택적으로 하나의 라이브러리 크레이트를 포함할 수 있습니다. 패키지가 커짐에 따라 외부 종속성이 되는 별도의 크레이트로 부품을 추출할 수 있습니다. 이 장에서는 이러한 모든 기술을 다룹니다. 함께 발전하는 상호 관련된 일련의 패키지로 구성된 대규모 프로젝트의 경우 Cargo는 *작업 공간을* 제공하며 이에 대해서는 14장의 [`Cargo 작업 공간`](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html) 섹션 에서 다룰 것입니다.

또한 구현 세부 사항을 캡슐화하여 더 높은 수준에서 코드를 재사용할 수 있도록 합니다. 작업을 구현하면 다른 코드가 구현 작동 방식을 알 필요 없이 공용 인터페이스를 통해 코드를 호출할 수 있습니다. 코드를 작성하는 방식에 따라 다른 코드에서 사용할 수 있도록 공개되는 부분과 변경 권한이 있는 개인 구현 세부 정보인 부분이 정의됩니다. 이것은 머리 속에 간직해야 하는 세부 사항의 양을 제한하는 또 다른 방법입니다.

관련 개념은 범위입니다. 코드가 작성되는 중첩 컨텍스트에는 `범위 내`로 정의되는 일련의 이름이 있습니다. 코드를 읽고 쓰고 컴파일할 때 프로그래머와 컴파일러는 특정 지점의 특정 이름이 변수, 함수, 구조체, 열거형, 모듈, 상수 또는 기타 항목을 참조하는지 여부와 해당 항목이 무엇을 의미하는지 알아야 합니다. 범위를 생성하고 범위에 포함되거나 포함되지 않는 이름을 변경할 수 있습니다. 동일한 범위에 동일한 이름을 가진 두 개의 항목이 있을 수 없습니다. 이름 충돌을 해결하기 위한 도구를 사용할 수 있습니다.

Rust에는 어떤 세부 정보가 노출되고 어떤 세부 정보가 비공개인지, 프로그램의 각 범위에 어떤 이름이 있는지 등 코드 구성을 관리할 수 있는 여러 기능이 있습니다. *모듈 시스템* 이라고도 통칭하는 이러한 기능에는 다음이 포함됩니다.

- **패키지: 상자** 를 만들고 테스트하고 공유할 수 있는 Cargo 기능
- **상자:** 라이브러리 또는 실행 파일을 생성하는 모듈 트리
- **모듈** 및 **사용:** 경로의 구성, 범위 및 개인정보 보호를 제어할 수 있습니다.
- **경로:** 구조체, 함수 또는 모듈과 같은 항목의 이름을 지정하는 방법

이 장에서는 이러한 모든 기능을 다루고, 상호 작용하는 방법에 대해 설명하고, 범위를 관리하는 데 사용하는 방법을 설명합니다. 마지막에는 모듈 시스템에 대한 확실한 이해가 있어야 하며 프로처럼 스코프를 다룰 수 있어야 합니다!

------

## [패키지 및 크레이트](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html#packages-and-crates)

우리가 다룰 모듈 시스템의 첫 번째 부분은 패키지와 크레이트입니다.

크레이트 *는* Rust 컴파일러가 한 번에 고려하는 가장 작은 양의 코드입니다. `cargo` 대신 `rustc`를 실행하고 단일 소스 코드 파일을 전달하더라도(1장의 `Rust 프로그램 작성 및 실행` 섹션에서 끝까지 수행한 것처럼) 컴파일러는 해당 파일을 다음과 같이 간주합니다. 상자. 크레이트는 모듈을 포함할 수 있으며 모듈은 크레이트와 함께 컴파일되는 다른 파일에서 정의될 수 있습니다. 다음 섹션에서 살펴보겠습니다.

크레이트는 바이너리 크레이트 또는 라이브러리 크레이트의 두 가지 형태 중 하나로 올 수 있습니다. *바이너리 크레이트는* 명령줄 프로그램이나 서버와 같이 실행할 수 있는 실행 파일로 컴파일할 수 있는 프로그램입니다. 각각에는 실행 파일이 실행될 때 발생하는 일을 정의하는 `main`이라는 함수가 있어야 합니다. 지금까지 우리가 만든 모든 크레이트는 바이너리 크레이트였습니다.

*라이브러리 크레이트에는* `메인` 기능이 없으며 실행 파일로 컴파일되지 않습니다. 대신 여러 프로젝트와 공유할 기능을 정의합니다. [예를 들어 2장](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number) 에서 사용한 `rand` 크레이트는 난수를 생성하는 기능을 제공합니다. Rustaceans가 `크레이트`라고 말하는 대부분의 경우 라이브러리 크레이트를 의미하며 `라이브러리`의 일반적인 프로그래밍 개념과 상호 교환적으로 `크레이트`를 사용합니다.

크레이트 *루트는 Rust 컴파일러* [가](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html) 시작하고 크레이트의 루트 모듈을 구성하는 소스 파일입니다.

패키지 *는* 일련의 기능을 제공하는 하나 이상의 상자 묶음입니다. 패키지에는 이러한 상자를 만드는 방법을 설명하는 *Cargo.toml 파일이 포함되어 있습니다.* Cargo는 실제로 코드를 빌드하는 데 사용했던 명령줄 도구용 바이너리 크레이트를 포함하는 패키지입니다. Cargo 패키지에는 바이너리 크레이트가 의존하는 라이브러리 크레이트도 포함되어 있습니다. 다른 프로젝트는 Cargo 명령줄 도구가 사용하는 것과 동일한 논리를 사용하기 위해 Cargo 라이브러리 크레이트에 의존할 수 있습니다.

패키지는 원하는 만큼 많은 바이너리 크레이트를 포함할 수 있지만 최대 하나의 라이브러리 크레이트만 포함할 수 있습니다. 패키지는 라이브러리든 바이너리 크레이트든 적어도 하나의 크레이트를 포함해야 합니다.

패키지를 만들 때 어떤 일이 발생하는지 살펴보겠습니다. 먼저 `cargo new` 명령을 입력합니다.

```bash
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

`cargo new`를 실행한 후 `ls`를 사용하여 Cargo가 생성하는 것을 확인합니다. 프로젝트 디렉토리에는 패키지를 제공하는 *Cargo.toml* 파일이 있습니다. *main.rs를* 포함하는 *src* 디렉토리 도 있습니다. *텍스트 편집기에서 Cargo.toml을* 열고 *src/main.rs* 에 대한 언급이 없음에 유의하십시오 . *Cargo는 src/main.rs가* 패키지와 같은 이름을 가진 바이너리 크레이트의 크레이트 루트라는 규칙을 따릅니다. 마찬가지로 Cargo는 패키지 디렉토리에 *src/lib.rs가* 포함되어 있으면 패키지에 패키지와 동일한 이름의 라이브러리 크레이트가 포함되어 있고 *src/lib.rs가 포함되어 있음을 알고 있습니다.*상자 루트입니다. Cargo는 크레이트 루트 파일을 `rustc`에 전달하여 라이브러리 또는 바이너리를 빌드합니다.

*여기에는 src/main.rs* 만 포함하는 패키지가 있습니다. 즉, `my-project`라는 이름의 바이너리 크레이트만 포함되어 있습니다. 패키지에 *src/main.rs* 및 *src/lib.rs 가* 포함되어 있으면 패키지와 동일한 이름을 가진 바이너리와 라이브러리라는 두 개의 크레이트가 있습니다. 패키지는 *src/bin* 디렉토리에 파일을 배치하여 여러 바이너리 크레이트를 가질 수 있습니다. 각 파일은 별도의 바이너리 크레이트가 됩니다.

------

## [범위 및 프라이버시를 제어하기 위한 모듈 정의](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#defining-modules-to-control-scope-and-privacy)

이 섹션에서는 모듈과 모듈 시스템의 다른 부분, 즉 항목 이름을 지정할 수 있는 *경로* 에 대해 설명합니다. 경로를 범위로 가져오는 `use` 키워드; 항목을 공개하기 위한 `pub` 키워드. 또한 `as` 키워드, 외부 패키지 및 glob 연산자에 대해서도 설명합니다.

먼저 나중에 코드를 구성할 때 쉽게 참조할 수 있도록 규칙 목록부터 시작하겠습니다. 그러면 각 규칙에 대해 자세히 설명하겠습니다.

### [모듈 치트 시트](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#modules-cheat-sheet)

여기에서는 모듈, 경로, `use` 키워드 및 `pub` 키워드가 컴파일러에서 작동하는 방법과 대부분의 개발자가 코드를 구성하는 방법에 대한 빠른 참조를 제공합니다. 우리는 이 장 전체에서 이러한 각 규칙의 예를 살펴보겠지만 모듈이 작동하는 방식을 상기시키기 위해 참조하기에 좋은 곳입니다.

- **크레이트 루트에서 시작** : 크레이트를 컴파일할 때 컴파일러는 먼저 크레이트 루트 파일(일반적으로 라이브러리 크레이트의 경우 *src/lib.rs* 또는 바이너리 크레이트의 경우 *src/main.rs )에서 컴파일할 코드를 찾습니다.*

- 모듈 선언

  : 크레이트 루트 파일에서 새 모듈을 선언할 수 있습니다. 예를 들어 `garden` 모듈을 다음과 같이 선언합니다.

  ```
  mod garden;
  ```

  . 컴파일러는 다음 위치에서 모듈의 코드를 찾습니다.

  - 인라인, `mod garden` 다음의 세미콜론을 대체하는 중괄호 내
  - *src/garden.rs* 파일에서
  - *src/garden/mod.rs* 파일에서

- 하위 모듈 선언

  : 크레이트 루트 이외의 모든 파일에서 하위 모듈을 선언할 수 있습니다. 예를 들어 다음과 같이 선언할 수 있습니다.

  ```
  mod vegetables;
  ```

  ~에

  src/garden.rs

  . 컴파일러는 다음 위치에서 상위 모듈의 이름이 지정된 디렉토리 내에서 하위 모듈의 코드를 찾습니다.

  - 인라인, 세미콜론 대신 중괄호 안에 있는 `mod vegetables` 바로 뒤에
  - *src/garden/vegetables.rs* 파일에서
  - *src/garden/vegetables/mod.rs* 파일에서

- **모듈의 코드 경로** : 모듈이 크레이트의 일부가 되면, 개인 정보 보호 규칙이 허용하는 한 코드 경로를 사용하여 동일한 크레이트의 다른 곳에서 해당 모듈의 코드를 참조할 수 있습니다. 예를 들어 정원 야채 모듈의 `Asparagus` 유형은 `crate::garden::vegetables::Asparagus`에서 찾을 수 있습니다.

- **비공개 대 공개** : 모듈 내의 코드는 기본적으로 상위 모듈에서 비공개입니다. 모듈을 공개하려면 `mod` 대신 `pub mod`로 모듈을 선언하십시오. 공용 모듈 내의 항목도 공용으로 만들려면 선언 전에 `pub`를 사용하십시오.

- **`use` 키워드** : 범위 내에서 `use` 키워드는 긴 경로의 반복을 줄이기 위해 항목에 대한 바로 가기를 만듭니다. `crate::garden::vegetables::Asparagus`를 참조할 수 있는 모든 범위에서 `use crate::garden::vegetables::Asparagus;`로 바로 가기를 만들 수 있습니다. 그런 다음 범위에서 해당 유형을 사용하려면 `Asparagus`만 작성하면 됩니다.

여기에서 이러한 규칙을 설명하는 `backyard`라는 이름의 바이너리 크레이트를 만듭니다. `backyard`라고도 하는 크레이트의 디렉토리에는 다음 파일과 디렉토리가 포함되어 있습니다.

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

이 경우 크레이트 루트 파일은 *src/main.rs* 이며 다음을 포함합니다:

파일 이름: src/main.rs

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!(`I'm growing {:?}!`, plant);
}
```

`펍 모드 가든;` *line은 src/garden.rs* 에서 찾은 코드를 포함하도록 컴파일러에 지시합니다.

파일명: src/garden.rs

```rust
pub mod vegetables;
```

여기 `pub mod vegetables;` *src/garden/vegetables.rs* 의 코드 도 포함되어 있음을 의미합니다. 해당 코드는 다음과 같습니다.

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

이제 이러한 규칙에 대해 자세히 알아보고 실제로 시연해 보겠습니다.

### [관련 코드를 모듈로 그룹화](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html#grouping-related-code-in-modules)

*모듈을 사용* 하면 가독성과 손쉬운 재사용을 위해 크레이트 내에서 코드를 구성할 수 있습니다. 모듈은 또한 모듈 내의 코드가 기본적으로 비공개이기 때문에 항목의 *프라이버시를* 제어할 수 있습니다. 개인 항목은 외부에서 사용할 수 없는 내부 구현 세부 정보입니다. 우리는 모듈과 그 안에 있는 항목을 공개하도록 선택할 수 있습니다. 이렇게 하면 외부 코드가 모듈을 사용하고 의존할 수 있도록 노출됩니다.

예를 들어 레스토랑의 기능을 제공하는 라이브러리 크레이트를 작성해 보겠습니다. 우리는 함수의 서명을 정의하지만 식당 구현보다는 코드 구성에 집중하기 위해 본문을 비워 둡니다.

레스토랑 업계에서 레스토랑의 일부는 *프론트 오브 하우스(front of house)* , 다른 일부는 *백 오브 하우스(back of house)* 라고 합니다. 집 앞은 고객이 있는 곳입니다. 여기에는 호스트가 고객을 앉히고 서버가 주문 및 지불을 받고 바텐더가 음료를 만드는 곳이 포함됩니다. 백오브하우스는 셰프와 요리사가 주방에서 일하고, 식기 세척기가 청소하고, 관리자가 관리 업무를 수행하는 곳입니다.

이러한 방식으로 크레이트를 구성하기 위해 함수를 중첩된 모듈로 구성할 수 있습니다. `cargo new restaurant --lib`를 실행하여 `restaurant`라는 새 라이브러리를 만듭니다. 그런 다음 목록 7-1의 코드를 *src/lib.rs* 에 입력하여 일부 모듈 및 함수 서명을 정의합니다. 집 앞 부분은 다음과 같습니다.

파일 이름: src/lib.rs

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

목록 7-1: 함수를 포함하는 다른 모듈을 포함하는 `front_of_house` 모듈

`mod` 키워드 뒤에 모듈 이름(이 경우 `front_of_house`)을 사용하여 모듈을 정의합니다. 그러면 모듈 본문이 중괄호 안에 들어갑니다. 모듈 내부에 `hosting` 및 `serving` 모듈이 있는 경우와 같이 다른 모듈을 배치할 수 있습니다. 모듈은 또한 구조체, 열거형, 상수, 특성 및 목록 7-1에서와 같이 함수와 같은 다른 항목에 대한 정의를 보유할 수 있습니다.

모듈을 사용하여 관련 정의를 함께 그룹화하고 관련 이유를 명명할 수 있습니다. 이 코드를 사용하는 프로그래머는 모든 정의를 읽을 필요 없이 그룹을 기반으로 코드를 탐색할 수 있으므로 관련된 정의를 더 쉽게 찾을 수 있습니다. 이 코드에 새로운 기능을 추가하는 프로그래머는 프로그램을 체계적으로 유지하기 위해 코드를 어디에 배치해야 하는지 알 것입니다.

앞에서 우리는 *src/main.rs* 와 *src/lib.rs를* 크레이트 루트라고 부른다고 언급했습니다. *이름이 붙은 이유는 이 두 파일 중 하나의 내용이 모듈* 트리로 알려진 크레이트 모듈 구조의 루트에서 `크레이트`라는 모듈을 형성하기 때문입니다.

목록 7-2는 목록 7-1의 구조에 대한 모듈 트리를 보여줍니다.

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

목록 7-2: 목록 7-1의 코드에 대한 모듈 트리

이 트리는 일부 모듈이 서로 중첩되는 방식을 보여줍니다. 예를 들어 `hosting`은 `front_of_house` 내부에 중첩됩니다. 트리는 또한 일부 모듈이 서로 *형제 임을 보여줍니다. 즉, 동일한 모듈에서 정의된다는 의미입니다.* `hosting` 및 `serving`은 `front_of_house` 내에 정의된 형제입니다. 모듈 A가 모듈 B 안에 포함되어 있으면 모듈 A는 모듈 B의 *자식 이고 모듈 B는 모듈 A의* *부모* 라고 말합니다. 전체 모듈 트리는 `crate`라는 암시적 모듈 아래에 뿌리를 두고 있습니다.

모듈 트리는 컴퓨터에 있는 파일 시스템의 디렉토리 트리를 상기시킬 수 있습니다. 이것은 매우 적절한 비교입니다! 파일 시스템의 디렉토리와 마찬가지로 모듈을 사용하여 코드를 구성합니다. 그리고 디렉토리의 파일과 마찬가지로 모듈을 찾을 방법이 필요합니다.

------

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

------

## [`use` 키워드를 사용하여 경로를 범위로 가져오기](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#bringing-paths-into-scope-with-the-use-keyword)

함수를 호출하는 경로를 작성해야 하는 것은 불편하고 반복적으로 느껴질 수 있습니다. 목록 7-7에서 `add_to_waitlist` 함수에 대한 절대 경로를 선택했는지 상대 경로를 선택했는지에 관계없이 `add_to_waitlist`를 호출할 때마다 `front_of_house` 및 `hosting`도 지정해야 했습니다. 다행스럽게도 이 프로세스를 단순화하는 방법이 있습니다. `use` 키워드를 사용하여 경로에 대한 바로 가기를 한 번 만든 다음 범위의 다른 모든 곳에서 더 짧은 이름을 사용할 수 있습니다.

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

범위에 `use`와 경로를 추가하는 것은 파일 시스템에서 심볼릭 링크를 만드는 것과 유사합니다. 크레이트 루트에 `use crate::front_of_house::hosting`을 추가함으로써 `hosting` 모듈이 크레이트 루트에 정의된 것처럼 이제 해당 범위에서 `hosting`이 유효한 이름이 됩니다. `사용`으로 범위에 포함된 경로는 다른 경로와 마찬가지로 개인 정보도 확인합니다.

`사용`은 `사용`이 발생하는 특정 범위에 대한 바로 가기만 생성한다는 점에 유의하십시오. 목록 7-12는 `eat_at_restaurant` 함수를 `customer`라는 이름의 새 하위 모듈로 이동합니다. 이 모듈은 `use` 문과 다른 범위이므로 함수 본문이 컴파일되지 않습니다.

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

컴파일러 오류는 바로 가기가 `고객` 모듈 내에서 더 이상 적용되지 않음을 보여줍니다.

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

`사용`이 해당 범위에서 더 이상 사용되지 않는다는 경고도 있습니다! 이 문제를 해결하려면 `customer` 모듈 내에서 `use`를 이동하거나 하위 `customer` 모듈 내에서 `super::hosting`을 사용하여 상위 모듈의 바로 가기를 참조하십시오.

### [관용적인 `사용` 경로 만들기](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths)

Listing 7-11에서 왜 우리가 `use crate::front_of_house::hosting`을 지정한 다음 `use` 경로를 지정하지 않고 `eat_at_restaurant`에서 `hosting::add_to_waitlist`를 호출했는지 궁금할 수 있습니다. `add_to_waitlist` 함수를 사용하여 Listing 7-13과 같은 결과를 얻을 수 있습니다.

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

Listing 7-11과 7-13 모두 동일한 작업을 수행하지만 Listing 7-11은 `use`를 사용하여 함수를 범위로 가져오는 관용적인 방법입니다. `use`를 사용하여 함수의 상위 모듈을 범위로 가져오는 것은 함수를 호출할 때 상위 모듈을 지정해야 함을 의미합니다. 함수를 호출할 때 부모 모듈을 지정하면 전체 경로의 반복을 최소화하면서 함수가 로컬로 정의되지 않았음을 분명히 할 수 있습니다. 목록 7-13의 코드는 `add_to_waitlist`가 정의된 위치가 명확하지 않습니다.

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

이 관용구의 예외는 Rust가 허용하지 않기 때문에 `use` 문을 사용하여 이름이 같은 두 항목을 범위로 가져오는 경우입니다. 목록 7-15는 이름은 같지만 상위 모듈이 다른 두 개의 `결과` 유형을 범위로 가져오는 방법과 이를 참조하는 방법을 보여줍니다.

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

보시다시피 상위 모듈을 사용하면 두 가지 `결과` 유형이 구별됩니다. 대신에 우리가 `use std::fmt::Result`와 `use std::io::Result`를 지정했다면, 우리는 같은 범위에 두 개의 `Result` 유형을 가지게 될 것이고 러스트는 언제 우리가 의미하는 것이 무엇인지 알지 못할 것입니다. 우리는 `결과`를 사용했습니다.

### [`as` 키워드로 새 이름 제공](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#providing-new-names-with-the-as-keyword)

`use`를 사용하여 동일한 이름의 두 유형을 동일한 범위로 가져오는 문제에 대한 또 다른 솔루션이 있습니다. 경로 뒤에 유형에 대해 `as` 및 새 로컬 이름 또는 *alias 를* 지정할 수 있습니다. Listing 7-16은 `as`를 사용하여 두 개의 `Result` 유형 중 하나의 이름을 변경하여 Listing 7-15의 코드를 작성하는 또 다른 방법을 보여줍니다.

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

`use` 키워드를 사용하여 이름을 범위로 가져오면 새 범위에서 사용할 수 있는 이름은 비공개입니다. 코드 범위에 정의된 것처럼 해당 이름을 참조하도록 코드를 호출하는 코드를 활성화하려면 `pub`와 `use`를 결합할 수 있습니다. *이 기술을 재내보내기라고* 부르는 이유는 항목을 범위로 가져오는 동시에 다른 사람이 해당 항목을 범위로 가져올 수 있도록 만들기 때문입니다.

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

이 변경 전에는 외부 코드에서 `restaurant::front_of_house::hosting::add_to_waitlist()` 경로를 사용하여 `add_to_waitlist` 함수를 호출해야 했습니다. 이제 이 `pub use`가 루트 모듈에서 `hosting` 모듈을 다시 내보냈으므로 이제 외부 코드에서 `restaurant::hosting::add_to_waitlist()` 경로를 대신 사용할 수 있습니다.

다시 내보내기는 코드의 내부 구조가 코드를 호출하는 프로그래머가 도메인에 대해 생각하는 방식과 다를 때 유용합니다. 예를 들어, 이 식당 은유에서 식당을 운영하는 사람들은 `집 앞`과 `집 뒤`를 생각합니다. 그러나 레스토랑을 방문하는 고객은 아마도 그러한 용어로 레스토랑의 부분에 대해 생각하지 않을 것입니다. `pub use`를 사용하면 하나의 구조로 코드를 작성할 수 있지만 다른 구조를 노출할 수 있습니다. 이렇게 하면 라이브러리에서 작업하는 프로그래머와 라이브러리를 호출하는 프로그래머를 위해 라이브러리가 잘 정리됩니다. [14장의 ``pub use` 로 편리한 공용 API 내보내기`](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use) 섹션 에서 `pub use`의 또 다른 예와 이것이 크레이트 문서에 미치는 영향을 살펴보겠습니다.

### [외부 패키지 사용](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#using-external-packages)

2장에서 우리는 임의의 숫자를 얻기 위해 `rand`라는 외부 패키지를 사용하는 추측 게임 프로젝트를 프로그래밍했습니다. 프로젝트에서 `rand`를 사용하기 위해 *Cargo.toml* 에 다음 행을 추가했습니다.

파일 이름: Cargo.toml

```toml
rand = `0.8.5`
```

*Cargo.toml* 에 `rand`를 종속 항목으로 추가하면 Cargo가 [crates.io](https://crates.io/) 에서 `rand` 패키지와 모든 종속 항목을 다운로드 하고 프로젝트에서 `rand`를 사용할 수 있도록 합니다.

그런 다음 `rand` 정의를 패키지 범위로 가져오기 위해 상자 이름 `rand`로 시작하는 `use` 행을 추가하고 범위로 가져오고자 하는 항목을 나열했습니다. [2장의 `난수 생성`](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number) 섹션 에서 `Rng` 특성을 범위로 가져오고 `rand::thread_rng` 함수를 호출했습니다.

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

[Rust 커뮤니티의 구성원은 crates.io](https://crates.io/) 에서 사용할 수 있는 많은 패키지를 만들었습니다. 패키지에 패키지를 가져오려면 패키지의 *Cargo.toml* 파일에 패키지를 나열하고 `use`를 사용하여 상자에서 항목을 범위로 가져옵니다. .

표준 `std` 라이브러리는 패키지 외부에 있는 크레이트이기도 합니다. 표준 라이브러리는 Rust 언어와 함께 제공되기 때문에 `std`를 포함하도록 *Cargo.toml을* 변경할 필요가 없습니다. 그러나 항목을 패키지 범위로 가져오려면 `사용`으로 참조해야 합니다. 예를 들어 `HashMap`의 경우 다음 행을 사용합니다.

```rust
use std::collections::HashMap;
```

이것은 표준 라이브러리 크레이트의 이름인 `std`로 시작하는 절대 경로입니다.

### [중첩된 경로를 사용하여 큰 `사용` 목록 정리](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#using-nested-paths-to-clean-up-large-use-lists)

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

더 큰 프로그램에서 중첩된 경로를 사용하여 동일한 크레이트 또는 모듈에서 많은 항목을 범위로 가져오면 필요한 별도의 `use` 문 수를 크게 줄일 수 있습니다!

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

### [글롭 연산자](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#the-glob-operator)

경로에 정의된 *모든* 공용 항목을 범위로 가져오려면 해당 경로 뒤에 `*` glob 연산자를 지정할 수 있습니다.

```rust
use std::collections::*;
```

이 `use` 문은 `std::collections`에 정의된 모든 공용 항목을 현재 범위로 가져옵니다. glob 연산자를 사용할 때 주의하십시오! Glob은 범위 내에 어떤 이름이 있고 프로그램에서 사용된 이름이 정의된 위치를 구분하기 어렵게 만들 수 있습니다.

glob 연산자는 테스트 중인 모든 항목을 `tests` 모듈로 가져오기 위해 테스트할 때 자주 사용됩니다. [11장의 `테스트 작성 방법`](https://doc.rust-lang.org/book/ch11-01-writing-tests.html#how-to-write-tests) 섹션 에서 이에 대해 이야기하겠습니다. glob 연산자는 때때로 prelude 패턴의 일부로 사용되기도 합니다. 해당 패턴에 대한 자세한 내용은 [표준 라이브러리 문서를 참조하십시오.](https://doc.rust-lang.org/std/prelude/index.html#other-preludes)

------

## [모듈을 다른 파일로 분리](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#separating-modules-into-different-files)

지금까지 이 장의 모든 예제는 하나의 파일에 여러 모듈을 정의했습니다. 모듈이 커지면 해당 정의를 별도의 파일로 이동하여 코드를 더 쉽게 탐색할 수 있습니다.

예를 들어, 여러 식당 모듈이 있는 Listing 7-17의 코드부터 시작해 봅시다. 크레이트 루트 파일에 모든 모듈을 정의하는 대신 모듈을 파일로 추출합니다. 이 경우 크레이트 루트 파일은 *src/lib.rs* 이지만 이 절차는 크레이트 루트 파일이 *src/main.rs* 인 바이너리 크레이트에서도 작동합니다.

먼저 `front_of_house` 모듈을 자체 파일로 추출합니다. `front_of_house` 모듈의 중괄호 안에 있는 코드를 제거하고 `mod front_of_house;`만 남깁니다. 선언, 그래서 *src/lib.rs는* 목록 7-21에 표시된 코드를 포함합니다. *목록 7-22에서 src/front_of_house.rs* 파일을 생성할 때까지는 컴파일되지 않습니다.

파일 이름: src/lib.rs

```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

*Listing 7-21: 본문이 src/front_of_house.rs* 에 있는 `front_of_house` 모듈 선언

다음으로 목록 7-22에 표시된 것처럼 중괄호 안에 있던 코드를 *src/front_of_house.rs 라는 새 파일에 배치합니다.* 컴파일러는 `front_of_house`라는 이름의 크레이트 루트에서 모듈 선언을 발견했기 때문에 이 파일을 살펴봐야 한다는 것을 압니다.

파일 이름: src/front_of_house.rs

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

*목록 7-22: src/front_of_house.rs* 의 `front_of_house` 모듈 내부 정의

모듈 트리에서 *한 번만* `mod` 선언을 사용하여 파일을 로드하면 됩니다. 컴파일러가 파일이 프로젝트의 일부임을 알게 되면(그리고 모듈 트리에서 `mod` 문을 넣은 위치로 인해 코드가 상주하는 위치를 알게 되면) 프로젝트의 다른 파일은 다음을 사용하여 로드된 파일의 코드를 참조해야 합니다. [`모듈 트리에서 항목을 참조하기 위한 경로`](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) 섹션 에서 설명한 대로 선언된 위치의 경로입니다. 즉, `mod`는 다른 프로그래밍 언어에서 볼 수 있는 `include` 작업이 *아닙니다.*

다음으로 `호스팅` 모듈을 자체 파일로 추출합니다. `hosting`은 루트 모듈이 아닌 `front_of_house`의 하위 모듈이기 때문에 프로세스가 약간 다릅니다. `호스팅`을 위한 파일을 모듈 트리의 조상 이름을 따서 명명될 새 디렉토리(이 경우 *src/front_of_house/ )* 에 배치합니다.

`호스팅` 이동을 시작하려면 `호스팅` 모듈의 선언만 포함하도록 *src/front_of_house.rs를* 변경합니다.

파일 이름: src/front_of_house.rs

```rust
pub mod hosting;
```

그런 다음 `hosting` 모듈에서 만든 정의를 포함하기 위해 *src/front_of_house* 디렉터리와 *hosting.rs* 파일을 만듭니다.

파일 이름: src/front_of_house/hosting.rs

```rust
pub fn add_to_waitlist() {}
```

대신에 *hosting.rs를* *src* 디렉토리 에 넣으면 컴파일러는 *hosting.rs* 코드가 `front_of_house` 모듈의 자식으로 선언되지 않고 크레이트 루트에 선언된 `hosting` 모듈에 있을 것으로 예상합니다. 어떤 모듈의 코드를 검사할 파일에 대한 컴파일러의 규칙은 디렉토리와 파일이 모듈 트리와 더 밀접하게 일치함을 의미합니다.

> ### [대체 파일 경로](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#alternate-file-paths)
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

*src/lib.rs* 의 `pub use crate::front_of_house::hosting` 문도 변경되지 않았으며 `use`는 크레이트의 일부로 컴파일되는 파일에 영향을 미치지 않습니다. `mod` 키워드는 모듈을 선언하고 Rust는 해당 모듈에 들어가는 코드의 모듈과 동일한 이름을 가진 파일을 찾습니다.

## [요약](https://doc.rust-lang.org/book/ch07-05-separating-modules-into-different-files.html#summary)

Rust를 사용하면 패키지를 여러 크레이트로 분할하고 크레이트를 모듈로 분할하여 한 모듈에 정의된 항목을 다른 모듈에서 참조할 수 있습니다. 절대 또는 상대 경로를 지정하여 이를 수행할 수 있습니다. 이러한 경로는 `use` 문을 사용하여 범위로 가져올 수 있으므로 해당 범위에서 항목을 여러 번 사용하는 경우 더 짧은 경로를 사용할 수 있습니다. 모듈 코드는 기본적으로 비공개이지만 `pub` 키워드를 추가하여 정의를 공개할 수 있습니다.

다음 장에서는 깔끔하게 정리된 코드에서 사용할 수 있는 표준 라이브러리의 일부 컬렉션 데이터 구조를 살펴보겠습니다.

------

# [공통 컬렉션](https://doc.rust-lang.org/book/ch08-00-common-collections.html#common-collections)

*Rust의 표준 라이브러리에는 collections* 라는 매우 유용한 데이터 구조가 많이 포함되어 있습니다. 대부분의 다른 데이터 유형은 하나의 특정 값을 나타내지만 컬렉션에는 여러 값이 포함될 수 있습니다. 기본 제공 배열 및 튜플 유형과 달리 이러한 컬렉션이 가리키는 데이터는 힙에 저장됩니다. 즉, 데이터 양은 컴파일 타임에 알 필요가 없으며 프로그램 실행에 따라 늘어나거나 줄어들 수 있습니다. 컬렉션 종류마다 기능과 비용이 다르며 현재 상황에 적합한 컬렉션을 선택하는 것은 시간이 지남에 따라 발전하게 될 기술입니다. 이 장에서는 Rust 프로그램에서 매우 자주 사용되는 세 가지 모음에 대해 논의할 것입니다.

- 벡터 *를* 사용하면 가변 개수의 값을 서로 옆에 저장할 수 있습니다.
- 문자열 *은* 문자 모음입니다. 앞에서 `문자열` 유형에 대해 언급했지만 이 장에서는 이에 대해 자세히 설명합니다.
- 해시 *맵을 사용* 하면 값을 특정 키와 연결할 수 있습니다. *map* 이라는 보다 일반적인 데이터 구조의 특정 구현입니다.

표준 라이브러리에서 제공하는 다른 종류의 컬렉션에 대해 알아보려면 [설명서를](https://doc.rust-lang.org/std/collections/index.html) 참조하십시오 .

벡터, 문자열 및 해시 맵을 만들고 업데이트하는 방법과 각 항목을 특별하게 만드는 방법에 대해 설명합니다.

------

## [벡터를 사용하여 값 목록 저장](https://doc.rust-lang.org/book/ch08-01-vectors.html#storing-lists-of-values-with-vectors)

우리가 살펴볼 첫 번째 컬렉션 유형은 `Vec*`, 벡터* 라고도 합니다. 벡터를 사용하면 모든 값을 메모리에 나란히 배치하는 단일 데이터 구조에 둘 이상의 값을 저장할 수 있습니다. 벡터는 동일한 유형의 값만 저장할 수 있습니다. 벡터는 다음과 같은 경우에 유용합니다. 파일의 텍스트 줄이나 장바구니의 항목 가격과 같은 항목 목록입니다.

### [새 벡터 만들기](https://doc.rust-lang.org/book/ch08-01-vectors.html#creating-a-new-vector)

새로운 빈 벡터를 생성하기 위해 목록 8-1에 표시된 것처럼 `Vec::new` 함수를 호출합니다.

```rust
    let v: Vec<i32> = Vec::new();
```

목록 8-1: `i32` 유형의 값을 담을 새 빈 벡터 만들기

여기에 유형 주석을 추가했습니다. 우리가 이 벡터에 어떤 값도 삽입하지 않기 때문에 Rust는 우리가 어떤 종류의 요소를 저장하려고 하는지 알지 못합니다. 이것은 중요한 포인트입니다. 벡터는 제네릭을 사용하여 구현됩니다. 10장에서 고유한 유형으로 제네릭을 사용하는 방법을 다룰 것입니다. 지금은 `Vec` 표준 라이브러리에서 제공하는 유형은 모든 유형을 보유할 수 있습니다. 특정 유형을 보유하기 위해 벡터를 만들 때 꺾쇠 괄호 안에 유형을 지정할 수 있습니다. Listing 8-1에서 Rust에게 `Vec`의 `v`는 `i32` 유형의 요소를 보유합니다.

더 자주 `Vec` 초기 값을 사용하면 Rust는 저장하려는 값의 유형을 추론하므로 이 유형 주석을 거의 수행할 필요가 없습니다. Rust는 `vec!` 매크로를 편리하게 제공하며, 이 매크로는 사용자가 지정한 값을 보유하는 새 벡터를 생성합니다. . 목록 8-2는 새로운 `Vec[`는 값 `1`, `2` 및 `3`을 포함합니다. 정수 유형은 `i32`입니다. 3장의 `데이터 유형`](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) 섹션 에서 논의한 것처럼 이것이 기본 정수 유형이기 때문입니다.

```rust
    let v = vec![1, 2, 3];
```

목록 8-2: 값을 포함하는 새 벡터 만들기

초기 `i32` 값을 제공했기 때문에 Rust는 `v`의 유형이 `Vec`임을 추론할 수 있습니다.`, 타입 어노테이션은 필요하지 않습니다. 다음으로 벡터를 수정하는 방법을 살펴보겠습니다.

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

모든 변수와 마찬가지로 값을 변경하려면 3장에서 설명한 것처럼 `mut` 키워드를 사용하여 변경 가능하게 만들어야 합니다. 내부에 배치하는 숫자는 모두 `i32` 유형이고 Rust 데이터에서 이것을 추론하므로 `Vec`이 필요하지 않습니다.` 주석.

### [벡터의 요소 읽기](https://doc.rust-lang.org/book/ch08-01-vectors.html#reading-elements-of-vectors)

벡터에 저장된 값을 참조하는 방법에는 인덱싱을 통하거나 `get` 메서드를 사용하는 두 가지 방법이 있습니다. 다음 예제에서는 명확성을 높이기 위해 이러한 함수에서 반환되는 값의 유형에 주석을 달았습니다.

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

여기에 몇 가지 세부 사항을 기록하십시오. 벡터는 0부터 시작하여 숫자로 인덱싱되기 때문에 인덱스 값 `2`를 사용하여 세 번째 요소를 얻습니다. `&` 및 `[]`를 사용하면 인덱스 값에 있는 요소에 대한 참조를 제공합니다. 인수로 전달된 인덱스와 함께 `get` 메서드를 사용하면 `match`와 함께 사용할 수 있는 `Option<&T>`를 얻습니다.

Rust가 요소를 참조하는 이 두 가지 방법을 제공하는 이유는 기존 요소 범위 밖의 인덱스 값을 사용하려고 할 때 프로그램이 동작하는 방식을 선택할 수 있도록 하기 위함입니다. 예를 들어, 목록 8-5에 표시된 것처럼 5개 요소의 벡터가 있고 각 기술을 사용하여 인덱스 100의 요소에 액세스하려고 하면 어떤 일이 발생하는지 봅시다.

```rust
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
```

목록 8-5: 5개의 요소를 포함하는 벡터에서 인덱스 100의 요소에 액세스 시도

이 코드를 실행할 때 첫 번째 `[]` 메서드는 존재하지 않는 요소를 참조하기 때문에 프로그램을 패닉 상태로 만듭니다. 이 방법은 벡터의 끝을 지나 요소에 액세스하려는 시도가 있는 경우 프로그램을 중단시키려는 경우에 가장 적합합니다.

`get` 메서드에 벡터 외부에 있는 인덱스가 전달되면 당황하지 않고 `None`을 반환합니다. 벡터 범위를 벗어난 요소에 액세스하는 것이 정상적인 상황에서 가끔 발생할 수 있는 경우 이 방법을 사용합니다. 그러면 코드는 6장에서 설명한 대로 `Some(&element)` 또는 `없음`을 처리하는 논리를 갖게 됩니다. 예를 들어 인덱스는 사람이 숫자를 입력하는 것에서 나올 수 있습니다. 사용자가 실수로 너무 큰 숫자를 입력하여 프로그램이 `없음` 값을 얻는 경우 사용자에게 현재 벡터에 몇 개의 항목이 있는지 알려주고 유효한 값을 입력할 수 있는 또 다른 기회를 제공할 수 있습니다. 오타로 인해 프로그램이 충돌하는 것보다 더 사용자 친화적일 것입니다!

프로그램에 유효한 참조가 있는 경우 차용 검사기는 소유권 및 차용 규칙(4장에서 다룸)을 시행하여 이 참조 및 벡터 내용에 대한 다른 참조가 유효한지 확인합니다. 동일한 범위에서 가변 및 불변 참조를 가질 수 없다는 규칙을 상기하십시오. 이 규칙은 벡터의 첫 번째 요소에 대한 불변 참조를 보유하고 끝에 요소를 추가하려고 시도하는 목록 8-6에 적용됩니다. 이 프로그램은 나중에 함수에서 해당 요소를 참조하려고 하면 작동하지 않습니다.

```rust
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

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

목록 8-6의 코드는 제대로 작동하는 것처럼 보일 수 있습니다. 왜 첫 번째 요소에 대한 참조가 벡터의 끝에서 변경 사항에 관심을 가져야 합니까? 이 오류는 벡터가 작동하는 방식 때문입니다. 벡터가 메모리에서 서로 옆에 있는 값을 넣기 때문에 벡터 끝에 새 요소를 추가하려면 새 메모리를 할당하고 이전 요소를 새 공간에 복사해야 할 수 있습니다. 벡터가 현재 저장된 위치에 모든 요소를 나란히 놓을 공간이 충분하지 않습니다. 이 경우 첫 번째 요소에 대한 참조는 할당 해제된 메모리를 가리킵니다. 차용 규칙은 프로그램이 그러한 상황에서 종료되는 것을 방지합니다.

> 참고: `Vec`의 구현 세부 사항에 대한 자세한 내용은` 유형, [“The Rustonomicon”](https://doc.rust-lang.org/nomicon/vec/vec.html) 참조 .

### [벡터의 값에 대한 반복](https://doc.rust-lang.org/book/ch08-01-vectors.html#iterating-over-the-values-in-a-vector)

벡터의 각 요소에 차례로 액세스하려면 인덱스를 사용하여 한 번에 하나씩 액세스하는 대신 모든 요소를 반복합니다. 목록 8-7은 `i32` 값의 벡터에 있는 각 요소에 대한 불변 참조를 가져오고 인쇄하기 위해 `for` 루프를 사용하는 방법을 보여줍니다.

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

불변이든 가변이든 벡터를 반복하는 것은 빌림 검사기의 규칙 때문에 안전합니다. Listing 8-7 및 Listing 8-8의 `for` 루프 본문에서 항목을 삽입하거나 제거하려고 시도하면 Listing 8-6의 코드에서 얻은 것과 유사한 컴파일러 오류가 발생합니다. `for` 루프가 보유한 벡터에 대한 참조는 전체 벡터의 동시 수정을 방지합니다.

### [열거형을 사용하여 여러 유형 저장](https://doc.rust-lang.org/book/ch08-01-vectors.html#using-an-enum-to-store-multiple-types)

벡터는 동일한 유형의 값만 저장할 수 있습니다. 이는 불편할 수 있습니다. 다양한 유형의 항목 목록을 저장해야 하는 사용 사례가 분명히 있습니다. 다행스럽게도 열거형의 변형은 동일한 열거형 유형으로 정의되므로 다른 유형의 요소를 나타내기 위해 하나의 유형이 필요할 때 열거형을 정의하고 사용할 수 있습니다!

예를 들어 행의 일부 열에 정수, 일부 부동 소수점 숫자 및 일부 문자열이 포함된 스프레드시트의 행에서 값을 가져오고 싶다고 가정해 보겠습니다. 변형이 다른 값 유형을 보유할 열거형을 정의할 수 있으며 모든 열거형 변형은 동일한 유형, 즉 열거형으로 간주됩니다. 그런 다음 해당 열거형을 보유할 벡터를 생성할 수 있으므로 궁극적으로 다른 유형을 보유합니다. Listing 8-9에서 이를 증명했습니다.

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

Listing 8-9: 하나의 벡터에 다른 유형의 값을 저장하는 `enum` 정의

Rust는 각 요소를 저장하는 데 필요한 힙의 메모리 양을 정확히 알기 위해 컴파일 시간에 벡터에 어떤 유형이 있는지 알아야 합니다. 또한 이 벡터에서 어떤 유형이 허용되는지 명시해야 합니다. Rust가 벡터가 어떤 유형이든 가질 수 있도록 허용했다면, 하나 이상의 유형이 벡터의 요소에 대해 수행되는 작업에서 오류를 일으킬 가능성이 있습니다. 열거형과 `일치` 표현식을 사용한다는 것은 6장에서 논의한 것처럼 Rust가 컴파일 타임에 가능한 모든 경우를 처리한다는 것을 의미합니다.

프로그램이 벡터에 저장하기 위해 런타임에 얻을 수 있는 전체 유형 세트를 모른다면 enum 기술이 작동하지 않습니다. 대신 17장에서 다룰 특성 개체를 사용할 수 있습니다.

벡터를 사용하는 가장 일반적인 방법에 대해 논의했으므로 `Vec`에 정의된 모든 유용한 방법에 대한 [API 문서를 검토하십시오.](https://doc.rust-lang.org/std/vec/struct.Vec.html)예를 들어 `push` 외에도 `pop` 메서드는 마지막 요소를 제거하고 반환합니다.

### [벡터를 삭제하면 요소가 삭제됨](https://doc.rust-lang.org/book/ch08-01-vectors.html#dropping-a-vector-drops-its-elements)

다른 `구조체`와 마찬가지로 벡터는 목록 8-10에 설명된 대로 범위를 벗어나면 해제됩니다.

```rust
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
```

Listing 8-10: 벡터와 해당 요소가 놓이는 위치 표시

벡터가 삭제되면 모든 내용도 삭제됩니다. 즉, 보유하고 있는 정수가 정리됩니다. 차용 검사기는 벡터 자체가 유효한 동안에만 벡터 내용에 대한 참조가 사용되는지 확인합니다.

다음 컬렉션 유형인 `문자열`로 이동하겠습니다!

------

## [UTF-8로 인코딩된 텍스트를 문자열과 함께 저장](https://doc.rust-lang.org/book/ch08-02-strings.html#storing-utf-8-encoded-text-with-strings)

4장에서 문자열에 대해 이야기했지만 이제 더 자세히 살펴보겠습니다. 새로운 Rustacean은 일반적으로 세 가지 이유의 조합으로 문자열에 집착합니다: 가능한 오류를 노출하는 Rust의 성향, 문자열은 많은 프로그래머가 인정하는 것보다 더 복잡한 데이터 구조, UTF-8입니다. 이러한 요소는 다른 프로그래밍 언어에서 왔을 때 어렵게 보일 수 있는 방식으로 결합됩니다.

문자열은 바이트 모음으로 구현되고 해당 바이트가 텍스트로 해석될 때 유용한 기능을 제공하는 일부 메서드로 구현되기 때문에 모음의 맥락에서 문자열에 대해 논의합니다. 이 섹션에서는 생성, 업데이트 및 읽기와 같은 모든 컬렉션 유형이 갖는 `문자열`에 대한 작업에 대해 설명합니다. 또한 `문자열`이 다른 컬렉션과 다른 방식, 즉 사람과 컴퓨터가 `문자열` 데이터를 해석하는 방식의 차이로 인해 `문자열`에 대한 인덱싱이 복잡해지는 방식에 대해서도 설명합니다.

### [문자열이란 무엇입니까?](https://doc.rust-lang.org/book/ch08-02-strings.html#what-is-a-string)

*먼저 문자열* 이라는 용어의 의미를 정의합니다. Rust는 핵심 언어에 단 하나의 문자열 유형을 가지고 있는데, 이는 일반적으로 차용된 형식 `&str`에서 볼 수 있는 문자열 슬라이스 `str`입니다. 4장에서 다른 곳에 저장된 일부 UTF-8 인코딩 문자열 데이터에 대한 참조인 *문자열 슬라이스에* 대해 이야기했습니다. 예를 들어 문자열 리터럴은 프로그램의 바이너리에 저장되므로 문자열 조각입니다.

핵심 언어로 코딩되지 않고 Rust의 표준 라이브러리에서 제공하는 `문자열` 유형은 확장 가능하고 변경 가능하며 소유된 UTF-8 인코딩 문자열 유형입니다. Rustacean이 Rust에서 `문자열`을 언급할 때, 그들은 `문자열` 또는 문자열 슬라이스 `&str` 유형 중 하나를 참조할 수 있습니다. 이 섹션은 주로 `문자열`에 관한 것이지만 Rust의 표준 라이브러리에서 두 가지 유형이 많이 사용되며 `문자열`과 문자열 슬라이스는 모두 UTF-8로 인코딩됩니다.

### [새 문자열 만들기](https://doc.rust-lang.org/book/ch08-02-strings.html#creating-a-new-string)

`Vec에서 사용할 수 있는 많은 동일한 작업`는 `String`과 함께 사용할 수 있습니다. `String`은 실제로 일부 추가 보증, 제한 및 기능이 있는 바이트 벡터 주위의 래퍼로 구현되기 때문입니다. `Vec과 동일한 방식으로 작동하는 함수의 예입니다.` 및 `String`은 목록 8-11에 표시된 것처럼 인스턴스를 생성하는 `새` 함수입니다.

```rust
    let mut s = String::new();
```

목록 8-11: 비어 있는 새 `문자열` 만들기

이 줄은 데이터를 로드할 수 있는 `s`라는 새 빈 문자열을 만듭니다. 종종 문자열을 시작하려는 초기 데이터가 있습니다. 이를 위해 문자열 리터럴처럼 `Display` 특성을 구현하는 모든 유형에서 사용할 수 있는 `to_string` 메서드를 사용합니다. 목록 8-12는 두 가지 예를 보여줍니다.

```rust
    let data = `initial contents`;

    let s = data.to_string();

    // the method also works on a literal directly:
    let s = `initial contents`.to_string();
```

목록 8-12: `to_string` 메서드를 사용하여 문자열 리터럴에서 `문자열` 생성

이 코드는 `초기 내용`을 포함하는 문자열을 생성합니다.

문자열 리터럴에서 `문자열`을 생성하기 위해 `String::from` 함수를 사용할 수도 있습니다. 목록 8-13의 코드는 `to_string`을 사용하는 목록 8-12의 코드와 동일합니다.

```rust
    let s = String::from(`initial contents`);
```

Listing 8-13: `String::from` 함수를 사용하여 문자열 리터럴에서 `String` 생성

문자열은 매우 많은 용도로 사용되기 때문에 문자열에 대해 다양한 일반 API를 사용하여 많은 옵션을 제공할 수 있습니다. 그들 중 일부는 중복되는 것처럼 보일 수 있지만 모두 자리가 있습니다! 이 경우 `String::from`과 `to_string`은 동일한 작업을 수행하므로 어떤 것을 선택하느냐는 스타일과 가독성의 문제입니다.

문자열은 UTF-8로 인코딩되어 있으므로 Listing 8-14와 같이 적절하게 인코딩된 데이터를 문자열에 포함할 수 있습니다.

```rust
    let hello = String::from(`السلام عليكم`);
    let hello = String::from(`Dobrý den`);
    let hello = String::from(`Hello`);
    let hello = String::from(`שָׁלוֹם`);
    let hello = String::from(`नमस्ते`);
    let hello = String::from(`こんにちは`);
    let hello = String::from(`안녕하세요`);
    let hello = String::from(`你好`);
    let hello = String::from(`Olá`);
    let hello = String::from(`Здравствуйте`);
    let hello = String::from(`Hola`);
```

Listing 8-14: 다양한 언어로 된 인사말을 문자열에 저장하기

이들은 모두 유효한 `문자열` 값입니다.

### [문자열 업데이트](https://doc.rust-lang.org/book/ch08-02-strings.html#updating-a-string)

`문자열`은 `Vec`의 내용과 마찬가지로 크기가 커지고 내용이 변경될 수 있습니다.`, 더 많은 데이터를 푸시하면 추가로 `+` 연산자 또는 `format!` 매크로를 사용하여 `문자열` 값을 연결할 수 있습니다.

#### [`push_str` 및 `push`를 사용하여 문자열에 추가](https://doc.rust-lang.org/book/ch08-02-strings.html#appending-to-a-string-with-push_str-and-push)

Listing 8-15와 같이 문자열 조각을 추가하기 위해 `push_str` 메서드를 사용하여 `문자열`을 늘릴 수 있습니다.

```rust
    let mut s = String::from(`foo`);
    s.push_str(`bar`);
```

목록 8-15: `push_str` 메서드를 사용하여 문자열 슬라이스를 `문자열`에 추가

이 두 줄 뒤에 `s`는 `foobar`를 포함합니다. `push_str` 메서드는 매개변수의 소유권을 반드시 갖고 싶지 않기 때문에 문자열 슬라이스를 사용합니다. 예를 들어 Listing 8-16의 코드에서 `s1`에 내용을 추가한 후 `s2`를 사용할 수 있기를 원합니다.

```rust
    let mut s1 = String::from(`foo`);
    let s2 = `bar`;
    s1.push_str(s2);
    println!(`s2 is {s2}`);
```

목록 8-16: `문자열`에 내용을 추가한 후 문자열 슬라이스 사용

`push_str` 메서드가 `s2`의 소유권을 가져간 경우 마지막 줄에 해당 값을 인쇄할 수 없습니다. 그러나 이 코드는 예상대로 작동합니다!

`푸시` 방법은 단일 문자를 매개변수로 사용하여 `문자열`에 추가합니다. Listing 8-17은 `push` 메소드를 사용하여 문자 `l`을 `String`에 추가합니다.

```rust
    let mut s = String::from(`lo`);
    s.push('l');
```

목록 8-17: `push`를 사용하여 `문자열` 값에 문자 하나 추가

결과적으로 `s`에는 `lol`이 포함됩니다.

#### [`+` 연산자 또는 `형식!` 매크로](https://doc.rust-lang.org/book/ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro)

종종 두 개의 기존 문자열을 결합하고 싶을 것입니다. 그렇게 하는 한 가지 방법은 목록 8-18에 표시된 것처럼 `+` 연산자를 사용하는 것입니다.

```rust
    let s1 = String::from(`Hello, `);
    let s2 = String::from(`world!`);
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

Listing 8-18: `+` 연산자를 사용하여 두 개의 `String` 값을 새로운 `String` 값으로 결합

문자열 `s3`에는 `Hello, world!`가 포함됩니다. 추가 후 `s1`이 더 이상 유효하지 않은 이유와 `s2`에 대한 참조를 사용한 이유는 `+` 연산자를 사용할 때 호출되는 메서드의 시그니처와 관련이 있습니다. `+` 연산자는 서명이 다음과 같은 `추가` 방법을 사용합니다.

```rust
fn add(self, s: &str) -> String {
```

표준 라이브러리에서 제네릭 및 관련 유형을 사용하여 정의된 `추가`를 볼 수 있습니다. 여기서 우리는 `문자열` 값으로 이 메서드를 호출할 때 발생하는 구체적인 유형으로 대체했습니다. 10장에서 제네릭에 대해 논의할 것입니다. 이 서명은 `+` 연산자의 까다로운 부분을 이해하는 데 필요한 단서를 제공합니다.

첫째, `s2`에는 `&`가 있습니다. 즉, 첫 번째 문자열에 두 번째 문자열의 *참조를* 추가한다는 의미입니다. 이는 `add` 함수의 `s` 매개변수 때문입니다. `&str`만 `String`에 추가할 수 있습니다. 두 개의 `문자열` 값을 함께 추가할 수 없습니다. 하지만 잠깐만요. `&s2`의 유형은 `add`에 대한 두 번째 매개변수에 지정된 `&str`이 아니라 `&String`입니다. 그렇다면 Listing 8-18이 컴파일되는 이유는 무엇입니까?

`add` 호출에서 `&s2`를 사용할 수 있는 이유는 컴파일러가 `&String` 인수를 `&str`로 *강제 할 수 있기 때문입니다.* 우리가 `add` 메소드를 호출할 때, Rust는 여기서 `&s2`를 `&s2[..]`로 바꾸는 *역참조 강제를 사용합니다.* 15장에서 역참조 강제에 대해 더 자세히 논의할 것입니다. `add`는 `s` 매개변수의 소유권을 가지지 않기 때문에 `s2`는 이 작업 후에도 여전히 유효한 `문자열`입니다.

둘째, 서명에서 `add`가 `self`의 소유권을 갖는 것을 볼 수 있습니다. `self`에는 `&`가 *없기 때문입니다.* 이는 Listing 8-18의 `s1`이 `add` 호출로 이동되고 그 이후에는 더 이상 유효하지 않음을 의미합니다. 따라서 `let s3 = s1 + &s2;` 두 문자열을 모두 복사하고 새 문자열을 만드는 것처럼 보이지만 이 문은 실제로 `s1`의 소유권을 가져오고 `s2` 내용의 복사본을 추가한 다음 결과의 소유권을 반환합니다. 즉, 복사를 많이 하는 것처럼 보이지만 그렇지 않습니다. 구현은 복사보다 효율적입니다.

여러 문자열을 연결해야 하는 경우 `+` 연산자의 동작이 다루기 어려워집니다.

```rust
    let s1 = String::from(`tic`);
    let s2 = String::from(`tac`);
    let s3 = String::from(`toe`);

    let s = s1 + `-` + &s2 + `-` + &s3;
```

이 시점에서 `s`는 `tic-tac-toe`가 됩니다. 모든 `+` 및 ``` 문자를 사용하면 무슨 일이 일어나고 있는지 보기가 어렵습니다. 더 복잡한 문자열 결합을 위해 대신 `포맷!`을 사용할 수 있습니다. 매크로:

```rust
    let s1 = String::from(`tic`);
    let s2 = String::from(`tac`);
    let s3 = String::from(`toe`);

    let s = format!(`{s1}-{s2}-{s3}`);
```

이 코드는 또한 `s`를 `tic-tac-toe`로 설정합니다. `포맷!` 매크로는 `println!`처럼 작동하지만 출력을 화면에 인쇄하는 대신 내용이 포함된 `문자열`을 반환합니다. `포맷!`을 사용하는 코드 버전 훨씬 읽기 쉽고 `형식!` 매크로는 이 호출이 해당 매개변수의 소유권을 가지지 않도록 참조를 사용합니다.

### [문자열로 인덱싱](https://doc.rust-lang.org/book/ch08-02-strings.html#indexing-into-strings)

다른 많은 프로그래밍 언어에서 인덱스로 참조하여 문자열의 개별 문자에 액세스하는 것은 유효하고 일반적인 작업입니다. 그러나 Rust에서 인덱싱 구문을 사용하여 `문자열`의 일부에 액세스하려고 하면 오류가 발생합니다. Listing 8-19의 유효하지 않은 코드를 고려하십시오.

```rust
    let s1 = String::from(`hello`);
    let h = s1[0];
```

Listing 8-19: 문자열로 인덱싱 구문 사용 시도

이 코드는 다음 오류를 발생시킵니다.

```bash
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`
  = help: the following other types implement trait `Index<Idx>`:
            <String as Index<RangeFrom<usize>>>
            <String as Index<RangeFull>>
            <String as Index<RangeInclusive<usize>>>
            <String as Index<RangeTo<usize>>>
            <String as Index<RangeToInclusive<usize>>>
            <String as Index<std::ops::Range<usize>>>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to previous error
```

오류와 메모는 이야기를 말해줍니다. Rust 문자열은 인덱싱을 지원하지 않습니다. 하지만 왜 안돼? 이 질문에 답하기 위해 우리는 Rust가 문자열을 메모리에 저장하는 방법을 논의할 필요가 있습니다.

#### [내부 대표](https://doc.rust-lang.org/book/ch08-02-strings.html#internal-representation)

`문자열`은 `Vec`에 대한 래퍼입니다.`. Listing 8-14에서 적절하게 인코딩된 UTF-8 예제 문자열 중 일부를 살펴보겠습니다. 먼저 다음과 같습니다.

```rust
    let hello = String::from(`Hola`);
```

이 경우 `len`은 4가 됩니다. 즉, 문자열 `Hola`를 저장하는 벡터의 길이는 4바이트입니다. 이러한 각 문자는 UTF-8로 인코딩될 때 1바이트를 사용합니다. 그러나 다음 줄은 당신을 놀라게 할 수 있습니다. (이 문자열은 아라비아 숫자 3이 아닌 대문자 키릴 문자 Ze로 시작합니다.)

```rust
    let hello = String::from(`Здравствуйте`);
```

문자열의 길이를 묻는 질문에 12라고 답할 수 있습니다. 사실 Rust의 대답은 24입니다. UTF-8로 `Здравствуйте`를 인코딩하는 데 걸리는 바이트 수입니다. 해당 문자열의 각 유니코드 스칼라 값은 2바이트의 저장 공간을 차지하기 때문입니다. . 따라서 문자열의 바이트에 대한 인덱스가 항상 유효한 유니코드 스칼라 값과 상관되지는 않습니다. 시연을 위해 다음과 같은 잘못된 Rust 코드를 고려하십시오.

```rust
let hello = `Здравствуйте`;
let answer = &hello[0];
```

당신은 이미 `대답`이 첫 글자인 `З`가 아니라는 것을 알고 있습니다. UTF-8로 인코딩하면 `З`의 첫 번째 바이트는 `208`이고 두 번째 바이트는 `151`이므로 `answer`는 실제로 `208`이어야 하지만 `208`은 유효하지 않습니다. 캐릭터 자체. `208`을 반환하는 것은 사용자가 이 문자열의 첫 번째 문자를 요청한 경우 원하는 것이 아닐 가능성이 높습니다. 그러나 이것은 Rust가 바이트 인덱스 0에 있는 유일한 데이터입니다. 사용자는 일반적으로 문자열에 라틴 문자만 포함되어 있어도 바이트 값이 반환되는 것을 원하지 않습니다. 바이트 값을 반환하면 `h`가 아니라 `104`를 반환합니다.

그렇다면 대답은 예상치 못한 값을 반환하고 즉시 발견되지 않을 수 있는 버그를 유발하지 않기 위해 Rust가 이 코드를 전혀 컴파일하지 않고 개발 프로세스 초기에 오해를 방지한다는 것입니다.

#### [바이트 및 스칼라 값과 문자소 클러스터! 어머!](https://doc.rust-lang.org/book/ch08-02-strings.html#bytes-and-scalar-values-and-grapheme-clusters-oh-my)

UTF-8에 대한 또 다른 요점은 실제로 Rust의 관점에서 문자열을 보는 세 가지 관련 방법이 있다는 것입니다: 바이트, 스칼라 값 및 문자소 클러스터(우리가 문자라고 부르는 것에 가장 가까운 것 *)* .

Devanagari 스크립트에 쓰여진 힌디어 단어 `namaste`를 보면 다음과 같이 `u8` 값의 벡터로 저장됩니다.

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

그것은 18바이트이며 컴퓨터가 궁극적으로 이 데이터를 저장하는 방법입니다. Rust의 `char` 유형인 유니코드 스칼라 값으로 보면 해당 바이트는 다음과 같습니다.

```text
['न', 'म', 'स', '्', 'त', 'े']
```

여기에는 6개의 `char` 값이 있지만 네 번째와 여섯 번째는 문자가 아닙니다. 자체적으로 의미가 없는 분음 부호입니다. 마지막으로, 그것들을 문자소 클러스터로 보면 힌디어 단어를 구성하는 4개의 문자라고 부르는 것을 얻을 수 있습니다.

```text
[`न`, `म`, `स्`, `ते`]
```

Rust는 컴퓨터가 저장하는 원시 문자열 데이터를 해석하는 다양한 방법을 제공하므로 데이터가 어떤 인간 언어로 되어 있는지에 관계없이 각 프로그램이 필요한 해석을 선택할 수 있습니다.

Rust가 문자를 얻기 위해 `문자열`로 색인하는 것을 허용하지 않는 마지막 이유는 색인 작업이 항상 일정한 시간(O(1))이 걸릴 것으로 예상되기 때문입니다. 그러나 `문자열`로 성능을 보장하는 것은 불가능합니다. 왜냐하면 Rust는 얼마나 많은 유효한 문자가 있는지 확인하기 위해 내용을 처음부터 인덱스까지 살펴봐야 하기 때문입니다.

### [문자열 슬라이싱](https://doc.rust-lang.org/book/ch08-02-strings.html#slicing-strings)

문자열 인덱싱 작업의 반환 유형(바이트 값, 문자, 문자소 클러스터 또는 문자열 슬라이스)이 무엇인지 명확하지 않기 때문에 문자열로 인덱싱하는 것은 좋지 않은 생각인 경우가 많습니다. 문자열 슬라이스를 생성하기 위해 인덱스를 사용해야 한다면 Rust는 더 구체적으로 지정하도록 요청합니다.

단일 숫자와 함께 `[]`를 사용하여 인덱싱하는 대신 범위와 함께 `[]`를 사용하여 특정 바이트를 포함하는 문자열 슬라이스를 만들 수 있습니다.

```rust
let hello = `Здравствуйте`;

let s = &hello[0..4];
```

여기서 `s`는 문자열의 처음 4바이트를 포함하는 `&str`입니다. 앞서 우리는 이러한 각 문자가 2바이트이며 `s`가 `Зд`가 됨을 의미한다고 언급했습니다.

`&hello[0..1]`과 같은 것으로 문자 바이트의 일부만 슬라이스하려고 하면 Rust는 벡터에서 유효하지 않은 인덱스에 액세스하는 것과 같은 방식으로 런타임에 패닉을 일으킬 것입니다.

```bash
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/main.rs:4:14
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

범위를 사용하여 스트링 슬라이스를 만들면 프로그램이 중단될 수 있으므로 주의해야 합니다.

### [문자열 반복 방법](https://doc.rust-lang.org/book/ch08-02-strings.html#methods-for-iterating-over-strings)

문자열 조각에서 작동하는 가장 좋은 방법은 문자 또는 바이트를 원하는지 여부를 명시하는 것입니다. 개별 유니코드 스칼라 값의 경우 `chars` 메서드를 사용합니다. `Зд`에서 `chars`를 호출하면 `char` 유형의 두 값을 분리하고 반환하며 결과를 반복하여 각 요소에 액세스할 수 있습니다.

```rust
for c in `Зд`.chars() {
    println!(`{c}`);
}
```

이 코드는 다음을 인쇄합니다.

```text
З
д
```

또는 `bytes` 메서드는 도메인에 적합할 수 있는 각 원시 바이트를 반환합니다.

```rust
for b in `Зд`.bytes() {
    println!(`{b}`);
}
```

이 코드는 이 문자열을 구성하는 4바이트를 인쇄합니다.

```text
208
151
208
180
```

그러나 유효한 유니코드 스칼라 값은 1바이트 이상으로 구성될 수 있음을 기억하십시오.

Devanagari 스크립트를 사용하여 문자열에서 문자소 클러스터를 가져오는 것은 복잡하므로 이 기능은 표준 라이브러리에서 제공되지 않습니다. 필요한 기능인 경우 [crates.io](https://crates.io/) 에서 크레이트를 사용할 수 있습니다.

### [문자열은 그렇게 간단하지 않습니다](https://doc.rust-lang.org/book/ch08-02-strings.html#strings-are-not-so-simple)

요약하면 문자열은 복잡합니다. 다른 프로그래밍 언어는 이러한 복잡성을 프로그래머에게 제시하는 방법에 대해 다른 선택을 합니다. Rust는 `문자열` 데이터의 올바른 처리를 모든 Rust 프로그램의 기본 동작으로 만들기로 선택했습니다. 이는 프로그래머가 UTF-8 데이터를 미리 처리하는 데 더 많은 생각을 해야 한다는 것을 의미합니다. 이러한 트레이드오프는 다른 프로그래밍 언어에서 명백한 것보다 더 많은 문자열의 복잡성을 드러내지만 개발 수명 주기 후반에 비ASCII 문자와 관련된 오류를 처리하지 않아도 됩니다.

좋은 소식은 표준 라이브러리가 이러한 복잡한 상황을 올바르게 처리하는 데 도움이 되는 `String` 및 `&str` 유형으로 구성된 많은 기능을 제공한다는 것입니다. 문자열에서 검색하기 위한 `contains` 및 문자열의 일부를 다른 문자열로 대체하기 위한 `replace`와 같은 유용한 메서드에 대한 설명서를 확인하십시오.

조금 덜 복잡한 것으로 전환해 봅시다: 해시 맵!

------

## [해시 맵에 연관된 값과 함께 키 저장](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#storing-keys-with-associated-values-in-hash-maps)

공통 컬렉션의 마지막은 *해시 맵* 입니다. `HashMap<K, V>` 유형 은 이러한 키와 값을 메모리에 배치하는 방법을 결정하는 *해싱 함수를* 사용하여 `K` 유형의 키를 `V` 유형의 값에 대한 매핑을 저장합니다. 많은 프로그래밍 언어가 이러한 종류의 데이터 구조를 지원하지만 몇 가지 예를 들면 해시, 맵, 객체, 해시 테이블, 사전 또는 연관 배열과 같은 다른 이름을 사용하는 경우가 많습니다.

해시 맵은 벡터와 마찬가지로 인덱스를 사용하지 않고 모든 유형의 키를 사용하여 데이터를 조회하려는 경우에 유용합니다. 예를 들어 게임에서 각 키가 팀의 이름이고 값이 각 팀의 점수인 해시 맵에서 각 팀의 점수를 추적할 수 있습니다. 팀 이름이 주어지면 점수를 검색할 수 있습니다.

이 섹션에서는 해시 맵의 기본 API를 살펴보겠지만 표준 라이브러리에서 `HashMap<K, V>`에 정의된 함수에는 더 많은 장점이 숨어 있습니다. 항상 그렇듯이 자세한 내용은 표준 라이브러리 문서를 확인하세요.

### [새 해시 맵 생성](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#creating-a-new-hash-map)

빈 해시 맵을 만드는 한 가지 방법은 `new`를 사용하고 `insert`로 요소를 추가하는 것입니다. *목록 8-20에서 이름이 Blue* 및 *Yellow* 인 두 팀의 점수를 추적하고 있습니다. 파란색 팀은 10점으로 시작하고 노란색 팀은 50점으로 시작합니다.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from(`Blue`), 10);
    scores.insert(String::from(`Yellow`), 50);
```

목록 8-20: 새 해시 맵 생성 및 일부 키와 값 삽입

먼저 표준 라이브러리의 컬렉션 부분에서 `HashMap`을 `사용`해야 합니다. 세 가지 공통 컬렉션 중에서 가장 적게 사용되는 컬렉션이므로 서곡에서 자동으로 범위에 포함된 기능에 포함되지 않습니다. 해시 맵도 표준 라이브러리의 지원이 적습니다. 예를 들어 이를 구성하는 내장 매크로가 없습니다.

벡터와 마찬가지로 해시 맵은 데이터를 힙에 저장합니다. 이 `HashMap`에는 `String` 유형의 키와 `i32` 유형의 값이 있습니다. 벡터와 마찬가지로 해시 맵은 동질적입니다. 모든 키는 서로 같은 유형이어야 하고 모든 값은 같은 유형이어야 합니다.

### [해시 맵의 값에 액세스](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#accessing-values-in-a-hash-map)

목록 8-21에 표시된 것처럼 `get` 메서드에 키를 제공하여 해시 맵에서 값을 가져올 수 있습니다.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from(`Blue`), 10);
    scores.insert(String::from(`Yellow`), 50);

    let team_name = String::from(`Blue`);
    let score = scores.get(&team_name).copied().unwrap_or(0);
```

목록 8-21: 해시 맵에 저장된 Blue 팀의 점수에 액세스

여기에서 `점수`는 파란색 팀과 관련된 값을 가지며 결과는 `10`이 됩니다. `get` 메서드는 `Option<&V>`를 반환합니다. 해시 맵에 해당 키에 대한 값이 없으면 `get`은 `None`을 반환합니다. 이 프로그램은 `옵션`을 얻기 위해 `복사됨`을 호출하여 `옵션`을 처리합니다.` 옵션<&i32>` 대신 `unwrap_or`를 사용하여 `scores`에 키에 대한 항목이 없는 경우 `score`를 0으로 설정합니다.

`for` 루프를 사용하여 벡터와 유사한 방식으로 해시 맵의 각 키/값 쌍을 반복할 수 있습니다.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from(`Blue`), 10);
    scores.insert(String::from(`Yellow`), 50);

    for (key, value) in &scores {
        println!(`{key}: {value}`);
    }
```

이 코드는 임의의 순서로 각 쌍을 인쇄합니다.

```text
Yellow: 50
Blue: 10
```

### [해시 맵 및 소유권](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hash-maps-and-ownership)

`i32`와 같이 `복사` 특성을 구현하는 유형의 경우 값이 해시 맵에 복사됩니다. `문자열`과 같은 소유된 값의 경우 목록 8-22에 표시된 것처럼 값이 이동되고 해시 맵이 해당 값의 소유자가 됩니다.

```rust
    use std::collections::HashMap;

    let field_name = String::from(`Favorite color`);
    let field_value = String::from(`Blue`);

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
```

Listing 8-22: 키와 값이 삽입되면 해시 맵이 소유함을 보여줌

변수 `field_name` 및 `field_value`가 `삽입` 호출로 해시 맵으로 이동된 후에는 사용할 수 없습니다.

값에 대한 참조를 해시 맵에 삽입하면 값이 해시 맵으로 이동되지 않습니다. 참조가 가리키는 값은 적어도 해시 맵이 유효한 동안에는 유효해야 합니다. [10장의 `수명이 있는 참조 유효성 검사`](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes) 섹션 에서 이러한 문제에 대해 자세히 설명합니다.

### [해시 맵 업데이트](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#updating-a-hash-map)

키와 값 쌍의 수는 증가할 수 있지만 각 고유 키는 한 번에 하나의 값만 연결할 수 있습니다(반대의 경우도 마찬가지입니다. 예를 들어 파란색 팀과 노란색 팀 모두 ` 점수`해시 맵).

해시 맵의 데이터를 변경하려면 키에 이미 할당된 값이 있는 경우를 처리하는 방법을 결정해야 합니다. 이전 값을 완전히 무시하고 이전 값을 새 값으로 바꿀 수 있습니다. 이전 값을 유지하고 새 값을 무시하고 키에 이미 값이 *없는 경우에만 새 값을 추가할 수 있습니다.* 또는 이전 값과 새 값을 결합할 수 있습니다. 각 작업을 수행하는 방법을 살펴보겠습니다!

#### [값 덮어쓰기](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#overwriting-a-value)

키와 값을 해시 맵에 삽입한 다음 동일한 키를 다른 값으로 삽입하면 해당 키와 연결된 값이 대체됩니다. Listing 8-23의 코드가 `insert`를 두 번 호출하더라도 Blue 팀의 키 값을 두 번 모두 삽입하기 때문에 해시 맵에는 하나의 키/값 쌍만 포함됩니다.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from(`Blue`), 10);
    scores.insert(String::from(`Blue`), 25);

    println!(`{:?}`, scores);
```

Listing 8-23: 저장된 값을 특정 키로 바꾸기

이 코드는 `{`Blue`: 25}`를 인쇄합니다. `10`의 원래 값을 덮어썼습니다.

#### [키가 없는 경우에만 키와 값 추가](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#adding-a-key-and-value-only-if-a-key-isnt-present)

특정 키가 값과 함께 해시 맵에 이미 존재하는지 확인한 후 다음 조치를 취하는 것이 일반적입니다. 키가 해시 맵에 존재하는 경우 기존 값은 그대로 유지되어야 합니다. 키가 없으면 키와 값을 삽입합니다.

해시 맵에는 확인하려는 키를 매개변수로 사용하는 `항목`이라는 특수 API가 있습니다. `entry` 메서드의 반환 값은 존재하거나 존재하지 않을 수 있는 값을 나타내는 `Entry`라는 열거형입니다. Yellow 팀의 키에 연결된 값이 있는지 확인하고 싶다고 가정해 보겠습니다. 그렇지 않은 경우 값 50을 삽입하고 Blue 팀에도 동일하게 삽입하려고 합니다. `entry` API를 사용하는 코드는 Listing 8-24와 같습니다.

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from(`Blue`), 10);

    scores.entry(String::from(`Yellow`)).or_insert(50);
    scores.entry(String::from(`Blue`)).or_insert(50);

    println!(`{:?}`, scores);
```

Listing 8-24: `entry` 메서드를 사용하여 키에 아직 값이 없는 경우에만 삽입

`Entry`의 `or_insert` 메소드는 해당 `Entry` 키가 존재하는 경우 해당 `Entry` 키의 값에 대한 변경 가능한 참조를 반환하고, 그렇지 않은 경우 이 키의 새 값으로 매개변수를 삽입하고 변경 가능한 참조를 반환하도록 정의됩니다. 새로운 가치에. 이 기술은 논리를 직접 작성하는 것보다 훨씬 깨끗하며, 추가로 차용 검사기와 더 잘 작동합니다.

목록 8-24의 코드를 실행하면 `{`Yellow`: 50, `Blue`: 10}`가 인쇄됩니다. `entry`에 대한 첫 번째 호출은 노란색 팀에 이미 값이 없기 때문에 값이 50인 노란색 팀의 키를 삽입합니다. `entry`에 대한 두 번째 호출은 Blue 팀이 이미 값 10을 가지고 있기 때문에 해시 맵을 변경하지 않습니다.

#### [이전 값을 기반으로 값 업데이트](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#updating-a-value-based-on-the-old-value)

해시 맵의 또 다른 일반적인 사용 사례는 키 값을 조회한 다음 이전 값을 기반으로 업데이트하는 것입니다. 예를 들어 Listing 8-25는 각 단어가 일부 텍스트에 나타나는 횟수를 세는 코드를 보여줍니다. 단어를 키로 사용하는 해시 맵을 사용하고 해당 단어를 본 횟수를 추적하기 위해 값을 증가시킵니다. 단어를 처음 본 경우 먼저 값 0을 삽입합니다.

```rust
    use std::collections::HashMap;

    let text = `hello world wonderful world`;

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!(`{:?}`, map);
```

Listing 8-25: 단어와 카운트를 저장하는 해시 맵을 사용하여 단어의 발생 횟수 세기

이 코드는 `{`world`: 2, `hello`: 1, `wonderful`: 1}`을 인쇄합니다. 동일한 키/값 쌍이 다른 순서로 인쇄된 것을 볼 수 있습니다. [`해시 맵에서 값 액세스`](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#accessing-values-in-a-hash-map) 섹션에서 해시 맵을 반복하는 것이 임의의 순서로 발생한다는 것을 기억하십시오.

`split_whitespace` 메서드는 `text` 값의 공백으로 구분된 하위 슬라이스에 대한 반복자를 반환합니다. `or_insert` 메서드는 지정된 키의 값에 대한 변경 가능한 참조(`&mut V`)를 반환합니다. 여기에서 `count` 변수에 가변 참조를 저장하므로 해당 값에 할당하려면 먼저 별표(`*`)를 사용하여 `count`를 역참조해야 합니다. 변경 가능한 참조는 `for` 루프의 끝에서 범위를 벗어나므로 이러한 모든 변경은 안전하고 차용 규칙에 의해 허용됩니다.

### [해싱 함수](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hashing-functions)

기본적으로 `HashMap`은 해시 테이블 [1](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#siphash) 과 관련된 서비스 거부(DoS) 공격에 대한 저항을 제공할 수 있는 *SipHash* 라는 해싱 기능을 사용합니다. 이것은 사용 가능한 가장 빠른 해싱 알고리즘은 아니지만 성능 저하와 함께 제공되는 더 나은 보안을 위한 트레이드 오프는 그만한 가치가 있습니다. 코드를 프로파일링하고 기본 해시 함수가 용도에 비해 너무 느린 경우 다른 해시를 지정하여 다른 함수로 전환할 수 있습니다. 해셔 *는* `BuildHasher` 특성을 구현하는 유형입니다. 우리는 10장에서 트레이트와 이를 구현하는 방법에 대해 이야기할 것입니다. 처음부터 자신의 해셔를 구현할 필요는 없습니다. [crates.io](https://crates.io/)에는 많은 일반적인 해싱 알고리즘을 구현하는 해셔를 제공하는 다른 Rust 사용자가 공유하는 라이브러리가 있습니다.

1

https://en.wikipedia.org/wiki/SipHash

## [요약](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#summary)

벡터, 문자열 및 해시 맵은 데이터를 저장, 액세스 및 수정해야 할 때 프로그램에 필요한 많은 기능을 제공합니다. 다음은 해결하기 위해 갖추어야 할 몇 가지 연습입니다.

- 정수 목록이 주어지면 벡터를 사용하고 목록의 중앙값(정렬할 때 중간 위치의 값)과 최빈값(가장 자주 발생하는 값, 여기에서 해시 맵이 도움이 됨)을 반환합니다.
- 문자열을 돼지 라틴어로 변환합니다. 각 단어의 첫 자음이 단어의 끝으로 이동되고 “ay”가 추가되므로 “first”는 “irst-fay”가 됩니다. 모음으로 시작하는 단어는 대신 끝에 `hay`가 추가됩니다(`apple`은 `apple-hay`가 됨). UTF-8 인코딩에 대한 세부 사항을 명심하십시오!
- 해시 맵과 벡터를 사용하여 사용자가 회사의 부서에 직원 이름을 추가할 수 있는 텍스트 인터페이스를 만듭니다. 예를 들어 `엔지니어링에 Sally 추가` 또는 `영업에 Amir 추가`가 있습니다. 그런 다음 사용자가 부서의 모든 사람 또는 부서별로 회사의 모든 사람 목록을 사전순으로 정렬하여 검색하도록 합니다.

표준 라이브러리 API 문서는 이러한 연습에 도움이 될 벡터, 문자열 및 해시 맵에 있는 메서드를 설명합니다!

작업이 실패할 수 있는 더 복잡한 프로그램에 들어가고 있으므로 오류 처리에 대해 논의하기에 완벽한 시기입니다. 다음에 그렇게 하겠습니다!

------

# [오류 처리](https://doc.rust-lang.org/book/ch09-00-error-handling.html#error-handling)

오류는 소프트웨어의 삶의 사실이므로 Rust에는 문제가 발생하는 상황을 처리하기 위한 여러 기능이 있습니다. 대부분의 경우 Rust는 코드가 컴파일되기 전에 오류 가능성을 인정하고 조치를 취할 것을 요구합니다. 이 요구 사항은 코드를 프로덕션에 배포하기 전에 오류를 발견하고 적절하게 처리하도록 하여 프로그램을 더욱 강력하게 만듭니다!

*Rust는 오류를 복구 가능한* 오류 와 *복구 불가능한* 오류의 두 가지 주요 범주로 그룹화합니다. *파일을 찾을 수 없음* 오류와 같은 복구 가능한 오류의 경우 사용자에게 문제를 보고하고 작업을 다시 시도하려고 할 가능성이 높습니다. 복구할 수 없는 오류는 항상 버그의 증상입니다. 예를 들어 배열의 끝을 넘어선 위치에 액세스하려고 시도하므로 프로그램을 즉시 중지하려고 합니다.

대부분의 언어는 이러한 두 종류의 오류를 구분하지 않고 예외와 같은 메커니즘을 사용하여 동일한 방식으로 처리합니다. 녹에는 예외가 없습니다. 대신 복구 가능한 오류에 대한 `Result<T, E>` 유형과 `패닉!` 프로그램에서 복구할 수 없는 오류가 발생하면 실행을 중지하는 매크로입니다. 이 장에서는 `패닉!` 먼저 `Result<T, E>` 값 반환에 대해 설명합니다. 또한 오류 복구를 시도할지 또는 실행을 중지할지 결정할 때 고려해야 할 사항을 살펴보겠습니다.

## [복구할 수 없는 `패닉!` 오류](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unrecoverable-errors-with-panic)

때로는 코드에서 나쁜 일이 발생하고 이에 대해 할 수 있는 일이 없습니다. 이러한 경우 Rust는 `패닉!` 매크로. 실제로 패닉을 일으키는 방법에는 두 가지가 있습니다. 코드를 패닉 상태로 만드는 작업(예: 끝을 지나 배열에 액세스)을 수행하거나 `패닉!`을 명시적으로 호출하는 것입니다. 매크로. 두 경우 모두 프로그램에 패닉이 발생합니다. 기본적으로 이러한 패닉은 실패 메시지를 인쇄하고, 풀고, 스택을 정리하고, 종료합니다. 환경 변수를 통해 패닉이 발생할 때 러스트가 호출 스택을 표시하도록 하여 패닉의 원인을 쉽게 추적할 수 있습니다.

> ### [패닉에 대한 응답으로 스택 풀기 또는 중단](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic)
>
> 기본적으로 패닉이 발생하면 프로그램은 *풀기* 시작합니다. 즉, Rust가 스택을 백업하고 만나는 각 함수에서 데이터를 정리합니다. 그러나, 이 걷기 및 청소는 많은 작업입니다. 따라서 Rust는 정리하지 않고 프로그램을 종료하는 *즉시 중단* 의 대안을 선택할 수 있습니다.
>
> 프로그램이 사용하고 있던 메모리는 운영 체제에서 정리해야 합니다. *프로젝트에서 결과 바이너리를 가능한 한 작게 만들어야 하는 경우 Cargo.toml* 파일 의 적절한 `[profile]` 섹션에 `panic = 'abort'`를 추가하여 패닉 시 해제에서 중단으로 전환할 수 있습니다. . 예를 들어 릴리스 모드에서 패닉 상태에서 중단하려면 다음을 추가하십시오.
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

`패닉!` 간단한 프로그램에서:

파일 이름: src/main.rs

```rust
fn main() {
    panic!(`crash and burn`);
}
```

프로그램을 실행하면 다음과 같이 표시됩니다.

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`패닉!` 마지막 두 줄에 포함된 오류 메시지가 발생합니다. 첫 번째 줄은 패닉 메시지와 소스 코드에서 패닉이 발생한 위치를 보여줍니다. *src/main.rs:2:5* 는 두 번째 줄, *src/main.rs* 파일의 다섯 번째 문자임을 나타냅니다.

이 경우 표시된 줄은 코드의 일부이며 해당 줄로 이동하면 `패닉!` 매크로 호출. 다른 경우에는 `패닉!` 호출은 우리 코드가 호출하는 코드에 있을 수 있으며 오류 메시지에 의해 보고된 파일 이름과 줄 번호는 `패닉!` 매크로가 호출되지만 결국 `패닉!` 부르다. `panic!` 함수의 역추적을 사용할 수 있습니다. 문제를 일으키는 코드 부분을 파악하기 위해 전화가 왔습니다. 다음에 백트레이스에 대해 더 자세히 설명하겠습니다.

### [`패닉!` 역추적](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#using-a-panic-backtrace)

다른 예를 살펴보고 `패닉!` 호출은 매크로를 직접 호출하는 코드가 아니라 코드의 버그 때문에 라이브러리에서 발생합니다. 목록 9-1에는 유효한 인덱스 범위를 벗어난 벡터의 인덱스에 액세스하려고 시도하는 코드가 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

목록 9-1: 벡터의 끝을 넘어 요소에 액세스하려고 시도하면 `패닉!` 호출이 발생합니다.

여기에서 우리는 벡터의 100번째 요소(인덱싱이 0에서 시작하기 때문에 인덱스 99에 있음)에 액세스하려고 시도하지만 벡터에는 3개의 요소만 있습니다. 이 상황에서 Rust는 당황할 것입니다. `[]`를 사용하면 요소를 반환해야 하지만 유효하지 않은 인덱스를 전달하면 Rust가 반환할 수 있는 올바른 요소가 없습니다.

C에서 데이터 구조의 끝을 넘어 읽으려는 시도는 정의되지 않은 동작입니다. 메모리가 해당 구조에 속하지 않더라도 데이터 구조의 해당 요소에 해당하는 메모리 위치에 있는 모든 것을 얻을 수 있습니다. 이를 *버퍼 오버 읽기* 라고 하며 공격자가 데이터 구조 뒤에 저장된 허용되지 않아야 하는 데이터를 읽는 방식으로 인덱스를 조작할 수 있는 경우 보안 취약성을 초래할 수 있습니다.

이러한 종류의 취약성으로부터 프로그램을 보호하기 위해 존재하지 않는 인덱스에서 요소를 읽으려고 하면 Rust는 실행을 중지하고 계속 진행을 거부합니다. 그것을 시도하고 보자 :

```bash
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

이 오류는 색인 99에 액세스하려고 시도하는 `main.rs`의 4번째 줄을 가리킵니다. 다음 줄은 `RUST_BACKTRACE` 환경 변수를 설정하여 오류의 원인이 된 정확한 역추적을 얻을 수 있음을 알려줍니다. 역 *추적*이 지점에 도달하기 위해 호출된 모든 함수의 목록입니다. Rust의 백트레이스는 다른 언어에서와 마찬가지로 작동합니다. 백트레이스를 읽는 핵심은 맨 위에서 시작하여 작성한 파일이 보일 때까지 읽는 것입니다. 그곳이 문제의 발원지입니다. 해당 지점 위의 줄은 코드가 호출한 코드입니다. 아래 줄은 코드를 호출한 코드입니다. 이러한 전후 줄에는 핵심 Rust 코드, 표준 라이브러리 코드 또는 사용 중인 크레이트가 포함될 수 있습니다. `RUST_BACKTRACE` 환경 변수를 0을 제외한 모든 값으로 설정하여 백트레이스를 가져오도록 합시다. Listing 9-2는 여러분이 보게 될 것과 유사한 출력을 보여줍니다.

```bash
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:142:14
   2: core::panicking::panic_bounds_check
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:84:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:242:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/alloc/src/vec/mod.rs:2591:9
   6: panic::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/ops/function.rs:248:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

Listing 9-2: `panic!` 호출에 의해 생성된 백트레이스 환경 변수 `RUST_BACKTRACE`가 설정되면 표시됩니다.

출력이 엄청나네요! 표시되는 정확한 출력은 운영 체제 및 Rust 버전에 따라 다를 수 있습니다. 이 정보로 백트레이스를 얻으려면 디버그 기호를 활성화해야 합니다. 여기에 있는 것처럼 `--release` 플래그 없이 `cargo build` 또는 `cargo run`을 사용할 때 디버그 기호가 기본적으로 활성화됩니다.

목록 9-2의 출력에서 백트레이스의 6행은 문제를 일으키는 프로젝트의 *src/main.rs* 4행을 가리킵니다. 프로그램이 패닉 상태가 되는 것을 원하지 않으면 우리가 작성한 파일을 언급하는 첫 번째 줄이 가리키는 위치에서 조사를 시작해야 합니다. 목록 9-1에서 일부러 패닉을 일으키는 코드를 작성했는데 패닉을 해결하는 방법은 벡터 인덱스 범위를 벗어나는 요소를 요청하지 않는 것입니다. 나중에 코드 패닉이 발생하면 패닉을 유발하는 값과 대신 코드가 수행해야 하는 작업을 파악해야 합니다.

`패닉!`으로 돌아오겠습니다. `패닉!`을 사용해야 할 때와 사용하지 말아야 할 때 [`패닉!](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic) ` 또는 이 장 뒷부분의 [`패닉!`` 섹션을 참조하십시오 . ](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic)다음으로 `결과`를 사용하여 오류를 복구하는 방법을 살펴보겠습니다.

------

## [`결과`로 복구 가능한 오류](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#recoverable-errors-with-result)

대부분의 오류는 프로그램을 완전히 중지해야 할 정도로 심각하지 않습니다. 때때로 함수가 실패하면 쉽게 해석하고 대응할 수 있는 이유 때문입니다. 예를 들어 파일을 열려고 하는데 파일이 없기 때문에 해당 작업이 실패하는 경우 프로세스를 종료하는 대신 파일을 만들 수 있습니다.

2장의 [`결과`를 사용한 잠재적 실패 처리`](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result) 에서 `결과` 열거형이 다음과 같이 `Ok` 및 `Err`의 두 가지 변형을 갖는 것으로 정의되었음을 상기하십시오 .

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T`와 `E`는 제네릭 유형 매개변수입니다. 제네릭에 대해서는 10장에서 자세히 설명하겠습니다. 지금 알아야 할 것은 `T`가 성공 시 반환될 값의 유형을 나타낸다는 것입니다. `Ok` 변형 내의 경우이고 `E`는 `Err` 변형 내의 실패 사례에서 반환될 오류 유형을 나타냅니다. `Result`에는 이러한 제네릭 유형 매개변수가 있기 때문에 반환하려는 성공적인 값과 오류 값이 다를 수 있는 다양한 상황에서 `Result` 유형과 그에 정의된 함수를 사용할 수 있습니다.

함수가 실패할 수 있으므로 `결과` 값을 반환하는 함수를 호출해 보겠습니다. 목록 9-3에서 우리는 파일 열기를 시도합니다.

파일 이름: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open(`hello.txt`);
}
```

목록 9-3: 파일 열기

`File::open`의 반환 유형은 `Result<T, E>`입니다. 일반 매개변수 `T`는 파일 핸들인 성공 값 `std::fs::File`의 유형으로 `File::open` 구현으로 채워졌습니다. 오류 값에 사용된 `E` 유형은 `std::io::Error`입니다. 이 반환 유형은 `File::open`에 대한 호출이 성공하고 읽고 쓸 수 있는 파일 핸들을 반환할 수 있음을 의미합니다. 함수 호출도 실패할 수 있습니다. 예를 들어 파일이 존재하지 않거나 파일에 액세스할 수 있는 권한이 없을 수 있습니다. `File::open` 함수는 성공 또는 실패 여부를 알려주는 동시에 파일 핸들 또는 오류 정보를 제공하는 방법이 필요합니다. 이 정보는 정확히 `결과` 열거형이 전달하는 것입니다.

`File::open`이 성공한 경우 변수 `greeting_file_result`의 값은 파일 핸들을 포함하는 `Ok`의 인스턴스가 됩니다. 실패한 경우 `greeting_file_result`의 값은 발생한 오류 종류에 대한 자세한 정보를 포함하는 `Err`의 인스턴스가 됩니다.

`File::open` 반환 값에 따라 다른 작업을 수행하려면 목록 9-3의 코드에 추가해야 합니다. 목록 9-4는 기본 도구인 6장에서 논의한 `일치` 표현식을 사용하여 `결과`를 처리하는 한 가지 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open(`hello.txt`);

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!(`Problem opening the file: {:?}`, error),
    };
}
```

목록 9-4: 반환될 수 있는 `결과` 변형을 처리하기 위해 `일치` 표현식 사용

`Option` 열거형과 마찬가지로 `Result` 열거형과 그 변형은 prelude에 의해 범위로 가져왔으므로 `Ok` 및 `Err` 변형 앞에 `Result::`를 지정할 필요가 없습니다. `일치`팔에서.

결과가 `Ok`이면 이 코드는 `Ok` 변형에서 내부 `file` 값을 반환한 다음 해당 파일 핸들 값을 변수 `greeting_file`에 할당합니다. `일치` 후에 파일 핸들을 사용하여 읽거나 쓸 수 있습니다.

`match`의 다른 부분은 `File::open`에서 `Err` 값을 얻는 경우를 처리합니다. 이 예에서는 `패닉!` 매크로. *현재 디렉터리에 hello.txt* 라는 파일이 없고 이 코드를 실행하면 `패닉!` 매크로:

```bash
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/error-handling`
thread 'main' panicked at 'Problem opening the file: Os { code: 2, kind: NotFound, message: `No such file or directory` }', src/main.rs:8:23
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

늘 그렇듯이 이 출력은 무엇이 잘못되었는지 정확히 알려줍니다.

### [다른 오류에 대한 일치](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#matching-on-different-errors)

목록 9-4의 코드는 `패닉!` `파일::열기`가 실패한 이유에 관계없이. 그러나 실패 이유에 따라 다른 작업을 수행하려고 합니다. 파일이 없기 때문에 `File::open`이 실패하면 파일을 만들고 새 파일에 대한 핸들을 반환하려고 합니다. 다른 이유로 `File::open`이 실패한 경우(예: 파일을 열 수 있는 권한이 없었기 때문에) 우리는 여전히 코드가 `패닉!` Listing 9-4에서와 같은 방식으로. 이를 위해 목록 9-5에 표시된 내부 `일치` 표현식을 추가합니다.

파일 이름: src/main.rs

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open(`hello.txt`);

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create(`hello.txt`) {
                Ok(fc) => fc,
                Err(e) => panic!(`Problem creating the file: {:?}`, e),
            },
            other_error => {
                panic!(`Problem opening the file: {:?}`, other_error);
            }
        },
    };
}
```

Listing 9-5: 다양한 종류의 오류를 다양한 방식으로 처리하기

`File::open`이 `Err` 변형 내에서 반환하는 값의 유형은 표준 라이브러리에서 제공하는 구조체인 `io::Error`입니다. 이 구조체에는 `io::ErrorKind` 값을 가져오기 위해 호출할 수 있는 `kind` 메서드가 있습니다. 열거형 `io::ErrorKind`는 표준 라이브러리에서 제공되며 `io` 작업에서 발생할 수 있는 다양한 종류의 오류를 나타내는 변형이 있습니다. 우리가 사용하려는 변형은 `ErrorKind::NotFound`이며 열려고 하는 파일이 아직 존재하지 않음을 나타냅니다. 따라서 `greeting_file_result`에서 일치하지만 `error.kind()`에서도 내부 일치가 있습니다.

내부 일치에서 확인하려는 조건은 `error.kind()`에 의해 반환된 값이 `ErrorKind` 열거형의 `NotFound` 변형인지 여부입니다. 그렇다면 `File::create`로 파일 생성을 시도합니다. 그러나 `File::create`도 실패할 수 있으므로 내부 `일치` 식에 두 번째 팔이 필요합니다. 파일을 만들 수 없으면 다른 오류 메시지가 인쇄됩니다. 외부 `일치`의 두 번째 팔은 동일하게 유지되므로 누락된 파일 오류 외에 다른 오류가 발생하면 프로그램 패닉이 발생합니다.

> ### [`결과`와 함께 `일치` 사용에 대한 대안](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#alternatives-to-using-match-with-resultt-e)
>
> 그것은 많은 `일치`입니다! `일치` 표현은 매우 유용하지만 매우 원시적이기도 합니다. 13장에서는 `Result<T, E>`에 정의된 많은 메서드와 함께 사용되는 클로저에 대해 배웁니다. 이러한 메서드는 코드에서 `Result<T, E>` 값을 처리할 때 `match`를 사용하는 것보다 더 간결할 수 있습니다.
>
> 예를 들어 Listing 9-5에 표시된 것과 동일한 논리를 작성하는 또 다른 방법이 있습니다. 이번에는 클로저와 `unwrap_or_else` 메서드를 사용합니다.
>
> ```rust
> use std::fs::File;
> use std::io::ErrorKind;
> 
> fn main() {
>     let greeting_file = File::open(`hello.txt`).unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create(`hello.txt`).unwrap_or_else(|error| {
>                 panic!(`Problem creating the file: {:?}`, error);
>             })
>         } else {
>             panic!(`Problem opening the file: {:?}`, error);
>         }
>     });
> }
> ```
>
> 이 코드는 목록 9-5와 동일한 동작을 갖지만 `일치` 표현식을 포함하지 않으며 더 읽기 쉽습니다. 13장을 읽은 후 이 예제로 돌아와서 표준 라이브러리 문서에서 `unwrap_or_else` 메서드를 찾아보십시오. 오류를 처리할 때 더 많은 이러한 메서드를 사용하여 거대한 중첩된 `일치` 식을 정리할 수 있습니다.

### [Panic on Error에 대한 바로 가기: `unwrap` 및 `expect`](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#shortcuts-for-panic-on-error-unwrap-and-expect)

`일치`를 사용하는 것은 충분히 잘 작동하지만 약간 장황할 수 있고 항상 의도를 잘 전달하지 못합니다. `Result<T, E>` 유형에는 다양하고 보다 구체적인 작업을 수행하기 위해 정의된 많은 도우미 메서드가 있습니다. `unwrap` 메소드는 Listing 9-4에서 작성한 `match` 표현식처럼 구현된 단축 메소드입니다. `결과` 값이 `Ok` 변형인 경우 `unwrap`은 `Ok` 내부의 값을 반환합니다. `Result`가 `Err` 변형인 경우 `unwrap`은 `panic!` 우리를 위한 매크로 다음은 작동 중인 `포장 해제`의 예입니다.

파일 이름: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open(`hello.txt`).unwrap();
}
```

*hello.txt* 파일 없이 이 코드를 실행하면 `패닉!` `unwrap` 메소드가 만드는 호출:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os {
code: 2, kind: NotFound, message: `No such file or directory` }',
src/main.rs:4:49
```

마찬가지로 `예상` 방법을 사용하면 `패닉!` 에러 메시지. `unwrap` 대신 `expect`를 사용하고 좋은 오류 메시지를 제공하면 의도를 전달할 수 있고 패닉의 원인을 더 쉽게 추적할 수 있습니다. `예상` 구문은 다음과 같습니다.

파일 이름: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open(`hello.txt`)
        .expect(`hello.txt should be included in this project`);
}
```

우리는 `unwrap`과 같은 방식으로 `expect`를 사용합니다: 파일 핸들을 반환하거나 `panic!` 매크로. `panic!` 호출에서 `expect`가 사용하는 오류 메시지 기본 `panic!`이 아니라 `expect`에 전달하는 매개변수가 됩니다. `unwrap`이 사용하는 메시지. 다음과 같습니다.

```text
thread 'main' panicked at 'hello.txt should be included in this project: Os {
code: 2, kind: NotFound, message: `No such file or directory` }',
src/main.rs:5:10
```

프로덕션 품질 코드에서 대부분의 Rustacean은 `unwrap`이 아닌 `expect`를 선택하고 작업이 항상 성공할 것으로 예상되는 이유에 대해 더 많은 컨텍스트를 제공합니다. 이렇게 하면 가정이 잘못된 것으로 판명될 경우 디버깅에 사용할 더 많은 정보를 얻을 수 있습니다.

### [오류 전파](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#propagating-errors)

함수의 구현이 실패할 수 있는 것을 호출할 때 함수 자체 내에서 오류를 처리하는 대신 호출 코드에 오류를 반환하여 수행할 작업을 결정할 수 있습니다. 이를 오류 *전파* 라고 하며 코드 컨텍스트에서 사용할 수 있는 것보다 오류를 처리하는 방법을 지시하는 더 많은 정보 또는 논리가 있을 수 있는 호출 코드에 더 많은 제어를 제공합니다.

예를 들어 Listing 9-6은 파일에서 사용자 이름을 읽는 함수를 보여줍니다. 파일이 존재하지 않거나 읽을 수 없는 경우 이 함수는 함수를 호출한 코드에 해당 오류를 반환합니다.

파일 이름: src/main.rs

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open(`hello.txt`);

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

목록 9-6: `match`를 사용하여 호출 코드에 오류를 반환하는 함수

이 함수는 훨씬 더 짧은 방법으로 작성할 수 있지만 오류 처리를 탐색하기 위해 수동으로 많은 작업을 수행하는 것으로 시작할 것입니다. 마지막에는 더 짧은 방법을 보여드리겠습니다. 먼저 함수의 반환 유형인 `Result<String, io::Error>`를 살펴보겠습니다. 이는 함수가 `Result<T, E>` 유형의 값을 반환하고 있음을 의미합니다. 여기서 일반 매개변수 `T`는 구체적인 유형 `String`으로 채워지고 일반 유형 `E`는 다음으로 채워졌습니다. 구체적인 유형 `io::Error`.

이 함수가 아무 문제 없이 성공하면 이 함수를 호출하는 코드는 이 함수가 파일에서 읽은 사용자 이름인 `문자열`을 포함하는 `Ok` 값을 받습니다. 이 함수에 문제가 발생하면 호출 코드는 문제가 무엇인지에 대한 자세한 정보가 포함된 `io::Error` 인스턴스를 보유하는 `Err` 값을 받습니다. 이 함수의 반환 유형으로 `io::Error`를 선택한 이유는 실패할 수 있는 이 함수의 본문에서 호출하는 두 작업에서 반환된 오류 값의 유형이기 때문입니다. ` 함수 및 `read_to_string` 메서드.

함수 본문은 `File::open` 함수를 호출하여 시작합니다. 그런 다음 목록 9-4의 `일치`와 유사한 `일치`로 `결과` 값을 처리합니다. `File::open`이 성공하면 패턴 변수 `file`의 파일 핸들이 가변 변수 `username_file`의 값이 되고 함수가 계속됩니다. `Err`의 경우 `panic!`을 호출하는 대신 `return` 키워드를 사용하여 함수 전체에서 일찍 반환하고 `File::open`의 오류 값을 이제 패턴 변수 `e`에 전달합니다. `, 이 함수의 오류 값으로 호출 코드로 돌아갑니다.

따라서 `username_file`에 파일 핸들이 있으면 함수는 `username` 변수에 새 `String`을 생성하고 `username_file`의 파일 핸들에서 `read_to_string` 메서드를 호출하여 파일 내용을 ` 사용자 이름`. `read_to_string` 메서드도 `File::open`이 성공했지만 실패할 수 있으므로 `Result`를 반환합니다. 따라서 `결과`를 처리하기 위해 또 다른 `일치`가 필요합니다. `read_to_string`이 성공하면 함수가 성공한 것이며 이제 `Ok`로 래핑된 `username`에 있는 파일에서 사용자 이름을 반환합니다. `read_to_string`이 실패하면 `match`에서 오류 값을 반환한 것과 동일한 방식으로 오류 값을 반환합니다. `File::open`의 반환 값을 처리했습니다. 그러나 `return`이라고 명시적으로 말할 필요는 없습니다. 이것이 함수의 마지막 표현식이기 때문입니다.

이 코드를 호출하는 코드는 사용자 이름이 포함된 `Ok` 값 또는 `io::Error`가 포함된 `Err` 값을 가져오는 것을 처리합니다. 해당 값으로 수행할 작업을 결정하는 것은 호출 코드에 달려 있습니다. 호출 코드가 `Err` 값을 받으면 `패닉!`을 호출할 수 있습니다. 예를 들어 프로그램을 충돌시키거나, 기본 사용자 이름을 사용하거나, 파일이 아닌 다른 곳에서 사용자 이름을 조회합니다. 호출 코드가 실제로 무엇을 하려고 하는지에 대한 정보가 충분하지 않으므로 적절하게 처리할 수 있도록 모든 성공 또는 오류 정보를 위쪽으로 전파합니다.

이러한 오류 전파 패턴은 Rust에서 매우 일반적이어서 Rust는 물음표 연산자 `?`를 제공합니다. 이것을 더 쉽게 하기 위해.

#### [전파 오류의 바로 가기: `?` 운영자](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator)

Listing 9-7은 Listing 9-6과 동일한 기능을 가진 `read_username_from_file`의 구현을 보여주지만 이 구현은 `?` 운영자.

파일 이름: src/main.rs

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open(`hello.txt`)?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

목록 9-7: `?`를 사용하여 호출 코드에 오류를 반환하는 함수 운영자

`?` Listing 9-6에서 `결과` 값을 처리하기 위해 정의한 `일치` 표현식과 거의 동일한 방식으로 작동하도록 정의된 `결과` 값 뒤에 배치됩니다. `Result`의 값이 `Ok`이면 `Ok` 내부의 값이 이 식에서 반환되고 프로그램이 계속됩니다. 값이 `Err`이면 `return` 키워드를 사용한 것처럼 전체 함수에서 `Err`가 반환되어 오류 값이 호출 코드로 전파됩니다.

목록 9-6의 `일치` 표현식이 수행하는 것과 `?` 연산자는 다음을 수행합니다. `?`가 있는 오류 값 호출된 연산자는 표준 라이브러리의 `From` 특성에 정의된 `from` 함수를 거치며 값을 한 유형에서 다른 유형으로 변환하는 데 사용됩니다. 때 `?` 연산자가 `from` 함수를 호출하면 받은 오류 유형이 현재 함수의 반환 유형에 정의된 오류 유형으로 변환됩니다. 이는 여러 가지 이유로 부품이 실패하더라도 함수가 실패할 수 있는 모든 방식을 나타내기 위해 함수가 하나의 오류 유형을 반환할 때 유용합니다.

예를 들어 목록 9-7의 `read_username_from_file` 함수를 변경하여 우리가 정의한 `OurError`라는 사용자 지정 오류 유형을 반환할 수 있습니다. [` io::Error](io::Error) `에서 `OurError`의 인스턴스를 구성하기 위해 `impl From io::Error for OurError`도 정의하면 `?` `read_username_from_file` 본문의 연산자 호출은 함수에 더 이상 코드를 추가할 필요 없이 `from`을 호출하고 오류 유형을 변환합니다.

목록 9-7의 맥락에서 `?` `File::open` 호출이 끝나면 `Ok` 내부의 값을 `username_file` 변수로 반환합니다. 오류가 발생하면 `?` 연산자는 전체 함수에서 일찍 반환하고 호출 코드에 `Err` 값을 제공합니다. `?`에도 동일하게 적용됩니다. `read_to_string` 호출이 끝날 때.

`?` 연산자는 많은 상용구를 제거하고 이 함수의 구현을 더 간단하게 만듭니다. Listing 9-8과 같이 `?` 바로 뒤에 메소드 호출을 연결하여 이 코드를 더 단축할 수도 있습니다.

파일 이름: src/main.rs

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open(`hello.txt`)?.read_to_string(&mut username)?;

    Ok(username)
}
```

Listing 9-8: `?` 다음에 메서드 호출을 연결합니다. 운영자

`username`에서 새 `String` 생성을 함수의 시작 부분으로 이동했습니다. 그 부분은 변하지 않았습니다. 변수 `username_file`을 만드는 대신 `read_to_string`에 대한 호출을 `File::open(`hello.txt`)?`의 결과에 직접 연결했습니다. 우리는 여전히 `?` `read_to_string` 호출이 끝날 때 `File::open`과 `read_to_string`이 모두 성공하면 오류를 반환하는 대신 `username`을 포함하는 `Ok` 값을 반환합니다. 기능은 Listing 9-6 및 Listing 9-7과 동일합니다. 이것은 그것을 작성하는 다른, 더 인체 공학적인 방법입니다.

목록 9-9는 `fs::read_to_string`을 사용하여 이것을 더 짧게 만드는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string(`hello.txt`)
}
```

목록 9-9: 파일을 열고 읽는 대신 `fs::read_to_string` 사용

파일을 문자열로 읽는 것은 상당히 일반적인 작업이므로 표준 라이브러리는 편리한 `fs::read_to_string` 함수를 제공하여 파일을 열고 새 `문자열`을 만들고 파일의 내용을 읽고 그 내용을 `문자열`, 반환합니다. 물론 `fs::read_to_string`을 사용하는 것은 모든 오류 처리를 설명할 기회를 주지 않으므로 먼저 긴 방법을 사용했습니다.

#### [`?` 통신수는 사용될 수 있습니다](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#where-the--operator-can-be-used)

`?` 연산자는 반환 유형이 `?` 값과 호환되는 함수에서만 사용할 수 있습니다. 에 사용됩니다. 이것은 `?` 때문입니다. 연산자는 Listing 9-6에서 정의한 `일치` 표현식과 같은 방식으로 함수 외부에서 값의 조기 반환을 수행하도록 정의됩니다. 목록 9-6에서 `일치`는 `결과` 값을 사용하고 있었고 초기 반환 암은 `Err(e)` 값을 반환했습니다. 함수의 반환 유형은 이 `반환`과 호환되도록 `결과`여야 합니다.

목록 9-10에서 `?`를 사용하면 발생하는 오류를 살펴보겠습니다. 우리가 사용하는 값의 유형과 호환되지 않는 반환 유형이 있는 `main` 함수의 연산자 `?` 에:

파일 이름: src/main.rs

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open(`hello.txt`)?;
}
```

목록 9-10: `?` 사용 시도 `()`를 반환하는 `main` 함수에서 컴파일되지 않습니다.

이 코드는 실패할 수 있는 파일을 엽니다. `?` 연산자는 `File::open`에 의해 반환된 `결과` 값을 따르지만 이 `주요` 함수의 반환 유형은 `결과`가 아니라 `()`입니다. 이 코드를 컴파일하면 다음과 같은 오류 메시지가 나타납니다.

```bash
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
 --> src/main.rs:4:48
  |
3 | fn main() {
  | --------- this function should return `Result` or `Option` to accept `?`
4 |     let greeting_file = File::open(`hello.txt`)?;
  |                                                ^ cannot use the `?` operator in a function that returns `()`
  |
  = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `error-handling` due to previous error
```

이 오류는 `?`만 사용할 수 있음을 나타냅니다. `Result`, `Option` 또는 `FromResidual`을 구현하는 다른 유형을 반환하는 함수의 연산자.

오류를 수정하려면 두 가지 선택이 있습니다. 한 가지 선택은 `?`를 사용하는 값과 호환되도록 함수의 반환 유형을 변경하는 것입니다. 제한이 없는 한 연산자를 켜십시오. 다른 기술은 `methods to handle the`적절한 방식으로 `일치` 또는 `Result<T, E> Result<T, E>` 중 하나를 사용하는 것입니다.

값과 함께 `?`사용할 수 있는 오류 메시지도 언급되었습니다. on 을 `Option<T>`사용할 때와 마찬가지로 를 반환하는 함수에서만 on 을 사용할 수 있습니다. an에서 호출될 때 연산자 의 동작은 a에서 호출될 때의 동작과 유사합니다. 값이 이면 해당 시점의 함수에서 이 일찍 반환됩니다. 값이 인 경우 내부 값은 표현식의 결과 값이고 함수는 계속됩니다. 목록 9-11에는 주어진 텍스트에서 첫 줄의 마지막 문자를 찾는 함수의 예가 있습니다:`?``Result``?``Option``Option``?``Option<T>``Result<T, E>``None``None``Some``Some`

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

Listing 9-11: 값 `?`에 연산자 사용하기`Option<T>`

이 함수는 `Option<char>`거기에 문자가 있을 수도 있고 없을 수도 있기 때문에 반환합니다. 이 코드는 `text`문자열 슬라이스 인수를 사용하고 `lines`문자열의 행에 대한 반복자를 반환하는 메서드를 호출합니다. 이 함수는 첫 번째 줄을 검사하기를 원하기 때문에 `next`반복자에서 첫 번째 값을 가져오기 위해 반복자를 호출합니다. `text`가 빈 문자열인 경우 에 대한 이 호출은 를 `next`반환합니다 `None`. 이 경우 `?`중지하고 `None`에서 반환하는 데 사용합니다 `last_char_of_first_line`. `text`가 빈 문자열이 아닌 경우 에서 첫 번째 줄의 문자열 조각을 포함하는 값을 `next`반환합니다.`Some``text`

는 `?`문자열 조각을 추출하고 `chars`해당 문자열 조각을 호출하여 해당 문자의 이터레이터를 얻을 수 있습니다. 이 첫 줄의 마지막 문자에 관심이 있으므로 `last`iterator의 마지막 항목을 반환하도록 호출합니다. 예 를 들어 에서와 같이 빈 줄로 시작하지만 다른 줄에 문자가 있는 경우 `Option`첫 번째 줄이 빈 문자열일 가능성이 있기 때문 입니다. 그러나 첫 번째 줄에 마지막 문자가 있는 경우에는 Variant로 반환됩니다. 중간에 있는 연산자는 이 논리를 간결하게 표현하는 방법을 제공하여 함수를 한 줄로 구현할 수 있도록 합니다. on 연산자를 사용할 수 없다면 더 많은 메서드 호출을 사용하여 이 논리를 구현해야 합니다.`text```\nhi```Some``?``?``Option``match`표현.

를 반환하는 함수에서 `?`a에 연산자를 사용할 수 있고 를 반환하는 함수에서 an에 연산자를 사용할 수 있지만 혼합하여 일치시킬 수는 없습니다. 연산자 는 a를 an으로 또는 그 반대로 자동으로 변환하지 않습니다. 이 경우 on 메서드 또는 on 메서드와 같은 메서드를 사용하여 명시적으로 변환을 수행할 수 있습니다.`Result``Result``?``Option``Option``?``Result``Option``ok``Result``ok_or``Option`

지금까지 `main`사용한 모든 함수는 return 입니다 `()`. 이 `main`함수는 실행 가능한 프로그램의 시작 및 종료 지점이기 때문에 특별하며 프로그램이 예상대로 작동할 수 있는 반환 유형에 대한 제한이 있습니다.

`main`운 좋게도 `Result<(), E>`. `main`Listing 9-12에는 Listing 9-10의 코드가 있지만 반환 유형 of를 be로 변경 하고 끝에 `Result<(), Box<dyn Error>>`반환 값을 추가했습니다. `Ok(())`이 코드는 이제 컴파일됩니다.

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open(`hello.txt`)?;

    Ok(())
}
```

Listing 9-12: `main`return으로 변경하면 값 에 연산자 `Result<(), E>`를 사용할 수 있습니다.`?``Result`

유형 `Box<dyn Error>`은 17장의 [`다른 유형의 값을 허용하는 특성 개체 사용`](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) 섹션 에서 이야기할 *특성 개체* 입니다. 지금은 `모든 종류의 오류`를 의미한다고 읽을 수 있습니다. 오류 유형이 있는 함수 에서 값을 사용 하는 것은 모든 값이 조기에 반환될 수 있기 때문에 허용됩니다. 이 함수의 본문은 유형의 오류만 반환하지만 을 지정하면 다른 오류를 반환하는 더 많은 코드가 의 본문에 추가되더라도 이 서명은 계속 정확합니다.`Box<dyn Error>``?``Result``main``Box<dyn Error>``Err``main``std::io::Error``Box<dyn Error>``main`

```
main`함수가 를 반환 하면 실행 파일은 if 반환 `Result<(), E>`값으로 종료 하고 값을 반환 하면 0이 아닌 값으로 종료합니다. C로 작성된 실행 파일은 종료 시 정수를 반환합니다. 성공적으로 종료된 프로그램은 정수를 반환 하고 오류가 발생한 프로그램은 . Rust는 또한 이 규칙과 호환되도록 실행 파일에서 정수를 반환합니다.`0``main``Ok(())``main``Err``0``0
```

함수 는 를 반환하는 함수를 포함하는 [특성 ](https://doc.rust-lang.org/std/process/trait.Termination.html)[을](https://doc.rust-lang.org/std/process/trait.Termination.html)`main` 구현하는 모든 유형을 반환할 수 있습니다. 자신의 유형에 대한 특성 구현에 대한 자세한 내용은 표준 라이브러리 문서를 참조하세요 .[`std::process::Termination`](https://doc.rust-lang.org/std/process/trait.Termination.html)`report``ExitCode``Termination`

지금까지 를 호출 `panic!`하거나 반환하는 방법에 대해 자세히 살펴보았 `Result`으므로 어떤 경우에 어떤 것을 사용하는 것이 적절한지 결정하는 방법에 대한 주제로 돌아가 보겠습니다.

------

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

------

# [일반 유형, 특성 및 수명](https://doc.rust-lang.org/book/ch10-00-generics.html#generic-types-traits-and-lifetimes)

모든 프로그래밍 언어에는 개념의 중복을 효과적으로 처리하기 위한 도구가 있습니다. Rust에서 그러한 도구 중 하나는 *제네릭* 입니다 : 구체적인 유형 또는 기타 속성에 대한 추상 대체물입니다. 코드를 컴파일하고 실행할 때 제네릭의 위치에 무엇이 있는지 모른 채 제네릭의 동작 또는 제네릭이 다른 제네릭과 어떻게 관련되는지 표현할 수 있습니다.

`i32`함수는 or 와 같은 구체적인 유형 대신 일부 일반 유형의 매개변수를 사용할 수 있습니다 `String`. 이는 함수가 알 수 없는 값을 가진 매개변수를 사용하여 여러 구체적인 값에서 동일한 코드를 실행하는 것과 같은 방식입니다. 사실 우리는 이미 6장에서 `Option<T>`, 8장에서 `Vec<T>`및 `HashMap<K, V>`, 9장에서 제네릭을 사용했습니다 `Result<T, E>`. 이 장에서는 제네릭을 사용하여 고유한 유형, 함수 및 메서드를 정의하는 방법을 살펴봅니다!

먼저 코드 중복을 줄이기 위해 함수를 추출하는 방법을 검토합니다. 그런 다음 동일한 기술을 사용하여 매개 변수 유형만 다른 두 함수에서 일반 함수를 만듭니다. 또한 구조체 및 열거형 정의에서 제네릭 형식을 사용하는 방법도 설명합니다.

*그런 다음 특성을* 사용하여 일반적인 방식으로 동작을 정의하는 방법을 배웁니다. 특성을 일반 유형과 결합하여 모든 유형이 아닌 특정 동작이 있는 유형만 허용하도록 일반 유형을 제한할 수 있습니다.

*마지막으로 수명에* 대해 논의할 것입니다. 참조가 서로 어떻게 관련되어 있는지에 대한 컴파일러 정보를 제공하는 다양한 제네릭입니다. 수명을 통해 빌린 값에 대한 충분한 정보를 컴파일러에 제공하여 우리의 도움 없이 할 수 있는 것보다 더 많은 상황에서 참조가 유효하도록 할 수 있습니다.

## [함수를 추출하여 중복 제거](https://doc.rust-lang.org/book/ch10-00-generics.html#removing-duplication-by-extracting-a-function)

제네릭을 사용하면 특정 유형을 여러 유형을 나타내는 자리 표시자로 대체하여 코드 중복을 제거할 수 있습니다. 제네릭 구문을 살펴보기 전에 먼저 특정 값을 여러 값을 나타내는 자리 표시자로 대체하는 함수를 추출하여 제네릭 형식을 포함하지 않는 방식으로 중복을 제거하는 방법을 살펴보겠습니다. 그런 다음 동일한 기술을 적용하여 일반 함수를 추출합니다! 함수로 추출할 수 있는 중복 코드를 인식하는 방법을 살펴보면 제네릭을 사용할 수 있는 중복 코드를 인식하기 시작할 것입니다.

목록에서 가장 큰 숫자를 찾는 목록 10-1의 짧은 프로그램으로 시작합니다.

파일 이름: src/main.rs

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!(`The largest number is {}`, largest);
}
```

Listing 10-1: 숫자 목록에서 가장 큰 숫자 찾기

정수 목록을 변수에 저장 `number_list`하고 목록의 첫 번째 숫자에 대한 참조를 라는 변수에 배치합니다 `largest`. 그런 다음 목록의 모든 숫자를 반복하고 현재 숫자가 에 저장된 숫자보다 크면 `largest`해당 변수의 참조를 바꿉니다. 그러나 현재 숫자가 지금까지 본 가장 큰 숫자보다 작거나 같으면 변수가 변경되지 않고 코드가 목록의 다음 숫자로 이동합니다. 목록의 모든 숫자를 고려한 후 `largest`가장 큰 숫자(이 경우 100)를 참조해야 합니다.

우리는 이제 두 개의 서로 다른 숫자 목록에서 가장 큰 숫자를 찾는 임무를 받았습니다. 이를 위해 Listing 10-1의 코드를 복제하고 Listing 10-2와 같이 프로그램의 서로 다른 두 위치에서 동일한 논리를 사용하도록 선택할 수 있습니다.

파일 이름: src/main.rs

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!(`The largest number is {}`, largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!(`The largest number is {}`, largest);
}
```

*목록 10-2: 두 개의* 숫자 목록 에서 가장 큰 숫자를 찾는 코드

이 코드는 작동하지만 코드 복제는 지루하고 오류가 발생하기 쉽습니다. 또한 코드를 변경하고 싶을 때 여러 위치에서 코드를 업데이트해야 한다는 점을 기억해야 합니다.

이 중복을 제거하기 위해 매개변수에 전달된 정수 목록에서 작동하는 함수를 정의하여 추상화를 만듭니다. 이 솔루션은 코드를 더 명확하게 만들고 목록에서 가장 큰 숫자를 찾는 개념을 추상적으로 표현할 수 있게 해줍니다.

Listing 10-3에서는 가장 큰 수를 찾는 코드를 이라는 함수로 추출합니다 `largest`. 그런 다음 목록 10-2의 두 목록에서 가장 큰 숫자를 찾는 함수를 호출합니다. `i32`우리는 미래에 가질 수 있는 다른 값 목록에서도 이 함수를 사용할 수 있습니다.

파일 이름: src/main.rs

```rust
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!(`The largest number is {}`, result);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!(`The largest number is {}`, result);
}
```

Listing 10-3: 두 목록에서 가장 큰 숫자를 찾는 추상화된 코드

이 함수에는 함수에 전달할 수 있는 구체적인 값 조각을 나타내는 이라는 `largest`매개변수가 있습니다. 결과적으로 함수를 호출하면 코드는 전달한 특정 값에서 실행됩니다.`list``i32`

요약하면 Listing 10-2에서 Listing 10-3으로 코드를 변경하기 위해 수행한 단계는 다음과 같습니다.

1. 중복 코드를 식별합니다.
2. 중복 코드를 함수 본문으로 추출하고 함수 서명에서 해당 코드의 입력 및 반환 값을 지정합니다.
3. 대신 함수를 호출하도록 중복 코드의 두 인스턴스를 업데이트합니다.

다음으로 제네릭과 동일한 단계를 사용하여 코드 중복을 줄입니다. 함수 본문이 `list`특정 값 대신 추상에서 작동할 수 있는 것과 같은 방식으로 제네릭을 사용하면 코드가 추상 유형에서 작동할 수 있습니다.

예를 들어, 값 조각에서 가장 큰 항목을 찾는 함수 `i32`와 값 조각에서 가장 큰 항목을 찾는 함수의 두 가지 함수가 있다고 가정합니다 `char`. 그 중복을 어떻게 제거할까요? 알아 보자!

------

## [일반 데이터 유형](https://doc.rust-lang.org/book/ch10-01-syntax.html#generic-data-types)

제네릭을 사용하여 함수 서명 또는 구조체와 같은 항목에 대한 정의를 만든 다음 다양한 구체적인 데이터 유형과 함께 사용할 수 있습니다. 제네릭을 사용하여 함수, 구조체, 열거형 및 메서드를 정의하는 방법을 먼저 살펴보겠습니다. 그런 다음 제네릭이 코드 성능에 미치는 영향에 대해 설명합니다.

### [함수 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-function-definitions)

제네릭을 사용하는 함수를 정의할 때 일반적으로 매개 변수의 데이터 유형과 반환 값을 지정하는 함수 서명에 제네릭을 배치합니다. 이렇게 하면 코드가 더 유연해지고 함수 호출자에게 더 많은 기능을 제공하는 동시에 코드 중복을 방지할 수 있습니다.

우리의 `largest`함수에 대해 계속해서 Listing 10-4는 슬라이스에서 가장 큰 값을 찾는 두 함수를 보여줍니다. 그런 다음 이들을 제네릭을 사용하는 단일 함수로 결합합니다.

파일 이름: src/main.rs

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!(`The largest number is {}`, result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!(`The largest char is {}`, result);
}
```

목록 10-4: 서명의 이름과 유형만 다른 두 함수

이 함수는 슬라이스에서 `largest_i32`가장 큰 것을 찾는 Listing 10-3에서 추출한 것입니다. `i32`이 함수는 슬라이스에서 `largest_char`가장 큰 것을 찾습니다. `char`함수 본문은 동일한 코드를 가지고 있으므로 단일 함수에 제네릭 형식 매개 변수를 도입하여 중복을 제거하겠습니다.

새로운 단일 함수에서 유형을 매개변수화하려면 함수에 대한 값 매개변수와 마찬가지로 유형 매개변수의 이름을 지정해야 합니다. 모든 식별자를 유형 매개변수 이름으로 사용할 수 있습니다. `T`그러나 관례에 따라 Rust의 유형 매개변수 이름은 짧고 종종 문자일 뿐이며 Rust의 유형 이름 지정 규칙은 UpperCamelCase이기 때문에 사용할 것입니다. `유형`의 줄임말은 `T`대부분의 Rust 프로그래머가 기본적으로 선택하는 것입니다.

함수 본문에서 매개변수를 사용할 때 컴파일러가 해당 이름의 의미를 알 수 있도록 서명에 매개변수 이름을 선언해야 합니다. 유사하게 함수 시그니처에서 타입 매개변수 이름을 사용할 때 사용하기 전에 타입 매개변수 이름을 선언해야 합니다. 제네릭 함수를 정의하려면 다음과 같이 함수 이름과 매개변수 목록 사이의 `largest`꺾쇠 괄호 안에 유형 이름 선언을 배치합니다.`<>`

```rust
fn largest<T>(list: &[T]) -> &T {
```

우리는 이 정의를 다음과 같이 읽습니다. 함수는 `largest`일부 유형에 대해 일반적입니다 `T`. 이 함수에는 `list`유형 값의 슬라이스인 이라는 이름의 매개변수가 하나 있습니다 `T`. 이 `largest`함수는 같은 유형의 값에 대한 참조를 반환합니다 `T`.

`largest`Listing 10-5는 서명에 일반 데이터 유형을 사용하는 결합된 함수 정의를 보여줍니다. `i32`이 목록은 값 조각 또는 `char`값으로 함수를 호출하는 방법도 보여줍니다. 이 코드는 아직 컴파일되지 않지만 이 장의 뒷부분에서 수정할 것입니다.

파일 이름: src/main.rs

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!(`The largest number is {}`, result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!(`The largest char is {}`, result);
}
```

목록 10-5: `largest`일반 유형 매개변수를 사용하는 함수; 이것은 아직 컴파일되지 않습니다

지금 이 코드를 컴파일하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error
```

도움말 텍스트는 특성인 을 언급하며 `std::cmp::PartialOrd`다음 *섹션* 에서 특성에 대해 이야기할 것입니다. 지금은 이 오류가 가능한 모든 유형에 대해 본문이 `largest`작동하지 않음을 나타냅니다 `T`. `T`본문에 있는 유형의 값을 비교하고 싶기 때문에 값을 정렬할 수 있는 유형만 사용할 수 있습니다. 비교를 가능하게 하기 위해 표준 라이브러리에는 `std::cmp::PartialOrd`유형에 대해 구현할 수 있는 특성이 있습니다(이 특성에 대한 자세한 내용은 부록 C 참조). 도움말 텍스트의 제안에 따라 유효한 유형을 `T`구현하는 유형으로만 제한 `PartialOrd`하고 이 예제는 컴파일할 것입니다. 표준 라이브러리는 및 `PartialOrd`모두에서 구현하기 때문입니다.`i32``char`

### [구조체 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-struct-definitions)

구문을 사용하여 하나 이상의 필드에서 일반 유형 매개변수를 사용하도록 구조체를 정의할 수도 있습니다 `<>`. 목록 10-6은 모든 유형의 값을 `Point<T>`보유 `x`하고 `y`조정하는 구조체를 정의합니다.

파일 이름: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

목록 10-6: 유형의 값을 보유 `Point<T>`하는 구조체`x``y``T`

구조체 정의에서 제네릭을 사용하는 구문은 함수 정의에서 사용되는 구문과 유사합니다. 먼저 구조체 이름 바로 뒤에 꺾쇠 괄호 안에 유형 매개변수의 이름을 선언합니다. 그런 다음 구체적인 데이터 유형을 지정하는 구조체 정의에서 일반 유형을 사용합니다.

를 정의하기 위해 하나의 제네릭 유형만 사용했기 때문에 `Point<T>`이 정의에서는 `Point<T>`구조체가 일부 유형에 대해 제네릭 `T`하고 필드 `x`및 필드 `y`가 유형에 관계없이 *동일한* 유형임을 나타냅니다. 목록 10-7에서와 같이 다른 유형의 값을 가진 a 인스턴스를 생성하면 `Point<T>`코드가 컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

목록 10-7: 필드 `x`및 필드는 `y`둘 다 동일한 일반 데이터 유형을 갖기 때문에 동일한 유형이어야 합니다 `T`.

이 예에서 정수 값 5를 에 할당하면 제네릭 형식이 의 이 인스턴스에 대한 정수가 될 것임을 `x`컴파일러에 알립니다. 그런 다음 와 동일한 유형을 갖도록 정의한 에 대해 4.0을 지정하면 다음과 같은 유형 불일치 오류가 발생합니다.`T``Point<T>``y``x`

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found floating-point number

For more information about this error, try `rustc --explain E0308`.
error: could not compile `chapter10` due to previous error
```

및 가 모두 제네릭이지만 다른 유형을 가질 수 있는 `Point`구조체를 정의하려면 여러 제네릭 유형 매개변수를 사용할 수 있습니다. 예를 들어 Listing 10-8에서 유형에 대한 정의를 일반으로 변경하고 여기서 is of type 및 is of type 입니다.`x``y``Point``T``U``x``T``y``U`

파일 이름: src/main.rs

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

목록 10-8: 서로 다른 유형의 값이 될 수 `Point<T, U>`있도록 두 가지 유형에 대한 제네릭`x``y`

이제 표시된 모든 인스턴스가 `Point`허용됩니다! 정의에서 제네릭 형식 매개 변수를 원하는 만큼 사용할 수 있지만 몇 개 이상 사용하면 코드를 읽기 어려워집니다. 코드에 많은 제네릭 형식이 필요한 경우 코드를 더 작은 조각으로 재구성해야 함을 나타낼 수 있습니다.

### [열거형 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-enum-definitions)

구조체와 마찬가지로 열거형을 정의하여 변형에 일반 데이터 유형을 담을 수 있습니다. `Option<T>`6장에서 사용한 표준 라이브러리가 제공하는 열거형을 다시 살펴보겠습니다.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

이제 이 정의가 더 이해하기 쉬울 것입니다. 보시 `Option<T>`다시피 열거형은 유형에 대해 일반적 `T`이며 두 가지 변형이 있습니다. `Some`하나의 유형 값을 보유하는 `T`과 `None`값을 보유하지 않는 변형입니다. 열거형 을 사용하면 `Option<T>`옵셔널 값의 추상적인 개념을 표현할 수 있고 `Option<T>`제네릭이기 때문에 옵셔널 값의 타입에 관계없이 이 추상을 사용할 수 있습니다.

열거형은 여러 제네릭 형식도 사용할 수 있습니다. `Result`9장에서 사용한 enum 의 정의는 한 가지 예입니다.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

열거 형은 두 가지 유형 및 `Result`에 대해 일반적 이며 두 가지 변형이 있습니다. 유형의 값을 보유하는 과 유형의 값을 보유하는 . 이 정의는 성공(일부 유형의 값 반환 ) 또는 실패(일부 유형의 오류 반환 ) 작업이 있는 모든 곳에서 enum 을 사용하는 것을 편리하게 만듭니다. 사실 이것은 목록 9-3에서 파일을 여는 데 사용한 것입니다. 파일이 성공적으로 열렸을 때 유형으로 채워지고 파일을 여는 데 문제가 있을 때 유형으로 채워졌습니다.`T``E``Ok``T``Err``E``Result``T``E``T``std::fs::File``E``std::io::Error`

보유한 값의 유형만 다른 여러 구조체 또는 열거형 정의가 있는 코드의 상황을 인식하는 경우 대신 제네릭 유형을 사용하여 중복을 방지할 수 있습니다.

### [메서드 정의에서](https://doc.rust-lang.org/book/ch10-01-syntax.html#in-method-definitions)

우리는 구조체와 열거형에 메서드를 구현할 수 있고(5장에서 했던 것처럼) 정의에 제네릭 유형을 사용할 수도 있습니다. Listing 10-9는 `Point<T>`Listing 10-6에서 정의한 구조체에 `x`구현된 메서드를 보여줍니다.

파일 이름: src/main.rs

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!(`p.x = {}`, p.x());
}
```

목록 10-9: 유형 필드 에 대한 참조를 반환하는 구조체 `x`에 명명된 메서드 구현`Point<T>``x``T`

여기에서 필드의 데이터에 대한 참조를 반환하는 `x`on 이라는 메서드를 정의했습니다.`Point<T>``x`

유형에 메서드를 구현하고 있음을 지정하는 데 사용할 수 있도록 `T`바로 뒤에 선언해야 합니다. 뒤에 일반 유형으로 선언함으로써 Rust는 꺾쇠 괄호 안에 있는 유형이 구체적인 유형이 아닌 일반 유형임을 식별할 수 있습니다. 이 일반 매개변수에 대해 구조체 정의에 선언된 일반 매개변수와 다른 이름을 선택할 수 있지만 동일한 이름을 사용하는 것이 관례입니다. 제네릭 형식을 선언하는 에서 작성된 메서드는 어떤 구체적인 형식이 제네릭 형식을 대체하는지에 관계없이 해당 형식의 모든 인스턴스에서 정의됩니다.`impl``T``Point<T>``T``impl``Point``impl`

유형에 대한 메소드를 정의할 때 제네릭 유형에 대한 제약 조건을 지정할 수도 있습니다. 예를 들어 제네릭 유형이 있는 `Point<f32>`인스턴스가 아닌 인스턴스에서만 메서드를 구현할 수 있습니다. `Point<T>`Listing 10-10에서 구체적인 type 을 사용합니다 `f32`. 즉, . 뒤에 어떤 유형도 선언하지 않습니다 `impl`.

파일 이름: src/main.rs

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

Listing 10-10: `impl`일반 유형 매개변수에 대한 특정 구체적인 유형을 가진 구조체에만 적용되는 블록`T`

이 코드는 유형 에 메소드가 `Point<f32>`있음 을 의미합니다 `distance_from_origin`. 유형이 아닌 `Point<T>`다른 인스턴스에는 이 메소드가 정의되지 않습니다. 이 메서드는 포인트가 좌표(0.0, 0.0)의 포인트에서 얼마나 떨어져 있는지 측정하고 부동 소수점 유형에만 사용할 수 있는 수학 연산을 사용합니다.`T``f32`

구조체 정의의 일반 유형 매개변수는 동일한 구조체의 메서드 서명에서 사용하는 매개변수와 항상 동일하지 않습니다. 목록 10-11은 예제를 더 명확하게 하기 위해 일반 유형 `X1`과 `Y1`구조체 `Point`및 `X2` `Y2`메서드 서명을 사용합니다. `mixup`메서드는 ( type ) 의 값 과 전달된 값 ( type ) 을 `Point`사용하여 새 인스턴스를 만듭니다.`x``self` `Point``X1``y``Point``Y2`

파일 이름: src/main.rs

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: `Hello`, y: 'c' };

    let p3 = p1.mixup(p2);

    println!(`p3.x = {}, p3.y = {}`, p3.x, p3.y);
}
```

목록 10-11: 구조체의 정의와 다른 제네릭 유형을 사용하는 메서드

에서 for (값 포함 ) 및 for (값 포함 ) 가 있는 `main`a를 정의했습니다. 변수 는 for (값 포함 ) 및 for (값 포함 ) 문자열 슬라이스가 있는 구조체 입니다. 인수를 사용하여 호출 하면 for 가 있는 가 제공됩니다. 변수 는 for 를 갖게 됩니다. from 에서 왔기 때문입니다. 매크로 호출이 인쇄됩니다.`Point``i32``x``5``f64``y``10.4``p2``Point``x```Hello```char``y``c``mixup``p1``p2``p3``i32``x``x``p1``p3``char``y``y``p2``println!``p3.x = 5, p3.y = c`

```
impl`이 예제의 목적은 일부 제네릭 매개 변수가 선언되고 일부는 메서드 정의로 선언되는 상황을 보여 주는 것입니다. 여기에서 일반 매개변수 `X1`및 는 구조체 정의와 함께 이동하기 때문에 `Y1`뒤에 선언됩니다. `impl`일반 매개변수 `X2`및 는 메서드에만 관련되기 때문에 `Y2`뒤에 선언됩니다.`fn mixup
```

### [제네릭을 사용한 코드 성능](https://doc.rust-lang.org/book/ch10-01-syntax.html#performance-of-code-using-generics)

제네릭 형식 매개 변수를 사용할 때 런타임 비용이 있는지 궁금할 수 있습니다. 좋은 소식은 제네릭 유형을 사용해도 구체적인 유형을 사용할 때보다 프로그램 실행 속도가 느려지지 않는다는 것입니다.

Rust는 컴파일 타임에 제네릭을 사용하여 코드의 단일형화를 수행함으로써 이를 달성합니다. *단일형화는* 컴파일할 때 사용되는 구체적인 유형을 채워 일반 코드를 특정 코드로 바꾸는 프로세스입니다. 이 프로세스에서 컴파일러는 Listing 10-5에서 제네릭 함수를 생성하는 데 사용한 단계와 반대로 수행합니다. 컴파일러는 제네릭 코드가 호출되는 모든 위치를 살펴보고 제네릭 코드가 호출되는 구체적인 유형에 대한 코드를 생성합니다. .

표준 라이브러리의 일반 열거형을 사용하여 이것이 어떻게 작동하는지 살펴보겠습니다 `Option<T>`.

```rust
let integer = Some(5);
let float = Some(5.0);
```

Rust는 이 코드를 컴파일할 때 단일형화를 수행합니다. 이 과정에서 컴파일러는 인스턴스에서 사용된 값을 읽고 `Option<T>`두 가지 종류를 식별합니다 `Option<T>`. 하나는 이고 `i32`다른 하나는 입니다 `f64`. 따라서 의 일반 정의를 및 `Option<T>`에 특화된 두 가지 정의로 확장하여 일반 정의를 특정 정의로 바꿉니다.`i32``f64`

코드의 단일형 버전은 다음과 유사합니다(컴파일러는 설명을 위해 여기에서 사용하는 이름과 다른 이름을 사용함).

파일 이름: src/main.rs

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

제네릭은 `Option<T>`컴파일러에서 만든 특정 정의로 대체됩니다. Rust는 제네릭 코드를 각 인스턴스의 유형을 지정하는 코드로 컴파일하기 때문에 제네릭 사용에 대한 런타임 비용을 지불하지 않습니다. 코드가 실행되면 각 정의를 직접 복사한 것처럼 수행됩니다. 단일형화 프로세스는 Rust의 제네릭을 실행 시간에 매우 효율적으로 만듭니다.

------

## [특성: 공유 행동 정의](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-defining-shared-behavior)

*특성은* 특정 유형이 가지고 있고 다른 유형과 공유할 수 있는 기능을 정의합니다. 특성을 사용하여 추상적인 방식으로 공유 동작을 정의할 수 있습니다. *특성 경계를* 사용하여 제네릭 유형이 특정 동작이 있는 모든 유형이 될 수 있음을 지정할 수 있습니다.

> 참고: 특성은 약간의 차이는 있지만 종종 다른 언어에서 *인터페이스* 라고 하는 기능과 유사합니다.

### [특성 정의](https://doc.rust-lang.org/book/ch10-02-traits.html#defining-a-trait)

유형의 동작은 해당 유형에서 호출할 수 있는 메서드로 구성됩니다. 모든 유형에서 동일한 메서드를 호출할 수 있는 경우 다른 유형은 동일한 동작을 공유합니다. 특성 정의는 어떤 목적을 달성하는 데 필요한 일련의 동작을 정의하기 위해 메서드 서명을 함께 그룹화하는 방법입니다.

예를 들어 다양한 종류와 양의 텍스트를 포함하는 여러 구조체가 있다고 가정해 보겠습니다. `NewsArticle`특정 위치에 보관된 뉴스 기사를 포함하는 구조체와 `Tweet`새 트윗인지 여부를 나타내는 메타데이터와 함께 최대 280자를 포함할 수 있는 구조체가 있습니다. , 리트윗 또는 다른 트윗에 대한 답글.

또는 인스턴스 `aggregator`에 저장될 수 있는 데이터의 요약을 표시할 수 있는 이름이 지정된 미디어 수집기 라이브러리 크레이트를 만들고 싶습니다. 이렇게 하려면 각 유형의 요약이 필요하며 인스턴스에서 메서드를 호출하여 해당 요약을 요청합니다. 목록 10-12는 이 동작을 표현하는 공개 특성의 정의를 보여줍니다.`NewsArticle``Tweet``summarize``Summary`

파일 이름: src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

Listing 10-12: 메서드가 `Summary`제공하는 동작으로 구성된 트레이트`summarize`

여기에서 우리는 키워드를 사용하여 특성을 선언 한 다음 이 경우 `trait`특성의 이름을 사용합니다. 우리는 또한 몇 가지 예에서 볼 수 있듯이 이 상자에 의존하는 상자가 이 특성을 사용할 수 있도록 `Summary`특성을 로 선언했습니다. `pub`중괄호 안에는 이 특성을 구현하는 유형의 동작을 설명하는 메서드 서명을 선언합니다. 이 경우에는 `fn summarize(&self) -> String`.

메서드 서명 뒤에 중괄호 안에 구현을 제공하는 대신 세미콜론을 사용합니다. 이 특성을 구현하는 각 유형은 메서드 본문에 대한 고유한 사용자 지정 동작을 제공해야 합니다. 컴파일러는 특성이 있는 모든 유형이 이 서명으로 정의된 `Summary`메서드를 갖도록 강제합니다.`summarize`

특성은 본문에 여러 메서드를 가질 수 있습니다. 메서드 시그니처는 한 줄에 하나씩 나열되고 각 줄은 세미콜론으로 끝납니다.

### [유형에 특성 구현](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type)

이제 특성 메서드의 원하는 서명을 정의했으므로 `Summary`미디어 수집기의 유형에 구현할 수 있습니다. 목록 10-13 은 헤드라인, 작성자 및 위치를 사용하여 의 반환 값을 생성하는 구조체 `Summary`의 특성 구현을 보여줍니다. 구조체의 경우 트윗 콘텐츠가 이미 280자로 제한되어 있다고 가정하고 트윗의 전체 텍스트가 뒤에 오는 사용자 이름으로 정의합니다.`NewsArticle``summarize``Tweet``summarize`

파일 이름: src/lib.rs

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!(`{}, by {} ({})`, self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!(`{}: {}`, self.username, self.content)
    }
}
```

Listing 10-13: 및 유형 `Summary`에 대한 특성 구현`NewsArticle``Tweet`

유형에 특성을 구현하는 것은 일반 메서드를 구현하는 것과 유사합니다. 차이점은 뒤에 `impl`구현하려는 특성 이름을 입력한 다음 `for`키워드를 사용한 다음 특성을 구현하려는 유형의 이름을 지정한다는 것입니다. 블록 내에서 `impl`특성 정의가 정의한 메서드 시그니처를 넣습니다. 각 서명 뒤에 세미콜론을 추가하는 대신 중괄호를 사용하고 특성의 메서드가 특정 유형에 대해 갖고 싶은 특정 동작으로 메서드 본문을 채웁니다.

이제 라이브러리가 및 `Summary`에 대해 특성을 구현했으므로 크레이트 사용자는 일반 메서드를 호출하는 것과 같은 방식으로 및 의 인스턴스에서 특성 메서드를 호출할 수 있습니다. 유일한 차이점은 사용자가 특성과 유형을 범위로 가져와야 한다는 것입니다. 다음은 바이너리 크레이트가 라이브러리 크레이트를 어떻게 사용할 수 있는지에 대한 예입니다.`NewsArticle``Tweet``NewsArticle``Tweet``aggregator`

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from(`horse_ebooks`),
        content: String::from(
            `of course, as you probably already know, people`,
        ),
        reply: false,
        retweet: false,
    };

    println!(`1 new tweet: {}`, tweet.summarize());
}
```

이 코드는 `1 new tweet: horse_ebooks: of course, as you probably already know, people`.

크레이트 에 의존하는 다른 크레이트 `aggregator`도 특성을 범위로 가져와 자체 유형에 `Summary`구현할 수 있습니다. `Summary`주의해야 할 한 가지 제한 사항은 트레이트 또는 유형 중 적어도 하나가 크레이트에 로컬인 경우에만 유형에 트레이트를 구현할 수 있다는 것입니다. 예를 들어, 유형이 우리 크레이트 에 로컬이기 때문에 크레이트 기능 의 일부로 `Display`사용자 정의 유형과 같은 표준 라이브러리 특성을 구현할 수 있습니다. 특성이 우리 크레이트에 로컬이기 때문에 우리는 크레이트에서 on을 구현할 수도 있습니다.`Tweet``aggregator``Tweet``aggregator``Summary``Vec<T>``aggregator``Summary``aggregator`

그러나 외부 유형에 외부 특성을 구현할 수는 없습니다. 예를 들어, and 가 표준 라이브러리에 정의되어 있고 우리 크레이트에 국한되지 않기 때문에 우리는 크레이트 내에서 `Display`특성을 구현할 수 없습니다. *이 제한은 coherence* 라는 속성의 일부 이며, 특히 부모 유형이 없기 때문에 이름이 지정된 *고아 규칙 의 일부입니다.* 이 규칙은 다른 사람의 코드가 귀하의 코드를 손상시킬 수 없으며 그 반대의 경우도 마찬가지입니다. 규칙이 없으면 두 개의 크레이트가 동일한 유형에 대해 동일한 특성을 구현할 수 있으며 러스트는 어떤 구현을 사용할지 알 수 없습니다.`Vec<T>``aggregator``Display``Vec<T>``aggregator`

### [기본 구현](https://doc.rust-lang.org/book/ch10-02-traits.html#default-implementations)

때로는 모든 유형의 모든 메소드에 대한 구현을 요구하는 대신 특성의 일부 또는 모든 메소드에 대한 기본 동작을 갖는 것이 유용합니다. 그런 다음 특정 유형에 특성을 구현하면서 각 메서드의 기본 동작을 유지하거나 재정의할 수 있습니다.

Listing 10-14에서 우리는 Listing 10-12에서 했던 것처럼 메서드 서명만 정의하는 대신 특성의 `summarize`메서드 에 대한 기본 문자열을 지정합니다.`Summary`

파일 이름: src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from(`(Read more...)`)
    }
}
```

Listing 10-14: 메서드 `Summary`의 기본 구현으로 트레이트 정의하기`summarize`

의 인스턴스를 요약하기 위해 기본 구현을 사용하려면 로 `NewsArticle`빈 블록을 지정합니다.`impl``impl Summary for NewsArticle {}`

```
summarize`더 이상 메서드를 직접 정의하지 않지만 기본 구현을 제공하고 특성을 구현하도록 `NewsArticle`지정했습니다. 결과적으로 다음과 같이 의 인스턴스에서 메서드를 계속 호출할 수 있습니다.`NewsArticle``Summary``summarize``NewsArticle
    let article = NewsArticle {
        headline: String::from(`Penguins win the Stanley Cup Championship!`),
        location: String::from(`Pittsburgh, PA, USA`),
        author: String::from(`Iceburgh`),
        content: String::from(
            `The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.`,
        ),
    };

    println!(`New article available! {}`, article.summarize());
```

이 코드는 `New article available! (Read more...)`.

`Summary`기본 구현을 만들려면 Listing 10-13의 on 구현에 대해 아무것도 변경할 필요가 없습니다 `Tweet`. 그 이유는 기본 구현을 재정의하는 구문이 기본 구현이 없는 특성 메서드를 구현하는 구문과 동일하기 때문입니다.

기본 구현은 다른 메서드에 기본 구현이 없더라도 동일한 트레이트에서 다른 메서드를 호출할 수 있습니다. 이러한 방식으로 특성은 많은 유용한 기능을 제공할 수 있으며 구현자는 그 중 일부만 지정하면 됩니다. 예를 들어 구현이 필요한 메서드를 `Summary`갖도록 특성을 정의한 `summarize_author`다음 `summarize`해당 메서드를 호출하는 기본 구현이 있는 메서드를 정의할 수 있습니다 `summarize_author`.

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!(`(Read more from {}...)`, self.summarize_author())
    }
}
```

이 버전의 를 사용하려면 유형에 특성을 구현할 때 `Summary`정의하기만 하면 됩니다.`summarize_author`

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!(`@{}`, self.username)
    }
}
```

를 정의한 후 구조체의 인스턴스를 `summarize_author`호출할 수 있으며 의 기본 구현은 우리가 제공한 정의를 호출합니다. 우리가 구현했기 때문에 특성 은 더 이상 코드를 작성할 필요 없이 메서드 의 동작을 제공했습니다.`summarize``Tweet``summarize``summarize_author``summarize_author``Summary``summarize`

```rust
    let tweet = Tweet {
        username: String::from(`horse_ebooks`),
        content: String::from(
            `of course, as you probably already know, people`,
        ),
        reply: false,
        retweet: false,
    };

    println!(`1 new tweet: {}`, tweet.summarize());
```

이 코드는 `1 new tweet: (Read more from @horse_ebooks...)`.

동일한 메서드의 재정의 구현에서 기본 구현을 호출하는 것은 불가능합니다.

### [매개변수로서의 특성](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters)

이제 특성을 정의하고 구현하는 방법을 알았으므로 특성을 사용하여 다양한 유형을 허용하는 함수를 정의하는 방법을 탐색할 수 있습니다. 우리는 목록 10-13의 및 유형에서 `Summary`구현한 특성을 사용하여 특성을 구현하는 유형의 매개변수 에서 메소드를 호출하는 함수를 정의합니다. 이를 위해 다음과 같은 구문을 사용합니다.`NewsArticle``Tweet``notify``summarize``item``Summary``impl Trait`

```rust
pub fn notify(item: &impl Summary) {
    println!(`Breaking news! {}`, item.summarize());
}
```

매개변수 에 대한 구체적인 유형 대신 키워드와 특성 이름을 `item`지정합니다. `impl`이 매개변수는 지정된 특성을 구현하는 모든 유형을 허용합니다. 의 본문에서 와 같이 특성 에서 오는 `notify`모든 메서드를 호출할 수 있습니다. 또는 의 모든 인스턴스를 호출하고 전달할 수 있습니다. a 또는 an 과 같은 다른 유형으로 함수를 호출하는 코드는 해당 유형이 를 구현하지 않기 때문에 컴파일되지 않습니다.`item``Summary``summarize``notify``NewsArticle``Tweet``String``i32``Summary`

#### [특성 바인딩 구문](https://doc.rust-lang.org/book/ch10-02-traits.html#trait-bound-syntax)

구문 은 간단한 경우에 작동하지만 실제로는 *특성 경계*`impl Trait` 로 알려진 더 긴 형식의 구문 설탕입니다. 다음과 같이 보입니다.

```rust
pub fn notify<T: Summary>(item: &T) {
    println!(`Breaking news! {}`, item.summarize());
}
```

이 긴 형식은 이전 섹션의 예와 동일하지만 더 장황합니다. 콜론과 꺾쇠 괄호 안에 제네릭 형식 매개 변수 선언과 함께 특성 범위를 배치합니다.

구문 `impl Trait`은 편리하고 단순한 경우에 더 간결한 코드를 작성하는 반면, 더 완전한 특성 바인딩 구문은 다른 경우에 더 복잡함을 표현할 수 있습니다. 예를 들어 `Summary`. 구문을 사용하면 `impl Trait`다음과 같습니다.

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

`impl Trait`이 함수가 서로 다른 유형을 허용 `item1`하고 가지도록 하려면 사용하는 것이 적절합니다 `item2`(두 유형 모두 구현하는 한 `Summary`). 그러나 두 매개변수가 동일한 유형을 갖도록 하려면 다음과 같이 특성 경계를 사용해야 합니다.

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

및 매개 변수 `T`의 유형으로 지정된 제네릭 유형은 및 에 대한 인수로 전달된 값의 구체적인 유형이 동일해야 하도록 함수를 제한합니다.`item1``item2``item1``item2`

#### [`+`구문을 사용하여 여러 특성 경계 지정](https://doc.rust-lang.org/book/ch10-02-traits.html#specifying-multiple-trait-bounds-with-the--syntax)

하나 이상의 특성 경계를 지정할 수도 있습니다. on `notify`뿐만 아니라 표시 형식을 사용하고 싶다고 가정해 보겠습니다. 정의 에서 and 를 모두 구현해야 한다고 지정합니다. 다음 구문을 사용하여 그렇게 할 수 있습니다.`summarize``item``notify``item``Display``Summary``+`

```rust
pub fn notify(item: &(impl Summary + Display)) {
```

구문 `+`은 제네릭 형식에 대한 특성 범위에서도 유효합니다.

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

두 개의 특성 경계가 지정되면 의 본문은 format 을 `notify`호출 `summarize`하고 사용할 수 있습니다.`{}``item`

#### [`where`절로 더 명확한 특성 경계](https://doc.rust-lang.org/book/ch10-02-traits.html#clearer-trait-bounds-with-where-clauses)

너무 많은 특성 범위를 사용하면 단점이 있습니다. 각 제네릭에는 고유한 특성 경계가 있으므로 여러 제네릭 유형 매개변수가 있는 함수는 함수 이름과 해당 매개변수 목록 사이에 많은 특성 경계 정보를 포함할 수 있으므로 함수 서명을 읽기 어렵게 만듭니다. 이러한 이유로 Rust는 `where`함수 서명 뒤의 절 내부에 특성 범위를 지정하기 위한 대체 구문을 가지고 있습니다. 따라서 다음과 같이 작성하는 대신:

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

`where`다음과 같은 절을 사용할 수 있습니다.

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

이 함수의 시그니처는 덜 복잡합니다. 함수 이름, 매개변수 목록 및 반환 유형이 서로 가깝고 많은 특성 범위가 없는 함수와 비슷합니다.

### [특성을 구현하는 반환 형식](https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits)

`impl Trait`다음과 같이 반환 위치의 구문을 사용하여 특성을 구현하는 일부 유형의 값을 반환할 수도 있습니다.

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from(`horse_ebooks`),
        content: String::from(
            `of course, as you probably already know, people`,
        ),
        reply: false,
        retweet: false,
    }
}
```

`impl Summary`반환 유형에 를 사용하여 함수가 구체적인 유형의 이름을 지정하지 않고 특성을 `returns_summarizable`구현하는 일부 유형을 반환하도록 지정합니다. `Summary`이 경우 는 `returns_summarizable`를 반환 `Tweet`하지만 이 함수를 호출하는 코드는 이를 알 필요가 없습니다.

구현하는 특성에 의해서만 반환 유형을 지정하는 기능은 13장에서 다룰 클로저와 반복자의 맥락에서 특히 유용합니다. 클로저와 반복자는 컴파일러만 아는 유형이나 지정하기에 매우 긴 유형을 만듭니다. 구문 `impl Trait`을 사용하면 함수가 `Iterator`매우 긴 유형을 작성할 필요 없이 특성을 구현하는 일부 유형을 반환하도록 간결하게 지정할 수 있습니다.

그러나 `impl Trait`단일 유형을 반환하는 경우에만 사용할 수 있습니다. 예를 들어 반환 유형이 다음과 같이 지정된 a `NewsArticle`또는 a를 반환하는 다음 코드는 작동하지 않습니다.`Tweet``impl Summary`

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                `Penguins win the Stanley Cup Championship!`,
            ),
            location: String::from(`Pittsburgh, PA, USA`),
            author: String::from(`Iceburgh`),
            content: String::from(
                `The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.`,
            ),
        }
    } else {
        Tweet {
            username: String::from(`horse_ebooks`),
            content: String::from(
                `of course, as you probably already know, people`,
            ),
            reply: false,
            retweet: false,
        }
    }
}
```

구문이 컴파일러에서 구현되는 방식에 대한 제한으로 인해 a `NewsArticle`또는 a를 반환하는 것은 허용되지 않습니다. 17장의 [`다른 유형의 값을 허용하는 트레이트 개체 사용`](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) 섹션에서 이 동작을 사용하여 함수를 작성하는 방법을 다룰 것입니다.`Tweet``impl Trait`

### [특성 경계를 사용하여 메서드를 조건부로 구현](https://doc.rust-lang.org/book/ch10-02-traits.html#using-trait-bounds-to-conditionally-implement-methods)

제네릭 타입 매개변수를 사용 하는 블록에 바인딩된 트레이트를 사용함으로써 `impl`지정된 트레이트를 구현하는 타입에 대해 조건부로 메서드를 구현할 수 있습니다. 예를 들어 목록 10-15의 유형은 `Pair<T>`항상 `new`새로운 인스턴스를 반환하는 함수를 구현합니다 `Pair<T>`( 블록 유형에 대한 유형 별칭인 5장의 [`메소드 정의`](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#defining-methods) 섹션에서 기억하십시오 . 이 경우에는 입니다.) . 그러나 다음 블록 에서는 내부 유형이 비교를 가능하게 하는 특성 *과* 인쇄를 가능하게 하는 특성 을 구현하는 경우 에만 메소드를 구현합니다.`Self``impl``Pair<T>``impl``Pair<T>``cmp_display``T``PartialOrd``Display`

파일 이름: src/lib.rs

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!(`The largest member is x = {}`, self.x);
        } else {
            println!(`The largest member is y = {}`, self.y);
        }
    }
}
```

Listing 10-15: 트레잇 경계에 따라 제네릭 타입에 대한 조건부 구현 메서드

또한 다른 특성을 구현하는 모든 유형에 대한 특성을 조건부로 구현할 수 있습니다. 트레잇 범위를 만족하는 모든 유형에 대한 트레잇 구현을 *블랭킷 구현* 이라고 하며 Rust 표준 라이브러리에서 광범위하게 사용됩니다. 예를 들어 표준 라이브러리는 `ToString`특성을 구현하는 모든 유형에서 `Display`특성을 구현합니다. `impl`표준 라이브러리의 블록은 다음 코드와 유사합니다.

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

표준 라이브러리에는 이 포괄적인 구현이 있기 때문에 특성을 구현하는 모든 유형에서 특성 `to_string`에 의해 정의된 메서드를 호출할 수 있습니다. 예를 들어 정수는 다음을 구현하기 때문에 정수를 다음과 같이 해당 값 으로 변환할 수 있습니다.`ToString``Display``String``Display`

```rust
let s = 3.to_string();
```

일괄 구현은 `구현자` 섹션의 특성에 대한 문서에 나타납니다.

특성 및 특성 범위를 사용하면 제네릭 형식 매개 변수를 사용하여 중복을 줄이는 코드를 작성할 수 있을 뿐만 아니라 제네릭 형식이 특정 동작을 수행하도록 컴파일러에 지정할 수도 있습니다. 그런 다음 컴파일러는 특성 바인딩 정보를 사용하여 코드와 함께 사용되는 모든 구체적인 유형이 올바른 동작을 제공하는지 확인할 수 있습니다. 동적 형식 언어에서는 메서드를 정의하지 않은 형식에서 메서드를 호출하면 런타임에 오류가 발생합니다. 그러나 Rust는 이러한 오류를 컴파일 시간으로 이동하므로 코드가 실행되기 전에 문제를 수정해야 합니다. 또한 컴파일 타임에 이미 확인했기 때문에 런타임에 동작을 확인하는 코드를 작성할 필요가 없습니다. 이렇게 하면 제네릭의 유연성을 포기하지 않고도 성능이 향상됩니다.

------

## [수명으로 참조 유효성 검사](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes)

수명은 우리가 이미 사용하고 있는 또 다른 종류의 제네릭입니다. 타입이 우리가 원하는 행동을 하도록 보장하는 대신, 라이프타임은 참조가 우리가 필요로 하는 한 유효하도록 보장합니다.

[4장의 `참조 및 차용`](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html#references-and-borrowing) 섹션 에서 논의하지 않은 한 가지 세부 사항은 Rust의 모든 참조에는 해당 참조가 유효한 범위인 *수명이 있다는 것입니다.* 대부분의 경우 수명은 대부분의 경우 유형이 유추되는 것처럼 암시적이고 유추됩니다. 여러 유형이 가능한 경우에만 유형에 주석을 달아야 합니다. 비슷한 방식으로 참조의 수명이 몇 가지 다른 방식으로 관련될 수 있는 경우 수명에 주석을 달아야 합니다. Rust는 런타임에 사용되는 실제 참조가 확실히 유효하도록 일반 수명 매개변수를 사용하여 관계에 주석을 달도록 요구합니다.

수명에 주석을 다는 것은 대부분의 다른 프로그래밍 언어가 가지고 있는 개념도 아니므로 생소하게 느껴질 것입니다. 이 장에서 라이프타임 전체를 다루지는 않겠지만, 개념에 익숙해질 수 있도록 라이프타임 구문을 만날 수 있는 일반적인 방법에 대해 논의할 것입니다.

### [수명이 있는 댕글링 참조 방지](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#preventing-dangling-references-with-lifetimes)

수명의 주요 목표는 프로그램이 참조하려는 데이터가 아닌 다른 데이터를 참조하게 하는 *매달린 참조를 방지하는 것입니다.* 외부 범위와 내부 범위가 있는 Listing 10-16의 프로그램을 고려하십시오.

```rust
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!(`r: {}`, r);
}
```

목록 10-16: 값이 범위를 벗어난 참조를 사용하려는 시도

> 참고: Listings 10-16, 10-17 및 10-23의 예제는 초기 값을 제공하지 않고 변수를 선언하므로 변수 이름은 외부 범위에 존재합니다. 언뜻 보기에 이는 Rust가 null 값을 갖지 않는 것과 충돌하는 것처럼 보일 수 있습니다. 그러나 값을 주기 전에 변수를 사용하려고 하면 Rust가 실제로 null 값을 허용하지 않는다는 것을 보여주는 컴파일 타임 오류가 발생합니다.

외부 범위는 초기값이 없는 명명된 변수를 선언 `r`하고 내부 범위는 초기값 5로 명명된 변수를 선언합니다 `x`. 내부 범위 내에서 의 값을 에 대한 `r`참조로 설정하려고 시도합니다 `x`. 그런 다음 내부 범위가 종료되고 에 값을 인쇄하려고 시도합니다 `r`. `r`이 코드는 사용하려고 시도하기 전에 참조하는 값이 범위를 벗어났기 때문에 컴파일되지 않습니다. 오류 메시지는 다음과 같습니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!(`r: {}`, r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

변수는 `x``충분히 오래 살지` 않습니다. 그 이유는 `x`내부 범위가 7행에서 끝날 때 범위를 벗어나기 때문입니다. 그러나 `r`외부 범위에는 여전히 유효합니다. 그 범위가 더 크기 때문에 우리는 그것이 `더 오래 산다`고 말합니다. Rust가 이 코드가 작동하도록 허용하면 범위를 벗어날 `r`때 할당 해제된 메모리를 참조하게 되고 우리가 시도한 모든 작업이 올바르게 작동하지 않을 것입니다. 그렇다면 Rust는 이 코드가 유효하지 않다는 것을 어떻게 판단할까요? 차입 검사기를 사용합니다.`x``r`

### [차용 체커](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-borrow-checker)

Rust 컴파일러에는 범위를 비교하여 모든 차용이 유효한지 확인하는 *차용 검사기가 있습니다.* 목록 10-17은 목록 10-16과 동일한 코드를 보여주지만 변수의 수명을 보여주는 주석이 있습니다.

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!(`r: {}`, r); //          |
}                         // ---------+
```

`r`Listing 10-17: 및 `x`의 수명에 `'a`대한 `'b`주석

```
r`여기에서 with 의 수명 과 with `'a`의 수명에 주석을 달았습니다. 보시다시피 내부 블록은 외부 수명 블록보다 훨씬 작습니다. 컴파일 시간에 Rust는 두 수명의 크기를 비교하여 수명이 이지만 수명이 . 다음 보다 짧기 때문에 프로그램이 거부됩니다. 참조 대상이 참조만큼 오래 살지 않습니다.`x``'b``'b``'a``r``'a``'b``'b``'a
```

목록 10-18은 코드를 수정하여 매달린 참조가 없고 오류 없이 컴파일합니다.

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!(`r: {}`, r); //   |       |
                          // --+       |
}                         // ----------+
```

Listing 10-18: 데이터가 참조보다 수명이 길기 때문에 유효한 참조

여기서 `x`수명은 이며 `'b`, 이 경우에는 수명이 입니다 `'a`. 이것은 `r`참조할 수 있음을 의미합니다 `x`. Rust는 참조가 유효한 `r`동안 참조가 항상 유효하다는 것을 알고 있기 때문입니다.`x`

이제 참조의 수명이 어디에 있고 Rust가 참조가 항상 유효한지 확인하기 위해 수명을 분석하는 방법을 알았으므로 함수 컨텍스트에서 매개 변수의 일반적인 수명과 반환 값을 살펴보겠습니다.

### [함수의 일반 수명](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#generic-lifetimes-in-functions)

우리는 두 문자열 조각 중 더 긴 것을 반환하는 함수를 작성할 것입니다. 이 함수는 두 개의 문자열 슬라이스를 가져와 단일 문자열 슬라이스를 반환합니다. 함수 를 구현한 후 `longest`목록 10-19의 코드는 를 인쇄해야 합니다 `The longest string is abcd`.

파일 이름: src/main.rs

```rust
fn main() {
    let string1 = String::from(`abcd`);
    let string2 = `xyz`;

    let result = longest(string1.as_str(), string2);
    println!(`The longest string is {}`, result);
}
```

목록 10-19: 두 스트링 슬라이스 중 더 긴 것을 찾는 함수를 `main`호출하는 함수`longest`

`longest`우리는 함수가 매개 변수의 소유권을 갖는 것을 원하지 않기 때문에 함수가 문자열이 아닌 참조인 문자열 슬라이스를 취하기를 원한다는 점에 유의하십시오 . 목록 10-19에서 사용하는 매개변수가 우리가 원하는 매개변수인 이유에 대한 자세한 내용은 4장의 [`매개변수로서의 스트링 슬라이스`](https://doc.rust-lang.org/book/ch04-03-slices.html#string-slices-as-parameters) 섹션을 참조하십시오 .

Listing 10-20과 같이 함수를 구현하려고 하면 `longest`컴파일되지 않습니다.

파일 이름: src/main.rs

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

목록 10-20: `longest`두 문자열 슬라이스 중 더 긴 문자열을 반환하지만 아직 컴파일되지 않은 함수 의 구현

대신 수명에 대해 말하는 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
x`도움말 텍스트는 Rust가 반환되는 참조 가 또는 를 참조하는지 여부를 알 수 없기 때문에 반환 유형에 일반 수명 매개변수가 필요함을 나타냅니다 `y`. `if`실제로 이 함수 본문의 블록이 에 대한 참조를 반환 `x`하고 블록이 ! `else`에 대한 참조를 반환하기 때문에 우리도 모릅니다.`y
```

이 함수를 정의할 때 이 함수에 전달될 구체적인 값을 모르기 때문에 케이스 `if`또는 `else`케이스가 실행될지 알 수 없습니다. 우리는 또한 전달될 참조의 구체적인 수명을 모르기 때문에 목록 10-17 및 10-18에서 우리가 반환하는 참조가 항상 유효한지 여부를 결정하기 위해 범위를 볼 수 없습니다. . 차용 검사기는 반환 값의 수명이 어떻게 관련되어 있는지 `x`모르기 때문에 이것을 결정할 수 없습니다. `y`이 오류를 수정하기 위해 차용 검사기가 분석을 수행할 수 있도록 참조 간의 관계를 정의하는 일반 수명 매개변수를 추가합니다.

### [평생 주석 구문](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotation-syntax)

평생 주석은 참조가 지속되는 기간을 변경하지 않습니다. 오히려 수명에 영향을 주지 않고 서로에 대한 여러 참조의 수명 관계를 설명합니다. 서명이 일반 유형 매개변수를 지정하면 함수가 모든 유형을 허용할 수 있는 것처럼 함수는 일반 수명 매개변수를 지정하여 모든 수명의 참조를 허용할 수 있습니다.

수명 주석에는 약간 특이한 구문이 있습니다. 수명 매개변수의 이름은 아포스트로피( `'`)로 시작해야 하며 일반적으로 모두 소문자이며 일반 유형과 같이 매우 짧습니다. `'a`대부분의 사람들은 첫 번째 평생 주석의 이름을 사용합니다. `&`참조 유형에서 주석을 구분하기 위해 공백을 사용하여 참조 뒤에 수명 매개변수 주석을 배치합니다.

`i32`다음은 몇 가지 예입니다: 수명 매개변수가 없는 an에 대한 참조 , `i32`이라는 수명 매개변수가 있는 an에 대한 참조 `'a`, 그리고 `i32`life가 있는 an에 대한 변경 가능한 참조 `'a`.

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

하나의 라이프타임 주석 자체는 큰 의미가 없습니다. 왜냐하면 주석은 여러 참조의 일반적인 라이프타임 매개변수가 서로 어떻게 관련되어 있는지 Rust에 알리기 위한 것이기 때문입니다. 수명 주석이 함수 컨텍스트에서 서로 어떻게 관련되는지 살펴보겠습니다 `longest`.

### [함수 서명의 수명 주석](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-function-signatures)

*함수 서명에서 라이프타임 주석을 사용하려면 일반 유형 매개* 변수에서 했던 것처럼 함수 이름과 매개변수 목록 사이의 꺾쇠괄호 안에 일반 수명 *매개* 변수 를 선언해야 합니다.

서명이 다음 제약 조건을 표현하기를 원합니다. 반환된 참조는 두 매개 변수가 모두 유효한 한 유효합니다. 이것은 매개변수의 수명과 반환 값 간의 관계입니다. 목록 10-21에 표시된 대로 수명에 이름을 지정한 `'a`다음 각 참조에 추가합니다.

파일 이름: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

목록 10-21: `longest`서명의 모든 참조가 동일한 수명을 가져야 한다고 지정하는 함수 정의`'a`

`main`이 코드는 목록 10-19의 함수 와 함께 사용할 때 원하는 결과를 컴파일하고 생성해야 합니다.

함수 시그니처는 이제 러스트에게 어떤 수명 동안 `'a`함수가 두 개의 매개변수를 취한다고 알려줍니다. 둘 다 최소한 수명만큼 사는 문자열 조각입니다 `'a`. 함수 서명은 또한 Rust에게 함수에서 반환된 문자열 조각이 적어도 평생만큼은 살아있을 것이라고 알려줍니다 `'a`. 실제로 이는 함수가 반환하는 참조의 수명이 `longest`함수 인수가 참조하는 값의 수명 중 더 작은 수명과 동일함을 의미합니다. 이러한 관계는 Rust가 이 코드를 분석할 때 사용하기를 원하는 것입니다.

이 함수 서명에서 수명 매개변수를 지정할 때 전달되거나 반환된 값의 수명을 변경하지 않는다는 점을 기억하십시오. 오히려 우리는 차용 검사기가 이러한 제약 조건을 준수하지 않는 모든 값을 거부하도록 지정하고 있습니다. 함수 가 얼마나 오래 지속 되는지 `longest`정확히 알 필요는 없으며 이 서명을 충족하는 일부 범위만 대체할 수 있습니다.`x``y``'a`

함수의 수명에 주석을 추가할 때 주석은 함수 본문이 아닌 함수 서명에 포함됩니다. 수명 주석은 서명의 유형과 마찬가지로 함수 계약의 일부가 됩니다. 함수 서명이 수명 계약을 포함한다는 것은 Rust 컴파일러가 수행하는 분석이 더 간단할 수 있음을 의미합니다. 함수에 주석을 달거나 호출하는 방식에 문제가 있는 경우 컴파일러 오류는 코드의 일부와 제약 조건을 더 정확하게 가리킬 수 있습니다. 대신에 Rust 컴파일러가 우리가 의도한 수명 관계에 대해 더 많은 추론을 했다면, 컴파일러는 문제의 원인에서 몇 단계 떨어진 우리 코드의 사용만을 지적할 수 있을 것입니다.

에 대한 구체적인 참조를 전달할 때 `longest`대체되는 구체적인 수명은 의 범위 와 겹치는 범위 `'a`의 일부입니다. 즉, 일반 수명은 및 의 수명 중 더 작은 수명과 동일한 구체적인 수명을 갖게 됩니다. 동일한 수명 매개변수로 반환된 참조에 주석을 달았기 때문에 반환된 참조는 및 의 수명 중 더 작은 수명의 길이에 대해서도 유효합니다.`x``y``'a``x``y``'a``x``y`

`longest`구체적인 수명이 다른 참조를 전달하여 수명 주석이 함수를 제한하는 방법을 살펴보겠습니다. 목록 10-22는 간단한 예입니다.

파일 이름: src/main.rs

```rust
fn main() {
    let string1 = String::from(`long string is long`);

    {
        let string2 = String::from(`xyz`);
        let result = longest(string1.as_str(), string2.as_str());
        println!(`The longest string is {}`, result);
    }
}
```

목록 10-22: 구체적인 수명이 다른 값 `longest`에 대한 참조와 함께 함수 사용`String`

이 예에서 `string1`is는 외부 범위 끝까지 유효하고 `string2`내부 범위 끝까지 유효하며 `result`내부 범위 끝까지 유효한 것을 참조합니다. 이 코드를 실행하면 빌림 검사기가 승인하는 것을 볼 수 있습니다. 컴파일하고 인쇄합니다 `The longest string is long string is long`.

다음으로, 참조의 수명이 `result`두 인수의 더 작은 수명이어야 함을 보여주는 예제를 시도해 봅시다. 내부 범위 외부로 변수 선언을 이동 하지만 범위 내부의 변수 `result`에 대한 값 할당은 . 그런 다음 내부 범위가 종료된 후 사용하는 를 내부 범위 외부로 이동합니다. 목록 10-23의 코드는 컴파일되지 않습니다.`result``string2``println!``result`

파일 이름: src/main.rs

```rust
fn main() {
    let string1 = String::from(`long string is long`);
    let result;
    {
        let string2 = String::from(`xyz`);
        result = longest(string1.as_str(), string2.as_str());
    }
    println!(`The longest string is {}`, result);
}
```

목록 10-23: 범위를 벗어난 `result`후 사용 시도`string2`

이 코드를 컴파일하려고 하면 다음 오류가 발생합니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!(`The longest string is {}`, result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

이 오류는 for 가 문에 `result`대해 유효하려면 외부 범위가 끝날 때까지 유효해야 함을 보여줍니다. 러스트는 우리가 동일한 수명 매개변수를 사용하여 함수 매개변수와 반환 값의 수명에 주석을 달았기 때문에 이것을 알고 있습니다.`println!``string2``'a`

인간으로서 우리는 이 코드를 볼 수 있고 이것이 `string1`에 대한 참조를 포함할 것 `string2`입니다. 가 아직 범위를 벗어나지 않았기 때문에 에 대한 참조는 여전히 명령문에 유효합니다. 그러나 컴파일러는 이 경우 참조가 유효한지 확인할 수 없습니다. 우리는 러스트에게 함수에 의해 반환된 참조의 수명이 전달 된 참조의 수명 중 더 작은 수명과 같다고 말했습니다. 따라서 차용 검사기는 목록 10-23의 코드가 유효하지 않은 참조를 가질 수 있는 것으로 허용하지 않습니다.`result``string1``string1``string1``println!``longest`

함수에 전달된 참조의 값과 수명 및 `longest`반환된 참조가 사용되는 방법을 변경하는 더 많은 실험을 설계해 보십시오. 컴파일하기 전에 실험이 차용 검사기를 통과할지 여부에 대한 가설을 세우십시오. 그런 다음 당신이 옳은지 확인하십시오!

### [수명의 관점에서 생각하기](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#thinking-in-terms-of-lifetimes)

수명 매개변수를 지정하는 방법은 함수가 수행하는 작업에 따라 다릅니다. 예를 들어 가장 긴 문자열 조각이 아닌 항상 첫 번째 매개변수를 반환하도록 함수의 구현을 변경한 경우 매개변수 `longest`에 수명을 지정할 필요가 없습니다 `y`. 다음 코드는 컴파일됩니다.

파일 이름: src/main.rs

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
'a`매개변수 및 반환 유형에 대해 수명 매개변수를 지정했지만 `x`매개변수에 대해서는 수명 매개변수를 지정하지 않았습니다 `y`. 의 수명은 수명 또는 반환 값 `y`과 관련이 없기 때문입니다.`x
```

함수에서 참조를 반환할 때 반환 유형에 대한 수명 매개변수는 매개변수 중 하나에 대한 수명 매개변수와 일치해야 합니다. 반환된 참조가 매개 변수 중 하나를 참조 *하지 않는* 경우 이 함수 내에서 생성된 값을 참조해야 합니다. 그러나 이것은 값이 함수의 끝에서 범위를 벗어나기 때문에 댕글링 참조가 됩니다. `longest`컴파일되지 않는 함수 의 시도된 구현을 고려하십시오 .

파일 이름: src/main.rs

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from(`really long string`);
    result.as_str()
}
```

`'a`여기 에서 반환 유형에 대해 수명 매개변수를 지정했지만 반환 값 수명이 매개변수의 수명과 전혀 관련이 없기 때문에 이 구현은 컴파일에 실패합니다. 우리가 얻는 오류 메시지는 다음과 같습니다.

```bash
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return reference to local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error
```

문제는 `result`범위를 벗어나 함수가 끝날 때 정리된다는 것입니다 `longest`. `result`또한 함수에서 참조를 반환하려고 합니다. 댕글링 참조를 변경하는 수명 매개변수를 지정할 수 있는 방법이 없으며 Rust는 댕글링 참조를 생성하도록 허용하지 않습니다. 이 경우 가장 좋은 해결 방법은 참조가 아닌 소유한 데이터 유형을 반환하여 호출 함수가 값 정리를 담당하도록 하는 것입니다.

궁극적으로 수명 구문은 다양한 매개 변수의 수명과 함수의 반환 값을 연결하는 것입니다. 일단 연결되면 Rust는 메모리 안전 작업을 허용하고 댕글링 포인터를 생성하거나 메모리 안전을 위반하는 작업을 허용하지 않는 충분한 정보를 갖게 됩니다.

### [구조체 정의의 수명 주석](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-struct-definitions)

지금까지 우리가 정의한 모든 구조체는 소유 유형을 보유합니다. 참조를 보유하도록 구조체를 정의할 수 있지만 이 경우 구조체 정의의 모든 참조에 수명 주석을 추가해야 합니다. `ImportantExcerpt`목록 10-24 에는 스트링 슬라이스를 보유하는 명명된 구조체가 있습니다.

파일 이름: src/main.rs

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from(`Call me Ishmael. Some years ago...`);
    let first_sentence = novel.split('.').next().expect(`Could not find a '.'`);
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

목록 10-24: 수명 주석이 필요한 참조를 보유하는 구조체

`part`이 구조체에는 참조인 문자열 슬라이스를 포함하는 단일 필드가 있습니다. 일반 데이터 유형과 마찬가지로 구조체 이름 뒤의 꺾쇠 괄호 안에 일반 수명 매개변수의 이름을 선언하여 구조체 정의 본문에서 수명 매개변수를 사용할 수 있습니다. 이 주석은 의 `ImportantExcerpt`인스턴스가 해당 필드에 보유하고 있는 참조보다 오래 지속될 수 없음을 의미합니다 `part`.

여기서 함수 는 변수가 소유한 첫 번째 문장에 대한 참조를 보유하는 구조체 `main`의 인스턴스를 만듭니다. 인스턴스가 생성되기 전에 데이터가 존재합니다. 또한 범위를 벗어날 때까지 범위를 벗어나지 않으므로 인스턴스의 참조가 유효합니다.`ImportantExcerpt``String``novel``novel``ImportantExcerpt``novel``ImportantExcerpt``ImportantExcerpt`

### [평생 제거](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision)

모든 참조에는 수명이 있으며 참조를 사용하는 함수 또는 구조체에 대한 수명 매개 변수를 지정해야 한다는 것을 배웠습니다. 그러나 4장에서 우리는 Listing 4-9에 수명 주석 없이 컴파일된 Listing 10-25에 다시 표시된 함수가 있었습니다.

파일 이름: src/lib.rs

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

목록 10-25: 매개변수와 반환 유형이 참조인 경우에도 수명 주석 없이 컴파일된 목록 4-9에서 정의한 함수

이 함수가 수명 주석 없이 컴파일되는 이유는 역사적입니다. Rust의 초기 버전(1.0 이전)에서는 모든 참조에 명시적 수명이 필요했기 때문에 이 코드가 컴파일되지 않았을 것입니다. 당시 함수 서명은 다음과 같이 작성되었을 것입니다.

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

많은 Rust 코드를 작성한 후 Rust 팀은 Rust 프로그래머가 특정 상황에서 동일한 수명 주석을 반복해서 입력하고 있음을 발견했습니다. 이러한 상황은 예측 가능했으며 몇 가지 결정론적 패턴을 따랐습니다. 개발자는 차용 검사기가 이러한 상황에서 수명을 추론할 수 있고 명시적인 주석이 필요하지 않도록 컴파일러 코드에 이러한 패턴을 프로그래밍했습니다.

Rust 역사의 이 부분은 더 결정적인 패턴이 나타나고 컴파일러에 추가될 가능성이 있기 때문에 관련이 있습니다. 앞으로는 더 적은 수명 주석이 필요할 수 있습니다.

Rust의 참조 분석에 프로그래밍된 패턴을 *수명 제거 규칙* 이라고 합니다. 이것은 프로그래머가 따라야 할 규칙이 아닙니다. 이들은 컴파일러가 고려할 특정 사례의 집합이며, 코드가 이러한 경우에 적합하면 수명을 명시적으로 작성할 필요가 없습니다.

제거 규칙은 완전한 추론을 제공하지 않습니다. Rust가 결정적으로 규칙을 적용하지만 참조의 수명에 대해 여전히 모호한 경우 컴파일러는 나머지 참조의 수명을 추측하지 않습니다. 추측하는 대신 컴파일러는 수명 주석을 추가하여 해결할 수 있는 오류를 제공합니다.

*함수 또는 메소드 매개변수에 대한 수명을 입력 수명* 이라고 하고 반환 값에 대한 수명을 *출력 수명* 이라고 합니다.

컴파일러는 명시적인 주석이 없을 때 세 가지 규칙을 사용하여 참조의 수명을 파악합니다. 첫 번째 규칙은 입력 수명에 적용되고 두 번째 및 세 번째 규칙은 출력 수명에 적용됩니다. 컴파일러가 세 가지 규칙의 끝에 도달하고 수명을 파악할 수 없는 참조가 여전히 있는 경우 컴파일러는 오류와 함께 중지됩니다. 이러한 규칙은 블록 `fn`뿐만 아니라 정의 에도 적용됩니다 `impl`.

첫 번째 규칙은 컴파일러가 참조인 각 매개변수에 수명 매개변수를 할당한다는 것입니다. 즉, 하나의 매개변수가 있는 함수는 하나의 라이프타임 매개변수를 얻습니다: `fn foo<'a>(x: &'a i32)`; 두 개의 매개변수가 있는 함수는 두 개의 별도 수명 매개변수를 얻습니다: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; 등등.

두 번째 규칙은 정확히 하나의 입력 수명 매개변수가 있는 경우 해당 수명이 모든 출력 수명 매개변수에 할당된다는 것입니다 `fn foo<'a>(x: &'a i32) -> &'a i32`.

세 번째 규칙은 입력 라이프타임 매개변수가 여러 개인 경우 그 중 하나가 메서드이기 `&self`때문에 모든 출력 라이프타임 매개변수에 `&mut self`수명이 `self`할당된다는 것입니다. 이 세 번째 규칙은 더 적은 수의 기호가 필요하기 때문에 메소드를 읽고 쓰기에 훨씬 더 좋습니다.

우리가 컴파일러라고 가정해 봅시다. `first_word`목록 10-25에 있는 함수 서명에서 참조의 수명을 파악하기 위해 이러한 규칙을 적용할 것입니다. 서명은 참조와 연결된 수명 없이 시작됩니다.

```rust
fn first_word(s: &str) -> &str {
```

그런 다음 컴파일러는 각 매개변수가 자체 수명을 갖도록 지정하는 첫 번째 규칙을 적용합니다. 평소와 같이 호출하므로 `'a`이제 서명은 다음과 같습니다.

```rust
fn first_word<'a>(s: &'a str) -> &str {
```

정확히 하나의 입력 수명이 있기 때문에 두 번째 규칙이 적용됩니다. 두 번째 규칙은 하나의 입력 매개변수의 수명이 출력 수명에 할당되도록 지정하므로 서명은 이제 다음과 같습니다.

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

이제 이 함수 서명의 모든 참조에는 수명이 있으며 컴파일러는 프로그래머가 이 함수 서명의 수명에 주석을 달 필요 없이 분석을 계속할 수 있습니다.

또 다른 예를 살펴보겠습니다. 이번에는 `longest`목록 10-20에서 작업을 시작했을 때 수명 매개변수가 없었던 함수를 사용했습니다.

```rust
fn longest(x: &str, y: &str) -> &str {
```

첫 번째 규칙을 적용해 보겠습니다. 각 매개변수는 고유한 수명을 갖습니다. 이번에는 하나가 아닌 두 개의 매개변수가 있으므로 두 개의 수명이 있습니다.

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

두 개 이상의 입력 수명이 있기 때문에 두 번째 규칙이 적용되지 않는 것을 볼 수 있습니다. 세 번째 규칙도 적용되지 않습니다. 는 `longest`메서드가 아니라 함수이므로 매개변수 중 어느 것도 `self`. 세 가지 규칙을 모두 살펴본 후에도 여전히 반환 유형의 수명이 무엇인지 파악하지 못했습니다. 이것이 Listing 10-20의 코드를 컴파일하려고 시도하는 동안 오류가 발생한 이유입니다. 컴파일러는 수명 생략 규칙을 통해 작업했지만 여전히 서명에 있는 참조의 모든 수명을 파악할 수 없었습니다.

세 번째 규칙은 실제로 메서드 서명에만 적용되기 때문에 세 번째 규칙이 메서드 서명에서 수명에 자주 주석을 달 필요가 없음을 의미하는 이유를 알아보기 위해 해당 컨텍스트에서 수명을 살펴보겠습니다.

### [메서드 정의의 수명 주석](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-annotations-in-method-definitions)

수명이 있는 구조체에 메서드를 구현할 때 목록 10-11에 표시된 제네릭 형식 매개 변수와 동일한 구문을 사용합니다. 수명 매개변수를 선언하고 사용하는 위치는 구조 필드 또는 메서드 매개변수 및 반환 값과 관련되는지 여부에 따라 다릅니다.

구조체 필드의 수명 이름은 항상 키워드 다음에 선언한 다음 구조체 이름 다음에 사용해야 합니다 `impl`. 이러한 수명은 구조체 유형의 일부이기 때문입니다.

블록 내부의 메서드 서명에서 `impl`참조는 구조체 필드의 참조 수명에 연결되거나 독립적일 수 있습니다. 또한 수명 생략 규칙은 종종 메서드 서명에 수명 주석이 필요하지 않도록 합니다. `ImportantExcerpt`목록 10-24에서 정의한 구조체 이름을 사용하는 몇 가지 예를 살펴보겠습니다.

```
level`먼저, 유일한 매개변수가 에 대한 참조 이고 반환 값이 어떤 것에도 참조가 아닌 `self`an 이라는 이름의 메서드를 사용합니다.`i32
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

이후의 수명 매개변수 선언 과 유형 이름 이후의 사용이 필요하지만 첫 번째 생략 규칙 때문에 `impl`참조의 수명에 주석을 달 필요가 없습니다.`self`

다음은 세 번째 평생 생략 규칙이 적용되는 예입니다.

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!(`Attention please: {}`, announcement);
        self.part
    }
}
```

두 개의 입력 수명이 있으므로 Rust는 첫 번째 수명 생략 규칙을 적용하고 둘 다 `&self`자신 `announcement`의 수명을 제공합니다. 그런 다음 매개변수 중 하나가 이므로 `&self`반환 유형은 의 수명을 가져오고 `&self`모든 수명이 고려되었습니다.

### [정적 수명](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#the-static-lifetime)

우리가 논의해야 할 하나의 특별한 수명은 `'static`영향을 받는 참조가 프로그램의 전체 기간 동안 존재할 *수 있음을 나타냅니다.* 모든 문자열 리터럴에는 `'static`다음과 같이 주석을 달 수 있는 수명이 있습니다.

```rust
let s: &'static str = `I have a static lifetime.`;
```

이 문자열의 텍스트는 항상 사용 가능한 프로그램의 바이너리에 직접 저장됩니다. 따라서 모든 문자열 리터럴의 수명은 `'static`.

`'static`오류 메시지에서 수명을 사용하라는 제안을 볼 수 있습니다. 그러나 `'static`참조의 수명으로 지정하기 전에 참조가 실제로 프로그램의 전체 수명 동안 지속되는지 여부와 원하는지 여부를 생각하십시오. 대부분의 경우 수명 을 제안하는 오류 메시지 `'static`는 댕글링 참조를 만들려고 시도하거나 사용 가능한 수명이 일치하지 않아 발생합니다. 이러한 경우 솔루션은 `'static`수명을 지정하는 것이 아니라 해당 문제를 수정하는 것입니다.

## [일반 유형 매개변수, 특성 경계 및 수명 함께](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#generic-type-parameters-trait-bounds-and-lifetimes-together)

제네릭 형식 매개 변수, 특성 범위 및 수명을 모두 하나의 함수로 지정하는 구문을 간단히 살펴보겠습니다!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!(`Announcement! {}`, ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

이것은 `longest`두 문자열 슬라이스 중 더 긴 것을 반환하는 Listing 10-21의 함수입니다. `ann`그러나 이제 제네릭 형식이라는 이름의 추가 매개변수가 있으며 , 이 매개변수는 절 에서 지정한 대로 특성을 `T`구현하는 모든 형식으로 채울 수 있습니다. 이 추가 매개변수는 를 사용하여 인쇄될 것이므로 특성 바인딩이 필요합니다. 수명은 제네릭 유형이므로 수명 매개변수 선언 과 제네릭 유형 매개변수는 함수 이름 뒤의 꺾쇠 괄호 안에 있는 동일한 목록에 들어갑니다.`Display``where``{}``Display``'a``T`

## [요약](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#summary)

이 장에서 많은 내용을 다루었습니다! 이제 일반 유형 매개변수, 특성 및 특성 경계, 일반 수명 매개변수에 대해 알았으므로 다양한 상황에서 작동하는 반복 없이 코드를 작성할 준비가 되었습니다. 일반 유형 매개변수를 사용하면 다른 유형에 코드를 적용할 수 있습니다. 특성 및 특성 범위는 유형이 일반적이더라도 코드에 필요한 동작을 갖도록 합니다. 이 유연한 코드에 매달린 참조가 없도록 수명 주석을 사용하는 방법을 배웠습니다. 그리고 이 모든 분석은 컴파일 시간에 발생하므로 런타임 성능에 영향을 미치지 않습니다!

믿거나 말거나, 이 장에서 논의한 주제에 대해 더 많은 것을 배울 수 있습니다. 17장에서는 특성을 사용하는 또 다른 방법인 특성 개체에 대해 설명합니다. 매우 고급 시나리오에서만 필요한 수명 주석과 관련된 더 복잡한 시나리오도 있습니다. [그런 경우 Rust Reference 를](https://doc.rust-lang.org/reference/index.html) 읽어야 합니다. 그러나 다음에는 Rust로 테스트를 작성하여 코드가 제대로 작동하는지 확인할 수 있습니다.