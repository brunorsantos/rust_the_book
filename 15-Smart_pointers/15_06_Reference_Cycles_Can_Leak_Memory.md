O memory safety torna bem dificil, mas nao impossivel criar memoria que nao é limpa (Memory leak).

Podemos ver que é possivel memory leak usando `Rc<T>` e `RefCell<T>`: Criando referencias onde itens se referem um ao outro em ciclo. O memory leak pode acontecer se o o contador de referencia nunca chegar a 0

## Criando um cicle de referencia (Reference cycle)

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
```

Deixando o segundo elemento como `RefCell<Rc<List>>` permite que no modificamos o `List` que o `Cons` está apontando.

```rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

O codigo cria uma listas em `a` e uma lista em `b` que aponta para a lista em `a`. Depois modifica a lista `a` para a apontar para `b`, criando um reference cycle. 

Os prints retornam:

```
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/cons-list`
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
```

O contador de referencias `Rc<List>` para `a` e `b` estão com 2 no fim. No fim do main Rust faz o `Drop` da variavel `b` que faz o contador diminuir de 2 para 1. Sendo assim o memoria que `Rc<List>` tem na heap nao será desalocada pq o contador é 1 e nao 0. Ao dropar `a`, tambem ocorrerá o mesmo.

A memoria alocada para a lista será mantida para sempre.

Se o ultimo `println!` for descomentado, Rust vai tentar printar esse cycle eternamente ate ocorrer um overflow.

## Previnindo Reference cycles: Tornando Rc<T> em Weak<T>

Ao chamar `Rc::clone` é incrementado o `strong_count` de um `Rc<T>`, e uma instancia de `Rc<T>` só é limpa quando o `strong_count` é zerado.

É possivel tambem criar uma weak reference para um valor de `Rc<T>` chamando `Rc:downgrade` e passando a referencia de um `Rc<T>`. referencias fracas nao expressao relacao de ownership e seu contador nao afeta quando o instancia `Rc<T>` é limpada.

Usando `Rc:downgrade` vc recebe um smart pointer do tipo `Weak<T>` que incramenta `weak_count`. 