+++
title = "Recoverable Errors with Result"
weight = 2
template ="book_page_rust.html"

+++



## 요약

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

```
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

```
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
