+++
title = "What is Ownership?"
weight = 1
template = "book_page_rust.html"

+++

## 요약

- Rust 메모리 관리는 소유권 시스템이다. (가비지 컬렉터가 없다)
- 하나의 값에는 하나의 소유자만 있다.
- 범위를 벗어나면 값과 소유자는 메모리에서 지워진다. 
- 메모리에 데이터를 저장하는 2가지 방식
  - 스택 : 정해진 위치에 정해진 크기의 데이터
    - let s2 = s1;   // copy 된다. 변수 s1, s2 모두 사용가능
  
  - 힙 : 크기를 모르는 데이터
    - 별도의 공간에 값을 저장하고, 주소(포인터)만 메모해둔다.
  
    - 값을 얻으려면 주소를 얻고, 주소의 위치로 찾아간다.
  
    - let s2 = s1;   // move 된다. 변수 s1은 지워진다.
- string 의 2가지 type
  - string literals : 스택, 불변
  - String : 힙, 변경가능
- 값을 복제(메모리의 2군데에 값을 저장하려면)
  - 스택 : 파이썬 처럼 복사하면됨
  - 힙 : clone() 으로 복사
- 함수에 값을 전달하기
  - 변수에 값을 할당할 때와 비슷 (사용하면 소유권이 옮겨진다)
  - 소유권 옮기지 않고 여러번 사용하려면 참조(reference) 기능 사용 



<!-- more -->

## [소유권이란 무엇입니까?](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#what-is-ownership)

*소유권*은 Rust 프로그램이 메모리를 관리하는 방식을 지배하는 일련의 규칙입니다. 모든 프로그램은 실행하는 동안 컴퓨터의 메모리를 사용하는 방식을 관리해야 합니다.

- 일부 언어에는 프로그램이 실행될 때 더 이상 사용되지 않는 메모리를 정기적으로 찾는 *가비지 수집* 기능이 있습니다.
- 다른 언어에서는 프로그래머가 *명시적으로 메모리를 할당하고 해제*해야 합니다.
- Rust 에서 메모리는 소유권 시스템을 통해 관리됩니다. 컴파일러는 일련의 규칙을 확인하며, 규칙 중 하나라도 위반되면 프로그램이 컴파일되지 않습니다. 소유권 시스템은 프로그램이 실행되는 동안 느려지게 하지 않습니다. (가비지 수집기능은 수집이 진행되는 동안 프로그램이 느려지기도 합니다)

소유권은 많은 프로그래머에게 새로운 개념이기 때문에 익숙해지는 데 시간이 걸립니다. 좋은 소식은 Rust와 소유권 시스템의 규칙에 대한 경험이 많을수록 안전하고 효율적인 코드를 자연스럽게 개발하는 것이 더 쉬워진다는 것입니다. 견뎌내세요!

소유권을 이해하면 Rust 만의 기능을 이해하기 기초를 갖게 됩니다. 이 장에서는 매우 일반적인 데이터 구조인 문자열(String) 예제를 통해 소유권을 배웁니다.

