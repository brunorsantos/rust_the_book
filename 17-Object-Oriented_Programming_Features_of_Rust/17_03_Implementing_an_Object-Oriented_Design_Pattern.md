# Implementando um design pattern orientado a objeto

Vamos implementar um workflow para posts em um blog


1 - A blog post starts as an empty draft.
2 - When the draft is done, a review of the post is requested.
3 - When the post is approved, it gets published.
4 - Only published blog posts return content to print, so unapproved posts can’t accidentally be published.

Usando esses tests que devem passar no futuro:

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```


# Definindo post e criando instancia em state de Draft

Comecando com a struct `Post` que tenha um context. E criando um pubic `new` que criará a instancia.

CRiando tambem um trait privada `State` que definirá o comportamento de todos os stados de um `Post` devem ter.

`Post` será dono de uma trait object de `Box<dyn State>` dentro de um `Option<T>`.(Explicacao do pq depois)

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

## Salvando o texto do content do Post

Nao vamos deixar o `content` publico e sim criar um metodo para adicionar um texto

```rust
impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

## Garantindo que o draft tenha content vazio

Por agora vamos implementar um metodo que retorna o content como vazio sempre

```rust
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        ""
    }
}
```


