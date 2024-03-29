+++
title = "Packages and Crates"
weight = 1
template = "book_page_rust.html"

+++

## 요약

- 크레이트(crates)
  - Rust 컴파일러가 한 번에 고려하는 가장 작은 양의 코드
  - 크레이트는 모듈을 포함할 수 있음
  - 바이너리 크레이트
    - `main` 함수 있음, 실행 파일로 컴파일된다
    - 프로젝트 디렉토리/src/main.rs 파일이 있다
  - 라이브러리 크레이트
    - `main` 함수가 없음, 실행 파일로 컴파일안됨
    -  여러 프로젝트와 공유할 기능을 정의
    - 일반적으로 말하는 크레이트, 라이브러리와 같은 의미
    - 프로젝트 디렉토리/src/lib.rs 파일이 있다
- 패키지(packages)
  - 일련의 기능을 제공하는 하나 이상의 크레이트(crates) 묶음


## [패키지 및 크레이트](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html#packages-and-crates)

우리가 다룰 모듈 시스템의 첫 번째 부분은 패키지(packages)와 크레이트(crates)입니다.

*크레이트*는 Rust 컴파일러가 한 번에 고려하는 가장 작은 양의 코드입니다. `cargo` 대신 `rustc`를 실행하고 단일 소스 코드 파일을 전달하더라도(1장의 `Writing and Running a Rust Program` 섹션에서 한 것처럼), 컴파일러는 해당 파일을 크레이트로  간주합니다. 크레이트는 모듈을 포함할 수 있으며, 모듈은 크레이트와 함께 컴파일되는 다른 파일에서 정의될 수 있습니다. 다음 섹션에서 살펴보겠습니다.

크레이트는 *바이너리 크레이트* 또는 *라이브러리 크레이트*의 두 가지 형태 중 하나로 올 수 있습니다. **바이너리 크레이트**는 명령줄 프로그램(command-line program)이나 서버(server)와 같이 실행할 수 있는 실행 파일로 컴파일할 수 있는 프로그램입니다. 각각에는 실행 파일이 실행될 때 발생하는 일을 정의하는 `main`이라는 함수가 있어야 합니다. 지금까지 우리가 만든 모든 크레이트는 바이너리 크레이트였습니다.

**라이브러리 크레이트**에는 `main` 함수가 없으며 실행 파일로 컴파일되지 않습니다. 대신 여러 프로젝트와 공유할 기능을 정의합니다. [예를 들어 2장](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#generating-a-random-number) 에서 사용한 `rand` 크레이트는 난수를 생성하는 기능을 제공합니다. Rustaceans가 `크레이트(crate)`라고 말하는 대부분의 경우 라이브러리 크레이트를 의미하며 `라이브러리`의 일반적인 프로그래밍 개념과 상호 교환적으로 `크레이트`를 사용합니다.

크레이트 루트(*crate root*)는 Rust 컴파일러가 시작하고 크레이트의 루트 모듈을 구성하는 소스 파일입니다. ["범위 및 개인정보 보호를 제어하기 위한 모듈 정의"](https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html)  섹션에서 모듈에 대해 자세히 설명합니다.

*패키지*는 일련의 기능을 제공하는 하나 이상의 크레이트(crates) 묶음입니다. 패키지에는 이러한 상자를 만드는 방법을 설명하는 *Cargo.toml* 파일이 포함되어 있습니다. Cargo는 실제로 코드를 빌드하는 데 사용했던 명령줄 도구용 바이너리 크레이트를 포함하는 패키지입니다. Cargo 패키지에는 바이너리 크레이트가 의존하는 라이브러리 크레이트도 포함되어 있습니다. 다른 프로젝트는 Cargo 명령줄 도구가 사용하는 것과 동일한 논리를 사용하기 위해 Cargo 라이브러리 크레이트에 의존할 수 있습니다.

패키지는 원하는 만큼 많은 바이너리 크레이트를 포함할 수 있지만, 라이브러리 크레이트는 하나만 포함할 수 있습니다. 패키지는 라이브러리든 바이너리 크레이트든 적어도 하나의 크레이트를 포함해야 합니다.

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

`cargo new`를 실행한 후 `ls`를 사용하여 Cargo가 생성하는 것을 확인합니다. 프로젝트 디렉토리에는 패키지를 제공하는 *Cargo.toml* 파일이 있습니다. *main.rs*를 포함하는 *src* 디렉토리도 있습니다. 텍스트 편집기에서 *Cargo.toml*을 열고 *src/main.rs* 에 대한 언급이 없음에 유의하십시오. *Cargo*는 src/main.rs가 패키지와 같은 이름을 가진 바이너리 크레이트의 크레이트 루트라는 규칙을 따릅니다. 마찬가지로 Cargo는 패키지 디렉토리에 *src/lib.rs*가 포함되어 있으면 패키지에 패키지와 동일한 이름의 라이브러리 크레이트가 포함되어 있고 *src/lib.rs*가 포함되어 있음을 알고 있습니다.상자 루트입니다. Cargo는 크레이트 루트 파일을 `rustc`에 전달하여 라이브러리 또는 바이너리를 빌드합니다.

여기에는 *src/main.rs* 만 포함하는 패키지가 있습니다. 즉, `my-project`라는 이름의 바이너리 크레이트만 포함되어 있습니다. 패키지에 *src/main.rs* 및 *src/lib.rs* 가 모두 포함되어 있으면, 패키지와 동일한 이름을 가진 바이너리 크레이트와 라이브러리 크레이트가 있는 것입니다. 패키지는 *src/bin* 디렉토리에 파일을 배치하여 여러 바이너리 크레이트를 가질 수 있습니다. 각 파일은 별도의 바이너리 크레이트가 됩니다.