> ### [스택(Stack)과 힙(Heap)](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap)
>
> 많은 프로그래밍 언어에서는 스택과 힙에 대해 자주 생각할 필요가 없습니다. 그러나 Rust와 같은 시스템 프로그래밍 언어에서는 값이 스택에 있는지 힙에 있는지에 따라 언어의 작동 방식과 특정 결정을 내려야 하는 이유가 달라집니다. 소유권의 일부는 이 장의 뒷부분에서 스택과 힙과 관련하여 설명할 것이므로 준비를 위해 간단히 설명합니다.
>
> 스택과 힙은 모두 코드에서 런타임에 사용할 수 있는 메모리의 일부이지만 서로 다른 방식으로 구성됩니다. 스택은 값을 가져온 순서대로 저장하고 반대 순서로 값을 제거합니다. *이것을 last in, first out* 이라고 합니다. 접시 더미를 생각해 보십시오. 접시를 더 추가하면 더미 위에 놓고 접시가 필요하면 위에서 하나를 치웁니다. 중간이나 바닥에서 접시를 추가하거나 제거하는 것은 허락되지 않습니다! 데이터를 추가하는 것을 *스택에 pushing 하는 것을* , 데이터를 제거하는 것을 *스택에서 popping 하는 것을* 말합니다. 스택에 저장된 모든 데이터는 알려지고 고정된 크기를 가져야 합니다. 컴파일 시 크기를 알 수 없거나 크기가 변경될 수 있는 데이터는 대신 힙에 저장해야 합니다.
>
> 힙은 덜 조직적입니다. 힙에 데이터를 넣을 때 일정량의 공간을 요청합니다. 메모리 할당자는 힙에서 충분히 큰 빈 지점을 찾아 사용 중인 것으로 표시하고 해당 위치의 주소인 *포인터를 반환합니다.* 이 프로세스를 *힙에 할당(*allocating on the heap*) 한다고 하며 때로는 그냥* *할당*(*allocating*) 으로 줄여서 이야기합니다. (스택에 값을 푸시하는 것은 할당으로 간주되지 않습니다). 힙에 대한 포인터는 알려지고, 고정된 크기이므로 포인터를 스택에 저장할 수 있습니다. 하지만 실제 데이터를 원할 때는 포인터를 따라 찾아가야 합니다. 식당에 앉아 있다고 생각해보세요. 들어갈 때 그룹의 인원수를 말하면 호스트가 모든 사람에게 맞는 빈 테이블을 찾아 그곳으로 안내합니다. 그룹의 누군가가 늦게 오면 그들은 당신을 찾기 위해 당신이 어디에 앉았는지 물어볼 수 있습니다.
>
> 할당자가 새 데이터를 저장할 위치를 검색할 필요가 없기 때문에 스택에 푸시하는 것이 힙에 할당하는 것보다 빠릅니다. 해당 위치는 항상 스택의 맨 위에 있습니다. 상대적으로 힙에 공간을 할당하려면 할당자가 먼저 데이터를 보유할 수 있을 만큼 충분히 큰 공간을 찾은 다음 다음 할당을 준비하기 위해 기록을 수행해야 하기 때문에 더 많은 작업이 필요합니다.
>
> 힙에 있는 데이터에 액세스하려면 스택에 있는 포인터(주소)를 알아내고, 그 위치로 찾아가야 합니다. 그래서 힙에 있는 데이터에 액세스하는 것이 스택에 있는 데이터에 액세스하는 것보다 느립니다. 최신 프로세서가 메모리 안에서 점프를 적게하면 더 빠릅니다. 비유를 계속해서 많은 테이블에서 주문을 받는 식당의 서버를 생각해 보십시오. 다음 테이블로 이동하기 전에 한 테이블에서 모든 주문을 받는 것이 가장 효율적입니다. 테이블 A에서 주문을 받은 다음 테이블 B에서 주문을 받고, 다시 A에서 주문을 받고, 다시 B에서 주문을 받는 것은 훨씬 더 느린 프로세스입니다. 마찬가지로, 프로세서는 다른 데이터와 멀리 떨어져 있는 데이터(힙에 있을 수 있으므로)보다, 가까운 데이터(스택에 있는 데이터)에 대해 작업하는 경우 작업을 더 잘 수행할 수 있습니다.
>
> 코드가 함수를 호출하면, 함수에 값이 전달 됩니다. 이 값은 힙의 데이터에 대한 포인터 일수도 있습니다. 그리고 함수의 로컬 변수가 스택으로 푸시됩니다. 함수가 끝나면 해당 값이 스택에서 제거됩니다.
>
> 코드의 어떤 부분이 힙에서 어떤 데이터를 사용하는지 추적하고, 힙에서 중복 데이터의 양을 최소화하고, 공간이 부족하지 않도록 힙에서 사용하지 않는 데이터를 정리하는 것은 모두 소유권이 해결하는 문제입니다. 소유권을 이해하면 스택과 힙에 대해 자주 생각할 필요가 없지만 소유권의 주요 목적이 힙 데이터를 관리하는 것임을 알면 소유권이 작동하는 방식을 설명하는 데 도움이 될 수 있습니다.

