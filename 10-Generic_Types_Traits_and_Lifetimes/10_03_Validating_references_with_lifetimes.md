## Borrow checker

Compara escopos para determinar se todos os borrows sao validos.

```rust
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```
Neste exemplo, rust ve que `'r` tem lifetime de `'a` mas ele se refere para a um item na memoria com lifetime de `'b`. O programa é rejeitado, pois `'b` é menor do que `'a`: O sujeito da referencia, nao vive tanto quanto a referencia.

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

Neste caso x tem lifetime `'b` que nesse caso é maior do que `'a`. Significa que r pode referenciar x. Pois o Rust sabe que que a referencia de  `r` sempre vai estar valida quando `x` for valido.

Em resumo, nao pode ocorrer de uma de uma referencia morrer antes de quem está referenciando