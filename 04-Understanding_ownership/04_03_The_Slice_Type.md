## String Slices

É um referencia a uma parte de uma String.

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

![Alt text](image-5.png)

![Alt text](image-6.png)

Representado por `&str`, siginifica que será sempre uma referencia a um slice de uma String existente em que essa String deve estar com lifetime ativo.

Slices podem ser apontados para strings literais e Strings.

## deref coercions

Imporante dizer que é possivel toda vez que uma funcao espera uma referencia de uma String é possivel mudar para um slice.

Ao inves de 

```rust
fn first_word(s: &String) -> &str {
```

Usar: 

```rust
fn first_word(s: &str) -> &str {
```

Dando mais liberdade para quem for utilizar a função.


## Outros slice

O mesmo comportamento se da para slices tipo `&[i32]` em relacao ao array `[i32,7]` ou para outras collections como Vectors