### [소유권 규칙](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ownership-rules)

먼저 소유권 규정을 살펴보겠습니다. 이러한 규칙을 설명하는 예제를 통해 작업할 때 이러한 규칙을 염두에 두십시오.

- Rust의 각 값에는 *소유자가* 있습니다.
- 한 번에 한 명의 소유자만 있을 수 있습니다.
- 소유자가 범위를 벗어나면 값이 삭제됩니다.

### [변수 범위](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variable-scope)

우리는 Rust 의 기본 구문을 이미 배웠기때문에, 이제부터 예제에  `fn main() {` 코드를 생략하겠습니다. 따라서 다음 예제를  `main` 함수 안에 넣어야 합니다.  결과적으로 우리의 예제는 좀 더 간결해지고 실제 세부 사항에 집중할 수 있습니다.

소유권의 첫 번째 예로서 일부 변수의 *범위(scope)*를 살펴보겠습니다. 범위(scope)는 아이템이 유효한, 프로그램 내의 범위입니다. 다음 변수를 사용하십시오.

```rust
let s = `hello`;
```

변수 `s`는 문자열 리터럴을 참조하며, 문자열 값은 프로그램의 텍스트에 하드코딩됩니다. 변수는 선언된 시점부터 현재 *범위*가 끝날 때까지 유효합니다. 목록 4-1은 변수 `s`가 유효한 위치에 주석이 달린 프로그램을 보여줍니다.

```rust
    {                      // s is not valid here, it’s not yet declared
        let s = `hello`;   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

목록 4-1: 변수와 변수가 유효한 범위

즉, 여기에는 두 가지 중요한 시점이 있습니다.

- `s`가 *범위에 포함* 되면 유효합니다.
- *범위를 벗어날* 때까지 유효합니다.

이 시점에서 범위와 변수가 유효한 시기는 다른 프로그래밍 언어와 유사합니다. 이제 `문자열(String)` 유형을 도입하여 이러한 이해를 바탕으로 빌드하겠습니다.

### [문자열 유형](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-string-type)

소유권 규칙을 설명하려면 3장의 [데이터 유형](https://doc.rust-lang.org/book/ch03-02-data-types.html#data-types) 섹션에서 다룬 것보다 더 복잡한 데이터 유형이 필요합니다. 이전에 다룬 유형은 알려진 크기이며, 스택에 저장(store)하고 꺼내기(pop)를 할 수 있었습니다. 코드의 다른 부분에 있는 다른 범위(scope)에서, 동일한 값을 사용해야 하는 경우, 새롭고 독립적인 인스턴스를 만들기 위해 신속하고 간단하게 복사할 수 있습니다. 
그러나 우리는 힙에 저장된 데이터를 살펴보고 Rust가 해당 데이터를 정리할 시기를 어떻게 아는지 살펴보고자 합니다. `문자열(String)` 유형이 좋은 예입니다.

소유권과 관련된 `문자열(String)` 부분에 집중하겠습니다. 이러한 측면은 다른 복합 데이터 유형에도 적용됩니다. (복합 데이터 유형은 표준 라이브러리에서 제공하기도 하고 사용자가 생성할 수 도 있습니다. ) [8장](https://doc.rust-lang.org/book/ch08-02-strings.html)에서 `문자열(String)`에 대해 더 깊이 논의할 것입니다.

우리는 문자열 값이 우리 프로그램에 하드코딩되는 `문자열 리터럴(string literals)`을 이미 보았습니다. 문자열 리터럴은 편리하지만 텍스트를 사용하려는 모든 상황에 적합하지는 않습니다. 한 가지 이유는 그것들이 불변이라는 것입니다. 다른 하나는 코드를 작성할 때 모든 문자열 값을 알 수 있는 것은 아니라는 것입니다. 예를 들어 사용자 입력을 받아 저장하려면 어떻게 해야 할까요? 
이러한 상황을 위해 Rust에는 두 번째 문자열 유형인 `String`이 있습니다. 이 유형은 힙에 할당된 데이터를 관리하므로 컴파일 시 알 수 없는 양의 텍스트를 저장할 수 있습니다. 다음과 같이 `from` 함수를 사용하여 문자열 리터럴에서 `문자열(String)`을 만들 수 있습니다.

```rust
let s = String::from(`hello`);
```

이중 콜론 `::` 연산자를 사용하면, `string_from`과 같은 일종의 이름을 사용하는 대신, `String` 유형 아래에서 이 특정 `from` 함수의 이름을 지정할 수 있습니다. 이 구문은 5장의 [메서드 구문](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#method-syntax) 섹션과 7장의 [모듈 트리에서 항목을 참조하기 위한 경로](https://doc.rust-lang.org/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html) 에서 모듈의 네임스페이스에 대해 설명할 때 더 자세히 설명합니다.

이러한 종류의 문자열은 변경될 *수 있습니다* .

```rust
    let mut s = String::from(`hello`);

    s.push_str(`, world!`); // push_str() appends a literal to a String

    println!(`{}`, s); // This will print `hello, world!`
