Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. 

Unlike functions, closures can capture values from the scope in which they’re defined

```rust
struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }
```

Estamos chamando o metodo `unwrap_or_else` do `Option<T>`  definido na std library, que espera uma closure sem argumentos e retorna um T (Mesmo tipo da option).

Interesante reaparar que a closure chama `self.most_stocked()` da instancia atual de Inventory. A std nao sabe nada sobre Inventory. A closure captura uma referencia imutavel da instancia de Inventory e passa ela para o `unwrap_or_else`.

## Inferencia de tipo e anotacao

Outra diferenca entre funcoes e closures, é que closures geralamente nao exigem que se anote o tipo na definicao.

Nao como funcoes que definem um contrato com usuarios (Tanto que nem nome tem). Sa geralmente curtas e relavantes apenas um contexto limitado. O compilador pode inferir os tipos e tipo do retorno.

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

As linhas 3 e 4, precisariam da closure ser avaliadas para poderem complilar, similar ao que ocorre com `let v = Vec::new();`

```rust
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
```

Nesse caso, rust vai inferir os tipos do parametros e retorno como String, apos a primeira execucao, depois vai dar erro na segunda pq estamos pasando um valor incopativel (inteiro)

## Capturando referencias ou movendo ownership

