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