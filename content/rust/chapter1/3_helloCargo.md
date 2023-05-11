+++
title = "Hello, Cargo!"
weight = 3
template ="book_page_rust.html"
+++


## 요약

- cargo : Rust의 build system, package manager  

- `$ cargo new hello_cargo` 명령어를 실행하면 hello_cargo 폴더만들고, 그안에 git 도 설치됩니다. 실행하면 다음과 같은 파일 구조가 만들어집니다.  
  
``` 
├── .git
├── .gitignore
├── Cargo.toml : config 파일
└── src
    └── main.rs : 우리가 만드는 코드
```

- 처음에 cargo를  안썼어도, 위와같은 구조만 만들면 cargo 사용할 수 있습니다.

- `$ cargo build` 를 사용하면 다음과 같은 구조가 만들어집니다.

``` 
..
├── Cargo.lock : Cargo가 dependencies 의 version 관리하는 파일
├── Cargo.toml
├── src
│   └── main.rs
└── target
    └── debug
        └── hello_cargo  (여기 실행파일이 있다)
```
<br>

- `$ cargo run` :  코드를 build 하는 동시에 실행시킨다. (build 보다 더 많이 사용)  

- `$ cargo check` : compile되는지 확인만하고, 실행은 안한다.  

- `$ cargo build --release`
  - release 용으로 compile (그냥 build 와 다르게 compile이 오래걸리는 대신, 실행이 빠르다)  
  - ./target/release 폴더에 저장  



## [Hello, Cargo!](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html#hello-cargo)

Cargo는 Rust의 빌드 시스템이자 패키지 관리자입니다. 대부분의 Rustaceans는 이 도구를 사용하여 Rust 프로젝트를 관리합니다. 왜냐하면 Cargo가 코드 빌드, 코드가 의존하는 라이브러리 다운로드, 해당 라이브러리 빌드와 같은 많은 작업을 처리하기 때문입니다. (코드에 *종속성*(*dependency*)이 필요한 라이브러리를 호출합니다.)

우리가 지금까지 작성한 것과 같은 가장 단순한 Rust 프로그램은 종속성이 없습니다. Cargo로 "Hello, world!" 프로젝트를 만들면 코드 빌드를 처리하는 Cargo의 일부만 사용합니다. 더 복잡한 Rust 프로그램을 작성하면서 종속성을 추가하게 되고 Cargo를 사용하여 프로젝트를 시작하면 종속성을 추가하는 것이 훨씬 쉬워질 것입니다.

