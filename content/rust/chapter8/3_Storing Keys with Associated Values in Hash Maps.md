+++
title = "Storing Keys with Associated Values in Hash Maps"
weight = 3
template ="book_page_rust.html"

+++

## 요약

- 잘 안쓴다
- 파이썬의 딕셔너리와 비슷
  - key ,  value 를 매칭해서 저장한다.
  - 내부 값에 순서가 없다.
- 모든 key 는 같은 타입,  모든 value 는 같은 타입
- 해시 맵 만들기
  - 빈 해시 맵만 만들수 있다.
  - `new`사용
- 해시 맵에 내용추가 : 
  - .insert 사용
  - 추가한 key ,  value은 해시맵이 소유한다. (다시 사용할 수 없다)
  - 참조한 건 소유권이 해시맵으로 넘어가지 않는다
- 해시 맵의 내용 수정
  - `.insert` : 키,값이 없으면 새로 만들고,  키가 있으면, 값을 덮어쓴다
  -  `.entry(key).or_insert(value)` : 키가 없을 때만 키,값 추가
  
- 해시 맵의 내용 읽기
  - [index]  : 값을 바로 얻음
  - .get(index) : Option<&T> 형태로 값을 얻음
- 해시 맵 내부 값 iteration
  - 일반적인 for 문 사용가능
  - 참조한 벡터를 for 문이 사용중이면, for문 내부에서 벡터를 수정할수 없다

## [해시 맵에 연관된 값과 함께 키 저장](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#storing-keys-with-associated-values-in-hash-maps)

공통 컬렉션의 마지막은 *해시 맵(hash map)* 입니다. `HashMap<K, V>` 타입은  `K` 타입의 키를 `V` 타입의 값에 매핑한 내용을 저장합니다. 이 때 *해싱 함수를* 사용하는데, 이것은 키와 값을 메모리에 배치하는 방법을 결정합니다.   많은 프로그래밍 언어가 이러한 종류의 데이터 구조를 지원하지만 다른 이름(예: 해시, 맵, 객체, 해시 테이블, 사전 또는 연관 배열)을 사용하는 경우가 많습니다.

해시 맵은 벡터와 마찬가지로 인덱스를 사용하지 않고 모든 타입의 키를 사용하여 데이터를 조회하려는 경우에 유용합니다. 예를 들어 게임에서 각 키가 팀의 이름이고 값이 각 팀의 점수인 해시 맵에서 각 팀의 점수를 추적할 수 있습니다. 팀 이름이 주어지면 점수를 검색할 수 있습니다.

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

벡터와 마찬가지로 해시 맵은 데이터를 힙에 저장합니다. 이 `HashMap`에는 `String` 타입의 키와 `i32` 타입의 값이 있습니다. 벡터와 마찬가지로 해시 맵은 동질적입니다. 모든 키는 서로 같은 타입이어야 하고 모든 값은 같은 타입이어야 합니다.

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

여기에서 `score`는 파란색 팀과 관련된 값을 가지며 결과는 `10`이 됩니다. `get` 메서드는 `Option<&V>`를 반환합니다. 해시 맵에 해당 키에 대한 값이 없으면 `get`은 `None`을 반환합니다. 이 프로그램은 `Option<&i32>` 대신 `Option<i32>`을 얻기 위해 `copied()`을 호출하여 `Option`을 처리합니다. `unwrap_or`를 사용하여 `scores`에 키에 대한 항목이 없는 경우 `score`를 0으로 설정합니다.

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

```
Yellow: 50
Blue: 10
```

### [해시 맵 및 소유권](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hash-maps-and-ownership)

`i32`처럼, `Copy` 특성을 구현하는 타입의 경우 값이 해시 맵에 복사됩니다. `String`과 같은 소유된 값의 경우 목록 8-22에 표시된 것처럼 값이 이동되고 해시 맵이 해당 값의 소유자가 됩니다.

```rust
    use std::collections::HashMap;

    let field_name = String::from(`Favorite color`);
    let field_value = String::from(`Blue`);

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
```

Listing 8-22: 키와 값이 삽입되면 해시 맵이 키와 값을 소유함을 보여줌

변수 `field_name` 및 `field_value`가 `insert` 호출로 해시 맵으로 이동된 후에는 사용할 수 없습니다.

값에 대한 참조를 해시 맵에 삽입하면 값이 해시 맵으로 이동되지 않습니다. 참조가 가리키는 값은 적어도 해시 맵이 유효한 동안에는 유효해야 합니다. [10장의 `수명이 있는 참조 유효성 검사`](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes) 섹션 에서 이러한 문제에 대해 자세히 설명합니다.

### [해시 맵 업데이트](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#updating-a-hash-map)

키와 값 쌍의 수는 증가할 수 있지만 각 고유 키는 한 번에 하나의 값만 연결할 수 있습니다(반대의 경우도 마찬가지입니다. 예를 들어 파란색 팀과 노란색 팀 모두 ` scores`해시 맵에 10 이라는 값을 가질 수 있습니다).

