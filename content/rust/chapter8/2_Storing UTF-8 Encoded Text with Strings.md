+++
title = "Storing UTF-8 Encoded Text with Strings"
weight = 2
template ="book_page_rust.html"

+++



## 요약



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

```
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

그것은 18바이트이며 컴퓨터가 궁극적으로 이 데이터를 저장하는 방법입니다. Rust의 `char` 유형인 유니코드 스칼라 값으로 보면 해당 바이트는 다음과 같습니다.

```
['न', 'म', 'स', '्', 'त', 'े']
```

여기에는 6개의 `char` 값이 있지만 네 번째와 여섯 번째는 문자가 아닙니다. 자체적으로 의미가 없는 분음 부호입니다. 마지막으로, 그것들을 문자소 클러스터로 보면 힌디어 단어를 구성하는 4개의 문자라고 부르는 것을 얻을 수 있습니다.

```
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

```
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

```
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
