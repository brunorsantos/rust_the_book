# Unit tests

A anotacao `#[cfg(test)]` nos modulos de teste falam que esse conteudo so deve ser compilado e executa no cargo test.

Rust permite testar funcoes privadas como:

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

# Integration tests

Para criar um estrutura de teste integrado, devemos ter uma pasta test na mesmo nivel que src.

```
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs

```

Em que esse teste vai se comportar como se estivesse fora da crate, portando precisamos importar a crate atual

```rust
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

Não precisamos anotar como `#[cfg(test)]` pois o cargo test ja identificará que é um modulo de teste por causa do diretorio test

Só é possivel essa estrutura de integration tests com library crates. Binary crates que contem um arquivo main.rs nao vao suportar