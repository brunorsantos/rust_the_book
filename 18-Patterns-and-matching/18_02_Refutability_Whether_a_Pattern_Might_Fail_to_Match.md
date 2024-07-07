Patterns podem ser refutavies ou irrefutaveis.

- Patterns que dao match com qualquer valor sao irrefutaveis. Por exemplo `x` em `let x = 5;` 

- Patterns que podem falhar para algum possivel sao refutaveis. Por exemplo `Some(x)` em ` if let Some(x) = a_value`. Pois se a variavel tem `None` o pattern nao vai dar match.



Function parameters, let statements, e for loops só podem aceitar pattern irrefutaveis .`if let`, `while let` expression aceitam refutaveis ou irrefutaveis(porem com um warn nesses casos)


Por exemplo, caso seja tentado utilizar isso:

```rust
    let Some(x) = some_option_value;
```


```
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
  |
  = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
  = note: the matched value is of type `Option<i32>`
help: you might want to use `let else` to handle the variant that isn't matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` (bin "patterns") due to 1 previous error
```

Significa que estamos tentando utilizar um pattern refutavel com `let` statement, o que acaba fazendo o codigo nao cobrir todos os casos. Sendo assum Rust nao compila.

Para corrigir isso faria sentido

```rust
    if let Some(x) = some_option_value {
        println!("{x}");
    }
```



No proximo exemplo seria tentando usar um pattern irrefutavel com `if let` statement

```rust
    if let x = 5 {
        println!("{x}");
    };
```


Rust compila, porem da o warning

```
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `if let` pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^
  |
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`
  = note: `#[warn(irrefutable_let_patterns)]` on by default

warning: `patterns` (bin "patterns") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
5
```