해시 맵의 데이터를 변경하려면, 키에 이미 할당된 값이 있는 경우를 처리하는 방법을 결정해야 합니다. 

- 이전 값을 완전히 무시하고 이전 값을 새 값으로 바꿀 수 있습니다. 
- 이전 값을 유지하고 새 값을 무시하고 키에 이미 값이 *없는 경우에만*  새 값을 추가할 수 있습니다.
- 또는 이전 값과 새 값을 결합할 수 있습니다. 

각 작업을 수행하는 방법을 살펴보겠습니다!

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

이 코드는 `{Blue: 25}`를 인쇄합니다. `10`의 원래 값을 덮어썼습니다.

#### [키가 없는 경우에만 키와 값 추가](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#adding-a-key-and-value-only-if-a-key-isnt-present)

특정 키가 값과 함께 해시 맵에 이미 존재하는지 확인한 후 다음 조치를 취하는 것이 일반적입니다. 키가 해시 맵에 존재하는 경우 기존 값은 그대로 유지되어야 합니다. 키가 없으면 키와 값을 삽입합니다.

해시 맵에는 확인하려는 키를 매개변수로 사용하는 `entry`이라는 특수 API가 있습니다. `entry` 메서드의 반환 값은 존재하거나 존재하지 않을 수 있는 값을 나타내는 `Entry`라는 열거형입니다. Yellow 팀의 키에 연결된 값이 있는지 확인하고 싶다고 가정해 보겠습니다. 그렇지 않은 경우 값 50을 삽입하고 Blue 팀에도 동일하게 삽입하려고 합니다. `entry` API를 사용하는 코드는 Listing 8-24와 같습니다.

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

목록 8-24의 코드를 실행하면 `{Yellow: 50, Blue: 10}`가 인쇄됩니다. `entry`에 대한 첫 번째 호출은 노란색 팀에 이미 값이 없기 때문에 값이 50인 노란색 팀의 키를 삽입합니다. `entry`에 대한 두 번째 호출은 Blue 팀이 이미 값 10을 가지고 있기 때문에 해시 맵을 변경하지 않습니다.

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