```

차이점은 무엇입니까? `문자열(String)`은 변경할 수 있지만 리터럴은 변경할 수 없는 이유는 무엇입니까? 차이점은 이 두 가지 유형이 메모리를 처리하는 방식입니다.

### [메모리와 할당](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#memory-and-allocation)

문자열 리터럴의 경우 컴파일 시간에 내용을 알기 때문에 텍스트가 최종 실행 파일에 직접 하드코딩됩니다. 이것이 문자열 리터럴이 빠르고 효율적인 이유입니다. 그러나 이러한 속성은 문자열 리터럴이 불변(immutable)하기 때문에 가능합니다. 불행하게도 우리는 컴파일 타임에 크기를 알 수 없고 프로그램을 실행하는 동안 크기가 변경될 수 있는 각 텍스트 조각에 대한 메모리 덩어리를 바이너리에 넣을 수 없습니다.

`문자열(String)` 유형을 사용하여 변경 가능하고 크기 변경이 가능한 텍스트 조각을 지원하려면, 컴파일 시간에 알 수 없는 내용을 보관할 메모리 양을 힙에 할당해야 합니다. 이는 다음을 의미합니다.

- 런타임 시 메모리 할당자에서 메모리를 요청해야 합니다.
- `문자열(String)` 작업이 완료되면 이 메모리를 할당자에게 반환하는 방법이 필요합니다.

첫 번째 부분은 우리가 수행합니다. `String::from`을 호출하면 필요한 메모리를 요청합니다. 이것은 프로그래밍 언어에서 거의 보편적입니다.

그러나 두 번째 부분은 다릅니다. *가비지 컬렉터(garbage collector)* 가 있는 언어에서 가비지 컬렉터는 더 이상 사용되지 않는 메모리를 추적하고 정리하므로 우리는 그것에 대해 생각할 필요가 없습니다. 반면에 가비지 컬렉터가 없는 대부분의 언어에서는 메모리를 더 이상 사용하지 않는 시기를 식별하고, 메모리를 해제하는 코드를 호출하는 것은 우리의 책임입니다. 이를 올바르게 수행하는 것은 역사적으로 어려운 프로그래밍 문제였습니다. 잊어버리면 메모리를 낭비하게 됩니다. 너무 일찍 수행하면 유효하지 않은 변수가 생깁니다. 두 번 하면 그것도 버그입니다. 정확히 하나의 `할당(allocate)`과 정확히 하나의 `자유(free)`를 쌍으로 연결해야 합니다.

Rust는 다른 경로를 취합니다. 메모리를 소유한 변수가 범위를 벗어나면 메모리가 자동으로 반환(free)됩니다. 다음은 문자열 리터럴 대신 `문자열(String)`을 사용하는 Listing 4-1의 범위 예제 버전입니다.

```rust
    {
        let s = String::from(`hello`); // s is valid from this point forward

        // do stuff with s
    }                                  // this scope is now over, and s is no
                                       // longer valid
