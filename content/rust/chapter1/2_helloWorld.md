+++
title = "Hello, World!"
weight = 2
template ="book_page_rust.html"
+++

## 요약

- 파일 형식은 카멜_형식 을 따른다.  
- main fn: 프로그램에서 처음 실행되는 함수  
- indent 는 4 space 이다. tab이 아니다  
- 함수뒤에 !가 붙으면 Rust macro를 의미한다  
- 코드의 끝에는 semicolon (;) 를 붙여야한다.  
- compile하기 :  $ rustc main.rs  
- 파일실행하기 : $ ./main  (윈도우는  .\main.exe)  



## [Hello, World!](https://doc.rust-lang.org/book/ch01-02-hello-world.html#hello-world)

이제 Rust를 설치했으므로 첫 번째 Rust 프로그램을 작성할 차례입니다. 새로운 언어를 배울 때 텍스트를 `Hello, world!`화면에 출력하는 작은 프로그램을 작성하는 것이 전통적이므로 여기서도 똑같이 할 것입니다!

> 참고: 이 책은 명령줄에 대한 기본적인 지식이 있다고 가정합니다. Rust는 편집이나 도구 또는 코드가 있는 위치에 대해 특정한 요구를 하지 않으므로 명령줄 대신 통합 개발 환경(IDE)을 선호하는 경우 선호하는 IDE를 자유롭게 사용하세요. 이제 많은 IDE가 어느 정도 Rust를 지원합니다. 자세한 내용은 IDE 설명서를 확인하십시오. Rust 팀은 `rust-analyzer`를 통해서 훙륭한 IDE를 지원하는데 집중하고 있습니다. 자세한 내용은 [부록 D](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html) 를 참조하십시오.



### [프로젝트 디렉토리 생성](https://doc.rust-lang.org/book/ch01-02-hello-world.html#creating-a-project-directory)

Rust 코드를 저장할 디렉토리를 만드는 것으로 시작합니다. 당신의 코드가 어디에 있는지는 Rust에게 중요하지 않지만 이 책의 연습과 프로젝트를 위해 우리는 당신의 홈 디렉토리에 *프로젝트* 디렉토리를 만들고 거기에 모든 프로젝트를 보관할 것을 제안합니다.  
터미널을 열고 다음 명령을 입력하여 *projects* 디렉토리를 만들고, *projects* 디렉토리 내에 "Hello, world!" 프로젝트 디렉토리인 *hello_world* 디렉토리를 만듭니다.  

Windows의 Linux, macOS 및 PowerShell의 경우 다음을 입력합니다.  

```
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```
<br>

Windows CMD의 경우 다음을 입력합니다.

```
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```



### [Rust 프로그램 작성 및 실행](https://doc.rust-lang.org/book/ch01-02-hello-world.html#writing-and-running-a-rust-program)

다음으로 새 소스 파일을 만들고 이름을 *main.rs* 로 지정합니다. Rust 파일은 항상 *.rs* 확장자로 끝납니다. 파일 이름에 두 개 이상의 단어를 사용하는 경우 규칙은 밑줄을 사용하여 구분하는 것입니다. 예를 들어 *helloworld.rs* 대신 *hello_world.rs* 를 사용합니다.

이제 방금 만든 *main.rs* 파일을 열고 Listing 1-1의 코드를 입력합니다.


파일명: main.rs

```rust
fn main() {
    println!("Hello, world!");
}
```

목록 1-1: `Hello, world!` 를 프린트하는 프로그램

파일을 저장하고 *~/projects/hello_world* 디렉터리의 터미널 창으로 돌아갑니다. Linux 또는 macOS에서 다음 명령을 입력하여 파일을 컴파일하고 실행합니다.

```
$ rustc main.rs
$ ./main
Hello, world!
```
<br>
  
Windows에서는 `./main` 대신 `.\main.exe` 명령을 입력합니다.

```
> rustc main.rs
> .\main.exe
Hello, world!
```

<br>