대부분의 Rust 프로젝트가 Cargo를 사용하기 때문에 이 책의 나머지 부분에서도 Cargo를 사용하고 있다고 가정합니다. Cargo는 [설치](https://doc.rust-lang.org/book/ch01-01-installation.html#installation) 섹션에서 논의된 공식 설치 프로그램을 사용한 경우 Rust와 함께 설치됩니다. 다른 방법으로 Rust를 설치했다면, 터미널에 다음을 입력하여 Cargo가 설치되어 있는지 확인하세요:

<br>

```
$ cargo --version
```

버전 번호가 보이면 가지고 있는 것입니다! `command not found`와 같은 오류가 표시되면 문서를 참조해서 Cargo를 별도로 설치하는 방법을 선택하세요. 



### [Cargo로 프로젝트 만들기](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html#creating-a-project-with-cargo)

Cargo를 사용하여 새 프로젝트를 생성하고 원래의 "Hello, world!"프로젝트와 어떻게 다른지 살펴보겠습니다. *프로젝트* 디렉토리(또는 코드를 저장하기로 결정한 위치) 로 다시 이동합니다. 그런 다음 운영 체제에서 다음을 실행합니다.

```
$ cargo new hello_cargo
$ cd hello_cargo
```

*첫 번째 명령은 hello_cargo* 라는 새 디렉터리와 프로젝트를 만듭니다. 우리는 프로젝트 이름을 *hello_cargo* 로 지정 했고, Cargo는 같은 이름의 디렉토리에 파일을 생성합니다.

<br>

*hello_cargo* 디렉토리 로 이동하여 파일을 확인해보십시오. Cargo가 우리를 위해 두 개의 파일과 하나의 디렉토리를 생성한 것을 볼 수 있습니다: 

```
├── .git
│   ├── ...
├── .gitignore
├── Cargo.toml
└── src
    └── main.rs
```

또한 *.gitignore* 파일과 함께 새로운 Git 리포지토리를 초기화했습니다. 기존 Git 리포지토리 내에서 `cargo new`실행하는 경우 Git 파일이 생성되지 않습니다. `cargo new --vcs=git` 를 사용하여 이 동작을 재정의할 수 있습니다 

> 참고: Git은 일반적인 버전 제어 시스템입니다.  `--vcs`플래그 를 사용하여 `cargo new` 를 바꿀 수 있습니다. 그러면 다른 버전 제어 시스템을 사용하거나, 버전 제어 시스템을 사용하지 않도록 변경할 수 있습니다. 사용 가능한 옵션을 보려면 `cargo new --help`실행하십시오.  

<br>

텍스트 편집기에서 *Cargo.toml* 을 엽니다. 목록 1-2의 코드와 유사해야 합니다.

파일 이름: Cargo.toml

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

Listing 1-2: `cargo new`에 의해 생성된 *Cargo.toml 의 내용*

이 파일은 Cargo의 구성 형식인 [*TOML*](https://toml.io/) ( *Tom's Obvious, Minimal Language* ) 형식입니다.

첫 번째 줄에 있는  `[package]` 는 다음 문이 패키지를 구성하고 있음을 나타내는 섹션 머리글입니. 이 파일에 더 많은 정보를 추가하게되면, 다른 섹션도 추가할 것입니다.  
다음 세 줄은 Cargo가 프로그램을 컴파일하는 데 필요한 구성 정보(사용할 Rust의 name, version 및 edition)를 설정합니다. [부록 E](https://doc.rust-lang.org/book/appendix-05-editions.html)에서 `edition` 키 에 대해 이야기하겠습니다.  
마지막 줄의 `[dependencies]` 는 프로젝트의 종속성을 나열하는 섹션의 시작 부분입니다. Rust에서는 코드 패키지를 *크레이트(crates)* 라고 합니다. 이 프로젝트에는 다른 크레이트가 필요하지 않지만, 2장의 첫 번째 프로젝트에는 필요할 것이므로, 이 [dependencies] 섹션을 사용할 것입니다.

<br>

이제 src/main.rs를 열고 살펴보십시오.  
파일 이름: src/main.rs  

```rust
fn main() {
    println!("Hello, world!");
}
```

목록 1-1에서 작성한 것과 같은 프로그램입니다. 이전에 만들었던 프로그램과 Cargo가 생성한 프로젝트의 차이점은 Cargo가 src 디렉토리에 코드를 배치하고 최상위 디렉토리에 *Cargo.toml* 이라는 이름의 config 파일이 있다는 것입니다.

``` 
├── Cargo.toml
└── src
    └── main.rs
```

*Cargo는 소스 파일이 src* 디렉토리 안에 있을 것으로 예상합니다. 최상위 프로젝트 디렉토리는 README 파일, 라이센스 정보, 구성 파일 및 코드와 관련되지 않은 모든 항목을 위한 것입니다. Cargo를 사용하면 프로젝트를 정리하는 데 도움이 됩니다. 모든 것은 그것을 위한 장소가 있고, 모든 것이 제자리에 있습니다.  

<br>

이전 장의 "Hello, world!"에서 했던 것처럼 Cargo를 사용하지 않고 프로젝트를 시작한 경우, 프로젝트에서 Cargo를 사용하는 프로젝트로 변환할 수 있습니다. 프로젝트 코드를 *src* 디렉토리로 이동하고 적절한 *Cargo.toml* 파일을 생성합니다.



### [Cargo 프로젝트 구축 및 실행](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html#building-and-running-a-cargo-project)

이제 Cargo 로 "Hello, world!"를 빌드하고 실행할 때 무엇이 다른지 살펴보겠습니다. *hello_cargo* 디렉터리에서 다음 명령을 입력하여 프로젝트를 빌드합니다.

```
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

이 명령은 현재 디렉터리가 아닌 *target/debug/hello_cargo* (Windows의 경우 *target\debug\hello_cargo.exe* ) 에 실행 파일을 생성합니다. 기본 빌드가 *디버그* 빌드이기 때문에 Cargo는 debug 라는 디렉토리에 바이너리를 넣습니다. 

``` 
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    └── debug
        ├── build
        ├── deps
        ├── examples
        ├── hello_cargo  (여기 실행파일이 있다)
        ├── hello_cargo.d
        └── incremental
```

<br>

다음 명령으로 실행 파일을 실행할 수 있습니다.  

```
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

모든 것이 잘 되었으면 터미널에 `Hello, world!` 가 인쇄되어야 합니다. 처음으로 `cargo build`실행하면 Cargo가 최상위 레벨에 새 파일인 *Cargo.lock* 을 생성하게 됩니다. 이 파일은 프로젝트의 정확한 종속성 버전을 추적합니다. 이 프로젝트에는 종속성이 없으므로 파일이 약간 희박합니다. 이 파일을 수동으로 변경할 필요가 없습니다. Cargo는 당신을 위해 내용물을 관리합니다.  

<br>

방금 `cargo build`로 프로젝트를 빌드하고, `./target/debug/hello_cargo`를 사용하여 실행했습니다.  그런데 `cargo run` 이라는 하나의 명령어를 사용해서 코드를 컴파일한 다음 실행할 수 도 있습니다.   

```
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

`cargo build` 를 실행한 다음 바이너리에 대한 전체 경로를 사용하는 것보다,  `cargo run` 을 사용하는 것이 더 편리하므로 대부분의 개발자는 `cargo run` 을 사용합니다.

이번에는 Cargo가 `hello_cargo` 를 컴파일 중임을 나타내는 출력이 표시되지 않았습니다. Cargo는 파일이 변경되지 않았음을 알아내서 다시 빌드하지 않고 바이너리만 실행했습니다. 소스 코드를 수정했다면 Cargo는 프로젝트를 실행하기 전에 다시 빌드했을 것이고, 다음과 같은 출력을 보았을 것입니다:

```
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

<br>

Cargo는 `cargo check`이라는 명령도 제공합니다. 이 명령은 코드를 신속하게 검사하여 컴파일되는지 확인합니다. 하지만 실행 파일을 생성하지는 않습니다.

```
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

실행 파일을 원하지 않는 이유는 무엇일까요?  `cargo check` 는 실행 파일 생성 단계를 건너뛰기 때문에 `cargo build` 보다 훨씬 빠릅니다. 코드를 작성하는 동안 지속적으로 작업을 확인하는 경우, `cargo check`를 사용하면 프로젝트가 아직 컴파일 되는지 확인하는 프로세스가 빨라집니다! 따라서 많은 Rustacean은 프로그램이 컴파일되는지 확인하기 위해 프로그램을 작성할 때 주기적으로 `cargo check` 을 실행합니다. 그런 다음 실행 파일을 사용할 준비가 되면 `cargo build` 를 실행합니다.  

Cargo에 대해 지금까지 배운 내용을 요약해 보겠습니다.

- `cargo new`를 사용하여 프로젝트를 만들 수 있습니다.  
- `cargo build`를 사용하여 프로젝트를 빌드할 수 있습니다.  
- `cargo run`을 사용하여 한 번에 프로젝트를 빌드하고 실행할 수 있습니다.  
- `cargo check`을 사용하여 바이너리를 생성하지 않고 오류를 확인하기 위해 프로젝트를 빌드할 수 있습니다.  
- Cargo는 빌드 결과를 코드와 같은 디렉토리에 저장하지 않고,  *target/debug* 디렉토리에 저장합니다.  


Cargo 사용의 또 다른 이점은 작업 중인 운영 체제에 관계없이 명령이 동일하다는 것입니다. 따라서 이제부터 Linux,  macOS 대 Windows에 대한 특정 지침을 더 이상 제공하지 않습니다.



### [출시를 위한 구축](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html#building-for-release)

프로젝트가 최종적으로 릴리스 준비가 되면 `cargo build --release`를 사용하여 최적화하여 컴파일할 수 있습니다. 이 명령은 *target/debug* 대신 *target/release* 에서 실행 파일을 생성합니다.

``` 
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    ├── debug
    │   ├── hello_cargo
    └── release
        └── hello_cargo (여기에 파일 생성)

```

최적화는 Rust 코드를 더 빠르게 실행하게 하지만, 최적화를 켜면 프로그램을 컴파일하는 데 걸리는 시간이 길어집니다. 이것이 두 가지 다른 프로필이 있는 이유입니다. 하나는 신속하고 자주 재빌드하려는 개발용이고 다른 하나는 반복적으로 재빌드되지 않고 최대한 빠르게 실행되는 사용자에게 제공할 최종 프로그램을 빌드하기 위한 것입니다. 코드의 실행 시간을 벤치마킹하는 경우, `cargo build --release` 를 실행한 다음,  *target/release* 에서 실행 파일을 실행하고 벤치마킹해야 합니다.



### [컨벤션으로서의 Cargo](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html#cargo-as-convention)

간단한 프로젝트에서는 Cargo가 단순히 `rustc`를 사용하는 것보다 많은 가치를 제공하지는 않습니다. 하지만 프로그램이 더 복잡해짐에 따라 Cargo는 그 가치를 입증할 것입니다. 프로그램이 여러 파일로 확장되거나 종속성이 필요하면 Cargo가 빌드를 조정하도록 하는 것이 훨씬 쉽습니다.

이 `hello_cargo`프로젝트는 간단하지만 이제 남은 Rust 경력에서 사용할 실제 도구를 많이 사용합니다. 실제로 기존에 있는 프로젝트에서 작업하려면, 다음 명령을 사용하여, Git 으로 코드를 확인하고, 해당 프로젝트의 디렉터리로 변경한 다음, 빌드할 수 있습니다.

```
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Cargo에 대한 자세한 내용은 [문서](https://doc.rust-lang.org/cargo/) 를 확인하세요.

## [요약](https://doc.rust-lang.org/book/ch01-03-hello-cargo.html#summary)

당신은 이미 Rust 여행을 훌륭하게 시작했습니다! 이 장에서는 다음 방법을 배웠습니다.

- `rustup`을 사용하여 안정적인 최신 버전의 Rust를 설치합니다.  
-  최신 Rust 버전으로 업데이트  
-  로컬로 설치된 문서 열기  
-  `rustc` 를 직접 사용하여,  "Hello, world!"를 작성하고 실행합니다.  
-  Cargo의 규칙을 사용하여 새 프로젝트를 생성하고 실행합니다.  

지금은 Rust 코드를 읽고 쓰는 데 익숙해지기 위해 보다 실질적인 프로그램을 구축할 수 있는 좋은 시간입니다. 그래서 2장에서는 추측 게임 프로그램을 만들 것입니다. Rust에서 일반적인 프로그래밍 개념이 어떻게 작동하는지 배우는 것으로 시작하려면 3장을 보고 2장으로 돌아가세요.