```

`String`이 필요로 하는 메모리를 할당자에게 반환할 수 있는 자연스러운 지점이 있습니다. `s`가 범위를 벗어날 때입니다. 변수가 범위를 벗어나면 Rust는 우리를 위해 특별한 함수를 호출합니다. 이 함수를 [`drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop) 이라고 하며 `String`의 작성자가 메모리를 반환하는 코드를 넣을 수 있는 곳입니다. Rust는 닫는 중괄호에서 자동으로 `drop`을 호출합니다.

> 참고: C++에서 항목의 수명이 끝날 때 리소스 할당을 취소하는 이 패턴을 *리소스 획득이 초기화(*Resource Acquisition Is Initialization, RAII)* 라고도 합니다. Rust의 `드롭` 기능은 RAII 패턴을 사용해 본 적이 있다면 익숙할 것입니다.

이 패턴은 Rust 코드 작성 방식에 지대한 영향을 미칩니다. 지금 당장은 간단해 보일 수 있지만, 힙에 할당한 데이터를 여러 변수가 사용하도록 하려는 복잡한 상황에서는 코드가 예상치 못한 방식으로 작동할 수 있습니다. 이제 그러한 상황 중 일부를 살펴보겠습니다.

#### [Move와 상호 작용하는 변수 및 데이터](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move)

여러 변수는 Rust에서 다른 방식으로 동일한 데이터와 상호 작용할 수 있습니다. 목록 4-2에서 정수를 사용하는 예를 살펴보겠습니다.

```rust
    let x = 5;
    let y = x;
```

목록 4-2: 변수 `x`의 정수 값을 `y`에 할당

이것이 무엇을 하는지 짐작할 수 있을 것입니다. 값 `5`를 `x`에 바인딩합니다. 그런 다음 `x`의 값을 복사하여 `y`에 바인딩합니다. 이제 두 개의 변수 `x`와 `y`가 있고 둘 다 `5`입니다. 정수는 알려지고 고정된 크기를 가진 단순한 값이고, 이 두 `5` 값이 스택에 푸시되기 때문에 이것이 실제로 일어나고 있는 일입니다.

이제 `문자열(String)` 버전을 살펴보겠습니다.

```rust
    let s1 = String::from(`hello`);
    let s2 = s1;
```

이것은 매우 유사해 보이기 때문에 작동 방식이 동일할 것이라고 가정할 수 있습니다. 즉, 두 번째 줄은 `s1`의 값을 복사하여 `s2`에 바인딩합니다. 그러나 이것은 실제로 일어나는 일이 아닙니다.

 `문자열(String)`에 무슨 일이 일어나고 있는지 보려면 그림 4-1을 살펴보십시오. `문자열(String)`은 왼쪽에 표시된 것처럼 세 부분으로 구성됩니다. 문자열의 내용을 보유하는 메모리에 대한 포인터, 길이 및 용량입니다. 이 데이터 그룹은 스택에 저장됩니다. 오른쪽에는 내용을 보유하는 힙의 메모리가 있습니다.

