A trait `Drop` tambem é importante para os smart pointers. Para customizar o que acontece quando um valor sai de escopo. Ela é quase sempre utilizado quando smart pointers são utilizados para desalocar espaco na heap por exemplo.


```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

Vai imprimir:

```
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/debug/drop-example`
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

## Dropando valores cedo com `std::mem::drop`

As vezes precisamos chamar o Drpro mais cedo. Por exemplo quando usando smart pointers que lidam com lock: Voce pode querer forçar o dropr que libera o lock para outra parte do codigo possa adquirir o lock.

Rust nao permite chamar o `drop` da trait `Drop` manualmente, ao inves disso devemos chamar a. funcao `std::mem::drop` fornecida para std, se vc quer forçar o valor ser dropado antes de sair do escopo.

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```


```
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/drop-example`
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```

Não precisamos preocupar com problemas de acidentalmente limpar valores em uso. O sistema de ownership do rust garante que depois de uma chamada do drop o valor nao vai ser utilizado mais.


