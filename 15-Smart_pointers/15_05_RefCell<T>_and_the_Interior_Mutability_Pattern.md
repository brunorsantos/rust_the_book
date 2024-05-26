# RefCell<T> and the Interior Mutability Pattern

Mutability Pattern é um design pattern no rust que permite mutar o dado mesmo quando  há referencia imutavel. Normalmente isso nao é permitido.

Relembrando as regras:

- At any given time, you can have either (but not both) one mutable reference or any number of immutable references.
- References must always be valid.

Com `RefCell<T>`essa regras são seguidas porem apenas em tempo de execucao, dando panic quando acontecer um problema.

Here is a recap of the reasons to choose Box<T>, Rc<T>, or RefCell<T>:

- Rc<T> enables multiple owners of the same data; Box<T> and RefCell<T> have single owners.
- Box<T> allows immutable or mutable borrows checked at compile time; Rc<T> allows only immutable borrows checked at compile time; RefCell<T> allows immutable or mutable borrows checked at runtime.
- Because RefCell<T> allows mutable borrows checked at runtime, you can mutate the value inside the RefCell<T> even when the RefCell<T> is immutable.

# Interior Mutability: Um Mutable Borrow para um valor imutavel

Esse codigo nao compila, pois se vc tem um valor imutavel, nao se pode fazer um mutable borrow

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}
```

Porem existem situacoes em que isso pode ser util. Por exemplo:

## A Use Case for Interior Mutability: Mock Objects

Com essa implementacao, em que um LimitTracker precisa mandar uma mensagem quando acionar um limite de uso(usando um trait) precisamos testar se as mensagens sao enviadas (sem enviar na pratica claro).

Tendo a implementacao na seguinte forma:

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

A ideia seria implementar uma struct de Mock para a trait de messenger que armazenasse as mensagens enviadas, porem isso não possivel pois a chamada da funcao da trait a ser implementada, faz uma referencia immutavel a self.

A implementacao (que nao funciona) seria:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

Teriamos o erro:

```
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
error[E0596]: cannot borrow `self.sent_messages` as mutable, as it is behind a `&` reference
  --> src/lib.rs:58:13
   |
58 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^ `self` is a `&` reference, so the data it refers to cannot be borrowed as mutable
   |
help: consider changing this to be a mutable reference
   |
2  |     fn send(&mut self, msg: &str);
   |             ~~~~~~~~~

For more information about this error, try `rustc --explain E0596`.
error: could not compile `limit-tracker` (lib test) due to 1 previous error
```

É uma situacao que a interior mutability pode ajudar com:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```
Podemos agora chamar `borrow_mut` em `RefCell<Vec<String>>` em `self.sent_messages` para obter uma referencia mutavel para o valor dentro de `RefCell<Vec<String>>`.


## Mantendo controle dos Borrows em tempo de execucao com `RefCell<T>`

Com `RefCell<T>` nós podemos usar `borrow` e `borrow_mut`. Que são smart pointers que retornam `Ref<T>` e `RefMut<T>`. Ambos implementam `Deref` entao podem ser tratados como referencias normais.

`RefCell<T>` mantem controle da quantidade de Refs ou RefMuts ativos. Controlando tambem o limite de ter apenas um `RefMut` ativo por vez(Caso seja feita a tentativa de ter mais de um que ocorre o panic)

## Tendo multiplos owners, combinando `Rc<T>` and `RefCell<T>`

Um jeito comum de usar `RefCell<T>` é combinando com `Rc<T>`. Sendo que o `Rc<T>` permite multiplos donos, porem só te da acesso imutavel a esse dado.

Sendo assim combinando os dois você consegue tambem mutar o dado.

Usando a Cons list como exemplo:

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