![두 개의 테이블: 첫 번째 테이블에는 길이(5), 용량(5) 및 두 번째 테이블의 첫 번째 값에 대한 포인터로 구성된 스택의 s1 표현이 포함됩니다.  두 번째 테이블에는 바이트 단위로 힙에 있는 문자열 데이터의 표현이 포함됩니다.](https://doc.rust-lang.org/book/img/trpl04-01.svg)

그림 4-1: `s1`에 바인딩된 ``hello`` 값을 보유하는 `문자열(String)`의 메모리 표현

길이(length)는 `문자열(String)`의 내용이 현재 사용 중인 메모리 양(바이트)입니다. 용량(capacity)은 `문자열(String)`이 할당자로부터 받은 총 메모리 양(바이트)입니다. 길이와 용량의 차이는 중요하지만 이 맥락에서는 중요하지 않으므로 지금은 용량을 무시해도 됩니다.

`s1`을 `s2`에 할당하면 `문자열(String)` 데이터가 복사됩니다. 즉, 스택에 있는 포인터, 길이 및 용량을 복사합니다. (포인터가 참조하는) 힙에 있는 데이터를 복사하지 않습니다. 즉, 메모리의 데이터 표현은 그림 4-2와 같습니다.

![3개의 테이블: 테이블 s1 및 s2는 각각 스택의 해당 문자열을 나타내며 둘 다 힙의 동일한 문자열 데이터를 가리킵니다.](https://doc.rust-lang.org/book/img/trpl04-02.svg)

그림 4-2: `s1`의 포인터, 길이 및 용량의 복사본이 있는 변수 `s2`의 메모리 표현

표현은 그림 4-3처럼 보이지 *않습니다* . Rust가 힙 데이터도 복사했다면 메모리는 그림 4-3처럼 생겼을 것입니다. Rust가 이렇게 하면 힙의 데이터가 큰 경우 `s2 = s1` 작업은 런타임 성능을 떨어뜨립니다. 

![테이블 4개: s1 및 s2에 대한 스택 데이터를 나타내는 테이블 2개와 각각 힙에 있는 자체 문자열 데이터 복사본을 가리킵니다.](https://doc.rust-lang.org/book/img/trpl04-03.svg)

그림 4-3: Rust가 힙 데이터도 복사한 경우 `s2 = s1`이 무엇을 할 수 있는지에 대한 또 다른 가능성

이전에 우리는 변수가 범위를 벗어나면 Rust가 자동으로 `drop` 함수를 호출하고 해당 변수에 대한 힙 메모리를 정리한다고 말했습니다. 그러나 그림 4-2는 동일한 위치를 가리키는 두 데이터 포인터를 보여줍니다. 이것은 문제입니다. `s2`와 `s1`이 범위를 벗어나면 둘 다 동일한 메모리를 해제하려고 합니다. 이것은 *이중 자유 오류(double free error)*로 알려져 있으며 이전에 언급한 메모리 안전 버그 중 하나입니다. 메모리를 두 번 해제하면 메모리 손상이 발생하여 잠재적으로 보안 취약성이 발생할 수 있습니다.

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

다른 언어로 작업하면서 *얕은 복사(shallow copy)* 와 *깊은 복사(deep copy)* 용어를 들어본 적이 있다면, 데이터를 복사하지 않고 포인터, 길이, 용량을 복사한다는 개념은 아마도 얕은 복사를 하는 것처럼 들릴 것입니다. 그러나 Rust는 첫 번째 변수도 무효화하기 때문에 얕은 복사본이라고 하는 대신 *이동(move*) 이라고 합니다 . 이 예에서는 `s1`이 `s2`로 *이동되었다*고 말합니다. 따라서 실제로 일어나는 일은 그림 4-4에 나와 있습니다.

![3개의 테이블: 테이블 s1 및 s2는 각각 스택의 해당 문자열을 나타내며 둘 다 힙의 동일한 문자열 데이터를 가리킵니다.  s1이 더 이상 유효하지 않기 때문에 테이블 s1이 회색으로 표시됩니다.  s2만 힙 데이터에 액세스하는 데 사용할 수 있습니다.](https://doc.rust-lang.org/book/img/trpl04-04.svg)

그림 4-4: `s1`이 무효화된 후 메모리의 표현

이중 자유 오류(double free error)가 해결되었습니다! `s2`만 유효하면 범위를 벗어나면 단독으로 메모리를 해제하고 완료됩니다.

추가로 이것이 암시하는 디자인 선택이 있습니다: Rust는 데이터의 `깊은` 복사본을 자동으로 생성하지 않습니다. 따라서 *자동* 복사는 런타임 성능 측면에서 저렴하다고 가정할 수 있습니다.

#### [클론과 상호 작용하는 변수 및 데이터](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone)

스택 데이터뿐만 아니라 `String`의 힙 데이터를 깊이 복사하려는 경우 `복제(clone)`이라는 일반적인 방법을 사용할 수 *있습니다* . 5장에서 메서드 구문에 대해 논의하겠지만 메서드는 많은 프로그래밍 언어에서 공통적인 기능이기 때문에 이전에 본 적이 있을 것입니다.

다음은 작동 중인 `복제` 방법의 예입니다.

```rust
    let s1 = String::from(`hello`);
    let s2 = s1.clone();

    println!(`s1 = {}, s2 = {}`, s1, s2);
```

*이것은 잘 작동하며 힙 데이터가 복사* 되는 그림 4-3에 표시된 동작을 명시적으로 생성합니다.

`복제(clone)`에 대한 호출을 보면 임의의 코드가 실행되고 있고, 해당 코드가 비쌀 수 있음을 알 수 있습니다. 뭔가 다른 일이 벌어지고 있다는 시각적 지표입니다.

#### [스택 전용 데이터: 복사](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#stack-only-data-copy)

우리가 아직 이야기하지 않은 또 다른 주름이 있습니다. 정수를 사용하는 이 코드(목록 4-2에 그 일부가 표시됨)는 작동하고 유효합니다.

```rust
    let x = 5;
    let y = x;

    println!(`x = {}, y = {}`, x, y);
```

그러나 이 코드는 우리가 방금 배운 것과 모순되는 것 같습니다. `복제(clone)` 함수를 사용하지 않았지만  `x`는 여전히 유효하고 `y`로 이동(move)되지 않았습니다.

그 이유는 정수는 컴파일 타임에 알려진 크기를 가지고, 스택에 완전히 저장되기 때문에 실제 값의 복사본을 빠르게 만들 수 있기 때문입니다. 즉, 변수 `y`를 만든 후에 `x`가 유효하지 않도록 할 이유가 없습니다. 즉, 여기에서는 깊은 복사와 얕은 복사 사이에 차이가 없으므로, `clone`은 일반적인 얕은 복사와 다른 작업을 수행하지 않으며 생략할 수 있습니다.

Rust에는 `복사(copy)` 특성이라는 특수 주석이 있습니다. 정수와 마찬가지로 스택에 저장된 유형에 배치할 수 있습니다(특성에 대한 자세한 내용은 [10장](https://doc.rust-lang.org/book/ch10-02-traits.html) 에서 설명 ). 유형이 `복사` 특성을 구현하는 경우 이를 사용하는 변수는 이동하지 않고 복사되어 다른 변수에 할당된 후에도 여전히 유효합니다.

러스트는 타입 또는 그 부분이 `Drop` 특성을 구현한 경우, `Copy`로 타입 주석을 달지 못하게 합니다. 값이 범위를 벗어나고 해당 타입에 `복사` 주석을 추가할 때 유형에 특별한 일이 발생해야 하는 경우 컴파일 타임 오류가 발생합니다. 특성을 구현하기 위해 유형에 `복사` 주석을 추가하는 방법에 대해 알아보려면 부록 C의 [`파생 가능한 특성`](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html)을 참조하세요.

그렇다면 `복사` 특성을 구현하는 타입은 무엇입니까? 지정된 유형에 대한 설명서를 확인하여 확인할 수 있지만, 일반적으로 간단한 스칼라 값 그룹은 `복사`를 구현할 수 있으며, 할당이 필요하거나 리소스의 일부 형식은 `복사`를 구현할 수 없습니다. 다음은 `복사`를 구현하는 몇 가지 유형입니다.

- `u32`와 같은 모든 정수 유형.
- 값이 `true` 및 `false`인 부울 유형 `bool`.
- `f64`와 같은 모든 부동 소수점 유형.
- 문자 유형 `char`.
- 튜플(`복사`도 구현하는 유형만 포함하는 경우). 예를 들어 `(i32, i32)`는 `복사`를 구현하지만 `(i32, String)`은 구현하지 않습니다.

### [소유권 및 함수](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#ownership-and-functions)

함수에 값을 전달하는 메커니즘은, 변수에 값을 할당할 때와 비슷합니다. 변수를 함수에 전달하면 할당과 마찬가지로 이동하거나 복사합니다. 목록 4-3에는 변수가 범위에 들어가고 나가는 위치를 보여주는 몇 가지 주석이 있는 예제가 있습니다.

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

그러나 이것은 일반적이어야 하는 개념에 대해 너무 많은 의식과 많은 작업입니다. 운 좋게도 Rust에는 소유권을 이전하지 않고 값을 사용하는 참조(*references*)라는 기능이 *있습니다* .

