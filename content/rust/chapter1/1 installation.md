+++
title = "Installation"
weight = 1
template ="book_page_rust.html"
+++


첫 번째 단계는 Rust를 설치하는 것입니다. Rust 버전 및 관련 도구를 관리하기 위한 명령줄 도구인 rustup을 통해 Rust를 다운로드합니다. 다운로드하려면 인터넷 연결이 필요합니다.

> Note: Rustup을 사용하지 않으려면 다른 Rust 설치 방법 페이지에서 더 많은 옵션을 확인하세요.

다음 단계는 Rust 컴파일러의 안정적인 최신 버전을 설치합니다. Rust의 안정성은 컴파일하는 책의 모든 예제가 최신 Rust 버전으로 계속 컴파일되도록 보장합니다. 즉, 이 단계를 사용하여 설치한 더 새롭고 안정적인 버전의 Rust는 이 책의 내용과 예상대로 작동해야 합니다. Rust는 종종 오류 메시지와 경고를 개선하기 때문에 결과가 버전마다 약간 다를 수 있습니다. 


> 명령줄(Command Line) 표기법  
이 장과 책 전반에 걸쳐 터미널에서 사용되는 몇 가지 명령을 보여드리겠습니다.  
터미널에 입력해야 하는 줄은 모두 \$로 시작합니다. \$ 문자를 입력할 필요가 없습니다. 각 명령의 시작을 나타내기 위해 표시되는 명령줄 프롬프트입니다.  
\$로 시작하지 않는 행은 일반적으로 이전 명령의 결과물을 표시합니다.  
또한 PowerShell 관련 예제에서는 \$ 대신 \>를 사용합니다.


## rustup을 Linux 또는 macOS에 설치
Linux 또는 macOS를 사용하는 경우 터미널을 열고 다음 명령을 입력합니다:  
`$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh`  
이 명령은 스크립트를 다운로드하고 rustup 안정적인 최신 버전의 Rust를 설치하는 도구 설치를 시작합니다. 비밀번호를 입력하라는 메시지가 표시될 수 있습니다. 설치에 성공하면 다음 줄이 나타납니다:  
`Rust is installed now. Great!`  

또한 Rust가 컴파일된 출력을 하나의 파일로 결합하는 데 사용하는 프로그램인 링커(linker)가 필요합니다. 이미 가지고 있을 가능성이 높습니다. 링커 오류가 발생하면 일반적으로 링커를 포함하는 C 컴파일러를 설치해야 합니다. 일부 일반적인 Rust 패키지는 C 코드에 의존하고 C 컴파일러가 필요하기 때문에 A C 컴파일러도 유용합니다.  
macOS에서는 다음을 실행하여 C 컴파일러를 얻을 수 있습니다:  
`$ xcode-select --install`  
Linux 사용자는 일반적으로 배포 문서에 따라 GCC 또는 Clang을 설치해야 합니다. 예를 들어 Ubuntu를 사용하는 경우 build-essential 패키지를 설치할 수 있습니다.  


## rustup을 Windows에 설치
Windows에서는 [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) 로 이동하여 Rust 설치 지침을 따르세요. 설치 중 특정 시점에 Visual Studio 2013 이상용 MSVC 빌드 도구도 필요하다는 메시지가 표시됩니다.

빌드 도구를 얻으려면 [Visual Studio 2022](https://doc.rust-lang.org/book/ch01-01-installation.html#:~:text=%EB%B9%8C%EB%93%9C%20%EB%8F%84%EA%B5%AC%EB%A5%BC%20%EC%96%BB%EC%9C%BC%EB%A0%A4%EB%A9%B4%20Visual%20Studio%202022%EB%A5%BC)를 설치해야 합니다. 설치할 워크로드를 물으면 다음을 포함합니다:

- -“Desktop Development with C++”
- -The Windows 10 or 11 SDK
- -선택한 다른 언어 팩과 함께 영어 언어 팩 구성 요소  
  
이 책의 나머지 부분에서는 cmd.exe 와 PowerShell 모두에서 작동하는 명령을 사용합니다. 구체적인 차이점이 있는 경우 어떤 것을 사용해야 하는지 설명하겠습니다.


## 문제 해결
Rust가 올바르게 설치되었는지 확인하려면 셸을 열고 다음 줄을 입력하십시오:  
`$ rustc --version`  

릴리스된 최신 안정 버전의 버전 번호, 커밋 해시 및 커밋 날짜가 다음 형식으로 표시되어야 합니다:  
`rustc x.y.z (abcabcabc yyyy-mm-dd)`  
이 정보가 보이면 Rust를 성공적으로 설치한 것입니다!  

이 정보가 보이지 않으면 Rust가 %PATH% 시스템 변수에 있는지 다음과 같이 확인하십시오.  
Windows CMD에서 다음을 사용합니다:  
`> echo %PATH%`  

PowerShell에서 다음을 사용합니다:  
`> echo $env:Path`  

Linux 및 macOS에서는 다음을 사용합니다:  
`$ echo $PATH`  

모든 것이 정확한데 Rust가 여전히 작동하지 않는다면 도움을 받을 수 있는 여러 곳이 있습니다. [커뮤니티 페이지](https://doc.rust-lang.org/book/ch01-01-installation.html#:~:text=%EC%97%AC%EB%9F%AC%20%EA%B3%B3%EC%9D%B4%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.-,%EC%BB%A4%EB%AE%A4%EB%8B%88%ED%8B%B0%20%ED%8E%98%EC%9D%B4%EC%A7%80,-%EC%97%90%EC%84%9C%20%EB%8B%A4%EB%A5%B8%20Rustacean) 에서 다른 Rustacean(우리가 스스로를 부르는 별명)과 연락하는 방법을 알아보세요.

## 업데이트 및 제거
rustup을 통해 Rust를 설치하면 새 버전으로 업데이트하는 것이 쉽습니다. 셸에서 다음 업데이트 스크립트를 실행합니다:  
`$ rustup update`  

Rust 및 rustup을 제거하려면 셸에서 다음 제거 스크립트를 실행합니다:  
`$ rustup self uninstall`  

## 로컬 문서
Rust 설치에는 문서의 로컬 사본도 포함되어 있어 오프라인에서 읽을 수 있습니다. `rustup doc`을 실행하여 브라우저에서 로컬 문서를 엽니다.  

타입(type)이나 함수(function)가 표준 라이브러리에서 제공되었는데 그것이 무엇을 하는지 또는 어떻게 사용하는지 잘 모를 때마다 API(Application Programming Interface) 문서를 사용하여 알아보세요!  