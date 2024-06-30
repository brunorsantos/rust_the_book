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

## Pedindo review de um post muda o state

Ao pedir um reviw de um post, se muda o state de `Draft` para `PedingReview`

```rust
impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

Se adiciona um method `request_review` na trait state. Fazendo os tipo que implementam a trait terem que implementar o metodo.

Ao inves de esperar `self` ou `&mut self` o parametro é `self: Box<Self>`, significando que o metodo so é valido se chamado dentro de um `Box`.
A sintaxe pega ownership de `Box<Self>`, invalidando o estado antigo. Entao o estado se transforma em um novo tipo. (Retorna ele para `Post`)

Em `Post`, para o consumir o estado antigo. o metodo `request_review` precisa pegar ownership do valor em `state`. 
Para isso que precisamos do `Option` no campo `state`: Chamamos `take` para tirar o `Some` para fora de `state` e deixar `None` no lugar, pq Rust nao nos deixa manter campos despopulados em structs.

A gente esta deixando o campo `state` temporariamente como `None`. Ao inves de usar algo como `self.state = self.state.request_review();`.


## Adicionando `approve`

```rust
impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

## Adicionando logica para retornar content.

De novo vamos colocar regra no State.
Para isto vamos fazer o state saber se deve se retornar um conteudo de Post.

```rust
trait State {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// --snip--
struct Published {}

impl State for Published {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

A trait precisará receber a instancia de post para isto. 
Assim teremos um implementacao default que retorna vazio, em a struct `Published` pode sobrescrever essa implementacao.

Será preciso deixar explicito o lifetime do retorno vinculado com o `Post`.


```rust
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }
    // --snip--
}
```

Chamamos `as_ref` na `Option` pq queremos a referencia do valor dentro ao inves de pegar o ownership do valor.
`state` é `Option<Box<dyn State>>`, quando usamos `as_ref`, an `Option<&Box<dyn State>>` é retornado. 
Se nao usassemos, teríamos um erro pq nao podemos mover `state` para fora de `&self`

Usamos `unwrap` pois temos certeza que sempre teremos valor no `Some`.
