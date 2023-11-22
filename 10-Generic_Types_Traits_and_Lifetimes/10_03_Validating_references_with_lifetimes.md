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

## Generic lifetime in functions

Na funcao abaixo como exemplo, nao vai compilar pois o borrow checker nao vai consegior determinar qual lifetime da referencia de retorno para fazer os checks apropriados na complilação.

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Para corrigir, devemos adicionar os lifetimes na assinatura da funcao. Dizendo: A referencia retornada será valida, enquanto ambos parametros.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Agora a funcao diz que considerando o lifetime `'a`, a funcao recebe 2 parametros, que sao string slices que vivem pelo menos ao mesmo tempo que `'a`. E diz tambem que o string slice retornado vai viver pelo menos no mesmo lifetime de `'a`. 
Na pratica, o lifetime do retorno é o mesmo do menos dos lifetimes da referencias do passados como parametros.

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

Este exemplo é dado como valido pelo borrow checker, pois `result` é vivo enquanto o menor lifetime `string2` é vivo.

Se por exemplo, a funcao sempre retornasse o primeiro paramtro sempre, nao precisariamos colocar lifeime nos dois parametros.

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

## Lifetime em structs

Structs podem ter referencias, mas nesse caso precisamos adicionar a anotacao de lifetime nas definicao da struct.

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

A anotacao está em resumo dizendo que `ImportantExcerpt` nao pode viver mais do que o campo `part`.

Na hora da utilizacao na main o borrow checker ve que está tudo ok, pois o `novel` nao sao de escopo antes do `ImportantExcerpt`.

## Lifetime Elision

3 Regras para do rust para inferir automaticamente o lifetime na compiliacao

1 - The first rule is that the compiler assigns a lifetime parameter to each parameter that’s a reference. In other words, a function with one parameter gets one lifetime parameter: fn foo<'a>(x: &'a i32); a function with two parameters gets two separate lifetime parameters: fn foo<'a, 'b>(x: &'a i32, y: &'b i32); and so on.

2 - The second rule is that, if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: fn foo<'a>(x: &'a i32) -> &'a i32.

3 - The third rule is that, if there are multiple input lifetime parameters, but one of them is &self or &mut self because this is a method, the lifetime of self is assigned to all output lifetime parameters. This third rule makes methods much nicer to read and write because fewer symbols are necessary.

## Lifetime em metodos

É necessaria a anotacao depois do `impl` e do nome do tipo.

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

Mas nesse caso nao é necessario na assinatura, a primeira regra do lifetime elision resolve

Outro exemplo que é terceira regra do lifetime elision resolve

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```
A primeira regra do lifetime elision, da `&self` e `announcement` diferente lifetimes. Entao porque um dos parametros é `&self`, o retorno terá mesmo lifetime.

## Static lifetime

`'static` é uma anotacao que diz que a referencia vai existir durante todo o programa.

Toda string literal (str definida estaticamente no codigo fonte por exemplo), vai ter lifetime `'static`.

Exemplo:

```rust
let s: &'static str = "I have a static lifetime.";
```



## Generic Type Parameters, Trait Bounds, and Lifetimes Together

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```