운영 체제에 관계없이 `Hello, world!` 문자열이 터미널에 프린트되어야 합니다. 이 출력이 표시되지 않으면 설치 섹션의 [문제해결](https://doc.rust-lang.org/book/ch01-01-installation.html#troubleshooting) 부분을 다시 참조하여 도움을 받으세요.

`Hello, world! 가 프린트 되었다면 축하합니다! 공식적으로 Rust 프로그램을 작성했습니다. 그것은 당신을 Rust 프로그래머로 만들어줍니다. 환영합니다!



### [Rust 프로그램 분석](https://doc.rust-lang.org/book/ch01-02-hello-world.html#anatomy-of-a-rust-program)

이 "Hello, world!"프로그램을 자세히 검토해 봅시다.  퍼즐의 첫 번째 조각은 다음과 같습니다.

```rust
fn main() {

}
```

이 줄은 `main` 이라는 함수를 정의합니다. 이 `main` 의 기능은 특별합니다. 항상 모든 실행 가능한 Rust 프로그램에서 실행되는 첫 번째 코드입니다. 여기서 첫 번째 줄은 매개변수가 없고 아무 것도 반환하지 않는 `main` 이라는 이름의 함수를 선언합니다. 매개변수가 있으면  `()`괄호 안에 들어갑니다.  
함수 본문은 `{}` 로 감싸져 있습니다. Rust는 모든 함수 본문 주위에 중괄호가 필요합니다. 여는 중괄호를 함수 선언과 같은 줄에 배치하고, 그 사이에 공백을 하나 추가하는 것이 좋습니다.

> 참고: Rust 프로젝트 전체에서 표준 스타일을 고수하려면, 코드를 특정 스타일로 포맷하기 위해 호출되는 자동 포맷터 도구인 `rustfmt`를 사용할 수 있습니다. (자세한 내용은 [부록 D](https://doc.rust-lang.org/book/appendix-04-useful-development-tools.html) `rustfmt` 참조 ). Rust 팀은 이 도구를 `rustc` 처럼 표준 Rust 배포판에 포함시켰으므로 컴퓨터에 이미 설치되어 있어야 합니다!

<br>

`main` 함수 본문에는 다음 코드가 포함됩니다.

```rust
    println!("Hello, world!");
```

이 줄은 이 작은 프로그램의 모든 작업을 수행합니다. 화면에 텍스트를 인쇄합니다. 여기에서 주목해야 할 네 가지 중요한 세부 사항이 있습니다.

첫째, Rust 스타일은 탭이 아닌 네 개의 공백으로 들여쓰기하는 것입니다.  
둘째, `println!` 는 Rust 매크로를 호출합니다. 만약 함수를 호출했다면 `println`(`!`없이 )를 입력합니다. 19장에서 Rust 매크로에 대해 더 자세히 논의할 것입니다. 지금은 `!`  를 사용함으로써 일반적인 함수 대신 매크로를 호출하고, 매크로가 항상 함수와 동일한 규칙을 따르지 않는다는 것을 의미한다는 것을 알아야 합니다.  
셋째, `"Hello, world!"` 문자열이 보입니다. 이 문자열을 `println!`의 인수로 전달하면 문자열이 화면에 출력됩니다.  
넷째, 세미콜론( `;`)으로 행을 종료합니다. 이는 이 표현(expression)이 끝났고 다음 표현을 시작할 준비가 되었음을 나타냅니다. 대부분의 Rust 코드 줄은 세미콜론으로 끝납니다.  



### [컴파일과 실행은 별도의 단계입니다.](https://doc.rust-lang.org/book/ch01-02-hello-world.html#compiling-and-running-are-separate-steps)

방금 새로 만든 프로그램을 실행했으므로 프로세스의 각 단계를 살펴보겠습니다.  
Rust 프로그램을 실행하기 전에 다음과 같이 `rustc`명령을 입력하고 소스 파일의 이름을 전달합니다. 그러면 Rust 컴파일러가 컴파일합니다. 

```
$ rustc main.rs
```

C 또는 C++ 배경 지식이 있는 경우 이것이 `gcc` 또는 `clang`와 유사하다는 것을 알 수 있습니다. 성공적으로 컴파일한 후 Rust는 바이너리 실행 파일을 생성합니다.  

Linux, macOS 및 Windows의 PowerShell에서는 셸에 `ls` 명령을 입력하여 실행 파일을 볼 수 있습니다.

```
$ ls
> main  main.rs
```

Linux 및 macOS에서는 두 개의 파일이 표시됩니다. Windows에서 PowerShell을 사용하면 CMD를 사용하여 볼 수 있는 것과 동일한 세 개의 파일이 표시됩니다. Windows에서 CMD를 사용하면 다음을 입력합니다.

```
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

여기에는 확장자가 *.rs* 인 소스 코드 파일, 실행 파일( Windows에서는 *main.exe*, 다른 모든 플랫폼에서는 *main*), Windows를 사용하는 경우 확장자가 *.pdb* 인 디버깅 정보가 포함된 파일이 표시됩니다. 여기에서 다음과 같이 *main* 또는 *main.exe* 파일을 실행합니다.

```
$ ./main (Linux 및 macOS)
> .\main.exe (Windows)
```

*main.rs* 가 "Hello, world!"프로그램인 경우,  `Hello, world!` 가 터미널에 인쇄됩니다.  

Ruby, Python 또는 JavaScript와 같은 동적 언어에 더 익숙한 경우 별도의 단계로 프로그램을 컴파일하고 실행하는 데 익숙하지 않을 수 있습니다.  
Rust는 *미리 컴파일된* 언어입니다. 즉, 프로그램을 컴파일하고 다른 사람에게 실행 파일을 제공할 수 있으며 Rust가 설치되지 않은 상태에서도 실행할 수 있습니다. 누군가에게 *.rb* , *.py* 또는 *.js* 파일을 제공하는 경우 Ruby, Python 또는 JavaScript 구현이 각각 설치되어 있어야 합니다. 그러나 이러한 언어에서는 프로그램을 컴파일하고 실행하는 데 하나의 명령만 필요합니다. 언어 설계에서는 모든 것이 트레이드 오프입니다.

 `rustc`로 간단하게 컴파일하는 것은 간단한 프로그램에 적합하지만 프로젝트가 커짐에 따라 모든 옵션을 관리하고 코드를 쉽게 공유할 수 있게 만들고 싶을 것입니다. 다음으로 실제 Rust 프로그램을 작성하는 데 도움이 되는 Cargo 도구를 소개합니다.
