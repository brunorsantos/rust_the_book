Muitas das operacoes suportadas por Vec<T> tambem sao suportadas por String. Por String é um wrapper de Vec com extra funcionalidade. ::new() é um exemplo.

## Concatenar

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {s2}");
```

`push` aceita apenas um caracter

```rust
let mut s = String::from("lo");
s.push('l');
```

O operador + utiliza `fn add(self, s: &str) -> String {` em que o self é movido e criado uma nova instancia de String.

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

Ou pode se usar `format!` 

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");
println!("{}",s);
println!("{}",s1);
```

Para indexar String precisamos usar slices e sempre com ranges:

```rust
let s = &hello[0..4];
```

Em que em tempo de execucao será verificado se o formato passando suporta quebrar a string no range passado.

Pode se iterar usando o metodo `chars()`.