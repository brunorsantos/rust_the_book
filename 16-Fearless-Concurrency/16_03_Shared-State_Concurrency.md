# Usando mutex para permitir acesso a um dado de uma trhead

Mutex permite apenas uma thread acessar um dados em um determinado tempo. Sendo que para acessoar primeiro devemos adquirir um lock. Que é uma estrutura que faz parte do mutex que controla quem tem acesso exclusivo ao dado.

Para usar mutex
- Deve ser tentar adquirir o lock antes
- Quando se termina de usar o dado que o mutex faz o guard, deve se fazer o unlock para outra thread pode adquirir o lock

# Api do Mutex<T>

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

Para acessar o dado dentro do mutex usamos `lock`. Essa chamada faz o block da thread atual, ela nao podera ser feito ate chegar a sua vez de ter o lock.

A chamada para o `lock` falha caso a outra thread com o lock panicar.

Depois de adquirir o lock, podemos tratar o valor retornado como uma referencia mutavel para o dado dentro.

`Mutex<T>` é um smart pointer. A chamada para um `lock` retorna um smart pointer chamado `MutexGuard`, dentro de um `LockResult`. O `MutexGuard` implementa `Deref` que aponta para o dado interno. O `MutexGuard` tambem implementa o `Drop` que libera o lock quando o `MutexGuard` sai de escopo. Sendo assim nao temos o risco de esquecer de liberar o lock e manter o block do mutex para outra threads.




# Compartilhando mutex entre threads


Apenas a tentativa incial:

```
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

```
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: borrow of moved value: `counter`
  --> src/main.rs:21:29
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because `counter` has type `Mutex<i32>`, which does not implement the `Copy` trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ------- value moved into closure here, in previous iteration of loop
...
21 |     println!("Result: {}", *counter.lock().unwrap());
   |                             ^^^^^^^ value borrowed here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `shared-state` (bin "shared-state") due to 1 previous error
```

O codigo cria um contador dentro de `Mutex<T>`. Depois cria 10 threads, usando `thread::spawn` e passa para todas threads a mesma closures: Que move o contador para a thread (Sempre é necessario ter o ownership em threads), adquire o lock do `Mutex<T>` e adicona 1. Quando a thread termina, como `num` sai de escopo e libera o lock.

Na thread main, fazemos o join de todas a threads para ter certeza que elas vão finalizar. Depois tentar adquirir o lock e printar.

O problema é que movemos o ownership no `num` para o closure e depois ao usar na thread main ele ja nao está mais disponivel


# Multiple ownerships com threads multiplas

Para consertar precisamos de ter multiplos ownerships. Como `Rc<T>` nao pode ser shareado entre threads devemos usar `Arc<T>`.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```


# Similarities Between RefCell<T>/Rc<T> and Mutex<T>/Arc<T>

`counter` é imutavel mas podemos pegar uma referencia mutavel do valor dentro dele. O que significa que `Mutex<T>` prove mutabilidade interior como `RefCell<T>` faz.