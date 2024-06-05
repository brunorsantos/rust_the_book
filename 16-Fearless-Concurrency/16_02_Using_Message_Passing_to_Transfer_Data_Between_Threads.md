Rust std prove canais para passar mensagens entre threads


`channel` tem tranmissores e receptores. O canal é fechado se ou o transmissor ou receptor sao dropados

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

Usando `thread::spawn` para criar um thread e o `move` para explicitamente mover o ownership para a closure. A thread precisar ser dona do transmiter para poder mandar mensagens para o canal.
O metodo `send` retorna um `Result<T, E>`, para o caso do receptor ja tiver sido dropado.



```rust

use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

O receiver tem 2 metodos uteis, `recv` e `try_recv`. `recv` bloqueia a execucao da thread e espera ate o valor ser enviado para o canal. Quando ele enviado ele é retornado em um `Result<T, E>`. Quando o transmiter é fechado será retornado um erro como um sinal de  que nao será recebidas mais mensagens.

O metodo `try_recv` nao bloqueia. Ele retorna imediatamente um `Ok` com a mensagem ou um `Err`se nao existem mensagens. É util em algum loop com exemplo.

## Ownership na transferencia

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

```
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: `val`
  --> src/main.rs:10:31
   |
8  |         let val = String::from("hi");
   |             --- move occurs because `val` has type `String`, which does not implement the `Copy` trait
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
   |
9  |         tx.send(val.clone()).unwrap();
   |                    ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `message-passing` (bin "message-passing") due to 1 previous error
```
O metodo `send` pega o ownership do parametro, e o valor é movido, o receiver que passa a ser dono dele.


## Enviando multiplas mensagens

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

Estamos aqui tratando o rx como um iterator. Para cada valor é printado, e quando o canal é fechado a iteracao encerra.
(Entendi que o canal é fechado quando o tx é dropado no fim da closure)
