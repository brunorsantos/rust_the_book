## Rexportando nome com pub use

Pub use, faz com o que a gente importou para o nosso escopo, que a principio vira um nome privado seja agora publico.

Isto é fora do nosso escopo será possivel tambem acessar esse item.

Ex: os modulos filhos de um modulo que da pub use 

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```