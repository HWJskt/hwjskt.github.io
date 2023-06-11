+++
title = "Traits: Defining Shared Behavior"
weight = 2
template ="book_page_rust.html"

+++



## 요약




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
