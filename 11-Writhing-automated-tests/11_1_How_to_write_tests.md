O attribute `#[test]` indica que uma funcao é de test para ser executada com `cargo test` 

Um teste falha quando ocorre um panic nele.





## Checking results with assert! macro

A macro `assert!` recebe um booleano e chama a macro `panic!` caso o valor seja falso.


Ao criar um projeto novo como lib, se é criado um modulo inline de teste.


```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```

Nota que dentro do modulo de `tests` usamos ` use super::*;` pois precisamos utilizar tudo no na raiz da crate.

## Testing Equality with the assert_eq! and assert_ne! Macros

Pode se utilizar as macros `assert_eq!` and `assert_ne!` para receber 2 valorer e checkar se eles são iguais.

Nas implementacoes essas macros utilizam os operadores `==` e `!=`, e quando falham elas printam os parametros utilzando debug formating. Entao os parametros precisam implementar `PartialEq` e `Debug`

É possivel passar um argumento a mais na macros utilizando a macro `format!` para ter uma mensagem custumizada nas falhas.

É possivel adicionar a diretriz `should_panic` nos testes

```rust
// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}

```