Trais vão definir um comportamento comum a serem utilizados entre tipos.

A trait `Summary` poderia servir para diversos tipo diferentes de publicacoes como tweets, artigos, etc

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

## Implementando

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle{
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
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
        format!("{}: {}", self.username, self.content)
    }
}
```

Summary precisa estar no escopo, e outras crates tambem podem trazer ela para o scope e implementar para o proprio tipo. 
Porem existe uma regra que ou o tipo ou a crate devem estar na sua trait para isso ser permitido. 
Ex: 
- pode se implementar Dicplay para um tipo seu Tweet
- Pode se implementar Summary para um Vec<T>
- Nao se pode se implementar Display para um Vec<T>

## Default implementations

É possivel criar deafult implementation e alem disso elas chamarem outras implementaçoes (mesmo que nao sejam default implementations).

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

```rust
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());

```

## Trait as parameters (Trait bound)

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

A funçao exige que o parametro implemente a trait `Summmary`, sendo assim nos casos acima seria possivel passar tanto `Tweet` quanto `NewsArticle` 

Essa sintaxe é um atalho para essa usando generics:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

## Se quisermos garantir mais de um trait bound, usamos + nas duas formas:

```rust
pub fn notify(item: &(impl Summary + Display)) {
```

```rust
pub fn notify<T: Summary + Display>(item: &T) {
```

Podemos tambem usar o where para melhorar legibilidade em casos como 

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

Fica melhor assim:

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

## Retornando types que implementam Traits

É possivel usar `impl Trait` para retornar um valor em uma função.


```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

A função retorna um Tweet, mas o chamador nao precisa de saber disso.
Só pode ser utilizado se vc está retornando um single type.

## Usando trait baound para condicionalmento implementar metodos

Esse caso abaixo, o cmp_display so vai ser implementado quanto o tipo T implementar `Display` e `PartialOrd` 

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
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## Blanket implementations

Sao implementacoes condicionais, que imeplementam uma trait para qualquer tipo que implemente outra trait. É muito utilizada na std do Rust. Por exemplo, a Std implementa a `ToString` trait para qualquer tipo que implemente a trait Display. Seria como:

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