이 코드는 `{world: 2, hello: 1, wonderful: 1}`을 인쇄합니다. 동일한 키/값 쌍이 다른 순서로 인쇄된 것을 볼 수 있습니다. [`해시 맵에서 값 액세스`](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#accessing-values-in-a-hash-map) 섹션에서 해시 맵을 반복하는 것이 임의의 순서로 발생한다는 것을 기억하십시오.

`split_whitespace` 메서드는 `text` 값의 공백으로 구분된 하위 슬라이스에 대한 반복자를 반환합니다. `or_insert` 메서드는 지정된 키의 값에 대한 변경 가능한 참조(`&mut V`)를 반환합니다. 여기에서 `count` 변수에 가변 참조를 저장하므로 해당 값에 할당하려면 먼저 별표(`*`)를 사용하여 `count`를 역참조해야 합니다. 변경 가능한 참조는 `for` 루프의 끝에서 범위를 벗어나므로 이러한 모든 변경은 안전하고 차용 규칙에 의해 허용됩니다.

### [해싱 함수](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#hashing-functions)

기본적으로 `HashMap`은 해시 테이블 [1](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#siphash) 과 관련된 서비스 거부(DoS) 공격에 대한 저항을 제공할 수 있는 *SipHash* 라는 해싱 기능을 사용합니다. 이것은 사용 가능한 가장 빠른 해싱 알고리즘은 아니지만 성능 저하와 함께 제공되는 더 나은 보안을 위한 트레이드 오프는 그만한 가치가 있습니다. 코드를 프로파일링하고 기본 해시 함수가 용도에 비해 너무 느린 경우 다른 해시를 지정하여 다른 함수로 전환할 수 있습니다. *해셔*는 `BuildHasher` 특성을 구현하는 타입입니다. 우리는 10장에서 트레이트와 이를 구현하는 방법에 대해 이야기할 것입니다. 처음부터 자신의 해셔를 구현할 필요는 없습니다. [crates.io](https://crates.io/)에는 많은 일반적인 해싱 알고리즘을 구현하는 해셔를 제공하는 다른 Rust 사용자가 공유하는 라이브러리가 있습니다.

1. https://en.wikipedia.org/wiki/SipHash

## [요약](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#summary)

벡터, 문자열 및 해시 맵은 데이터를 저장, 액세스 및 수정해야 할 때 프로그램에 필요한 많은 기능을 제공합니다. 다음은 해결하기 위해 갖추어야 할 몇 가지 연습입니다.

- 정수 목록이 주어지면 벡터를 사용하고 목록의 중앙값(정렬할 때 중간 위치의 값)과 최빈값(가장 자주 발생하는 값, 여기에서 해시 맵이 도움이 됨)을 반환합니다.
- 문자열을 돼지 라틴어로 변환합니다. 각 단어의 첫 자음이 단어의 끝으로 이동되고 “ay”가 추가되므로 “first”는 “irst-fay”가 됩니다. 모음으로 시작하는 단어는 대신 끝에 `hay`가 추가됩니다(`apple`은 `apple-hay`가 됨). UTF-8 인코딩에 대한 세부 사항을 명심하십시오!
- 해시 맵과 벡터를 사용하여 사용자가 회사의 부서에 직원 이름을 추가할 수 있는 텍스트 인터페이스를 만듭니다. 예를 들어 `엔지니어링에 Sally 추가` 또는 `영업에 Amir 추가`가 있습니다. 그런 다음 사용자가 부서의 모든 사람 또는 부서별로 회사의 모든 사람 목록을 사전순으로 정렬하여 검색하도록 합니다.

표준 라이브러리 API 문서는 이러한 연습에 도움이 될 벡터, 문자열 및 해시 맵에 있는 메서드를 설명합니다!

작업이 실패할 수 있는 더 복잡한 프로그램에 들어가고 있으므로 오류 처리에 대해 논의하기에 완벽한 시기입니다. 다음에 그렇게 하겠습니다!





```rust
use std::collections::HashMap;

fn main() {
    let numbers = vec![4, 2, 8, 6, 4, 5, 3, 8, 4, 7, 2, 9, 4, 6, 1];

    let median = calculate_median(&numbers);
    println!("Median: {}", median);

    let mode = calculate_mode(&numbers);
    println!("Mode: {}", mode);
}

fn calculate_median(numbers: &Vec<i32>) -> f64 {
    let mut sorted_numbers = numbers.clone();
    sorted_numbers.sort();

    let length = sorted_numbers.len();
    if length % 2 == 0 {
        let middle1 = sorted_numbers[length / 2 - 1];
        let middle2 = sorted_numbers[length / 2];
        return (middle1 + middle2) as f64 / 2.0;
    } else {
        return sorted_numbers[length / 2] as f64;
    }
}

fn calculate_mode(numbers: &Vec<i32>) -> i32 {
    let mut freq_map: HashMap<i32, i32> = HashMap::new();
    for &num in numbers {
        *freq_map.entry(num).or_insert(0) += 1;
    }

    let mut mode = 0;
    let mut max_freq = 0;

    for (&num, &freq) in &freq_map {
        if freq > max_freq {
            mode = num;
            max_freq = freq;
        }
    }

    mode
}
```



```rust
fn main() {
  	let input = "Convert strings to pig latin";
    let words: Vec<&str> = input.split_whitespace().collect();
    
    let pig_latin_words: Vec<String> = words
        .iter()
        .map(|word| convert_to_pig_latin(word))
        .collect();
    
    let pig_latin_sentence = pig_latin_words.join(" ");
    println!("{}", pig_latin_sentence);
}

fn convert_to_pig_latin(word: &str) -> String {
    let vowels = ['a', 'e', 'i', 'o', 'u'];
    let mut chars = word.chars();
    
    let first_char = chars.next().unwrap();
    
    if vowels.contains(&first_char.to_ascii_lowercase()) {
        format!("{}-hay", word)
    } else {
        let rest: String = chars.collect();
        format!("{}-{}ay", rest, first_char)
    }
}
```

```rust
use std::collections::{HashMap, BTreeMap};
use std::io::{self, Write};

fn main() {
    let mut departments: HashMap<String, Vec<String>> = HashMap::new();

    loop {
        println!("사용법: [부서]에 [직원 이름] 추가");
        print!("명령어 입력: ");
        io::stdout().flush().unwrap();

        let mut input = String::new();
        io::stdin().read_line(&mut input).expect("입력을 읽을 수 없습니다.");

        let input = input.trim();

        if input == "종료" {
            println!("프로그램을 종료합니다.");
            break;
        }

        let parts: Vec<&str> = input.split_whitespace().collect();
        if parts.len() >= 4 && parts[1] == "에" && parts[2] == "추가" {
            let department = parts[0];
            let employee = parts[3];
            departments.entry(department.to_string()).or_default().push(employee.to_string());
            println!("'{}' 부서에 '{}'를 추가했습니다.", department, employee);
        } else {
            println!("잘못된 명령어 형식입니다.");
        }
    }

    println!("부서별 직원 목록:");
    let mut sorted_departments: BTreeMap<String, Vec<String>> = BTreeMap::new();
    for (department, employees) in &departments {
        let mut sorted_employees = employees.clone();
        sorted_employees.sort();
        sorted_departments.insert(department.clone(), sorted_employees);
    }

    for (department, employees) in &sorted_departments {
        println!("{} 부서:", department);
        for employee in employees {
            println!(" - {}", employee);
        }
    }
}

```

